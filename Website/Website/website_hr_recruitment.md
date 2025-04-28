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
    'version': '1.0',
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
        ],
        'website.assets_editor': [
            'website_hr_recruitment/static/src/js/**/*',
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.http import request
from werkzeug.exceptions import NotFound


class WebsiteHrRecruitment(http.Controller):
    def sitemap_jobs(env, rule, qs):
        if not qs or qs.lower() in '/jobs':
            yield {'loc': '/jobs'}

    @http.route([
        '/jobs',
        '/jobs/country/<model("res.country"):country>',
        '/jobs/department/<model("hr.department"):department>',
        '/jobs/country/<model("res.country"):country>/department/<model("hr.department"):department>',
        '/jobs/office/<int:office_id>',
        '/jobs/country/<model("res.country"):country>/office/<int:office_id>',
        '/jobs/department/<model("hr.department"):department>/office/<int:office_id>',
        '/jobs/country/<model("res.country"):country>/department/<model("hr.department"):department>/office/<int:office_id>',
    ], type='http', auth="public", website=True, sitemap=sitemap_jobs)
    def jobs(self, country=None, department=None, office_id=None, **kwargs):
        env = request.env(context=dict(request.env.context, show_address=True, no_tag_br=True))

        Country = env['res.country']
        Jobs = env['hr.job']

        # List jobs available to current UID
        domain = request.website.website_domain()
        job_ids = Jobs.search(domain, order="is_published desc, sequence, no_of_recruitment desc").ids
        # Browse jobs as superuser, because address is restricted
        jobs = Jobs.sudo().browse(job_ids)

        # Default search by user country
        if not (country or department or office_id or kwargs.get('all_countries')):
            country_code = request.session['geoip'].get('country_code')
            if country_code:
                countries_ = Country.search([('code', '=', country_code)])
                country = countries_[0] if countries_ else None
                if not any(j for j in jobs if j.address_id and j.address_id.country_id == country):
                    country = False

        # Filter job / office for country
        if country and not kwargs.get('all_countries'):
            jobs = [j for j in jobs if not j.address_id or j.address_id.country_id.id == country.id]
            offices = set(j.address_id for j in jobs if not j.address_id or j.address_id.country_id.id == country.id)
        else:
            offices = set(j.address_id for j in jobs if j.address_id)

        # Deduce departments and countries offices of those jobs
        departments = set(j.department_id for j in jobs if j.department_id)
        countries = set(o.country_id for o in offices if o.country_id)

        if department:
            jobs = [j for j in jobs if j.department_id and j.department_id.id == department.id]
        if office_id and office_id in [x.id for x in offices]:
            jobs = [j for j in jobs if j.address_id and j.address_id.id == office_id]
        else:
            office_id = False

        # Render page
        return request.render("website_hr_recruitment.index", {
            'jobs': jobs,
            'countries': countries,
            'departments': departments,
            'offices': offices,
            'country_id': country,
            'department_id': department,
            'office_id': office_id,
        })

    @http.route('/jobs/add', type='http', auth="user", website=True)
    def jobs_add(self, **kwargs):
        # avoid branding of website_description by setting rendering_bundle in context
        job = request.env['hr.job'].with_context(rendering_bundle=True).create({
            'name': _('Job Title'),
        })
        return request.redirect("/jobs/detail/%s?enable_editor=1" % slug(job))

    @http.route('''/jobs/detail/<model("hr.job"):job>''', type='http', auth="public", website=True, sitemap=True)
    def jobs_detail(self, job, **kwargs):
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
    </data>
    <data>
        <record id="hr_recruitment.model_hr_applicant" model="ir.model">
            <field name="website_form_key">apply_job</field>
            <field name="website_form_default_field_id" ref="hr_recruitment.field_hr_applicant__description" />
            <field name="website_form_access">True</field>
            <field name="website_form_label">Apply for a Job</field>
        </record>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>hr.applicant</value>
            <value eval="[
                'description',
                'email_from',
                'partner_name',
                'partner_phone',
                'job_id',
                'department_id',
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
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
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
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
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
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_ceo" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_cto" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_trainee" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

    <record id="hr.job_hrm" model="hr.job">
        <field name="website_description" type="html">
            <!-- Description text and ratings -->
            <section class="pt32">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-8 pb32">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
                            <div class="card bg-primary">
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
            <section class="pt40">
                <div class="container">
                    <h2>What's great in the job?</h2>
                    <br/>
                    <div class="row">
                        <div class="col-lg-8 pb40">
                            <ul class="lead">
                                <li>Great team of smart people, in a friendly and open culture</li>
                                <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                                <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                                <li>Expand your knowledge of various business industries</li>
                                <li>Create content that will help our users on a daily basis</li>
                                <li>Real responsibilities and challenges in a fast evolving company</li>
                            </ul>
                        </div>
                        <div class="col-lg-3 offset-lg-1 pb40">
                            <div>
                                <h5>Our Product</h5>
                                <p>Discover our products.</p>
                                <p><a href="/" class="btn btn-outline-primary" target="_blank"><small><b>READ</b></small></a></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- What we offer -->
            <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
                <div class="container">
                    <h2>What We Offer</h2>
                    <br/>
                    <p class="lead">
                        Each employee has a chance to see the impact of his work.
                        You can make a real contribution to the success of the company.
                        <br/>
                        Several activities are often organized all over the year, such as weekly
                        sports sessions, team building events, monthly drink, and much more
                    </p>
                    <div class="row pt16">
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                            <h3>Perks</h3>
                            <p>A full-time position <br/>Attractive salary package.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                            <h3>Trainings</h3>
                            <p>12 days / year, including <br/>6 of your choice.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                            <h3>Sport Activity</h3>
                            <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                        </div>
                        <div class="col-lg-3 text-center pt16 pb32">
                            <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                            <h3>Eat &amp; Drink</h3>
                            <p>Fruit, coffee and <br/>snacks provided.</p>
                        </div>
                    </div>
                </div>
            </section>
            <!-- Photos -->
            <section class="pt24 pb16">
                <div class="container">
                    <div class="row">
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                        </div>
                        <div class="col-md-6 col-lg-3 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                        </div>
                        <div class="col-md-12 col-lg-6 mt16 mb16">
                            <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
                        </div>
                    </div>
                </div>
            </section>
        </field>
    </record>

</odoo>

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Department(models.Model):
    _inherit = 'hr.department'

    def name_get(self):
        # Get department name using superuser, because model is not accessible
        # for portal users
        self_sudo = self.sudo()
        return super(Department, self_sudo).name_get()

```

## File: models\hr_recruitment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import api, fields, models, _
from odoo.tools.translate import html_translate
from odoo.exceptions import UserError


class RecruitmentSource(models.Model):
    _inherit = 'hr.recruitment.source'

    url = fields.Char(compute='_compute_url', string='Url Parameters')

    @api.depends('source_id', 'source_id.name', 'job_id', 'job_id.company_id')
    def _compute_url(self):
        for source in self:
            source.url = urls.url_join(source.job_id.get_base_url(), "%s?%s" % (
                source.job_id.website_url,
                urls.url_encode({
                    'utm_campaign': self.env.ref('hr_recruitment.utm_campaign_job').name,
                    'utm_medium': self.env.ref('utm.utm_medium_website').name,
                    'utm_source': source.source_id.name
                })
            ))


class Applicant(models.Model):

    _inherit = 'hr.applicant'

    def website_form_input_filter(self, request, values):
        if 'partner_name' in values:
            applicant_job = self.env['hr.job'].sudo().search([('id', '=', values['job_id'])]).name if 'job_id' in values else False
            name = '%s - %s' % (values['partner_name'], applicant_job) if applicant_job else _("%s's Application", values['partner_name'])
            values.setdefault('name', name)
        if values.get('job_id'):
            job = self.env['hr.job'].browse(values.get('job_id'))
            if not job.sudo().website_published:
                raise UserError(_("You cannot apply for this job."))
            stage = self.env['hr.recruitment.stage'].sudo().search([
                ('fold', '=', False),
                '|', ('job_ids', '=', False), ('job_ids', '=', values['job_id']),
            ], order='sequence asc', limit=1)
            if stage:
                values['stage_id'] = stage.id
        return values


class Job(models.Model):

    _name = 'hr.job'
    _inherit = ['hr.job', 'website.seo.metadata', 'website.published.multi.mixin']

    def _get_default_website_description(self):
        default_description = self.env.ref("website_hr_recruitment.default_website_description", raise_if_not_found=False)
        return (default_description._render() if default_description else "")

    website_published = fields.Boolean(help='Set if the application is published on the website of the company.')
    website_description = fields.Html('Website description', translate=html_translate, sanitize_attributes=False, default=_get_default_website_description, prefetch=False, sanitize_form=False)

    def _compute_website_url(self):
        super(Job, self)._compute_website_url()
        for job in self:
            job.website_url = "/jobs/detail/%s" % job.id

    def set_open(self):
        self.write({'website_published': False})
        return super(Job, self).set_open()

    def get_backend_menu_id(self):
        return self.env.ref('hr_recruitment.menu_hr_recruitment_root').id

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.addons.http_routing.models.ir_http import url_for


class Website(models.Model):
    _inherit = "website"

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Jobs'), url_for('/jobs'), 'website_hr_recruitment'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_department
from . import hr_recruitment
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_job_public,hr.job.public,hr.model_hr_job,,1,0,0,0
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


    <record id="hr_recruitment.group_hr_recruitment_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_publisher'))]"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="98.162%" x2="0%" y1="1.838%" y2="100%"><stop offset="0%" stop-color="#797DA5"/><stop offset="50.799%" stop-color="#6D7194"/><stop offset="100%" stop-color="#626584"/></linearGradient><path id="d" d="M21.98 54.23h26.04V25.582h-2.603v-3.906a3.906 3.906 0 0 0-3.907-3.906H28.49a3.906 3.906 0 0 0-3.907 3.906v3.906H21.98V54.23zm7.812-31.25h10.416v2.603H29.792V22.98zm26.041 6.51v20.833a3.906 3.906 0 0 1-3.906 3.906h-1.302V25.583h1.302a3.906 3.906 0 0 1 3.906 3.907zM19.375 54.23h-1.302a3.906 3.906 0 0 1-3.906-3.907V29.49a3.906 3.906 0 0 1 3.906-3.907h1.302V54.23z"/><path id="e" d="M21.98 52.23h26.04V23.582h-2.603v-3.906a3.906 3.906 0 0 0-3.907-3.906H28.49a3.906 3.906 0 0 0-3.907 3.906v3.906H21.98V52.23zm7.812-31.25h10.416v2.603H29.792V20.98zm26.041 6.51v20.833a3.906 3.906 0 0 1-3.906 3.906h-1.302V23.583h1.302a3.906 3.906 0 0 1 3.906 3.907zM19.375 52.23h-1.302a3.906 3.906 0 0 1-3.906-3.907V27.49a3.906 3.906 0 0 1 3.906-3.907h1.302V52.23z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M43.024 69H4c-2 0-4-.146-4-4.078v-22.77l15-15.976L26 17h18v7.137h8l4 25.49L43.024 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\website_hr_recruitment_editor.js

```javascript
odoo.define('website_hr_recruitment.form', function (require) {
'use strict';

var core = require('web.core');
var FormEditorRegistry = require('website.form_editor_registry');

const _lt = core._lt;

FormEditorRegistry.add('apply_job', {
    formFields: [{
        type: 'char',
        modelRequired: true,
        name: 'partner_name',
        fillWith: 'name',
        string: _lt('Your Name'),
    }, {
        type: 'email',
        required: true,
        fillWith: 'email',
        name: 'email_from',
        string: _lt('Your Email'),
    }, {
        type: 'char',
        required: true,
        fillWith: 'phone',
        name: 'partner_phone',
        string: _lt('Phone Number'),
    }, {
        type: 'text',
        name: 'description',
        string: _lt('Short Introduction'),
    }, {
        type: 'binary',
        custom: true,
        name: 'Resume',
    }],
    fields: [{
        name: 'job_id',
        type: 'many2one',
        relation: 'hr.job',
        string: _lt('Applied Job'),
    }, {
        name: 'department_id',
        type: 'many2one',
        relation: 'hr.department',
        string: _lt('Department'),
    }],
    successPage: '/job-thank-you',
});

});

```

## File: views\hr_job_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_job_website_inherit" model="ir.ui.view">
        <field name="name">hr.job.form.inherit</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('o_kanban_card_header')]" position="before">
                <field name="website_published" invisible="1"/>
                <div class="ribbon ribbon-top-right" attrs="{'invisible': [('website_published', '=', False)]}">
                    <span class="o_recruitment_purple">Published</span>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('o_primary')]" position="replace">
                <field name="website_url" invisible="1"/>
                <div class="o_primary col-11">
                    <span><a t-attf-href="#{record.website_url.raw_value}" class="mr-2">
                        <t t-esc="record.name.value"/></a>
                    </span>
                </div>
            </xpath>
            <xpath expr="//div[@name='kanban_boxes']/div/div[1]" position="inside">
                <field name="website_url" invisible="1"/>
                <span><t t-if="record.no_of_recruitment.raw_value != 0">
                    <span class="mr-2">
                        <t t-if="record.is_published.raw_value">Published</t>
                        <t t-else="">Not published</t>
                    </span>
                    <field name="is_published" widget='boolean_toggle'/></t>
                </span>
            </xpath> 
        </field>
    </record>

    <record id="view_hr_job_kanban_referal_extends" model="ir.ui.view"> 
        <field name="model">hr.job</field>
        <field name="name">hr.job.view.kanban</field>
        <field name="inherit_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('o_link_trackers')]" position="replace">
                <div class="o_link_trackers col-6">
                    <a role="button" name="%(hr_recruitment.action_hr_job_sources)d" type="action" class="btn btn-sm py-0">
                        <span title='Link Trackers'><i class='fa fa-lg fa-link' role="img" aria-label="Link Trackers"/></span>
                    </a>
                </div>
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
        <field name="name">hr.recruitment.tree.inherit.url</field>
        <field name="model">hr.recruitment.source</field>
        <field name="inherit_id" ref="hr_recruitment.hr_recruitment_source_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="before">
                <field name="url" widget="url"/>
            </xpath>
        </field>
    </record>

    <record id="hr_recruitment_source_kanban_inherit_website" model="ir.ui.view" >
        <field name="name">hr.recruitment.kanban.inherit.website</field>
        <field name="model">hr.recruitment.source</field>
        <field name="inherit_id" ref="hr_recruitment.hr_recruitment_source_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="after">
                <field name="url"/>
            </xpath>
            <xpath expr="//div[hasclass('o_kanban_record_body')]/div" position="before">
                <div class="pull-left">
                    <a role="button" t-att-href="record.url.value" title="share it" class="fa fa-share-alt"/>
                </div>
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
            <xpath expr="//field[@name='no_of_recruitment']" position="after">
                <field name="website_published" string="Is Published"/>
            </xpath>
        </field>
    </record>

    <record id="view_hr_job_form_inherit_website" model="ir.ui.view">
        <field name="name">hr.job.form</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_form"/>
        <field name="arch" type="xml">
            <field name="company_id" position="after">
                <field name="website_id" options="{'no_create': True}" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>

    <record id="view_hr_job_tree_inherit_website" model="ir.ui.view">
        <field name="name">hr.job.tree</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_tree"/>
        <field name="arch" type="xml">
            <field name="department_id" position="after">
                <field name="website_id" groups="website.group_multi_website"/>
            </field>
            <xpath expr="//field[@name='state']" position="after">
                <field name="website_published" string="Published"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_hr_recruitment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="job_edit_options" inherit_id="website.user_navbar" name="Edit Job Options">
    <xpath expr="//div[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'hr.job'" t-set="action" t-value="'hr_recruitment.action_hr_job'"/>
    </xpath>
</template>

<template id="index" name="Jobs">
    <t t-call="website.layout">
        <div id="wrap" class="o_website_hr_recruitment_jobs_list bg-100">
            <div class="oe_structure">
                <section class="pt48 pb32 bg-200">
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-12 text-center">
                                <h2 class="h1 text-secondary">Our Job Offers</h2>
                                <h3 class="text-muted">Join us and help disrupt the enterprise market!</h3>
                                <p>
                                    Join us, we offer you an extraordinary chance to learn, to
                                    develop and to be part of an exciting experience and
                                    team.
                                </p>
                            </div>
                        </div>
                    </div>
                </section>
            </div>

            <div class="container oe_website_jobs">
                <div class="row pt48 pb48">
                    <div class="d-none" id="jobs_grid_left">

                    </div>
                    <div class="col-lg" id="jobs_grid">
                        <div t-if="not jobs">
                            <div class="text-center text-muted">
                                <h3 class="css_editable_hidden"><a t-attf-href="/contactus">Contact us</a> for job opportunities.</h3>
                                <h4 groups="hr_recruitment.group_hr_recruitment_manager">
                                    Create new job pages from the <strong><i>+New</i></strong> top-right button.
                                </h4>
                            </div>
                        </div>
                        <a t-foreach="jobs" t-as="job" t-attf-href="/jobs/detail/#{ slug(job) }" t-attf-class="text-decoration-none#{' mt-3' if job_index else ''}">
                            <div class="card card-default mb32">
                                <div class="card-body" t-att-data-publish="job.website_published and 'on' or 'off'">
                                    <span t-if="not job.website_published" class="badge badge-danger mb8 p-2">unpublished</span>
                                    <h3 class="text-secondary mt0 mb4">
                                        <span t-field="job.name"/>
                                    </h3>
                                    <h5 t-if="job.no_of_recruitment &gt;= 1">
                                        <t t-esc="job.no_of_recruitment"/> open positions
                                    </h5>
                                    <div t-if="editable"
                                       t-field="job.description"
                                       class="mt16 mb0 css_non_editable_mode_hidden"/>
                                    <div t-esc="job.description or ''"
                                        class="mt16 mb0 css_editable_mode_hidden o_website_hr_recruitment_job_description"
                                    />
                                    <div class="o_job_infos mt16">
                                        <span t-field="job.address_id" t-options='{
                                            "widget": "contact",
                                            "fields": ["address"],
                                            "no_tag_br": True
                                            }'/>
                                        <div>
                                            <i class="fa fa-fw fa-clock-o" title="Publication date" role="img" aria-label="Publication date"/><span t-field="job.write_date"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>

<template id="detail" name="Job Detail" track="1">
    <t t-call="website.layout">
        <t t-set="additional_title">Job Detail</t>
        <div id="wrap" class="js_hr_recruitment">
            <!-- Breadcrumb -->
            <section class="bg-200">
                <div class="container">
                    <nav aria-label="breadcrumb">
                        <ol class="breadcrumb pl-0">
                            <li class="breadcrumb-item"><a href="/jobs" class="text-secondary font-weight-bold">Jobs</a></li>
                            <li class="breadcrumb-item active" aria-current="page"><span t-field="job.name"/></li>
                        </ol>
                    </nav>
                </div>
            </section>
            <!-- Job name -->
            <section class="pb32">
                <div class="container">
                    <div class="mt32">
                        <div class="float-right">
                            <a role="button" t-attf-href="/jobs/apply/#{slug(job)}" class="btn btn-primary btn-lg float-right">Apply Now!</a>
                        </div>
                        <h1 t-field="job.name"/>
                        <h5 class="font-weight-light o_not_editable" t-field="job.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["city"],
                            "no_tag_br": True
                        }'/>
                    </div>
                </div>
            </section>

            <div t-field="job.website_description"/>

            <div class="oe_structure">
                <section class="o_job_bottom_bar mt32 mb32">
                    <div class="text-center">
                        <a role="button" t-attf-href="/jobs/apply/#{slug(job)}" class="btn btn-primary btn-lg">Apply Now!</a>
                    </div>
                </section>
            </div>
        </div>
    </t>
</template>

<template id="apply">
    <t t-call="website.layout">
        <t t-set="additional_title">Apply Job</t>

        <div id="wrap"  class="container">
            <h1 class="text-center mt-2">
                Job Application Form
            </h1>
            <h2 t-if="job" class="text-center text-muted">
                <span t-field="job.name"/>
            </h2>
            <span class="hidden" data-for="hr_recruitment_form" t-att-data-values="{'department_id': job and job.department_id.id or '', 'job_id': job and job.id or ''}" />
            <div id="jobs_section">
                <section id="forms" class="s_website_form" data-vcss="001" data-snippet="s_website_form">
                    <div class="container">
                        <form id="hr_recruitment_form" action="/website/form/" method="post" enctype="multipart/form-data" class="o_mark_required" data-mark="*" data-model_name="hr.applicant" data-success-mode="redirect" data-success-page="/job-thank-you" hide-change-model="true">
                            <div class="s_website_form_rows row s_col_no_bgcolor">
                                <div class="form-group col-12 s_website_form_field s_website_form_required s_website_form_model_required" data-type="char" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment1">
                                            <span class="s_website_form_label_content">Your Name</span>
                                            <span class="s_website_form_mark"> *</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="recruitment1" type="text" class="form-control s_website_form_input" name="partner_name" required="" data-fill-with="name"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_required" data-type="email" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment2">
                                            <span class="s_website_form_label_content">Your Email</span>
                                            <span class="s_website_form_mark"> *</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="recruitment2" type="email" class="form-control s_website_form_input" name="email_from" required="" data-fill-with="email"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_required" data-type="char" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment3">
                                            <span class="s_website_form_label_content">Your Phone Number</span>
                                            <span class="s_website_form_mark"> *</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="recruitment3" type="tel" class="form-control s_website_form_input" name="partner_phone" required="" data-fill-with="phone"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field" data-type="text" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment4">
                                            <span class="s_website_form_label_content">Short Introduction</span>
                                        </label>
                                        <div class="col-sm">
                                            <textarea id="recruitment4" class="form-control s_website_form_input" name="description"></textarea>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_custom" data-type="binary" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment5">
                                            <span class="s_website_form_label_content">Resume</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="recruitment5" type="file" class="form-control s_website_form_input" name="Resume"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_dnone">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment6">
                                            <span class="s_website_form_label_content">Job</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="recruitment6" type="hidden" class="form-control s_website_form_input" name="job_id"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_dnone">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="recruitment7">
                                            <span class="s_website_form_label_content">Department</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="recruitment7" type="hidden" class="form-control s_website_form_input" name="department_id"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_submit" data-name="Submit Button">
                                    <div style="width: 200px;" class="s_website_form_label"/>
                                    <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send">Submit</a>
                                    <span id="s_website_form_result"></span>
                                </div>
                            </div>
                        </form>
                    </div>
                </section>
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
                <div class="col-lg-8 pb32">
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
                    <div class="card bg-primary">
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
                    <div class="card bg-primary">
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
                    <div class="card bg-primary">
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
    <section class="pt40">
        <div class="container">
            <h2>What's great in the job?</h2>
            <br/>
            <div class="row">
                <div class="col-lg-8 pb40">
                    <ul class="lead">
                        <li>Great team of smart people, in a friendly and open culture</li>
                        <li>No dumb managers, no stupid tools to use, no rigid working hours</li>
                        <li>No waste of time in enterprise processes, real responsibilities and autonomy</li>
                        <li>Expand your knowledge of various business industries</li>
                        <li>Create content that will help our users on a daily basis</li>
                        <li>Real responsibilities and challenges in a fast evolving company</li>
                    </ul>
                </div>
                <div class="col-lg-3 offset-lg-1 pb40">
                    <div>
                        <h5>Our Product</h5>
                        <p>Discover our products.</p>
                        <p><a href="/" class="btn btn-primary" target="_blank"><small><b>READ</b></small></a></p>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- What we offer -->
    <section class="s_features pt40 pb40 bg-200" data-name="Features" data-snippet="s_features">
        <div class="container">
            <h2>What We Offer</h2>
            <br/>
            <p class="lead">
                Each employee has a chance to see the impact of his work.
                You can make a real contribution to the success of the company.
                <br/>
                Several activities are often organized all over the year, such as weekly
                sports sessions, team building events, monthly drink, and much more
            </p>
            <div class="row pt16">
                <div class="col-lg-3 text-center pt16 pb32">
                    <i class="fa fa-2x fa-gift rounded-circle bg-primary m-3"></i>
                    <h3>Perks</h3>
                    <p>A full-time position <br/>Attractive salary package.</p>
                </div>
                <div class="col-lg-3 text-center pt16 pb32">
                    <i class="fa fa-2x fa-bar-chart rounded-circle bg-primary m-3"></i>
                    <h3>Trainings</h3>
                    <p>12 days / year, including <br/>6 of your choice.</p>
                </div>
                <div class="col-lg-3 text-center pt16 pb32">
                    <i class="fa fa-2x fa-futbol-o rounded-circle bg-primary m-3"></i>
                    <h3>Sport Activity</h3>
                    <p>Play any sport with colleagues, <br/>the bill is covered.</p>
                </div>
                <div class="col-lg-3 text-center pt16 pb32">
                    <i class="fa fa-2x fa-coffee rounded-circle bg-primary m-3"></i>
                    <h3>Eat &amp; Drink</h3>
                    <p>Fruit, coffee and <br/>snacks provided.</p>
                </div>
            </div>
        </div>
    </section>
    <!-- Photos -->
    <section class="pt24 pb16">
        <div class="container">
            <div class="row">
                <div class="col-md-12 col-lg-6 mt16 mb16">
                    <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_3.jpg"/>
                </div>
                <div class="col-md-6 col-lg-3 mt16 mb16">
                    <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_2.jpg"/>
                </div>
                <div class="col-md-6 col-lg-3 mt16 mb16">
                    <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_4.jpg"/>
                </div>
            </div>
            <div class="row">
                <div class="col-md-6 col-lg-3 mt16 mb16">
                    <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_6.jpg"/>
                </div>
                <div class="col-md-6 col-lg-3 mt16 mb16">
                    <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_5.jpg"/>
                </div>
                <div class="col-md-12 col-lg-6 mt16 mb16">
                    <img alt="" class="img img-fluid" src="/website_hr_recruitment/static/src/img/job_image_1.jpg"/>
                </div>
            </div>
        </div>
    </section>
</template>

<record id="thankyou" model="website.page">
    <field name="url">/job-thank-you</field>
    <field name="is_published">True</field>
    <field name="website_indexed" eval="False"/>
    <field name="name">Thank you (Recruitment)</field>
    <field name="type">qweb</field>
    <field name="key">website_hr_recruitment.thankyou</field>
    <field name="cache_time">0</field>
    <field name="arch" type="xml">
        <t name="Thank you (Recruitment)" t-name="website_hr_recruitment.thankyou">
            <t t-call="website.layout">
                <div id="wrap">
                    <div class="oe_structure">
                        <div class="container">
                            <div class="row">
                                <div class="col-lg-12">
                                    <h1 class="text-center">Congratulations!</h1>
                                    <p class="text-center">
                                        Your application has been posted successfully.
                                    </p>
                                </div>
                                <t t-if="request.session.get('form_builder_model_model', '') == 'hr.applicant'">
                                    <t t-set="job" t-value="request.website._website_form_last_record().sudo().job_id"/>
                                    <t t-set="responsible" t-value="job and job.user_id.website_published and job.user_id "/>
                                </t>
                                <t t-if="responsible">
                                    <div class="col-lg-12">
                                        <h3 class="mb32 text-center">Your application has been sent to:</h3>
                                    </div>
                                    <div class="col-lg-1 offset-lg-4">
                                        <p t-field="responsible.avatar_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded-circle d-block mx-auto o_image_64_cover'}"/>
                                    </div>
                                    <div class="col-lg-5 o_responsible_data">
                                        <h4 class="mt0" t-field="responsible.name"/>
                                        <p t-field="responsible.function"/>
                                        <t t-if='responsible.email'>
                                            <i class="fa fa-envelope" role="img" aria-label="Email" title="Email"></i> <a t-attf-href="mailto:#{responsible.email}" t-esc="responsible.email"/>
                                        </t>
                                        <t t-if='responsible.phone'>
                                            <br/><i class="fa fa-phone" role="img" aria-label="Phone" title="Phone"></i> <span t-field="responsible.phone"/>
                                        </t>
                                    </div>
                                    <div class="col-lg-12 mt32 text-center">
                                        <span>
                                            We usually reply between one and three days.<br/>
                                            Feel free to contact him/her if you have further questions.
                                        </span>
                                    </div>
                                </t>
                            </div>
                            <div class="row" id="o_recruitment_thank_cta">
                                <div class="col-lg-12 text-center mt32 mb32">
                                    In the meantime,
                                    <h3 class="mt8 mb32">Look around on our website:</h3>
                                    <a role="button" href="/" class="btn btn-primary btn-lg">Continue To Our Website</a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </t>
    </field>
</record>

<template id="job_filter_by_countries" inherit_id="website_hr_recruitment.index" active="False" customize_show="True" name="Filter by Countries">
    <xpath expr="//div[@id='jobs_grid_left']" position="inside">
        <ul class="nav nav-pills flex-column mb32">
            <li class="nav-item"><a t-attf-href="/jobs#{ '/department/%s' % slug(department_id) if department_id else '' }#{ '/office/%s' % office_id if office_id else '' }?all_countries=1" t-attf-class="nav-link#{'' if country_id else ' active'}">All Countries</a></li>
            <t t-foreach="countries" t-as="country">
                <li class="nav-item">
                    <a t-attf-href="/jobs/country/#{ slug(country) }#{ '/department/%s' % slug(department_id) if department_id else '' }#{ '/office/%s' % office_id if office_id else '' }"
                        t-attf-class="nav-link#{' active' if country_id and country_id.id == country.id else ''}"><span t-field="country.name"/></a>
                </li>
            </t>
        </ul>
    </xpath>
    <xpath expr="//div[@id='jobs_grid_left']" position="attributes">
        <attribute name="class">col-lg-3</attribute>
    </xpath>
</template>

<template id="job_filter_by_departments" inherit_id="website_hr_recruitment.index" active="False" customize_show="True" name="Filter by Departments">
    <xpath expr="//div[@id='jobs_grid_left']" position="inside">
        <ul class="nav nav-pills flex-column mb32">
            <li class="nav-item"><a t-attf-href="/jobs#{ '/country/%s' % slug(country_id) if country_id else '' }#{ '/office/%s' % office_id if office_id else ''}" t-attf-class="nav-link#{'' if department_id else ' active'}">All Departments</a></li>
            <t t-foreach="departments" t-as="department">
                <li class="nav-item">
                    <a t-attf-href="/jobs#{ '/country/%s' % slug(country_id) if country_id else '' }/department/#{ slug(department) }#{ '/office/%s' % office_id if office_id else '' }" t-attf-class="nav-link#{' active' if department_id and department_id.id == department.id else ''}"><span t-field="department.name"/></a>
                </li>
            </t>
        </ul>
    </xpath>
    <xpath expr="//div[@id='jobs_grid_left']" position="attributes">
        <attribute name="class">col-lg-3</attribute>
    </xpath>
</template>

<template id="job_filter_by_offices" inherit_id="website_hr_recruitment.index" active="False" customize_show="True" name="Filter by Offices">
    <xpath expr="//div[@id='jobs_grid_left']" position="inside">
        <ul class="nav nav-pills flex-column mb32">
            <li class="nav-item"><a t-attf-href="/jobs#{ '/country/%s' % slug(country_id) if country_id else '' }#{ '/department/%s' % slug(department_id) if department_id else '' }" t-attf-class="nav-link#{'' if office_id else ' active'}">All Offices</a></li>
            <t t-foreach="offices" t-as="thisoffice">
                <li class="nav-item">
                    <a t-attf-href="/jobs#{ '/country/%s' % slug(country_id) if country_id else '' }#{ '/department/%s' % slug(department_id) if department_id else '' }/office/#{ thisoffice.id }"
                        t-attf-class="nav-link#{' active' if office_id == thisoffice.id else ''}">
                        <span t-field="thisoffice.city"/><t t-if="thisoffice.country_id">,
                            <span t-field="thisoffice.country_id.name"/>
                        </t>
                    </a>
                </li>
            </t>
        </ul>
    </xpath>
    <xpath expr="//div[@id='jobs_grid_left']" position="attributes">
        <attribute name="class">col-lg-3</attribute>
    </xpath>
</template>

<template id="job_right_side_bar" inherit_id="website_hr_recruitment.index" active="True" customize_show="True" name="Right Side Bar">
    <xpath expr="//div[@id='jobs_grid']" position="after">
        <div class="col-lg-3 oe_structure oe_empty" id="jobs_grid_left">
            <section class="">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-12">
                            <img src="/website_hr_recruitment/static/src/img/job_image_1.jpg" class="img-fluid" alt="About us"/>
                            <h4 class="mt24 mb8">About us</h4>
                            <p>
                                We are a team of passionate people whose goal is to improve everyone's life through disruptive products.
                                We build great products to solve your business problems.
                            </p>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </xpath>
</template>

<!-- User Navbar -->
<template id="user_navbar_inherit_website_hr_recruitment" inherit_id="website.user_navbar">
    <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_hr_recruitment']/a" position="attributes">
        <attribute name="href">/jobs/add</attribute>
    </xpath>
    <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_hr_recruitment']" position="attributes">
        <attribute name="name"/>
        <attribute name="t-att-data-module-id"/>
        <attribute name="t-att-data-module-shortdesc"/>
        <attribute name="t-if">is_designer</attribute>
    </xpath>
</template>

</odoo>

```

