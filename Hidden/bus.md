# Odoo Module: bus

Category: Hidden

This file contains the source code of the Odoo module.

## File: websocket.py

```python
import base64
import functools
import hashlib
import json
import logging
import os
import psycopg2
import random
import socket
import struct
import selectors
import threading
import time
from collections import defaultdict, deque
from contextlib import closing, suppress
from enum import IntEnum
from psycopg2.pool import PoolError
from urllib.parse import urlparse
from weakref import WeakSet

from werkzeug.local import LocalStack
from werkzeug.exceptions import BadRequest, HTTPException, ServiceUnavailable

import odoo
from odoo import api
from .models.bus import dispatch
from odoo.http import root, Request, Response, SessionExpiredException, get_default_session
from odoo.modules.registry import Registry
from odoo.service import model as service_model
from odoo.service.server import CommonServer
from odoo.service.security import check_session
from odoo.tools import config

_logger = logging.getLogger(__name__)


MAX_TRY_ON_POOL_ERROR = 10
DELAY_ON_POOL_ERROR = 0.03


def acquire_cursor(db):
    """ Try to acquire a cursor up to `MAX_TRY_ON_POOL_ERROR` """
    for tryno in range(1, MAX_TRY_ON_POOL_ERROR + 1):
        with suppress(PoolError):
            return odoo.registry(db).cursor()
        time.sleep(random.uniform(DELAY_ON_POOL_ERROR, DELAY_ON_POOL_ERROR * tryno))
    raise PoolError('Failed to acquire cursor after %s retries' % MAX_TRY_ON_POOL_ERROR)


# ------------------------------------------------------
# EXCEPTIONS
# ------------------------------------------------------

class UpgradeRequired(HTTPException):
    code = 426
    description = "Wrong websocket version was given during the handshake"

    def get_headers(self, environ=None):
        headers = super().get_headers(environ)
        headers.append((
            'Sec-WebSocket-Version',
            '; '.join(WebsocketConnectionHandler.SUPPORTED_VERSIONS)
        ))
        return headers


class WebsocketException(Exception):
    """ Base class for all websockets exceptions """


class ConnectionClosed(WebsocketException):
    """
    Raised when the other end closes the socket without performing
    the closing handshake.
    """


class InvalidCloseCodeException(WebsocketException):
    def __init__(self, code):
        super().__init__(f"Invalid close code: {code}")


class InvalidDatabaseException(WebsocketException):
    """
    When raised: the database probably does not exists anymore, the
    database is corrupted or the database version doesn't match the
    server version.
    """


class InvalidStateException(WebsocketException):
    """
    Raised when an operation is forbidden in the current state.
    """


class InvalidWebsocketRequest(WebsocketException):
    """
    Raised when a websocket request is invalid (format, wrong args).
    """


class PayloadTooLargeException(WebsocketException):
    """
    Raised when a websocket message is too large.
    """


class ProtocolError(WebsocketException):
    """
    Raised when a frame format doesn't match expectations.
    """


class RateLimitExceededException(Exception):
    """
    Raised when a client exceeds the number of request in a given
    time.
    """


# ------------------------------------------------------
# WEBSOCKET LIFECYCLE
# ------------------------------------------------------


class LifecycleEvent(IntEnum):
    OPEN = 0
    CLOSE = 1


# ------------------------------------------------------
# WEBSOCKET
# ------------------------------------------------------


class Opcode(IntEnum):
    CONTINUE = 0x00
    TEXT = 0x01
    BINARY = 0x02
    CLOSE = 0x08
    PING = 0x09
    PONG = 0x0A


class CloseCode(IntEnum):
    CLEAN = 1000
    GOING_AWAY = 1001
    PROTOCOL_ERROR = 1002
    INCORRECT_DATA = 1003
    ABNORMAL_CLOSURE = 1006
    INCONSISTENT_DATA = 1007
    MESSAGE_VIOLATING_POLICY = 1008
    MESSAGE_TOO_BIG = 1009
    EXTENSION_NEGOTIATION_FAILED = 1010
    SERVER_ERROR = 1011
    RESTART = 1012
    TRY_LATER = 1013
    BAD_GATEWAY = 1014
    SESSION_EXPIRED = 4001
    KEEP_ALIVE_TIMEOUT = 4002


class ConnectionState(IntEnum):
    OPEN = 0
    CLOSING = 1
    CLOSED = 2


DATA_OP = {Opcode.TEXT, Opcode.BINARY}
CTRL_OP = {Opcode.CLOSE, Opcode.PING, Opcode.PONG}
HEARTBEAT_OP = {Opcode.PING, Opcode.PONG}

VALID_CLOSE_CODES = {
    code for code in CloseCode if code is not CloseCode.ABNORMAL_CLOSURE
}
CLEAN_CLOSE_CODES = {CloseCode.CLEAN, CloseCode.GOING_AWAY, CloseCode.RESTART}
RESERVED_CLOSE_CODES = range(3000, 5000)

_XOR_TABLE = [bytes(a ^ b for a in range(256)) for b in range(256)]


class Frame:
    def __init__(
        self,
        opcode,
        payload=b'',
        fin=True,
        rsv1=False,
        rsv2=False,
        rsv3=False
    ):
        self.opcode = opcode
        self.payload = payload
        self.fin = fin
        self.rsv1 = rsv1
        self.rsv2 = rsv2
        self.rsv3 = rsv3


class CloseFrame(Frame):
    def __init__(self, code, reason):
        if code not in VALID_CLOSE_CODES and code not in RESERVED_CLOSE_CODES:
            raise InvalidCloseCodeException(code)
        payload = struct.pack('!H', code)
        if reason:
            payload += reason.encode('utf-8')
        self.code = code
        self.reason = reason
        super().__init__(Opcode.CLOSE, payload)


_websocket_instances = WeakSet()


class Websocket:
    __event_callbacks = defaultdict(set)
    # Maximum size for a message in bytes, whether it is sent as one
    # frame or many fragmented ones.
    MESSAGE_MAX_SIZE = 2 ** 20
    # Proxies usually close a connection after 1 minute of inactivity.
    # Therefore, a PING frame have to be sent if no frame is either sent
    # or received within CONNECTION_TIMEOUT - 15 seconds.
    CONNECTION_TIMEOUT = 60
    INACTIVITY_TIMEOUT = CONNECTION_TIMEOUT - 15
    # How many requests can be made in excess of the given rate.
    RL_BURST = int(config['websocket_rate_limit_burst'])
    # How many seconds between each request.
    RL_DELAY = float(config['websocket_rate_limit_delay'])

    def __init__(self, sock, session):
        # Session linked to the current websocket connection.
        self._session = session
        self._db = session.db
        self.__socket = sock
        self._close_sent = False
        self._close_received = False
        self._timeout_manager = TimeoutManager()
        # Used for rate limiting.
        self._incoming_frame_timestamps = deque(maxlen=self.RL_BURST)
        # Used to notify the websocket that bus notifications are
        # available.
        self.__notif_sock_w, self.__notif_sock_r = socket.socketpair()
        self._channels = set()
        self._last_notif_sent_id = 0
        # Websocket start up
        self.__selector = selectors.DefaultSelector()
        self.__selector.register(self.__socket, selectors.EVENT_READ)
        self.__selector.register(self.__notif_sock_r, selectors.EVENT_READ)
        self.state = ConnectionState.OPEN
        _websocket_instances.add(self)
        self._trigger_lifecycle_event(LifecycleEvent.OPEN)

    # ------------------------------------------------------
    # PUBLIC METHODS
    # ------------------------------------------------------

    def get_messages(self):
        while self.state is not ConnectionState.CLOSED:
            try:
                readables = {
                    selector_key[0].fileobj for selector_key in
                    self.__selector.select(self.INACTIVITY_TIMEOUT)
                }
                if self._timeout_manager.has_timed_out() and self.state is ConnectionState.OPEN:
                    self.disconnect(
                        CloseCode.ABNORMAL_CLOSURE
                        if self._timeout_manager.timeout_reason is TimeoutReason.NO_RESPONSE
                        else CloseCode.KEEP_ALIVE_TIMEOUT
                    )
                    continue
                if not readables:
                    self._send_ping_frame()
                    continue
                if self.__notif_sock_r in readables:
                    self._dispatch_bus_notifications()
                if self.__socket in readables:
                    message = self._process_next_message()
                    if message is not None:
                        yield message
            except Exception as exc:
                self._handle_transport_error(exc)

    def disconnect(self, code, reason=None):
        """
        Initiate the closing handshake that is, send a close frame
        to the other end which will then send us back an
        acknowledgment. Upon the reception of this acknowledgment,
        the `_terminate` method will be called to perform an
        orderly shutdown. Note that we don't need to wait for the
        acknowledgment if the connection was failed beforewards.
        """
        if code is not CloseCode.ABNORMAL_CLOSURE:
            self._send_close_frame(code, reason)
        else:
            self._terminate()

    @classmethod
    def onopen(cls, func):
        cls.__event_callbacks[LifecycleEvent.OPEN].add(func)
        return func

    @classmethod
    def onclose(cls, func):
        cls.__event_callbacks[LifecycleEvent.CLOSE].add(func)
        return func

    def subscribe(self, channels, last):
        """ Subscribe to bus channels. """
        self._channels = channels
        if self._last_notif_sent_id < last:
            self._last_notif_sent_id = last
        # Dispatch past notifications if there are any.
        self.trigger_notification_dispatching()

    def trigger_notification_dispatching(self):
        """
        Warn the socket that notifications are available. Ignore if a
        dispatch is already planned or if the socket is already in the
        closing state.
        """
        if self.state is not ConnectionState.OPEN:
            return
        readables = {
            selector_key[0].fileobj for selector_key in
            self.__selector.select(0)
        }
        if self.__notif_sock_r not in readables:
            # Send a random bit to mark the socket as readable.
            self.__notif_sock_w.send(b'x')

    # ------------------------------------------------------
    # PRIVATE METHODS
    # ------------------------------------------------------

    def _get_next_frame(self):
        #     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
        #    +-+-+-+-+-------+-+-------------+-------------------------------+
        #    |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
        #    |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
        #    |N|V|V|V|       |S|             |   (if payload len==126/127)   |
        #    | |1|2|3|       |K|             |                               |
        #    +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
        #    |     Extended payload length continued, if payload len == 127  |
        #    + - - - - - - - - - - - - - - - +-------------------------------+
        #    |                               |Masking-key, if MASK set to 1  |
        #    +-------------------------------+-------------------------------+
        #    | Masking-key (continued)       |          Payload Data         |
        #    +-------------------------------- - - - - - - - - - - - - - - - +
        #    :                     Payload Data continued ...                :
        #    + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
        #    |                     Payload Data continued ...                |
        #    +---------------------------------------------------------------+
        def recv_bytes(n):
            """ Pull n bytes from the socket """
            data = bytearray()
            while len(data) < n:
                received_data = self.__socket.recv(n - len(data))
                if not received_data:
                    raise ConnectionClosed()
                data.extend(received_data)
            return data

        def is_bit_set(byte, n):
            """
            Check whether nth bit of byte is set or not (from left
            to right).
             """
            return byte & (1 << (7 - n))

        def apply_mask(payload, mask):
            # see: https://www.willmcgugan.com/blog/tech/post/speeding-up-websockets-60x/
            a, b, c, d = (_XOR_TABLE[n] for n in mask)
            payload[::4] = payload[::4].translate(a)
            payload[1::4] = payload[1::4].translate(b)
            payload[2::4] = payload[2::4].translate(c)
            payload[3::4] = payload[3::4].translate(d)
            return payload

        self._limit_rate()
        first_byte, second_byte = recv_bytes(2)
        fin, rsv1, rsv2, rsv3 = (is_bit_set(first_byte, n) for n in range(4))
        try:
            opcode = Opcode(first_byte & 0b00001111)
        except ValueError as exc:
            raise ProtocolError(exc)
        payload_length = second_byte & 0b01111111

        if rsv1 or rsv2 or rsv3:
            raise ProtocolError("Reserved bits must be unset")
        if not is_bit_set(second_byte, 0):
            raise ProtocolError("Frame must be masked")
        if opcode in CTRL_OP:
            if not fin:
                raise ProtocolError("Control frames cannot be fragmented")
            if payload_length > 125:
                raise ProtocolError(
                    "Control frames payload must be smaller than 126"
                )
        if payload_length == 126:
            payload_length = struct.unpack('!H', recv_bytes(2))[0]
        elif payload_length == 127:
            payload_length = struct.unpack('!Q', recv_bytes(8))[0]
        if payload_length > self.MESSAGE_MAX_SIZE:
            raise PayloadTooLargeException()

        mask = recv_bytes(4)
        payload = apply_mask(recv_bytes(payload_length), mask)
        frame = Frame(opcode, bytes(payload), fin, rsv1, rsv2, rsv3)
        self._timeout_manager.acknowledge_frame_receipt(frame)
        return frame

    def _process_next_message(self):
        """
        Process the next message coming throught the socket. If a
        data message can be extracted, return its decoded payload.
        As per the RFC, only control frames will be processed once
        the connection reaches the closing state.
        """
        frame = self._get_next_frame()
        if frame.opcode in CTRL_OP:
            return self._handle_control_frame(frame)
        if self.state is not ConnectionState.OPEN:
            # After receiving a control frame indicating the connection
            # should be closed, a peer discards any further data
            # received.
            return
        if frame.opcode is Opcode.CONTINUE:
            raise ProtocolError("Unexpected continuation frame")
        message = frame.payload
        if not frame.fin:
            message = self._recover_fragmented_message(frame)
        return (
            message.decode('utf-8')
            if message is not None and frame.opcode is Opcode.TEXT else message
        )

    def _recover_fragmented_message(self, initial_frame):
        message_fragments = bytearray(initial_frame.payload)
        while True:
            frame = self._get_next_frame()
            if frame.opcode in CTRL_OP:
                # Control frames can be received in the middle of a
                # fragmented message, process them as soon as possible.
                self._handle_control_frame(frame)
                if self.state is not ConnectionState.OPEN:
                    return
                continue
            if frame.opcode is not Opcode.CONTINUE:
                raise ProtocolError("A continuation frame was expected")
            message_fragments.extend(frame.payload)
            if len(message_fragments) > self.MESSAGE_MAX_SIZE:
                raise PayloadTooLargeException()
            if frame.fin:
                return bytes(message_fragments)

    def _send(self, message):
        if self.state is not ConnectionState.OPEN:
            raise InvalidStateException(
                "Trying to send a frame on a closed socket"
            )
        opcode = Opcode.BINARY
        if not isinstance(message, (bytes, bytearray)):
            opcode = Opcode.TEXT
        self._send_frame(Frame(opcode, message))

    def _send_frame(self, frame):
        if frame.opcode in CTRL_OP and len(frame.payload) > 125:
            raise ProtocolError(
                "Control frames should have a payload length smaller than 126"
            )
        if isinstance(frame.payload, str):
            frame.payload = frame.payload.encode('utf-8')
        elif not isinstance(frame.payload, (bytes, bytearray)):
            frame.payload = json.dumps(frame.payload).encode('utf-8')

        output = bytearray()
        first_byte = (
              (0b10000000 if frame.fin else 0)
            | (0b01000000 if frame.rsv1 else 0)
            | (0b00100000 if frame.rsv2 else 0)
            | (0b00010000 if frame.rsv3 else 0)
            | frame.opcode
        )
        payload_length = len(frame.payload)
        if payload_length < 126:
            output.extend(
                struct.pack('!BB', first_byte, payload_length)
            )
        elif payload_length < 65536:
            output.extend(
                struct.pack('!BBH', first_byte, 126, payload_length)
            )
        else:
            output.extend(
                struct.pack('!BBQ', first_byte, 127, payload_length)
            )
        output.extend(frame.payload)
        self.__socket.sendall(output)
        self._timeout_manager.acknowledge_frame_sent(frame)
        if not isinstance(frame, CloseFrame):
            return
        self.state = ConnectionState.CLOSING
        self._close_sent = True
        if frame.code not in CLEAN_CLOSE_CODES or self._close_received:
            return self._terminate()
        # After sending a control frame indicating the connection
        # should be closed, a peer does not send any further data.
        self.__selector.unregister(self.__notif_sock_r)

    def _send_close_frame(self, code, reason=None):
        """ Send a close frame. """
        self._send_frame(CloseFrame(code, reason))

    def _send_ping_frame(self):
        """ Send a ping frame """
        self._send_frame(Frame(Opcode.PING))

    def _send_pong_frame(self, payload):
        """ Send a pong frame """
        self._send_frame(Frame(Opcode.PONG, payload))

    def _terminate(self):
        """ Close the underlying TCP socket. """
        with suppress(OSError, TimeoutError):
            self.__socket.shutdown(socket.SHUT_WR)
            # Call recv until obtaining a return value of 0 indicating
            # the other end has performed an orderly shutdown. A timeout
            # is set to ensure the connection will be closed even if
            # the other end does not close the socket properly.
            self.__socket.settimeout(1)
            while self.__socket.recv(4096):
                pass
        self.__selector.unregister(self.__socket)
        self.__selector.close()
        self.__socket.close()
        self.state = ConnectionState.CLOSED
        dispatch.unsubscribe(self)
        self._trigger_lifecycle_event(LifecycleEvent.CLOSE)

    def _handle_control_frame(self, frame):
        if frame.opcode is Opcode.PING:
            self._send_pong_frame(frame.payload)
        elif frame.opcode is Opcode.CLOSE:
            self.state = ConnectionState.CLOSING
            self._close_received = True
            code, reason = CloseCode.CLEAN, None
            if len(frame.payload) >= 2:
                code = struct.unpack('!H', frame.payload[:2])[0]
                reason = frame.payload[2:].decode('utf-8')
            elif frame.payload:
                raise ProtocolError("Malformed closing frame")
            if not self._close_sent:
                self._send_close_frame(code, reason)
            else:
                self._terminate()

    def _handle_transport_error(self, exc):
        """
        Find out which close code should be sent according to given
        exception and call `self.disconnect` in order to close the
        connection cleanly.
        """
        code, reason = CloseCode.SERVER_ERROR, str(exc)
        if isinstance(exc, (ConnectionClosed, OSError)):
            code = CloseCode.ABNORMAL_CLOSURE
        elif isinstance(exc, (ProtocolError, InvalidCloseCodeException)):
            code = CloseCode.PROTOCOL_ERROR
        elif isinstance(exc, UnicodeDecodeError):
            code = CloseCode.INCONSISTENT_DATA
        elif isinstance(exc, PayloadTooLargeException):
            code = CloseCode.MESSAGE_TOO_BIG
        elif isinstance(exc, (PoolError, RateLimitExceededException)):
            code = CloseCode.TRY_LATER
        elif isinstance(exc, SessionExpiredException):
            code = CloseCode.SESSION_EXPIRED
        if code is CloseCode.SERVER_ERROR:
            reason = None
            _logger.error(exc, exc_info=True)
        self.disconnect(code, reason)

    def _limit_rate(self):
        """
        This method is a simple rate limiter designed not to allow
        more than one request by `RL_DELAY` seconds. `RL_BURST` specify
        how many requests can be made in excess of the given rate at the
        begining. When requests are received too fast, raises the
        `RateLimitExceededException`.
        """
        now = time.time()
        if len(self._incoming_frame_timestamps) >= self.RL_BURST:
            elapsed_time = now - self._incoming_frame_timestamps[0]
            if elapsed_time < self.RL_DELAY * self.RL_BURST:
                raise RateLimitExceededException()
        self._incoming_frame_timestamps.append(now)

    def _trigger_lifecycle_event(self, event_type):
        """
        Trigger a lifecycle event that is, call every function
        registered for this event type. Every callback is given both the
        environment and the related websocket.
        """
        if not self.__event_callbacks[event_type]:
            return
        with closing(acquire_cursor(self._db)) as cr:
            env = api.Environment(cr, self._session.uid, self._session.context)
            for callback in self.__event_callbacks[event_type]:
                try:
                    service_model.retrying(functools.partial(callback, env, self), env)
                except Exception:
                    _logger.warning(
                        'Error during Websocket %s callback',
                        LifecycleEvent(event_type).name,
                        exc_info=True
                    )

    def _dispatch_bus_notifications(self):
        """
        Dispatch notifications related to the registered channels. If
        the session is expired, close the connection with the
        `SESSION_EXPIRED` close code. If no cursor can be acquired,
        close the connection with the `TRY_LATER` close code.
        """
        session = root.session_store.get(self._session.sid)
        if not session:
            raise SessionExpiredException()
        with acquire_cursor(session.db) as cr:
            env = api.Environment(cr, session.uid, session.context)
            if session.uid is not None and not check_session(session, env):
                raise SessionExpiredException()
            # Mark the notification request as processed.
            self.__notif_sock_r.recv(1)
            notifications = env['bus.bus']._poll(self._channels, self._last_notif_sent_id)
        if not notifications:
            return
        self._last_notif_sent_id = notifications[-1]['id']
        self._send(notifications)


class TimeoutReason(IntEnum):
    KEEP_ALIVE = 0
    NO_RESPONSE = 1


class TimeoutManager:
    """
    This class handles the Websocket timeouts. If no response to a
    PING/CLOSE frame is received after `TIMEOUT` seconds or if the
    connection is opened for more than `self._keep_alive_timeout` seconds,
    the connection is considered to have timed out. To determine if the
    connection has timed out, use the `has_timed_out` method.
    """
    TIMEOUT = 15
    # Timeout specifying how many seconds the connection should be kept
    # alive.
    KEEP_ALIVE_TIMEOUT = int(config['websocket_keep_alive_timeout'])

    def __init__(self):
        super().__init__()
        self._awaited_opcode = None
        # Time in which the connection was opened.
        self._opened_at = time.time()
        # Custom keep alive timeout for each TimeoutManager to avoid multiple
        # connections timing out at the same time.
        self._keep_alive_timeout = (
            self.KEEP_ALIVE_TIMEOUT + random.uniform(0, self.KEEP_ALIVE_TIMEOUT / 2)
        )
        self.timeout_reason = None
        # Start time recorded when we started awaiting an answer to a
        # PING/CLOSE frame.
        self._waiting_start_time = None

    def acknowledge_frame_receipt(self, frame):
        if self._awaited_opcode is frame.opcode:
            self._awaited_opcode = None
            self._waiting_start_time = None

    def acknowledge_frame_sent(self, frame):
        """
        Acknowledge a frame was sent. If this frame is a PING/CLOSE
        frame, start waiting for an answer.
        """
        if self.has_timed_out():
            return
        if frame.opcode is Opcode.PING:
            self._awaited_opcode = Opcode.PONG
        elif frame.opcode is Opcode.CLOSE:
            self._awaited_opcode = Opcode.CLOSE
        if self._awaited_opcode is not None:
            self._waiting_start_time = time.time()

    def has_timed_out(self):
        """
        Determine whether the connection has timed out or not. The
        connection times out when the answer to a CLOSE/PING frame
        is not received within `TIMEOUT` seconds or if the connection
        is opened for more than `self._keep_alive_timeout` seconds.
        """
        now = time.time()
        if now - self._opened_at >= self._keep_alive_timeout:
            self.timeout_reason = TimeoutReason.KEEP_ALIVE
            return True
        if self._awaited_opcode and now - self._waiting_start_time >= self.TIMEOUT:
            self.timeout_reason = TimeoutReason.NO_RESPONSE
            return True
        return False


# ------------------------------------------------------
# WEBSOCKET SERVING
# ------------------------------------------------------


_wsrequest_stack = LocalStack()
wsrequest = _wsrequest_stack()

class WebsocketRequest:
    def __init__(self, db, httprequest, websocket):
        self.db = db
        self.httprequest = httprequest
        self.session = None
        self.ws = websocket

    def __enter__(self):
        _wsrequest_stack.push(self)
        return self

    def __exit__(self, *args):
        _wsrequest_stack.pop()

    def serve_websocket_message(self, message):
        try:
            jsonrequest = json.loads(message)
            event_name = jsonrequest['event_name']  # mandatory
        except KeyError as exc:
            raise InvalidWebsocketRequest(
                f'Key {exc.args[0]!r} is missing from request'
            ) from exc
        except ValueError as exc:
            raise InvalidWebsocketRequest(
                f'Invalid JSON data, {exc.args[0]}'
            ) from exc
        data = jsonrequest.get('data')
        self.session = self._get_session()

        try:
            self.registry = Registry(self.db)
            self.registry.check_signaling()
            threading.current_thread().dbname = self.registry.db_name
        except (
            AttributeError, psycopg2.OperationalError, psycopg2.ProgrammingError
        ) as exc:
            raise InvalidDatabaseException() from exc

        with closing(acquire_cursor(self.db)) as cr:
            self.env = api.Environment(cr, self.session.uid, self.session.context)
            threading.current_thread().uid = self.env.uid
            service_model.retrying(
                functools.partial(self._serve_ir_websocket, event_name, data),
                self.env,
            )

    def _serve_ir_websocket(self, event_name, data):
        """
        Delegate most of the processing to the ir.websocket model
        which is extensible by applications. Directly call the
        appropriate ir.websocket method since only two events are
        tolerated: `subscribe` and `update_presence`.
        """
        ir_websocket = self.env['ir.websocket']
        ir_websocket._authenticate()
        if event_name == 'subscribe':
            ir_websocket._subscribe(data)
        if event_name == 'update_presence':
            ir_websocket._update_bus_presence(**data)

    def _get_session(self):
        session = root.session_store.get(self.ws._session.sid)
        if not session:
            raise SessionExpiredException()
        return session

    def update_env(self, user=None, context=None, su=None):
        """
        Update the environment of the current websocket request.
        """
        Request.update_env(self, user, context, su)


class WebsocketConnectionHandler:
    SUPPORTED_VERSIONS = {'13'}
    # Given by the RFC in order to generate Sec-WebSocket-Accept from
    # Sec-WebSocket-Key value.
    _HANDSHAKE_GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
    _REQUIRED_HANDSHAKE_HEADERS = {
        'connection', 'host', 'sec-websocket-key',
        'sec-websocket-version', 'upgrade', 'origin',
    }

    @classmethod
    def websocket_allowed(cls, request):
        return not request.registry.in_test_mode()

    @classmethod
    def open_connection(cls, request):
        """
        Open a websocket connection if the handshake is successfull.
        :return: Response indicating the server performed a connection
        upgrade.
        :raise: UpgradeRequired if there is no intersection between the
        versions the client supports and those we support.
        :raise: BadRequest if the handshake data is incorrect.
        """
        if not cls.websocket_allowed(request):
            raise ServiceUnavailable("Websocket is disabled in test mode")
        public_session = cls._handle_public_configuration(request)
        try:
            response = cls._get_handshake_response(request.httprequest.headers)
            socket = request.httprequest._HTTPRequest__environ['socket']
            session, db, httprequest = (public_session or request.session), request.db, request.httprequest
            response.call_on_close(lambda: cls._serve_forever(
                Websocket(socket, session),
                db,
                httprequest,
            ))
            # Force save the session. Session must be persisted to handle
            # WebSocket authentication.
            request.session.is_dirty = True
            return response
        except KeyError as exc:
            raise RuntimeError(
                f"Couldn't bind the websocket. Is the connection opened on the evented port ({config['gevent_port']})?"
            ) from exc
        except HTTPException as exc:
            # The HTTP stack does not log exceptions derivated from the
            # HTTPException class since they are valid responses.
            _logger.error(exc)
            raise



    @classmethod
    def _get_handshake_response(cls, headers):
        """
        :return: Response indicating the server performed a connection
        upgrade.
        :raise: BadRequest
        :raise: UpgradeRequired
        """
        cls._assert_handshake_validity(headers)
        # sha-1 is used as it is required by
        # https://datatracker.ietf.org/doc/html/rfc6455#page-7
        accept_header = hashlib.sha1(
            (headers['sec-websocket-key'] + cls._HANDSHAKE_GUID).encode()).digest()
        accept_header = base64.b64encode(accept_header)
        return Response(status=101, headers={
            'Upgrade': 'websocket',
            'Connection': 'Upgrade',
            'Sec-WebSocket-Accept': accept_header.decode(),
        })

    @classmethod
    def _handle_public_configuration(cls, request):
        if not os.getenv('ODOO_BUS_PUBLIC_SAMESITE_WS'):
            return
        headers = request.httprequest.headers
        origin_url = urlparse(headers.get('origin'))
        if origin_url.netloc != headers.get('host') or origin_url.scheme != request.httprequest.scheme:
            session = root.session_store.new()
            session.update(get_default_session(), db=request.session.db)
            root.session_store.save(session)
            return session
        return None

    @classmethod
    def _assert_handshake_validity(cls, headers):
        """
        :raise: UpgradeRequired if there is no intersection between
        the version the client supports and those we support.
        :raise: BadRequest in case of invalid handshake.
        """
        missing_or_empty_headers = {
            header for header in cls._REQUIRED_HANDSHAKE_HEADERS
            if header not in headers
        }
        if missing_or_empty_headers:
            raise BadRequest(
                f"""Empty or missing header(s): {', '.join(missing_or_empty_headers)}"""
            )

        if headers['upgrade'].lower() != 'websocket':
            raise BadRequest('Invalid upgrade header')
        if 'upgrade' not in headers['connection'].lower():
            raise BadRequest('Invalid connection header')
        if headers['sec-websocket-version'] not in cls.SUPPORTED_VERSIONS:
            raise UpgradeRequired()

        key = headers['sec-websocket-key']
        try:
            decoded_key = base64.b64decode(key, validate=True)
        except ValueError:
            raise BadRequest("Sec-WebSocket-Key should be b64 encoded")
        if len(decoded_key) != 16:
            raise BadRequest(
                "Sec-WebSocket-Key should be of length 16 once decoded"
            )

    @classmethod
    def _serve_forever(cls, websocket, db, httprequest):
        """
        Process incoming messages and dispatch them to the application.
        """
        current_thread = threading.current_thread()
        current_thread.type = 'websocket'
        for message in websocket.get_messages():
            with WebsocketRequest(db, httprequest, websocket) as req:
                try:
                    req.serve_websocket_message(message)
                except SessionExpiredException:
                    websocket.disconnect(CloseCode.SESSION_EXPIRED)
                except PoolError:
                    websocket.disconnect(CloseCode.TRY_LATER)
                except Exception:
                    _logger.exception("Exception occurred during websocket request handling")


def _kick_all():
    """ Disconnect all the websocket instances. """
    for websocket in _websocket_instances:
        if websocket.state is ConnectionState.OPEN:
            websocket.disconnect(CloseCode.GOING_AWAY)


CommonServer.on_stop(_kick_all)

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import models
from . import controllers
from . import websocket

```

