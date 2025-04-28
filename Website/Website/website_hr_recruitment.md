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
    'sequence': 142,
    'version': '1.0',
    'summary': 'Manage your online hiring process',
    'description': "This module allows to publish your available job positions on your website and keep track of application submissions easily. It comes as an add-on of *Recruitment* app.",
    'depends': ['website_partner', 'hr_recruitment', 'website_mail', 'website_form'],
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
        job_ids = Jobs.search(domain, order="is_published desc, no_of_recruitment desc").ids
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

    @http.route('''/jobs/detail/<model("hr.job", "[('website_id', 'in', (False, current_website_id))]"):job>''', type='http', auth="public", website=True)
    def jobs_detail(self, job, **kwargs):
        if not job.can_access_from_current_website():
            raise NotFound()

        return request.render("website_hr_recruitment.detail", {
            'job': job,
            'main_object': job,
        })

    @http.route('''/jobs/apply/<model("hr.job", "[('website_id', 'in', (False, current_website_id))]"):job>''', type='http', auth="public", website=True)
    def jobs_apply(self, job, **kwargs):
        if not job.can_access_from_current_website():
            raise NotFound()

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
        <record id="menu_jobs" model="website.menu">
            <field name="name">Jobs</field>
            <field name="url">/jobs</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">50</field>
        </record>
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
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
            </div>
        </div>
    </div>
</section>
        </field>
    </record>

    <record id="hr.job_developer" model="hr.job">
        <field name="is_published">True</field>
        <field name="website_description" type="html">
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
            </div>
        </div>
    </div>
</section>
    </field>
    </record>

    <record id="hr.job_consultant" model="hr.job">
        <field name="is_published">True</field>
        <field name="website_description" type="html">
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
            </div>
        </div>
    </div>
</section>
        </field>
    </record>

    <record id="hr.job_ceo" model="hr.job">
        <field name="website_description" type="html">
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
            </div>
        </div>
    </div>
</section>
        </field>
    </record>

    <record id="hr.job_cto" model="hr.job">
        <field name="website_description" type="html">
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
            </div>
        </div>
    </div>
</section>
        </field>
    </record>

    <record id="hr.job_trainee" model="hr.job">
        <field name="website_description" type="html">
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
            </div>
        </div>
    </div>
</section>
        </field>
    </record>

    <record id="hr.job_hrm" model="hr.job">
        <field name="website_description" type="html">
<section class="mb32">
    <div class="container">
        <div class="row">
            <div class="col-md-10">
                <p class="lead mt0" style="text-align: justify; ">Join our experienced team of Python &amp; JavaScript
                    Developers, and work on an amazing Open Source product! Develop things people care about.</p>
            </div>
        </div>
    </div>
</section>
<section class="o_tasks">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Responsibilities</h5>
                <p>Develop and improve Apps people care about</p>
                <p>Coach small distributed dev teams</p>
                <p>Be responsible of what you develop</p>
                <p>Framework and/or front-end</p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Must Have</h5>
                <p>Several programming languages &amp; Expert in one specific language</p>
                <p>Passion for development</p>
                <p>Quick &amp; Autonomous learner</p>
                <p>Read &amp; Written English</p>
                <p><br/></p>
            </div>
            <div class="col-sm-6 col-md-4">
                <h5 class="fw_semibold text-uppercase">Nice to Have</h5>
                <p>Contribution to Open Source Projects</p>
                <p>Bachelor or Master Degree</p>
                <p>Python, JavaScript</p>
                <p>Linux, GitHub</p>
            </div>

        </div>
    </div>
</section>
<section class="mb32 mt32">
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4">What's great in the job?</h3>
                <ul>
                    <li>You develop an <b>Open Source</b> Software and interact with the community</li>
                    <li>Great <b>team</b> of experienced people in a friendly, supporting and open culture</li>
                    <li>
                        <p>Large apps scope - <b>Diversity</b> in the job, you'll never get bored​</p>
                    </li>
                    <li>No customer deadlines - Time to focus on <b>quality</b>, refactoring and testing</li>
                    <li>No solution architect, no business analysts, no Gantt chart</li>
                    <li>
                        <p>No specific start date, we recruit all year long​</p>
                    </li>
                </ul>
            </div>
            <div class="col-md-8">
                <h3 class="mt0 text-violet-dark display-4"><br/>What do we offer?</h3>
                <ul>
                    <li><b>Flexible </b>working hours</li>
                    <li>Real responsibilities and <b>autonomy</b></li>
                    <li>Advanced <b>training</b>: Technical and functional training sessions (5 weeks)</li>
                    <li>A pleasant and friendly working environment located in a renovated farm in the countryside
                        between Louvain-la-Neuve and Namur</li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="pt0 pb0">
    <div class="container">
        <h3 class="mt0 text-violet-dark">Working Environment</h3>
    </div>
