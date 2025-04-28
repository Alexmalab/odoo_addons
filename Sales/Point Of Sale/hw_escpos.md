# Odoo Module: hw_escpos

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import escpos

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'ESC/POS Hardware Driver',
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'website': 'https://www.odoo.com/page/point-of-sale-hardware',
    'summary': 'Hardware Driver for ESC/POS Printers and Cashdrawers',
    'description': """
ESC/POS Hardware Driver
=======================

This module allows Odoo to print with ESC/POS compatible printers and
to open ESC/POS controlled cashdrawers in the point of sale and other modules
that would need such functionality.

""",
    'depends': ['hw_proxy'],
    'external_dependencies': {
        'python' : ['pyusb','pyserial','qrcode'],
    },
    'installable': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from __future__ import print_function
import logging
import math
import os
import os.path
import subprocess
import time
import netifaces as ni
import traceback

try: 
    from .. escpos import *
    from .. escpos.exceptions import *
    from .. escpos.printer import Usb
except ImportError:
    escpos = printer = None

try:
    from queue import Queue
except ImportError:
    from Queue import Queue # pylint: disable=deprecated-module
from threading import Thread, Lock

try:
    import usb.core
except ImportError:
    usb = None

from odoo import http, _
from odoo.addons.hw_proxy.controllers import main as hw_proxy

_logger = logging.getLogger(__name__)

# workaround https://bugs.launchpad.net/openobject-server/+bug/947231
# related to http://bugs.python.org/issue7980
from datetime import datetime
datetime.strptime('2012-01-01', '%Y-%m-%d')

class EscposDriver(Thread):
    def __init__(self):
        Thread.__init__(self)
        self.queue = Queue()
        self.lock  = Lock()
        self.status = {'status':'connecting', 'messages':[]}

    def connected_usb_devices(self):
        connected = []

        # printers can either define bDeviceClass=7, or they can define one of
        # their interfaces with bInterfaceClass=7. This class checks for both.
        class FindUsbClass(object):
            def __init__(self, usb_class):
                self._class = usb_class
            def __call__(self, device):
                # first, let's check the device
                if device.bDeviceClass == self._class:
                    return True
                # transverse all devices and look through their interfaces to
                # find a matching class
                for cfg in device:
                    intf = usb.util.find_descriptor(cfg, bInterfaceClass=self._class)

                    if intf is not None:
                        return True

                return False

        printers = usb.core.find(find_all=True, custom_match=FindUsbClass(7))

        # if no printers are found after this step we will take the
        # first epson or star device we can find.
        # epson
        if not printers:
            printers = usb.core.find(find_all=True, idVendor=0x04b8)
        # star
        if not printers:
            printers = usb.core.find(find_all=True, idVendor=0x0519)

        for printer in printers:
            try:
                if usb.__version__ == '1.0.0b1':
                    description = usb.util.get_string(printer, 256, printer.iManufacturer) + " " + usb.util.get_string(printer, 256, printer.iProduct)
                else:
                    description = usb.util.get_string(printer, printer.iManufacturer) + " " + usb.util.get_string(printer, printer.iProduct)
            except Exception as e:
                _logger.error("Can not get printer description: %s" % e)
                description = 'Unknown printer'
            connected.append({
                'vendor': printer.idVendor,
                'product': printer.idProduct,
                'name': description
            })

        return connected

    def lockedstart(self):
        with self.lock:
            if not self.is_alive():
                self.daemon = True
                self.start()
    
    def get_escpos_printer(self):
  
        printers = self.connected_usb_devices()
        if len(printers) > 0:
            try:
                print_dev = Usb(printers[0]['vendor'], printers[0]['product'])
            except HandleDeviceError:
                # Escpos printers are now integrated to PrinterDriver, if the IoTBox is printing
                # through Cups at the same time, we get an USBError(16, 'Resource busy'). This means
                # that the Odoo instance connected to this IoTBox is up to date and no longer uses
                # this escpos library.
                return None
            self.set_status(
                'connected',
                "Connected to %s (in=0x%02x,out=0x%02x)" % (printers[0]['name'], print_dev.in_ep, print_dev.out_ep)
            )
            return print_dev
        else:
            self.set_status('disconnected','Printer Not Found')
            return None

    def get_status(self):
        self.push_task('status')
        return self.status

    def open_cashbox(self,printer):
        printer.cashdraw(2)
        printer.cashdraw(5)

    def set_status(self, status, message = None):
        _logger.info(status+' : '+ (message or 'no message'))
        if status == self.status['status']:
            if message != None and (len(self.status['messages']) == 0 or message != self.status['messages'][-1]):
                self.status['messages'].append(message)
        else:
            self.status['status'] = status
            if message:
                self.status['messages'] = [message]
            else:
                self.status['messages'] = []

        if status == 'error' and message:
            _logger.error('ESC/POS Error: %s', message)
        elif status == 'disconnected' and message:
            _logger.warning('ESC/POS Device Disconnected: %s', message)

    def run(self):
        printer = None
        if not escpos:
            _logger.error('ESC/POS cannot initialize, please verify system dependencies.')
            return
        while True:
            try:
                error = True
                timestamp, task, data = self.queue.get(True)

                printer = self.get_escpos_printer()

                if printer == None:
                    if task != 'status':
                        self.queue.put((timestamp,task,data))
                    error = False
                    time.sleep(5)
                    continue
                elif task == 'receipt': 
                    if timestamp >= time.time() - 1 * 60 * 60:
                        self.print_receipt_body(printer,data)
                        printer.cut()
                elif task == 'xml_receipt':
                    if timestamp >= time.time() - 1 * 60 * 60:
                        printer.receipt(data)
                elif task == 'cashbox':
                    if timestamp >= time.time() - 12:
                        self.open_cashbox(printer)
                elif task == 'status':
                    pass
                error = False

            except NoDeviceError as e:
                print("No device found %s" % e)
            except HandleDeviceError as e:
                printer = None
                print("Impossible to handle the device due to previous error %s" % e)
            except TicketNotPrinted as e:
                print("The ticket does not seems to have been fully printed %s" % e)
            except NoStatusError as e:
                print("Impossible to get the status of the printer %s" % e)
            except Exception as e:
                self.set_status('error')
                _logger.exception(e)
            finally:
                if error:
                    self.queue.put((timestamp, task, data))
                if printer:
                    printer.close()
                    printer = None

    def push_task(self,task, data = None):
        self.lockedstart()
        self.queue.put((time.time(),task,data))

    def print_receipt_body(self,eprint,receipt):

        def check(string):
            return string != True and bool(string) and string.strip()
        
        def price(amount):
            return ("{0:."+str(receipt['precision']['price'])+"f}").format(amount)
        
        def money(amount):
            return ("{0:."+str(receipt['precision']['money'])+"f}").format(amount)

        def quantity(amount):
            if math.floor(amount) != amount:
                return ("{0:."+str(receipt['precision']['quantity'])+"f}").format(amount)
            else:
                return str(amount)

        def printline(left, right='', width=40, ratio=0.5, indent=0):
            lwidth = int(width * ratio) 
            rwidth = width - lwidth 
            lwidth = lwidth - indent
            
            left = left[:lwidth]
            if len(left) != lwidth:
                left = left + ' ' * (lwidth - len(left))

            right = right[-rwidth:]
            if len(right) != rwidth:
                right = ' ' * (rwidth - len(right)) + right

            return ' ' * indent + left + right + '\n'
        
        def print_taxes():
            taxes = receipt['tax_details']
            for tax in taxes:
                eprint.text(printline(tax['tax']['name'],price(tax['amount']), width=40,ratio=0.6))

        # Receipt Header
        if receipt['company']['logo']:
            eprint.set(align='center')
            eprint.print_base64_image(receipt['company']['logo'])
            eprint.text('\n')
        else:
            eprint.set(align='center',type='b',height=2,width=2)
            eprint.text(receipt['company']['name'] + '\n')

        eprint.set(align='center',type='b')
        if check(receipt['company']['contact_address']):
            eprint.text(receipt['company']['contact_address'] + '\n')
        if check(receipt['company']['phone']):
            eprint.text('Tel:' + receipt['company']['phone'] + '\n')
        if check(receipt['company']['vat']):
            eprint.text('VAT:' + receipt['company']['vat'] + '\n')
        if check(receipt['company']['email']):
            eprint.text(receipt['company']['email'] + '\n')
        if check(receipt['company']['website']):
            eprint.text(receipt['company']['website'] + '\n')
        if check(receipt['header']):
            eprint.text(receipt['header']+'\n')
        if check(receipt['cashier']):
            eprint.text('-'*32+'\n')
            eprint.text('Served by '+receipt['cashier']+'\n')

        # Orderlines
        eprint.text('\n\n')
        eprint.set(align='center')
        for line in receipt['orderlines']:
            pricestr = price(line['price_display'])
            if line['discount'] == 0 and line['unit_name'] == 'Units' and line['quantity'] == 1:
                eprint.text(printline(line['product_name'],pricestr,ratio=0.6))
            else:
                eprint.text(printline(line['product_name'],ratio=0.6))
                if line['discount'] != 0:
                    eprint.text(printline('Discount: '+str(line['discount'])+'%', ratio=0.6, indent=2))
                if line['unit_name'] == 'Units':
                    eprint.text( printline( quantity(line['quantity']) + ' x ' + price(line['price']), pricestr, ratio=0.6, indent=2))
                else:
                    eprint.text( printline( quantity(line['quantity']) + line['unit_name'] + ' x ' + price(line['price']), pricestr, ratio=0.6, indent=2))

        # Subtotal if the taxes are not included
        taxincluded = True
        if money(receipt['subtotal']) != money(receipt['total_with_tax']):
            eprint.text(printline('','-------'));
            eprint.text(printline(_('Subtotal'),money(receipt['subtotal']),width=40, ratio=0.6))
            print_taxes()
            #eprint.text(printline(_('Taxes'),money(receipt['total_tax']),width=40, ratio=0.6))
            taxincluded = False


        # Total
        eprint.text(printline('','-------'));
        eprint.set(align='center',height=2)
        eprint.text(printline(_('         TOTAL'),money(receipt['total_with_tax']),width=40, ratio=0.6))
        eprint.text('\n\n');
        
        # Paymentlines
        eprint.set(align='center')
        for line in receipt['paymentlines']:
            eprint.text(printline(line['journal'], money(line['amount']), ratio=0.6))

        eprint.text('\n');
        eprint.set(align='center',height=2)
        eprint.text(printline(_('        CHANGE'),money(receipt['change']),width=40, ratio=0.6))
        eprint.set(align='center')
        eprint.text('\n');

        # Extra Payment info
        if receipt['total_discount'] != 0:
            eprint.text(printline(_('Discounts'),money(receipt['total_discount']),width=40, ratio=0.6))
        if taxincluded:
            print_taxes()
            #eprint.text(printline(_('Taxes'),money(receipt['total_tax']),width=40, ratio=0.6))

        # Footer
        if check(receipt['footer']):
            eprint.text('\n'+receipt['footer']+'\n\n')
        eprint.text(receipt['name']+'\n')
        eprint.text(      str(receipt['date']['date']).zfill(2)
                    +'/'+ str(receipt['date']['month']+1).zfill(2)
                    +'/'+ str(receipt['date']['year']).zfill(4)
                    +' '+ str(receipt['date']['hour']).zfill(2)
                    +':'+ str(receipt['date']['minute']).zfill(2) )