## File: __manifest__.py

```python
{
    'name' : 'IM Bus',
    'version': '1.0',
    'category': 'Hidden',
    'description': "Instant Messaging Bus allow you to send messages to users, in live.",
    'depends': ['base', 'web'],
    'data': [
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_common': [
            'bus/static/src/*.js',
            'bus/static/src/services/**/*.js',
            'bus/static/src/workers/websocket_worker.js',
            'bus/static/src/workers/websocket_worker_utils.js',
        ],
        'web.assets_frontend': [
            'bus/static/src/*.js',
            'bus/static/src/services/**/*.js',
            ('remove', 'bus/static/src/services/assets_watchdog_service.js'),
            'bus/static/src/workers/websocket_worker.js',
            'bus/static/src/workers/websocket_worker_utils.js',
        ],
        'web.qunit_suite_tests': [
            'bus/static/tests/**/*.js',
        ],
        'web.qunit_mobile_suite_tests': [
            'bus/static/tests/helpers/**/*.js',
        ],
        'bus.websocket_worker_assets': [
            'web/static/src/legacy/js/promise_extension.js',
            'web/static/src/boot.js',
            'bus/static/src/workers/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo.http import Controller, request, route


class BusController(Controller):
    @route('/bus/get_model_definitions', methods=['POST'], type='http', auth='user')
    def get_model_definitions(self, model_names_to_fetch, **kwargs):
        return request.make_response(json.dumps(
            request.env['ir.model']._get_model_definitions(json.loads(model_names_to_fetch)),
        ))

```