</section>
<section class="s_text_block pb32 pt0">
    <div class="container">
        <div class="row">
            <div class="pt0 col-lg-11 pb0">
                <p>Programming Languages: Python &amp; JavaScript<br/>Database: postgresql (with object relational
                    mapping)<br/>Collaboration platform: GitHub<br/>Development model: open with external
                    community<br/>Framework: Odoo (ORM, Workflows, Report Engine, BI)</p>
            </div>
        </div>
    </div>
</section>
<section
    style="border-style: solid; border-color: rgb(227, 227, 227); -moz-border-top-colors: none; -moz-border-right-colors: none; -moz-border-bottom-colors: none; -moz-border-left-colors: none; border-image: none; border-width: 2px 0px;">
    <div class="container">
        <div class="row">
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Product Complexity:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Personal Evolution:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <h6 class="mb0">Autonomy:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Customer Relationship:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
                <h6 class="mb0">Quality of Product / Tools:</h6>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h6 class="mb0">Team Size:</h6>
                <p class="text-uppercase">100 people</p>
                <h6 class="mb0">Users of the Product:</h6>
                <p class="text-uppercase">4,300,000</p>
                <h6 class="mb0">Release Cycle:</h6>
                <p class="text-uppercase">2 months</p>
                <h6 class="mb0">Company Growth:</h6>
                <p class="text-uppercase">60% Year over Year</p>
                <h6 class="mb0">Company Maturity:</h6>
                <p class="text-uppercase">Profitable</p>
            </div>
            <div class="col-sm-6 col-md-4 mt16 mb16">
                <h3 class="fw_light mt0">Need More Info?</h3>
                <ul>
                    <li><a href="https://www.odoo.com/r/HRGuidebook">Recruitment Guidebook</a></li>
                    <li><a
                            href="https://www.linkedin.com/pulse/20140818150823-57131497-how-i-grew-from-1-to-250-employees-in-5-years?trk=prof-post">The
                            founder’s story</a></li>
                    <li><a href="http://www.slideshare.net/openobject/the-odoo-culture">The Odoo culture</a></li>
                    <li><a href="http://www.slideshare.net/openobject/be-a-team-leader-not-a-manager">Who is your
                            manager?</a></li>
                    <li><a href="https://odoo.com/fr_FR/blog/customer-reviews-6">What people say about us?</a></li>
                </ul>
            </div>
        </div>
    </div>
</section>
<section class="o_perk mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-map-marker"></span>
                <h4 class="mb0 mt8 text-violet-dark">Evolution</h4>
                <p>12 days of pro training per year,<br/> including 6 of your choice, for personal development.</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-calendar"></span>
                <h4 class="mt8 mb0 text-violet-dark">Great Work Place</h4>
                <p>BabyFoot &amp; Video Games <br/>at Lunch Time, <br/>Drinks at the Office, <br/>Several Team Buildings
                </p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-futbol-o"></span>
                <h4 class="mt8 mb0 text-violet-dark">Sport Activity</h4>
                <p>Play any sport with colleagues and the bill is covered</p>
            </div>
            <div class="text-center col-md-3 mt16 mb16"><span class="fa fa-2x fa-coffee"></span>
                <h4 class="mb0 mt8 text-violet-dark">Eat &amp; Drink</h4>
                <p>Fruit Basket, coffee and soup provided all day</p>
            </div>
        </div>
    </div>
</section>
<section class="mt16 mb16">
    <div class="container">
        <div class="row">
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469355"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469360"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469369"/>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469367"/>
            </div>
            <div class="col-sm-6 col-md-3 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469356"/>
            </div>
            <div class="col-sm-12 col-md-6 mt16 mb16">
                <img class="img img-fluid" src="//odoocdn.com/web/image/6469456"/>
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