driver = EscposDriver()

hw_proxy.drivers['escpos'] = driver

class EscposProxy(hw_proxy.Proxy):
    
    @http.route('/hw_proxy/open_cashbox', type='json', auth='none', cors='*')
    def open_cashbox(self):
        _logger.info('ESC/POS: OPEN CASHBOX') 
        driver.push_task('cashbox')
        
    @http.route('/hw_proxy/print_receipt', type='json', auth='none', cors='*')
    def print_receipt(self, receipt):
        _logger.info('ESC/POS: PRINT RECEIPT') 
        driver.push_task('receipt',receipt)

    @http.route('/hw_proxy/print_xml_receipt', type='json', auth='none', cors='*')
    def print_xml_receipt(self, receipt):
        _logger.info('ESC/POS: PRINT XML RECEIPT') 
        driver.push_task('xml_receipt',receipt)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: escpos\constants.py

```python
# -*- coding: utf-8 -*-

""" ESC/POS Commands (Constants) """

# Control characters
ESC = '\x1b'

# Feed control sequences
CTL_LF    = '\x0a'             # Print and line feed
CTL_FF    = '\x0c'             # Form feed
CTL_CR    = '\x0d'             # Carriage return
CTL_HT    = '\x09'             # Horizontal tab
CTL_VT    = '\x0b'             # Vertical tab

# RT Status commands
DLE_EOT_PRINTER   = '\x10\x04\x01'  # Transmit printer status
DLE_EOT_OFFLINE   = '\x10\x04\x02'
DLE_EOT_ERROR     = '\x10\x04\x03'
DLE_EOT_PAPER     = '\x10\x04\x04'

# Printer hardware
HW_INIT   = '\x1b\x40'         # Clear data in buffer and reset modes
HW_SELECT = '\x1b\x3d\x01'     # Printer select
HW_RESET  = '\x1b\x3f\x0a\x00' # Reset printer hardware
# Cash Drawer (ESC p <pin> <on time: 2*ms> <off time: 2*ms>)
_CASH_DRAWER = lambda m, t1='', t2='': ESC + 'p' + m + chr(t1) + chr(t2)
CD_KICK_2 = _CASH_DRAWER('\x00', 50, 50)  # Sends a pulse to pin 2 []
CD_KICK_5 = _CASH_DRAWER('\x01', 50, 50)  # Sends a pulse to pin 5 []
# Paper
PAPER_FULL_CUT  = '\x1d\x56\x00' # Full cut paper
PAPER_PART_CUT  = '\x1d\x56\x01' # Partial cut paper
# Text format   
TXT_NORMAL      = '\x1b\x21\x00' # Normal text
TXT_2HEIGHT     = '\x1b\x21\x10' # Double height text
TXT_2WIDTH      = '\x1b\x21\x20' # Double width text
TXT_DOUBLE      = '\x1b\x21\x30' # Double height & Width
TXT_UNDERL_OFF  = '\x1b\x2d\x00' # Underline font OFF
TXT_UNDERL_ON   = '\x1b\x2d\x01' # Underline font 1-dot ON
TXT_UNDERL2_ON  = '\x1b\x2d\x02' # Underline font 2-dot ON
TXT_BOLD_OFF    = '\x1b\x45\x00' # Bold font OFF
TXT_BOLD_ON     = '\x1b\x45\x01' # Bold font ON
TXT_FONT_A      = '\x1b\x4d\x00' # Font type A
TXT_FONT_B      = '\x1b\x4d\x01' # Font type B
TXT_ALIGN_LT    = '\x1b\x61\x00' # Left justification
TXT_ALIGN_CT    = '\x1b\x61\x01' # Centering
TXT_ALIGN_RT    = '\x1b\x61\x02' # Right justification
TXT_COLOR_BLACK = '\x1b\x72\x00' # Default Color
TXT_COLOR_RED   = '\x1b\x72\x01' # Alternative Color ( Usually Red )

# Text Encoding

TXT_ENC_PC437   = '\x1b\x74\x00' # PC437 USA
TXT_ENC_KATAKANA= '\x1b\x74\x01' # KATAKANA (JAPAN)
TXT_ENC_PC850   = '\x1b\x74\x02' # PC850 Multilingual
TXT_ENC_PC860   = '\x1b\x74\x03' # PC860 Portuguese
TXT_ENC_PC863   = '\x1b\x74\x04' # PC863 Canadian-French
TXT_ENC_PC865   = '\x1b\x74\x05' # PC865 Nordic
TXT_ENC_KANJI6  = '\x1b\x74\x06' # One-pass Kanji, Hiragana
TXT_ENC_KANJI7  = '\x1b\x74\x07' # One-pass Kanji 
TXT_ENC_KANJI8  = '\x1b\x74\x08' # One-pass Kanji
TXT_ENC_PC851   = '\x1b\x74\x0b' # PC851 Greek
TXT_ENC_PC853   = '\x1b\x74\x0c' # PC853 Turkish
TXT_ENC_PC857   = '\x1b\x74\x0d' # PC857 Turkish 
TXT_ENC_PC737   = '\x1b\x74\x0e' # PC737 Greek
TXT_ENC_8859_7  = '\x1b\x74\x0f' # ISO8859-7 Greek
TXT_ENC_WPC1252 = '\x1b\x74\x10' # WPC1252
TXT_ENC_PC866   = '\x1b\x74\x11' # PC866 Cyrillic #2
TXT_ENC_PC852   = '\x1b\x74\x12' # PC852 Latin2
TXT_ENC_PC858   = '\x1b\x74\x13' # PC858 Euro
TXT_ENC_KU42    = '\x1b\x74\x14' # KU42 Thai
TXT_ENC_TIS11   = '\x1b\x74\x15' # TIS11 Thai
TXT_ENC_TIS18   = '\x1b\x74\x1a' # TIS18 Thai
TXT_ENC_TCVN3   = '\x1b\x74\x1e' # TCVN3 Vietnamese
TXT_ENC_TCVN3B  = '\x1b\x74\x1f' # TCVN3 Vietnamese
TXT_ENC_PC720   = '\x1b\x74\x20' # PC720 Arabic
TXT_ENC_WPC775  = '\x1b\x74\x21' # WPC775 Baltic Rim
TXT_ENC_PC855   = '\x1b\x74\x22' # PC855 Cyrillic 
TXT_ENC_PC861   = '\x1b\x74\x23' # PC861 Icelandic
TXT_ENC_PC862   = '\x1b\x74\x24' # PC862 Hebrew
TXT_ENC_PC864   = '\x1b\x74\x25' # PC864 Arabic
TXT_ENC_PC869   = '\x1b\x74\x26' # PC869 Greek
TXT_ENC_PC936   = '\x1C\x21\x00' # PC936 GBK(Guobiao Kuozhan)
TXT_ENC_8859_2  = '\x1b\x74\x27' # ISO8859-2 Latin2
TXT_ENC_8859_9  = '\x1b\x74\x28' # ISO8859-2 Latin9
TXT_ENC_PC1098  = '\x1b\x74\x29' # PC1098 Farsi
TXT_ENC_PC1118  = '\x1b\x74\x2a' # PC1118 Lithuanian
TXT_ENC_PC1119  = '\x1b\x74\x2b' # PC1119 Lithuanian
TXT_ENC_PC1125  = '\x1b\x74\x2c' # PC1125 Ukrainian
TXT_ENC_WPC1250 = '\x1b\x74\x2d' # WPC1250 Latin2
TXT_ENC_WPC1251 = '\x1b\x74\x2e' # WPC1251 Cyrillic
TXT_ENC_WPC1253 = '\x1b\x74\x2f' # WPC1253 Greek
TXT_ENC_WPC1254 = '\x1b\x74\x30' # WPC1254 Turkish
TXT_ENC_WPC1255 = '\x1b\x74\x31' # WPC1255 Hebrew
TXT_ENC_WPC1256 = '\x1b\x74\x32' # WPC1256 Arabic
TXT_ENC_WPC1257 = '\x1b\x74\x33' # WPC1257 Baltic Rim
TXT_ENC_WPC1258 = '\x1b\x74\x34' # WPC1258 Vietnamese
TXT_ENC_KZ1048  = '\x1b\x74\x35' # KZ-1048 Kazakhstan

TXT_ENC_KATAKANA_MAP = {
  # Maps UTF-8 Katakana symbols to KATAKANA Page Codes

  # Half-Width Katakanas
  '\xef\xbd\xa1':'\xa1',  # ｡
  '\xef\xbd\xa2':'\xa2',  # ｢
  '\xef\xbd\xa3':'\xa3',  # ｣
  '\xef\xbd\xa4':'\xa4',  # ､
  '\xef\xbd\xa5':'\xa5',  # ･

  '\xef\xbd\xa6':'\xa6',  # ｦ
  '\xef\xbd\xa7':'\xa7',  # ｧ
  '\xef\xbd\xa8':'\xa8',  # ｨ
  '\xef\xbd\xa9':'\xa9',  # ｩ
  '\xef\xbd\xaa':'\xaa',  # ｪ
  '\xef\xbd\xab':'\xab',  # ｫ
  '\xef\xbd\xac':'\xac',  # ｬ
  '\xef\xbd\xad':'\xad',  # ｭ
  '\xef\xbd\xae':'\xae',  # ｮ
  '\xef\xbd\xaf':'\xaf',  # ｯ
  '\xef\xbd\xb0':'\xb0',  # ｰ
  '\xef\xbd\xb1':'\xb1',  # ｱ
  '\xef\xbd\xb2':'\xb2',  # ｲ
  '\xef\xbd\xb3':'\xb3',  # ｳ
  '\xef\xbd\xb4':'\xb4',  # ｴ
  '\xef\xbd\xb5':'\xb5',  # ｵ
  '\xef\xbd\xb6':'\xb6',  # ｶ
  '\xef\xbd\xb7':'\xb7',  # ｷ
  '\xef\xbd\xb8':'\xb8',  # ｸ
  '\xef\xbd\xb9':'\xb9',  # ｹ
  '\xef\xbd\xba':'\xba',  # ｺ
  '\xef\xbd\xbb':'\xbb',  # ｻ
  '\xef\xbd\xbc':'\xbc',  # ｼ
  '\xef\xbd\xbd':'\xbd',  # ｽ
  '\xef\xbd\xbe':'\xbe',  # ｾ
  '\xef\xbd\xbf':'\xbf',  # ｿ
  '\xef\xbe\x80':'\xc0',  # ﾀ
  '\xef\xbe\x81':'\xc1',  # ﾁ
  '\xef\xbe\x82':'\xc2',  # ﾂ
  '\xef\xbe\x83':'\xc3',  # ﾃ
  '\xef\xbe\x84':'\xc4',  # ﾄ
  '\xef\xbe\x85':'\xc5',  # ﾅ
  '\xef\xbe\x86':'\xc6',  # ﾆ
  '\xef\xbe\x87':'\xc7',  # ﾇ
  '\xef\xbe\x88':'\xc8',  # ﾈ
  '\xef\xbe\x89':'\xc9',  # ﾉ
  '\xef\xbe\x8a':'\xca',  # ﾊ
  '\xef\xbe\x8b':'\xcb',  # ﾋ
  '\xef\xbe\x8c':'\xcc',  # ﾌ
  '\xef\xbe\x8d':'\xcd',  # ﾍ
  '\xef\xbe\x8e':'\xce',  # ﾎ
  '\xef\xbe\x8f':'\xcf',  # ﾏ
  '\xef\xbe\x90':'\xd0',  # ﾐ
  '\xef\xbe\x91':'\xd1',  # ﾑ
  '\xef\xbe\x92':'\xd2',  # ﾒ
  '\xef\xbe\x93':'\xd3',  # ﾓ
  '\xef\xbe\x94':'\xd4',  # ﾔ
  '\xef\xbe\x95':'\xd5',  # ﾕ
  '\xef\xbe\x96':'\xd6',  # ﾖ
  '\xef\xbe\x97':'\xd7',  # ﾗ
  '\xef\xbe\x98':'\xd8',  # ﾘ
  '\xef\xbe\x99':'\xd9',  # ﾙ
  '\xef\xbe\x9a':'\xda',  # ﾚ
  '\xef\xbe\x9b':'\xdb',  # ﾛ
  '\xef\xbe\x9c':'\xdc',  # ﾜ
  '\xef\xbe\x9d':'\xdd',  # ﾝ

  '\xef\xbe\x9e':'\xde',  # ﾞ
  '\xef\xbe\x9f':'\xdf',  # ﾟ
}

# Barcod format
BARCODE_TXT_OFF = '\x1d\x48\x00' # HRI barcode chars OFF
BARCODE_TXT_ABV = '\x1d\x48\x01' # HRI barcode chars above
BARCODE_TXT_BLW = '\x1d\x48\x02' # HRI barcode chars below
BARCODE_TXT_BTH = '\x1d\x48\x03' # HRI barcode chars both above and below
BARCODE_FONT_A  = '\x1d\x66\x00' # Font type A for HRI barcode chars
BARCODE_FONT_B  = '\x1d\x66\x01' # Font type B for HRI barcode chars
BARCODE_HEIGHT  = '\x1d\x68\x64' # Barcode Height [1-255]
BARCODE_WIDTH   = '\x1d\x77\x03' # Barcode Width  [2-6]
BARCODE_UPC_A   = '\x1d\x6b\x00' # Barcode type UPC-A
BARCODE_UPC_E   = '\x1d\x6b\x01' # Barcode type UPC-E
BARCODE_EAN13   = '\x1d\x6b\x02' # Barcode type EAN13
BARCODE_EAN8    = '\x1d\x6b\x03' # Barcode type EAN8
BARCODE_CODE39  = '\x1d\x6b\x04' # Barcode type CODE39
BARCODE_ITF     = '\x1d\x6b\x05' # Barcode type ITF
BARCODE_NW7     = '\x1d\x6b\x06' # Barcode type NW7
# Image format  
S_RASTER_N      = '\x1d\x76\x30\x00' # Set raster image normal size
S_RASTER_2W     = '\x1d\x76\x30\x01' # Set raster image double width
S_RASTER_2H     = '\x1d\x76\x30\x02' # Set raster image double height
S_RASTER_Q      = '\x1d\x76\x30\x03' # Set raster image quadruple

```