## File: controllers\websocket.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo.http import Controller, request, route, SessionExpiredException
from odoo.addons.base.models.assetsbundle import AssetsBundle
from ..models.bus import channel_with_db
from ..websocket import WebsocketConnectionHandler


class WebsocketController(Controller):
    @route('/websocket', type="http", auth="public", cors='*', websocket=True)
    def websocket(self):
        """
        Handle the websocket handshake, upgrade the connection if
        successfull.
        """
        return WebsocketConnectionHandler.open_connection(request)

    @route('/websocket/health', type='http', auth='none', save_session=False)
    def health(self):
        data = json.dumps({
            'status': 'pass',
        })
        headers = [('Content-Type', 'application/json'),
                   ('Cache-Control', 'no-store')]
        return request.make_response(data, headers)

    @route('/websocket/peek_notifications', type='json', auth='public', cors='*')
    def peek_notifications(self, channels, last, is_first_poll=False):
        if not all(isinstance(c, str) for c in channels):
            raise ValueError("bus.Bus only string channels are allowed.")
        if is_first_poll:
            # Used to detect when the current session is expired.
            request.session['is_websocket_session'] = True
        elif 'is_websocket_session' not in request.session:
            raise SessionExpiredException()
        channels = list(set(
            channel_with_db(request.db, c)
            for c in request.env['ir.websocket']._build_bus_channel_list(channels)
        ))
        last_known_notification_id = request.env['bus.bus'].sudo().search([], limit=1, order='id desc').id or 0
        if last > last_known_notification_id:
            last = 0
        notifications = request.env['bus.bus']._poll(channels, last)
        return {'channels': channels, 'notifications': notifications}

    @route('/websocket/update_bus_presence', type='json', auth='public', cors='*')
    def update_bus_presence(self, inactivity_period, im_status_ids_by_model):
        if 'is_websocket_session' not in request.session:
            raise SessionExpiredException()
        request.env['ir.websocket']._update_bus_presence(int(inactivity_period), im_status_ids_by_model)
        return {}

    @route('/bus/websocket_worker_bundle', type='http', auth='public', cors='*')
    def get_websocket_worker_bundle(self, v=None):  # pylint: disable=unused-argument
        """
        :param str v: Version of the worker, frontend only argument used to
            prevent new worker versions to be loaded from the browser cache.
        """
        bundle = 'bus.websocket_worker_assets'
        files, _ = request.env["ir.qweb"]._get_asset_content(bundle)
        asset = AssetsBundle(bundle, files)
        stream = request.env['ir.binary']._get_stream_from(asset.js(
            is_minified="assets" not in request.session.debug
        ))
        return stream.get_response(content_security_policy=None)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main
