# Odoo Module: website_hr_recruitment

Category: Website/Website

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
    'name': 'Online Jobs',
    'category': 'Website/Website',
    'sequence': 310,
    'version': '1.1',
    'summary': 'Manage your online hiring process',
    'description': "This module allows to publish your available job positions on your website and keep track of application submissions easily. It comes as an add-on of *Recruitment* app.",
    'depends': ['hr_recruitment', 'website_mail'],
    'data': [
        'security/ir.model.access.csv',
        'security/website_hr_recruitment_security.xml',
        'data/config_data.xml',
        'views/website_hr_recruitment_templates.xml',
        'views/hr_recruitment_views.xml',
        'views/hr_job_views.xml',
        'views/website_pages_views.xml',
        'views/snippets.xml',
    ],
    'demo': [
        'data/hr_job_demo.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': ['hr_recruitment', 'website_mail'],
    'assets': {
        'web.assets_frontend': [
            'website_hr_recruitment/static/src/scss/**/*',
            'website_hr_recruitment/static/src/js/website_hr_applicant_form.js',
        ],
        'web.assets_backend': [
            'website_hr_recruitment/static/src/js/widgets/copy_link_menuitem.js',
            'website_hr_recruitment/static/src/js/widgets/copy_link_menuitem.xml',
            'website_hr_recruitment/static/src/fields/**/*',
        ],
        'website.assets_wysiwyg': [
            'website_hr_recruitment/static/src/js/website_hr_recruitment_editor.js',
        ],
        'website.assets_editor': [
            'website_hr_recruitment/static/src/js/systray_items/new_content.js',
        ],
        'web.assets_tests': [
            'website_hr_recruitment/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import warnings
from datetime import datetime
from dateutil.relativedelta import relativedelta
from operator import itemgetter
from werkzeug.urls import url_encode

from odoo import http, _
from odoo.addons.website.controllers.form import WebsiteForm
from odoo.osv.expression import AND
from odoo.http import request
from odoo.tools import email_normalize
from odoo.tools.misc import groupby


class WebsiteHrRecruitment(WebsiteForm):
    _jobs_per_page = 12

    def sitemap_jobs(env, rule, qs):
        if not qs or qs.lower() in '/jobs':
            yield {'loc': '/jobs'}

    @http.route([
        '/jobs',
        '/jobs/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=sitemap_jobs)
    def jobs(self, country_id=None, department_id=None, office_id=None, contract_type_id=None,
             is_remote=False, is_other_department=False, is_untyped=None, page=1, search=None, **kwargs):
        env = request.env(context=dict(request.env.context, show_address=True, no_tag_br=True))

        Country = env['res.country']
        Jobs = env['hr.job']
        Department = env['hr.department']

        country = Country.browse(int(country_id)) if country_id else None
        department = Department.browse(int(department_id)) if department_id else None
        office_id = int(office_id) if office_id else None
        contract_type_id = int(contract_type_id) if contract_type_id else None

        # Default search by user country
        if not (country or department or office_id or contract_type_id or kwargs.get('all_countries')):
            if request.geoip.country_code:
                countries_ = Country.search([('code', '=', request.geoip.country_code)])
                country = countries_[0] if countries_ else None
                if country:
                    country_count = Jobs.search_count(AND([
                        request.website.website_domain(),
                        [('address_id.country_id', '=', country.id)]
                    ]))
                    if not country_count:
                        country = False

        options = {
            'displayDescription': True,
            'allowFuzzy': not request.params.get('noFuzzy'),
            'country_id': country.id if country else None,
            'department_id': department.id if department else None,
            'office_id': office_id,
            'contract_type_id': contract_type_id,
            'is_remote': is_remote,
            'is_other_department': is_other_department,
            'is_untyped': is_untyped,
        }
        total, details, fuzzy_search_term = request.website._search_with_fuzzy("jobs", search,
            limit=1000, order="is_published desc, sequence, no_of_recruitment desc", options=options)
        # Browse jobs as superuser, because address is restricted
        jobs = details[0].get('results', Jobs).sudo()

        def sort(records_list, field_name):
            """ Sort records in the given collection according to the given
            field name, alphabetically. None values instead of records are
            placed at the end.

            :param list records_list: collection of records or None values
            :param str field_name: field on which to sort
            :return: sorted list
            """
            return sorted(
                records_list,
                key=lambda item: (item is None, item.sudo()[field_name] if item and item.sudo()[field_name] else ''),
            )

        # Countries
        if country or is_remote:
            cross_country_options = options.copy()
            cross_country_options.update({
                'allowFuzzy': False,
                'country_id': None,
                'is_remote': False,
            })
            cross_country_total, cross_country_details, _ = request.website._search_with_fuzzy("jobs",
                fuzzy_search_term or search, limit=1000, order="is_published desc, sequence, no_of_recruitment desc",
                options=cross_country_options)
            # Browse jobs as superuser, because address is restricted
            cross_country_jobs = cross_country_details[0].get('results', Jobs).sudo()
        else:
            cross_country_total = total
            cross_country_jobs = jobs
        country_offices = set(j.address_id or None for j in cross_country_jobs)
        countries = sort(set(o and o.country_id or None for o in country_offices), 'name')
        count_per_country = {'all': cross_country_total}
        for c, jobs_list in groupby(cross_country_jobs, lambda job: job.address_id.country_id):
            count_per_country[c] = len(jobs_list)
        count_remote = len(cross_country_jobs.filtered(lambda job: not job.address_id))
        if count_remote:
            count_per_country[None] = count_remote

        # Departments
        if department or is_other_department:
            cross_department_options = options.copy()
            cross_department_options.update({
                'allowFuzzy': False,
                'department_id': None,
                'is_other_department': False,
            })
            cross_department_total, cross_department_details, _ = request.website._search_with_fuzzy("jobs",
                fuzzy_search_term or search, limit=1000, order="is_published desc, sequence, no_of_recruitment desc",
                options=cross_department_options)
            cross_department_jobs = cross_department_details[0].get('results', Jobs)
        else:
            cross_department_total = total
            cross_department_jobs = jobs
        departments = sort(set(j.department_id or None for j in cross_department_jobs), 'name')
        count_per_department = {'all': cross_department_total}
        for d, jobs_list in groupby(cross_department_jobs, lambda job: job.department_id):
            count_per_department[d] = len(jobs_list)
        count_other_department = len(cross_department_jobs.filtered(lambda job: not job.department_id))
        if count_other_department:
            count_per_department[None] = count_other_department

        # Offices
        if office_id or is_remote:
            cross_office_options = options.copy()
            cross_office_options.update({
                'allowFuzzy': False,
                'office_id': None,
                'is_remote': False,
            })
            cross_office_total, cross_office_details, _ = request.website._search_with_fuzzy("jobs",
                fuzzy_search_term or search, limit=1000, order="is_published desc, sequence, no_of_recruitment desc",
                options=cross_office_options)
            # Browse jobs as superuser, because address is restricted
            cross_office_jobs = cross_office_details[0].get('results', Jobs).sudo()
        else:
            cross_office_total = total
            cross_office_jobs = jobs
        offices = sort(set(j.address_id or None for j in cross_office_jobs), 'city')
        count_per_office = {'all': cross_office_total}
        for o, jobs_list in groupby(cross_office_jobs, lambda job: job.address_id):
            count_per_office[o] = len(jobs_list)
        count_remote = len(cross_office_jobs.filtered(lambda job: not job.address_id))
        if count_remote:
            count_per_office[None] = count_remote

        # Employment types
        if contract_type_id or is_untyped:
            cross_type_options = options.copy()
            cross_type_options.update({
                'allowFuzzy': False,
                'contract_type_id': None,
                'is_untyped': False,
            })
            cross_type_total, cross_type_details, _ = request.website._search_with_fuzzy("jobs",
                fuzzy_search_term or search, limit=1000, order="is_published desc, sequence, no_of_recruitment desc",
                options=cross_type_options)
            cross_type_jobs = cross_type_details[0].get('results', Jobs)
        else:
            cross_type_total = total
            cross_type_jobs = jobs
        employment_types = sort(set(j.contract_type_id for j in jobs if j.contract_type_id), 'name')
        count_per_employment_type = {'all': cross_type_total}
        for t, jobs_list in groupby(cross_type_jobs, lambda job: job.contract_type_id):
            count_per_employment_type[t] = len(jobs_list)
        count_untyped = len(cross_type_jobs.filtered(lambda job: not job.contract_type_id))
        if count_untyped:
            count_per_employment_type[None] = count_untyped

        pager = request.website.pager(
            url=request.httprequest.path.partition('/page/')[0],
            url_args=request.httprequest.args,
            total=total,
            page=page,
            step=self._jobs_per_page,
        )
        offset = pager['offset']
        jobs = jobs[offset:offset + self._jobs_per_page]

        office = env['res.partner'].browse(int(office_id)) if office_id else None
        contract_type = env['hr.contract.type'].browse(int(contract_type_id)) if contract_type_id else None

        # Render page
        return request.render("website_hr_recruitment.index", {
            'jobs': jobs,
            'countries': countries,
            'departments': departments,
            'offices': offices,
            'employment_types': employment_types,
            'country_id': country,
            'department_id': department,
            'office_id': office,
            'contract_type_id': contract_type,
            'is_remote': is_remote,
            'is_other_department': is_other_department,
            'is_untyped': is_untyped,
            'pager': pager,
            'search': fuzzy_search_term or search,
            'search_count': total,
            'original_search': fuzzy_search_term and search,
            'count_per_country': count_per_country,
            'count_per_department': count_per_department,
            'count_per_office': count_per_office,
            'count_per_employment_type': count_per_employment_type,
        })

    @http.route('/jobs/add', type='json', auth="user", website=True)
    def jobs_add(self, **kwargs):
        # avoid branding of website_description by setting rendering_bundle in context
        job = request.env['hr.job'].with_context(rendering_bundle=True).create({
            'name': _('Job Title'),
        })
        return f"/jobs/{request.env['ir.http']._slug(job)}"

    @http.route('''/jobs/detail/<model("hr.job"):job>''', type='http', auth="public", website=True, sitemap=True)
    def jobs_detail(self, job, **kwargs):
        redirect_url = f"/jobs/{request.env['ir.http']._slug(job)}"
        return request.redirect(redirect_url, code=301)

    @http.route('''/jobs/<model("hr.job"):job>''', type='http', auth="public", website=True, sitemap=True)
    def job(self, job, **kwargs):
        return request.render("website_hr_recruitment.detail", {
            'job': job,
            'main_object': job,
        })

    @http.route('''/jobs/apply/<model("hr.job"):job>''', type='http', auth="public", website=True, sitemap=True)
    def jobs_apply(self, job, **kwargs):
        error = {}
        default = {}
        if 'website_hr_recruitment_error' in request.session:
            error = request.session.pop('website_hr_recruitment_error')
            default = request.session.pop('website_hr_recruitment_default')
        return request.render("website_hr_recruitment.apply", {
            'job': job,
            'error': error,
            'default': default,
        })

    # Compatibility routes

    @http.route([
        '/jobs/country/<model("res.country"):country>',
        '/jobs/department/<model("hr.department"):department>',
        '/jobs/country/<model("res.country"):country>/department/<model("hr.department"):department>',
        '/jobs/office/<int:office_id>',
        '/jobs/country/<model("res.country"):country>/office/<int:office_id>',
        '/jobs/department/<model("hr.department"):department>/office/<int:office_id>',
        '/jobs/country/<model("res.country"):country>/department/<model("hr.department"):department>/office/<int:office_id>',
        '/jobs/employment_type/<int:contract_type_id>',
        '/jobs/country/<model("res.country"):country>/employment_type/<int:contract_type_id>',
        '/jobs/department/<model("hr.department"):department>/employment_type/<int:contract_type_id>',
        '/jobs/office/<int:office_id>/employment_type/<int:contract_type_id>',
        '/jobs/country/<model("res.country"):country>/department/<model("hr.department"):department>/employment_type/<int:contract_type_id>',
        '/jobs/country/<model("res.country"):country>/office/<int:office_id>/employment_type/<int:contract_type_id>',
        '/jobs/department/<model("hr.department"):department>/office/<int:office_id>/employment_type/<int:contract_type_id>',
        '/jobs/country/<model("res.country"):country>/department/<model("hr.department"):department>/office/<int:office_id>/employment_type/<int:contract_type_id>',
    ], type='http', auth="public", website=True, sitemap=False)
    def jobs_compatibility(self, country=None, department=None, office_id=None, contract_type_id=None, **kwargs):
        """
        Deprecated since Odoo 16.3: those routes are kept by compatibility.
        They should not be used in Odoo code anymore.
        """
        warnings.warn(
            "This route is deprecated since Odoo 16.3: the jobs list is now available at /jobs or /jobs/page/XXX",
            DeprecationWarning
        )
        url_params = {
            'country_id': country and country.id,
            'department_id': department and department.id,
            'office_id': office_id,
            'contract_type_id': contract_type_id,
            **kwargs,
        }
        return request.redirect(
            '/jobs?%s' % url_encode(url_params),
            code=301,
        )

    @http.route('/website_hr_recruitment/check_recent_application', type='json', auth="public", website=True)
    def check_recent_application(self, field, value, job_id):
        def refused_applicants_condition(applicant):
            return not applicant.active \
                and applicant.job_id.id == int(job_id) \
                and applicant.create_date >= (datetime.now() - relativedelta(months=6))

        field_domain = {
            'name': [('partner_name', '=ilike', value)],
            'email': [('email_normalized', '=', email_normalize(value))],
            'phone': [('partner_phone', '=', value)],
            'linkedin': [('linkedin_profile', '=ilike', value)],
        }.get(field, [])

        applications_by_status = http.request.env['hr.applicant'].sudo().search(AND([
            field_domain,
            [
                ('job_id.website_id', 'in', [http.request.website.id, False]),
                '|',
                    ('application_status', '=', 'ongoing'),
                    '&',
                        ('application_status', '=', 'refused'),
                        ('active', '=', False),
            ]
        ]), order='create_date DESC').grouped('application_status')
        refused_applicants = applications_by_status.get('refused', http.request.env['hr.applicant'])
        if any(applicant for applicant in refused_applicants if refused_applicants_condition(applicant)):
            return {
                'message':  _(
                    'We\'ve found a previous closed application in our system within the last 6 months.'
                    ' Please consider before applying in order not to duplicate efforts.'
                )
            }

        if 'ongoing' not in applications_by_status:
            return {'message': None}

        ongoing_application = applications_by_status.get('ongoing')[0]
        if ongoing_application.job_id.id == int(job_id):
            recruiter_contact = "" if not ongoing_application.user_id else _(
                ' In case of issue, contact %(contact_infos)s',
                contact_infos=", ".join(
                    [value for value in itemgetter('name', 'email', 'phone')(ongoing_application.user_id) if value]
                ))
            return {
                'message':  _(
                    'An application already exists for %(value)s.'
                    ' Duplicates might be rejected. %(recruiter_contact)s',
                    value=value,
                    recruiter_contact=recruiter_contact
                )
            }

        return {
            'message':  _(
                'We found a recent application with a similar name, email, phone number.'
                ' You can continue if it\'s not a mistake.'
            )
        }

    def extract_data(self, model, values):
        candidate = request.env['hr.candidate']
        if model.sudo().model == 'hr.applicant':
            # pop the fields since there are only useful to generate a candidate record
            partner_name = values.pop('partner_name')
            partner_phone = values.pop('partner_phone', None)
            partner_email = values.pop('email_from', None)

            company_id = (
                request.env["hr.department"]
                .sudo()
                .search([("id", "=", values.get("department_id"))])
                .company_id.id
                or request.env["hr.job"]
                .sudo()
                .search([("id", "=", values.get("job_id"))])
                .company_id.id
            )
            if partner_phone and partner_email:
                candidate = request.env['hr.candidate'].sudo().search([
                    ('email_from', '=', partner_email),
                    ('partner_phone', '=', partner_phone),
                ], limit=1)
            if not candidate:
                candidate = request.env['hr.candidate'].sudo().create({
                    'partner_name': partner_name,
                    'email_from': partner_email,
                    'partner_phone': partner_phone,
                    'company_id': company_id,
                })
        data = super().extract_data(model, values)
        if candidate:
            data['record']['candidate_id'] = candidate.id
        return data

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\config_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="action_open_website" model="ir.actions.act_url">
            <field name="name">Website Recruitment Form</field>
            <field name="target">self</field>
            <field name="url">/jobs</field>
        </record>
        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_open_website"/>
            <field name="state">open</field>
        </record>
        <record id="website_menu_jobs" model="website.menu">
            <field name="name">Jobs</field>
            <field name="url">/jobs</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence">59</field>
        </record>
    </data>
    <data>
        <record id="hr_recruitment.model_hr_applicant" model="ir.model">
            <field name="website_form_key">apply_job</field>
            <field name="website_form_access">True</field>
            <field name="website_form_label">Apply for a Job</field>
        </record>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>hr.applicant</value>
            <value eval="[
                'email_from',
                'partner_name',
                'partner_phone',
                'job_id',
                'department_id',
                'linkedin_profile',
                'applicant_properties',
            ]"/>
        </function>
    </data>
</odoo>

```

## File: data\hr_job_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="hr.job_marketing" model="hr.job">
        <field name="is_published">True</field>
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40" itemprop="description">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_developer" model="hr.job">
        <field name="is_published">True</field>
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_consultant" model="hr.job">
        <field name="is_published">True</field>
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_ceo" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_cto" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_trainee" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_hrm" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt0">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32" itemprop="description">
                            <p class="lead">
                                As an employee of our company, you will <b>collaborate with each department to create and deploy
                                disruptive products.</b> Come work at a growing company that offers great benefits with opportunities to
                                moving forward and learn alongside accomplished leaders. We're seeking an experienced and outstanding member of staff.
                                <br/><br/>
                                This position is both <b>creative and rigorous</b> by nature you need to think outside the box.
                                We expect the candidate to be proactive and have a "get it done" spirit. To be successful,
                                you will have solid solving problem skills.
                            </p>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb32">
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Customer Relationship</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Personal Evolution</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Autonomy</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Administrative Work</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                        <i class="fa fa-star-o"/>
                                    </span>
                                </div>
                            </div>
                            <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                                <h6 class="s_rating_title">Technical Expertise</h6>
                                <div class="s_rating_icons o_not_editable">
                                    <span class="s_rating_active_icons text-primary">
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                        <i class="fa fa-star"/>
                                    </span>
                                    <span class="s_rating_inactive_icons text-primary">
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Responsabilities, Must Have and Nice to have -->
            <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Responsibilities</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Lead the entire sales cycle</li>
                                    <li class="list-group-item">Achieve monthly sales objectives</li>
                                    <li class="list-group-item">Qualify the customer needs</li>
                                    <li class="list-group-item">Negotiate and contract</li>
                                    <li class="list-group-item">Master demos of our software</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Must Have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Bachelor Degree or Higher</li>
                                    <li class="list-group-item">Passion for software products</li>
                                    <li class="list-group-item">Perfect written English</li>
                                    <li class="list-group-item">Highly creative and autonomous</li>
                                    <li class="list-group-item">Valid work permit for Belgium</li>
                                </ul>
                            </div>
                        </div>
                        <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                            <div class="card text-bg-primary">
                                <h4 class="card-header">Nice to have</h4>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item">Experience in writing online content</li>
                                    <li class="list-group-item">Additional languages</li>
                                    <li class="list-group-item">Google Adwords experience</li>
                                    <li class="list-group-item">Strong analytical skills</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What's great -->
            <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
                <div class="container">
                    <div class="row align-items-center">
                        <div class="col-sm-7 pb40">
                            <h2>What's great in the job?</h2>
                            <br/>
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                            <div class="card shadow text-center">
                                <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                                <div class="card-body p-0 pt-3">
                                    <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                                    <p>Discover our products.</p>
                                    <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <div class="col-lg-12">
                        <h3>What We Offer</h3>
                        <p class="lead">
                            Each employee has a chance to see the impact of his work.
                            You can make a real contribution to the success of the company.
                            <br/>
                            Several activities are often organized all over the year, such as weekly
                            sports sessions, team building events, monthly drink, and much more.
                        </p>
                    </div>
                    <div class="row">
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3">
                            <div class="s_hr pt-4 pb32">
                                <hr class="w-100 mx-auto"/>
                            </div>
                            <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                            <h3 class="h5-fs">Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-8">
                            <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-4">
                            <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-3">
                            <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                        <div class="col-lg-6">
                            <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

</odoo>

```

## File: migrations\17.0.1.1\pre-migrate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

def migrate(cr, version):
    # Remove the csrf_token and its surrounding div from the
    # website_hr_recruitment.apply (it was set with a t-att- breaking the
    # possibility to properly edit the form, and it was actually useless).
    cr.execute(r"""
        UPDATE ir_ui_view
        SET arch_db = REGEXP_REPLACE(arch_db::text, '<div[^<]*>[^<]*<input[^>]+id=\\"csrf_token\\"[^>]*/>[^<]*</div>', '', 'g')::jsonb
        WHERE key = 'website_hr_recruitment.apply'
        AND website_id IS NOT NULL
        AND arch_db::text LIKE '%csrf_token%'
    """)

```

## File: models\hr_applicant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import UserError


class Applicant(models.Model):

    _inherit = 'hr.applicant'

    def website_form_input_filter(self, request, values):
        if 'partner_name' in values:
            applicant_job = self.env['hr.job'].sudo().search([('id', '=', values['job_id'])]).name if 'job_id' in values else False
            name = '%s - %s' % (values['partner_name'], applicant_job) if applicant_job else _("%s's Application", values['partner_name'])
            values.setdefault('name', name)
        if values.get('job_id'):
            job = self.env['hr.job'].browse(values.get('job_id'))
            if not job.sudo().active:
                raise UserError(_("The job offer has been closed."))
            stage = self.env['hr.recruitment.stage'].sudo().search([
                ('fold', '=', False),
                '|', ('job_ids', '=', False), ('job_ids', '=', values['job_id']),
            ], order='sequence asc', limit=1)
            if stage:
                values['stage_id'] = stage.id
        return values

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Department(models.Model):
    _inherit = 'hr.department'

    # Get department name using superuser, because model is not accessible for portal users
    display_name = fields.Char(compute='_compute_display_name', search='_search_display_name', compute_sudo=True)

```

## File: models\hr_job.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_join

from odoo import api, fields, models, _
from odoo.tools import mute_logger
from odoo.tools.translate import html_translate


class Job(models.Model):
    _name = 'hr.job'
    _inherit = [
        'hr.job',
        'website.seo.metadata',
        'website.published.multi.mixin',
        'website.searchable.mixin',
    ]

    @mute_logger('odoo.addons.base.models.ir_qweb')
    def _get_default_website_description(self):
        return self.env['ir.qweb']._render("website_hr_recruitment.default_website_description", raise_if_not_found=False)

    def _get_default_job_details(self):
        return _("""
            <span class="text-muted small">Time to Answer</span>
            <h6>2 open days</h6>
            <span class="text-muted small">Process</span>
            <h6>1 Phone Call</h6>
            <h6>1 Onsite Interview</h6>
            <span class="text-muted small">Days to get an Offer</span>
            <h6>4 Days after Interview</h6>
        """)

    description = fields.Html(
        'Job Description', translate=html_translate,
        prefetch=False,
        sanitize_overridable=True,
        sanitize_attributes=False, sanitize_form=False)
    website_published = fields.Boolean(help='Set if the application is published on the website of the company.', tracking=True)
    website_description = fields.Html(
        'Website description', translate=html_translate,
        default=_get_default_website_description, prefetch=False,
        sanitize_overridable=True,
        sanitize_attributes=False, sanitize_form=False)
    job_details = fields.Html(
        'Process Details',
        translate=True,
        help="Complementary information that will appear on the job submission page",
        sanitize_attributes=False,
        default=_get_default_job_details)
    published_date = fields.Date(compute='_compute_published_date', store=True)
    full_url = fields.Char('job URL', compute='_compute_full_url')

    @api.depends('website_url')
    def _compute_full_url(self):
        for job in self:
            job.full_url = url_join(job.get_base_url(), (job.website_url or '/jobs'))

    @api.depends('website_published')
    def _compute_published_date(self):
        for job in self:
            job.published_date = job.website_published and fields.Date.today()

    @api.onchange('website_published')
    def _onchange_website_published(self):
        if self.website_published:
            self.is_published = True
        else:
            self.is_published = False

    def _compute_website_url(self):
        super(Job, self)._compute_website_url()
        for job in self:
            job.website_url = f'/jobs/{self.env["ir.http"]._slug(job)}'

    def set_open(self):
        self.write({'website_published': False})
        return super(Job, self).set_open()

    def get_backend_menu_id(self):
        return self.env.ref('hr_recruitment.menu_hr_recruitment_root').id

    def toggle_active(self):
        self.filtered('active').website_published = False
        return super().toggle_active()

    @api.model
    def _search_get_detail(self, website, order, options):
        requires_sudo = False
        with_description = options['displayDescription']
        country_id = options.get('country_id')
        department_id = options.get('department_id')
        office_id = options.get('office_id')
        contract_type_id = options.get('contract_type_id')
        is_remote = options.get('is_remote')
        is_other_department = options.get('is_other_department')
        is_untyped = options.get('is_untyped')

        domain = [website.website_domain()]
        if country_id:
            domain.append([('address_id.country_id', '=', int(country_id))])
            requires_sudo = True
        if department_id:
            domain.append([('department_id', '=', int(department_id))])
        elif is_other_department:
            domain.append([('department_id', '=', None)])
        if office_id:
            domain.append([('address_id', '=', int(office_id))])
        elif is_remote:
            domain.append([('address_id', '=', None)])
        if contract_type_id:
            domain.append([('contract_type_id', '=', int(contract_type_id))])
        elif is_untyped:
            domain.append([('contract_type_id', '=', None)])

        if requires_sudo and not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            # Rule must be reinforced because of sudo.
            domain.append([('website_published', '=', True)])


        search_fields = ['name']
        fetch_fields = ['name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate':  False},
        }
        if with_description:
            search_fields.append('description')
            fetch_fields.append('description')
            mapping['description'] = {'name': 'description', 'type': 'text', 'html': True, 'match': True}
        return {
            'model': 'hr.job',
            'requires_sudo': requires_sudo,
            'base_domain': domain,
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-briefcase',
        }

```

## File: models\hr_recruitment_source.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import api, fields, models


class RecruitmentSource(models.Model):
    _inherit = 'hr.recruitment.source'

    url = fields.Char(compute='_compute_url', string='Tracker URL')

    @api.depends('source_id', 'source_id.name', 'job_id', 'job_id.company_id')
    def _compute_url(self):
        for source in self:
            source.url = urls.url_join(source.job_id.get_base_url(), "%s?%s" % (
                source.job_id.website_url,
                urls.url_encode({
                    'utm_campaign': self.env.ref('hr_recruitment.utm_campaign_job').name,
                    'utm_medium': source.medium_id.name or self.env['utm.medium']._fetch_or_create_utm_medium('website').name,
                    'utm_source': source.source_id.name or None
                })
            ))

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class Website(models.Model):
    _inherit = "website"

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Jobs'), self.env['ir.http']._url_for('/jobs'), 'website_hr_recruitment'))
        return suggested_controllers

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['jobs', 'all']:
            result.append(self.env['hr.job']._search_get_detail(self, order, options))
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_department
from . import hr_job
from . import hr_applicant
from . import hr_recruitment_source
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_job_public_public,hr.job.public,hr.model_hr_job,base.group_public,1,0,0,0
access_hr_job_public_portal,hr.job.public,hr.model_hr_job,base.group_portal,1,0,0,0
access_hr_job_public_employee,hr.job.public,hr.model_hr_job,base.group_user,1,0,0,0
access_hr_department_public,hr.department.public,hr.model_hr_department,base.group_public,1,0,0,0

```

## File: security\website_hr_recruitment_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
    <record id="hr_job_public" model="ir.rule">
        <field name="name">Job Positions: Public</field>
        <field name="model_id" ref="hr.model_hr_job"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="hr_job_portal" model="ir.rule">
        <field name="name">Job Positions: Portal</field>
        <field name="model_id" ref="hr.model_hr_job"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="hr_job_officer" model="ir.rule">
        <field name="name">Job Positions: HR Officer</field>
        <field name="model_id" ref="hr.model_hr_job"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="hr_department_public" model="ir.rule">
        <field name="name">Job department: Public</field>
        <field name="model_id" ref="hr.model_hr_department"/>
        <field name="domain_force">['|', ('jobs_ids.website_published', '=', True), ('child_ids', 'not in', [])]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    </data>


    <record id="hr_recruitment.group_hr_recruitment_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_restricted_editor'))]"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 21a4 4 0 0 1 4-4h46v21a4 4 0 0 1-4 4H0V21Z" fill="#1AD3BB"/><path d="M0 21a4 4 0 0 1 4-4h21v25H0V21Z" fill="#03AF89"/><path d="M34 17a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" fill="#985184"/><path d="M25 17h-9a9 9 0 0 0 9 9v-9Z" fill="#005E7A"/></svg>

```

## File: static\src\fields\boolean_toggle_labeled_field.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { useRecordObserver } from "@web/model/relational_model/utils";
import {
    BooleanToggleField,
    booleanToggleField,
} from "@web/views/fields/boolean_toggle/boolean_toggle_field";

export class BooleanToggleFieldLabeled extends BooleanToggleField {
    static template = "website_hr_recruitment.BooleanToggleFieldLabeled";
    static props = {
        ...BooleanToggleField.props,
        true_label: { type: String },
        false_label: { type: String },
    };
    setup() {
        super.setup(...arguments);
        useRecordObserver((record) => {
            this.state.label = record.data[this.props.name]
                ? this.props.true_label
                : this.props.false_label;
        });
    }
    async onChange(newValue) {
        super.onChange(...arguments);
        this.state.label = newValue ? this.props.true_label : this.props.false_label;
    }
}

export const booleanToggleFieldLabeled = {
    ...booleanToggleField,
    component: BooleanToggleFieldLabeled,
    displayName: _t("ToggleLabeled"),
    supportedOptions: [
        {
            label: _t("Autosave"),
            name: "autosave",
            type: "boolean",
            default: true,
            help: _t(
                "If checked, the record will be saved immediately when the field is modified."
            ),
        },
        {
            label: _t("Label"),
            name: "true_label",
            type: "string",
            help: _t("A clickable label for the toggle. Contains text for the true state."),
        },
        {
            label: _t("Label"),
            name: "false_label",
            type: "string",
            help: _t("A clickable label for the toggle. Contains text for the false state."),
        },
    ],
    extractProps({ options }, dynamicInfo) {
        return {
            autosave: "autosave" in options ? Boolean(options.autosave) : true,
            readonly: dynamicInfo.readonly,
            true_label: _t(options.true_label),
            false_label: _t(options.false_label),
        };
    },
};

registry.category("fields").add("boolean_toggle_labeled", booleanToggleFieldLabeled);

```

## File: static\src\fields\boolean_toggle_labeled_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_hr_recruitment.BooleanToggleFieldLabeled" t-inherit="web.BooleanToggleField" >
        <xpath expr="//CheckBox" position="inside">
            <label invisible="1" string="Published"/>
            <label invisible="1" string="Not Published"/>
            <t t-out="this.state.label"></t>
        </xpath>
    </t>

</templates>

```

## File: static\src\img\job_congratulations.svg

```svg
<svg viewBox="270 11.222 1305 1144.667" xmlns="http://www.w3.org/2000/svg" overflow="visible">
    <g id="Master/Scenes/Work Station" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <ellipse id="Oval" fill="#C9C9C9" opacity=".5" cx="932.5" cy="1056.5" rx="637.5" ry="61.5"/>
        <g id="Background" opacity=".5" transform="translate(295.000000, 111.000000) scale(0.966183574879227 0.965556831228473)">
            <g id="Background/Blob 1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M869.490312,1 C892.755389,1.92225471 915.476128,2.67306463 938.178097,3.75966986 C969.204039,5.24710117 998.516884,13.6857318 1027.26162,24.1202161 C1037.65031,27.8920014 1048.00522,31.8411434 1058.0448,36.3637386 C1111.46613,60.4322218 1153.95818,95.8373436 1182.91939,145.017167 C1196.72302,168.457808 1203.97709,193.894302 1207.15551,220.364195 C1211.99823,260.684461 1207.17053,300.347325 1194.25536,338.780517 C1181.74563,376.012412 1161.11839,409.312902 1133.93534,438.957479 C1119.10311,455.13359 1102.71795,469.583429 1084.53084,482.282167 C1056.54443,501.823326 1028.65312,521.486269 1000.60163,540.943479 C992.90959,546.280732 984.675715,550.915653 976.903588,556.152405 C967.836315,562.262933 958.607617,568.242218 950.137238,575.042073 C912.903355,604.937314 894.836382,642.935391 897.166393,689.27869 C897.984775,705.586045 899.121,721.881576 899.722899,738.196025 C900.311032,754.162856 895.142962,768.633978 886.071935,782.031501 C881.843627,788.278003 876.617995,793.530126 870.611521,798.221801 C851.124267,813.448463 829.09302,824.464677 805.695301,833.153971 C770.656284,846.163675 734.397203,854.918 697.265931,860.621791 C670.549635,864.725824 643.740739,867.970269 616.719113,869.629145 C583.069094,871.693577 549.420325,871.303392 515.807846,869.280344 C473.354588,866.722861 431.187889,861.708396 389.257694,854.931006 C351.62338,848.84649 314.022853,842.549146 277.041743,833.480308 C234.792454,823.121496 192.917319,811.586217 152.869153,794.88986 C129.05974,784.962616 106.556736,772.934286 86.3599665,757.26778 C63.4264977,739.478905 47.2553176,717.39327 38.2080661,690.617141 C33.6781836,677.210159 33.1914089,663.507582 34.9307837,649.805006 C39.2904828,615.461659 50.0132903,583.102343 69.9622923,553.767549 C79.7365775,539.393381 91.5968611,526.686367 104.247997,514.582365 C116.075746,503.265827 128.207572,492.221235 139.714975,480.622109 C146.848914,473.432069 153.392217,465.6603 159.632693,457.750192 C174.762751,438.569659 178.94601,417.075212 174.700184,393.621565 C172.520334,381.58023 170.856041,369.344984 170.448101,357.153486 C169.504584,329.005799 178.127628,302.888255 191.235506,277.901065 C202.254884,256.896122 217.020799,238.458123 234.095453,221.59032 C286.091497,170.219551 345.922235,129.172122 414.627539,99.9957664 C469.984706,76.486548 525.927504,54.2980456 583.469525,35.9262588 C614.35907,26.0640453 646.160847,19.6460984 678.441891,15.7442516 C713.601037,11.4947856 748.8215,7.45105345 784.134562,4.65591226 C812.657806,2.39875298 841.359992,2.13035321 869.490312,1" id="Fill-1" fill="#7C6576" opacity=".15"/>
            </g>
        </g>
        <g id="Decoration" transform="translate(179.000000, 233.000000) scale(1 1)">
            <g id="Decoration/Waves" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M732.812429,712.260407 C735.104447,713.111515 736.9486,715.190826 739,716.70097 C738.847199,717.525646 738.694398,718.34856 738.541596,719.173236 C735.104447,720.064872 731.611095,721.885151 728.23893,721.684268 C714.528972,720.861354 703.608075,726.228796 694.178308,735.629749 C693.544271,736.260591 692.565992,736.554866 691.715925,736.942535 C691.457744,737.060597 691.101207,736.960156 690.634022,736.960156 C686.280065,733.945155 688.18569,730.3557 690.221283,727.083428 C697.121927,715.98026 720.625214,707.745835 732.812429,712.260407 Z M737.268501,686.975459 C738.410059,687.472161 739.692571,688.099574 740.390384,689.051152 C741.255253,690.227552 742.247155,691.977339 741.944364,693.158968 C741.657234,694.269141 739.77436,695.672108 738.561455,695.703478 C735.456974,695.783648 732.336832,695.130092 729.21669,694.847756 C721.097011,694.114031 712.869441,692.270132 704.878536,692.988172 C693.645676,693.999005 683.533492,699.131596 675.916725,707.671392 C669.639897,714.710624 664.269702,722.562008 658.474904,730.033457 C656.689479,732.335717 656.424972,736.302365 651.635998,735.981687 C648.870852,732.246833 650.068096,728.158188 651.872662,724.653386 C663.262138,702.538801 679.665072,686.395101 705.052554,681.691242 C716.327178,679.601606 726.978818,682.505137 737.268501,686.975459 Z M871.201407,713.09351 C882.321974,713.776363 889.56221,719.594065 890,728.25923 C886.494164,731.212652 883.079755,729.009359 880.421368,727.314001 C869.654197,720.445108 859.483054,721.625804 849.26444,728.286141 C847.849095,729.207824 845.816624,729.265008 843,730 C844.891815,718.292272 855.929747,712.15669 871.201407,713.09351 Z M604.089609,692.869712 C604.389187,692.971901 604.721858,692.998329 604.993568,693.144565 C608.259317,694.892344 613.139653,696.04461 611.7602,700.909146 C610.410357,705.667969 606.296384,703.352866 602.933098,702.700972 C586.778525,699.570121 571.96857,701.284425 561.018295,715.543275 C559.283529,717.80376 558.205397,720.60514 556.352193,722.735247 C555.129496,724.137699 553.056834,724.782545 550.965013,726 C548.376798,719.017691 551.508434,714.267678 554.323074,709.885896 C563.909572,694.971629 587.248793,687.170049 604.089609,692.869712 Z M897.386614,689.118459 C905.540978,695.234501 913.448934,701.813642 917.789245,711.369848 C918.410547,712.734772 917.482994,714.78738 917.250666,716.690265 C912.123607,717.391878 909.795046,714.164112 907.183115,711.557874 C902.45559,706.843316 898.13464,701.663918 893.023422,697.408977 C874.833188,682.262503 853.362199,683.998254 837.915901,701.883281 C833.163735,707.3865 829.316241,713.654006 825.035772,719.555908 C823.284511,721.97238 822.550565,725.713734 818.201453,724.881548 C816.233705,714.463559 828.929027,696.298236 845.447203,685.815831 C861.315916,675.746037 880.998682,676.8237 897.386614,689.118459 Z M747.074812,655.016818 C751.091539,656.478953 755.382765,658.433708 754.972753,665.39501 C751.696133,664.958291 748.746132,664.612409 745.810029,664.165209 C734.61983,662.456764 723.483489,659.780549 712.234221,659.17089 C682.680352,657.570751 660.815945,671.269754 645.079131,695.712047 C640.968587,702.095132 637.627685,708.979571 633.487607,715.339948 C631.588695,718.258978 628.612634,720.468776 626.126503,723 C625.417668,722.638397 624.708834,722.275046 624,721.913443 C624.205006,719.293129 623.722026,716.365364 624.722733,714.099666 C636.164846,688.221441 651.79047,665.947023 678.814774,654.470046 C701.525265,644.825542 724.42339,646.764575 747.074812,655.016818 Z M890.159377,645.206828 C897.364025,646.912757 904.538858,650.585584 910.59476,654.932726 C925.884468,665.90742 936.728269,680.859697 944.986956,697.617736 C946.686397,701.066375 947.04242,705.177069 948,708.919954 C942.670174,711.664504 940.042968,709.426128 937.987504,706.450385 C935.095473,702.267881 932.457744,697.906728 929.818262,693.556083 C923.329168,682.859872 915.575578,673.21279 905.877014,665.26288 C896.997479,657.985534 886.916584,653.655906 875.190617,654.097276 C868.517374,654.349487 861.752932,654.160329 855.200702,655.221718 C838.03968,657.997794 825.812123,668.667733 816.236326,682.218835 C809.105338,692.309038 803.352844,703.369554 796.93215,713.965932 C795.071358,717.034502 793.024663,719.990979 791.063904,723 C790.043188,722.583151 789.022471,722.168053 788,721.751204 C788.438452,718.446185 788.192919,714.85918 789.424093,711.879934 C798.293105,690.405193 810.283898,670.886842 829.395155,657.064262 C847.57514,643.914245 868.3455,640.045254 890.159377,645.206828 Z M589.228145,663.417243 C597.502447,664.869177 605.413043,668.586979 613.390085,671.553047 C615.381723,672.294119 617.343635,673.822471 618.65857,675.519651 C619.651766,676.800978 620.394914,679.365409 619.77067,680.559656 C619.118448,681.807217 616.507811,682.969474 615.084463,682.64781 C610.781197,681.677484 606.58984,680.10115 602.440449,678.526593 C585.321808,672.029324 567.930387,670.458321 550.241707,675.676041 C532.140361,681.014607 518.833633,691.999186 511.828804,710.186564 C510.791893,712.875396 509.648319,715.534016 508.342127,718.094893 C507.964432,718.832411 506.84359,719.175401 505.616083,720 C500.615132,717.007275 501.954548,713.143746 502.918018,709.422389 C507.326199,692.399045 518.328292,680.632519 532.98143,672.477166 C550.524978,662.717045 569.647497,659.98023 589.228145,663.417243 Z M911.063863,612.203907 C940.988916,628.742292 961.548902,653.71816 973.728049,685.640679 C974.963157,688.877868 974.614029,692.726544 975,696.291505 L971.826264,697.673081 L971.826264,697.673081 C969.431488,694.961033 966.577055,692.530939 964.736674,689.482307 C960.780468,682.926869 957.266376,676.098287 953.713686,669.306711 C943.015265,648.859734 926.967633,633.461503 907.469068,621.940143 C880.69319,606.117217 853.566429,608.760539 827.820392,624.326182 C800.233975,641.005544 782.860006,666.256318 772.340535,696.497684 C770.977355,700.418611 769.900144,704.454083 768.298363,708.269278 C767.408875,710.383935 765.707093,712.154961 764.42637,714 C759.491201,711.324958 761.145614,707.967939 761.796502,705.009181 C766.847462,682.05281 777.470444,661.838446 792.435601,643.950204 C809.009557,624.142911 828.873041,609.386126 854.727851,603.902114 C874.612388,599.683372 893.498663,602.495866 911.063863,612.203907 Z M633.827513,645.774776 C636.902169,647.385616 639.936532,649.27844 642.501373,651.590717 C644.14119,653.069376 644.882262,655.557894 646,657.544126 C641.176906,661.051314 637.744853,658.712601 634.444196,657.070038 C615.502208,647.653501 595.670233,641.358186 574.615404,638.945452 C559.902602,637.258829 545.328203,637.754065 531.30041,642.531949 C507.093838,650.776485 493.433953,668.315953 488.905185,693.372084 C488.283245,696.826401 487.71912,700.298341 486.913227,703.71036 C486.117846,707.074793 485.58876,711.480809 480.968891,710.957375 C476.044184,710.400454 476.95169,705.791762 477.293318,702.522498 C479.052267,685.712666 483.80178,669.954991 494.637536,656.61005 C507.992584,640.161512 525.629376,631.137992 546.258484,628.91031 C577.008556,625.593461 606.32554,631.368867 633.827513,645.774776 Z M446.123306,579.516569 C450.841005,581.253968 455.481824,583.546563 459.638641,586.352322 C462.528669,588.301641 464.532818,591.568021 467,594.319486 C462.621276,597.906373 460.045761,595.100614 457.419575,593.780051 C428.552495,579.25736 398.808273,578.234537 368.924267,589.382261 C335.80951,601.736706 318.310339,627.288026 312.179077,661.303909 C309.980978,673.490219 308.65478,685.832405 306.811383,698.085268 C306.315151,701.374416 306.241764,705.015598 301.394765,708 C300.823399,704.278254 299.888595,701.372665 300.010906,698.512613 C300.936973,677.145412 303.702943,656.040922 311.294945,635.918974 C324.107868,601.955633 349.786132,583.138485 384.288242,575.740529 C405.09155,571.28144 425.926309,572.074828 446.123306,579.516569 Z M441.581317,695.483009 C442.451133,695.853048 442.848413,697.361532 444,699.209956 C441.738829,702.124234 439.64392,705.075693 437.223487,707.724393 C436.754451,708.237844 434.719047,707.982889 433.929737,707.379141 C430.422472,704.705654 432.468377,701.662128 434.031245,699.084249 C435.658868,696.393057 437.92879,693.928491 441.581317,695.483009 Z M455.043106,627.344013 C457.772,629.233323 461.21598,631.10686 459.573736,637.442534 C453.290444,635.404252 447.368201,634.175675 442.075163,631.6379 C424.743092,623.334048 407.943831,625.414392 391.677379,633.972372 C367.723753,646.571864 352.954074,666.32725 347.147506,692.880998 C346.143231,697.46056 347.059873,703.119728 341.286606,707 C340.436565,704.495525 339.365689,702.613225 339.204444,700.655563 C338.215943,688.585358 340.84143,677.086504 346.442936,666.5288 C359.612435,641.710132 379.345652,624.655514 406.993888,617.769521 C424.413592,613.431818 440.441682,617.236728 455.043106,627.344013 Z M280,688.822765 C275.716661,690.745091 272.859384,692.508088 269.75813,693.296069 C265.859656,694.283209 261.775622,694.704043 257.744851,694.953425 C251.932362,695.311913 247.721185,697.582336 245.248773,703.181326 C244.546051,704.774605 242.451631,705.744427 240.992924,707 C239.654488,705.148679 237.455261,703.432441 237.144277,701.418328 C236.367674,696.40989 238.80916,692.326247 242.473966,688.997679 C250.83103,681.403627 268.198053,680.669333 280,688.822765 Z M446.998932,661.714541 C449.732022,661.880345 452.336454,664.135284 455,665.435539 L454.671403,668.786531 L454.671403,668.786531 C451.197667,669.505598 447.753487,670.554529 444.244979,670.875665 C437.346187,671.509212 430.334386,671.21251 423.508616,672.242242 C409.026022,674.427368 399.705352,682.993343 395.284075,696.995953 C394.40434,699.77972 393.322926,702.497166 392.062436,706 C386.152912,702.191737 386.657108,697.873843 387.519457,693.508827 C390.520292,678.335114 404.947251,664.154482 421.549207,661.669163 C429.859747,660.426503 438.531917,661.199675 446.998932,661.714541 Z M257.520071,645.677293 C265.548141,647.465091 273.317638,650.563584 281.051955,653.472302 C283.727392,654.479712 285.943731,656.728647 288,659.39438 C281.161013,661.854375 274.680862,661.428709 268.190157,660.144617 C262.87974,659.094641 257.620334,657.741377 252.281773,656.863441 C225.149218,652.401043 200.718486,667.363203 191.737039,693.789967 C190.848744,696.400718 189.747611,698.936978 188.55501,702 C182.657087,699.685441 182.89807,695.943128 183.058139,692.475723 C183.533069,682.052227 187.562935,672.863162 194.797696,665.708426 C212.239927,648.461859 232.939823,640.200391 257.520071,645.677293 Z M285.060877,614.55927 C288.576774,617.145637 291.683764,620.446566 294.402818,623.872979 C295.65575,625.451987 295.51071,628.13421 296,630.314496 C295.1385,630.738005 294.278748,631.161514 293.418996,631.586766 C290.664993,629.587734 287.912737,627.58696 285.15524,625.591414 C270.965833,615.322632 255.207205,608.447149 237.690628,607.859813 C196.773763,606.491688 162.829278,633.64331 157.087112,673.486255 C156.185421,679.741289 155.142185,686.001551 153.672568,692.143301 C152.999794,694.963208 151.133503,697.497289 149.889309,700 C144.438969,698.569133 143.792407,695.188033 144.04579,691.571651 C146.313998,659.057327 157.391171,631.137114 185.817164,612.305785 C215.75122,592.475811 256.150835,593.293201 285.060877,614.55927 Z M1006.00115,548.66532 C1044.75382,553.683161 1077.04335,585.204553 1083.03536,624.045387 C1086.54612,646.79434 1080.14754,667.66336 1069.13891,687.304274 C1067.85117,689.600604 1065.54091,691.317141 1063.0614,694 C1059.51226,688.750241 1060.91691,684.370172 1062.03714,680.35379 C1063.8309,673.916336 1066.34357,667.6862 1068.36766,661.310239 C1083.03187,615.058601 1056.78671,569.110916 1009.63056,558.736142 C995.584052,555.645672 981.951084,557.966599 968.682802,563.219871 C965.088291,564.641242 961.268687,565.482819 957.241438,566.684571 C955.934502,559.769227 960.200803,557.987682 963.613843,556.121804 C976.873401,548.877911 991.237486,546.753761 1006.00115,548.66532 Z M984.7075,589.027849 C1031.66325,591.635546 1054.26592,634.183272 1035.27631,670.823235 C1032.16662,676.823199 1029.75323,683.670359 1022.38329,687 C1017.97396,683.48074 1019.59107,679.961481 1020.69077,676.814501 C1022.35172,672.05315 1024.47045,667.444885 1026.45763,662.798349 C1039.99433,631.168504 1019.10699,600.194497 982.643145,597.932985 C978.207501,597.658125 973.749055,597.757283 969.304641,597.5868 C968.80653,597.567664 968.334727,596.913566 967,595.923719 C968.276848,594.417205 969.267809,592.101765 970.813006,591.649463 C975.327576,590.327349 980.119265,588.772125 984.7075,589.027849 Z M974.57712,625.11588 C985.029673,626.238336 993.287505,631.02935 997.861155,640.858285 C1002.21375,650.221426 999.357635,658.903389 994.355918,666.977719 C993.173471,668.888171 990.168231,669.672665 987.96123,671 C984.73143,665.772961 986.440189,661.885515 987.601583,657.954292 C991.980497,643.134717 987.227902,635.668019 972.050824,633.409098 C970.491186,633.177952 968.917514,632.890772 967.431561,632.386454 C966.624549,632.111531 965.994729,631.321784 965,630.544295 C966.357884,625.301496 970.459608,624.672851 974.57712,625.11588 Z M578.282058,496.432659 C596.627593,500.500739 613.003408,508.689389 627.069836,521.107093 C631.047795,524.617015 634.258877,529.13652 637.233592,533.584287 C638.846136,535.997139 640.724811,539.214859 636.004486,543.555894 C633.208359,540.870086 630.48927,538.432738 627.968028,535.804671 C608.608744,515.614748 585.222476,504.169884 557.012582,503.877682 C527.050076,503.566234 502.959961,520.081762 492.186905,548.204877 C483.346801,571.290573 481.848063,595.168888 485.470597,619.500378 C486.424817,625.906073 488.573125,632.420249 484.959346,640 C482.825044,638.42351 480.632965,637.588897 479.832821,636.036903 C478.551189,633.557562 477.807073,630.642542 477.512928,627.841254 C474.767576,601.697062 475.226302,575.61761 483.380068,550.451507 C492.74543,521.54977 511.540937,501.736035 542.300086,495.477317 C554.256725,493.045218 566.372692,493.792344 578.282058,496.432659 Z M720.021947,599.428728 C734.087194,601.008246 747.377425,604.273054 759.164645,612.299334 C763.043212,614.938328 766.702336,618.220764 769.602118,621.905132 C771.490024,624.300852 771.826154,627.951725 773,631.472147 C766.515984,633.864342 763.396759,630.317477 760.375065,627.336489 C751.999659,619.072224 741.983657,614.164435 730.816449,610.984244 C708.786817,604.710242 689.676123,608.805355 673.769993,626.060182 C670.006373,630.141191 665.819541,633.825559 661.507313,638 C658.600565,633.1592 660.439706,630.049523 662.486098,627.287129 C676.861352,607.909824 694.883539,596.608159 720.021947,599.428728 Z M792.383571,605.457346 C794.104444,607.937113 794.172928,611.555335 795,614.613948 C788.718815,617.198643 785.982979,613.849732 783.26997,610.778878 C778.223248,605.06562 773.406561,599.153001 768.310671,593.488708 C740.704713,562.808144 685.97394,567.664501 663.239106,595.391379 C655.811258,604.451799 649.11917,614.1103 642.081153,623.490747 C639.933574,626.355245 639.224153,630.758669 632,631 C632.856924,627.168427 632.960528,624.024124 634.23011,621.451671 C650.176278,589.139497 674.970891,568.621082 711.685182,563.773469 C748.446884,558.918861 772.927175,577.406944 792.383571,605.457346 Z M615.130754,548.575527 C616.882502,550.011212 617.776751,552.480591 619,554.366994 C615.512254,558.560935 612.208258,557.327116 609.046011,555.743512 C604.515267,553.474259 600.103522,550.977037 595.623527,548.603371 C565.810561,532.807353 531.86935,545.370034 519.727864,576.74106 C514.63887,589.893676 512.692872,603.549217 513.536371,617.60501 C513.755121,621.259482 514.83837,625.103637 510.267375,629 C508.851627,626.102526 507.259128,623.88026 506.639629,621.414362 C499.998387,595.018637 514.24162,557.506359 536.554095,542.39599 C561.178316,525.721159 592.57503,530.06824 615.130754,548.575527 Z M817.215409,594.653606 C817.905566,597.046535 818.083783,599.681919 817.966719,602.180263 C817.902071,603.591072 816.750645,604.949173 816.156586,606.186047 C810.886931,606.634062 809.127468,602.879518 807.464102,599.587045 C804.88344,594.476157 802.557873,589.200118 800.574764,583.823934 C786.892184,546.733538 742.785039,521.203693 695.86485,537.896216 C662.288278,549.841534 638.945248,572.95561 623.997671,605.061616 C621.446711,610.543215 619.42691,616.274298 616.956323,621.796306 C616.238211,623.400376 614.80548,624.681173 612.237047,628 C611.622022,623.776358 610.68201,621.430866 611.106588,619.368238 C612.062324,614.701852 613.280145,610.000326 615.112993,605.618561 C630.282468,569.399598 655.725671,543.214422 692.408824,529.462987 C746.277758,509.26716 800.835101,537.883918 817.215409,594.653606 Z M596.889681,570.02921 C599.448978,570.68037 601.710561,572.523275 604,573.758899 C602.847446,578.732987 599.624821,579.027852 596.484024,579.069976 C591.116464,579.140182 585.745422,578.869889 580.377862,578.901482 C559.548318,579.027852 544.33704,592.521427 541.450432,613.209352 C541.001249,616.431803 539.996681,619.577028 538.957293,624 C532.024558,619.415554 532.79757,614.285258 533.330322,609.800856 C535.466551,591.78251 545.463479,579.505252 561.696735,572.216124 C573.123039,567.084073 584.975892,566.99456 596.889681,570.02921 Z M598,599.789073 C590.619224,607.949507 579.368197,607.912283 572.390535,615 C568.789627,608.279444 569.190933,606.423338 574.042758,602.267828 C580.761921,596.51001 590.094996,595.154731 598,599.789073 Z M906.798226,495.733156 C932.765775,509.754371 944.601953,534.447377 949.094897,562.577626 C950.923745,574.032764 949.448138,586.042919 949.251506,597.796642 C949.211484,600.287195 948.663352,603.016616 947.500963,605.168187 C946.451682,607.110748 944.238271,608.413985 942.539931,610 C941.175691,608.05217 938.796971,606.183377 938.636881,604.138946 C938.139212,597.805424 938.127031,591.377058 938.579458,585.032998 C939.738366,568.726727 936.40259,553.423352 929.880685,538.56961 C918.663985,513.023002 898.294345,499.565586 872.211949,493.794107 C851.00706,489.102805 829.990102,490.660718 809.965002,499.010569 C801.354975,502.600618 793.715925,508.570568 785.65925,513.500738 C783.324033,514.930435 781.0706,516.49713 778.599654,518.120029 C775.352623,513.739607 777.67392,510.555282 780.04742,508.416006 C785.66099,503.354107 791.192776,497.656396 797.806906,494.36142 C834.223771,476.217971 871.05304,476.434006 906.798226,495.733156 Z M899.742375,540.007798 C911.451399,553.059108 916.364617,568.649107 915.979027,586.023097 C915.89179,589.936542 914.34943,593.816325 913.189171,599 C906.384642,595.017464 906.766742,590.152676 906.138632,585.92566 C905.479116,581.487825 905.775724,576.890547 904.96267,572.493459 C900.209968,546.796534 877.1147,526.65709 850.80909,525.082146 C833.994926,524.075882 818.046157,527.053926 803.31348,535.740036 C801.029602,537.088217 798.468307,537.954525 794,539.952879 C795.078256,535.802042 795.027658,533.397993 796.184428,532.099417 C798.265916,529.762689 800.869084,527.718273 803.601364,526.196477 C833.328431,509.646275 875.113471,512.551684 899.742375,540.007798 Z M318,573.003534 C313.800947,573.003534 309.782766,573.038386 305.766358,572.996564 C296.279477,572.900723 293.135507,576.460794 294.508002,585.818397 C294.871518,588.303302 295.311283,590.777752 295.591457,593.27137 C295.65352,593.818537 295.187156,594.423209 294.961953,595 C287.883145,593.675647 284.439496,588.149956 285.074319,579.132154 C285.732194,569.772809 293.139054,562.469696 302.422011,562.027084 C310.610873,561.636748 316.631052,565.473888 318,573.003534 Z M297.038027,477.343479 C312.301827,482.821699 325.047474,491.957906 334.47967,505.391546 C336.797693,508.692142 339.390677,512.148659 337.128342,516.877062 C331.883206,516.398791 328.749002,512.682992 325.08924,509.755551 C317.679223,503.827091 310.56853,497.341523 302.535501,492.406395 C280.115762,478.636388 258.766281,484.253009 245.030427,506.901691 C233.910181,525.231917 231.371145,544.925128 236.069841,565.723794 C237.401138,571.617217 238.83685,577.496624 239.863602,583.44786 C240.18207,585.296124 239.30498,587.352865 238.974331,589.315004 C238.283448,589.542752 237.592566,589.772252 236.901684,590 C234.762907,587.153148 231.898443,584.607624 230.601951,581.415645 C217.833681,549.960115 220.861729,520.189922 241.87186,493.298116 C254.337325,477.345231 273.41925,468.869495 297.038027,477.343479 Z M838.471424,547.080611 C852.020096,547.816382 862.971518,553.878572 868.695251,566.641911 C871.797281,573.557806 874.73338,581.399576 867.498804,590 C865.305027,587.16781 863.395951,585.280856 862.21173,583.011936 C861.057202,580.801102 860.650236,578.19774 859.900928,575.77216 C855.272337,560.769824 842.923604,553.53885 827.600347,556.84982 C825.451982,557.314517 823.315844,557.883067 821.143026,558.166462 C820.29416,558.277356 819.363202,557.736969 817,557.09449 C822.281834,547.751254 830.335583,546.638797 838.471424,547.080611 Z M851,585.874364 C844.296688,586.813838 833.296881,582.331349 830.55357,577.540701 C829.032264,574.883856 830.759009,572.388587 834.355951,572.047112 C841.530354,571.365827 849.726636,578.175345 851,585.874364 Z M291.748471,519.111884 C300.779768,519.83178 308.960999,523.321596 316.246781,528.571353 C318.711449,530.347215 320.113098,533.635389 322,536.231967 C321.535693,536.914718 321.071386,537.59747 320.608824,538.280221 C317.402312,537.593932 314.11027,537.178267 311.008489,536.162984 C305.607864,534.397735 300.416701,531.933816 294.974184,530.347215 C280.692376,526.181723 267.663849,534.194325 265.832803,549.211321 C264.846587,557.307055 266.2814,565.715865 266.743962,573.976097 C266.967388,577.932872 267.475333,581.873728 267.864583,585.940168 C262.48665,586.484954 261.310172,583.203856 260.088311,580.391485 C254.820346,568.252305 253.490263,555.56657 256.81547,542.78532 C260.74288,527.685191 275.656564,517.829514 291.748471,519.111884 Z M458.021393,465.82064 C474.501379,471.922608 487.390063,483.051576 497.889184,496.992937 C500.419617,500.355924 503.397005,503.967114 501.278746,509.880307 C494.689187,508.677742 490.899625,504.926719 486.972672,501.294553 C480.699634,495.491479 474.708335,489.270653 467.880515,484.201703 C443.266624,465.927263 410.251001,466.67537 388.080585,485.77133 C378.633637,493.909617 371.024948,503.598304 365.534518,514.914299 C359.084089,528.208932 355.421484,542.214967 354.292789,556.951629 C353.990181,560.915898 353.084095,564.834721 352.419747,569 C345.958883,568.216935 344.826711,564.233439 345.019754,560.022714 C345.379754,552.120395 345.612797,544.107958 347.118882,536.382178 C351.645835,513.183864 362.783216,493.44642 380.743201,478.169652 C403.84405,458.523099 430.073592,455.474737 458.021393,465.82064 Z M443.536674,504.247954 C454.533294,507.377488 462.860204,514.875769 470.286153,523.278953 C471.833153,525.029252 472.507128,527.556684 474,530.565448 C467.848668,532.658805 464.491014,529.884581 461.112408,527.791224 C456.268867,524.785961 451.734376,521.273111 446.873375,518.297603 C433.877528,510.342494 420.867713,511.215894 408.947433,520.706014 C402.111649,526.149443 396.980009,532.959856 393.657277,541.032235 C392.094563,544.832134 391.053917,548.845569 389.767079,552.759237 C388.752624,555.846764 389.141993,560.945385 384.242578,559.847947 C380.010154,558.899285 381.070006,554.213735 381.244611,550.858412 C382.479068,526.979085 402.198951,505.081096 425.821255,502.212356 C431.581473,501.512237 437.868997,502.634178 443.536674,504.247954 Z M455,542.775657 C440.823429,546.450884 428.284057,549.701766 415.562774,553 C413.00532,550.501283 413.95946,547.845941 415.767869,545.281659 C422.348766,535.947894 443.807126,533.673188 455,542.775657 Z M1086.72513,460.569003 C1088.70382,471.89731 1088.3752,483.369212 1085.76725,494.681759 C1081.66131,512.478881 1075.11872,529.28134 1065.24452,544.681113 L1064.98489,545.087788 C1062.56045,548.897719 1060.05943,553.039438 1053.92302,551.764588 C1052.16633,546.98565 1054.82147,543.194371 1056.55195,539.245487 C1062.15763,526.451452 1068.31043,513.867557 1073.26414,500.826608 C1079.2736,485.001301 1080.23323,468.554329 1075.66583,452.0951 C1066.21988,418.055893 1029.90963,395.217157 995.188261,401.885606 C986.583082,403.536957 978.355461,407.154871 969.95654,409.88144 C967.6842,410.618681 965.427591,411.404956 962.029567,412.55022 C961.620546,405.561308 965.53072,403.227001 969.079067,401.794545 C976.670433,398.730001 984.438342,395.553382 992.436981,394.148945 C1035.56426,386.569889 1079.17921,417.367684 1086.72513,460.569003 Z M1028.5604,448.919808 C1046.81102,460.735968 1054.52887,480.741683 1049.48932,501.881149 C1047.07903,511.990569 1041.75221,520.609867 1035.81405,528.885021 C1034.32339,530.963855 1031.81149,532.314223 1029.76905,534 C1029.05612,533.594715 1028.34494,533.192923 1027.63202,532.789385 C1028.31692,529.637943 1028.64098,526.353736 1029.7708,523.369998 C1031.33504,519.228054 1033.36347,515.239838 1035.43569,511.312764 C1048.34899,486.847165 1040.11265,456.728536 1004.43994,447.917077 C996.131779,445.864447 987.585394,444.777863 978.898875,446.666283 C976.066429,447.281198 972.946709,446.568455 968,446.392017 C972.587617,439.830589 977.464259,438.995562 982.036111,438.49769 C998.671697,436.684388 1014.40167,439.751978 1028.5604,448.919808 Z M996.726197,485.135963 C1004.347,492.077146 1006.10293,501.385233 1001.40926,511.799609 C999.875916,515.201708 997.991031,518.615945 995.523192,521.386869 C994.143534,522.935327 991.237596,523.172884 989.018837,524 C988.442949,521.770082 987.091555,519.401444 987.471359,517.339724 C988.082577,514.013921 989.857938,510.899665 991.10334,507.68137 C995.390703,496.602874 990.534518,488.943816 978.536261,488.231144 C973.21018,487.913823 967.848769,488.177391 962.70464,488.177391 C960.651935,483.648195 963.455414,481.921135 966.35252,480.416027 C975.215189,475.822674 988.873982,477.988434 996.726197,485.135963 Z M651.431354,448.798165 C672.830383,452.454757 688.084515,473.236889 687.955479,497.574583 C687.931067,502.51621 688.818625,507.897409 683.695552,514 C681.937874,510.722715 680.394675,508.780096 679.824476,506.576924 C678.863682,502.863613 678.535861,498.983691 677.908119,495.178212 C677.435569,492.329864 677.119954,489.431886 676.380613,486.652664 C669.646336,461.347205 652.329373,453.112341 628.530974,458.412006 C617.052013,460.966126 607.914879,467.933679 599.686226,476.198675 C597.30255,478.593273 594.829944,480.897475 592.435806,483.205223 C588.841982,480.372826 589.902169,477.941007 591.332026,475.874314 C606.378654,454.122645 629.924213,445.125621 651.431354,448.798165 Z M644.163814,488.322714 C652.768391,491.2888 659.139337,499.826628 658.997684,511 C654.78436,507.729745 651.540153,505.908248 649.201145,503.260396 C643.752674,497.085591 637.511289,494.059726 629.309214,496.072867 C628.505936,496.271544 627.53682,495.770456 626.705902,495.606943 C624.969785,492.484377 626.716267,490.638265 628.915349,489.294999 C633.78166,486.320122 638.898455,486.506491 644.163814,488.322714 Z M722.270765,487.081851 C723.324405,491.708289 723.823314,496.520697 723.992525,501.268191 C724.097191,504.155986 723.076695,507.085888 722.562086,510 C716.814166,509.570164 715.823325,505.99814 714.951107,502.689281 C713.65499,497.778625 712.728694,492.766212 711.709942,487.783624 C704.430406,452.175667 671.512876,424.345107 637.229452,426.397793 C603.554837,428.411881 578.361675,444.057909 564.974864,476.442974 C563.800857,479.281645 563.656069,484.904601 558.919922,483.250171 C554.005843,481.534336 556.549232,476.618417 557.592406,473.220082 C562.239586,458.075822 571.439748,445.854447 584.069473,436.857718 C643.638514,394.426774 709.058398,428.983826 722.270765,487.081851 Z M875.746394,351.828132 C908.932477,356.235295 935.309052,372.101781 948.950783,403.964376 C962.276239,435.094779 955.837175,464.329892 936.965561,491.427987 C935.890927,492.969443 934.414398,494.260412 932.978058,495.49708 C932.331531,496.052355 931.325045,496.187232 929.219462,497 C927.828554,490.007395 930.923149,485.018683 933.411407,480.033475 C938.84049,469.157452 943.30153,458.041453 944.681954,445.818408 C948.603056,411.130394 931.599382,381.140318 899.701112,367.423112 C871.131585,355.135256 844.071788,357.796019 818.906141,376.572003 C816.875695,378.088936 814.770112,379.509528 812.634824,380.875818 C812.189244,381.157835 811.462338,380.993179 810.039977,381.093023 C810.160546,379.350127 809.660797,377.228874 810.497788,376.183136 C813.637815,372.257678 816.7726,368.104506 820.705934,365.075895 C837.008911,352.523539 856.151368,349.225174 875.746394,351.828132 Z M744.187339,451.040393 C765.147245,452.334171 775.674802,473.28953 773.782981,489.333423 C773.648984,490.473554 772.711007,491.521147 771.944052,493 C766.178667,492.388904 765.052037,488.579156 763.701491,485.11686 C762.21166,481.300128 761.236657,477.268639 759.535252,473.554921 C754.543874,462.659949 745.648961,458.591795 733.806121,461.374902 C731.149107,461.99822 728.418043,462.31599 725.724004,462.773439 C723.68408,457.636739 726.279385,454.836173 729.475912,453.74493 C734.160507,452.143858 739.333485,450.741829 744.187339,451.040393 Z M772.452179,420.822087 C794.395934,432.888617 807.189678,460.122905 797.977352,477 C792.644173,471.582445 791.245533,465.400922 789.453959,459.447891 C786.684375,450.243629 782.777532,441.789384 775.44506,435.220643 C763.705488,424.701238 750.38033,421.047092 735.405539,426.272782 C728.346564,428.735624 721.720336,432.457794 714.915815,435.646233 C712.092571,436.968353 709.236439,439.124212 707,434.849123 C716.999927,416.124897 750.771534,408.902072 772.452179,420.822087 Z M911.295447,424.520549 C915.356621,439.491783 911.67933,453.397891 905.337437,466.884245 C904.684314,468.269434 903.527452,469.463985 902.427913,470.569338 C901.807793,471.193722 900.824635,471.452571 899.76331,472 C895.526696,470.007916 896.389999,466.495973 897.239406,463.160675 C898.566497,457.94348 900.253152,452.813733 901.50555,447.579048 C903.447549,439.451557 903.93218,431.339806 901.022656,423.228055 C893.384939,401.925528 873.053012,392.243196 850.388255,399.228607 C845.778179,400.648775 841.142048,401.986742 836.13593,403.476869 C835.250045,397.500618 838.876962,396.164401 841.717005,394.602565 C864.103837,382.282778 901.771316,389.434341 911.295447,424.520549 Z M842.944331,455.376862 C843.157355,462.994325 842.9583,463.643031 838.669872,470 C833.600936,467.79897 832.78376,463.773124 832.088811,459.629491 C830.826379,452.084107 829.993488,444.422694 828.130397,437.02674 C819.595447,403.144589 791.348746,379.880825 756.721786,378.555285 C742.560894,378.012059 729.147334,381.914844 716.566665,388.526725 C704.364901,394.941708 693.785265,403.489159 684.035028,413.212719 C681.424605,415.816334 678.646556,421.703914 674.356382,417.387996 C670.745442,413.757703 675.196257,409.629892 677.824142,406.795978 C695.367234,387.885051 716.227921,375.018166 741.958489,370.324276 C792.628639,361.080652 841.43046,401.376029 842.944331,455.376862 Z M540.421586,454.46841 C542.809139,455.141423 544.816497,457.211962 547,458.639674 C546.879663,459.467179 546.757582,460.292908 546.637245,461.120413 C542.850995,461.994088 539.104857,463.214037 535.269774,463.636668 C531.848022,464.011354 528.330349,463.377407 524.856277,463.418249 L523.521605,463.433672 C516.70343,463.517362 509.956113,463.768473 504.083348,468.457861 C502.813707,469.470045 500.201176,468.740207 498.209514,468.814789 C498.272299,466.728269 497.510166,463.888826 498.556573,462.706169 C501.21968,459.692701 504.322277,456.469693 507.907966,455.02955 C518.532489,450.758843 529.575574,451.408772 540.421586,454.46841 Z M876.215979,425.08377 C882.368325,432.196948 884.049852,450.371194 879.12279,458.458708 C878.421146,459.614316 876.75863,460.173819 875.55754,461 C871.316568,458.193772 872.080427,454.160473 871.855763,450.51412 C871.584437,446.100848 871.828112,441.623084 871.183498,437.27256 C870.103381,429.990311 867.485177,427.637263 860.328754,426.22892 C857.776222,425.726937 855.249612,425.090742 851,424.126864 C860.370231,416.604082 869.851066,417.731802 876.215979,425.08377 Z M576.882861,379.685934 C585.715123,385.745586 592.425473,393.728045 597.731128,402.942084 C600.533521,407.806998 600.48629,408.338577 598.962642,414.517527 C594.575374,415.440334 591.975901,412.566649 589.498879,409.719279 C586.7857,406.601734 584.32967,403.261382 581.682965,400.084188 C563.696568,378.505232 539.717036,371.810492 513.220249,374.880669 C472.155215,379.638566 447.126098,405.596471 430.511159,441.536836 C428.322773,446.271926 427.848711,452.068419 420.489367,457 C420.373912,452.101752 419.5185,448.926313 420.38091,446.326312 C422.976884,438.50701 425.361193,430.410516 429.391601,423.315776 C446.715011,392.835062 471.23158,371.261369 506.777544,365.540314 C531.643974,361.538558 555.51155,365.021016 576.882861,379.685934 Z M537.408526,415.723433 C542.375011,417.777491 547.409723,419.730587 552.171531,422.195457 C555.024767,423.671594 557.405672,426.047687 560,428.01819 C559.748089,428.758 559.49443,429.497809 559.244268,430.235877 C556.964828,430.380357 554.562932,431.0575 552.42869,430.571837 C545.962987,429.097441 539.392321,427.701378 533.232759,425.328767 C511.054138,416.783536 485.119601,422.251161 469.170517,444.475027 C468.337813,445.634351 466.917318,446.37242 464.917778,448 C461.230088,441.750789 463.523523,437.567821 466.254303,433.708628 C472.471594,424.914473 481.078537,419.004704 491.053493,415.317843 C506.472516,409.618702 522.022741,409.368037 537.408526,415.723433 Z" id="Combined-Shape" fill="#3AADAA"/>
            </g>
        </g>
        <g id="Chair" transform="translate(346.000000, 525.000000) scale(1 1)">
            <g id="Chair/Wooden Chair" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M234.240046,466.469073 C233.243531,472.184318 232.27503,477.904563 231.260505,483.617808 C226.214895,512.019026 221.360383,540.457246 216.013617,568.803461 C213.763453,580.727973 210.482756,592.459476 207.625278,604.268982 C207.24308,605.84805 206.860882,607.539123 205.98643,608.852179 C205.295072,609.890223 203.78329,611.093275 202.754758,610.994271 C201.748238,610.897267 200.359519,609.376201 200.082376,608.248153 C199.573112,606.176064 199.467057,603.900967 199.696176,601.765875 C201.279995,586.978241 202.763763,572.172606 204.715773,557.431974 C208.224588,530.950838 214.690933,505.057728 220.760073,479.089614 C221.084241,477.699554 221.560487,475.206448 222.255847,474.1074 C222.868163,473.139359 226.444013,471.641295 227.466542,470.933264 C230.616172,468.758171 231.062403,468.604164 234.240046,466.469073 Z M86.6076724,416.303922 C90.1405,418.669023 93.3871796,420.844116 96.9620289,423.237219 C94.3436744,436.54479 91.6122614,450.049369 89.0409312,463.583949 C84.2284415,488.918036 79.4639768,514.261122 74.7755513,539.61821 C73.0266466,549.077615 71.687954,558.613024 69.8730151,568.059429 C69.1936637,571.595581 67.7959406,575.014728 66.5162786,578.406873 C66.486263,578.485877 66.455247,578.56488 66.4232304,578.643883 C65.372687,581.186992 61.6387553,580.997984 60.8963713,578.348871 C60.5631989,577.15682 60.3791037,575.973769 60.1389794,574.82472 C58.3980788,566.493362 59.0414116,558.123003 60.6222294,549.895651 C67.4107413,514.581136 74.4033588,479.305623 81.3029281,444.01211 C83.0768458,434.937721 84.7817278,425.851331 86.6076724,416.303922 Z M386.896019,405.330451 C395.975716,427.381397 404.543149,448.20729 413.122587,469.027183 C422.327349,491.36414 431.637165,513.658097 440.687847,536.058057 C442.784932,541.24728 444.123625,546.757516 445.610394,552.174748 C446.024608,553.679813 446.12466,555.152876 445.832509,556.564937 C445.191177,559.66907 441.495265,560.83612 439.18607,558.664027 C438.114516,557.656983 437.250069,556.433931 436.45966,555.23588 C432.664697,549.484633 428.661626,543.752387 425.808149,537.52312 C419.231747,523.166504 412.854448,508.685883 407.191519,493.951251 C398.0708,470.218234 389.62343,446.225205 380.790861,422.381182 C380.376647,421.263134 379.702298,420.284092 379.722829,419.30405 C379.722829,419.30405 380.453686,417.362967 381.116029,416.302921 C383.106059,413.113785 384.65486,409.649636 386.896019,405.330451 Z M399.805698,339.756639 C402.796245,345.451883 403.701713,350.428097 403.508613,355.489314 C403.07739,366.813799 400.376993,377.487257 392.527933,386.135628 C387.779476,391.366852 383.075043,396.785084 377.596208,401.182273 C355.251649,419.120042 330.656925,433.53366 304.921612,446.013196 C291.530684,452.507474 277.130235,455.360596 262.461646,457.277679 C234.521192,460.929835 207.408166,456.61765 180.576285,449.252335 C175.149477,447.763271 169.771695,446.096199 164.341886,444.616136 C142.015336,438.528875 122.777384,426.625364 104.568964,412.837773 C91.7963566,403.165358 80.2193675,392.124885 68.6073603,381.117413 C65.2446207,377.929276 66.0460353,374.77114 67.0955783,370.422954 C69.4988215,372.496043 71.5688924,374.162114 73.5038935,375.971192 C81.7331507,383.666522 89.5782091,391.817871 98.2266832,399.00518 C106.439932,405.831472 115.281506,411.964735 124.204122,417.861988 C136.913697,426.264349 151.311145,430.96355 165.79964,435.173731 C189.237766,441.983023 213.067093,446.735227 237.540754,447.825273 C255.544068,448.628308 273.076137,446.653223 289.926855,439.878933 C319.305053,428.067426 346.14894,411.914733 371.144871,392.606905 C374.438575,390.061796 377.440127,387.097669 380.36364,384.119541 C393.229296,371.008979 398.569058,359.025465 399.805698,339.756639 Z M278.228803,293.794668 C288.292009,294.810711 298.224147,295.518742 308.06824,296.8778 C321.739312,298.764881 335.353355,301.08098 348.973401,303.329077 C358.774471,304.946146 367.74111,308.851314 376.391585,313.616518 C392.539939,322.511899 398.836196,338.135569 393.210286,355.698323 C390.679977,363.598661 386.747943,370.80297 381.055998,376.794227 C375.813286,382.313464 370.432502,387.871702 364.35836,392.400896 C342.88225,408.417583 319.964394,422.063169 295.449712,433.015638 C275.200236,442.063026 254.08031,445.079156 232.111945,443.224076 C201.496107,440.639965 171.801745,434.113685 143.273987,422.580191 C134.96669,419.222047 127.379765,414.64185 120.386147,409.021609 C117.990908,407.096527 115.23148,405.626464 112.655147,403.924391 C97.8264761,394.127971 83.4510394,378.437298 70.8395151,366.293777 C78.3764141,370.161943 79.6160554,368.729881 82.1443634,367.209816 C85.3920435,365.255732 84.596632,360.361522 84.3655124,356.355351 C98.1366366,351.21213 112.00281,346.033908 126.297205,340.695679 C127.134638,343.025779 127.610884,344.392838 128.117146,345.749896 C129.482853,349.408053 132.292306,351.088125 136.03124,351.456141 C140.30245,351.876159 142.612645,350.365094 143.595153,346.191915 C144.004365,344.45384 143.99536,342.59176 143.983354,340.786683 C143.969347,338.814598 143.731224,336.843514 143.556133,334.376408 C153.225135,330.91526 162.740058,327.510114 172.736229,323.93196 C173.602677,325.841042 174.140956,327.772125 175.249529,329.28419 C176.708284,331.274275 178.304109,333.545372 180.394191,334.616418 C185.565866,337.266532 189.959139,334.432411 190.090207,328.616161 C190.167246,325.197014 189.798055,321.767867 189.608958,317.939703 C197.779184,314.974576 206.39264,311.839442 214.884033,308.75831 C215.196195,310.16137 215.84553,311.486427 216.571906,312.669477 C217.425348,314.059537 218.206752,315.5236 219.249291,316.762653 C221.092245,318.955747 223.405441,320.43181 226.419,319.616775 C229.570631,318.764739 231.352553,316.46964 231.584673,313.298504 C231.820795,310.076366 231.6367,306.824227 231.6367,303.16907 C240.484277,300.327948 249.233803,297.518828 258.295491,294.609703 C260.114432,299.882929 262.972911,303.64809 268.884969,303.383079 C274.652953,303.125068 276.826077,299.042893 278.228803,293.794668 Z M129.943091,65.4008733 C156.366761,58.9055948 179.575767,65.7138867 199.346995,84.3146844 C208.843908,93.2500676 216.209719,103.814521 222.588019,115.107005 C234.278066,135.806893 242.241186,157.997844 248.889625,180.721819 C257.777223,211.101122 264.257575,242.016447 269.673377,273.175784 C270.230665,276.382921 271.30322,279.500055 271.869513,282.706192 C272.499839,286.269345 273.053125,289.8755 273.214209,293.482654 C273.358283,296.703793 271.759456,298.50387 269.120091,299.019892 C265.985469,299.633918 264.245569,298.655876 263.011931,295.087723 C262.187504,292.700621 261.624213,290.187513 261.255022,287.684406 C257.86927,264.807425 251.697077,242.584472 245.786019,220.280515 C241.041565,202.376747 237.01048,184.284972 232.539166,166.307201 C229.196437,152.867624 224.389951,139.955071 218.251775,127.521537 C210.507769,111.833865 199.293968,99.0153148 185.08962,89.0988896 C168.615097,77.5983964 150.261602,72.8241917 130.280265,77.5843958 C118.867361,80.3035124 109.337431,86.7587892 100.975105,94.7801332 C87.070912,108.118705 77.7470886,124.369402 70.8975451,142.28017 C61.4346497,167.024231 58.7302507,192.679332 59.7827952,218.878455 C61.0534525,250.524812 65.2106031,281.867156 70.661423,313.027493 C73.193733,327.502113 76.5794845,341.828728 79.5760347,356.222345 C79.7241113,356.934376 79.8681859,357.648406 79.9712392,358.368437 C80.4284757,361.557574 79.4669783,363.56666 77.1057568,363.912675 C74.4603883,364.300691 73.9581284,362.125598 73.1156926,360.317521 C64.9904892,342.870772 60.6992693,324.332977 57.3165193,305.51617 C52.1018216,276.505926 48.6070137,247.300674 47.1142414,217.861411 C46.9951798,215.50431 47.0982331,213.136209 47.0982331,210.774108 C46.5369428,187.588113 48.2918506,164.629129 54.6761534,142.228168 C56.5141042,135.778891 58.6001834,129.337615 61.2835716,123.203352 C74.6274747,92.6920437 97.6944078,73.3282133 129.943091,65.4008733 Z M93.5172469,109.693773 C94.7358773,118.176137 96.0855755,126.639499 97.1951495,135.135864 C102.630962,176.755649 109.114316,218.200426 119.049455,259.012176 C122.611298,273.643804 127.35075,287.984419 131.230757,302.544043 C134.120252,313.386508 136.574522,324.348978 139.032793,335.301448 C139.621098,337.92556 139.66512,340.729681 139.533052,343.429796 C139.473021,344.651849 138.384458,346.811941 137.737123,346.822613 C136.117285,346.845943 134.209298,346.224916 132.940642,345.208873 C131.95213,344.416839 131.645972,342.675764 131.215749,341.299705 C119.921907,305.205157 111.077331,268.486583 104.957165,231.198983 C100.517868,204.147823 97.5653411,176.853653 93.8804348,149.676487 C92.3966672,138.728018 90.7288043,127.803549 89.0139172,115.968042 C90.0084317,114.582982 91.8433809,112.025873 93.5172469,109.693773 Z M131.577937,81.64557 C135.435932,100.028358 139.020787,117.995129 142.993842,135.873896 C152.801916,180.003788 162.724049,224.108679 172.733227,268.19357 C176.627242,285.346306 180.906456,302.412037 184.950548,319.531772 C185.677924,322.613904 186.014098,325.740038 185.741957,328.857171 C185.639904,330.027222 184.614374,330.91926 183.440767,330.843257 C180.979493,330.68625 179.762864,329.081181 178.956447,327.168099 C176.753307,321.939875 175.144475,316.479641 173.725741,310.986405 C168.347959,290.160512 162.740058,269.380621 157.89255,248.426722 C147.168002,202.069734 137.426962,155.506737 129.809022,108.521723 C128.477333,100.30937 127.012575,92.1190191 125.551819,83.581653 C127.73695,82.8796229 129.563895,82.2925977 131.577937,81.64557 Z M182.309181,92.83905 C185.238697,94.3271138 188.655464,96.6532135 189.950134,99.2603254 C189.950134,99.2603254 190.117221,101.358415 190.275302,103.005486 C193.386912,135.211867 199.686171,166.923227 205.674269,198.684589 C208.968973,216.164339 211.430246,233.804095 214.902042,251.245843 C217.876581,266.194484 221.79761,280.955117 225.208374,295.818755 C225.842702,298.581873 226.370976,301.366993 226.84322,304.162112 C227.912773,310.503384 228.338994,311.390422 226.454018,313.842528 C225.599576,314.954575 223.985742,315.245588 222.889174,314.37155 C221.724572,313.44451 221.066231,312.001449 220.513946,310.612389 C218.912117,306.575216 217.293279,302.487041 216.30777,298.27386 C206.661779,257.048092 200.075372,215.278301 194.066263,173.396505 C190.303317,147.16538 188.023137,120.712245 182.535298,94.7331312 C182.46226,94.3571151 182.223137,93.4540764 182.309181,92.83905 Z" id="Fill" fill="#FFF"/>
                <path d="M227.026564,314.230485 C228.912742,311.77834 228.48625,310.891287 227.416014,304.54991 C226.943468,301.754744 226.414858,298.969579 225.780125,296.206415 C222.367184,281.342532 218.443654,266.581656 215.467217,251.632768 C211.993206,234.190733 209.530362,216.550685 206.233555,199.070647 C200.241637,167.308762 193.938359,135.596879 190.824764,103.389966 C190.666581,101.742869 190.499388,99.644744 190.499388,99.644744 C189.203892,97.0375892 185.784944,94.7114511 182.85356,93.2233627 C182.76746,93.8383993 183.006736,94.7414529 183.079821,95.1174752 C188.571161,121.097018 190.852796,147.550588 194.618144,173.782146 C200.631087,215.664633 207.221696,257.435113 216.873841,298.661561 C217.85998,302.874811 219.47985,306.963054 221.082701,311.000293 C221.635339,312.389376 222.2941,313.832461 223.459445,314.759517 C224.556712,315.633568 226.171577,315.342551 227.026564,314.230485 M234.81756,466.859547 C231.637889,468.994674 231.191373,469.148683 228.039734,471.323813 C227.016553,472.031855 223.438421,473.529944 222.825714,474.498001 C222.12991,475.597066 221.65336,478.090214 221.328986,479.480297 C215.255973,505.448839 208.785502,531.342376 205.274448,557.823949 C203.321193,572.564824 201.836478,587.370703 200.251648,602.158581 C200.022384,604.293708 200.128506,606.568843 200.638095,608.640966 C200.915415,609.769033 202.305019,611.290123 203.312182,611.387129 C204.341371,611.486135 205.854117,610.283063 206.545916,609.245002 C207.420926,607.931924 207.803368,606.240823 208.18581,604.66173 C211.045111,592.852028 214.327902,581.120332 216.579501,569.195624 C221.929679,540.848941 226.787288,512.410252 231.836118,484.008566 C232.85129,478.295226 233.820409,472.574887 234.81756,466.859547 M89.4987666,116.352736 C91.2147479,128.188439 92.883675,139.113087 94.3683893,150.061738 C98.0556468,177.239351 101.010058,204.533972 105.452187,231.585578 C111.576259,268.873792 120.426478,305.592972 131.727526,341.688115 C132.158023,343.064197 132.464377,344.8053 133.453519,345.597348 C134.722985,346.613408 136.63219,347.234445 138.253061,347.211115 C138.900809,347.200443 139.990067,345.040314 140.050136,343.818242 C140.182289,341.118082 140.138238,338.313915 139.549558,335.689759 C137.089718,324.737109 134.633882,313.774458 131.742544,302.931814 C127.860061,288.37195 123.117585,274.031098 119.55347,259.399229 C109.61199,218.586806 103.1245,177.141345 97.6852191,135.520874 C96.5749371,127.02437 95.2243777,118.560867 94.0049697,110.078364 C92.3300357,112.410502 90.4939156,114.967654 89.4987666,116.352736 M87.0909864,416.693569 C85.2638767,426.241136 83.5579069,435.327675 81.7828573,444.402214 C74.8788856,479.69631 67.8818063,514.972404 61.0889629,550.287501 C59.5071365,558.51499 58.8633932,566.885487 60.6054046,575.216981 C60.845682,576.366049 61.0298947,577.54912 61.3632797,578.74119 C62.1061374,581.390348 65.8424516,581.579359 66.8936654,579.036208 C66.9257024,578.957203 66.9567382,578.878199 66.9867729,578.799194 C68.2672514,575.406993 69.6658663,571.987789 70.3456512,568.45158 C72.1617482,559.005019 73.5012949,549.469452 75.2513156,540.009891 C79.9427326,514.652385 84.7102374,489.30888 89.5257978,463.974376 C92.0987687,450.439573 94.8319245,436.934771 97.4519498,423.626981 C93.8748194,421.233838 90.6260682,419.058709 87.0909864,416.693569 M387.570938,405.719917 C385.328349,410.039174 383.778559,413.503379 381.78726,416.692569 C381.124495,417.752632 380.393171,419.693747 380.393171,419.693747 C380.372627,420.673805 381.047406,421.652863 381.461884,422.77093 C390.30009,446.615345 398.75285,470.60877 407.879388,494.342179 C413.545931,509.077054 419.927299,523.557914 426.507897,537.914766 C429.363194,544.144136 433.368819,549.876477 437.166204,555.627818 C437.957117,556.825889 438.822116,558.048962 439.894354,559.056022 C442.205022,561.228151 445.903292,560.061081 446.545033,556.956897 C446.837371,555.544813 446.737255,554.071726 446.322777,552.566636 C444.835059,547.149315 443.495512,541.638988 441.397089,536.449679 C432.340632,514.049349 423.024876,491.755026 413.81424,469.417699 C405.229328,448.597463 396.656429,427.771227 387.570938,405.719917 M126.059982,83.9658131 C127.52167,92.50332 128.987362,100.693806 130.319901,108.906294 C137.942703,155.892084 147.689958,202.455848 158.421349,248.813601 C163.271949,269.767845 168.883429,290.548079 174.264642,311.374316 C175.684281,316.867642 177.29414,322.327966 179.498686,327.556276 C180.305618,329.46939 181.523023,331.074485 183.985867,331.231495 C185.160223,331.307499 186.186408,330.415446 186.288526,329.245377 C186.56084,326.128192 186.224452,323.002006 185.496611,319.919823 C181.449939,302.799806 177.167995,285.733793 173.271496,268.580775 C163.255931,224.495157 153.327467,180.389538 143.513135,136.258918 C139.537544,118.379856 135.950402,100.41279 132.089945,82.0296981 C130.074618,82.6767365 128.246507,83.2637714 126.059982,83.9658131 M400.488854,340.145024 C399.251425,359.414168 393.908256,371.397879 381.034391,384.508658 C378.109013,387.486835 375.105545,390.451011 371.809739,392.996162 C346.797859,412.304308 319.936844,428.457267 290.539901,440.268969 C273.678431,447.043371 256.135175,449.018488 238.120374,448.21544 C213.631097,447.125376 189.786565,442.373094 166.333484,435.563689 C151.835745,431.353439 137.42911,426.65416 124.711425,418.251661 C115.783116,412.354311 106.935901,406.220947 98.717411,399.394542 C90.0634185,392.207115 82.2133544,384.055631 73.9788463,376.360174 C72.0426107,374.551067 69.9712189,372.884968 67.5664422,370.811845 C66.5162296,375.160103 65.7143036,378.31829 69.0791889,381.50648 C80.6986054,392.514133 92.2829814,403.554789 105.063739,413.227363 C123.283777,427.015182 142.534004,438.918889 164.8748,445.00625 C170.308074,446.486338 175.689287,448.153437 181.119557,449.642525 C207.968559,457.007963 235.098885,461.320219 263.057167,457.668002 C277.735115,455.750888 292.144754,452.897718 305.544225,446.403333 C331.29596,433.923592 355.906377,419.509736 378.265193,401.571671 C383.747524,397.17441 388.454959,391.756088 393.206445,386.524778 C401.060514,377.876264 403.762634,367.20263 404.194132,355.877958 C404.387356,350.816657 403.481309,345.840362 400.488854,340.145024 M195.321956,103.160953 C193.413753,116.794762 225.309582,286.032811 232.094416,299.15859 C238.995384,296.593438 245.956422,294.021285 252.905446,291.41513 C255.163052,290.56908 257.168368,289.838037 257.168368,289.838037 C257.168368,289.838037 256.770909,286.906863 256.550655,285.34177 C255.6376,278.878386 254.917769,272.342998 253.363975,266.022623 C244.961273,231.846594 236.432425,197.699566 227.723369,163.599541 C224.070151,149.299692 218.510731,135.690884 211.092165,122.889124 C207.098554,115.995715 201.23178,109.068304 195.321956,103.160953 M97.5450573,105.699103 C98.0236099,108.363262 98.467122,110.490388 98.7824861,112.636515 C102.786109,139.939137 106.189038,167.344764 110.934518,194.516377 C115.241491,219.170841 120.617698,243.660295 126.269224,268.049743 C129.786285,283.232645 134.738002,298.081526 138.946862,313.108418 C140.497653,318.642747 141.762113,324.25708 143.294883,330.390445 C153.277409,326.803232 162.714306,323.41303 171.661637,320.197839 C165.086044,293.700266 158.357275,268.081745 152.431432,242.279213 C146.501585,216.45768 141.07532,190.511139 135.979436,164.511596 C130.906578,138.629059 126.518512,112.613514 121.748003,86.2029459 C111.813533,91.1672407 104.158694,97.6786273 97.5450573,105.699103 M85.9857102,122.285088 C84.9284895,124.315209 83.5408872,126.175319 82.4946792,128.21144 C75.8099607,141.219213 71.1796142,154.98303 68.1361,169.230876 C63.3425651,191.663208 63.5908518,214.368556 65.5951661,237.059903 C66.8886596,251.709773 68.6156537,266.328641 70.4798062,280.919507 C73.3581296,303.442845 77.2205895,325.807172 82.2774284,347.952487 C82.6198237,349.451576 83.2305289,350.889662 83.7451231,352.441754 C97.8393972,347.190442 111.422081,342.128142 125.349162,336.937833 C124.80253,334.909713 124.363023,333.188611 123.872456,331.482509 C113.165093,294.2693 104.620227,256.55306 98.8085162,218.282788 C94.9650783,192.972285 92.222912,167.495773 88.9321123,142.100265 C88.0741216,135.478872 86.9498234,128.892481 85.9857102,122.285088 M213.780269,305.277954 C213.780269,305.277954 213.469911,304.257893 212.942302,301.146708 C210.728746,291.23412 208.093703,281.396536 206.330668,271.405942 C201.536132,244.238329 196.855727,217.044715 192.61483,189.785096 C189.406126,169.159871 187.06342,148.400639 184.098998,127.735412 C182.451095,116.24273 180.167458,104.841053 178.513548,93.3493702 C178.325331,92.0402925 178.005962,90.0921768 178.005962,90.0921768 C178.005962,90.0921768 175.892522,88.3780751 174.485898,87.6850339 C165.356356,83.189767 155.639136,81.1386452 145.610557,80.3996013 C142.830346,80.1955892 139.991068,80.797625 137.260916,81.0226383 C137.387061,93.0293512 187.50493,306.058 189.556299,313.829461 C197.478446,310.978292 205.649882,308.203127 213.780269,305.277954 M47.5563374,211.160365 C47.5563374,213.522506 47.4532183,215.890646 47.5723559,218.247786 C49.0660806,247.687534 52.5631185,276.893268 57.7811435,305.903991 C61.1660519,324.721108 65.46001,343.259209 73.5903978,360.706245 C74.4333712,362.514352 74.9359515,364.689481 77.5830079,364.301458 C79.9457361,363.955438 80.907847,361.946318 80.4503187,358.757129 C80.3471996,358.037086 80.2030332,357.323044 80.0548621,356.611001 C77.0563999,342.217147 73.668488,327.890296 71.1345622,313.415437 C65.6802643,282.254587 61.5204612,250.911726 60.248993,219.264846 C59.1957769,193.065291 61.9019016,167.409768 71.370835,142.665298 C78.2247489,124.754235 87.5545216,108.50327 101.467586,95.164478 C109.835248,87.1430017 119.371259,80.6876184 130.791446,77.968457 C150.785532,73.2081743 169.150737,77.9824578 185.635772,89.4831407 C199.849184,99.3997295 211.07014,112.218491 218.819087,127.906422 C224.961179,140.34016 229.770733,153.252927 233.115595,166.692725 C237.589761,184.670792 241.623419,202.762867 246.3709,220.66693 C252.28573,242.971254 258.461861,265.194574 261.849773,288.071932 C262.2192,290.575081 262.782851,293.08823 263.607803,295.475372 C264.842228,299.043583 266.583239,300.021641 269.71986,299.407605 C272.36091,298.891574 273.960757,297.091467 273.816591,293.870276 C273.655405,290.263062 273.101765,286.656848 272.471037,283.093636 C271.904383,279.887446 270.831144,276.770261 270.2735,273.56307 C264.854242,242.40322 258.369755,211.487385 249.476486,181.107581 C242.823805,158.383232 234.855604,136.191914 223.158098,115.491685 C216.775728,104.199014 209.405218,93.6343872 199.902245,84.6988566 C180.118401,66.0977522 156.894586,59.2893479 130.454056,65.7847336 C98.184796,73.7122043 75.1031445,93.076354 61.750727,123.588166 C59.0656267,129.72253 56.9782164,136.163912 55.1390929,142.613295 C48.7507165,165.014625 46.9946888,187.973989 47.5563374,211.160365 M126.805844,341.08408 C112.502328,346.422396 98.6273069,351.600704 84.8473958,356.744009 C85.0786629,360.750247 85.8745819,365.644538 82.6248295,367.598654 C80.0949083,369.118744 78.854476,370.550829 71.3127679,366.682599 C83.9323392,378.82632 98.3169486,394.517252 113.155082,404.313834 C115.733058,406.015935 118.494246,407.486022 120.891014,409.411136 C127.889094,415.03147 135.48086,419.611742 143.793458,422.969942 C172.339419,434.503626 202.052728,441.030014 232.688101,443.614167 C254.670484,445.469277 275.803885,442.453098 296.066282,433.405561 C320.596606,422.452911 343.529085,408.807101 365.018898,392.79015 C371.096916,388.260881 376.481133,382.702551 381.727191,377.183223 C387.422767,371.191867 391.35731,363.987439 393.889234,356.08697 C399.518734,338.523928 393.218459,322.9 377.059802,314.004472 C368.403807,309.239189 359.431447,305.333957 349.624123,303.716861 C335.995386,301.468727 322.372657,299.15259 308.692861,297.265478 C298.842487,295.906397 288.904012,295.198355 278.834385,294.182295 C277.430764,299.430606 275.256253,303.512849 269.484589,303.770864 C263.568758,304.03588 260.708455,300.270656 258.888354,294.997343 C249.820884,297.906516 241.065775,300.715683 232.212552,303.556851 C232.212552,307.212068 232.396765,310.464261 232.160492,313.686453 C231.928224,316.857641 230.145165,319.152777 226.991524,320.004828 C223.976042,320.819876 221.661369,319.343789 219.81724,317.150658 C218.774035,315.911585 217.992133,314.447498 217.138147,313.057415 C216.411307,311.874345 215.761557,310.549267 215.449196,309.146183 C206.952385,312.227366 198.333434,315.362552 190.157994,318.327728 C190.347212,322.155956 190.716639,325.585159 190.63955,329.004362 C190.508398,334.820708 186.112322,337.654876 180.937347,335.004719 C178.845932,333.933655 177.249088,331.66252 175.789403,329.672402 C174.680122,328.160312 174.1415,326.229198 173.274499,324.320084 C163.271949,327.898297 153.750956,331.303499 144.075784,334.764704 C144.250987,337.231851 144.489262,339.202968 144.503278,341.175085 C144.515292,342.980192 144.524302,344.842303 144.114829,346.580406 C143.131694,350.753654 140.820025,352.264743 136.54609,351.844718 C132.80477,351.476697 129.993524,349.796597 128.626946,346.13838 C128.120361,344.781299 127.643811,343.414218 126.805844,341.08408 M67.7136121,359.080148 C59.6172636,338.859947 55.2702443,318.085714 51.8132526,297.081467 C47.7085131,272.135986 44.6519838,247.060497 43.4425874,221.810998 C41.918828,190.007109 43.9822105,158.556242 55.1290813,128.321447 C67.7166156,94.1774194 91.6612635,71.8460935 126.683702,62.2275224 C155.849379,54.2170467 181.736269,61.2024615 203.638559,82.2377105 C213.982503,92.1713003 221.855594,103.921998 228.673466,116.436741 C240.304896,137.790009 248.167975,160.601363 254.820657,183.881746 C263.217352,213.26349 269.374462,243.137264 274.636538,273.21405 C275.53958,278.373356 276.926181,283.447657 278.24971,289.268003 C285.121644,290.040049 291.957537,290.637084 298.739368,291.603142 C316.376733,294.115291 334.027113,296.567436 351.600405,299.478609 C362.653167,301.308718 372.906005,305.799985 382.378943,311.643331 C398.11211,321.349908 407.163561,335.782765 408.358941,354.123854 C409.122823,365.866551 406.178424,377.263228 398.497555,386.839796 C395.434017,390.659023 392.315417,394.435247 389.170786,398.293476 C394.407833,410.872223 399.683925,423.406967 404.852893,435.987714 C418.800998,469.92973 432.764121,503.865745 446.55104,537.872764 C448.445227,542.547041 449.570527,547.57034 450.662788,552.516633 C451.419662,555.941837 450.927093,559.431044 448.934793,562.486225 C446.677186,565.949431 443.352347,566.913488 440.08157,564.392339 C437.104132,562.096202 434.267857,559.331038 432.187455,556.226854 C428.454144,550.656523 425.042205,544.824177 422.010704,538.840821 C409.655438,514.459374 401.004449,488.580837 391.601592,463.007319 C386.02215,447.833418 380.322569,432.70552 374.692068,417.55062 C374.069349,415.87252 373.55075,414.155418 372.764842,411.798278 C364.579391,417.261603 356.686277,422.528915 348.395704,428.063244 C349.509991,432.073482 350.648305,436.051718 351.716539,440.047956 C354.954277,452.152674 358.320164,464.226391 361.28759,476.397114 C362.174614,480.03533 362.356825,483.986564 362.080506,487.735787 C361.66002,493.451126 357.450159,495.416243 352.345265,492.640078 C348.995397,490.81897 346.695741,487.873795 345.545413,484.392589 C343.161661,477.17116 340.79693,469.902728 339.152031,462.491288 C337.503127,455.059847 336.751259,447.430394 335.574901,439.892946 C335.411712,438.846884 335.066313,437.828824 334.660845,436.225729 C332.796693,437.168785 331.2439,437.92783 329.716136,438.733878 C323.123524,442.207084 316.599991,445.817298 309.938299,449.153496 C294.942985,456.660942 278.881439,460.388163 262.293285,462.02426 C255.084962,462.734303 247.847605,463.140327 240.041592,463.728362 C238.461768,472.164862 236.80886,480.681368 235.278092,489.219875 C230.151172,517.827574 225.250514,546.479275 219.873305,575.039971 C217.85998,585.731606 214.814463,596.231229 212.141377,606.792856 C211.656817,608.70997 210.83587,610.580081 209.920813,612.341185 C208.404062,615.260359 205.825084,616.227416 202.60837,615.9564 C199.450724,615.688384 197.430391,613.913279 196.803667,611.047109 C196.152916,608.069932 195.674363,604.909744 195.940671,601.901566 C197.029928,589.601835 198.168243,577.294105 199.771094,565.053378 C203.620538,535.651632 210.234175,506.787918 217.261289,478.01121 C218.37958,473.430938 219.052357,468.739659 219.893328,464.093383 C219.983432,463.596354 219.824248,463.055322 219.801221,462.771305 C203.04187,459.038083 186.436697,455.338863 169.238839,451.507636 C168.708226,455.488872 168.116543,459.708123 167.586932,463.936374 C166.093207,475.865082 163.693436,487.597779 159.819964,498.990455 C158.830821,501.897628 157.627432,504.7918 156.076641,507.430956 C153.90213,511.134176 151.198008,512.116235 147.31953,511.256184 C143.12769,510.327128 140.464615,507.615967 140.251369,503.398717 C140.106201,500.523546 140.253371,497.545369 140.849059,494.736203 C143.940628,480.167338 147.222418,465.639475 150.426117,451.094611 C150.810561,449.346508 151.124924,447.583403 151.557423,445.395273 C134.124294,438.645872 118.775571,428.707282 103.264661,417.478615 C102.618916,420.446792 102.066278,422.822933 101.588726,425.213075 C95.7800193,454.242798 89.771081,483.23652 84.2677265,512.324247 C81.3653753,527.666158 79.573306,543.215081 76.8361456,558.592994 C75.5696832,565.708417 73.3501204,572.66783 71.3267841,579.629243 C70.7340998,581.668364 69.5597438,583.650482 68.2512329,585.349583 C65.2978227,589.18081 61.564512,588.859791 59.1036706,584.612539 C58.207636,583.069447 57.6549979,581.288342 57.1694372,579.554239 C54.4042444,569.685653 54.9098281,559.684059 56.7879968,549.821473 C63.175372,516.274482 69.8971334,482.790493 76.3425757,449.254502 C79.1197824,434.807644 81.485514,420.282782 84.1575993,405.815923 C84.3948733,404.531847 84.8694213,402.598732 84.8694213,402.598732 C84.8694213,402.598732 82.9371902,400.558611 81.9340319,399.638556 C76.5768462,394.727265 71.5099957,389.495954 66.1087591,384.636665 C62.9711362,381.815498 61.7136843,378.769317 62.3263917,374.523065 C63.1323223,368.925733 63.6158807,363.3214 67.7136121,359.080148" id="Stroke" fill="#000"/>
            </g>
        </g>
        <g transform="translate(389.000000, 252.000000) scale(1 1.0010405827263267)" id="Master/Character/Sitting On Chair">
            <g id="Master/Character/Sitting On Chair" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <g id="Head" transform="translate(66.000000, 125.000000) scale(1 1)">
                    <g id="Head/Dreadlocks in Ponytail" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M46.2864204,0.2883065 C74.537814,3.60882838 93.9892072,19.4637724 106.856123,43.937742 L107.809096,45.7788478 C110.815497,51.6419077 112.642998,55.8581731 111.664994,64.6537933 C111.525279,65.9134368 111.175992,67.0982796 111.249841,67.5381076 C111.39255,68.3978167 113.889454,70.9919043 114.422366,71.7099908 L114.894947,72.3379075 C116.839528,74.8861723 119.367505,77.7800469 120.0409,77.6511595 L120.528284,77.5617538 C122.073931,77.2878576 124.363179,76.9667832 127.184321,76.7046816 C137.369535,75.7577866 147.195921,77.0336602 155.88898,82.8099935 L156.750129,83.3910216 C158.887764,84.8067645 162.742411,87.0305676 162.935899,87.988955 C163.004772,88.3313139 162.959352,88.7056409 162.818618,89.0311009 C178.854939,102.568272 185.325901,118.77459 182.231457,137.649228 C179.359075,155.169399 162.919732,166.519566 132.913429,171.699728 L132,171.854814 L133.910304,192.494242 C134.612117,194.341274 135,196.370486 135,198.5 C135,207.060414 128.731986,214 121,214 C113.268014,214 107,207.060414 107,198.5 C107,196.177635 107.461319,193.974561 108.288036,191.996977 L108.155048,188.101769 C107.286313,164.00281 105.914459,150.747007 104.040114,148.334508 C79.8550399,148.334508 79.8550399,136.031774 81.1800588,129.332295 C82.5050777,122.632817 88.3386778,124.290406 94.8453794,118.867719 L95.4477596,118.046065 L95.4477596,118.046065 C95.2249263,117.518129 95.0440032,116.881733 94.9041996,116.137606 C93.3124481,107.651219 96.1666232,100.163172 100.357071,92.912493 C103.110452,88.1491857 105.2431,86.8426672 106.332876,85.8921998 C104.016603,84.1169304 102.603487,81.2994381 100.61754,78.6365338 C98.9050346,80.7249688 96.7524277,85.3127439 95.4880081,86.7618824 C89.4193932,93.716351 86.9354625,97.2090441 80.5325298,103.843366 C74.3381715,110.261264 64.282693,117.079097 55.9806362,120.527907 C52.3410639,122.039878 48.421064,122.97339 44.5469704,123.789217 C42.6767871,124.183167 40.557113,123.978712 38.6589868,123.555838 C32.7360745,122.237352 30.7291702,117.10802 32.8757893,111.38128 C33.8617771,108.752285 35.5942414,106.240977 37.8007383,104.398886 L38.4932084,103.816814 C39.8963016,102.62909 40.7394076,101.868162 42.4302902,100.549144 C39.9413696,100.549144 35.5173982,99.9876401 33.510494,99.4670273 C23.116706,96.7722081 20.0539566,89.0258497 27.8909625,81.6056222 C31.7530806,77.9493649 36.4874186,75.8300123 38.6639766,74.0896498 C38.053223,74.5753556 35.3976426,74.6651164 34.6641396,74.7379224 C33.5304533,74.8486274 32.4027547,75.1109284 31.2660745,75.2345989 C26.8697134,75.7168694 22.5498382,75.3843234 18.1886974,74.9143111 L16.5511053,74.7329357 C12.9833864,74.3359934 9.21906878,74.0507535 5.96471039,72.42708 C-0.407285418,69.2485498 -1.98107358,62.9912235 2.74128886,57.6484603 C5.28310144,54.7721248 6.64232749,54.134823 9.9615535,52.0164677 C11.5962174,50.9732475 16.6389261,48.8618737 19.2396185,47.7857412 L18.6719399,47.3839017 C16.7790767,46.0631197 15.8419439,45.5992198 14.871533,44.6530887 C10.3777042,40.2767502 10.9036309,35.2780701 16.4253619,32.1793273 C18.8803517,30.801 23.2384575,29.5992025 26.8610644,29.1404249 C24.6176428,27.9266592 23.6436306,27.1128278 21.4101887,25.0004566 C15.444364,18.7630772 15.8215941,11.5632624 22.6087427,6.22947521 C29.5735287,0.755062746 37.8725917,-0.701057166 46.2864204,0.2883065 Z" id="Path" fill="#FFF"/>
                        <path d="M46.2864204,0.2883065 C74.8231816,3.64236901 94.3812668,19.7853532 107.244016,44.6820116 C110.581205,51.1427957 112.702875,55.3196657 111.664994,64.6537933 C111.525279,65.9134368 111.175992,67.0982796 111.249841,67.5381076 C111.39255,68.3978167 113.889454,70.9919043 114.422366,71.7099908 C116.405319,74.3828685 119.311389,77.7907875 120.0409,77.6511595 C121.571775,77.3599356 124.049718,76.9959056 127.184321,76.7046816 C137.55815,75.7402515 147.559738,77.081677 156.370757,83.1355454 C158.391632,84.5238461 162.730318,86.9706684 162.935899,87.988955 L162.941909,88.0340373 C164.755742,89.5027575 167.207456,91.6238813 168.624795,92.9894411 C174.304795,98.4584411 178.562795,104.938441 182.182795,111.941441 C188.064795,123.320441 187.175795,134.344441 181.023795,145.195441 C176.545795,153.096441 170.231795,159.302441 162.604795,164.152441 C154.867795,169.072441 146.434795,172.325441 137.467795,174.211441 C136.433795,174.429441 135.410795,174.701441 133.934795,175.055441 C134.885795,180.408441 135.800795,185.544441 136.709795,190.682441 C137.088795,192.826441 137.467795,194.971441 137.821795,197.120441 C138.112795,198.889441 138.843795,201.268441 136.509795,201.736441 C134.339795,202.261441 133.886795,199.266441 133.415795,196.646441 C132.454795,191.305441 132.868795,191.745441 131.801795,187.476441 C130.149795,180.871441 130.310795,180.858441 127.469795,179.075441 C126.811795,178.785441 124.846795,177.312441 123.460795,176.388441 C122.815795,175.959441 122.227795,175.455441 121.738795,174.854441 C121.329795,174.351441 121.207795,174.027441 121.258795,173.130441 C121.403795,170.569441 124.111795,170.880441 126.007795,170.623441 C128.344795,170.307441 130.717795,170.262441 133.055795,169.951441 C150.000795,167.693441 164.260795,160.475441 174.492795,146.526441 C182.331795,135.840441 183.937795,124.430441 177.223795,112.337441 C173.212795,105.115441 167.914795,96.8194411 161.400795,91.6684411 C160.887215,91.2623761 160.42483,90.9147002 160.050114,90.5852118 C156.0761,91.9876082 152.482857,92.5866575 148.173486,93.3972014 C143.129779,94.3436794 135.635075,94.6757944 130.570411,93.934769 C125.992753,93.2655522 122.988883,92.5723993 118.587865,91.1591651 C118.009046,92.80478 117.52204,94.1083066 117.088924,95.4307826 C114.078069,104.62628 108.887662,112.334739 101.335075,118.411546 C100.045113,119.44884 98.9268211,119.996036 97.9909229,120.040305 C97.8664528,120.290024 97.7117223,120.499897 97.5275946,120.636741 C96.2445946,121.589741 94.5995946,122.037741 93.1615946,122.800741 C90.6095946,124.153741 87.8185946,125.245741 85.6255946,127.051741 C78.4145946,132.988741 79.8735946,142.202741 88.5275946,145.633741 C90.8375946,146.549741 93.4415946,146.814741 95.9415946,147.133741 C97.5415946,147.337741 99.2015946,147.139741 100.830595,147.044741 C106.198595,146.731741 106.591595,147.112741 106.836595,152.652741 C107.052595,157.545741 107.227595,162.443741 107.620595,167.322741 C108.450595,177.625741 109.386595,187.920741 110.340595,198.212741 C110.430595,199.185741 110.671595,200.172741 110.448595,201.150741 C110.038595,202.951741 107.453595,203.004741 106.840595,201.261741 C106.397595,200.001741 106.020595,198.836741 105.918595,197.647741 C104.745595,183.916741 103.738595,170.171741 102.674595,156.431741 C102.564595,155.008741 102.430595,153.588741 102.253595,151.525741 C100.169595,151.525741 98.2475946,151.539741 96.3255946,151.523741 C92.0005946,151.486741 87.9005946,150.585741 84.1995946,148.300741 C76.9335946,143.817741 74.7955946,134.743741 79.5015946,127.620741 C83.2881551,121.890502 89.2819836,119.682306 95.2302304,117.456944 C95.1042589,117.063264 94.995467,116.623392 94.9041996,116.137606 C93.3124481,107.651219 96.1666232,100.163172 100.357071,92.912493 C103.110452,88.1491857 105.2431,86.8426672 106.332876,85.8921998 C104.016603,84.1169304 102.603487,81.2994381 100.61754,78.6365338 C98.9050346,80.7249688 96.7524277,85.3127439 95.4880081,86.7618824 C89.4193932,93.716351 86.9354625,97.2090441 80.5325298,103.843366 C74.3381715,110.261264 64.282693,117.079097 55.9806362,120.527907 C52.3410639,122.039878 48.421064,122.97339 44.5469704,123.789217 C42.6767871,124.183167 40.557113,123.978712 38.6589868,123.555838 C32.7360745,122.237352 30.7291702,117.10802 32.8757893,111.38128 C33.8617771,108.752285 35.5942414,106.240977 37.8007383,104.398886 C39.6569501,102.850013 40.4792719,102.071088 42.4302902,100.549144 C39.9413696,100.549144 35.5173982,99.9876401 33.510494,99.4670273 C23.116706,96.7722081 20.0539566,89.0258497 27.8909625,81.6056222 C31.7530806,77.9493649 36.4874186,75.8300123 38.6639766,74.0896498 C38.053223,74.5753556 35.3976426,74.6651164 34.6641396,74.7379224 C33.5304533,74.8486274 32.4027547,75.1109284 31.2660745,75.2345989 C26.3201683,75.7771532 21.4710645,75.2884554 16.5511053,74.7329357 C12.9833864,74.3359934 9.21906878,74.0507535 5.96471039,72.42708 C-0.407285418,69.2485498 -1.98107358,62.9912235 2.74128886,57.6484603 C5.28310144,54.7721248 6.64232749,54.134823 9.9615535,52.0164677 C11.5962174,50.9732475 16.6389261,48.8618737 19.2396185,47.7857412 C16.9572764,46.1471076 15.934364,45.6893275 14.871533,44.6530887 C10.3777042,40.2767502 10.9036309,35.2780701 16.4253619,32.1793273 C18.8803517,30.801 23.2384575,29.5992025 26.8610644,29.1404249 C24.6176428,27.9266592 23.6436306,27.1128278 21.4101887,25.0004566 C15.444364,18.7630772 15.8215941,11.5632624 22.6087427,6.22947521 C29.5735287,0.755062746 37.8725917,-0.701057166 46.2864204,0.2883065 Z M95.3063788,79.7565494 C94.3583137,79.7565494 91.655829,81.6884016 91.3863789,81.8320189 C89.9543015,82.5949858 88.5142404,83.3569554 87.1570103,84.2406008 C76.6274993,91.0943378 65.4982124,96.4909576 53.1414305,99.32341 C50.4249743,100.171151 46.2614714,101.243294 45.9830396,101.417829 C44.367337,102.435118 42.0720213,104.678141 39.7188238,107.253279 C37.2788035,109.92117 34.5224288,113.744981 36.7389053,117.519922 C38.9114715,121.220063 41.9043635,120.918865 46.4949948,119.945459 C55.9995975,116.979363 57.7779681,115.919188 62.7807582,112.841389 C74.8062163,105.443103 84.7060124,95.0029227 93.0649532,83.7738446 C93.3623463,83.3739103 96.7494338,79.8453128 95.3063788,79.7565494 Z M106.726073,90.8799092 C101.463813,96.285505 98.8780895,102.915837 98.4469694,110.383937 C98.377112,111.595708 98.293283,112.903224 98.9718981,115.049505 C100.837092,113.920513 101.551633,113.451763 102.246216,112.838397 C107.810859,107.918507 111.488354,101.771886 113.550146,94.6897572 C114.143934,92.6511893 115.448273,88.2648774 113.799637,87.1897423 C111.93644,85.68575 108.323812,89.239281 106.726073,90.8799092 Z M48.4021027,72.680405 C43.6957077,74.3798765 39.8166242,77.4477017 35.5134064,80.081683 C33.5743637,81.2695178 32.2610439,82.9261035 30.0944656,84.7312933 C25.5537324,90.3413443 28.0875613,93.5108985 33.5404329,95.3170856 C35.905606,96.1009967 38.4913289,96.4371011 40.9962169,96.5468088 C50.4239764,96.9587112 59.5204121,94.9331087 67.7985178,90.7063716 C77.7691694,85.6149386 87.2927333,79.6368683 96.9260733,73.9021494 C98.53978,72.9417087 102.842998,70.9230877 104.340941,69.4729518 C103.53758,68.7887749 101.136481,68.5374446 100.091613,68.3180293 C99.0387616,68.095622 97.2444236,67.8223501 96.1805947,67.6358471 C79.7012264,64.7385674 63.9653407,67.0623752 48.4021027,72.680405 Z M124.036745,80.8586127 C122.165564,81.334345 117.961144,82.5690549 117.533018,84.1169304 C116.805503,86.7469223 120.223527,87.6265783 122.307274,88.5501173 C133.393649,90.998593 145.078174,91.6299107 156.887074,87.4111523 C155.258398,86.3559641 152.067539,84.8429956 151.511673,84.476971 C141.563975,78.8459757 135.063242,78.0550832 124.036745,80.8586127 Z M109.021389,71.5982885 C107.152203,72.8748868 106.096358,74.3399828 103.836969,75.8838688 C105.504566,78.4211079 105.846867,79.8054191 107.447601,81.8918594 C108.935564,83.8326878 108.99145,84.8509744 113.864505,82.2010356 C114.593018,81.514864 115.845462,80.0707122 115.845462,80.0707122 C113.83756,77.5643908 112.891491,74.6780818 109.021389,71.5982885 Z M24.5946897,50.4376747 C22.1806164,50.6501086 17.0341196,52.7943948 11.7578875,55.9270473 C9.92862071,57.0131531 8.17220526,58.4722651 6.86686923,60.1398216 C4.03963914,63.7502011 4.74220532,67.0952875 8.90171645,68.9922328 C11.6531013,70.2468895 16.1409424,71.6401769 19.1507998,71.9563344 C37.021329,74.1215648 48.5547911,67.6348498 64.7267867,63.1029259 C54.7890679,62.197339 41.9662372,52.2967209 41.9662372,52.2967209 C38.6849338,53.1314965 30.9137934,51.7850843 24.5946897,50.4376747 Z M52.7971332,36.3921016 C49.0707381,37.3176353 45.0040376,39.1188357 44.2974795,43.5899217 C43.6128767,47.9253692 46.352286,50.6211857 49.6974591,52.4493143 C53.9507788,54.7731221 59.8387624,58.0384211 64.5032429,59.1015881 C74.5347702,61.3884943 85.3566845,62.1913549 95.518945,63.8758662 C98.1815111,64.3176889 100.513751,64.912105 103.069535,65.4925583 C103.069535,65.4925583 106.230085,65.5843138 106.111328,64.8014 C104.766073,61.7126306 103.136399,59.2222666 101.963792,57.3402815 C94.1517353,48.9227116 89.123996,46.4343423 78.0186602,39.667374 C69.9022245,36.6663707 61.3786198,34.2607809 52.7971332,36.3921016 Z M40.4902495,4.5828629 C35.3267872,4.88705233 30.4657079,6.2105257 26.3181723,9.41199482 C20.9511256,13.5539578 20.6557285,18.8069603 25.8471337,23.0496549 C28.3081112,25.0602972 31.2950154,27.0031202 34.3238341,27.6374299 C43.6837321,29.5992025 53.1264611,31.6766667 62.6310637,32.3159632 C78.623426,33.3910983 88.1779268,39.8010179 102.240228,51.536746 C104.736134,54.8529095 106.920676,58.1221979 107.512468,59.2362294 C107.807865,60.0929465 108.906623,61.4652896 109.014403,61.2089726 C109.491429,60.0779864 108.764912,56.7887511 108.323812,55.4054372 C107.735014,51.536746 104.308008,46.2797542 102.669352,43.3425808 C95.7494745,30.9356413 86.3706152,20.7677355 74.0966643,13.499104 C63.7737317,7.38539506 52.6693939,3.86377902 40.4902495,4.5828629 Z M24.2154636,33.5237448 C21.866258,34.2647702 17.8913701,35.6939619 17.049089,38.2760814 C15.8355656,41.9961686 22.0588649,44.8276236 24.5238343,45.7541547 C29.2432028,47.5274295 35.5603107,48.8010358 40.5611049,48.2933885 C40.3195977,48.3183221 39.4533656,46.0892618 39.6838951,44.3010268 C39.8385794,43.1052133 40.2537322,41.9502908 40.7696792,40.8641849 C41.9832026,38.3119857 43.8962983,36.177673 46.0489052,34.3435603 C41.2786405,33.3871089 37.5193127,32.513437 33.1771743,32.3379047 C30.1623271,32.2152316 27.0935898,32.6151659 24.2154636,33.5237448 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Bottom/Sitting On Chair" transform="translate(89.000000, 467.000000) scale(1 1)">
                    <g id="Bottom/Sitting On Chair/Crossed Legs" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M35.0001671,52.0873119 C41.6850075,82.5346087 46.9435192,100.257055 50.7757023,105.254651 C56.5239768,112.751044 89.9762429,149.113559 128.963398,152.928233 C154.954835,155.47135 187.397077,143.867013 226.290124,118.115225 L244.2161,125.857832 L325.343175,127.810512 L327.449286,135.122736 L315.829708,235.68057 L319.154094,240.054065 L336.903485,245.714781 L330.300504,262.58703 L325.343175,265.3897 L311.214032,317.382919 L311.214032,330.162868 L319.154094,338.811826 L340.135272,343.242305 L348.634995,337.381129 L355.022252,347.300358 L416.912606,349.127903 L422.283315,343.242305 L420.510605,322.904228 L418.920292,320.688989 L376.737094,304.258363 L379.880353,273.6807 L368.677977,269.941992 L376.737094,249.240644 L396.449368,245.714781 L404.731332,196.468692 L409.283919,133.582142 L423.750644,149.928138 L432.184225,162.759808 L452.177347,152.928233 L462.484266,175.1857 L455.664543,182.207515 L468.527586,224.97273 L471.8441,243.835338 L489.529115,251.513885 L500.847494,249.240644 L506.133745,243.835338 L513.603692,251.513885 L570.168109,237.350394 L576.816881,231.722241 L576.816881,218.051252 L571.714808,208.682528 L515.504504,192.305833 L511.9171,186.364269 L503.400565,156.5852 L494.998528,160.296124 L481.30617,133.582142 L493.626851,115.889735 L462.484266,70.6131688 L420.510605,14.9625249 L392.543953,-19.0272074 C377.696319,-28.3555673 361.752768,-31.517821 344.713298,-28.5139683 C327.673829,-25.5101157 297.050202,-18.7616229 252.842419,-8.26848996 L155.942732,31.1591111 L35.0001671,52.0873119 Z" id="Fill" fill="#FFF"/>
                        <path d="M370.2161,257.0987 C358.0641,259.6817 346.6121,259.0407 334.8991,257.5587 C334.4451,260.3467 334.0761,262.6107 333.6431,265.2717 C344.0271,270.2747 354.5791,273.2647 366.0871,273.6807 C367.4771,268.0967 368.7581,262.9517 370.2161,257.0987 M465.1741,175.1857 C474.8081,171.9327 483.2411,167.3357 491.3551,161.7117 C488.1521,153.7277 485.1381,146.2147 481.8361,137.9847 C475.1841,147.9437 467.4111,155.8637 458.4991,162.8687 C460.7711,167.0597 462.9081,171.0047 465.1741,175.1857 M354.5611,342.6977 C354.8961,343.2677 355.4421,343.6867 356.0781,343.8617 C356.4471,343.9637 356.8181,344.0527 357.1931,344.0827 C369.8351,345.1297 382.4691,346.3017 395.1271,347.1137 C401.2701,347.5077 407.4581,347.1337 413.6231,347.2697 C416.9211,347.3417 418.6181,345.7737 419.4361,342.7477 C420.5461,338.6467 420.8931,333.7697 420.0141,329.5467 C419.3761,329.5987 418.6111,330.3157 418.1271,330.4847 C409.2721,333.5697 400.0941,334.3527 390.8021,334.4177 C368.9871,334.5707 347.9871,329.8627 327.2361,323.7307 C322.6031,322.3617 318.9081,320.6877 313.3821,318.8397 C312.5071,323.2377 312.6111,324.8177 313.0081,327.2017 C313.1281,327.9277 313.1791,328.6677 313.1801,329.4047 C313.1821,332.2167 314.8861,333.6437 317.1761,334.6057 C321.0031,336.2147 325.0491,337.2937 328.9281,338.7897 C334.6631,341.0017 340.1281,341.8287 344.8511,336.6827 C344.8771,336.6547 344.9101,336.6277 344.9441,336.6087 C348.0741,334.8007 349.3781,334.8397 351.3591,337.5247 C352.5501,339.1387 353.5181,340.9197 354.5611,342.6977 M513.2781,248.6037 C516.2411,248.0357 518.9091,247.6247 521.5271,247.0057 C535.2951,243.7517 549.0671,240.5117 562.8021,237.1207 C567.3611,235.9947 571.7971,234.4057 575.1231,231.0967 C575.8231,219.0397 574.8731,215.3107 570.7041,211.7447 C556.2801,221.8587 540.8821,229.1867 523.6391,232.1677 C506.4331,235.1427 488.9411,236.0147 471.8441,230.3717 C473.1051,244.0237 475.2891,245.9637 488.0311,247.7747 C488.7491,247.8757 489.4831,247.8687 490.1991,247.9767 C495.3321,248.7577 500.0281,248.3827 503.0101,243.2857 C503.1861,242.9857 503.5101,242.7587 503.7951,242.5367 C506.4381,240.4827 508.8491,240.8367 510.6551,243.6607 C511.5301,245.0287 512.1911,246.5327 513.2781,248.6037 M314.3091,314.4237 C316.4071,315.2707 318.3761,316.1247 320.3861,316.8657 C334.7521,322.1507 349.6321,325.3287 364.6801,327.9457 C378.7601,330.3927 392.8231,331.0027 406.9611,328.8187 C410.8181,328.2227 414.6881,327.5507 418.0021,325.5147 C419.1691,324.7977 419.2501,323.0857 418.0931,322.3547 C417.1341,321.7477 415.6881,321.5657 414.7251,321.2077 C402.8201,316.7947 390.8591,312.5297 378.9851,308.0367 C374.1101,306.1927 372.8881,303.4737 373.9871,298.2447 C374.6221,295.2257 375.4251,292.2417 376.0101,289.2137 C376.6881,285.7057 377.2091,282.1687 377.9391,277.8087 C359.6761,279.3477 343.2291,275.3577 327.6981,267.4197 C323.2481,283.0427 318.8851,298.3607 314.3091,314.4237 M457.6351,181.9657 C458.1751,183.6967 458.6491,185.2487 459.1451,186.7937 C461.8671,195.2597 464.9671,203.6257 467.2361,212.2117 C469.2471,219.8287 471.3491,225.6607 471.3491,225.6607 C473.5011,227.2447 478.1681,227.3727 484.8381,229.0827 C486.4081,229.4857 488.0711,229.5767 489.6991,229.6977 C515.8671,231.6357 540.2341,226.0427 562.6031,212.1787 C563.7561,211.4627 564.7711,210.5227 566.1991,209.4167 C555.6431,206.5317 545.6731,203.9297 535.7801,201.0587 C529.1651,199.1387 522.6591,196.8457 516.1091,194.7037 C512.2781,193.4517 510.2741,190.6967 509.4091,186.8357 C508.0211,180.6357 506.5131,174.4617 504.9811,168.2957 C504.3111,165.6007 503.4571,162.9507 502.5341,159.7507 C488.6691,169.9987 473.7561,176.4947 457.6351,181.9657 M318.3551,237.3427 C320.4761,238.2177 322.1011,238.9657 323.7781,239.5687 C337.7291,244.5797 352.1801,247.3697 366.9351,247.7467 C374.8541,247.9497 382.8411,246.7237 390.7461,245.7317 C396.3891,245.0237 395.6331,244.6957 396.8771,237.7027 C397.9181,231.8417 398.7761,225.9517 399.3811,220.0297 C401.5001,199.2997 403.7841,178.5767 405.2891,157.7977 C406.4151,142.2487 407.4411,126.6077 404.8391,111.0527 C401.8221,93.0117 394.3821,77.5287 379.1821,66.4907 C369.7851,59.6657 359.2241,55.4777 348.1241,52.4017 C347.0181,52.0947 345.1451,52.0357 344.7931,52.1257 C344.7931,52.1257 343.5091,52.9297 342.6841,53.6657 C338.6211,57.2957 334.3621,60.7087 330.3441,64.3877 C305.5311,87.1017 277.6321,105.3407 248.1171,121.2497 C247.0411,121.8287 246.0221,122.5127 244.2161,123.6097 C246.6461,124.0527 248.2081,124.4877 249.7921,124.5987 C255.0441,124.9687 260.3011,125.3977 265.5611,125.4877 C282.4511,125.7767 299.3441,125.9527 316.2361,126.1057 C319.1411,126.1317 322.2151,126.6767 324.9581,125.4547 C327.0901,124.5057 329.5701,125.9507 329.4831,128.2827 L329.4791,128.3877 C329.1621,134.5517 328.8171,140.7267 328.1321,146.8577 C325.5541,169.9537 322.7921,193.0297 320.2031,216.1247 C319.4191,223.1187 318.9711,230.1487 318.3551,237.3427 M483.9661,131.7687 C487.1711,139.5957 490.9191,148.3997 494.2701,156.5847 C496.5901,156.0107 498.2521,155.1007 500.5261,154.7417 C505.2871,153.9877 505.7661,154.1177 506.9921,158.8647 C509.1271,167.1277 511.3391,175.3887 512.9661,183.7587 C513.8131,188.1137 515.9611,190.4627 520.0061,191.6197 C535.3521,196.0117 550.6771,200.4767 566.0171,204.8887 C566.6641,205.0747 567.3211,205.2237 567.9751,205.3857 C574.9591,207.1227 577.9431,212.0687 579.1341,218.4117 C579.2671,219.1177 579.3341,219.8377 579.3731,220.5547 C580.1391,234.3937 580.9251,235.1617 570.2281,239.1267 C563.8541,241.4907 557.2071,243.1827 550.5811,244.8027 C539.3861,247.5387 528.1391,250.0547 516.8611,252.4237 C510.3941,253.7827 510.3571,253.5827 507.7781,247.6977 C507.5721,247.2267 507.1921,246.8317 506.9041,246.4197 C506.6641,246.4857 506.4041,246.4737 506.3251,246.5877 C501.5841,253.3957 494.7201,253.1087 487.7081,252.0627 C484.6641,251.6087 481.6241,250.9727 478.6681,250.1197 C473.1321,248.5237 470.0031,244.7227 468.9661,238.8937 C468.0751,233.8877 467.1451,228.8887 465.7821,223.9907 C462.3741,211.7417 458.7941,199.5127 454.6851,187.4777 C452.1041,179.9207 451.4471,178.7407 459.3011,173.7197 C456.3871,168.1897 453.4491,162.6147 450.3411,156.7147 C448.0891,157.9027 446.2121,158.9167 444.3121,159.8877 C442.2111,160.9607 440.1381,162.1107 437.9671,163.0227 C431.8011,165.6137 431.7061,165.6327 427.5591,160.3637 C422.1811,153.5297 416.5201,145.5057 410.7791,138.0347 C410.3911,139.5077 410.4021,141.4787 410.3651,142.3837 C409.0381,175.4647 405.8121,208.3597 400.7331,241.0747 C399.6861,247.8207 399.4651,248.3517 392.7341,249.7747 C387.4281,250.8957 382.0081,251.4777 376.1901,252.3677 C374.7761,257.6567 373.3441,263.0107 371.8091,268.7507 C374.1641,269.6107 376.1911,270.2627 378.1471,271.0807 C381.9521,272.6737 382.7791,273.9827 382.0911,278.1027 C380.9571,284.9027 379.6211,291.6677 378.3691,298.4477 C378.1421,299.6777 377.9081,300.9067 377.5681,302.7227 C380.9321,304.1487 384.1661,305.6887 387.5181,306.9067 C396.9031,310.3147 406.2961,313.7087 415.7661,316.8687 C420.6171,318.4867 423.6211,321.3877 423.8511,326.5717 C424.0831,331.8087 424.7051,337.1567 423.9541,342.2797 C422.8701,349.6767 420.1051,351.7427 412.3321,351.5837 C399.2571,351.3177 386.1871,350.6867 373.1221,350.0527 C368.2331,349.8147 363.3741,348.9257 358.4851,348.7277 C354.1381,348.5507 351.3731,346.4947 349.5071,342.7877 C349.1121,342.0057 348.4561,340.5347 347.9201,339.7617 C346.9171,340.4457 346.6511,340.7557 346.3301,341.0717 C341.5651,345.7637 336.0911,345.8207 330.1871,343.7567 C326.4211,342.4407 322.5511,341.4177 318.7901,340.0897 C309.9601,336.9707 307.5201,333.4257 307.9991,321.9137 C308.1331,318.6977 309.1011,315.4707 310.0051,312.3407 C313.3281,300.8317 316.8181,289.3707 320.1711,277.8707 C321.1871,274.3867 322.0601,270.8527 322.8301,267.3057 C323.7511,263.0647 324.6621,258.9217 330.2251,258.9937 C330.8891,254.6977 331.4641,250.9727 332.0831,246.9657 C328.1111,245.6377 324.4991,244.5257 320.9561,243.2257 C314.5911,240.8917 313.7091,239.8717 314.3661,233.2937 C316.2081,214.8557 318.3221,196.4447 320.2731,178.0167 C321.8221,163.3827 323.2951,148.7427 324.7771,134.1017 C324.8831,133.0507 324.7911,131.9807 324.7911,130.2527 C317.0141,130.2527 309.4261,130.3527 301.8421,130.2317 C286.5781,129.9897 271.3041,129.8897 256.0561,129.2257 C245.0401,128.7447 233.9131,128.3287 223.3591,124.5527 C222.7771,124.3447 221.2061,123.8917 220.7151,123.8867 C220.7151,123.8867 219.1501,124.9947 217.8431,125.6847 C205.8061,132.0517 193.8351,138.5487 181.6801,144.6837 C173.5561,148.7847 164.9231,151.5637 155.9621,153.4007 C144.5201,155.7447 133.1951,155.2637 121.9181,152.6287 C106.9371,149.1297 92.9971,143.0677 79.7701,135.2697 C62.6481,125.1767 50.8761,110.6497 43.8091,92.1737 C40.0351,82.3077 36.9811,72.2437 35.4441,61.7687 C35.1291,59.6207 34.8751,57.4087 35.0661,55.2677 C35.0821,55.0877 35.1471,54.9117 35.2461,54.7397 C36.0331,53.3807 38.0301,53.4127 38.9581,54.6807 C39.0211,54.7667 39.0821,54.8547 39.1411,54.9447 C39.3981,55.3407 39.2161,56.0107 39.2701,56.5527 C40.9211,72.8827 45.9331,88.1867 53.5911,102.6597 C58.7081,112.3317 66.1671,120.5937 75.1571,126.8327 C93.3141,139.4327 113.1311,147.9127 135.3391,150.3077 C145.0951,151.3597 154.5331,149.7167 163.8211,146.9467 C177.4911,142.8697 190.1771,136.4607 202.4021,129.3407 C230.5001,112.9737 258.4861,96.3977 286.1721,79.3457 C307.0821,66.4667 328.5481,54.6717 350.4341,43.5887 C350.7581,43.4247 351.0741,43.0687 351.3821,43.0817 C351.6811,43.0957 351.9811,43.1197 352.2801,43.1517 C353.7971,43.3117 354.6111,45.0227 353.8131,46.3227 L353.7991,46.3457 C353.3941,46.9947 352.3241,47.2287 350.5291,48.1977 C354.7931,49.7427 357.4621,50.7097 360.1301,51.6757 C368.2071,54.5997 375.7621,58.4797 382.6581,63.6487 C398.4571,75.4897 406.1971,91.8667 409.1541,110.8567 C410.0181,116.4067 409.9971,120.8717 410.6701,126.4567 C410.7841,127.4067 410.6091,129.9877 410.6721,130.5877 C410.6721,130.5877 412.0391,132.4307 412.5811,133.1787 C418.4431,141.2737 424.6021,149.1527 430.5021,157.2207 C432.3641,159.7667 434.2931,160.3407 436.9501,158.7367 C444.5621,154.1417 452.4761,149.9757 459.7221,144.8627 C469.3531,138.0677 479.0751,131.2567 486.7861,122.1257 C490.9661,117.1777 491.2341,116.7827 487.4471,111.3097 C476.1781,95.0247 464.8601,78.7687 453.2281,62.7427 C439.3511,43.6257 425.2251,24.6867 411.0031,5.8247 C406.4191,-0.2553 401.3621,-6.0063 396.2201,-11.6323 C386.5781,-22.1823 374.3971,-26.8403 360.1711,-26.7103 C345.0191,-26.5713 330.0771,-24.7403 315.3571,-21.2923 C274.4791,-11.7143 234.0611,-0.5713 195.2831,15.7737 C182.7431,21.0587 170.5081,27.0687 158.1361,32.7527 C157.4771,33.0557 156.8461,33.6687 156.1951,33.6797 C155.4241,33.6917 154.2781,33.4197 153.9661,32.8817 C153.6241,32.2897 153.7291,30.9207 154.1951,30.4797 C155.3591,29.3817 156.7561,28.4497 158.2041,27.7487 C164.4151,24.7427 170.6451,21.7697 176.9281,18.9177 C212.9231,2.5797 250.3421,-9.6143 288.6511,-19.1403 C302.0341,-22.4683 315.4701,-25.6723 329.0131,-28.2453 C342.7841,-30.8603 356.7421,-31.8533 370.7791,-30.1683 C377.9981,-29.3023 384.4581,-26.6523 390.2971,-22.4013 C399.2141,-15.9113 406.7671,-7.9583 412.8951,1.0757 C428.6281,24.2687 446.3931,45.9237 463.0011,68.4477 C473.1331,82.1877 482.5661,96.4467 492.1781,110.5647 C496.0591,116.2657 495.7591,118.2907 491.2731,123.5747 C490.0961,124.9597 488.9171,126.3437 487.7071,127.7007 C486.5081,129.0437 485.2701,130.3537 483.9661,131.7687" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Body" transform="translate(-101.000000, 239.000000) scale(1.0103092783505154 1)">
                    <g id="Body/Drawings" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M80.9074874,55 L8,246.958048 L18.3355116,256.047728 L28.0192646,260.216544 L25.014775,270.52318 L322.093791,442 L409,252.659351 L386.669782,237.794704 C386.669782,152.249127 363.382027,103.056371 316.806516,90.2164383 C305.283725,87.0398362 276.480995,86.3498887 267.585288,86.3498887 C251.359501,86.3498887 227.669318,97.2589783 196.514739,119.077157 L80.9074874,55 Z" id="Fill" fill="#FFF"/>
                        <path d="M77.9255499,53.8903639 C78.725041,52.0083753 80.2011013,51.6197906 81.9459906,52.3360252 C83.28214,52.884439 84.5323442,53.6486223 85.7975388,54.3578643 C121.427857,74.3225269 157.057177,94.2901863 192.682499,114.266836 C193.617009,114.791208 194.619998,115.271226 195.480959,115.890141 C197.058246,114.712297 198.655476,113.556148 200.273466,112.423706 C217.696375,100.229738 236.406465,90.8597526 257.376116,86.5283818 C261.628409,85.6503203 266.001625,85.3636267 270.317878,84.7922373 C272.826281,84.4605918 274.175422,85.4525316 275.249738,87.8769405 C281.391828,101.732133 291.630311,114.875086 301.226202,127.41868 C300.834452,125.068192 300.791479,117.504276 300.732517,113.304764 C300.643573,106.959546 300.693541,98.2988021 300.732517,91.9555815 C300.767494,86.4854277 301.407087,86.0458975 306.628763,85.757206 C323.170233,84.8441818 336.891499,92.5229751 348.567066,104.24245 C363.762393,119.494149 374.054842,137.630766 379.790191,158.339633 C386.48293,182.499812 389.228183,207.104515 387.797094,232.152745 C387.725139,233.417393 387.790098,234.745973 387.401346,235.914724 C387.375002,235.993843 387.335937,236.075068 387.286495,236.156621 L390.631489,237.9956 C396.199945,241.05533 401.777394,244.099077 407.327861,247.189774 C412.856341,250.268483 412.923299,250.367378 410.395908,255.980379 C401.683454,275.342685 392.941019,294.693004 384.176598,314.033334 C365.375567,355.522992 346.555547,397.003659 327.744522,438.489321 C327.594617,438.819968 327.474693,439.1636 327.321791,439.493248 C324.84137,444.84353 324.414641,445.046313 319.257924,442.108453 C288.339606,424.498274 257.443273,406.849137 226.540945,389.211987 C160.321099,351.419378 94.1012529,313.627768 27.8844048,275.829165 C26.7671161,275.191846 25.662819,274.530553 24.5175481,273.84129 C23.1254343,273.003186 22.484842,271.321982 22.9705329,269.771639 C23.8679616,266.9077 24.73441,264.139658 25.6598209,261.186814 C19.9204745,258.283917 14.5988621,255.613771 9.29623757,252.905665 C4.36237834,250.386357 4.22746422,250.094669 6.15123959,244.965151 C29.670268,182.283143 53.2012887,119.60613 76.7323095,56.9281173 C77.1150658,55.9102052 77.4988215,54.8912942 77.9255499,53.8903639 Z M102.078175,84.3597995 C76.7742828,146.63724 51.7422175,208.245396 26.6032204,270.120267 C125.496268,326.549957 223.966584,382.738905 322.745704,439.103664 C351.092659,376.522548 379.159792,314.559772 407.415805,252.182438 C391.285073,243.316913 375.164335,234.455384 359.042598,225.594854 L360.391764,226.335228 C359.924716,230.269553 358.859743,234.134791 357.73523,237.969528 C357.026762,240.382852 355.365099,242.346415 353.629482,243.099011 C354.01471,243.548455 354.383795,244.008261 354.73554,244.475974 C360.916605,252.697188 366.671941,261.239059 372.595171,269.654065 C373.706463,271.233377 375.201512,273.682759 376.402747,275.398925 C376.605618,275.201136 376.757521,275.024325 376.88444,274.857504 C377.399113,274.175233 377.918782,273.49496 378.506408,272.872625 C379.556739,271.761812 380.653041,270.707939 382.172074,272.180365 C382.926594,272.911583 382.559827,275.498818 381.903245,276.852372 C379.793588,281.211713 377.440087,285.47116 374.878717,289.580768 C374.662854,289.9254 374.300085,290.179129 373.864363,290.387905 C372.37631,291.100144 370.521491,290.082232 370.547474,288.432995 C370.550472,288.215228 370.587449,288.014442 370.671395,287.836632 C371.027169,287.081439 371.712732,285.397239 371.739715,284.777901 C371.739715,284.777901 369.116385,281.764122 367.12965,280.070932 C367.551381,282.927878 368.043068,285.775835 368.378855,288.642771 C368.977474,293.747315 367.579364,298.234519 364.043614,302.033459 C359.047795,307.398725 352.392032,307.641466 347.096403,302.499961 C340.784421,296.369513 339.361327,288.659752 340.99029,280.461514 C342.611258,272.310226 346.015091,264.957085 353.954037,259.840553 C349.763705,254.648103 345.902163,249.364749 340.302727,245.718646 C336.711014,243.380145 334.811223,243.873618 332.754532,247.652579 C328.848019,254.823915 329.563564,262.240988 331.943049,269.673045 C332.236862,270.590065 332.69457,271.461134 333.13529,272.322213 C333.95177,273.920505 334.921153,275.820475 332.669586,276.653584 C331.838116,276.961255 329.860375,275.621687 329.137835,274.543839 C322.50006,264.645418 325.058431,245.774586 333.958766,237.925975 C334.648869,237.317183 335.35391,236.793962 336.071449,236.362435 C335.975849,236.306544 335.883156,236.24445 335.793198,236.178442 C335.642294,236.067561 334.808825,235.083612 334.754859,235.088607 C329.715067,235.567096 326.94583,234.126635 327.487485,228.155017 C327.696352,225.843488 328.26499,223.565922 328.853615,220.228489 C326.186313,222.553005 324.297516,224.373059 322.227833,225.955368 C320.079201,227.599611 317.816641,229.464618 314.848531,227.800396 C312.782846,226.643633 312.265175,223.589896 313.664285,219.832912 C315.986806,213.604568 318.343306,211.383942 322.779482,206.577079 C322.924975,206.419436 323.136363,206.223998 323.396471,206.003054 C292.470777,189.005194 261.540049,172.0046 230.578875,154.987316 L237.376398,158.724278 C236.203199,160.106655 235.162279,161.397858 233.962621,162.518966 C219.10308,176.40113 204.030675,190.060531 189.386996,204.166455 C183.04988,210.271806 177.377206,217.064102 171.445811,223.594865 C180.955785,232.270423 190.837869,240.44818 201.000043,248.24984 C201.146552,247.942956 201.297863,247.641224 201.417538,247.3523 C201.82328,246.375344 202.235018,245.401385 202.754687,244.480369 C212.482494,227.239795 222.218297,210.004216 231.983081,192.78462 C232.873514,191.2123 233.883871,189.692924 234.978174,188.256459 L234.978174,188.256459 L235.008155,188.2185 C236.403267,186.418424 239.231467,188.16256 238.317049,190.24833 C237.978264,191.021504 237.613497,191.809662 237.188767,192.563856 C226.710437,211.195942 216.198129,229.810049 205.698813,248.430148 C205.520926,248.744812 205.404,249.095437 205.220117,249.406105 C204.947933,249.865343 204.684635,250.379605 204.379063,250.821598 C217.267787,260.555733 230.586584,269.713083 244.151932,278.538975 C244.47723,277.83359 244.939341,277.148562 245.340278,276.481069 C254.5744,261.110496 263.824511,245.750912 273.077621,230.392326 C273.729206,229.310482 274.279856,228.101774 275.166291,227.25468 C275.67197,226.772195 276.972142,226.541442 277.534784,226.863098 C278.094427,227.182756 278.541143,228.409445 278.394237,229.101706 C278.1384,230.30442 277.476821,231.45519 276.830232,232.534037 C267.703042,247.757767 258.569856,262.976502 249.37471,278.158276 C248.845047,279.031343 248.169477,279.816504 247.391972,280.689571 L247.42011,280.656017 C249.934416,282.277727 252.456643,283.888786 254.985638,285.490739 C258.820197,287.919144 262.82065,290.085828 266.619232,292.566177 C268.128272,293.551125 270.967464,294.52908 270.967464,294.52908 C272.040781,293.813844 273.432895,293.123582 274.904958,291.997785 C279.915255,288.167984 285.147461,284.596336 290.496086,281.240002 C289.839113,277.848645 289.176591,274.457922 288.503601,271.069952 C287.511233,266.067298 287.697115,261.43425 291.951406,257.726213 C294.024087,255.918145 296.060791,255.434662 298.190435,257.236736 C300.259118,258.988863 302.341792,260.804923 304.04171,262.897686 C312.325437,273.091791 320.582181,283.308871 328.83093,293.530946 L325.958844,289.972743 C327.37194,290.171386 328.781567,290.448156 330.186767,290.819045 C332.983986,291.558255 336.138978,293.22847 336.212931,296.763691 C336.248556,298.487292 335.561278,299.695123 334.518495,300.584533 C338.894968,306.013489 343.272146,311.442146 347.647652,316.870802 C349.469492,319.130387 351.736049,322.148161 353.797737,324.829296 C354.866057,323.587623 355.557616,322.108204 356.318132,320.841558 C356.951729,319.787684 358.446777,319.474019 359.328216,320.332102 C360.085734,321.069315 360.045759,322.156153 359.512099,323.34688 C356.415071,330.255497 353.245089,337.134145 349.935195,343.941869 C349.860243,344.096704 349.743318,344.230561 349.596411,344.349434 C348.430154,345.291427 346.67627,344.614151 346.201572,343.19267 C346.075652,342.816073 345.999701,342.441473 346.018689,342.070869 C346.085646,340.804223 346.915118,339.577534 347.474762,338.179028 C344.713872,334.76828 341.972208,331.381782 339.201131,327.957911 C338.870538,329.545851 337.914738,331.017133 336.13498,332.105919 C340.474218,335.905858 339.334943,339.155385 333.894407,341.175226 C333.216826,341.426743 332.5247,341.652348 331.822404,341.853738 C331.675875,343.527688 330.139867,343.982384 328.73609,344.576091 C323.290201,346.877895 317.687889,348.272156 311.923734,348.794084 L311.616388,349.269575 C311.342563,349.697118 310.63901,350.787952 310.63901,350.787952 C310.63901,350.787952 311.175669,352.439188 311.373543,353.07351 C313.965893,361.395615 317.343742,370.812551 320.04902,379.100693 C320.126971,379.341436 320.816532,378.283566 320.836519,378.244608 C320.876494,378.16769 320.915469,378.089773 320.954444,378.010858 C321.178301,377.557343 321.381172,377.093838 321.60503,376.639324 C322.473477,374.881203 323.356915,373.052157 324.575139,371.501814 C324.784006,371.223112 325.113796,371.032316 325.50055,370.886472 C326.923644,370.350045 328.588584,371.299031 328.624561,372.817408 C328.628559,373.002211 328.600577,373.17103 328.525624,373.314877 C325.236718,379.701051 321.821892,386.02729 318.215188,392.238652 C317.732495,393.068765 315.659814,393.828952 314.871316,393.458348 C312.986516,392.571296 313.814989,390.85513 314.695428,389.541534 C316.036574,387.54367 317.29977,384.66175 317.29977,384.66175 C317.29977,384.66175 316.101533,380.967698 315.213099,378.870939 C311.093656,369.144649 307.815689,359.146215 305.032337,349.007095 C304.796703,349.000436 304.5593,348.992602 304.321632,348.983381 C290.3905,348.442959 277.156924,344.73692 264.127219,340.141831 C260.094085,338.719559 256.112333,337.190347 252.178292,335.562041 C245.579791,344.389874 238.968075,353.208167 232.311672,361.991878 C229.862231,365.225422 229.241626,365.286357 225.525991,363.01978 C217.942818,358.392725 210.428602,353.655787 202.886403,348.960805 C188.573514,340.051327 174.251631,331.153837 159.952733,322.222382 C155.215749,319.263544 155.068842,318.732112 158.108907,314.218936 C162.601048,307.551062 167.125168,300.905165 171.6303,294.246282 L170.995959,295.182399 C165.376838,292.252544 159.789257,289.264106 154.233774,286.224955 C149.445618,283.60547 144.960517,280.674466 140.784124,277.438989 C140.194528,277.264889 139.633434,276.89987 139.114898,276.3582 C138.89684,276.130361 138.714434,275.89996 138.565799,275.667744 C127.497768,266.562622 118.725217,255.209684 112.362428,241.751486 C110.069888,236.901669 108.484897,231.535405 107.603458,226.236068 C107.570325,226.036785 107.538541,225.837837 107.508107,225.639221 C104.535357,227.978927 102.27628,231.017313 100.570934,234.45918 C99.9763129,235.658898 100.363067,237.600822 100.833767,239.024301 C102.801514,244.965951 103.68795,251.028471 103.121311,257.246825 C102.927434,259.378547 102.38378,261.541236 101.616269,263.543096 C100.804786,265.658835 99.2577704,267.219167 96.6874066,267.129263 C94.1310339,267.03836 92.6899513,265.408103 91.9933947,263.237423 C91.3368126,261.192608 90.8761059,259.028921 90.7272007,256.890206 C90.3204596,251.024475 91.1369399,245.253643 93.3545282,239.822448 C94.5867438,236.804673 94.3079213,234.43121 92.5630321,231.725102 C84.5421379,219.285397 72.6607014,219.034665 64.9206285,231.613221 C57.9540632,242.934122 55.8434068,255.161054 60.0697165,268.099227 C60.4814544,269.360878 60.9931286,272.710299 58.3618037,271.934128 C56.8277802,271.481612 55.4146797,268.388917 55.0948833,266.953451 C51.7600062,251.971463 54.1844629,237.994401 63.7673626,225.859371 C72.2359717,215.134833 84.6640603,215.345607 93.5464061,225.825407 C94.7146624,227.203934 95.7550001,228.690345 97.1351216,230.493418 C100.180386,226.152845 103.560597,222.876492 107.029446,220.741648 C106.632364,211.08199 109.620562,202.2376 116.03509,194.108405 C120.718155,188.172785 125.547411,182.382281 130.540375,176.744534 C130.312432,175.298131 130.089364,173.825182 129.841202,172.315796 C125.301092,172.827249 121.197704,173.384654 117.077327,173.713302 C114.938688,173.88412 112.688121,173.996 110.63143,173.520508 C107.49043,172.793286 106.696935,170.59064 108.965491,168.352032 C112.301367,165.06055 116.09795,162.227578 119.763617,159.281726 C120.869913,158.392676 122.195069,157.777334 123.831028,156.787392 C122.53985,149.653017 121.253668,142.556601 119.973483,135.458188 C119.55475,133.141664 118.97312,130.832132 118.826214,128.497627 C118.771249,127.630554 119.546755,126.277999 120.30927,125.918383 C121.043802,125.571754 122.561836,125.895408 123.18344,126.495766 C124.600538,127.866302 125.783785,129.516538 126.874091,131.175765 C130.350877,136.468108 133.742718,141.816392 137.537303,147.717086 C139.324165,146.40349 140.901161,145.330636 142.381219,144.137911 C144.780692,142.202979 146.989286,139.998335 149.539662,138.297154 C150.696925,137.525978 152.68466,137.14938 153.905883,137.614883 C154.738353,137.931544 155.365953,140.057272 155.260021,141.304939 C155.030167,143.991068 154.386577,146.662213 153.701013,149.284411 C153.280431,150.893148 152.841687,152.49798 152.390564,154.111508 L153.043532,153.491814 L153.043532,153.491814 C162.257006,144.802135 171.344025,135.97246 180.9041,127.684719 C154.667969,113.264491 128.398314,98.8259974 102.078175,84.3597995 Z M214.136242,318.746097 C218.732316,331.922016 223.697155,344.829221 229.34556,357.716448 C232.938981,354.240362 241.67609,342.844559 248.066215,333.814064 C238.088123,329.477021 228.41482,324.507561 218.983788,319.039222 C216.965356,318.940397 215.295509,318.846528 214.136242,318.746097 Z M209.169403,318.562294 C197.537808,316.889082 170.193215,317.801107 164.839623,320.140607 C184.696982,332.504393 204.3045,344.713345 223.917015,356.925293 C219.029127,344.210882 214.145236,331.504462 209.169403,318.562294 Z M306.487853,279.091579 C303.38283,279.341312 300.734515,280.326259 298.301064,281.743744 C292.817555,284.939329 287.340042,288.190854 282.141351,291.822972 C272.960196,298.238116 273.052137,298.365979 270.952474,309.557018 C270.717623,310.804685 270.380838,312.03537 270.184962,313.288031 C269.300525,318.943986 268.449067,324.604936 267.481683,330.951153 C276.881699,332.836139 285.532193,334.759083 294.252642,336.272466 C304.269265,338.010608 314.326863,339.609899 324.57334,338.850711 C326.894862,338.678894 331.178136,338.391202 333.894407,337.059625 C334.631937,336.698011 334.153242,335.406392 333.350753,335.227583 C331.214113,334.751092 327.945194,334.599254 326.324226,334.426439 C316.942198,333.423511 307.547179,332.530465 298.157156,331.593467 C296.892961,331.466602 295.548816,331.532532 294.395551,331.098995 C294.013794,330.955149 293.683004,330.676447 293.370203,330.350795 C292.704627,329.656537 292.717619,328.559709 293.398186,327.879436 C293.703991,327.573763 294.026785,327.297059 294.390554,327.097272 C294.940204,326.796593 295.812649,327.035338 296.53219,327.104265 C306.107095,328.021285 315.687996,328.892354 325.252908,329.90627 C327.688357,330.164993 330.870332,330.147013 333.147882,329.12011 C334.732873,328.404875 335.136616,326.175257 333.62258,325.319172 C332.77412,324.839685 330.702439,324.261303 329.847983,324.084492 C324.718248,323.026623 319.53055,322.139571 314.323865,321.586162 C308.732424,320.990798 303.092015,320.860937 297.474591,320.505317 C296.018517,320.412417 294.112731,320.303533 294.536461,318.35062 C294.747327,317.381656 297.079842,316.498599 297.928302,316.316794 C298.8737,316.115009 299.989989,316.429673 300.948379,316.499598 C302.243555,316.594497 303.537731,316.728354 304.830908,316.84423 C307.359298,317.069989 309.887689,317.295748 312.41508,317.520507 C317.471861,317.970027 322.529641,318.411555 327.58942,318.821117 C331.134164,319.10881 335.358475,319.638244 338.977171,318.302671 C340.633117,317.690326 341.024868,315.43174 339.597776,314.393849 C337.662008,312.988351 334.881778,312.804548 332.687175,312.628735 C322.203849,311.790631 311.701534,311.185278 301.203217,310.53797 C300.107914,310.470043 299.008614,310.474038 297.902318,310.490021 C296.02951,310.516992 294.717346,308.464186 295.692725,306.866893 C296.428257,305.661182 297.889327,305.235637 299.47132,304.909985 C307.822004,303.184829 316.182681,301.509619 324.513378,299.69356 C326.283252,299.306973 328.042132,299.19809 330.347664,298.368976 C331.291709,298.029597 331.513634,297.354233 331.421295,296.743347 C330.557075,295.672635 329.694221,294.60209 328.83103,293.531546 L330.316,295.374 L330.215488,295.331431 C325.779169,293.503826 321.633745,293.7612 317.765674,294.117519 C310.737148,294.764828 303.747597,295.869647 296.759046,296.90554 C295.950561,297.025412 295.152069,297.255166 294.397549,297.199226 C292.660655,297.069365 291.949108,294.883701 293.244283,293.719944 C295.451878,291.735066 297.767404,289.671271 300.036959,287.55753 C302.406451,285.351888 305.4715,283.818527 307.530189,280.860688 C308.07884,280.07253 307.445244,279.013662 306.487853,279.091579 Z M254.464355,289.662075 L254.399265,289.86684 C254.071183,290.823727 253.543081,291.771152 253.03528,292.62961 C248.051452,301.052608 242.977682,309.420664 237.93789,317.810697 C237.446121,318.629446 236.970895,319.458234 236.413638,320.424197 C241.874048,322.462039 247.390537,324.37083 252.972919,326.127309 C255.028611,326.773618 257.088299,327.405942 259.638676,328.199095 C262.653757,318.207773 265.562905,308.563081 268.452066,298.984318 C267.81647,298.403938 267.467692,297.979392 267.026973,297.695695 C262.849632,295.002691 258.658213,292.330558 254.464355,289.662075 Z M230.032484,273.993989 L229.994422,274.077013 C229.756948,274.587592 229.471879,275.138004 229.111609,275.633474 C221.523439,286.053338 213.909286,296.453223 206.249162,306.820143 C206.05856,307.078119 205.869511,307.315113 205.681778,307.531512 C208.148342,308.667684 210.622558,309.787321 213.105592,310.886634 C213.342326,310.89733 213.565754,310.940722 213.771274,311.026947 C216.017438,311.968329 218.226339,312.998778 220.425347,314.053338 C223.314792,315.276103 226.217639,316.467094 229.13526,317.62398 L228.910637,317.971525 C232.049638,313.069764 235.246603,308.278884 238.233702,303.360141 C241.242786,298.403439 244.024016,293.307885 246.954151,288.301236 C247.430741,287.486407 247.91985,286.573179 248.556731,285.902329 C242.362492,281.961947 236.175439,278.010546 230.032484,273.993989 Z M174.87021,297.190514 L173.015562,299.929136 C169.775161,304.714614 166.522351,309.518759 163.235644,314.372771 C178.178132,314.002168 192.239181,313.652541 206.317219,313.303914 C206.291525,313.078653 206.24404,312.806963 206.177683,312.495599 C204.540413,311.74342 202.907198,310.983674 201.276828,310.216314 C192.40062,306.038818 183.597721,301.687133 174.87021,297.190514 Z M321.168411,304.78171 L321.081411,304.799904 C318.588844,305.338751 316.167548,305.876523 314.40981,306.386407 C315.941567,306.657125 321.202878,306.77723 322.795914,306.996889 C322.262904,306.267811 321.720301,305.528577 321.168411,304.78171 Z M207.707881,258.763947 L207.51911,259.0276 C207.086496,259.610532 206.614796,260.168603 206.140432,260.716018 C196.986259,271.284722 187.843079,281.864415 178.624947,292.377179 C178.215025,292.844779 177.747631,293.262179 177.279468,293.635727 C184.581724,297.37093 191.917446,301.031094 199.315438,304.548001 C199.45623,304.03704 199.68036,303.492147 199.988148,302.913319 C200.582769,301.796512 201.325297,300.75063 202.076818,299.728722 C208.743574,290.659415 215.415327,281.592106 222.125056,272.553766 C222.747002,271.716138 223.403893,270.833524 224.167799,270.12358 C222.447893,268.984351 220.733076,267.834496 219.023531,266.676847 C215.209805,264.094452 211.437298,261.45725 207.707881,258.763947 Z M355.350148,264.427651 C353.585272,266.137823 351.960306,267.23565 350.977932,268.749032 C346.963487,274.925431 345.727274,281.671222 347.878904,288.791612 C348.421559,290.585694 349.500872,292.383772 350.79205,293.747315 C352.934686,296.007899 355.585998,298.317431 358.902887,297.268552 C362.466618,296.142755 362.732449,292.457693 362.661494,289.436922 C362.460622,280.776178 359.941226,272.710798 355.350148,264.427651 Z M307.387253,286.958023 L307.177807,287.13112 C305.310662,288.65755 303.249914,290.322268 301.487036,291.934852 C304.257273,290.964889 307.927936,290.427463 309.557899,290.237666 L309.973968,290.188224 C309.101643,289.08976 308.237347,288.010414 307.387253,286.958023 Z M186.504769,242.447624 L186.385925,242.596333 C185.727863,243.432074 184.904678,244.530787 183.86741,245.364325 C178.913564,249.34507 173.818807,253.150004 168.826984,257.083799 C161.551616,262.816672 154.324216,268.609481 147.064838,274.360335 C146.610327,274.72035 146.151019,275.07589 145.6857,275.423439 L145.585983,275.497528 C148.559391,277.702649 151.684643,279.766596 154.959312,281.690801 C160.269232,284.811706 165.73221,287.679643 171.208374,290.510266 L171.242647,290.423268 L171.242647,290.423268 C171.86525,288.954837 172.90259,287.611273 173.957918,286.387581 C182.978176,275.942744 192.044405,265.536865 201.10064,255.123994 C201.315451,254.877315 201.530263,254.628405 201.75175,254.386587 C196.580928,250.520547 191.496793,246.541866 186.504769,242.447624 Z M293.269567,271.25875 C293.695257,273.785316 294.129718,276.305744 294.570917,278.822358 C295.982945,278.100245 297.466132,277.494438 298.968347,276.924835 C296.279712,273.886866 294.230236,271.807804 293.269567,271.25875 Z M168.35903,226.667764 L168.219732,226.760189 C167.642252,227.131985 167.012603,227.441126 166.396631,227.692312 C151.61704,233.714875 136.817461,239.685494 122.019881,245.662107 C121.262223,245.968341 120.499487,246.271192 119.726163,246.536336 C125.082892,255.822973 131.76968,263.899393 139.727241,270.798993 C140.276426,270.239092 140.883782,269.727408 141.497382,269.243803 C153.759576,259.571141 166.026767,249.905471 178.322939,240.275763 C179.30069,239.510334 180.372085,238.774858 181.507936,238.282645 C177.047169,234.509075 172.662747,230.637565 168.35903,226.667764 Z M97.3759683,243.03002 C94.7526382,249.468139 94.2079849,255.021204 95.741009,259.363563 C96.1977183,260.660177 98.1164969,260.527319 98.4063124,259.182756 C99.531596,253.950348 98.9069936,248.844805 97.3759683,243.03002 Z M81.3933424,57.1488814 C57.5205392,120.773882 33.7696584,184.072231 9.81990422,247.901014 C11.4888418,248.846004 12.9818914,249.745043 14.5249091,250.547186 C15.9510013,251.288393 17.4300598,251.924713 19.2568969,252.780798 C19.9904299,251.140551 20.7389534,249.752035 21.248629,248.281607 C31.5910453,218.414528 44.4698469,189.574351 56.7690176,160.499425 C64.1243354,143.111009 71.3657257,125.671647 78.9528959,108.384123 C82.1598544,101.076933 85.5986654,93.8576486 89.6560825,86.9859927 C91.05819,84.6115305 92.6631683,83.3708565 97.1493125,84.3597995 C98.0767222,84.5635817 99.5228016,80.5169066 101.130778,79.1703458 C103.013579,80.1153358 105.116241,81.088296 107.142951,82.2011066 C133.982865,96.9493443 160.82178,111.701578 187.661695,126.451813 L184.428533,124.674123 C186.798434,122.68091 189.201047,120.725135 191.645157,118.816793 C191.208672,118.561507 190.775379,118.297269 190.33899,118.05279 C155.972867,98.8003661 121.61174,79.5399508 87.245617,60.2875269 C85.5167176,59.3195614 83.7538398,58.4125308 81.3933424,57.1488814 Z M243.714113,204.725857 C244.694489,202.89881 245.480988,199.257702 248.437107,201.031805 C250.814593,202.458281 248.418119,204.931637 247.47372,206.732713 C240.246321,220.512985 232.95896,234.261291 225.682592,248.014592 C225.006022,249.292227 224.375424,250.614813 223.525964,251.772576 C223.421031,251.915423 223.298109,252.045284 223.159198,252.168153 C222.062896,253.139115 220.25005,252.789489 219.750368,251.41296 C219.682411,251.227159 219.645435,251.045353 219.649138,250.863547 C219.675416,249.864615 220.352984,248.850699 220.854665,247.899715 C228.446832,233.494111 236.014015,219.074522 243.714113,204.725857 Z M260.515318,211.90898 C261.031989,211.44148 262.341156,211.261672 262.901799,211.60031 C263.437458,211.923964 263.808222,213.166636 263.644326,213.862892 C263.362505,215.061611 262.675943,216.189406 262.043345,217.279241 C255.947226,227.773026 249.855104,238.270806 243.684032,248.721637 C243.112396,249.689602 242.39985,250.574656 241.577373,251.566596 C240.584006,252.766314 238.633247,252.619471 237.945685,251.221964 C237.860739,251.04815 237.805774,250.879331 237.786786,250.706515 C237.706837,249.957316 238.445367,249.090243 238.895081,248.317069 C245.370958,237.204945 251.852832,226.095818 258.352695,214.997679 C258.989289,213.909842 259.604897,212.730103 260.515318,211.90898 Z M355.323765,216.557412 C354.82908,218.282569 352.510556,229.351738 349.965176,233.268552 C349.314591,234.267485 350.707704,241.058227 352.972262,240.047307 C353.146151,239.969391 353.277068,239.808563 353.360015,239.637745 C357.273524,231.578359 357.487388,226.667607 357.312499,220.874798 C357.271525,219.522244 356.649921,216.848102 355.323765,216.557412 Z M128.804577,185.27508 L128.044859,186.181218 C124.397056,190.56336 120.888001,195.067552 117.48017,199.667464 C112.702212,206.118569 111.012288,213.2939 111.390047,221.188463 C111.689856,227.454766 113.150926,233.375438 115.807235,239.027397 C115.958279,239.34883 116.110733,239.668982 116.264596,239.987854 L116.254551,239.993165 C117.196951,239.459735 118.232292,239.085136 119.24065,238.675573 C133.185773,233.007631 147.069934,227.178861 161.128985,221.809599 C161.614247,221.624375 162.029681,221.405728 162.390596,221.161922 C161.165888,220.036703 160.114795,218.797142 160.082051,216.98096 C160.421835,216.425553 160.552752,216.059944 160.804591,215.821199 L162.931,213.802 L160.007463,211.3808 C150.180593,203.241999 140.420001,195.160081 130.675371,187.058739 C130.017472,186.511745 129.382175,185.915417 128.804577,185.27508 Z M346.824176,234.751967 C346.2557,235.08578 345.618285,235.488951 344.944159,235.885674 C345.383108,236.101688 345.823771,236.354484 346.264932,236.645343 C346.583402,236.855363 346.900196,237.071462 347.214749,237.293365 C347.098267,236.529234 346.994973,235.475435 346.824176,234.751967 Z M348.347206,209.777658 L348.377187,210.935421 C347.498746,217.741147 343.761126,223.138379 339.965542,228.484665 C339.173046,229.60247 337.679997,232.052851 337.819908,233.169658 C337.936808,234.104453 338.729151,234.737008 339.692106,235.007155 C340.436503,234.893088 341.188901,234.879665 341.947258,234.971789 C342.288231,234.859734 342.600791,234.691658 342.860699,234.466272 C347.914482,230.084955 352.391632,216.302685 351.677087,213.269926 C351.35729,211.653653 349.875234,210.919438 348.347206,209.777658 Z M360.405807,218.635234 L360.414525,218.701121 L360.414525,218.701121 C360.534728,219.640911 360.603924,220.575625 360.628897,221.505749 C370.628444,227.002476 380.630842,232.498601 390.631489,237.9956 L383.460021,234.054238 C383.430293,233.194661 383.480905,232.320215 383.530809,231.46448 C383.778449,227.223233 383.901501,222.997535 383.900804,218.787513 C383.57205,218.849034 383.220368,218.884076 382.847644,218.88952 C375.369519,218.996883 367.886953,218.784603 360.405807,218.635234 Z M336.391817,214.203928 C334.415075,218.105758 331.133164,224.525896 331.129163,228.977139 C331.126169,231.891024 333.028958,231.948963 333.830447,230.967012 C338.917209,224.735672 346.410439,211.087259 344.06893,209.43902 C340.449234,206.513147 337.662008,211.693611 336.391817,214.203928 Z M332.55426,205.242505 C326.96182,206.397271 320.479946,214.86622 316.979174,221.514115 C316.268627,222.864671 317.640753,224.238203 319.112816,223.433064 C325.294881,220.053676 330.844348,214.048094 333.332764,207.654927 C334.170231,205.506223 333.839442,205.526202 332.55426,205.242505 Z M381.568998,187.378357 L381.51513,187.408968 C380.663385,187.85704 379.566048,188.051521 378.593952,188.202217 C371.626387,189.280065 364.643832,190.270007 357.667273,191.288918 C357.636293,191.03419 357.605313,190.778464 357.574333,190.523736 C354.673179,190.523736 351.764031,190.654596 348.872872,190.480782 C348.679258,190.469128 348.488922,190.449935 348.302983,190.422889 C348.138315,193.549593 347.972356,196.677071 347.826538,199.805316 C347.742591,201.606391 347.814545,203.415458 347.814545,204.458343 C351.410998,207.146181 354.823795,209.228764 357.609802,211.8997 C357.744517,211.890255 357.881442,211.88618 358.01845,211.888003 C360.902614,211.92796 363.780782,212.325535 366.662947,212.551294 C368.106028,212.664173 369.551109,212.731102 371.839652,212.870952 C374.79577,212.725108 378.590354,212.563281 382.380941,212.340519 C382.869901,212.311676 383.348219,212.302351 383.801673,212.334744 C383.548442,203.953103 382.801682,195.633774 381.568998,187.378357 Z M151.489134,161.742408 L150.116919,163.053441 C149.158872,163.961336 148.197253,164.865386 147.231232,165.764697 C142.429147,170.235212 137.888414,174.921361 133.553043,179.783855 C134.407711,180.404731 135.206614,181.120486 136.020968,181.797362 C146.695207,190.663202 157.362462,199.53864 168.339313,208.669165 L181.7,195.99 L180.743341,194.904244 C171.789452,184.746086 162.339901,174.024108 152.891229,163.302129 C152.429209,162.778178 151.94911,162.268191 151.489134,161.742408 Z M270.31388,88.8548953 C264.048869,89.9057722 257.861807,90.9446619 251.841639,91.9555815 C251.266006,99.3536747 250.794306,106.025544 250.219672,112.688423 C249.074401,125.988209 244.277455,138.045323 237.472786,149.37921 C236.79926,150.501886 235.760868,151.42851 234.775473,152.342958 C266.193299,169.608641 297.610741,186.873724 329.027205,204.139784 L327.307215,203.193264 C328.280865,202.578637 329.171206,202.082554 329.707072,201.899079 C334.924751,200.112987 337.819908,201.899079 337.819908,205.878825 C338.865242,204.904866 340.443238,204.599193 343.109541,205.380358 C343.338395,202.530404 343.722151,199.694435 343.766123,196.852472 C343.959999,184.34484 344.299783,171.830215 344.045944,159.328576 C343.963997,155.277905 342.722787,151.06341 341.17877,147.260474 C334.220199,130.115797 326.91485,113.110971 319.868335,96.0012576 C319.461594,95.0143124 319.113816,94.1941889 318.434248,93.1662875 C317.457313,91.3648584 315.856387,90.6048875 314.226926,90.6048875 C312.796837,90.6368533 312.137257,90.5988939 310.052584,90.5988939 C309.396002,96.3097904 308.761406,104.005703 308.082838,109.695622 C307.265358,116.542304 306.438884,123.388987 305.555447,130.227678 C305.323594,132.015767 305.089743,134.365256 302.899137,134.397222 C301.40209,134.420197 299.39137,133.329363 298.44997,132.09668 C289.85644,120.842708 281.298888,109.54778 274.788032,96.8873106 C273.47187,94.3280459 271.989814,91.8536904 270.31388,88.8548953 Z M174.113818,139.782968 L172.848919,140.983943 C167.462949,146.13369 162.174671,151.38903 156.834781,156.588972 C158.083824,157.848344 159.251291,159.198219 160.42943,160.530092 C168.967995,170.186771 177.48957,179.858435 186.024137,189.518111 C186.38979,189.931961 186.764186,190.337643 187.18396,190.785693 L203.483659,175.31661 C198.119281,169.082222 192.918876,163.10382 187.846977,157.016847 C183.433786,151.718509 179.218469,146.255348 174.957182,140.832144 C174.694334,140.4972 174.395113,140.144767 174.113818,139.782968 Z M372.775922,150.726117 L372.737895,150.746732 C372.356047,150.934033 371.940194,151.081036 371.509862,151.190374 C363.816759,153.145285 356.078685,154.921386 348.3566,156.764417 C348.247255,156.740294 348.138225,156.716818 348.029714,156.693661 C349.017935,162.974276 349.233919,169.37459 348.994794,175.881885 C348.895469,178.578726 348.772273,181.274768 348.638335,183.970474 C348.816116,183.955821 348.997997,183.945505 349.181675,183.938774 C355.695529,183.702027 362.216378,183.536204 368.71624,183.082689 C371.771295,182.868917 374.787375,182.069771 377.816447,181.516363 C378.704881,181.353537 379.750216,180.720214 380.424787,181.006907 L380.526765,181.050182 C378.800337,171.465315 376.415971,161.967649 373.384268,152.558811 C373.186476,151.945036 372.983683,151.334142 372.775922,150.726117 Z M124.689106,136.313525 L124.64451,136.320266 C125.689844,142.961169 127.154912,151.262297 128.266204,158.194888 C128.400119,159.031993 127.968394,159.852117 127.207878,160.230712 C122.402937,162.620158 118.035717,165.077532 114.07224,169.777509 C119.195978,169.107225 123.251397,168.64372 127.285828,168.030376 C131.951765,167.320522 133.376351,168.064939 134.320412,172.540869 C138.147379,168.347747 142.07013,164.242948 146.095945,160.229352 C147.539108,154.846011 149.035511,149.142774 150.491057,143.825245 C147.68984,145.338628 145.860005,146.946909 144.069145,148.41534 C137.426373,153.86052 136.074234,153.648746 131.52513,146.535349 C129.653321,143.608477 127.719552,140.718566 125.813765,137.812671 C125.813765,137.812671 124.874363,136.185411 124.64451,136.320266 Z M200.514474,117.503508 L200.483062,117.526887 C193.198911,122.79587 186.250297,128.567154 179.551024,134.698628 C180.333351,135.501171 180.989328,136.511866 181.716879,137.384829 C190.776112,148.264201 199.831348,159.146571 208.867596,170.043924 L208.943563,170.137436 C215.992496,163.445991 223.040061,156.753118 230.085027,150.057561 C230.006471,149.979903 229.928671,149.899463 229.850938,149.816342 C220.440928,139.758092 211.130855,129.60794 201.851762,119.428819 C201.340871,118.868169 200.883481,118.206474 200.514474,117.503508 Z M357.724773,121.717806 L357.660693,121.780791 C357.407311,122.015311 357.099135,122.212045 356.737865,122.360386 C350.768639,124.80965 344.724744,127.081128 338.410158,129.51137 C339.629466,132.273086 340.888291,135.015663 342.200119,137.732657 C344.12818,141.726312 345.573762,145.777899 346.629245,149.883852 C346.908273,149.806333 347.195057,149.738314 347.483156,149.674993 C354.887443,148.048732 362.29173,146.420472 369.709008,144.849151 C369.976964,144.792541 370.247373,144.741027 370.516486,144.700224 C367.165171,136.500347 362.871418,128.849623 357.724773,121.717806 Z M247.045692,93.6128103 C242.912324,95.0952259 239.657396,96.1490996 236.487413,97.4167448 C225.82165,101.684226 215.889748,107.081661 206.521963,113.329188 L206.306926,113.095588 C215.9297,123.530453 225.518078,133.998717 235.076933,144.492587 L235.011353,144.59732 C244.716175,129.283687 246.95575,112.207937 247.045692,93.6128103 Z M323.482035,94.0103854 C324.311507,96.1471017 325.858522,99.6143961 326.751953,101.724141 C329.745501,108.794931 332.597133,115.934749 335.59161,123.005842 L335.531365,123.026674 C341.512557,120.932912 347.481757,118.806185 353.479939,116.763368 C353.603459,116.721289 353.727208,116.681914 353.85089,116.645715 C351.641316,113.905865 349.299308,111.247218 346.831171,108.66772 C340.733053,102.294532 334.981715,97.886243 326.640025,94.9843445 C325.637663,94.562795 324.263537,94.3670042 323.482035,94.0103854 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Face" transform="translate(195.000000, 236.000000) scale(1 1)" fill="#000">
                    <g id="Face/Happy" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M14.5698,29.4833 C18.2688,29.5713 21.9628,29.5283 25.6528,29.6263 C28.4078,29.7003 31.1608,29.8523 33.9108,30.1953 C34.3098,30.2453 34.7088,30.2993 35.1058,30.3613 C38.5848,30.8983 37.7838,31.8753 37.5038,34.6443 C36.9518,40.1123 28.8908,47.7373 23.3768,47.9563 C15.5378,48.2683 10.4838,40.1833 12.3748,31.9173 L12.4569551,31.5476857 C12.8014445,29.9640669 12.8539034,29.4432655 14.5698,29.4833 Z M16.1028,39.5813 C18.1088,43.8933 24.5298,45.4403 28.0218,42.3103 C27.8608,37.9353 18.7918,36.0493 16.1028,39.5813 Z M39.2596,16.7863 C39.1986,20.9173 35.4296,26.0733 31.5066,26.9713 C31.1106,27.0613 30.6746,26.9733 30.2216,26.8133 C29.0186,26.3893 28.4826,25.0013 29.1006,23.8853 C29.2056,23.6963 29.3256,23.5213 29.4686,23.3683 C30.9246,21.8083 32.9396,21.1873 34.1216,19.4703 C34.5366,18.8663 34.9366,17.5253 34.8036,16.5443 C34.6996,15.7723 33.4096,15.0573 32.6366,15.1593 C31.7936,15.2693 30.9686,15.4053 30.3646,15.8453 C28.7806,16.9963 27.6356,18.7373 26.1236,20.0143 C25.5806,20.4723 24.4366,20.8293 23.7746,20.3423 C23.0616,19.8193 22.6306,18.0323 22.9756,17.6723 C25.5616,14.9763 27.8726,12.5913 31.0576,10.8593 C34.8016,8.8223 39.3296,12.0563 39.2596,16.7863 Z M10.1324943,10.8040858 C12.9292938,11.0171438 14.9189287,13.8713174 14.8893669,17.6232828 C14.8610905,21.1608501 13.4395554,22.8880941 10.6376148,22.787595 C7.28299777,22.6656562 4.90648939,20.2724383 4.889692,16.9948286 C4.87307175,13.3581022 7.22258896,10.5829878 10.1324943,10.8040858 Z M38.0260693,0.125467154 C40.7291822,0.109776068 43.417257,2.94955508 43.3721424,5.89841651 C43.3332938,8.45215718 42.0324912,10.0180959 39.2742383,10.1223302 C36.4483136,10.2289886 33.5747679,7.59889025 33.3805248,4.81122839 C33.2201175,2.49262094 35.5522886,0.14007674 38.0260693,0.125467154 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
            </g>
        </g>
        <g id="Table" transform="translate(821.000000, 617.000000)">
            <path d="M236.737009,80.6903695 C235.992395,82.8453128 235.473664,84.5568563 234.819003,86.214351 C184.36017,214.032618 133.890342,341.84588 83.4205145,469.660144 C82.6858955,471.519821 82.0522241,473.430544 81.1666833,475.216154 C79.3916038,478.79338 75.9943655,480.580992 72.2742947,480.080541 C67.8915674,479.491009 66.4573112,477.259997 67.8765752,472.951111 C72.9939215,457.415101 78.1572439,441.894104 83.3105715,426.371105 C120.76215,313.563375 158.214728,200.756646 195.670305,87.9499161 C196.291982,86.0792291 196.960636,84.2255575 197.736233,81.9935447 C210.661531,81.5611547 223.456895,81.1337693 236.737009,80.6903695 Z M471.55224,72.5590366 C473.568194,79.5973838 475.362264,86.2493826 477.377219,92.83332 C486.312585,122.022643 495.542799,151.123887 504.23729,180.385275 C522.920602,243.267982 541.141153,306.288815 559.956396,369.130486 C568.294072,396.980601 577.61224,424.536451 586.487637,452.22442 C586.543608,452.397576 586.626565,452.561724 586.681537,452.73488 C588.368662,458.041666 588.185757,459.125644 585.454173,460.033462 C581.134413,461.465754 578.197936,460.04247 576.577776,455.566434 C565.570484,425.148002 554.596175,394.71756 543.598878,364.295125 C509.373627,269.609738 475.142378,174.925351 440.915127,80.2399633 C440.241477,78.3772835 439.594812,76.5055956 438.636309,73.7891459 C450.043394,73.3627614 460.700867,72.9644022 471.55224,72.5590366 Z M503.028532,49.9529175 L504.743028,49.9776727 C509.206713,50.076762 513.708379,51.145726 518.065119,52.2737433 C520.722741,52.9613634 522.987567,54.7239529 523.123496,57.9949026 C523.249431,61.0206312 521.556309,63.8311658 518.023141,64.959183 C514.605913,66.0501669 510.9668,66.8258664 507.392654,66.974 C490.153594,67.6906463 472.903539,68.104019 455.659481,68.7085642 C432.248622,69.5293044 408.842759,70.4591429 385.4319,71.2788821 C367.099406,71.9214616 348.762915,72.4329229 330.430422,73.0674951 C306.474845,73.8972434 282.523265,74.8230783 258.567688,75.6498239 C240.053289,76.2894006 221.534892,76.8128727 203.020493,77.436435 C182.329223,78.1330633 161.640952,78.9187718 140.949681,79.5833712 C136.818824,79.7154903 132.675972,79.5113062 128.542116,79.3761843 C127.661572,79.348159 126.768036,79.0118558 125.922474,78.701576 C123.40478,77.7777428 120.926065,76.1592833 121.018373,73.4888751 C121.084983,71.7042658 122.812087,69.3421356 124.458233,68.3952817 C127.193815,66.8248655 130.42314,65.4316091 133.511539,65.2734664 C153.824005,64.2325277 174.156461,63.59195 194.477923,62.7241675 C214.61648,61.8643921 234.74804,60.8464742 254.886597,59.9826952 C269.039258,59.3761482 283.198915,58.9347502 297.354574,58.371242 C305.156528,58.0609622 312.953485,57.6525939 320.75444,57.2892663 L320.75444,57.2892663 L320.760436,57.4213854 C351.420537,56.0621597 382.078638,54.6438807 412.740738,53.3627253 C435.967693,52.3918498 459.200645,51.598134 482.430599,50.7203424 C489.868741,50.4400897 497.31488,49.8115228 504.743028,49.9776727 Z" id="Fill" fill="#FFF" transform="translate(327.558009, 265.056720) rotate(2.000000) translate(-327.558009, -265.056720)"/>
            <path d="M321.381799,57.9123091 C321.3798,57.8682694 321.377801,57.8242297 321.375802,57.7801899 C313.574847,58.1435176 305.777891,58.5518859 297.975937,58.8621657 C283.820277,59.4256738 269.66062,59.8670719 255.507959,60.4736189 C235.369403,61.3373978 215.237842,62.3553158 195.099286,63.2150912 C174.777824,64.0828737 154.445368,64.7234514 134.132901,65.7643901 C131.044503,65.9225327 127.815177,67.3157892 125.079596,68.8862054 C123.43345,69.8330593 121.706345,72.1951894 121.639735,73.9797988 C121.547428,76.650207 124.026142,78.2686665 126.543837,79.1924996 C127.389398,79.5027794 128.282935,79.8390827 129.163478,79.867108 C133.297334,80.0022299 137.440186,80.206414 141.571044,80.0742948 C162.262314,79.4096955 182.950586,78.6239869 203.641856,77.9273587 C222.156255,77.3037964 240.674651,76.7803243 259.18905,76.1407476 C283.144628,75.314002 307.096207,74.3881671 331.051784,73.5584188 C349.384278,72.9238465 367.720769,72.4123853 386.053262,71.7698058 C409.464122,70.9500666 432.869984,70.0202281 456.280844,69.1994879 C473.524902,68.5949427 490.774956,68.1815699 508.014016,67.4649237 C511.588163,67.3167901 515.227276,66.5410906 518.644504,65.4501067 C522.177672,64.3220895 523.870794,61.5115549 523.744859,58.4858263 C523.608929,55.2148765 521.344104,53.452287 518.686482,52.7646669 C514.329741,51.6366497 509.828076,50.5676857 505.36439,50.4685963 C497.936242,50.3024465 490.490104,50.9310133 483.051961,51.2112661 C459.822007,52.0890577 436.589055,52.8827734 413.3621,53.853649 C382.700001,55.1348043 352.041899,56.5530833 321.381799,57.9123091 M439.257672,74.2800696 C440.216175,76.9965193 440.862839,78.8682072 441.53649,80.730887 C475.763741,175.416275 509.994989,270.100661 544.220241,364.786049 C555.217538,395.208484 566.191847,425.638926 577.199139,456.057358 C578.819299,460.533394 581.755776,461.956678 586.075536,460.524386 C588.807119,459.616567 588.990024,458.53259 587.302899,453.225804 C587.247928,453.052648 587.164971,452.8885 587.109,452.715344 C578.233602,425.027375 568.915435,397.471525 560.577758,369.621409 C541.762515,306.779739 523.541964,243.758906 504.858653,180.876198 C496.164161,151.61481 486.933948,122.513567 477.998582,93.3242437 C475.983627,86.7403063 474.189557,80.0883075 472.173602,73.0499603 C461.32223,73.4553258 450.664756,73.8536851 439.257672,74.2800696 M198.357596,82.4844683 C197.581998,84.7164812 196.913345,86.5701528 196.291667,88.4408398 C158.836091,201.247569 121.383513,314.054299 83.931934,426.862029 C78.7786064,442.385028 73.615284,457.906025 68.4979377,473.442035 C67.0786737,477.750921 68.5129299,479.981933 72.8956571,480.571464 C76.615728,481.071916 80.0129663,479.284304 81.7880458,475.707078 C82.6735866,473.921467 83.307258,472.010744 84.041877,470.151067 C134.511705,342.336804 184.981532,214.523542 235.440366,86.7052747 C236.095026,85.0477799 236.613757,83.3362365 237.358371,81.1812931 C224.078258,81.624693 211.282893,82.0520784 198.357596,82.4844683 M507.020532,46.0185833 C511.781063,46.0846429 516.639543,47.6170248 521.228164,49.1594157 C525.450974,50.5796965 528.220538,53.8286264 528.137581,58.6289554 C528.054624,63.4072645 525.878752,67.3978632 521.39008,68.9322469 C516.651537,70.5517073 511.584165,71.6046569 506.589755,71.9910053 C498.270069,72.6335848 489.895412,72.5334945 481.544743,72.785722 C480.292392,72.8237563 479.048037,73.0719801 477.245972,73.289176 C477.622776,75.1408458 477.826671,76.8684038 478.32941,78.5048795 C484.946978,100.045305 491.82741,121.506659 498.204104,143.117147 C507.719169,175.365229 516.724499,207.762444 526.234568,240.011527 C547.158717,310.964512 568.163824,381.893476 591.471737,452.111799 C591.528708,452.284956 591.60167,452.451105 591.668635,452.621259 C593.191845,456.519775 594.197324,460.355233 590.124436,463.437013 C587.219942,465.634995 579.650867,465.905238 576.615442,463.499068 C575.189181,462.369049 574.235676,460.37325 573.579016,458.581634 C567.197326,441.174937 560.923579,423.728203 554.619848,406.292479 L508.522,278.792 L531.294873,386.390811 C531.370512,386.748208 531.418018,387.110794 531.437044,387.475238 L531.446564,387.840178 C531.446564,391.706171 528.312557,394.840178 524.446564,394.840178 L524.446564,394.840178 L520.062186,394.840178 C512.467121,394.840178 505.903941,389.534888 504.311886,382.108557 L504.311886,382.108557 L443.757791,99.6463443 C441.531779,93.4886613 439.305782,87.3309752 437.079801,81.1732859 C436.339185,79.1264401 435.644545,77.061578 434.702034,74.3501328 C373.397664,76.469542 312.175448,78.5862391 251.221577,80.6942092 L174.274583,391.971644 C172.439183,399.396217 165.776971,404.612037 158.128901,404.612037 L153.744524,404.612037 C149.878531,404.612037 146.744524,401.47803 146.744524,397.612037 C146.744524,397.124953 146.795363,396.639199 146.896215,396.16267 L180.327605,238.201556 C149.287326,316.850093 118.242294,395.49676 87.2032376,474.14567 C85.927899,477.376583 84.6445645,480.453358 81.5841515,482.59629 C76.6517094,486.051406 69.9122043,485.801181 65.5724548,481.7195 C63.0387687,479.336351 62.4220885,476.599883 63.5225178,473.299907 C96.5174075,374.340665 129.501303,275.376419 162.464209,176.405166 C172.113206,147.433039 181.698235,118.439893 191.305253,89.4537533 C191.925931,87.5810645 192.484641,85.6883576 193.261239,83.1951092 C191.115352,83.0810063 189.395244,82.8658122 187.683131,82.9178592 C174.247099,83.331232 160.790077,83.4723592 147.387028,84.3851824 C141.027326,84.8175723 134.775568,85.3890877 128.469838,84.2310434 C125.177545,83.6264982 122.193093,82.4684539 119.861302,79.9401739 C116.366115,76.1517575 116.087259,72.1741705 119.323581,68.1725618 C122.964693,63.6715028 127.948108,61.4575062 133.583186,61.2232949 C162.977943,60.000192 192.381695,58.9662595 221.77945,57.7972053 C247.729993,56.7642738 273.676538,55.6402602 299.624082,54.5542808 C326.117343,53.4452807 352.610604,52.3482914 379.102866,51.2172715 C411.219212,49.845034 443.334557,48.4297577 475.449903,47.0615238 C485.973446,46.6121185 496.503985,45.8734524 507.020532,46.0185833 Z" id="Stroke" fill="#000" transform="translate(328.000000, 265.500000) rotate(2.000000) translate(-328.000000, -265.500000)"/>
        </g>
        <g id="Table Objects" transform="translate(926.000000, 421.000000) scale(1 1)">
            <g id="Table Objects/Laptop" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <polygon id="Fill" fill="#FFF" transform="translate(260.072896, 204.573006) rotate(1.000000) translate(-260.072896, -204.573006)" points="89.2299428 249.108687 89.2299428 257.956521 144.591165 256.044687 282.602375 246.639785 307.266721 186.618687 325.629079 205.346476 325.629079 209.218687 308.536165 210.700682 305.706017 213.686688 314.506783 245.426687 318.949754 247.862259 370.303384 245.426687 378.625086 231.318313 383.84583 241.226076 400.212132 245.426687 408.409219 241.226076 411.494104 238.848447 423.607558 236.663611 430.91585 226.258481 423.607558 216.76411 408.409219 213.686688 383.84583 213.686688 386.551628 192.571514 378.625086 178.009493 366.069376 170.222393 332.172165 202.560687 333.093846 186.618687 328.439122 170.222393 324.32796 170.222393 324.32796 180.369336 328.439122 200.437274 318.949754 194.548275 310.168908 178.009493 317.859607 151.189491 232.4141 151.189491 173.783293 154.22149 149.305898 231.318313 147.209881 234.224114 125.270641 238.848447 98.6720391 244.080432"/>
                <path d="M235.5511,213.2737 C236.9121,213.7927 238.4221,213.7867 239.7791,213.2557 C241.6551,212.5227 243.4941,211.8367 244.9981,210.7067 C248.2811,208.2387 248.5931,203.8037 246.2731,200.4297 C243.7721,196.7947 240.9961,195.9447 236.8781,198.2907 C234.4271,199.6867 232.1361,201.6647 230.3541,203.8547 C228.1011,206.6237 228.5941,209.2597 231.5301,211.3597 C232.6841,212.1857 234.0571,212.7047 235.5511,213.2737 M317.2738,148.3855 C319.2168,148.5135 320.6108,150.2935 320.3148,152.2185 C318.9608,161.0555 314.7258,169.1185 312.4378,177.8095 C315.9798,182.4715 321.8238,191.3665 325.6368,196.3855 C323.0248,186.1435 322.1718,181.0975 321.8888,175.7605 C321.7868,173.8465 322.2658,171.7435 323.0468,169.9775 C324.2908,167.1665 327.2718,166.6595 329.1698,169.0585 C330.6938,170.9855 331.8958,173.4075 332.4678,175.7955 C333.5678,180.3755 333.5218,181.3965 334.3118,186.0515 C334.6108,187.8135 334.5298,196.7475 334.1478,198.2705 C335.6488,196.9315 338.5578,193.4425 339.5348,192.4745 C346.5078,185.5665 353.4188,178.5935 360.4408,171.7355 C364.3458,167.9225 365.7958,167.6215 370.4998,170.4275 C382.0258,177.3035 389.4248,186.7355 388.4138,201.0195 C388.1848,204.2525 387.4368,207.7445 387.1668,211.5725 C389.0708,211.7845 391.6888,211.8995 393.7958,211.9685 C401.0588,212.2045 408.3238,212.4085 415.5898,212.4845 C423.7448,212.5705 429.6138,216.8905 430.7408,223.8265 C431.8928,230.9195 427.5148,237.5065 419.9158,240.0015 C418.5468,240.4505 413.5388,240.5495 412.1068,240.6995 L412.09026,240.715741 C411.941968,240.861536 410.804179,241.983914 408.9868,243.8795 C407.5858,245.3425 405.2978,246.5445 403.2948,246.7445 C394.7618,247.5965 386.4648,246.6165 381.3818,240.4335 C380.9468,239.8655 379.9698,238.4555 379.2218,236.1055 C377.8128,238.6345 376.1108,242.4435 374.7818,244.7555 C373.7388,246.5705 372.4308,247.6055 370.0348,247.6875 C352.4188,248.2915 334.8088,249.0895 317.1968,249.7975 C317.1878,249.7975 317.1788,249.7985 317.1698,249.7985 C315.6228,249.8505 314.2048,248.9455 313.3918,247.6275 C307.3778,237.8765 305.2908,227.0315 304.2848,215.9065 C303.6838,209.2715 304.5788,208.5195 311.3958,207.9765 C315.1908,207.6735 318.9758,207.2495 323.8348,206.7755 C320.8288,203.5685 318.2928,200.9395 315.8448,198.2315 C313.3138,195.4295 310.8688,192.5515 308.0028,189.2665 C304.8348,197.8245 301.9418,205.8265 298.9098,213.7755 C295.3488,223.1085 291.6248,232.3795 288.0558,241.7085 C286.3208,246.2465 283.1818,248.7055 278.3148,248.9885 C269.6088,249.4925 260.9048,250.0295 252.1988,250.5515 C227.3518,252.0395 202.5038,253.5035 177.6578,255.0225 C150.2758,256.6975 122.8968,258.4235 95.5158,260.1205 C94.2468,260.1985 92.9728,260.2225 91.7008,260.2395 C86.9648,260.3035 86.2488,259.6525 86.0388,255.1365 C85.7278,248.4475 87.1988,245.6055 93.7368,243.8155 C105.2818,240.6545 116.9788,238.0325 128.6518,235.3625 C134.4768,234.0305 140.3918,233.0915 146.6648,231.8985 C150.4018,220.1165 154.2498,208.3995 157.8238,196.6005 C161.0368,185.9995 164.0098,175.3245 166.9628,164.6475 C167.5708,162.4485 168.0508,160.2155 168.5228,157.9245 C169.1638,154.8145 171.7478,152.4665 174.9068,152.1535 C199.5238,149.7175 295.7918,146.9715 317.2738,148.3855 Z M141.1268,249.0805 C124.4208,250.4805 107.5348,250.6125 91.1508,251.9855 C91.0138,252.5145 90.5838,255.4875 90.4468,256.0165 C90.4468,256.0165 92.2508,255.8605 93.2098,255.8405 C96.1078,255.7805 98.9978,255.4075 101.8948,255.2285 C114.3958,254.4525 126.8978,253.7075 139.3968,252.9195 C139.8658,252.8895 140.3018,252.8465 140.3018,252.8465 L141.1268,249.0805 Z M316.2108,152.3625 C268.1178,152.9515 221.1068,153.0295 174.5658,156.0925 C164.5278,188.5965 154.6878,220.4585 144.7288,252.7025 C190.6388,249.9235 236.4408,247.1505 282.0758,244.3875 C283.2928,241.5745 284.3118,239.4475 285.1488,237.2505 C290.8358,222.3185 296.4338,207.3515 302.1738,192.4405 C303.0138,190.2605 304.8908,186.2565 305.1898,184.1275 C305.1898,184.1275 302.0888,179.9195 299.8398,176.4865 C297.6458,172.5195 302.3848,171.1585 307.2928,173.5905 C307.7328,173.8085 309.2528,175.0065 309.2528,175.0065 C311.5268,168.1205 313.6988,159.9695 316.2108,152.3625 Z M144.5988,236.9625 C127.7608,239.6945 110.0398,243.6285 94.2478,248.4335 C109.4258,247.1065 127.0168,245.7785 142.2568,244.4455 C143.0638,241.8695 143.6838,239.8855 144.5988,236.9625 Z M381.8468,209.1905 C368.2528,209.5975 354.2748,209.9885 342.4138,210.3435 C339.7928,213.5595 338.2388,216.7415 335.6948,218.2235 C333.1818,219.6885 329.6698,219.4335 326.3258,219.9775 C326.2758,221.0535 326.2638,222.4935 326.1308,223.9225 C325.9158,226.2495 325.8508,228.6335 325.2478,230.8635 C324.9958,231.7925 323.5368,232.9545 322.5988,232.9815 C321.6518,233.0075 320.1618,231.9505 319.8078,231.0255 C319.0478,229.0345 318.6028,226.8365 318.5028,224.7005 C318.3108,220.5765 318.4478,216.4385 318.4478,211.8165 C314.9838,212.1205 312.0098,212.3815 308.5438,212.6845 C308.9988,223.4695 310.8738,233.4375 315.6818,242.7815 L317.6238,245.3985 C331.2138,244.8215 349.4058,244.2405 362.9948,243.6185 C365.4868,243.5045 367.9678,243.1645 370.2548,242.9475 C375.0448,231.9545 379.7388,221.2075 381.8468,209.1905 Z M385.9118,215.5375 C384.2418,220.5705 382.5288,227.4945 381.2518,231.9925 C381.2518,231.9925 381.6638,232.8565 382.2358,234.0315 C385.6068,240.9605 396.6698,245.1375 403.7518,242.0455 C405.4448,241.3055 406.9928,239.7415 408.0758,238.1865 C410.8198,234.2465 412.3508,222.4595 411.0068,216.9785 C402.9928,216.6705 394.5038,215.8665 385.9118,215.5375 Z M416.3148,217.6915 L416.4378,219.4275 C420.0478,220.0425 423.3178,220.9145 422.9578,227.3405 C422.4648,230.6205 417.5188,232.9165 414.6668,233.4175 L413.5618,236.5095 C420.2108,237.4685 425.7038,232.8145 426.4528,227.1595 C427.1688,221.7525 423.1308,218.0495 416.3148,217.6915 Z M416.0288,222.3385 C415.8538,224.1975 415.3648,228.2345 415.1908,230.0935 C415.6068,230.3835 418.0738,229.6315 419.0358,228.1165 C419.9978,226.6015 420.2538,224.9695 418.9798,223.5685 C418.1608,222.6665 416.0448,222.1695 416.0288,222.3385 Z M249.6801,197.6567 C253.1871,202.5127 252.8681,208.7947 248.8781,213.1477 C245.8681,216.4307 242.0191,217.7147 236.4911,217.8417 C235.5031,217.5977 233.2771,217.3807 231.3431,216.5117 C224.2921,213.3417 222.4691,206.3277 227.4181,200.4447 C229.5671,197.8907 232.4421,195.6777 235.4171,194.1487 C241.3521,191.0997 245.9281,192.4617 249.6801,197.6567 Z M364.8618,173.1075 C354.1268,184.0065 343.1478,195.1525 332.1798,206.2875 C349.3248,205.7755 366.4758,205.2635 383.4808,204.7555 C385.6148,189.8195 378.4748,177.6825 364.8618,173.1075 Z M326.8408,174.3855 C323.4268,174.3855 328.0298,190.9925 331.2568,198.7955 C330.4118,190.5395 330.4288,175.3405 326.8408,174.3855 Z" id="Stroke" fill="#000" transform="translate(258.461766, 204.121465) rotate(1.000000) translate(-258.461766, -204.121465)"/>
            </g>
        </g>
        <g id="Hanging Lamp" transform="translate(781.000000, 17.000000) scale(1 1)" fill="#000">
            <g id="Hanging Lamp/Sofisticated" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M244.393421,181.178753 L253.885421,177.597753 C250.622421,177.354753 247.596421,177.129753 243.893421,176.854753 C244.096421,178.606753 244.213421,179.622753 244.393421,181.178753 M238.868421,177.617753 C234.330421,179.259753 230.646421,180.592753 226.649421,182.037753 L239.795421,182.037753 C239.450421,180.390753 239.220421,179.297753 238.868421,177.617753 M236.493421,148.302753 C232.769421,149.469753 229.961421,150.472753 227.017421,152.497753 C221.621421,156.209753 219.592421,161.927753 223.214421,167.334753 C226.734421,172.588753 231.509421,174.438753 238.699421,173.892753 C237.786421,166.890753 237.520421,156.189753 236.493421,148.302753 M243.893421,172.488753 C246.856421,172.643753 250.601421,171.320753 252.478421,168.700753 C258.273421,160.617753 257.357421,156.902753 252.874421,148.505753 C251.405421,145.752753 249.116421,144.663753 246.098421,145.302753 C244.392421,145.662753 242.737421,146.263753 240.559421,146.903753 C241.669421,155.472753 242.772421,163.838753 243.893421,172.488753 M157.402421,328.041753 C155.489421,329.088753 156.144421,331.975753 158.324421,332.042753 L158.330421,332.042753 C168.674421,332.331753 179.071421,333.250753 189.358421,332.571753 C213.095421,331.004753 236.811421,328.956753 260.477421,326.524753 C288.494421,323.644753 316.327421,319.345753 343.834421,313.227753 C352.676421,311.260753 361.352421,308.528753 370.062421,306.001753 L370.114421,305.986753 C372.309421,305.357753 372.102421,302.201753 369.842421,301.869753 C369.225421,301.778753 368.611421,301.705753 367.996421,301.659753 C364.373421,301.392753 360.735421,301.267753 357.101421,301.217753 C327.231421,300.801753 297.567421,303.741753 267.962421,307.132753 C234.390421,310.977753 201.032421,316.262753 168.188421,324.379753 C164.543421,325.280753 160.843421,326.157753 157.402421,328.041753 M224.870421,185.779753 C224.382421,186.430753 223.951421,186.811753 223.763421,187.287753 C219.387421,198.357753 216.173421,209.628753 217.442421,221.753753 C218.093421,227.968753 216.813421,233.669753 212.319421,238.353753 C208.803421,242.018753 205.505421,245.912753 201.804421,249.376753 C193.581421,257.071753 185.344421,264.767753 176.792421,272.089753 C160.512421,286.027753 155.411421,305.039753 151.716421,325.428753 C153.698421,324.787753 155.187421,324.291753 156.685421,323.823753 C162.223421,322.091753 167.695421,320.093753 173.314421,318.688753 C199.518421,312.135753 226.200421,308.187753 252.946421,304.777753 C269.979421,302.604753 287.055421,300.812753 304.161421,299.317753 C327.245421,297.300753 352.140421,294.888753 375.536421,297.711753 C375.952421,297.761753 375.083421,297.860753 375.536421,297.711753 C367.545421,278.076753 355.081421,263.255753 337.451421,252.633753 C332.012421,249.357753 326.317421,246.507753 320.822421,243.319753 C314.073421,239.403753 307.229421,235.609753 300.752421,231.275753 C290.187421,224.206753 281.275421,215.548753 276.225421,203.502753 C275.036421,200.668753 273.215421,198.103753 271.951421,195.294753 C269.578421,190.023753 265.548421,186.105753 261.619421,182.082753 C259.898421,180.321753 258.186421,180.116753 255.794421,181.233753 C251.872421,183.065753 247.756421,184.567753 243.600421,185.797753 C237.391421,187.636753 231.053421,186.937753 224.870421,185.779753 M222.407421,172.930753 C214.579421,166.273753 214.589421,151.823753 226.649421,147.796753 C229.460421,146.857753 233.991421,144.836753 235.118421,144.324753 C235.118421,144.324753 234.622421,139.575753 234.393421,136.708753 C233.305421,123.128753 219.931189,22.4091633 218.854189,8.82716328 C218.731189,7.27216328 218.837189,5.69716328 218.837189,4.12616328 C222.285189,3.49116328 222.846189,5.35416328 223.064189,7.45316328 C223.476189,11.4291633 236.089421,102.553753 236.414421,106.536753 C237.288421,117.218753 238.119421,127.903753 239.001421,138.583753 C239.087421,139.633753 239.410421,140.663753 239.689421,142.029753 C241.982421,141.628753 244.062421,141.225753 246.156421,140.906753 C250.865421,140.186753 254.282421,142.104753 256.576421,146.167753 C260.884421,153.801753 261.852421,161.542753 257.237421,169.435753 C256.871421,170.061753 256.816421,170.032753 256.470421,170.669753 C256.213421,171.142753 254.793421,172.896753 253.885421,173.185753 C258.494421,174.902753 260.628421,175.970753 263.069421,177.715753 C271.034421,183.409753 275.913421,191.497753 279.697421,200.248753 C284.382421,211.081753 291.806421,219.571753 301.295421,226.313753 C306.025421,229.674753 310.993421,232.725753 316.004421,235.658753 C321.801421,239.052753 327.718421,242.250753 333.664421,245.379753 C351.900421,254.974753 366.011421,268.632753 375.258421,287.167753 C376.960421,290.579753 378.960421,293.844753 380.622421,297.274753 C382.793421,301.756753 381.973421,305.428753 377.821421,308.183753 C375.462421,309.748753 372.681421,310.913753 369.927421,311.610753 C341.028421,318.925753 311.775421,324.519753 282.210421,328.224753 C259.514421,331.069753 236.733421,333.346753 213.939421,335.273753 C199.836421,336.465753 185.642421,336.654753 171.482421,337.030753 C166.600421,337.160753 161.690421,336.579753 156.807421,336.156753 C154.834421,335.985753 152.859421,335.498753 150.957421,334.919753 C148.484421,334.166753 146.885421,332.566753 147.006421,329.750753 C147.092421,327.761753 146.894421,325.700753 147.346421,323.795753 C149.144421,316.200753 151.012421,308.616753 153.104421,301.098753 C156.616421,288.480753 163.814421,278.157753 173.383421,269.374753 C182.219421,261.265753 191.086421,253.190753 199.842421,244.995753 C202.622421,242.393753 205.063421,239.429753 207.709421,236.680753 C211.022421,233.237753 212.585421,229.308753 212.365421,224.392753 C211.982421,215.839753 212.493421,207.289753 214.742421,198.974753 C215.828421,194.955753 216.933421,190.826753 218.847421,187.176753 C220.886421,183.287753 221.389421,179.827753 229.404421,176.854753 C228.348421,176.043753 223.907421,174.205753 222.407421,172.930753 Z" id="Stroke" fill="#000" transform="translate(264.394231, 170.524654) rotate(8.000000) translate(-264.394231, -170.524654)"/>
            </g>
        </g>
    </g>
</svg>

```

## File: static\src\img\job_not_found.svg

```svg
<svg viewBox="53.813 46.528 1309.438 1020.047" xmlns="http://www.w3.org/2000/svg" overflow="visible">
    <g id="Master/Spot Illustrations/Coding" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <ellipse id="Oval" fill="#C9C9C9" opacity=".5" cx="676" cy="1038.5" rx="520" ry="23.5"/>
        <g id="Background" opacity=".5" transform="translate(36.000000, 63.000000) scale(1 1)">
            <g id="Background/Blob 1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M869.490312,1 C892.755389,1.92225471 915.476128,2.67306463 938.178097,3.75966986 C969.204039,5.24710117 998.516884,13.6857318 1027.26162,24.1202161 C1037.65031,27.8920014 1048.00522,31.8411434 1058.0448,36.3637386 C1111.46613,60.4322218 1153.95818,95.8373436 1182.91939,145.017167 C1196.72302,168.457808 1203.97709,193.894302 1207.15551,220.364195 C1211.99823,260.684461 1207.17053,300.347325 1194.25536,338.780517 C1181.74563,376.012412 1161.11839,409.312902 1133.93534,438.957479 C1119.10311,455.13359 1102.71795,469.583429 1084.53084,482.282167 C1056.54443,501.823326 1028.65312,521.486269 1000.60163,540.943479 C992.90959,546.280732 984.675715,550.915653 976.903588,556.152405 C967.836315,562.262933 958.607617,568.242218 950.137238,575.042073 C912.903355,604.937314 894.836382,642.935391 897.166393,689.27869 C897.984775,705.586045 899.121,721.881576 899.722899,738.196025 C900.311032,754.162856 895.142962,768.633978 886.071935,782.031501 C881.843627,788.278003 876.617995,793.530126 870.611521,798.221801 C851.124267,813.448463 829.09302,824.464677 805.695301,833.153971 C770.656284,846.163675 734.397203,854.918 697.265931,860.621791 C670.549635,864.725824 643.740739,867.970269 616.719113,869.629145 C583.069094,871.693577 549.420325,871.303392 515.807846,869.280344 C473.354588,866.722861 431.187889,861.708396 389.257694,854.931006 C351.62338,848.84649 314.022853,842.549146 277.041743,833.480308 C234.792454,823.121496 192.917319,811.586217 152.869153,794.88986 C129.05974,784.962616 106.556736,772.934286 86.3599665,757.26778 C63.4264977,739.478905 47.2553176,717.39327 38.2080661,690.617141 C33.6781836,677.210159 33.1914089,663.507582 34.9307837,649.805006 C39.2904828,615.461659 50.0132903,583.102343 69.9622923,553.767549 C79.7365775,539.393381 91.5968611,526.686367 104.247997,514.582365 C116.075746,503.265827 128.207572,492.221235 139.714975,480.622109 C146.848914,473.432069 153.392217,465.6603 159.632693,457.750192 C174.762751,438.569659 178.94601,417.075212 174.700184,393.621565 C172.520334,381.58023 170.856041,369.344984 170.448101,357.153486 C169.504584,329.005799 178.127628,302.888255 191.235506,277.901065 C202.254884,256.896122 217.020799,238.458123 234.095453,221.59032 C286.091497,170.219551 345.922235,129.172122 414.627539,99.9957664 C469.984706,76.486548 525.927504,54.2980456 583.469525,35.9262588 C614.35907,26.0640453 646.160847,19.6460984 678.441891,15.7442516 C713.601037,11.4947856 748.8215,7.45105345 784.134562,4.65591226 C812.657806,2.39875298 841.359992,2.13035321 869.490312,1" id="Fill-1" fill="#3AADAA" opacity=".15"/>
            </g>
        </g>
        <g id="Decoration" transform="translate(136.000000, 235.000000) scale(1 1)">
            <g id="Decoration/Nature 2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M159.101179,445.223791 C145.030323,445.223791 130.92852,444.647067 116.900348,445.41426 C106.498202,445.983535 96.2177049,448.337252 86.8976034,453.596937 C73.0732485,461.399742 65.7721111,476.37436 68.7664098,491.801207 C71.5174084,505.969262 80.0446503,512.079136 92.988643,512.015787 C99.6484499,511.982306 105.880347,509.761598 111.553082,506.098014 C119.064439,501.24906 124.272982,494.398595 129.074958,487.048018 C138.348107,472.85549 146.324722,457.689339 159.101179,445.223791 M88.8708675,407.593112 C85.1482943,395.078069 81.7921203,383.048245 77.9545148,371.173099 C75.7156453,364.24231 73.3116831,357.240539 69.9118394,350.83099 C63.2942865,338.353028 53.4952382,328.956946 39.7563335,324.371734 C26.8386316,320.059856 13.9060181,323.740314 5.68866407,333.758281 C-3.18480017,344.576117 -4.6077924,359.436737 1.94904902,370.969688 C4.19856967,374.926658 7.02325184,378.323189 10.9087875,380.605201 C16.0586134,383.628813 21.2020486,386.860073 26.739576,389.0033 C46.9075085,396.809177 67.9573553,401.760951 88.8708675,407.593112 M174.424744,430.905581 C180.34148,424.222211 185.596406,418.28191 190.854539,412.345867 C201.314139,400.538721 207.578353,386.778089 209.344606,371.185332 C211.594121,351.31939 204.096096,335.58398 187.636365,324.298473 C178.390262,317.957894 167.869719,318.620057 158.930465,325.387529 C154.504139,328.73986 151.303072,333.048175 148.985131,338.047395 C144.231642,348.294947 143.794355,358.987488 146.054561,369.822682 C147.115169,374.910262 148.569228,379.971228 150.408184,384.833119 C156.513093,400.970937 166.045731,415.329857 174.424744,430.905581 M351.825084,590.052225 C351.740315,589.61741 351.655546,589.18153 351.571837,588.74565 C340.501034,587.542451 329.445066,586.169802 318.35625,585.17655 C301.587925,583.674949 284.869402,584.28241 268.422139,588.066786 C254.582576,591.251161 241.625668,596.587227 231.02851,606.482449 C224.385817,612.684946 219.583665,620.121549 218.201934,629.304868 C214.744428,652.282883 235.610263,671.856985 258.957705,667.907422 C272.621373,665.594807 283.857475,658.704919 293.594229,649.400107 C302.367798,641.016079 310.392223,631.835956 318.703802,622.965958 C325.697226,615.501647 332.480848,607.833783 339.64381,600.539987 C343.386352,596.728968 347.744526,593.528607 351.825084,590.052225 M284.653935,509.520101 C270.328536,509.332944 257.681937,503.884235 244.504688,500.672954 C233.135459,497.901546 221.720685,495.094187 210.166099,493.35796 C197.424173,491.444091 184.796639,492.716126 173.106478,498.898661 C155.551642,508.184623 147.10151,532.04559 159.630541,550.962222 C170.116415,566.794459 192.263852,576.579507 210.946714,572.680925 C230.83704,568.529629 247.533941,558.924336 259.925278,542.52957 C268.025881,531.812965 276.037512,521.028688 284.653935,509.520101 M303.657797,491.10942 C314.990642,482.895985 325.949174,475.056707 336.790399,467.060558 C343.369144,462.207116 348.928391,456.28102 353.590779,449.604491 C367.639796,429.484807 373.973264,407.286591 369.849418,382.947306 C366.713077,364.434484 353.492669,350.917134 332.947022,353.404801 C326.848166,354.143576 321.411557,356.575067 316.378055,359.920221 C302.928366,368.858652 295.260807,381.57681 292.017824,397.131355 C288.649003,413.291122 290.001223,429.303558 293.728361,445.202581 C296.316562,456.249222 299.212959,467.225908 301.756371,478.283149 C302.690555,482.343759 303.000883,486.546401 303.657797,491.10942 M408.49198,675.614354 C406.169796,675.878515 404.139871,676.126699 402.105709,676.335471 C392.545893,677.311163 382.997725,678.413609 373.424142,679.208222 C362.120305,680.145568 350.763523,680.508789 339.486159,681.681536 C326.809978,682.998081 314.491707,685.951785 303.112688,692.07116 C295.180412,696.336083 288.256216,701.797186 283.350298,709.468508 C276.997913,719.403306 275.589566,730.068807 280.220168,741.02723 C288.704134,761.102403 309.761568,770.436448 327.189067,769.557686 C338.914349,768.967584 349.018444,764.066758 357.663364,755.982152 C365.502456,748.650619 371.739421,740.051538 377.188558,730.932657 C387.686567,713.361687 397.781131,695.54573 408.018649,677.818181 C408.280199,677.365486 408.259021,676.746625 408.49198,675.614354 M426.057241,667.518728 C427.552286,667.668099 428.305125,667.83336 429.049457,667.805817 C448.747685,667.071676 468.45761,666.530339 488.139888,665.4911 C500.794604,664.82158 513.029304,661.62653 524.028411,655.341306 C537.795373,647.472329 548.80086,636.514237 557.541451,623.20647 C562.389182,615.825861 563.928887,607.951587 562.601848,599.521145 C560.852667,588.407327 555.642339,579.289356 544.529455,575.330502 C531.231423,570.594073 517.838755,570.44894 504.462036,576.033922 C486.880903,583.375334 472.710939,594.939384 460.982385,609.640218 C449.772737,623.688482 440.142138,638.800351 431.676951,654.629411 C429.531147,658.641234 428.011645,662.985697 426.057241,667.518728 M375.13056,577.800128 C389.862581,573.455185 403.884753,569.46717 417.812631,565.168042 C430.164008,561.354762 442.017432,556.353496 452.643981,548.780211 C472.161658,534.870638 486.004778,516.974117 491.428238,493.113153 C493.522823,483.900126 494.058918,474.774466 490.870953,465.675443 C486.962545,454.517952 478.252588,448.025043 466.524183,447.808754 C452.776416,447.554109 441.022583,452.957054 430.590977,461.409344 C415.585611,473.569432 405.498335,489.473544 397.344604,506.776605 C386.740304,529.282304 383.414607,554.0553 375.13056,577.800128 M419.277517,666.080881 C420.073571,664.582566 420.574161,663.791465 420.933395,662.940898 C430.229909,640.868632 443.355775,621.127196 458.430859,602.698245 C471.374982,586.875151 487.270565,574.793065 506.957239,568.388859 C518.286934,564.702007 529.826005,562.820353 541.638222,566.466854 C557.295732,571.300538 566.499781,581.745203 569.934825,597.752002 C573.790748,615.72222 566.820753,629.960987 554.791721,642.251202 C549.161947,648.002351 542.740369,653.104691 536.167871,657.785464 C521.89947,667.945545 505.344936,671.852206 488.243049,672.984172 C469.165373,674.246748 450.024991,674.560003 430.912242,675.301197 C429.023605,675.374466 427.138157,675.552863 424.751056,675.716392 C425.318603,677.302843 425.635325,678.550554 426.187993,679.683581 C433.805246,695.284742 441.466076,710.863603 449.087581,726.461578 C450.896506,730.164358 450.749836,732.069373 448.74854,733.093027 C446.544245,734.219683 444.580148,733.164173 442.700013,729.32441 C436.933135,717.542837 431.336308,705.677374 425.566241,693.896863 C423.052664,688.76479 420.277633,683.763329 417.197572,677.896435 C415.537442,680.362123 414.282248,682.011225 413.251309,683.789876 C404.937906,698.153945 396.907215,712.688977 388.293033,726.870402 C382.891766,735.760473 377.041987,744.446662 370.658671,752.653943 C362.077437,763.687952 351.332295,771.945142 337.364673,775.451474 C313.824202,781.35872 287.120771,769.921196 275.379763,748.705868 C265.614544,731.061648 269.894533,710.067192 286.117465,695.928242 C297.181455,686.285297 310.245677,680.884569 324.421612,677.82635 C337.493274,675.005994 350.800883,674.352937 364.08511,673.348397 C378.318438,672.272711 392.52732,670.852976 406.740455,669.512881 C408.227344,669.372713 409.66747,668.75576 411.820749,668.172787 C396.772236,641.864677 381.22207,616.422 364.524056,591.09082 C356.430657,596.343947 349.073795,601.892277 342.655406,608.507798 C335.131681,616.264841 328.171252,624.563444 320.693229,632.368272 C312.153444,641.280642 303.646608,650.271591 294.513768,658.555328 C284.926041,667.24895 273.535765,672.808961 260.583139,675.012365 C236.008542,679.194054 212.939965,661.696373 211.153359,637.171161 C210.275467,625.11456 214.946575,614.890767 222.648855,605.95185 C231.484104,595.698323 243.090133,589.699756 255.578305,585.171894 C267.672169,580.786324 280.275127,578.998116 293.029004,578.381162 C309.665375,577.576256 326.219909,578.662561 342.679851,581.294965 C347.603911,582.083943 352.604494,582.396136 359.011192,583.079987 C340.493624,555.275686 320.652841,530.23971 297.0858,507.01955 C291.416702,513.678608 285.890021,519.792921 280.799098,526.250222 C274.944006,533.674896 269.689408,541.574231 263.794991,548.967049 C249.913457,566.376594 231.364005,576.200058 209.774878,579.99416 C187.634146,583.884893 161.801166,570.435104 151.25796,550.490849 C139.793287,528.801922 149.418213,501.734567 172.429397,491.052041 C185.122694,485.158599 198.510015,484.619163 212.138596,486.075003 C228.74202,487.847283 244.533446,493.001655 260.466228,497.591106 C267.679609,499.669208 274.83666,502.195423 282.509181,501.969243 C284.721979,501.903406 286.927337,501.603956 289.13482,501.411755 C289.275113,500.998683 289.414343,500.585611 289.554636,500.171478 C287.918951,498.679534 286.32259,497.14193 284.644393,495.699895 C262.335734,476.542494 236.675995,463.467549 208.888483,454.368288 C199.180656,451.189015 189.26983,448.63519 179.522679,445.568477 C176.839051,444.723219 174.769734,445.202128 172.413455,446.440282 C162.908628,451.439619 155.770707,458.869603 149.978321,467.73525 C144.065836,476.784603 138.210743,485.872183 132.277001,494.90667 C127.107429,502.779458 120.645465,509.307905 112.213026,513.715774 C105.045346,517.462092 97.437658,519.509399 89.3400083,518.668389 C74.6698602,517.144589 64.7165213,507.545182 62.1838137,492.840249 C58.8975645,473.757179 67.6945518,455.918634 84.9090986,446.817249 C95.8901875,441.013006 107.844822,438.801107 120.034341,438.096018 C129.075778,437.573572 138.171419,437.994077 147.479624,437.196604 C146.012928,436.64124 144.578117,435.973317 143.075285,435.546441 C122.459704,429.683794 102.340461,422.433268 82.3073075,414.820641 C71.7311542,410.802482 60.4854223,408.518375 49.5043334,405.593954 C38.3074914,402.610067 27.166979,399.519992 16.85972,394.03219 C9.06710059,389.883419 2.24696436,384.635602 -2.59632079,377.122791 C-11.6271292,363.115514 -10.5069136,348.764188 -2.97681148,334.830181 C4.26101425,321.438795 16.1678221,315.083435 31.3991409,315.416866 C38.6677885,315.576148 45.4284066,317.944144 51.6916235,321.587459 C64.4773861,329.024875 73.9758367,339.552367 79.3898579,353.254884 C83.0619119,362.54847 85.5361642,372.311407 88.644922,381.832235 C90.9183007,388.790743 93.0694548,395.809779 95.7892193,402.596263 C97.3621976,406.5231 99.9512348,412.03214 99.9512348,412.03214 C99.9512348,412.03214 109.511329,417.998852 114.379059,419.401597 C130.866635,424.153516 147.272373,429.185772 163.725939,434.058745 C164.769631,434.367752 166.944167,435.746074 168.77116,435.931904 C165.195823,429.471418 160.971101,422.125322 157.65403,416.279666 C149.733872,402.314864 142.310052,388.148305 139.133273,372.198847 C136.411383,358.531372 137.151108,345.229185 144.1955,332.893309 C151.097474,320.806976 161.164535,313.586183 175.689076,314.018368 C182.311527,314.215878 188.130484,316.786693 193.281988,320.688045 C211.171427,334.233403 218.665393,352.276891 215.662918,374.458531 C213.429927,390.951672 206.669308,405.478209 195.508602,417.917087 C191.383786,422.515033 187.297231,427.150145 183.346717,431.896755 C181.941665,433.585146 180.991501,435.652629 179.421711,438.222382 C182.382736,439.086753 184.534953,439.741934 186.70205,440.344021 C211.296841,447.185722 235.372974,455.331414 257.611485,468.178055 C269.016641,474.765966 279.758594,482.272406 289.741692,490.860902 C291.32211,492.22011 292.931225,493.544276 294.943149,495.234791 C296.597965,489.814949 295.506445,485.038607 294.546716,480.474641 C292.523101,470.856119 289.876671,461.370333 287.595852,451.803844 C283.770752,435.768374 281.283746,419.637334 283.499732,403.057119 C286.421433,381.201477 295.699879,363.375675 315.301527,351.972131 C331.371413,342.623328 350.047341,344.117395 362.63967,355.802337 C370.282432,362.895705 374.642132,371.897273 376.215111,382.07222 C380.008326,406.612298 374.354107,429.19639 361.214424,450.027317 C353.131654,462.839977 341.822153,472.385228 329.527415,480.985405 C321.310729,486.733369 313.472408,493.019707 305.45447,499.051193 C304.381019,499.859284 303.248049,500.588797 301.662317,501.686782 C326.098747,525.551504 346.191419,551.636619 365.46678,579.758423 C367.007873,576.716133 368.612736,574.403355 369.400288,571.839973 C372.22421,562.629215 374.935471,553.378104 377.386342,544.061157 C381.268835,529.296759 385.706121,514.745799 392.435918,500.983817 C400.136071,485.236116 409.719548,470.803025 422.976141,459.158434 C435.355905,448.282646 449.465946,441.221135 466.436043,441.426078 C481.794901,441.609783 493.347788,450.207837 498.067786,464.818262 C501.842934,476.501081 500.623876,488.098949 497.312119,499.587443 C488.811659,529.079073 469.307791,549.217653 442.603296,563.011492 C430.020533,569.512329 416.371758,573.134407 402.791004,576.903024 C395.917726,578.810163 389.095464,580.941358 382.348663,583.259445 C378.751007,584.494413 375.364852,586.342087 371.571636,588.052778 C387.526737,614.149574 403.17362,639.739852 419.277517,666.080881" id="Fill-1" fill="#3AADAA" transform="translate(281.000000, 545.500000) rotate(17.000000) translate(-281.000000, -545.500000)"/>
                <path d="M849.456076,717.789654 C840.859098,688.560318 830.944316,660.073104 818.545212,632.508223 C806.203621,605.072243 790.485004,579.502206 774.938925,553.849571 C747.02125,507.781704 719.251108,461.633744 694.400389,413.802555 C684.964467,395.637467 676.014907,377.262131 671.022508,357.249872 C670.671177,355.84322 670.068538,354.497889 669.259602,352.206447 C667.764258,354.316426 666.696512,355.446504 666.057615,356.781822 C661.497816,366.298001 660.25503,376.513752 661.6466,386.740766 C663.263222,398.633486 665.309943,410.565002 668.445665,422.133592 C676.703815,452.61565 691.025862,480.520931 706.275622,507.980688 C729.867302,550.455584 758.687685,589.348773 788.234486,627.725104 C810.015749,656.014586 831.792011,684.276536 848.185784,716.211551 C848.488354,716.799742 849.027228,717.26654 849.456076,717.789654 M856.12011,707.914304 C856.37892,707.856736 856.63898,707.797917 856.899039,707.740349 C856.899039,706.684109 857.07408,705.592827 856.872783,704.577884 C853.108167,685.654405 850.061216,666.545707 845.335128,647.863761 C836.746901,613.928897 824.456572,581.147889 812.218756,548.394413 C794.963534,502.212662 776.680576,456.470178 753.685284,412.760081 C743.159111,392.751577 730.86128,374.013314 715.453984,357.405054 C709.013752,350.46315 701.843352,344.643813 692.241142,342.792889 C686.58484,341.702859 681.124833,341.955656 676.647553,345.716323 C680.838517,365.19921 688.058929,383.045174 696.79719,400.317965 C715.060144,436.419124 735.201029,471.47155 755.918298,506.197343 C766.508235,523.950698 776.820608,541.885517 787.936917,559.304729 C816.347206,603.823276 837.65586,651.578125 853.944609,701.695749 C854.623516,703.783201 855.392443,705.841869 856.12011,707.914304 M969.272403,154.755749 C968.455965,156.218717 967.967103,156.959588 967.60702,157.758026 C946.627191,204.27517 933.529177,253.123803 925.408558,303.335288 C920.618705,332.945071 916.530264,362.682504 912.781901,392.443715 C909.261091,420.410318 906.2579,448.452009 903.661052,476.518729 C901.273003,502.326546 899.900187,528.22572 897.772197,554.059818 C894.915289,588.731798 891.754562,623.377497 888.743869,658.034459 C888.198743,664.314336 886.415833,677.145659 885.920719,683.428039 C887.297286,681.299289 889.372764,672.607829 889.710341,670.397732 C894.815266,636.925912 900.286526,603.5029 904.802565,569.949734 C908.56093,542.021927 910.981487,513.915159 914.354764,485.931036 C919.144616,446.198111 929.344464,407.840552 945.494431,371.221285 C955.301688,348.983912 965.961641,327.119477 976.396543,305.162434 C988.235518,280.251921 995.479685,254.131237 997.358868,226.59514 C998.434115,210.860406 998.23657,195.207018 992.430233,180.174358 C988.080482,168.91363 980.928836,160.224673 969.272403,154.755749 M877.048677,679.830063 C877.458771,679.837571 877.871366,679.843829 878.281461,679.851338 C878.534019,677.975384 878.864095,676.106939 879.026632,674.223476 C880.990834,651.573119 882.942534,628.922761 884.874228,606.271152 C886.999717,581.350627 889.067693,556.425097 891.220689,531.507075 C894.468937,493.909158 897.21957,456.257428 901.195485,418.737102 C905.601499,377.153253 911.006493,335.673276 918.470711,294.494903 C926.640091,249.426961 937.848921,205.25382 956.39569,163.191909 C957.841022,159.91556 959.006291,156.515316 960.285335,153.213938 C959.371375,152.777174 958.988787,152.465558 958.578692,152.42426 C957.450933,152.311628 956.31067,152.246551 955.17916,152.279089 C941.44975,152.670799 929.729552,158.304918 918.965825,166.328093 C902.036928,178.947918 890.951877,195.911595 883.152582,215.213019 C869.121853,249.930052 863.000444,286.39664 859.959744,323.456424 C857.054075,358.871777 857.216613,394.360967 858.608183,429.841397 C860.408597,475.754081 864.402016,521.510332 868.411689,567.270337 C871.388624,601.241493 873.689154,635.270217 876.287252,669.273912 C876.556064,672.791795 876.796119,676.310929 877.048677,679.830063 M869.303144,732.488171 C869.303144,718.640654 870.055818,704.745581 869.161862,690.955632 C866.5,649.893646 863.260504,608.867952 860.11603,567.838504 C857.476672,533.383029 854.167161,498.971355 852.03542,464.485844 C849.481082,423.161049 848.187034,381.762417 850.498816,340.371294 C852.853108,298.236798 858.473152,256.654201 873.576628,216.886234 C881.522207,195.964157 892.621011,177.164573 910.201308,162.747637 C919.803518,154.872135 930.302184,148.727417 942.444979,145.727643 C966.963122,139.670528 985.122301,150.04897 996.214855,168.907373 C1002.45754,179.521091 1005.05939,191.288665 1005.70704,203.421667 C1007.71875,241.047117 999.28056,276.483745 983.146847,310.33476 C972.104305,333.505729 960.850465,356.604113 950.741888,380.185565 C935.97724,414.628526 926.856391,450.724679 922.456628,487.949657 C919.522203,512.777573 917.211671,537.680577 914.227234,562.502236 C907.034329,622.331268 898.598637,681.950053 881.098359,739.848067 C877.49378,751.774577 873.271558,763.512115 869.426923,775.367292 C868.606734,777.897764 867.157651,779.626045 864.538298,778.785057 C861.921445,777.94532 858.104317,777.843951 857.979288,773.910581 C857.090333,745.574794 842.34569,722.56151 828.972612,699.179042 C815.738316,676.043114 799.009466,655.273716 782.785731,634.193953 C751.158452,593.101932 720.372616,551.436737 695.548153,505.705515 C680.444676,477.884083 666.737772,449.486975 658.749683,418.674529 C654.509957,402.320317 651.39299,385.829695 653.891065,368.79093 C654.892545,361.96416 656.829241,355.506574 660.588856,349.658454 C669.559671,335.707065 684.053007,330.751244 699.66535,336.307771 C708.937484,339.607898 716.091631,345.822698 722.583125,352.953575 C736.893919,368.673291 748.7604,386.091253 758.811463,404.803234 C777.444502,439.48898 792.923065,475.597647 807.24136,512.223172 C823.785168,554.544137 839.786351,597.071595 852.152947,640.844265 C860.452357,670.221275 865.561033,700.216511 868.146628,730.638498 C868.201641,731.294268 868.407938,731.937524 868.54422,732.587037 C868.796778,732.554499 869.049336,732.520709 869.303144,732.488171" id="Fill-1" fill="#A1483A"/>
            </g>
        </g>
        <g transform="translate(319.000000, 101.000000) scale(1 1)" id="Master/Character/Standing">
            <g id="Master/Character/Standing" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <g id="Bottom/Standing" transform="translate(39.000000, 389.000000) scale(1 1)">
                    <g id="Bottom/Standing/Short pants" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M230.430095,-7.08101859 L203.423254,78.4568591 L179.429458,155.104726 L150.739388,250.232606 L135.45799,304.147624 L131,322.274045 L132.923504,327.457889 L176.723679,345.61793 L175.985048,403.69567 L175.985048,447.088194 L175.985048,473.938924 L137.091392,538.393328 L134.375499,546.648338 L142.277158,570 L157.670257,569.221339 L193.03525,545.621738 L214.068493,515.939199 L214.068493,499.117278 L215.585061,464.47339 L225.787232,351.146535 L237.900125,348.895615 L256.815194,239.089639 L280.35476,106.656135 L287.174404,70.574341 L290.606063,72.5777037 L290.606063,102.196281 L287.174404,350.131221 L311.921,354.026164 L314.649513,417.375828 L318.984671,475.686005 L318.021667,502.345342 L320.209039,543.242513 L332.311148,545.125642 L390.793656,547.531148 L454.404368,539.961556 L461,514.257664 L455.939313,505.740807 L432.920556,495.095051 L391.654392,485.259623 L357.254242,476.745486 L358.779033,424.325981 L359.617567,341.423628 L384.711915,323.116907 L378.684948,250.232606 L369.163483,136.361782 L357.254242,-7.08101859 C326.408267,-7.69367286 303.048321,-8 287.174404,-8 C271.300487,-8 252.385718,-7.69367286 230.430095,-7.08101859 Z" id="Fill" fill="#FFF"/>
                        <path d="M379.529583,485.337135 C377.238295,488.793735 375.130978,491.974016 372.599272,495.795567 C377.443173,496.399299 381.26791,496.87582 385.427142,497.394049 C387.204145,494.301356 388.895434,491.357772 390.827141,487.997101 C386.525748,486.984625 383.107631,486.179649 379.529583,485.337135 L379.529583,485.337135 Z M410.373128,492.945827 C404.867555,491.562144 400.3801,490.432884 395.667906,489.248359 C393.902401,492.408829 392.406583,495.087564 390.666165,498.203196 C396.14247,499.111401 400.743863,499.874668 405.391248,500.64732 C407.028182,498.116651 408.441422,495.93112 410.373128,492.945827 L410.373128,492.945827 Z M435.185316,499.036325 C427.963367,497.300205 421.555704,495.760115 415.006925,494.185615 C413.466159,496.699601 412.125044,498.889303 410.442118,501.635814 C417.449781,502.873517 423.717375,503.98088 430.003785,505.091372 C431.659533,503.157135 433.111449,501.459595 435.185316,499.036325 L435.185316,499.036325 Z M374.738992,484.263139 C369.696485,482.887797 365.558159,481.581275 361.335163,480.661601 C357.440391,479.812831 354.464434,481.952482 353.829939,485.624924 C353.022971,490.294202 355.205548,493.823793 359.305199,494.351407 C361.909031,494.685076 364.528542,494.89049 367.338297,495.174109 C369.723663,491.658074 372.026449,488.261951 374.738992,484.263139 L374.738992,484.263139 Z M159.617103,508.814907 C167.73278,507.880635 172.565182,511.0703 175.186784,517.803111 C180.604553,513.275642 182.311521,507.806602 180.532428,501.386605 C178.986435,495.809122 174.769711,494.033379 169.53487,496.495229 C164.592711,498.820483 162.091318,503.112299 159.617103,508.814907 L159.617103,508.814907 Z M162.947415,533.991262 C164.974244,531.565907 167.105602,529.392889 168.787483,526.917483 C171.562744,522.833168 170.821629,517.009605 167.495497,514.54984 C164.496544,512.330942 157.156476,512.720918 154.818149,515.792756 C151.232784,520.505828 148.438709,525.817419 145.131392,530.740077 C152.561355,526.291854 157.634176,529.927802 162.947415,533.991262 L162.947415,533.991262 Z M355.132378,531.457465 C349.895445,519.319219 333.761304,511.144333 320.16305,513.118192 C320.311482,515.636349 320.370018,518.223325 320.632387,520.79049 C321.173851,526.105208 322.005906,527.417986 327.390225,528.222963 C336.325414,529.556595 345.343182,530.346973 355.132378,531.457465 L355.132378,531.457465 Z M142.43662,562.657584 C144.316062,568.604188 144.546027,567.773144 149.730694,567.338332 C159.29097,566.536484 167.757867,562.953716 175.303858,557.234423 C190.156467,545.977272 202.221272,532.335431 211.006983,515.801098 C212.986774,512.077563 212.583289,508.790925 211.810816,504.006948 C210.434161,506.082994 209.594789,507.219553 208.888169,508.434316 C205.837995,513.669789 202.996881,519.032472 199.769007,524.155331 C191.243574,537.681431 180.049501,548.610127 166.433477,556.981044 C159.164489,561.45012 150.88261,563.379143 142.43662,562.657584 L142.43662,562.657584 Z M186.011868,503.710817 C188.574934,500.104066 190.229637,497.384665 192.270055,494.992676 C195.540786,491.156528 196.54636,486.555026 196.796186,481.841954 C197.02197,477.563694 195.967266,473.381364 191.593748,471.305319 C187.526502,469.37421 184.024761,471.588937 181.267271,474.084154 C176.213265,478.65646 172.417796,484.275651 169.100027,491.112734 C178.746017,489.51738 183.653681,493.989585 186.011868,503.710817 L186.011868,503.710817 Z M140.98993,558.093621 C150.88261,558.093621 157.337312,557.213569 161.655429,555.406544 C162.578426,555.021782 164.417101,551.925961 164.475638,550.830068 C164.839401,544.055548 162.050551,538.748128 156.431041,534.923449 C151.285049,531.42097 147.001427,534.402092 142.800383,536.902523 C137.993067,539.76269 136.280872,544.395473 137.787144,549.424488 C139.328955,554.577585 139.820244,555.277247 140.98993,558.093621 L140.98993,558.093621 Z M441.677649,499.639015 C435.791588,500.64732 426.909709,518.511114 428.682531,529.040451 C435.994375,528.411694 443.068938,526.764204 450.076601,524.521324 C458.058481,521.965629 459.952557,511.644836 453.42782,506.396851 C450.23235,503.825516 446.933397,501.183276 441.677649,499.639015 L441.677649,499.639015 Z M169.727204,549.324387 C170.360653,549.2295 170.678423,548.559034 170.943929,548.368217 C188.652286,535.662734 200.937648,516.440282 209.747402,497.061423 C209.747402,497.061423 209.012559,494.145992 207.469702,492.303514 C204.757159,489.671702 203.332421,489.016876 202.338345,489.671702 C199.715697,491.39948 196.267266,496.518169 194.613608,499.784995 C194.193399,500.617081 193.477372,501.299017 192.912912,502.060199 C184.001765,514.07749 175.092708,526.097909 166.298634,537.961921 C167.38156,541.449803 168.574243,545.609193 169.727204,549.324387 L169.727204,549.324387 Z M454.203429,528.725551 C410.084627,539.181898 366.238646,537.88476 321.997544,532.375054 C321.997544,533.562706 321.849112,535.052746 322.023676,536.503163 C322.717753,542.304829 322.963397,542.762581 328.991619,543.20365 C340.338305,544.033651 351.700671,544.907446 363.069309,545.124331 C385.095784,545.543502 407.079401,544.47889 428.952217,541.720909 C436.627825,540.753269 444.26162,539.462388 452.109702,538.290376 C452.874858,534.79311 453.540712,531.75151 454.203429,528.725551 L454.203429,528.725551 Z M289.680132,331.181099 C289.680132,335.205979 289.844243,338.949326 289.644591,342.674947 C289.281874,349.461979 289.191978,349.457808 295.94668,350.432747 C296.321941,350.486968 296.699293,350.54536 297.07769,350.572471 C312.932738,351.663151 328.526462,350.549531 343.708339,345.414159 C356.039694,341.243299 367.549447,335.610552 377.519479,327.147877 C378.915994,325.96231 380.817388,324.454544 380.990907,322.926966 C381.467562,318.739423 381.314949,314.445522 380.49021,309.281997 C367.022618,319.175278 352.759556,324.964432 337.376982,328.242728 C321.886743,331.542921 306.258524,332.311402 289.680132,331.181099 L289.680132,331.181099 Z M133.093765,324.537961 C137.313625,327.101998 140.864495,329.669162 144.748814,331.553348 C170.751594,344.161859 198.456116,348.257643 227.077362,347.548597 C230.062727,347.474564 233.044956,347.248295 236.1683,347.086674 C236.87492,343.496606 237.550181,340.363248 238.100007,337.206949 C238.665512,333.955764 239.107672,330.683724 239.677358,326.943505 C203.148449,330.08312 169.970758,321.218999 137.940802,304.196676 C136.389583,310.706346 134.832092,317.244169 133.093765,324.537961 L133.093765,324.537961 Z M319.892318,507.509428 C343.434472,510.101618 349.082205,513.389298 361.382202,531.560693 C381.750837,532.197792 402.352573,532.312491 422.53201,529.758882 C422.813194,522.679889 424.543159,516.177518 427.802392,509.152747 C423.923299,508.46664 419.348039,507.785747 415.615287,507.217468 C397.421913,504.451145 379.222266,501.719231 361.019484,499.010258 C359.713909,498.816313 358.324711,498.673461 357.046314,498.903901 C347.592659,500.60874 338.159909,499.912206 328.721932,498.843423 C325.805557,498.512882 322.890227,498.170872 319.892318,497.824691 L319.892318,507.509428 Z M214.640432,443.77347 C217.319525,415.792212 220.017434,387.607624 222.768652,358.865184 C207.67458,362.393731 193.223365,360.225927 178.65194,359.04036 L178.65194,448.540764 C179.935564,448.679445 179.935564,448.679445 180.435215,448.609583 C191.679463,447.04551 202.916393,445.439729 214.640432,443.77347 L214.640432,443.77347 Z M313.619497,363.130931 C315.285699,396.618767 317.251901,429.487231 320.387788,462.479778 C332.387785,461.236862 343.599628,460.074235 355.193005,458.873027 C355.856768,425.299688 356.511123,392.16846 357.165478,358.992395 C349.513912,360.475136 342.709036,362.334297 335.791268,362.982865 C328.6958,363.64916 321.48953,363.130931 313.619497,363.130931 L313.619497,363.130931 Z M152.779822,306.649142 C173.417099,315.777069 194.915699,321.793535 217.552626,323.027067 C223.596527,323.356565 229.697919,322.87066 235.762725,322.559931 C238.013247,322.444189 240.147742,322.614152 240.523003,321.929088 C240.523003,321.929088 241.111504,319.673696 241.533804,317.252511 C244.009064,303.071586 246.389203,288.875021 248.799655,274.684712 C255.904531,232.854112 262.828571,190.994316 270.168639,149.206468 C275.31254,119.921815 281.014629,90.7341357 286.488843,61.5068327 C287.18501,57.78434 288.144592,54.1077267 288.729958,50.3706359 C288.912884,49.2027951 289.591281,46.4468992 289.591281,46.4468992 C289.591281,46.4468992 291.412187,45.840039 292.78257,45.5188828 C295.912185,44.7848114 299.038665,43.8578377 302.21741,43.5575358 C306.188489,43.1821584 307.487792,44.884912 306.547026,48.6491134 C306.182217,50.1099572 305.548768,51.5082381 304.979082,52.9117325 C303.980824,55.3683692 302.927166,57.8031088 301.91532,60.2545319 C299.03553,67.2261248 295.148074,73.9808329 293.581176,81.2350015 C292.045636,88.3369337 292.796159,95.9466682 292.684312,103.335347 C292.25365,131.760802 292.012187,160.189385 291.447727,188.611712 C290.863406,217.974568 289.786752,247.32804 289.318459,276.692981 C289.072815,292.03549 289.664452,307.391555 289.897553,322.74032 C289.912187,323.66938 290.097205,324.595311 290.193372,325.433653 C309.330649,333.165386 358.364432,321.298246 380.506935,303.433409 C379.785681,292.82274 379.158503,281.866933 378.272092,270.933023 C376.635159,250.726248 374.838295,230.534071 373.12401,210.334595 C371.810073,194.853405 370.416694,179.37847 369.251189,163.886853 C368.156764,149.340978 367.61112,134.748181 366.296137,120.225246 C364.059204,95.511856 361.376975,70.8391324 358.941436,46.1434691 C357.528196,31.8165643 355.489413,10.4075388 354.149343,-3.92666504 C354.078263,-4.67950531 353.862932,-5.54495881 354.13889,-6.17163056 C354.470249,-6.92447082 355.21659,-7.941118 355.857357,-8 C356.563977,-8.05790209 357.665719,-7.32383069 358.031572,-6.63668147 C358.53645,-5.68885349 358.58976,-4.46991959 358.705788,-3.35317177 C362.070595,29.058583 366.429935,68.5180487 369.411119,100.964213 C371.299969,121.523426 372.005544,142.18691 373.648749,162.771148 C376.256762,195.445667 379.370698,228.080563 381.946307,260.757167 C383.523659,280.774168 384.419477,300.844347 385.812857,320.876989 C386.065818,324.507723 385.070697,327.105126 382.162683,329.147804 C376.738643,332.957885 371.469307,337.004662 365.888472,340.56762 C364.690563,341.334015 362.177672,343.192134 362.177672,343.192134 C362.177672,343.192134 362.039693,345.242111 362.032376,346.822867 C361.9937,355.349148 361.736557,363.875429 361.560947,372.399625 C360.888822,405.177373 360.213561,437.95512 359.543526,470.731825 C359.517394,472.034176 359.54039,473.33757 359.54039,475.073691 C362.545616,476.045501 365.347009,477.136181 368.239342,477.855655 C389.414946,483.124494 410.599958,488.355795 431.812147,493.475526 C434.382529,494.094899 436.987406,494.562035 439.575559,494.94784 C442.137579,495.329473 443.498554,495.587024 446.585313,496.702729 C449.987751,497.933133 452.895764,500.269857 455.7766,502.460602 C461.722243,506.980771 463.430256,513.396597 460.803428,521.050126 C459.076599,526.080183 457.964404,531.283331 457.246286,536.551128 C456.686007,540.65734 454.766844,542.414314 451.027821,543.038901 C417.942116,548.557991 384.675575,550.914528 351.148755,549.011573 C343.186736,548.56112 335.21322,548.255604 327.271062,547.578882 C320.132736,546.972022 318.623329,545.851103 317.905211,538.646985 C316.670716,526.258487 315.781169,513.822025 315.214619,501.38452 C314.849811,493.347272 314.857128,485.286042 315.551204,477.270691 C316.669671,464.35458 314.591623,451.649097 313.801379,438.860197 C312.176989,412.575436 310.497199,386.29276 308.829952,360.010085 C308.747373,358.712947 308.556084,357.421023 308.352252,355.543093 C305.238315,355.410669 302.241452,355.470103 299.297899,355.107238 C295.553649,354.646358 291.825079,353.957124 288.139365,353.146934 C285.551213,352.576569 284.718112,350.802911 284.730656,348.075168 C284.857136,320.599626 284.58745,293.118871 284.950168,265.646458 C285.300342,239.121872 286.262014,212.605628 286.957136,186.086256 C287.012536,184.002912 287.112885,181.919567 287.148425,179.836222 C287.673163,148.761228 288.192676,117.686234 288.703825,86.6112403 C288.743546,84.1566891 288.709052,81.7010951 287.80278,79.0432145 C287.344941,80.9962198 286.79616,82.9325416 286.44285,84.9043158 C277.808706,133.193493 269.038674,181.458687 260.628223,229.787487 C253.874567,268.600469 247.559935,307.48957 241.051922,346.345304 C240.183282,351.532811 240.189553,351.523427 235.060287,351.741354 C232.818127,351.836241 230.579103,351.983264 228.085027,352.120903 C227.417083,358.501276 226.736596,364.505229 226.170045,370.520652 C223.948791,394.100611 221.656458,417.674313 219.602451,441.269911 C218.156807,457.873063 216.906633,474.497069 215.890606,491.132545 C215.604195,495.815379 216.305588,500.582672 216.817783,505.283231 C217.370745,510.350826 216.572139,515.008635 214.10315,519.526719 C204.562734,536.988025 191.554027,551.342041 175.299676,562.757685 C167.702466,568.094301 159.175988,571.106705 149.865537,571.871015 C142.559965,572.470576 140.350209,571.149456 137.93453,564.292562 C136.487841,560.188435 134.938712,556.108291 133.753347,551.928047 C132.020246,545.814608 133.037319,540.448797 136.76589,534.846289 C146.002124,520.967751 154.551599,506.628334 163.198286,492.366077 C166.528599,486.874097 169.451246,481.127694 172.394799,475.412573 C173.163092,473.922533 173.572848,472.274001 173.585391,470.598358 C173.862395,436.483849 174.11222,402.369341 174.227203,368.25379 C174.250199,361.2603 173.709781,354.263682 173.410827,346.804098 C171.541838,346.258759 169.613267,345.642514 167.655428,345.134712 C155.441146,341.959644 143.967978,337.190266 133.420943,330.14464 C129.256484,327.363719 128.236275,324.179267 129.518853,319.530844 C138.93906,285.395481 147.415364,250.975457 157.780518,217.131012 C177.301418,153.388798 197.895837,89.9739964 218.033462,26.4205139 C221.235203,16.3134769 224.383635,6.1897564 227.554017,-3.92666504 C227.780846,-4.65030929 227.81116,-5.51576279 228.244957,-6.07465806 C228.296177,-6.13930639 228.348442,-6.20291201 228.401752,-6.26443219 C229.50663,-7.53550185 231.672483,-7.2967201 232.277709,-5.72743394 C232.35297,-5.53244623 232.390601,-5.34162937 232.375967,-5.15394066 C232.197221,-2.92044502 231.514643,-0.699461954 230.826838,1.46000092 C222.274227,28.3516222 213.582592,55.1994493 205.123013,82.1202666 C187.918488,136.86385 170.704556,191.604305 153.77808,246.43339 C148.532785,263.425475 144.202124,280.699092 139.149164,298.937221 C143.935574,301.676434 148.16066,304.606463 152.779822,306.649142 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Head" transform="translate(195.000000, 0.000000) scale(1 1)">
                    <g id="Head/Braids" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M132.564158,54.6777031 C155.473343,54.6777031 171.213204,63.8953385 179.783741,82.3306094 L178.829019,93.685776 L178.829019,97.425145 L188.000193,122.406477 L185.554053,147.783329 L181.376158,153.253048 C181.192876,153.770314 181.003564,154.288153 180.808221,154.806564 C179.015011,159.565486 176.362158,163.549907 172.849662,166.759825 L176.83815,174.069571 L178.829019,180.876492 L182.126651,187.315702 L185.554053,195.525022 C187.950142,197.561524 187.474073,201.342156 184.125847,206.866919 C182.097358,207.845652 178.687713,205.939853 173.89691,201.149523 L166.487247,194.41394 L169.844218,189.902206 L161.193388,182.052733 L158.990123,174.295056 C151.500962,176.533866 142.303451,177.233527 131.397591,176.394039 L130.484815,176.320518 L132.077401,191.665688 C134.51512,194.460131 136.017585,198.283443 136.017585,202.5 C136.017585,211.060414 129.824916,218 122.185872,218 C114.653359,218 108.527161,211.252618 108.357759,202.857181 L106.670384,202.741239 L106.272869,198.088067 C105.162615,185.361618 104.075782,175.175598 103.012369,167.530007 L100.740209,167.314234 L76.7324972,161.13577 L74.0752549,161.13577 L70.9062464,171.6836 L70.9062464,177.734707 L66.9203793,185.584427 L68.3097844,189.902206 L63.69148,195.525022 L54.740638,194.41394 L53.2196713,188.318638 L54.740638,175.582282 L55.6033686,164.524827 L57.4985167,161.13577 L54.740638,149.752157 L57.4985167,142.413849 C45.8320121,121.391986 44.9127192,105.149295 54.740638,93.685776 C69.4825163,76.4904976 80.0040997,80.7386953 80.0040997,80.7386953 L80.8928017,79.3824452 C88.1647507,68.4080703 99.5749318,54.6777031 132.564158,54.6777031 Z" id="Path" fill="#FFF"/>
                        <path d="M183.961266,203.430944 C186.709237,207.823745 190.814664,209.222354 195.357883,209.842453 C197.389558,210.119674 197.597935,213.119076 195.635385,213.730838 C195.314805,213.830887 194.971183,213.897587 194.616542,213.94657 C190.357838,214.541656 182.413466,210.051932 181.104097,206.154168 C180.817579,205.298536 181.163204,204.003102 181.646078,203.182904 C181.834419,202.862954 183.674748,202.973425 183.961266,203.430944 Z M162.238393,61.3472448 C167.607963,64.2545113 172.133744,68.0515534 175.737932,72.7393708 C180.728345,79.2267342 182.003892,86.3819291 180.261521,94.0299997 C180.191619,94.3351502 179.767948,95.4642762 179.451411,96.2818788 C185.780988,104.780439 189.486341,114.212498 189.891781,124.716228 C190.180345,132.193673 190.003695,139.603962 187.959606,146.940077 C185.420475,156.061526 180.240257,162.544117 173.338347,167.230033 C173.562606,167.317829 173.966461,167.519875 174.770804,168.038966 C178.281713,170.248328 180.000833,176.128387 179.248108,179.220701 C182.106077,182.102694 186.102895,187.503578 185.000923,191.968931 C188.632225,192.483821 194.34274,193.480861 198.94586,194.132658 C200.984943,194.421353 201.24742,197.080128 199.270161,197.622796 C198.360167,197.8728 197.295071,197.784505 196.252753,197.730933 C193.972886,197.612875 190.260237,197.454142 186.931544,197.278544 C189.643089,200.817294 194.204994,202.248865 197.246264,203.732025 C198.276651,204.234018 199.483831,204.509816 200.372133,205.159629 C201.551113,206.02373 200.967589,207.938445 199.470816,207.990033 C197.971873,208.042614 196.208284,207.901738 194.98375,207.215219 C191.656141,205.348124 188.508579,203.201262 185.379455,201.054401 C183.849059,200.00478 182.518232,198.711108 180.758981,197.245805 C181.085451,192.760611 180.191726,186.88452 176.957394,182.252498 C175.743706,183.83685 174.053871,185.168222 174.053871,185.168222 C172.090712,186.590865 171.92585,185.214849 171.45838,184.205904 C171.25881,183.773357 172.639529,180.830847 175.047381,178.823869 C175.047381,178.823869 176.890148,175.875407 169.626459,169.931855 C169.57438,169.819507 169.511396,169.705937 169.445698,169.591953 C166.699394,171.083045 163.743273,172.352649 160.623294,173.442641 C161.827758,176.448535 162.836274,179.500433 164.566867,182.150924 C166.636491,185.32098 170.12387,188.776698 174.287858,190.314115 C174.756858,192.122317 172.842134,192.858293 170.1314,193.483179 C172.167677,197.887137 174.988131,199.736997 177.902169,201.31409 C179.509247,202.183971 179.319926,204.444471 177.543965,204.967193 C176.808195,205.184415 174.748252,204.266924 174.114672,203.861244 C169.228898,200.729872 165.716778,200.008773 165.220885,193.897786 C165.159571,193.153874 165.086424,191.077865 166.330995,189.6317 C166.330995,189.6317 161.176299,186.653078 160.381366,185.797083 C157.50175,182.579417 157.546929,181.076715 157.082231,176.722352 C157.004411,175.994476 156.9488,175.26073 157.073097,174.587058 C155.751662,174.98031 154.40492,175.346018 153.036725,175.687146 C147.462933,177.076387 141.331761,176.597269 134.988829,177.003216 C135.096355,178.173948 135.176451,179.753633 135.395892,181.315277 C136.304376,187.782365 137.258943,194.244441 138.19376,200.708522 C138.234356,200.996193 138.261786,201.284867 138.274953,201.574543 C138.347368,203.194322 138.360535,205.033613 136.034464,205 C135.146827,204.986503 133.742406,203.177282 133.519674,202.034616 C132.612287,197.380758 131.73233,190.905652 131.237492,186.19867 C130.334493,182.328641 130.819457,182.004886 129.3119,180.557509 C128.788534,180.055337 127.365461,178.522762 126.966079,177.933386 C124.105671,173.703518 123.259727,173.24144 121.062029,168.846185 C121.009363,168.628678 120.992905,168.403152 121.001683,168.172614 C121.068612,166.519758 122.721,165.341008 124.498469,165.655742 C125.044877,165.751966 125.527646,165.910336 125.872168,166.204021 C135.163285,174.137531 145.856627,173.70853 156.635551,170.139202 C172.154394,165.000213 182.65024,155.695504 184.52536,139.804429 C185.98464,127.4476 185.517232,115.3604 178.771626,104.227426 C176.499318,100.475673 173.999889,97.3132949 170.246357,93.8371849 C169.735298,93.2311889 169.160975,91.8480961 168.383092,91.0798913 C168.247457,90.9459433 167.22618,90.1250189 167.181869,90.0089369 C167.03967,89.6368204 166.905954,89.5643908 166.881314,89.5119716 C163.282414,93.38074 158.571483,95.7912263 152.737313,96.5824968 C151.320352,96.7748482 149.85095,96.2088143 148.681984,96.6906945 C148.681984,96.6906945 146.647765,98.701367 145.64595,99.6901731 C143.07204,102.229812 139.397991,103.637383 135.61797,103.71252 C134.419507,103.736564 133.147847,102.974171 132.193009,103.325814 C132.193009,103.325814 130.599064,105.142131 129.814655,106.110901 C125.489481,111.451656 119.720033,114.203682 112.251761,113.153765 C108.870352,116.039208 106.889579,120.783978 101.878834,121.803295 C101.491598,122.189859 101.008318,122.519068 100.570552,122.646964 C97.9548196,123.407739 90.1515104,124.956889 87.6630535,125.937176 C85.5399653,126.773126 83.5562219,128.238545 81.9652772,129.80921 C76.9367949,134.777802 78.1097052,140.552274 85.0330574,142.870683 C89.4317451,144.344121 94.3186882,144.723005 99.0322732,145.257251 C104.495248,145.875694 105.438843,146.301688 105.9655,150.958553 C107.665068,165.994633 109.245041,181.041738 110.808555,196.089847 C111.014829,198.073475 111.019218,200.105215 110.782222,202.080824 C110.704321,202.733346 109.606021,203.709624 108.907102,203.759741 C108.157712,203.812865 106.904706,203.101205 106.630405,202.453694 C106.146539,201.311028 106.222246,199.96589 106.089484,198.703945 C105.046934,188.791646 104.008016,178.879346 102.96309,168.96758 C94.4972906,167.91768 83.1451071,167.152076 75.772964,162.76266 C75.772964,162.76266 74.2881134,169.902859 72.4076678,172.975084 C73.2733741,174.669656 73.4530903,177.890843 72.1238476,179.33048 C70.1677898,182.280736 68.8056722,183.620398 68.8056722,183.620398 C69.1136006,185.306973 71.2921121,192.042273 66.3521071,196.832065 C64.2579746,197.60687 64.2481121,201.053 64.2031831,204.095233 C64.1724998,206.171709 63.2476186,208.351159 62.1989086,210.243682 C62.0104257,210.583596 61.5162061,210.780546 60.9200742,210.932508 C59.6982229,211.24243 58.4818508,210.44863 58.3854177,209.295921 C58.363501,209.034986 58.3624052,208.776052 58.3974718,208.521116 C58.7108794,206.212698 59.3365987,203.939272 59.6280897,201.630854 C59.6993187,201.061998 60.0609429,198.955529 59.5327524,198.054757 C57.39917,200.564123 54.4590561,204.309179 52.4646442,206.490628 C51.5846921,207.452386 50.5403654,208.448135 49.3327599,208.949008 C49.1311271,209.032987 48.9086734,209.072977 48.6708781,209.084974 C47.1487692,209.159955 45.9115763,207.789301 46.3290879,206.451638 C46.3893585,206.258687 46.4759292,206.084731 46.6030455,205.938768 C48.5240368,203.720327 52.977493,199.713338 54.4316603,196.756084 C51.7622167,198.433661 46.6348246,200.10424 43.8141563,200.50014 C41.5885241,200.813061 40.6976137,200.603114 40.1310692,199.179473 C39.6894494,198.069753 40.403931,196.846061 41.6718072,196.609121 C45.9313013,195.815322 50.4011949,195.352438 53.6645788,191.848322 C52.0175452,189.046029 51.1134849,186.667629 51.0345851,184.305225 C50.9688352,182.371713 51.6920835,180.497186 52.8394183,178.870596 C53.0465303,178.575671 53.1944674,178.316736 53.1900841,178.174772 C53.1900841,178.174772 52.3134195,175.857356 52.1556199,175.227515 C51.1080057,171.069564 52.271778,167.380495 54.4765894,163.739414 C54.8042428,163.19855 55.8376112,162.083831 56.0041775,161.420998 C56.0041775,161.420998 55.508862,160.088335 55.3357207,159.293535 C54.2826274,154.463754 52.9950262,150.628721 55.0902546,144.154354 C55.0902546,144.154354 53.3873337,141.099125 52.4186193,139.388557 C44.152768,124.796238 43.9642851,110.087949 52.0109702,95.463638 C55.4551663,89.2052168 60.3842129,83.8615649 67.7043623,81.0532734 C70.9480213,79.8095871 74.5270045,79.2937173 78.5103492,78.3109652 C85.9137817,64.6264175 99.4505788,57.6711722 115.657915,54.4399873 C132.096472,51.1628141 147.719731,53.4852282 162.238393,61.3472448 Z M90.4592876,70.510933 C87.3537033,73.1572654 84.9790381,76.7333632 83.0536634,80.2394787 C81.7956498,82.5309006 80.7787188,83.7395957 77.8517549,83.4456698 C71.3107412,82.7868361 66.3115614,85.6981016 62.0783673,89.7890695 C53.3643212,98.2119446 50.1743579,108.439364 50.7430941,119.818494 C51.319501,131.332589 57.7531233,140.428294 65.2376473,149.096108 C66.210745,150.222823 67.1958968,151.34454 68.0922864,152.522243 C68.2577568,152.739189 68.0254407,153.21007 67.9377742,153.856907 C63.3637767,155.875397 61.3540231,150.15684 58.5366423,148.950145 C57.2775328,152.620219 58.9914121,157.524981 61.9468676,163.050587 C62.1923336,163.510471 62.5386162,163.955359 62.824628,164.418242 C63.7144425,165.856879 61.9326218,167.449478 60.3546255,166.640682 L57.8901022,164.284276 C54.6584973,170.771639 56.7307132,175.608419 58.6078713,179.778367 C58.0807767,180.582164 56.4436056,180.181266 55.7203573,179.821356 C55.7203573,179.821356 54.8864301,183.408451 56.0688315,186.044786 C57.1219248,188.392194 57.6939485,190.641627 57.5164239,191.345449 C58.3558302,192.443172 62.1978128,193.236972 63.2772061,194.146742 C64.5319323,193.501905 65.705567,191.251473 65.0984768,189.878819 C64.928623,189.495916 57.9547562,184.91907 59.7968476,184.140267 C62.267946,183.484432 65.6376255,185.574905 65.6376255,185.574905 C66.4090903,180.415207 66.9263224,178.904588 68.884572,175.046561 C63.6399261,175.553433 62.1780878,175.491449 62.8969528,173.564935 C63.7922465,172.283258 65.5346174,172.290256 66.4660735,171.690408 C67.4216379,171.075563 69.4883747,168.535204 70.0593025,167.616435 C72.6366964,163.465483 72.7068296,158.477741 71.2855371,153.488999 C70.7957008,151.770433 72.9216124,150.23482 74.5270045,151.239567 C75.2064195,151.66546 75.623931,152.402274 75.7126933,153.308045 C76.4447083,160.766164 82.7753224,161.640943 88.6928084,163.060585 C91.7677095,163.798399 99.0896731,163.701423 102.171149,164.418242 C102.288564,164.445549 102.394994,164.473475 102.491468,164.502772 L101.217902,152.482107 L101.217902,152.482107 C101.144389,151.789491 100.832784,151.118927 100.570552,150.230855 C95.8459951,149.615419 91.655776,149.304694 87.5873465,148.486786 C80.997544,147.163699 75.2712405,144.334097 74.167454,137.64449 C73.1371802,131.400923 76.9803713,127.668561 82.4140334,123.544594 C87.5647694,119.635358 93.7952476,119.562161 97.0114725,118.598555 C97.0479254,118.411718 97.2153605,118.268312 97.5837719,118.177273 C101.086404,117.311734 104.884401,115.729097 105.913125,113.692749 C107.660019,110.235434 109.921477,106.998522 111.710978,103.556235 C113.085332,100.911404 114.240096,98.1693953 115.464779,95.3813028 C115.888666,94.4155388 117.350419,94.2291985 117.987342,95.0907722 C119.172695,96.6936999 118.289962,99.0620257 114.618098,108.913019 C116.29835,108.81484 117.810358,109.022218 119.071093,108.598444 C123.602748,107.076665 126.851818,104.097223 128.446856,99.9847111 C129.898777,96.2408728 130.737811,92.2976705 131.845597,88.4416273 C132.108888,87.5269567 132.368901,86.6102823 132.655133,85.6044451 C132.964308,84.5134524 134.461022,84.091682 135.34157,84.8791203 C137.34083,86.6643811 136.238506,88.9866228 136.020007,90.8941068 C135.717387,93.5419432 134.807342,96.1326752 134.155124,98.7745006 C139.945329,99.360571 143.810565,97.3569113 145.590233,92.2696192 C146.97442,88.3123913 147.580753,84.1237406 148.480966,80.0292619 C148.860061,78.3001036 148.586938,75.73041 151.288669,76.0229443 C154.058135,76.3234932 153.153552,78.8661374 152.918667,80.5341842 C152.388808,84.2890425 151.602214,88.0128441 150.810158,92.3557766 C155.36257,91.9800904 159.162256,91.028352 162.024584,87.9928075 C163.708113,86.2075467 165.107595,84.1958723 166.812974,82.4296463 C167.259803,81.9668009 167.834454,81.6081458 168.428769,81.2735346 C169.31478,80.7736215 170.535093,81.100218 170.851915,82.0088777 C171.354462,83.4515127 170.477191,84.6106299 169.726649,85.8008038 C169.618966,85.9715825 169.50982,86.140496 169.39921,86.3075367 C171.094972,87.5075705 173.790199,89.9387388 175.743544,91.9059107 C178.5694,81.7176993 172.998854,74.262195 164.686479,68.116537 C143.191759,52.2235465 109.794125,54.03309 90.4592876,70.510933 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Body" transform="translate(28.000000, 114.000000) scale(1 1)">
                    <g id="Body/Magnifying Glass" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M450.17935,155.578699 C450.218316,155.66562 450.246291,155.757537 450.266273,155.85345 L450.17935,155.578699 Z M434.161126,159.905132 C428.340152,152.478189 424.316803,140.211558 429.558105,142.023758 C433.270281,143.307256 446.202251,158.119553 446.138498,158.012034 C441.630347,150.379821 438.245508,142.945097 435.662432,134.294524 C434.899591,131.736157 435.076229,129.392021 436.571957,128.070447 C437.741181,127.035131 439.793634,126.832097 439.793634,126.832097 C438.580477,121.348315 438.718315,117.427501 445.351238,119.921036 C444.301832,116.744272 444.140534,113.890424 447.289749,112.536933 C450.372067,111.213365 452.954144,112.390313 455.216707,114.515803 C461.749782,120.652892 466.733212,128.094598 471.440061,135.658987 C473.367134,138.755957 475.557806,141.588613 475.923251,145.324927 C476.238771,148.566522 475.527851,151.813105 474.768006,154.945983 C469.092627,178.358284 463.534072,201.807489 457.284566,225.068183 C454.61562,235.000433 450.752489,244.674352 446.725606,254.167739 C439.802124,270.493413 421.012668,277.456361 404.843232,270.239073 C395.752043,266.181592 388.333314,259.79116 380.902602,253.397736 C379.585603,252.263676 378.232658,251.173502 376.092909,249.397108 C376.092909,252.497071 376.003045,254.608596 376.110881,256.711145 C376.387461,262.157026 373.113834,265.371989 373.113834,271.780339 C373.113834,278.18869 362.16866,278.003994 355.817309,280.139457 C326.856305,289.875216 291.481447,289.951965 260.799058,288.473797 C242.285184,287.582109 224.826991,281.342286 209.67401,270.239073 C208.525755,269.397255 207.257485,268.613288 205.702846,267.705641 C204.908054,269.092046 204.317862,271.842914 203.728757,273.080705 C193.902685,293.72618 185.344686,314.858392 178.792639,336.796515 C177.656365,340.607634 178.263443,343.156027 180.752662,346.090419 C185.199905,351.334823 189.36358,356.841546 193.332551,362.458982 C194.970063,364.776972 196.190209,367.563747 196.904125,370.317608 C198.076344,374.845868 194.588642,377.793226 190.187329,376.15647 C188.52785,375.53907 187.031124,374.403016 185.577332,373.343762 C184.282299,372.399211 183.147024,371.234231 181.254899,369.559573 C182.242398,374.052923 183.162001,377.672539 183.798035,381.341028 C184.197428,383.64306 184.478002,386.044833 184.271316,388.352849 C183.803028,393.605232 178.975362,393.574312 175.990895,392.20985 C174.853623,391.691194 174.026879,390.652886 173.029394,389.851963 C168.619094,394.193706 166.567211,395.819491 159.848418,391.316167 C157.488004,393.45163 154.579423,394.295442 151.986362,391.665262 C149.981408,389.631534 150.448698,390.518235 147.03089,383.980186 C147.058848,383.976196 146.424811,384.785099 146.300001,384.873869 C144.462792,386.187463 143.024976,388.612177 140.23122,387.457171 C137.401519,386.286207 137.072019,383.498435 137.023094,381.03183 C136.944213,377.064117 137.04506,373.029577 137.678099,369.121708 C139.443417,358.217977 141.224711,347.300283 143.557167,336.50926 C146.158216,324.484435 149.448218,312.609223 152.309871,300.638259 C158.429574,275.038629 167.833288,249.34624 178.792639,225.068183 C177.833097,223.791493 176.803661,222.752163 175.990895,221.405654 C173.224099,216.818546 170.307529,212.611479 173.384854,207.585509 C176.129685,203.10313 178.611914,198.462162 181.315806,193.954848 C193.257664,174.05544 205.230476,154.173984 217.206283,134.294524 C223.122295,124.47598 229.959908,115.360613 237.828954,106.998293 C247.311548,96.923413 254.461049,91.8405908 267.28357,87.4679275 C272.153172,85.807233 270.682523,81.5128244 273.87068,85.9493222 C275.724863,88.5306299 303.378645,88.5830364 303.810989,87.4679275 C304.814464,84.8806353 303.913561,84.6931215 306.498633,84.3609826 C310.739191,83.8163945 314.558389,86.1533357 318.069056,88.1711044 C330.25754,95.1779384 341.428569,103.525297 351.011012,113.831577 C367.368163,131.423966 381.697395,150.57332 394.04963,171.164934 C396.744536,175.655292 398.653635,180.60845 401.137861,185.234457 C403.29159,189.246056 403.884689,193.000323 400.234234,196.541143 C404.938088,202.357064 411.272465,210.252554 416.266878,216.430537 C417.000763,214.784803 417.371518,211.79608 417.820836,210.566268 C420.683487,202.740557 421.941258,194.437597 424.806904,186.612883 C424.9417,186.242842 425.063515,185.854848 425.19831,185.474833 L434.161126,159.905132 Z" id="Fill" fill="#FFF"/>
                        <path d="M495.359491,3.35298718 C512.970344,7.73107325 523.217005,19.8847601 527.87676,36.7823731 C534.838888,62.0373326 524.206165,91.0726394 502.911715,106.424928 C492.136969,114.192532 480.199035,117.595045 466.924884,115.927774 C465.860712,115.793832 464.795539,115.665888 463.246288,115.474972 C462.620786,116.964813 461.980575,118.431894 461.365852,119.903975 C465.3302,124.647592 468.733406,129.845676 472.002407,135.102891 C473.931719,138.206534 476.128075,141.045293 476.492134,144.789656 C476.809186,148.038236 476.09607,151.292813 475.336947,154.431441 C469.651026,177.894184 464.083124,201.393911 457.822109,224.70472 C455.148676,234.658369 451.280049,244.35413 447.247396,253.866972 C440.310272,270.227819 421.491223,277.205769 405.293598,269.972931 C396.186123,265.906708 388.755919,259.502508 381.312713,253.09431 C379.994499,251.958806 378.63828,250.866284 376.494932,249.085062 C376.494932,252.192704 376.405918,254.308779 376.511935,256.415858 C376.78898,261.873472 379.953493,274.892346 375.18272,276.869481 C368.981715,279.439358 359.696235,281.890852 351.369862,283.95967 C321.540372,291.371273 291.7412,289.728295 261.008221,288.246942 C242.463216,287.353333 225.196419,282.53344 210.017959,271.406304 C208.866773,270.563673 207.55156,269.943944 205.994307,269.034341 C205.198178,270.424733 204.431054,271.580228 203.840958,272.820686 C193.998364,293.511641 185.425975,314.687383 178.863912,336.673772 C177.723727,340.493102 178.332826,343.045986 180.82623,345.9877 C185.281952,351.243402 189.451627,356.76199 193.427271,362.391529 C195.067537,364.715513 196.290735,367.508292 197.004851,370.267086 C198.179041,374.805102 194.685475,377.758811 190.276761,376.118528 C188.614491,375.500798 187.116249,374.361296 185.660013,373.29976 C184.360802,372.353174 183.224618,371.185684 181.330311,369.508418 C182.318471,374.01045 183.239621,377.637864 183.876724,381.314257 C184.276789,383.620249 184.557834,386.028196 184.350801,388.341185 C183.880724,393.604884 179.046941,393.573897 176.056457,392.207494 C174.917272,391.686722 174.089138,390.646177 173.089976,389.843528 C168.67226,394.193626 166.616927,395.823914 159.886837,391.309887 C157.522454,393.45195 154.609982,394.296581 152.011561,391.661733 C150.003236,389.622624 150.471311,390.511236 147.047757,383.9601 C147.076761,383.954103 146.440658,384.765748 146.315638,384.854709 C144.47534,386.170134 143.035107,388.601071 140.236653,387.443577 C137.402194,386.27009 137.07214,383.476311 137.023132,381.004392 C136.94412,377.02813 137.045136,372.984898 137.679239,369.06861 C139.448525,358.141387 141.230814,347.20017 143.569193,336.385898 C146.173615,324.335166 149.469149,312.434368 152.335613,300.437613 C158.465607,274.781829 166.192859,249.657812 177.170637,225.326449 C176.209482,224.048008 175.142309,222.851531 174.329177,221.50312 C171.556728,216.907129 170.363534,212.222177 173.446034,207.18438 C176.194479,202.692343 178.681882,198.041377 181.390321,193.524351 C193.352259,173.582069 205.345202,153.65778 217.342146,133.735489 C223.267106,123.89579 230.116216,114.761783 237.999493,106.380447 C247.496032,96.2838613 256.951963,88.3456316 269.796044,83.9635472 C274.674835,82.2992748 274.972883,82.8610292 278.1664,87.3070856 C280.024701,89.8939547 282.29607,92.4318452 284.927496,94.1600897 C289.925306,97.4426547 296.24503,97.6494583 299.907623,91.5601203 C300.525724,90.5335691 301.941953,87.9127149 302.375023,86.796203 C303.380186,84.204336 304.088301,84.0154186 306.67772,83.6825641 C310.925409,83.1368027 314.857946,85.4905797 318.373515,87.5126957 C330.582493,94.534626 341.772306,102.899969 351.369862,113.228454 C367.754516,130.859746 382.108842,150.048357 394.482847,170.684336 C397.180284,175.185368 399.093594,180.149198 401.581997,184.785172 C403.740347,188.805414 404.332443,192.56777 400.67685,196.116218 C405.388614,201.94467 409.939351,207.575209 414.944162,213.765503 C415.678281,212.118223 416.28738,210.943736 416.737453,209.712275 C418.94383,203.675683 421.120579,197.62784 423.315136,191.586522 C423.220672,191.494825 423.124838,191.398297 423.029772,191.298824 C420.661388,188.819908 421.565535,186.406963 422.597702,183.780111 C425.952524,175.248658 429.298783,166.71367 432.640495,158.176913 C428.900583,154.222529 425.822131,149.788939 424.823963,144.280579 C424.166856,140.656163 427.118334,138.364165 430.654907,139.687587 C433.701772,140.828966 435.780416,143.051064 437.584205,145.537231 L438.535556,143.102644 C437.062919,140.082718 435.770923,136.966772 434.791378,133.683512 C434.026254,131.119632 434.238288,128.021987 435.735531,126.698565 C436.908721,125.661019 438.93905,126.02286 438.93905,126.02286 C437.723853,120.527263 438.328951,116.573991 444.973027,119.071899 C443.922857,115.888291 444.657976,113.287428 447.811487,111.93202 C448.887447,111.46928 449.902634,111.311186 450.864762,111.391808 L451.179333,110.574114 L451.179333,110.574114 C448.19685,107.961256 445.564423,105.990118 443.327061,103.642144 C433.13941,92.944821 428.625679,79.9505016 429.023743,65.3908665 C429.648844,42.4998736 439.032365,23.942986 458.01244,10.8047296 C469.260262,3.01913313 481.884308,0.00245191193 495.359491,3.35298718 Z M183.040588,223.084429 C182.206453,224.899635 181.684369,226.043135 181.156283,227.183637 C165.772791,260.364131 157.780496,295.798641 149.53816,331.14219 C146.879729,342.543206 146.587682,354.216103 147.553839,365.826027 C148.014913,371.370604 149.572166,376.858205 151.013399,382.270838 C151.392461,383.694216 152.269603,384.984652 153.348778,386.616938 C154.054892,387.685471 155.724162,387.084734 155.58314,385.81229 C154.863023,379.691966 154.035889,373.584636 153.332775,367.463312 C152.980718,364.398651 152.74668,361.315999 152.607658,358.235346 C152.566651,357.308751 152.690671,356.397149 153.463796,355.795412 C154.388946,355.074728 155.762169,355.592501 156.116226,356.709013 C156.492287,357.896494 156.878349,359.115961 157.043376,360.364415 C157.925519,367.019506 158.438602,373.729572 159.510776,380.351677 C160.016858,383.47931 161.248057,386.244101 163.595438,388.376169 C165.403731,390.017452 168.316203,388.69603 168.428221,386.256096 C168.538239,383.86914 168.161178,381.575143 167.815121,379.31613 C166.747949,372.336181 165.422734,365.394216 164.256545,358.429261 C164.017506,357.004884 163.706456,355.479551 163.991502,354.119145 C164.084517,353.676339 164.374564,353.274515 164.731622,352.887684 C165.481743,352.076038 166.796957,352.077038 167.535076,352.898679 C167.768114,353.159565 167.944142,353.430446 167.999151,353.724318 C168.570244,356.750995 168.722268,359.854638 169.220349,362.898307 L169.436656,364.232826 C170.546175,371.131481 171.587733,378.069402 175.21532,384.246975 C175.875427,385.370483 176.429517,386.691906 177.587705,387.586515 C178.612871,388.377169 180.099112,387.568522 180.035101,386.276087 C179.959089,384.741758 179.888078,383.308385 179.566025,381.933986 C177.870751,374.693151 175.934437,367.509292 174.209157,360.273455 C173.870103,358.849078 173.744082,357.374722 173.616061,355.958341 C173.513045,354.814841 174.483202,353.821276 175.621386,353.974209 C176.99961,354.161127 177.692722,355.254649 178.398836,356.379157 C180.329149,359.450814 182.171448,362.59544 184.311794,365.516163 C186.38113,368.339928 188.743512,370.977775 192.226077,372.130271 C192.303089,372.039311 192.377101,371.946352 192.449113,371.851393 C193.04721,371.059739 193.126222,369.996204 192.634143,369.135581 C191.25792,366.732631 189.836689,364.527595 188.167419,362.262585 C184.077756,356.714011 179.700047,351.370347 175.306335,346.052672 C173.387024,343.730687 172.778926,341.419697 173.746082,338.514967 C176.152472,331.285127 178.266815,323.956331 180.758219,316.756479 C188.193423,295.27387 197.980009,274.794823 207.647575,254.264798 C209.871936,249.540863 211.884262,244.714973 214.21664,239.432282 C203.653928,233.892704 193.461277,228.54804 183.040588,223.084429 Z M142.770064,358.581145 C142.567031,358.557205 142.001939,367.209423 141.973935,367.411334 C140.825749,375.745691 140.225651,382.479747 141.26582,382.479747 C146.704701,381.548154 143.632203,374.002453 143.728219,369.696336 C143.503182,369.670347 142.9951,358.608183 142.770064,358.581145 Z M308.887549,88.4083042 C308.21444,89.6907435 307.705363,91.1630104 307.387312,91.8087281 C300.246155,106.281401 282.528163,104.116437 274.42285,91.2510614 C273.772745,90.2195124 272.975616,89.2799231 272.329511,88.4083042 C261.635778,90.6203371 249.949429,100.386068 242.208175,108.337592 C234.183875,116.579989 227.30576,125.698002 221.383801,135.541699 C207.900616,157.949903 194.427433,180.364105 180.980255,202.793299 C179.310984,205.579081 177.818742,208.470817 175.909433,211.91931 C189.080567,212.589017 198.073024,218.742327 206.171336,223.084429 C214.432675,227.514492 218.796382,229.476634 226.355606,236.884396 C226.62265,236.21269 226.906696,235.598958 227.11573,234.961237 C232.837657,217.553847 238.549582,200.143458 244.268509,182.735068 C244.513548,181.990394 244.791593,181.257714 245.08164,180.530032 C245.673736,179.049679 246.281835,177.350422 248.420181,178.189055 C250.333491,178.938728 249.634378,180.505043 249.166302,181.898434 C248.531199,183.790607 247.898097,185.683779 247.290998,187.585947 C241.50206,205.732015 235.701121,223.874084 229.950189,242.032146 C228.736992,245.860472 226.839685,247.371811 222.284947,247.470768 C220.153602,247.516748 218.015255,247.273854 215.75989,247.155906 C213.175471,252.81843 210.70007,258.24106 207.981629,264.198455 C211.160144,266.537433 213.836578,268.692491 216.692041,270.576667 C228.906123,278.636298 242.477998,282.375954 256.76476,283.592204 C256.0812,282.974775 255.346152,281.827654 255.523632,281.151944 C256.61981,276.995761 257.071883,272.622673 259.924345,269.027244 C261.349576,267.23103 262.575775,265.062977 263.16087,262.869936 C263.998005,259.72631 262.1127,257.833138 258.923183,258.499846 C256.468785,259.013622 254.194417,260.348039 251.752021,260.974765 C250.791866,261.220657 249.014578,260.853817 248.645518,260.191107 C248.171441,259.33748 248.433483,257.744177 248.949567,256.771602 C249.394639,255.933968 250.563829,255.228277 251.545988,254.972389 C254.873527,254.104768 258.232071,253.245144 261.632622,252.806335 C264.39707,252.449491 266.571422,253.672957 267.447564,256.599677 C268.117673,258.837699 268.866794,261.05273 269.650921,263.495662 C268.923803,265.14894 268.280699,266.806215 267.48357,268.385525 C266.113348,271.097339 264.713121,273.800158 263.201876,276.436005 C261.955674,278.611055 260.619458,280.74912 259.132217,282.76224 C258.864003,283.125764 258.203995,283.49266 257.660416,283.664256 C258.597028,283.73782 259.53667,283.79991 260.479135,283.851864 C276.873791,284.755469 298.675516,285.705143 315.029073,283.851864 C337.516699,281.303436 366.107668,276.711363 370.912867,269.537359 C372.488028,267.185695 372.721321,259.79638 372.190235,255.027464 C371.226079,246.371249 370.650985,237.664055 369.431788,229.045823 C367.678504,216.652241 364.831043,204.494556 361.595518,192.371855 C361.432382,191.760586 361.26851,191.149796 361.103711,190.539539 L360.8682,190.617322 L360.8682,190.617322 C356.79254,191.920752 352.950918,191.612887 349.615377,188.730147 C347.959109,187.299773 346.709906,185.779437 347.03996,183.135593 C347.417021,180.10092 345.835765,178.575586 342.873285,179.349248 C338.272539,180.549723 333.033691,179.777061 329.215072,183.797304 C328.081888,184.990782 325.956544,185.301646 324.235265,185.8634 C321.494821,186.758009 318.735374,187.611636 315.942922,188.324325 C310.396023,189.739706 305.392212,188.390296 301.325553,184.517989 C297.114871,180.508741 295.248569,175.398975 296.73781,169.641492 C298.525099,162.732512 302.397727,157.15595 308.436705,153.135708 C310.064969,152.051182 311.719237,150.762745 312.825416,149.190432 C315.160795,145.866885 314.473683,142.518349 311.151145,140.372287 C310.50504,139.95547 309.162823,140.295321 308.273679,140.633173 C305.609247,141.645731 303.010826,142.831212 300.385401,143.947724 C298.224051,144.865323 296.091705,145.861888 293.893349,146.677531 C289.500637,148.308818 285.173936,150.438887 280.623199,151.219545 C277.131633,151.819283 273.274008,150.951663 269.717432,150.13302 C268.53424,149.861139 267.436062,147.844021 266.899975,146.401652 C265.440739,142.473369 265.886811,138.565077 267.755114,134.799724 C270.575571,129.11221 274.810257,124.439253 279.975094,121.120704 C288.761518,115.473172 297.796981,110.085528 308.051643,107.299746 C308.725752,107.116825 309.398861,106.756983 309.94895,106.322173 C313.246485,103.723309 317.164119,103.044606 321.168768,102.625789 C322.614002,102.473855 324.069238,102.418879 325.511472,102.250953 C326.869692,102.093022 328.21691,101.844131 329.570129,101.635222 C328.238914,102.603798 327.04472,103.997189 325.556479,104.47498 C311.850258,108.878056 299.058186,115.258266 286.531156,122.224221 C285.450981,122.823959 284.591842,123.836516 283.668692,124.69814 C282.741542,125.562762 281.997422,127.022124 280.94125,127.284009 C275.205321,128.70039 274.297174,133.911112 272.147826,138.095283 C270.949632,140.426264 270.423546,142.945163 272.746923,145.096222 C275.469364,147.616121 278.7789,148.300821 281.696373,146.810473 C287.017235,144.093661 292.190073,141.049991 297.649958,138.655038 C301.100517,137.1417 304.912134,136.298069 308.646739,135.618366 C314.169634,134.613805 318.426324,136.694895 320.650684,140.976024 C322.322955,144.196616 321.849879,146.761494 319.374478,149.372353 C317.91124,150.915678 316.689042,152.687904 315.369829,154.36617 C314.70172,155.217798 314.231644,156.558212 313.375506,156.875073 C308.316686,158.741257 307.23251,163.798047 304.681097,167.581393 C302.179692,171.291771 302.053671,175.747823 304.591082,179.423216 C306.975469,182.877706 311.030126,183.217557 314.983766,182.527859 C316.942083,182.186008 318.899401,181.672233 320.761702,180.979535 C326.704665,178.769502 332.544611,176.270594 338.541583,174.227487 C341.591077,173.187942 344.872609,172.603197 348.08913,172.360303 C349.801407,172.23136 351.208635,172.796113 350.441511,175.770813 C349.125298,180.874581 353.793054,185.782436 359.063908,186.057316 L359.80812,185.834019 C356.680704,174.720222 353.076504,163.830082 347.816286,153.484855 C347.159179,152.19342 346.276036,150.899985 346.087006,149.526586 C346.067002,149.375652 346.067002,149.220719 346.083005,149.063788 C346.273036,147.278568 348.332369,146.172052 349.847615,147.13763 C350.016642,147.244583 350.153664,147.371528 350.24668,147.524461 C351.653908,149.832452 352.675073,152.382337 353.76925,154.872249 C358.010346,164.524188 361.242697,174.507359 364.15165,184.597435 C365.397111,184.226333 366.626254,183.817173 367.814326,183.311516 C373.991327,180.683665 380.051309,177.778935 386.166299,175.001149 C387.801564,174.257474 389.81189,172.959042 391.116101,174.795239 C392.703358,177.031262 390.471997,178.656551 388.94575,179.627127 C381.882605,184.116164 374.430398,187.743578 366.008033,189.012024 C365.815399,189.041001 365.62373,189.076365 365.432871,189.11709 C366.190202,191.824316 366.930707,194.535539 367.668502,197.247724 C368.118575,198.897003 368.805687,200.483309 369.246758,201.719769 C378.570269,192.986586 383.273031,190.184811 397.352312,185.699772 C395.034936,181.253716 392.979603,176.131955 389.93411,171.687897 C380.598597,158.064853 371.177071,144.479792 361.239461,131.293556 C349.663585,115.93627 335.679319,103.005923 319.258659,92.8503623 C316.728249,91.2850466 313.135702,89.4355134 308.887549,88.4083042 Z M273.969521,197.835867 C276.718966,200.736598 280.743618,203.094568 278.289221,208.193339 C278.094189,208.599161 278.165201,209.380819 278.441245,209.725669 C282.141845,214.338652 280.312549,219.207524 278.563265,223.658578 C277.23405,227.041099 278.865314,229.682944 279.357394,232.600669 C279.453409,233.176417 281.754782,234.116006 282.337877,233.755164 C285.828442,231.595108 289.175985,229.185162 292.441514,226.689253 C294.117785,225.407813 295.464003,223.705557 297.068263,222.319163 C298.431484,221.140679 299.987736,220.187095 301.398965,219.058589 C307.931023,213.831874 315.187199,210.448353 323.693577,210.170474 C329.81857,209.969562 334.119267,213.612969 335.49649,219.738292 C337.996895,230.865427 333.547174,239.465668 325.972947,247.026362 C325.844926,247.154306 325.709904,247.298243 325.548878,247.367213 C317.246533,250.948648 315.490248,259.659839 310.990519,266.192983 C310.362417,267.103585 311.172549,269.19767 311.651626,270.633042 C312.433753,272.975018 312.991843,275.280011 311.239559,277.321119 C310.62246,278.041803 309.317248,278.701515 308.484113,278.545583 C307.766997,278.411642 306.874852,277.141197 306.73983,276.267579 C305.076561,265.540269 306.447783,255.70057 315.229206,248.06191 C318.239694,245.442055 321.140164,242.68526 323.947618,239.850499 C326.634054,237.137685 329.137459,234.303924 329.130458,230.051783 C329.128458,228.666389 329.873579,227.264002 330.348656,225.8956 C332.04193,221.024729 329.363496,216.598664 324.236665,216.223828 C319.57591,215.883977 315.228206,217.264373 311.24056,219.614346 C306.86085,222.192219 302.550152,224.88904 298.265457,227.621845 C297.384315,228.1836 296.522175,229.012237 296.041097,229.923839 C293.479682,234.782715 288.827928,236.980754 284.234184,239.105825 C280.186528,240.978006 279.343392,240.028421 276.346906,236.48697 C273.147388,232.704623 272.469278,228.459479 273.46844,224.267312 C275.511922,215.686336 275.088566,207.19559 274.066989,198.636488 L273.969521,197.835867 Z M227.932462,262.7242 C231.19099,263.520851 234.317496,263.816722 237.518015,262.490302 C238.806224,261.957535 240.706532,261.278832 241.151604,263.22798 C241.366638,264.170567 240.198449,265.898812 239.191286,266.538532 C237.727049,267.469125 235.841743,267.738008 233.655389,268.444699 C231.542047,267.590073 229.112653,266.865389 226.962305,265.647922 C225.498067,264.818284 223.206696,262.451319 223.367722,262.21942 C224.902971,260.014384 226.410215,262.352362 227.932462,262.7242 Z M459.524319,124.519516 L456.937939,131.09883 C458.332127,133.272808 459.732567,135.44298 461.121644,137.619791 C461.609723,138.385456 462.152811,139.118136 462.612885,139.901793 C463.499029,141.410134 465.240311,143.228339 462.981945,144.557758 C460.868603,145.803213 459.80043,143.537204 458.837274,142.092835 C457.527341,140.130996 456.228812,138.161267 454.940145,136.185041 L453.292933,140.379117 C454.178571,141.531243 455.061505,142.685364 455.936804,143.845069 C456.808945,145.001564 457.761099,146.201039 458.207172,147.537455 C458.296186,147.804338 458.284184,148.104207 458.219174,148.422068 C457.972133,149.636537 456.665922,150.351225 455.486731,149.969392 C455.218687,149.88243 454.970647,149.769479 454.756612,149.610549 C453.924478,148.995818 453.479406,147.874308 452.819299,147.008686 L451.412996,145.16321 C450.466393,147.573625 449.519825,149.984055 448.573192,152.394461 C449.118765,153.375213 449.641947,154.36566 450.093857,155.388023 C450.131863,155.475985 450.161868,155.566945 450.180871,155.662903 C450.48092,157.099275 449.042687,158.295752 447.605454,157.997882 C447.166383,157.906922 446.812325,157.764984 446.659301,157.504098 L446.603872,157.408568 C442.421246,168.057003 438.23601,178.70447 434.039556,189.347677 C432.687075,192.777517 429.984981,194.288577 427.194846,193.745385 L428.990438,188.661477 C425.963947,197.218736 422.961461,205.784991 419.835955,214.306266 C418.237696,218.442458 418.237696,218.442458 420.05299,222.594643 C420.822114,223.776126 421.612242,225.417409 419.841956,226.39898 C417.821628,227.52049 416.811465,225.835226 416.110351,224.38586 C410.873502,213.560592 401.72602,205.765 394.536856,196.45507 C394.446841,196.339121 394.536856,196.45507 393.991767,195.773368 C387.486713,199.874575 379.926488,204.412591 373.141389,208.911625 C372.677314,209.21949 371.354099,210.125094 371.354099,210.125094 C371.265085,210.685849 371.54213,211.927306 371.611141,212.502055 C372.635307,221.048319 373.807497,229.580589 375.18272,238.075875 C375.394754,239.387302 375.875832,241.029584 375.875832,241.029584 C376.176881,241.497379 377.238053,242.647876 377.616114,243.063695 C385.241349,251.440033 393.754729,258.8348 403.731345,264.284418 C420.292028,273.329464 437.107753,267.246123 444.290917,249.620828 C449.222716,237.51512 453.655434,225.238487 456.662921,212.457075 C461.193655,193.198494 466.070446,174.021877 470.760205,154.80128 C471.318296,152.51228 471.879387,150.171304 471.965401,147.837324 C472.035412,145.914165 471.89839,143.691137 471.002245,142.072844 C467.598541,135.935073 463.895565,129.987778 459.524319,124.519516 Z M225.846124,166.205994 C225.478064,168.31907 225.710102,170.69903 224.795954,172.517235 C223.563754,174.967164 224.579919,176.293584 226.145172,177.668983 C227.232348,178.624565 228.499554,179.372238 229.610734,180.302831 C233.04629,183.180573 233.78341,188.556223 232.340176,192.68142 C229.174663,201.723467 222.11852,206.172522 213.685153,208.535489 C209.840531,209.612018 205.45382,208.92232 201.331152,208.703416 C200.378998,208.652438 199.364833,207.357004 198.633715,206.433408 C198.115631,205.780693 198.026616,204.788127 197.746571,203.948494 C198.716728,203.679612 199.673883,203.27179 200.661043,203.18083 C201.732217,203.081873 202.831395,203.277788 203.918571,203.344758 C211.75284,203.795561 218.524938,201.204694 224.52791,196.284845 C230.049805,191.758823 229.543723,185.005775 223.342718,181.553285 C221.760462,180.67167 220.001177,180.067934 218.524938,179.045381 C215.892511,177.220179 215.345422,174.913187 217.404756,172.567213 C219.65212,170.006333 222.432571,167.910249 225.006988,165.641241 C225.837122,164.909561 226.026153,165.174445 225.846124,166.205994 Z M187.067441,198.288469 C189.048762,199.662868 190.841052,201.309148 193.100418,203.177331 C191.800208,204.240866 191.245118,204.998535 190.522001,205.228435 C188.550681,205.854161 184.240983,204.035956 183.625883,202.034831 C183.289829,200.940309 183.952936,199.38399 184.559034,198.246487 C184.750065,197.887644 186.463343,197.869652 187.067441,198.288469 Z M273.633222,197.593831 L273.967269,197.833222 L273.741193,197.87737 C272.434278,198.15405 270.920194,198.873402 269.95387,198.421211 C266.238268,196.682971 262.614681,196.98184 258.858073,197.952416 C258.011935,198.17032 257.062782,198.114345 256.179639,198.014388 C255.701561,197.959413 255.26449,197.546593 254.808416,197.294703 C255.146471,196.736947 255.349504,195.97428 255.841584,195.653421 C260.740324,192.458924 267.457638,193.229149 273.633222,197.593831 Z M244.076778,144.489787 C244.738885,145.372402 245.400992,146.255016 245.962083,147.002689 C244.578859,150.95696 243.995764,154.797281 242.008442,157.667027 C240.177146,160.310871 236.860608,161.943157 234.150169,163.956277 C233.062993,164.764924 231.707774,165.247713 230.718613,166.14632 C228.840309,167.853574 227.834146,165.750493 226.43992,165.377656 C226.585944,164.845889 226.56194,164.122205 226.907996,163.81334 C229.189366,161.775231 231.505741,159.771107 233.896128,157.860942 C237.635734,154.873248 240.735236,151.479732 242.125461,146.731807 C242.378502,145.868185 243.406669,145.231463 244.076778,144.489787 Z M428.01448,143.951723 C427.935467,144.9183 428.884621,146.939417 429.43371,147.739067 C429.772765,148.232851 429.994801,148.572703 430.198834,148.879568 C431.387697,150.681398 432.793272,152.322608 434.343627,153.82179 L435.771748,150.171396 C435.32541,149.711578 434.905099,149.210337 434.473526,148.743628 C433.147311,147.308255 431.462038,145.05524 430.162828,143.593879 C428.979636,142.763242 428.072489,143.252029 428.01448,143.951723 Z M231.234597,139.158418 C231.709674,139.587231 232.161747,139.995052 232.61282,140.403874 C232.273765,140.775711 231.983718,141.406435 231.587654,141.480403 C228.100089,142.140115 224.595521,142.713864 221.087953,143.265623 C220.845914,143.303606 220.544865,142.967753 220.27082,142.805824 C220.564868,142.393004 220.786904,141.707304 221.162965,141.608347 C224.420493,140.751722 227.705025,140.00105 231.234597,139.158418 Z M438.525983,131.419501 C437.730107,132.891654 439.295263,135.75362 440.554704,137.941064 L442.350581,133.345179 C442.081058,132.99668 441.811359,132.648322 441.541471,132.300116 C441.088398,131.715372 440.464297,131.259571 439.338114,131.041666 C439.020063,130.980693 438.679008,131.134626 438.525983,131.419501 Z M441.583478,122.538384 C441.732995,125.065696 443.192444,127.021282 444.265852,128.44642 L445.838468,124.419062 L445.838468,124.419062 L446.273462,123.309001 C446.24284,123.277375 446.211765,123.245408 446.180223,123.213089 C444.505952,121.496839 441.456458,120.408315 441.583478,122.538384 Z M474.55512,16.2373546 C469.025224,18.297454 464.096426,21.3601151 459.920749,25.5003052 C447.486735,37.8309147 441.7178,52.8733387 442.434916,70.2867262 C442.76897,78.3961811 445.148356,86.006854 449.98914,92.690932 C457.588371,103.185344 469.92337,107.35752 482.306376,103.625152 C488.879441,101.644018 494.575364,98.1655387 499.409147,93.3276536 C510.604961,82.1235516 516.144859,68.5045054 516.565927,54.4996278 C516.651941,43.6283803 514.781638,35.0511299 509.592797,27.4054723 C501.618505,15.6536098 487.849274,11.2845198 474.55512,16.2373546 Z" id="Stroke" fill="#000"/>
                        <path d="M437,131.728466 C440.830648,145.387104 445.364506,153.98386 450.601576,157.518734 C455.838645,161.053607 463.025484,161.896501 472.162091,160.047415 L474,141.795541 L453.043002,114.315097 C451.076262,113.894968 449.807291,113.894968 449.23609,114.315097 C448.006933,115.219166 447.933306,118.106139 447.05799,118.606796 C445.572521,119.456445 443.798981,120.181385 441.737367,120.781618 L437,131.728466 Z" id="Fill-2" fill="#FFF"/>
                        <path d="M447.239281,156.981914 C447.391541,157.240949 447.74383,157.382423 448.180707,157.474081 C449.610761,157.770976 451.040814,156.57742 450.743259,155.146746 C450.723356,155.051102 450.695491,154.959444 450.657675,154.87177 C449.63763,152.562368 448.255345,150.417354 447.084035,148.170719 C445.077781,144.322048 443.162087,141.767558 441.361832,137.824239 C440.084039,135.608489 438.312644,132.527958 439.145597,130.981715 C439.298853,130.697772 439.638205,130.544343 439.954667,130.606113 C441.076219,130.823305 441.695212,131.276617 442.147017,131.860443 C445.912725,136.727323 449.640616,141.623096 453.368507,146.520861 C454.025316,147.383649 454.468165,148.501487 455.296143,149.114206 C455.509108,149.272616 455.755909,149.385197 456.021619,149.470878 C457.195914,149.851461 458.496596,149.139113 458.741407,147.929616 C458.805097,147.612796 458.818034,147.312912 458.729465,147.046903 C458.286616,145.715858 457.338223,144.520309 456.470438,143.367601 C452.768422,138.452899 448.927082,133.638822 445.326572,128.649398 C444.278662,127.197803 442.360978,125.044818 442.187819,122.130668 C442.062428,120.006576 445.097684,121.092533 446.762596,122.803164 C448.613605,124.703091 448.849459,125.378576 450.193928,127.499679 C453.196343,132.239034 456.244536,136.952485 459.355424,141.621103 C460.314764,143.059747 461.377601,145.319335 463.480386,144.077956 C465.72747,142.751893 463.994887,140.940637 463.112175,139.436238 C462.655394,138.656142 462.115019,137.925861 461.628383,137.161706 C458.323437,131.972028 454.952811,126.821205 451.757333,121.563779 C450.950254,120.234727 449.946132,118.807042 450.147155,117.1263 C450.276527,116.049309 451.4966,115.484413 452.386278,116.105102 C453.239135,116.700884 454.046215,117.364413 454.770695,118.110635 C461.544789,125.075703 466.787322,133.157614 471.459625,141.601178 C472.352289,143.214172 472.488627,145.428927 472.418965,147.346786 C472.332386,149.673125 471.775093,152.006438 471.21979,154.286947 C469.829544,159.992704 468.423374,165.695472 467.015215,171.396248 C468.481094,171.60248 469.944983,171.803731 471.408872,172 C472.864799,165.972441 474.314756,159.944882 475.772674,153.91832 C476.529995,150.789967 477.238553,147.546044 476.923085,144.309096 C476.55985,140.576991 474.376456,137.746529 472.455786,134.654042 C467.76557,127.097177 462.797702,119.663852 456.287328,113.534672 C454.030292,111.41058 451.456793,110.235953 448.385712,111.558031 C445.246959,112.909997 444.515512,115.502346 445.561431,118.675532 C438.950545,116.184805 438.34847,120.125135 439.556601,125.603737 C439.556601,125.603737 437.536414,125.24308 436.37008,126.276234 C434.880317,127.596319 434.668347,130.683824 435.430644,133.23931 C438.004142,141.880139 442.744117,149.358297 447.239281,156.981914" id="Stroke-2" fill="#000"/>
                        <path d="M508,47.3229894 C508,60.2454711 506.56148,77.9608955 481,96 C502.29697,74.4895331 497.251237,51.4355036 497.251237,39.2245729 C497.251237,27.0136423 508,34.4005076 508,47.3229894 Z" id="Path-4" fill="#FFF"/>
                    </g>
                </g>
                <g id="Face" transform="translate(324.000000, 111.000000) scale(1 1)" fill="#000">
                    <g id="Face/Glasses" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M22.4484,41.6261 C25.2544,42.9281 27.9254,42.9451 30.8474,42.0381 C31.0464,41.9761 31.2714,41.9981 31.5124,42.0721 C32.9654,42.5201 33.4174,44.4161 32.3924,45.5391 C32.2004,45.7481 31.9924,45.9251 31.7564,46.0491 C31.4084,46.2321 31.0424,46.3801 30.6564,46.5071 C28.1374,47.3381 25.3884,47.1601 22.9264,46.1751 C22.2694,45.9121 21.6154,45.6191 20.9894,45.2771 C20.7804,45.1621 20.6224,44.9531 20.4944,44.6921 C19.9164,43.5201 20.5314,42.0391 21.7774,41.6461 C22.0354,41.5641 22.2664,41.5421 22.4484,41.6261 Z M24.2534,8.5388 C30.9174,15.8318 30.0534,29.1978 22.5064,35.5628 C16.9234,40.2718 9.3594,39.7988 4.4884,34.3448 C1.3284,30.8078 -0.0096,26.5578 0.000344386183,21.8438 C0.0074,16.3238 1.7614,11.4878 5.8534,7.7068 C11.4554,2.5298 19.1314,2.9328 24.2534,8.5388 Z M32.3428,3.8702 C32.8578,4.9892 32.9348,6.3182 33.1428,7.5672 C34.3608,14.9012 35.5898,22.2332 36.7628,29.5742 C37.6928,35.3932 35.9348,37.7332 30.1308,38.2172 C29.9308,38.2332 29.7278,38.2002 29.5208,38.1322 C28.1158,37.6752 27.5848,35.8742 28.4948,34.7092 C28.5808,34.5992 28.6768,34.4992 28.7828,34.4122 C29.6188,33.7342 30.7338,33.4002 32.1388,32.8142 C32.4788,32.6722 32.6768,32.3062 32.6188,31.9422 C31.3298,23.7802 30.0808,15.9402 28.8678,8.0952 C28.6748,6.8512 28.4318,5.5452 28.6288,4.3362 C28.7408,3.6542 29.6528,2.7022 30.2978,2.6222 C30.9438,2.5412 32.0508,3.2342 32.3428,3.8702 Z M5.1834,16.3948 C3.7444,20.1978 3.8564,24.4828 5.5954,28.1598 C8.9784,35.3168 16.1494,36.3758 20.8674,31.0358 C23.0404,28.5758 24.0324,25.6718 24.4294,21.4798 C24.5504,20.2098 24.3764,18.9318 23.9594,17.7268 C23.3824,16.0548 22.6134,13.8668 21.3304,12.0398 C17.0324,5.9198 8.4714,7.7028 5.1834,16.3948 Z M60.5992,7.9002 C61.3052,9.3962 61.7392,11.0202 62.0582,12.4372 C62.5262,14.5202 62.6802,16.6682 62.4322,18.7882 C61.9252,23.1352 60.7082,26.4702 58.3022,29.3802 C52.5982,36.2812 43.5012,35.5042 39.6742,27.4532 C36.1602,20.0612 36.6582,12.5652 41.7122,5.9252 C47.2872,-1.3998 56.6812,-0.4008 60.5992,7.9002 Z M45.6762,7.9902 C43.9012,9.9082 42.8972,12.1222 42.3722,14.8562 C41.8042,17.8122 41.9982,20.8862 42.9432,23.7432 C43.1682,24.4222 43.4422,25.1112 43.7862,25.7632 C46.2722,30.4732 50.8372,30.9212 54.4782,27.1102 C58.6362,22.7602 59.4752,13.7892 56.1862,8.8382 C53.4622,4.7352 49.0492,4.3452 45.6762,7.9902 Z M20.0354,19.0249 C20.6484,19.7929 21.3244,20.6719 21.5984,21.6629 C22.3144,24.2529 20.7634,26.5949 18.7654,26.4889 C16.4454,26.3659 15.9344,24.5109 15.6504,22.7139 C15.3794,21.0069 16.0664,19.3699 17.5114,18.5349 C18.3524,18.0489 19.4294,18.2659 20.0354,19.0249 Z M46.3902,15.5461 C47.4632,15.9621 48.5932,17.3901 48.8312,18.5441 C49.2222,20.4311 48.3842,22.4941 46.1912,22.4601 C45.5202,22.4491 44.8612,21.7491 44.3322,21.1131 C43.8162,20.4931 43.546944,18.7343856 43.626944,17.9323856 C43.882944,15.3643856 45.0232,15.0161 46.3902,15.5461 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
            </g>
        </g>
        <g id="Coding" transform="translate(915.000000, 179.000000)">
            <path d="M419.914029,470.442608 L424.181918,472.946708 L426.764789,633.093074 L424.181918,636.010555 L184.198748,618.661641 L190.212192,484.077522 L419.914029,470.442608 Z M437.45767,2.43634201 L444.089299,321.897558 L332.674679,310.641561 L328.971468,425.151143 L326.839914,428.828365 L2.42559465,408.130806 L25.3011718,123.373837 L106.636571,117.573661 L117,42.2386115 L437.45767,2.43634201 Z" id="Path-6" fill="#FFF"/>
            <path d="M21.0622105,406.28844 C29.0983826,312.397304 31.1967164,218.903021 38.9050637,125.092277 C34.7594194,125.278581 31.613832,125.420224 27.549825,125.603976 C19.938422,218.854531 12.3589086,311.733753 4.74495441,405.027694 C10.5463053,405.475591 15.5159762,405.859685 21.0622105,406.28844 M441.398656,318.967731 C439.397267,213.997851 437.413736,110.03861 435.419999,5.44389143 C329.138435,18.5209429 223.887541,31.4716645 118.103453,44.4874651 C116.912059,56.2999422 115.499988,67.7653312 114.664482,79.2728302 C113.83918,90.643791 111.554611,101.954777 112.052088,113.689414 C145.469808,111.097739 178.034163,108.517548 210.606172,106.06241 C252.200378,102.927134 293.800962,99.8901136 335.400271,96.8173638 C341.070237,96.3988165 341.739918,97.0317417 341.890437,102.836534 C341.914673,103.762952 341.894263,104.691923 341.864925,105.619618 C340.821498,139.923909 339.762764,174.2282 338.728267,208.532491 C337.939956,234.724876 337.142717,260.918536 336.436044,287.113473 C336.257463,293.779605 336.409257,300.453393 336.409257,307.566145 C371.129347,311.335623 405.641517,315.084684 441.398656,318.967731 M325.888249,425.151143 C327.853923,416.04391 336.342927,108.314654 334.898967,102.878644 C333.081261,102.878644 331.232942,102.757418 329.405031,102.897784 C307.226472,104.601323 285.051739,106.372493 262.871904,108.067099 C225.904237,110.891017 188.934019,113.704727 151.959974,116.458462 C117.759046,119.005476 83.5491887,121.458061 49.3469851,123.996142 C47.8111833,124.109711 46.6478518,124.095675 44.2102129,124.444039 C38.647396,218.250955 33.0003906,312.5734 27.4222667,406.632976 C127.193257,412.823138 226.59943,418.99033 325.888249,425.151143 M0.0763210993,410.075783 C-1.47351209,402.436019 21.0673128,127.215639 23.4641331,120.272602 C50.6812448,118.238564 77.8843251,116.204526 105.535135,114.137311 C106.675506,102.719136 107.815877,91.7003677 108.869509,80.6726668 C109.949927,69.3655091 110.901512,58.0455907 111.994687,46.738433 C112.660541,39.8732362 113.090412,39.4700016 119.798703,38.6431154 C161.418421,33.5146347 203.044517,28.4295401 244.661683,23.2793664 C287.888636,17.9301276 331.105383,12.5145337 374.328509,7.14615398 C393.411229,4.77651263 412.495224,2.4158037 431.591975,0.153351295 C432.742551,0.0180890538 433.905883,-0.00998424167 435.121513,0 C438.133164,0.0359538781 440.646063,2.34306834 440.981541,5.33670248 C441.105272,6.43283706 441.195839,7.50600258 441.216248,8.58044416 C441.874449,44.0574332 442.442083,79.5369744 443.090079,115.013964 C444.143711,172.747972 445.231783,230.483256 446.324957,288.217264 C446.51757,298.419355 446.916828,308.617617 447,318.820984 C447.040559,324.172775 446.247147,324.944791 440.8374,324.420331 C424.465294,322.834189 408.113597,321.037498 391.756798,319.29185 C373.375649,317.330547 354.997051,315.333515 336.053369,313.290545 C335.68345,321.347581 335.22424,328.940131 335.006116,336.539062 C334.196121,364.816527 333.466487,393.09782 332.704964,421.377837 C332.693484,421.841046 332.689657,422.305531 332.666697,422.768741 C332.287849,430.834709 331.423004,431.394899 323.560311,430.848746 C303.681627,429.46805 283.787637,428.310664 263.9013,427.044814 C230.141723,424.895931 196.385974,422.694729 162.625122,420.58668 C143.430151,419.388461 124.219873,418.398239 105.023626,417.19364 C71.4923794,415.089418 37.9662347,412.897149 4.4400901,410.720193 C3.07521643,410.630869 1.72564975,410.324615 0.0763210993,410.075783 M426.963981,474.665031 C427.211219,480.721147 427.448262,486.420721 427.542569,492.124115 C428.142822,528.215051 428.638572,564.308535 429.223532,600.399471 C429.39303,610.807943 429.75624,621.211321 429.988185,631.618519 C430.136018,638.241282 428.968647,639.363115 422.376062,638.912344 C382.196082,636.156786 342.017377,633.382127 301.838672,630.613835 C268.355781,628.307775 234.872889,626.005534 201.391272,623.685466 C196.543371,623.349298 191.696744,622.977476 186.857763,622.529252 C181.642828,622.0441 180.870528,621.406145 181.015812,616.0924 C181.302557,605.687749 181.878596,595.292011 182.360328,584.893726 C183.859048,552.543931 185.346298,520.191589 186.888349,487.844341 C187.26048,480.044991 187.659375,479.750844 195.307181,479.339547 C230.209779,477.460063 265.111102,475.583126 300.012425,473.681994 C333.757848,471.844531 367.500721,469.985421 401.244869,468.113577 C407.021825,467.793962 412.791133,467.330458 418.568089,467.014664 L418.636907,467.010844 L418.636907,467.010844 C423.050231,466.780365 426.785562,470.252827 426.963981,474.665031 Z M411.957661,473.021119 C400.408848,473.664167 388.867682,474.456199 377.317595,475.0776 C355.368223,476.256735 333.408656,477.300893 311.461833,478.506768 C282.354033,480.106112 253.252605,481.804779 224.147353,483.462698 C213.820711,484.050992 203.494069,484.643106 192.660207,485.26196 C190.633876,529.226102 188.636856,572.563749 186.597781,616.776196 C266.400817,622.258025 344.972762,627.653266 424.181918,633.093074 C423.172576,579.013334 422.396453,526.017226 421.255845,472.998198 C417.483555,472.998198 414.712962,472.868315 411.957661,473.021119 Z M327.359928,507.113243 C333.159213,512.505403 338.919981,517.937193 344.69744,523.349806 C353.024684,531.151692 361.376322,538.92801 369.665049,546.769525 C370.23766,547.310275 370.785876,547.874035 371.336661,548.463363 C373.575747,550.862862 373.564192,554.591864 371.241654,556.913382 C370.859058,557.295614 370.47261,557.661228 370.073323,558.012779 C355.198293,571.076431 340.283462,584.09534 325.369914,597.114249 C324.319701,598.032118 323.313139,599.232506 322.067776,599.636471 C320.77876,600.053219 318.659075,600.234747 317.904154,599.498407 C316.400731,598.034674 316.850088,596.082605 318.675765,594.760772 C319.426835,594.217465 320.339674,593.873584 321.030401,593.270193 C336.288028,579.952146 351.522545,566.60981 367.324537,552.780415 C363.409987,548.877555 359.856208,546.29525 355.683599,542.4627 C345.566627,533.171515 338.810851,525.468064 326.362353,514.31046 C325.231255,513.297992 324.288888,512.144903 323.171912,511.147775 C321.528546,509.680208 319.877477,508.028555 322.007433,505.938422 C324.132254,503.853402 325.833395,505.692975 327.359928,507.113243 Z M273.926438,503.640871 C274.525201,504.241939 274.76496,505.914254 274.34887,506.666541 C273.478633,508.24355 272.128878,509.575302 270.881877,510.921033 C257.714151,525.128088 244.524861,539.314811 231.354598,553.520595 C229.805678,555.19291 228.345557,556.946553 226.005051,559.620224 C228.808584,561.546691 230.743149,563.070328 232.852776,564.292795 C248.401601,573.296103 263.982141,582.239685 279.557606,591.197246 C280.5547,591.771628 281.569554,592.316782 282.584407,592.861937 C284.388309,593.832795 285.845893,595.323392 284.432709,597.196486 C283.790814,598.047893 281.605074,598.105077 280.20965,597.901756 C278.92586,597.713684 277.746093,596.721223 276.548566,596.033744 C260.364188,586.75087 244.183616,577.460371 228.00685,568.163519 C226.810591,567.476039 225.530607,566.859723 224.500531,565.96511 C220.215311,562.240524 219.80937,557.609888 223.587162,553.51297 C238.439545,537.407406 253.36043,521.36665 268.272435,505.318269 C269.048797,504.484653 269.870829,503.462965 270.869191,503.138922 C271.782559,502.842836 273.31372,503.024554 273.926438,503.640871 Z M142.45467,385.025057 L142.493997,385.025057 C145.46502,385.057841 145.955961,389.752792 143.091499,390.60108 C142.147673,390.879744 141.141686,390.924822 140.161071,390.933018 C124.946947,391.038201 109.732823,390.994489 94.5186992,390.968535 C93.6141991,390.967169 92.5257543,390.978097 91.8521364,390.474042 C91.0072596,389.842948 90.0025414,388.684578 90,387.751598 C89.997467,386.925167 91.1429981,385.76543 92.0246637,385.35563 C93.0192333,384.893921 94.2840113,385.034619 95.4308109,385.033253 C111.105431,385.005933 126.780051,384.978613 142.45467,385.025057 Z M136.341086,346.462617 C138.014885,346.758472 138.578772,348.748221 137.300459,349.818878 C135.802714,351.075051 133.950308,351.232679 132.103004,351.335743 C115.225949,352.283935 98.3488939,353.23819 81.4667355,354.100293 C73.6054902,354.500426 65.732763,354.70898 57.8651388,354.976947 C56.4783856,355.024235 55.082702,354.998772 53.6985002,354.900558 C51.6942781,354.759905 49.8661112,354.109994 50.0077208,351.849852 C50.162088,349.407833 52.3028166,349.563036 54.1743594,349.487859 C73.1474917,348.728821 92.1218996,347.989182 111.091205,347.16588 C118.030074,346.865174 124.956186,346.244363 131.897607,346.006709 C133.344321,345.956995 134.802517,346.191012 136.341086,346.462617 Z M291.036336,343.091941 C291.047779,343.095939 291.060494,343.099937 291.071938,343.103935 C293.129191,343.836924 293.681012,346.791533 292.03699,348.280166 C291.89967,348.406773 291.75345,348.520053 291.59833,348.618673 C290.522659,349.302351 288.92441,349.162417 287.549941,349.241047 C265.432564,350.489792 243.312643,351.715882 221.193995,352.947302 L221.173651,352.631451 L221.173651,352.631451 C206.177828,353.067246 191.183276,353.519033 176.187453,353.924176 C173.419443,353.998807 170.622189,354.108089 167.893595,353.746926 C167.655828,353.716274 167.426962,353.622984 167.20191,353.488381 C165.74861,352.614126 165.612562,350.448478 166.79758,349.207729 C167.166309,348.822577 167.557925,348.480072 167.995313,348.224192 C168.891706,347.701771 170.250917,348.044277 171.402877,348.005628 C203.002945,346.956788 234.604284,345.958592 266.201808,344.824459 C273.576387,344.55925 280.936978,343.834258 288.303928,343.311837 C289.224479,343.246535 290.236576,342.814738 291.036336,343.091941 Z M204.374369,296.65349 C204.498062,296.665506 204.589876,296.750951 204.65236,296.864434 C204.656185,296.869775 204.658736,296.87645 204.662561,296.88179 C205.270825,297.947192 205.056594,299.31566 204.021142,299.921791 C203.170593,300.419779 202.276687,300.847008 201.585536,301.085989 C199.418994,301.83631 197.066275,302.179428 194.770939,302.320948 C147.419438,305.24079 100.047535,307.46238 52.5940188,306.870935 C50.0640487,306.838893 46.0127812,308.000421 46,303.916381 C45.9872774,299.613385 50.1405599,301.174105 52.5621391,301.122037 C80.5499328,300.531927 108.550478,300.407764 136.52807,299.494562 C156.630111,298.839033 176.698996,297.140798 196.788284,296.023327 C199.299127,295.883143 201.844399,296.410503 204.374369,296.65349 Z M272.181049,294.000328 C275.851051,293.951818 275.98122,299.29759 272.315009,299.468587 C272.066045,299.480715 271.820873,299.489204 271.579492,299.498906 C257.17625,300.104065 242.762897,300.486079 228.350809,300.940858 C227.893322,300.954198 227.335998,301.091238 226.994778,300.895986 C225.588197,300.088299 223.314666,299.361866 223.073285,298.260695 C222.548818,295.867953 224.948727,295.733338 226.786256,295.659361 C233.416018,295.390132 240.048309,295.190029 246.678072,294.941417 C255.139045,294.623678 263.594962,294.210133 272.055935,294.002754 C272.098903,294.001541 272.140608,294.000328 272.181049,294.000328 Z M182.301245,269.000161 C185.830069,268.966231 185.931911,274.297015 182.408179,274.506879 C181.667279,274.550862 180.899645,274.533269 180.16511,274.552119 C167.907157,274.878852 155.646657,275.084946 143.388704,275.426759 C127.433887,275.871619 111.484162,276.449687 95.5306187,276.849307 C92.9654742,276.913397 88.9961837,277.883544 89,273.935095 C89.0025488,270.789656 92.6090274,271.127699 95.130889,271.069893 C123.114515,270.428992 151.096867,269.771755 179.081766,269.202485 C180.118008,269.181121 181.22172,269.010215 182.301245,269.000161 Z M124.99663,244.024487 C125.079628,246.504074 123.620889,247.475291 121.563566,247.756075 C114.997986,248.649975 108.419832,249.508606 101.821557,249.995571 C100.958889,250.060681 100.049693,249.397377 99.1379814,248.700163 C98.2627385,248.028721 97.8364348,246.863533 98.0577606,245.730898 L98.0615332,245.710552 C106.051897,242.255679 114.603121,243.136013 122.807266,242.016944 C123.947848,241.862309 124.955132,242.784693 124.99663,244.024487 Z M127.412594,221.076766 C127.794557,221.230978 128.167548,221.40525 128.534129,221.593314 C130.209382,222.447121 130.547766,224.797912 129.03145,225.901217 C128.822524,226.052921 128.604626,226.161998 128.375192,226.210895 C123.630137,227.202615 118.796641,227.825731 113.972117,228.411235 C108.916878,229.024322 103.834721,229.436807 98.1898747,229.993474 C97.8412375,229.959623 96.768409,230.12888 96.0467812,229.713887 C95.1636524,229.204862 94.0626254,228.240724 94.0023829,227.406977 C93.9447039,226.607081 94.9457542,225.189084 95.7430054,224.975945 C98.1834659,224.321485 100.740566,224.043151 103.268186,223.748519 C110.630583,222.887189 118.000671,222.089801 125.366914,221.267337 C126.056498,221.190858 126.853749,220.849837 127.412594,221.076766 Z M189.640126,196.022412 C191.317061,196.081222 192.482905,197.790452 191.802723,199.300731 C191.178482,200.690888 189.661739,201.235189 187.954292,201.527986 C182.732782,202.426395 177.504916,203.344825 172.242722,203.935423 C145.902514,206.893418 119.567392,209.981544 92.9919805,209.999062 L88.9312302,209.999062 L88.9312302,209.999062 C87.2670091,209.999062 85.806207,208.618914 86.0210682,206.99477 C86.1647328,205.918681 86.9224683,205.280535 88.2497768,205.002754 C90.0347779,204.628625 91.906232,204.569815 93.7433592,204.528523 C124.910953,203.845332 155.843343,200.680878 186.648596,196.121262 C187.62755,195.976115 188.63066,195.987377 189.640126,196.022412 Z M178.789795,168.060918 C179.628237,168.136744 180.892273,169.301489 180.990388,170.087101 C181.097424,170.937352 180.292111,172.385514 179.490621,172.749729 C178.0775,173.391147 176.376404,173.49805 174.777247,173.672078 C155.466303,175.767872 136.168101,177.99916 116.835496,179.863746 C97.4977929,181.729575 78.1244118,183.223729 58.7663214,184.884454 C58.6236078,184.896884 58.4821683,184.914287 58.3063247,184.939148 C56.7199093,185.164142 55.0570402,184.767606 53.8771039,183.709765 C53.5458043,183.411431 53.2782162,183.075805 53.1558902,182.69543 C52.3671423,180.252823 54.7359344,179.817753 56.5708245,179.602704 C62.5405884,178.901619 68.5154492,178.221667 74.5043266,177.702069 C91.7866975,176.199213 109.09073,174.917621 126.361633,173.285487 C141.787452,171.826138 157.175044,170.005059 172.583024,168.370439 C174.646001,168.151661 176.747205,167.875702 178.789795,168.060918 Z M242.720562,133.00731 C244.557842,133.050025 247.373214,132.79244 246.958963,135.545625 C246.77613,136.764948 244.582134,138.392006 243.139927,138.579694 C234.623488,139.691581 226.049515,140.343958 217.513898,141.31864 C176.910904,145.953879 136.313025,150.633128 95.7151456,155.317554 C82.7979291,156.808701 69.8922196,158.426698 56.9660532,159.844064 C55.5698738,159.996803 54.1609089,160.014925 52.6956877,159.991625 C49.2717239,159.937261 49.028799,154.843804 52.4310273,154.455485 C52.5064619,154.447719 52.5793394,154.438658 52.6534955,154.430892 C61.1891123,153.447149 69.7349576,152.550131 78.2629032,151.501669 C102.924903,148.472778 127.565168,145.254904 152.242511,142.36063 C172.309394,140.007415 192.414633,137.997214 212.507086,135.871813 C221.904447,134.877715 231.310759,133.988464 240.715791,133.062969 C241.381917,132.996955 242.051879,132.991777 242.720562,133.00731 Z" fill="#7C6576"/>
        </g>
    </g>
</svg>

```

## File: static\src\img\job_product.svg

```svg
<svg viewBox="53.813 46.528 1309.438 984.256" xmlns="http://www.w3.org/2000/svg" overflow="visible">
    <g id="Master/Spot Illustrations/Super Idea" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <ellipse id="Oval" fill="#C9C9C9" opacity=".5" cx="550" cy="1008.5" rx="394" ry="23.5"/>
        <g id="Background" opacity=".5" transform="translate(36.000000, 63.000000) scale(1 1)">
            <g id="Background/Blob 1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M869.490312,1 C892.755389,1.92225471 915.476128,2.67306463 938.178097,3.75966986 C969.204039,5.24710117 998.516884,13.6857318 1027.26162,24.1202161 C1037.65031,27.8920014 1048.00522,31.8411434 1058.0448,36.3637386 C1111.46613,60.4322218 1153.95818,95.8373436 1182.91939,145.017167 C1196.72302,168.457808 1203.97709,193.894302 1207.15551,220.364195 C1211.99823,260.684461 1207.17053,300.347325 1194.25536,338.780517 C1181.74563,376.012412 1161.11839,409.312902 1133.93534,438.957479 C1119.10311,455.13359 1102.71795,469.583429 1084.53084,482.282167 C1056.54443,501.823326 1028.65312,521.486269 1000.60163,540.943479 C992.90959,546.280732 984.675715,550.915653 976.903588,556.152405 C967.836315,562.262933 958.607617,568.242218 950.137238,575.042073 C912.903355,604.937314 894.836382,642.935391 897.166393,689.27869 C897.984775,705.586045 899.121,721.881576 899.722899,738.196025 C900.311032,754.162856 895.142962,768.633978 886.071935,782.031501 C881.843627,788.278003 876.617995,793.530126 870.611521,798.221801 C851.124267,813.448463 829.09302,824.464677 805.695301,833.153971 C770.656284,846.163675 734.397203,854.918 697.265931,860.621791 C670.549635,864.725824 643.740739,867.970269 616.719113,869.629145 C583.069094,871.693577 549.420325,871.303392 515.807846,869.280344 C473.354588,866.722861 431.187889,861.708396 389.257694,854.931006 C351.62338,848.84649 314.022853,842.549146 277.041743,833.480308 C234.792454,823.121496 192.917319,811.586217 152.869153,794.88986 C129.05974,784.962616 106.556736,772.934286 86.3599665,757.26778 C63.4264977,739.478905 47.2553176,717.39327 38.2080661,690.617141 C33.6781836,677.210159 33.1914089,663.507582 34.9307837,649.805006 C39.2904828,615.461659 50.0132903,583.102343 69.9622923,553.767549 C79.7365775,539.393381 91.5968611,526.686367 104.247997,514.582365 C116.075746,503.265827 128.207572,492.221235 139.714975,480.622109 C146.848914,473.432069 153.392217,465.6603 159.632693,457.750192 C174.762751,438.569659 178.94601,417.075212 174.700184,393.621565 C172.520334,381.58023 170.856041,369.344984 170.448101,357.153486 C169.504584,329.005799 178.127628,302.888255 191.235506,277.901065 C202.254884,256.896122 217.020799,238.458123 234.095453,221.59032 C286.091497,170.219551 345.922235,129.172122 414.627539,99.9957664 C469.984706,76.486548 525.927504,54.2980456 583.469525,35.9262588 C614.35907,26.0640453 646.160847,19.6460984 678.441891,15.7442516 C713.601037,11.4947856 748.8215,7.45105345 784.134562,4.65591226 C812.657806,2.39875298 841.359992,2.13035321 869.490312,1" id="Fill-1" fill="#7C6576" opacity=".15"/>
            </g>
        </g>
        <g id="Decoration" transform="translate(-9.000000, 183.000000) scale(1 1)">
            <g id="Decoration/Scribbles" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <path d="M868.673,577.7326 C865.821,574.6136 865.834,574.6476 862.135,575.9506 C861.127,576.3046 860.024,576.3956 858.96,576.5836 C856.207,577.0696 853.451,576.6306 850.606,577.6746 C847.736,578.7286 844.264,578.7696 841.174,578.3596 C839.248,578.1046 837.862,578.3506 836.12,578.9616 C829.944,581.1286 825.07,586.4206 818.011,586.4676 C817.385,586.4716 816.778,587.1896 816.138,587.5266 C812.427,589.4796 808.363,590.8686 805.513,594.1916 C800.456,594.2776 797.504,598.5416 793.308,600.3516 C789.042,602.1926 785.85,605.2216 783.534,609.2936 C782.862,610.4756 781.614,611.6936 780.365,612.1216 C775.428,613.8086 772.862,617.8646 770.117,621.7716 C768.89,623.5176 767.478,625.2436 766.748,627.2016 C765.769,629.8316 765.433,632.6936 764.705,635.4256 C763.618,639.5126 766.958,643.3366 771.223,642.7806 C774.392,642.3686 777.544,641.7816 780.724,641.5226 C784.263,641.2346 788.027,642.0556 790.671,638.5716 C791.026,638.1046 792.337,638.3706 793.203,638.2816 C794.078,638.1916 795.482,638.3886 795.741,637.9366 C797.999,634.0046 801.922,634.8556 805.382,634.2596 C807.441,633.9056 810.887,633.5946 811.173,632.5626 C811.856,630.0926 813.353,630.8936 814.735,630.4076 C818.099,629.2216 821.625,628.1076 824.545,626.1566 C827.216,624.3716 830.044,622.1526 832.922,621.4296 C833.924,621.1776 835.118,620.7696 835.725,620.0266 C837.792,617.4906 840.208,615.9116 843.586,615.9656 C844.248,615.9766 845.316,615.7236 845.521,615.2806 C847.165,611.7146 851.124,611.5316 853.821,609.5246 C856.148,607.7946 859.637,607.3636 861.489,605.3336 C863.666,602.9496 866,601.5866 868.884,600.5406 C870.315,600.0206 872.287,598.8826 872.554,597.6956 C873.179,594.9196 875.571,594.8546 877.236,593.8346 C880.52,591.8216 884.085,590.2706 887.389,588.2876 C888.77,587.4596 889.782,586.0166 890.963,584.8546 C887.031,584.1126 885.17,579.8396 881.284,580.9596 C880.957,581.0526 880.242,580.5086 880.002,580.0916 C877.226,575.2816 873.179,576.6366 868.673,577.7326 M903.006,449.4846 C904.851,447.9696 906.443,446.8086 907.849,445.4556 C908.705,444.6306 909.915,443.4786 909.869,442.5276 C909.629,437.5806 910.986,433.0776 912.808,428.5836 C913.707,426.3636 912.281,423.8636 910.469,423.7986 C905.857,423.6346 900.66,420.6336 896.672,425.7046 C896.253,426.2366 895.065,426.3046 894.224,426.3366 C892.234,426.4116 890.113,426.7526 888.277,426.2096 C886.097,425.5636 884.767,429.1456 882.599,426.9986 C880.332,428.9436 877.41,427.4676 874.998,428.4176 C869.418,430.6136 864.242,433.8126 858.252,435.0136 C857.285,435.2086 856.135,435.8976 855.623,436.7086 C852.653,441.4046 847.692,442.9976 843.095,445.2956 C839.448,447.1196 835.916,449.2866 832.657,451.7316 C829.969,453.7496 829.607,458.3946 825.151,458.5756 C824.987,463.2406 818.807,463.2466 818.194,467.5626 C812.438,470.3186 811.586,475.4616 811.89,480.4716 C811.027,482.1736 809.04,480.7666 808.674,483.2686 C808.263,486.0836 804.48,488.3266 807.25,491.8066 C807.287,491.8536 807.049,492.0886 806.983,492.2526 C805.018,497.0986 806.119,502.0576 806.534,506.9716 C806.616,507.9516 807.225,509.0646 807.93,509.7826 C811.221,513.1306 814.617,516.3766 818.016,519.6186 C818.369,519.9566 818.995,520.2336 819.465,520.1936 C825.996,519.6326 832.61,522.7516 839.085,519.7466 C840.683,519.0046 842.374,518.4536 843.935,517.6446 C846.612,516.2546 848.964,514.4306 852.444,515.0466 C854.759,515.4586 857.326,515.3406 858.634,512.0806 C859.403,510.1606 861.544,508.6676 863.311,507.2946 C867.408,504.1086 871.833,501.3236 875.78,497.9686 C878.657,495.5236 881.063,492.4966 883.527,489.5996 C885.218,487.6126 886.741,485.4646 888.175,483.2816 C890.433,479.8416 892.65,476.3686 894.695,472.8006 C896.008,470.5106 896.404,467.3486 898.241,465.7896 C901.053,463.3996 901.439,460.8776 900.531,457.9506 C906.738,454.2566 906.277,453.2676 903.006,449.4846 M976.689,425.0606 C979.375,428.3646 982.101,432.2816 985.428,435.5926 C987.415,437.5716 990.039,438.7426 989.776,442.1846 C989.715,442.9636 990.324,444.0076 990.944,444.5896 C994.995,448.3996 995.461,453.7666 997.281,458.5676 C1000.015,465.7796 998.577,473.1976 998.755,480.5366 C998.767,481.0406 998.593,481.6976 998.257,482.0356 C996.119,484.1796 996.875,486.5216 997.323,487.8636 C996.001,490.5116 994.938,492.4006 994.095,494.3816 C992.209,498.8176 990.445,503.3026 988.597,507.7536 C988.411,508.2016 988.048,508.8016 987.657,508.9026 C983.293,510.0306 983.937,513.0776 984.948,516.3836 C982.04,518.0896 981.08,522.1996 976.998,522.3546 C976.174,524.7726 975.336,527.1856 974.537,529.6116 C974.374,530.1056 974.36,530.6466 974.248,531.3446 C971.7,532.9766 968.368,533.2896 966.855,536.9296 C965.72,539.6576 961.241,540.1146 960.475,543.8126 C960.354,544.3966 959.544,544.8676 959.002,545.3306 C955.621,548.2146 952.441,551.1116 950.085,555.1146 C948.503,557.8046 945.375,560.0056 942.443,561.3216 C940.746,562.0826 940.089,562.2676 939.686,564.2096 C939.11,566.9796 938.214,567.2046 935.112,567.5186 C927.166,574.5896 919.918,581.0396 912.152,587.9496 C912.99,593.9406 913.927,600.2646 914.739,606.6056 C915.167,609.9476 915.288,613.3286 915.721,616.6696 C915.986,618.7136 913.746,619.3876 913.672,621.7046 C913.583,624.4816 914.104,628.1556 911.142,630.4086 C910.905,630.5876 910.816,631.0786 910.811,631.4266 C910.7,638.3536 906.819,643.8486 903.905,649.7126 C903.609,650.3106 903.526,651.3426 903.1,651.4976 C898.146,653.2866 896.843,657.9996 894.437,661.8796 C893.575,663.2716 891.695,664.0326 890.838,665.4246 C888.491,669.2336 885.713,672.6966 882.576,675.8126 C879.67,678.7016 876.261,681.0796 873.251,683.8726 C870.244,686.6626 867.512,689.7466 864.543,692.5796 C863.606,693.4726 862.303,693.9926 861.146,694.6446 C860.532,694.9906 859.571,695.0806 859.282,695.5836 C856.372,700.6416 850.823,702.7496 846.946,706.7196 C845.784,707.9106 844.763,709.8326 842.398,708.7016 C842.007,708.5136 841.07,709.0936 840.585,709.5366 C835.587,714.0956 830.332,717.8166 823.632,720.0396 C818.148,721.8586 813.519,726.1696 808.382,729.1576 C805.159,731.0306 801.672,732.4426 798.358,734.1676 C796.63,735.0666 795.031,736.2096 793.358,737.2126 C789.985,739.2326 787.095,741.9226 782.534,741.7536 C780.067,741.6636 777.367,743.3576 775.05,744.7186 C771.718,746.6766 768.136,747.8166 764.534,749.0686 C760.831,750.3576 756.905,751.6016 753.787,753.8496 C748.549,757.6276 741.99,757.6806 736.606,760.8226 C736.057,761.1426 735.028,760.5386 734.229,760.5616 C733.329,760.5866 732.422,760.7856 731.549,761.0226 C731.266,761.0996 731.057,761.7906 730.854,761.7716 C722.433,760.9776 714.326,763.5676 706.027,764.027943 C705.784,764.0416 705.517,763.6246 705,763.1876 C705.909,760.5696 708.409,759.8596 710.587,758.8086 C713.265,757.5186 716.243,756.8606 718.275,754.3046 C718.909,753.5056 720.551,753.5556 721.681,753.0906 C724.8,751.8096 728.255,750.9616 730.914,749.0296 C735.594,745.6286 740.334,742.8146 746.203,742.2166 C748.562,741.9776 751.938,739.6386 753.469,737.5066 C756.774,738.9056 759.342,738.0276 762.4,735.9866 C767.867,732.3366 773.668,728.7446 779.855,726.7806 C783.402,725.6546 786.896,724.7366 789.998,722.6966 C791.675,721.5946 793.009,719.9236 794.754,718.9866 C797.11,717.7226 799.01,716.1246 802.364,716.0216 C805.788,715.9176 809.849,713.0426 812.225,710.2046 C813.93,708.1686 816.179,709.3426 817.104,708.0026 C818.886,705.4226 821.725,704.7036 823.984,703.0296 C825.448,701.9436 826.764,701.2186 828.543,700.6876 C830.702,700.0436 832.682,698.4606 834.465,696.9586 C838.027,693.9576 841.306,690.6196 844.894,687.6536 C846.642,686.2086 848.944,685.4506 850.78,684.0896 C853.937,681.7526 857.139,679.4056 859.923,676.6586 C863.134,673.4906 865.766,669.7396 868.935,666.5246 C870.883,664.5466 873.392,663.1306 875.605,661.4066 C879.818,658.1246 884.321,655.1496 886.046,649.5716 C886.396,648.4396 888.152,647.8126 889.021,646.7556 C891.159,644.1536 892.582,638.5146 892.898,635.1226 C893.036,633.6346 894.646,632.2856 896.164,630.0076 C896.222,629.7286 893.092,628.2656 896.348,626.8406 C899,625.6786 900.758,620.5646 900.301,617.6666 C900.085,616.2906 899.719,614.6536 900.234,613.4976 C902.543,608.3166 900.083,603.3226 899.941,598.2546 C899.929,597.8366 899.318,597.4366 898.806,596.8076 C896.005,598.7316 892.886,600.2456 890.593,602.5736 C885.195,608.0566 878.244,610.9396 871.848,614.7806 C869.242,616.3446 866.438,618.0696 864.685,620.4336 C862.511,623.3656 858.092,622.9586 856.695,626.5756 C851.372,626.8586 847.652,630.6106 843.231,632.8336 C838.627,635.1466 833.79,636.9476 829.628,640.2116 C827.188,642.1266 824.091,643.7566 821.083,644.2456 C817.732,644.7906 815.196,646.7366 812.235,647.6866 C806.152,649.6386 800.225,652.4976 793.592,652.3496 C793.061,652.3376 792.486,652.4446 791.997,652.6496 C781.969,656.8656 781.971,656.8706 771.317,656.0156 C766.056,658.6276 761.996,652.6686 756.842,653.9366 C756.194,654.0956 755.142,652.7746 754.334,652.0736 C753.344,651.2126 752.409,650.2896 751.461,649.3816 C751.086,649.0236 750.402,648.6096 750.43,648.2666 C750.836,643.2406 747.751,638.3076 750.027,633.2406 C750.246,632.7496 750.531,632.1896 750.483,631.6916 C750.167,628.4866 751.145,625.8266 752.973,623.1266 C755.919,618.7726 758.366,614.0826 761.038,609.5416 C762.355,607.3046 765.657,606.5946 765.759,603.4186 C770.668,603.6096 770.57,596.8796 775.153,596.6216 C776.522,591.9216 781.644,591.4136 784.566,588.3306 C786.696,586.0816 790.297,585.2846 793.085,583.5816 C795.48,582.1186 797.65,580.2906 800.592,578.1296 C801.007,578.1836 802.392,578.3656 803.776,578.5466 C803.663,578.2716 803.551,577.9966 803.439,577.7216 C805.569,576.3196 807.543,574.5036 809.87,573.6186 C812.898,572.4666 815.832,571.2426 818.768,569.8586 C821.769,568.4436 825.409,568.4416 828.714,567.6026 C832.601,566.6166 836.382,565.1866 840.292,564.3296 C845.16,563.2606 849.956,562.0016 855.153,562.4566 C857.286,562.6436 859.807,561.6706 862.231,561.9196 C863.288,562.0286 864.366,562.0096 865.408,562.1966 C872.239,563.4196 879.239,563.0446 886.07,564.8686 C888.552,565.5316 889.13,566.6286 889.799,568.4806 C895.391,566.1706 897.287,573.5036 904.475,572.1626 C907.687,570.6016 912.768,567.5486 916.845,563.0136 C917.081,562.7506 917.412,562.5576 917.726,562.3786 C920.345,560.8766 923.484,559.8646 925.449,557.7556 C927.388,555.6776 928.753,553.2196 931.598,552.0326 C932.632,551.5996 933.437,550.3556 934.087,549.3186 C935.415,547.2026 936.746,545.6416 939.39,544.5596 C942.065,543.4656 944.029,540.4716 946.106,538.1326 C947.843,536.1756 949.302,533.9706 950.852,531.8516 C951.046,531.5866 951.03,530.8826 951.132,530.8786 C955.304,530.7096 955.248,526.0356 958.117,524.2646 C959.619,523.3366 959.944,520.5016 960.986,518.0976 C964.111,519.1286 965.239,516.2626 966.702,513.8216 C967.709,512.1406 968.884,510.5536 969.801,508.8286 C970.434,507.6356 970.71,506.2606 971.226,505.0006 C971.609,504.0626 971.92,502.9736 972.606,502.3026 C976.212,498.7746 979.805,495.4306 978.807,489.5146 C978.564,488.0766 979.919,486.1856 980.905,484.7536 C985.127,478.6266 986.33,472.2706 983.071,465.3046 C982.501,464.0846 981.992,462.3566 982.421,461.2316 C984.064,456.9246 980.788,453.7946 979.715,450.3306 C978.826,447.4616 975.321,445.4096 972.996,442.9766 C972.25,442.1966 971.249,441.4636 970.924,440.5176 C970.095,438.1036 968.01,437.5866 966.099,436.5846 C964.622,435.8106 963.558,434.2736 962.105,433.4306 C958.118,431.1186 954.16,428.6366 949.895,426.9916 C948.446,426.4326 946.46,426.8116 945,425.5636 C944.74,425.3406 944.243,425.0716 944.025,425.1756 C939.233,427.4566 935.32,423.4916 930.882,423.3696 C930.535,423.3596 930.185,423.4896 929.319,423.6496 C928.033,428.3876 926.74,433.3706 925.314,438.3136 C924.488,441.1696 923.331,443.9316 922.535,446.7956 C921.819,449.3756 921.797,452.2116 920.747,454.6206 C918.878,458.9106 916.474,462.9666 914.27,467.1096 C913.844,467.9096 912.832,468.6976 912.902,469.4116 C913.308,473.5646 909.864,475.4236 907.972,478.2026 C907.484,478.9196 906.752,479.7986 906.835,480.5216 C907.318,484.7006 904.096,487.0876 902.239,490.0316 C900.363,493.0056 898.501,496.0646 895.201,498.0366 C893.328,499.1546 892.076,501.3886 890.688,503.2286 C888.58,506.0206 887.009,509.2776 883.384,510.6146 C882.611,510.8996 882.101,511.8466 881.419,512.4286 C877.934,515.4056 875.332,519.6376 870.236,520.3376 C866.03,525.6746 858.23,526.1446 854.39,532.0316 C853.951,532.7046 852.877,533.3366 852.097,533.3376 C850.286,533.3396 848.435,532.5996 846.674,532.8036 C844.837,533.0166 843.127,534.3206 841.289,534.5426 C838.642,534.8596 835.777,534.0746 833.275,534.7536 C828.487,536.0526 824.185,535.9626 819.444,534.1076 C815.022,532.3776 810.502,531.0866 806.146,528.9556 C802.044,526.9486 801.069,522.9676 798.281,520.2576 C796.887,518.9016 795.71,517.2826 794.627,515.6546 C793.35,513.7346 791.417,512.3326 792.486,509.1346 C793.172,507.0886 791.562,504.2966 791.058,501.8236 C790.637,499.7546 790.608,497.8086 790.969,495.5716 C791.966,489.4066 792.488,483.2016 795.222,477.4426 C795.535,476.7826 795.022,475.7716 795.085,474.9386 C795.153,474.0486 795.444,472.4136 795.65,472.4116 C798.745,472.3856 798.76,470.0786 799.277,467.9916 C799.472,467.2066 800.108,466.1686 800.784,465.9276 C803.518,464.9576 804.263,462.5576 805.529,460.3656 C807.092,457.6606 807.898,454.1936 811.536,453.1226 C811.958,452.9986 812.44,452.3806 812.527,451.9226 C813.24,448.1706 816.512,447.1116 819.154,445.2796 C821.355,443.7526 823.671,442.0596 825.138,439.8986 C826.688,437.6156 827.517,434.9656 830.932,435.0616 C831.231,435.0706 831.711,434.8046 831.819,434.5456 C833.846,429.6436 838.942,429.1616 842.824,427.0426 C848.116,424.1546 853.621,421.6476 859.103,419.1256 C860.862,418.3156 862.86,418.0396 864.708,417.4036 C866.114,416.9196 867.768,417.4246 868.866,415.4996 C869.395,414.5716 871.773,414.7036 873.311,414.3456 C876.424,413.6216 879.553,414.5446 882.736,411.9476 C885.144,409.9836 889.712,410.6626 893.319,410.1756 C896.855,409.6986 900.406,409.3186 903.925,408.7416 C907.443,408.1656 910.925,406.9726 914.42,408.7546 C916.547,407.6826 917.13,406.0256 916.98,403.7986 C916.63,398.5996 916.503,393.3866 916.16,388.1876 C915.955,385.0836 914.806,382.0226 916.08,378.8726 C916.301,378.3286 916.034,377.2376 915.594,376.8326 C912.896,374.3486 914.978,371.2456 914.854,368.6016 C914.762,366.6676 915.099,365.2836 913.772,363.4616 C912.503,361.7176 911.824,359.2336 911.699,357.0286 C911.391,351.5856 909.925,346.6196 907.23,341.8996 C906.27,340.2196 905.942,338.1816 905.305,336.3146 C903.731,331.6936 901.412,327.4796 898.785,323.3566 C895.846,318.7436 897.129,313.5576 901.531,310.6076 C902.089,310.2346 903.086,309.8656 903.555,310.1126 C907.96,312.4276 913.344,313.0156 914.486,319.8686 C915.305,324.7736 918.625,329.2636 920.841,333.9346 C921.072,334.4206 921.73,335.0226 921.603,335.3146 C920.01,338.9666 922.446,341.9366 923.666,344.7816 C926.137,350.5486 926.128,357.0816 929.528,362.5086 C930.058,363.3536 930.181,364.5506 930.209,365.5926 C930.333,370.2236 930.241,374.8616 930.45,379.4866 C930.601,382.8246 930.86,386.1926 931.503,389.4626 C932.411,394.0806 932.83,398.4846 931.062,403.0916 C929.657,406.7516 932.571,410.5286 936.651,410.5046 C940.706,410.4816 944.583,410.6356 948.058,413.0116 C948.328,413.1966 948.766,413.2216 949.114,413.1916 C951.784,412.9616 954.069,413.5826 956.361,415.1906 C958.609,416.7666 961.343,417.6556 963.886,418.7956 C964.996,419.2936 966.132,419.8736 967.311,420.0466 C969.34,420.3456 970.984,420.7166 972.254,422.7606 C972.981,423.9296 975.048,424.2646 976.689,425.0606" id="Fill-1" fill="#000" transform="translate(851.988274, 537.030200) rotate(17.000000) translate(-851.988274, -537.030200)"/>
                <path d="M174.70118,235.13317 C176.872731,235.878581 178.801372,237.611067 180.565045,239.208161 C184.468129,242.744924 188.242506,246.427178 191.964195,250.155529 C192.813896,251.00656 193.125418,252.371773 193.884464,253.344799 C194.743065,254.444831 195.653442,255.732385 196.845665,256.303266 C200.705558,258.151126 203.405612,260.985447 205.378406,264.726229 C206.20344,266.291126 207.429886,267.668176 208.60318,269.019074 C210.102195,270.748162 212.104347,272.145331 213.234807,274.07036 C216.499406,279.629917 220.861031,284.847612 220.31842,291.810021 C219.728475,292.177043 219.343393,292.593745 219.13845,292.518569 C213.157419,290.341458 206.717893,289.022581 202.595814,283.395427 C201.903299,282.449962 200.547656,282.011424 199.564257,281.252517 C198.780244,280.647799 197.951032,280.013806 197.400964,279.21459 C195.838871,276.946182 194.606149,274.418885 192.827505,272.350843 C192.170782,271.587106 190.355858,271.819122 188.585338,271.514618 C187.150424,268.251654 186.007922,264.327093 181.36213,262.748183 C179.179058,262.006417 177.548293,259.440558 175.885485,257.513121 C175.491938,257.055985 175.743179,256.041979 175.800984,255.91982 L175.806178,255.910004 L165.091178,244.009429 C165.526168,242.072977 166.111525,239.469607 166.767946,236.550619 C169.11861,234.202622 172.090085,234.235853 174.70118,235.13317 Z M221.614009,197.718242 C222.560354,198.305976 223.506026,199.541566 223.681607,200.609343 C224.432312,205.173299 227.269006,208.440336 229.692372,212.122595 C231.122806,214.295593 232.39286,216.569939 234.171925,218.521878 C234.524779,218.909279 235.01275,219.569289 234.896133,219.929962 C233.56292,224.075982 236.2141,226.734972 238.185332,229.802999 C239.267572,231.48617 239.434278,233.750132 240.057974,235.738528 C240.625883,237.548397 240.214345,240.320901 244.222463,238.556839 C242.780135,244.04178 244.495587,247.720926 246.925148,251.514677 C248.871027,254.553053 250.142535,258.015628 250.291562,261.995043 C250.464208,266.564108 250.803179,268.68911 248.858303,273.14366 C243.89839,271.208412 238.880977,269.74881 236.360098,264.103607 C234.86005,260.745888 232.599391,257.728543 230.312248,253.946504 C230.513374,253.567602 231.163185,252.341988 231.815448,251.11013 C228.526168,248.534532 227.744496,244.764124 226.604693,241.274463 C225.440642,237.70706 224.672678,234.080402 221.882384,231.225065 C220.814472,230.133182 220.164228,228.421669 219.815131,226.881716 C218.985913,223.2214 217.466319,220.209614 214.719379,217.445767 C213.065475,215.780786 212.971417,212.644031 211.918312,210.283341 C210.842301,207.873301 209.450869,205.604632 208.079219,203.05306 C208.425724,202.30508 209.344783,201.197664 209.390538,200.054393 C209.550205,196.089032 212.833138,195.20534 215.378187,194.917415 C217.342664,194.696069 219.652036,196.500198 221.614009,197.718242 Z M261.426019,170.207004 C265.069511,170.928301 267.037682,173.271097 268.38521,176.64773 C270.070085,180.869785 272.420289,184.786124 273.330658,189.370485 C273.621615,190.844372 275.531098,191.996646 276.701407,193.296407 C276.484086,193.376359 276.266691,193.457724 276.049295,193.539089 C276.828199,195.565358 277.594023,197.598022 278.390245,199.618118 C278.976515,201.10465 280.471161,202.903609 280.057836,203.994337 C278.524727,208.036415 281.73917,210.999657 281.924458,214.597903 C281.992989,215.924891 282.894475,217.341557 283.744535,218.469465 C286.266981,221.814915 287.000871,223.970527 286.605258,227.493012 C286.505928,228.375027 286.444617,229.423309 286.818363,230.168675 C289.147841,234.80475 286.071944,238.479911 284.954158,243.054987 C281.588203,240.956156 278.936097,238.814722 276.686642,236.04296 C275.349162,234.393146 273.548768,233.118426 271.209965,230.997661 C271.237479,230.05382 272.533837,227.624371 270.137242,225.998355 C271.020386,218.834289 266.672962,212.967323 264.625122,206.686276 C262.458944,200.042155 262.502332,192.958701 259.532706,186.55358 C258.417673,184.145036 256.94821,181.946625 256.77497,178.712963 C256.537161,174.252359 257.981923,172.01762 261.426019,170.207004 Z" id="Combined-Shape" fill="#3AADAA" transform="translate(226.379901, 231.367224) rotate(-41.000000) translate(-226.379901, -231.367224)"/>
                <path d="M718.267557,507.653363 C721.630557,508.424363 723.380557,510.701363 724.321557,513.853363 C721.598557,517.977363 721.077557,522.685363 721.041557,527.535363 C721.005557,532.279363 719.677557,534.184363 714.171557,536.696363 C713.549557,534.493363 712.609557,532.693363 712.630557,530.904363 C712.667557,527.690363 713.016557,524.423363 713.724557,521.289363 C714.523557,517.753363 715.345557,514.339363 714.778557,510.636363 C714.359557,507.901363 715.665557,507.056363 718.267557,507.653363 Z M748.009457,502.559663 C748.767457,504.057663 749.595457,505.693663 750.762457,508.001663 C750.535457,510.429663 750.528457,514.372663 749.681457,518.126663 C749.168457,520.396663 747.734457,522.778663 746.025457,524.371663 C744.104457,526.162663 739.555457,522.592663 740.365457,519.364663 C741.780457,513.728663 740.104457,506.690663 748.009457,502.559663 Z M795.462857,497.103263 C797.714857,502.535263 795.201857,515.489263 786.825857,516.698263 C783.927857,510.840263 788.364857,505.074263 787.301857,499.506263 C788.498857,498.220263 789.483857,496.898263 790.723857,495.891263 C792.679857,494.303263 794.489857,494.759263 795.462857,497.103263 Z M656.739557,489.315663 C656.192557,494.902663 657.824557,500.696663 655.115557,506.099663 C653.791557,508.743663 652.813557,511.559663 651.459557,514.827663 C646.248557,512.017663 647.023557,507.477663 647.050557,503.571663 C647.074557,499.883663 648.154557,496.210663 648.655557,492.515663 C648.837557,491.173663 648.685557,489.785663 648.685557,487.772663 C649.105557,487.460663 649.998557,486.533663 651.088557,486.038663 C654.129557,484.656663 657.052557,486.124663 656.739557,489.315663 Z M701.147057,461.891163 C707.224057,462.032163 708.736057,463.431163 708.278057,468.894163 C707.806057,474.526163 707.672057,480.358163 702.861057,484.953163 C702.186057,484.453163 701.209057,484.104163 700.919057,483.452163 C699.144057,479.438163 699.281057,471.335163 701.147057,461.891163 Z M595.914657,450.601163 C598.572657,455.095163 598.885657,459.880163 597.528657,464.866163 C596.216657,469.686163 598.518657,475.109163 595.010657,479.548163 C590.858657,478.198163 587.374657,472.658163 588.984657,468.515163 C589.978657,465.958163 589.512657,464.108163 588.831657,461.876163 C586.882657,455.493163 588.746657,452.479163 595.914657,450.601163 Z M823.928357,454.554663 C827.955357,455.902663 827.663357,459.342663 827.208357,462.571663 C826.721357,466.025663 825.772357,469.413663 824.929357,473.264663 C823.465357,473.889663 821.516357,474.721663 819.709357,475.493663 C815.054357,471.647663 817.497357,467.171663 817.757357,463.013663 C818.010357,458.963663 819.527357,455.745663 823.928357,454.554663 Z M749.145457,440.630063 L749.499361,440.92421 C752.662281,443.563321 755.169493,446.00992 752.497457,450.623063 C751.626457,452.127063 751.326457,454.613063 751.934457,456.203063 C753.944457,461.458063 749.451457,463.815063 747.548457,468.582063 C745.870457,464.003063 742.863457,461.184063 743.919457,456.827063 C744.246457,455.473063 744.142457,453.891063 743.805457,452.520063 C742.496457,447.196063 744.925457,443.697063 749.145457,440.630063 Z M856.744957,447.414463 C858.283957,447.741463 860.930957,451.187463 860.307957,452.987463 C858.550957,458.068463 858.833957,464.201463 852.901957,467.856463 C849.619957,464.248463 848.924957,460.536463 850.677957,456.305463 C851.283957,454.841463 851.762957,453.313463 852.150957,451.776463 C852.814957,449.142463 854.958957,447.036463 856.744957,447.414463 Z M787.943957,435.349163 C789.776957,438.942163 790.828957,442.101163 789.260957,445.934163 C788.211957,448.501163 787.956957,451.495163 787.841957,454.317163 C787.680957,458.275163 784.728957,460.029163 782.484957,463.218163 C779.357957,457.180163 777.686957,451.999163 780.833957,446.259163 C781.492957,445.056163 781.599957,443.467163 781.670957,442.041163 C781.858957,438.209163 783.805957,436.081163 787.943957,435.349163 Z M661.761457,432.322063 C662.299457,433.394063 662.382457,434.693063 662.759457,436.260063 C660.941457,438.111063 659.804457,440.450063 660.738457,443.598063 C661.956457,447.701063 658.937457,450.292063 656.621457,452.921063 C656.047457,453.574063 654.373457,453.259063 652.008457,453.522063 C653.440457,447.316063 651.018457,441.759063 654.127457,436.620063 C654.512457,435.985063 654.186457,434.915063 654.174457,434.046063 C654.144457,431.733063 654.771457,429.661063 657.273457,429.154063 C659.633457,428.676063 660.863457,430.533063 661.761457,432.322063 Z M610.891557,413.925963 C612.244557,416.086963 613.522557,418.125963 614.724557,420.042963 C614.206557,423.023963 613.830557,425.847963 613.187557,428.608963 C612.876557,429.944963 612.292557,431.350963 611.434557,432.399963 C609.333557,434.966963 607.857557,434.624963 606.366557,431.532963 C605.760557,430.276963 604.995557,429.098963 604.169557,427.649963 C605.943557,423.067963 606.228557,417.857963 610.891557,413.925963 Z M730.292157,415.201163 C730.122157,419.953163 730.086157,424.877163 727.571157,429.150163 C726.878157,430.325163 725.302157,431.493163 724.024157,431.626163 C723.191157,431.713163 721.809157,430.088163 721.284157,428.949163 C719.147157,424.314163 721.341157,419.881163 722.440157,415.481163 C722.897157,413.655163 724.334157,412.074163 725.187157,410.619163 C728.938157,410.398163 730.388157,412.542163 730.292157,415.201163 Z M692.461557,391.015663 C697.001557,394.446663 697.800557,396.952663 696.587557,401.363663 C694.919557,407.429663 693.254557,413.515663 692.065557,419.685663 C691.420557,423.030663 689.803557,425.507663 687.846557,427.921663 C687.249557,427.677663 686.730557,427.631663 686.624557,427.398663 C684.618557,422.944663 684.506557,413.140663 685.744557,408.799663 C686.799557,405.102663 687.415557,401.209663 687.636557,397.369663 C687.828557,394.020663 690.358557,392.959663 692.461557,391.015663 Z M773.057257,392.460963 C773.508257,394.180963 774.672257,396.108963 774.311257,397.686963 C772.954257,403.612963 771.276257,409.483963 769.334257,415.246963 C768.682257,417.181963 766.882257,418.730963 765.127257,421.106963 C763.962257,419.748963 762.537257,418.854963 762.411257,417.802963 C761.976257,414.182963 760.911257,410.597963 763.100257,406.898963 C764.602257,404.362963 764.795257,401.085963 765.828257,398.231963 C766.846257,395.419963 767.967257,392.499963 773.057257,392.460963 Z M637.815257,394.162663 C639.870257,394.860663 641.658257,395.465663 644.149257,396.309663 C643.438257,399.817663 642.826257,403.114663 642.087257,406.383663 C641.613257,408.482663 640.798257,410.511663 640.417257,412.622663 C639.946257,415.233663 638.396257,416.280663 635.973257,416.424663 C632.399257,414.501663 633.097257,410.872663 632.640257,409.057663 C634.541257,403.584663 636.081257,399.152663 637.815257,394.162663 Z M575.462957,394.937663 C578.981957,395.121663 580.074957,398.058663 579.963957,400.160663 C579.713957,404.880663 578.452957,409.547663 577.602957,414.235663 C577.048957,414.371663 576.496957,414.506663 575.943957,414.642663 C574.712957,412.991663 573.057957,411.501663 572.351957,409.650663 C571.365957,407.063663 570.801957,404.229663 570.577957,401.458663 C570.297957,397.986663 573.736957,397.009663 575.462957,394.937663 Z M809.133885,391.790267 L809.187757,391.916763 C809.616757,392.903763 809.927757,393.940763 809.952757,394.010763 C809.586757,398.760763 809.457757,402.546763 808.957757,406.282763 C808.526757,409.502763 806.589757,411.818763 803.559757,413.096763 C801.369757,414.021763 799.652757,413.115763 799.096757,410.790763 C798.179757,406.965763 800.007757,403.563763 800.808757,400.007763 C801.277757,397.927763 801.831757,395.865763 802.364757,393.799763 C802.543757,393.109763 802.576757,392.228763 803.024757,391.791763 C803.987757,390.850763 805.070757,389.807763 806.298757,389.415763 C808.155854,388.822537 808.598799,390.510685 809.133885,391.790267 Z M848.733357,379.184663 C848.047357,389.449663 848.421357,400.134663 840.842357,409.473663 C835.674357,402.722663 836.758357,396.403663 839.589357,389.783663 C840.899357,386.719663 840.677357,383.116663 842.962357,380.255663 C844.469357,378.369663 845.928357,377.790663 848.733357,379.184663 Z M731.378557,374.914863 C736.434557,378.234863 736.749557,379.241863 734.875557,384.227863 C734.306557,385.740863 733.502557,387.428863 733.699557,388.918863 C734.350557,393.834863 731.958557,396.482863 727.335557,398.280863 C724.493557,393.498863 722.725557,389.145863 726.589557,384.227863 C727.425557,383.164863 726.711557,380.883863 726.711557,378.224863 C727.772557,377.472863 729.631557,376.153863 731.378557,374.914863 Z M675.106957,359.954063 C675.769957,360.204063 676.424957,360.476063 677.694957,360.980063 C675.192957,367.794063 675.066957,375.246063 670.661957,381.929063 C669.946957,381.187063 669.040957,380.638063 668.723957,379.852063 L668.467986,379.210074 C667.158922,375.890909 666.104381,372.531124 667.443957,368.843063 C667.920957,367.530063 667.987957,366.032063 668.046957,364.610063 C668.275957,359.121063 669.932957,358.003063 675.106957,359.954063 Z M608.668057,363.223163 C608.329057,369.227163 606.536057,374.380163 602.442057,379.453163 C597.094057,371.858163 601.142057,364.661163 601.708057,357.584163 C607.710057,355.411163 609.026057,356.869163 608.668057,363.223163 Z M761.781657,340.030763 C762.724657,340.394763 763.971657,341.843763 763.886657,342.697763 C763.227657,349.317763 763.214657,356.046763 760.473657,362.347763 C758.794657,366.207763 755.685657,368.575763 752.125657,372.135763 C752.125657,366.581763 750.988657,361.909763 752.404657,358.217763 C754.297657,353.284763 754.603657,348.048763 756.572657,343.244763 C756.972657,342.271763 757.603657,341.173763 758.446657,340.660763 C759.383657,340.093763 760.862657,339.674763 761.781657,340.030763 Z M703.128057,343.202163 C704.459057,343.734163 706.200057,344.432163 708.092057,345.190163 C707.516057,350.357163 707.236057,355.332163 706.283057,360.175163 C705.922057,362.005163 704.012057,363.531163 702.624057,365.452163 C700.791057,364.583163 699.372057,363.911163 697.385057,362.970163 C697.718057,361.168163 697.642057,359.102163 698.488057,357.533163 C700.056057,354.624163 700.523057,351.909163 698.941057,349.473163 C700.408057,347.276163 701.652057,345.411163 703.128057,343.202163 Z M810.764857,341.483563 C810.347857,346.341563 809.558857,351.232563 808.261857,355.923563 C807.605857,358.299563 805.465857,360.263563 803.865857,362.602563 C800.113857,357.271563 800.112857,351.755563 803.447857,334.457563 C808.741857,333.633563 811.254857,335.789563 810.764857,341.483563 Z M642.251557,341.718763 C646.041557,341.863763 647.413557,343.835763 647.518557,347.004763 C647.714557,352.938763 646.942557,358.429763 641.129557,361.754763 C635.087557,355.560763 635.426557,349.810763 642.251557,341.718763 Z M734.649957,331.610463 C739.032957,335.874463 736.648957,340.476463 735.278957,345.129463 C733.880957,345.752463 732.455957,346.387463 730.472957,347.271463 C728.786957,343.675463 726.516957,340.323463 727.453957,336.538463 C728.233957,333.380463 730.854957,331.272463 734.649957,331.610463 Z M845.946157,310.042963 C846.288157,316.217963 843.216157,321.491963 841.728157,327.180963 C841.154157,329.377963 840.082157,331.278963 840.886157,333.895963 C841.765157,336.755963 839.848157,339.533963 835.184157,344.252963 C832.995157,339.814963 831.905157,336.144963 833.125157,331.168963 C834.408157,325.938963 835.804157,320.496963 835.766157,314.851963 C835.747157,311.987963 837.547157,309.110963 838.560157,306.139963 C839.355157,306.075963 840.405157,305.976963 841.456157,305.912963 C845.010157,305.696963 845.752157,306.564963 845.946157,310.042963 Z M569.635357,309.233463 C573.415357,320.865463 568.422357,330.374463 563.722357,340.616463 C560.595357,337.913463 560.920357,334.570463 560.199357,331.868463 C559.612357,329.667463 560.463357,327.018463 560.929357,324.617463 C561.493357,321.707463 562.667357,319.069463 562.718357,315.865463 C562.764357,312.940463 565.684357,310.442463 569.635357,309.233463 Z M686.905857,301.387263 C693.922857,302.486263 693.927857,306.765263 691.961857,311.704263 C690.441857,315.520263 688.620857,319.075263 689.067857,323.512263 C689.240857,325.238263 687.211857,327.199263 686.150857,329.033263 C684.874857,331.239263 683.560857,333.423263 681.844857,336.325263 C678.471857,328.682263 677.653857,322.211263 681.587857,315.561263 C682.738857,313.615263 683.691857,311.105263 683.538857,308.938263 C683.310857,305.701263 684.467857,303.519263 686.905857,301.387263 Z M713.605957,297.560663 C714.439957,298.183663 715.071957,299.077663 715.779957,299.830663 C713.826957,305.219663 712.232957,310.533663 709.943957,315.529663 C708.849957,317.917663 706.437957,319.700663 704.273957,322.153663 C702.992957,319.196663 702.087957,317.109663 701.161957,314.973663 C702.623957,310.133663 705.300957,305.614663 705.845957,300.252663 C706.168957,297.081663 711.027957,295.637663 713.605957,297.560663 Z M611.262157,294.248463 C612.192157,295.208463 613.716157,296.039463 613.957157,297.150463 C615.640157,304.906463 616.715157,312.729463 614.430157,320.580463 C614.347157,320.863463 613.915157,321.043463 613.394157,321.484463 C605.154157,312.327463 604.177157,301.108463 611.262157,294.248463 Z M745.843357,281.834563 C746.382357,281.946563 747.070357,282.188563 747.767357,282.216563 C751.011357,282.349563 752.137357,283.380563 752.496357,286.177563 C753.507357,294.039563 751.469357,301.428563 748.799357,308.623563 C747.660357,311.692563 745.657357,314.534563 741.948357,316.349563 C741.607357,314.897563 741.087357,313.769563 741.122357,312.658563 C741.266357,308.026563 738.800357,303.122563 742.619357,298.816563 C743.220357,298.139563 743.372357,296.668563 743.138357,295.710563 C741.914357,290.711563 743.725357,286.336563 745.843357,281.834563 Z M659.526757,284.779563 C662.632757,287.419563 663.161757,288.479563 661.300757,292.027563 C659.377757,295.690563 657.760757,299.398563 657.428757,303.628563 C657.162757,307.016563 654.839757,309.355563 652.726757,311.833563 C648.082757,308.871563 649.255757,304.122563 649.500757,300.505563 C649.853757,295.307563 651.936757,290.226563 653.327757,284.916563 C655.183757,284.301563 657.047757,282.672563 659.526757,284.779563 Z M785.044557,290.485363 C787.052557,296.449363 783.451557,306.755363 778.416557,309.677363 C775.143557,304.362363 776.588557,298.845363 777.943557,293.443363 C778.921557,289.550363 782.188557,290.280363 785.044557,290.485363 Z M826.303257,266.469163 C827.393257,266.998163 828.800257,268.636163 828.673257,269.620163 C827.968257,275.087163 828.275257,280.800163 825.228257,285.799163 C824.054257,287.725163 823.168257,289.827163 821.930257,292.288163 C818.534257,285.583163 818.556257,279.051163 819.962257,272.501163 C820.352257,270.684163 821.731257,268.944163 823.034257,267.512163 C823.734257,266.741163 825.528257,266.092163 826.303257,266.469163 Z M696.650057,254.679263 C697.946057,258.340263 698.981057,261.261263 700.156057,264.577263 C698.421057,269.102263 696.677057,274.115263 694.573057,278.972263 C693.337057,281.822263 691.382057,284.196263 686.814057,283.356263 C686.677057,281.558263 686.095057,279.458263 686.461057,277.538263 C687.181057,273.749263 688.419057,270.057263 689.459057,266.330263 C689.746057,265.301263 690.547057,264.179263 690.335057,263.286263 C689.033057,257.846263 693.808057,257.519263 696.650057,254.679263 Z M626.988657,245.188863 C629.643657,245.580863 630.880657,247.192863 630.827657,250.132863 C630.757657,254.030863 631.005657,257.971863 630.497657,261.811863 C630.022657,265.399863 628.739657,268.880863 627.812657,272.407863 C627.222657,272.426863 626.631657,272.445863 626.041657,272.464863 C622.793657,267.615863 620.422657,262.518863 621.687657,256.269863 C622.172657,253.872863 622.395657,251.423863 622.826657,249.014863 C623.236657,246.725863 624.056657,244.755863 626.988657,245.188863 Z M675.163757,243.580163 C677.391757,250.649163 676.710757,256.891163 670.115757,266.232163 C665.099757,261.124163 667.162757,254.984163 667.519757,249.280163 C667.687757,246.598163 671.837757,243.854163 675.163757,243.580163 Z M790.540257,243.142763 C793.011257,243.892763 793.014257,245.249763 792.290257,247.685763 C791.203257,251.345763 791.709257,255.434763 789.506257,258.870763 C788.656257,260.197763 787.866257,261.564763 786.558257,263.721763 C785.445257,262.321763 784.697257,261.751763 784.454257,261.015763 C782.714257,255.750763 783.235257,250.545763 785.196257,245.452763 C786.006257,243.347763 788.177257,242.424763 790.540257,243.142763 Z M731.500957,236.217363 C732.650957,237.111363 733.882957,238.070363 735.052957,238.979363 C733.323957,245.990363 733.654957,253.474363 728.694957,260.154363 C726.962957,256.042363 725.484957,252.534363 724.256957,249.617363 C724.729957,246.630363 725.032957,244.082363 725.581957,241.589363 C725.790957,240.635363 726.374957,239.574363 727.118957,238.971363 C728.458957,237.884363 730.048957,237.105363 731.500957,236.217363 Z M664.019757,212.412663 C663.907757,217.791663 664.567757,223.627663 660.595757,228.422663 C659.565757,229.664663 659.512757,231.715663 658.780757,234.165663 C651.665757,225.682663 656.893757,217.242663 657.079757,208.870663 C662.568757,205.983663 664.143757,206.482663 664.019757,212.412663 Z M704.872757,205.517163 C707.678757,205.760163 709.885757,207.965163 709.183757,210.612163 C707.702757,216.201163 706.974757,222.199163 702.422757,227.262163 C696.562757,220.219163 700.341757,212.918163 700.411757,205.785163 C702.061757,205.672163 703.484757,205.397163 704.872757,205.517163 Z M768.885757,196.707463 C770.921757,198.764463 771.332757,201.277463 770.576757,204.109463 C768.954757,210.183463 768.839757,216.802463 763.785757,222.086463 C761.224757,216.559463 759.372757,211.346463 761.419757,205.361463 C762.322757,202.720463 762.562757,199.854463 763.183757,196.679463 C764.694757,196.324463 766.799757,194.599463 768.885757,196.707463 Z M744.124757,182.051063 C744.270757,182.060063 744.406757,182.254063 744.911757,182.644063 C745.157757,190.534063 742.183757,197.492063 737.302757,204.197063 C734.218757,200.570063 734.050757,196.579063 734.416757,192.502063 C734.543757,191.086063 735.151757,189.719063 735.360757,188.303063 C736.136757,183.061063 736.967757,182.236063 742.015757,182.014063 C742.716757,181.982063 743.423757,182.009063 744.124757,182.051063 Z" id="Combined-Shape" fill="#7C6576"/>
            </g>
        </g>
        <g transform="translate(172.000000, 63.000000) scale(1 1)" id="Master/Character/Standing">
            <g id="Master/Character/Standing" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <g id="Bottom/Standing" transform="translate(39.000000, 389.000000) scale(1 1)">
                    <g id="Bottom/Standing/Leg up" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <polygon id="Fill" fill="#FFF" points="71 20.0283944 112.024602 20.0283944 168.665799 115.003096 198.846117 164.414507 202.055345 87.2927375 213.688798 17.3713448 222.615962 0 359.620875 0 368.90728 69.0247971 371.994765 169.492541 374.317261 271.370266 375.188013 431.347298 378 498.689128 363.764085 537 337.597323 511.227581 330.526055 480.277878 319.521331 399.923673 309.885321 298.171226 302.445652 216.327367 296.682978 164.414507 292.345302 177.643761 278.452205 230.229948 259.521725 275.928889 238.403859 300.849528 215.913441 308.236315 190.356845 300.849528 160.96401 268.5 138.16659 223.249198 117.424802 169.492541 92.6126844 97.4829677 77.4557645 51.0183283"/>
                        <path d="M86.2137842,33 C86.2137842,37.0306346 86.2389821,41.3544858 86.2072108,45.6772429 C86.1699617,50.512035 85.4151188,55.0929978 81,58.1564551 C82.426423,61.9365427 83.7290474,65.3873085 85.0930233,69 C98.6451372,60.8249453 111.672476,52.9671772 125,44.9277899 C122.486779,40.6925602 120.195956,36.8315098 117.921568,33 L86.2137842,33 Z M332,465 C333.781217,478.540576 334.762257,491.033518 336.497437,503.584359 C337.384209,504.876695 338.01339,505.838027 338.73903,506.706503 C343.962107,512.956251 349.238895,519.164488 354.546375,525.344321 C355.373956,526.30893 356.440494,527.067071 357.485109,528 C362.401269,520.453546 366.787996,513.119022 371.822539,506.256424 C373.040343,504.597035 374.151823,503.15176 374.967346,501.586319 C374.967346,501.586319 375.086825,499.054084 374.868694,496.439917 C374.505874,489.902861 374.403934,483.351604 374.157304,476.806901 C374.06194,474.280128 373.871213,471.756633 373.694735,468.728438 C359.771644,467.483076 346.370313,466.285781 332,465 L332,465 Z M127.361754,48.9609389 C113.79995,57.1142435 99.9667132,65.5593229 86.8109645,73.4689354 C86.8109645,73.4689354 87.5004921,75.134347 87.744563,75.8435676 C100.837926,113.946962 113.576675,152.178212 127.206337,190.090368 C136.414266,215.703175 147.524416,240.571793 161.641126,263.957494 C169.815858,277.498252 179.474717,289.715642 192.80449,298.773776 C205.652687,307.504074 218.562176,308.171769 232.013436,300.477436 C237.967452,297.07121 242.950657,292.585308 247.354878,287.386907 C256.190682,276.961692 262.570454,265.112573 267.676241,252.52582 C278.540132,225.744821 286.430297,198.040414 291.381762,169.648643 C297.99466,131.733208 304.560495,93.7620417 306.098251,55.1734488 C306.263519,51.0088269 306.727582,46.855133 306.860015,42.6905111 C306.938818,40.2208062 307.898684,38.5390027 310.135818,37.5336361 C313.390826,36.0692978 316.585638,34.4519689 319.919449,33.2018174 C320.150386,33.1143942 320.424008,33.1373428 320.71952,33.2214876 C322.088724,33.6116136 322.781536,35.2289425 322.129221,36.4922074 C321.983654,36.7730544 321.808535,37.0014475 321.577598,37.1347678 C318.507559,38.9127368 315.249267,40.3694256 311.859637,42.0446723 C311.13837,51.6349955 310.352527,61.1127613 309.725385,70.6003622 C307.885551,98.3681506 304.37881,125.928309 299.669227,153.34859 C299.309141,155.450025 299.334314,157.672759 299.530227,159.806978 C302.022377,187.096124 304.637109,214.37325 307.101897,241.664582 C308.976755,262.430867 310.572519,283.223378 312.487874,303.985291 C314.951567,330.677774 317.436055,357.372443 320.241228,384.03105 C322.402843,404.565663 323.589268,425.280586 327.571016,445.543094 C328.938032,452.498701 328.923804,451.861605 329.769843,455.935525 C329.954811,456.825056 330.484544,460.283735 330.484544,460.283735 C330.484544,460.283735 333.065347,460.925203 334.449875,461.042132 C346.923757,462.095581 360.793112,463.449547 373.995924,464.606812 C374.20716,459.735155 373.995924,456.069938 373.940105,452.0441 C373.292168,405.569937 372.483341,359.097958 371.74894,312.62598 C371.499396,296.936797 371.383381,281.243243 371.121798,265.55406 C370.340333,218.485418 369.719758,171.414591 368.643876,124.353599 C367.829577,88.7974977 365.243301,53.3550466 359.972246,18.1409886 C359.032081,11.8640041 358.651199,5.50506031 358.004356,-0.816728592 C357.821577,-2.59360478 357.346569,-4.73765834 359.886876,-4.98244325 C362.417333,-5.22722816 362.476435,-2.8591527 362.684388,-1.19701944 C363.649727,6.48419996 364.504522,14.1807184 365.3429,21.8772369 C367.212286,39.0493355 369.730703,56.1842794 370.74201,73.4077392 C372.392499,101.550354 373.45853,129.742145 374.044081,157.928472 C375.14842,211.148428 375.727404,264.379311 376.540609,317.60473 C377.250932,364.075616 377.907625,410.548687 378.739436,457.01848 C379.108279,477.647072 379.849247,498.268016 380.443554,518.892237 C380.477483,520.064801 380.714987,521.2319 380.852893,522.367309 C397.260366,528.22794 413.554013,533.93121 429.756817,539.880358 C438.69988,543.164191 447.517077,546.803181 456.34303,550.402831 C459.44481,551.667188 462.78081,553.008041 461.837361,557.783533 C460.996795,562.037763 456.582723,565.659269 452.251833,565.989291 C449.07344,566.230798 445.867683,566.214406 442.675061,566.276695 C428.757549,566.549892 414.841131,566.905049 400.922524,567 C395.966681,567.031813 390.973626,566.647151 386.068129,565.952136 C384.350877,565.71063 382.651137,564.169796 381.296161,562.857356 C379.774822,561.386461 378.650782,559.503584 377.235609,557.643656 C377.421672,557.424005 375.846704,559.064282 374.975491,560.305692 C371.385569,565.421041 366.273215,566.309479 360.762467,565.924817 C355.791301,565.578402 350.387813,565.392628 345.544702,564.177446 C344.166741,563.832124 342.817237,563.330534 341.63519,562.531704 C338.820166,560.629157 338.206158,556.515897 337.907363,553.427673 C337.543993,549.67394 337.249576,545.912557 336.769095,542.171938 C333.568812,517.282558 330.634489,492.35493 327.124465,467.508169 C324.960662,452.19272 321.636701,437.03354 319.700551,421.69405 C316.692897,397.861397 314.236866,373.953342 311.895755,350.043102 C309.399228,324.551595 307.280299,299.02184 305.047542,273.504105 C302.902346,248.976439 300.844708,224.441123 298.699511,199.912363 C297.992471,191.815883 297.132204,183.734703 295.391967,175.493975 C294.912581,177.457719 294.389416,179.412719 293.959282,181.386298 C288.154116,208.006657 280.799155,234.153838 270.018446,259.244292 C264.889674,271.180834 258.319461,282.258444 249.67191,292.023614 C241.839752,300.869747 232.496106,307.313929 220.757719,309.879799 C210.190435,312.191049 200.704506,309.427384 191.799749,303.809352 C180.683032,296.792549 171.488237,287.690702 164.432071,276.746413 C157.449236,265.915773 150.812259,254.784616 145.008188,243.290654 C128.859014,211.311253 116.842627,177.636936 105.346123,143.785587 C98.4530359,123.488109 91.8784451,103.082446 84.9569014,82.7958962 C81.2586256,71.9576071 77.1148931,61.2723086 73.1780189,50.514886 C72.9580268,49.9149444 72.7172394,49.3215595 72.4490897,48.7401954 C69.4786487,42.3124058 68.6326092,41.8763827 61.2480971,43.6368671 C45.5936317,47.3709298 29.9917018,51.3279216 14.3645986,55.1789128 C12.051945,55.749349 9.74476373,56.4509201 7.39161401,56.7230247 C3.40001538,57.184182 1.03810975,55.73405 0.452558537,51.9103785 C-0.113291886,48.2200274 -0.0224493615,44.4029127 0.065109698,40.6469942 C0.13844041,37.544564 2.38980273,35.6015838 4.77031466,34.139431 C10.3500157,30.7113495 15.9045436,27.1794529 21.7644336,24.2857456 C30.6659065,19.890545 39.8223952,16.0089557 48.8606791,11.8891382 C54.7884274,9.1866691 59.342593,11.4236098 65.0043807,13.3359919 C65.4082469,10.6018319 65.7628611,8.20425108 66.2291131,5.03953186 C74.8296017,2.24199002 83.7945549,-0.823285331 92.8591065,-3.55744537 C97.0816422,-4.83163826 100.812753,-3.39789806 103.26769,0.333979051 C108.605509,8.4479432 113.872186,16.6110829 119.023943,24.8441611 C121.548927,28.8809266 123.582486,33.2269516 126.130455,37.248418 C143.583164,64.7878133 159.014354,93.469174 174.582355,122.083875 C180.557166,133.066412 187.294836,143.636967 193.705253,154.383462 L196.675694,159.177531 C196.839868,156.725311 197.22075,152.148707 197.285324,149.694301 C197.712175,133.413919 197.604915,117.09966 198.604183,100.854247 C200.508592,69.8802137 204.515514,39.1968622 214.269593,9.53854741 C214.887979,7.65567061 215.044491,4.39915705 218.037916,5.27885283 C221.103578,6.1804044 219.459656,8.93423466 218.82157,10.8641014 C209.636624,38.6023845 206.011679,67.3383847 203.564403,96.2623447 C202.861742,104.564269 202.816868,112.925203 202.546529,121.262097 C202.124057,134.359182 201.865758,147.462824 201.318514,160.555539 C201.271451,166.646749 200.719829,164.442592 204.22438,171.035393 C204.896396,171.752263 205.635175,172.588247 205.86064,173.490892 C205.90223,173.655903 205.926309,173.825285 205.937254,173.999039 C206.040136,175.545336 204.560388,176.841385 203.057655,176.450166 C202.840947,176.393341 202.645033,176.309196 202.48086,176.184618 C201.260506,175.252469 200.303923,173.873368 199.488529,172.531422 C189.581222,156.237926 179.445166,140.076658 169.914362,123.564605 C156.907464,101.033465 144.382141,78.2247565 131.617124,55.5537397 C130.465723,53.5080372 129.073534,51.5978406 127.361754,48.9609389 L127.361754,48.9609389 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Head" transform="translate(195.000000, 0.000000) scale(1 1)">
                    <g id="Head/Short 2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M158.083394,64.0450245 L160.14485,64.0559838 C172.081015,64.1816887 179.556674,65.7847012 179.556674,73.661629 C179.556674,81.9007143 196.343789,69.2320339 196.343789,76.2007285 C196.343789,83.169423 177.375122,92.2895869 173.142929,92.2895869 C170.321466,92.2895869 166.832484,91.4321275 162.675982,89.7172087 C162.355753,89.8801448 162.034247,90.0400505 161.711464,90.1969258 C178.54858,113.09506 184.554728,135.68621 179.730441,157.969875 C178.701421,162.722982 163.088243,165.681395 132.890907,166.845115 L131.033485,166.91378 L133.516589,190.302878 C134.46397,192.453544 135,194.902136 135,197.5 C135,206.060414 129.179702,213 122,213 C115.821567,213 110.649817,207.861014 109.327491,200.972469 L108.533842,200.972567 L108.458037,198.171625 C107.621355,168.402615 106.148629,152.1788 104.03986,149.50018 C79.854786,149.50018 79.854786,137.358858 81.1798049,130.747277 C81.4799339,129.249693 82.4258773,127.952931 83.7370172,126.842852 C82.3295911,127.4597 80.755144,128.195416 79.0148066,129.049221 C64.8979655,116.544786 59.6496322,106.708885 63.2698067,99.5415167 C68.7000684,88.7904648 77.1433365,86.4823543 79.0148066,86.4823543 C80.8862767,86.4823543 79.8390517,61.629205 106.479791,58.5998547 C132.854123,55.6007979 145.312908,63.8791574 158.083394,64.0450245 L158.083394,64.0450245 Z" id="Path" fill="#FFF"/>
                        <path d="M175.323807,54.7995655 C175.344807,54.9995655 175.331807,55.1865655 175.269807,55.3565655 C173.476807,60.2225655 169.348807,63.2625655 164.033807,64.7865655 C160.561807,65.7815655 156.897807,65.9395655 153.327807,65.3915655 C145.414807,64.1765655 137.531807,62.7705655 129.633807,61.4575655 C121.541807,60.1125655 113.472807,59.3335655 105.252807,60.7085655 C95.2228066,62.3865655 88.2708066,67.6115655 84.7208066,77.2105655 C84.2168066,78.5725655 82.8128066,81.6665655 82.2968066,83.4185655 C80.6798066,88.9055655 82.4698066,86.5955655 77.5608066,88.7905655 C75.9288066,89.5195655 72.6578066,91.2685655 71.2418066,92.3825655 C63.4678066,98.4995655 62.0648066,106.974566 68.0158066,114.860566 C71.2728066,119.177566 75.3838066,122.852566 79.1378066,126.790566 C79.7228359,127.403182 80.5892918,127.792285 81.0777939,128.445094 C81.419045,127.944624 81.8103514,127.459674 82.2498407,126.993015 C86.2828407,122.710015 91.2598407,120.287015 97.1298407,119.916015 C97.9715513,119.862769 98.6221744,119.853223 99.1509804,119.935049 C99.1769638,119.893904 99.2042185,119.854837 99.2287066,119.820266 C103.587707,113.662266 103.222707,106.478266 103.703707,99.4692655 L103.704707,99.4482655 C103.838707,97.7832655 106.032707,97.2402655 107.080707,98.5422655 C107.360707,98.8892655 107.537707,99.2042655 107.558707,99.5302655 C108.009707,106.802266 107.864707,114.027266 103.817707,120.509266 C103.348707,121.260266 102.740707,122.022266 102.009707,122.480266 C101.927293,122.531835 101.829034,122.577492 101.720783,122.616376 C101.864336,123.27571 101.41684,123.955425 100.712841,124.057015 C99.8268407,124.185015 98.9818407,124.299015 98.1388407,124.291015 C94.1318407,124.253015 90.6948407,125.752015 87.5628407,128.066015 C84.4108407,130.397015 82.7568407,133.409015 83.4178407,137.481015 C84.1158407,141.778015 86.1528407,145.101015 90.4668407,146.292015 C93.2248407,147.053015 96.1658407,147.219015 99.0408407,147.473015 C100.992841,147.645015 102.971841,147.505015 105.452841,147.505015 C105.846841,149.741015 106.391841,151.805015 106.549841,153.899015 C107.391841,165.128015 109.589597,186.677083 110.290597,197.916083 C110.403597,199.730083 111.631597,202.613083 108.936597,203.014083 C105.807597,203.481083 106.171597,200.348083 106.015597,198.316083 C105.222597,187.994083 103.110841,167.350015 102.365841,157.025015 C102.262841,155.591015 102.087841,154.162015 101.886841,152.128015 C98.7218407,151.928015 96.0138407,151.896015 93.3438407,151.563015 C84.1658407,150.422015 78.8868407,144.368015 79.0018407,135.420015 C79.0191774,134.043224 79.206374,132.744747 79.5842218,131.526626 C79.3809148,131.578272 79.1866168,131.594292 79.0148066,131.563566 C78.0158066,131.385566 76.9448066,130.768566 76.2198066,130.027566 C72.2858066,126.011566 68.0368066,122.211566 64.6598066,117.757566 C56.9398066,107.576566 59.0088066,95.7245655 69.4088066,88.2635655 C71.0218066,87.1055655 72.7178066,85.8975655 74.5948066,85.3005655 C77.9648066,84.2275655 77.6188066,83.8815655 78.3598066,81.0715655 C79.3718066,77.2315655 81.3888066,73.6925655 83.2928066,70.2105655 C85.8668066,65.5035655 89.7818066,61.7375655 94.6138066,59.3985655 C98.4608066,57.5365655 102.766807,56.8915655 106.921807,56.1355655 C109.742807,55.6225655 112.609807,55.4345655 115.474807,55.4765655 C117.131807,55.5005655 118.787807,55.6035655 120.433807,55.7655655 C125.113807,56.2255655 129.727807,57.1635655 134.317807,58.1645655 C138.654807,59.1095655 143.162807,59.5685655 147.566807,60.2395655 C151.659807,60.8635655 155.857807,61.2445655 159.982807,61.0855655 C164.535807,60.9095655 168.337807,58.7765655 170.824807,54.7315655 C171.109807,54.2675655 171.344807,53.6385655 171.772807,53.4255655 C171.997807,53.3125655 172.231807,53.2165655 172.470807,53.1295655 C173.726807,52.6725655 175.183807,53.4705655 175.323807,54.7995655 Z M182.184607,70.2457655 C182.573607,70.4607655 182.686607,71.5507655 182.579607,72.1887655 C182.466607,72.8587655 182.050607,73.5567655 181.581607,74.0737655 C176.495968,79.6717339 171.207078,85.227518 165.018375,89.6133189 C166.126955,90.6247472 167.258256,91.7007752 167.704741,92.3455145 C180.693741,111.101515 187.398741,131.430515 182.901741,154.475515 C182.693741,155.541515 182.307741,156.573515 182.099741,157.639515 C181.392741,161.257515 179.300741,163.215515 175.617741,163.992515 C163.528741,166.539515 151.400741,168.675515 139.002741,168.824515 C137.207741,168.845515 135.411741,168.827515 132.917741,168.827515 C133.880741,172.969515 134.755741,176.566515 135.542741,180.184515 C135.830741,181.504515 138.883144,200.886821 137.713144,201.130821 C135.869144,201.516821 133.811144,201.663821 133.723144,199.281821 C133.624144,196.639821 131.523741,180.843515 130.920741,178.233515 C130.639741,177.014515 127.197741,175.290515 124.913741,172.993515 C123.269741,171.339515 121.911741,169.332515 120.754741,167.297515 C120.230741,166.376515 119.306741,164.076515 119.714741,163.111515 C119.937741,162.580515 122.493741,163.094515 123.426741,163.265515 C132.994741,165.016515 142.588741,164.868515 152.157741,163.543515 C159.336741,162.549515 166.459741,161.154515 173.616741,159.982515 C176.277741,159.546515 177.592741,158.076515 178.180741,155.416515 C180.533741,144.767515 180.179512,134.982781 177.578741,123.676515 C174.97797,112.370248 168.182135,101.215515 161.202135,92.3455145 L161.062353,92.1678154 C153.277191,96.6829704 144.170756,98.9139544 135.239607,99.4837655 C135.083607,99.4937655 134.927607,99.5027655 134.771607,99.5097655 C133.169607,99.5847655 131.604607,99.5217655 129.998607,99.3747655 C127.988607,99.1897655 122.949607,98.7667655 123.063607,95.8157655 C123.138607,93.8907655 124.967607,94.3277655 126.449607,94.3517655 C131.513607,94.4347655 136.628607,94.8037655 141.635607,94.2567655 C148.389591,93.5172833 154.51121,91.1993352 160.04669,87.6675977 C160.281576,87.3712072 160.604968,87.1671011 160.959037,87.0682708 C163.006238,85.6996017 164.971043,84.1616506 166.856607,82.4757655 C170.369607,79.3357655 173.644607,75.9307655 177.024607,72.6427655 C177.930607,71.7607655 178.713607,70.6777655 179.769607,70.0527655 C180.361607,69.7027655 181.517607,69.8787655 182.184607,70.2457655 Z M199.016207,74.1716655 C201.411207,76.0866655 198.942207,78.1666655 197.573207,79.5396655 L196.831459,80.281599 C190.516612,86.5751974 183.915225,92.4903259 174.608207,94.1136655 C172.385207,94.5006655 170.367207,94.6076655 169.144207,92.5546655 C168.696207,91.8016655 168.908207,90.7786655 169.649207,90.3106655 C169.743207,90.2506655 169.827207,90.2176655 169.911207,90.2156655 C180.434207,89.9496655 187.445207,83.3856655 194.365207,76.6416655 C195.704207,75.3366655 196.934207,72.5076655 199.016207,74.1716655 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Body" transform="translate(28.000000, 114.000000) scale(1 1)">
                    <g id="Body/Puffy Top" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M410.833588,128.65121 C412.500429,128.471187 413.928579,128.301165 415.358726,128.165147 C431.169244,126.654952 442.995922,133.344817 449.556423,147.859695 C454.086554,157.878991 454.680784,168.188324 450.725908,178.600671 C450.339408,179.617803 449.932935,180.629934 449.607357,181.667068 C449.509484,181.980108 449.652299,182.369159 449.717215,183.07825 C464.366237,187.876871 472.787327,197.413104 471.128476,213.782222 C487.580163,223.935535 495.163339,237.750322 489.779314,257.626893 C491.436167,257.772912 492.967184,257.878926 494.493207,258.047947 C503.479565,259.043076 510.135942,263.427643 514.132764,271.648707 C517.212774,277.983526 516.805302,283.929295 512.135351,289.399003 C511.437256,290.215108 510.80108,291.084221 509.943191,292.172361 C510.915931,292.69943 511.635998,293.226498 512.440956,293.506534 C525.140503,297.913104 532.076518,311.016799 528.498153,323.976475 C525.25136,335.740997 517.141865,342.123822 505.577847,344.717158 C501.60799,345.607273 499.726427,348.617662 500.938857,352.139118 C501.400259,353.481291 502.259146,354.739454 503.145997,355.872601 C506.94907,360.73623 510.955879,365.443839 514.641105,370.392479 C517.435485,374.143964 520.124002,378.030467 522.33314,382.139998 C522.973311,383.330152 523.375789,384.39329 524.240669,384.903356 C524.240669,384.903356 525.864565,385.208395 526.773388,385.382418 C536.640605,387.268662 539.861432,390.770115 539.992263,400.869421 C540.096128,408.833451 539.129381,415.923368 538.587083,423.881398 C538.429288,426.189696 538.011828,429.423114 537.143953,431.592395 C535.485102,435.738931 532.091498,437.424149 527.571354,436.724059 C523.632457,436.11498 519.691562,435.528904 515.473027,434.889821 C512.485897,438.849334 508.195456,439.005354 503.701278,435.488899 C503.265842,435.10685 501.614981,435.409889 500.56534,435.488899 C496.660399,435.780937 493.75017,432.538517 491.707815,431.200344 C490.213751,431.043324 485.607718,432.500512 483.021069,429.926179 C481.156485,428.070939 479.999983,426.478734 478.742612,423.489347 C478.378084,423.365331 475.556739,422.319195 474.002752,421.870137 C467.515157,419.992895 464.071618,415.615328 463.166791,408.996472 C462.057228,400.85942 461.321181,393.471464 462.434739,384.269274 C462.602522,382.262014 463.07491,379.740688 464.161502,379.13861 C465.532725,378.380512 467.988544,378.545533 467.988544,378.545533 C468.600751,376.426259 468.698624,373.448874 469.147043,371.397609 C469.563504,369.488362 469.625423,367.501105 470.002934,364.184676 C467.636001,365.742877 466.26278,366.688 464.84961,367.568114 C458.902315,371.267592 452.562528,372.72278 445.596553,371.166579 C435.975018,369.017301 429.533363,361.336308 429.236747,351.397022 C429.171831,349.238743 429.227759,347.076463 429.227759,344.762164 C427.684758,344.388115 426.492303,343.96406 425.26689,343.825042 C418.028268,343.003936 412.614282,339.253451 409.209692,333.011644 C407.059478,329.071134 405.233843,324.78558 404.204177,320.432017 C401.290951,308.107422 401.787308,295.913845 407.524875,283.922294 C402.454444,281.286953 397.559785,278.742624 392.243672,275.979267 C386.47115,280.188811 379.905656,281.307956 373.137424,277.353445 C372.636074,277.585475 372.211623,277.650483 372.003893,277.897515 C362.242538,289.489014 343.443892,287.723786 332.540018,278.464588 C329.376117,275.777241 326.454902,272.799856 323.033334,269.59044 C321.778959,270.565567 320.212988,271.731717 318.702944,272.965877 C307.800069,281.87903 295.1944,286.616643 281.282423,287.591769 C274.694957,288.053829 268.095506,286.67665 262.564672,282.922165 C258.987306,280.494851 256.270825,280.775887 252.70145,282.642129 C246.226837,286.026566 239.245881,287.486755 232.257934,284.380353 C228.699544,282.797149 225.574593,280.235817 221.739561,277.789501 C214.582832,288.228851 203.890684,293.352514 190.836596,291.043215 C177.044464,288.6029 170.967337,278.266563 168.334747,265.119862 C166.615974,265.922966 165.197811,266.662062 163.721723,267.258139 C161.374764,268.205261 159.045781,269.2814 156.607939,269.911482 C145.65413,272.741848 137.135166,268.336278 133.037475,257.842921 C130.502759,251.355082 130.402888,244.731225 132.034774,238.025358 C132.628005,235.589042 133.204259,233.148727 133.811472,230.613399 C132.684932,230.015321 131.933905,229.54026 131.12695,229.204217 C129.787685,228.646144 128.383504,228.243092 127.056224,227.661017 C113.656583,221.780256 111.105887,209.577678 121.019045,198.681269 C122.340333,197.229081 123.854372,195.953916 125.498242,194.388713 C124.81313,193.394585 124.25685,192.502469 123.618677,191.673362 C118.819894,185.437555 118.502305,179.777823 122.700866,173.10496 C125.754909,168.249332 129.981434,164.556855 134.651384,161.327437 C136.107498,160.321307 137.548631,159.294174 139.84965,157.675965 C138.669179,157.01988 137.963094,156.753845 137.408812,156.300787 C131.248792,151.264135 131.356652,143.576141 137.304946,137.793393 C141.684273,133.536842 146.88154,130.565458 152.474294,128.364173 C163.172435,124.155629 174.19915,121.311261 185.802118,121.251253 C186.864741,121.246253 187.927365,121.065229 188.991985,120.964216 C189.186733,120.255124 189.475359,119.743058 189.439406,119.253995 C189.004969,113.249218 192.481465,109.803772 197.295229,107.335453 C205.057173,103.353938 213.345435,103.249925 221.749548,104.283058 C222.808176,104.414075 223.849827,104.689111 225.182101,104.953145 C228.094328,97.6682028 234.157473,96.1320041 241.057534,95.9209768 C248.031499,95.7079492 254.787747,96.8761003 261.490064,98.605324 C262.832325,98.9503686 264.192563,99.2224038 265.609727,99.5434453 C264.922617,88.4540109 269.141152,82.0131778 280.562356,88.9640769 C289.810375,79.9519112 291.596061,79.8228945 300.41963,87.7869246 C301.364406,87.2048494 302.32616,86.3847433 303.43772,85.9716898 C306.281037,84.9165534 309.148322,83.4163593 312.082521,83.2143332 C319.779549,82.6852648 324.299694,88.2869893 322.683787,95.7949605 C322.235368,97.8762297 321.611177,99.9184938 320.946038,102.43782 C324.571342,101.994762 327.747227,101.347679 330.936096,101.28067 C335.084722,101.193659 339.283282,101.269669 343.389962,101.813739 C349.307296,102.59684 353.722576,105.940273 356.296242,110.680886 C363.269208,109.348714 369.794754,107.984537 376.364244,106.881394 C382.771942,105.806255 388.991885,107.007411 395.101969,108.979666 C402.657181,111.418981 407.375069,116.439631 409.122805,124.223638 C409.360497,125.282775 409.62815,126.337911 409.943741,127.377046 C410.043612,127.705088 410.347219,127.970122 410.833588,128.65121" id="Fill" fill="#FFF"/>
                        <path d="M299.856255,87.7869246 C300.802252,87.2048494 301.766268,86.3847433 302.880443,85.9716898 C305.730447,84.9155532 308.604476,83.4163593 311.546578,83.2143332 C319.26171,82.6852648 323.792486,88.2869893 322.172779,95.7949605 C321.723305,97.8752295 321.097645,99.9184938 320.430943,102.43782 C324.063772,101.994762 327.248128,101.347679 330.444498,101.28067 C334.602881,101.193659 338.811317,101.269669 342.926655,101.812739 C348.857907,102.59684 353.284573,105.940273 355.863291,110.680886 C362.85366,109.348714 369.394555,107.984537 375.979496,106.881394 C382.402266,105.806255 388.636838,107.007411 394.761293,108.979666 C402.334276,111.417981 407.063261,116.439631 408.815107,124.223638 C409.053359,125.282775 409.321641,126.337911 409.636974,127.377046 C409.73808,127.705088 410.0414,127.970122 410.529915,128.65121 C412.199675,128.471187 413.631184,128.301165 415.065696,128.165147 C430.913401,126.654952 442.767897,133.344817 449.343828,147.858695 C453.883614,157.878991 454.480243,168.188324 450.516064,178.600671 C450.127655,179.617803 449.721226,180.629934 449.394882,181.667068 C449.295778,181.980108 449.43993,182.369159 449.503997,183.07825 C464.187475,187.876871 472.629373,197.413104 470.966621,213.782222 C487.456002,223.935535 495.058015,237.750322 489.661327,257.626893 C491.322077,257.772912 492.856695,257.878926 494.385306,258.047947 C503.392801,259.042076 510.064834,263.427643 514.071057,271.647707 C517.158311,277.982526 516.750882,283.929295 512.068946,289.399003 C511.37021,290.215108 510.731537,291.084221 509.872631,292.172361 C510.847659,292.69943 511.568419,293.226498 512.37627,293.505534 C525.105688,297.913104 532.058017,311.016799 528.471236,323.976475 C525.216805,335.739997 517.088237,342.123822 505.497019,344.717158 C501.516823,345.607273 499.631836,348.617662 500.846116,352.139118 C501.308604,353.481291 502.170512,354.739454 503.058448,355.872601 C506.871468,360.73623 510.887701,365.443839 514.581595,370.392479 C517.382547,374.143964 520.076387,378.030467 522.290721,382.139998 C522.933398,383.330152 523.335823,384.39329 524.202736,384.903356 C524.202736,384.903356 525.830452,385.208395 526.742413,385.382418 C536.631838,387.268662 539.861242,390.770115 539.99238,400.869421 C540.095489,408.833451 539.127468,415.923368 538.583895,423.881398 C538.424728,426.189696 538.006287,429.423114 537.13637,431.592395 C535.473617,435.738931 532.073032,437.424149 527.541256,436.724059 C523.594095,436.11498 519.64293,435.528904 515.415474,434.889821 C512.421318,438.849334 508.120785,439.005354 503.616036,435.488899 C503.178575,435.10685 501.524832,435.409889 500.471722,435.488899 C496.557596,435.780937 493.641522,432.538517 491.593363,431.200344 C490.096786,431.042324 485.479919,432.499512 482.887186,429.926179 C481.018216,428.070939 479.858994,426.478734 478.598666,423.489347 C478.23328,423.365331 475.404298,422.319195 473.847658,421.870137 C467.344803,419.992895 463.892164,415.615328 462.986209,408.996472 C461.874036,400.85942 461.135258,393.471464 462.252435,384.269274 C462.420612,382.262014 462.893111,379.740688 463.98326,379.13861 C465.356707,378.380512 467.817301,378.545533 467.817301,378.545533 C468.43295,376.426259 468.531053,373.448874 468.979526,371.397609 C469.396966,369.488362 469.460033,367.501105 469.838432,364.184676 C467.465931,365.741877 466.08948,366.688 464.672987,367.568114 C458.711703,371.267592 452.357005,372.72278 445.373643,371.166579 C435.730479,369.017301 429.272672,361.336308 428.975358,351.397022 C428.911291,349.238743 428.96735,347.076463 428.96735,344.761164 C427.420719,344.388115 426.224458,343.96406 424.997165,343.825042 C417.741517,343.003936 412.314796,339.253451 408.901198,333.011644 C406.745926,329.071134 404.915997,324.78558 403.884911,320.432017 C400.964833,308.107422 401.462357,295.913845 407.213419,283.922294 C402.131062,281.286953 397.22489,278.742624 391.895272,275.979267 C386.110174,280.188811 379.529237,281.307956 372.745086,277.353445 C372.242557,277.585475 371.817108,277.650483 371.608889,277.897515 C361.823574,289.489014 342.981713,287.723786 332.053193,278.464588 C328.879848,275.777241 325.951762,272.799856 322.522147,269.59044 C321.264821,270.565567 319.695167,271.731717 318.181571,272.965877 C307.254053,281.87803 294.618734,286.616643 280.674035,287.591769 C274.070074,288.053829 267.456102,286.67665 261.911257,282.922165 C258.326479,280.494851 255.603609,280.775887 252.025837,282.642129 C245.534995,286.026566 238.538621,287.486755 231.534237,284.380353 C227.967478,282.797149 224.835176,280.235817 220.991124,277.789501 C213.817562,288.228851 203.100265,293.352514 190.015473,291.043215 C176.1909,288.6029 170.09948,278.266563 167.460698,265.119862 C165.736881,265.922966 164.316384,266.662062 162.836824,267.258139 C160.484344,268.205261 158.149884,269.2814 155.706308,269.911482 C144.725733,272.741848 136.187733,268.336278 132.079403,257.842921 C129.538725,251.355082 129.43962,244.731225 131.075345,238.025358 C131.66897,235.589042 132.24758,233.148727 132.856222,230.613399 C131.72603,230.015321 130.974238,229.54026 130.164384,229.204217 C128.82297,228.646144 127.415487,228.243092 126.085084,227.661017 C112.652926,221.780256 110.097232,209.577678 120.032705,198.680268 C121.357101,197.229081 122.875701,195.953916 124.523438,194.388713 C123.836714,193.394585 123.279127,192.502469 122.639452,191.673362 C117.829382,185.437555 117.511047,179.777823 121.718482,173.10496 C124.779708,168.249332 129.017175,164.556855 133.698109,161.327437 C135.157648,160.321307 136.60217,159.294174 138.9076,157.675965 C137.725354,157.01988 137.017608,156.753845 136.462023,156.299787 C130.286513,151.264135 130.394627,143.576141 136.357913,137.793393 C140.74754,133.536842 145.957031,130.564458 151.56294,128.364173 C162.285243,124.155629 173.337893,121.311261 184.968152,121.251253 C186.034276,121.246253 187.099399,121.064229 188.166524,120.964216 C188.36173,120.255124 188.651035,119.743058 188.614997,119.252995 C188.178536,113.249218 191.664211,109.803772 196.488296,107.335453 C204.269498,103.352938 212.576254,103.249925 221.001134,104.283058 C222.063254,104.414075 223.107355,104.688111 224.441761,104.953145 C227.360838,97.6682028 233.437243,96.1320041 240.354535,95.9209768 C247.344903,95.7079492 254.116041,96.8751002 260.834122,98.6043238 C262.17954,98.9503686 263.543978,99.2224038 264.964476,99.5434453 C264.27575,88.4540109 268.504207,82.0131778 279.951273,88.9640769 C289.222045,79.9519112 291.011932,79.8228945 299.856255,87.7869246 Z M506.028579,423.378332 C505.380897,425.282579 505.42194,427.567874 505.721255,429.603138 C506.014564,431.600396 506.046598,434.318748 509.914676,434.728801 C510.66847,434.808811 511.384225,434.316747 511.510358,433.569651 C512.096976,430.104202 511.01984,425.748639 510.091862,423.908401 C509.037751,421.819131 506.697284,421.414078 506.028579,423.378332 Z M497.900011,355.978615 C493.896791,356.741713 490.373077,357.4138 486.435927,358.164897 L486.045307,358.746647 C483.039611,363.220074 479.743623,367.945787 473.498289,369.156319 C472.875633,372.436743 472.346075,375.853185 471.721416,379.149612 C473.482272,379.632674 474.78865,379.673679 476.171107,379.922712 C483.325648,381.210878 490.504215,382.372028 497.641739,383.744206 C499.211393,384.046245 502.05439,383.743206 501.521829,386.389548 C501.024304,388.866869 498.558705,387.773727 496.889946,387.496691 C489.001631,386.190522 481.131335,384.772339 473.247025,383.439166 C471.722417,383.181133 469.901498,382.954104 468.045542,382.732075 C467.280736,382.641063 466.583,383.186134 466.479891,383.949232 C466.226624,385.833476 466.121514,387.478689 466.139533,389.036891 C466.2036,394.836641 466.196593,400.666395 466.79122,406.42514 C467.684161,415.074258 470.029633,417.115522 478.834915,418.924756 C481.355572,411.056739 486.092565,408.156364 493.187043,416.885493 C497.034098,415.270284 499.925146,417.455566 502.512873,419.997895 C503.247648,419.600844 503.71414,419.318807 504.205658,419.087778 C509.602346,416.55845 512.960887,418.092649 514.22622,423.703375 C514.544556,425.116557 514.8709,425.83665 515.08913,427.417855 C515.476538,428.495994 515.74382,430.581264 516.071165,430.744285 C520.237557,431.580393 525.46807,433.073587 529.809647,432.294486 C530.575454,432.158468 531.16908,431.545389 531.29321,430.77829 L531.295213,430.768288 C533.087101,419.893882 533.23726,409.501538 532.404382,398.630131 L532.401379,398.593127 C532.209176,396.045797 530.75164,393.733498 528.481246,392.558346 C527.411118,392.004274 526.313962,391.66323 524.635192,392.103287 C524.635192,394.653617 524.887458,397.359967 524.57613,399.999308 C524.129659,403.758795 523.830344,407.690303 522.459899,411.149751 C520.255576,416.71147 513.671636,417.114522 510.291072,412.161882 C509.395128,410.850712 508.806507,409.226502 508.46715,407.6623 C507.430057,402.87068 506.502078,398.052057 505.724258,393.213431 L505.720254,393.183427 C505.409927,391.146163 508.190858,390.172037 509.180902,391.980271 C510.064834,393.598481 510.081852,395.274697 510.418206,396.794894 C511.201031,400.337352 511.764626,403.929817 512.547451,407.472275 C513.000929,409.526541 514.113101,411.694821 516.405517,411.207758 C517.687869,410.936723 519.264531,408.873456 519.493773,407.425269 C520.200518,402.968693 520.681024,398.413103 520.59193,393.909521 C520.43977,386.216526 517.770957,379.473654 512.382277,373.695906 C508.054715,369.054306 504.295753,363.882637 500.287528,358.943998 C499.401594,357.852857 498.524669,356.754715 497.900011,355.978615 Z M495.377352,420.88701 C494.599532,421.204051 493.880774,422.466214 493.742629,423.406336 C493.434304,425.504608 494.092998,428.654015 495.503485,430.219217 C496.670715,431.515385 498.351487,431.727412 500.378624,431.984446 C501.076359,432.073457 501.687003,431.504384 501.652967,430.803293 C501.471776,427.181824 500.544799,424.181436 498.132256,421.775125 C497.494584,421.138043 496.081094,420.601973 495.377352,420.88701 Z M523.621324,420.279732 C526.559421,420.41775 529.100099,423.168105 528.981974,426.081482 C528.894882,428.22776 526.859737,429.877973 524.423169,429.77496 C521.499087,429.651944 518.84629,426.755569 519.100558,423.963208 C519.314784,421.610904 521.09466,420.160716 523.621324,420.279732 Z M483.842193,416.515445 C482.646933,419.676854 481.840082,422.488217 483.723067,425.4486 C485.071489,427.567874 487.107635,428.444988 489.860537,427.45586 C489.564224,423.236314 490.475185,418.841746 486.388877,415.752346 C485.510952,415.08826 484.231603,415.486312 483.842193,416.515445 Z M286.376046,88.1799755 C285.366982,89.2101087 284.595168,90.4712719 283.720246,91.6304218 C281.364763,94.7518255 280.705068,94.8698408 277.526718,92.8295769 C276.216336,91.9884681 274.889938,91.1723625 273.600579,90.3782598 C272.732664,89.8441907 271.597468,90.0152129 270.934769,90.789313 C267.669327,94.6118074 270.250047,98.6463293 270.986824,102.650847 C271.284137,104.268056 269.840616,105.701242 268.229918,105.367199 C268.03271,105.326193 267.839506,105.280187 267.648305,105.22618 C262.587971,103.782994 257.600714,102.021766 252.476312,100.876618 C246.964502,99.6454585 241.311543,99.0963875 235.690619,100.403557 C232.615377,101.118649 229.790399,102.160784 228.922484,105.835259 C227.844348,110.398849 227.75125,110.332841 223.305564,109.079679 C215.148966,106.780381 207.078459,107.092422 199.282241,110.449856 C193.368007,112.996185 192.418005,115.029448 193.373012,121.427276 C193.824488,124.458668 193.069692,125.395789 190.028486,125.462798 C189.123533,125.482801 188.214574,125.281774 187.307618,125.279774 C172.27277,125.237769 158.095827,128.642209 145.082109,136.235191 C142.315192,137.8504 139.748487,140.098691 137.682309,142.553008 C134.76123,146.022457 135.234729,149.60192 138.540213,152.740326 C139.971722,154.100502 141.769617,155.067627 143.257185,156.376797 C144.713721,157.657962 144.594595,159.589212 142.985899,160.674352 C142.06593,161.295433 140.980786,161.65848 140.001754,162.197549 C134.191629,165.394963 129.020178,169.368477 125.300257,174.939198 C122.565374,179.034727 121.096826,184.07738 125.863851,189.395067 C127.569649,191.299314 128.473602,193.920653 129.266438,195.350838 C125.718698,199.336353 122.225015,202.162719 120.166846,205.801189 C115.894342,213.356167 117.870425,218.852878 125.599573,222.902401 C128.000103,224.159564 130.639886,224.953667 133.145527,226.016804 C136.571138,227.470992 137.292898,228.729155 136.576143,232.344623 C135.940473,235.551038 135.132621,238.723448 134.402852,241.91086 C133.308699,246.683477 133.967393,251.310076 135.640156,255.82866 C139.070772,265.091859 146.042121,268.655319 155.569163,265.831954 C158.156891,265.064855 160.586452,263.740684 163.050049,262.590535 C164.365435,261.977456 165.550685,261.078339 166.880086,260.503265 C169.798162,259.240102 171.279724,260.096212 171.830304,263.288625 C171.983465,264.17774 171.907385,265.10486 172.004488,266.007977 C173.526091,280.060795 185.192389,289.237982 199.159111,287.21772 C206.514865,286.153583 212.778467,282.849155 216.656555,276.369317 C219.438487,271.722716 221.512674,266.644059 223.779063,261.70142 C226.041447,256.766782 227.344821,255.937674 232.022752,258.573015 C242.837152,264.663803 262.277644,258.340985 264.046508,242.024875 C264.513,237.719318 264.521008,233.358754 264.631124,229.021193 C264.736235,224.809648 264.993506,224.629625 268.983712,223.621494 C271.77065,222.918404 274.579611,221.982282 277.11128,220.64611 C282.947431,217.567711 284.974568,213.30616 284.06561,206.648299 C283.722248,204.134974 283.139634,201.647652 282.562025,199.173332 C280.853224,191.855386 280.924299,191.819381 287.530262,188.314928 C290.988908,186.47969 293.421472,183.013242 293.589649,179.104736 C293.795867,174.308116 290.301183,172.157838 286.469144,170.752656 C284.441006,170.00756 282.233679,169.752527 280.110441,169.262463 C276.896053,168.521367 276.500636,167.783272 277.616813,164.792885 C278.200428,163.227683 279.04732,160.571339 279.947269,157.979004 C276.733882,158.297045 273.789778,158.844116 272.343254,159.910254 C267.486134,163.486716 263.420849,162.500589 259.370579,158.852117 C259.313519,158.80011 259.246449,158.753104 259.179378,158.717099 C254.584534,156.277784 250.933686,156.915866 247.795378,160.916384 C246.568084,162.480586 245.475933,164.197808 244.611021,165.985039 C243.278617,168.734395 241.25148,169.487492 238.363436,168.980427 C235.508426,168.478362 232.610372,167.987298 229.72433,167.947293 C224.477799,167.874284 219.722787,169.674517 217.460402,174.584152 C214.885688,180.168874 211.148749,183.168262 204.966232,182.990239 C204.608856,182.979238 204.247475,183.097253 203.889097,183.161261 C199.301261,183.969366 196.518327,186.811733 195.007735,191.065283 C194.892614,191.388325 194.786502,191.716368 194.674384,192.078414 C194.222908,193.537603 192.250829,193.963658 191.368899,192.716497 C191.253778,192.554476 191.172693,192.389455 191.134652,192.215432 C190.883388,191.084286 191.210733,189.674103 191.693241,188.556959 C194.122802,182.927231 198.362271,179.698813 204.554799,178.968719 C207.695109,178.597671 210.908496,178.689683 212.664347,174.793179 C217.102024,164.941904 225.956358,162.825631 234.921808,164.216811 C236.688671,164.490846 238.455533,164.75388 240.166336,165.013914 C243.497848,160.632347 245.35881,155.425674 251.163929,153.805464 C257.221314,152.115245 261.5719,155.755716 265.155677,157.583953 C270.861692,156.340792 275.402478,155.351664 280.302644,154.283526 C280.986364,150.741068 281.409811,146.731549 282.588053,142.957061 C283.761289,139.196574 285.346961,135.45409 287.446174,132.142662 C288.402181,130.632467 291.011932,129.461315 292.898921,129.41931 C295.486649,129.363302 296.898137,131.776615 297.477748,134.169924 C297.986284,136.271196 298.069371,138.475481 298.329646,140.63576 C298.44777,141.621888 298.537865,142.610016 298.640974,143.597144 C301.623117,140.582754 303.648252,137.243322 305.674388,133.90389 C308.310166,129.558328 310.599579,124.947731 313.668815,120.933212 C317.399747,116.888689 321.281839,116.039579 324.521254,122.261384 C324.67842,122.561423 325.001761,122.77345 324.931687,122.702441 C327.265146,122.396401 329.5926,121.401273 331.265363,122.062358 C333.028221,122.757448 334.166421,125.034743 335.596929,126.642951 C339.528073,125.989866 343.73751,126.829975 343.754528,132.840752 C343.760534,134.647986 343.550312,136.45522 343.444201,138.171442 C348.495525,142.620017 349.829932,145.220353 349.048108,151.629182 C348.660699,154.797592 347.773764,157.902994 346.332245,160.752363 C340.400993,172.477879 330.216257,178.181617 317.568926,179.963847 C311.812858,180.774952 306.030763,181.409034 300.245665,181.99411 C298.21953,182.199137 296.970213,183.029244 296.05725,184.873483 C294.319419,188.385937 291.785748,191.158295 287.978735,192.608483 C287.1869,192.909522 286.573253,193.680622 285.730365,194.352709 C286.021672,195.823899 286.251915,197.237082 286.583264,198.627262 C288.162929,205.251118 289.229053,211.828969 285.995645,218.339811 C285.559185,219.217925 285.740376,220.476088 285.846487,221.536225 C286.839534,231.408502 281.224615,240.043619 271.281134,242.224901 C271.00184,242.286909 270.80263,242.710964 270.409215,243.136019 C270.059847,251.80914 267.459105,259.802174 260.136387,265.419901 C252.907767,270.965618 245.035469,274.203037 237.885933,270.759592 C234.519384,273.99201 231.957684,276.450328 229.40199,278.902645 C233.110899,281.747013 238.468547,282.840154 244.229619,281.271951 C247.185735,280.467847 250.141851,279.333701 252.789642,277.813504 C256.039068,275.949263 258.609777,276.055277 261.779118,278.204555 C268.824545,282.985173 276.666811,284.499369 285.18479,283.125191 C297.351615,281.161937 308.117964,276.362316 317.514869,268.399286 C318.36076,267.682194 319.253702,267.019108 320.150647,266.385026 C322.185792,264.94584 324.982741,265.267881 326.638486,267.131122 C329.133115,269.937485 331.389494,272.529821 333.945188,274.783112 C344.884719,284.423359 358.419986,284.224333 369.103248,274.444068 C371.16442,272.556824 373.017373,272.055759 375.355838,273.803985 C378.495147,276.149289 381.790621,276.781371 385.620658,274.366058 C378.063693,264.625798 379.599311,254.5875 384.299266,244.250163 C378.073703,237.514291 376.689244,228.985188 376.819381,220.42008 C376.918485,213.970246 378.638298,207.529413 379.829554,201.119584 C379.907636,200.69953 380.001736,200.255472 380.151894,199.835418 C380.876658,197.815156 383.948896,198.674268 383.635566,200.797542 C383.524449,201.55464 383.420339,202.315739 383.338253,203.078837 C382.893784,207.19337 381.848682,211.37491 382.231085,215.416433 C382.88978,222.389335 386.521608,227.973057 392.143534,232.334621 C399.219993,237.827332 398.978739,237.870338 395.840431,246.445447 C394.228732,250.848016 392.895327,255.44161 392.184577,260.064208 C390.96329,268.003235 394.196698,273.341926 401.49339,276.641352 C403.148134,277.389449 404.86194,278.006529 406.54071,278.699619 C412.177652,281.024919 412.581077,281.752013 410.163529,287.147711 C405.34545,297.905103 404.955039,308.93653 407.847087,320.197986 C410.740137,331.463444 416.274971,339.980545 429.369774,340.446605 C432.095647,340.543618 433.277893,341.964802 433.125733,344.65715 C433.002603,346.834432 432.865459,349.020715 432.938536,351.196996 C433.212825,359.263039 438.145024,365.235812 446.003307,367.247072 C452.277921,368.85328 457.90285,367.246072 463.08231,363.760621 C465.793167,361.935385 468.2908,359.796108 470.906557,357.828854 C471.937644,357.052753 472.994759,356.402669 474.04887,356.909735 C474.766626,357.25578 475.172054,358.059884 475.114994,358.853986 C475.002875,360.446192 474.297131,361.97639 474.044866,364.113667 C477.668685,362.099406 479.028118,361.850374 482.447723,356.669704 C483.862214,354.527427 485.362796,353.332272 488.172758,353.423284 C494.372292,353.62531 495.995003,352.447158 497.207281,346.496388 C497.861971,343.278972 499.430624,341.811782 502.47183,341.223706 C504.962455,340.742644 507.437064,340.067556 509.828585,339.223447 C517.215371,336.61711 521.907317,331.36043 524.303843,323.999478 C527.961699,312.758024 521.874282,300.491437 511.084909,297.277022 C506.284849,295.846837 505.011507,291.399261 508.301976,287.626773 C512.791708,282.478107 513.217157,278.232558 509.850608,272.209779 C505.632161,264.664803 497.816923,260.973326 489.150789,262.379508 C488.73435,262.447517 488.31691,262.504524 487.883453,262.556531 C485.591036,262.829566 483.817166,260.580275 484.601994,258.410994 C484.808211,257.839921 485.004418,257.29385 485.199624,256.747779 C487.038562,251.580111 488.544149,246.47145 487.137667,240.799716 C484.785187,231.31649 479.992134,223.674501 471.252923,219.054904 C467.588059,217.117653 466.482895,214.61733 466.927363,210.708824 C467.186636,208.419528 466.954392,205.96221 466.404812,203.710919 C464.347644,195.28983 458.312282,190.636228 450.575126,187.816863 C443.821007,185.355545 443.432597,184.821476 446.213529,178.649677 C450.442987,169.267464 450.209741,159.828243 446.164477,150.630053 C442.2964,141.833915 436.333114,134.670989 426.351592,132.687732 C421.98499,131.82062 417.301052,132.168665 412.795303,132.471704 C407.197402,132.848753 406.763945,132.762742 405.379486,127.217025 C405.202299,126.511934 405.026113,125.806842 404.875955,125.09575 C403.655669,119.294 400.305137,115.341489 394.722252,113.252219 C388.66787,110.985925 382.484352,109.981795 376.00252,110.723891 C369.563733,111.459987 363.302133,112.765156 357.378889,115.601522 C355.859287,116.328616 353.957282,115.89356 353.104383,114.442372 C353.040316,114.333358 352.983256,114.221344 352.937207,114.105329 C350.571714,108.355585 345.917808,105.933272 340.136714,105.3792 C336.722115,105.052158 333.193395,104.807126 329.831852,105.305191 C325.737536,105.913269 321.720302,107.044416 317.461813,108.113554 C315.502748,108.605617 313.781934,106.729375 314.447636,104.825128 C314.586782,104.427077 314.758964,104.018024 314.942157,103.60297 C316.111389,100.946627 317.402751,98.2952839 318.145533,95.5109237 C319.615083,89.9902096 316.559862,86.51376 310.9039,87.1728452 C309.153055,87.3768716 307.372177,88.0579597 305.801522,88.8900673 C303.630233,90.0382158 301.542032,92.7225631 299.609995,92.5545413 C297.620899,92.3815189 295.88607,89.5441519 293.99808,87.9219421 C291.167096,85.4896275 288.965775,85.5346333 286.376046,88.1799755 Z M317.728093,122.497414 C316.487786,123.9306 315.246478,125.278774 314.318499,126.816973 C311.787832,131.014516 309.61354,135.425087 307.101892,139.635631 C305.152838,142.903054 303.021591,146.076464 300.76221,149.13786 C299.54793,150.783073 298.206516,153.509426 295.764942,152.07124 C294.584698,151.37615 294.31141,148.664799 294.160251,146.822561 C293.954033,144.305235 294.365467,141.742904 294.307406,139.205576 C294.265362,137.352336 294.257353,135.388082 292.289279,133.834881 C291.805769,133.453832 291.079003,133.580848 290.730636,134.088914 C284.229783,143.583142 285.242851,154.886604 282.406862,165.557984 C290.392279,166.817147 296.112308,170.412612 298.308624,178.025597 C303.790402,177.515531 308.884772,177.2765 313.897055,176.5034 C318.185576,175.841314 322.546172,175.006206 326.581426,173.476008 C336.206571,169.827536 342.458161,162.842633 345.066911,152.824337 C345.953846,149.415896 345.637513,146.141473 342.469173,142.864049 C341.060688,145.045331 339.880444,146.786556 338.788293,148.580788 C336.239606,152.76933 333.830066,157.046883 331.164256,161.157415 C330.481537,162.210551 329.606614,163.112668 328.531481,163.072663 C327.139013,163.021656 326.426262,161.339438 327.206084,160.186289 C328.007929,159.002136 328.799764,157.842986 329.447447,156.609827 C332.563732,150.665058 335.756097,144.748292 338.553045,138.652504 C339.506049,136.577236 340.685292,134.21593 339.736292,131.802618 C339.325859,130.757483 338.014477,130.404437 337.132548,131.102527 C334.887181,132.877757 333.639866,135.289069 332.311466,137.635372 C329.088068,143.324108 325.87368,149.017845 322.60123,154.677577 C321.78537,156.088759 320.965506,157.611956 319.770246,158.651091 C319.6321,158.771106 319.480941,158.87612 319.319771,158.969132 C317.620981,159.95726 315.384623,158.464067 315.820082,156.549819 C315.850114,156.421802 315.889155,156.297786 315.940209,156.178771 C316.717028,154.379538 318.192583,152.895346 319.232679,151.191126 C322.351967,146.079465 325.514301,140.987806 328.43538,135.76213 C329.76378,133.385823 330.872949,130.846494 330.451505,127.632079 C330.322369,126.649951 329.165149,126.183891 328.374316,126.780968 C325.430212,129.006256 324.096807,132.002644 322.537163,134.771002 C319.505967,140.149698 316.609915,145.605403 313.628772,151.012103 C313.016127,152.124246 312.49758,153.46442 311.541572,154.15951 C310.538515,154.888604 308.817701,155.583694 307.846678,155.197644 C306.007739,154.466549 306.793567,152.816336 307.62144,151.543171 C309.307217,148.953836 311.281298,146.522522 312.700794,143.79817 C315.374613,138.668506 317.911287,133.449831 320.146643,128.117141 C320.829362,126.48693 321.330891,124.488672 319.949435,122.602428 C319.408865,121.864332 318.326724,121.805325 317.728093,122.497414 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
                <g id="Face" transform="translate(324.000000, 111.000000) scale(1 1)" fill="#000">
                    <g id="Face/Glasses 2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                        <path d="M30.8272308,41.3801 C31.3570769,43.5651 30.7130769,45.3391 29.0535385,46.7271 C26.9933846,48.4501 24.2892308,48.4231 21.5366154,46.6521 C19.2093846,45.1551 18.3607692,43.8481 18.7344615,41.6601 C18.8841538,40.7761 19.9524615,40.3191 20.7601538,40.8061 C21.9469231,41.5201 23.1250769,42.0451 24.3796923,42.8761 C24.9698462,43.2681 25.7883077,43.2061 26.274,42.7061 C27.1473846,41.8081 27.8786154,41.2671 28.5732308,40.6901 C29.3583077,40.0371 30.5946154,40.4251 30.8272308,41.3801 Z M57.7401611,5.69244571 C59.78055,12.01096 59.4997722,18.2595314 56.3214944,24.20056 C54.3349389,27.9116457 51.2527167,30.6188457 46.7761056,30.2537029 C45.3226056,30.1343886 43.2294389,30.2670743 43.2294389,30.2670743 C43.2294389,30.2670743 40.8058833,32.5649029 39.7767167,33.9791886 C38.8362167,35.2721029 37.9168278,36.9990743 35.40355,36.6935886 C34.64355,36.6010171 34.2424389,35.7514171 34.6351056,35.1106171 C37.0639389,31.1423886 39.3471056,27.4127886 42.0641056,22.9724457 C42.4356611,22.3666171 41.9321611,21.6106171 41.2122722,21.6918743 C37.0692167,22.1578171 33.0148278,22.51576 30.1838278,23.13496 C30.1838278,23.13496 29.8259944,24.6809029 29.22855,25.7526743 C28.2278833,27.5465029 27.7043278,29.6129029 26.5854389,31.3193029 C23.2319389,36.4302743 17.7008278,37.6223886 12.6932722,34.1499314 C5.87543889,29.4226171 3.77593889,22.28536 3.02227222,14.6430743 C2.81116111,12.5057029 4.14749444,11.1490171 6.24805,10.5894743 C11.78655,9.11450286 17.2796611,7.41016 22.89205,6.28078857 C26.9876056,5.45690286 28.9942167,7.03267429 29.8523833,11.0513029 C30.3527167,13.3964457 30.3548278,15.8660457 30.6398278,18.4631886 C31.6774389,18.1268457 34.2572167,18.0311886 34.2572167,18.0311886 C33.8919944,14.8652457 33.5742722,12.1086743 33.2227722,9.06101714 C33.0707722,7.74033143 33.6998833,6.37747429 34.9296056,5.80353143 C35.04255,5.75107429 35.1576056,5.70787429 35.2758278,5.67496 C40.4322167,4.25656 45.57805,2.74970286 50.8178278,1.69336 C54.9988833,0.849931429 56.4967167,1.84044571 57.7401611,5.69244571 Z M25.5921611,9.95587429 C18.7584944,10.2654743 13.05005,12.2382743 6.78638333,14.4641029 C7.39227222,20.7095886 8.99777222,25.9943886 13.4226611,30.0428457 C17.0917722,33.3990743 21.20105,33.0370171 23.5897722,29.0214743 C27.1163833,23.09176 27.2757722,16.77016 25.5921611,9.95587429 Z M53.0102167,4.89736 C47.6078833,6.22113143 42.5317167,7.46467429 37.0312167,8.81313143 C37.5347167,11.9646743 38.0223833,15.2201029 38.4171611,17.6886743 C42.5686611,17.8563314 44.4507167,18.0178171 45.8746611,19.5308457 C47.1392167,20.87416 46.5428278,24.2478743 45.4556056,26.3194171 C49.01705,26.6495886 52.3093278,24.1954171 53.4799389,21.42136 C55.72405,16.1036457 55.98055,10.7489029 53.0102167,4.89736 Z M22.9646307,16.3637855 C23.0306909,17.5429088 23.0317738,18.7007361 22.720966,19.7610504 C22.5953434,20.1880902 22.0744425,20.4918378 21.2893009,20.8180021 C19.6486257,21.4994726 17.8660835,20.1768818 18.0079505,18.3532758 C18.1194947,16.9286887 18.2678594,15.7775864 18.7291978,14.7822808 C18.9068022,14.3978328 19.4450304,14.1915983 20.1132129,14.0503725 C21.5448781,13.746625 22.8790773,14.8540145 22.9646307,16.3637855 Z M45.1267884,11.150925 C47.7910746,12.9513347 47.3696056,14.43246 45.386981,16.8625697 C44.318256,17.0803532 44.0290337,17.0074302 43.841953,16.8635551 C41.7367584,15.2562928 41.3722737,13.3642366 43.0796532,11.3775776 C43.3936046,11.0100063 44.7042442,10.8651457 45.1267884,11.150925 Z" id="Stroke" fill="#000"/>
                    </g>
                </g>
            </g>
        </g>
        <g id="Super-idea" transform="translate(1060.869072, 426.982785) scale(-1, 1) rotate(20.000000) translate(-1060.869072, -426.982785) translate(762.369072, 107.482785)">
            <path d="M459.553293,195.340804 C423.530644,141.964608 372.411844,113.744406 307.61648,111.463089 L307.562892,111.463089 C292.267979,111.516778 278.364398,113.258272 265.058076,116.787139 C204.994996,132.71634 162.461925,170.645564 138.640716,229.521522 C123.88168,265.996248 121.028864,304.254443 130.161188,343.23403 C144.89002,406.109371 181.648283,451.326708 239.412936,477.628729 C244.856475,480.107233 249.058728,483.958238 252.260351,489.402358 C254.120333,492.563208 255.371363,495.341398 255.982263,497.663715 C256.408042,499.28514 257.126118,500.699615 257.819835,502.067234 C258.590524,503.587137 259.317369,505.021136 259.583359,506.612299 C259.972113,508.952187 260.561578,511.270599 261.130583,513.51287 C261.761944,515.999184 262.415714,518.568473 262.772316,521.099691 C263.767099,528.175973 264.643015,535.395753 265.49165,542.377346 C266.134703,547.680897 266.800165,553.164065 267.514343,558.552543 C267.92843,561.679227 267.815409,563.991782 266.010964,566.550333 C262.652474,571.312108 263.162045,575.204112 267.66244,579.172257 C269.351915,580.662874 269.777694,582.206205 269.917997,584.213218 C270.300905,589.718837 270.815348,595.326954 271.268408,600.15218 C271.701981,604.7851 273.588269,606.9893 278.22507,608.285658 C279.096114,608.528726 279.991516,609.143715 280.751488,609.711848 C281.749194,610.457645 282.743977,611.28544 283.705633,612.085902 C285.577307,613.644851 287.513285,615.255538 289.695768,616.474779 C297.977508,621.098913 304.775356,624.564329 311.088965,627.381566 C318.319412,630.606844 326.102299,633.663243 334.882893,636.7255 C338.204358,637.884218 341.519003,638.463089 344.817084,638.463089 C350.615277,638.463089 356.357933,636.673762 361.978799,633.103896 L362.889791,632.530882 C363.938162,631.873917 365.020634,631.193524 366.05439,630.439918 C380.335034,620.020241 391.378979,606.130267 398.878338,589.153632 C400.668169,585.103488 400.913698,580.904965 399.629541,576.318902 C398.04237,570.654167 396.645191,564.819577 395.29478,559.17827 C394.731622,556.823739 394.167489,554.470185 393.588742,552.119559 C393.349058,551.144361 393.126913,549.456557 393.140553,548.434503 C393.277933,548.231459 393.458183,547.947393 393.675456,547.57059 C395.973883,543.566326 395.69815,541.88145 392.268535,538.987096 C392.01716,538.775266 391.726812,538.580031 391.436464,538.390653 C391.272777,538.283274 391.046735,538.134896 390.912278,538.024588 C390.648237,537.146032 390.149384,535.378182 390.008107,534.371747 C389.734322,532.420376 389.470281,530.51098 389.245212,528.600609 C387.980542,517.837319 386.879558,505.331559 389.855138,493.112794 C393.560486,477.895224 400.684732,467.211004 411.634167,460.450026 C413.60717,459.231761 415.394078,457.85438 417.357337,456.310074 C464.717202,419.065147 489.76898,369.295913 491.819928,308.38561 C493.207364,267.155942 482.350489,229.122267 459.553293,195.340804 L459.553293,195.340804 Z" id="Path-Copy" fill-opacity=".666" fill="#FFF"/>
            <path d="M265.338111,297.167001 C250.976378,295.497361 237.460999,283.028241 235.904488,269.625255 C235.657392,267.501855 235.772185,265.252574 236.228437,263.294089 C236.669124,261.398057 237.561199,260.096303 238.809327,259.531299 C239.287954,259.31369 239.814249,259.206349 240.378484,259.206349 C241.327956,259.206349 242.387356,259.511783 243.510963,260.117771 C244.999376,260.919901 246.562697,262.142613 248.157148,263.750776 C254.978557,270.628406 262.979024,286.331416 265.338111,297.167001 M356.990342,253.765137 C357.95927,250.87864 359.131517,248.440048 360.473035,246.516692 C361.687114,244.777768 363.146343,243.844878 364.693126,243.817554 C364.715501,243.817554 364.737875,243.816579 364.76025,243.816579 C366.328435,243.816579 367.911212,244.776793 369.226464,246.525475 C371.223662,249.18558 371.849185,252.339453 370.986294,255.406478 C369.774161,259.715731 368.355791,263.301895 366.649465,266.370872 C363.465428,272.098001 359.110115,276.995678 353.378264,281.284438 C353.210939,270.228317 354.344273,261.655675 356.990342,253.765137 M385.03186,486.394524 C384.902475,486.065671 384.776009,485.747551 384.690401,485.455779 C379.770853,468.756448 376.632538,451.32232 374.075969,436.292631 L371.928956,423.661524 C367.803229,399.398557 363.677502,375.134615 359.518699,350.877504 C356.157609,331.276064 354.031025,313.024192 353.019293,295.079707 C352.869479,292.41765 353.564072,290.826076 355.673145,288.999328 C360.375753,284.928177 364.817647,280.454985 368.877222,275.700755 C373.507842,270.276132 376.534283,264.202584 377.869964,257.647953 C378.917691,252.503392 378.272711,247.845769 375.952537,243.802917 C373.400832,239.356073 369.4113,236.758421 365.009292,236.674164 C360.762935,236.611071 356.845392,238.92183 354.261584,243.056409 C352.213799,246.335189 350.514283,250.388775 349.209733,255.106899 C346.716397,264.124518 345.645322,273.654446 346.026668,283.434185 C346.154107,286.689545 345.152103,288.407976 342.455448,289.564332 C341.601312,289.931243 340.805546,290.380123 340.037019,290.814366 C339.715016,290.99587 339.39204,291.17835 339.066145,291.353023 C328.323301,297.123089 315.699997,300.294527 299.340093,301.333783 C290.514676,301.895859 281.977213,301.211805 273.947562,299.303087 C273.35317,297.662721 272.777261,296.045776 272.209134,294.447371 C270.757688,290.367437 269.386985,286.51292 267.845066,282.653524 C264.079283,273.228009 259.994414,266.286951 254.991204,260.809633 C251.864563,257.386431 248.697063,254.861966 245.575285,253.307474 C242.393193,251.721754 239.018483,251.555864 236.075704,252.842004 C232.946144,254.209138 230.611378,257.029278 229.500418,260.784261 C229.08016,262.205066 228.846684,263.813229 228.767885,265.844901 C228.356383,276.410182 232.639706,285.703959 241.497227,293.46764 C246.293226,297.671504 252.220614,300.863434 259.619879,303.225912 C261.699766,303.889474 263.811757,304.506197 265.854678,305.102427 C266.554135,305.307351 267.249701,305.510323 267.942348,305.714271 C267.966669,305.782579 267.990016,305.847959 268.012391,305.912364 C268.187499,306.408084 268.325639,306.798415 268.420003,307.180939 C271.689648,320.368268 273.635287,334.034727 274.36782,348.959027 C275.151913,364.937221 274.989452,382.103973 273.870709,401.439988 C272.213025,430.090273 268.821777,455.32126 263.505319,478.57327 C261.700739,486.462832 260.133527,494.421679 258.970035,500.473759 L258.87178,500.951914 C258.613011,502.164867 258.259877,503.825725 259.13055,505.134309 C259.62377,505.873986 260.399107,506.347262 261.437105,506.538524 L262.16672,506.674164 L262.680368,506.137459 C263.392472,505.395831 263.872072,504.733244 264.378911,504.0326 C264.731072,503.546638 265.094906,503.044087 265.571587,502.467373 L265.804091,502.185359 L266.084263,500.594761 C266.490902,498.271317 266.91116,495.869806 267.477341,493.569781 C276.467164,457.064087 281.303049,417.986113 281.849774,377.419026 C282.07255,360.923644 281.861448,341.443207 278.612231,322.025222 C277.997409,318.358064 277.310599,314.730914 276.58293,310.891034 C276.35529,309.689791 276.123759,308.470983 275.891255,307.229731 C302.89283,310.556325 325.836775,306.843303 345.911875,295.898426 C345.99651,296.521003 346.06558,297.050878 346.122004,297.582703 C346.588957,302.00808 347.004351,306.52128 347.405153,310.887131 C348.281663,320.428769 349.187358,330.295357 350.70301,339.922868 C354.513544,364.142898 358.72877,388.709347 362.805856,412.466835 C364.265085,420.969217 365.724314,429.472575 367.168951,437.976908 C369.871443,453.88777 372.929014,470.814468 377.944871,487.44549 C378.025615,487.711891 378.094685,488.010494 378.167647,488.31788 C378.493541,489.69282 378.899207,491.404421 380.351626,492.191913 C381.216463,492.658359 382.255434,492.679827 383.436436,492.251439 C384.425794,491.891359 385.108713,491.304887 385.468656,490.50666 C386.096125,489.116106 385.505623,487.606502 385.03186,486.394524" id="Fill-4" fill="#000"/>
            <path d="M290.581427,608.473084 C294.907418,608.259302 299.235358,608.072852 303.563299,607.885427 C311.656021,607.535957 320.024476,607.175749 328.25068,606.634949 C347.69913,605.35909 363.25711,602.933299 377.215253,599.002248 C380.914756,597.959694 384.54703,596.344127 387.750602,594.916961 C387.9328,594.836915 388.108178,594.751012 388.27771,594.66218 C381.487657,606.46119 373.594671,615.729958 364.305481,622.842359 C362.511754,624.215835 360.642029,625.469242 358.723588,626.741196 C351.422016,631.583993 343.847633,632.590428 336.208946,629.728287 L333.710783,628.796042 C327.543322,626.496177 321.165407,624.117242 315.102199,621.369313 C309.437488,618.801 303.802982,615.861742 298.355545,613.0201 C296.532588,612.069307 294.709631,611.118514 292.882776,610.179435 L291.900659,609.678658 C291.232274,609.340902 290.464508,608.950433 289.648026,608.521892 C289.996833,608.501393 290.312514,608.48675 290.581427,608.473084 L290.581427,608.473084 Z M384.680512,484.928555 C384.484673,485.45276 384.288834,485.976965 384.088124,486.499218 C377.495858,483.758123 370.626885,483.290535 364.727363,483.205608 C335.262879,482.777068 311.716429,486.306912 290.623323,494.297869 C282.342556,497.436267 275.405381,498.422202 268.129141,497.484099 C264.683937,497.042869 263.264349,496.372238 262.165313,493.489597 C260.483633,489.081197 257.880243,485.030077 255.556484,481.647635 C252.604287,477.354424 248.603719,474.042267 243.664879,471.802925 C183.975939,444.738512 147.577202,398.189675 135.480015,333.451795 C128.558428,296.414793 133.027645,258.569519 148.404401,224.007117 C164.018893,188.911725 189.554909,160.390861 222.254144,141.530211 C248.675819,126.289213 276.529749,118.602822 305.239133,118.602822 C309.911009,118.602822 314.606269,118.806842 319.321015,119.213907 C377.583547,124.251939 423.413725,152.191003 455.537136,202.254065 C477.04238,235.769033 487.150975,274.61098 484.769731,314.581383 C482.396282,354.441479 467.841854,391.679573 442.680951,422.271877 C431.949764,435.319413 420.097132,446.201796 407.450427,454.617389 C397.59613,461.173371 390.465064,470.256666 385.650937,482.387575 C385.31577,483.230013 384.998141,484.079284 384.680512,484.928555 L384.680512,484.928555 Z M385.196903,559.202674 C381.768262,558.587685 378.530589,558.007838 375.205226,557.778437 C372.56871,557.596869 369.956551,557.478752 367.356085,557.42311 C374.024348,556.198012 380.148938,554.592206 385.85457,552.57836 C385.984155,552.53248 386.157585,552.459267 386.349526,552.371412 C386.630131,553.55551 386.913659,554.789394 387.20011,556.033039 C387.481689,557.258137 387.766191,558.494948 388.051667,559.70638 C387.087088,559.541407 386.13615,559.370576 385.196903,559.202674 L385.196903,559.202674 Z M285.036558,559.160699 L285.762428,558.692136 C286.123902,558.464687 286.234974,558.423688 286.22718,558.421735 C286.304151,558.466639 286.539937,558.47152 286.75234,558.516424 C287.144991,558.600375 287.682817,558.713611 288.460327,558.819038 C301.925463,560.750886 316.0356,561.41566 331.857622,560.826051 C317.483443,564.006425 303.802008,569.388069 290.922439,576.944629 C289.03128,578.054539 287.517183,578.404009 285.856937,578.106276 C285.247986,577.996944 284.639034,577.894446 284.029108,577.790972 C282.585163,577.547904 281.091526,577.296052 279.664144,576.967081 C278.826227,576.771846 277.963951,576.453614 277.051011,576.115858 C276.848352,576.040692 276.64277,575.964551 276.434265,575.888409 C276.410881,575.71465 276.386523,575.542844 276.362165,575.370061 C276.229657,574.42903 276.104944,573.540712 276.083509,572.672894 C276.032844,570.618048 275.666499,567.653409 275.495018,566.355098 C277.925953,563.754572 281.849549,561.220425 285.036558,559.160699 L285.036558,559.160699 Z M266.501048,504.45886 C267.710182,504.58381 268.880343,504.703879 270.456797,504.767331 C269.360684,505.691767 268.310365,506.678678 267.28148,507.749541 L266.372437,504.446169 C266.415308,504.450074 266.458178,504.454955 266.501048,504.45886 L266.501048,504.45886 Z M376.759271,492.490972 C374.996721,493.126461 373.163047,493.488621 371.222196,493.873233 C370.468071,494.022588 369.712971,494.170966 368.964691,494.336916 C364.426297,495.33847 359.593658,496.363452 354.783428,496.909133 C336.618161,498.965931 318.656527,498.806815 302.746816,498.12935 C329.497813,490.596219 354.352778,488.483779 378.467258,491.675867 C377.822256,492.029241 377.142179,492.354307 376.759271,492.490972 L376.759271,492.490972 Z M297.002211,520.722888 L296.570587,520.856624 C296.05517,521.012811 295.477397,521.188523 294.923007,521.481375 C284.356481,524.348396 280.410475,522.540523 273.895181,519.549527 C271.938741,518.651447 270.846526,517.68894 270.64679,516.689339 C270.439259,515.652642 271.123234,514.272333 272.679226,512.586481 C277.358897,507.517212 282.134051,505.230037 288.086187,505.230037 C288.293718,505.230037 288.503197,505.232965 288.713651,505.238822 C306.413192,505.703481 323.92274,505.649791 340.757083,505.073849 C356.208862,504.546715 369.092327,503.187882 381.550014,498.30799 C381.407763,500.340383 381.264537,502.446966 381.123261,504.542811 C380.964446,506.889532 380.806606,509.223563 380.64974,511.428739 C354.356675,507.94868 326.976266,510.994342 297.002211,520.722888 L297.002211,520.722888 Z M304.067023,525.795086 C322.269315,519.031179 361.853112,514.983963 376.368567,518.717827 C374.48033,519.90583 373.605389,520.137184 371.901299,520.5872 C366.679905,521.964581 360.59721,523.505959 354.496003,524.352301 C336.271302,526.883519 318.102138,526.475479 304.067023,525.795086 L304.067023,525.795086 Z M376.481589,534.036919 C370.72724,533.665973 365.291496,533.318456 359.746627,533.484405 C332.602978,534.309272 309.063349,539.510325 287.784147,549.381392 C284.008648,551.132647 278.640131,550.2824 275.561272,547.441735 C273.699342,545.725622 273.524938,544.531761 274.868529,542.712174 C276.434265,540.590949 278.520289,538.489247 280.899585,536.635493 C285.135939,533.334074 289.306039,531.786839 294.116268,531.786839 C295.055515,531.786839 296.018146,531.84541 297.01098,531.96255 C316.642603,534.263392 336.814,533.95004 358.680718,531.001996 C365.029403,530.146868 373.249761,528.562538 381.589961,523.902285 L382.778635,534.416651 C380.652663,534.305367 378.538383,534.169679 376.481589,534.036919 L376.481589,534.036919 Z M383.59804,545.853501 C355.201413,554.648825 324.396261,554.598063 297.311072,552.983472 C297.351019,552.967854 297.39194,552.951259 297.434811,552.932711 C311.311111,547.306047 326.804786,543.580968 344.801495,541.54467 C353.988381,540.505045 366.091414,539.557181 378.240241,541.462672 C378.793656,541.549551 379.345122,541.655954 379.895614,541.761381 C380.562051,541.888283 381.228488,542.016162 381.899796,542.11378 C383.958539,542.416393 384.447649,543.221737 384.570413,544.423406 C384.634719,545.046205 384.216734,545.661194 383.59804,545.853501 L383.59804,545.853501 Z M384.328781,567.474769 C382.709458,568.341611 381.035572,569.238715 379.349994,569.939608 C369.274526,574.135202 358.779125,575.833744 349.840691,576.942677 C333.337618,578.986784 316.863774,579.742343 300.864425,579.192757 C301.829005,578.736884 302.689331,578.335677 303.219363,578.090657 C318.317463,571.103206 334.283685,566.722139 350.674711,565.07143 C358.970092,564.236801 367.879296,563.644264 376.829422,564.948432 C379.127848,565.284236 381.465248,565.794775 383.725676,566.289695 C384.389189,566.436121 385.059523,566.582547 385.735703,566.725068 C385.262182,566.974968 384.793533,567.225845 384.328781,567.474769 L384.328781,567.474769 Z M278.157841,529.491855 C275.289436,531.095708 273.576577,532.59023 271.951408,534.177488 L271.05698,527.173443 C273.063109,528.261876 275.464814,529.045744 278.157841,529.491855 L278.157841,529.491855 Z M385.99195,584.455309 C379.571165,583.018381 373.277042,584.072649 367.187527,585.093726 L366.994611,585.12594 C355.910719,586.98067 344.703089,589.17706 333.864726,591.300238 C329.953796,592.066534 326.042866,592.833807 322.129012,593.586436 C318.002757,594.380066 313.877476,595.185409 309.753169,595.990752 C301.693574,597.563368 293.35922,599.190649 285.149579,600.703718 C283.324673,601.040621 283.116168,601.081497 279.077602,601.012189 C279.214007,600.65979 279.363078,600.306415 279.499483,599.983302 C279.60471,599.735354 279.704091,599.501072 279.789831,599.286314 C281.604994,594.739297 284.792977,590.7282 289.824378,586.665366 C290.377793,586.218278 291.607388,585.938116 292.813599,585.997663 C308.915252,586.721008 323.205639,586.51894 336.499294,585.382674 C349.24538,584.292288 362.792359,582.883669 375.966172,578.754455 C380.891372,577.210148 386.069896,575.380799 390.934688,571.622531 C391.108117,572.299995 391.280572,572.960865 391.44913,573.609044 C392.244177,576.66642 392.931074,579.305993 393.351981,582.018779 C393.563409,583.385423 393.211679,585.261628 392.456579,586.799102 C392.114592,587.493161 391.770656,588.180387 391.423797,588.858828 C391.4199,588.8354 391.415028,588.810019 391.410157,588.785615 C390.768078,585.523243 387.778857,584.85554 385.99195,584.455309 L385.99195,584.455309 Z M373.626824,592.068487 C359.966824,596.225034 345.789458,597.970432 331.686141,599.041294 C345.411421,595.84335 359.424126,592.987066 373.626824,592.068487 L373.626824,592.068487 Z M276.279348,556.021325 L274.584027,557.284493 L274.256654,555.176934 L276.279348,556.021325 Z M281.301006,584.686662 L277.338436,589.183894 L277.056857,583.696822 L281.301006,584.686662 Z M459.553293,195.340804 C423.530644,141.964608 372.411844,113.744406 307.61648,111.463089 L307.562892,111.463089 C292.267979,111.516778 278.364398,113.258272 265.058076,116.787139 C204.994996,132.71634 162.461925,170.645564 138.640716,229.521522 C123.88168,265.996248 121.028864,304.254443 130.161188,343.23403 C144.89002,406.109371 181.648283,451.326708 239.412936,477.628729 C244.856475,480.107233 249.058728,483.958238 252.260351,489.402358 C254.120333,492.563208 255.371363,495.341398 255.982263,497.663715 C256.408042,499.28514 257.126118,500.699615 257.819835,502.067234 C258.590524,503.587137 259.317369,505.021136 259.583359,506.612299 C259.972113,508.952187 260.561578,511.270599 261.130583,513.51287 C261.761944,515.999184 262.415714,518.568473 262.772316,521.099691 C263.767099,528.175973 264.643015,535.395753 265.49165,542.377346 C266.134703,547.680897 266.800165,553.164065 267.514343,558.552543 C267.92843,561.679227 267.815409,563.991782 266.010964,566.550333 C262.652474,571.312108 263.162045,575.204112 267.66244,579.172257 C269.351915,580.662874 269.777694,582.206205 269.917997,584.213218 C270.300905,589.718837 270.815348,595.326954 271.268408,600.15218 C271.701981,604.7851 273.588269,606.9893 278.22507,608.285658 C279.096114,608.528726 279.991516,609.143715 280.751488,609.711848 C281.749194,610.457645 282.743977,611.28544 283.705633,612.085902 C285.577307,613.644851 287.513285,615.255538 289.695768,616.474779 C297.977508,621.098913 304.775356,624.564329 311.088965,627.381566 C318.319412,630.606844 326.102299,633.663243 334.882893,636.7255 C338.204358,637.884218 341.519003,638.463089 344.817084,638.463089 C350.615277,638.463089 356.357933,636.673762 361.978799,633.103896 L362.889791,632.530882 C363.938162,631.873917 365.020634,631.193524 366.05439,630.439918 C380.335034,620.020241 391.378979,606.130267 398.878338,589.153632 C400.668169,585.103488 400.913698,580.904965 399.629541,576.318902 C398.04237,570.654167 396.645191,564.819577 395.29478,559.17827 C394.731622,556.823739 394.167489,554.470185 393.588742,552.119559 C393.349058,551.144361 393.126913,549.456557 393.140553,548.434503 C393.277933,548.231459 393.458183,547.947393 393.675456,547.57059 C395.973883,543.566326 395.69815,541.88145 392.268535,538.987096 C392.01716,538.775266 391.726812,538.580031 391.436464,538.390653 C391.272777,538.283274 391.046735,538.134896 390.912278,538.024588 C390.648237,537.146032 390.149384,535.378182 390.008107,534.371747 C389.734322,532.420376 389.470281,530.51098 389.245212,528.600609 C387.980542,517.837319 386.879558,505.331559 389.855138,493.112794 C393.560486,477.895224 400.684732,467.211004 411.634167,460.450026 C413.60717,459.231761 415.394078,457.85438 417.357337,456.310074 C464.717202,419.065147 489.76898,369.295913 491.819928,308.38561 C493.207364,267.155942 482.350489,229.122267 459.553293,195.340804 L459.553293,195.340804 Z" id="Fill-1" fill="#000"/>
            <path d="M592.766754,119.0029 C584.470809,89.6766327 565.912474,71.6351038 537.605793,65.3818521 C524.559555,62.498213 510.217958,62.3108497 493.759701,64.807076 C488.479754,65.6082494 483.266136,66.6494822 478.262233,67.9024746 C472.095629,69.4482222 466.058757,71.3628414 460.318412,73.5926604 C454.591722,75.818576 448.994764,78.4221458 443.683604,81.331157 C438.389025,84.2313855 433.231006,87.517075 428.350981,91.0974714 C423.490464,94.664206 418.769433,98.6242084 414.316642,102.868183 C409.882384,107.095569 405.593463,111.721102 401.569857,116.616945 C399.565369,119.056572 397.581366,121.616228 395.675396,124.225653 C394.725337,125.527438 393.773327,126.868257 392.847654,128.212004 C392.399936,128.86192 391.96685,129.517692 391.541567,130.18127 C391.377696,130.436943 391.187489,130.676027 390.999233,130.916086 C390.677345,131.325943 390.312538,131.791424 390.02674,132.341804 C389.902862,132.580888 389.795565,132.82485 389.688269,133.069789 C389.595604,133.280573 389.50489,133.491356 389.394668,133.693357 C389.371258,133.737271 389.34102,133.783136 389.309806,133.831928 C389.120575,134.128587 388.833802,134.577478 388.802588,135.162989 C388.724555,136.611151 389.554637,137.930501 390.917298,138.524795 C392.220459,139.093716 393.685539,138.818526 394.659008,137.809496 C394.663885,137.804617 394.984798,137.431842 395.23158,137.145917 L395.452024,136.887317 C396.495724,135.768992 397.290691,134.482821 398.060297,133.239587 C398.42803,132.64627 398.794788,132.051976 399.188857,131.481104 C415.440325,107.92309 436.288947,90.9335285 461.153371,80.98473 C487.20293,70.5606921 511.12705,67.6321639 534.294243,72.0273958 C556.99031,76.3338252 573.024259,89.0354988 581.949349,109.776231 C589.288407,126.830199 591.135852,145.513838 587.440963,165.307019 C583.037918,188.890404 573.231048,210.737751 557.460462,232.096197 C542.518983,252.332414 523.868957,270.792583 500.444253,288.530623 C457.822312,320.802983 410.906575,348.349299 357.013659,372.74264 C313.191953,392.579735 268.013438,407.592223 229.599458,419.663497 C227.133597,420.438322 224.66676,421.21022 222.198948,421.981142 C202.99391,427.986528 183.135339,434.195866 164.143917,442.007552 C133.099234,454.778511 108.564501,471.793445 89.1409685,494.025277 C81.4956287,502.774755 75.8625798,511.267584 71.9209087,519.988762 C70.2704985,523.640396 69.2394798,527.579906 68.2426009,531.391579 C68.0299594,532.203486 67.8173179,533.015394 67.5997993,533.823399 C67.0516134,535.854144 67.129647,537.380375 67.8378017,538.491869 C68.1957807,539.052983 68.9000338,539.771209 70.2227029,539.999558 C70.4850908,540.045423 70.7357737,540.066892 70.9776778,540.066892 C71.6585207,540.066892 72.2554776,539.887335 72.7578188,539.528222 C73.9127157,538.703628 74.1468165,537.245707 74.3340971,536.073711 C74.3809172,535.782907 74.4257865,535.495031 74.4862625,535.227648 C76.2732315,527.359363 77.7792795,523.744812 79.7359715,520.228822 L80.7367521,518.649895 C84.966172,511.973124 89.3389787,505.069956 94.4257925,498.996261 C105.817719,485.393877 119.649171,473.811504 136.710238,463.58654 C151.147425,454.935623 167.193079,447.688955 185.761169,441.432776 C200.275415,436.543763 215.112524,431.792346 229.460948,427.197065 C238.824002,424.199251 248.506018,421.097997 258.011484,417.986985 C327.378457,395.279914 382.480893,371.290574 431.420627,342.490291 C462.600894,324.140393 495.514481,303.566531 524.971182,277.497654 C548.916761,256.307054 565.593512,236.06791 577.455591,213.80485 C590.000464,190.259523 596.136829,168.332156 596.214863,146.768782 C596.430431,137.196662 595.270657,127.853866 592.766754,119.0029" id="Fill-6" fill="#3AADAA"/>
            <path d="M102.748794,333.381612 C110.102245,333.381612 117.663675,332.480225 125.397932,330.674524 C127.217987,330.249195 128.838851,329.637539 130.215609,328.855166 C131.418564,328.173273 132.021018,326.182222 131.670481,325.031099 C131.300416,323.813642 129.791841,322.874209 128.605485,322.701541 C127.71401,322.571795 126.904554,322.852747 126.190788,323.100531 C125.965234,323.179549 125.73968,323.259542 125.517055,323.318074 C109.641373,327.438701 94.3515455,327.249448 80.0723169,322.754219 C53.592667,314.42029 32.6493428,296.742761 17.8223402,270.216226 C8.72206594,253.932725 5.81036876,235.898154 9.16926826,216.612957 C12.2332876,199.016397 19.3123636,182.570959 30.808783,166.340137 C44.9259252,146.411092 63.6156622,129.765671 87.9481551,115.450785 C109.016462,103.056712 133.538381,92.9619563 165.119848,83.681766 C174.437864,80.9434611 183.935543,78.2236913 193.11979,75.5946455 C209.234696,70.9813773 225.897377,66.2100737 242.186086,61.1675736 C275.659863,50.802597 305.762047,35.6360755 331.661701,16.0865102 C330.688206,18.1907224 329.661008,20.3476131 328.653338,22.4654826 C326.55012,26.8826698 324.563097,31.0549997 323.129706,34.5815306 C323.020346,34.8498005 322.92368,35.1239237 322.835802,35.4029245 C322.183551,37.4925038 323.442162,39.1421203 325.022016,39.6913421 C326.877222,40.3381166 328.802731,39.4572156 329.501851,37.644686 C329.572153,37.4612869 329.644409,37.2837409 329.724475,37.1149747 C334.400571,27.2367862 339.063973,17.153737 343.574077,7.40334254 L344.310301,5.81128213 C344.379627,5.663002 344.434307,5.50691764 344.476293,5.34010249 C344.867839,3.79974503 344.082794,2.11208296 342.608394,1.32678356 C340.806891,0.367840311 338.752494,0.147371162 336.139583,0.625379494 C330.651102,1.63505015 325.155786,2.61155289 319.66047,3.58805563 C315.557536,4.31774998 311.453624,5.0464688 307.352642,5.78884501 C306.024705,6.02980023 304.371619,6.32831155 302.914794,7.04629958 C301.761637,7.61503194 300.836963,9.39439357 301.047871,10.6401418 C301.270496,11.9619812 302.874761,13.1852923 304.053305,13.3169885 C305.243566,13.4496602 306.379147,13.2282155 307.478601,13.0135995 L308.121088,12.8906831 C314.389732,11.7551694 320.656423,10.6069739 326.92409,9.45877837 L329.773297,8.93784684 C329.265556,9.36025012 328.751957,9.77875129 328.237381,10.1777419 C309.667744,24.5628661 287.746043,36.6037985 261.218548,46.9882856 C242.362819,54.3710755 222.543359,59.9208498 203.378104,65.289176 C198.667833,66.6071133 193.959515,67.9260261 189.258032,69.2722536 L184.64638,70.5911664 C161.483644,77.209143 137.531956,84.0534419 114.954098,93.8701722 C86.633888,106.185228 64.172224,120.517673 46.288037,137.687928 C25.5966302,157.553564 12.1698201,178.64056 5.24208983,202.154668 C-1.52550681,225.125407 -0.935746553,245.360768 7.04359261,264.01675 C19.5340119,293.216231 40.6335641,314.313958 69.7534652,326.721688 C80.1660536,331.159361 91.2035536,333.381612 102.748794,333.381612" id="Fill-8" fill="#3AADAA"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\website_hr_applicant_form.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";

publicWidget.registry.hrRecruitment = publicWidget.Widget.extend({
    selector : '#hr_recruitment_form',
    events: {
        'click #apply-btn': '_onClickApplyButton',
        "focusout #recruitment1" : "_onFocusOutName",
        'focusout #recruitment2' : '_onFocusOutMail',
        "focusout #recruitment3" : "_onFocusOutPhone",
        'focusout #recruitment4' : '_onFocusOutLinkedin',
    },

    _onClickApplyButton (ev) {
        const linkedinProfileEl = document.querySelector("#recruitment4");
        const resumeEl = document.querySelector("#recruitment6");

        const isLinkedinEmpty = !linkedinProfileEl || linkedinProfileEl.value.trim() === "";
        const isResumeEmpty = !resumeEl || !resumeEl.files.length;
        if (isLinkedinEmpty && isResumeEmpty) {
            linkedinProfileEl?.setAttribute("required", true);
            resumeEl?.setAttribute("required", true);
        } else {
            linkedinProfileEl?.removeAttribute("required");
            resumeEl?.removeAttribute("required");
        }
    },

    hideWarningMessage(targetEl, messageContainerId) {
        targetEl.classList.remove("border-warning");
        document.querySelector(messageContainerId)?.classList.add("d-none");
    },

    showWarningMessage(targetEl, messageContainerId, message) {
        targetEl.classList.add("border-warning");
        document.querySelector(messageContainerId).textContent = message;
        document.querySelector(messageContainerId)?.classList.remove("d-none");
    },

    async _onFocusOutName(ev) {
        const field = "name"
        const messageContainerId = "#warning-message";
        await this.checkRedundant(ev.currentTarget, field, messageContainerId);
    },

    async _onFocusOutMail (ev) {
        const field = "email"
        const messageContainerId = "#warning-message";
        await this.checkRedundant(ev.currentTarget, field, messageContainerId);
    },

    async _onFocusOutPhone (ev) {
        const field = "phone"
        const messageContainerId = "#warning-message";
        await this.checkRedundant(ev.currentTarget, field, messageContainerId);
    },

    async _onFocusOutLinkedin (ev) {
        const targetEl = ev.currentTarget;
        const linkedin = targetEl.value;
        const field = "linkedin";
        const messageContainerId = "#linkedin-message";
        const linkedin_regex = /^(https?:\/\/)?([\w\.]*)linkedin\.com\/in\/(.*?)(\/.*)?$/;
        let hasWarningMessage = false;
        if (!linkedin_regex.test(linkedin) && linkedin !== "") {
            const message = _t("The profile that you gave us doesn't seems like a linkedin profile")
            this.showWarningMessage(targetEl, "#linkedin-message", message);
            hasWarningMessage = true;
        } else {
            this.hideWarningMessage(targetEl, "#linkedin-message");
        }
        await this.checkRedundant(targetEl, field, messageContainerId, hasWarningMessage);
    },

    async checkRedundant(targetEl, field, messageContainerId, keepPreviousWarningMessage = false) {
        const value = targetEl.value;
        if (!value) {
            this.hideWarningMessage(targetEl, messageContainerId);
            return;
        }
        const job_id = document.querySelector("#recruitment7").value;
        const data = await rpc("/website_hr_recruitment/check_recent_application", {
            field: field,
            value: value,
            job_id: job_id,
        });

        if (data.message) {
            this.showWarningMessage(targetEl, messageContainerId, data.message);
        } else if (!keepPreviousWarningMessage) {
            this.hideWarningMessage(targetEl, messageContainerId);
        }
    },
});

```

## File: static\src\js\website_hr_recruitment_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import FormEditorRegistry from "@website/js/form_editor_registry";

FormEditorRegistry.add('apply_job', {
    formFields: [{
        type: 'char',
        modelRequired: true,
        name: 'partner_name',
        fillWith: 'name',
        string: _t('Your Name'),
    }, {
        type: 'email',
        required: true,
        fillWith: 'email',
        name: 'email_from',
        string: _t('Your Email'),
    }, {
        type: 'char',
        required: true,
        fillWith: 'phone',
        name: 'partner_phone',
        string: _t('Phone Number'),
    }, {
        type: 'char',
        name: 'linkedin_profile',
        string: _t('LinkedIn Profile'),
    }, {
        type: 'text',
        name: 'description',
        string: _t('Short Introduction'),
    }, {
        type: 'binary',
        custom: true,
        name: 'Resume',
    }],
    fields: [{
        name: 'job_id',
        type: 'many2one',
        relation: 'hr.job',
        string: _t('Applied Job'),
    }, {
        name: 'department_id',
        type: 'many2one',
        relation: 'hr.department',
        string: _t('Department'),
    }],
    successPage: '/job-thank-you',
});

```

## File: static\src\js\systray_items\new_content.js

```javascript
/** @odoo-module **/

import { NewContentModal, MODULE_STATUS } from '@website/systray_items/new_content';
import { rpc } from "@web/core/network/rpc";
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newJobElement = this.state.newContentElements.find(element => element.moduleXmlId === 'base.module_website_hr_recruitment');
        newJobElement.createNewContent = () => this.createNewJob();
        newJobElement.status = MODULE_STATUS.INSTALLED;
        newJobElement.model = 'hr.job';
    },

    async createNewJob() {
        const url = await rpc('/jobs/add');
        this.website.goToWebsite({ path: url, edition: true });
        this.websiteContext.showNewContentModal = false;
    }
});

```

## File: static\src\js\widgets\copy_link_menuitem.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { CopyButton } from "@web/core/copy_button/copy_button";
import { useService } from "@web/core/utils/hooks";
import { Component } from "@odoo/owl";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

export class CopyButtonJob extends CopyButton {
    static template = "website_hr_recruitment.CopyButtonJob";

    setup() {
        super.setup();
        this.notification = useService("notification");
    }

    showTooltip() {
        this.notification.add(_t("The job link has been copied to the clipboard."), { type: 'success', });
    }
}
export class CopyClipboardCharField extends Component {
    static components = { CopyButtonJob };
    static template = "website_hr_recruitment.CopyJobLinkButton";
    static props = { ...standardFieldProps }

    setup() {
        this.copyText = _t("Share Job");
        this.successText = _t("Copied");
    }
}

export const copyClipboardJobLinkButton = {
    component: CopyClipboardCharField,
};

registry.category("fields").add("CopyClipboardJobLinkButton", copyClipboardJobLinkButton);

```

## File: static\src\js\widgets\copy_link_menuitem.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_hr_recruitment.CopyJobLinkButton">
        <CopyButtonJob
            content="props.record.data[props.name]"
            copyText="copyText"
            successText="successText"/>
    </t>

    <t t-name="website_hr_recruitment.CopyButtonJob">
        <a
            t-ref="button"
            role="button"
            class="oe_kanban_action"
            t-on-click.stop="onClick">
            <span t-esc="props.copyText"/>
        </a>
    </t>
</templates>

```

## File: views\hr_job_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_job_website_inherit" model="ir.ui.view">
        <field name="name">hr.job.kanban.inherit</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//kanban" position="attributes">
                <attribute name="default_order">is_published desc, is_favorite desc, create_date DESC</attribute>
            </xpath>
            <xpath expr="//t[@t-name='card']/div" position="before">
                <field name="website_published" invisible="1"/>
                <widget name="web_ribbon" title="Published" bg_color="text-bg-success" invisible="not website_published"/>
            </xpath>
            <xpath expr="//div[@name='kanban_boxes']" position="attributes">
                <attribute name="class" add="border-top pt-2 mx-n2 px-3" separator=" "/>
            </xpath>
            <xpath expr="//div[@name='kanban_boxes']" position="inside">
                <field name="website_url" invisible="1"/>
                <div class="col-6">
                    <field name="is_published" widget="boolean_toggle_labeled" options="{'false_label': 'Not Published', 'true_label': 'Published'}"/>
                </div>
                <div class="col-6" name="bottom_right"></div>
            </xpath>
        </field>
    </record>

    <record id="hr_job_form_inherit" model="ir.ui.view">
        <field name="name">hr.job.form.inherit</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='job_description_page']" position="after">
                <page string="Application Info" name="recruitment_page">
                    <separator string="Process Details"/>
                    <field name="job_details" nolabel="1"/>
                </page>
            </xpath>
        </field>
    </record>

    <record id="view_hr_job_kanban_referal_extends" model="ir.ui.view">
        <field name="model">hr.job</field>
        <field name="name">hr.job.view.kanban</field>
        <field name="inherit_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='bottom_right']" position="replace">
                <field name="website_url" invisible="1"/>
                <div class="o_link_trackers col-6 text-end">
                    <a type="object" name="open_website_url">
                        <i class="fa fa-fw fa-external-link" role="img"></i>
                        Job Page
                    </a>
                </div>
            </xpath>

            <xpath expr="//div[@name='menu_new_applications']" position="after">
                <field name="full_url" widget="CopyClipboardJobLinkButton"/>
            </xpath>
        </field>
    </record>

    <record id="hr_job_search_view_inherit" model="ir.ui.view">
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_job_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='message_needaction']" position="before">
                <filter name="published" string="Published" domain="[('is_published', '=', True)]"/>
                <separator name="published_separator"/>
            </xpath>
            <xpath expr="//group" position="inside">
                <filter string="Published" name="groupby_published" domain="[]" context="{'group_by': 'is_published'}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_recruitment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_hr_recruitment_tree_url" model="ir.ui.view" >
        <field name="name">hr.recruitment.list.inherit.url</field>
        <field name="model">hr.recruitment.source</field>
        <field name="inherit_id" ref="hr_recruitment.hr_recruitment_source_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="before">
                <field name="url" widget="CopyClipboardURL"/>
            </xpath>
        </field>
    </record>

    <record id="view_hr_job_form_website_published_button" model="ir.ui.view" >
        <field name="name">hr.job.form.inherit.published.button</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.hr_job_survey"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <field name="is_published" widget="website_redirect_button"/>
            </div>
            <xpath expr="//div[hasclass('oe_title')]" position="before">
                <div class="float-end">
                    <field name="website_published" widget="boolean_toggle_labeled" nolabel="1" options="{'false_label': 'Not Published', 'true_label': 'Published'}"/>
                </div>
            </xpath>
            <xpath expr="//div[@name='recruitment_target']" position="after">
                <field name="website_id" options="{'no_create': True}" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" groups="website.group_multi_website"/>
            </xpath>
        </field>
    </record>

    <record id="view_hr_job_form_inherit_website" model="ir.ui.view">
        <field name="name">hr.job.form</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_form"/>
        <field name="arch" type="xml">
            <field name="description" position="attributes">
                <attribute name="placeholder">e.g. Summarize the position in one or two lines that will be displayed on the Jobs list page...</attribute>
            </field>
        </field>
    </record>

    <record id="view_hr_job_tree_inherit_website" model="ir.ui.view">
        <field name="name">hr.job.list</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.hr_job_view_tree_inherit"/>
        <field name="arch" type="xml">
            <field name="no_of_employee" position="before">
                <field name="website_id" groups="website.group_multi_website" optional="hide"/>
            </field>
            <xpath expr="//field[@name='alias_id']" position="before">
                <field name="is_published" string="Published"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>


<template id="jobs_searchbar_input_snippet_options" inherit_id="website.searchbar_input_snippet_options" name="jobs search bar snippet options">
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='scope_opt']" position="inside">
        <we-button data-set-search-type="jobs" data-select-data-attribute="jobs" data-name="search_jobs_opt" data-form-action="/jobs">Jobs</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/div[@data-dependencies='limit_opt']" position="inside">
        <we-checkbox string="Description" data-dependencies="search_jobs_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
            data-apply-to=".search-query"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="Hr Recruitment Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(.o_website_hr_recruitment_jobs_list)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Jobs Page">
            <we-checkbox string="Countries Filter"
                         data-customize-website-views="website_hr_recruitment.job_filter_by_countries"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Offices Filter"
                         data-customize-website-views="website_hr_recruitment.job_filter_by_offices"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Departments Filter"
                         data-customize-website-views="website_hr_recruitment.job_filter_by_departments"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Employment Types Filter"
                         data-customize-website-views="website_hr_recruitment.job_filter_by_employment_type"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Search Bar"
                         data-customize-website-views="website_hr_recruitment.job_search_bar"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Sidebar"
                         data-customize-website-views="website_hr_recruitment.job_right_side_bar"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_hr_recruitment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="index" name="Jobs">
    <t t-call="website.layout">
        <div id="wrap" class="o_website_hr_recruitment_jobs_list o_colored_level o_cc o_cc2">
            <!-- Snippet area -->
            <div class="oe_structure" id="oe_structure_hr_recruitment_index_top"/>

            <!-- Topbar -->
            <nav class="navbar navbar-light border-top shadow-sm d-print-none w-100 o_colored_level">
                <div class="container">
                    <div class="d-flex flex-column flex-md-row flex-md-wrap flex-lg-nowrap align-items-center justify-content-between w-100">
                        <!-- Title -->
                        <span class="navbar-brand h5 my-0 me-sm-auto">Our Job Offers</span>
                        <!-- Customizations -->
                        <t t-call="website_hr_recruitment.topbar_customizations"/>
                    </div>
                </div>
            </nav>

            <!-- Content -->
            <div class="container oe_website_jobs">
                <div class="row pt48 pb16">
                    <div class="d-none" id="jobs_grid_left"/>
                    <div class="col-lg" id="jobs_grid">
                        <div class="row flex-column">
                            <t t-if="not jobs">
                                <div class="col">
                                    <div t-if="search" class="alert alert-warning text-center" role="alert">
                                        No results found for '<span t-out="search"/>'.
                                    </div>
                                    <div class="alert alert-info text-center text-muted" groups="hr_recruitment.group_hr_recruitment_manager">
                                        Create new job pages from the <strong>+ <i>New</i></strong> top-right button.
                                    </div>
                                    <div class="col-lg-12 text-center pt24">
                                        <p class="h5 fw-light text-muted pb24">
                                            There are currently no open job opportunities,<br class="mb-2"/>
                                            but feel free to <span class="fw-bold">contact us</span> for a spontaneous application.
                                        </p>
                                        <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_not_found.svg" style="width: 50% !important;" alt="Job not found"/>
                                    </div>
                                </div>
                            </t>
                            <t t-elif="search and original_search">
                                <div class="col">
                                    <div class="alert alert-info text-center" role="alert">
                                        No results found for '<span t-out="original_search"/>'. Showing results for '<span t-out="search"/>'.
                                        <t t-set="search" t-value="original_search"/>
                                    </div>
                                </div>
                            </t>
                            <div t-foreach="jobs" t-as="job" class="col-lg mb32">
                                <div t-attf-class="card #{not job.website_published and 'o_jobs_unpublished'}">
                                    <a t-attf-href="/jobs/#{ slug(job) }"
                                            t-attf-class="text-decoration-none text-reset"
                                            draggable="false">
                                        <div class="card-body p-4">
                                            <div class="mt0 d-flex justify-content-between align-items-center">
                                                <h3 t-field="job.name"/>
                                                <span t-if="not job.website_published" class="badge text-bg-danger mb8">unpublished</span>
                                            </div>
                                            <h5 t-if="job.no_of_recruitment >= 1" class="text-reset">
                                                <span t-field="job.no_of_recruitment"/>
                                                <t t-if="job.no_of_recruitment == 1">
                                                    open position
                                                </t>
                                                <t t-else="">
                                                    open positions
                                                </t>
                                            </h5>
                                            <t t-set="job_desc_edition_placeholder">Insert a Job Description...</t>
                                            <div class="oe_empty text-muted mb16"
                                                 t-field="job.description"
                                                 t-att-data-editor-message="job_desc_edition_placeholder"/>
                                            <div class="o_job_infos d-flex flex-column">
                                                <span t-field="job.address_id" class="fw-light" t-options='{"widget": "contact", "fields": ["city"], "no_tag_br": True, "null_text": "Remote"}'/>
                                                <div t-if="job.department_id"
                                                        class="d-inline-flex align-items-center">
                                                    <i class="fa fa-sitemap fa-fw"/><span t-field="job.department_id" class="fw-light" t-options='{"fields": ["name"], "no_tag_br": True}'/>
                                                </div>
                                                <div t-if="job.contract_type_id"
                                                        class="d-inline-flex align-items-center">
                                                    <i class="fa fa-suitcase fa-fw" title="Employment type" role="img" aria-label="Employment type"/><span t-field="job.contract_type_id" class="fw-light"/>
                                                </div>
                                            </div>
                                        </div>
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                    <!-- Pager -->
                    <div class='navbar mb-3 w-100'>
                        <t t-call="website.pager">
                            <t t-set="classname" t-valuef="mx-auto"/>
                        </t>
                    </div>
                </div>
            </div>

            <!-- Snippet area -->
            <div class="oe_structure" id="oe_structure_hr_recruitment_index_bottom"/>
        </div>
    </t>
</template>

<!-- Job Detail -->
<template id="detail" name="Job Detail" track="1">
    <t t-call="website.layout">
        <!-- Topbar -->
        <nav class="navbar navbar-light border-top shadow-sm d-print-none">
            <div class="container">
                <div class="d-flex flex-column flex-md-row flex-md-wrap flex-lg-nowrap justify-content-between w-100">
                    <!-- Title -->
                    <span class="navbar-brand h4 my-0 me-auto">
                        <a t-attf-href="/jobs">
                            <i class="fa fa-long-arrow-left text-primary me-2"/>All Jobs
                        </a>
                    </span>
                </div>
            </div>
        </nav>
        <!-- Content -->
        <div id="wrap" class="js_hr_recruitment">
            <div itemscope="itemscope" itemtype="https://schema.org/JobPosting">
                <meta t-if="job.contract_type_id" itemprop="employmentType" t-att-content="job.contract_type_id.sudo().name"/>
                <meta t-if="job.published_date" itemprop="datePosted" t-att-content="job.published_date"/>
                <meta itemprop="title" t-att-content="job.name"/>
                <meta itemprop="directApply" content="true"/>
                <span itemprop="hiringOrganization" itemscope="itemscope" itemtype="https://schema.org/Organization">
                    <meta itemprop="name" t-att-content="job.company_id.name"/>
                    <meta itemprop="logo" t-attf-content="/logo.png?company=#{job.company_id.id}"/>
                </span>
                <span t-if="job.address_id.sudo().contact_address" itemprop="jobLocation" itemscope="itemscope" itemtype="https://schema.org/Place">
                    <meta itemprop="address" t-att-content="job.address_id.sudo().contact_address"/>
                </span>
                <t t-else="">
                    <meta itemprop="jobLocationType" content="TELECOMMUTE"/>
                    <span itemprop="applicantLocationRequirements" itemscope="itemscope" itemtype="https://schema.org/Country">
                        <meta itemprop="name" t-att-content="job.company_id.country_id.name"/>
                    </span>
                </t>
                <!-- Job name -->
                <section class="pb32">
                    <div class="container">
                        <div class="mt32">
                            <div class="row">
                                <div class="col-md-9">
                                    <h1 t-field="job.name"/>
                                    <h5 class="fw-light o_not_editable" t-field="job.address_id" t-options='{
                                        "widget": "contact",
                                        "fields": ["city"],
                                        "no_tag_br": True,
                                        "null_text": "Remote"
                                    }'/>
                                </div>
                                <div class="col-md-3">
                                    <div class="text-center">
                                        <br/>
                                        <a role="button" t-attf-href="/jobs/apply/#{slug(job)}" class="btn btn-primary btn-lg">Apply Now!</a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </section>
                <!-- Job Description -->
                <div t-field="job.website_description"/>
                <div class="oe_structure">
                    <section class="o_job_bottom_bar mt24 mb48">
                        <div class="text-center">
                            <a role="button" t-attf-href="/jobs/apply/#{slug(job)}" class="btn btn-primary btn-lg">Apply Now!</a>
                        </div>
                    </section>
                </div>
            </div>
        </div>
    </t>
</template>

<!-- Apply -->
<template id="apply">
    <t t-call="website.layout">
        <t t-set="additional_title">Apply Job</t>

        <div id="wrap"  class="container">
            <nav aria-label="breadcrumb" class="mt-5">
                <ol class="breadcrumb ps-0 mb-0 pb-1">
                    <li class="breadcrumb-item"><a href="/jobs" class="text-secondary fw-bold">Jobs</a></li>
                    <li class="breadcrumb-item active" aria-current="page">
                        <a t-attf-href="/jobs/detail/#{slug(job)}">
                            <span t-field="job.name"/>
                        </a>
                    </li>
                </ol>
            </nav>
            <h1 class="mb-4">
                Job Application Form
            </h1>
            <span class="hidden" data-for="hr_recruitment_form" t-att-data-values="{'department_id': job and job.department_id.id or '', 'job_id': job and job.id or ''}" />
            <div id="jobs_section" class="container">
                <div class="row">
                    <section id="forms" class="col-12 col-md-9 s_website_form" data-vcss="001" data-snippet="s_website_form">
                        <div class="container">
                            <form id="hr_recruitment_form" action="/website/form/" method="post"
                                enctype="multipart/form-data" class="o_mark_required row"
                                data-mark="*" data-model_name="hr.applicant"
                                data-success-mode="redirect" data-success-page="/job-thank-you"
                                hide-change-model="true">
                                <div class="s_website_form_rows row s_col_no_bgcolor">
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_required s_website_form_model_required"
                                        data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment1">
                                                <span class="s_website_form_label_content">Your Name</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="recruitment1" type="text"
                                                    class="form-control s_website_form_input"
                                                    name="partner_name" required=""
                                                    data-fill-with="name"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_required"
                                        data-type="email" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment2">
                                                <span class="s_website_form_label_content">Your Email</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="recruitment2" type="email"
                                                    class="form-control s_website_form_input"
                                                    name="email_from" required=""
                                                    data-fill-with="email"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_required"
                                        data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment3">
                                                <span class="s_website_form_label_content">Your Phone Number</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="recruitment3" type="tel"
                                                    class="form-control s_website_form_input"
                                                    name="partner_phone" required=""
                                                    data-fill-with="phone"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_required"
                                        data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment4">
                                                <span class="s_website_form_label_content">LinkedIn Profile</span>
                                            </label>
                                            <div class="col-sm" >
                                                <i class="fa fa-linkedin fa-2x o_linkedin_icon"></i>
                                                <input id="recruitment4" type="text"
                                                    class="form-control s_website_form_input pl64"
                                                    placeholder="e.g. https://www.linkedin.com/in/fpodoo"
                                                    style="padding-inline-start: calc(40px + 0.375rem)"
                                                    name="linkedin_profile"
                                                    data-fill-with="linkedin_profile"/>
                                                <div class="alert alert-warning mt-2 d-none" id="linkedin-message"></div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_custom"
                                        data-type="binary" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment6">
                                                <span class="s_website_form_label_content">Resume</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="recruitment6" type="file"
                                                    class="form-control s_website_form_input o_resume_input"
                                                    name="Resume"/>
                                                <span class="text-muted small">Provide either a resume file or a linkedin profile</span>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field"
                                        data-type="text" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment5">
                                                <span class="s_website_form_label_content">Short Introduction</span>
                                            </label>
                                            <div class="col-sm">
                                                <textarea id="recruitment5"
                                                    class="form-control s_website_form_input"
                                                    placeholder="Optional introduction, or any question you might have about the job…"
                                                    name="applicant_notes"  rows="5"></textarea>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_dnone"
                                         data-type="record" data-model="hr.job">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment7">
                                                <span class="s_website_form_label_content">Job</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="recruitment7" type="hidden"
                                                    class="form-control s_website_form_input"
                                                    name="job_id"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 mb-0 py-2 s_website_form_field s_website_form_dnone"
                                         data-type="record" data-model="hr.department">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-4 col-sm-auto s_website_form_label" style="width: 200px" for="recruitment8">
                                                <span class="s_website_form_label_content">Department</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="recruitment8" type="hidden"
                                                    class="form-control s_website_form_input"
                                                    name="department_id"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 s_website_form_submit mb64" data-name="Submit Button">
                                        <div class="alert alert-warning mt-2 d-none" id="warning-message"></div>
                                        <div style="width: 200px" class="s_website_form_label"/>
                                        <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send" id="apply-btn">I'm feeling lucky</a>
                                        <span id="s_website_form_result"></span>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </section>
                    <section class="col-12 col-md-3 ps-5">
                        <a role="button" t-attf-href="/jobs/#{slug(job)}" class="btn btn-outline-primary btn-lg mb16 o_apply_description_link">
                            <i class="oi oi-arrow-left"></i> Job Description
                        </a>
                        <div t-if="job.name" class="d-flex flex-column align-items-baseline">
                            <span class="text-muted small">Job</span>
                            <h6 t-field="job.name"/>
                        </div>
                        <div class="d-flex flex-column align-items-baseline">
                            <span class="text-muted small">Location</span>
                            <h6 t-if="job.address_id" t-field="job.address_id" t-options='{
                                "widget": "contact",
                                "fields": ["city"],
                                "no_tag_br": True,
                                "no_marker": True
                            }'/>
                            <h6 t-else="">Remote</h6>
                        </div>
                        <div t-if="job.department_id" class="d-flex flex-column align-items-baseline">
                            <span class="text-muted small">Department</span>
                            <h6 t-field="job.department_id"/>
                        </div>
                        <div t-if="job.contract_type_id" class="d-flex flex-column align-items-baseline">
                            <span class="text-muted small">Employment Type</span>
                            <h6 t-field="job.contract_type_id"/>
                        </div>
                        <hr t-if="job.job_details" class="w-50 my-3"/>
                        <div t-field="job.job_details"/>
                    </section>
                </div>
            </div>
            <div class="oe_structure mt-2"/>
        </div>
    </t>
</template>

<template id="default_website_description">
    <!-- Description text and ratings -->
    <section class="pt32">
        <div class="container">
            <div class="row">
                <div class="col-lg-8 pb32" itemprop="description">
                    <p class="lead">
                        As an employee of our company, you will <b>collaborate with each department
                        to create and deploy disruptive products.</b> Come work at a growing company
                        that offers great benefits with opportunities to moving forward and learn
                        alongside accomplished leaders. We're seeking an experienced and outstanding
                        member of staff.
                        <br/><br/>
                        This position is both <b>creative and rigorous</b> by nature you need to think
                        outside the box. We expect the candidate to be proactive and have a "get it done"
                        spirit. To be successful, you will have solid solving problem skills.
                    </p>
                </div>
                <div class="col-lg-3 offset-lg-1 pb32">
                    <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                        <h6 class="s_rating_title">Customer Relationship</h6>
                        <div class="s_rating_icons o_not_editable">
                            <span class="s_rating_active_icons text-primary">
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                            </span>
                            <span class="s_rating_inactive_icons text-primary">
                            </span>
                        </div>
                    </div>
                    <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                        <h6 class="s_rating_title">Personal Evolution</h6>
                        <div class="s_rating_icons o_not_editable">
                            <span class="s_rating_active_icons text-primary">
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                            </span>
                            <span class="s_rating_inactive_icons text-primary">
                            </span>
                        </div>
                    </div>
                    <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                        <h6 class="s_rating_title">Autonomy</h6>
                        <div class="s_rating_icons o_not_editable">
                            <span class="s_rating_active_icons text-primary">
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                            </span>
                            <span class="s_rating_inactive_icons text-primary">
                                <i class="fa fa-star-o"/>
                            </span>
                        </div>
                    </div>
                    <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                        <h6 class="s_rating_title">Administrative Work</h6>
                        <div class="s_rating_icons o_not_editable">
                            <span class="s_rating_active_icons text-primary">
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                            </span>
                            <span class="s_rating_inactive_icons text-primary">
                                <i class="fa fa-star-o"/>
                                <i class="fa fa-star-o"/>
                                <i class="fa fa-star-o"/>
                            </span>
                        </div>
                    </div>
                    <div class="s_rating pb8" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
                        <h6 class="s_rating_title">Technical Expertise</h6>
                        <div class="s_rating_icons o_not_editable">
                            <span class="s_rating_active_icons text-primary">
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                                <i class="fa fa-star"/>
                            </span>
                            <span class="s_rating_inactive_icons text-primary">
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- Responsabilities, Must Have and Nice to have -->
    <section class="s_comparisons pt24 pb24 bg-200" data-snippet="s_comparisons">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                    <div class="card text-bg-primary">
                        <h4 class="card-header">Responsibilities</h4>
                        <ul class="list-group list-group-flush" itemprop="responsibilities">
                            <li class="list-group-item">Lead the entire sales cycle</li>
                            <li class="list-group-item">Achieve monthly sales objectives</li>
                            <li class="list-group-item">Qualify the customer needs</li>
                            <li class="list-group-item">Negotiate and contract</li>
                            <li class="list-group-item">Master demos of our software</li>
                        </ul>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                    <div class="card text-bg-primary">
                        <h4 class="card-header">Must Have</h4>
                        <ul class="list-group list-group-flush" itemprop="skills">
                            <li class="list-group-item">Bachelor Degree or Higher</li>
                            <li class="list-group-item">Passion for software products</li>
                            <li class="list-group-item">Perfect written English</li>
                            <li class="list-group-item">Highly creative and autonomous</li>
                            <li class="list-group-item">Valid work permit for Belgium</li>
                        </ul>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16" data-name="Box">
                    <div class="card text-bg-primary">
                        <h4 class="card-header">Nice to have</h4>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item">Experience in writing online content</li>
                            <li class="list-group-item">Additional languages</li>
                            <li class="list-group-item">Google Adwords experience</li>
                            <li class="list-group-item">Strong analytical skills</li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- What's great -->
    <section class="s_comparisons pt24 pb24" data-snippet="s_comparisons">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-sm-7 pb40">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <ul class="lead">
                        <li>Great team of smart people, in a friendly and open culture</li>
                        <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                        <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                        <li>Expand your knowledge of various business industries</li>
                        <li>Create content that will help our users on a daily basis</li>
                        <li>Real responsibilities and challenges in a fast evolving company</li>
                    </ul>
                </div>
                <div data-name="Box" class="col-sm-4 offset-sm-1 pt16 pb16">
                    <div class="card shadow text-center">
                        <h5 class="card-header o_colored_level text-bg-primary">Our Product</h5>
                        <div class="card-body p-0 pt-3">
                            <img class="img-fluid o_we_custom_image pb24" src="/website_hr_recruitment/static/src/img/job_product.svg" style="width: 75% !important;" alt="Our Product"/>
                            <p>Discover our products.</p>
                            <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- What we offer -->
    <section class="s_features pt64 pb64" data-name="Features" data-snippet="s_features">
        <div class="container" itemprop="employerOverview">
            <div class="col-lg-12">
                <h3>What We Offer</h3>
                <p class="lead">
                    Each employee has a chance to see the impact of his work.
                    You can make a real contribution to the success of the company.
                    <br/>
                    Several activities are often organized all over the year, such as weekly
                    sports sessions, team building events, monthly drink, and much more
                </p>
            </div>
            <div class="row">
                <div class="col-lg-3">
                    <div class="s_hr pt-4 pb32">
                        <hr class="w-100 mx-auto"/>
                    </div>
                    <i class="fa fa-gift mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                    <h3 class="h5-fs">Perks</h3>
                    <p>A full-time position <br/>Attractive salary package.</p>
                </div>
                <div class="col-lg-3">
                    <div class="s_hr pt-4 pb32">
                        <hr class="w-100 mx-auto"/>
                    </div>
                    <i class="fa fa-bar-chart mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                    <h3 class="h5-fs">Trainings</h3>
                    <p>12 days / year, including <br/>6 of your choice.</p>
                </div>
                <div class="col-lg-3">
                    <div class="s_hr pt-4 pb32">
                        <hr class="w-100 mx-auto"/>
                    </div>
                    <i class="fa fa-futbol-o mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                    <h3 class="h5-fs">Sport Activity</h3>
                    <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                </div>
                <div class="col-lg-3">
                    <div class="s_hr pt-4 pb32">
                        <hr class="w-100 mx-auto"/>
                    </div>
                    <i class="fa fa-coffee mb-3 rounded fs-5 bg-o-color-3" role="img"/>
                    <h3 class="h5-fs">Eat &amp; Drink</h3>
                    <p>Fruit, coffee and <br/>snacks provided.</p>
                </div>
            </div>
        </div>
    </section>
    <!-- Photos -->
    <section class="o_jobs_image_gallery s_image_gallery pt24 pb24 o_grid o_spc-medium" data-vcss="001" data-columns="3" style="overflow: hidden;" data-snippet="s_images_wall" data-name="Images Wall">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-8">
                    <img src="/website_hr_recruitment/static/src/img/job_image_9.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                </div>
                <div class="col-lg-4">
                    <img src="/website_hr_recruitment/static/src/img/job_image_10.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                </div>
            </div>
            <div class="row s_nb_column_fixed">
                <div class="col-lg-3">
                    <img src="/website_hr_recruitment/static/src/img/job_image_11.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                </div>
                <div class="col-lg-3">
                    <img src="/website_hr_recruitment/static/src/img/job_image_12.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                </div>
                <div class="col-lg-6">
                    <img src="/website_hr_recruitment/static/src/img/job_image_13.jpg" class="img img-fluid d-block h-100 w-100 shadow" style="object-fit: cover;"/>
                </div>
            </div>
        </div>
    </section>
</template>

<!-- Thank You -->
<record id="thankyou" model="website.page">
    <field name="url">/job-thank-you</field>
    <field name="is_published">True</field>
    <field name="website_indexed" eval="False"/>
    <field name="name">Thank you (Recruitment)</field>
    <field name="type">qweb</field>
    <field name="key">website_hr_recruitment.thankyou</field>
    <field name="arch" type="xml">
        <t name="Thank you (Recruitment)" t-name="website_hr_recruitment.thankyou">
            <t t-if="request.session.get('form_builder_model_model', '') == 'hr.applicant'">
                <t t-set="job_sudo" t-value="request.website._website_form_last_record().sudo().job_id"/>
            </t>
            <t t-call="website.layout">
                <div id="wrap">
                    <div class="oe_structure"/>
                    <div id="jobs_thankyou" class="container">
                        <div class="row pt24 pb24">
                            <div class="col pt24 pb24 text-center">
                                <h1 class="fw-bolder">Congratulations!</h1>
                                <p class="fw-light pb24">
                                    Your application has been posted successfully,<br class="mb-2"/>
                                    We usually respond within 3 days...
                                </p>
                                <img class="img-fluid o_we_custom_image" t-attf-style="width:{{ '50%' if (job_sudo and job_sudo.user_id) else '25%' }} !important;" src="/website_hr_recruitment/static/src/img/job_congratulations.svg" alt="Congratulations!"/>
                                <div class="row" id="o_recruitment_thank_cta">
                                    <div class="col-lg-12 text-center mt32 mb48">
                                        <p>
                                            <span class="h5 fw-light">In the meantime,</span><br/>
                                            <span class="h3 mt8 mb32 fw-bold">Take a look around our website:</span>
                                        </p>
                                        <a role="button" href="/" class="btn btn-primary btn-lg">Home</a><a role="button" href="/" class="btn btn-primary btn-lg ms-3">About Us</a><a role="button" href="/" class="btn btn-primary btn-lg ms-3">Products</a>
                                    </div>
                                </div>
                            </div>
                            <!-- HR Recruiter Block -->
                            <t t-if="job_sudo and job_sudo.user_id">
                                <div class="col-lg-4 align-self-center ps-4 o_cc o_cc2 rounded">
                                    <section class="s_text_block pt24" data-snippet="s_text_block" data-name="Text">
                                        <div class="container">
                                            <p class="text-center mb-0 fs-3 lead">Your <b>contact</b> information is:</p>
                                        </div>
                                    </section>
                                    <section class="s_company_team pt12" data-snippet="s_company_team" data-name="Team">
                                        <div class="container-fluid">
                                            <div class="row justify-content-center">
                                                <div class="o_jobs_hr_recruiter col-10 o_cc o_cc1 mt24 p-4 border border-primary rounded shadow">
                                                    <div class="row align-items-center">
                                                        <div class="col-lg-5 pb16 o_not_editable">
                                                            <img t-att-src="image_data_uri(job_sudo.user_id.avatar_256)" class="img-fluid" loading="lazy"/>
                                                        </div>
                                                        <div class="col-lg-7 px-1">
                                                            <h2 class="o_default_snippet_text text-truncate" t-field="job_sudo.user_id.name"/>
                                                            <p class="fw-light text-truncate" t-field="job_sudo.user_id.job_title"/>
                                                        </div>
                                                    </div>
                                                    <div class="row">
                                                        <ul class="list-unstyled mb-0">
                                                            <li t-if="job_sudo.user_id.work_phone"><i class="fa fa-phone fa-fw me-2"></i><span class="o_force_ltr" t-field="job_sudo.user_id.work_phone"/></li>
                                                            <li class="d-inline-flex align-items-baseline"><i class="fa fa-envelope fa-fw me-2"></i><span><a class="text-break" t-attf-href="mailto:#{job_sudo.user_id.email}" t-field="job_sudo.user_id.email"/></span></li>
                                                        </ul>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </section>
                                    <section class="s_text_block pt24" data-snippet="s_text_block" data-name="Text">
                                        <div class="container">
                                            <p class="lead alert px-0">
                                                I usually <strong>answer applications within 3 days</strong>.
                                                <br/><br/>
                                                The next step is either a call or a meeting in our offices.
                                                <br/><br/>
                                                Feel free to <strong>contact me if you want a faster
                                                feedback</strong> or if you don't get news from me
                                                quickly enough.
                                            </p>
                                        </div>
                                    </section>
                                </div>
                            </t>
                        </div>
                    </div>
                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </field>
</record>

<!-- Topbar Customizations -->
<template id="topbar_customizations" name="Topbar">
    <!-- Sets -->
    <t t-set="current_country_param" t-value="'&amp;is_remote=1' if is_remote else ('&amp;country_id=%s' % country_id.id if country_id else '')"/>
    <t t-set="current_department_param" t-value="'&amp;department_id=%s' % department_id.id if department_id else '&amp;is_other_department=1' if is_other_department else ''"/>
    <t t-set="current_office_param" t-value="'' if is_remote else '&amp;office_id=%s' % office_id.id if office_id else ''"/>
    <t t-set="current_employment_type_param" t-value="'&amp;is_untyped=1' if is_untyped else ('&amp;contract_type_id=%s' % contract_type_id.id if contract_type_id else '')"/>
    <t t-set="opt_jobs_filter_countries" t-value="is_view_active('website_hr_recruitment.job_filter_by_countries')"/>
    <t t-set="opt_jobs_filter_departments" t-value="is_view_active('website_hr_recruitment.job_filter_by_departments')"/>
    <t t-set="opt_jobs_filter_offices" t-value="is_view_active('website_hr_recruitment.job_filter_by_offices')"/>
    <t t-set="opt_jobs_filter_employment_type" t-value="is_view_active('website_hr_recruitment.job_filter_by_employment_type')"/>
    <t t-set="opt_jobs_search_bar" t-value="is_view_active('website_hr_recruitment.job_search_bar')"/>
    <!-- Filters -->
    <ul class="o_jobs_topbar_filters flex-md-nowrap nav py-2 py-md-0 ps-lg-3"/>
    <!-- Search bar -->
    <div t-if="opt_jobs_search_bar" class="o_jobs_topbar_search_bar d-flex align-items-center justify-content-end w-md-100 w-lg-25 me-2 me-lg-0 ps-lg-2"/>
</template>

<!-- Filter - Countries -->
<template id="job_filter_by_countries" inherit_id="website_hr_recruitment.topbar_customizations" active="False" priority="10">
    <xpath expr="//ul[hasclass('o_jobs_topbar_filters')]" position="inside">
        <t t-set="non_country_params" t-valuef="#{current_department_param}#{current_office_param}#{current_employment_type_param}"/>
        <li t-attf-class="nav-item dropdown w-100 w-md-auto me-2 my-1 flex-fill">
            <button type="button" class="btn btn-light dropdown-toggle w-100" data-bs-toggle="dropdown" id="countriesDropdown">
                <i class="fa fa-map-marker"/>
                <t t-if="country_id" t-out="country_id.name"/>
                <t t-elif="is_remote">Remote</t>
                <t t-else="">All Countries</t>
            </button>
            <div class="dropdown-menu w-100 w-md-auto" aria-labelledby="countriesDropdown">
                <a t-attf-href="/jobs?all_countries=1#{non_country_params}"
                    t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{'' if country_id or is_remote else ' active'}">
                    All Countries
                    <span t-attf-class="badge text-bg-primary ms-2" t-out="count_per_country.get('all', '0')"/>
                </a>
                <t t-foreach="countries" t-as="country">
                    <t t-if="country">
                        <a t-attf-href="/jobs?country_id=#{country.id}#{non_country_params}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link #{' active' if country_id and country_id.id == country.id else ''}">
                            <t t-out="country.name"/>
                            <span t-attf-class="badge #{' bg-light text-primary' if country_id and country_id.id == country.id else ' text-bg-primary'} ms-2" t-out="count_per_country.get(country, '0')"/>
                        </a>
                    </t>
                    <t t-else="">
                        <a t-attf-href="/jobs?is_remote=1#{current_department_param}#{current_employment_type_param}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{' active' if is_remote else ''}">
                            Remote
                            <span t-attf-class="badge #{' bg-light text-primary' if is_remote else ' text-bg-primary'} ms-2" t-out="count_per_country.get(None, '0')"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Filter - Departments -->
<template id="job_filter_by_departments" inherit_id="website_hr_recruitment.topbar_customizations" active="False" priority="30">
    <xpath expr="//ul[hasclass('o_jobs_topbar_filters')]" position="inside">
        <t t-set="non_department_params" t-valuef="#{current_country_param}#{current_office_param}#{current_employment_type_param}"/>
        <li t-attf-class="nav-item dropdown w-100 w-md-auto me-2 my-1 flex-fill">
            <button type="button" class="btn btn-light dropdown-toggle w-100" data-bs-toggle="dropdown" id="departmentsDropdown">
                <i class="fa fa-sitemap"/>
                <t t-if="department_id" t-out="department_id.name"/>
                <t t-elif="is_other_department">Others</t>
                <t t-else="">All Departments</t>
            </button>
            <div class="dropdown-menu w-100 w-md-auto" aria-labelledby="departmentsDropdown">
                <a t-attf-href="/jobs?all_departments=1#{non_department_params}"
                    t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{'' if department_id or is_other_department else ' active'}">
                    All Departments
                    <span t-attf-class="badge text-bg-primary ms-2" t-out="count_per_department.get('all', '0')"/>
                </a>
                <t t-foreach="departments" t-as="department">
                    <t t-if="department">
                        <a t-attf-href="/jobs?department_id=#{department.id}#{non_department_params}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link #{' active' if department_id and department_id.id == department.id else ''}">
                            <t t-out="department.name"/>
                            <span t-attf-class="badge #{' text-bg-primary' if department_id and department_id.id == department.id else ' text-bg-primary'} ms-2" t-out="count_per_department.get(department, '0')"/>
                        </a>
                    </t>
                    <t t-else="">
                        <a t-attf-href="/jobs?is_other_department=1#{non_department_params}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link #{' active' if is_other_department else ''}">
                            Others
                            <span t-attf-class="badge #{' text-bg-primary' if is_other_department else ' text-bg-primary'} ms-2" t-out="count_per_department.get(None, '0')"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Filter - Offices -->
<template id="job_filter_by_offices" inherit_id="website_hr_recruitment.topbar_customizations" active="False" priority="20">
    <xpath expr="//ul[hasclass('o_jobs_topbar_filters')]" position="inside">
        <t t-set="non_location_params" t-valuef="#{current_department_param}#{current_employment_type_param}"/>
        <li t-attf-class="nav-item dropdown w-100 w-md-auto me-2 my-1 flex-fill">
            <button type="button" class="btn btn-light dropdown-toggle w-100" data-bs-toggle="dropdown" id="officesDropdown">
                <i class="fa fa-building"/>
                <t t-if="office_id">
                    <span t-if="len(office_id.contact_address) &lt;= 5" class="fst-italic">No address specified</span>
                    <t t-out="office_id.city"/><t t-if="office_id.country_id">, <t t-out="office_id.country_id.name"/></t>
                </t>
                <t t-elif="is_remote">Remote</t>
                <t t-else="">All Offices</t>
            </button>
            <div class="dropdown-menu w-100 w-md-auto" aria-labelledby="officesDropdown">
                <a t-attf-href="/jobs?#{'all_countries=1' if is_remote else current_country_path}#{non_location_params}"
                    t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{'' if office_id or is_remote else ' active'}">
                    All Offices
                    <span t-attf-class="badge text-bg-primary ms-2" t-out="count_per_office.get('all', '0')"/>
                </a>
                <t t-foreach="offices" t-as="thisoffice">
                    <t t-if="thisoffice">
                        <a t-attf-href="/jobs?office_id=#{thisoffice.id}#{'' if is_remote else current_country_param}#{non_location_params}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link #{' active' if office_id and office_id.id == thisoffice.id else ''}">
                            <t t-if="not (thisoffice.city or thisoffice.country_id)"><span class="fst-italic">No address specified</span></t>
                            <t t-else="">
                                <t t-out="thisoffice.city"/><t t-if="thisoffice.country_id">, <t t-out="thisoffice.country_id.name"/></t>
                            </t>
                            <span t-attf-class="badge #{' text-bg-primary' if office_id and office_id.id == thisoffice.id else ' text-bg-primary'} ms-2" t-out="count_per_office.get(thisoffice, '0')"/>
                        </a>
                    </t>
                    <t t-else="">
                        <a t-attf-href="/jobs?is_remote=1#{non_location_params}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{' active' if is_remote else ''}">
                            Remote
                            <span t-attf-class="badge #{' text-bg-primary' if is_remote else ' text-bg-primary'} ms-2" t-out="count_per_office.get(None, '0')"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Filter - Employment Type -->
<template id="job_filter_by_employment_type" inherit_id="website_hr_recruitment.topbar_customizations" active="False" priority="40">
    <xpath expr="//ul[hasclass('o_jobs_topbar_filters')]" position="inside">
        <t t-set="non_employment_type_params" t-valuef="#{current_country_param}#{current_department_param}#{current_office_param}"/>
        <li t-attf-class="nav-item dropdown w-100 w-md-auto me-2 my-1 flex-fill">
            <button type="button" class="btn btn-light dropdown-toggle w-100" data-bs-toggle="dropdown" id="officesDropdown">
                <i class="fa fa-suitcase"/>
                <t t-if="contract_type_id and 'name' in contract_type_id" t-out="contract_type_id.name"/>
                <t t-elif="is_untyped">Unspecified type</t>
                <t t-else="">All Types</t>
            </button>
            <div class="dropdown-menu w-100 w-md-auto" aria-labelledby="employmentDropdown">
                <a t-attf-href="/jobs?all_employment_types=1#{non_employment_type_params}"
                    t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{'' if contract_type_id or is_untyped else ' active'}">
                    All Types
                    <span t-attf-class="badge text-bg-primary ms-2" t-out="count_per_employment_type.get('all', '0')"/>
                </a>
                <t t-foreach="employment_types" t-as="employment_type">
                    <t t-if="employment_type">
                        <a t-attf-href="/jobs?contract_type_id=#{employment_type.id}#{non_employment_type_params}"
                            t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link #{' active' if contract_type_id and contract_type_id == employment_type.id else ''}">
                            <t t-out="employment_type.name"/>
                            <span t-attf-class="badge #{' text-bg-primary' if contract_type_id and contract_type_id == employment_type.id else ' text-bg-primary'} ms-2" t-out="count_per_employment_type.get(employment_type, '0')"/>
                        </a>
                    </t>
                </t>
                <t t-if="count_per_employment_type.get(None, 0)">
                    <a t-attf-href="/jobs?is_untyped=1#{non_employment_type_params}"
                        t-attf-class="dropdown-item d-flex align-items-center justify-content-between nav-link#{' active' if is_untyped else ''}">
                        Unspecified type
                        <span t-attf-class="badge #{' text-bg-primary' if is_remote else ' text-bg-primary'} ms-2" t-out="count_per_employment_type.get(None, '0')"/>
                    </a>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Search Bar -->
<template id="job_search_bar" inherit_id="website_hr_recruitment.topbar_customizations" active="True">
    <xpath expr="//div[hasclass('o_jobs_topbar_search_bar')]" position="inside">
        <t t-call="website.website_search_box_input">
            <t t-set="_form_classes" t-valuef="w-100"/>
            <t t-set="_classes" t-valuef="my-1"/>
            <t t-set="search_type" t-valuef="jobs"/>
            <t t-set="display_description" t-valuef="true"/>
            <input t-if="country_id" type="hidden" name="country_id" t-att-value="country_id.id"/>
            <input t-if="office_id" type="hidden" name="office_id" t-att-value="office_id.id"/>
            <input t-if="department_id" type="hidden" name="department_id" t-att-value="department_id.id"/>
            <input t-if="contract_type_id" type="hidden" name="contract_type_id" t-att-value="contract_type_id.id"/>
            <input t-if="is_remote" type="hidden" name="is_remote" value="1"/>
            <input t-if="is_other_department" type="hidden" name="is_other_department" value="1"/>
        </t>
    </xpath>
</template>

<!-- Right Side Bar -->
<template id="job_right_side_bar" inherit_id="website_hr_recruitment.index" active="True" name="Right Side Bar">
    <xpath expr="//div[@id='jobs_grid']" position="after">
        <div class="col-lg-4 oe_structure oe_empty" id="jobs_grid_right">
            <section class="">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-12 pb24">
                            <img src="/website_hr_recruitment/static/src/img/job_image_8.jpg" class="img-fluid" alt="About us"/>
                            <p class="mt24 mb8">
                                We are a team of passionate people whose goal is to improve everyone's life through disruptive products.
                                We build great products to solve your business problems.
                            </p>
                            <div class="s_hr pt16 pb16" data-snippet="s_hr" data-name="Separator">
                                <hr class="w-100 mx-auto" style="border-top-color: var(--400)  !important;"/>
                            </div>
                            <div id="connect">
                                <ul class="list-unstyled text-nowrap">
                                    <li><i class="fa fa-comment fa-fw me-2"></i><span><a href="/contactus">Contact us</a></span></li>
                                    <li><i class="fa fa-envelope fa-fw me-2"></i><span><a href="mailto:info@yourcompany.example.com">info@yourcompany.example.com</a></span></li>
                                    <li><i class="fa fa-phone fa-fw me-2"></i><span class="o_force_ltr"><a href="tel:+1 (650) 555-0187">+1 (650) 555-0187</a></span></li>
                                </ul>
                                <!-- Social media -->
                                <div class="s_social_media text-start" data-name="Social Media" data-snippet="s_social_media">
                                    <h5 class="s_social_media_title d-none">Follow us</h5>
                                    <a href="/website/social/facebook" class="s_social_media_facebook text-decoration-none" target="_blank" aria-label="Facebook">
                                        <i class="fa fa-facebook rounded-empty-circle btn btn-outline-primary shadow-sm"></i>
                                    </a>
                                    <a href="/website/social/twitter" class="s_social_media_twitter text-decoration-none" target="_blank" aria-label="X">
                                        <i class="fa fa-twitter rounded-empty-circle btn btn-outline-primary shadow-sm"></i>
                                    </a>
                                    <a href="/website/social/linkedin" class="s_social_media_linkedin text-decoration-none" target="_blank" aria-label="LinkedIn">
                                        <i class="fa fa-linkedin rounded-empty-circle btn btn-outline-primary shadow-sm"></i>
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_pages_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<record id="job_pages_tree_view" model="ir.ui.view">
    <field name="name">Job Pages List</field>
    <field name="model">hr.job</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="hr.view_hr_job_tree"/>
    <field name="arch" type="xml">
        <xpath expr="//list" position="attributes">
            <attribute name="js_class">website_pages_list</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
            <attribute name="multi_edit">1</attribute>
        </xpath>

        <field name="name" position="after">
            <field name="website_url"/>
            <field name="company_id" column_invisible="True"/>
        </field>
        <xpath expr="//list">
            <field name="is_seo_optimized"/>
            <field name="is_published" position="move"/>

            <field name="website_id" groups="website.group_multi_website"/>
        </xpath>
    </field>
</record>

<record id="job_pages_kanban_view" model="ir.ui.view">
    <field name="name">Job Pages Kanban</field>
    <field name="model">hr.job</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="hr_job_website_inherit"/>
    <field name="arch" type="xml">
        <kanban position="attributes">
            <attribute name="js_class">website_pages_kanban</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </kanban>
        <xpath expr="//div[hasclass('o_kanban_card_header_title')]" position="inside">
            <div class="text-muted fw-bold ps-3">
                <span class="me-3" t-if="record.website_id.value" groups="website.group_multi_website">
                    <i class="fa fa-globe me-1" title="Website"/>
                    <field name="website_id"/>
                </span>
                <field name="is_seo_optimized" widget="boolean"/> SEO Optimized
            </div>
        </xpath>
        <xpath expr="//div[hasclass('o_link_trackers')]" position="replace"/>
    </field>
</record>

<record id="action_job_pages_list" model="ir.actions.act_window">
    <field name="name">Job Pages</field>
    <field name="res_model">hr.job</field>
    <field name="view_mode">list,kanban,form</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'list', 'view_id': ref('job_pages_tree_view')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('job_pages_kanban_view')}),
    ]"/>
    <field name="context">{'create_action': '/jobs/add'}</field>
</record>

<menuitem id="menu_job_pages"
    parent="website.menu_content"
    sequence="70"
    name="Jobs"
    action="action_job_pages_list"
    groups="hr_recruitment.group_hr_recruitment_interviewer"/>

</odoo>

```