## File: escpos\escpos.py

```python
# -*- coding: utf-8 -*-

from __future__ import print_function
import base64
import copy
import io
import math
import re
import traceback
import codecs
from hashlib import md5

from PIL import Image
from xml.etree import ElementTree as ET


try:
    import jcconv
except ImportError:
    jcconv = None

try: 
    import qrcode
except ImportError:
    qrcode = None

from .constants import *
from .exceptions import *

def utfstr(stuff):
    """ converts stuff to string and does without failing if stuff is a utf8 string """
    if isinstance(stuff, str):
        return stuff
    else:
        return str(stuff)

class StyleStack:
    """ 
    The stylestack is used by the xml receipt serializer to compute the active styles along the xml
    document. Styles are just xml attributes, there is no css mechanism. But the style applied by
    the attributes are inherited by deeper nodes.
    """
    def __init__(self):
        self.stack = []
        self.defaults = {   # default style values
            'align':     'left',
            'underline': 'off',
            'bold':      'off',
            'size':      'normal',
            'font'  :    'a',
            'width':     48,
            'indent':    0,
            'tabwidth':  2,
            'bullet':    ' - ',
            'line-ratio':0.5,
            'color':    'black',

            'value-decimals':           2,
            'value-symbol':             '',
            'value-symbol-position':    'after',
            'value-autoint':            'off',
            'value-decimals-separator':  '.',
            'value-thousands-separator': ',',
            'value-width':               0,
            
        }

        self.types = { # attribute types, default is string and can be ommitted
            'width':    'int',
            'indent':   'int',
            'tabwidth': 'int',
            'line-ratio':       'float',
            'value-decimals':   'int',
            'value-width':      'int',
        }

        self.cmds = { 
            # translation from styles to escpos commands
            # some style do not correspond to escpos command are used by
            # the serializer instead
            'align': {
                'left':     TXT_ALIGN_LT,
                'right':    TXT_ALIGN_RT,
                'center':   TXT_ALIGN_CT,
                '_order':   1,
            },
            'underline': {
                'off':      TXT_UNDERL_OFF,
                'on':       TXT_UNDERL_ON,
                'double':   TXT_UNDERL2_ON,
                # must be issued after 'size' command
                # because ESC ! resets ESC -
                '_order':   10,
            },
            'bold': {
                'off':      TXT_BOLD_OFF,
                'on':       TXT_BOLD_ON,
                # must be issued after 'size' command
                # because ESC ! resets ESC -
                '_order':   10,
            },
            'font': {
                'a':        TXT_FONT_A,
                'b':        TXT_FONT_B,
                # must be issued after 'size' command
                # because ESC ! resets ESC -
                '_order':   10,
            },
            'size': {
                'normal':           TXT_NORMAL,
                'double-height':    TXT_2HEIGHT,
                'double-width':     TXT_2WIDTH,
                'double':           TXT_DOUBLE,
                '_order':   1,
            },
            'color': {
                'black':    TXT_COLOR_BLACK,
                'red':      TXT_COLOR_RED,
                '_order':   1,
            },
        }

        self.push(self.defaults) 

    def get(self,style):
        """ what's the value of a style at the current stack level"""
        level = len(self.stack) -1
        while level >= 0:
            if style in self.stack[level]:
                return self.stack[level][style]
            else:
                level = level - 1
        return None

    def enforce_type(self, attr, val):
        """converts a value to the attribute's type"""
        if not attr in self.types:
            return utfstr(val)
        elif self.types[attr] == 'int':
            return int(float(val))
        elif self.types[attr] == 'float':
            return float(val)
        else:
            return utfstr(val)

    def push(self, style={}):
        """push a new level on the stack with a style dictionnary containing style:value pairs"""
        _style = {}
        for attr in style:
            if attr in self.cmds and not style[attr] in self.cmds[attr]:
                print('WARNING: ESC/POS PRINTING: ignoring invalid value: %s for style %s' % (style[attr], utfstr(attr)))
            else:
                _style[attr] = self.enforce_type(attr, style[attr])
        self.stack.append(_style)

    def set(self, style={}):
        """overrides style values at the current stack level"""
        _style = {}
        for attr in style:
            if attr in self.cmds and not style[attr] in self.cmds[attr]:
                print('WARNING: ESC/POS PRINTING: ignoring invalid value: %s for style %s' % (style[attr], attr))
            else:
                self.stack[-1][attr] = self.enforce_type(attr, style[attr])

    def pop(self):
        """ pop a style stack level """
        if len(self.stack) > 1 :
            self.stack = self.stack[:-1]

    def to_escpos(self):
        """ converts the current style to an escpos command string """
        cmd = ''
        ordered_cmds = sorted(self.cmds, key=lambda x: self.cmds[x]['_order'])
        for style in ordered_cmds:
            cmd += self.cmds[style][self.get(style)]
        return cmd

class XmlSerializer:
    """ 
    Converts the xml inline / block tree structure to a string,
    keeping track of newlines and spacings.
    The string is outputted asap to the provided escpos driver.
    """
    def __init__(self,escpos):
        self.escpos = escpos
        self.stack = ['block']
        self.dirty = False

    def start_inline(self,stylestack=None):
        """ starts an inline entity with an optional style definition """
        self.stack.append('inline')
        if self.dirty:
            self.escpos._raw(' ')
        if stylestack:
            self.style(stylestack)

    def start_block(self,stylestack=None):
        """ starts a block entity with an optional style definition """
        if self.dirty:
            self.escpos._raw('\n')
            self.dirty = False
        self.stack.append('block')
        if stylestack:
            self.style(stylestack)

    def end_entity(self):
        """ ends the entity definition. (but does not cancel the active style!) """
        if self.stack[-1] == 'block' and self.dirty:
            self.escpos._raw('\n')
            self.dirty = False
        if len(self.stack) > 1:
            self.stack = self.stack[:-1]

    def pre(self,text):
        """ puts a string of text in the entity keeping the whitespace intact """
        if text:
            self.escpos.text(text)
            self.dirty = True

    def text(self,text):
        """ puts text in the entity. Whitespace and newlines are stripped to single spaces. """
        if text:
            text = utfstr(text)
            text = text.strip()
            text = re.sub('\s+',' ',text)
            if text:
                self.dirty = True
                self.escpos.text(text)

    def linebreak(self):
        """ inserts a linebreak in the entity """
        self.dirty = False
        self.escpos._raw('\n')

    def style(self,stylestack):
        """ apply a style to the entity (only applies to content added after the definition) """
        self.raw(stylestack.to_escpos())

    def raw(self,raw):
        """ puts raw text or escpos command in the entity without affecting the state of the serializer """
        self.escpos._raw(raw)

class XmlLineSerializer:
    """ 
    This is used to convert a xml tree into a single line, with a left and a right part.
    The content is not output to escpos directly, and is intended to be fedback to the
    XmlSerializer as the content of a block entity.
    """
    def __init__(self, indent=0, tabwidth=2, width=48, ratio=0.5):
        self.tabwidth = tabwidth
        self.indent = indent
        self.width  = max(0, width - int(tabwidth*indent))
        self.lwidth = int(self.width*ratio)
        self.rwidth = max(0, self.width - self.lwidth)
        self.clwidth = 0
        self.crwidth = 0
        self.lbuffer  = ''
        self.rbuffer  = ''
        self.left    = True

    def _txt(self,txt):
        if self.left:
            if self.clwidth < self.lwidth:
                txt = txt[:max(0, self.lwidth - self.clwidth)]
                self.lbuffer += txt
                self.clwidth += len(txt)
        else:
            if self.crwidth < self.rwidth:
                txt = txt[:max(0, self.rwidth - self.crwidth)]
                self.rbuffer += txt
                self.crwidth  += len(txt)

    def start_inline(self,stylestack=None):
        if (self.left and self.clwidth) or (not self.left and self.crwidth):
            self._txt(' ')

    def start_block(self,stylestack=None):
        self.start_inline(stylestack)

    def end_entity(self):
        pass

    def pre(self,text):
        if text:
            self._txt(text)
    def text(self,text):
        if text:
            text = utfstr(text)
            text = text.strip()
            text = re.sub('\s+',' ',text)
            if text:
                self._txt(text)

    def linebreak(self):
        pass
    def style(self,stylestack):
        pass
    def raw(self,raw):
        pass

    def start_right(self):
        self.left = False

    def get_line(self):
        return ' ' * self.indent * self.tabwidth + self.lbuffer + ' ' * (self.width - self.clwidth - self.crwidth) + self.rbuffer
    

class Escpos:
    """ ESC/POS Printer object """
    device    = None
    encoding  = None
    img_cache = {}

    def _check_image_size(self, size):
        """ Check and fix the size of the image to 32 bits """
        if size % 32 == 0:
            return (0, 0)
        else:
            image_border = 32 - (size % 32)
            if (image_border % 2) == 0:
                return (int(image_border / 2), int(image_border / 2))
            else:
                return (int(image_border / 2), int((image_border / 2) + 1))

    def _print_image(self, line, size):
        """ Print formatted image """
        i = 0
        cont = 0
        buffer = ""

       
        self._raw(S_RASTER_N)
        buffer = b"%02X%02X%02X%02X" % (int((size[0]/size[1])/8), 0, size[1], 0)
        self._raw(codecs.decode(buffer, 'hex'))
        buffer = ""

        while i < len(line):
            hex_string = int(line[i:i+8],2)
            buffer += "%02X" % hex_string
            i += 8
            cont += 1
            if cont % 4 == 0:
                self._raw(codecs.decode(buffer, "hex"))
                buffer = ""
                cont = 0

    def _raw_print_image(self, line, size, output=None ):
        """ Print formatted image """
        i = 0
        cont = 0
        buffer = ""
        raw = b""

        def __raw(string):
            if output:
                output(string)
            else:
                self._raw(string)
       
        raw += S_RASTER_N.encode('utf-8')
        buffer = "%02X%02X%02X%02X" % (int((size[0]/size[1])/8), 0, size[1], 0)
        raw += codecs.decode(buffer, 'hex')
        buffer = ""

        while i < len(line):
            hex_string = int(line[i:i+8],2)
            buffer += "%02X" % hex_string
            i += 8
            cont += 1
            if cont % 4 == 0:
                raw += codecs.decode(buffer, 'hex')
                buffer = ""
                cont = 0

        return raw

    def _convert_image(self, im):
        """ Parse image and prepare it to a printable format """
        pixels   = []
        pix_line = ""
        im_left  = ""
        im_right = ""
        switch   = 0
        img_size = [ 0, 0 ]


        if im.size[0] > 512:
            print("WARNING: Image is wider than 512 and could be truncated at print time ")
        if im.size[1] > 255:
            raise ImageSizeError()

        im_border = self._check_image_size(im.size[0])
        for i in range(im_border[0]):
            im_left += "0"
        for i in range(im_border[1]):
            im_right += "0"

        for y in range(im.size[1]):
            img_size[1] += 1
            pix_line += im_left
            img_size[0] += im_border[0]
            for x in range(im.size[0]):
                img_size[0] += 1
                RGB = im.getpixel((x, y))
                im_color = (RGB[0] + RGB[1] + RGB[2])
                im_pattern = "1X0"
                pattern_len = len(im_pattern)
                switch = (switch - 1 ) * (-1)
                for x in range(pattern_len):
                    if im_color <= (255 * 3 / pattern_len * (x+1)):
                        if im_pattern[x] == "X":
                            pix_line += "%d" % switch
                        else:
                            pix_line += im_pattern[x]
                        break
                    elif im_color > (255 * 3 / pattern_len * pattern_len) and im_color <= (255 * 3):
                        pix_line += im_pattern[-1]
                        break 
            pix_line += im_right
            img_size[0] += im_border[1]

        return (pix_line, img_size)

    def image(self,path_img):
        """ Open image file """
        im_open = Image.open(path_img)
        im = im_open.convert("RGB")
        # Convert the RGB image in printable image
        pix_line, img_size = self._convert_image(im)
        self._print_image(pix_line, img_size)

    def print_base64_image(self,img):

        print('print_b64_img')

        id = md5(img).digest()

        if id not in self.img_cache:
            print('not in cache')

            img = img[img.find(b',')+1:]
            f = io.BytesIO(b'img')
            f.write(base64.decodebytes(img))
            f.seek(0)
            img_rgba = Image.open(f)
            img = Image.new('RGB', img_rgba.size, (255,255,255))
            channels = img_rgba.split()
            if len(channels) > 3:
                # use alpha channel as mask
                img.paste(img_rgba, mask=channels[3])
            else:
                img.paste(img_rgba)

            print('convert image')
        
            pix_line, img_size = self._convert_image(img)

            print('print image')

            buffer = self._raw_print_image(pix_line, img_size)
            self.img_cache[id] = buffer

        print('raw image')

        self._raw(self.img_cache[id])

    def qr(self,text):
        """ Print QR Code for the provided string """
        qr_code = qrcode.QRCode(version=4, box_size=4, border=1)
        qr_code.add_data(text)
        qr_code.make(fit=True)
        qr_img = qr_code.make_image()
        im = qr_img._img.convert("RGB")
        # Convert the RGB image in printable image
        self._convert_image(im)

    def barcode(self, code, bc, width=255, height=2, pos='below', font='a'):
        """ Print Barcode """
        # Align Bar Code()
        self._raw(TXT_ALIGN_CT)
        # Height
        if height >=2 or height <=6:
            self._raw(BARCODE_HEIGHT)
        else:
            raise BarcodeSizeError()
        # Width
        if width >= 1 or width <=255:
            self._raw(BARCODE_WIDTH)
        else:
            raise BarcodeSizeError()
        # Font
        if font.upper() == "B":
            self._raw(BARCODE_FONT_B)
        else: # DEFAULT FONT: A
            self._raw(BARCODE_FONT_A)
        # Position
        if pos.upper() == "OFF":
            self._raw(BARCODE_TXT_OFF)
        elif pos.upper() == "BOTH":
            self._raw(BARCODE_TXT_BTH)
        elif pos.upper() == "ABOVE":
            self._raw(BARCODE_TXT_ABV)
        else:  # DEFAULT POSITION: BELOW 
            self._raw(BARCODE_TXT_BLW)
        # Type 
        if bc.upper() == "UPC-A":
            self._raw(BARCODE_UPC_A)
        elif bc.upper() == "UPC-E":
            self._raw(BARCODE_UPC_E)
        elif bc.upper() == "EAN13":
            self._raw(BARCODE_EAN13)
        elif bc.upper() == "EAN8":
            self._raw(BARCODE_EAN8)
        elif bc.upper() == "CODE39":
            self._raw(BARCODE_CODE39)
        elif bc.upper() == "ITF":
            self._raw(BARCODE_ITF)
        elif bc.upper() == "NW7":
            self._raw(BARCODE_NW7)
        else:
            raise BarcodeTypeError()
        # Print Code
        if code:
            self._raw(code)
            # We are using type A commands
            # So we need to add the 'NULL' character
            # https://github.com/python-escpos/python-escpos/pull/98/files#diff-a0b1df12c7c67e38915adbe469051e2dR444
            self._raw('\x00')
        else:
            raise BarcodeCodeError()

    def receipt(self,xml):
        """
        Prints an xml based receipt definition
        """

        def strclean(string):
            if not string:
                string = ''
            string = string.strip()
            string = re.sub('\s+',' ',string)
            return string

        def format_value(value, decimals=3, width=0, decimals_separator='.', thousands_separator=',', autoint=False, symbol='', position='after'):
            decimals = max(0,int(decimals))
            width    = max(0,int(width))
            value    = float(value)

            if autoint and math.floor(value) == value:
                decimals = 0
            if width == 0:
                width = ''

            if thousands_separator:
                formatstr = "{:"+str(width)+",."+str(decimals)+"f}"
            else:
                formatstr = "{:"+str(width)+"."+str(decimals)+"f}"


            ret = formatstr.format(value)
            ret = ret.replace(',','COMMA')
            ret = ret.replace('.','DOT')
            ret = ret.replace('COMMA',thousands_separator)
            ret = ret.replace('DOT',decimals_separator)

            if symbol:
                if position == 'after':
                    ret = ret + symbol
                else:
                    ret = symbol + ret
            return ret

        def print_elem(stylestack, serializer, elem, indent=0):

            elem_styles = {
                'h1': {'bold': 'on', 'size':'double'},
                'h2': {'size':'double'},
                'h3': {'bold': 'on', 'size':'double-height'},
                'h4': {'size': 'double-height'},
                'h5': {'bold': 'on'},
                'em': {'font': 'b'},
                'b':  {'bold': 'on'},
            }

            stylestack.push()
            if elem.tag in elem_styles:
                stylestack.set(elem_styles[elem.tag])
            stylestack.set(elem.attrib)

            if elem.tag in ('p','div','section','article','receipt','header','footer','li','h1','h2','h3','h4','h5'):
                serializer.start_block(stylestack)
                serializer.text(elem.text)
                for child in elem:
                    print_elem(stylestack,serializer,child)
                    serializer.start_inline(stylestack)
                    serializer.text(child.tail)
                    serializer.end_entity()
                serializer.end_entity()

            elif elem.tag in ('span','em','b','left','right'):
                serializer.start_inline(stylestack)
                serializer.text(elem.text)
                for child in elem:
                    print_elem(stylestack,serializer,child)
                    serializer.start_inline(stylestack)
                    serializer.text(child.tail)
                    serializer.end_entity()
                serializer.end_entity()

            elif elem.tag == 'value':
                serializer.start_inline(stylestack)
                serializer.pre(format_value( 
                                              elem.text,
                                              decimals=stylestack.get('value-decimals'),
                                              width=stylestack.get('value-width'),
                                              decimals_separator=stylestack.get('value-decimals-separator'),
                                              thousands_separator=stylestack.get('value-thousands-separator'),
                                              autoint=(stylestack.get('value-autoint') == 'on'),
                                              symbol=stylestack.get('value-symbol'),
                                              position=stylestack.get('value-symbol-position') 
                                            ))
                serializer.end_entity()

            elif elem.tag == 'line':
                width = stylestack.get('width')
                if stylestack.get('size') in ('double', 'double-width'):
                    width = width / 2

                lineserializer = XmlLineSerializer(stylestack.get('indent')+indent,stylestack.get('tabwidth'),width,stylestack.get('line-ratio'))
                serializer.start_block(stylestack)
                for child in elem:
                    if child.tag == 'left':
                        print_elem(stylestack,lineserializer,child,indent=indent)
                    elif child.tag == 'right':
                        lineserializer.start_right()
                        print_elem(stylestack,lineserializer,child,indent=indent)
                serializer.pre(lineserializer.get_line())
                serializer.end_entity()

            elif elem.tag == 'ul':
                serializer.start_block(stylestack)
                bullet = stylestack.get('bullet')
                for child in elem:
                    if child.tag == 'li':
                        serializer.style(stylestack)
                        serializer.raw(' ' * indent * stylestack.get('tabwidth') + bullet)
                    print_elem(stylestack,serializer,child,indent=indent+1)
                serializer.end_entity()

            elif elem.tag == 'ol':
                cwidth = len(str(len(elem))) + 2
                i = 1
                serializer.start_block(stylestack)
                for child in elem:
                    if child.tag == 'li':
                        serializer.style(stylestack)
                        serializer.raw(' ' * indent * stylestack.get('tabwidth') + ' ' + (str(i)+')').ljust(cwidth))
                        i = i + 1
                    print_elem(stylestack,serializer,child,indent=indent+1)
                serializer.end_entity()

            elif elem.tag == 'pre':
                serializer.start_block(stylestack)
                serializer.pre(elem.text)
                serializer.end_entity()

            elif elem.tag == 'hr':
                width = stylestack.get('width')
                if stylestack.get('size') in ('double', 'double-width'):
                    width = width / 2
                serializer.start_block(stylestack)
                serializer.text('-'*width)
                serializer.end_entity()

            elif elem.tag == 'br':
                serializer.linebreak()

            elif elem.tag == 'img':
                if 'src' in elem.attrib and 'data:' in elem.attrib['src']:
                    self.print_base64_image(bytes(elem.attrib['src'], 'utf-8'))

            elif elem.tag == 'barcode' and 'encoding' in elem.attrib:
                serializer.start_block(stylestack)
                self.barcode(strclean(elem.text),elem.attrib['encoding'])
                serializer.end_entity()

            elif elem.tag == 'cut':
                self.cut()
            elif elem.tag == 'partialcut':
                self.cut(mode='part')
            elif elem.tag == 'cashdraw':
                self.cashdraw(2)
                self.cashdraw(5)

            stylestack.pop()

        try:
            stylestack      = StyleStack() 
            serializer      = XmlSerializer(self)
            root            = ET.fromstring(xml.encode('utf-8'))

            self._raw(stylestack.to_escpos())

            print_elem(stylestack,serializer,root)

            if 'open-cashdrawer' in root.attrib and root.attrib['open-cashdrawer'] == 'true':
                self.cashdraw(2)
                self.cashdraw(5)
            if not 'cut' in root.attrib or root.attrib['cut'] == 'true' :
                self.cut()

        except Exception as e:
            errmsg = str(e)+'\n'+'-'*48+'\n'+traceback.format_exc() + '-'*48+'\n'
            self.text(errmsg)
            self.cut()

            raise e

    def text(self,txt):
        """ Print Utf8 encoded alpha-numeric text """
        if not txt:
            return
        try:
            txt = txt.decode('utf-8')
        except:
            try:
                txt = txt.decode('utf-16')
            except:
                pass

        self.extra_chars = 0
        
        def encode_char(char):  
            """ 
            Encodes a single utf-8 character into a sequence of 
            esc-pos code page change instructions and character declarations 
            """ 
            char_utf8 = char.encode('utf-8')
            encoded  = ''
            encoding = self.encoding # we reuse the last encoding to prevent code page switches at every character
            encodings = {
                    # TODO use ordering to prevent useless switches
                    # TODO Support other encodings not natively supported by python ( Thai, Khazakh, Kanjis )
                    'cp437': TXT_ENC_PC437,
                    'cp850': TXT_ENC_PC850,
                    'cp852': TXT_ENC_PC852,
                    'cp857': TXT_ENC_PC857,
                    'cp858': TXT_ENC_PC858,
                    'cp860': TXT_ENC_PC860,
                    'cp863': TXT_ENC_PC863,
                    'cp865': TXT_ENC_PC865,
                    'cp1251': TXT_ENC_WPC1251,    # win-1251 covers more cyrillic symbols than cp866
                    'cp866': TXT_ENC_PC866,
                    'cp862': TXT_ENC_PC862,
                    'cp720': TXT_ENC_PC720,
                    'cp936': TXT_ENC_PC936,
                    'iso8859_2': TXT_ENC_8859_2,
                    'iso8859_7': TXT_ENC_8859_7,
                    'iso8859_9': TXT_ENC_8859_9,
                    'cp1254'   : TXT_ENC_WPC1254,
                    'cp1255'   : TXT_ENC_WPC1255,
                    'cp1256'   : TXT_ENC_WPC1256,
                    'cp1257'   : TXT_ENC_WPC1257,
                    'cp1258'   : TXT_ENC_WPC1258,
                    'katakana' : TXT_ENC_KATAKANA,
            }
            remaining = copy.copy(encodings)

            if not encoding :
                encoding = 'cp437'

            while True: # Trying all encoding until one succeeds
                try:
                    if encoding == 'katakana': # Japanese characters
                        if jcconv:
                            # try to convert japanese text to a half-katakanas 
                            kata = jcconv.kata2half(jcconv.hira2kata(char_utf8))
                            if kata != char_utf8:
                                self.extra_chars += len(kata.decode('utf-8')) - 1
                                # the conversion may result in multiple characters
                                return encode_str(kata.decode('utf-8')) 
                        else:
                             kata = char_utf8
                        
                        if kata in TXT_ENC_KATAKANA_MAP:
                            encoded = TXT_ENC_KATAKANA_MAP[kata]
                            break
                        else: 
                            raise ValueError()
                    else:
                        # First 127 symbols are covered by cp437.
                        # Extended range is covered by different encodings.
                        encoded = char.encode(encoding)
                        if ord(encoded) <= 127:
                            encoding = 'cp437'
                        break

                except (UnicodeEncodeError, UnicodeWarning, TypeError, ValueError):
                    #the encoding failed, select another one and retry
                    if encoding in remaining:
                        del remaining[encoding]
                    if len(remaining) >= 1:
                        (encoding, _) = remaining.popitem()
                    else:
                        encoding = 'cp437'
                        encoded  = b'\xb1'    # could not encode, output error character
                        break;

            if encoding != self.encoding:
                # if the encoding changed, remember it and prefix the character with
                # the esc-pos encoding change sequence
                self.encoding = encoding
                encoded = bytes(encodings[encoding], 'utf-8') + encoded

            return encoded
        
        def encode_str(txt):
            buffer = b''
            for c in txt:
                buffer += encode_char(c)
            return buffer

        txt = encode_str(txt)

        # if the utf-8 -> codepage conversion inserted extra characters, 
        # remove double spaces to try to restore the original string length
        # and prevent printing alignment issues
        while self.extra_chars > 0: 
            dspace = txt.find('  ')
            if dspace > 0:
                txt = txt[:dspace] + txt[dspace+1:]
                self.extra_chars -= 1
            else:
                break

        self._raw(txt)
        
    def set(self, align='left', font='a', type='normal', width=1, height=1):
        """ Set text properties """
        # Align
        if align.upper() == "CENTER":
            self._raw(TXT_ALIGN_CT)
        elif align.upper() == "RIGHT":
            self._raw(TXT_ALIGN_RT)
        elif align.upper() == "LEFT":
            self._raw(TXT_ALIGN_LT)
        # Font
        if font.upper() == "B":
            self._raw(TXT_FONT_B)
        else:  # DEFAULT FONT: A
            self._raw(TXT_FONT_A)
        # Type
        if type.upper() == "B":
            self._raw(TXT_BOLD_ON)
            self._raw(TXT_UNDERL_OFF)
        elif type.upper() == "U":
            self._raw(TXT_BOLD_OFF)
            self._raw(TXT_UNDERL_ON)
        elif type.upper() == "U2":
            self._raw(TXT_BOLD_OFF)
            self._raw(TXT_UNDERL2_ON)
        elif type.upper() == "BU":
            self._raw(TXT_BOLD_ON)
            self._raw(TXT_UNDERL_ON)
        elif type.upper() == "BU2":
            self._raw(TXT_BOLD_ON)
            self._raw(TXT_UNDERL2_ON)
        elif type.upper == "NORMAL":
            self._raw(TXT_BOLD_OFF)
            self._raw(TXT_UNDERL_OFF)
        # Width
        if width == 2 and height != 2:
            self._raw(TXT_NORMAL)
            self._raw(TXT_2WIDTH)
        elif height == 2 and width != 2:
            self._raw(TXT_NORMAL)
            self._raw(TXT_2HEIGHT)
        elif height == 2 and width == 2:
            self._raw(TXT_2WIDTH)
            self._raw(TXT_2HEIGHT)
        else: # DEFAULT SIZE: NORMAL
            self._raw(TXT_NORMAL)


    def cut(self, mode=''):
        """ Cut paper """
        # Fix the size between last line and cut
        # TODO: handle this with a line feed
        self._raw("\n\n\n\n\n\n")
        if mode.upper() == "PART":
            self._raw(PAPER_PART_CUT)
        else: # DEFAULT MODE: FULL CUT
            self._raw(PAPER_FULL_CUT)


    def cashdraw(self, pin):
        """ Send pulse to kick the cash drawer

        For some reason, with some printers (ex: Epson TM-m30), the cash drawer
        only opens 50% of the time if you just send the pulse. But if you read
        the status afterwards, it opens all the time.
        """
        if pin == 2:
            self._raw(CD_KICK_2)
        elif pin == 5:
            self._raw(CD_KICK_5)
        else:
            raise CashDrawerError()

        self.get_printer_status()

    def hw(self, hw):
        """ Hardware operations """
        if hw.upper() == "INIT":
            self._raw(HW_INIT)
        elif hw.upper() == "SELECT":
            self._raw(HW_SELECT)
        elif hw.upper() == "RESET":
            self._raw(HW_RESET)
        else: # DEFAULT: DOES NOTHING
            pass


    def control(self, ctl):
        """ Feed control sequences """
        if ctl.upper() == "LF":
            self._raw(CTL_LF)
        elif ctl.upper() == "FF":
            self._raw(CTL_FF)
        elif ctl.upper() == "CR":
            self._raw(CTL_CR)
        elif ctl.upper() == "HT":
            self._raw(CTL_HT)
        elif ctl.upper() == "VT":
            self._raw(CTL_VT)

```