from . import websocket

```

## File: models\bus.py

```python
# -*- coding: utf-8 -*-
import contextlib
import datetime
import json
import logging
import math
import os
import random
import selectors
import threading
import time
from psycopg2 import InterfaceError, sql

import odoo
from odoo import api, fields, models
from odoo.service.server import CommonServer
from odoo.tools.misc import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools import date_utils

_logger = logging.getLogger(__name__)

# longpolling timeout connection
TIMEOUT = 50

# custom function to call instead of NOTIFY postgresql command (opt-in)
ODOO_NOTIFY_FUNCTION = os.environ.get('ODOO_NOTIFY_FUNCTION')


def get_notify_payload_max_length(default=8000):
    try:
        length = int(os.environ.get('ODOO_NOTIFY_PAYLOAD_MAX_LENGTH', default))
    except ValueError:
        _logger.warning("ODOO_NOTIFY_PAYLOAD_MAX_LENGTH has to be an integer, "
                        "defaulting to %d bytes", default)
        length = default
    return length


# max length in bytes for the NOTIFY query payload
NOTIFY_PAYLOAD_MAX_LENGTH = get_notify_payload_max_length()


#----------------------------------------------------------
# Bus
#----------------------------------------------------------
def json_dump(v):
    return json.dumps(v, separators=(',', ':'), default=date_utils.json_default)

def hashable(key):
    if isinstance(key, list):
        key = tuple(key)
    return key


def channel_with_db(dbname, channel):
    if isinstance(channel, models.Model):
        return (dbname, channel._name, channel.id)
    if isinstance(channel, str):
        return (dbname, channel)
    return channel


def get_notify_payloads(channels):
    """
    Generates the json payloads for the imbus NOTIFY.
    Splits recursively payloads that are too large.

    :param list channels:
    :return: list of payloads of json dumps
    :rtype: list[str]
    """
    if not channels:
        return []
    payload = json_dump(channels)
    if len(channels) == 1 or len(payload.encode()) < NOTIFY_PAYLOAD_MAX_LENGTH:
        return [payload]
    else:
        pivot = math.ceil(len(channels) / 2)
        return (get_notify_payloads(channels[:pivot]) +
                get_notify_payloads(channels[pivot:]))


class ImBus(models.Model):

    _name = 'bus.bus'
    _description = 'Communication Bus'

    channel = fields.Char('Channel')
    message = fields.Char('Message')

    @api.autovacuum
    def _gc_messages(self):
        timeout_ago = datetime.datetime.utcnow()-datetime.timedelta(seconds=TIMEOUT*2)
        domain = [('create_date', '<', timeout_ago.strftime(DEFAULT_SERVER_DATETIME_FORMAT))]
        records = self.search(domain, limit=models.GC_UNLINK_LIMIT)
        if len(records) >= models.GC_UNLINK_LIMIT:
            self.env.ref('base.autovacuum_job')._trigger()
        return records.unlink()

    @api.model
    def _sendmany(self, notifications):
        channels = set()
        values = []
        for target, notification_type, message in notifications:
            channel = channel_with_db(self.env.cr.dbname, target)
            channels.add(channel)
            values.append({
                'channel': json_dump(channel),
                'message': json_dump({
                    'type': notification_type,
                    'payload': message,
                })
            })
        self.sudo().create(values)
        if channels:
            # We have to wait until the notifications are commited in database.
            # When calling `NOTIFY imbus`, notifications will be fetched in the
            # bus table. If the transaction is not commited yet, there will be
            # nothing to fetch, and the websocket will return no notification.
            @self.env.cr.postcommit.add
            def notify():
                with odoo.sql_db.db_connect('postgres').cursor() as cr:
                    if ODOO_NOTIFY_FUNCTION:
                        query = sql.SQL("SELECT {}('imbus', %s)").format(sql.Identifier(ODOO_NOTIFY_FUNCTION))
                    else:
                        query = "NOTIFY imbus, %s"
                    payloads = get_notify_payloads(list(channels))
                    if len(payloads) > 1:
                        _logger.info("The imbus notification payload was too large, "
                                     "it's been split into %d payloads.", len(payloads))
                    for payload in payloads:
                        cr.execute(query, (payload,))

    @api.model
    def _sendone(self, channel, notification_type, message):
        self._sendmany([[channel, notification_type, message]])

    @api.model
    def _poll(self, channels, last=0):
        # first poll return the notification in the 'buffer'
        if last == 0:
            timeout_ago = datetime.datetime.utcnow()-datetime.timedelta(seconds=TIMEOUT)
            domain = [('create_date', '>', timeout_ago.strftime(DEFAULT_SERVER_DATETIME_FORMAT))]
        else:  # else returns the unread notifications
            domain = [('id', '>', last)]
        channels = [json_dump(channel_with_db(self.env.cr.dbname, c)) for c in channels]
        domain.append(('channel', 'in', channels))
        notifications = self.sudo().search_read(domain)
        # list of notification to return
        result = []
        for notif in notifications:
            result.append({
                'id': notif['id'],
                'message': json.loads(notif['message']),
            })
        return result


#----------------------------------------------------------
# Dispatcher
#----------------------------------------------------------

class BusSubscription:
    def __init__(self, channels, last):
        self.last_notification_id = last
        self.channels = channels


class ImDispatch(threading.Thread):
    def __init__(self):
        super().__init__(daemon=True, name=f'{__name__}.Bus')
        self._channels_to_ws = {}

    def subscribe(self, channels, last, db, websocket):
        """
        Subcribe to bus notifications. Every notification related to the
        given channels will be sent through the websocket. If a subscription
        is already present, overwrite it.
        """
        channels = {hashable(channel_with_db(db, c)) for c in channels}
        for channel in channels:
            self._channels_to_ws.setdefault(channel, set()).add(websocket)
        outdated_channels = websocket._channels - channels
        self._clear_outdated_channels(websocket, outdated_channels)
        websocket.subscribe(channels, last)
        with contextlib.suppress(RuntimeError):
            if not self.is_alive():
                self.start()

    def unsubscribe(self, websocket):
        self._clear_outdated_channels(websocket, websocket._channels)

    def _clear_outdated_channels(self, websocket, outdated_channels):
        """ Remove channels from channel to websocket map. """
        for channel in outdated_channels:
            self._channels_to_ws[channel].remove(websocket)
            if not self._channels_to_ws[channel]:
                self._channels_to_ws.pop(channel)

    def loop(self):
        """ Dispatch postgres notifications to the relevant websockets """
        _logger.info("Bus.loop listen imbus on db postgres")
        with odoo.sql_db.db_connect('postgres').cursor() as cr, \
             selectors.DefaultSelector() as sel:
            cr.execute("listen imbus")
            cr.commit()
            conn = cr._cnx
            sel.register(conn, selectors.EVENT_READ)
            while not stop_event.is_set():
                if sel.select(TIMEOUT):
                    conn.poll()
                    channels = []
                    while conn.notifies:
                        channels.extend(json.loads(conn.notifies.pop().payload))
                    # relay notifications to websockets that have
                    # subscribed to the corresponding channels.
                    websockets = set()
                    for channel in channels:
                        websockets.update(self._channels_to_ws.get(hashable(channel), []))
                    for websocket in websockets:
                        websocket.trigger_notification_dispatching()

    def run(self):
        while not stop_event.is_set():
            try:
                self.loop()
            except Exception as exc:
                if isinstance(exc, InterfaceError) and stop_event.is_set():
                    continue
                _logger.exception("Bus.loop error, sleep and retry")
                time.sleep(TIMEOUT)

# Partially undo a2ed3d3d5bdb6025a1ba14ad557a115a86413e65
# IMDispatch has a lazy start, so we could initialize it anyway
# And this avoids the Bus unavailable error messages
dispatch = ImDispatch()
stop_event = threading.Event()
CommonServer.on_stop(stop_event.set)

```

## File: models\bus_presence.py

```python
# -*- coding: utf-8 -*-
import datetime
import time

from psycopg2 import OperationalError

from odoo import api, fields, models
from odoo import tools
from odoo.service.model import PG_CONCURRENCY_ERRORS_TO_RETRY
from odoo.tools.misc import DEFAULT_SERVER_DATETIME_FORMAT

UPDATE_PRESENCE_DELAY = 60
DISCONNECTION_TIMER = UPDATE_PRESENCE_DELAY + 5
AWAY_TIMER = 1800  # 30 minutes