from odoo import api, fields, models
from odoo.tools.translate import html_translate


class RecruitmentSource(models.Model):
    _inherit = 'hr.recruitment.source'

    url = fields.Char(compute='_compute_url', string='Url Parameters')

    @api.depends('source_id', 'source_id.name', 'job_id')
    def _compute_url(self):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        for source in self:
            source.url = urls.url_join(base_url, "%s?%s" % (source.job_id.website_url,
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
            values.setdefault('name', '%s\'s Application' % values['partner_name'])
        return values


class Job(models.Model):

    _name = 'hr.job'
    _inherit = ['hr.job', 'website.seo.metadata', 'website.published.multi.mixin']

    def _get_default_website_description(self):
        default_description = self.env["ir.model.data"].xmlid_to_object("website_hr_recruitment.default_website_description")
        return (default_description.render() if default_description else "")

    website_description = fields.Html('Website description', translate=html_translate, sanitize_attributes=False, default=_get_default_website_description, prefetch=False)

    def _compute_website_url(self):
        super(Job, self)._compute_website_url()
        for job in self:
            job.website_url = "/jobs/detail/%s" % job.id

    def set_open(self):
        self.write({'website_published': False})
        return super(Job, self).set_open()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_department
from . import hr_recruitment

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
<data>
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

    <record id="hr_recruitment.group_hr_recruitment_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_publisher'))]"/>
    </record>
</data>
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
var FormEditorRegistry = require('website_form.form_editor_registry');

var _t = core._t;

FormEditorRegistry.add('apply_job', {
    defaultTemplateName: 'website_hr_recruitment.default_job_form',
    defaultTemplatePath: '/website_hr_recruitment/static/src/xml/website_hr_recruitment.xml',
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

});

```

## File: static\src\xml\website_hr_recruitment.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>

    <t t-name="website_hr_recruitment.default_job_form">
        <div class="form-group row form-field o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="partner_name">Your Name</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="partner_name" required=""/>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="email_from">Your Email</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="email_from" required="" />
            </div>
        </div>
        <div class="form-group row form-field o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="partner_phone">Phone Number</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="partner_phone" required="" />
            </div>
        </div>
        <div class="form-group row form-field">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="description">Short Introduction</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <textarea class="form-control o_website_form_input" name="description"></textarea>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="Resume">Resume</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="file" class="form-control o_website_form_input" name="Resume" />
            </div>
        </div>
    </t>

</templates>

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
            <xpath expr="//div[@name='kanban_boxes']/div/div[1]" position="inside">
                <field name="website_url" invisible="1"/>
                <span>
                    <a t-attf-href="#{record.website_url.raw_value}">Job Description</a>
                </span>
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
            <xpath expr="//field[@name='email']" position="after">
                <field name="url" widget="url"/>
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
        </field>
    </record>
</odoo>

```

## File: views\website_hr_recruitment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="index" name="Jobs">
    <t t-call="website.layout">
        <div id="wrap">
            <div class="oe_structure">
                <section class="mb16">
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-12 text-center mb16">
                                <h2>Our Job Offers</h2>
                                <h3 class="text-muted">Join us and help disrupt the enterprise market!</h3>
                            </div>
                            <div class="col-lg-12 text-center">
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
                <div class="row">
                    <div class="d-none" id="jobs_grid_left">

                    </div>
                    <div class="col-lg-12" id="jobs_grid">
                        <div t-if="not jobs">
                            <div class="text-center text-muted">
                                <h3 class="css_editable_hidden"><a t-attf-href="/contactus">Contact us</a> for job opportunities.</h3>
                                <h4 groups="hr_recruitment.group_hr_recruitment_manager">
                                    Create new job pages from the <strong><i>+New</i></strong> top-right button.
                                </h4>
                            </div>
                        </div>
                        <ul class="list-unstyled" t-if="jobs">
                            <li t-foreach="jobs" t-as="job" t-attf-class="media#{' mt-3' if job_index else ''}">
                                <div class="media-body" t-att-data-publish="job.website_published and 'on' or 'off'">
                                    <h3>
                                        <a t-attf-href="/jobs/detail/#{ slug(job) }">
                                            <span t-field="job.name"/>
                                        </a>
                                        <small t-if="job.no_of_recruitment &gt; 1">
                                            <t t-esc="job.no_of_recruitment"/> open positions
                                        </small>
                                    </h3>

                                    <span t-field="job.address_id" t-options='{
                                        "widget": "contact",
                                        "fields": ["address"],
                                        "no_tag_br": True
                                        }'/>
                                    <span t-if="not job.website_published" class="badge badge-danger">unpublished</span>
                                    <div class="text-muted">
                                        <i class="fa fa-clock-o" title="Publication date" role="img" aria-label="Publication date"/> <span t-field="job.write_date"/>
                                    </div>
                                </div>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>

<template id="detail" name="Job Detail">
    <t t-call="website.layout">
        <t t-set="additional_title">Job Detail</t>
        <div id="wrap" class="js_hr_recruitment">
            <div class="oe_structure" id="oe_structure_website_hr_recruitment_detail_1"/>

            <!-- Breadcrumb -->
            <section class="mb16 bg-white">
                <div class="container">
                    <div class="float-right">
                        <a role="button" t-attf-href="/jobs/apply/#{job.id}" class="btn btn-primary btn-lg float-right mt32 mb4">Apply Now!</a>
                    </div>
                    <label class="mb0 mt16"><a href="/jobs">Jobs</a></label> /
                    <h1 class="mb0 mt0" t-field="job.name"/>
                    <h4 class="mt0" t-field="job.address_id" t-options='{
                        "widget": "contact",
                        "fields": ["city"],
                        "no_tag_br": True
                    }'/>
                </div>
            </section>

            <div t-field="job.website_description"/>

            <div class="oe_structure">
                <section class="o_job_bottom_bar mt32 mb32">
                    <div class="text-center">
                        <a role="button" t-attf-href="/jobs/apply/#{job.id}" class="btn btn-primary btn-lg">Apply Now!</a>
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

            <div class="row mt-3">
                <section id="forms" class="col">
                    <span class="hidden" data-for="hr_recruitment_form" t-att-data-values="{'department_id': job and job.department_id.id or '', 'job_id': job and job.id or ''}" />
                    <form id="hr_recruitment_form" action="/website_form/" method="post" class="s_website_form" enctype="multipart/form-data" data-model_name="hr.applicant" data-success_page="/job-thank-you" hide-change-model="true">
                        <div class="form-group row form-field o_website_form_required_custom">
                            <div class="col-lg-3 col-md-4 text-right">
                                <label class="col-form-label" for="partner_name">Your Name</label>
                            </div>
                            <div class="col-lg-7 col-md-8">
                                <input type="text" class="form-control o_website_form_input" name="partner_name" required=""/>
                            </div>
                        </div>
                        <div class="form-group row form-field o_website_form_required_custom">
                            <div class="col-lg-3 col-md-4 text-right">
                                <label class="col-form-label" for="email_from">Your Email</label>
                            </div>
                            <div class="col-lg-7 col-md-8">
                                <input type="email" class="form-control o_website_form_input" name="email_from" required=""/>
                            </div>
                        </div>
                        <div class="form-group row form-field o_website_form_required_custom">
                            <div class="col-lg-3 col-md-4 text-right">
                                <label class="col-form-label" for="partner_phone">Your Phone Number</label>
                            </div>
                            <div class="col-lg-7 col-md-8">
                                <input type="text" class="form-control o_website_form_input" name="partner_phone" required=""/>
                            </div>
                        </div>
                        <div class="form-group row form-field">
                            <div class="col-lg-3 col-md-4 text-right">
                                <label class="col-form-label" for="description">Short Introduction</label>
                            </div>
                            <div class="col-lg-7 col-md-8">
                                <textarea class="form-control o_website_form_input" name="description"></textarea>
                            </div>
                        </div>
                        <div class="form-group row form-field o_website_form_custom">
                          <div class="col-lg-3 col-md-4 text-right">
                            <label class="col-form-label" for="Resume">Resume</label>
                          </div>
                          <div class="col-lg-7 col-md-8">
                            <input type="file" class="form-control o_website_form_input" name="Resume"/>
                          </div>
                        </div>
                        <div class="form-group row form-field d-none">
                            <div class="col-lg-3 col-md-4">
                                <label class="col-form-label" for="job_id">Job</label>
                            </div>
                            <div class="col-lg-7 col-md-8">
                                <input type="hidden" class="form-control o_website_form_input" name="job_id"/>
                            </div>
                        </div>
                        <div class="form-group row form-field d-none">
                            <div class="col-lg-3 col-md-4">
                                <label class="col-form-label" for="department_id">Department</label>
                            </div>
                            <div class="col-lg-7 col-md-8">
                                <input type="hidden" class="form-control o_website_form_input" name="department_id"/>
                            </div>
                        </div>
                        <div class="form-group row">
                            <div class="offset-lg-3 offset-md-4 col-md-8 col-lg-7">
                                <a href="#" role="button" class="btn btn-primary btn-lg o_website_form_send">Submit</a>
                                <span id="o_website_form_result"></span>
                            </div>
                        </div>
                    </form>
                </section>
            </div>
        </div>
    </t>