## File: escpos\exceptions.py

```python
""" ESC/POS Exceptions classes """


class Error(Exception):
    """ Base class for ESC/POS errors """
    def __init__(self, msg, status=None):
        Exception.__init__(self)
        self.msg = msg
        self.resultcode = 1
        if status is not None:
            self.resultcode = status

    def __str__(self):
        return self.msg

# Result/Exit codes
# 0  = success
# 10 = No Barcode type defined
# 20 = Barcode size values are out of range
# 30 = Barcode text not supplied
# 40 = Image height is too large
# 50 = No string supplied to be printed
# 60 = Invalid pin to send Cash Drawer pulse


class BarcodeTypeError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 10

    def __str__(self):
        return "No Barcode type is defined"

class BarcodeSizeError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 20

    def __str__(self):
        return "Barcode size is out of range"

class BarcodeCodeError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 30

    def __str__(self):
        return "Code was not supplied"

class ImageSizeError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 40

    def __str__(self):
        return "Image height is longer than 255px and can't be printed"

class TextError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 50

    def __str__(self):
        return "Text string must be supplied to the text() method"


class CashDrawerError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 60

    def __str__(self):
        return "Valid pin must be set to send pulse"

class NoStatusError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 70

    def __str__(self):
        return "Impossible to get status from the printer: " + str(self.msg)

class TicketNotPrinted(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 80

    def __str__(self):
        return "A part of the ticket was not been printed: " + str(self.msg)

class NoDeviceError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 90

    def __str__(self):
        return str(self.msg)

class HandleDeviceError(Error):
    def __init__(self, msg=""):
        Error.__init__(self, msg)
        self.msg = msg
        self.resultcode = 100

    def __str__(self):
        return str(self.msg)

```