class BusPresence(models.Model):
    """ User Presence
        Its status is 'online', 'away' or 'offline'. This model should be a one2one, but is not
        attached to res_users to avoid database concurrence errors. Since the 'update' method is executed
        at each poll, if the user have multiple opened tabs, concurrence errors can happend, but are 'muted-logged'.
    """

    _name = 'bus.presence'
    _description = 'User Presence'
    _log_access = False

    user_id = fields.Many2one('res.users', 'Users', ondelete='cascade')
    last_poll = fields.Datetime('Last Poll', default=lambda self: fields.Datetime.now())
    last_presence = fields.Datetime('Last Presence', default=lambda self: fields.Datetime.now())
    status = fields.Selection([('online', 'Online'), ('away', 'Away'), ('offline', 'Offline')], 'IM Status', default='offline')

    def init(self):
        self.env.cr.execute("CREATE UNIQUE INDEX IF NOT EXISTS bus_presence_user_unique ON %s (user_id) WHERE user_id IS NOT NULL" % self._table)

    @api.model
    def update(self, inactivity_period, identity_field, identity_value):
        """ Updates the last_poll and last_presence of the current user
            :param inactivity_period: duration in milliseconds
        """
        # This method is called in method _poll() and cursor is closed right
        # after; see bus/controllers/main.py.
        try:
            # Hide transaction serialization errors, which can be ignored, the presence update is not essential
            # The errors are supposed from presence.write(...) call only
            with tools.mute_logger('odoo.sql_db'):
                self._update(inactivity_period=inactivity_period, identity_field=identity_field, identity_value=identity_value)
                # commit on success
                self.env.cr.commit()
        except OperationalError as e:
            if e.pgcode in PG_CONCURRENCY_ERRORS_TO_RETRY:
                # ignore concurrency error
                return self.env.cr.rollback()
            raise

    @api.model
    def _update(self, inactivity_period, identity_field, identity_value):
        presence = self.search([(identity_field, '=', identity_value)], limit=1)
        # compute last_presence timestamp
        last_presence = datetime.datetime.now() - datetime.timedelta(milliseconds=inactivity_period)
        values = {
            'last_poll': time.strftime(DEFAULT_SERVER_DATETIME_FORMAT),
        }
        # update the presence or a create a new one
        if not presence:  # create a new presence for the user
            values[identity_field] = identity_value
            values['last_presence'] = last_presence
            self.create(values)
        else:  # update the last_presence if necessary, and write values
            if presence.last_presence < last_presence:
                values['last_presence'] = last_presence
            presence.write(values)

```

## File: models\ir_model.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrModel(models.Model):
    _inherit = 'ir.model'

    def _get_model_definitions(self, model_names_to_fetch):
        fields_by_model_names = {}
        for model_name in model_names_to_fetch:
            model = self.env[model_name]
            # get fields, relational fields are kept only if the related model is in model_names_to_fetch
            fields_data_by_fname = {
                fname: field_data
                for fname, field_data in model.fields_get(
                    attributes=['name', 'type', 'relation', 'required', 'readonly', 'selection', 'string']
                ).items()
                if not field_data.get('relation') or field_data['relation'] in model_names_to_fetch
            }
            for fname, field_data in fields_data_by_fname.items():
                if fname in model._fields:
                    inverse_fields = [
                        field for field in model.pool.field_inverses[model._fields[fname]]
                        if field.model_name in model_names_to_fetch
                    ]
                    if inverse_fields:
                        field_data['inverse_fname_by_model_name'] = {field.model_name: field.name for field in inverse_fields}
                    if field_data['type'] == 'many2one_reference':
                        field_data['model_name_ref_fname'] = model._fields[fname].model_field
            fields_by_model_names[model_name] = fields_data_by_fname
        return fields_by_model_names

```

## File: models\ir_websocket.py

```python
from odoo import models
from odoo.http import SessionExpiredException
from odoo.service import security
from ..models.bus import dispatch
from ..websocket import wsrequest


class IrWebsocket(models.AbstractModel):
    _name = 'ir.websocket'
    _description = 'websocket message handling'

    def _get_im_status(self, im_status_ids_by_model):
        im_status = {}
        if 'res.partner' in im_status_ids_by_model:
            im_status['partners'] = self.env['res.partner'].with_context(active_test=False).search_read(
                [('id', 'in', im_status_ids_by_model['res.partner'])],
                ['im_status']
            )
        return im_status

    def _build_bus_channel_list(self, channels):
        """
            Return the list of channels to subscribe to. Override this
            method to add channels in addition to the ones the client
            sent.

            :param channels: The channel list sent by the client.
        """
        channels.append('broadcast')
        return channels

    def _subscribe(self, data):
        if not all(isinstance(c, str) for c in data['channels']):
            raise ValueError("bus.Bus only string channels are allowed.")
        last_known_notification_id = self.env['bus.bus'].sudo().search([], limit=1, order='id desc').id or 0
        if data['last'] > last_known_notification_id:
            data['last'] = 0
        channels = set(self._build_bus_channel_list(data['channels']))
        dispatch.subscribe(channels, data['last'], self.env.registry.db_name, wsrequest.ws)

    def _update_bus_presence(self, inactivity_period, im_status_ids_by_model):
        if self.env.user and not self.env.user._is_public():
            self.env['bus.presence'].update(
                inactivity_period,
                identity_field='user_id',
                identity_value=self.env.uid
            )
            im_status_notification = self._get_im_status(im_status_ids_by_model)
            if im_status_notification:
                self.env['bus.bus']._sendone(self.env.user.partner_id, 'bus/im_status', im_status_notification)

    @classmethod
    def _authenticate(cls):
        if wsrequest.session.uid is not None:
            if not security.check_session(wsrequest.session, wsrequest.env):
                wsrequest.session.logout(keep_db=True)
                raise SessionExpiredException()
        else:
            public_user = wsrequest.env.ref('base.public_user')
            wsrequest.update_env(user=public_user.id)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models
from odoo.addons.bus.models.bus_presence import AWAY_TIMER
from odoo.addons.bus.models.bus_presence import DISCONNECTION_TIMER


class ResPartner(models.Model):
    _inherit = 'res.partner'

    im_status = fields.Char('IM Status', compute='_compute_im_status')

    def _compute_im_status(self):
        self.env.cr.execute("""
            SELECT
                U.partner_id as id,
                CASE WHEN max(B.last_poll) IS NULL THEN 'offline'
                    WHEN age(now() AT TIME ZONE 'UTC', max(B.last_poll)) > interval %s THEN 'offline'
                    WHEN age(now() AT TIME ZONE 'UTC', max(B.last_presence)) > interval %s THEN 'away'
                    ELSE 'online'
                END as status
            FROM bus_presence B
            RIGHT JOIN res_users U ON B.user_id = U.id
            WHERE U.partner_id IN %s AND U.active = 't'
         GROUP BY U.partner_id
        """, ("%s seconds" % DISCONNECTION_TIMER, "%s seconds" % AWAY_TIMER, tuple(self.ids)))
        res = dict(((status['id'], status['status']) for status in self.env.cr.dictfetchall()))
        for partner in self:
            partner.im_status = res.get(partner.id, 'im_partner')  # if not found, it is a partner, useful to avoid to refresh status in js

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models
from odoo.addons.bus.models.bus_presence import AWAY_TIMER
from odoo.addons.bus.models.bus_presence import DISCONNECTION_TIMER


class ResUsers(models.Model):

    _inherit = "res.users"

    im_status = fields.Char('IM Status', compute='_compute_im_status')

    def _compute_im_status(self):
        """ Compute the im_status of the users """
        self.env.cr.execute("""
            SELECT
                user_id as id,
                CASE WHEN age(now() AT TIME ZONE 'UTC', last_poll) > interval %s THEN 'offline'
                     WHEN age(now() AT TIME ZONE 'UTC', last_presence) > interval %s THEN 'away'
                     ELSE 'online'
                END as status
            FROM bus_presence
            WHERE user_id IN %s
        """, ("%s seconds" % DISCONNECTION_TIMER, "%s seconds" % AWAY_TIMER, tuple(self.ids)))
        res = dict(((status['id'], status['status']) for status in self.env.cr.dictfetchall()))
        for user in self:
            user.im_status = res.get(user.id, 'offline')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import bus
from . import bus_presence
from . import ir_model
from . import ir_websocket
from . import res_users
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_bus_bus,bus.bus public,model_bus_bus,,0,0,0,0
access_bus_presence,bus.presence,model_bus_presence,base.group_user,1,1,1,1
access_bus_presence_portal,bus.presence,model_bus_presence,base.group_portal,1,1,1,1

```

## File: static\src\im_status_service.js

```javascript
/** @odoo-module **/

import { browser } from '@web/core/browser/browser';
import { registry } from '@web/core/registry';

export const UPDATE_BUS_PRESENCE_DELAY = 60000;
/**
 * This service updates periodically the user presence in order for the
 * im_status to be up to date.
 *
 * In order to receive bus notifications related to im_status, one must
 * register model/ids to monitor to this service.
 */
export const imStatusService = {
    dependencies: ['bus_service', 'multi_tab', 'presence'],

    start(env, { bus_service, multi_tab, presence }) {
        const imStatusModelToIds = {};
        let updateBusPresenceTimeout;
        const throttledUpdateBusPresence = _.throttle(
            function updateBusPresence() {
                clearTimeout(updateBusPresenceTimeout);
                if (!multi_tab.isOnMainTab()) {
                    return;
                }
                const now = new Date().getTime();
                bus_service.send("update_presence", {
                    inactivity_period: now - presence.getLastPresence(),
                    im_status_ids_by_model: { ...imStatusModelToIds },
                });
                updateBusPresenceTimeout = browser.setTimeout(throttledUpdateBusPresence, UPDATE_BUS_PRESENCE_DELAY);
            },
            UPDATE_BUS_PRESENCE_DELAY
        );

        bus_service.addEventListener('connect', () => {
            // wait for im_status model/ids to be registered before starting.
            browser.setTimeout(throttledUpdateBusPresence, 250);
        });
        multi_tab.bus.addEventListener('become_main_tab', throttledUpdateBusPresence);
        bus_service.addEventListener('reconnect', throttledUpdateBusPresence);
        multi_tab.bus.addEventListener('no_longer_main_tab', () => clearTimeout(updateBusPresenceTimeout));
        bus_service.addEventListener('disconnect', () => clearTimeout(updateBusPresenceTimeout));

        return {
            /**
             * Register model/ids whose im_status should be monitored.
             * Notification related to the im_status are then sent
             * through the bus. Overwrite registration if already
             * present.
             *
             * @param {string} model model related to the given ids.
             * @param {Number[]} ids ids whose im_status should be
             * monitored.
             */
            registerToImStatus(model, ids) {
                if (!ids.length) {
                    return this.unregisterFromImStatus(model);
                }
                imStatusModelToIds[model] = ids;
            },
            /**
             * Unregister model from im_status notifications.
             *
             * @param {string} model model to unregister.
             */
            unregisterFromImStatus(model) {
                delete imStatusModelToIds[model];
            },
        };
    },
};

registry.category('services').add('im_status', imStatusService);

```

## File: static\src\multi_tab_service.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';
import { browser } from '@web/core/browser/browser';

const { EventBus } = owl;

let multiTabId = 0;
/**
 * This service uses a Master/Slaves with Leader Election architecture in
 * order to keep track of the main tab. Tabs are synchronized thanks to the
 * localStorage.
 *
 * localStorage used keys are:
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.lastPresenceByTab:
 *   mapping of tab ids to their last recorded presence.
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.main : id of the current
 *   main tab.
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.heartbeat : last main tab
 *   heartbeat time.
 *
 * trigger:
 * - become_main_tab : when this tab became the main.
 * - no_longer_main_tab : when this tab is no longer the main.
 * - shared_value_updated: when one of the shared values changes.
 */