</template>

<template id="default_website_description">
    <!-- Description -->
    <section class="mb32">
        <div class="container">
            <p class="mt0 lead">
                Short summary of the job: A sales job for smart people and can
                learn quickly new industries management practices. You will be
                in charge of the full sales cycle from the opportunity
                qualification to the negotiation, going through astonishing
                product demos.
            </p>
        </div>
    </section>

    <!-- Responsibilities -->
    <section class="o_tasks">
        <div class="container">
            <div class="row">
                <div class="col-md-6 col-lg-4">
                    <h4><strong>Responsibilities</strong></h4>
                    <p>Full sales cycle</p>
                    <p>Achieve monthly targets</p>
                    <p>US + Canada Territory</p>
                </div>
                <div class="col-md-6 col-lg-4">
                    <h4><strong>Challenges</strong></h4>
                    <p>Full sales cycle</p>
                    <p>Achieve monthly targets</p>
                    <p>US + Canada Territory</p>
                </div>
                <div class="col-md-6 col-lg-4">
                    <h4><strong>Must Have</strong></h4>
                    <p>Full sales cycle</p>
                    <p>Achieve monthly targets</p>
                    <p>US + Canada Territory</p>
                </div>
            </div>
        </div>
    </section>

    <!-- What's great -->
    <section class="mb32 mt16">
        <div class="container">
            <h4><strong>What's great in the job?</strong></h4>
            <ul>
                <li>No outbound calls, you get leads and focus on providing value to them</li>
                <li>You sell management software to directors of SMEs: interesting projects and people</li>
                <li>Large apps scope: CRM, MRP, Accounting, Inventory, HR, Project Mgt, etc.</li>
                <li>Direct coordination with functional consultants for qualification and follow ups</li>
                <li>High commissions for good performers</li>
            </ul>
        </div>
    </section>

    <!-- Datas -->
    <section style="border: 0 solid #e3e3e3; border-width: 2px 0;">
        <div class="container">
            <div class="row">
                <div class="col-md-6 col-lg-4 mt16 mb16">
                    <div class="s_rating s_rating_3">
                        <div class="s_rating_stars">
                            <h6 class="mb0">Job Complexity:</h6>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <div class="s_rating_bar"/>
                        </div>
                    </div>
                    <div class="s_rating s_rating_1">
                        <div class="s_rating_stars">
                            <h6 class="mb0">Personal Evolution:</h6>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <div class="s_rating_bar"/>
                        </div>
                    </div>
                    <div class="s_rating s_rating_4">
                        <div class="s_rating_stars">
                            <h6 class="mb0">Variability of the Job:</h6>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <div class="s_rating_bar"/>
                        </div>
                    </div>
                    <div class="s_rating s_rating_5">
                        <div class="s_rating_stars">
                            <h6 class="mb0">Job Security:</h6>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <div class="s_rating_bar"/>
                        </div>
                    </div>
                    <div class="s_rating s_rating_2">
                        <div class="s_rating_stars">
                            <h6 class="mb0">Overachieving Possibilities:</h6>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <i class="fa"/>
                            <div class="s_rating_bar"/>
                        </div>
                    </div>
                </div>
                <div class="col-md-6 col-lg-4 mt16 mb16">
                    <h6 class="mb0">Team / Company Size:</h6>
                    <p class="text-uppercase ">10  / 40 people</p>
                    <h6 class="mb0">Avg Deal Size:</h6>
                    <p class="text-uppercase ">$15k</p>
                    <h6 class="mb0">Sales Cycle:</h6>
                    <p class="text-uppercase ">3 months</p>
                    <h6 class="mb0">Company Growth:</h6>
                    <p class="text-uppercase ">50% YoY</p>
                    <h6 class="mb0">Company Maturity:</h6>
                    <p class="text-uppercase ">Profitable</p>
                </div>
                <div class="col-md-6 col-lg-4 mt16 mb16">
                    <h3 class="mt0">Need More Info?</h3>
                    <ul>
                        <li><a href="#">The founder’s story</a></li>
                        <li><a href="#">The culture</a></li>
                        <li><a href="https://twitter.com/odoo/likes">What people say about us?</a></li>
                    </ul>
                </div>
            </div>
        </div>
    </section>

    <!-- Perks -->
    <section class="o_perk mt16 mb16">
        <div class="container">
            <div class="row">
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mb0 mt8"><span class="fa fa-2x fa-heart"/>
                    Benefits</h4>
                    <p>Healthcare, dental, vision, life insurance, Flexible Spending Account (FSA), Health Savings Account (HSA)</p>
                </div>
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mt8 mb0"><span class="fa fa-2x fa-sun-o"/>
                    PTOs</h4>
                    <p>Vacation, Sick, and paid leaves</p>
                </div>
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mt8 mb0"><span class="fa fa-2x fa-car"/>
                    Save on commute</h4>
                    <p>Pre-tax commuter benefitsbr <br/>(parking and transit) </p>
                    </div>
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mb0 mt8"><span class="fa fa-2x fa-check-circle"/>
                    Discount Programs</h4>
                    <p>Brand-name product and services in categories like travel, electronics, health, fitness, cellular, and more</p>
                </div>
            </div><div class="row">
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mb0 mt8"><span class="fa fa-2x fa-map-marker"/>
                    Prime location</h4>
                    <p>Only a couple blocs from BART, Caltrain, Highway 101, carpool pickup, and Bay Bridge.</p>
                </div>
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mt8 mb0"><span class="fa fa-2x fa-calendar"/>
                    Sponsored Events</h4>
                    <p>Tuesday Dinners, Monthly Lunch Mixers, Monthly Happy Hour, Annual day event</p>
                </div>
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mt8 mb0"><span class="fa fa-2x fa-futbol-o"/>
                    Sport Activity</h4>
                    <p>Play any sport with colleagues and the bill is covered</p>
                    </div>
                <div class="text-center col-lg-3 mt16 mb16">
                    <h4 class="mb0 mt8"><span class="fa fa-2x fa-coffee"/>
                    Eat &amp; Drink</h4>
                    <p>Peet's and Philz coffee provided all day to order and pantry snacks</p>
                </div>
            </div>
        </div>
    </section>

    <!-- Photos -->
    <section class="mt16 mb16">
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
                                        <p t-field="responsible.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded-circle d-block mx-auto o_image_64_cover'}"/>
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