## File: escpos\LICENSE.txt

```text
The MIT License (MIT)

Copyright (c) 2014 Frederic van der Essen & Manuel F. Martinez

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

```

## File: escpos\printer.py

```python
#!/usr/bin/python

from __future__ import print_function
import serial
import socket
import usb.core
import usb.util

from .escpos import *
from .constants import *
from .exceptions import *
from time import sleep

class Usb(Escpos):
    """ Define USB printer """

    def __init__(self, idVendor, idProduct, interface=0, in_ep=None, out_ep=None):
        """
        @param idVendor  : Vendor ID
        @param idProduct : Product ID
        @param interface : USB device interface
        @param in_ep     : Input end point
        @param out_ep    : Output end point
        """

        self.errorText = "ERROR PRINTER\n\n\n\n\n\n"+PAPER_FULL_CUT

        self.idVendor  = idVendor
        self.idProduct = idProduct
        self.interface = interface
        self.in_ep     = in_ep
        self.out_ep    = out_ep

        # pyusb dropped the 'interface' parameter from usb.Device.write() at 1.0.0b2
        # https://github.com/pyusb/pyusb/commit/20cd8c1f79b24082ec999c022b56c3febedc0964#diff-b5a4f98a864952f0f55d569dd14695b7L293
        if usb.version_info < (1, 0, 0) or (usb.version_info == (1, 0, 0) and usb.version_info[3] in ("a1", "a2", "a3", "b1")):
            self.write_kwargs = dict(interface=self.interface)
        else:
            self.write_kwargs = {}

        self.open()

    def open(self):
        """ Search device on USB tree and set is as escpos device """
        
        self.device = usb.core.find(idVendor=self.idVendor, idProduct=self.idProduct)
        if self.device is None:
            raise NoDeviceError()
        try:
            if self.device.is_kernel_driver_active(self.interface):
                self.device.detach_kernel_driver(self.interface) 
            self.device.set_configuration()
            usb.util.claim_interface(self.device, self.interface)

            cfg = self.device.get_active_configuration()
            intf = cfg[(0,0)] # first interface
            if self.in_ep is None:
                # Attempt to detect IN/OUT endpoint addresses
                try:
                    is_IN = lambda e: usb.util.endpoint_direction(e.bEndpointAddress) == usb.util.ENDPOINT_IN
                    is_OUT = lambda e: usb.util.endpoint_direction(e.bEndpointAddress) == usb.util.ENDPOINT_OUT
                    endpoint_in = usb.util.find_descriptor(intf, custom_match=is_IN)
                    endpoint_out = usb.util.find_descriptor(intf, custom_match=is_OUT)
                    self.in_ep = endpoint_in.bEndpointAddress
                    self.out_ep = endpoint_out.bEndpointAddress
                except usb.core.USBError:
                    # default values for officially supported printers
                    self.in_ep = 0x82
                    self.out_ep = 0x01

        except usb.core.USBError as e:
            raise HandleDeviceError(e)

    def close(self):
        i = 0
        while True:
            try:
                if not self.device.is_kernel_driver_active(self.interface):
                    usb.util.release_interface(self.device, self.interface)
                    self.device.attach_kernel_driver(self.interface)
                    usb.util.dispose_resources(self.device)
                else:
                    self.device = None
                    return True
            except usb.core.USBError as e:
                i += 1
                if i > 10:
                    return False
        
            sleep(0.1)

    def _raw(self, msg):
        """ Print any command sent in raw format """
        if len(msg) != self.device.write(self.out_ep, msg, timeout=5000, **self.write_kwargs):
            self.device.write(self.out_ep, self.errorText, **self.write_kwargs)
            raise TicketNotPrinted()
    
    def __extract_status(self):
        maxiterate = 0
        rep = None
        while rep == None:
            maxiterate += 1
            if maxiterate > 10000:
                raise NoStatusError()
            r = self.device.read(self.in_ep, 20, self.interface).tolist()
            while len(r):
                rep = r.pop()
        return rep

    def get_printer_status(self):
        status = {
            'printer': {}, 
            'offline': {}, 
            'error'  : {}, 
            'paper'  : {},
        }

        self.device.write(self.out_ep, DLE_EOT_PRINTER, **self.write_kwargs)
        printer = self.__extract_status()    
        self.device.write(self.out_ep, DLE_EOT_OFFLINE, **self.write_kwargs)
        offline = self.__extract_status()
        self.device.write(self.out_ep, DLE_EOT_ERROR, **self.write_kwargs)
        error = self.__extract_status()
        self.device.write(self.out_ep, DLE_EOT_PAPER, **self.write_kwargs)
        paper = self.__extract_status()
            
        status['printer']['status_code']     = printer
        status['printer']['status_error']    = not ((printer & 147) == 18)
        status['printer']['online']          = not bool(printer & 8)
        status['printer']['recovery']        = bool(printer & 32)
        status['printer']['paper_feed_on']   = bool(printer & 64)
        status['printer']['drawer_pin_high'] = bool(printer & 4)
        status['offline']['status_code']     = offline
        status['offline']['status_error']    = not ((offline & 147) == 18)
        status['offline']['cover_open']      = bool(offline & 4)
        status['offline']['paper_feed_on']   = bool(offline & 8)
        status['offline']['paper']           = not bool(offline & 32)
        status['offline']['error']           = bool(offline & 64)
        status['error']['status_code']       = error
        status['error']['status_error']      = not ((error & 147) == 18)
        status['error']['recoverable']       = bool(error & 4)
        status['error']['autocutter']        = bool(error & 8)
        status['error']['unrecoverable']     = bool(error & 32)
        status['error']['auto_recoverable']  = not bool(error & 64)
        status['paper']['status_code']       = paper
        status['paper']['status_error']      = not ((paper & 147) == 18)
        status['paper']['near_end']          = bool(paper & 12)
        status['paper']['present']           = not bool(paper & 96)

        return status

    def __del__(self):
        """ Release USB interface """
        if self.device:
            self.close()
        self.device = None



class Serial(Escpos):
    """ Define Serial printer """

    def __init__(self, devfile="/dev/ttyS0", baudrate=9600, bytesize=8, timeout=1):
        """
        @param devfile  : Device file under dev filesystem
        @param baudrate : Baud rate for serial transmission
        @param bytesize : Serial buffer size
        @param timeout  : Read/Write timeout
        """
        self.devfile  = devfile
        self.baudrate = baudrate
        self.bytesize = bytesize
        self.timeout  = timeout
        self.open()


    def open(self):
        """ Setup serial port and set is as escpos device """
        self.device = serial.Serial(port=self.devfile, baudrate=self.baudrate, bytesize=self.bytesize, parity=serial.PARITY_NONE, stopbits=serial.STOPBITS_ONE, timeout=self.timeout, dsrdtr=True)

        if self.device is not None:
            print("Serial printer enabled")
        else:
            print("Unable to open serial printer on: %s" % self.devfile)


    def _raw(self, msg):
        """ Print any command sent in raw format """
        self.device.write(msg)


    def __del__(self):
        """ Close Serial interface """
        if self.device is not None:
            self.device.close()



class Network(Escpos):
    """ Define Network printer """

    def __init__(self,host,port=9100):
        """
        @param host : Printer's hostname or IP address
        @param port : Port to write to
        """
        self.host = host
        self.port = port
        self.open()


    def open(self):
        """ Open TCP socket and set it as escpos device """
        self.device = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.device.connect((self.host, self.port))

        if self.device is None:
            print("Could not open socket for %s" % self.host)


    def _raw(self, msg):
        self.device.send(msg)


    def __del__(self):
        """ Close TCP connection """
        self.device.close()


```

## File: escpos\__init__.py

```python
__all__ = ["constants","escpos","exceptions","printer"]

```