export const multiTabService = {
    start() {
        const bus = new EventBus();

        // CONSTANTS
        const TAB_HEARTBEAT_PERIOD = 10000; // 10 seconds
        const MAIN_TAB_HEARTBEAT_PERIOD = 1500; // 1.5 seconds
        const HEARTBEAT_OUT_OF_DATE_PERIOD = 5000; // 5 seconds
        const HEARTBEAT_KILL_OLD_PERIOD = 15000; // 15 seconds
        // Keys that should not trigger the `shared_value_updated` event.
        const PRIVATE_LOCAL_STORAGE_KEYS = ['main', 'heartbeat'];

        // PROPERTIES
        let _isOnMainTab = false;
        let lastHeartbeat = 0;
        let heartbeatTimeout;
        const sanitizedOrigin = location.origin.replace(/:\/{0,2}/g, '_');
        const localStoragePrefix = `${this.name}.${sanitizedOrigin}.`;
        const now = new Date().getTime();
        const tabId = `${this.name}${multiTabId++}:${now}`;

        function generateLocalStorageKey(baseKey) {
            return localStoragePrefix + baseKey;
        }

        function getItemFromStorage(key, defaultValue) {
            const item = browser.localStorage.getItem(generateLocalStorageKey(key));
            try {
                return item ? JSON.parse(item) : defaultValue;
            } catch {
                return item;
            }
        }

        function setItemInStorage(key, value) {
            browser.localStorage.setItem(generateLocalStorageKey(key), JSON.stringify(value));
        }

        function startElection() {
            if (_isOnMainTab) {
                return;
            }
            // Check who's next.
            const now = new Date().getTime();
            const lastPresenceByTab = getItemFromStorage('lastPresenceByTab', {});
            const heartbeatKillOld = now - HEARTBEAT_KILL_OLD_PERIOD;
            let newMain;
            for (const [tab, lastPresence] of Object.entries(lastPresenceByTab)) {
                // Check for dead tabs.
                if (lastPresence < heartbeatKillOld) {
                    continue;
                }
                newMain = tab;
                break;
            }
            if (newMain === tabId) {
                // We're next in queue. Electing as main.
                lastHeartbeat = now;
                setItemInStorage('heartbeat', lastHeartbeat);
                setItemInStorage('main', true);
                _isOnMainTab = true;
                bus.trigger('become_main_tab');
                // Removing main peer from queue.
                delete lastPresenceByTab[newMain];
                setItemInStorage('lastPresenceByTab', lastPresenceByTab);
            }
        }

        function heartbeat() {
            const now = new Date().getTime();
            let heartbeatValue = getItemFromStorage('heartbeat', 0);
            const lastPresenceByTab = getItemFromStorage('lastPresenceByTab', {});
            if (heartbeatValue + HEARTBEAT_OUT_OF_DATE_PERIOD < now) {
                // Heartbeat is out of date. Electing new main.
                startElection();
                heartbeatValue = getItemFromStorage('heartbeat', 0);
            }
            if (_isOnMainTab) {
                // Walk through all tabs and kill old ones.
                const cleanedTabs = {};
                for (const [tabId, lastPresence] of Object.entries(lastPresenceByTab)) {
                    if (lastPresence + HEARTBEAT_KILL_OLD_PERIOD > now) {
                        cleanedTabs[tabId] = lastPresence;
                    }
                }
                if (heartbeatValue !== lastHeartbeat) {
                    // Someone else is also main...
                    // It should not happen, except in some race condition situation.
                    _isOnMainTab = false;
                    lastHeartbeat = 0;
                    lastPresenceByTab[tabId] = now;
                    setItemInStorage('lastPresenceByTab', lastPresenceByTab);
                    bus.trigger('no_longer_main_tab');
                } else {
                    lastHeartbeat = now;
                    setItemInStorage('heartbeat', now);
                    setItemInStorage('lastPresenceByTab', cleanedTabs);
                }
            } else {
                // Update own heartbeat.
                lastPresenceByTab[tabId] = now;
                setItemInStorage('lastPresenceByTab', lastPresenceByTab);
            }
            const hbPeriod = _isOnMainTab ? MAIN_TAB_HEARTBEAT_PERIOD : TAB_HEARTBEAT_PERIOD;
            heartbeatTimeout = browser.setTimeout(heartbeat, hbPeriod);
        }

        function onStorage({ key, newValue }) {
            if (key === generateLocalStorageKey('main') && !newValue) {
                // Main was unloaded.
                startElection();
            }
            if (PRIVATE_LOCAL_STORAGE_KEYS.includes(key)) {
                return;
            }
            if (key && key.includes(localStoragePrefix)) {
                // Only trigger the shared_value_updated event if the key is
                // related to this service/origin.
                const baseKey = key.replace(localStoragePrefix, '');
                bus.trigger('shared_value_updated', { key: baseKey, newValue });
            }
        }

        function onPagehide() {
            clearTimeout(heartbeatTimeout);
            const lastPresenceByTab = getItemFromStorage('lastPresenceByTab', {});
            delete lastPresenceByTab[tabId];
            setItemInStorage('lastPresenceByTab', lastPresenceByTab);

            // Unload main.
            if (_isOnMainTab) {
                _isOnMainTab = false;
                bus.trigger('no_longer_main_tab');
                browser.localStorage.removeItem(generateLocalStorageKey('main'));
            }
        }

        browser.addEventListener('pagehide', onPagehide);
        browser.addEventListener('storage', onStorage);

        // REGISTER THIS TAB
        const lastPresenceByTab = getItemFromStorage('lastPresenceByTab', {});
        lastPresenceByTab[tabId] = now;
        setItemInStorage('lastPresenceByTab', lastPresenceByTab);

        if (!getItemFromStorage('main')) {
            startElection();
        }
        heartbeat();

        return {
            bus,
            get currentTabId() {
                return tabId;
            },
            /**
             * Determine whether or not this tab is the main one.
             *
             * @returns {boolean}
             */
            isOnMainTab() {
                return _isOnMainTab;
            },
            /**
             * Get value shared between all the tabs.
             *
             * @param {string} key
             * @param {any} defaultValue Value to be returned if this
             * key does not exist.
             */
            getSharedValue(key, defaultValue) {
                return getItemFromStorage(key, defaultValue);
            },
            /**
             * Set value shared between all the tabs.
             *
             * @param {string} key
             * @param {any} value
             */
            setSharedValue(key, value) {
                if (value === undefined) {
                    return this.removeSharedValue(key);
                }
                setItemInStorage(key, value);
            },
            /**
             * Remove value shared between all the tabs.
             *
             * @param {string} key
             */
            removeSharedValue(key) {
                browser.localStorage.removeItem(generateLocalStorageKey(key));
            },
        };
    },
};

registry.category('services').add('multi_tab', multiTabService);

```

## File: static\src\services\assets_watchdog_service.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";
import { registry } from "@web/core/registry";
import { session } from "@web/session";

export const assetsWatchdogService = {
    dependencies: ["bus_service", "notification"],

    start(env, { bus_service, notification }) {
        let isNotificationDisplayed = false;
        let bundleNotifTimerID = null;

        bus_service.addEventListener('notification', onNotification.bind(this));
        bus_service.start();

        /**
         * Displays one notification on user's screen when assets have changed
         */
        function displayBundleChangedNotification() {
            if (!isNotificationDisplayed) {
                // Wrap the notification inside a delay.
                // The server may be overwhelmed with recomputing assets
                // We wait until things settle down
                browser.clearTimeout(bundleNotifTimerID);
                bundleNotifTimerID = browser.setTimeout(() => {
                    notification.add(
                        env._t("The page appears to be out of date."),
                        {
                            title: env._t("Refresh"),
                            type: "warning",
                            sticky: true,
                            buttons: [
                                {
                                    name: env._t("Refresh"),
                                    primary: true,
                                    onClick: () => {
                                        browser.location.reload();
                                    },
                                },
                            ],
                            onClose: () => {
                                isNotificationDisplayed = false;
                            },
                        }
                    );
                    isNotificationDisplayed = true;
                }, getBundleNotificationDelay());
            }
        }

        /**
         * Computes a random delay to avoid hammering the server
         * when bundles change with all the users reloading
         * at the same time
         *
         * @return {number} delay in milliseconds
         */
        function getBundleNotificationDelay() {
            return 10000 + Math.floor(Math.random() * 50) * 1000;
        }

        /**
         * Reacts to bus's notification
         *
         * @param {CustomEvent} ev
         * @param {Array} [ev.detail] list of received notifications
         */
        function onNotification({ detail: notifications }) {
            for (const { payload, type } of notifications) {
                if (type === 'bundle_changed') {
                    if (payload.server_version !== session.server_version) {
                        displayBundleChangedNotification();
                        break;
                    }
                }
            }
        }
    },
};

registry.category("services").add("assetsWatchdog", assetsWatchdogService);

```

## File: static\src\services\bus_service.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";
import { Deferred } from "@web/core/utils/concurrency";
import { registry } from '@web/core/registry';
import { session } from '@web/session';
import { isIosApp } from '@web/core/browser/feature_detection';
import { WORKER_VERSION } from "@bus/workers/websocket_worker";
import legacySession from "web.session";

const { EventBus } = owl;

/**
 * Communicate with a SharedWorker in order to provide a single websocket
 * connection shared across multiple tabs.
 *
 *  @emits connect
 *  @emits disconnect
 *  @emits reconnect
 *  @emits reconnecting
 *  @emits notification
 */