<template id="job_countries" inherit_id="website_hr_recruitment.index" active="False" customize_show="True" name="Filter by Countries">
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
    <xpath expr="//div[@id='jobs_grid']" position="attributes">
        <attribute name="class">col-lg-9</attribute>
    </xpath>
</template>

<template id="job_departments" inherit_id="website_hr_recruitment.index" active="False" customize_show="True" name="Filter by Departments">
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
    <xpath expr="//div[@id='jobs_grid']" position="attributes">
        <attribute name="class">col-lg-9</attribute>
    </xpath>
</template>

<template id="job_offices" inherit_id="website_hr_recruitment.index" active="False" customize_show="True" name="Filter by Offices">
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
    <xpath expr="//div[@id='jobs_grid']" position="attributes">
        <attribute name="class">col-lg-9</attribute>
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
        <attribute name="groups">hr_recruitment.group_hr_recruitment_manager</attribute>
    </xpath>
</template>

<template id="website_hr_recuitment_assets_editor" name="Website HR Recruitment Assets Editor" inherit_id="website.assets_editor">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_hr_recruitment/static/src/js/website_hr_recruitment_editor.js"></script>
    </xpath>
</template>

<template id="assets_tests" name="Website HR Recruitment Assets Tests" inherit_id="web.assets_tests">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_hr_recruitment/static/tests/tours/website_hr_recruitment.js"></script>
    </xpath>
</template>

</odoo>

```

