# Odoo Module: auth_passkey

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import controllers

```

## File: __manifest__.py

```python
{
    'name': 'Passkeys',
    'version': '1.0',
    'summary': 'Log in with a Passkey',
    'description': """
The implementation of Passkeys using the webauthn protocol.
===========================================================

Passkeys are a secure alternative to a username and a password.
When a user logs in with a Passkey, MFA will not be required.
""",
    'category': 'Hidden/Tools',
    'depends': ['base_setup', 'web'],
    'data': [
        'views/auth_passkey_key_views.xml',
        'views/auth_passkey_login_templates.xml',
        'views/res_users_identitycheck_views.xml',
        'views/res_users_views.xml',
        'security/ir.model.access.csv',
        'security/security.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'auth_passkey/static/src/views/*',
            'auth_passkey/static/lib/simplewebauthn.js',
        ],
        'web.assets_frontend': [
            'auth_passkey/static/lib/simplewebauthn.js',
            'auth_passkey/static/src/login_passkeys.js',
        ],
    },
    'license': 'LGPL-3',
    'installable': True,
}

```

## File: controllers\main.py

```python
from odoo import http
from odoo.http import request
from odoo.addons.web.controllers.home import CREDENTIAL_PARAMS


CREDENTIAL_PARAMS.append('webauthn_response')


class WebauthnController(http.Controller):
    @http.route(['/auth/passkey/start-auth'], type='json', auth='public')
    def json_start_authentication(self):
        auth_options = request.env['auth.passkey.key']._start_auth()
        return auth_options

```

## File: controllers\__init__.py

```python
from . import main

```

## File: models\auth_passkey_key.py

```python
import base64
import json
import logging
from werkzeug.urls import url_parse

from odoo import api, Command, fields, models, _
from odoo.exceptions import AccessDenied
from odoo.http import request
from odoo.tools import sql, SQL

from odoo.addons.base.models.res_users import check_identity

from .._vendor.webauthn import base64url_to_bytes, generate_authentication_options, generate_registration_options, options_to_json, verify_authentication_response, verify_registration_response
from .._vendor.webauthn.helpers import bytes_to_base64url
from .._vendor.webauthn.helpers.structs import AuthenticatorSelectionCriteria, ResidentKeyRequirement, UserVerificationRequirement

_logger = logging.getLogger(__name__)


class PassKey(models.Model):
    _name = 'auth.passkey.key'
    _description = 'Passkey'
    _order = 'id desc'

    name = fields.Char(required=True)
    credential_identifier = fields.Char(required=True, groups='base.group_system')
    public_key = fields.Char(required=True, groups='base.group_system', compute='_compute_public_key', inverse='_inverse_public_key')
    sign_count = fields.Integer(default=0, groups='base.group_system')

    _sql_constraints = [
        ('unique_identifier', 'UNIQUE(credential_identifier)', 'The credential identifier should be unique.'),
    ]

    def init(self):
        super().init()
        if not sql.column_exists(self.env.cr, 'auth_passkey_key', 'public_key'):
            self.env.cr.execute(SQL('ALTER TABLE auth_passkey_key ADD COLUMN public_key varchar'))

    def unlink(self):
        for passkey in self:
            _logger.info(
                "Passkey (#%d) deleted by %s (#%d) from %s",
                passkey.id,
                self.env.user.login, self.env.user.id,
                request.httprequest.environ['REMOTE_ADDR'] if request else 'n/a'
            )
        return super().unlink()

    def _compute_public_key(self):
        query = 'SELECT public_key FROM auth_passkey_key WHERE id = %s'
        for passkey in self:
            self.env.cr.execute(SQL(query, passkey.id))
            public_key = self.env.cr.fetchone()[0]
            passkey.public_key = public_key

    def _inverse_public_key(self):
        pass

    @api.model
    def _get_session_challenge(self):
        challenge = request.session.pop('webauthn_challenge', None)
        if not challenge:
            raise AccessDenied('Cannot find a challenge for this session')
        return challenge

    @api.model
    def _start_auth(self):
        assert request
        authentication_options = json.loads(options_to_json(generate_authentication_options(
            rp_id=url_parse(self.get_base_url()).host,
            user_verification=UserVerificationRequirement.REQUIRED,
        )))
        request.session['webauthn_challenge'] = authentication_options['challenge']
        return authentication_options

    @api.model
    def _verify_auth(self, auth, public_key, sign_count):
        parsed_url = url_parse(self.get_base_url())
        auth_verification = verify_authentication_response(
            credential=auth,
            expected_challenge=base64url_to_bytes(self._get_session_challenge()),
            expected_origin=parsed_url.replace(path='').to_url(),
            expected_rp_id=parsed_url.host,
            credential_public_key=base64url_to_bytes(public_key),
            credential_current_sign_count=sign_count,
            require_user_verification=True,
        )
        return auth_verification.new_sign_count

    @api.model
    def _start_registration(self):
        assert request
        registration_options = json.loads(options_to_json(generate_registration_options(
            rp_id=url_parse(self.get_base_url()).host,
            rp_name='Odoo',
            user_id=str(self.env.user.id).encode(),
            user_name=self.env.user.login,
            authenticator_selection=AuthenticatorSelectionCriteria(
                resident_key=ResidentKeyRequirement.REQUIRED,
                user_verification=UserVerificationRequirement.REQUIRED
            )
        )))
        request.session['webauthn_challenge'] = registration_options['challenge']
        return registration_options

    @api.model
    def _verify_registration_options(self, registration):
        parsed_url = url_parse(self.get_base_url())
        verification = verify_registration_response(
            credential=registration,
            expected_challenge=base64url_to_bytes(self._get_session_challenge()),
            expected_origin=parsed_url.replace(path='').to_url(),
            expected_rp_id=parsed_url.host,
            require_user_verification=True,
        )
        return {
            'credential_id': verification.credential_id,
            'credential_public_key': verification.credential_public_key,
        }

    @check_identity
    def action_delete_passkey(self):
        for key in self:
            if key.create_uid.id == self.env.user.id:
                # Force to go through `res.users.auth_passkey_key_ids` to trigger the session token cache invalidation
                # See `res.users.write` and `_get_invalidation_fields`
                # `self.env.user` is already sudo, so no need to re-apply `sudo` to get delete access right.
                self.env.user.write({'auth_passkey_key_ids': [Command.delete(key.id)]})
                new_token = self.env.user._compute_session_token(request.session.sid)
                request.session.session_token = new_token
            else:
                _logger.info(
                    "%s (#%d) attempted to delete passkey (#%d) belonging to %s (#%d) from %s but was denied.",
                    self.env.user.login, self.env.user.id,
                    key.id,
                    key.create_uid.login, key.create_uid.id,
                    request.httprequest.environ['REMOTE_ADDR'] if request else 'n/a'
                )

    def action_rename_passkey(self):
        return {
            'name': _('Rename Passkey'),
            'type': 'ir.actions.act_window',
            'res_model': 'auth.passkey.key',
            'view_id': self.env.ref('auth_passkey.auth_passkey_key_rename').id,
            'view_mode': 'form',
            'target': 'new',
            'res_id': self.id,
            'context': {
                'dialog_size': 'medium',
            }
        }


class PassKeyCreate(models.TransientModel):
    _name = 'auth.passkey.key.create'
    _description = 'Create a Passkey'

    name = fields.Char('Name', required=True)

    @check_identity
    def make_key(self, registration=None):
        # We add in these fields with JS, if we didn't give them default values we would get a XML validation warning.
        assert registration, "registration can not be empty"
        self.ensure_one()
        verification = request.env['auth.passkey.key']._verify_registration_options(registration)
        # Force to go through `res.users.auth_passkey_key_ids` to trigger the session token cache invalidation
        # See `res.users.write` and `_get_invalidation_fields`
        # `self.env.user` is already sudo, so no need to re-apply `sudo` to get create access right.
        self.env.user.write({'auth_passkey_key_ids': [Command.create({
            'name': self.name,
            'credential_identifier': bytes_to_base64url(verification['credential_id']),
        })]})
        passkey = self.env.user.auth_passkey_key_ids[0]
        self.env.cr.execute(SQL(
            "UPDATE auth_passkey_key SET public_key = %s WHERE id = %s",
            base64.urlsafe_b64encode(verification['credential_public_key']).decode(),
            passkey.id,
        ))
        ip = request.httprequest.environ['REMOTE_ADDR'] if request else 'n/a'
        _logger.info(
            "Passkey (#%d) created by %s (#%d) from %s",
            passkey.id,
            self.env.user.login, self.env.user.id,
            ip
        )
        new_token = self.env.user._compute_session_token(request.session.sid)
        request.session.session_token = new_token
        return True

```

## File: models\res_users.py

```python
import json

from odoo import fields, models, _
from odoo.tools import SQL
from odoo.exceptions import AccessDenied
from odoo.modules.registry import Registry

from odoo.addons.base.models.res_users import check_identity
from .._vendor.webauthn.helpers.exceptions import InvalidAuthenticationResponse


class UsersPasskey(models.Model):
    _inherit = 'res.users'

    auth_passkey_key_ids = fields.One2many('auth.passkey.key', 'create_uid')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['auth_passkey_key_ids']

    @check_identity
    def action_create_passkey(self):
        return {
            'name': _('Create Passkey'),
            'type': 'ir.actions.act_window',
            'res_model': 'auth.passkey.key.create',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'dialog_size': 'medium',
                'registration': self.env['auth.passkey.key']._start_registration(),
            }
        }

    @classmethod
    def _login(cls, db, credential, user_agent_env):
        if credential['type'] == 'webauthn':
            webauthn = json.loads(credential['webauthn_response'])
            with Registry(db).cursor() as cr:
                cr.execute(SQL("""
                    SELECT login
                      FROM auth_passkey_key key
                      JOIN res_users usr ON usr.id = key.create_uid
                     WHERE credential_identifier=%s
                """, webauthn['id']))
                res = cr.fetchone()
                if not res:
                    raise AccessDenied(_('Unknown passkey'))
                credential['login'] = res[0]
        return super()._login(db, credential, user_agent_env=user_agent_env)

    def _check_credentials(self, credential, env):
        if credential['type'] == 'webauthn':
            webauthn = json.loads(credential['webauthn_response'])
            passkey = self.env['auth.passkey.key'].sudo().search([
                ("create_uid", "=", self.env.user.id),
                ("credential_identifier", "=", webauthn['id']),
            ])
            if not passkey:
                raise AccessDenied(_('Unknown passkey'))
            try:
                new_sign_count = self.env['auth.passkey.key']._verify_auth(
                    webauthn,
                    passkey.public_key,
                    passkey.sign_count,
                )
            except InvalidAuthenticationResponse as e:
                raise AccessDenied(e.args[0]) from None
            passkey.sign_count = new_sign_count
            return {
                'uid': self.env.user.id,
                'auth_method': 'passkey',
                'mfa': 'skip',
            }
        else:
            return super()._check_credentials(credential, env)

    def _get_session_token_fields(self):
        return super()._get_session_token_fields() | {'auth_passkey_key_ids'}

    def _get_session_token_query_params(self):
        params = super()._get_session_token_query_params()
        params['select'] = SQL("%s, ARRAY_AGG(key.id ORDER BY key.id DESC)", params['select'])
        params['joins'] = SQL("%s LEFT JOIN auth_passkey_key key ON res_users.id = key.create_uid", params['joins'])
        return params

```

## File: models\res_users_identitycheck.py

```python
from odoo import api, fields, _
from odoo.exceptions import UserError, AccessDenied
from odoo.addons.base.models.res_users import CheckIdentity


class CheckIdentityPasskeys(CheckIdentity):
    _inherit = 'res.users.identitycheck'

    auth_method = fields.Selection(selection_add=[('webauthn', 'Passkey')])

    @api.model
    def _get_default_auth_method(self):
        if self.env.user.auth_passkey_key_ids:
            return 'webauthn'
        else:
            return super()._get_default_auth_method()

    def _check_identity(self):
        if self.auth_method == 'webauthn':
            try:
                credential = {
                    'webauthn_response': self.password,
                    'type': 'webauthn',
                }
                self.create_uid._check_credentials(credential, {'interactive': True})
            except AccessDenied:
                raise UserError(_("Incorrect Passkey. Please provide a valid passkey or use a different authentication method."))
        else:
            super()._check_identity()

    def action_use_password(self):
        self.ensure_one()
        self.auth_method = 'password'
        self.password = ''
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'res.users.identitycheck',
            'res_id': self.id,
            'name': _('Security Control'),
            'target': 'new',
            'views': [(False, 'form')],
        }

```

## File: models\__init__.py

```python
from . import auth_passkey_key
from . import res_users
from . import res_users_identitycheck

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
auth_passkey.access_auth_passkey_key,access_auth_passkey_key,model_auth_passkey_key,base.group_user,1,1,0,0
auth_passkey.access_auth_passkey_key_admin,access_auth_passkey_key_admin,model_auth_passkey_key,base.group_erp_manager,1,1,0,1
auth_passkey.access_auth_passkey_key_create,access_auth_passkey_key_create,model_auth_passkey_key_create,base.group_user,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.rule" id="rule_auth_passkey_key_user">
        <field name="name">Passkeys: Users can only access own Passkeys</field>
        <field name="model_id" ref="model_auth_passkey_key"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
        <field name="perm_unlink" eval="1"/>
        <field name="perm_write" eval="1"/>
        <field name="perm_read" eval="1"/>
        <field name="perm_create" eval="1"/>
    </record>

    <record model="ir.rule" id="rule_auth_passkey_key_admin">
        <field name="name">Passkeys: Admins can view and delete other peoples Passkeys</field>
        <field name="model_id" ref="model_auth_passkey_key"/>
        <field name="groups" eval="[(4, ref('base.group_erp_manager'))]"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_unlink" eval="1"/>
        <field name="perm_write" eval="0"/>
        <field name="perm_read" eval="1"/>
        <field name="perm_create" eval="0"/>
    </record>
</odoo>

```

## File: static\lib\simplewebauthn.js

```javascript
/** @odoo-module **/

/*!
  * simplewebauthn/browser@9.0.1 (https://github.com/MasterKale/SimpleWebAuthn)
  * Copyright 2020 Matthew Miller
  * Licensed under MIT (https://github.com/MasterKale/SimpleWebAuthn/blob/master/LICENSE.md)
  */

function utf8StringToBuffer(value) {
    return new TextEncoder().encode(value);
}

function bufferToBase64URLString(buffer) {
    const bytes = new Uint8Array(buffer);
    let str = '';
    for (const charCode of bytes) {
        str += String.fromCharCode(charCode);
    }
    const base64String = btoa(str);
    return base64String.replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
}

function base64URLStringToBuffer(base64URLString) {
    const base64 = base64URLString.replace(/-/g, '+').replace(/_/g, '/');
    const padLength = (4 - (base64.length % 4)) % 4;
    const padded = base64.padEnd(base64.length + padLength, '=');
    const binary = atob(padded);
    const buffer = new ArrayBuffer(binary.length);
    const bytes = new Uint8Array(buffer);
    for (let i = 0; i < binary.length; i++) {
        bytes[i] = binary.charCodeAt(i);
    }
    return buffer;
}

function browserSupportsWebAuthn() {
    return (window?.PublicKeyCredential !== undefined &&
        typeof window.PublicKeyCredential === 'function');
}

function toPublicKeyCredentialDescriptor(descriptor) {
    const { id } = descriptor;
    return {
        ...descriptor,
        id: base64URLStringToBuffer(id),
        transports: descriptor.transports,
    };
}

function isValidDomain(hostname) {
    return (hostname === 'localhost' ||
        /^([a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,}$/i.test(hostname));
}

class WebAuthnError extends Error {
    constructor({ message, code, cause, name, }) {
        super(message, { cause });
        this.name = name ?? cause.name;
        this.code = code;
    }
}

function identifyRegistrationError({ error, options, }) {
    const { publicKey } = options;
    if (!publicKey) {
        throw Error('options was missing required publicKey property');
    }
    if (error.name === 'AbortError') {
        if (options.signal instanceof AbortSignal) {
            return new WebAuthnError({
                message: 'Registration ceremony was sent an abort signal',
                code: 'ERROR_CEREMONY_ABORTED',
                cause: error,
            });
        }
    }
    else if (error.name === 'ConstraintError') {
        if (publicKey.authenticatorSelection?.requireResidentKey === true) {
            return new WebAuthnError({
                message: 'Discoverable credentials were required but no available authenticator supported it',
                code: 'ERROR_AUTHENTICATOR_MISSING_DISCOVERABLE_CREDENTIAL_SUPPORT',
                cause: error,
            });
        }
        else if (publicKey.authenticatorSelection?.userVerification === 'required') {
            return new WebAuthnError({
                message: 'User verification was required but no available authenticator supported it',
                code: 'ERROR_AUTHENTICATOR_MISSING_USER_VERIFICATION_SUPPORT',
                cause: error,
            });
        }
    }
    else if (error.name === 'InvalidStateError') {
        return new WebAuthnError({
            message: 'The authenticator was previously registered',
            code: 'ERROR_AUTHENTICATOR_PREVIOUSLY_REGISTERED',
            cause: error,
        });
    }
    else if (error.name === 'NotAllowedError') {
        return new WebAuthnError({
            message: error.message,
            code: 'ERROR_PASSTHROUGH_SEE_CAUSE_PROPERTY',
            cause: error,
        });
    }
    else if (error.name === 'NotSupportedError') {
        const validPubKeyCredParams = publicKey.pubKeyCredParams.filter((param) => param.type === 'public-key');
        if (validPubKeyCredParams.length === 0) {
            return new WebAuthnError({
                message: 'No entry in pubKeyCredParams was of type "public-key"',
                code: 'ERROR_MALFORMED_PUBKEYCREDPARAMS',
                cause: error,
            });
        }
        return new WebAuthnError({
            message: 'No available authenticator supported any of the specified pubKeyCredParams algorithms',
            code: 'ERROR_AUTHENTICATOR_NO_SUPPORTED_PUBKEYCREDPARAMS_ALG',
            cause: error,
        });
    }
    else if (error.name === 'SecurityError') {
        const effectiveDomain = window.location.hostname;
        if (!isValidDomain(effectiveDomain)) {
            return new WebAuthnError({
                message: `${window.location.hostname} is an invalid domain`,
                code: 'ERROR_INVALID_DOMAIN',
                cause: error,
            });
        }
        else if (publicKey.rp.id !== effectiveDomain) {
            return new WebAuthnError({
                message: `The RP ID "${publicKey.rp.id}" is invalid for this domain`,
                code: 'ERROR_INVALID_RP_ID',
                cause: error,
            });
        }
    }
    else if (error.name === 'TypeError') {
        if (publicKey.user.id.byteLength < 1 || publicKey.user.id.byteLength > 64) {
            return new WebAuthnError({
                message: 'User ID was not between 1 and 64 characters',
                code: 'ERROR_INVALID_USER_ID_LENGTH',
                cause: error,
            });
        }
    }
    else if (error.name === 'UnknownError') {
        return new WebAuthnError({
            message: 'The authenticator was unable to process the specified options, or could not create a new credential',
            code: 'ERROR_AUTHENTICATOR_GENERAL_ERROR',
            cause: error,
        });
    }
    return error;
}

class BaseWebAuthnAbortService {
    createNewAbortSignal() {
        if (this.controller) {
            const abortError = new Error('Cancelling existing WebAuthn API call for new one');
            abortError.name = 'AbortError';
            this.controller.abort(abortError);
        }
        const newController = new AbortController();
        this.controller = newController;
        return newController.signal;
    }
    cancelCeremony() {
        if (this.controller) {
            const abortError = new Error('Manually cancelling existing WebAuthn API call');
            abortError.name = 'AbortError';
            this.controller.abort(abortError);
            this.controller = undefined;
        }
    }
}
const WebAuthnAbortService = new BaseWebAuthnAbortService();

const attachments = ['cross-platform', 'platform'];
function toAuthenticatorAttachment(attachment) {
    if (!attachment) {
        return;
    }
    if (attachments.indexOf(attachment) < 0) {
        return;
    }
    return attachment;
}

async function startRegistration(creationOptionsJSON) {
    if (!browserSupportsWebAuthn()) {
        throw new Error('WebAuthn is not supported in this browser');
    }
    const publicKey = {
        ...creationOptionsJSON,
        challenge: base64URLStringToBuffer(creationOptionsJSON.challenge),
        user: {
            ...creationOptionsJSON.user,
            id: utf8StringToBuffer(creationOptionsJSON.user.id),
        },
        excludeCredentials: creationOptionsJSON.excludeCredentials?.map(toPublicKeyCredentialDescriptor),
    };
    const options = { publicKey };
    options.signal = WebAuthnAbortService.createNewAbortSignal();
    let credential;
    try {
        credential = (await navigator.credentials.create(options));
    }
    catch (err) {
        throw identifyRegistrationError({ error: err, options });
    }
    if (!credential) {
        throw new Error('Registration was not completed');
    }
    const { id, rawId, response, type } = credential;
    let transports = undefined;
    if (typeof response.getTransports === 'function') {
        transports = response.getTransports();
    }
    let responsePublicKeyAlgorithm = undefined;
    if (typeof response.getPublicKeyAlgorithm === 'function') {
        try {
            responsePublicKeyAlgorithm = response.getPublicKeyAlgorithm();
        }
        catch (error) {
            warnOnBrokenImplementation('getPublicKeyAlgorithm()', error);
        }
    }
    let responsePublicKey = undefined;
    if (typeof response.getPublicKey === 'function') {
        try {
            const _publicKey = response.getPublicKey();
            if (_publicKey !== null) {
                responsePublicKey = bufferToBase64URLString(_publicKey);
            }
        }
        catch (error) {
            warnOnBrokenImplementation('getPublicKey()', error);
        }
    }
    let responseAuthenticatorData;
    if (typeof response.getAuthenticatorData === 'function') {
        try {
            responseAuthenticatorData = bufferToBase64URLString(response.getAuthenticatorData());
        }
        catch (error) {
            warnOnBrokenImplementation('getAuthenticatorData()', error);
        }
    }
    return {
        id,
        rawId: bufferToBase64URLString(rawId),
        response: {
            attestationObject: bufferToBase64URLString(response.attestationObject),
            clientDataJSON: bufferToBase64URLString(response.clientDataJSON),
            transports,
            publicKeyAlgorithm: responsePublicKeyAlgorithm,
            publicKey: responsePublicKey,
            authenticatorData: responseAuthenticatorData,
        },
        type,
        clientExtensionResults: credential.getClientExtensionResults(),
        authenticatorAttachment: toAuthenticatorAttachment(credential.authenticatorAttachment),
    };
}
function warnOnBrokenImplementation(methodName, cause) {
    console.warn(`The browser extension that intercepted this WebAuthn API call incorrectly implemented ${methodName}. You should report this error to them.\n`, cause);
}

function bufferToUTF8String(value) {
    return new TextDecoder('utf-8').decode(value);
}

function browserSupportsWebAuthnAutofill() {
    const globalPublicKeyCredential = window
        .PublicKeyCredential;
    if (globalPublicKeyCredential.isConditionalMediationAvailable === undefined) {
        return new Promise((resolve) => resolve(false));
    }
    return globalPublicKeyCredential.isConditionalMediationAvailable();
}

function identifyAuthenticationError({ error, options, }) {
    const { publicKey } = options;
    if (!publicKey) {
        throw Error('options was missing required publicKey property');
    }
    if (error.name === 'AbortError') {
        if (options.signal instanceof AbortSignal) {
            return new WebAuthnError({
                message: 'Authentication ceremony was sent an abort signal',
                code: 'ERROR_CEREMONY_ABORTED',
                cause: error,
            });
        }
    }
    else if (error.name === 'NotAllowedError') {
        return new WebAuthnError({
            message: error.message,
            code: 'ERROR_PASSTHROUGH_SEE_CAUSE_PROPERTY',
            cause: error,
        });
    }
    else if (error.name === 'SecurityError') {
        const effectiveDomain = window.location.hostname;
        if (!isValidDomain(effectiveDomain)) {
            return new WebAuthnError({
                message: `${window.location.hostname} is an invalid domain`,
                code: 'ERROR_INVALID_DOMAIN',
                cause: error,
            });
        }
        else if (publicKey.rpId !== effectiveDomain) {
            return new WebAuthnError({
                message: `The RP ID "${publicKey.rpId}" is invalid for this domain`,
                code: 'ERROR_INVALID_RP_ID',
                cause: error,
            });
        }
    }
    else if (error.name === 'UnknownError') {
        return new WebAuthnError({
            message: 'The authenticator was unable to process the specified options, or could not create a new assertion signature',
            code: 'ERROR_AUTHENTICATOR_GENERAL_ERROR',
            cause: error,
        });
    }
    return error;
}

async function startAuthentication(requestOptionsJSON, useBrowserAutofill = false) {
    if (!browserSupportsWebAuthn()) {
        throw new Error('WebAuthn is not supported in this browser');
    }
    let allowCredentials;
    if (requestOptionsJSON.allowCredentials?.length !== 0) {
        allowCredentials = requestOptionsJSON.allowCredentials?.map(toPublicKeyCredentialDescriptor);
    }
    const publicKey = {
        ...requestOptionsJSON,
        challenge: base64URLStringToBuffer(requestOptionsJSON.challenge),
        allowCredentials,
    };
    const options = {};
    if (useBrowserAutofill) {
        if (!(await browserSupportsWebAuthnAutofill())) {
            throw Error('Browser does not support WebAuthn autofill');
        }
        const eligibleInputs = document.querySelectorAll('input[autocomplete$=\'webauthn\']');
        if (eligibleInputs.length < 1) {
            throw Error('No <input> with "webauthn" as the only or last value in its `autocomplete` attribute was detected');
        }
        options.mediation = 'conditional';
        publicKey.allowCredentials = [];
    }
    options.publicKey = publicKey;
    options.signal = WebAuthnAbortService.createNewAbortSignal();
    let credential;
    try {
        credential = (await navigator.credentials.get(options));
    }
    catch (err) {
        throw identifyAuthenticationError({ error: err, options });
    }
    if (!credential) {
        throw new Error('Authentication was not completed');
    }
    const { id, rawId, response, type } = credential;
    let userHandle = undefined;
    if (response.userHandle) {
        userHandle = bufferToUTF8String(response.userHandle);
    }
    return {
        id,
        rawId: bufferToBase64URLString(rawId),
        response: {
            authenticatorData: bufferToBase64URLString(response.authenticatorData),
            clientDataJSON: bufferToBase64URLString(response.clientDataJSON),
            signature: bufferToBase64URLString(response.signature),
            userHandle,
        },
        type,
        clientExtensionResults: credential.getClientExtensionResults(),
        authenticatorAttachment: toAuthenticatorAttachment(credential.authenticatorAttachment),
    };
}

function platformAuthenticatorIsAvailable() {
    if (!browserSupportsWebAuthn()) {
        return new Promise((resolve) => resolve(false));
    }
    return PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();
}

export { WebAuthnAbortService, WebAuthnError, base64URLStringToBuffer, browserSupportsWebAuthn, browserSupportsWebAuthnAutofill, bufferToBase64URLString, platformAuthenticatorIsAvailable, startAuthentication, startRegistration };

```

## File: static\src\login_passkeys.js

```javascript
/** @odoo-module **/

import { rpc } from "@web/core/network/rpc";
import publicWidget from "@web/legacy/js/public/public_widget";
import { startAuthentication } from "../lib/simplewebauthn.js";

publicWidget.registry.passkeyLogin = publicWidget.Widget.extend({
    selector: '.passkey_login_link',
    events: { 'click': '_onclick' },

    async _onclick() {
        const serverOptions = await rpc("/auth/passkey/start-auth");
        const auth = await startAuthentication(serverOptions).catch(e => console.error(e));
        if(!auth) return false;
        const form = document.querySelector('form.oe_login_form');
        form.querySelector('input[name="webauthn_response"]').value = JSON.stringify(auth);
        form.querySelector('input[name="type"]').value = 'webauthn';
        form.submit();
    }
})

```

## File: static\src\views\auth_passkey_identity_check_form_view.js

```javascript
/** @odoo-module **/

import { rpc } from "@web/core/network/rpc";
import { registry } from "@web/core/registry";
import { FormController } from "@web/views/form/form_controller";
import { formView } from "@web/views/form/form_view";
import { startAuthentication } from "../../lib/simplewebauthn.js";

export class PassKeyIdentityCheckFormController extends FormController {
    /**
     * @override
     */
    async beforeExecuteActionButton(clickParams) {
        if (
            clickParams.name === "run_check" &&
            this.model.root.data.auth_method == "webauthn"
        ) {
            const serverOptions = await rpc("/auth/passkey/start-auth");
            const auth = await startAuthentication(serverOptions).catch(e => console.log(e));
            // In case the user cancelled the passkey browser check, just interrupt.
            if(!auth) return false;
            this.model.root.update({password: JSON.stringify(auth)});
        }
        return super.beforeExecuteActionButton(...arguments);
    }
}

export const PassKeyIdentityCheckFormView = {
    ...formView,
    Controller: PassKeyIdentityCheckFormController,
};

registry.category("views").add("auth_passkey_identity_check_view_form", PassKeyIdentityCheckFormView);

```

## File: static\src\views\auth_passkey_key_create_form_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { FormController } from "@web/views/form/form_controller";
import { formView } from "@web/views/form/form_view";
import { startRegistration } from "../../lib/simplewebauthn.js"

export class PassKeyNameFormController extends FormController {
    /**
     * @override
     */
    async beforeExecuteActionButton(clickParams) {
        if (clickParams.name === "make_key") {
            const name = document.querySelector("div[name='name'].o_field_widget input").value
            if(name.length > 0) {
                const serverOptions = this.props.context.registration;
                const registration = await startRegistration(serverOptions).catch(e => console.error(e));
                // In case the user cancelled the passkey browser check, just interrupt.
                if(!registration) return false;
                clickParams.args = JSON.stringify([registration]);
            }
        }
        return super.beforeExecuteActionButton(...arguments);
    }
}

export const PassKeyNameFormView = {
    ...formView,
    Controller: PassKeyNameFormController,
};

registry.category("views").add("auth_passkey_key_create_view_form", PassKeyNameFormView);

```

## File: views\auth_passkey_key_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="auth_passkey_key_rename" model="ir.ui.view">
        <field name="name">auth.passkey.key.rename</field>
        <field name="model">auth.passkey.key</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                </sheet>
                <footer>
                    <button special="save" data-hotkey="s" string="Save" class="btn-primary"/>
                    <button special="cancel" data-hotkey="x" string="Cancel" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_auth_passkey_key_create" model="ir.actions.act_window">
        <field name="name">Create Passkey Wizard</field>
        <field name="res_model">auth.passkey.key.create</field>
        <field name="target">new</field>
        <field name="view_mode">form</field>
    </record>

    <record id="auth_passkey_key_create_view_form" model="ir.ui.view">
        <field name="name">Create Passkey Wizard Form</field>
        <field name="model">auth.passkey.key.create</field>
        <field name="arch" type="xml">
            <form js_class="auth_passkey_key_create_view_form">
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                </sheet>
                <footer>
                    <button name="make_key" type="object" string="Create" class="btn-primary" data-hotkey="q"/>
                    <button special="cancel" data-hotkey="x" string="Cancel" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\auth_passkey_login_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template priority="32" id="auth_passkey_login" inherit_id="web.login">
        <xpath expr="//div[hasclass('oe_login_buttons')]" position="after">
            <em t-attf-class="d-block text-center text-muted small my-1">- or -</em>
            <div class="text-center mt-2">
                <button class="btn btn-link btn-sm passkey_login_link p-0 border-0" type="button">Log in with Passkey</button>
            </div>
        </xpath>
        <xpath expr="//input[@name='redirect']" position="before">
            <input type="hidden" name="webauthn_response"/>
        </xpath>
    </template>
</odoo>

```

## File: views\res_users_identitycheck_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_users_identitycheck_view_form_passkey" model="ir.ui.view">
        <field name="model">res.users.identitycheck</field>
        <field name="inherit_id" ref="base.res_users_identitycheck_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="attributes">
                <attribute name="js_class">auth_passkey_identity_check_view_form</attribute>
            </xpath>
            <xpath expr="//sheet" position="inside">
                <div invisible="auth_method != 'webauthn'">
                    <h3><strong>Use your passkey to authenticate</strong></h3>
                    <p class="mb-0 mt-3">Or choose a different method:</p>
                    <button type="object" name="action_use_password" class="btn btn-link" role="button">Use password</button>
                </div>
            </xpath>
            <xpath expr="//footer/button[@id='password_confirm']" position="before">
                <button string="Use Passkey" type="object" name="run_check" class="btn btn-primary" data-hotkey="q" invisible="auth_method != 'webauthn'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="auth_passkey_users_form">
        <field name="name">auth.passkey.users.form</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form"/>
        <field name="arch" type="xml">
            <page name="account_security" position="inside">
                <group string="Passkeys">
                    <field colspan="4" name="auth_passkey_key_ids" string="">
                        <kanban create="false" can_open="false">
                            <templates>
                                <t t-name="menu">
                                    <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                </t>
                                <t t-name="card">
                                    <field name="name" class="fw-bold fs-5"/>
                                    <small>Created: <field name="create_date"/></small>
                                    <small>Last used: <field name="write_date"/></small>
                                </t>
                            </templates>
                        </kanban>
                    </field>
                </group>
            </page>
        </field>
    </record>

    <record model="ir.ui.view" id="auth_passkey_users_preferences">
        <field name="name">auth.passkey.users.preferences</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
        <field name="arch" type="xml">
            <page name="page_account_security" position="inside">
                <group string="Passkeys">
                    <div colspan="2">
                        <div class="text-muted mb-3" colspan="2">
                            Passkeys are a replacement for your username and password, offering a more secure way of logging in.
                        </div>
                        <div class="mb-2 gap-1" style="display: flex;" colspan="4">
                            <button colspan="2" type="object" name="action_create_passkey" class="btn-secondary" string="Add Passkey"/>
                        </div>
                        <field colspan="4" name="auth_passkey_key_ids">
                            <kanban create="false" can_open="false">
                                <templates>
                                    <t t-name="menu">
                                        <a role="menuitem" type="object" name="action_rename_passkey" class="dropdown-item">Rename</a>
                                        <a role="menuitem" type="object" name="action_delete_passkey" class="dropdown-item">Delete</a>
                                    </t>
                                    <t t-name="card">
                                        <field name="name" class="fw-bold fs-5"/>
                                        <small>Created: <field name="create_date"/></small>
                                        <small>Last used: <field name="write_date"/></small>
                                    </t>
                                </templates>
                            </kanban>
                        </field>
                    </div>
                </group>
            </page>
        </field>
    </record>
</odoo>

```

## File: _vendor\webauthn\INFO

```text
Modified imports to be compatible, otherwise the code is the same as v2.0.0
https://github.com/duo-labs/py_webauthn/releases/tag/v2.0.0

We were not able to add the package to our requirements for the following reasons:
- Their requirements.txt required pyOpenSSL 23.3.0, which is not compatible with our requirements.
- The package is not in the debian apt repositories.

```

## File: _vendor\webauthn\LICENSE

```text
Copyright (c) 2017-2021 Duo Security, Inc. All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

```

## File: _vendor\webauthn\py.typed

```typed

```

## File: _vendor\webauthn\__init__.py

```python
from .registration.generate_registration_options import generate_registration_options
from .registration.verify_registration_response import verify_registration_response
from .authentication.generate_authentication_options import (
    generate_authentication_options,
)
from .authentication.verify_authentication_response import (
    verify_authentication_response,
)
from .helpers import base64url_to_bytes, options_to_json

__version__ = "2.0.0"

__all__ = [
    "generate_registration_options",
    "verify_registration_response",
    "generate_authentication_options",
    "verify_authentication_response",
    "base64url_to_bytes",
    "options_to_json",
]

```

## File: _vendor\webauthn\authentication\generate_authentication_options.py

```python
from typing import List, Optional

from ..helpers import generate_challenge
from ..helpers.structs import (
    PublicKeyCredentialDescriptor,
    PublicKeyCredentialRequestOptions,
    UserVerificationRequirement,
)


def generate_authentication_options(
    *,
    rp_id: str,
    challenge: Optional[bytes] = None,
    timeout: int = 60000,
    allow_credentials: Optional[List[PublicKeyCredentialDescriptor]] = None,
    user_verification: UserVerificationRequirement = UserVerificationRequirement.PREFERRED,
) -> PublicKeyCredentialRequestOptions:
    """Generate options for retrieving a credential via navigator.credentials.get()

    Args:
        `rp_id`: The Relying Party's unique identifier as specified in attestations.
        (optional) `challenge`: A byte sequence for the authenticator to return back in its response. Defaults to 64 random bytes.
        (optional) `timeout`: How long in milliseconds the browser should give the user to choose an authenticator. This value is a *hint* and may be ignored by the browser.
        (optional) `allow_credentials`: A list of credentials registered to the user.
        (optional) `user_verification`: The RP's preference for the authenticator's enforcement of the "user verified" flag.

    Returns:
        Authentication options ready for the browser. Consider using `helpers.options_to_json()` in this library to quickly convert the options to JSON.
    """

    if not rp_id:
        raise ValueError("rp_id cannot be an empty string")

    ########
    # Set defaults for required values
    ########

    if not challenge:
        challenge = generate_challenge()

    if not allow_credentials:
        allow_credentials = []

    return PublicKeyCredentialRequestOptions(
        rp_id=rp_id,
        challenge=challenge,
        timeout=timeout,
        allow_credentials=allow_credentials,
        user_verification=user_verification,
    )

```

## File: _vendor\webauthn\authentication\verify_authentication_response.py

```python
from dataclasses import dataclass
import hashlib
from typing import List, Union

from cryptography.exceptions import InvalidSignature

from ...webauthn.helpers import (
    bytes_to_base64url,
    byteslike_to_bytes,
    decode_credential_public_key,
    decoded_public_key_to_cryptography,
    parse_authenticator_data,
    parse_backup_flags,
    parse_client_data_json,
    parse_authentication_credential_json,
    verify_signature,
)
from ...webauthn.helpers.exceptions import InvalidAuthenticationResponse
from ...webauthn.helpers.structs import (
    AuthenticationCredential,
    ClientDataType,
    CredentialDeviceType,
    PublicKeyCredentialType,
    TokenBindingStatus,
)


@dataclass
class VerifiedAuthentication:
    """
    Information about a verified authentication of which an RP can make use
    """

    credential_id: bytes
    new_sign_count: int
    credential_device_type: CredentialDeviceType
    credential_backed_up: bool


expected_token_binding_statuses = [
    TokenBindingStatus.SUPPORTED,
    TokenBindingStatus.PRESENT,
]


def verify_authentication_response(
    *,
    credential: Union[str, dict, AuthenticationCredential],
    expected_challenge: bytes,
    expected_rp_id: str,
    expected_origin: Union[str, List[str]],
    credential_public_key: bytes,
    credential_current_sign_count: int,
    require_user_verification: bool = False,
) -> VerifiedAuthentication:
    """Verify a response from navigator.credentials.get()

    Args:
        - `credential`: The value returned from `navigator.credentials.get()`. Can be either a
          stringified JSON object, a plain dict, or an instance of RegistrationCredential
        - `expected_challenge`: The challenge passed to the authenticator within the preceding
          authentication options.
        - `expected_rp_id`: The Relying Party's unique identifier as specified in the preceding
          authentication options.
        - `expected_origin`: The domain, with HTTP protocol (e.g. "https://domain.here"), on which
          the authentication ceremony should have occurred.
        - `credential_public_key`: The public key for the credential's ID as provided in a
          preceding authenticator registration ceremony.
        - `credential_current_sign_count`: The current known number of times the authenticator was
          used.
        - (optional) `require_user_verification`: Whether or not to require that the authenticator
          verified the user.

    Returns:
        Information about the authenticator

    Raises:
        `helpers.exceptions.InvalidAuthenticationResponse` if the response cannot be verified
    """
    if isinstance(credential, str) or isinstance(credential, dict):
        credential = parse_authentication_credential_json(credential)

    # FIDO-specific check
    if bytes_to_base64url(credential.raw_id) != credential.id:
        raise InvalidAuthenticationResponse("id and raw_id were not equivalent")

    # FIDO-specific check
    if credential.type != PublicKeyCredentialType.PUBLIC_KEY:
        raise InvalidAuthenticationResponse(
            f'Unexpected credential type "{credential.type}", expected "public-key"'
        )

    response = credential.response

    client_data_bytes = byteslike_to_bytes(response.client_data_json)
    authenticator_data_bytes = byteslike_to_bytes(response.authenticator_data)
    signature_bytes = byteslike_to_bytes(response.signature)

    client_data = parse_client_data_json(client_data_bytes)

    if client_data.type != ClientDataType.WEBAUTHN_GET:
        raise InvalidAuthenticationResponse(
            f'Unexpected client data type "{client_data.type}", expected "{ClientDataType.WEBAUTHN_GET}"'
        )

    if expected_challenge != client_data.challenge:
        raise InvalidAuthenticationResponse("Client data challenge was not expected challenge")

    if isinstance(expected_origin, str):
        if expected_origin != client_data.origin:
            raise InvalidAuthenticationResponse(
                f'Unexpected client data origin "{client_data.origin}", expected "{expected_origin}"'
            )
    else:
        try:
            expected_origin.index(client_data.origin)
        except ValueError:
            raise InvalidAuthenticationResponse(
                f'Unexpected client data origin "{client_data.origin}", expected one of {expected_origin}'
            )

    if client_data.token_binding:
        status = client_data.token_binding.status
        if status not in expected_token_binding_statuses:
            raise InvalidAuthenticationResponse(
                f'Unexpected token_binding status of "{status}", expected one of "{",".join(expected_token_binding_statuses)}"'
            )

    auth_data = parse_authenticator_data(authenticator_data_bytes)

    # Generate a hash of the expected RP ID for comparison
    expected_rp_id_hash = hashlib.sha256()
    expected_rp_id_hash.update(expected_rp_id.encode("utf-8"))
    expected_rp_id_hash_bytes = expected_rp_id_hash.digest()

    if auth_data.rp_id_hash != expected_rp_id_hash_bytes:
        raise InvalidAuthenticationResponse("Unexpected RP ID hash")

    if not auth_data.flags.up:
        raise InvalidAuthenticationResponse("User was not present during authentication")

    if require_user_verification and not auth_data.flags.uv:
        raise InvalidAuthenticationResponse(
            "User verification is required but user was not verified during authentication"
        )

    if (
        auth_data.sign_count > 0 or credential_current_sign_count > 0
    ) and auth_data.sign_count <= credential_current_sign_count:
        # Require the sign count to have been incremented over what was reported by the
        # authenticator the last time this credential was used, otherwise this might be
        # a replay attack
        raise InvalidAuthenticationResponse(
            f"Response sign count of {auth_data.sign_count} was not greater than current count of {credential_current_sign_count}"
        )

    client_data_hash = hashlib.sha256()
    client_data_hash.update(client_data_bytes)
    client_data_hash_bytes = client_data_hash.digest()

    signature_base = authenticator_data_bytes + client_data_hash_bytes

    try:
        decoded_public_key = decode_credential_public_key(credential_public_key)
        crypto_public_key = decoded_public_key_to_cryptography(decoded_public_key)

        verify_signature(
            public_key=crypto_public_key,
            signature_alg=decoded_public_key.alg,
            signature=signature_bytes,
            data=signature_base,
        )
    except InvalidSignature:
        raise InvalidAuthenticationResponse("Could not verify authentication signature")

    parsed_backup_flags = parse_backup_flags(auth_data.flags)

    return VerifiedAuthentication(
        credential_id=credential.raw_id,
        new_sign_count=auth_data.sign_count,
        credential_device_type=parsed_backup_flags.credential_device_type,
        credential_backed_up=parsed_backup_flags.credential_backed_up,
    )

```

## File: _vendor\webauthn\authentication\__init__.py

```python

```

## File: _vendor\webauthn\helpers\aaguid_to_string.py

```python
import codecs


def aaguid_to_string(val: bytes) -> str:
    """
    Take aaguid bytes and convert them to a GUID string
    """
    if len(val) != 16:
        raise ValueError(f"AAGUID was {len(val)} bytes, expected 16 bytes")

    # Convert to a hexadecimal string representation
    to_hex = codecs.encode(val, encoding="hex").decode("utf-8")

    # Split up the hex string into segments
    # 8 chars
    seg_1 = to_hex[0:8]
    # 4 chars
    seg_2 = to_hex[8:12]
    # 4 chars
    seg_3 = to_hex[12:16]
    # 4 chars
    seg_4 = to_hex[16:20]
    # 12 chars
    seg_5 = to_hex[20:32]

    # "00000000-0000-0000-0000-000000000000"
    return f"{seg_1}-{seg_2}-{seg_3}-{seg_4}-{seg_5}"

```

## File: _vendor\webauthn\helpers\algorithms.py

```python
from cryptography.hazmat.primitives.asymmetric.ec import (
    ECDSA,
    SECP256R1,
    SECP384R1,
    SECP521R1,
    EllipticCurve,
    EllipticCurveSignatureAlgorithm,
)
from cryptography.hazmat.primitives.hashes import (
    SHA1,
    SHA256,
    SHA384,
    SHA512,
    HashAlgorithm,
)

from .cose import COSECRV, COSEAlgorithmIdentifier
from .exceptions import UnsupportedAlgorithm, UnsupportedEC2Curve


def is_rsa_pkcs(alg_id: COSEAlgorithmIdentifier) -> bool:
    """Determine if the specified COSE algorithm ID denotes an RSA PKCSv1 public key"""
    return alg_id in (
        COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_1,
        COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_256,
        COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_384,
        COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_512,
    )


def is_rsa_pss(alg_id: COSEAlgorithmIdentifier) -> bool:
    """Determine if the specified COSE algorithm ID denotes an RSA PSS public key"""
    return alg_id in (
        COSEAlgorithmIdentifier.RSASSA_PSS_SHA_256,
        COSEAlgorithmIdentifier.RSASSA_PSS_SHA_384,
        COSEAlgorithmIdentifier.RSASSA_PSS_SHA_512,
    )


def get_ec2_sig_alg(alg_id: COSEAlgorithmIdentifier) -> EllipticCurveSignatureAlgorithm:
    """Turn an "ECDSA" COSE algorithm identifier into a corresponding signature
    algorithm
    """
    if alg_id == COSEAlgorithmIdentifier.ECDSA_SHA_256:
        return ECDSA(SHA256())
    if alg_id == COSEAlgorithmIdentifier.ECDSA_SHA_512:
        return ECDSA(SHA512())

    raise UnsupportedAlgorithm(f"Unrecognized EC2 signature alg {alg_id}")


def get_ec2_curve(crv_id: COSECRV) -> EllipticCurve:
    """Turn an EC2 COSE crv identifier into a corresponding curve"""
    if crv_id == COSECRV.P256:
        return SECP256R1()
    elif crv_id == COSECRV.P384:
        return SECP384R1()
    elif crv_id == COSECRV.P521:
        return SECP521R1()

    raise UnsupportedEC2Curve(f"Unrecognized EC2 curve {crv_id}")


def get_rsa_pkcs1_sig_alg(alg_id: COSEAlgorithmIdentifier) -> HashAlgorithm:
    """Turn an "RSASSA_PKCS1" COSE algorithm identifier into a corresponding signature
    algorithm
    """
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_1:
        return SHA1()
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_256:
        return SHA256()
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_384:
        return SHA384()
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_512:
        return SHA512()

    raise UnsupportedAlgorithm(f"Unrecognized RSA PKCS1 signature alg {alg_id}")


def get_rsa_pss_sig_alg(alg_id: COSEAlgorithmIdentifier) -> HashAlgorithm:
    """Turn an "RSASSA_PSS" COSE algorithm identifier into a corresponding signature
    algorithm
    """
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PSS_SHA_256:
        return SHA256()
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PSS_SHA_384:
        return SHA384()
    if alg_id == COSEAlgorithmIdentifier.RSASSA_PSS_SHA_512:
        return SHA512()

    raise UnsupportedAlgorithm(f"Unrecognized RSA PSS signature alg {alg_id}")

```

## File: _vendor\webauthn\helpers\base64url_to_bytes.py

```python
from base64 import urlsafe_b64decode


def base64url_to_bytes(val: str) -> bytes:
    """
    Convert a Base64URL-encoded string to bytes.
    """
    # Padding is optional in Base64URL. Unfortunately, Python's decoder requires the
    # padding. Given the fact that urlsafe_b64decode will ignore too _much_ padding,
    # we can tack on a constant amount of padding to ensure encoded values can always be
    # decoded.
    return urlsafe_b64decode(f"{val}===")

```

## File: _vendor\webauthn\helpers\byteslike_to_bytes.py

```python
from typing import Union


def byteslike_to_bytes(val: Union[bytes, memoryview]) -> bytes:
    """
    Massage bytes subclasses into bytes for ease of concatenation, comparison, etc...
    """
    if isinstance(val, memoryview):
        val = val.tobytes()

    return bytes(val)

```

## File: _vendor\webauthn\helpers\bytes_to_base64url.py

```python
from base64 import urlsafe_b64encode


def bytes_to_base64url(val: bytes) -> str:
    """
    Base64URL-encode the provided bytes
    """
    return urlsafe_b64encode(val).decode("utf-8").rstrip("=")

```

## File: _vendor\webauthn\helpers\cose.py

```python
from enum import Enum


class COSEAlgorithmIdentifier(int, Enum):
    """Various registered values indicating cryptographic algorithms that may be used in credential responses

    Members:
        `ECDSA_SHA_256`
        `EDDSA`
        `ECDSA_SHA_512`
        `RSASSA_PSS_SHA_256`
        `RSASSA_PSS_SHA_384`
        `RSASSA_PSS_SHA_512`
        `RSASSA_PKCS1_v1_5_SHA_256`
        `RSASSA_PKCS1_v1_5_SHA_384`
        `RSASSA_PKCS1_v1_5_SHA_512`
        `RSASSA_PKCS1_v1_5_SHA_1`

    https://www.w3.org/TR/webauthn-2/#sctn-alg-identifier
    https://www.iana.org/assignments/cose/cose.xhtml#algorithms
    """

    ECDSA_SHA_256 = -7
    EDDSA = -8
    ECDSA_SHA_512 = -36
    RSASSA_PSS_SHA_256 = -37
    RSASSA_PSS_SHA_384 = -38
    RSASSA_PSS_SHA_512 = -39
    RSASSA_PKCS1_v1_5_SHA_256 = -257
    RSASSA_PKCS1_v1_5_SHA_384 = -258
    RSASSA_PKCS1_v1_5_SHA_512 = -259
    RSASSA_PKCS1_v1_5_SHA_1 = -65535  # Deprecated; here for legacy support


class COSEKTY(int, Enum):
    """
    Possible values for COSEKey.KTY representing a public key's key type

    https://tools.ietf.org/html/rfc8152#section-13
    https://www.iana.org/assignments/cose/cose.xhtml#table-key-type
    """

    OKP = 1
    EC2 = 2
    RSA = 3


class COSECRV(int, Enum):
    """Possible values for COSEKey.CRV representing an EC2 public key's curve

    https://tools.ietf.org/html/rfc8152#section-13.1
    https://www.iana.org/assignments/cose/cose.xhtml#table-elliptic-curves
    """

    P256 = 1  # EC2, NIST P-256 also known as secp256r1
    P384 = 2  # EC2, NIST P-384 also known as secp384r1
    P521 = 3  # EC2, NIST P-521 also known as secp521r1
    ED25519 = 6  # OKP, Ed25519 for use w/ EdDSA only


class COSEKey(int, Enum):
    """
    COSE keys for public keys

    https://tools.ietf.org/html/rfc8152
    https://www.iana.org/assignments/cose/cose.xhtml#table-key-common-parameters
    https://www.iana.org/assignments/cose/cose.xhtml#table-key-type-parameters
    """

    KTY = 1
    ALG = 3
    # EC2, OKP
    CRV = -1
    X = -2
    # EC2
    Y = -3
    # RSA
    N = -1
    E = -2

```

## File: _vendor\webauthn\helpers\decoded_public_key_to_cryptography.py

```python
import codecs
from typing import Union

from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric.ec import (
    EllipticCurvePublicKey,
    EllipticCurvePublicNumbers,
)
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey
from cryptography.hazmat.primitives.asymmetric.rsa import RSAPublicKey, RSAPublicNumbers

from .algorithms import get_ec2_curve
from .cose import COSECRV, COSEAlgorithmIdentifier
from .decode_credential_public_key import (
    DecodedEC2PublicKey,
    DecodedOKPPublicKey,
    DecodedRSAPublicKey,
)
from .exceptions import UnsupportedPublicKey


def decoded_public_key_to_cryptography(
    public_key: Union[DecodedOKPPublicKey, DecodedEC2PublicKey, DecodedRSAPublicKey]
) -> Union[Ed25519PublicKey, EllipticCurvePublicKey, RSAPublicKey]:
    """Convert raw decoded public key parameters (crv, x, y, n, e, etc...) into
    public keys using primitives from the cryptography.io library
    """
    if isinstance(public_key, DecodedEC2PublicKey):
        """
        alg is -7 (ES256), where kty is 2 (with uncompressed points) and
        crv is 1 (P-256).
        https://www.w3.org/TR/webauthn-2/#sctn-public-key-easy
        """
        x = int(codecs.encode(public_key.x, "hex"), 16)
        y = int(codecs.encode(public_key.y, "hex"), 16)
        curve = get_ec2_curve(public_key.crv)

        ecc_pub_key = EllipticCurvePublicNumbers(x, y, curve).public_key(default_backend())

        return ecc_pub_key
    elif isinstance(public_key, DecodedRSAPublicKey):
        """
        alg is -257 (RS256)
        https://www.w3.org/TR/webauthn-2/#sctn-public-key-easy
        """
        e = int(codecs.encode(public_key.e, "hex"), 16)
        n = int(codecs.encode(public_key.n, "hex"), 16)

        rsa_pub_key = RSAPublicNumbers(e, n).public_key(default_backend())

        return rsa_pub_key
    elif isinstance(public_key, DecodedOKPPublicKey):
        """
        -8 (EdDSA), where crv is 6 (Ed25519).
        https://www.w3.org/TR/webauthn-2/#sctn-public-key-easy
        """
        if public_key.alg != COSEAlgorithmIdentifier.EDDSA or public_key.crv != COSECRV.ED25519:
            raise UnsupportedPublicKey(
                f"OKP public key with alg {public_key.alg} and crv {public_key.crv} is not supported"
            )

        okp_pub_key = Ed25519PublicKey.from_public_bytes(public_key.x)

        return okp_pub_key
    else:
        raise UnsupportedPublicKey(f"Unrecognized decoded public key: {public_key}")

```

## File: _vendor\webauthn\helpers\decode_credential_public_key.py

```python
from typing import Union
from dataclasses import dataclass

import cbor2

from .cose import COSECRV, COSEKTY, COSEAlgorithmIdentifier, COSEKey
from .exceptions import InvalidPublicKeyStructure, UnsupportedPublicKeyType
from .parse_cbor import parse_cbor


@dataclass
class DecodedOKPPublicKey:
    kty: COSEKTY
    alg: COSEAlgorithmIdentifier
    crv: COSECRV
    x: bytes


@dataclass
class DecodedEC2PublicKey:
    kty: COSEKTY
    alg: COSEAlgorithmIdentifier
    crv: COSECRV
    x: bytes
    y: bytes


@dataclass
class DecodedRSAPublicKey:
    kty: COSEKTY
    alg: COSEAlgorithmIdentifier
    n: bytes
    e: bytes


def decode_credential_public_key(
    key: bytes,
) -> Union[DecodedOKPPublicKey, DecodedEC2PublicKey, DecodedRSAPublicKey]:
    """
    Decode a CBOR-encoded public key and turn it into a data structure.

    Supports OKP, EC2, and RSA public keys
    """
    # Occassionally we might be given a public key in an "uncompressed" format,
    # typically from older U2F security keys. As per the FIDO spec this is indicated by
    # a leading 0x04 "uncompressed point compression method" format byte. In that case
    # we need to fill in some blanks to turn it into a full EC2 key for signature
    # verification
    #
    # See https://fidoalliance.org/specs/fido-v2.0-id-20180227/fido-registry-v2.0-id-20180227.html#public-key-representation-formats
    if key[0] == 0x04:
        return DecodedEC2PublicKey(
            kty=COSEKTY.EC2,
            alg=COSEAlgorithmIdentifier.ECDSA_SHA_256,
            crv=COSECRV.P256,
            x=key[1:33],
            y=key[33:65],
        )

    decoded_key: dict = parse_cbor(key)

    kty = decoded_key[COSEKey.KTY]
    alg = decoded_key[COSEKey.ALG]

    if not kty:
        raise InvalidPublicKeyStructure("Credential public key missing kty")
    if not alg:
        raise InvalidPublicKeyStructure("Credential public key missing alg")

    if kty == COSEKTY.OKP:
        crv = decoded_key[COSEKey.CRV]
        x = decoded_key[COSEKey.X]

        if not crv:
            raise InvalidPublicKeyStructure("OKP credential public key missing crv")
        if not x:
            raise InvalidPublicKeyStructure("OKP credential public key missing x")

        return DecodedOKPPublicKey(
            kty=kty,
            alg=alg,
            crv=crv,
            x=x,
        )
    elif kty == COSEKTY.EC2:
        crv = decoded_key[COSEKey.CRV]
        x = decoded_key[COSEKey.X]
        y = decoded_key[COSEKey.Y]

        if not crv:
            raise InvalidPublicKeyStructure("EC2 credential public key missing crv")
        if not x:
            raise InvalidPublicKeyStructure("EC2 credential public key missing x")
        if not y:
            raise InvalidPublicKeyStructure("EC2 credential public key missing y")

        return DecodedEC2PublicKey(
            kty=kty,
            alg=alg,
            crv=crv,
            x=x,
            y=y,
        )
    elif kty == COSEKTY.RSA:
        n = decoded_key[COSEKey.N]
        e = decoded_key[COSEKey.E]

        if not n:
            raise InvalidPublicKeyStructure("RSA credential public key missing n")
        if not e:
            raise InvalidPublicKeyStructure("RSA credential public key missing e")

        return DecodedRSAPublicKey(
            kty=kty,
            alg=alg,
            n=n,
            e=e,
        )

    raise UnsupportedPublicKeyType(f'Unsupported credential public key type "{kty}"')

```

## File: _vendor\webauthn\helpers\encode_cbor.py

```python
from typing import Any

import cbor2

from .exceptions import InvalidCBORData


def encode_cbor(val: Any) -> bytes:
    """
    Attempt to encode data into CBOR.

    Raises:
        `helpers.exceptions.InvalidCBORData` if data cannot be decoded
    """
    try:
        to_return = cbor2.dumps(val)
    except Exception as exc:
        raise InvalidCBORData("Data could not be encoded to CBOR") from exc

    return to_return

```

## File: _vendor\webauthn\helpers\exceptions.py

```python
class InvalidRegistrationResponse(Exception):
    pass


class InvalidAuthenticationResponse(Exception):
    pass


class InvalidPublicKeyStructure(Exception):
    pass


class UnsupportedPublicKeyType(Exception):
    pass


class InvalidJSONStructure(Exception):
    pass


class InvalidAuthenticatorDataStructure(Exception):
    pass


class SignatureVerificationException(Exception):
    pass


class UnsupportedAlgorithm(Exception):
    pass


class UnsupportedPublicKey(Exception):
    pass


class UnsupportedEC2Curve(Exception):
    pass


class InvalidTPMPubAreaStructure(Exception):
    pass


class InvalidTPMCertInfoStructure(Exception):
    pass


class InvalidCertificateChain(Exception):
    pass


class InvalidBackupFlags(Exception):
    pass


class InvalidCBORData(Exception):
    pass

```

## File: _vendor\webauthn\helpers\generate_challenge.py

```python
import secrets


def generate_challenge() -> bytes:
    """
    Create a random value for the authenticator to sign, going above and beyond the recommended
    number of random bytes as per https://www.w3.org/TR/webauthn-2/#sctn-cryptographic-challenges:

    "In order to prevent replay attacks, the challenges MUST contain enough entropy to make
    guessing them infeasible. Challenges SHOULD therefore be at least 16 bytes long."
    """
    return secrets.token_bytes(64)

```

## File: _vendor\webauthn\helpers\generate_user_handle.py

```python
import secrets


def generate_user_handle() -> bytes:
    """
    Convenience method RP's can use to generate a privacy-preserving random sequence of
    bytes as per best practices defined in the WebAuthn spec. This value is intended to
    be used as the value of `user_id` when calling `generate_registration_options()`,
    and can then be used during authentication verification to match the credential to
    a user.

    See https://www.w3.org/TR/webauthn-2/#sctn-user-handle-privacy:

    "It is RECOMMENDED to let the user handle be 64 random bytes, and store this value
    in the user's account."
    """
    return secrets.token_bytes(64)

```

## File: _vendor\webauthn\helpers\hash_by_alg.py

```python
import hashlib
from typing import Optional

from .cose import COSEAlgorithmIdentifier

SHA_256 = [
    COSEAlgorithmIdentifier.ECDSA_SHA_256,
    COSEAlgorithmIdentifier.RSASSA_PSS_SHA_256,
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_256,
]
SHA_384 = [
    COSEAlgorithmIdentifier.RSASSA_PSS_SHA_384,
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_384,
]
SHA_512 = [
    COSEAlgorithmIdentifier.ECDSA_SHA_512,
    COSEAlgorithmIdentifier.RSASSA_PSS_SHA_512,
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_512,
]
SHA_1 = [
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_1,
]


def hash_by_alg(to_hash: bytes, alg: Optional[COSEAlgorithmIdentifier] = None) -> bytes:
    """
    Generate a hash of `to_hash` by the specified COSE algorithm ID. Defaults to hashing
    with SHA256
    """
    # Default to SHA256 for hashing
    hash = hashlib.sha256()

    if alg in SHA_384:
        hash = hashlib.sha384()
    elif alg in SHA_512:
        hash = hashlib.sha512()
    elif alg in SHA_1:
        hash = hashlib.sha1()

    hash.update(to_hash)
    return hash.digest()

```

## File: _vendor\webauthn\helpers\known_root_certs.py

```python
#######################################
#
# Google Hardware Attestation Root 1
#
# Downloaded from https://developer.android.com/training/articles/security-key-attestation#root_certificate
# (first entry)
#
# Valid until 2026-05-24 @ 09:28 PST
#
# SHA256 Fingerprint
# C1:98:4A:3E:F4:5C:1E:2A:91:85:51:DE:10:60:3C:86:F7:05:1B:22:49:C4:89:1C:AE:32:30:EA:BD:0C:97:D5
#
#######################################
google_hardware_attestation_root_1 = """-----BEGIN CERTIFICATE-----
MIIFYDCCA0igAwIBAgIJAOj6GWMU0voYMA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNV
BAUTEGY5MjAwOWU4NTNiNmIwNDUwHhcNMTYwNTI2MTYyODUyWhcNMjYwNTI0MTYy
ODUyWjAbMRkwFwYDVQQFExBmOTIwMDllODUzYjZiMDQ1MIICIjANBgkqhkiG9w0B
AQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xUFmOr75gvMsd/dTEDDJdS
Sxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5jlRfdnJLmN0pTy/4lj4/7
tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y//0rb+T+W8a9nsNL/ggj
nar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73XpXyTqRxB/M0n1n/W9nGq
C4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYImQQcHtGl/m00QLVWutHQ
oVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB+TxywElgS70vE0XmLD+O
JtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7quvmag8jfPioyKvxnK/Eg
sTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgpZrt3i5MIlCaY504LzSRi
igHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7gLiMm0jhO2B6tUXHI/+M
RPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82ixPvZtXQpUpuL12ab+9E
aDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+NpUFgNPN9PvQi8WEg5Um
AGMCAwEAAaOBpjCBozAdBgNVHQ4EFgQUNmHhAHyIBQlRi0RsR/8aTMnqTxIwHwYD
VR0jBBgwFoAUNmHhAHyIBQlRi0RsR/8aTMnqTxIwDwYDVR0TAQH/BAUwAwEB/zAO
BgNVHQ8BAf8EBAMCAYYwQAYDVR0fBDkwNzA1oDOgMYYvaHR0cHM6Ly9hbmRyb2lk
Lmdvb2dsZWFwaXMuY29tL2F0dGVzdGF0aW9uL2NybC8wDQYJKoZIhvcNAQELBQAD
ggIBACDIw41L3KlXG0aMiS//cqrG+EShHUGo8HNsw30W1kJtjn6UBwRM6jnmiwfB
Pb8VA91chb2vssAtX2zbTvqBJ9+LBPGCdw/E53Rbf86qhxKaiAHOjpvAy5Y3m00m
qC0w/Zwvju1twb4vhLaJ5NkUJYsUS7rmJKHHBnETLi8GFqiEsqTWpG/6ibYCv7rY
DBJDcR9W62BW9jfIoBQcxUCUJouMPH25lLNcDc1ssqvC2v7iUgI9LeoM1sNovqPm
QUiG9rHli1vXxzCyaMTjwftkJLkf6724DFhuKug2jITV0QkXvaJWF4nUaHOTNA4u
JU9WDvZLI1j83A+/xnAJUucIv/zGJ1AMH2boHqF8CY16LpsYgBt6tKxxWH00XcyD
CdW2KlBCeqbQPcsFmWyWugxdcekhYsAWyoSf818NUsZdBWBaR/OukXrNLfkQ79Iy
ZohZbvabO/X+MVT3rriAoKc8oE2Uws6DF+60PV7/WIPjNvXySdqspImSN78mflxD
qwLqRBYkA3I75qppLGG9rp7UCdRjxMl8ZDBld+7yvHVgt1cVzJx9xnyGCC23Uaic
MDSXYrB4I4WHXPGjxhZuCuPBLTdOLU8YRvMYdEvYebWHMpvwGCF6bAx3JBpIeOQ1
wDB5y0USicV3YgYGmi+NZfhA4URSh77Yd6uuJOJENRaNVTzk
-----END CERTIFICATE-----
""".encode(
    "ascii"
)

#######################################
#
# Google Hardware Attestation Root 2
#
# Downloaded from https://developer.android.com/training/articles/security-key-attestation#root_certificate
# (second entry)
#
# Valid until 2034-11-18 @ 12:37 PST
#
# SHA256 Fingerprint
# 1E:F1:A0:4B:8B:A5:8A:B9:45:89:AC:49:8C:89:82:A7:83:F2:4E:A7:30:7E:01:59:A0:C3:A7:3B:37:7D:87:CC
#
#######################################
google_hardware_attestation_root_2 = """-----BEGIN CERTIFICATE-----
MIIFHDCCAwSgAwIBAgIJANUP8luj8tazMA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNV
BAUTEGY5MjAwOWU4NTNiNmIwNDUwHhcNMTkxMTIyMjAzNzU4WhcNMzQxMTE4MjAz
NzU4WjAbMRkwFwYDVQQFExBmOTIwMDllODUzYjZiMDQ1MIICIjANBgkqhkiG9w0B
AQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xUFmOr75gvMsd/dTEDDJdS
Sxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5jlRfdnJLmN0pTy/4lj4/7
tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y//0rb+T+W8a9nsNL/ggj
nar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73XpXyTqRxB/M0n1n/W9nGq
C4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYImQQcHtGl/m00QLVWutHQ
oVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB+TxywElgS70vE0XmLD+O
JtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7quvmag8jfPioyKvxnK/Eg
sTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgpZrt3i5MIlCaY504LzSRi
igHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7gLiMm0jhO2B6tUXHI/+M
RPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82ixPvZtXQpUpuL12ab+9E
aDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+NpUFgNPN9PvQi8WEg5Um
AGMCAwEAAaNjMGEwHQYDVR0OBBYEFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMB8GA1Ud
IwQYMBaAFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMA8GA1UdEwEB/wQFMAMBAf8wDgYD
VR0PAQH/BAQDAgIEMA0GCSqGSIb3DQEBCwUAA4ICAQBOMaBc8oumXb2voc7XCWnu
XKhBBK3e2KMGz39t7lA3XXRe2ZLLAkLM5y3J7tURkf5a1SutfdOyXAmeE6SRo83U
h6WszodmMkxK5GM4JGrnt4pBisu5igXEydaW7qq2CdC6DOGjG+mEkN8/TA6p3cno
L/sPyz6evdjLlSeJ8rFBH6xWyIZCbrcpYEJzXaUOEaxxXxgYz5/cTiVKN2M1G2ok
QBUIYSY6bjEL4aUN5cfo7ogP3UvliEo3Eo0YgwuzR2v0KR6C1cZqZJSTnghIC/vA
D32KdNQ+c3N+vl2OTsUVMC1GiWkngNx1OO1+kXW+YTnnTUOtOIswUP/Vqd5SYgAI
mMAfY8U9/iIgkQj6T2W6FsScy94IN9fFhE1UtzmLoBIuUFsVXJMTz+Jucth+IqoW
Fua9v1R93/k98p41pjtFX+H8DslVgfP097vju4KDlqN64xV1grw3ZLl4CiOe/A91
oeLm2UHOq6wn3esB4r2EIQKb6jTVGu5sYCcdWpXr0AUVqcABPdgL+H7qJguBw09o
jm6xNIrw2OocrDKsudk/okr/AwqEyPKw9WnMlQgLIKw1rODG2NvU9oR3GVGdMkUB
ZutL8VuFkERQGt6vQ2OCw0sV47VMkuYbacK/xyZFiRcrPJPb41zgbQj9XAEyLKCH
ex0SdDrx+tWUDqG8At2JHA==
-----END CERTIFICATE-----
""".encode(
    "ascii"
)

#######################################
#
# GlobalSign Root CA
#
# Downloaded from https://pki.goog/roots.pem
#
# Valid until 2028-01-28 @ 04:00 PST
#
# SHA256 Fingerprint
# EB:D4:10:40:E4:BB:3E:C7:42:C9:E3:81:D3:1E:F2:A4:1A:48:B6:68:5C:96:E7:CE:F3:C1:DF:6C:D4:33:1C:99
#
#######################################
globalsign_root_ca = """-----BEGIN CERTIFICATE-----
MIIDdTCCAl2gAwIBAgILBAAAAAABFUtaw5QwDQYJKoZIhvcNAQEFBQAwVzELMAkG
A1UEBhMCQkUxGTAXBgNVBAoTEEdsb2JhbFNpZ24gbnYtc2ExEDAOBgNVBAsTB1Jv
b3QgQ0ExGzAZBgNVBAMTEkdsb2JhbFNpZ24gUm9vdCBDQTAeFw05ODA5MDExMjAw
MDBaFw0yODAxMjgxMjAwMDBaMFcxCzAJBgNVBAYTAkJFMRkwFwYDVQQKExBHbG9i
YWxTaWduIG52LXNhMRAwDgYDVQQLEwdSb290IENBMRswGQYDVQQDExJHbG9iYWxT
aWduIFJvb3QgQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDaDuaZ
jc6j40+Kfvvxi4Mla+pIH/EqsLmVEQS98GPR4mdmzxzdzxtIK+6NiY6arymAZavp
xy0Sy6scTHAHoT0KMM0VjU/43dSMUBUc71DuxC73/OlS8pF94G3VNTCOXkNz8kHp
1Wrjsok6Vjk4bwY8iGlbKk3Fp1S4bInMm/k8yuX9ifUSPJJ4ltbcdG6TRGHRjcdG
snUOhugZitVtbNV4FpWi6cgKOOvyJBNPc1STE4U6G7weNLWLBYy5d4ux2x8gkasJ
U26Qzns3dLlwR5EiUWMWea6xrkEmCMgZK9FGqkjWZCrXgzT/LCrBbBlDSgeF59N8
9iFo7+ryUp9/k5DPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8E
BTADAQH/MB0GA1UdDgQWBBRge2YaRQ2XyolQL30EzTSo//z9SzANBgkqhkiG9w0B
AQUFAAOCAQEA1nPnfE920I2/7LqivjTFKDK1fPxsnCwrvQmeU79rXqoRSLblCKOz
yj1hTdNGCbM+w6DjY1Ub8rrvrTnhQ7k4o+YviiY776BQVvnGCv04zcQLcFGUl5gE
38NflNUVyRRBnMRddWQVDf9VMOyGj/8N7yy5Y0b2qvzfvGn9LhJIZJrglfCm7ymP
AbEVtQwdpf5pLGkkeB6zpxxxYu7KyJesF12KwvhHhm4qxFYxldBniYUr+WymXUad
DKqC5JlR3XC321Y9YeRq4VzW9v493kHMB65jUr9TU/Qr6cf9tveCX4XSQRjbgbME
HMUfpIBvFSDJ3gyICh3WZlXi/EjJKSZp4A==
-----END CERTIFICATE-----
""".encode(
    "ascii"
)

#######################################
#
# GlobalSign R2
#
# Downloaded from https://pki.goog/repo/certs/gsr2.pem
#
# Valid until 2021-12-15 @ 00:00 PST
#
# SHA256 Fingerprint
# 69:E2:D0:6C:30:F3:66:16:61:65:E9:1D:68:D1:CE:E5:CC:47:58:4A:80:22:7E:76:66:60:86:C0:10:72:41:EB
#
#######################################
globalsign_r2 = """-----BEGIN CERTIFICATE-----
MIIDvDCCAqSgAwIBAgINAgPk9GHsmdnVeWbKejANBgkqhkiG9w0BAQUFADBMMSAw
HgYDVQQLExdHbG9iYWxTaWduIFJvb3QgQ0EgLSBSMjETMBEGA1UEChMKR2xvYmFs
U2lnbjETMBEGA1UEAxMKR2xvYmFsU2lnbjAeFw0wNjEyMTUwODAwMDBaFw0yMTEy
MTUwODAwMDBaMEwxIDAeBgNVBAsTF0dsb2JhbFNpZ24gUm9vdCBDQSAtIFIyMRMw
EQYDVQQKEwpHbG9iYWxTaWduMRMwEQYDVQQDEwpHbG9iYWxTaWduMIIBIjANBgkq
hkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAps8kDr4ubyiZRULEqz4hVJsL03+EcPoS
s8u/h1/Gf4bTsjBc1v2t8Xvc5fhglgmSEPXQU977e35ziKxSiHtKpspJpl6op4xa
Ebx6guu+jOmzrJYlB5dKmSoHL7Qed7+KD7UCfBuWuMW5Oiy81hK561l94tAGhl9e
SWq1OV6INOy8eAwImIRsqM1LtKB9DHlN8LgtyyHK1WxbfeGgKYSh+dOUScskYpEg
vN0L1dnM+eonCitzkcadG6zIy+jgoPQvkItN+7A2G/YZeoXgbfJhE4hcn+CTClGX
ilrOr6vV96oJqmC93Nlf33KpYBNeAAHJSvo/pOoHAyECjoLKA8KbjwIDAQABo4Gc
MIGZMA4GA1UdDwEB/wQEAwIBhjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBSb
4gdXZxwewGoG3lm0mi3f3BmGLjAfBgNVHSMEGDAWgBSb4gdXZxwewGoG3lm0mi3f
3BmGLjA2BgNVHR8ELzAtMCugKaAnhiVodHRwOi8vY3JsLmdsb2JhbHNpZ24ubmV0
L3Jvb3QtcjIuY3JsMA0GCSqGSIb3DQEBBQUAA4IBAQANeX81Z1YqDIs4EaLjG0qP
OxIzaJI/y4kiRj3a+y3KOx74clIkLuMgi/9/5iv/n+1LyhGU9g7174slbzJOPbSp
p1eT19ST2mYbdgTLx/hm3tTLoHIY/w4ZbnQYwfnPwAG4RefnEFYPQJmpD+Wh8BJw
Bgtm2drTale/T6NBwmwnEFunfaMfMX3g6IBrx7VKnxIkJh/3p190WveLKgl9n7i5
SWce/4woPimEn9WfEQWRvp6wKhaCKFjuCMuulEZusoOUJ4LfJnXxcuQTgIrSnwI7
KfSSjsd42w3lX1fbgJp7vPmLM6OBRvAXuYRKTFqMAWbb7OaGIEE+cbxY6PDepnva
-----END CERTIFICATE-----
""".encode(
    "ascii"
)

#######################################
#
# Apple WebAuthn Root CA
#
# Downloaded from https://www.apple.com/certificateauthority/Apple_WebAuthn_Root_CA.pem
#
# Valid until 2045-03-14 @ 17:00 PST
#
# SHA256 Fingerprint
# 09:15:DD:5C:07:A2:8D:B5:49:D1:F6:77:BB:5A:75:D4:BF:BE:95:61:A7:73:42:43:27:76:2E:9E:02:F9:BB:29
#
#######################################
apple_webauthn_root_ca = """-----BEGIN CERTIFICATE-----
MIICEjCCAZmgAwIBAgIQaB0BbHo84wIlpQGUKEdXcTAKBggqhkjOPQQDAzBLMR8w
HQYDVQQDDBZBcHBsZSBXZWJBdXRobiBSb290IENBMRMwEQYDVQQKDApBcHBsZSBJ
bmMuMRMwEQYDVQQIDApDYWxpZm9ybmlhMB4XDTIwMDMxODE4MjEzMloXDTQ1MDMx
NTAwMDAwMFowSzEfMB0GA1UEAwwWQXBwbGUgV2ViQXV0aG4gUm9vdCBDQTETMBEG
A1UECgwKQXBwbGUgSW5jLjETMBEGA1UECAwKQ2FsaWZvcm5pYTB2MBAGByqGSM49
AgEGBSuBBAAiA2IABCJCQ2pTVhzjl4Wo6IhHtMSAzO2cv+H9DQKev3//fG59G11k
xu9eI0/7o6V5uShBpe1u6l6mS19S1FEh6yGljnZAJ+2GNP1mi/YK2kSXIuTHjxA/
pcoRf7XkOtO4o1qlcaNCMEAwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUJtdk
2cV4wlpn0afeaxLQG2PxxtcwDgYDVR0PAQH/BAQDAgEGMAoGCCqGSM49BAMDA2cA
MGQCMFrZ+9DsJ1PW9hfNdBywZDsWDbWFp28it1d/5w2RPkRX3Bbn/UbDTNLx7Jr3
jAGGiQIwHFj+dJZYUJR786osByBelJYsVZd2GbHQu209b5RCmGQ21gpSAk9QZW4B
1bWeT0vT
-----END CERTIFICATE-----
""".encode(
    "ascii"
)

```

## File: _vendor\webauthn\helpers\options_to_json.py

```python
import json
from typing import Union, Dict, Any

from .structs import (
    PublicKeyCredentialCreationOptions,
    PublicKeyCredentialRequestOptions,
)
from .bytes_to_base64url import bytes_to_base64url


def options_to_json(
    options: Union[
        PublicKeyCredentialCreationOptions,
        PublicKeyCredentialRequestOptions,
    ]
) -> str:
    """
    Prepare options for transmission to the front end as JSON
    """
    if isinstance(options, PublicKeyCredentialCreationOptions):
        _rp = {"name": options.rp.name}
        if options.rp.id:
            _rp["id"] = options.rp.id

        _user: Dict[str, Any] = {
            "id": bytes_to_base64url(options.user.id),
            "name": options.user.name,
            "displayName": options.user.display_name,
        }

        reg_to_return: Dict[str, Any] = {
            "rp": _rp,
            "user": _user,
            "challenge": bytes_to_base64url(options.challenge),
            "pubKeyCredParams": [
                {"type": param.type, "alg": param.alg} for param in options.pub_key_cred_params
            ],
        }

        # Begin handling optional values

        if options.timeout is not None:
            reg_to_return["timeout"] = options.timeout

        if options.exclude_credentials is not None:
            _excluded = options.exclude_credentials
            json_excluded = []

            for cred in _excluded:
                json_excluded_cred: Dict[str, Any] = {
                    "id": bytes_to_base64url(cred.id),
                    "type": cred.type.value,
                }

                if cred.transports:
                    json_excluded_cred["transports"] = [
                        transport.value for transport in cred.transports
                    ]

                json_excluded.append(json_excluded_cred)

            reg_to_return["excludeCredentials"] = json_excluded

        if options.authenticator_selection is not None:
            _selection = options.authenticator_selection
            json_selection: Dict[str, Any] = {}

            if _selection.authenticator_attachment is not None:
                json_selection[
                    "authenticatorAttachment"
                ] = _selection.authenticator_attachment.value

            if _selection.resident_key is not None:
                json_selection["residentKey"] = _selection.resident_key.value

            if _selection.require_resident_key is not None:
                json_selection["requireResidentKey"] = _selection.require_resident_key

            if _selection.user_verification is not None:
                json_selection["userVerification"] = _selection.user_verification.value

            reg_to_return["authenticatorSelection"] = json_selection

        if options.attestation is not None:
            reg_to_return["attestation"] = options.attestation.value

        return json.dumps(reg_to_return)

    if isinstance(options, PublicKeyCredentialRequestOptions):
        auth_to_return: Dict[str, Any] = {"challenge": bytes_to_base64url(options.challenge)}

        if options.timeout is not None:
            auth_to_return["timeout"] = options.timeout

        if options.rp_id is not None:
            auth_to_return["rpId"] = options.rp_id

        if options.allow_credentials is not None:
            _allowed = options.allow_credentials
            json_allowed = []

            for cred in _allowed:
                json_allowed_cred: Dict[str, Any] = {
                    "id": bytes_to_base64url(cred.id),
                    "type": cred.type.value,
                }

                if cred.transports:
                    json_allowed_cred["transports"] = [
                        transport.value for transport in cred.transports
                    ]

                json_allowed.append(json_allowed_cred)

            auth_to_return["allowCredentials"] = json_allowed

        if options.user_verification:
            auth_to_return["userVerification"] = options.user_verification.value

        return json.dumps(auth_to_return)

    raise TypeError(
        "Options was not instance of PublicKeyCredentialCreationOptions or PublicKeyCredentialRequestOptions"
    )

```

## File: _vendor\webauthn\helpers\parse_attestation_object.py

```python
from .parse_attestation_statement import parse_attestation_statement
from .parse_authenticator_data import parse_authenticator_data
from .structs import AttestationObject
from .parse_cbor import parse_cbor


def parse_attestation_object(val: bytes) -> AttestationObject:
    """
    Decode and peel apart the CBOR-encoded blob `response.attestationObject` into
    structured data.
    """
    attestation_dict = parse_cbor(val)

    decoded_attestation_object = AttestationObject(
        fmt=attestation_dict["fmt"],
        auth_data=parse_authenticator_data(attestation_dict["authData"]),
    )

    if "attStmt" in attestation_dict:
        decoded_attestation_object.att_stmt = parse_attestation_statement(
            attestation_dict["attStmt"]
        )

    return decoded_attestation_object

```

## File: _vendor\webauthn\helpers\parse_attestation_statement.py

```python
from .structs import AttestationStatement


def parse_attestation_statement(val: dict) -> AttestationStatement:
    """
    Turn `response.attestationObject.attStmt` into structured data
    """
    attestation_statement = AttestationStatement()

    # Populate optional fields that may exist in the attestation statement
    if "sig" in val:
        attestation_statement.sig = val["sig"]
    if "x5c" in val:
        attestation_statement.x5c = val["x5c"]
    if "response" in val:
        attestation_statement.response = val["response"]
    if "alg" in val:
        attestation_statement.alg = val["alg"]
    if "ver" in val:
        attestation_statement.ver = val["ver"]
    if "certInfo" in val:
        attestation_statement.cert_info = val["certInfo"]
    if "pubArea" in val:
        attestation_statement.pub_area = val["pubArea"]

    return attestation_statement

```

## File: _vendor\webauthn\helpers\parse_authentication_credential_json.py

```python
import json
from json.decoder import JSONDecodeError
from typing import Union

from .exceptions import InvalidAuthenticationResponse, InvalidJSONStructure
from .base64url_to_bytes import base64url_to_bytes
from .structs import (
    AuthenticationCredential,
    AuthenticatorAssertionResponse,
    AuthenticatorAttachment,
    PublicKeyCredentialType,
)


def parse_authentication_credential_json(json_val: Union[str, dict]) -> AuthenticationCredential:
    """
    Parse a JSON form of an authentication credential, as either a stringified JSON object or a
    plain dict, into an instance of AuthenticationCredential
    """
    if isinstance(json_val, str):
        try:
            json_val = json.loads(json_val)
        except JSONDecodeError:
            raise InvalidJSONStructure("Unable to decode credential as JSON")

    if not isinstance(json_val, dict):
        raise InvalidJSONStructure("Credential was not a JSON object")

    cred_id = json_val.get("id")
    if not isinstance(cred_id, str):
        raise InvalidJSONStructure("Credential missing required id")

    cred_raw_id = json_val.get("rawId")
    if not isinstance(cred_raw_id, str):
        raise InvalidJSONStructure("Credential missing required rawId")

    cred_response = json_val.get("response")
    if not isinstance(cred_response, dict):
        raise InvalidJSONStructure("Credential missing required response")

    response_client_data_json = cred_response.get("clientDataJSON")
    if not isinstance(response_client_data_json, str):
        raise InvalidJSONStructure("Credential response missing required clientDataJSON")

    response_authenticator_data = cred_response.get("authenticatorData")
    if not isinstance(response_authenticator_data, str):
        raise InvalidJSONStructure("Credential response missing required authenticatorData")

    response_signature = cred_response.get("signature")
    if not isinstance(response_signature, str):
        raise InvalidJSONStructure("Credential response missing required signature")

    cred_type = json_val.get("type")
    try:
        # Simply try to get the single matching Enum. We'll set the literal value below assuming
        # the code can get past here (this is basically a mypy optimization)
        PublicKeyCredentialType(cred_type)
    except ValueError as cred_type_exc:
        raise InvalidJSONStructure("Credential had unexpected type") from cred_type_exc

    response_user_handle = cred_response.get("userHandle")
    if isinstance(response_user_handle, str):
        # The `userHandle` string will most likely be base64url-encoded for ease of JSON
        # transmission as per the L3 Draft spec:
        # https://w3c.github.io/webauthn/#dictdef-authenticatorassertionresponsejson
        response_user_handle = base64url_to_bytes(response_user_handle)
    elif response_user_handle is not None:
        # If it's not a string, and it's not None, then it's definitely not valid
        raise InvalidJSONStructure("Credential response had unexpected userHandle")

    cred_authenticator_attachment = json_val.get("authenticatorAttachment")
    if isinstance(cred_authenticator_attachment, str):
        try:
            cred_authenticator_attachment = AuthenticatorAttachment(cred_authenticator_attachment)
        except ValueError as cred_attachment_exc:
            raise InvalidJSONStructure(
                "Credential had unexpected authenticatorAttachment"
            ) from cred_attachment_exc
    else:
        cred_authenticator_attachment = None

    try:
        authentication_credential = AuthenticationCredential(
            id=cred_id,
            raw_id=base64url_to_bytes(cred_raw_id),
            response=AuthenticatorAssertionResponse(
                client_data_json=base64url_to_bytes(response_client_data_json),
                authenticator_data=base64url_to_bytes(response_authenticator_data),
                signature=base64url_to_bytes(response_signature),
                user_handle=response_user_handle,
            ),
            authenticator_attachment=cred_authenticator_attachment,
            type=PublicKeyCredentialType.PUBLIC_KEY,
        )
    except Exception as exc:
        raise InvalidAuthenticationResponse(
            "Could not parse authentication credential from JSON data"
        ) from exc

    return authentication_credential

```

## File: _vendor\webauthn\helpers\parse_authenticator_data.py

```python
from typing import Union

from .byteslike_to_bytes import byteslike_to_bytes
from .exceptions import InvalidAuthenticatorDataStructure
from .structs import AttestedCredentialData, AuthenticatorData, AuthenticatorDataFlags
from .parse_cbor import parse_cbor
from .encode_cbor import encode_cbor


def parse_authenticator_data(val: bytes) -> AuthenticatorData:
    """
    Turn `response.attestationObject.authData` into structured data
    """
    val = byteslike_to_bytes(val)

    # Don't bother parsing if there aren't enough bytes for at least:
    # - rpIdHash (32 bytes)
    # - flags (1 byte)
    # - signCount (4 bytes)
    if len(val) < 37:
        raise InvalidAuthenticatorDataStructure(
            f"Authenticator data was {len(val)} bytes, expected at least 37 bytes"
        )

    pointer = 0

    rp_id_hash = val[pointer:32]
    pointer += 32

    # Cast byte to ordinal so we can use bitwise operators on it
    flags_bytes = ord(val[pointer : pointer + 1])
    pointer += 1

    sign_count = val[pointer : pointer + 4]
    pointer += 4

    # Parse flags
    flags = AuthenticatorDataFlags(
        up=flags_bytes & (1 << 0) != 0,
        uv=flags_bytes & (1 << 2) != 0,
        be=flags_bytes & (1 << 3) != 0,
        bs=flags_bytes & (1 << 4) != 0,
        at=flags_bytes & (1 << 6) != 0,
        ed=flags_bytes & (1 << 7) != 0,
    )

    # The value to return
    authenticator_data = AuthenticatorData(
        rp_id_hash=rp_id_hash,
        flags=flags,
        sign_count=int.from_bytes(sign_count, "big"),
    )

    # Parse AttestedCredentialData if present
    if flags.at is True:
        aaguid = val[pointer : pointer + 16]
        pointer += 16

        credential_id_len = int.from_bytes(val[pointer : pointer + 2], "big")
        pointer += 2

        credential_id = val[pointer : pointer + credential_id_len]
        pointer += credential_id_len

        """
        Some authenticators incorrectly compose authData when using EdDSA for their public keys.
        A CBOR "Map of 3 items" (0xA3) should be "Map of 4 items" (0xA4), and if we manually adjust
        the single byte there's a good chance the authData can be correctly parsed. Let's try to
        detect when this happens and gracefully handle it.
        """
        # Decodes to `{1: "OKP", 3: -8, -1: "Ed25519"}` (it's missing key -2 a.k.a. COSEKey.X)
        bad_eddsa_cbor = bytearray.fromhex("a301634f4b500327206745643235353139")
        # If we find the bytes here then let's fix the bad data
        if val[pointer : pointer + len(bad_eddsa_cbor)] == bad_eddsa_cbor:
            # Make a mutable copy of the bytes...
            _val = bytearray(val)
            # ...Fix the bad byte...
            _val[pointer] = 0xA4
            # ...Then replace `val` with the fixed bytes
            val = bytes(_val)

        # Load the next CBOR-encoded value
        credential_public_key = parse_cbor(val[pointer:])
        credential_public_key_bytes = encode_cbor(credential_public_key)
        pointer += len(credential_public_key_bytes)

        attested_cred_data = AttestedCredentialData(
            aaguid=aaguid,
            credential_id=credential_id,
            credential_public_key=credential_public_key_bytes,
        )
        authenticator_data.attested_credential_data = attested_cred_data

    if flags.ed is True:
        extension_object = parse_cbor(val[pointer:])
        extension_bytes = encode_cbor(extension_object)
        pointer += len(extension_bytes)
        authenticator_data.extensions = extension_bytes

    # We should have parsed all authenticator data by this point
    if len(val) > pointer:
        raise InvalidAuthenticatorDataStructure(
            "Leftover bytes detected while parsing authenticator data"
        )

    return authenticator_data

```

## File: _vendor\webauthn\helpers\parse_backup_flags.py

```python
from enum import Enum
from dataclasses import dataclass

from .structs import AuthenticatorDataFlags, CredentialDeviceType
from .exceptions import InvalidBackupFlags


@dataclass
class ParsedBackupFlags:
    credential_device_type: CredentialDeviceType
    credential_backed_up: bool


def parse_backup_flags(flags: AuthenticatorDataFlags) -> ParsedBackupFlags:
    """Convert backup eligibility and backup state flags into more useful representations

    Raises:
        `helpers.exceptions.InvalidBackupFlags` if an invalid backup state is detected
    """
    credential_device_type = CredentialDeviceType.SINGLE_DEVICE

    # A credential that can be backed up can typically be used on multiple devices
    if flags.be:
        credential_device_type = CredentialDeviceType.MULTI_DEVICE

    if credential_device_type == CredentialDeviceType.SINGLE_DEVICE and flags.bs:
        raise InvalidBackupFlags(
            "Single-device credential indicated that it was backed up, which should be impossible."
        )

    return ParsedBackupFlags(
        credential_device_type=credential_device_type,
        credential_backed_up=flags.bs,
    )

```

## File: _vendor\webauthn\helpers\parse_cbor.py

```python
from typing import Any

import cbor2

from .exceptions import InvalidCBORData


def parse_cbor(data: bytes) -> Any:
    """
    Attempt to decode CBOR-encoded data.

    Raises:
        `helpers.exceptions.InvalidCBORData` if data cannot be decoded
    """
    try:
        to_return = cbor2.loads(data)
    except Exception as exc:
        raise InvalidCBORData("Could not decode CBOR data") from exc

    return to_return

```

## File: _vendor\webauthn\helpers\parse_client_data_json.py

```python
import json
from json.decoder import JSONDecodeError
from typing import Union

from .base64url_to_bytes import base64url_to_bytes
from .byteslike_to_bytes import byteslike_to_bytes
from .exceptions import InvalidJSONStructure
from .structs import CollectedClientData, TokenBinding


def parse_client_data_json(val: bytes) -> CollectedClientData:
    """
    Break apart `response.clientDataJSON` buffer into structured data
    """
    val = byteslike_to_bytes(val)

    try:
        json_dict = json.loads(val)
    except JSONDecodeError:
        raise InvalidJSONStructure("Unable to decode client_data_json bytes as JSON")

    # Ensure required values are present in client data
    if "type" not in json_dict:
        raise InvalidJSONStructure('client_data_json missing required property "type"')
    if "challenge" not in json_dict:
        raise InvalidJSONStructure('client_data_json missing required property "challenge"')
    if "origin" not in json_dict:
        raise InvalidJSONStructure('client_data_json missing required property "origin"')

    client_data = CollectedClientData(
        type=json_dict["type"],
        challenge=base64url_to_bytes(json_dict["challenge"]),
        origin=json_dict["origin"],
    )

    # Populate optional values if set
    if "crossOrigin" in json_dict:
        cross_origin = bool(json_dict["crossOrigin"])
        client_data.cross_origin = cross_origin

    if "tokenBinding" in json_dict:
        token_binding_dict = json_dict["tokenBinding"]

        # Some U2F devices set a string to `token_binding`, in which case ignore it
        if type(token_binding_dict) is dict:
            if "status" not in token_binding_dict:
                raise InvalidJSONStructure('token_binding missing required property "status"')

            status = token_binding_dict["status"]
            try:
                # This will raise ValidationError on an unexpected status
                token_binding = TokenBinding(status=status)

                # Handle optional values
                if "id" in token_binding_dict:
                    id = token_binding_dict["id"]
                    token_binding.id = f"{id}"

                client_data.token_binding = token_binding
            except Exception:
                # If we encounter a status we don't expect then ignore token_binding
                # completely
                pass

    return client_data

```

## File: _vendor\webauthn\helpers\parse_registration_credential_json.py

```python
import json
from json.decoder import JSONDecodeError
from typing import Union, Optional, List

from .base64url_to_bytes import base64url_to_bytes
from .exceptions import InvalidRegistrationResponse, InvalidJSONStructure
from .structs import (
    AuthenticatorAttachment,
    AuthenticatorAttestationResponse,
    AuthenticatorTransport,
    PublicKeyCredentialType,
    RegistrationCredential,
)


def parse_registration_credential_json(json_val: Union[str, dict]) -> RegistrationCredential:
    """
    Parse a JSON form of a registration credential, as either a stringified JSON object or a
    plain dict, into an instance of RegistrationCredential
    """
    if isinstance(json_val, str):
        try:
            json_val = json.loads(json_val)
        except JSONDecodeError:
            raise InvalidJSONStructure("Unable to decode credential as JSON")

    if not isinstance(json_val, dict):
        raise InvalidJSONStructure("Credential was not a JSON object")

    cred_id = json_val.get("id")
    if not isinstance(cred_id, str):
        raise InvalidJSONStructure("Credential missing required id")

    cred_raw_id = json_val.get("rawId")
    if not isinstance(cred_raw_id, str):
        raise InvalidJSONStructure("Credential missing required rawId")

    cred_response = json_val.get("response")
    if not isinstance(cred_response, dict):
        raise InvalidJSONStructure("Credential missing required response")

    response_client_data_json = cred_response.get("clientDataJSON")
    if not isinstance(response_client_data_json, str):
        raise InvalidJSONStructure("Credential response missing required clientDataJSON")

    response_attestation_object = cred_response.get("attestationObject")
    if not isinstance(response_attestation_object, str):
        raise InvalidJSONStructure("Credential response missing required attestationObject")

    cred_type = json_val.get("type")
    try:
        # Simply try to get the single matching Enum. We'll set the literal value below assuming
        # the code can get past here (this is basically a mypy optimization)
        PublicKeyCredentialType(cred_type)
    except ValueError as cred_type_exc:
        raise InvalidJSONStructure("Credential had unexpected type") from cred_type_exc

    transports: Optional[List[AuthenticatorTransport]] = None
    response_transports = cred_response.get("transports")
    if isinstance(response_transports, list):
        transports = []
        for val in response_transports:
            try:
                transport_enum = AuthenticatorTransport(val)
                transports.append(transport_enum)
            except ValueError:
                pass

    cred_authenticator_attachment = json_val.get("authenticatorAttachment")
    if isinstance(cred_authenticator_attachment, str):
        try:
            cred_authenticator_attachment = AuthenticatorAttachment(cred_authenticator_attachment)
        except ValueError as cred_attachment_exc:
            raise InvalidJSONStructure(
                "Credential had unexpected authenticatorAttachment"
            ) from cred_attachment_exc
    else:
        cred_authenticator_attachment = None

    try:
        registration_credential = RegistrationCredential(
            id=cred_id,
            raw_id=base64url_to_bytes(cred_raw_id),
            response=AuthenticatorAttestationResponse(
                client_data_json=base64url_to_bytes(response_client_data_json),
                attestation_object=base64url_to_bytes(response_attestation_object),
                transports=transports,
            ),
            authenticator_attachment=cred_authenticator_attachment,
            type=PublicKeyCredentialType.PUBLIC_KEY,
        )
    except Exception as exc:
        raise InvalidRegistrationResponse(
            "Could not parse registration credential from JSON data"
        ) from exc

    return registration_credential

```

## File: _vendor\webauthn\helpers\pem_cert_bytes_to_open_ssl_x509.py

```python
from cryptography.hazmat.backends import default_backend
from cryptography.x509 import load_pem_x509_certificate
from OpenSSL.crypto import X509


def pem_cert_bytes_to_open_ssl_x509(cert: bytes) -> X509:
    """Convert PEM-formatted certificate bytes into an X509 instance usable for cert
    chain validation
    """
    cert_crypto = load_pem_x509_certificate(cert, default_backend())
    cert_openssl = X509().from_cryptography(cert_crypto)
    return cert_openssl

```

## File: _vendor\webauthn\helpers\snake_case_to_camel_case.py

```python
def snake_case_to_camel_case(snake_case: str) -> str:
    """
    Helper method for converting a snake_case'd value to camelCase

    input: pub_key_cred_params
    output: pubKeyCredParams
    """
    parts = snake_case.split("_")
    converted = parts[0].lower() + "".join(part.title() for part in parts[1:])

    # Massage "clientDataJson" to "clientDataJSON"
    converted = converted.replace("Json", "JSON")

    return converted

```

## File: _vendor\webauthn\helpers\structs.py

```python
from enum import Enum
from dataclasses import dataclass, field
from typing import List, Literal, Optional, Union

from .cose import COSEAlgorithmIdentifier


################
#
# Fundamental data structures
#
################


class AuthenticatorTransport(str, Enum):
    """How an authenticator communicates to the client/browser.

    Members:
        `USB`: USB wired connection
        `NFC`: Near Field Communication
        `BLE`: Bluetooth Low Energy
        `INTERNAL`: Direct connection (read: a platform authenticator)
        `CABLE`: Cloud Assisted Bluetooth Low Energy
        `HYBRID`: A combination of (often separate) data-transport and proximity mechanisms

    https://www.w3.org/TR/webauthn-2/#enum-transport
    """

    USB = "usb"
    NFC = "nfc"
    BLE = "ble"
    INTERNAL = "internal"
    CABLE = "cable"
    HYBRID = "hybrid"


class AuthenticatorAttachment(str, Enum):
    """How an authenticator is connected to the client/browser.

    Members:
        `PLATFORM`: A non-removable authenticator, like TouchID or Windows Hello
        `CROSS_PLATFORM`: A "roaming" authenticator, like a YubiKey

    https://www.w3.org/TR/webauthn-2/#enumdef-authenticatorattachment
    """

    PLATFORM = "platform"
    CROSS_PLATFORM = "cross-platform"


class ResidentKeyRequirement(str, Enum):
    """The Relying Party's preference for the authenticator to create a dedicated "client-side" credential for it. Requiring an authenticator to store a dedicated credential should not be done lightly due to the limited storage capacity of some types of authenticators.

    Members:
        `DISCOURAGED`: The authenticator should not create a dedicated credential
        `PREFERRED`: The authenticator can create and store a dedicated credential, but if it doesn't that's alright too
        `REQUIRED`: The authenticator MUST create a dedicated credential. If it cannot, the RP is prepared for an error to occur.

    https://www.w3.org/TR/webauthn-2/#enum-residentKeyRequirement
    """

    DISCOURAGED = "discouraged"
    PREFERRED = "preferred"
    REQUIRED = "required"


class UserVerificationRequirement(str, Enum):
    """The degree to which the Relying Party wishes to verify a user's identity.

    Members:
        `REQUIRED`: User verification must occur
        `PREFERRED`: User verification would be great, but if not that's okay too
        `DISCOURAGED`: User verification should not occur, but it's okay if it does

    https://www.w3.org/TR/webauthn-2/#enumdef-userverificationrequirement
    """

    REQUIRED = "required"
    PREFERRED = "preferred"
    DISCOURAGED = "discouraged"


class AttestationConveyancePreference(str, Enum):
    """The Relying Party's interest in receiving an attestation statement.

    Members:
        `NONE`: The Relying Party isn't interested in receiving an attestation statement
        `INDIRECT`: The Relying Party is interested in an attestation statement, but the client is free to generate it as it sees fit
        `DIRECT`: The Relying Party is interested in an attestation statement generated directly by the authenticator
        `ENTERPRISE`: The Relying Party is interested in a statement with identifying information. Typically used within organizations

    https://www.w3.org/TR/webauthn-2/#enum-attestation-convey
    """

    NONE = "none"
    INDIRECT = "indirect"
    DIRECT = "direct"
    ENTERPRISE = "enterprise"


class PublicKeyCredentialType(str, Enum):
    """The type of credential that should be returned by an authenticator. There's but a single member because this is a specific subclass of a higher-level `CredentialType` that can be of other types.

    Members:
        `PUBLIC_KEY`: The literal string `"public-key"`

    https://www.w3.org/TR/webauthn-2/#enumdef-publickeycredentialtype
    """

    PUBLIC_KEY = "public-key"


class AttestationFormat(str, Enum):
    """The "syntax" of an attestation statement. Formats should be registered with the IANA and include documented signature verification steps.

    Members:
        `PACKED`
        `TPM`
        `ANDROID_KEY`
        `ANDROID_SAFETYNET`
        `FIDO_U2F`
        `APPLE`
        `NONE`

    https://www.iana.org/assignments/webauthn/webauthn.xhtml
    """

    PACKED = "packed"
    TPM = "tpm"
    ANDROID_KEY = "android-key"
    ANDROID_SAFETYNET = "android-safetynet"
    FIDO_U2F = "fido-u2f"
    APPLE = "apple"
    NONE = "none"


class ClientDataType(str, Enum):
    """Specific values included in authenticator registration and authentication responses to help avoid certain types of "signature confusion attacks".

    Members:
        `WEBAUTHN_CREATE`: The string "webauthn.create". Synonymous with `navigator.credentials.create()` in the browser
        `WEBAUTHN_GET`: The string "webauthn.get". Synonymous with `navigator.credentials.get()` in the browser

    https://www.w3.org/TR/webauthn-2/#dom-collectedclientdata-type
    """

    WEBAUTHN_CREATE = "webauthn.create"
    WEBAUTHN_GET = "webauthn.get"


class TokenBindingStatus(str, Enum):
    """
    https://www.w3.org/TR/webauthn-2/#dom-tokenbinding-status
    """

    PRESENT = "present"
    SUPPORTED = "supported"


@dataclass
class TokenBinding:
    """
    https://www.w3.org/TR/webauthn-2/#dictdef-tokenbinding
    """

    status: TokenBindingStatus
    id: Optional[str] = None


@dataclass
class PublicKeyCredentialRpEntity:
    """Information about the Relying Party.

    Attributes:
        `name`: A user-readable name for the Relying Party
        (optional) `id`: A unique, constant value assigned to the Relying Party. Authenticators use this value to associate a credential with a particular Relying Party user

    https://www.w3.org/TR/webauthn-2/#dictdef-publickeycredentialrpentity
    """

    name: str
    id: Optional[str] = None


@dataclass
class PublicKeyCredentialUserEntity:
    """Information about a user of a Relying Party.

    Attributes:
        `id`: An "opaque byte sequence" that uniquely identifies a user. Typically something like a UUID, but never user-identifying like an email address. Cannot exceed 64 bytes. These bytes should be stored internally alongside your normal user identifier and only used for WebAuthn.
        `name`: A value which a user can see to determine which account this credential is associated with. A username or email address is fine here.
        `display_name`: A user-friendly representation of a user, like a full name.

    https://www.w3.org/TR/webauthn-2/#dictdef-publickeycredentialuserentity
    """

    id: bytes
    name: str
    display_name: str


@dataclass
class PublicKeyCredentialParameters:
    """Information about a cryptographic algorithm that may be used when creating a credential.

    Attributes:
        `type`: The literal string `"public-key"`
        `alg`: A numeric indicator of a particular algorithm

    https://www.w3.org/TR/webauthn-2/#dictdef-publickeycredentialparameters
    """

    type: Literal["public-key"]
    alg: COSEAlgorithmIdentifier


@dataclass
class PublicKeyCredentialDescriptor:
    """Information about a generated credential.

    Attributes:
        `type`: The literal string `"public-key"`
        `id`: The sequence of bytes representing the credential's ID
        (optional) `transports`: The types of connections to the client/browser the authenticator supports

    https://www.w3.org/TR/webauthn-2/#dictdef-publickeycredentialdescriptor
    """

    id: bytes
    type: Literal[PublicKeyCredentialType.PUBLIC_KEY] = PublicKeyCredentialType.PUBLIC_KEY
    transports: Optional[List[AuthenticatorTransport]] = None


@dataclass
class AuthenticatorSelectionCriteria:
    """A Relying Party's requirements for the types of authenticators that may interact with the client/browser.

    Attributes:
        (optional) `authenticator_attachment`: How the authenticator can be connected to the client/browser
        (optional) `resident_key`: Whether the authenticator should be able to store a credential on itself
        (optional) `require_resident_key`: DEPRECATED, set a value for `resident_key` instead
        (optional) `user_verification`: How the authenticator should be capable of determining user identity

    https://www.w3.org/TR/webauthn-2/#dictdef-authenticatorselectioncriteria
    """

    authenticator_attachment: Optional[AuthenticatorAttachment] = None
    resident_key: Optional[ResidentKeyRequirement] = None
    require_resident_key: Optional[bool] = False
    user_verification: Optional[
        UserVerificationRequirement
    ] = UserVerificationRequirement.PREFERRED


@dataclass
class CollectedClientData:
    """Decoded ClientDataJSON

    Attributes:
        `type`: Either `"webauthn.create"` or `"webauthn.get"`, for registration and authentication ceremonies respectively
        `challenge`: The challenge passed to the authenticator within the options
        `origin`: The base domain with protocol on which the registration or authentication ceremony took place (e.g. "https://foo.bar")
        (optional) `cross_origin`: Whether or not the the registration or authentication ceremony took place on a different origin (think within an <iframe>)
        (optional) `token_binding`: Information on the state of the Token Binding protocol

    https://www.w3.org/TR/webauthn-2/#dictdef-collectedclientdata
    """

    type: ClientDataType
    challenge: bytes
    origin: str
    cross_origin: Optional[bool] = None
    token_binding: Optional[TokenBinding] = None


################
#
# Registration
#
################


@dataclass
class PublicKeyCredentialCreationOptions:
    """Registration Options.

    Attributes:
        `rp`: Information about the Relying Party
        `user`: Information about the user
        `challenge`: A unique byte sequence to be returned by the authenticator. Helps prevent replay attacks
        `pub_key_cred_params`: Cryptographic algorithms supported by the Relying Party when verifying signatures
        (optional) `timeout`: How long the client/browser should give the user to interact with an authenticator
        (optional) `exclude_credentials`: A list of credentials associated with the user to prevent them from re-enrolling one of them
        (optional) `authenticator_selection`: Additional qualities about the authenticators the user can use to complete registration
        (optional) `attestation`: The Relying Party's desire for a declaration of an authenticator's provenance via attestation statement

    https://www.w3.org/TR/webauthn-2/#dictdef-publickeycredentialcreationoptions
    """

    rp: PublicKeyCredentialRpEntity
    user: PublicKeyCredentialUserEntity
    challenge: bytes
    pub_key_cred_params: List[PublicKeyCredentialParameters]
    timeout: Optional[int] = None
    exclude_credentials: Optional[List[PublicKeyCredentialDescriptor]] = None
    authenticator_selection: Optional[AuthenticatorSelectionCriteria] = None
    attestation: AttestationConveyancePreference = AttestationConveyancePreference.NONE


@dataclass
class AuthenticatorAttestationResponse:
    """The `response` property on a registration credential.

    Attributes:
        `client_data_json`: Information the authenticator collects about the client/browser it communicates with
        `attestation_object`: Encoded information about an attestation
        (optional) `transports`: The authenticator's supported methods of communication with a client/browser

    https://www.w3.org/TR/webauthn-2/#authenticatorattestationresponse
    """

    client_data_json: bytes
    attestation_object: bytes
    # Optional in L2, but becomes required in L3. Play it safe until L3 becomes Recommendation
    transports: Optional[List[AuthenticatorTransport]] = None


@dataclass
class RegistrationCredential:
    """A registration-specific subclass of PublicKeyCredential returned from `navigator.credentials.create()`

    Attributes:
        `id`: The Base64URL-encoded representation of raw_id
        `raw_id`: A byte sequence representing the credential's unique identifier
        `response`: The authenticator's attesation data
        `type`: The literal string `"public-key"`

    https://www.w3.org/TR/webauthn-2/#publickeycredential
    """

    id: str
    raw_id: bytes
    response: AuthenticatorAttestationResponse
    authenticator_attachment: Optional[AuthenticatorAttachment] = None
    type: Literal[PublicKeyCredentialType.PUBLIC_KEY] = PublicKeyCredentialType.PUBLIC_KEY


@dataclass
class AttestationStatement:
    """A collection of all possible fields that may exist in an attestation statement. Combinations of these fields are specific to a particular attestation format.

    https://www.w3.org/TR/webauthn-2/#sctn-defined-attestation-formats

    TODO: Decide if this is acceptable, or if we want to split this up into multiple
    format-specific classes that define only the fields that are present for a given
    attestation format.
    """

    sig: Optional[bytes] = None
    x5c: Optional[List[bytes]] = None
    response: Optional[bytes] = None
    alg: Optional[COSEAlgorithmIdentifier] = None
    ver: Optional[str] = None
    cert_info: Optional[bytes] = None
    pub_area: Optional[bytes] = None


@dataclass
class AuthenticatorDataFlags:
    """Flags the authenticator will set about information contained within the `attestationObject.authData` property.

    Attributes:
        `up`: [U]ser was [P]resent
        `uv`: [U]ser was [V]erified
        `be`: [B]ackup [E]ligible
        `bs`: [B]ackup [S]tate
        `at`: [AT]tested credential is included
        `ed`: [E]xtension [D]ata is included

    https://www.w3.org/TR/webauthn-2/#flags
    """

    up: bool
    uv: bool
    be: bool
    bs: bool
    at: bool
    ed: bool


@dataclass
class AttestedCredentialData:
    """Information about a credential.

    Attributes:
        `aaguid`: A 128-bit identifier indicating the type and vendor of the authenticator
        `credential_id`: The ID of the private/public key pair generated by the authenticator
        `credential_public_key`: The public key generated by the authenticator

    https://www.w3.org/TR/webauthn-2/#attested-credential-data
    """

    aaguid: bytes
    credential_id: bytes
    credential_public_key: bytes


@dataclass
class AuthenticatorData:
    """Context the authenticator provides about itself and the environment in which the registration or authentication ceremony took place.

    Attributes:
        `rp_id_hash`: A SHA-256 hash of the website origin on which the registration or authentication ceremony took place
        `flags`: Properties about the user and registration, where applicable
        `sign_count`: The number of times the credential was used
        (optional) `attested_credential_data`: Information about the credential created during a registration ceremony
        (optional) `extensions`: CBOR-encoded extension data corresponding to extensions specified in the registration or authentication ceremony options

    https://www.w3.org/TR/webauthn-2/#sctn-attestation
    https://www.w3.org/TR/webauthn-2/#sctn-attested-credential-data
    """

    rp_id_hash: bytes
    flags: AuthenticatorDataFlags
    sign_count: int
    attested_credential_data: Optional[AttestedCredentialData] = None
    extensions: Optional[bytes] = None


@dataclass
class AttestationObject:
    """Information about an attestation, including a statement and authenticator data.

    Attributes:
        `fmt`: The attestation statement's format
        `att_stmt`: An attestation statement to be verified according to the format
        `auth_data`: Contextual information provided by authenticator

    https://www.w3.org/TR/webauthn-2/#sctn-attestation
    """

    fmt: AttestationFormat
    auth_data: AuthenticatorData
    att_stmt: AttestationStatement = field(default_factory=AttestationStatement)


################
#
# Authentication
#
################


@dataclass
class PublicKeyCredentialRequestOptions:
    """Authentication Options.

    Attributes:
        `challenge`: A unique byte sequence to be returned by the authenticator. Helps prevent replay attacks
        (optional) `timeout`: How long the client/browser should give the user to interact with an authenticator
        (optional) `rp_id`: The unique, constant identifier assigned to the Relying Party
        (optional) `allow_credentials`: A list of credentials associated with the user that they can use to complete the authentication
        (optional) `user_verification`: How the authenticator should be capable of determining user identity

    https://www.w3.org/TR/webauthn-2/#dictionary-assertion-options
    """

    challenge: bytes
    timeout: Optional[int] = None
    rp_id: Optional[str] = None
    allow_credentials: Optional[List[PublicKeyCredentialDescriptor]] = None
    user_verification: Optional[
        UserVerificationRequirement
    ] = UserVerificationRequirement.PREFERRED


@dataclass
class AuthenticatorAssertionResponse:
    """The `response` property on an authentication credential.

    Attributes:
        `client_data_json`: Information the authenticator collects about the client/browser it communicates with
        `authenticator_data`: Contextual information provided by authenticator
        `signature`: A byte sequence signed by the authenticator's private key, to be verified with a user's public key
        (optional) `user_handle`: The user ID specified for the user during attestation

    https://www.w3.org/TR/webauthn-2/#authenticatorassertionresponse
    """

    client_data_json: bytes
    authenticator_data: bytes
    signature: bytes
    user_handle: Optional[bytes] = None


@dataclass
class AuthenticationCredential:
    """An authentication-specific subclass of PublicKeyCredential. Returned from `navigator.credentials.get()`

    Attributes:
        `id`: The Base64URL-encoded representation of raw_id
        `raw_id`: A byte sequence representing the credential's unique identifier
        `response`: The authenticator's assertion data
        `type`: The literal string `"public-key"`

    https://www.w3.org/TR/webauthn-2/#publickeycredential
    """

    id: str
    raw_id: bytes
    response: AuthenticatorAssertionResponse
    authenticator_attachment: Optional[AuthenticatorAttachment] = None
    type: Literal[PublicKeyCredentialType.PUBLIC_KEY] = PublicKeyCredentialType.PUBLIC_KEY


################
#
# Credential Backup State
#
################


class CredentialDeviceType(str, Enum):
    """A determination of the number of devices a credential can be used from

    Members:
        `SINGLE_DEVICE`: A credential that is bound to a single device
        `MULTI_DEVICE`: A credential that can be used from multiple devices (e.g. passkeys)

    https://w3c.github.io/webauthn/#sctn-credential-backup (L3 Draft)
    """

    SINGLE_DEVICE = "single_device"
    MULTI_DEVICE = "multi_device"

```

## File: _vendor\webauthn\helpers\validate_certificate_chain.py

```python
from typing import List, Optional

from cryptography.hazmat.backends import default_backend
from cryptography.x509 import load_der_x509_certificate
from OpenSSL.crypto import X509, X509Store, X509StoreContext, X509StoreContextError

from .exceptions import InvalidCertificateChain
from .pem_cert_bytes_to_open_ssl_x509 import pem_cert_bytes_to_open_ssl_x509


def validate_certificate_chain(
    *,
    x5c: List[bytes],
    pem_root_certs_bytes: Optional[List[bytes]] = None,
) -> bool:
    """Validate that the certificates in x5c chain back to a known root certificate

    Args:
        `x5c`: X5C certificates from a registration response's attestation statement
        (optional) `pem_root_certs_bytes`: Any additional (PEM-formatted)
        root certificates that may complete the certificate chain

    Raises:
        `helpers.exceptions.InvalidCertificateChain` if chain cannot be validated
    """
    if pem_root_certs_bytes is None or len(pem_root_certs_bytes) < 1:
        # We have no root certs to chain back to, so just pass on validation
        return True

    # Make sure we have at least one certificate to try and link back to a root cert
    if len(x5c) < 1:
        raise InvalidCertificateChain("x5c was empty")

    # Prepare leaf cert
    try:
        leaf_cert_bytes = x5c[0]
        leaf_cert_crypto = load_der_x509_certificate(leaf_cert_bytes, default_backend())
        leaf_cert = X509().from_cryptography(leaf_cert_crypto)
    except Exception as err:
        raise InvalidCertificateChain(f"Could not prepare leaf cert: {err}")

    # Prepare any intermediate certs
    try:
        # May be an empty array, that's fine
        intermediate_certs_bytes = x5c[1:]
        intermediate_certs_crypto = [
            load_der_x509_certificate(cert, default_backend()) for cert in intermediate_certs_bytes
        ]
        intermediate_certs = [X509().from_cryptography(cert) for cert in intermediate_certs_crypto]
    except Exception as err:
        raise InvalidCertificateChain(f"Could not prepare intermediate certs: {err}")

    # Prepare a collection of possible root certificates
    root_certs_store = X509Store()
    try:
        for cert in pem_root_certs_bytes:
            root_certs_store.add_cert(pem_cert_bytes_to_open_ssl_x509(cert))
    except Exception as err:
        raise InvalidCertificateChain(f"Could not prepare root certs: {err}")

    # Load certs into a "context" for validation
    context = X509StoreContext(
        store=root_certs_store,
        certificate=leaf_cert,
        chain=intermediate_certs,
    )

    # Validate the chain (will raise if it can't)
    try:
        context.verify_certificate()
    except X509StoreContextError:
        raise InvalidCertificateChain("Certificate chain could not be validated")

    return True

```

## File: _vendor\webauthn\helpers\verify_safetynet_timestamp.py

```python
import time


def verify_safetynet_timestamp(timestamp_ms: int) -> None:
    """Handle time drift between an RP and the Google SafetyNet API servers with a window of
    time within which the response is valid
    """
    # Buffer period in ms
    grace_ms = 10 * 1000
    # Get "now" in ms
    now = int(time.time()) * 1000

    # Make sure the response was generated in the past
    if timestamp_ms > (now + grace_ms):
        raise ValueError(f"Payload timestamp {timestamp_ms} was later than {now} + {grace_ms}")

    # Make sure the response arrived within the grace period
    if timestamp_ms < (now - grace_ms):
        raise ValueError("Payload has expired")

```

## File: _vendor\webauthn\helpers\verify_signature.py

```python
from typing import Union

from cryptography.hazmat.primitives.asymmetric.dsa import DSAPublicKey
from cryptography.hazmat.primitives.asymmetric.ec import EllipticCurvePublicKey
from cryptography.hazmat.primitives.asymmetric.ed448 import Ed448PublicKey
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey
from cryptography.hazmat.primitives.asymmetric.x25519 import X25519PublicKey
from cryptography.hazmat.primitives.asymmetric.x448 import X448PublicKey
from cryptography.hazmat.primitives.asymmetric.padding import MGF1, PSS, PKCS1v15
from cryptography.hazmat.primitives.asymmetric.rsa import RSAPublicKey

from .algorithms import (
    get_ec2_sig_alg,
    get_rsa_pkcs1_sig_alg,
    get_rsa_pss_sig_alg,
    is_rsa_pkcs,
    is_rsa_pss,
)
from .cose import COSEAlgorithmIdentifier
from .exceptions import UnsupportedAlgorithm, UnsupportedPublicKey


def verify_signature(
    *,
    public_key: Union[
        EllipticCurvePublicKey,
        RSAPublicKey,
        Ed25519PublicKey,
        DSAPublicKey,
        Ed448PublicKey,
        X25519PublicKey,
        X448PublicKey,
    ],
    signature_alg: COSEAlgorithmIdentifier,
    signature: bytes,
    data: bytes,
) -> None:
    """Verify a signature was signed with the private key corresponding to the provided
    public key.

    Args:
        `public_key`: A public key loaded via cryptography's `load_der_public_key`, `load_der_x509_certificate`, etc...
        `signature_alg`: Algorithm ID used to sign the signature
        `signature`: Signature to verify
        `data`: Data signed by private key

    Raises:
        `webauth.helpers.exceptions.UnsupportedAlgorithm` when the algorithm is not a recognized COSE algorithm ID
        `webauth.helpers.exceptions.UnsupportedPublicKey` when the public key is not a valid EC2, RSA, or OKP certificate
        `cryptography.exceptions.InvalidSignature` when the signature cannot be verified
    """
    if isinstance(public_key, EllipticCurvePublicKey):
        public_key.verify(signature, data, get_ec2_sig_alg(signature_alg))
    elif isinstance(public_key, RSAPublicKey):
        if is_rsa_pkcs(signature_alg):
            public_key.verify(signature, data, PKCS1v15(), get_rsa_pkcs1_sig_alg(signature_alg))
        elif is_rsa_pss(signature_alg):
            rsa_alg = get_rsa_pss_sig_alg(signature_alg)
            public_key.verify(
                signature,
                data,
                PSS(mgf=MGF1(rsa_alg), salt_length=PSS.MAX_LENGTH),
                rsa_alg,
            )
        else:
            raise UnsupportedAlgorithm(f"Unrecognized RSA signature alg {signature_alg}")
    elif isinstance(public_key, Ed25519PublicKey):
        public_key.verify(signature, data)
    else:
        raise UnsupportedPublicKey(
            f"Unsupported public key for signature verification: {public_key}"
        )

```

## File: _vendor\webauthn\helpers\__init__.py

```python
from .aaguid_to_string import aaguid_to_string
from .base64url_to_bytes import base64url_to_bytes
from .bytes_to_base64url import bytes_to_base64url
from .byteslike_to_bytes import byteslike_to_bytes
from .decode_credential_public_key import decode_credential_public_key
from .decoded_public_key_to_cryptography import decoded_public_key_to_cryptography
from .encode_cbor import encode_cbor
from .generate_challenge import generate_challenge
from .generate_user_handle import generate_user_handle
from .hash_by_alg import hash_by_alg
from .options_to_json import options_to_json
from .parse_attestation_object import parse_attestation_object
from .parse_authentication_credential_json import parse_authentication_credential_json
from .parse_authenticator_data import parse_authenticator_data
from .parse_backup_flags import parse_backup_flags
from .parse_cbor import parse_cbor
from .parse_client_data_json import parse_client_data_json
from .parse_registration_credential_json import parse_registration_credential_json
from .validate_certificate_chain import validate_certificate_chain
from .verify_safetynet_timestamp import verify_safetynet_timestamp
from .verify_signature import verify_signature

__all__ = [
    "aaguid_to_string",
    "base64url_to_bytes",
    "bytes_to_base64url",
    "byteslike_to_bytes",
    "decode_credential_public_key",
    "decoded_public_key_to_cryptography",
    "encode_cbor",
    "generate_challenge",
    "generate_user_handle",
    "hash_by_alg",
    "options_to_json",
    "parse_attestation_object",
    "parse_authenticator_data",
    "parse_authentication_credential_json",
    "parse_backup_flags",
    "parse_cbor",
    "parse_client_data_json",
    "parse_registration_credential_json",
    "validate_certificate_chain",
    "verify_safetynet_timestamp",
    "verify_signature",
]

```

## File: _vendor\webauthn\helpers\asn1\android_key.py

```python
from enum import Enum

from asn1crypto.core import (
    Boolean,
    Enumerated,
    Integer,
    Null,
    OctetString,
    Sequence,
    SetOf,
)


class Integers(SetOf):
    _child_spec = Integer


class SecurityLevel(Enumerated):
    _map = {
        0: "Software",
        1: "TrustedEnvironment",
        2: "StrongBox",
    }


class VerifiedBootState(Enumerated):
    _map = {
        0: "Verified",
        1: "SelfSigned",
        2: "Unverified",
        3: "Failed",
    }


class RootOfTrust(Sequence):
    _fields = [
        ("verifiedBootKey", OctetString),
        ("deviceLocked", Boolean),
        ("verifiedBootState", VerifiedBootState),
        ("verifiedBootHash", OctetString),
    ]


class AuthorizationList(Sequence):
    _fields = [
        ("purpose", Integers, {"explicit": 1, "optional": True}),
        ("algorithm", Integer, {"explicit": 2, "optional": True}),
        ("keySize", Integer, {"explicit": 3, "optional": True}),
        ("digest", Integers, {"explicit": 5, "optional": True}),
        ("padding", Integers, {"explicit": 6, "optional": True}),
        ("ecCurve", Integer, {"explicit": 10, "optional": True}),
        ("rsaPublicExponent", Integer, {"explicit": 200, "optional": True}),
        ("rollbackResistance", Null, {"explicit": 303, "optional": True}),
        ("activeDateTime", Integer, {"explicit": 400, "optional": True}),
        ("originationExpireDateTime", Integer, {"explicit": 401, "optional": True}),
        ("usageExpireDateTime", Integer, {"explicit": 402, "optional": True}),
        ("noAuthRequired", Null, {"explicit": 503, "optional": True}),
        ("userAuthType", Integer, {"explicit": 504, "optional": True}),
        ("authTimeout", Integer, {"explicit": 505, "optional": True}),
        ("allowWhileOnBody", Null, {"explicit": 506, "optional": True}),
        ("trustedUserPresenceRequired", Null, {"explicit": 507, "optional": True}),
        ("trustedConfirmationRequired", Null, {"explicit": 508, "optional": True}),
        ("unlockedDeviceRequired", Null, {"explicit": 509, "optional": True}),
        ("allApplications", Null, {"explicit": 600, "optional": True}),
        ("applicationId", OctetString, {"explicit": 601, "optional": True}),
        ("creationDateTime", Integer, {"explicit": 701, "optional": True}),
        ("origin", Integer, {"explicit": 702, "optional": True}),
        ("rollbackResistant", Null, {"explicit": 703, "optional": True}),
        ("rootOfTrust", RootOfTrust, {"explicit": 704, "optional": True}),
        ("osVersion", Integer, {"explicit": 705, "optional": True}),
        ("osPatchLevel", Integer, {"explicit": 706, "optional": True}),
        ("attestationApplicationId", OctetString, {"explicit": 709, "optional": True}),
        ("attestationIdBrand", OctetString, {"explicit": 710, "optional": True}),
        ("attestationIdDevice", OctetString, {"explicit": 711, "optional": True}),
        ("attestationIdProduct", OctetString, {"explicit": 712, "optional": True}),
        ("attestationIdSerial", OctetString, {"explicit": 713, "optional": True}),
        ("attestationIdImei", OctetString, {"explicit": 714, "optional": True}),
        ("attestationIdMeid", OctetString, {"explicit": 715, "optional": True}),
        ("attestationIdManufacturer", OctetString, {"explicit": 716, "optional": True}),
        ("attestationIdModel", OctetString, {"explicit": 717, "optional": True}),
        ("vendorPatchLevel", Integer, {"explicit": 718, "optional": True}),
        ("bootPatchLevel", Integer, {"explicit": 719, "optional": True}),
    ]


class KeyDescription(Sequence):
    """Attestation extension content as ASN.1 schema (DER-encoded)

    Corresponds to X.509 certificate extension with the following OID:

    `1.3.6.1.4.1.11129.2.1.17`

    See https://source.android.com/security/keystore/attestation#schema
    """

    _fields = [
        ("attestationVersion", Integer),
        ("attestationSecurityLevel", SecurityLevel),
        ("keymasterVersion", Integer),
        ("keymasterSecurityLevel", SecurityLevel),
        ("attestationChallenge", OctetString),
        ("uniqueId", OctetString),
        ("softwareEnforced", AuthorizationList),
        ("teeEnforced", AuthorizationList),
    ]


class KeyOrigin(int, Enum):
    """`Tag::ORIGIN`

    See https://source.android.com/security/keystore/tags#origin
    """

    GENERATED = 0
    DERIVED = 1
    IMPORTED = 2
    UNKNOWN = 3


class KeyPurpose(int, Enum):
    """`Tag::PURPOSE`

    See https://source.android.com/security/keystore/tags#purpose
    """

    ENCRYPT = 0
    DECRYPT = 1
    SIGN = 2
    VERIFY = 3
    DERIVE_KEY = 4
    WRAP_KEY = 5

```

## File: _vendor\webauthn\helpers\asn1\__init__.py

```python

```

## File: _vendor\webauthn\helpers\tpm\parse_cert_info.py

```python
from ..exceptions import InvalidTPMCertInfoStructure
from .structs import (
    TPM_ST,
    TPM_ST_MAP,
    TPMCertInfo,
    TPMCertInfoAttested,
    TPMCertInfoClockInfo,
)


def parse_cert_info(val: bytes) -> TPMCertInfo:
    """
    Turn `response.attestationObject.attStmt.certInfo` into structured data
    """
    pointer = 0

    # The constant "TPM_GENERATED_VALUE" indicating a structure generated by TPM
    magic_bytes = val[pointer : pointer + 4]
    pointer += 4

    # Type of the cert info structure
    type_bytes = val[pointer : pointer + 2]
    pointer += 2
    mapped_type = TPM_ST_MAP[type_bytes]

    # Name of parent entity
    qualified_signer_length = int.from_bytes(val[pointer : pointer + 2], "big")
    pointer += 2
    qualified_signer = val[pointer : pointer + qualified_signer_length]
    pointer += qualified_signer_length

    # Expected hash value of `attsToBeSigned`
    extra_data_length = int.from_bytes(val[pointer : pointer + 2], "big")
    pointer += 2
    extra_data_bytes = val[pointer : pointer + extra_data_length]
    pointer += extra_data_length

    # Info about the TPM's internal clock
    clock_info_bytes = val[pointer : pointer + 17]
    pointer += 17

    # Device firmware version
    firmware_version_bytes = val[pointer : pointer + 8]
    pointer += 8

    # Verify that type is set to TPM_ST_ATTEST_CERTIFY.
    if mapped_type != TPM_ST.ATTEST_CERTIFY:
        raise InvalidTPMCertInfoStructure(
            f'Cert Info type "{mapped_type}" was not "{TPM_ST.ATTEST_CERTIFY}"'
        )

    # Attested name
    attested_name_length = int.from_bytes(val[pointer : pointer + 2], "big")
    pointer += 2
    attested_name_bytes = val[pointer : pointer + attested_name_length]
    pointer += attested_name_length
    qualified_name_length = int.from_bytes(val[pointer : pointer + 2], "big")
    pointer += 2
    qualified_name_bytes = val[pointer : pointer + qualified_name_length]
    pointer += qualified_name_length

    return TPMCertInfo(
        magic=magic_bytes,
        type=mapped_type,
        extra_data=extra_data_bytes,
        attested=TPMCertInfoAttested(attested_name_bytes, qualified_name_bytes),
        # Note that the remaining fields in the "Standard Attestation Structure"
        # [TPMv2-Part1] section 31.2, i.e., qualifiedSigner, clockInfo and
        # firmwareVersion are ignored. These fields MAY be used as an input to risk
        # engines.
        qualified_signer=qualified_signer,
        clock_info=TPMCertInfoClockInfo(clock_info_bytes),
        firmware_version=firmware_version_bytes,
    )

```

## File: _vendor\webauthn\helpers\tpm\parse_pub_area.py

```python
from ..exceptions import InvalidTPMPubAreaStructure
from .structs import (
    TPM_ALG,
    TPM_ALG_MAP,
    TPMPubArea,
    TPMPubAreaObjectAttributes,
    TPMPubAreaParametersECC,
    TPMPubAreaParametersRSA,
    TPMPubAreaUnique,
)


def parse_pub_area(val: bytes) -> TPMPubArea:
    """
    Turn `response.attestationObject.attStmt.pubArea` into structured data
    """
    pointer = 0

    type_bytes = val[pointer : pointer + 2]
    pointer += 2
    mapped_type = TPM_ALG_MAP[type_bytes]

    name_alg_bytes = val[pointer : pointer + 2]
    pointer += 2
    mapped_name_alg = TPM_ALG_MAP[name_alg_bytes]

    object_attributes_bytes = val[pointer : pointer + 4]
    pointer += 4
    # Parse attributes from right to left by zero-index bit position
    object_attributes = TPMPubAreaObjectAttributes(object_attributes_bytes)

    auth_policy_length = int.from_bytes(val[pointer : pointer + 2], "big")
    pointer += 2
    auth_policy_bytes = val[pointer : pointer + auth_policy_length]
    pointer += auth_policy_length

    # Decode the rest of the bytes to public key parameters
    if mapped_type == TPM_ALG.RSA:
        rsa_bytes = val[pointer : pointer + 10]
        pointer += 10
        parameters = TPMPubAreaParametersRSA(rsa_bytes)
    elif mapped_type == TPM_ALG.ECC:
        ecc_bytes = val[pointer : pointer + 8]
        pointer += 8
        # mypy will error here because of the incompatible "reassignment", but
        # `parameters` in `TPMPubArea` is a Union of either type so ignore the error
        parameters = TPMPubAreaParametersECC(ecc_bytes)  # type: ignore
    else:
        raise InvalidTPMPubAreaStructure(f'Type "{mapped_type}" is unsupported')

    unique_length_bytes = val[pointer:]

    return TPMPubArea(
        type=mapped_type,
        name_alg=mapped_name_alg,
        object_attributes=object_attributes,
        auth_policy=auth_policy_bytes,
        parameters=parameters,
        unique=TPMPubAreaUnique(unique_length_bytes, mapped_type),
    )

```

## File: _vendor\webauthn\helpers\tpm\structs.py

```python
from dataclasses import dataclass
from enum import Enum
from typing import Mapping, Union

from ..cose import COSECRV, COSEAlgorithmIdentifier
from ..exceptions import InvalidTPMPubAreaStructure

################
#
# A whole lotta domain knowledge is captured here, with hazy connections to source
# documents. Good places to start searching for more info on these values are the
# following Trusted Computing Group TPM Library docs linked in the WebAuthn API:
#
# - https://www.trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-1-Architecture-01.38.pdf
# - https://www.trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-2-Structures-01.38.pdf
# - https://www.trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-3-Commands-01.38.pdf
#
################


class TPM_ST(str, Enum):
    RSP_COMMAND = "TPM_ST_RSP_COMMAND"
    NULL = "TPM_ST_NULL"
    NO_SESSIONS = "TPM_ST_NO_SESSIONS"
    SESSIONS = "TPM_ST_SESSIONS"
    ATTEST_NV = "TPM_ST_ATTEST_NV"
    ATTEST_COMMAND_AUDIT = "TPM_ST_ATTEST_COMMAND_AUDIT"
    ATTEST_SESSION_AUDIT = "TPM_ST_ATTEST_SESSION_AUDIT"
    ATTEST_CERTIFY = "TPM_ST_ATTEST_CERTIFY"
    ATTEST_QUOTE = "TPM_ST_ATTEST_QUOTE"
    ATTEST_TIME = "TPM_ST_ATTEST_TIME"
    ATTEST_CREATION = "TPM_ST_ATTEST_CREATION"
    CREATION = "TPM_ST_CREATION"
    VERIFIED = "TPM_ST_VERIFIED"
    AUTH_SECRET = "TPM_ST_AUTH_SECRET"
    HASHCHECK = "TPM_ST_HASHCHECK"
    AUTH_SIGNED = "TPM_ST_AUTH_SIGNED"
    FU_MANIFEST = "TPM_ST_FU_MANIFEST"


class TPM_ALG(str, Enum):
    ERROR = "TPM_ALG_ERROR"
    RSA = "TPM_ALG_RSA"
    SHA1 = "TPM_ALG_SHA1"
    HMAC = "TPM_ALG_HMAC"
    AES = "TPM_ALG_AES"
    MGF1 = "TPM_ALG_MGF1"
    KEYEDHASH = "TPM_ALG_KEYEDHASH"
    XOR = "TPM_ALG_XOR"
    SHA256 = "TPM_ALG_SHA256"
    SHA384 = "TPM_ALG_SHA384"
    SHA512 = "TPM_ALG_SHA512"
    NULL = "TPM_ALG_NULL"
    SM3_256 = "TPM_ALG_SM3_256"
    SM4 = "TPM_ALG_SM4"
    RSASSA = "TPM_ALG_RSASSA"
    RSAES = "TPM_ALG_RSAES"
    RSAPSS = "TPM_ALG_RSAPSS"
    OAEP = "TPM_ALG_OAEP"
    ECDSA = "TPM_ALG_ECDSA"
    ECDH = "TPM_ALG_ECDH"
    ECDAA = "TPM_ALG_ECDAA"
    SM2 = "TPM_ALG_SM2"
    ECSCHNORR = "TPM_ALG_ECSCHNORR"
    ECMQV = "TPM_ALG_ECMQV"
    KDF1_SP800_56A = "TPM_ALG_KDF1_SP800_56A"
    KDF2 = "TPM_ALG_KDF2"
    KDF1_SP800_108 = "TPM_ALG_KDF1_SP800_108"
    ECC = "TPM_ALG_ECC"
    SYMCIPHER = "TPM_ALG_SYMCIPHER"
    CAMELLIA = "TPM_ALG_CAMELLIA"
    CTR = "TPM_ALG_CTR"
    OFB = "TPM_ALG_OFB"
    CBC = "TPM_ALG_CBC"
    CFB = "TPM_ALG_CFB"
    ECB = "TPM_ALG_ECB"


class TPM_ECC_CURVE(str, Enum):
    NONE = "NONE"
    NIST_P192 = "NIST_P192"
    NIST_P224 = "NIST_P224"
    NIST_P256 = "NIST_P256"
    NIST_P384 = "NIST_P384"
    NIST_P521 = "NIST_P521"
    BN_P256 = "BN_P256"
    BN_P638 = "BN_P638"
    SM2_P256 = "SM2_P256"


class TPMCertInfoClockInfo:
    """
    10.11.1 TPMS_CLOCK_INFO
    """

    clock: bytes
    reset_count: int
    restart_count: int
    safe: bool

    def __init__(self, clock_info: bytes):
        self.clock = clock_info[0:8]
        self.reset_count = int.from_bytes(clock_info[8:12], "big")
        self.restart_count = int.from_bytes(clock_info[12:16], "big")
        self.safe = bool(clock_info[16])


class TPMCertInfoAttested:
    """
    10.12.3 TPMS_CERTIFY_INFO
    """

    name_alg: TPM_ALG
    name_alg_bytes: bytes
    name: bytes
    qualified_name: bytes

    def __init__(self, attested_name: bytes, qualified_name: bytes):
        self.name_alg = TPM_ALG_MAP[attested_name[0:2]]
        self.name_alg_bytes = attested_name[0:2]
        self.name = attested_name
        self.qualified_name = qualified_name


@dataclass
class TPMCertInfo:
    """
    10.12.8 TPMS_ATTEST
    """

    magic: bytes
    type: TPM_ST
    qualified_signer: bytes
    extra_data: bytes
    clock_info: TPMCertInfoClockInfo
    firmware_version: bytes
    attested: TPMCertInfoAttested


class TPMPubAreaParametersRSA:
    """
    12.2.3.5 TPMS_RSA_PARMS
    """

    symmetric: TPM_ALG
    scheme: TPM_ALG
    key_bits: bytes
    exponent: bytes

    def __init__(self, params: bytes):
        self.symmetric = TPM_ALG_MAP[params[0:2]]
        self.scheme = TPM_ALG_MAP[params[2:4]]
        self.key_bits = params[4:6]
        self.exponent = params[6:10]


class TPMPubAreaParametersECC:
    """
    12.2.3.6 TPMS_ECC_PARMS
    """

    symmetric: TPM_ALG
    scheme: TPM_ALG
    curve_id: TPM_ECC_CURVE
    kdf: TPM_ALG

    def __init__(self, params: bytes):
        self.symmetric = TPM_ALG_MAP[params[0:2]]
        self.scheme = TPM_ALG_MAP[params[2:4]]
        self.curve_id = TPM_ECC_CURVE_MAP[params[4:6]]
        self.kdf = TPM_ALG_MAP[params[6:8]]


class TPMPubAreaObjectAttributes:
    """
    8.3 TPMA_OBJECT (Object Attributes)
    """

    fixed_tpm: bool
    st_clear: bool
    fixed_parent: bool
    sensitive_data_origin: bool
    user_with_auth: bool
    admin_with_policy: bool
    no_da: bool
    encrypted_duplication: bool
    restricted: bool
    decrypt: bool
    sign_or_encrypt: bool

    def __init__(self, object_attributes: bytes):
        attrs = int.from_bytes(object_attributes[0:4], "big")
        self.fixed_tpm = attrs & (1 << 1) != 0
        self.st_clear = attrs & (1 << 2) != 0
        self.fixed_parent = attrs & (1 << 4) != 0
        self.sensitive_data_origin = attrs & (1 << 5) != 0
        self.user_with_auth = attrs & (1 << 6) != 0
        self.admin_with_policy = attrs & (1 << 7) != 0
        self.no_da = attrs & (1 << 10) != 0
        self.encrypted_duplication = attrs & (1 << 11) != 0
        self.restricted = attrs & (1 << 16) != 0
        self.decrypt = attrs & (1 << 17) != 0
        self.sign_or_encrypt = attrs & (1 << 18) != 0


class TPMPubAreaUnique:
    """
    12.2.3.2 TPMU_PUBLIC_ID
    """

    value: bytes

    def __init__(self, unique: bytes, alg_type: TPM_ALG):
        if alg_type == TPM_ALG.RSA:
            """
            As per 11.2.4.5 TPM2B_PUBLIC_KEY_RSA, extract `unique` of dynamic length
            """
            unique_length = int.from_bytes(unique[0:2], "big")
            rsa_unique = unique[2 : 2 + unique_length]
            self.value = rsa_unique
        elif alg_type == TPM_ALG.ECC:
            """
            As per 12.2.3.2 TPMU_PUBLIC_ID, `unique` is `TPMS_ECC_POINT` when `type`
            indicates ECC.

            As per 11.2.5.2 TPMS_ECC_POINT, `x` and `y` within are
            `TPM2B_ECC_PARAMETER`, which is a uint16 `size`, followed by bytes of
            the indicated length.

            `unique`, then, is structured thusly:
            [
                [x_len, ...x_bytes],
                [y_len, ...y_bytes]
            ]

            Get `unique` to this structure for easier comparison with the output from
            `decode_credential_public_key`, which will return bytes for its `x` and `y`
            and to which this value will be compared:

            [
                x_bytes,
                y_bytes,
            ]

            """
            pointer = 0
            unique_x_len = int.from_bytes(unique[0:2], "big")
            pointer += 2
            unique_x = unique[pointer : pointer + unique_x_len]
            pointer += unique_x_len
            unique_y_len = int.from_bytes(unique[pointer : pointer + 2], "big")
            pointer += 2
            unique_y = unique[pointer : pointer + unique_y_len]

            self.value = b"".join([unique_x, unique_y])
        else:
            raise InvalidTPMPubAreaStructure(
                f'Pub Area alg type "{alg_type}" was not "{TPM_ALG.RSA}" or "{TPM_ALG.ECC}"'
            )


@dataclass
class TPMPubArea:
    """
    12.2.4 TPMT_PUBLIC
    """

    type: TPM_ALG
    name_alg: TPM_ALG
    object_attributes: TPMPubAreaObjectAttributes
    auth_policy: bytes
    parameters: Union[TPMPubAreaParametersRSA, TPMPubAreaParametersECC]
    unique: TPMPubAreaUnique


@dataclass
class TPMManufacturerInfo:
    name: str
    id: str


"""
6.9 TPM_ST (Structure Tags)
"""
TPM_ST_MAP: Mapping[bytes, TPM_ST] = {
    b"\x00\xc4": TPM_ST.RSP_COMMAND,
    b"\x80\x00": TPM_ST.NULL,
    b"\x80\x01": TPM_ST.NO_SESSIONS,
    b"\x80\x02": TPM_ST.SESSIONS,
    b"\x80\x14": TPM_ST.ATTEST_NV,
    b"\x80\x15": TPM_ST.ATTEST_COMMAND_AUDIT,
    b"\x80\x16": TPM_ST.ATTEST_SESSION_AUDIT,
    b"\x80\x17": TPM_ST.ATTEST_CERTIFY,
    b"\x80\x18": TPM_ST.ATTEST_QUOTE,
    b"\x80\x19": TPM_ST.ATTEST_TIME,
    b"\x80\x1a": TPM_ST.ATTEST_CREATION,
    b"\x80\x21": TPM_ST.CREATION,
    b"\x80\x22": TPM_ST.VERIFIED,
    b"\x80\x23": TPM_ST.AUTH_SECRET,
    b"\x80\x24": TPM_ST.HASHCHECK,
    b"\x80\x25": TPM_ST.AUTH_SIGNED,
    b"\x80\x29": TPM_ST.FU_MANIFEST,
}


"""
6.3 TPM_ALG_ID
"""
TPM_ALG_MAP: Mapping[bytes, TPM_ALG] = {
    b"\x00\x00": TPM_ALG.ERROR,
    b"\x00\x01": TPM_ALG.RSA,
    b"\x00\x04": TPM_ALG.SHA1,
    b"\x00\x05": TPM_ALG.HMAC,
    b"\x00\x06": TPM_ALG.AES,
    b"\x00\x07": TPM_ALG.MGF1,
    b"\x00\x08": TPM_ALG.KEYEDHASH,
    b"\x00\x0a": TPM_ALG.XOR,
    b"\x00\x0b": TPM_ALG.SHA256,
    b"\x00\x0c": TPM_ALG.SHA384,
    b"\x00\x0d": TPM_ALG.SHA512,
    b"\x00\x10": TPM_ALG.NULL,
    b"\x00\x12": TPM_ALG.SM3_256,
    b"\x00\x13": TPM_ALG.SM4,
    b"\x00\x14": TPM_ALG.RSASSA,
    b"\x00\x15": TPM_ALG.RSAES,
    b"\x00\x16": TPM_ALG.RSAPSS,
    b"\x00\x17": TPM_ALG.OAEP,
    b"\x00\x18": TPM_ALG.ECDSA,
    b"\x00\x19": TPM_ALG.ECDH,
    b"\x00\x1a": TPM_ALG.ECDAA,
    b"\x00\x1b": TPM_ALG.SM2,
    b"\x00\x1c": TPM_ALG.ECSCHNORR,
    b"\x00\x1d": TPM_ALG.ECMQV,
    b"\x00\x20": TPM_ALG.KDF1_SP800_56A,
    b"\x00\x21": TPM_ALG.KDF2,
    b"\x00\x22": TPM_ALG.KDF1_SP800_108,
    b"\x00\x23": TPM_ALG.ECC,
    b"\x00\x25": TPM_ALG.SYMCIPHER,
    b"\x00\x26": TPM_ALG.CAMELLIA,
    b"\x00\x40": TPM_ALG.CTR,
    b"\x00\x41": TPM_ALG.OFB,
    b"\x00\x42": TPM_ALG.CBC,
    b"\x00\x43": TPM_ALG.CFB,
    b"\x00\x44": TPM_ALG.ECB,
}


"""
6.4 TPM_ECC_CURVE
"""
TPM_ECC_CURVE_MAP: Mapping[bytes, TPM_ECC_CURVE] = {
    b"\x00\x00": TPM_ECC_CURVE.NONE,
    b"\x00\x01": TPM_ECC_CURVE.NIST_P192,
    b"\x00\x02": TPM_ECC_CURVE.NIST_P224,
    b"\x00\x03": TPM_ECC_CURVE.NIST_P256,
    b"\x00\x04": TPM_ECC_CURVE.NIST_P384,
    b"\x00\x05": TPM_ECC_CURVE.NIST_P521,
    b"\x00\x10": TPM_ECC_CURVE.BN_P256,
    b"\x00\x11": TPM_ECC_CURVE.BN_P638,
    b"\x00\x20": TPM_ECC_CURVE.SM2_P256,
}


# Intentionally omit curves we can't map so a KeyError gets thrown
TPM_ECC_CURVE_COSE_CRV_MAP: Mapping[TPM_ECC_CURVE, COSECRV] = {
    TPM_ECC_CURVE.NIST_P256: COSECRV.P256,
    TPM_ECC_CURVE.NIST_P384: COSECRV.P384,
    TPM_ECC_CURVE.NIST_P521: COSECRV.P521,
    TPM_ECC_CURVE.BN_P256: COSECRV.P256,
    TPM_ECC_CURVE.SM2_P256: COSECRV.P256,
}


# Intentionally omit algs we can't map so a KeyError gets thrown
TPM_ALG_COSE_ALG_MAP: Mapping[TPM_ALG, COSEAlgorithmIdentifier] = {
    TPM_ALG.SHA256: COSEAlgorithmIdentifier.RSASSA_PSS_SHA_256,
    TPM_ALG.SHA384: COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_384,
    TPM_ALG.SHA512: COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_512,
    TPM_ALG.SHA1: COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_1,
}


# Sourced from https://trustedcomputinggroup.org/resource/vendor-id-registry/
# Latest version: https://trustedcomputinggroup.org/wp-content/uploads/TCG-TPM-Vendor-ID-Registry-Version-1.02-Revision-1.00.pdf
TPM_MANUFACTURERS: Mapping[str, TPMManufacturerInfo] = {
    "id:414D4400": TPMManufacturerInfo(name="AMD", id="AMD"),
    "id:41544D4C": TPMManufacturerInfo(name="Atmel", id="ATML"),
    "id:4252434D": TPMManufacturerInfo(name="Broadcom", id="BRCM"),
    "id:4353434F": TPMManufacturerInfo(name="Cisco", id="CSCO"),
    "id:464C5953": TPMManufacturerInfo(name="Flyslice Technologies", id="FLYS"),
    "id:48504500": TPMManufacturerInfo(name="HPE", id="HPE"),
    "id:49424d00": TPMManufacturerInfo(name="IBM", id="IBM"),
    "id:49465800": TPMManufacturerInfo(name="Infineon", id="IFX"),
    "id:494E5443": TPMManufacturerInfo(name="Intel", id="INTC"),
    "id:4C454E00": TPMManufacturerInfo(name="Lenovo", id="LEN"),
    "id:4D534654": TPMManufacturerInfo(name="Microsoft", id="MSFT"),
    "id:4E534D20": TPMManufacturerInfo(name="National Semiconductor", id="NSM"),
    "id:4E545A00": TPMManufacturerInfo(name="Nationz", id="NTZ"),
    "id:4E544300": TPMManufacturerInfo(name="Nuvoton Technology", id="NTC"),
    "id:51434F4D": TPMManufacturerInfo(name="Qualcomm", id="QCOM"),
    "id:534D5343": TPMManufacturerInfo(name="SMSC", id="SMSC"),
    "id:53544D20": TPMManufacturerInfo(name="ST Microelectronics", id="STM"),
    "id:534D534E": TPMManufacturerInfo(name="Samsung", id="SMSN"),
    "id:534E5300": TPMManufacturerInfo(name="Sinosun", id="SNS"),
    "id:54584E00": TPMManufacturerInfo(name="Texas Instruments", id="TXN"),
    "id:57454300": TPMManufacturerInfo(name="Winbond", id="WEC"),
    "id:524F4343": TPMManufacturerInfo(name="Fuzhou Rockchip", id="ROCC"),
    "id:474F4F47": TPMManufacturerInfo(name="Google", id="GOOG"),
}

```

## File: _vendor\webauthn\helpers\tpm\__init__.py

```python
from .parse_cert_info import parse_cert_info
from .parse_pub_area import parse_pub_area

__all__ = ["parse_cert_info", "parse_pub_area"]

```

## File: _vendor\webauthn\registration\generate_registration_options.py

```python
from typing import List, Optional

from ...webauthn.helpers import generate_challenge, generate_user_handle, byteslike_to_bytes
from ...webauthn.helpers.cose import COSEAlgorithmIdentifier
from ...webauthn.helpers.structs import (
    AttestationConveyancePreference,
    AuthenticatorSelectionCriteria,
    PublicKeyCredentialCreationOptions,
    PublicKeyCredentialDescriptor,
    PublicKeyCredentialParameters,
    PublicKeyCredentialRpEntity,
    PublicKeyCredentialUserEntity,
    ResidentKeyRequirement,
)


def _generate_pub_key_cred_params(
    supported_algs: List[COSEAlgorithmIdentifier],
) -> List[PublicKeyCredentialParameters]:
    """
    Take an array of algorithm ID ints and return an array of PublicKeyCredentialParameters
    """
    return [PublicKeyCredentialParameters(type="public-key", alg=alg) for alg in supported_algs]


default_supported_pub_key_algs = [
    COSEAlgorithmIdentifier.ECDSA_SHA_256,
    COSEAlgorithmIdentifier.EDDSA,
    COSEAlgorithmIdentifier.ECDSA_SHA_512,
    COSEAlgorithmIdentifier.RSASSA_PSS_SHA_256,
    COSEAlgorithmIdentifier.RSASSA_PSS_SHA_384,
    COSEAlgorithmIdentifier.RSASSA_PSS_SHA_512,
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_256,
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_384,
    COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_512,
]
default_supported_pub_key_params = _generate_pub_key_cred_params(
    default_supported_pub_key_algs,
)


def generate_registration_options(
    *,
    rp_id: str,
    rp_name: str,
    user_name: str,
    user_id: Optional[bytes] = None,
    user_display_name: Optional[str] = None,
    challenge: Optional[bytes] = None,
    timeout: int = 60000,
    attestation: AttestationConveyancePreference = AttestationConveyancePreference.NONE,
    authenticator_selection: Optional[AuthenticatorSelectionCriteria] = None,
    exclude_credentials: Optional[List[PublicKeyCredentialDescriptor]] = None,
    supported_pub_key_algs: Optional[List[COSEAlgorithmIdentifier]] = None,
) -> PublicKeyCredentialCreationOptions:
    """Generate options for registering a credential via navigator.credentials.create()

    Args:
        `rp_id`: A unique, constant identifier for this Relying Party.
        `rp_name`: A user-friendly, readable name for the Relying Party.
        `user_name`: A value that will help the user identify which account this credential is associated with. Can be an email address, etc...
        (optional) `user_id`: A collection of random bytes that identify a user account. For privacy reasons it should NOT be something like an email address. Defaults to 64 random bytes.
        (optional) `user_display_name`: A user-friendly representation of their account. Can be a full name ,etc... Defaults to the value of `user_name`.
        (optional) `challenge`: A byte sequence for the authenticator to return back in its response. Defaults to 64 random bytes.
        (optional) `timeout`: How long in milliseconds the browser should give the user to choose an authenticator. This value is a *hint* and may be ignored by the browser.
        (optional) `attestation`: The level of attestation to be provided by the authenticator.
        (optional) `authenticator_selection`: Require certain characteristics about an authenticator, like attachment, support for resident keys, user verification, etc...
        (optional) `exclude_credentials`: A list of credentials the user has previously registered so that they cannot re-register them.
        (optional) `supported_pub_key_algs`: A list of public key algorithm IDs the RP chooses to restrict support to. Defaults to all supported algorithm IDs.

    Returns:
        Registration options ready for the browser. Consider using `helpers.options_to_json()` in this library to quickly convert the options to JSON.
    """

    if not rp_id:
        raise ValueError("rp_id cannot be an empty string")

    if not rp_name:
        raise ValueError("rp_name cannot be an empty string")

    if not user_name:
        raise ValueError("user_name cannot be an empty string")

    if user_id:
        if not isinstance(user_id, bytes):
            raise ValueError("user_id must be bytes")
    else:
        user_id = generate_user_handle()

    ########
    # Set defaults for required values
    ########

    if not user_display_name:
        user_display_name = user_name

    pub_key_cred_params = default_supported_pub_key_params
    if supported_pub_key_algs:
        pub_key_cred_params = _generate_pub_key_cred_params(supported_pub_key_algs)

    if not challenge:
        challenge = generate_challenge()

    if not exclude_credentials:
        exclude_credentials = []

    ########
    # Generate the actual options
    ########

    options = PublicKeyCredentialCreationOptions(
        rp=PublicKeyCredentialRpEntity(
            name=rp_name,
            id=rp_id,
        ),
        user=PublicKeyCredentialUserEntity(
            id=user_id,
            name=user_name,
            display_name=user_display_name,
        ),
        challenge=challenge,
        pub_key_cred_params=pub_key_cred_params,
        timeout=timeout,
        exclude_credentials=exclude_credentials,
        attestation=attestation,
    )

    ########
    # Set optional values if specified
    ########

    if authenticator_selection is not None:
        # "Relying Parties SHOULD set [requireResidentKey] to true if, and only if,
        # residentKey is set to "required""
        #
        # See https://www.w3.org/TR/webauthn-2/#dom-authenticatorselectioncriteria-requireresidentkey
        if authenticator_selection.resident_key == ResidentKeyRequirement.REQUIRED:
            authenticator_selection.require_resident_key = True
        options.authenticator_selection = authenticator_selection

    return options

```

## File: _vendor\webauthn\registration\verify_registration_response.py

```python
import hashlib
from dataclasses import dataclass, asdict
from typing import List, Mapping, Optional, Union

from ...webauthn.helpers import (
    aaguid_to_string,
    bytes_to_base64url,
    byteslike_to_bytes,
    decode_credential_public_key,
    parse_attestation_object,
    parse_client_data_json,
    parse_backup_flags,
    parse_registration_credential_json,
)
from ...webauthn.helpers.cose import COSEAlgorithmIdentifier
from ...webauthn.helpers.exceptions import InvalidRegistrationResponse
from ...webauthn.helpers.structs import (
    AttestationFormat,
    ClientDataType,
    CredentialDeviceType,
    PublicKeyCredentialType,
    RegistrationCredential,
    TokenBindingStatus,
)
from .formats.android_key import verify_android_key
from .formats.android_safetynet import verify_android_safetynet
from .formats.apple import verify_apple
from .formats.fido_u2f import verify_fido_u2f
from .formats.packed import verify_packed
from .formats.tpm import verify_tpm
from .generate_registration_options import default_supported_pub_key_algs


@dataclass
class VerifiedRegistration:
    """Information about a verified attestation of which an RP can make use.

    Attributes:
        `credential_id`: The generated credential's ID
        `credential_public_key`: The generated credential's public key
        `sign_count`: How many times the authenticator says the credential was used
        `aaguid`: A 128-bit identifier indicating the type and vendor of the authenticator
        `fmt`: The attestation format
        `credential_type`: The literal string "public-key"
        `user_verified`: Whether the user was verified by the authenticator
        `attestation_object`: The raw attestation object for later scrutiny
    """

    credential_id: bytes
    credential_public_key: bytes
    sign_count: int
    aaguid: str
    fmt: AttestationFormat
    credential_type: PublicKeyCredentialType
    user_verified: bool
    attestation_object: bytes
    credential_device_type: CredentialDeviceType
    credential_backed_up: bool


expected_token_binding_statuses = [
    TokenBindingStatus.SUPPORTED,
    TokenBindingStatus.PRESENT,
]


def verify_registration_response(
    *,
    credential: Union[str, dict, RegistrationCredential],
    expected_challenge: bytes,
    expected_rp_id: str,
    expected_origin: Union[str, List[str]],
    require_user_verification: bool = False,
    supported_pub_key_algs: List[COSEAlgorithmIdentifier] = default_supported_pub_key_algs,
    pem_root_certs_bytes_by_fmt: Optional[Mapping[AttestationFormat, List[bytes]]] = None,
) -> VerifiedRegistration:
    """Verify an authenticator's response to navigator.credentials.create()

    Args:
        - `credential`: The value returned from `navigator.credentials.create()`. Can be either a
          stringified JSON object, a plain dict, or an instance of RegistrationCredential
        - `expected_challenge`: The challenge passed to the authenticator within the preceding
          registration options.
        - `expected_rp_id`: The Relying Party's unique identifier as specified in the precending
          registration options.
        - `expected_origin`: The domain, with HTTP protocol (e.g. "https://domain.here"), on which
          the registration should have occurred. Can also be a list of expected origins.
        - (optional) `require_user_verification`: Whether or not to require that the authenticator
          verified the user.
        - (optional) `supported_pub_key_algs`: A list of public key algorithm IDs the RP chooses to
          restrict support to. Defaults to all supported algorithm IDs.
        - (optional) `pem_root_certs_bytes_by_fmt`: A list of root certificates, in PEM format, to
          be used to validate the certificate chains for specific attestation statement formats.

    Returns:
        Information about the authenticator and registration

    Raises:
        `helpers.exceptions.InvalidRegistrationResponse` if the response cannot be verified
    """
    if isinstance(credential, str) or isinstance(credential, dict):
        credential = parse_registration_credential_json(credential)

    verified = False

    # FIDO-specific check
    if bytes_to_base64url(credential.raw_id) != credential.id:
        raise InvalidRegistrationResponse("id and raw_id were not equivalent")

    # FIDO-specific check
    if credential.type != PublicKeyCredentialType.PUBLIC_KEY:
        raise InvalidRegistrationResponse(
            f'Unexpected credential type "{credential.type}", expected "public-key"'
        )

    response = credential.response

    client_data_bytes = byteslike_to_bytes(response.client_data_json)
    attestation_object_bytes = byteslike_to_bytes(response.attestation_object)

    client_data = parse_client_data_json(client_data_bytes)

    if client_data.type != ClientDataType.WEBAUTHN_CREATE:
        raise InvalidRegistrationResponse(
            f'Unexpected client data type "{client_data.type}", expected "{ClientDataType.WEBAUTHN_CREATE}"'
        )

    if expected_challenge != client_data.challenge:
        raise InvalidRegistrationResponse("Client data challenge was not expected challenge")

    if isinstance(expected_origin, str):
        if expected_origin != client_data.origin:
            raise InvalidRegistrationResponse(
                f'Unexpected client data origin "{client_data.origin}", expected "{expected_origin}"'
            )
    else:
        try:
            expected_origin.index(client_data.origin)
        except ValueError:
            raise InvalidRegistrationResponse(
                f'Unexpected client data origin "{client_data.origin}", expected one of {expected_origin}'
            )

    if client_data.token_binding:
        status = client_data.token_binding.status
        if status not in expected_token_binding_statuses:
            raise InvalidRegistrationResponse(
                f'Unexpected token_binding status of "{status}", expected one of "{",".join(expected_token_binding_statuses)}"'
            )

    attestation_object = parse_attestation_object(attestation_object_bytes)

    auth_data = attestation_object.auth_data

    # Generate a hash of the expected RP ID for comparison
    expected_rp_id_hash = hashlib.sha256()
    expected_rp_id_hash.update(expected_rp_id.encode("utf-8"))
    expected_rp_id_hash_bytes = expected_rp_id_hash.digest()

    if auth_data.rp_id_hash != expected_rp_id_hash_bytes:
        raise InvalidRegistrationResponse("Unexpected RP ID hash")

    if not auth_data.flags.up:
        raise InvalidRegistrationResponse("User was not present during attestation")

    if require_user_verification and not auth_data.flags.uv:
        raise InvalidRegistrationResponse(
            "User verification is required but user was not verified during attestation"
        )

    if not auth_data.attested_credential_data:
        raise InvalidRegistrationResponse("Authenticator did not provide attested credential data")

    attested_credential_data = auth_data.attested_credential_data

    if not attested_credential_data.credential_id:
        raise InvalidRegistrationResponse("Authenticator did not provide a credential ID")

    if not attested_credential_data.credential_public_key:
        raise InvalidRegistrationResponse("Authenticator did not provide a credential public key")

    if not attested_credential_data.aaguid:
        raise InvalidRegistrationResponse("Authenticator did not provide an AAGUID")

    decoded_credential_public_key = decode_credential_public_key(
        attested_credential_data.credential_public_key
    )

    if decoded_credential_public_key.alg not in supported_pub_key_algs:
        raise InvalidRegistrationResponse(
            f'Unsupported credential public key alg "{decoded_credential_public_key.alg}", expected one of: {supported_pub_key_algs}'
        )

    # Prepare a list of possible root certificates for certificate chain validation
    pem_root_certs_bytes: List[bytes] = []
    if pem_root_certs_bytes_by_fmt:
        custom_certs = pem_root_certs_bytes_by_fmt.get(attestation_object.fmt)
        if custom_certs:
            # Load any provided custom root certs
            pem_root_certs_bytes.extend(custom_certs)

    if attestation_object.fmt == AttestationFormat.NONE:
        # A "none" attestation should not contain _anything_ in its attestation statement
        any_att_stmt_fields_set = any(
            [field is not None for field in asdict(attestation_object.att_stmt).values()]
        )

        if any_att_stmt_fields_set:
            raise InvalidRegistrationResponse(
                "None attestation had unexpected attestation statement"
            )

        # There's nothing else to verify, so mark the verification successful
        verified = True
    elif attestation_object.fmt == AttestationFormat.FIDO_U2F:
        verified = verify_fido_u2f(
            attestation_statement=attestation_object.att_stmt,
            client_data_json=client_data_bytes,
            rp_id_hash=auth_data.rp_id_hash,
            credential_id=attested_credential_data.credential_id,
            credential_public_key=attested_credential_data.credential_public_key,
            aaguid=attested_credential_data.aaguid,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    elif attestation_object.fmt == AttestationFormat.PACKED:
        verified = verify_packed(
            attestation_statement=attestation_object.att_stmt,
            attestation_object=attestation_object_bytes,
            client_data_json=client_data_bytes,
            credential_public_key=attested_credential_data.credential_public_key,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    elif attestation_object.fmt == AttestationFormat.TPM:
        verified = verify_tpm(
            attestation_statement=attestation_object.att_stmt,
            attestation_object=attestation_object_bytes,
            client_data_json=client_data_bytes,
            credential_public_key=attested_credential_data.credential_public_key,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    elif attestation_object.fmt == AttestationFormat.APPLE:
        verified = verify_apple(
            attestation_statement=attestation_object.att_stmt,
            attestation_object=attestation_object_bytes,
            client_data_json=client_data_bytes,
            credential_public_key=attested_credential_data.credential_public_key,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    elif attestation_object.fmt == AttestationFormat.ANDROID_SAFETYNET:
        verified = verify_android_safetynet(
            attestation_statement=attestation_object.att_stmt,
            attestation_object=attestation_object_bytes,
            client_data_json=client_data_bytes,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    elif attestation_object.fmt == AttestationFormat.ANDROID_KEY:
        verified = verify_android_key(
            attestation_statement=attestation_object.att_stmt,
            attestation_object=attestation_object_bytes,
            client_data_json=client_data_bytes,
            credential_public_key=attested_credential_data.credential_public_key,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    else:
        # Raise exception on an attestation format we're not prepared to verify
        raise InvalidRegistrationResponse(
            f'Unsupported attestation type "{attestation_object.fmt}"'
        )

    # If we got this far and still couldn't verify things then raise an error instead
    # of simply returning False
    if not verified:
        raise InvalidRegistrationResponse("Attestation statement could not be verified")

    parsed_backup_flags = parse_backup_flags(auth_data.flags)

    return VerifiedRegistration(
        credential_id=attested_credential_data.credential_id,
        credential_public_key=attested_credential_data.credential_public_key,
        sign_count=auth_data.sign_count,
        aaguid=aaguid_to_string(attested_credential_data.aaguid),
        fmt=attestation_object.fmt,
        credential_type=credential.type,
        user_verified=auth_data.flags.uv,
        attestation_object=attestation_object_bytes,
        credential_device_type=parsed_backup_flags.credential_device_type,
        credential_backed_up=parsed_backup_flags.credential_backed_up,
    )

```

## File: _vendor\webauthn\registration\__init__.py

```python

```

## File: _vendor\webauthn\registration\formats\android_key.py

```python
import hashlib
from typing import List

from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat
from cryptography.x509 import (
    Extension,
    ExtensionNotFound,
    ObjectIdentifier,
    UnrecognizedExtension,
)

from ....webauthn.helpers import (
    decode_credential_public_key,
    decoded_public_key_to_cryptography,
    parse_cbor,
    validate_certificate_chain,
    verify_signature,
)
from ....webauthn.helpers.asn1.android_key import (
    AuthorizationList,
    KeyDescription,
    KeyOrigin,
    KeyPurpose,
)
from ....webauthn.helpers.exceptions import (
    InvalidCertificateChain,
    InvalidRegistrationResponse,
)
from ....webauthn.helpers.known_root_certs import (
    google_hardware_attestation_root_1,
    google_hardware_attestation_root_2,
)
from ....webauthn.helpers.structs import AttestationStatement


def verify_android_key(
    *,
    attestation_statement: AttestationStatement,
    attestation_object: bytes,
    client_data_json: bytes,
    credential_public_key: bytes,
    pem_root_certs_bytes: List[bytes],
) -> bool:
    """Verify an "android-key" attestation statement

    See https://www.w3.org/TR/webauthn-2/#sctn-android-key-attestation

    Also referenced: https://source.android.com/security/keystore/attestation
    """
    if not attestation_statement.sig:
        raise InvalidRegistrationResponse(
            "Attestation statement was missing signature (Android Key)"
        )

    if not attestation_statement.alg:
        raise InvalidRegistrationResponse(
            "Attestation statement was missing algorithm (Android Key)"
        )

    if not attestation_statement.x5c:
        raise InvalidRegistrationResponse("Attestation statement was missing x5c (Android Key)")

    # Validate certificate chain
    try:
        # Include known root certificates for this attestation format
        pem_root_certs_bytes.append(google_hardware_attestation_root_1)
        pem_root_certs_bytes.append(google_hardware_attestation_root_2)

        validate_certificate_chain(
            x5c=attestation_statement.x5c,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    except InvalidCertificateChain as err:
        raise InvalidRegistrationResponse(f"{err} (Android Key)")

    # Extract attStmt bytes from attestation_object
    attestation_dict = parse_cbor(attestation_object)
    authenticator_data_bytes = attestation_dict["authData"]

    # Generate a hash of client_data_json
    client_data_hash = hashlib.sha256()
    client_data_hash.update(client_data_json)
    client_data_hash_bytes = client_data_hash.digest()

    verification_data = b"".join(
        [
            authenticator_data_bytes,
            client_data_hash_bytes,
        ]
    )

    # Verify that sig is a valid signature over the concatenation of authenticatorData
    # and clientDataHash using the public key in the first certificate in x5c with the
    # algorithm specified in alg.
    attestation_cert_bytes = attestation_statement.x5c[0]
    attestation_cert = x509.load_der_x509_certificate(attestation_cert_bytes, default_backend())
    attestation_cert_pub_key = attestation_cert.public_key()

    try:
        verify_signature(
            public_key=attestation_cert_pub_key,
            signature_alg=attestation_statement.alg,
            signature=attestation_statement.sig,
            data=verification_data,
        )
    except InvalidSignature:
        raise InvalidRegistrationResponse(
            "Could not verify attestation statement signature (Android Key)"
        )

    # Verify that the public key in the first certificate in x5c matches the
    # credentialPublicKey in the attestedCredentialData in authenticatorData.
    attestation_cert_pub_key_bytes = attestation_cert_pub_key.public_bytes(
        Encoding.DER,
        PublicFormat.SubjectPublicKeyInfo,
    )
    # Convert our raw public key bytes into the same format cryptography generates for
    # the cert subject key
    decoded_pub_key = decode_credential_public_key(credential_public_key)
    pub_key_crypto = decoded_public_key_to_cryptography(decoded_pub_key)
    pub_key_crypto_bytes = pub_key_crypto.public_bytes(
        Encoding.DER,
        PublicFormat.SubjectPublicKeyInfo,
    )

    if attestation_cert_pub_key_bytes != pub_key_crypto_bytes:
        raise InvalidRegistrationResponse(
            "Certificate public key did not match credential public key (Android Key)"
        )

    # Verify that the attestationChallenge field in the attestation certificate
    # extension data is identical to clientDataHash.
    ext_key_description_oid = "1.3.6.1.4.1.11129.2.1.17"
    try:
        cert_extensions = attestation_cert.extensions
        ext_key_description: Extension = cert_extensions.get_extension_for_oid(
            ObjectIdentifier(ext_key_description_oid)
        )
    except ExtensionNotFound:
        raise InvalidRegistrationResponse(
            f"Certificate missing extension {ext_key_description_oid} (Android Key)"
        )

    # Peel apart the Extension into an UnrecognizedExtension, then the bytes we actually
    # want
    ext_value_wrapper: UnrecognizedExtension = ext_key_description.value
    ext_value: bytes = ext_value_wrapper.value
    parsed_ext = KeyDescription.load(ext_value)

    # Verify the following using the appropriate authorization list from the attestation
    # certificate extension data:
    software_enforced: AuthorizationList = parsed_ext["softwareEnforced"]
    tee_enforced: AuthorizationList = parsed_ext["teeEnforced"]

    # The AuthorizationList.allApplications field is not present on either authorization
    # list (softwareEnforced nor teeEnforced), since PublicKeyCredential MUST be scoped
    # to the RP ID.
    if software_enforced["allApplications"].native is not None:
        raise InvalidRegistrationResponse(
            "allApplications field was present in softwareEnforced (Android Key)"
        )

    if tee_enforced["allApplications"].native is not None:
        raise InvalidRegistrationResponse(
            "allApplications field was present in teeEnforced (Android Key)"
        )

    # The value in the AuthorizationList.origin field is equal to KM_ORIGIN_GENERATED.
    origin = tee_enforced["origin"].native
    if origin != KeyOrigin.GENERATED:
        raise InvalidRegistrationResponse(
            f"teeEnforced.origin {origin} was not {KeyOrigin.GENERATED}"
        )

    # The value in the AuthorizationList.purpose field is equal to KM_PURPOSE_SIGN.
    purpose = tee_enforced["purpose"].native
    if purpose != [KeyPurpose.SIGN]:
        raise InvalidRegistrationResponse(
            f"teeEnforced.purpose {purpose} was not [{KeyPurpose.SIGN}]"
        )

    return True

```

## File: _vendor\webauthn\registration\formats\android_safetynet.py

```python
import base64
from dataclasses import dataclass
import hashlib
import json
from typing import List

from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.backends import default_backend
from cryptography.x509.oid import NameOID

from ....webauthn.helpers.cose import COSEAlgorithmIdentifier
from ....webauthn.helpers import (
    base64url_to_bytes,
    parse_cbor,
    validate_certificate_chain,
    verify_safetynet_timestamp,
    verify_signature,
)
from ....webauthn.helpers.exceptions import (
    InvalidCertificateChain,
    InvalidRegistrationResponse,
)
from ....webauthn.helpers.known_root_certs import globalsign_r2, globalsign_root_ca
from ....webauthn.helpers.structs import AttestationStatement


@dataclass
class SafetyNetJWSHeader:
    """Properties in the Header of a SafetyNet JWS"""

    alg: str
    x5c: List[str]


@dataclass
class SafetyNetJWSPayload:
    """Properties in the Payload of a SafetyNet JWS

    Values below correspond to camelCased properties in the JWS itself. This class
    handles converting the properties to Pythonic snake_case.
    """

    nonce: str
    timestamp_ms: int
    apk_package_name: str
    apk_digest_sha256: str
    cts_profile_match: bool
    apk_certificate_digest_sha256: List[str]
    basic_integrity: bool


def verify_android_safetynet(
    *,
    attestation_statement: AttestationStatement,
    attestation_object: bytes,
    client_data_json: bytes,
    pem_root_certs_bytes: List[bytes],
    verify_timestamp_ms: bool = True,
) -> bool:
    """Verify an "android-safetynet" attestation statement

    See https://www.w3.org/TR/webauthn-2/#sctn-android-safetynet-attestation

    Notes:
        - `verify_timestamp_ms` is a kind of escape hatch specifically for enabling
          testing of this method. Without this we can't use static responses in unit
          tests because they'll always evaluate as expired. This flag can be removed
          from this method if we ever figure out how to dynamically create
          safetynet-formatted responses that can be immediately tested.
    """

    if not attestation_statement.ver:
        # As of this writing, there is only one format of the SafetyNet response and
        # ver is reserved for future use (so for now just make sure it's present)
        raise InvalidRegistrationResponse("Attestation statement was missing version (SafetyNet)")

    if not attestation_statement.response:
        raise InvalidRegistrationResponse("Attestation statement was missing response (SafetyNet)")

    # Begin peeling apart the JWS in the attestation statement response
    jws = attestation_statement.response.decode("ascii")
    jws_parts = jws.split(".")

    if len(jws_parts) != 3:
        raise InvalidRegistrationResponse("Response JWS did not have three parts (SafetyNet)")

    header_json = json.loads(base64url_to_bytes(jws_parts[0]))
    payload_json = json.loads(base64url_to_bytes(jws_parts[1]))

    header = SafetyNetJWSHeader(
        alg=header_json.get("alg", ""),
        x5c=header_json.get("x5c", []),
    )
    payload = SafetyNetJWSPayload(
        nonce=payload_json.get("nonce", ""),
        timestamp_ms=payload_json.get("timestampMs", 0),
        apk_package_name=payload_json.get("apkPackageName", ""),
        apk_digest_sha256=payload_json.get("apkDigestSha256", ""),
        cts_profile_match=payload_json.get("ctsProfileMatch", False),
        apk_certificate_digest_sha256=payload_json.get("apkCertificateDigestSha256", []),
        basic_integrity=payload_json.get("basicIntegrity", False),
    )

    signature_bytes_str: str = jws_parts[2]

    # Verify that the nonce attribute in the payload of response is identical to the
    # Base64 encoding of the SHA-256 hash of the concatenation of authenticatorData and
    # clientDataHash.

    # Extract attStmt bytes from attestation_object
    attestation_dict = parse_cbor(attestation_object)
    authenticator_data_bytes = attestation_dict["authData"]

    # Generate a hash of client_data_json
    client_data_hash = hashlib.sha256()
    client_data_hash.update(client_data_json)
    client_data_hash_bytes = client_data_hash.digest()

    nonce_data = b"".join(
        [
            authenticator_data_bytes,
            client_data_hash_bytes,
        ]
    )
    # Start with a sha256 hash
    nonce_data_hash = hashlib.sha256()
    nonce_data_hash.update(nonce_data)
    nonce_data_hash_bytes = nonce_data_hash.digest()
    # Encode to base64
    nonce_data_hash_bytes = base64.b64encode(nonce_data_hash_bytes)
    # Finish by decoding to string
    nonce_data_str = nonce_data_hash_bytes.decode("utf-8")

    if payload.nonce != nonce_data_str:
        raise InvalidRegistrationResponse("Payload nonce was not expected value (SafetyNet)")

    # Verify that the SafetyNet response actually came from the SafetyNet service
    # by following the steps in the SafetyNet online documentation.
    x5c = [base64url_to_bytes(cert) for cert in header.x5c]

    if not payload.cts_profile_match:
        raise InvalidRegistrationResponse("Could not verify device integrity (SafetyNet)")

    if verify_timestamp_ms:
        try:
            verify_safetynet_timestamp(payload.timestamp_ms)
        except ValueError as err:
            raise InvalidRegistrationResponse(f"{err} (SafetyNet)")

    # Verify that the leaf certificate was issued to the hostname attest.android.com
    attestation_cert = x509.load_der_x509_certificate(x5c[0], default_backend())
    cert_common_name = attestation_cert.subject.get_attributes_for_oid(
        NameOID.COMMON_NAME,
    )[0]

    if cert_common_name.value != "attest.android.com":
        raise InvalidRegistrationResponse(
            'Certificate common name was not "attest.android.com" (SafetyNet)'
        )

    # Validate certificate chain
    try:
        # Include known root certificates for this attestation format with whatever
        # other certs were provided
        pem_root_certs_bytes.append(globalsign_r2)
        pem_root_certs_bytes.append(globalsign_root_ca)

        validate_certificate_chain(
            x5c=x5c,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    except InvalidCertificateChain as err:
        raise InvalidRegistrationResponse(f"{err} (SafetyNet)")

    # Verify signature
    verification_data = f"{jws_parts[0]}.{jws_parts[1]}".encode("utf-8")
    signature_bytes = base64url_to_bytes(signature_bytes_str)

    if header.alg != "RS256":
        raise InvalidRegistrationResponse(f"JWS header alg was not RS256: {header.alg} (SafetyNet")

    # Get cert public key bytes
    attestation_cert_pub_key = attestation_cert.public_key()

    try:
        verify_signature(
            public_key=attestation_cert_pub_key,
            signature_alg=COSEAlgorithmIdentifier.RSASSA_PKCS1_v1_5_SHA_256,
            signature=signature_bytes,
            data=verification_data,
        )
    except InvalidSignature:
        raise InvalidRegistrationResponse(
            "Could not verify attestation statement signature (Packed)"
        )

    return True

```

## File: _vendor\webauthn\registration\formats\apple.py

```python
import hashlib
from typing import List

import cbor2
from cryptography import x509
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat
from cryptography.x509 import (
    Extension,
    ExtensionNotFound,
    ObjectIdentifier,
    UnrecognizedExtension,
)

from ....webauthn.helpers import (
    decode_credential_public_key,
    decoded_public_key_to_cryptography,
    parse_cbor,
    validate_certificate_chain,
)
from ....webauthn.helpers.exceptions import (
    InvalidCertificateChain,
    InvalidRegistrationResponse,
)
from ....webauthn.helpers.known_root_certs import apple_webauthn_root_ca
from ....webauthn.helpers.structs import AttestationStatement


def verify_apple(
    *,
    attestation_statement: AttestationStatement,
    attestation_object: bytes,
    client_data_json: bytes,
    credential_public_key: bytes,
    pem_root_certs_bytes: List[bytes],
) -> bool:
    """
    https://www.w3.org/TR/webauthn-2/#sctn-apple-anonymous-attestation
    """

    if not attestation_statement.x5c:
        raise InvalidRegistrationResponse("Attestation statement was missing x5c (Apple)")

    # Validate the certificate chain
    try:
        # Include known root certificates for this attestation format
        pem_root_certs_bytes.append(apple_webauthn_root_ca)

        validate_certificate_chain(
            x5c=attestation_statement.x5c,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    except InvalidCertificateChain as err:
        raise InvalidRegistrationResponse(f"{err} (Apple)")

    # Concatenate authenticatorData and clientDataHash to form nonceToHash.
    attestation_dict = parse_cbor(attestation_object)
    authenticator_data_bytes = attestation_dict["authData"]

    client_data_hash = hashlib.sha256()
    client_data_hash.update(client_data_json)
    client_data_hash_bytes = client_data_hash.digest()

    nonce_to_hash = b"".join(
        [
            authenticator_data_bytes,
            client_data_hash_bytes,
        ]
    )

    # Perform SHA-256 hash of nonceToHash to produce nonce.
    nonce = hashlib.sha256()
    nonce.update(nonce_to_hash)
    nonce_bytes = nonce.digest()

    # Verify that nonce equals the value of the extension with
    # OID 1.2.840.113635.100.8.2 in credCert.
    attestation_cert_bytes = attestation_statement.x5c[0]
    attestation_cert = x509.load_der_x509_certificate(attestation_cert_bytes, default_backend())
    cert_extensions = attestation_cert.extensions

    # Still no documented name for this OID...
    ext_1_2_840_113635_100_8_2_oid = "1.2.840.113635.100.8.2"
    try:
        ext_1_2_840_113635_100_8_2: Extension = cert_extensions.get_extension_for_oid(
            ObjectIdentifier(ext_1_2_840_113635_100_8_2_oid)
        )
    except ExtensionNotFound:
        raise InvalidRegistrationResponse(
            f"Certificate missing extension {ext_1_2_840_113635_100_8_2_oid} (Apple)"
        )

    # Peel apart the Extension into an UnrecognizedExtension, then the bytes we actually
    # want
    ext_value_wrapper: UnrecognizedExtension = ext_1_2_840_113635_100_8_2.value
    # Ignore the first six ASN.1 structure bytes that define the nonce as an
    # OCTET STRING. Should trim off '0$\xa1"\x04'
    ext_value: bytes = ext_value_wrapper.value[6:]

    if ext_value != nonce_bytes:
        raise InvalidRegistrationResponse("Certificate nonce was not expected value (Apple)")

    # Verify that the credential public key equals the Subject Public Key of credCert.
    attestation_cert_pub_key = attestation_cert.public_key()
    attestation_cert_pub_key_bytes = attestation_cert_pub_key.public_bytes(
        Encoding.DER,
        PublicFormat.SubjectPublicKeyInfo,
    )
    # Convert our raw public key bytes into the same format cryptography generates for
    # the cert subject key
    decoded_pub_key = decode_credential_public_key(credential_public_key)
    pub_key_crypto = decoded_public_key_to_cryptography(decoded_pub_key)
    pub_key_crypto_bytes = pub_key_crypto.public_bytes(
        Encoding.DER,
        PublicFormat.SubjectPublicKeyInfo,
    )

    if attestation_cert_pub_key_bytes != pub_key_crypto_bytes:
        raise InvalidRegistrationResponse(
            "Certificate public key did not match credential public key (Apple)"
        )

    return True

```

## File: _vendor\webauthn\registration\formats\fido_u2f.py

```python
import hashlib
from typing import List

from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric.ec import (
    SECP256R1,
    EllipticCurvePublicKey,
)

from ....webauthn.helpers import (
    aaguid_to_string,
    validate_certificate_chain,
    verify_signature,
)
from ....webauthn.helpers.cose import COSEAlgorithmIdentifier
from ....webauthn.helpers.decode_credential_public_key import (
    DecodedEC2PublicKey,
    decode_credential_public_key,
)
from ....webauthn.helpers.exceptions import (
    InvalidCertificateChain,
    InvalidRegistrationResponse,
)
from ....webauthn.helpers.structs import AttestationStatement


def verify_fido_u2f(
    *,
    attestation_statement: AttestationStatement,
    client_data_json: bytes,
    rp_id_hash: bytes,
    credential_id: bytes,
    credential_public_key: bytes,
    aaguid: bytes,
    pem_root_certs_bytes: List[bytes],
) -> bool:
    """Verify a "fido-u2f" attestation statement

    See https://www.w3.org/TR/webauthn-2/#sctn-fido-u2f-attestation
    """
    if not attestation_statement.sig:
        raise InvalidRegistrationResponse("Attestation statement was missing signature (FIDO-U2F)")

    if not attestation_statement.x5c:
        raise InvalidRegistrationResponse(
            "Attestation statement was missing certificate (FIDO-U2F)"
        )

    if len(attestation_statement.x5c) > 1:
        raise InvalidRegistrationResponse(
            "Attestation statement contained too many certificates (FIDO-U2F)"
        )

    # Validate the certificate chain
    try:
        validate_certificate_chain(
            x5c=attestation_statement.x5c,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    except InvalidCertificateChain as err:
        raise InvalidRegistrationResponse(f"{err} (FIDO-U2F)")

    # FIDO spec requires AAGUID in U2F attestations to be all zeroes
    # See https://fidoalliance.org/specs/fido-v2.1-rd-20191217/fido-client-to-authenticator-protocol-v2.1-rd-20191217.html#u2f-authenticatorMakeCredential-interoperability
    actual_aaguid = aaguid_to_string(aaguid)
    expected_aaguid = "00000000-0000-0000-0000-000000000000"
    if actual_aaguid != expected_aaguid:
        raise InvalidRegistrationResponse(
            f"AAGUID {actual_aaguid} was not expected {expected_aaguid} (FIDO-U2F)"
        )

    # Get the public key from the leaf certificate
    leaf_cert_bytes = attestation_statement.x5c[0]
    leaf_cert = x509.load_der_x509_certificate(leaf_cert_bytes, default_backend())
    leaf_cert_pub_key = leaf_cert.public_key()

    # We need the cert's x and y points so make sure they exist
    if not isinstance(leaf_cert_pub_key, EllipticCurvePublicKey):
        raise InvalidRegistrationResponse("Leaf cert was not an EC2 certificate (FIDO-U2F)")

    if not isinstance(leaf_cert_pub_key.curve, SECP256R1):
        raise InvalidRegistrationResponse("Leaf cert did not use P-256 curve (FIDO-U2F)")

    decoded_public_key = decode_credential_public_key(credential_public_key)
    if not isinstance(decoded_public_key, DecodedEC2PublicKey):
        raise InvalidRegistrationResponse("Credential public key was not EC2 (FIDO-U2F)")

    # Convert the public key to "Raw ANSI X9.62 public key format"
    public_key_u2f = b"".join(
        [
            bytes([0x04]),
            decoded_public_key.x,
            decoded_public_key.y,
        ]
    )

    # Generate a hash of client_data_json
    client_data_hash = hashlib.sha256()
    client_data_hash.update(client_data_json)
    client_data_hash_bytes = client_data_hash.digest()

    # Prepare the signature base (called "verificationData" in the WebAuthn spec)
    verification_data = b"".join(
        [
            bytes([0x00]),
            rp_id_hash,
            client_data_hash_bytes,
            credential_id,
            public_key_u2f,
        ]
    )

    try:
        verify_signature(
            public_key=leaf_cert_pub_key,
            signature_alg=COSEAlgorithmIdentifier.ECDSA_SHA_256,
            signature=attestation_statement.sig,
            data=verification_data,
        )
    except InvalidSignature:
        raise InvalidRegistrationResponse(
            "Could not verify attestation statement signature (FIDO-U2F)"
        )

    # If we make it to here we're all good
    return True

```

## File: _vendor\webauthn\registration\formats\packed.py

```python
import hashlib
from typing import List

from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.backends import default_backend

from ....webauthn.helpers import (
    decode_credential_public_key,
    decoded_public_key_to_cryptography,
    parse_cbor,
    validate_certificate_chain,
    verify_signature,
)
from ....webauthn.helpers.exceptions import (
    InvalidCertificateChain,
    InvalidRegistrationResponse,
)
from ....webauthn.helpers.structs import AttestationStatement


def verify_packed(
    *,
    attestation_statement: AttestationStatement,
    attestation_object: bytes,
    client_data_json: bytes,
    credential_public_key: bytes,
    pem_root_certs_bytes: List[bytes],
) -> bool:
    """Verify a "packed" attestation statement

    See https://www.w3.org/TR/webauthn-2/#sctn-packed-attestation
    """
    if not attestation_statement.sig:
        raise InvalidRegistrationResponse("Attestation statement was missing signature (Packed)")

    if not attestation_statement.alg:
        raise InvalidRegistrationResponse("Attestation statement was missing algorithm (Packed)")

    # Extract attStmt bytes from attestation_object
    attestation_dict = parse_cbor(attestation_object)
    authenticator_data_bytes = attestation_dict["authData"]

    # Generate a hash of client_data_json
    client_data_hash = hashlib.sha256()
    client_data_hash.update(client_data_json)
    client_data_hash_bytes = client_data_hash.digest()

    verification_data = b"".join(
        [
            authenticator_data_bytes,
            client_data_hash_bytes,
        ]
    )

    if attestation_statement.x5c:
        # Validate the certificate chain
        try:
            validate_certificate_chain(
                x5c=attestation_statement.x5c,
                pem_root_certs_bytes=pem_root_certs_bytes,
            )
        except InvalidCertificateChain as err:
            raise InvalidRegistrationResponse(f"{err} (Packed)")

        attestation_cert_bytes = attestation_statement.x5c[0]
        attestation_cert = x509.load_der_x509_certificate(
            attestation_cert_bytes, default_backend()
        )
        attestation_cert_pub_key = attestation_cert.public_key()

        try:
            verify_signature(
                public_key=attestation_cert_pub_key,
                signature_alg=attestation_statement.alg,
                signature=attestation_statement.sig,
                data=verification_data,
            )
        except InvalidSignature:
            raise InvalidRegistrationResponse(
                "Could not verify attestation statement signature (Packed)"
            )
    else:
        # Self Attestation
        decoded_pub_key = decode_credential_public_key(credential_public_key)

        if decoded_pub_key.alg != attestation_statement.alg:
            raise InvalidRegistrationResponse(
                f"Credential public key alg {decoded_pub_key.alg} did not equal attestation statement alg {attestation_statement.alg}"
            )

        public_key = decoded_public_key_to_cryptography(decoded_pub_key)

        try:
            verify_signature(
                public_key=public_key,
                signature_alg=attestation_statement.alg,
                signature=attestation_statement.sig,
                data=verification_data,
            )
        except InvalidSignature:
            raise InvalidRegistrationResponse(
                "Could not verify attestation statement signature (Packed|Self)"
            )

    return True

```

## File: _vendor\webauthn\registration\formats\tpm.py

```python
from typing import List

import cbor2
from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.backends import default_backend
from cryptography.x509 import (
    ExtendedKeyUsage,
    GeneralName,
    Name,
    SubjectAlternativeName,
    Version,
    BasicConstraints,
)
from cryptography.x509.extensions import ExtensionNotFound
from cryptography.x509.oid import ExtensionOID

from ....webauthn.helpers import (
    decode_credential_public_key,
    hash_by_alg,
    parse_cbor,
    validate_certificate_chain,
    verify_signature,
)
from ....webauthn.helpers.decode_credential_public_key import (
    DecodedEC2PublicKey,
    DecodedRSAPublicKey,
)
from ....webauthn.helpers.exceptions import (
    InvalidCertificateChain,
    InvalidRegistrationResponse,
)
from ....webauthn.helpers.structs import AttestationStatement
from ....webauthn.helpers.tpm import parse_cert_info, parse_pub_area
from ....webauthn.helpers.tpm.structs import (
    TPM_ALG_COSE_ALG_MAP,
    TPM_ECC_CURVE_COSE_CRV_MAP,
    TPM_MANUFACTURERS,
    TPMPubAreaParametersECC,
    TPMPubAreaParametersRSA,
)


def verify_tpm(
    *,
    attestation_statement: AttestationStatement,
    attestation_object: bytes,
    client_data_json: bytes,
    credential_public_key: bytes,
    pem_root_certs_bytes: List[bytes],
) -> bool:
    """Verify a "tpm" attestation statement

    See https://www.w3.org/TR/webauthn-2/#sctn-tpm-attestation
    """
    if not attestation_statement.cert_info:
        raise InvalidRegistrationResponse("Attestation statement was missing certInfo (TPM)")

    if not attestation_statement.pub_area:
        raise InvalidRegistrationResponse("Attestation statement was missing pubArea (TPM)")

    if not attestation_statement.alg:
        raise InvalidRegistrationResponse("Attestation statement was missing alg (TPM)")

    if not attestation_statement.x5c:
        raise InvalidRegistrationResponse("Attestation statement was missing x5c (TPM)")

    if not attestation_statement.sig:
        raise InvalidRegistrationResponse("Attestation statement was missing sig (TPM)")

    att_stmt_ver = attestation_statement.ver
    if att_stmt_ver != "2.0":
        raise InvalidRegistrationResponse(
            f'Attestation statement ver "{att_stmt_ver}" was not "2.0" (TPM)'
        )

    # Validate the certificate chain
    try:
        validate_certificate_chain(
            x5c=attestation_statement.x5c,
            pem_root_certs_bytes=pem_root_certs_bytes,
        )
    except InvalidCertificateChain as err:
        raise InvalidRegistrationResponse(f"{err} (TPM)")

    # Verify that the public key specified by the parameters and unique fields of
    # pubArea is identical to the credentialPublicKey in the attestedCredentialData
    # in authenticatorData.
    pub_area = parse_pub_area(attestation_statement.pub_area)
    decoded_public_key = decode_credential_public_key(credential_public_key)

    if isinstance(pub_area.parameters, TPMPubAreaParametersRSA):
        if not isinstance(decoded_public_key, DecodedRSAPublicKey):
            raise InvalidRegistrationResponse(
                "Public key was not RSA key as indicated in pubArea (TPM)"
            )

        if pub_area.unique.value != decoded_public_key.n:
            unique_hex = pub_area.unique.value.hex()
            pub_key_n_hex = decoded_public_key.n.hex()
            raise InvalidRegistrationResponse(
                f'PubArea unique "{unique_hex}" was not same as public key modulus "{pub_key_n_hex}" (TPM)'
            )

        pub_area_exponent = int.from_bytes(pub_area.parameters.exponent, "big")
        if pub_area_exponent == 0:
            # "When zero, indicates that the exponent is the default of 2^16 + 1"
            pub_area_exponent = 65537

        pub_key_exponent = int.from_bytes(decoded_public_key.e, "big")

        if pub_area_exponent != pub_key_exponent:
            raise InvalidRegistrationResponse(
                f'PubArea exponent "{pub_area_exponent}" was not same as public key exponent "{pub_key_exponent}" (TPM)'
            )
    elif isinstance(pub_area.parameters, TPMPubAreaParametersECC):
        if not isinstance(decoded_public_key, DecodedEC2PublicKey):
            raise InvalidRegistrationResponse(
                "Public key was not ECC key as indicated in pubArea (TPM)"
            )

        pubKeyCoords = b"".join([decoded_public_key.x, decoded_public_key.y])
        if pub_area.unique.value != pubKeyCoords:
            unique_hex = pub_area.unique.value.hex()
            pub_key_xy_hex = pubKeyCoords.hex()
            raise InvalidRegistrationResponse(
                f'Unique "{unique_hex}" was not same as public key [x,y] "{pub_key_xy_hex}" (TPM)'
            )

        pub_area_crv = TPM_ECC_CURVE_COSE_CRV_MAP[pub_area.parameters.curve_id]
        if pub_area_crv != decoded_public_key.crv:
            raise InvalidRegistrationResponse(
                f'PubArea curve ID "{pub_area_crv}" was not same as public key crv "{decoded_public_key.crv}" (TPM)'
            )
    else:
        pub_area_param_type = type(pub_area.parameters)
        raise InvalidRegistrationResponse(
            f'Unsupported pub_area.parameters "{pub_area_param_type}" (TPM)'
        )

    # Validate that certInfo is valid:
    cert_info = parse_cert_info(attestation_statement.cert_info)

    # Verify that magic is set to TPM_GENERATED_VALUE.
    # a.k.a. 0xff544347
    magic_int = int.from_bytes(cert_info.magic, "big")
    if magic_int != int(0xFF544347):
        raise InvalidRegistrationResponse(
            f'CertInfo magic "{magic_int}" was not TPM_GENERATED_VALUE 4283712327 (0xff544347) (TPM)'
        )

    # Concatenate authenticatorData and clientDataHash to form attToBeSigned.
    attestation_dict = parse_cbor(attestation_object)
    authenticator_data_bytes: bytes = attestation_dict["authData"]
    client_data_hash = hash_by_alg(client_data_json)
    att_to_be_signed = b"".join(
        [
            authenticator_data_bytes,
            client_data_hash,
        ]
    )

    # Verify that extraData is set to the hash of attToBeSigned using the hash algorithm employed in "alg".
    att_to_be_signed_hash = hash_by_alg(att_to_be_signed, attestation_statement.alg)
    if cert_info.extra_data != att_to_be_signed_hash:
        raise InvalidRegistrationResponse(
            "PubArea extra data did not match hash of auth data and client data (TPM)"
        )

    # Verify that attested contains a TPMS_CERTIFY_INFO structure as specified in
    # [TPMv2-Part2] section 10.12.3, whose name field contains a valid Name for
    # pubArea, as computed using the algorithm in the nameAlg field of pubArea using
    # the procedure specified in [TPMv2-Part1] section 16.
    pub_area_hash = hash_by_alg(
        attestation_statement.pub_area,
        TPM_ALG_COSE_ALG_MAP[pub_area.name_alg],
    )

    attested_name = b"".join(
        [
            cert_info.attested.name_alg_bytes,
            pub_area_hash,
        ]
    )

    if attested_name != cert_info.attested.name:
        raise InvalidRegistrationResponse(
            "CertInfo attested name did not match PubArea hash (TPM)"
        )

    # Verify the sig is a valid signature over certInfo using the attestation
    # public key in aikCert with the algorithm specified in alg.
    attestation_cert_bytes = attestation_statement.x5c[0]
    attestation_cert = x509.load_der_x509_certificate(attestation_cert_bytes, default_backend())
    attestation_cert_pub_key = attestation_cert.public_key()

    try:
        verify_signature(
            public_key=attestation_cert_pub_key,
            signature_alg=attestation_statement.alg,
            signature=attestation_statement.sig,
            data=attestation_statement.cert_info,
        )
    except InvalidSignature:
        raise InvalidRegistrationResponse("Could not verify attestation statement signature (TPM)")

    # Verify that aikCert meets the requirements in § 8.3.1 TPM Attestation Statement
    # Certificate Requirements.
    # https://w3c.github.io/webauthn/#sctn-tpm-cert-requirements

    # Version MUST be set to 3.
    if attestation_cert.version != Version.v3:
        raise InvalidRegistrationResponse(
            f'Certificate Version "{attestation_cert.version}" was not "{Version.v3}"" Constraints CA was not False (TPM)'
        )

    # Subject field MUST be set to empty.
    if len(attestation_cert.subject) > 0:
        raise InvalidRegistrationResponse(
            f'Certificate Subject "{attestation_cert.subject}" was not empty (TPM)'
        )

    # Start extensions analysis
    cert_extensions = attestation_cert.extensions

    # The Subject Alternative Name extension MUST be set as defined in
    # [TPMv2-EK-Profile] section 3.2.9.
    try:
        # Ignore mypy because we're casting to a known type
        ext_subject_alt_name: SubjectAlternativeName = cert_extensions.get_extension_for_oid(
            ExtensionOID.SUBJECT_ALTERNATIVE_NAME
        ).value  # type: ignore[assignment]
    except ExtensionNotFound:
        raise InvalidRegistrationResponse(
            f"Certificate missing extension {ExtensionOID.SUBJECT_ALTERNATIVE_NAME} (TPM)"
        )

    # `type(tcg_at_tpm_values)` return "<class 'cryptography.x509.name.Name'>" so ignore mypy
    tcg_at_tpm_values: Name = ext_subject_alt_name.get_values_for_type(GeneralName)[0]  # type: ignore[arg-type, assignment]
    tcg_at_tpm_manufacturer = None
    tcg_at_tpm_model = None
    tcg_at_tpm_version = None
    for obj in tcg_at_tpm_values:
        oid = obj.oid.dotted_string
        if oid == "2.23.133.2.1":
            tcg_at_tpm_manufacturer = str(obj.value)
        elif oid == "2.23.133.2.2":
            tcg_at_tpm_model = obj.value
        elif oid == "2.23.133.2.3":
            tcg_at_tpm_version = obj.value

    if not tcg_at_tpm_manufacturer or not tcg_at_tpm_model or not tcg_at_tpm_version:
        raise InvalidRegistrationResponse(
            f"Certificate Subject Alt Name was invalid value {tcg_at_tpm_values} (TPM)",
        )

    try:
        TPM_MANUFACTURERS[tcg_at_tpm_manufacturer]
    except KeyError:
        raise InvalidRegistrationResponse(
            f'Unrecognized TPM Manufacturer "{tcg_at_tpm_manufacturer}" (TPM)'
        )

    # The Extended Key Usage extension MUST contain the OID 2.23.133.8.3
    # ("joint-iso-itu-t(2) internationalorganizations(23) 133 tcg-kp(8)
    # tcg-kp-AIKCertificate(3)").
    try:
        # Ignore mypy because we're casting to a known type
        ext_extended_key_usage: ExtendedKeyUsage = cert_extensions.get_extension_for_oid(
            ExtensionOID.EXTENDED_KEY_USAGE
        ).value  # type: ignore[assignment]
    except ExtensionNotFound:
        raise InvalidRegistrationResponse(
            f"Certificate missing extension {ExtensionOID.EXTENDED_KEY_USAGE} (TPM)"
        )

    ext_key_usage_oid = ext_extended_key_usage[0].dotted_string

    if ext_key_usage_oid != "2.23.133.8.3":
        raise InvalidRegistrationResponse(
            f'Certificate Extended Key Usage OID "{ext_key_usage_oid}" was not "2.23.133.8.3" (TPM)'
        )

    try:
        # Ignore mypy because we're casting to a known type
        ext_basic_constraints: BasicConstraints = cert_extensions.get_extension_for_oid(
            ExtensionOID.BASIC_CONSTRAINTS
        ).value  # type: ignore[assignment]
    except ExtensionNotFound:
        raise InvalidRegistrationResponse(
            f"Certificate missing extension {ExtensionOID.BASIC_CONSTRAINTS} (TPM)"
        )

    # The Basic Constraints extension MUST have the CA component set to false.
    if ext_basic_constraints.ca is not False:
        raise InvalidRegistrationResponse("Certificate Basic Constraints CA was not False (TPM)")

    # If aikCert contains an extension with OID 1.3.6.1.4.1.45724.1.1.4
    # (id-fido-gen-ce-aaguid) verify that the value of this extension matches the
    # aaguid in authenticatorData.
    # TODO: Implement this later if we can find a TPM that returns something here
    # try:
    #     fido_gen_ce_aaguid = cert_extensions.get_extension_for_oid(
    #         ObjectIdentifier("1.3.6.1.4.1.45724.1.1.4")
    #     )
    # except ExtensionNotFound:
    #     pass

    return True

```

## File: _vendor\webauthn\registration\formats\__init__.py

```python

```