export const busService = {
    dependencies: ['localization', 'multi_tab'],

    async start(env, { multi_tab: multiTab }) {
        const bus = new EventBus();
        let worker;
        let isActive = false;
        let isInitialized = false;
        let isUsingSharedWorker = browser.SharedWorker && !isIosApp();
        const startTs = new Date().getTime();
        const connectionInitializedDeferred = new Deferred();

        /**
        * Send a message to the worker.
        *
        * @param {WorkerAction} action Action to be
        * executed by the worker.
        * @param {Object|undefined} data Data required for the action to be
        * executed.
        */
        function send(action, data) {
            if (!worker) {
                return;
            }
            const message = { action, data };
            if (isUsingSharedWorker) {
                worker.port.postMessage(message);
            } else {
                worker.postMessage(message);
            }
        }

        /**
         * Handle messages received from the shared worker and fires an
         * event according to the message type.
         *
         * @param {MessageEvent} messageEv
         * @param {{type: WorkerEvent, data: any}[]}  messageEv.data
         */
        function handleMessage(messageEv) {
            const { type } = messageEv.data;
            let { data } = messageEv.data;
            if (type === 'notification') {
                multiTab.setSharedValue('last_notification_id', data[data.length - 1].id);
                data = data.map(notification => notification.message);
            } else if (type === 'initialized') {
                isInitialized = true;
                connectionInitializedDeferred.resolve();
                return;
            }
            bus.trigger(type, data);
        }

        /**
         * Initialize the connection to the worker by sending it usefull
         * initial informations (last notification id, debug mode,
         * ...).
         */
        function initializeWorkerConnection() {
            // User_id has different values according to its origin:
            //     - frontend: number or false,
            //     - backend: array with only one number
            //     - guest page: array containing null or number
            //     - public pages: undefined
            // Let's format it in order to ease its usage:
            //     - number if user is logged, false otherwise, keep
            //       undefined to indicate session_info is not available.
            let uid = Array.isArray(session.user_id) ? session.user_id[0] : session.user_id;
            if (!uid && uid !== undefined) {
                uid = false;
            }
            send('initialize_connection', {
                websocketURL: `${legacySession.prefix.replace("http", "ws")}/websocket`,
                debug: odoo.debug,
                lastNotificationId: multiTab.getSharedValue('last_notification_id', 0),
                uid,
                startTs,
            });
        }

        /**
         * Start the "bus_service" worker.
         */
        function startWorker() {
            let workerURL = `${legacySession.prefix}/bus/websocket_worker_bundle?v=${WORKER_VERSION}`;
            if (legacySession.prefix !== window.origin) {
                // Bus service is loaded from a different origin than the bundle
                // URL. The Worker expects an URL from this origin, give it a base64
                // URL that will then load the bundle via "importScripts" which
                // allows cross origin.
                const source = `importScripts("${workerURL}");`;
                workerURL = 'data:application/javascript;base64,' + window.btoa(source);
            }
            const workerClass = isUsingSharedWorker ? browser.SharedWorker : browser.Worker;
            worker = new workerClass(workerURL, {
                name: isUsingSharedWorker
                    ? 'odoo:websocket_shared_worker'
                    : 'odoo:websocket_worker',
            });
            worker.addEventListener("error", (e) => {
                if (!isInitialized && workerClass === browser.SharedWorker) {
                    console.warn(
                        'Error while loading "bus_service" SharedWorker, fallback on Worker.'
                    );
                    isUsingSharedWorker = false;
                    startWorker();
                } else if (!isInitialized) {
                    isInitialized = true;
                    connectionInitializedDeferred.resolve();
                    console.warn('Bus service failed to initialized.');
                }
            });
            if (isUsingSharedWorker) {
                worker.port.start();
                worker.port.addEventListener('message', handleMessage);
            } else {
                worker.addEventListener('message', handleMessage);
            }
            initializeWorkerConnection();
        }
        browser.addEventListener('pagehide', ({ persisted }) => {
            if (!persisted) {
                // Page is gonna be unloaded, disconnect this client
                // from the worker.
                send('leave');
            }
        });
        browser.addEventListener('online', () => {
            if (isActive) {
                send('start');
            }
        });
        browser.addEventListener('offline', () => send('stop'));

        return {
            addEventListener: bus.addEventListener.bind(bus),
            addChannel: async channel => {
                if (!worker) {
                    startWorker();
                    await connectionInitializedDeferred;
                }
                send('add_channel', channel);
                send('start');
                isActive = true;
            },
            deleteChannel: channel => send('delete_channel', channel),
            forceUpdateChannels: () => send('force_update_channels'),
            trigger: bus.trigger.bind(bus),
            removeEventListener: bus.removeEventListener.bind(bus),
            send: (eventName, data) => send('send', { event_name: eventName, data }),
            start: async () => {
                if (!worker) {
                    startWorker();
                    await connectionInitializedDeferred;
                }
                send('start');
                isActive = true;
            },
            stop: () => {
                send('leave');
                isActive = false;
            },
        };
    },
};
registry.category('services').add('bus_service', busService);

```

## File: static\src\services\presence_service.js

```javascript
/** @odoo-module **/

import { browser } from '@web/core/browser/browser';
import { registry } from '@web/core/registry';
import core from 'web.core';

export const presenceService = {
    start(env) {
        const LOCAL_STORAGE_PREFIX = 'presence';

        // map window_focus event from the wowlBus to the legacy one.
        env.bus.addEventListener('window_focus', isOdooFocused => {
            core.bus.trigger('window_focus', isOdooFocused);
        });

        let isOdooFocused = true;
        let lastPresenceTime = (
            browser.localStorage.getItem(`${LOCAL_STORAGE_PREFIX}.lastPresence`)
            || new Date().getTime()
        );

        function onPresence() {
            lastPresenceTime = new Date().getTime();
            browser.localStorage.setItem(`${LOCAL_STORAGE_PREFIX}.lastPresence`, lastPresenceTime);
        }

        function onFocusChange(isFocused) {
            try {
                isFocused = parent.document.hasFocus();
            } catch {}
            isOdooFocused = isFocused;
            browser.localStorage.setItem(`${LOCAL_STORAGE_PREFIX}.focus`, isOdooFocused);
            if (isOdooFocused) {
                lastPresenceTime = new Date().getTime();
                env.bus.trigger('window_focus', isOdooFocused);
            }
        }

        function onStorage({ key, newValue }) {
            if (key === `${LOCAL_STORAGE_PREFIX}.focus`) {
                isOdooFocused = JSON.parse(newValue);
                env.bus.trigger('window_focus', newValue);
            }
            if (key === `${LOCAL_STORAGE_PREFIX}.lastPresence`) {
                lastPresenceTime = JSON.parse(newValue);
            }
        }
        browser.addEventListener('storage', onStorage);
        browser.addEventListener('focus', () => onFocusChange(true));
        browser.addEventListener('blur', () => onFocusChange(false));
        browser.addEventListener('pagehide', () => onFocusChange(false));
        browser.addEventListener('click', onPresence);
        browser.addEventListener('keydown', onPresence);

        return {
            getLastPresence() {
                return lastPresenceTime;
            },
            isOdooFocused() {
                return isOdooFocused;
            }
        };
    },
};

registry.category('services').add('presence', presenceService);

```

## File: static\src\services\legacy\make_bus_service_to_legacy_env.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';

export function makeBusServiceToLegacyEnv(legacyEnv) {
    return {
        dependencies: ['bus_service'],
        start(_, { bus_service }) {
            legacyEnv.services['bus_service'] = bus_service;
        },
    };
}

registry.category('wowlToLegacyServiceMappers').add('bus_service_to_legacy_env', makeBusServiceToLegacyEnv);

```

## File: static\src\services\legacy\make_multi_tab_to_legacy_env.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';

export function makeMultiTabToLegacyEnv(legacyEnv) {
    return {
        dependencies: ['multi_tab'],
        start(_, { multi_tab }) {
            legacyEnv.services['multi_tab'] = multi_tab;
        },
    };
}

registry.category('wowlToLegacyServiceMappers').add('multi_tab_to_legacy_env', makeMultiTabToLegacyEnv);

```

## File: static\src\workers\websocket_worker.js

```javascript
/** @odoo-module **/

import { debounce } from '@bus/workers/websocket_worker_utils';

/**
 * Type of events that can be sent from the worker to its clients.
 *
 * @typedef { 'connect' | 'reconnect' | 'disconnect' | 'reconnecting' | 'notification' | 'initialized' } WorkerEvent
 */

/**
 * Type of action that can be sent from the client to the worker.
 *
 * @typedef {'add_channel' | 'delete_channel' | 'force_update_channels' | 'initialize_connection' | 'send' | 'leave' | 'stop' | 'start' } WorkerAction
 */

export const WEBSOCKET_CLOSE_CODES = Object.freeze({
    CLEAN: 1000,
    GOING_AWAY: 1001,
    PROTOCOL_ERROR: 1002,
    INCORRECT_DATA: 1003,
    ABNORMAL_CLOSURE: 1006,
    INCONSISTENT_DATA: 1007,
    MESSAGE_VIOLATING_POLICY: 1008,
    MESSAGE_TOO_BIG: 1009,
    EXTENSION_NEGOTIATION_FAILED: 1010,
    SERVER_ERROR: 1011,
    RESTART: 1012,
    TRY_LATER: 1013,
    BAD_GATEWAY: 1014,
    SESSION_EXPIRED: 4001,
    KEEP_ALIVE_TIMEOUT: 4002,
    RECONNECTING: 4003,
});
// Should be incremented on every worker update in order to force
// update of the worker in browser cache.
export const WORKER_VERSION = '1.0.5';
const INITIAL_RECONNECT_DELAY = 1000;
const MAXIMUM_RECONNECT_DELAY = 60000;

/**
 * This class regroups the logic necessary in order for the
 * SharedWorker/Worker to work. Indeed, Safari and some minor browsers
 * do not support SharedWorker. In order to solve this issue, a Worker
 * is used in this case. The logic is almost the same than the one used
 * for SharedWorker and this class implements it.
 */
export class WebsocketWorker {
    constructor() {
        // Timestamp of start of most recent bus service sender
        this.newestStartTs = undefined;
        this.websocketURL = "";
        this.currentUID = null;
        this.isWaitingForNewUID = true;
        this.channelsByClient = new Map();
        this.connectRetryDelay = INITIAL_RECONNECT_DELAY;
        this.connectTimeout = null;
        this.debugModeByClient = new Map();
        this.isDebug = false;
        this.isReconnecting = false;
        this.lastChannelSubscription = null;
        this.lastNotificationId = 0;
        this.messageWaitQueue = [];
        this._forceUpdateChannels = debounce(this._forceUpdateChannels, 300, true);

        this._onWebsocketClose = this._onWebsocketClose.bind(this);
        this._onWebsocketError = this._onWebsocketError.bind(this);
        this._onWebsocketMessage = this._onWebsocketMessage.bind(this);
        this._onWebsocketOpen = this._onWebsocketOpen.bind(this);
    }

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Send the message to all the clients that are connected to the
     * worker.
     *
     * @param {WorkerEvent} type Event to broadcast to connected
     * clients.
     * @param {Object} data
     */
    broadcast(type, data) {
        for (const client of this.channelsByClient.keys()) {
            client.postMessage({ type, data });
        }
    }

    /**
     * Register a client handled by this worker.
     *
     * @param {MessagePort} messagePort
     */
    registerClient(messagePort) {
        messagePort.onmessage = ev => {
            this._onClientMessage(messagePort, ev.data);
        };
        this.channelsByClient.set(messagePort, []);
    }

    /**
     * Send message to the given client.
     *
     * @param {number} client
     * @param {WorkerEvent} type
     * @param {Object} data
     */
    sendToClient(client, type, data) {
        client.postMessage({ type, data });
    }

    //--------------------------------------------------------------------------
    // PRIVATE
    //--------------------------------------------------------------------------

    /**
     * Called when a message is posted to the worker by a client (i.e. a
     * MessagePort connected to this worker).
     *
     * @param {MessagePort} client
     * @param {Object} message
     * @param {WorkerAction} [message.action]
     * Action to execute.
     * @param {Object|undefined} [message.data] Data required by the
     * action.
     */
    _onClientMessage(client, { action, data }) {
        switch (action) {
            case 'send':
                return this._sendToServer(data);
            case 'start':
                return this._start();
            case 'stop':
                return this._stop();
            case 'leave':
                return this._unregisterClient(client);
            case 'add_channel':
                return this._addChannel(client, data);
            case 'delete_channel':
                return this._deleteChannel(client, data);
            case 'force_update_channels':
                return this._forceUpdateChannels();
            case 'initialize_connection':
                return this._initializeConnection(client, data);
        }
    }

    /**
     * Add a channel for the given client. If this channel is not yet
     * known, update the subscription on the server.
     *
     * @param {MessagePort} client
     * @param {string} channel
     */
    _addChannel(client, channel) {
        const clientChannels = this.channelsByClient.get(client);
        if (!clientChannels.includes(channel)) {
            clientChannels.push(channel);
            this.channelsByClient.set(client, clientChannels);
            this._updateChannels();
        }
    }

