# Odoo Module: utm

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'UTM Trackers',
    'category': 'Hidden',
    'description': """
Enable management of UTM trackers: campaign, medium, source.
""",
    'version': '1.1',
    'depends': ['base', 'web'],
    'data': [
        'data/utm_medium_data.xml',
        'data/utm_source_data.xml',
        'data/utm_stage_data.xml',
        'data/utm_tag_data.xml',
        'views/utm_campaign_views.xml',
        'views/utm_medium_views.xml',
        'views/utm_source_views.xml',
        'views/utm_stage_views.xml',
        'views/utm_tag_views.xml',
        'views/utm_menus.xml',
        'security/ir.model.access.csv',
    ],
    'demo': [
        'data/utm_campaign_demo.xml',
        'data/utm_stage_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'utm/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\utm_campaign_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="utm.campaign" id="utm_campaign_fall_drive">
            <field name="name">Sale</field>
            <field name="stage_id" ref="utm.default_utm_stage" />
            <field name="is_auto_campaign" eval="True" />
        </record>
        <record model="utm.campaign" id="utm_campaign_christmas_special">
            <field name="name">Christmas Special</field>
            <field name="stage_id" ref="utm.default_utm_stage" />
            <field name="is_auto_campaign" eval="True" />
        </record>
        <record model="utm.campaign" id="utm_campaign_email_campaign_services">
            <field name="name">Email Campaign - Services</field>
            <field name="stage_id" ref="utm.default_utm_stage" />
            <field name="is_auto_campaign" eval="True" />
        </record>
        <record model="utm.campaign" id="utm_campaign_email_campaign_products">
            <field name="name">Email Campaign - Products</field>
            <field name="stage_id" ref="utm.default_utm_stage" />
            <field name="is_auto_campaign" eval="True" />
        </record>
    </data>
</odoo>

```

## File: data\utm_medium_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="utm.medium" id="utm_medium_website">
        <field name="name">Website</field>
    </record>
    <record model="utm.medium" id="utm_medium_phone">
        <field name="name">Phone</field>
    </record>
    <record model="utm.medium" id="utm_medium_direct">
        <field name="name">Direct</field>
    </record>
    <record model="utm.medium" id="utm_medium_email">
        <field name="name">Email</field>
    </record>
    <record model="utm.medium" id="utm_medium_banner">
        <field name="name">Banner</field>
    </record>
    <record model="utm.medium" id="utm_medium_twitter">
        <field name="name">X</field>
    </record>
    <record model="utm.medium" id="utm_medium_facebook">
        <field name="name">Facebook</field>
    </record>
    <record model="utm.medium" id="utm_medium_linkedin">
        <field name="name">LinkedIn</field>
    </record>
    <record model="utm.medium" id="utm_medium_television">
        <field name="name">Television</field>
    </record>
    <record model="utm.medium" id="utm_medium_google_adwords">
        <field name="name">Google Adwords</field>
    </record>
</odoo>

```

## File: data\utm_source_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="utm.source" id="utm_source_search_engine">
        <field name="name">Search engine</field>
    </record>
    <record model="utm.source" id="utm_source_mailing">
        <field name="name">Lead Recall</field>
    </record>
    <record model="utm.source" id="utm_source_newsletter">
        <field name="name">Newsletter</field>
    </record>
    <record model="utm.source" id="utm_source_facebook">
        <field name="name">Facebook</field>
    </record>
    <record model="utm.source" id="utm_source_twitter">
        <field name="name">X</field>
    </record>
    <record model="utm.source" id="utm_source_linkedin">
        <field name="name">LinkedIn</field>
    </record>
    <record model="utm.source" id="utm_source_monster">
        <field name="name">Monster</field>
    </record>
    <record model="utm.source" id="utm_source_glassdoor">
        <field name="name">Glassdoor</field>
    </record>
    <record model="utm.source" id="utm_source_craigslist">
        <field name="name">Craigslist</field>
    </record>
</odoo>

```

## File: data\utm_stage_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- This one is kept in data instead of demo to avoid crashing if the user starts creating campaigns
    before stages have been created, as they are mandatory -->
    <record id="default_utm_stage" model="utm.stage">
        <field name="name">New</field>
        <field name="sequence">10</field>
    </record>
</odoo>

```

## File: data\utm_stage_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="campaign_stage_1" model="utm.stage">
            <field name="name">Schedule</field>
            <field name="sequence">10</field>
        </record>
        <record id="campaign_stage_2" model="utm.stage">
            <field name="name">Design</field>
            <field name="sequence">20</field>
        </record>
        <record id="campaign_stage_3" model="utm.stage">
            <field name="name">Sent</field>
            <field name="sequence">30</field>
        </record>
    </data>
</odoo>

```

## File: data\utm_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- UTM Tag -->
    <record id="utm_tag_1" model="utm.tag">
        <field name="name">Marketing</field>
        <field name="color" eval="1"/>
    </record>
</odoo>

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request, Response


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def get_utm_domain_cookies(cls):
        return request.httprequest.host

    @classmethod
    def _set_utm(cls, response):
        # Make sure response is an odoo Response.
        response = Response.load(response)
        domain = cls.get_utm_domain_cookies()
        for url_parameter, __, cookie_name in request.env['utm.mixin'].tracking_fields():
            if url_parameter in request.params and request.cookies.get(cookie_name) != request.params[url_parameter]:
                response.set_cookie(cookie_name, request.params[url_parameter], max_age=31 * 24 * 3600, domain=domain, cookie_type='optional')

    @classmethod
    def _post_dispatch(cls, response):
        cls._set_utm(response)
        super()._post_dispatch(response)

```

## File: models\utm_campaign.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class UtmCampaign(models.Model):
    _name = 'utm.campaign'
    _description = 'UTM Campaign'
    _rec_name = 'title'

    active = fields.Boolean('Active', default=True)
    name = fields.Char(string='Campaign Identifier', required=True, compute='_compute_name',
                       store=True, readonly=False, precompute=True, translate=False)
    title = fields.Char(string='Campaign Name', required=True, translate=True)

    user_id = fields.Many2one(
        'res.users', string='Responsible',
        required=True, default=lambda self: self.env.uid)
    stage_id = fields.Many2one(
        'utm.stage', string='Stage', ondelete='restrict', required=True,
        default=lambda self: self.env['utm.stage'].search([], limit=1),
        copy=False, group_expand='_group_expand_stage_ids')
    tag_ids = fields.Many2many(
        'utm.tag', 'utm_tag_rel',
        'tag_id', 'campaign_id', string='Tags')

    is_auto_campaign = fields.Boolean(default=False, string="Automatically Generated Campaign", help="Allows us to filter relevant Campaigns")
    color = fields.Integer(string='Color Index')

    _sql_constraints = [
        ('unique_name', 'UNIQUE(name)', 'The name must be unique'),
    ]

    @api.depends('title')
    def _compute_name(self):
        new_names = self.env['utm.mixin'].with_context(
            utm_check_skip_record_ids=self.ids
        )._get_unique_names(self._name, [c.title for c in self])
        for campaign, new_name in zip(self, new_names):
            campaign.name = new_name

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('title') and vals.get('name'):
                vals['title'] = vals['name']
        new_names = self.env['utm.mixin']._get_unique_names(self._name, [vals.get('name') for vals in vals_list])
        for vals, new_name in zip(vals_list, new_names):
            if new_name:
                vals['name'] = new_name
        return super().create(vals_list)

    @api.model
    def _group_expand_stage_ids(self, stages, domain):
        """Read group customization in order to display all the stages in the
        Kanban view, even if they are empty.
        """
        stage_ids = stages.sudo()._search([], order=stages._order)
        return stages.browse(stage_ids)

```

## File: models\utm_medium.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import _, api, fields, models
from odoo.exceptions import UserError


class UtmMedium(models.Model):
    _name = 'utm.medium'
    _description = 'UTM Medium'
    _order = 'name'

    name = fields.Char(string='Medium Name', required=True, translate=False)
    active = fields.Boolean(default=True)

    _sql_constraints = [
        ('unique_name', 'UNIQUE(name)', 'The name must be unique'),
    ]

    @api.model_create_multi
    def create(self, vals_list):
        new_names = self.env['utm.mixin']._get_unique_names(self._name, [vals.get('name') for vals in vals_list])
        for vals, new_name in zip(vals_list, new_names):
            vals['name'] = new_name
        return super().create(vals_list)

    @property
    def SELF_REQUIRED_UTM_MEDIUMS_REF(self):
        return {
            'utm.utm_medium_email': 'Email',
            'utm.utm_medium_direct': 'Direct',
            'utm.utm_medium_website': 'Website',
            'utm.utm_medium_twitter': 'X',
            'utm.utm_medium_facebook': 'Facebook',
            'utm.utm_medium_linkedin': 'LinkedIn'
        }

    @api.ondelete(at_uninstall=False)
    def _unlink_except_utm_medium_record(self):
        for medium in self.SELF_REQUIRED_UTM_MEDIUMS_REF:
            utm_medium = self.env.ref(medium, raise_if_not_found=False)
            if utm_medium and utm_medium in self:
                raise UserError(_(
                    "Oops, you can't delete the Medium '%s'.\n"
                    "Doing so would be like tearing down a load-bearing wall \u2014 not the best idea.",
                    utm_medium.name
                ))

    def _fetch_or_create_utm_medium(self, name, module='utm'):
        try:
            return self.env.ref(f'{module}.utm_medium_{name}')
        except ValueError:
            utm_medium = self.sudo().env['utm.medium'].create({
                'name': self.SELF_REQUIRED_UTM_MEDIUMS_REF.get(f'{module}.utm_medium_{name}', name)
            })
            self.sudo().env['ir.model.data'].create({
                'name': f'utm_medium_{name}',
                'module': module,
                'res_id': utm_medium.id,
                'model': 'utm.medium',
            })
            return utm_medium

```

## File: models\utm_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from collections import defaultdict
import itertools

from odoo import api, fields, models
from odoo.http import request
from odoo.osv import expression


class UtmMixin(models.AbstractModel):
    """ Mixin class for objects which can be tracked by marketing. """
    _name = 'utm.mixin'
    _description = 'UTM Mixin'

    campaign_id = fields.Many2one('utm.campaign', 'Campaign', index='btree_not_null',
                                  help="This is a name that helps you keep track of your different campaign efforts, e.g. Fall_Drive, Christmas_Special")
    source_id = fields.Many2one('utm.source', 'Source', index='btree_not_null',
                                help="This is the source of the link, e.g. Search Engine, another domain, or name of email list")
    medium_id = fields.Many2one('utm.medium', 'Medium', index='btree_not_null',
                                help="This is the method of delivery, e.g. Postcard, Email, or Banner Ad")

    @api.model
    def default_get(self, fields):
        values = super(UtmMixin, self).default_get(fields)

        # We ignore UTM for salesmen, except some requests that could be done as superuser_id to bypass access rights.
        if not self.env.is_superuser() and self.env.user.has_group('sales_team.group_sale_salesman'):
            return values

        for url_param, field_name, cookie_name in self.env['utm.mixin'].tracking_fields():
            if field_name in fields:
                field = self._fields[field_name]
                value = False
                if request:
                    # ir_http dispatch saves the url params in a cookie
                    value = request.cookies.get(cookie_name)
                # if we receive a string for a many2one, we search/create the id
                if field.type == 'many2one' and isinstance(value, str) and value:
                    record = self._find_or_create_record(field.comodel_name, value)
                    value = record.id
                if value:
                    values[field_name] = value
        return values

    def tracking_fields(self):
        # This function cannot be overridden in a model which inherit utm.mixin
        # Limitation by the heritage on AbstractModel
        # record_crm_lead.tracking_fields() will call tracking_fields() from module utm.mixin (if not overridden on crm.lead)
        # instead of the overridden method from utm.mixin.
        # To force the call of overridden method, we use self.env['utm.mixin'].tracking_fields() which respects overridden
        # methods of utm.mixin, but will ignore overridden method on crm.lead
        return [
            # ("URL_PARAMETER", "FIELD_NAME_MIXIN", "NAME_IN_COOKIES")
            ('utm_campaign', 'campaign_id', 'odoo_utm_campaign'),
            ('utm_source', 'source_id', 'odoo_utm_source'),
            ('utm_medium', 'medium_id', 'odoo_utm_medium'),
        ]

    def _tracking_models(self):
        fnames = {fname for _, fname, _ in self.tracking_fields()}
        return {
            self._fields[fname].comodel_name
            for fname in fnames
            if fname in self._fields and self._fields[fname].type == "many2one"
        }

    @api.model
    def find_or_create_record(self, model_name, name):
        """ Version of ``_find_or_create_record`` used in frontend notably in
        website_links. For UTM models it calls _find_or_create_record. For other
        models (as through inheritance custom models could be used notably in
        website links) it simply calls a create. In the end it relies on
        standard ACLs, and is mainly a wrapper for UTM models.

        :return: id of newly created or found record. As the magic of call_kw
        for create is not called anymore we have to manually return an id
        instead of a recordset. """
        if model_name in self._tracking_models():
            record = self._find_or_create_record(model_name, name)
        else:
            record = self.env[model_name].create({self.env[model_name]._rec_name: name})
        return {'id': record.id, 'name': record.display_name}

    def _find_or_create_record(self, model_name, name):
        """Based on the model name and on the name of the record, retrieve the corresponding record or create it."""
        Model = self.env[model_name]

        cleaned_name = name.strip()
        if cleaned_name:
            record = Model.with_context(active_test=False).search([('name', '=ilike', cleaned_name)], limit=1)

        if not record:
            # No record found, create a new one
            record_values = {'name': cleaned_name}
            if 'is_auto_campaign' in record._fields:
                record_values['is_auto_campaign'] = True
            record = Model.create(record_values)

        return record

    @api.model
    def _get_unique_names(self, model_name, names):
        """Generate unique names for the given model.

        Take a list of names and return for each names, the new names to set
        in the same order (with a counter added if needed).

        E.G.
            The name "test" already exists in database
            Input: ['test', 'test [3]', 'bob', 'test', 'test']
            Output: ['test [2]', 'test [3]', 'bob', 'test [4]', 'test [5]']

        :param model_name: name of the model for which we will generate unique names
        :param names: list of names, we will ensure that each name will be unique
        :return: a list of new values for each name, in the same order
        """
        # Avoid conflicting with itself, otherwise each check at update automatically
        # increments counters
        skip_record_ids = self.env.context.get("utm_check_skip_record_ids") or []
        # Remove potential counter part in each names
        names_without_counter = {self._split_name_and_count(name)[0] for name in names}

        # Retrieve existing similar names
        search_domain = expression.OR([[('name', 'ilike', name)] for name in names_without_counter])
        if skip_record_ids:
            search_domain = expression.AND([
                [('id', 'not in', skip_record_ids)],
                search_domain
            ])
        existing_names = {vals['name'] for vals in self.env[model_name].search_read(search_domain, ['name'])}

        # Counter for each names, based on the names list given in argument
        # and the record names in database
        used_counters_per_name = {
            name: {
                self._split_name_and_count(existing_name)[1]
                for existing_name in existing_names
                if existing_name == name or existing_name.startswith(f'{name} [')
            } for name in names_without_counter
        }
        # Automatically incrementing counters for each name, will be used
        # to fill holes in used_counters_per_name
        current_counter_per_name = defaultdict(lambda: itertools.count(1))

        result = []
        for name in names:
            if not name:
                result.append(False)
                continue

            name_without_counter, asked_counter = self._split_name_and_count(name)
            existing = used_counters_per_name.get(name_without_counter, set())
            if asked_counter and asked_counter not in existing:
                count = asked_counter
            else:
                # keep going until the count is not already used
                for count in current_counter_per_name[name_without_counter]:
                    if count not in existing:
                        break
            existing.add(count)
            result.append(f'{name_without_counter} [{count}]' if count > 1 else name_without_counter)

        return result

    @staticmethod
    def _split_name_and_count(name):
        """
        Return the name part and the counter based on the given name.

        e.g.
            "Medium" -> "Medium", 1
            "Medium [1234]" -> "Medium", 1234
        """
        name = name or ''
        name_counter_re = r'(.*)\s+\[([0-9]+)\]'
        match = re.match(name_counter_re, name)
        if match:
            return match.group(1), int(match.group(2) or '1')
        return name, 1

```

## File: models\utm_source.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import _, api, fields, models, tools


class UtmSource(models.Model):
    _name = 'utm.source'
    _description = 'UTM Source'

    name = fields.Char(string='Source Name', required=True)

    _sql_constraints = [
        ('unique_name', 'UNIQUE(name)', 'The name must be unique'),
    ]

    @api.model_create_multi
    def create(self, vals_list):
        new_names = self.env['utm.mixin']._get_unique_names(self._name, [vals.get('name') for vals in vals_list])
        for vals, new_name in zip(vals_list, new_names):
            vals['name'] = new_name
        return super().create(vals_list)

    def _generate_name(self, record, content):
        """Generate the UTM source name based on the content of the source."""
        if not content:
            return False

        content = content.replace('\n', ' ')
        if len(content) >= 24:
            content = f'{content[:20]}...'

        create_date = record.create_date or fields.date.today()
        create_date = fields.date.strftime(create_date, tools.DEFAULT_SERVER_DATE_FORMAT)
        model_description = self.env['ir.model']._get(record._name).name
        return _(
            '%(content)s (%(model_description)s created on %(create_date)s)',
            content=content, model_description=model_description, create_date=create_date,
        )


class UtmSourceMixin(models.AbstractModel):
    """Mixin responsible of generating the name of the source based on the content
    (field defined by _rec_name) of the record (mailing, social post,...).
    """
    _name = 'utm.source.mixin'
    _description = 'UTM Source Mixin'

    name = fields.Char('Name', related='source_id.name', readonly=False)
    source_id = fields.Many2one('utm.source', string='Source', required=True, ondelete='restrict', copy=False)

    @api.model_create_multi
    def create(self, vals_list):
        """Create the UTM sources if necessary, generate the name based on the content in batch."""
        # Create all required <utm.source>
        utm_sources = self.env['utm.source'].create([
            {'name': values.get('name') or self.env['utm.source']._generate_name(self, values.get(self._rec_name))}
            for values in vals_list
            if not values.get('source_id')
        ])

        # Update "vals_list" to add the ID of the newly created source
        vals_list_missing_source = [values for values in vals_list if not values.get('source_id')]
        for values, source in zip(vals_list_missing_source, utm_sources):
            values['source_id'] = source.id

        for values in vals_list:
            if 'name' in values:
                del values['name']

        return super().create(vals_list)

    def write(self, values):
        if (values.get(self._rec_name) or values.get('name')) and len(self) > 1:
            raise ValueError(
                _('You cannot update multiple records with the same name. The name should be unique!')
            )

        if values.get(self._rec_name) and not values.get('name'):
            values['name'] = self.env['utm.source']._generate_name(self, values[self._rec_name])
        if values.get('name'):
            values['name'] = self.env['utm.mixin'].with_context(
                utm_check_skip_record_ids=self.source_id.ids
            )._get_unique_names("utm.source", [values['name']])[0]

        return super().write(values)

    def copy_data(self, default=None):
        """Increment the counter when duplicating the source."""
        default = default or {}
        default_name = default.get('name')
        vals_list = super().copy_data(default=default)
        for source, vals in zip(self, vals_list):
            vals['name'] = self.env['utm.mixin']._get_unique_names("utm.source", [default_name or source.name])[0]
        return vals_list

```

## File: models\utm_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import fields, models


class UtmStage(models.Model):
    """Stage for utm campaigns."""

    _name = 'utm.stage'
    _description = 'Campaign Stage'
    _order = 'sequence'

    name = fields.Char(required=True, translate=True)
    sequence = fields.Integer(default=1)

```

## File: models\utm_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


class UtmTag(models.Model):
    """Model of categories of utm campaigns, i.e. marketing, newsletter, ..."""

    _name = 'utm.tag'
    _description = 'UTM Tag'
    _order = 'name'

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char(required=True, translate=True)
    color = fields.Integer(
        string='Color Index', default=lambda self: self._default_color(),
        help='Tag color. No color means no display in kanban to distinguish internal tags from public categorization tags.')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import utm_campaign
from . import utm_medium
from . import utm_mixin
from . import utm_source
from . import utm_stage
from . import utm_tag
from . import ir_http

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_utm_campaign_user,access_utm_campaign_user,model_utm_campaign,base.group_user,1,1,1,0
access_utm_campaign_system,utm.campaign.system,model_utm_campaign,base.group_system,1,1,1,1
access_utm_medium_user,access_utm_medium_user,model_utm_medium,base.group_user,1,1,1,0
access_utm_medium_system,utm.medium.system,model_utm_medium,base.group_system,1,1,1,1
access_utm_source_user,access_utm_source_user,model_utm_source,base.group_user,1,1,1,0
access_utm_source_system,utm.source.system,model_utm_source,base.group_system,1,1,1,1
access_utm_stage_user,mail.utm.stage,model_utm_stage,base.group_user,1,0,0,0
access_utm_stage_system,mail.utm.stage,model_utm_stage,base.group_system,1,1,1,1
access_utm_tag_user,utm.tag,model_utm_tag,base.group_user,1,0,0,0
access_utm_tag_system,utm.tag,model_utm_tag,base.group_system,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M.983 23.814a3.356 3.356 0 0 1 0-4.746L11.068 8.983a3.356 3.356 0 0 1 4.746 0l17.203 17.203a3.356 3.356 0 0 1 0 4.746L22.932 41.017a3.356 3.356 0 0 1-4.746 0L.983 23.814Z" fill="#FBB945"/><path d="M16.983 19.068a3.356 3.356 0 0 0 0 4.746l4.53 4.53 7.063-6.598 4.441 4.44a3.356 3.356 0 0 1 0 4.747L28.56 35.39l5.627 5.627a3.356 3.356 0 0 0 4.746 0l10.085-10.085a3.356 3.356 0 0 0 0-4.746L31.814 8.983a3.356 3.356 0 0 0-4.746 0L16.983 19.068Z" fill="#985184"/></svg>

```

## File: static\src\js\utm_campaign_kanban_examples.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

const exampleData = {
    ghostColumns: [_t('Ideas'), _t('Design'), _t('Review'), _t('Send'), _t('Done')],
    applyExamplesText: _t("Use This For My Campaigns"),
    allowedGroupBys: ['stage_id'],
    examples: [{
        name: _t('Creative Flow'),
        columns: [_t('Ideas'), _t('Design'), _t('Review'), _t('Send'), _t('Done')],
        description: _t("Collect ideas, design creative content and publish it once reviewed."),
    }, {
        name: _t('Event-driven Flow'),
        columns: [_t('Later'), _t('This Month'), _t('This Week'), _t('Running'), _t('Sent')],
        description: _t("Track incoming events (e.g. Christmas, Black Friday, ...) and publish timely content."),
    }, {
        name: _t('Soft-Launch Flow'),
        columns: [_t('Pre-Launch'), _t('Soft-Launch'), _t('Deploy'), _t('Report'), _t('Done')],
        description: _t("Prepare your Campaign, test it with part of your audience and deploy it fully afterwards."),
    }, {
        name: _t('Audience-driven Flow'),
        columns: [_t('Gather Data'), _t('List-Building'), _t('Copywriting'), _t('Sent')],
        description: _t("Gather data, build a recipient list and write content based on your Marketing target."),
    }, {
        name: _t('Approval-based Flow'),
        columns: [_t('To be Approved'), _t('Approved'), _t('Deployed')],
        description: _t("Prepare Campaigns and get them approved before making them go live."),
    }],
};

registry.category("kanban_examples").add("utm_campaign", exampleData);

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="view_utm_campaign_view_search">
        <field name="name">utm.campaign.view.search</field>
        <field name="model">utm.campaign</field>
        <field name="arch" type="xml">
            <search string="UTM Campaigns">
                <field name="title" string="Campaigns"/>
                <field name="tag_ids"/>
                <field name="user_id"/>
                <field name="is_auto_campaign"/>
                <filter string="My Campaigns" name="filter_assigned_to_me" domain="[('user_id', '=', uid)]"
                    help="Campaigns that are assigned to me"/>
                <separator/>
                <filter string="Archived" name="filter_inactive" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Stage" name="group_stage_id"
                        context="{'group_by': 'stage_id'}"/>
                    <filter string="Responsible" name="group_user_id"
                        context="{'group_by': 'user_id'}"/>
                    <filter string="Tags" name="group_tags_id"
                        context="{'group_by': 'tag_ids'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_form">
        <field name="name">utm.campaign.view.form</field>
        <field name="model">utm.campaign</field>
        <field name="arch" type="xml">
            <form string="UTM Campaign">
                <header>
                    <field name="stage_id" widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <div class="oe_button_box d-flex justify-content-end" name="button_box">
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <group id="top-group">
                        <field name="active" invisible="1"/>
                        <field class="text-break" name="title" string="Campaign Name" placeholder="e.g. Black Friday"/>
                        <field name="name" invisible="1"/>
                        <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                    </group>
                    <notebook>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_tree">
        <field name="name">utm.campaign.view.list</field>
        <field name="model">utm.campaign</field>
        <field name="arch" type="xml">
            <list string="UTM Campaigns" multi_edit="1" sample="1">
                <field name="title" string="Name" readonly="1"/>
                <field name="name" column_invisible="True"/>
                <field name="user_id" widget="many2one_avatar_user"/>
                <field name="stage_id" decoration-bf="1"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
            </list>
        </field>
    </record>

    <record id="utm_campaign_view_form_quick_create" model="ir.ui.view">
        <field name="name">utm.campaign.view.form.quick.create</field>
        <field name="model">utm.campaign</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name" invisible="1"/>
                    <field class="o_text_overflow" name="title" string="Campaign Name" placeholder="e.g. Black Friday"/>
                    <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                    <field name="tag_ids" widget="many2many_tags" class="o_field_highlight" options="{'color_field': 'color', 'no_create_edit': True}"/>
                </group>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_kanban">
        <field name="name">utm.campaign.view.kanban</field>
        <field name="model">utm.campaign</field>
        <field name="arch" type="xml">
            <kanban highlight_color="color" default_group_by='stage_id' class="o_utm_kanban" on_create="quick_create" quick_create_view="utm.utm_campaign_view_form_quick_create" examples="utm_campaign" sample="1">
                <field name='user_id'/>
                <field name="stage_id"/>
                <field name='active'/>
                <templates>
                    <t t-name="menu">
                        <t t-if="widget.editable">
                            <a role="menuitem" type="open" class="dropdown-item">Edit</a>
                        </t>
                        <t t-if="widget.deletable">
                            <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                        </t>
                        <a role="menuitem" class="dropdown-item o_kanban_mailing_active" name="toggle_active" type="object">
                            <t t-if="record.active.raw_value">Archive</t>
                            <t t-else="">Restore</t>
                        </a>
                        <div role="separator" class="dropdown-divider"/>
                        <field name="color" widget="kanban_color_picker"/>
                    </t>
                    <t t-name="card">
                        <field name="title" class="fw-bold fs-5"/>
                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        <ul id="o_utm_actions" class="list-group list-group-horizontal my-0"/>
                        <footer class="mt-2 mb-0 pt-0">
                            <div class="py-auto"/>
                            <field name="user_id" widget="many2one_avatar_user" class="ms-auto"/>
                        </footer>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="utm_campaign_action" model="ir.actions.act_window">
        <field name="name">Campaigns</field>
        <field name="res_model">utm.campaign</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a campaign
            </p>
            <p>
                Campaigns are used to centralize your marketing efforts and track their results.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\utm_medium_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="utm_medium_view_tree" model="ir.ui.view">
        <field name="name">utm.medium.view.list</field>
        <field name="model">utm.medium</field>
        <field name="arch" type="xml">
            <list string="Mediums" editable="bottom" sample="1">
                <field name="name" placeholder='e.g. "Email"'/>
                <field name="active" column_invisible="True"/>
            </list>
        </field>
    </record>

    <record id="utm_medium_view_form" model="ir.ui.view">
        <field name="name">utm.medium.view.form</field>
        <field name="model">utm.medium</field>
        <field name="arch" type="xml">
            <form string="Medium">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="active" widget="boolean_toggle"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="utm_medium_view_search" model="ir.ui.view">
        <field name="name">utm.medium.view.search</field>
        <field name="model">utm.medium</field>
        <field name="arch" type="xml">
            <search string="Search UTM Medium">
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>
    
    <record id="utm_medium_action" model="ir.actions.act_window">
        <field name="name">Mediums</field>
        <field name="res_model">utm.medium</field>
        <field name="view_mode">list,form</field>
        <field name="search_view_id" ref="utm_medium_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Medium
            </p><p>
                UTM Mediums track the mean that was used to attract traffic (e.g. "Website", "X", ...).
            </p>
        </field>
    </record>
</odoo>

```

## File: views\utm_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="menu_link_tracker_root"
        name="Link Tracker"
        sequence="270"
        web_icon="utm,static/description/icon.png"
        groups="base.group_no_one"/>

    <menuitem id="marketing_utm"
        name="UTMs"
        parent="menu_link_tracker_root"
        sequence="99"
        groups="base.group_no_one"/>

    <menuitem id="menu_utm_campaign_act"
        action="utm_campaign_action"
        parent="marketing_utm"
        sequence="1"
        groups="base.group_no_one"/>
    <menuitem id="menu_utm_medium"
        action="utm_medium_action"
        parent="marketing_utm"
        sequence="5"
        groups="base.group_no_one"/>
    <menuitem id="menu_utm_source"
        action="utm_source_action"
        parent="marketing_utm"
        sequence="10"
        groups="base.group_no_one"/>

</odoo>

```

## File: views\utm_source_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="utm_source_view_tree" model="ir.ui.view">
        <field name="name">utm.source.view.list</field>
        <field name="model">utm.source</field>
        <field name="arch" type="xml">
            <list string="Source" editable="bottom" sample="1">
                <field name="name" placeholder='e.g. "Christmas Mailing"'/>
            </list>
        </field>
    </record>

    <record id="utm_source_view_form" model="ir.ui.view">
        <field name="name">utm.source.view.form</field>
        <field name="model">utm.source</field>
        <field name="arch" type="xml">
            <form string="Source">
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="utm_source_action" model="ir.actions.act_window">
        <field name="name">Sources</field>
        <field name="res_model">utm.source</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Sources yet!
            </p><p>
                UTM Sources track where traffic comes from  (e.g. "May Newsletter", "", ...).
            </p>
        </field>
    </record>
</odoo>

```

## File: views\utm_stage_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="utm_stage_view_search">
        <field name="name">utm.stage.view.search</field>
        <field name="model">utm.stage</field>
        <field name="arch" type="xml">
            <search string="Stages">
                <field name="name"/>
            </search>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_stage_view_tree">
        <field name="name">utm.stage.view.list</field>
        <field name="model">utm.stage</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <list string="Stages" editable="top">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </list>
        </field>
    </record>

    <record id="utm_stage_view_form" model="ir.ui.view">
        <field name="name">utm.stage.view.form</field>
        <field name="model">utm.stage</field>
        <field name="arch" type="xml">
            <form string="Stages">
                <sheet>
                    <group>
                        <group>
                            <field name="name" placeholder='e.g. "Brainstorming"'/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="action_view_utm_stage" model="ir.actions.act_window">
        <field name="name">UTM Stages</field>
        <field name="res_model">utm.stage</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Create a stage for your campaigns
            </p><p>
            Stages allow you to organize your workflow  (e.g. plan, design, in progress, done, …).
            </p>
        </field>
    </record>
</odoo>

```

## File: views\utm_tag_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="utm_tag_view_tree" model="ir.ui.view">
        <field name="name">utm.tag.view.list</field>
        <field name="model">utm.tag</field>
        <field name="arch" type="xml">
            <list string="Campaign Tags" editable="top">
                <field name="name"/>
            </list>
        </field>
    </record>

    <record id="action_view_utm_tag" model="ir.actions.act_window">
        <field name="name">Campaign Tags</field>
        <field name="res_model">utm.tag</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Tag
            </p><p>
                Assign tags to your campaigns to organize, filter and track them.
            </p>
        </field>
    </record>
</odoo>

```

