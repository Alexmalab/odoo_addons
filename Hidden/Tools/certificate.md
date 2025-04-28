# Odoo Module: certificate

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Certificate',
    'version': '0.1',
    'category': 'Hidden/Tools',
    'summary': 'Manage certificate',
    'installable': True,
    'data': [
        'security/ir.model.access.csv',
        'security/certificate_security.xml',
        'views/certificate_views.xml',
        'views/key_views.xml',
        'views/action_menus.xml',
        'views/res_config_settings_view.xml',
    ],
    'depends': ['base_setup'],
    'license': 'OEEL-1',
}

```

## File: data\neutralize.sql

```sql
UPDATE certificate_certificate
   SET pkcs12_password = 'dummy';
   
UPDATE certificate_key
   SET password = 'dummy';

```

## File: models\certificate.py

```python
import base64
import datetime
from importlib import metadata

from cryptography import x509
from cryptography.hazmat.primitives import constant_time, serialization
from cryptography.hazmat.primitives.serialization import Encoding, pkcs12

from odoo import _, api, fields, models
from .key import STR_TO_HASH, _get_formatted_value
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.tools import parse_version


class Certificate(models.Model):
    _name = 'certificate.certificate'
    _description = 'Certificate'
    _order = 'date_end DESC'
    _check_company_auto = True

    name = fields.Char(string='Name')
    content = fields.Binary(string='Certificate', readonly=False, required=True)
    pkcs12_password = fields.Char(string='Certificate Password', help='Password to decrypt the PKS file.')
    private_key_id = fields.Many2one(
        string='Private Key',
        comodel_name='certificate.key',
        check_company=True,
        domain=[('public', '=', False)],
        compute='_compute_private_key',
        store=True,
        readonly=False,
    )
    public_key_id = fields.Many2one(
        string='Public Key',
        comodel_name='certificate.key',
        check_company=True,
        domain=[('public', '=', True)],
        help="""Used to set a public key in case the one self-contained in the certificate is erroneus.
                When a public key is set this way, it will be used instead of the one in the certificate.
             """,
    )
    scope = fields.Selection(
        string="Certificate scope",
        selection=[
            ('general', 'General'),
        ],
    )
    content_format = fields.Selection(
        selection=[
            ('der', 'DER'),
            ('pem', 'PEM'),
            ('pkcs12', 'PKCS12'),
        ],
        string='Original certificate format',
        compute='_compute_pem_certificate',
        store=True,
    )
    pem_certificate = fields.Binary(
        string='Certificate in PEM format',
        compute='_compute_pem_certificate',
        store=True,
    )
    subject_common_name = fields.Char(
        string='Subject Name',
        compute='_compute_pem_certificate',
        store=True,
    )
    serial_number = fields.Char(
        string='Serial number',
        help='The serial number to add to electronic documents',
        compute='_compute_pem_certificate',
        store=True,
    )
    date_start = fields.Datetime(
        string='Available date',
        help='The date on which the certificate starts to be valid (UTC)',
        compute='_compute_pem_certificate',
        store=True,
    )
    date_end = fields.Datetime(
        string='Expiration date',
        help='The date on which the certificate expires (UTC)',
        compute='_compute_pem_certificate',
        store=True,
    )
    loading_error = fields.Text(string='Loading error', compute='_compute_pem_certificate', store=True)
    is_valid = fields.Boolean(string='Valid', compute='_compute_is_valid', search='_search_is_valid')
    active = fields.Boolean(name='Active', help='Set active to false to archive the certificate', default=True)
    company_id = fields.Many2one(
        comodel_name='res.company',
        string='Company',
        required=True,
        default=lambda self: self.env.company,
        ondelete='cascade',
    )
    country_code = fields.Char(related='company_id.country_code', depends=['company_id'])

    @api.depends('pem_certificate')
    def _compute_private_key(self):
        attachments = self.env['ir.attachment'].search([
            ('res_model', '=', 'certificate.key'),
            ('res_field', '=', 'content'),
            ('res_id', 'in', self.ids)
        ])
        content_to_key_id = {(att.datas, att.company_id.id): att.res_id for att in attachments}

        for certificate in self:
            if not certificate.pem_certificate:
                certificate.private_key_id = None
                continue

            if certificate.private_key_id:
                continue

            # Create the private key in case of PKCS12 File and no private key is set
            if certificate.content_format == 'pkcs12':
                content = certificate.with_context(bin_size=False).content
                pkcs12_password = certificate.pkcs12_password.encode('utf-8') if certificate.pkcs12_password else None
                key, _cert, _additional_certs = pkcs12.load_key_and_certificates(base64.b64decode(content), pkcs12_password)

                if key:
                    pem_key = base64.b64encode(key.private_bytes(
                        encoding=Encoding.PEM,
                        format=serialization.PrivateFormat.PKCS8,
                        encryption_algorithm=serialization.NoEncryption()
                    ))
                    key_id = content_to_key_id.get((pem_key, certificate.company_id.id))
                    if not key_id:
                        key_id = self.env['certificate.key'].create({
                            'name': (certificate.subject_common_name or certificate.name or "") + ".key",
                            'content': pem_key,
                            'company_id': certificate.company_id.id,
                        })
                    certificate.private_key_id = key_id

    @api.depends('content', 'pkcs12_password')
    def _compute_pem_certificate(self):
        for certificate in self:
            content = certificate.with_context(bin_size=False).content

            if not content:
                certificate.pem_certificate = None
                certificate.subject_common_name = None
                certificate.content_format = None
                certificate.date_start = None
                certificate.date_end = None
                certificate.serial_number = None
                certificate.loading_error = ""

            else:
                content = base64.b64decode(content)
                cert = None

                # Try to load the certificate in different format starting with DER then PKCS12 and
                # finally PEM. If none succeeded, we report an error.
                try:
                    cert = x509.load_der_x509_certificate(content)
                    certificate.content_format = 'der'
                except ValueError:
                    pass
                if not cert:
                    try:
                        pkcs12_password = certificate.pkcs12_password.encode('utf-8') if certificate.pkcs12_password else None
                        _key, cert, _additional_certs = pkcs12.load_key_and_certificates(content, pkcs12_password)
                        certificate.content_format = 'pkcs12'
                    except ValueError:
                        pass
                if not cert:
                    try:
                        cert = x509.load_pem_x509_certificate(content)
                        certificate.content_format = 'pem'
                    except ValueError:
                        pass

                if not cert:
                    certificate.pem_certificate = None
                    certificate.subject_common_name = None
                    certificate.content_format = None
                    certificate.date_start = None
                    certificate.date_end = None
                    certificate.serial_number = None
                    certificate.loading_error = _("This certificate could not be loaded. Either the content or the password is erroneous.")
                    continue

                try:
                    common_name = cert.subject.get_attributes_for_oid(x509.NameOID.COMMON_NAME)
                    certificate.subject_common_name = common_name[0].value if common_name else ""
                except ValueError:
                    certificate.subject_common_name = None

                certificate.loading_error = ""

                # Extract certificate data
                certificate.pem_certificate = base64.b64encode(cert.public_bytes(Encoding.PEM))
                certificate.serial_number = cert.serial_number
                if parse_version(metadata.version('cryptography')) < parse_version('42.0.0'):
                    certificate.date_start = cert.not_valid_before
                    certificate.date_end = cert.not_valid_after
                else:
                    certificate.date_start = cert.not_valid_before_utc.replace(tzinfo=None)
                    certificate.date_end = cert.not_valid_after_utc.replace(tzinfo=None)

    @api.depends('date_start', 'date_end', 'loading_error')
    def _compute_is_valid(self):
        # Certificate dates are UTC timezoned
        # https://cryptography.io/en/latest/x509/reference/#cryptography.x509.Certificate.not_valid_after
        utc_now = datetime.datetime.now(datetime.timezone.utc)
        for certificate in self:
            if not certificate.date_start or not certificate.date_end or certificate.loading_error:
                certificate.is_valid = False
            else:
                date_start = certificate.date_start.replace(tzinfo=datetime.timezone.utc)
                date_end = certificate.date_end.replace(tzinfo=datetime.timezone.utc)
                certificate.is_valid = date_start <= utc_now <= date_end

    def _search_is_valid(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise NotImplementedError("Operation not supported, only '=' and '!=' are allowed.")
        utc_now = datetime.datetime.now(datetime.timezone.utc)
        if (operator == '=' and value) or (operator == '!=' and not value):
            return [
                ('pem_certificate', '!=', False),
                ('date_start', '<=', utc_now),
                ('date_end', '>=', utc_now),
                ('loading_error', '=', '')
            ]
        else:
            return expression.OR([
                [('pem_certificate', '=', False)],
                [('date_start', '=', False)],
                [('date_end', '=', False)],
                [('date_start', '>', utc_now)],
                [('date_end', '<', utc_now)],
                [('loading_error', '!=', '')],
            ])

    @api.constrains('pem_certificate', 'private_key_id', 'public_key_id')
    def _constrains_certificate_key_compatibility(self):
        for certificate in self:
            pem_certificate = certificate.with_context(bin_size=False).pem_certificate
            if pem_certificate:
                cert = x509.load_pem_x509_certificate(base64.b64decode(pem_certificate))
                cert_public_key_bytes = cert.public_key().public_bytes(
                    encoding=Encoding.PEM,
                    format=serialization.PublicFormat.SubjectPublicKeyInfo
                )

                if certificate.private_key_id:
                    if certificate.private_key_id.loading_error:
                        raise UserError(certificate.private_key_id.loading_error)
                    pkey_public_key_bytes = base64.b64decode(
                        certificate.private_key_id._get_public_key_bytes(encoding='pem')
                    )
                    if not constant_time.bytes_eq(pkey_public_key_bytes, cert_public_key_bytes):
                        raise UserError(_("The certificate and private key are not compatible."))

                if certificate.public_key_id:
                    if certificate.public_key_id.loading_error:
                        raise UserError(certificate.public_key_id.loading_error)
                    pkey_public_key_bytes = base64.b64decode(
                        certificate.public_key_id._get_public_key_bytes(encoding='pem')
                    )
                    if not constant_time.bytes_eq(pkey_public_key_bytes, cert_public_key_bytes):
                        raise UserError(_("The certificate and public key are not compatible."))

    # -------------------------------------------------------
    #                   Business Methods                    #
    # -------------------------------------------------------

    def _get_der_certificate_bytes(self, formatting='encodebytes'):
        ''' Get the DER bytes of the certificate.

        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted DER bytes of the certificate
        :rtype: bytes
        '''
        self.ensure_one()
        cert = x509.load_pem_x509_certificate(base64.b64decode(self.with_context(bin_size=False).pem_certificate))
        return _get_formatted_value(cert.public_bytes(serialization.Encoding.DER), formatting=formatting)

    def _get_fingerprint_bytes(self, hashing_algorithm='sha256', formatting='encodebytes'):
        ''' Get the fingerprint bytes of the certificate.

        :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available.
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted fingerprint bytes of the certificate
        :rtype: bytes
        '''
        self.ensure_one()
        cert = x509.load_pem_x509_certificate(base64.b64decode(self.with_context(bin_size=False).pem_certificate))
        if hashing_algorithm not in STR_TO_HASH:
            raise UserError(f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256.")
        return _get_formatted_value(cert.fingerprint(STR_TO_HASH[hashing_algorithm]), formatting=formatting)

    def _get_signature_bytes(self, formatting='encodebytes'):
        ''' Get the signature bytes of the certificate.

        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted signature bytes of the certificate
        :rtype: bytes
        '''
        self.ensure_one()
        cert = x509.load_pem_x509_certificate(base64.b64decode(self.with_context(bin_size=False).pem_certificate))
        return _get_formatted_value(cert.signature, formatting=formatting)

    def _get_public_key_numbers_bytes(self, formatting='encodebytes'):
        ''' Get the certificate public key's public numbers bytes.

        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: A tuple containing the formatted public number bytes of the certificate's public key

        :rtype: tuple(bytes,bytes)
        '''
        self.ensure_one()
        if self.public_key_id or self.private_key_id:
            return (self.public_key_id or self.private_key_id)._get_public_key_numbers_bytes(formatting=formatting)

        # When no keys are set to the certificate, use the self-contained public key from the content
        return self.env['certificate.key']._numbers_public_key_bytes_with_key(
            self._get_public_key_bytes(encoding='pem'),
            formatting=formatting,
        )

    def _get_public_key_bytes(self, encoding='der', formatting='encodebytes'):
        ''' Get the certificate's public key bytes.

        :param optional,default='der' encoding: The formatting of the returned bytes
            - 'der' returns DER public key bytes
            - other returns PEM public key bytes
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted certificate public key bytes in the corresponding format
        :rtype: bytes
        '''
        self.ensure_one()
        if self.public_key_id or self.private_key_id:
            return (self.public_key_id or self.private_key_id)._get_public_key_bytes(encoding=encoding, formatting=formatting)

        # When no keys are set to the certificate, use the self-contained public key from the content
        try:
            cert = x509.load_pem_x509_certificate(base64.b64decode(self.with_context(bin_size=False).pem_certificate))
            public_key = cert.public_key()
        except ValueError:
            raise UserError(_("The public key from the certificate could not be loaded."))

        encoding = serialization.Encoding.DER if encoding == 'der' else serialization.Encoding.PEM
        return _get_formatted_value(
            public_key.public_bytes(
                encoding=encoding,
                format=serialization.PublicFormat.SubjectPublicKeyInfo
            ),
            formatting=formatting,
        )

    def _sign(self, message, hashing_algorithm='sha256', formatting='encodebytes'):
        ''' Compute and return the message's signature.

        :param str|bytes message: The message to sign
        :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available.
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted signature bytes of the message
        :rtype: bytes
        '''
        self.ensure_one()

        if not self.is_valid:
            raise UserError(self.loading_error or _("This certificate is not valid, its validity has expired."))
        if not self.private_key_id:
            raise UserError(_("No private key linked to the certificate, it is required to sign documents."))

        return self.private_key_id._sign(message, hashing_algorithm=hashing_algorithm, formatting=formatting)

```

## File: models\key.py

```python
import base64

from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import ec, padding, rsa
from cryptography.hazmat.primitives.serialization import Encoding

from odoo import _, api, fields, models
from odoo.exceptions import UserError


STR_TO_HASH = {
    'sha1': hashes.SHA1(),
    'sha256': hashes.SHA256(),
}

STR_TO_CURVE = {
    'SECP256R1': ec.SECP256R1(),
}


def _get_formatted_value(data, formatting='encodebytes'):
    if formatting == 'encodebytes':
        return base64.encodebytes(data)
    elif formatting == 'base64':
        return base64.b64encode(data)
    else:
        return data


def _int_to_bytes(value, byteorder='big'):
    return value.to_bytes((value.bit_length() + 7) // 8, byteorder=byteorder)


class Key(models.Model):
    _name = 'certificate.key'
    _description = 'Cryptographic Keys'

    name = fields.Char(string='Name', default="New key")
    content = fields.Binary(string='Key file', required=True)
    password = fields.Char(string='Private key password')
    pem_key = fields.Binary(
        string='Key bytes in PEM format',
        compute='_compute_pem_key',
        store=True,
    )
    public = fields.Boolean(
        string='Public/Private key',
        compute='_compute_pem_key',
        store=True,
    )
    loading_error = fields.Text(
        string='Loading error',
        compute='_compute_pem_key',
        store=True,
    )
    active = fields.Boolean(name='Active', help='Set active to false to archive the key.', default=True)
    company_id = fields.Many2one(
        comodel_name='res.company',
        string='Company',
        required=True,
        default=lambda self: self.env.company,
        ondelete='cascade',
    )

    @api.depends('content', 'password')
    def _compute_pem_key(self):
        for key in self:
            content = key.with_context(bin_size=False).content
            if not content:
                key.pem_key = None
                key.public = None
                key.loading_error = ""
            else:
                pkey_content = base64.b64decode(content)
                pkey_password = key.password.encode('utf-8') if key.password else None

                # Try to load the key in different format starting with DER then PEM for private then public keys.
                # If none succeeded, we report an error.
                pkey = None
                try:
                    pkey = serialization.load_der_private_key(pkey_content, pkey_password)
                    key.public = False
                except (ValueError, TypeError):
                    pass

                if not pkey:
                    try:
                        pkey = serialization.load_pem_private_key(pkey_content, pkey_password)
                        key.public = False
                    except (ValueError, TypeError):
                        pass

                if not pkey:
                    try:
                        pkey = serialization.load_der_public_key(pkey_content)
                        key.public = True
                    except (ValueError, TypeError):
                        pass

                if not pkey:
                    try:
                        pkey = serialization.load_pem_public_key(pkey_content)
                        key.public = True
                    except (ValueError, TypeError):
                        pass

                if not pkey:
                    key.pem_key = None
                    key.public = None
                    key.loading_error = _("This key could not be loaded. Either its content or its password is erroneous.")
                    continue

                if key.public:
                    key.pem_key = base64.b64encode(pkey.public_bytes(
                        encoding=Encoding.PEM,
                        format=serialization.PublicFormat.SubjectPublicKeyInfo,
                    ))
                else:
                    key.pem_key = base64.b64encode(pkey.private_bytes(
                        encoding=Encoding.PEM,
                        format=serialization.PrivateFormat.PKCS8,
                        encryption_algorithm=serialization.NoEncryption()
                    ))

                key.loading_error = ""

    # -------------------------------------------------------
    #                   Business Methods                    #
    # -------------------------------------------------------

    def _sign(self, message, hashing_algorithm='sha256', formatting='encodebytes'):
        ''' Compute and return the message's signature.

        :param str|bytes message: The message to sign
        :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available.
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted signature bytes of the message
        :rtype: bytes
        '''
        self.ensure_one()

        if self.public:
            raise UserError(_("Make sure to use a private key to sign documents."))

        pem_key = self.with_context(bin_size=False).pem_key
        if self.loading_error:
            raise UserError(self.name + " - " + self.loading_error)

        return self._sign_with_key(
            message,
            pem_key,
            pwd=None,
            hashing_algorithm=hashing_algorithm,
            formatting=formatting
        )

    def _get_public_key_numbers_bytes(self, formatting='encodebytes'):
        ''' Get the public key's public numbers bytes.

        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: A tuple containing formatted public number bytes of the public key
        :rtype: tuple(bytes,bytes)
        '''
        self.ensure_one()

        return self._numbers_public_key_bytes_with_key(
            self._get_public_key_bytes(encoding='PEM'),
            formatting=formatting,
        )

    def _get_public_key_bytes(self, encoding='der', formatting='encodebytes'):
        ''' Get the public key bytes.

        :param optional,default='der' encoding: The formatting of the returned bytes
            - 'der' returns DER public key bytes
            - other returns PEM public key bytes
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted public key bytes in the corresponding format
        :rtype: bytes
        '''
        self.ensure_one()

        if self.public:
            public_key = serialization.load_pem_public_key(base64.b64decode(self.with_context(bin_size=False).pem_key))
        else:
            public_key = serialization.load_pem_private_key(base64.b64decode(self.with_context(bin_size=False).pem_key), None).public_key()

        encoding = serialization.Encoding.DER if encoding == 'der' else serialization.Encoding.PEM
        return _get_formatted_value(
            public_key.public_bytes(
                encoding=encoding,
                format=serialization.PublicFormat.SubjectPublicKeyInfo
            ),
            formatting=formatting,
        )

    def _decrypt(self, message, hashing_algorithm='sha256'):
        ''' Decrypt the given message using the provided digest.

        :param str|bytes message: The message to encode
        :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available.
        :return: The decrypted text
        :rtype: str
        '''
        self.ensure_one()

        if not isinstance(message, bytes):
            message = message.encode('utf-8')

        if self.public:
            raise UserError(_("A private key is required to decrypt data."))
        if hashing_algorithm not in STR_TO_HASH:
            raise UserError(f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256.")

        private_key = serialization.load_pem_private_key(base64.b64decode(self.pem_key), None)
        if not isinstance(private_key, rsa.RSAPrivateKey):
            raise UserError(_("Unsupported asymmetric cryptography algorithm '%s'. Currently supported for decryption: RSA.", type(private_key)))

        return private_key.decrypt(
            message,
            padding.OAEP(
                mgf=padding.MGF1(algorithm=STR_TO_HASH[hashing_algorithm]),
                algorithm=STR_TO_HASH[hashing_algorithm],
                label=None
            )
        ).decode()

    @api.model
    def _sign_with_key(self, message, pem_key, pwd=None, hashing_algorithm='sha256', formatting='encodebytes'):
        ''' Compute and return the message's signature for a given private key.

        :param str|bytes message: The message to sign
        :param str|bytes pem_key: A base64 encoded private key in the PEM format
        :param str|bytes pwd: A password to decrypt the PEM key
        :param optional,default='sha1' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available.
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: The formatted signature bytes of the message
        :rtype: bytes
        '''

        if not isinstance(message, bytes):
            message = message.encode('utf-8')
        if not isinstance(pem_key, bytes):
            pem_key = pem_key.encode('utf-8')
        if pwd and not isinstance(pwd, bytes):
            pwd = pwd.encode('utf-8')

        if hashing_algorithm not in STR_TO_HASH:
            raise UserError(f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256.")

        try:
            private_key = serialization.load_pem_private_key(base64.b64decode(pem_key), pwd)
        except ValueError:
            raise UserError(_("The private key could not be loaded."))

        if isinstance(private_key, ec.EllipticCurvePrivateKey):
            signature = private_key.sign(
                message,
                ec.ECDSA(STR_TO_HASH[hashing_algorithm])
            )
        elif isinstance(private_key, rsa.RSAPrivateKey):
            signature = private_key.sign(
                message,
                padding.PKCS1v15(),
                STR_TO_HASH[hashing_algorithm]
            )
        else:
            raise UserError(_("Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: EC and RSA.", type(private_key)))

        return _get_formatted_value(signature, formatting=formatting)

    @api.model
    def _numbers_public_key_bytes_with_key(self, pem_key, formatting='encodebytes'):
        ''' Get the given public key's public numbers bytes.

        :param str|bytes pem_key: A base64 encoded public key in the PEM format
        :param optional,default='encodebytes' formatting: The formatting of the returned bytes
            - 'encodebytes' returns a base64-encoded block of 76 characters lines
            - 'base64' returns the raw base64-encoded data
            - other returns non-encoded data
        :return: A tuple containing the formatted public number bytes of the public key
        :rtype: tuple(bytes,bytes)
        '''
        if not isinstance(pem_key, bytes):
            pem_key = pem_key.encode('utf-8')

        try:
            public_key = serialization.load_pem_public_key(base64.b64decode(pem_key))
        except ValueError:
            raise UserError(_("The public key could not be loaded."))

        if isinstance(public_key, ec.EllipticCurvePublicKey):
            e = public_key.public_numbers().x
            n = public_key.public_numbers().y
        elif isinstance(public_key, rsa.RSAPublicKey):
            e = public_key.public_numbers().e
            n = public_key.public_numbers().n
        else:
            raise UserError(_("Unsupported asymmetric cryptography algorithm '%s'. Currently supported: EC, RSA.", type(public_key)))

        return (
            _get_formatted_value(_int_to_bytes(e), formatting=formatting),
            _get_formatted_value(_int_to_bytes(n), formatting=formatting)
        )

    @api.model
    def _generate_ec_private_key(self, company, name='id_ec', curve='SECP256R1'):
        ''' Generate an elliptic curve private key.

        :param res.company company: A company record
        :param str,optional,default='id_ec' name: The name of the newly created key.
        :param optional,default='SECP256R1' curve: The type of elliptic curve algorithm. Currently, only SECP256R1 is supported.
        :return: A certificate.key record
        :rtype: certificate.key
        '''
        if curve not in STR_TO_CURVE:
            raise UserError(f"Unsupported curve algorithm '{curve}'. Currently supported: SECP256R1.")

        private_key = ec.generate_private_key(STR_TO_CURVE[curve])

        return self.env['certificate.key'].create({
            'name': name,
            'content': base64.b64encode(private_key.private_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PrivateFormat.PKCS8,
                encryption_algorithm=serialization.NoEncryption())),
            'company_id': company.id,
        })

    @api.model
    def _generate_rsa_private_key(self, company, name='id_rsa', public_exponent=65537, key_size=2048):
        ''' Generate an RSA private key.

        :param res.company company: A company record
        :param str,optional,default='id_rsa' name: The name of the newly created key.
        :param int,optional,default=65537 public_exponent: The public exponent of the new key: either 65537 or 3 (for legacy purposes)
        :param int,optional,default=2048 key_size: The length of the modulus in bits; it is strongly recommended to be at least 2048 and must not be less than 512
        :return: A certificate.key record
        :rtype: certificate.key
        '''
        if (public_exponent not in [65537, 3]):
            raise UserError(_("The public exponent should be 65537 (or 3 for legacy purposes)."))
        if key_size < 512:
            raise UserError(_("The key size should be at least 512 bytes."))

        private_key = rsa.generate_private_key(
            public_exponent=public_exponent,
            key_size=key_size
        )

        return self.env['certificate.key'].create({
            'name': name,
            'content': base64.b64encode(private_key.private_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PrivateFormat.PKCS8,
                encryption_algorithm=serialization.NoEncryption())),
            'company_id': company.id,
        })

```

## File: models\__init__.py

```python
from . import certificate
from . import key

```

## File: security\certificate_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <record id="certificate_rule_company" model="ir.rule">
        <field name="name">Certificate multi-company</field>
        <field name="model_id" ref="model_certificate_certificate"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]</field>
    </record>

    <record id="key_rule_company" model="ir.rule">
        <field name="name">Key multi-company</field>
        <field name="model_id" ref="model_certificate_key"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]</field>
    </record>
</data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
certificate.access_certificate_admin,certificate.access_certificate,model_certificate_certificate,base.group_system,1,1,1,1
certificate.access_key_admin,certificate.access_key,model_certificate_key,base.group_system,1,1,1,1

```

## File: views\action_menus.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<odoo>
    <record id="certificate_certificate_action_view_list" model="ir.actions.act_window">
        <field name="name">Certificates</field>
        <field name="res_model">certificate.certificate</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">Create a first certificate</p>
        </field>
    </record>

    <record id="certificate_key_action_view_list" model="ir.actions.act_window">
        <field name="name">Keys</field>
        <field name="res_model">certificate.key</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">Create a first key</p>
        </field>
    </record>
</odoo>

```

## File: views\certificate_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<odoo>
    <record id="certificate_certificate_view_form" model="ir.ui.view">
        <field name="name">certificate.certificate.form</field>
        <field name="model">certificate.certificate</field>
        <field name="arch" type="xml">
            <form string="certificate form">
                <sheet>
                    <div class="alert alert-warning" role="alert" invisible="not loading_error">
                        <field name="loading_error"/>
                    </div>
                    <group id="certificate_input">
                        <field name="name" placeholder="e.g. New Certificate"/>
                        <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                        <field name="content" widget="binary"/>
                        <field name="pkcs12_password" password="True" />
                        <field name="subject_common_name" invisible="not pem_certificate"/>
                        <field name="private_key_id" invisible="not pem_certificate" options="{'no_quick_create': True}"/>
                        <field name="public_key_id" invisible="not pem_certificate" options="{'no_quick_create': True}"/>
                        <field name="scope" widget="radio" invisible="1"/> <!-- Should be overriden by modules adding options to this selection field -->
                    </group>
                    <group id="certificate_data">
                        <label for="date_start" string="Validity"/>
                        <div>
                            <field name="date_start" interval="year" widget="remaining_days" class="oe_inline"/> - <field name="date_end" widget="remaining_days" class="oe_inline"/>
                        </div>
                        <field name="serial_number"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="certificate_certificate_view_list" model="ir.ui.view">
        <field name="name">certificate.certificate.list</field>
        <field name="model">certificate.certificate</field>
        <field name="arch" type="xml">
            <list string="certificate list">
                <field name="name" string="Certificate"/>
                <field name="subject_common_name" string="Subject Name"/>
                <field name="is_valid" string="Validity"/>
                <field name="company_id" groups="base.group_multi_company" readonly="1" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="certificate_certificate_view_search" model="ir.ui.view">
        <field name="name">certificate.certificate.search</field>
        <field name="model">certificate.certificate</field>
        <field name="arch" type="xml">
            <search string="certificate search">
                <field name="name"/>
                <field name="scope"/>
                <separator/>
                <filter string="General" name="scope_general" domain="[('scope','=','general')]" help="General certificates"/>
                <separator/>
                <filter string="Valid" name="valid" domain="[('is_valid','=',True)]" help="Valid certificates"/>
                <filter string="Invalid" name="not_valid" domain="[('is_valid','=',False)]" help="Not valid certificates"/>
                <separator/>
                <filter string="Archived" name="archived" domain="[('active','=',False)]" help="Archived certificates"/>
            </search>
        </field>
    </record>
</odoo>

```

## File: views\key_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<odoo>
    <record id="certificate_key_view_form" model="ir.ui.view">
        <field name="name">certificate.key.form</field>
        <field name="model">certificate.key</field>
        <field name="arch" type="xml">
            <form string="key form">
                <sheet>
                    <div class="alert alert-warning" role="alert" invisible="not loading_error">
                        <field name="loading_error"/>
                    </div>
                    <group id="key_input">
                        <field name="name"/>
                        <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                        <field name="content" widget="binary"/>
                        <field name="password" password="True" />
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="certificate_key_view_list" model="ir.ui.view">
        <field name="name">certificate.key.list</field>
        <field name="model">certificate.key</field>
        <field name="arch" type="xml">
            <list string="key list">
                <field name="name" string="Key"/>
                <field name="company_id" groups="base.group_multi_company" readonly="1" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="certificate_key_view_search" model="ir.ui.view">
        <field name="name">certificate.key.search</field>
        <field name="model">certificate.key</field>
        <field name="arch" type="xml">
            <search string="key search">
                <filter string="Archived" name="archived" domain="[('active','=',False)]" help="Archived keys"/>
                <separator/>
                <filter string="Private" name="private" domain="[('pem_key', '!=', False), ('public','=',False)]"/>
                <filter string="Public" name="public" domain="[('pem_key', '!=', False), ('public','=',True)]"/>
                <separator/>
                <filter string="Invalid" name="invalid" domain="[('pem_key', '=', False)]"/>
            </search>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.certificate</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='user_default_rights']" position="after">
                <block title="Certificates and Keys" id="certificates_and_keys_settings" groups="base.group_system">
                    <setting string="Manage your certificates" id="certificates_settings" company_dependent="1" title="Add, edit and delete certificates.">
                        <div class="content-group">
                            <div class="row mt16">
                                <div class="col-6">
                                    <button name="%(certificate.certificate_certificate_action_view_list)d" icon="oi-arrow-right" type="action"
                                            string="Certificates" class="btn-link"/>
                                </div>
                            </div>
                        </div>
                    </setting>
                    <setting string="Manage your keys" id="keys_settings" company_dependent="1" title="Add, edit and delete keys.">
                        <div class="content-group">
                            <div class="row mt16">
                                <div class="col-6">
                                    <button name="%(certificate.certificate_key_action_view_list)d" icon="oi-arrow-right" type="action"
                                            string="Keys" class="btn-link"/>
                                </div>
                            </div>
                        </div>
                    </setting>
                </block>
            </xpath>
        </field>
    </record>
</odoo>

```