    /**
     * Remove a channel for the given client. If this channel is not
     * used anymore, update the subscription on the server.
     *
     * @param {MessagePort} client
     * @param {string} channel
     */
    _deleteChannel(client, channel) {
        const clientChannels = this.channelsByClient.get(client);
        if (!clientChannels) {
            return;
        }
        const channelIndex = clientChannels.indexOf(channel);
        if (channelIndex !== -1) {
            clientChannels.splice(channelIndex, 1);
            this._updateChannels();
        }
    }

    /**
     * Update the channels on the server side even if the channels on
     * the client side are the same than the last time we subscribed.
     */
    _forceUpdateChannels() {
        this._updateChannels({ force: true });
    }

    /**
     * Remove the given client from this worker client list as well as
     * its channels. If some of its channels are not used anymore,
     * update the subscription on the server.
     *
     * @param {MessagePort} client
     */
    _unregisterClient(client) {
        this.channelsByClient.delete(client);
        this.debugModeByClient.delete(client);
        this.isDebug = Object.values(this.debugModeByClient).some(debugValue => debugValue !== '');
        this._updateChannels();
    }

    /**
     * Initialize a client connection to this worker.
     *
     * @param {Object} param0
     * @param {String} [param0.debug] Current debugging mode for the
     * given client.
     * @param {Number} [param0.lastNotificationId] Last notification id
     * known by the client.
     * @param {String} [param0.websocketURL] URL of the websocket endpoint.
     * @param {Number|false|undefined} [param0.uid] Current user id
     *     - Number: user is logged whether on the frontend/backend.
     *     - false: user is not logged.
     *     - undefined: not available (e.g. livechat support page)
     * @param {Number} param0.startTs Timestamp of start of bus service sender.
     */
    _initializeConnection(client, { debug, lastNotificationId, uid, websocketURL, startTs }) {
        if (this.newestStartTs && this.newestStartTs > startTs) {
            this.debugModeByClient[client] = debug;
            this.isDebug = Object.values(this.debugModeByClient).some(debugValue => debugValue !== '');
            this.sendToClient(client, "initialized");
            return;
        }
        this.newestStartTs = startTs;
        this.websocketURL = websocketURL;
        this.lastNotificationId = lastNotificationId;
        this.debugModeByClient[client] = debug;
        this.isDebug = Object.values(this.debugModeByClient).some(debugValue => debugValue !== '');
        const isCurrentUserKnown = uid !== undefined;
        if (this.isWaitingForNewUID && isCurrentUserKnown) {
            this.isWaitingForNewUID = false;
            this.currentUID = uid;
        }
        if (this.currentUID !== uid && isCurrentUserKnown) {
            this.currentUID = uid;
            if (this.websocket) {
                this.websocket.close(WEBSOCKET_CLOSE_CODES.CLEAN);
            }
            this.channelsByClient.forEach((_, key) => this.channelsByClient.set(key, []));
        }
        this.sendToClient(client, 'initialized');
    }

    /**
     * Determine whether or not the websocket associated to this worker
     * is connected.
     *
     * @returns {boolean}
     */
    _isWebsocketConnected() {
        return this.websocket && this.websocket.readyState === 1;
    }

    /**
     * Determine whether or not the websocket associated to this worker
     * is connecting.
     *
     * @returns {boolean}
     */
    _isWebsocketConnecting() {
        return this.websocket && this.websocket.readyState === 0;
    }

    /**
     * Determine whether or not the websocket associated to this worker
     * is in the closing state.
     *
     * @returns {boolean}
     */
    _isWebsocketClosing() {
        return this.websocket && this.websocket.readyState === 2;
    }

    /**
     * Triggered when a connection is closed. If closure was not clean ,
     * try to reconnect after indicating to the clients that the
     * connection was closed.
     *
     * @param {CloseEvent} ev
     * @param {number} code  close code indicating why the connection
     * was closed.
     * @param {string} reason reason indicating why the connection was
     * closed.
     */
    _onWebsocketClose({ code, reason }) {
        if (this.isDebug) {
            console.debug(`%c${new Date().toLocaleString()} - [onClose]`, 'color: #c6e; font-weight: bold;', code, reason);
        }
        this.lastChannelSubscription = null;
        if (this.isReconnecting) {
            // Connection was not established but the close event was
            // triggered anyway. Let the onWebsocketError method handle
            // this case.
            return;
        }
        this.broadcast('disconnect', { code, reason });
        if (code === WEBSOCKET_CLOSE_CODES.CLEAN) {
            // WebSocket was closed on purpose, do not try to reconnect.
            return;
        }
        // WebSocket was not closed cleanly, let's try to reconnect.
        this.broadcast('reconnecting', { closeCode: code });
        this.isReconnecting = true;
        if (code === WEBSOCKET_CLOSE_CODES.KEEP_ALIVE_TIMEOUT) {
            // Don't wait to reconnect on keep alive timeout.
            this.connectRetryDelay = 0;
        }
        if (code === WEBSOCKET_CLOSE_CODES.SESSION_EXPIRED) {
            this.isWaitingForNewUID = true;
        }
        this._retryConnectionWithDelay();
    }

    /**
     * Triggered when a connection failed or failed to established.
     */
    _onWebsocketError() {
        if (this.isDebug) {
            console.debug(`%c${new Date().toLocaleString()} - [onError]`, 'color: #c6e; font-weight: bold;');
        }
        this._retryConnectionWithDelay();
    }

    /**
    * Handle data received from the bus.
    *
    * @param {MessageEvent} messageEv
    */
    _onWebsocketMessage(messageEv) {
        const notifications = JSON.parse(messageEv.data);
        if (this.isDebug) {
            console.debug(`%c${new Date().toLocaleString()} - [onMessage]`, 'color: #c6e; font-weight: bold;', notifications);
        }
        this.lastNotificationId = notifications[notifications.length - 1].id;
        this.broadcast('notification', notifications);
    }

    /**
     * Triggered on websocket open. Send message that were waiting for
     * the connection to open.
     */
    _onWebsocketOpen() {
        if (this.isDebug) {
            console.debug(`%c${new Date().toLocaleString()} - [onOpen]`, 'color: #c6e; font-weight: bold;');
        }
        this._updateChannels();
        this.messageWaitQueue.forEach(msg => this.websocket.send(msg));
        this.messageWaitQueue = [];
        this.broadcast(this.isReconnecting ? 'reconnect' : 'connect');
        this.connectRetryDelay = INITIAL_RECONNECT_DELAY;
        this.connectTimeout = null;
        this.isReconnecting = false;
    }

    /**
     * Try to reconnect to the server, an exponential back off is
     * applied to the reconnect attempts.
     */
    _retryConnectionWithDelay() {
        this.connectRetryDelay = Math.min(this.connectRetryDelay * 1.5, MAXIMUM_RECONNECT_DELAY) + 1000 * Math.random();
        this.connectTimeout = setTimeout(this._start.bind(this), this.connectRetryDelay);
    }

    /**
     * Send a message to the server through the websocket connection.
     * If the websocket is not open, enqueue the message and send it
     * upon the next reconnection.
     *
     * @param {any} message Message to send to the server.
     */
    _sendToServer(message) {
        const payload = JSON.stringify(message);
        if (!this._isWebsocketConnected()) {
            this.messageWaitQueue.push(payload);
        } else {
            this.websocket.send(payload);
        }
    }

    /**
     * Start the worker by opening a websocket connection.
     */
    _start() {
        if (this._isWebsocketConnected() || this._isWebsocketConnecting()) {
            return;
        }
        if (this.websocket) {
            this.websocket.removeEventListener('open', this._onWebsocketOpen);
            this.websocket.removeEventListener('message', this._onWebsocketMessage);
            this.websocket.removeEventListener('error', this._onWebsocketError);
            this.websocket.removeEventListener('close', this._onWebsocketClose);
        }
        if (this._isWebsocketClosing()) {
            // close event was not triggered and will never be, broadcast the
            // disconnect event for consistency sake.
            this.lastChannelSubscription = null;
            this.broadcast("disconnect", { code: WEBSOCKET_CLOSE_CODES.ABNORMAL_CLOSURE });
        }
        this.websocket = new WebSocket(this.websocketURL);
        this.websocket.addEventListener('open', this._onWebsocketOpen);
        this.websocket.addEventListener('error', this._onWebsocketError);
        this.websocket.addEventListener('message', this._onWebsocketMessage);
        this.websocket.addEventListener('close', this._onWebsocketClose);
    }

    /**
     * Stop the worker.
     */
    _stop() {
        clearTimeout(this.connectTimeout);
        this.connectRetryDelay = INITIAL_RECONNECT_DELAY;
        this.isReconnecting = false;
        this.lastChannelSubscription = null;
        if (this.websocket) {
            this.websocket.close();
        }
    }

    /**
     * Update the channel subscription on the server. Ignore if the channels
     * did not change since the last subscription.
     *
     * @param {boolean} force Whether or not we should update the subscription
     * event if the channels haven't change since last subscription.
     */
    _updateChannels({ force = false } = {}) {
        const allTabsChannels = [...new Set([].concat.apply([], [...this.channelsByClient.values()]))].sort();
        const allTabsChannelsString = JSON.stringify(allTabsChannels);
        const shouldUpdateChannelSubscription = allTabsChannelsString !== this.lastChannelSubscription;
        if (force || shouldUpdateChannelSubscription) {
            this.lastChannelSubscription = allTabsChannelsString;
            this._sendToServer({ event_name: 'subscribe', data: { channels: allTabsChannels, last: this.lastNotificationId } });
        }
    }
}

```

## File: static\src\workers\websocket_worker_script.js

```javascript
/** @odoo-module **/
/* eslint-env worker */
/* eslint-disable no-restricted-globals */

import { WebsocketWorker } from "./websocket_worker";

(function () {
    const websocketWorker = new WebsocketWorker();

    if (self.name.includes('shared')) {
        // The script is running in a shared worker: let's register every
        // tab connection to the worker in order to relay notifications
        // coming from the websocket.
        onconnect = function (ev) {
            const currentClient = ev.ports[0];
            websocketWorker.registerClient(currentClient);
        };
    } else {
        // The script is running in a simple web worker.
        websocketWorker.registerClient(self);
    }
})();


```

## File: static\src\workers\websocket_worker_utils.js

```javascript
/** @odoo-module **/

/**
 * Returns a function, that, as long as it continues to be invoked, will not
 * be triggered. The function will be called after it stops being called for
 * N milliseconds. If `immediate` is passed, trigger the function on the
 * leading edge, instead of the trailing.
 *
 * Inspired by https://davidwalsh.name/javascript-debounce-function
 */
 export function debounce(func, wait, immediate) {
    let timeout;
    return function () {
        const context = this;
        const args = arguments;
        function later() {
            timeout = null;
            if (!immediate) {
                func.apply(context, args);
            }
        }
        const callNow = immediate && !timeout;
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
        if (callNow) {
            func.apply(context, args);
        }
    };
}

```

