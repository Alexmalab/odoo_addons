# Odoo Module: note

Category: Productivity/Notes

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Notes',
    'version': '1.0',
    'category': 'Productivity/Notes',
    'description': "",
    'summary': 'Organize your work with memos',
    'sequence': 260,
    'depends': [
        'mail',
    ],
    'data': [
        'security/note_security.xml',
        'security/ir.model.access.csv',
        'data/mail_activity_data.xml',
        'data/note_data.xml',
        'data/res_users_data.xml',
        'views/note_views.xml',
        ],
    'demo': [
        'data/note_demo.xml',
    ],
    'test': [
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
    'assets': {
        'web.assets_backend': [
            'note/static/src/scss/note.scss',
            'note/static/src/js/systray_activity_menu.js',
        ],
        'web.qunit_suite_tests': [
            'note/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'note/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\note.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class NoteController(http.Controller):

    @http.route('/note/new', type='json', auth='user')
    def note_new_from_systray(self, note, activity_type_id=None, date_deadline=None):
        """ Route to create note and their activity directly from the systray """
        note = request.env['note.note'].create({'memo': note})
        if date_deadline:
            note.activity_schedule(
                activity_type_id=activity_type_id or request.env['mail.activity.type'].sudo().search([('category', '=', 'reminder')], limit=1).id,
                note=note.memo,
                date_deadline=date_deadline
            )
        return note.id

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import note
```

## File: data\mail_activity_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_activity_data_reminder" model="mail.activity.type">
            <field name="name">Reminder</field>
            <field name="category">reminder</field>
            <field name="icon">fa-tasks</field>
            <field name="delay_count">0</field>
            <field name="sequence">20</field>
        </record>
    </data>
</odoo>

```

## File: data\note_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
    <record model="note.stage" id="note_stage_00">
        <field name="name">New</field>
        <field name="sequence">0</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>

    <record model="note.stage" id="note_stage_01">
        <field name="name">Meeting Minutes</field>
        <field name="sequence">5</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>

    <record model="note.stage" id="note_stage_02">
        <field name="name">Notes</field>
        <field name="sequence">10</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>

    <record model="note.stage" id="note_stage_03">
        <field name="name">Todo</field>
        <field name="sequence">50</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    </data>
</odoo>

```

## File: data\note_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
  <data noupdate="1">
    <record id="note_1" model="note.note">
      <field name="name">Customer report #349872</field>
      <field name="memo"><![CDATA[<p><b>Customer report #349872</b></p>
    <p><br/></p>
    <p>* Calendar app in Home</p>
    <p>*  The calendar module should create a menu in Home, like described above.</p>
    <p>*  This module should become a main application (in the first screen at installation)</p>
    <p>*  We should use the term Calendar, not Meeting.</p>]]>
      </field>
      <field name="user_id" ref="base.user_demo"/>
      <field name="color">2</field>
    </record>

    <record id="note_2" model="note.note">
      <field name="memo"><![CDATA[<p><b>Call Raoulette</b></p>
    <p><br/></p>
    <p>* Followed by the telephone conversation and mail about D.544.3</p>]]>
      </field>
      <field name="user_id" ref="base.user_demo"/>
    </record>

    <record id="note_4" model="note.note">
      <field name="memo"><![CDATA[<p><b>Project N.947.5</b></p>]]>
      </field>
      <field name="stage_id" ref="note_stage_02"/>
      <field name="user_id" ref="base.user_admin"/>
    </record>

    <record id="note_5" model="note.note">
      <field name="memo"><![CDATA[<p><b>Shop for family dinner</b></p>
    <p><br/></p>
    <ul class="o_checklist">
     <li id="checklist-id-1"><p>stuffed turkey</p></li>
     <li id="checklist-id-2"><p>wine</p></li>
    </ul>]]>
      </field>
      <field name="user_id" ref="base.user_demo"/>
    </record>

    <record id="note_6" model="note.note">
      <field name="memo"><![CDATA[<p><b>Idea to develop</b></p>
    <p><br/></p>
    <p>* Create a module note_pad
    it transforms the html editable memo text field into widget='pad', similar to project_pad depends on 'memo' and 'pad' modules</p>]]>
      </field>
      <field name="stage_id" ref="note_stage_02"/>
      <field name="user_id" ref="base.user_admin"/>
    </record>

    <record id="note_8" model="note.note">
      <field name="memo"><![CDATA[<p><b>New computer specs</b></p>
     <p><br/></p>
     <ul class="o_checklist">
     <li id="checklist-id-1"><p>Motherboard according to processor </p></li>
     <li id="checklist-id-2"><p>Processor need to decide </p></li>
     <li id="checklist-id-3"><p>Graphic card with great performance for games ! </p></li>
     <li id="checklist-id-4"><p>Hard drive big, for lot of internet backups </p></li>
     <li id="checklist-id-5"><p>Tower silent, better when watching films </p></li>
     <li id="checklist-id-6"><p>Blueray drive ? is it interesting yet ? </p></li>
     <li id="checklist-id-7"><p>Screen a big one, full of pixels, of course !</p></li>
    </ul>]]>
      </field>
      <field name="stage_id" ref="note_stage_03"/>
      <field name="color">3</field>
      <field name="user_id" ref="base.user_admin"/>
    </record>

    <record id="note_9" model="note.note">
      <field name="memo"><![CDATA[<p><b>Read those books</b></p>
      <p><br/></p>
      <p>* Odoo: a modern approach to integrated business management</p>
      <p>* Odoo for Retail and Industrial Management</p>]]>
      </field>
      <field name="user_id" ref="base.user_demo"/>
    </record>

    <record id="note_12" model="note.note">
      <field name="memo"><![CDATA[<p><b>Read some documentation about Odoo before diving into the code</b></p>
      <p><br/></p>
      <p>* Odoo: a modern approach to integrated business management</p>
      <p>* Odoo for Retail and Industrial Management</p>]]>
      </field>
      <field name="color">7</field>
      <field name="stage_ids" eval="[(6,0,[ref('note_stage_03')])]"/>
      <field name="user_id" ref="base.user_admin"/>
    </record>

  </data>
</odoo>

```

## File: data\res_users_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
    <function
        model="res.users"
        name="_init_data_user_note_stages"
        eval="[]"/>
    </data>
</odoo>

```

## File: models\mail_activity.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class MailActivityType(models.Model):
    _inherit = "mail.activity.type"

    category = fields.Selection(selection_add=[('reminder', 'Reminder')])


class MailActivity(models.Model):
    _inherit = "mail.activity"

    note_id = fields.Many2one('note.note', string="Related Note", ondelete='cascade')

```

## File: models\note.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import html2plaintext
from odoo.addons.web_editor.controllers.main import handle_history_divergence

class Stage(models.Model):

    _name = "note.stage"
    _description = "Note Stage"
    _order = 'sequence'

    name = fields.Char('Stage Name', translate=True, required=True)
    sequence = fields.Integer(help="Used to order the note stages", default=1)
    user_id = fields.Many2one('res.users', string='Owner', required=True, ondelete='cascade', default=lambda self: self.env.uid, help="Owner of the note stage")
    fold = fields.Boolean('Folded by Default')


class Tag(models.Model):

    _name = "note.tag"
    _description = "Note Tag"

    name = fields.Char('Tag Name', required=True, translate=True)
    color = fields.Integer('Color Index')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
    ]


class Note(models.Model):

    _name = 'note.note'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = "Note"
    _order = 'sequence, id desc'

    def _get_default_stage_id(self):
        return self.env['note.stage'].search([('user_id', '=', self.env.uid)], limit=1)

    name = fields.Text(compute='_compute_name', string='Note Summary', store=True)
    user_id = fields.Many2one('res.users', string='Owner', default=lambda self: self.env.uid)
    memo = fields.Html('Note Content')
    sequence = fields.Integer('Sequence', default=0)
    stage_id = fields.Many2one('note.stage', compute='_compute_stage_id',
        inverse='_inverse_stage_id', string='Stage', default=_get_default_stage_id)
    stage_ids = fields.Many2many('note.stage', 'note_stage_rel', 'note_id', 'stage_id',
        string='Stages of Users',  default=_get_default_stage_id)
    open = fields.Boolean(string='Active', default=True)
    date_done = fields.Date('Date done')
    color = fields.Integer(string='Color Index')
    tag_ids = fields.Many2many('note.tag', 'note_tags_rel', 'note_id', 'tag_id', string='Tags')
    # modifying property of ``mail.thread`` field
    message_partner_ids = fields.Many2many(compute_sudo=True)

    @api.depends('memo')
    def _compute_name(self):
        """ Read the first line of the memo to determine the note name """
        for note in self:
            text = html2plaintext(note.memo) if note.memo else ''
            note.name = text.strip().replace('*', '').split("\n")[0]

    def _compute_stage_id(self):
        first_user_stage = self.env['note.stage'].search([('user_id', '=', self.env.uid)], limit=1)
        for note in self:
            for stage in note.stage_ids.filtered(lambda stage: stage.user_id == self.env.user):
                note.stage_id = stage
            # note without user's stage
            if not note.stage_id:
                note.stage_id = first_user_stage

    def _inverse_stage_id(self):
        for note in self.filtered('stage_id'):
            note.stage_ids = note.stage_id + note.stage_ids.filtered(lambda stage: stage.user_id != self.env.user)

    @api.model
    def name_create(self, name):
        return self.create({'memo': name}).name_get()[0]

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if groupby and groupby[0] == "stage_id" and (len(groupby) == 1 or lazy):
            stages = self.env['note.stage'].search([('user_id', '=', self.env.uid)])
            if stages:
                # if the user has some stages
                result = []
                for stage in stages:
                    # notes by stage for stages user
                    nb_stage_counts = self.search_count(domain + [('stage_ids', '=', stage.id)])
                    result.append({
                        '__context': {'group_by': groupby[1:]},
                        '__domain': domain + [('stage_ids.id', '=', stage.id)],
                        'stage_id': (stage.id, stage.name),
                        'stage_id_count': nb_stage_counts,
                        '__count': nb_stage_counts,
                        '__fold': stage.fold,
                    })
                # note without user's stage
                nb_notes_ws = self.search_count(domain + [('stage_ids', 'not in', stages.ids)])
                if nb_notes_ws:
                    # add note to the first column if it's the first stage
                    dom_not_in = ('stage_ids', 'not in', stages.ids)
                    if result and result[0]['stage_id'][0] == stages[0].id:
                        dom_in = result[0]['__domain'].pop()
                        result[0]['__domain'] = domain + ['|', dom_in, dom_not_in]
                        result[0]['stage_id_count'] += nb_notes_ws
                        result[0]['__count'] += nb_notes_ws
                    else:
                        # add the first stage column
                        result = [{
                            '__context': {'group_by': groupby[1:]},
                            '__domain': domain + [dom_not_in],
                            'stage_id': (stages[0].id, stages[0].name),
                            'stage_id_count': nb_notes_ws,
                            '__count': nb_notes_ws,
                            '__fold': stages[0].name,
                        }] + result
            else:  # if stage_ids is empty, get note without user's stage
                nb_notes_ws = self.search_count(domain)
                if nb_notes_ws:
                    result = [{  # notes for unknown stage
                        '__context': {'group_by': groupby[1:]},
                        '__domain': domain,
                        'stage_id': False,
                        'stage_id_count': nb_notes_ws,
                        '__count': nb_notes_ws
                    }]
                else:
                    result = []
            return result
        return super(Note, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    def action_close(self):
        return self.write({'open': False, 'date_done': fields.date.today()})

    def action_open(self):
        return self.write({'open': True})

    def write(self, vals):
        if len(self) == 1:
            handle_history_divergence(self, 'memo', vals)
        return super(Note, self).write(vals)

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, models, modules, _

_logger = logging.getLogger(__name__)


class Users(models.Model):
    _name = 'res.users'
    _inherit = ['res.users']

    @api.model_create_multi
    def create(self, vals_list):
        users = super().create(vals_list)
        user_group_id = self.env['ir.model.data']._xmlid_to_res_id('base.group_user')
        # for new employee, create his own 5 base note stages
        users.filtered_domain([('groups_id', 'in', [user_group_id])])._create_note_stages()
        return users

    @api.model
    def _init_data_user_note_stages(self):
        emp_group_id = self.env.ref('base.group_user').id
        query = """
SELECT res_users.id
FROM res_users
WHERE res_users.active IS TRUE AND EXISTS (
    SELECT 1 FROM res_groups_users_rel WHERE res_groups_users_rel.gid = %s AND res_groups_users_rel.uid = res_users.id
) AND NOT EXISTS (
    SELECT 1 FROM note_stage stage WHERE stage.user_id = res_users.id
)
GROUP BY id"""
        self.env.cr.execute(query, (emp_group_id,))
        uids = [res[0] for res in self.env.cr.fetchall()]
        self.browse(uids)._create_note_stages()

    def _create_note_stages(self):
        for num in range(4):
            stage = self.env.ref('note.note_stage_%02d' % (num,), raise_if_not_found=False)
            if not stage:
                break
            for user in self:
                stage.sudo().copy(default={'user_id': user.id})
        else:
            _logger.debug("Created note columns for %s", self)

    @api.model
    def systray_get_activities(self):
        """ If user have not scheduled any note, it will not appear in activity menu.
            Making note activity always visible with number of notes on label. If there is no notes,
            activity menu not visible for note.
        """
        activities = super(Users, self).systray_get_activities()
        notes_count = self.env['note.note'].search_count([('user_id', '=', self.env.uid)])
        if notes_count:
            note_index = next((index for (index, a) in enumerate(activities) if a["model"] == "note.note"), None)
            note_label = _('Notes')
            if note_index is not None:
                activities[note_index]['name'] = note_label
            else:
                activities.append({
                    'type': 'activity',
                    'name': note_label,
                    'model': 'note.note',
                    'icon': modules.module.get_module_icon(self.env['note.note']._original_module),
                    'total_count': 0,
                    'today_count': 0,
                    'overdue_count': 0,
                    'planned_count': 0
                })
        return activities

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_activity
from . import note
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_note_stage,note.stage,model_note_stage,base.group_user,1,1,1,1
access_note_note,note.note,model_note_note,base.group_user,1,1,1,1
access_note_tag,note.tag,model_note_tag,base.group_user,1,1,1,1

```

## File: security\note_security.xml

```xml
<?xml version="1.0"?>
<odoo noupdate="1">
    <record id="note_note_rule_global" model="ir.rule">
        <field name="name">Only followers can access a sticky notes</field>
        <field name="model_id" ref="model_note_note"/>
        <field name="domain_force">['|', ('user_id', '=', user.id), ('message_partner_ids', '=', user.partner_id.id)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="note_note_create_unlink_global" model="ir.rule">
        <field name="name">note: create / unlink: responsible</field>
        <field name="model_id" ref="model_note_note"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
        <field name="perm_write" eval="False"/>
        <field name="perm_read" eval="False"/>
    </record>

    <record id="note_stage_rule_global" model="ir.rule">
        <field name="name">Each user have his stage name</field>
        <field name="model_id" ref="model_note_stage"/>
        <field name="domain_force">['|',('user_id','=',False),('user_id','=',user.id)]</field>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M56.342 31.863l-1.875 1.876a.489.489 0 0 1-.692 0l-4.517-4.516a.489.489 0 0 1 0-.692l1.876-1.876c.761-.76 1.998-.76 2.763 0l2.445 2.446a1.95 1.95 0 0 1 0 2.762zM15 47v-2h17v2H15zm0-4.915v-2h19v2H15zm0-9v-2h22v2H15zm0 5v-2h14v2H15zm0-9v-2h28v2H15zm32.647 1.057a.494.494 0 0 1 .696 0l4.516 4.516c.192.192.192.5 0 .692L42.174 46.034l-4.944.867a.978.978 0 0 1-1.13-1.131l.862-4.944 10.685-10.684zm-6.515 9.769a.567.567 0 0 0 .806 0l6.266-6.266a.567.567 0 0 0 0-.805.567.567 0 0 0-.805 0l-6.267 6.265a.567.567 0 0 0 0 .806zm-1.468 3.422V41.38h-1.477l-.46 2.624 1.265 1.265 2.625-.46v-1.476h-1.953z"/><path id="e" d="M56.342 29.863l-1.875 1.876a.489.489 0 0 1-.692 0l-4.517-4.516a.489.489 0 0 1 0-.692l1.876-1.876c.761-.76 1.998-.76 2.763 0l2.445 2.446a1.95 1.95 0 0 1 0 2.762zM15 45v-2h17v2H15zm0-4.915v-2h19v2H15zm0-9v-2h22v2H15zm0 5v-2h14v2H15zm0-9v-2h28v2H15zm32.647 1.057a.494.494 0 0 1 .696 0l4.516 4.516c.192.192.192.5 0 .692L42.174 44.034l-4.944.867a.978.978 0 0 1-1.13-1.131l.862-4.944 10.685-10.684zm-6.515 9.769a.567.567 0 0 0 .806 0l6.266-6.266a.567.567 0 0 0 0-.805.567.567 0 0 0-.805 0l-6.267 6.265a.567.567 0 0 0 0 .806zm-1.468 3.422V39.38h-1.477l-.46 2.624 1.265 1.265 2.625-.46v-1.476h-1.953z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V41.668l15.07-16.471h27.883v1.843L31.949 38.148l1.998 1.89-4.983 4.239h1.513l21.088-19.832L56 30 25.947 68.643 4 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\systray_activity_menu.js

```javascript
/** @odoo-module **/

import ActivityMenu from '@mail/js/systray/systray_activity_menu';

import { _t } from 'web.core';
import datepicker from 'web.datepicker';
const urlRegExp = /http(s)?:\/\/(www\.)?[a-zA-Z0-9@:%_+~#=~#?&/=\-;!.]{3,2000}/g;

ActivityMenu.include({
    events: _.extend({}, ActivityMenu.prototype.events, {
        'click .o_note_show': '_onAddNoteClick',
        'click .o_note_save': '_onNoteSaveClick',
        'click .o_note_set_datetime': '_onNoteDateTimeSetClick',
        'keydown input.o_note_input': '_onNoteInputKeyDown',
        'click .o_note': '_onNewNoteClick',
    }),
    //--------------------------------------------------
    // Private
    //--------------------------------------------------
    /**
     * Moving notes at first place
     * @override
     */
    _getActivityData: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var reminderIndex = _.findIndex(self.activities, function (val) {
                return val.model === 'note.note';
            });
            if (reminderIndex > 0) {
                self.activities.splice(0, 0, self.activities.splice(reminderIndex, 1)[0]);
            }
        });
    },
    /**
     * Save the note to database using datepicker date and field as note
     * By default, when no datetime is set, it uses the current datetime.
     *
     * @private
     */
    _saveNote: function () {
        var note = this.$('.o_note_input').val().replace(urlRegExp, '<a href="$&">$&</a>').trim();
        if (! note) {
            return;
        }
        var params = {'note': note};
        var noteDateTime = this.noteDateTimeWidget.getValue();
        if (noteDateTime) {
            params = _.extend(params, {'date_deadline': noteDateTime});
        } else {
            params = _.extend(params, {'date_deadline': moment()});
        }
        this.$('.o_note_show').removeClass('d-none');
        this.$('.o_note').addClass('d-none');
        this._rpc({
            route: '/note/new',
            params: params,
        }).then(this._updateActivityPreview.bind(this));
    },
    //-----------------------------------------
    // Handlers
    //-----------------------------------------
    /**
     * @override
     */
    _onActivityFilterClick: function (ev) {
        var $el = $(ev.currentTarget);
        if (!$el.hasClass("o_note")) {
            this._super.apply(this, arguments);
        }
    },
    /**
     * When add new note button clicked, toggling quick note create view inside
     * Systray activity view
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onAddNoteClick: function (ev) {
        var self = this;
        ev.stopPropagation();
        if (!this.noteDateTimeWidget){
            this.noteDateTimeWidget = new datepicker.DateWidget(this, {useCurrent: true});
        }
        this.noteDateTimeWidget.appendTo(this.$('.o_note_datetime')).then(function() {
            self.noteDateTimeWidget.$input.attr('placeholder', _t("Today"));
            self.noteDateTimeWidget.setValue(false);
            self.$('.o_note_show, .o_note').toggleClass('d-none');
            self.$('.o_note_input').val('').focus();
        });
    },
    /**
     * When focusing on input for new quick note systerm tray must be open.
     * Preventing to close
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onNewNoteClick: function (ev) {
        ev.stopPropagation();
    },
    /**
     * Opens datetime picker for note.
     * Quick FIX due to no option for set custom icon instead of caret in datepicker.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onNoteDateTimeSetClick: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this.noteDateTimeWidget.$input.click();
    },
    /**
     * Saving note (quick create) and updating activity preview
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onNoteSaveClick: function (ev) {
        this._saveNote();
    },
    /**
     * Handling Enter key for quick create note.
     *
     * @private
     * @param {KeyboardEvent} ev
     */
    _onNoteInputKeyDown: function (ev) {
        if (ev.which === $.ui.keyCode.ENTER) {
            this._saveNote();
        }
    },
});

```

## File: static\src\xml\systray.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-extend="mail.systray.ActivityMenu.Previews">
        <t t-jquery="t[t-foreach*='activities'][t-as*='activity']" t-operation="after">
            <div class="o_note_show">
                <a role="button" class="btn btn-block text-center">Add new note</a>
            </div>
            <div class="o_note o_mail_preview d-none">
                <div class="o_mail_preview_image o_mail_preview_app">
                    <img src="/note/static/description/icon.png" alt="Channel"/>
                </div>
                <div class="o_preview_info">
                    <div class="o_preview_title">
                        <span class="o_preview_name"><strong>Add a note</strong></span>
                        <div class="o_note_datetime"/>
                        <span class="ml4">
                            <a class="o_note_set_datetime">
                                <span class="fa fa-calendar" role="img" aria-label="Set date and time" title="Set date and time"/>
                            </a>
                        </span>
                    </div>
                    <div class="o_note_input_box">
                        <p><input class="o_note_input" type="text" placeholder="Remember..." /></p>
                        <span class="ml8 mr4">
                            <a class="o_note_save">SAVE</a>
                        </span>
                    </div>
                </div>
            </div>
        </t>
    </t>
</templates>

```

## File: views\note_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- note Stage Form View -->
    <record id="view_note_stage_form" model="ir.ui.view">
      <field name="name">note.stage.form</field>
      <field name="model">note.stage</field>
      <field name="arch" type="xml">
        <form string="Stage of Notes">
          <group>
            <field name="name"/>
            <field name="fold"/>
          </group>
        </form>
      </field>
    </record>

    <!-- note Stage Tree View -->
    <record id="view_note_stage_tree" model="ir.ui.view">
      <field name="name">note.stage.tree</field>
      <field name="model">note.stage</field>
      <field name="field_parent"></field>
      <field name="arch" type="xml">
        <tree string="Stages of Notes" editable="bottom">
            <field name="sequence" widget="handle"/>
            <field name="name"/>
            <field name="fold"/>
        </tree>
      </field>
    </record>

    <!-- note Stage Action -->
    <record id="action_note_stage" model="ir.actions.act_window">
        <field name="name">Stages</field>
        <field name="res_model">note.stage</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('user_id','=',uid)]</field>
    </record>

    <!-- note Tag Form View -->
    <record id="note_tag_view_form" model="ir.ui.view">
      <field name="name">note.tag.form</field>
      <field name="model">note.tag</field>
      <field name="arch" type="xml">
        <form string="Tags">
          <group>
            <field name="name"/>
          </group>
        </form>
      </field>
    </record>

    <!-- note Tag Tree View -->
    <record id="note_tag_view_tree" model="ir.ui.view">
      <field name="name">note.tag.tree</field>
      <field name="model">note.tag</field>
      <field name="arch" type="xml">
        <tree string="Tags" editable="bottom">
            <field name="name"/>
        </tree>
      </field>
    </record>

    <!-- note Tag Action -->
    <record id="note_tag_action" model="ir.actions.act_window">
      <field name="name">Tags</field>
      <field name="res_model">note.tag</field>
      <field name="view_mode">tree,form</field>
      <field name="help" type="html">
        <p class="o_view_nocontent_smiling_face">
          Add a new tag
        </p>
      </field>
    </record>

    <!-- New note Kanban View -->
    <record id="view_note_note_kanban" model="ir.ui.view">
      <field name="name">note.note.kanban</field>
      <field name="model">note.note</field>
      <field name="arch" type="xml">
        <kanban default_group_by="stage_id" class="oe_notes oe_kanban_quickcreate_textarea o_kanban_small_column">
          <field name="color"/>
          <field name="sequence"/>
          <field name="name"/>
          <field name="stage_id"/>
          <field name="open"/>
          <field name="memo"/>
          <field name="date_done"/>
          <field name="message_partner_ids"/>
          <field name="activity_ids" />
          <field name="activity_state" />
          <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
          <templates>
            <t t-name="kanban-box">
              <t t-set="noteHasFollowers" t-value="record.message_partner_ids.raw_value.length &gt; 1"/>

              <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''} oe_kanban_global_click_edit oe_semantic_html_override oe_kanban_card">

                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>

                <div class="o_dropdown_kanban dropdown">
                    <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                        <span class="fa fa-ellipsis-v"/>
                    </a>
                    <div class="dropdown-menu" role="menu">
                        <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                    </div>
                </div>
                <div t-attf-class="d-flex #{noteHasFollowers ? 'flex-wrap' : 'flex-nowrap'}">
                  <div class="d-flex flex-grow-1">
                    <span class="mr-2">
                      <a name="action_close" type="object" t-if="record.open.raw_value">
                        <i class="fa fa-check" role="img" aria-label="Opened" title="Opened"/>
                      </a>
                      <a name="action_open" type="object" t-if="!record.open.raw_value">
                        <i class="fa fa-undo" role="img" aria-label="Closed" title="Closed"/>
                      </a>
                    </span>

                    <span t-attf-class="oe_kanban_content flex-grow-1 #{record.open.raw_value ? '' : 'note_text_line_through'}">
                      <!-- title -->
                      <field name="name"/>
                    </span>
                  </div>
                  <div t-attf-class="d-flex #{noteHasFollowers ? 'w-100 align-items-center justify-content-end' : 'align-items-end'}">
                    <div t-if="noteHasFollowers" class="d-flex align-items-center mr-2 mt-2">
                      <t t-foreach="record.message_partner_ids.raw_value" t-as="follower">
                        <img
                            t-if="follower_index &lt; 5"
                            t-att-src="kanban_image('res.partner', 'avatar_128', follower)"
                            class="oe_kanban_avatar o_image_24_cover rounded-circle border border-white bg-white ml-n2"
                            t-att-data-member_id="follower"
                            alt="Follower"/>
                        <small
                            t-if="follower_index == 5"
                            t-esc="'+' + (follower_size - 5)"
                            class="text-info font-weight-bold ml-1"/>
                      </t>
                    </div>
                    <field name="activity_ids" widget="kanban_activity" />
                  </div>
                </div>
              </div>
            </t>
          </templates>
        </kanban>
      </field>
    </record>

    <!-- New note Tree View -->
    <record id="view_note_note_tree" model="ir.ui.view">
      <field name="name">note.note.tree</field>
      <field name="model">note.note</field>
      <field name="arch" type="xml">
        <tree string="Stages">
          <field name="name"/>
          <field name="open"/>
          <field name="stage_id"/>
          <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
          <field name="activity_ids" widget="list_activity" optional="show"/>
        </tree>
      </field>
    </record>

    <!-- New note Form View -->
    <record id="view_note_note_form" model="ir.ui.view">
        <field name="name">note.note.form</field>
        <field name="model">note.note</field>
        <field name="arch" type="xml">
            <form string="Note" class="oe_form_nomargin o_note_form_view">
                <header>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" placeholder="Tags"/>
                    <field name="stage_id" domain="[('user_id','=',uid)]" widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                  <field name="memo" type="html" class="oe_memo" default_focus="1" options="{'resizable': false, 'collaborative': true}"/>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <!-- Search note  -->
    <record id="view_note_note_filter" model="ir.ui.view">
      <field name="name">note.note.search</field>
      <field name="model">note.note</field>
      <field name="arch" type="xml">
        <search string="Notes">
          <field name="memo" string="Note"/>
          <field name="tag_ids"/>
          <filter name="open_true" string="Active" domain="[('open', '=', True)]"/>
          <filter name="open_false" string="Archive" domain="[('open', '=', False)]"/>
          <filter invisible="1" string="Late Activities" name="activities_overdue"
                  domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                  help="Show all records which has next action date is before today"/>
          <filter invisible="1" string="Today Activities" name="activities_today"
                  domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
          <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                        ]"/>
          <group expand="0" string="Group By">
            <filter string="Stage" name="stage" help="By sticky note Category" context="{'group_by':'stage_id'}"/>
          </group>
        </search>
      </field>
    </record>

    <!-- Action -->
    <record id="action_note_note" model="ir.actions.act_window">
      <field name="name">Notes</field>
      <field name="res_model">note.note</field>
      <field name="view_mode">kanban,tree,form,activity</field>
      <field name="search_view_id" ref="view_note_note_filter"/>
      <field name="context">{}</field>
      <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new personal note
          </p><p>
            Notes are private, unless you share them by inviting follower on a note.
            (Useful for meeting minutes).
          </p>
        </field>
    </record>

    <menuitem
      id="menu_note_notes"
      name="Notes"
      sequence="15"
      action="note.action_note_note"
      web_icon="note,static/description/icon.png">
        <menuitem
          id="menu_note_configuration"
          name="Configuration"
          sequence="100"
          groups="base.group_no_one">
            <menuitem
              id="menu_notes_stage"
              name="Stages"
              action="note.action_note_stage"
              sequence="21"/>
            <menuitem
              id="notes_tag_menu"
              action="note_tag_action"
              sequence="22"/>
        </menuitem>
    </menuitem>

</odoo>

```

