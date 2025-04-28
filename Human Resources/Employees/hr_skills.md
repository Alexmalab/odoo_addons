# Odoo Module: hr_skills

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Skills Management',
    'category': 'Human Resources/Employees',
    'sequence': 270,
    'version': '1.0',
    'summary': 'Manage skills, knowledge and resume of your employees',
    'description':
        """
Skills and Resume for HR
========================

This module introduces skills and resume management for employees.
        """,
    'depends': ['hr'],
    'data': [
        'security/ir.model.access.csv',
        'security/hr_skills_security.xml',
        'views/hr_views.xml',
        'views/hr_employee_skill_log_views.xml',
        'data/hr_resume_data.xml',
        'data/hr_skill_data.xml',
        'data/ir_actions_server_data.xml',
        'data/report_paperformat.xml',
        'report/hr_employee_skill_report_views.xml',
        'report/hr_employee_cv_report.xml',
        'views/hr_department_views.xml',
        'views/hr_employee_cv_templates.xml',
        'wizard/hr_employee_cv_wizard_views.xml',
    ],
    'demo': [
        'data/hr_skill_demo.xml',
        'data/hr_resume_demo.xml',
        'data/hr.employee.skill.csv',
        'data/hr.resume.line.csv',
    ],
    'installable': True,
    'auto_install': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_skills/static/src/fields/skills_one2many/*',
            'hr_skills/static/src/fields/**/*',
            'hr_skills/static/src/scss/*.scss',
            'hr_skills/static/src/views/skills_list_renderer.js',
            'hr_skills/static/src/xml/**/*',
            'hr_skills/static/src/components/**/*',
        ],
        'web.assets_backend_lazy': [
            'hr_skills/static/src/views/skills_graph.js',
        ],
        'web.qunit_suite_tests': [
            'hr_skills/static/tests/legacy/**/*',
        ],
        'web.assets_unit_tests': [
            'hr_skills/static/tests/**/*',
            ('remove', 'hr_skills/static/tests/legacy/**/*'),
            ('remove', 'hr_skills/static/tests/tours/**/*'),
        ],
        'web.assets_tests': [
            'hr_skills/static/tests/tours/**/*',
        ],
        'web.report_assets_pdf': [
            '/hr_skills/static/src/scss/report_employee_cv.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import _

from odoo.http import request, route, Controller, content_disposition


class HrEmployeeCV(Controller):

    @route(["/print/cv"], type='http', auth='user')
    def print_employee_cv(self, employee_ids='', color_primary='#666666', color_secondary='#666666', **post):
        if not request.env.user._is_internal() or not employee_ids or re.search("[^0-9|,]", employee_ids):
            return request.not_found()

        ids = [int(s) for s in employee_ids.split(',')]
        employees = request.env['hr.employee'].browse(ids)
        if not request.env.user.has_group('hr.group_hr_user') and employees.ids != request.env.user.employee_id.ids:
            return request.not_found()

        resume_type_education = request.env.ref('hr_skills.resume_type_education', raise_if_not_found=False)
        skill_type_language = request.env.ref('hr_skills.hr_skill_type_lang', raise_if_not_found=False)

        report = request.env.ref('hr_skills.action_report_employee_cv', False)

        pdf_content, dummy = request.env['ir.actions.report'].sudo()._render_qweb_pdf(
            report, employees.ids, data={
            'color_primary': color_primary,
            'color_secondary': color_secondary,
            'resume_type_education': resume_type_education,
            'skill_type_language': skill_type_language,
            'show_skills': 'show_skills' in post,
            'show_contact': 'show_contact' in post,
            'show_others': 'show_others' in post,
        })

        if len(employees) == 1:
            report_name = _('Resume %s', employees.name)
        else:
            report_name = _('Resumes')

        pdfhttpheaders = [
            ('Content-Type', 'application/pdf'),
            ('Content-Length', len(pdf_content)),
            ('Content-Disposition', content_disposition(report_name + '.pdf'))
        ]

        return request.make_response(pdf_content, headers=pdfhttpheaders)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\hr.employee.skill.csv

```csv
id,employee_id:id,skill_id:id,skill_type_id:id,skill_level_id:id
employee_skill_admin_spark,hr.employee_admin,hr_skill_spark,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_admin_french,hr.employee_admin,hr_skill_french,hr_skill_type_lang,hr_skill_level_a1
employee_skill_admin_english,hr.employee_admin,hr_skill_english,hr_skill_type_lang,hr_skill_level_a2
employee_skill_al_analytics,hr.employee_al,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_al_digital_ad,hr.employee_al,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_al_public,hr.employee_al,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_al_com,hr.employee_al,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_al_french,hr.employee_al,hr_skill_french,hr_skill_type_lang,hr_skill_level_c1
employee_skill_al_nosql,hr.employee_al,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_beginner
employee_skill_al_django,hr.employee_al,hr_skill_django,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_al_python,hr.employee_al,hr_skill_python,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_mit_email,hr.employee_mit,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_mit_public,hr.employee_mit,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_mit_cms,hr.employee_mit,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_mit_com,hr.employee_mit,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_mit_js,hr.employee_mit,hr_skill_js,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_niv_email,hr.employee_niv,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_niv_public,hr.employee_niv,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_niv_c,hr.employee_niv,hr_skill_c,hr_skill_type_dev,hr_skill_level_expert
employee_skill_niv_android,hr.employee_niv,hr_skill_android,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_niv_nosql,hr.employee_niv,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_beginner
employee_skill_stw_com,hr.employee_stw,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_stw_digital_ad,hr.employee_stw,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_chs_digital_ad,hr.employee_chs,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_chs_email,hr.employee_chs,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_chs_arabic,hr.employee_chs,hr_skill_arabic,hr_skill_type_lang,hr_skill_level_c1
employee_skill_qdp_com,hr.employee_qdp,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_qdp_email,hr.employee_qdp,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_qdp_analytics,hr.employee_qdp,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_qdp_nosql,hr.employee_qdp,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_qdp_js,hr.employee_qdp,hr_skill_js,hr_skill_type_dev,hr_skill_level_expert
employee_skill_qdp_bengali,hr.employee_qdp,hr_skill_bengali,hr_skill_type_lang,hr_skill_level_b2
employee_skill_qdp_english,hr.employee_qdp,hr_skill_english,hr_skill_type_lang,hr_skill_level_b1
employee_skill_fme_spark,hr.employee_fme,hr_skill_spark,hr_skill_type_dev,hr_skill_level_beginner
employee_skill_fme_com,hr.employee_fme,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_fpi_django,hr.employee_fpi,hr_skill_django,hr_skill_type_dev,hr_skill_level_expert
employee_skill_fpi_cms,hr.employee_fpi,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jth_hadoop,hr.employee_jth,hr_skill_hadoop,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jth_nosql,hr.employee_jth,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_jth_c,hr.employee_jth,hr_skill_c,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_vad_sql,hr.employee_vad,hr_skill_sql,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_vad_js,hr.employee_vad,hr_skill_js,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_vad_spark,hr.employee_vad,hr_skill_spark,hr_skill_type_dev,hr_skill_level_expert
employee_skill_vad_python,hr.employee_vad,hr_skill_python,hr_skill_type_dev,hr_skill_level_expert
employee_skill_vad_french,hr.employee_vad,hr_skill_french,hr_skill_type_lang,hr_skill_level_a2
employee_skill_vad_public,hr.employee_vad,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_vad_cms,hr.employee_vad,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_vad_analytics,hr.employee_vad,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_vad_digital_ad,hr.employee_vad,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_han_bengali,hr.employee_han,hr_skill_bengali,hr_skill_type_lang,hr_skill_level_b2
employee_skill_han_python,hr.employee_han,hr_skill_python,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_han_react,hr.employee_han,hr_skill_react,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_han_analytics,hr.employee_han,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_han_digital_ad,hr.employee_han,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_jve_cms,hr.employee_jve,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jve_email,hr.employee_jve,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jve_digital_ad,hr.employee_jve,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jve_com,hr.employee_jve,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_jve_french,hr.employee_jve,hr_skill_french,hr_skill_type_lang,hr_skill_level_b1
employee_skill_jve_spark,hr.employee_jve,hr_skill_spark,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jve_c,hr.employee_jve,hr_skill_c,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_jve_js,hr.employee_jve,hr_skill_js,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jve_hadoop,hr.employee_jve,hr_skill_hadoop,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_jod_filipino,hr.employee_jod,hr_skill_filipino,hr_skill_type_lang,hr_skill_level_c1
employee_skill_jod_spark,hr.employee_jod,hr_skill_spark,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_jod_sql,hr.employee_jod,hr_skill_sql,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jod_analytics,hr.employee_jod,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jod_public,hr.employee_jod,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_jog_public,hr.employee_jog,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jog_cms,hr.employee_jog,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jog_filipino,hr.employee_jog,hr_skill_filipino,hr_skill_type_lang,hr_skill_level_c1
employee_skill_jog_german,hr.employee_jog,hr_skill_german,hr_skill_type_lang,hr_skill_level_c1
employee_skill_jog_bengali,hr.employee_jog,hr_skill_bengali,hr_skill_type_lang,hr_skill_level_a1
employee_skill_jog_django,hr.employee_jog,hr_skill_django,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_jog_react,hr.employee_jog,hr_skill_react,hr_skill_type_dev,hr_skill_level_beginner
employee_skill_jgo_digital_ad,hr.employee_jgo,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_jgo_public,hr.employee_jgo,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_jgo_analytics,hr.employee_jgo,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jgo_com,hr.employee_jgo,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_lur_english,hr.employee_lur,hr_skill_english,hr_skill_type_lang,hr_skill_level_c2
employee_skill_lur_french,hr.employee_lur,hr_skill_french,hr_skill_type_lang,hr_skill_level_a1
employee_skill_hne_spanish,hr.employee_hne,hr_skill_spanish,hr_skill_type_lang,hr_skill_level_c1
employee_skill_hne_cms,hr.employee_hne,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_hne_public,hr.employee_hne,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_hne_com,hr.employee_hne,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_hne_digital_ad,hr.employee_hne,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_hne_sql,hr.employee_hne,hr_skill_sql,hr_skill_type_dev,hr_skill_level_expert

```

## File: data\hr.resume.line.csv

```csv
id,employee_id:id,name,date_start,date_end,line_type_id:id,description
employee_resume_admin_park_lake_state_school,hr.employee_admin,Park Lake State School,2012-02-27,2012-11-03,resume_type_education,
employee_resume_admin_blue_mountains_grammar_school,hr.employee_admin,Blue Mountains Grammar School,2008-12-09,2011-12-16,resume_type_education,
employee_resume_admin_harrington_park_public_school,hr.employee_admin,Harrington Park Public School,2006-09-21,2008-11-15,resume_type_education,
employee_resume_admin_schultz_inc,hr.employee_admin,Schultz Inc,2009-03-23,,resume_type_experience,"Engineer, electrical"
employee_resume_admin_white_inc,hr.employee_admin,White Inc,2007-05-24,2008-11-22,resume_type_experience,"Designer, television/film set"
employee_resume_admin_greeneorr,hr.employee_admin,Greene-Orr,2008-12-23,2009-09-21,resume_type_experience,Magazine journalist
employee_resume_admin_lewisbailey,hr.employee_admin,Lewis-Bailey,2009-10-22,2011-12-22,resume_type_experience,Civil Service fast streamer
employee_resume_al_bathurst_west_public_school,hr.employee_al,Bathurst West Public School,1997-05-06,1998-03-18,resume_type_education,
employee_resume_al_jones_ltd,hr.employee_al,Jones Ltd,1998-02-05,1999-08-05,resume_type_experience,Energy manager
employee_resume_al_garcia_smith_and_king,hr.employee_al,"Garcia, Smith and King",1998-09-05,,resume_type_experience,Medical illustrator
employee_resume_mit_seymour_p12_college,hr.employee_mit,Seymour P-12 College,2013-08-11,2015-07-01,resume_type_education,
employee_resume_mit_darlington_primary_school,hr.employee_mit,Darlington Primary School,2012-08-08,2013-05-05,resume_type_education,
employee_resume_mit_sutherland_dianella_primary_school,hr.employee_mit,Sutherland Dianella Primary School,2010-03-25,2012-06-27,resume_type_education,
employee_resume_mit_burns_lester_and_cuevas,hr.employee_mit,"Burns, Lester and Cuevas",2012-04-24,2012-05-26,resume_type_experience,Police officer
employee_resume_mit_hill_group,hr.employee_mit,Hill Group,2012-05-28,2014-04-25,resume_type_experience,Glass blower/designer
employee_resume_mit_parker_roberson_and_acosta,hr.employee_mit,"Parker, Roberson and Acosta",2014-06-25,2016-04-25,resume_type_experience,Science writer
employee_resume_mit_robinson_crawford_and_norman,hr.employee_mit,"Robinson, Crawford and Norman",2017-10-23,2019-06-23,resume_type_experience,Psychiatric nurse
employee_resume_niv_kialla_west_primary_school,hr.employee_niv,Kialla West Primary School,2016-08-23,2017-03-26,resume_type_education,
employee_resume_niv_arroyo_ltd,hr.employee_niv,Arroyo Ltd,2017-05-23,2018-09-23,resume_type_experience,Insurance risk surveyor
employee_resume_stw_northern_bay_p12_college,hr.employee_stw,Northern Bay P-12 College,2016-03-25,2017-06-03,resume_type_education,
employee_resume_stw_whitsunday_anglican_school,hr.employee_stw,Whitsunday Anglican School,2014-05-06,2016-03-21,resume_type_education,
employee_resume_stw_tyndale_christian_school,hr.employee_stw,Tyndale Christian School,2011-03-04,2014-03-28,resume_type_education,
employee_resume_stw_green_ltd,hr.employee_stw,Green Ltd,2013-01-31,2015-04-01,resume_type_experience,Arboriculturist
employee_resume_stw_lynchhodges,hr.employee_stw,Lynch-Hodges,2012-12-01,,resume_type_experience,Publishing rights manager
employee_resume_stw_finley_rowe_and_adams,hr.employee_stw,"Finley, Rowe and Adams",2012-12-31,,resume_type_experience,"Copywriter, advertising"
employee_resume_chs_avoca_primary_school,hr.employee_chs,Avoca Primary School,1996-03-04,1996-10-26,resume_type_education,
employee_resume_chs_boyd_wilson_and_moore,hr.employee_chs,"Boyd, Wilson and Moore",1997-11-03,,resume_type_experience,Medical physicist
employee_resume_chs_freeman_williams_and_berger,hr.employee_chs,"Freeman, Williams and Berger",1997-02-02,1997-08-04,resume_type_experience,Human resources officer
employee_resume_chs_hanson_roach_and_jordan,hr.employee_chs,"Hanson, Roach and Jordan",1998-03-04,2003-03-04,resume_type_experience,Geographical information systems officer
employee_resume_chs_davis_plc,hr.employee_chs,Davis PLC,2004-10-05,2016-06-04,resume_type_experience,"Secretary, company"
employee_resume_qdp_parke_state_school,hr.employee_qdp,Parke State School,1997-06-26,1999-03-17,resume_type_education,
employee_resume_qdp_evans_cooper_and_white,hr.employee_qdp,"Evans, Cooper and White",1999-04-26,,resume_type_experience,"Therapist, speech and language"
employee_resume_qdp_rivera_shaw_and_hughes,hr.employee_qdp,"Rivera, Shaw and Hughes",1998-11-26,,resume_type_experience,Landscape architect
employee_resume_qdp_phillips_jones_and_brown,hr.employee_qdp,"Phillips, Jones and Brown",1999-12-27,2001-07-27,resume_type_experience,"Teacher, special educational needs"
employee_resume_qdp_hughes_parker_and_barber,hr.employee_qdp,"Hughes, Parker and Barber",2002-02-24,2007-07-27,resume_type_experience,"Engineer, drilling"
employee_resume_fme_st_michaels_primary_school,hr.employee_fme,St Michael's Primary School,2006-12-22,2009-01-23,resume_type_education,
employee_resume_fme_wodonga_primary_school,hr.employee_fme,Wodonga Primary School,2006-02-21,2006-09-27,resume_type_education,
employee_resume_fme_leinster_school,hr.employee_fme,Leinster School,2003-10-14,2005-11-09,resume_type_education,
employee_resume_fme_russellwebster,hr.employee_fme,Russell-Webster,2006-05-14,2008-11-14,resume_type_experience,"Biochemist, clinical"
employee_resume_fme_lewis_group,hr.employee_fme,Lewis Group,2004-07-14,2005-11-14,resume_type_experience,Sports development officer
employee_resume_fme_johnson_shaw_and_carroll,hr.employee_fme,"Johnson, Shaw and Carroll",2004-07-14,2005-11-14,resume_type_experience,"Engineer, mining"
employee_resume_fpi_st_raphaels_primary_school,hr.employee_fpi,St Raphael's Primary School,2006-08-13,2008-09-30,resume_type_education,
employee_resume_fpi_woodridge_state_school,hr.employee_fpi,Woodridge State School,2005-12-28,2006-08-09,resume_type_education,
employee_resume_fpi_our_lady_star_of_the_sea_school,hr.employee_fpi,Our Lady Star of the Sea School,2003-06-07,2005-12-26,resume_type_education,
employee_resume_fpi_chavez_group,hr.employee_fpi,Chavez Group,2004-04-06,,resume_type_experience,Mental health nurse
employee_resume_fpi_hubbarddean,hr.employee_fpi,Hubbard-Dean,2005-05-07,2007-08-08,resume_type_experience,Conference centre manager
employee_resume_jth_narellan_public_school,hr.employee_jth,Narellan Public School,2004-11-02,2007-10-09,resume_type_education,
employee_resume_jth_wilkinson_plc,hr.employee_jth,Wilkinson PLC,2006-02-01,,resume_type_experience,Architectural technologist
employee_resume_jth_simmonswilcox,hr.employee_jth,Simmons-Wilcox,2005-09-03,2006-12-03,resume_type_experience,IT sales professional
employee_resume_jth_goodman_inc,hr.employee_jth,Goodman Inc,2007-06-03,,resume_type_experience,Analytical chemist
employee_resume_ngh_armidale_city_public_school,hr.employee_ngh,Armidale City Public School,1994-11-19,1997-02-18,resume_type_education,
employee_resume_ngh_craigmore_south_junior_primary_school,hr.employee_ngh,Craigmore South Junior Primary School,1993-05-11,1994-08-24,resume_type_education,
employee_resume_ngh_stanleymendez,hr.employee_ngh,Stanley-Mendez,1993-12-12,1994-05-11,resume_type_experience,Glass blower/designer
employee_resume_ngh_jackson_schwartz_and_aguirre,hr.employee_ngh,"Jackson, Schwartz and Aguirre",1995-04-12,1997-07-12,resume_type_experience,Analytical chemist
employee_resume_vad_wycheproof_p12_college,hr.employee_vad,Wycheproof P-12 College,1999-06-02,2000-12-01,resume_type_education,
employee_resume_vad_christian_outreach_college,hr.employee_vad,Christian Outreach College,1996-08-02,1999-02-01,resume_type_education,
employee_resume_vad_thomas_chirnside_primary_school,hr.employee_vad,Thomas Chirnside Primary School,1995-05-05,1996-07-27,resume_type_education,
employee_resume_vad_loganmartin,hr.employee_vad,Logan-Martin,1996-04-04,1997-07-05,resume_type_experience,Petroleum engineer
employee_resume_vad_gallegos_little_and_walters,hr.employee_vad,"Gallegos, Little and Walters",1996-12-02,1997-08-05,resume_type_experience,"Lecturer, higher education"
employee_resume_han_king_island_district_high_school,hr.employee_han,King Island District High School,2014-08-17,2016-05-06,resume_type_education,
employee_resume_han_elphinstone_primary_school,hr.employee_han,Elphinstone Primary School,2013-07-18,2014-07-29,resume_type_education,
employee_resume_han_william_light_r12_school,hr.employee_han,William Light R-12 School,2011-01-29,2013-03-30,resume_type_education,
employee_resume_han_davis_plc,hr.employee_han,Davis PLC,2012-10-30,2013-08-29,resume_type_experience,"Engineer, production"
employee_resume_han_perezmorgan,hr.employee_han,Perez-Morgan,2013-05-01,,resume_type_experience,Geoscientist
employee_resume_jve_ellinbank_primary_school,hr.employee_jve,Ellinbank Primary School,2003-02-16,2004-05-18,resume_type_education,
employee_resume_jve_talbot_primary_school,hr.employee_jve,Talbot Primary School,2002-07-01,2003-02-09,resume_type_education,
employee_resume_jve_saundersadkins,hr.employee_jve,Saunders-Adkins,2003-07-01,,resume_type_experience,Jewellery designer
employee_resume_jve_davis_and_sons,hr.employee_jve,Davis and Sons,2004-12-28,2006-07-01,resume_type_experience,Health physicist
employee_resume_jve_arnoldcohen,hr.employee_jve,Arnold-Cohen,2003-12-29,,resume_type_experience,Personnel officer
employee_resume_jep_lawson_public_school,hr.employee_jep,Lawson Public School,1998-06-07,2000-02-17,resume_type_education,
employee_resume_jep_trinity_college,hr.employee_jep,Trinity College,1995-08-21,1998-04-10,resume_type_education,
employee_resume_jep_woodend_primary_school,hr.employee_jep,Woodend Primary School,1992-11-22,1995-05-05,resume_type_education,
employee_resume_jep_mcneil_rodriguez_and_warren,hr.employee_jep,"Mcneil, Rodriguez and Warren",1994-11-22,1996-01-22,resume_type_experience,Sub
employee_resume_jep_davis_sanchez_and_miller,hr.employee_jep,"Davis, Sanchez and Miller",1996-05-25,1997-11-22,resume_type_experience,Customer service manager
employee_resume_jep_cole_ltd,hr.employee_jep,Cole Ltd,1994-02-22,,resume_type_experience,Fast food restaurant manager
employee_resume_jep_garcia_and_sons,hr.employee_jep,Garcia and Sons,1998-07-25,1999-12-23,resume_type_experience,Careers information officer
employee_resume_jod_umbakumba_school,hr.employee_jod,Umbakumba School,2009-11-08,2010-09-30,resume_type_education,
employee_resume_jod_wilson_ltd,hr.employee_jod,Wilson Ltd,2011-02-07,2012-01-08,resume_type_experience,Trade union research officer
employee_resume_jog_port_curtis_road_state_school,hr.employee_jog,Port Curtis Road State School,2005-04-10,2006-09-30,resume_type_education,
employee_resume_jog_claremont_college,hr.employee_jog,Claremont College,2004-01-12,2005-03-10,resume_type_education,
employee_resume_jog_mandurah_catholic_college,hr.employee_jog,Mandurah Catholic College,2003-02-08,2003-09-28,resume_type_education,
employee_resume_jog_douglas_thompson_and_conner,hr.employee_jog,"Douglas, Thompson and Conner",2004-01-10,,resume_type_experience,Music therapist
employee_resume_jog_allenkeller,hr.employee_jog,Allen-Keller,2005-07-09,2007-03-11,resume_type_experience,Lexicographer
employee_resume_jgo_tottenham_central_school,hr.employee_jgo,Tottenham Central School,2001-04-13,2002-09-06,resume_type_education,
employee_resume_jgo_galilee_catholic_school,hr.employee_jgo,Galilee Catholic School,2000-08-07,2001-02-16,resume_type_education,
employee_resume_jgo_martin_stanley_and_duncan,hr.employee_jgo,"Martin, Stanley and Duncan",2001-05-07,,resume_type_experience,IT technical support officer
employee_resume_jgo_fox_and_sons,hr.employee_jgo,Fox and Sons,2003-03-07,2005-10-05,resume_type_experience,Merchant navy officer
employee_resume_lur_holy_family_primary_school,hr.employee_lur,Holy Family Primary School,2009-07-16,2012-07-23,resume_type_education,
employee_resume_lur_lindenow_primary_school,hr.employee_lur,Lindenow Primary School,2007-07-08,2009-07-16,resume_type_education,
employee_resume_lur_narrogin_primary_school,hr.employee_lur,Narrogin Primary School,2005-12-14,2007-06-10,resume_type_education,
employee_resume_lur_ramirez_inc,hr.employee_lur,Ramirez Inc,2006-11-13,,resume_type_experience,Glass blower/designer
employee_resume_lur_whitebell,hr.employee_lur,White-Bell,2006-04-15,2006-07-13,resume_type_experience,Sports coach
employee_resume_hne_st_peters_parish_primary_school,hr.employee_hne,St Peter's Parish Primary School,2008-05-18,2008-11-17,resume_type_education,
employee_resume_hne_dandenong_north_primary_school,hr.employee_hne,Dandenong North Primary School,2005-07-20,2008-02-15,resume_type_education,
employee_resume_hne_nortonsilva,hr.employee_hne,Norton-Silva,2007-05-21,2009-09-19,resume_type_experience,"Horticulturist, commercial"

```

## File: data\hr_resume_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="resume_type_experience" model="hr.resume.line.type">
            <field name="name">Experience</field>
            <field name="sequence">1</field>
        </record>

        <record id="resume_type_education" model="hr.resume.line.type">
            <field name="name">Education</field>
            <field name="sequence">2</field>
        </record>

        <record id="resume_type_social_media" model="hr.resume.line.type">
            <field name="name">Social Media</field>
            <field name="sequence">3</field>
        </record>
    </data>

</odoo>

```

## File: data\hr_resume_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <!-- Resume -->
    <record id="employee_resume_line_admin_1" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Université Libre de Bruxelles - Polytechnique</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-12)).strftime('%Y-09-17')"/>
        <field name="date_end" eval="(datetime.now()+relativedelta(years=-7)).strftime('%Y-09-10')"/>
        <field name="line_type_id" ref="resume_type_education"/>
        <field name="description">Master in Electrical engineering
            Master thesis: Better grid management and control through machine learning</field>
    </record>

    <record id="employee_resume_line_admin_2" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Saint-Joseph School</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-18)).strftime('%Y-09-01')"/>
        <field name="date_end" eval="(datetime.now()+relativedelta(years=-12)).strftime('%Y-06-30')"/>
        <field name="line_type_id" ref="resume_type_education"/>
        <field name="description">Science &amp; math</field>
    </record>

    <record id="employee_resume_line_admin_4" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Odoo SA</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-11-01')"/>
        <field name="line_type_id" ref="resume_type_experience"/>
        <field name="description">Job position: Development team leader
- Supported technical operations with investigating and correcting varied production support issues (Java, Perl, Shell scripts, SQL).
- Led quality assurance planning for multiple concurrent projects relative to overall system architecture or trading system changes/new developments.
- Configured and released business critical alpha and risk models using MATLAB and SQL with inputs from Portfolio Managers.
        </field>
    </record>

    <record id="employee_resume_line_admin_3" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Burtho Inc.</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-7)).strftime('%Y-09-10')"/>
        <field name="date_end" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-09-10')"/>
        <field name="line_type_id" ref="resume_type_experience"/>
        <field name="description">Job position: Product manager
- Coordinated and managed software deployment across five system environments from development to production.
- Developed stored procedures to assist Java level programming efforts.
- Developed multiple renewable energy plant architectures, both commercial installations and defense-related.
        </field>
    </record>

    <record id="resume_type_side_projects" model="hr.resume.line.type">
        <field name="name">Side Projects</field>
        <field name="sequence">10</field>
    </record>

    <record id="employee_resume_line_admin_5" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Encryption/decryption</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-11-01')"/>
        <field name="line_type_id" ref="resume_type_side_projects"/>
        <field name="description">Allows to encrypt/decrypt plain text or files. Available as a web app or as an API.</field>
    </record>

    <record id="employee_resume_line_admin_6" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Finance forecaster</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-1)).strftime('%Y-11-01')"/>
        <field name="line_type_id" ref="resume_type_side_projects"/>
        <field name="description">Enter your finance data and the app tries to forecast what will be your future incomes/expenses. The application uses machine learning to train itself.</field>
    </record>

    <record id="employee_resume_line_admin_7" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Map Generator</field>
        <field name="date_start" eval="datetime.now().strftime('%Y-11-01')"/>
        <field name="line_type_id" ref="resume_type_side_projects"/>
        <field name="description">A 2D/3D map generator for incremental games.</field>
    </record>
</data>

    <record id="hr.employee_admin" model="hr.employee">
        <field name="study_field">Civil Engineering: Applied Mathematics</field>
        <field name="study_school">Université Catholique de Louvain (UCL)</field>
    </record>
</odoo>

```

## File: data\hr_skill_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="hr_skill_type_lang" model="hr.skill.type">
            <field name="name">Languages</field>
        </record>
        <record id="hr_skill_type_softskill" model="hr.skill.type">
            <field name="name">Soft Skills</field>
        </record>
        
        <!-- Skills -->
        <!-- Languages -->
        <record id="hr_skill_french" model="hr.skill">
            <field name="name">French</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_spanish" model="hr.skill">
            <field name="name">Spanish</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_english" model="hr.skill">
            <field name="name">English</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_german" model="hr.skill">
            <field name="name">German</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_filipino" model="hr.skill">
            <field name="name">Filipino</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_arabic" model="hr.skill">
            <field name="name">Arabic</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_bengali" model="hr.skill">
            <field name="name">Bengali</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_mandarin_chinese" model="hr.skill">
            <field name="name">Mandarin Chinese</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_wu_chinese" model="hr.skill">
            <field name="name">Wu Chinese</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_hindi" model="hr.skill">
            <field name="name">Hindi</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_russian" model="hr.skill">
            <field name="name">Russian</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_portuguese" model="hr.skill">
            <field name="name">Portuguese</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_indonesian" model="hr.skill">
            <field name="name">Indonesian</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_urdu" model="hr.skill">
            <field name="name">Urdu</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_japanese" model="hr.skill">
            <field name="name">Japanese</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_punjabi" model="hr.skill">
            <field name="name">Punjabi</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_javanese" model="hr.skill">
            <field name="name">Javanese</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_telugu" model="hr.skill">
            <field name="name">Telugu</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_turkish" model="hr.skill">
            <field name="name">Turkish</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_korean" model="hr.skill">
            <field name="name">Korean</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_marathi" model="hr.skill">
            <field name="name">Marathi</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>

        <!-- Soft Skills -->
        <record id="hr_skill_communication" model="hr.skill">
            <field name="name">Communication</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_teamwork" model="hr.skill">
            <field name="name">Teamwork</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_problem_solving" model="hr.skill">
            <field name="name">Problem-Solving</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_time_management" model="hr.skill">
            <field name="name">Time Management</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_critical_thinking" model="hr.skill">
            <field name="name">Critical Thinking</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_decision_making" model="hr.skill">
            <field name="name">Decision-Making</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_organizational" model="hr.skill">
            <field name="name">Organizational</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_stress_management" model="hr.skill">
            <field name="name">Stress management</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_adaptability" model="hr.skill">
            <field name="name">Adaptability</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_conflict_management" model="hr.skill">
            <field name="name">Conflict Management</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_leadership" model="hr.skill">
            <field name="name">Leadership</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_creativity" model="hr.skill">
            <field name="name">Creativity</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_resourcefulness" model="hr.skill">
            <field name="name">Resourcefulness</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_persuasion" model="hr.skill">
            <field name="name">Persuasion</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_openness_to_criticism" model="hr.skill">
            <field name="name">Openness to criticism</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>

        <!--Skill Levels-->
        <!--Languages -->
        <record id="hr_skill_level_a1" model="hr.skill.level">
            <field name="name">A1</field>
            <field name="level_progress">10</field>
            <field name="default_level">1</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_a2" model="hr.skill.level">
            <field name="name">A2</field>
            <field name="level_progress">40</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_b1" model="hr.skill.level">
            <field name="name">B1</field>
            <field name="level_progress">60</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_b2" model="hr.skill.level">
            <field name="name">B2</field>
            <field name="level_progress">75</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_c1" model="hr.skill.level">
            <field name="name">C1</field>
            <field name="level_progress">85</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_c2" model="hr.skill.level">
            <field name="name">C2</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skill_type_lang"/>
        </record>

        <!--Soft Skills-->
        <record id="hr_skill_level_beginner_softskill" model="hr.skill.level">
            <field name="name">Beginner</field>
            <field name="default_level">1</field>
            <field name="level_progress">15</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_elementary_softskill" model="hr.skill.level">
            <field name="name">Elementary</field>
            <field name="level_progress">25</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_intermediate_softskill" model="hr.skill.level">
            <field name="name">Intermediate</field>
            <field name="level_progress">50</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_advanced_softskill" model="hr.skill.level">
            <field name="name">Advanced</field>
            <field name="level_progress">80</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_expert_softskill" model="hr.skill.level">
            <field name="name">Expert</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
    </data>
</odoo>

```

## File: data\hr_skill_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <!--Skill Types-->
    <record id="hr_skill_type_dev" model="hr.skill.type">
        <field name="name">Programming Languages</field>
    </record>
    <record id="hr_skill_type_it" model="hr.skill.type">
        <field name="name">IT</field>
    </record>
    <record id="hr_skill_type_marketing" model="hr.skill.type">
        <field name="name">Marketing</field>
    </record>

    <!--Skill Levels-->
    <!--Programming-->
    <record id="hr_skill_level_beginner" model="hr.skill.level">
        <field name="name">Beginner</field>
        <field name="default_level">1</field>
        <field name="level_progress">15</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_level_elementary" model="hr.skill.level">
        <field name="name">Elementary</field>
        <field name="level_progress">25</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_level_intermediate" model="hr.skill.level">
        <field name="name">Intermediate</field>
        <field name="level_progress">50</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_level_advanced" model="hr.skill.level">
        <field name="name">Advanced</field>
        <field name="level_progress">80</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_level_expert" model="hr.skill.level">
        <field name="name">Expert</field>
        <field name="level_progress">100</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>

    <!--Marketing-->
    <record id="hr_skill_level_ml1" model="hr.skill.level">
        <field name="name">L1</field>
        <field name="default_level">1</field>
        <field name="level_progress">25</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_level_ml2" model="hr.skill.level">
        <field name="name">L2</field>
        <field name="level_progress">50</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_level_ml3" model="hr.skill.level">
        <field name="name">L3</field>
        <field name="level_progress">75</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_level_ml4" model="hr.skill.level">
        <field name="name">L4</field>
        <field name="level_progress">100</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>

    <!--IT-->
    <record id="hr_skill_level_beginner_it" model="hr.skill.level">
        <field name="name">Beginner</field>
        <field name="default_level">1</field>
        <field name="level_progress">15</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_level_elementary_it" model="hr.skill.level">
        <field name="name">Elementary</field>
        <field name="level_progress">25</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_level_intermediate_it" model="hr.skill.level">
        <field name="name">Intermediate</field>
        <field name="level_progress">50</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_level_advanced_it" model="hr.skill.level">
        <field name="name">Advanced</field>
        <field name="level_progress">80</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_level_expert_it" model="hr.skill.level">
        <field name="name">Expert</field>
        <field name="level_progress">100</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>

    <!-- Skills -->
    <!-- Programming -->
    <record id="hr_skill_js" model="hr.skill">
        <field name="name">Javascript</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_python" model="hr.skill">
        <field name="name">Python</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_c" model="hr.skill">
        <field name="name">C/C++</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_android" model="hr.skill">
        <field name="name">Android</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_hadoop" model="hr.skill">
        <field name="name">Hadoop</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_spark" model="hr.skill">
        <field name="name">Spark</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_react" model="hr.skill">
        <field name="name">React</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_django" model="hr.skill">
        <field name="name">Django</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_sql" model="hr.skill">
        <field name="name">RDMS</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_nosql" model="hr.skill">
        <field name="name">NoSQL</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_go" model="hr.skill">
        <field name="name">Go</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_java" model="hr.skill">
        <field name="name">Java</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_kotlin" model="hr.skill">
        <field name="name">Kotlin</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_php" model="hr.skill">
        <field name="name">PHP</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_csharp" model="hr.skill">
        <field name="name">C#</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_swift" model="hr.skill">
        <field name="name">Swift</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_r" model="hr.skill">
        <field name="name">R</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_ruby" model="hr.skill">
        <field name="name">Ruby</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_matlab" model="hr.skill">
        <field name="name">Matlab</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_typescript" model="hr.skill">
        <field name="name">TypeScript</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_scala" model="hr.skill">
        <field name="name">Scala</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_html" model="hr.skill">
        <field name="name">HTML</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_css" model="hr.skill">
        <field name="name">CSS</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_nosql" model="hr.skill">
        <field name="name">NoSQL</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_rust" model="hr.skill">
        <field name="name">Rust</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>
    <record id="hr_skill_perl" model="hr.skill">
        <field name="name">Perl</field>
        <field name="skill_type_id" ref="hr_skill_type_dev"/>
    </record>

    <!-- Marketing -->
    <record id="hr_skill_com" model="hr.skill">
        <field name="name">Communication</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_analytics" model="hr.skill">
        <field name="name">Analytics</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_digital_ad" model="hr.skill">
        <field name="name">Digital advertising</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_public" model="hr.skill">
        <field name="name">Public Speaking</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_cms" model="hr.skill">
        <field name="name">CMS</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>
    <record id="hr_skill_email" model="hr.skill">
        <field name="name">Email Marketing</field>
        <field name="skill_type_id" ref="hr_skill_type_marketing"/>
    </record>

    <!-- IT -->
    <record id="hr_skill_web_development" model="hr.skill">
        <field name="name">Web Development</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_database_management" model="hr.skill">
        <field name="name">Database Management</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_cloud_computing" model="hr.skill">
        <field name="name">Cloud computing</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_network_administration" model="hr.skill">
        <field name="name">Network administration</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_cybersecurity" model="hr.skill">
        <field name="name">Cybersecurity</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_devops" model="hr.skill">
        <field name="name">DevOps</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_machine_learning" model="hr.skill">
        <field name="name">Machine Learning (AI)</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_data_analysis" model="hr.skill">
        <field name="name">Data analysis/visualization</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_agile_scrum" model="hr.skill">
        <field name="name">Agile and Scrum methodologies</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_mobile_app_development" model="hr.skill">
        <field name="name">Mobile app development</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_project_management" model="hr.skill">
        <field name="name">Project Management</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_system_administration" model="hr.skill">
        <field name="name">System Administration (Linux, Windows)</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_virtualization_containerization" model="hr.skill">
        <field name="name">Virtualization and Containerization</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_it_support" model="hr.skill">
        <field name="name">IT support</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_it_infrastructure_architecture" model="hr.skill">
        <field name="name">IT infrastructure and architecture</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_it_service_management" model="hr.skill">
        <field name="name">IT service management (ITSM)</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_big_data_technologies" model="hr.skill">
        <field name="name">Big data technologies (Hadoop,Spark)</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_iot_embedded_systems" model="hr.skill">
        <field name="name">IoT and embedded systems</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
    <record id="hr_skill_it_governance_compliance" model="hr.skill">
        <field name="name">IT governance and compliance (GDPR,HIPAA,...)</field>
        <field name="skill_type_id" ref="hr_skill_type_it"/>
    </record>
</data>
</odoo>

```

## File: data\ir_actions_server_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_open_skills_log_department" model="ir.actions.server">
        <field name="name">Skill History Report</field>
        <field name="model_id" ref="hr.model_hr_department"/>
        <field name="binding_model_id" ref="hr.model_hr_department"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">
action = env['ir.actions.act_window']._for_xml_id('hr_skills.action_hr_employee_skill_log_department')
action['domain'] = [('department_id', '=', record.id)]
        </field>
    </record>

    <record id="action_print_employees_cv" model="ir.actions.server">
        <field name="name">Print Resume</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">list,form</field>
        <field name="binding_type">report</field>
        <field name="state">code</field>
        <field name="code">
action = env['ir.actions.act_window']._for_xml_id('hr_skills.action_hr_employee_cv_wizard')
        </field>
    </record>
</odoo>
```

## File: data\report_paperformat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="paperformat_resume" model="report.paperformat">
        <field name="name">Resume</field>
        <field name="default" eval="True"/>
        <field name="format">A4</field>
        <field name="orientation">Portrait</field>
        <field name="margin_top">12</field>
        <field name="margin_bottom">12</field>
        <field name="margin_left">5</field>
        <field name="margin_right">5</field>
        <field name="header_line" eval="False"/>
        <field name="header_spacing">20</field>
        <field name="dpi">90</field>
    </record>
</odoo>

```

## File: data\scenarios\hr_skills_scenario.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Skill Type -->
        <record id="hr_skill_type_it" model="hr.skill.type" forcecreate="1">
            <field name="name">IT</field>
        </record>
        <record id="hr_skill_type_dev" model="hr.skill.type" forcecreate="1">
            <field name="name">Programming Languages</field>
        </record>
        <record id="hr_skill_type_marketing" model="hr.skill.type" forcecreate="1">
            <field name="name">Marketing</field>
        </record>
        <record id="hr_skill_type_softskill" model="hr.skill.type" forcecreate="1">
            <field name="name">Soft Skills</field>
        </record>

        <!-- Skill Level -->
        <record id="hr_skill_level_a1" model="hr.skill.level" forcecreate="1">
            <field name="name">A1</field>
            <field name="level_progress">10</field>
            <field name="default_level">1</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_a2" model="hr.skill.level" forcecreate="1">
            <field name="name">A2</field>
            <field name="level_progress">40</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_b1" model="hr.skill.level" forcecreate="1">
            <field name="name">B1</field>
            <field name="level_progress">60</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_b2" model="hr.skill.level" forcecreate="1">
            <field name="name">B2</field>
            <field name="level_progress">75</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_c1" model="hr.skill.level" forcecreate="1">
            <field name="name">C1</field>
            <field name="level_progress">85</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_level_c2" model="hr.skill.level" forcecreate="1">
            <field name="name">C2</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>

        <!--Programming-->
        <record id="hr_skill_level_beginner" model="hr.skill.level" forcecreate="1">
            <field name="name">Beginner</field>
            <field name="default_level">1</field>
            <field name="level_progress">15</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_level_elementary" model="hr.skill.level" forcecreate="1">
            <field name="name">Elementary</field>
            <field name="level_progress">25</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_level_intermediate" model="hr.skill.level" forcecreate="1">
            <field name="name">Intermediate</field>
            <field name="level_progress">50</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_level_advanced" model="hr.skill.level" forcecreate="1">
            <field name="name">Advanced</field>
            <field name="level_progress">80</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_level_expert" model="hr.skill.level" forcecreate="1">
            <field name="name">Expert</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>

        <!--Marketing-->
        <record id="hr_skill_level_ml1" model="hr.skill.level" forcecreate="1">
            <field name="name">L1</field>
            <field name="default_level">1</field>
            <field name="level_progress">25</field>
            <field name="skill_type_id" ref="hr_skill_type_marketing"/>
        </record>
        <record id="hr_skill_level_ml2" model="hr.skill.level" forcecreate="1">
            <field name="name">L2</field>
            <field name="level_progress">50</field>
            <field name="skill_type_id" ref="hr_skill_type_marketing"/>
        </record>
        <record id="hr_skill_level_ml3" model="hr.skill.level" forcecreate="1">
            <field name="name">L3</field>
            <field name="level_progress">75</field>
            <field name="skill_type_id" ref="hr_skill_type_marketing"/>
        </record>
        <record id="hr_skill_level_ml4" model="hr.skill.level" forcecreate="1">
            <field name="name">L4</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skill_type_marketing"/>
        </record>

        <!--Soft Skills-->
        <record id="hr_skill_level_beginner_softskill" model="hr.skill.level" forcecreate="1">
            <field name="name">Beginner</field>
            <field name="default_level">1</field>
            <field name="level_progress">15</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_elementary_softskill" model="hr.skill.level" forcecreate="1">
            <field name="name">Elementary</field>
            <field name="level_progress">25</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_intermediate_softskill" model="hr.skill.level" forcecreate="1">
            <field name="name">Intermediate</field>
            <field name="level_progress">50</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_advanced_softskill" model="hr.skill.level" forcecreate="1">
            <field name="name">Advanced</field>
            <field name="level_progress">80</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_level_expert_softskill" model="hr.skill.level" forcecreate="1">
            <field name="name">Expert</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>

        <!--IT-->
        <record id="hr_skill_level_beginner_it" model="hr.skill.level" forcecreate="1">
            <field name="name">Beginner</field>
            <field name="default_level">1</field>
            <field name="level_progress">15</field>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>
        <record id="hr_skill_level_elementary_it" model="hr.skill.level" forcecreate="1">
            <field name="name">Elementary</field>
            <field name="level_progress">25</field>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>
        <record id="hr_skill_level_intermediate_it" model="hr.skill.level" forcecreate="1">
            <field name="name">Intermediate</field>
            <field name="level_progress">50</field>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>
        <record id="hr_skill_level_advanced_it" model="hr.skill.level" forcecreate="1">
            <field name="name">Advanced</field>
            <field name="level_progress">80</field>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>
        <record id="hr_skill_level_expert_it" model="hr.skill.level" forcecreate="1">
            <field name="name">Expert</field>
            <field name="level_progress">100</field>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>

        <!-- Skill -->
        <record id="hr_skill_agile_scrum" model="hr.skill" forcecreate="1">
            <field name="name">Agile and Scrum methodologies</field>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>
        <record id="hr_skill_english" model="hr.skill" forcecreate="1">
            <field name="name">English</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_python" model="hr.skill" forcecreate="1">
            <field name="name">Python</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_problem_solving" model="hr.skill" forcecreate="1">
            <field name="name">Problem-Solving</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_js" model="hr.skill" forcecreate="1">
            <field name="name">Javascript</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_nosql" model="hr.skill" forcecreate="1">
            <field name="name">NoSQL</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_decision_making" model="hr.skill" forcecreate="1">
            <field name="name">Decision-Making</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_communication" model="hr.skill" forcecreate="1">
            <field name="name">Communication</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="hr_skill_french" model="hr.skill" forcecreate="1">
            <field name="name">French</field>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="hr_skill_analytics" model="hr.skill" forcecreate="1">
            <field name="name">Analytics</field>
            <field name="skill_type_id" ref="hr_skill_type_marketing"/>
        </record>
        <record id="hr_skill_spark" model="hr.skill" forcecreate="1">
            <field name="name">Spark</field>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="hr_skill_leadership" model="hr.skill" forcecreate="1">
            <field name="name">Leadership</field>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>

        <!-- Employee Skill -->
        <record id="employee_eg_skill_it_agile" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_eg"/>
            <field name="skill_id" ref="hr_skill_agile_scrum"/>
            <field name="skill_level_id" ref="hr_skill_level_advanced_it"/>
            <field name="skill_type_id" ref="hr_skill_type_it"/>
        </record>
        <record id="employee_eg_skill_lang_en" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_eg"/>
            <field name="skill_id" ref="hr_skill_english"/>
            <field name="skill_level_id" ref="hr_skill_level_b2"/>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="employee_eg_skill_dev_py" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_eg"/>
            <field name="skill_id" ref="hr_skill_python"/>
            <field name="skill_level_id" ref="hr_skill_level_beginner"/>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="employee_eg_skill_softskill_problem" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_eg"/>
            <field name="skill_id" ref="hr_skill_problem_solving"/>
            <field name="skill_level_id" ref="hr_skill_level_intermediate_softskill"/>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="employee_sj_skill_lang_en" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="skill_id" ref="hr_skill_english"/>
            <field name="skill_level_id" ref="hr_skill_level_b1"/>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="employee_sj_skill_dev_js" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="skill_id" ref="hr_skill_js"/>
            <field name="skill_level_id" ref="hr_skill_level_expert"/>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="employee_sj_skill_dev_nosql" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="skill_id" ref="hr_skill_nosql"/>
            <field name="skill_level_id" ref="hr_skill_level_expert"/>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="employee_sj_skill_softskill_decision" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="skill_id" ref="hr_skill_decision_making"/>
            <field name="skill_level_id" ref="hr_skill_level_intermediate_softskill"/>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="employee_sj_skill_softskill_communication" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="skill_id" ref="hr_skill_communication"/>
            <field name="skill_level_id" ref="hr_skill_level_beginner_softskill"/>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>
        <record id="employee_mw_skill_lang_en" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="skill_id" ref="hr_skill_english"/>
            <field name="skill_level_id" ref="hr_skill_level_c2"/>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="employee_mw_skill_lang_fr" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="skill_id" ref="hr_skill_french"/>
            <field name="skill_level_id" ref="hr_skill_level_a1"/>
            <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
        </record>
        <record id="employee_mw_skill_marketing_analytics" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="skill_id" ref="hr_skill_analytics"/>
            <field name="skill_level_id" ref="hr_skill_level_ml3"/>
            <field name="skill_type_id" ref="hr_skill_type_marketing"/>
        </record>
        <record id="employee_mw_skill_dev_spark" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="skill_id" ref="hr_skill_spark"/>
            <field name="skill_level_id" ref="hr_skill_level_intermediate"/>
            <field name="skill_type_id" ref="hr_skill_type_dev"/>
        </record>
        <record id="employee_mw_skill_softskill_leadership" model="hr.employee.skill" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="skill_id" ref="hr_skill_leadership"/>
            <field name="skill_level_id" ref="hr_skill_level_advanced_softskill"/>
            <field name="skill_type_id" ref="hr_skill_type_softskill"/>
        </record>

        <!-- Resume Line -->
        <record id="employee_resume_line_emp_eg_1" model="hr.resume.line" forcecreate="1">
            <field name="employee_id" ref="hr.employee_eg"/>
            <field name="name">Azure Interior</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-4)).strftime('%Y-09-1')"/>
            <field name="date_end" eval="(datetime.now()+relativedelta(years=-2)).strftime('%Y-08-31')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_experience"/>
            <field name="description">Agile Coach</field>
        </record>
        <record id="employee_resume_line_emp_sj_1" model="hr.resume.line" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="name">Beer &amp; Chips</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-6)).strftime('%Y-03-1')"/>
            <field name="date_end" eval="(datetime.now()+relativedelta(years=-4)).strftime('%Y-12-31')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_experience"/>
            <field name="description">Website master</field>
        </record>
        <record id="employee_resume_line_emp_sj_2" model="hr.resume.line" forcecreate="1">
            <field name="employee_id" ref="hr.employee_sj"/>
            <field name="name">Phillips</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-03-1')"/>
            <field name="date_end" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-12-31')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_experience"/>
            <field name="description">Software developper</field>
        </record>
        <record id="employee_resume_line_emp_mw_1" model="hr.resume.line" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="name">Park Lake State School</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-15)).strftime('%Y-09-1')"/>
            <field name="date_end" eval="(datetime.now()+relativedelta(years=-12)).strftime('%Y-12-1')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_education"/>
        </record>
        <record id="employee_resume_line_emp_mw_2" model="hr.resume.line" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="name">Blue Mountains Grammar School</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-11)).strftime('%Y-08-15')"/>
            <field name="date_end"  eval="(datetime.now()+relativedelta(years=-9)).strftime('%Y-02-1')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_education"/>
        </record>
        <record id="employee_resume_line_emp_mw_3" model="hr.resume.line" forcecreate="1">
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="name">Harrington Park Public School</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-8)).strftime('%Y-04-15')"/>
            <field name="date_end" eval="(datetime.now()+relativedelta(years=-8)).strftime('%Y-12-15')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_education"/>
        </record>
        <record id="employee_resume_line_emp_mw_4" model="hr.resume.line" >
            <field name="employee_id" ref="hr.employee_mw"/>
            <field name="name">Schultz Inc</field>
            <field name="date_start" eval="(datetime.now()+relativedelta(years=-7)).strftime('%Y-01-01')"/>
            <field name="date_end" eval="(datetime.now()+relativedelta(years=-4)).strftime('%Y-12-31')"/>
            <field name="line_type_id" ref="hr_skills.resume_type_experience"/>
            <field name="description">Engineer, electrical</field>
        </record>
        <function model="hr.resume.line" name="write">
            <value model="hr.resume.line" eval="obj().search([
                ('name', '=', 'My Company'), ('employee_id', '=', ref('hr.employee_mw'))
                ], order='id desc', limit=1).id"/>
            <value eval="{'date_start': (datetime.now()+relativedelta(years=-3)).strftime('%Y-01-01')}"/>
        </function>
        <function model="hr.resume.line" name="write">
            <value model="hr.resume.line" eval="obj().search([
                ('name', '=', 'My Company'), ('employee_id', '=', ref('hr.employee_eg'))
                ], order='id desc', limit=1).id"/>
            <value eval="{'date_start': (datetime.now()+relativedelta(years=-1)).strftime('%Y-01-01')}"/>
        </function>
        <function model="hr.resume.line" name="write">
            <value model="hr.resume.line" eval="obj().search([
                ('name', '=', 'My Company'), ('employee_id', '=', ref('hr.employee_sj'))
                ], order='id desc', limit=1).id"/>
            <value eval="{'date_start': (datetime.now()+relativedelta(years=-2)).strftime('%Y-01-01')}"/>
        </function>
    </data>
</odoo>

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools import convert


class Employee(models.Model):
    _inherit = 'hr.employee'

    resume_line_ids = fields.One2many('hr.resume.line', 'employee_id', string="Resume lines")
    employee_skill_ids = fields.One2many('hr.employee.skill', 'employee_id', string="Skills",
        domain=[('skill_type_id.active', '=', True)])
    skill_ids = fields.Many2many('hr.skill', compute='_compute_skill_ids', store=True, groups="hr.group_hr_user")

    @api.depends('employee_skill_ids.skill_id')
    def _compute_skill_ids(self):
        for employee in self:
            employee.skill_ids = employee.employee_skill_ids.skill_id

    @api.model_create_multi
    def create(self, vals_list):
        res = super(Employee, self).create(vals_list)
        if self.env.context.get('salary_simulation'):
            return res
        resume_lines_values = []
        for employee in res:
            line_type = self.env.ref('hr_skills.resume_type_experience', raise_if_not_found=False)
            resume_lines_values.append({
                'employee_id': employee.id,
                'name': employee.company_id.name or '',
                'date_start': employee.create_date.date(),
                'description': employee.job_title or '',
                'line_type_id': line_type and line_type.id,
            })
        self.env['hr.resume.line'].create(resume_lines_values)
        return res

    def write(self, vals):
        res = super().write(vals)
        if 'department_id' in vals:
            self.employee_skill_ids._create_logs()
        return res

    def _load_scenario(self):
        super()._load_scenario()
        demo_tag = self.env.ref('hr_skills.employee_resume_line_emp_eg_1', raise_if_not_found=False)
        if demo_tag:
            return
        convert.convert_file(self.env, 'hr_skills', 'data/scenarios/hr_skills_scenario.xml', None, mode='init', kind='data')

```

## File: models\hr_employee_public.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    resume_line_ids = fields.One2many('hr.resume.line', 'employee_id', string="Resume lines")
    employee_skill_ids = fields.One2many('hr.employee.skill', 'employee_id', string="Skills",
        domain=[('skill_type_id.active', '=', True)])

```

## File: models\hr_employee_skill.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

from collections import defaultdict

class EmployeeSkill(models.Model):
    _name = 'hr.employee.skill'
    _description = "Skill level for an employee"
    _order = "skill_type_id, skill_level_id"
    _rec_name = "skill_id"

    employee_id = fields.Many2one('hr.employee', required=True, ondelete='cascade')
    skill_id = fields.Many2one('hr.skill', compute='_compute_skill_id', store=True, domain="[('skill_type_id', '=', skill_type_id)]", readonly=False, required=True, ondelete='cascade')
    skill_level_id = fields.Many2one('hr.skill.level', compute='_compute_skill_level_id', domain="[('skill_type_id', '=', skill_type_id)]", store=True, readonly=False, required=True, ondelete='cascade')
    skill_type_id = fields.Many2one('hr.skill.type',
                                    default=lambda self: self.env['hr.skill.type'].search([], limit=1),
                                    required=True, ondelete='cascade')
    level_progress = fields.Integer(related='skill_level_id.level_progress')
    color = fields.Integer(related="skill_type_id.color")

    _sql_constraints = [
        ('_unique_skill', 'unique (employee_id, skill_id)', "Two levels for the same skill is not allowed"),
    ]

    @api.constrains('skill_id', 'skill_type_id')
    def _check_skill_type(self):
        for record in self:
            if record.skill_id not in record.skill_type_id.skill_ids:
                raise ValidationError(_("The skill %(name)s and skill type %(type)s doesn't match", name=record.skill_id.name, type=record.skill_type_id.name))

    @api.constrains('skill_type_id', 'skill_level_id')
    def _check_skill_level(self):
        for record in self:
            if record.skill_level_id not in record.skill_type_id.skill_level_ids:
                raise ValidationError(_("The skill level %(level)s is not valid for skill type: %(type)s", level=record.skill_level_id.name, type=record.skill_type_id.name))

    @api.depends('skill_type_id')
    def _compute_skill_id(self):
        for record in self:
            if record.skill_type_id:
                record.skill_id = record.skill_type_id.skill_ids[0] if record.skill_type_id.skill_ids else False
            else:
                record.skill_id = False

    @api.depends('skill_id')
    def _compute_skill_level_id(self):
        for record in self:
            if not record.skill_id:
                record.skill_level_id = False
            else:
                skill_levels = record.skill_type_id.skill_level_ids
                record.skill_level_id = skill_levels.filtered('default_level') or skill_levels[0] if skill_levels else False

    @api.depends('skill_id', 'skill_level_id')
    def _compute_display_name(self):
        for employee_skill in self:
            employee_skill.display_name = f"{employee_skill.skill_id.name}: {employee_skill.skill_level_id.name}"

    def _create_logs(self):
        today = fields.Date.context_today(self)
        employee_skills = self.env['hr.employee.skill'].search([
            ('employee_id', 'in', self.employee_id.ids)
        ])
        employee_skill_logs = self.env['hr.employee.skill.log'].search([
            ('employee_id', 'in', self.employee_id.ids),
        ])

        skills_by_employees = defaultdict(lambda: self.env['hr.employee.skill'])
        for skill in employee_skills:
            skills_by_employees[skill.employee_id.id] |= skill

        logs_by_employees = defaultdict(lambda: self.env['hr.employee.skill.log'])
        for log in employee_skill_logs:
            logs_by_employees[log.employee_id.id] |= log

        skill_to_create_vals = []
        for employee in skills_by_employees:
            employee_logs = logs_by_employees[employee]
            for employee_skill in skills_by_employees[employee]:
                existing_log = employee_logs.filtered(lambda l: l.department_id == employee_skill.employee_id.department_id and l.skill_id == employee_skill.skill_id and l.date == today)
                if existing_log:
                    existing_log.write({'skill_level_id': employee_skill.skill_level_id.id})
                else:
                    skill_to_create_vals.append({
                        'employee_id': employee_skill.employee_id.id,
                        'skill_id': employee_skill.skill_id.id,
                        'skill_level_id': employee_skill.skill_level_id.id,
                        'department_id': employee_skill.employee_id.department_id.id,
                        'skill_type_id': employee_skill.skill_type_id.id,
                    })

        if skill_to_create_vals:
            self.env['hr.employee.skill.log'].create(skill_to_create_vals)

    @api.model_create_multi
    def create(self, vals_list):
        employee_skills = super().create(vals_list)
        employee_skills._create_logs()
        return employee_skills

    def write(self, vals):
        res = super().write(vals)
        self._create_logs()
        return res

```

## File: models\hr_employee_skill_log.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrEmployeeSkillLog(models.Model):
    _name = 'hr.employee.skill.log'
    _description = "Skills History"
    _rec_name = 'skill_id'
    _order = "employee_id,date"

    employee_id = fields.Many2one('hr.employee', required=True, ondelete='cascade')
    department_id = fields.Many2one('hr.department')
    skill_id = fields.Many2one('hr.skill', compute='_compute_skill_id', store=True, domain="[('skill_type_id', '=', skill_type_id)]", readonly=False, required=True, ondelete='cascade')
    skill_level_id = fields.Many2one('hr.skill.level', compute='_compute_skill_level_id', domain="[('skill_type_id', '=', skill_type_id)]", store=True, readonly=False, required=True, ondelete='cascade')
    skill_type_id = fields.Many2one('hr.skill.type', required=True, ondelete='cascade')
    level_progress = fields.Integer(related='skill_level_id.level_progress', store=True, aggregator="avg")
    date = fields.Date(default=fields.Date.context_today)

    _sql_constraints = [
        ('_unique_skill_log', 'unique (employee_id, department_id, skill_id, date)', "Two levels for the same skill on the same day is not allowed"),
    ]

```

## File: models\hr_resume_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResumeLine(models.Model):
    _name = 'hr.resume.line'
    _description = "Resume line of an employee"
    _order = "line_type_id, date_end desc, date_start desc"

    employee_id = fields.Many2one('hr.employee', required=True, ondelete='cascade', index=True)
    name = fields.Char(required=True, translate=True)
    date_start = fields.Date(required=True)
    date_end = fields.Date()
    description = fields.Html(string="Description", translate=True)
    line_type_id = fields.Many2one('hr.resume.line.type', string="Type")

    # Used to apply specific template on a line
    display_type = fields.Selection([('classic', 'Classic')], string="Display Type", default='classic')

    _sql_constraints = [
        ('date_check', "CHECK ((date_start <= date_end OR date_end IS NULL))", "The start date must be anterior to the end date."),
    ]

```

## File: models\hr_resume_line_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResumeLineType(models.Model):
    _name = 'hr.resume.line.type'
    _description = "Type of a resume line"
    _order = "sequence"

    name = fields.Char(required=True, translate=True)
    sequence = fields.Integer('Sequence', default=10)

```

## File: models\hr_skill.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Skill(models.Model):
    _name = 'hr.skill'
    _description = "Skill"
    _order = "sequence, name"

    name = fields.Char(required=True, translate=True)
    sequence = fields.Integer(default=10)
    skill_type_id = fields.Many2one('hr.skill.type', required=True, ondelete='cascade')
    color = fields.Integer(related='skill_type_id.color')

    @api.depends('skill_type_id')
    @api.depends_context('from_skill_dropdown')
    def _compute_display_name(self):
        if not self._context.get('from_skill_dropdown'):
            return super()._compute_display_name()
        for record in self:
            record.display_name = f"{record.name} ({record.skill_type_id.name})"

```

## File: models\hr_skill_level.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SkillLevel(models.Model):
    _name = 'hr.skill.level'
    _description = "Skill Level"
    _order = "level_progress desc"

    skill_type_id = fields.Many2one('hr.skill.type', ondelete='cascade')
    name = fields.Char(required=True)
    level_progress = fields.Integer(string="Progress", help="Progress from zero knowledge (0%) to fully mastered (100%).")
    default_level = fields.Boolean(help="If checked, this level will be the default one selected when choosing this skill.")

    _sql_constraints = [
        ('check_level_progress', 'CHECK(level_progress BETWEEN 0 AND 100)', "Progress should be a number between 0 and 100."),
    ]

    @api.depends('level_progress')
    @api.depends_context('from_skill_level_dropdown')
    def _compute_display_name(self):
        if not self._context.get('from_skill_level_dropdown'):
            return super()._compute_display_name()
        for record in self:
            record.display_name = f"{record.name} ({record.level_progress}%)"

    @api.model_create_multi
    def create(self, vals_list):
        skill_levels = super().create(vals_list)
        for level in skill_levels:
            if level.default_level:
                level.skill_type_id.skill_level_ids.filtered(lambda r: r.id != level.id).default_level = False
        return skill_levels

    def write(self, vals):
        res = super().write(vals)
        if vals.get('default_level'):
            self.skill_type_id.skill_level_ids.filtered(lambda r: r.id != self.id).default_level = False
        return res

```

## File: models\hr_skill_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint
from odoo import fields, models


class SkillType(models.Model):
    _name = 'hr.skill.type'
    _description = "Skill Type"
    _order = "name"

    def _get_default_color(self):
        return randint(1, 11)

    active = fields.Boolean('Active', default=True)
    name = fields.Char(required=True, translate=True)
    skill_ids = fields.One2many('hr.skill', 'skill_type_id', string="Skills")
    skill_level_ids = fields.One2many('hr.skill.level', 'skill_type_id', string="Levels", copy=True)
    color = fields.Integer('Color', default=_get_default_color)

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", skill_type.name), color=0) for skill_type, vals in zip(self, vals_list)]

```

## File: models\resource_resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class Resource(models.Model):
    _inherit = ['resource.resource']

    employee_skill_ids = fields.One2many(related='employee_id.employee_skill_ids')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class User(models.Model):
    _inherit = ['res.users']

    resume_line_ids = fields.One2many(related='employee_id.resume_line_ids', readonly=False)
    employee_skill_ids = fields.One2many(related='employee_id.employee_skill_ids', readonly=False)

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + [
            'resume_line_ids',
            'employee_skill_ids',
        ]

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + [
            'resume_line_ids',
            'employee_skill_ids',
        ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import hr_employee_public
from . import hr_resume_line
from . import hr_resume_line_type
from . import hr_skill
from . import hr_employee_skill
from . import hr_employee_skill_log
from . import hr_skill_level
from . import hr_skill_type
from . import res_users
from . import resource_resource

```

## File: report\hr_employee_cv_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, models

class EmployeeResumeReport(models.AbstractModel):
    _name = 'report.hr_skills.report_employee_cv'
    _description = 'Employee Resume'

    def _get_report_values(self, docids, data=None):
        show_others = (data or {}).get('show_others')
        employees = self.env['hr.employee'].browse(docids)

        resume_lines = {}
        for employee in employees:
            resume_lines[employee] = defaultdict(self.env['hr.resume.line'].browse)
            for line in employee.resume_line_ids:
                if not show_others and not line.line_type_id:
                    continue
                resume_lines[employee][line.line_type_id.name or _('Other')] |= line

        return {
            'doc_ids': docids,
            'doc_model': 'hr.employee',
            'docs': employees,
            'resume_lines': resume_lines,
        }

```

## File: report\hr_employee_cv_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_report_employee_cv" model="ir.actions.report">
        <field name="name">Employee Resume</field>
        <field name="model">hr.employee</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_skills.report_employee_cv</field>
        <field name="report_file">hr_skills.report_employee_cv</field>
        <field name="paperformat_id" ref="hr_skills.paperformat_resume"/>
        <field name="print_report_name">'CV - %s' % (object.name)</field>
        <field name="binding_model_id" eval="False"/> <!-- Don't display it under Print menu -->
        <field name="binding_type">report</field>
    </record>
</odoo>

```

## File: report\hr_employee_skill_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools

class HrEmployeeSkillReport(models.BaseModel):
    _auto = False
    _name = 'hr.employee.skill.report'
    _inherit = "hr.manager.department.report"
    _description = 'Employee Skills Report'
    _order = 'employee_id, level_progress desc'

    id = fields.Id()
    display_name = fields.Char(related='employee_id.name')
    company_id = fields.Many2one('res.company', readonly=True)
    department_id = fields.Many2one('hr.department', readonly=True)

    skill_id = fields.Many2one('hr.skill', readonly=True)
    skill_type_id = fields.Many2one('hr.skill.type', readonly=True)
    skill_level = fields.Char(readonly=True)
    level_progress = fields.Float(readonly=True, aggregator='avg')
    active = fields.Boolean(related='employee_id.active')

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)

        self.env.cr.execute("""
        CREATE OR REPLACE VIEW %s AS (
            SELECT
                row_number() OVER () AS id,
                e.id AS employee_id,
                e.company_id AS company_id,
                e.department_id AS department_id,
                s.skill_id AS skill_id,
                s.skill_type_id AS skill_type_id,
                sl.level_progress / 100.0 AS level_progress,
                sl.name AS skill_level
            FROM hr_employee e
            LEFT OUTER JOIN hr_employee_skill s ON e.id = s.employee_id
            LEFT OUTER JOIN hr_skill_level sl ON sl.id = s.skill_level_id
            LEFT OUTER JOIN hr_skill_type st ON st.id = sl.skill_type_id
            WHERE st.active IS True
        )
        """ % (self._table, ))

```

## File: report\hr_employee_skill_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_skill_report_view_pivot" model="ir.ui.view">
        <field name="model">hr.employee.skill.report</field>
        <field name="arch" type="xml">
            <pivot disable_linking="True">
                <field name="employee_id" type="row"/>
                <field name="skill_type_id" type="col"/>
                <field name="skill_id" type="col"/>
                <field name="level_progress" type="measure" widget="percentage"/>
            </pivot>
        </field>
    </record>

    <record id="hr_employee_skill_report_view_graph" model="ir.ui.view">
        <field name="model">hr.employee.skill.report</field>
        <field name="arch" type="xml">
            <graph>
                <field name="employee_id" type="row"/>
                <field name="skill_type_id" type="col"/>
                <field name="skill_id" type="col"/>
                <field name="level_progress" type="measure" widget="percentage"/>
            </graph>
        </field>
    </record>

    <record id="hr_employee_skill_report_view_list" model="ir.ui.view">
        <field name="model">hr.employee.skill.report</field>
        <field name="arch" type="xml">
            <list expand="0" edit="1" editable="bottom">
                <field name="employee_id" widget="many2one_avatar_user" options="{'no_create': True, 'no_open': True}"/>
                <field name="skill_type_id" options="{'no_create': True, 'no_open': True}"/>
                <field name="skill_id" options="{'no_create': True, 'no_open': True}"/>
                <field name="skill_level" options="{'no_create': True, 'no_open': True}"/>
                <field name="level_progress" widget="percentage" options="{'no_create': True, 'no_open': True}"/>
            </list>
        </field>
    </record>

    <record id="hr_employee_skill_report_view_search" model="ir.ui.view">
        <field name="model">hr.employee.skill.report</field>
        <field name="arch" type="xml">
            <search>
                <field name="employee_id"/>
                <field name="department_id"/>
                <field name="skill_id"/>
                <field name="skill_type_id"/>
                <separator/>
                <filter string="Employees with Skills" name="employees_with_skills" domain="[('skill_id', '!=', False)]"/>
                <filter string="Employees without Skills" name="employees_without_skills" domain="[('skill_id', '=', False)]"/>
                <separator/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
                <separator/>
                <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                <filter string="Department" name="department" context="{'group_by': 'department_id'}"/>
                <separator/>
                <filter string="Skill Type" name="skill_type" context="{'group_by': 'skill_type_id'}"/>
                <filter string="Skill" name="skill" context="{'group_by': 'skill_id'}"/>
            </search>
        </field>
    </record>

    <record id="hr_employee_skill_report_action" model="ir.actions.act_window">
        <field name="name">Employee Skills</field>
        <field name="res_model">hr.employee.skill.report</field>
        <field name="search_view_id" ref="hr_employee_skill_report_view_search"/>
        <field name="view_mode">list,pivot</field>
        <field name="context">{
            'search_default_employee': 1,
            'search_default_employees_with_skills': 1,
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
            </p><p>
                This report will give you an overview of the skills per Employee.
                Create them in configuration and add them on the Employee.
            </p>
        </field>
    </record>

    <record id="action_hr_employee_skill_log_department" model="ir.actions.act_window">
        <field name="name">Skill History Report</field>
        <field name="res_model">hr.employee.skill.report</field>
        <field name="view_mode">graph,pivot,list</field>
        <field name="context">{'fill_temporal': 0, 'search_default_group_by_skill_type_id': 1, 'search_default_group_by_skill_id': 2}</field>
        <field name="target">current</field>
    </record>

    <menuitem
        id="hr_employee_skill_report_menu"
        name="Skills"
        action="hr_employee_skill_report_action"
        parent="hr.hr_menu_hr_reports"
        sequence="15"/>
</odoo>

```

## File: report\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee_cv_report
from . import hr_employee_skill_report

```

## File: security\hr_skills_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="hr_resume_rule_employee" model="ir.rule">
        <field name="name">Resume: employee: read all</field>
        <field name="model_id" ref="model_hr_resume_line"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_resume_rule_employee_hr_user" model="ir.rule">
        <field name="name">Resume: HR user: all</field>
        <field name="model_id" ref="model_hr_resume_line"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('hr.group_hr_user'))]"/>
    </record>

    <record id="hr_skills_rule_employee_update" model="ir.rule">
        <field name="name">Resume: employee: create/write/unlink own</field>
        <field name="model_id" ref="model_hr_resume_line"/>
        <field name="domain_force">[('employee_id.user_id','=',user.id)]</field>
        <field name="perm_read" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_skill_rule_employee" model="ir.rule">
        <field name="name">Employee skill: employee: read all</field>
        <field name="model_id" ref="model_hr_employee_skill"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_skill_rule_hr_user" model="ir.rule">
        <field name="name">Employee skill: HR user: read all</field>
        <field name="model_id" ref="model_hr_employee_skill"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('hr.group_hr_user'))]"/>
    </record>

    <record id="hr_skill_rule_employee_update" model="ir.rule">
        <field name="name">Employee skill: employee: create/write/unlink own</field>
        <field name="model_id" ref="model_hr_employee_skill"/>
        <field name="domain_force">[('employee_id.user_id','=',user.id)]</field>
        <field name="perm_read" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_employee_skill_report_hr_user" model="ir.rule">
        <field name="name">Employee Skill Report: HR user</field>
        <field name="model_id" ref="model_hr_employee_skill_report"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
    </record>

    <record id="hr_employee_skill_report_manager" model="ir.rule">
        <field name="name">Employee Skill Report: employee's manager</field>
        <field name="model_id" ref="model_hr_employee_skill_report"/>
        <field name="domain_force">[('has_department_manager_access', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="hr_employee_skill_report_multicompany" model="ir.rule">
        <field name="name">Employee Skill Report: Multi-Company Rule</field>
        <field name="model_id" ref="model_hr_employee_skill_report"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_resume_line,hr.resume.line,model_hr_resume_line,hr.group_hr_user,1,1,1,1
access_hr_resume_line_employee,hr.resume.line.employee,model_hr_resume_line,base.group_user,1,1,1,1
access_hr_resume_line_type,hr.resume.line.type,model_hr_resume_line_type,hr.group_hr_user,1,1,1,1
access_hr_resume_line_type_employee,hr.resume.line.type.employee,model_hr_resume_line_type,base.group_user,1,0,0,0
access_hr_skill_type,hr.skill.type,model_hr_skill_type,hr.group_hr_user,1,1,1,1
access_hr_skill_type_employee,hr.skill.type.employee,model_hr_skill_type,base.group_user,1,0,0,0
access_hr_skill_level,hr.skill.level,model_hr_skill_level,hr.group_hr_user,1,1,1,1
access_hr_skill_level_employee,hr.skill.level.employee,model_hr_skill_level,base.group_user,1,0,0,0
access_hr_skill,hr.skill,model_hr_skill,hr.group_hr_user,1,1,1,1
access_hr_skill_employee,hr.skill.employee,model_hr_skill,base.group_user,1,0,1,0
access_hr_employee_skill,hr.employee.skill,model_hr_employee_skill,hr.group_hr_user,1,1,1,1
access_hr_employee_skill_employee,hr.employee.skill,model_hr_employee_skill,base.group_user,1,1,1,1
access_hr_employee_skill_report_user,hr.employee.skill.report,model_hr_employee_skill_report,hr.group_hr_user,1,0,0,0
access_hr_employee_skill_report_manager,hr.employee.skill.report,model_hr_employee_skill_report,base.group_user,1,0,0,0
access_hr_employee_skill_log,hr.employee.skill.log,model_hr_employee_skill_log,hr.group_hr_user,1,1,1,0
access_hr_employee_skill_log_manager,hr.employee.skill.log,model_hr_employee_skill_log,base.group_user,1,0,0,0
access_hr_employee_cv_wizard,access_hr_employee_cv_wizard,hr_skills.model_hr_employee_cv_wizard,base.group_user,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M30.576 14.907a4.029 4.029 0 0 1 5.697 0l8.547 8.546a4.029 4.029 0 0 1 0 5.698l-15.669 15.67a4.029 4.029 0 0 1-5.698 0l-8.546-8.547a4.029 4.029 0 0 1 0-5.698l15.669-15.67Z" fill="#FBB945"/><path d="M18.721 9.047a4.029 4.029 0 0 1 5.264-2.18l11.167 4.625a4.029 4.029 0 0 1 2.18 5.264l-8.48 20.472a4.029 4.029 0 0 1-5.263 2.181l-11.167-4.626a4.029 4.029 0 0 1-2.18-5.264l8.48-20.472Z" fill="#985184"/><path d="M37.527 16.16c-.048.2-.113.4-.194.596l-8.48 20.472a4.029 4.029 0 0 1-5.265 2.18l-9.236-3.825a4.03 4.03 0 0 1 .555-5.007l15.669-15.67a4.029 4.029 0 0 1 5.697 0l1.254 1.254Z" fill="#712258"/><path d="M4 8.029A4.029 4.029 0 0 1 8.029 4h12.087a4.029 4.029 0 0 1 4.029 4.029v22.16a4.029 4.029 0 0 1-4.03 4.028H8.03A4.029 4.029 0 0 1 4 30.188V8.03Z" fill="#1AD3BB"/><path d="M23.973 6.861c.112.37.172.762.172 1.168v22.16a4.029 4.029 0 0 1-4.029 4.029h-8.658a4.03 4.03 0 0 1-1.217-4.699l8.48-20.472a4.029 4.029 0 0 1 5.252-2.186Z" fill="#005E7A"/></svg>

```

## File: static\src\components\avatar_card_resource\avatar_card_resource_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="resource_mail.AvatarCardResourcePopover" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_card_user_infos')]" position="inside">
            <div t-if="skills?.length" class="d-flex align-items-start pt-1">
                <i class="fa fa-fw fa-graduation-cap me-1" data-tooltip="Skills"/>
                <div class="d-flex flex-wrap gap-1 o_employee_skills_tags overflow-hidden">
                    <TagsList tags="skillTags" visibleItemsLimit="5"/>
                </div>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('o_avatar_card_buttons')]" position="attributes">
            <attribute t-if="skills?.length" name="class" add="pt-1" separator=" "/>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\avatar_card_resource\avatar_card_resource_popover_patch.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { AvatarCardResourcePopover } from "@resource_mail/components/avatar_card_resource/avatar_card_resource_popover";


export const patchAvatarCardResourcePopover = {
    loadAdditionalData() {
        const promises = super.loadAdditionalData();
        this.skills = false;
        if (this.record.employee_skill_ids?.length) {
            promises.push(
                this.orm
                    .read("hr.employee.skill", this.record.employee_skill_ids, ["display_name", "color"])
                    .then((skills) => {
                        this.skills = skills;
                    })
            );
        }
        return promises;
    },
    get fieldNames() {
        return [
            ...super.fieldNames,
            "employee_skill_ids",
        ];
    },
    get skillTags() {
        return this.skills.map(({ id, display_name, color }) => ({
            id,
            text: display_name,
            colorIndex: color,
        }));
    },
};

export const unpatchAvatarCardResourcePopover = patch(AvatarCardResourcePopover.prototype, patchAvatarCardResourcePopover);

```

## File: static\src\fields\auto_save_skill_type\auto_save_skill_type.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";


export class AutoSaveSkillTypeField extends X2ManyField {
     async onAdd({ context, editable } = {}) {
        await this.props.record.model.root.save();
        await super.onAdd({ context, editable });
     }
}

export const autoSaveSkillTypeField = {
    ...x2ManyField,
    component: AutoSaveSkillTypeField,
};

registry.category("fields").add("auto_save_skill_type", autoSaveSkillTypeField);

```

## File: static\src\fields\boolean_toggle_load\boolean_toggle_load.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';
import { ListBooleanToggleField, listBooleanToggleField } from "@web/views/fields/boolean_toggle/list_boolean_toggle_field";

export class ListBooleanToggleLoadField extends ListBooleanToggleField {
    async onChange(value) {
        await super.onChange(value);
        await this.props.record.model.root.save();
        return this.env.model.load();
    }
}

export const listBooleanToggleLoadField = {
    ...listBooleanToggleField,
    component: ListBooleanToggleLoadField,
};

registry.category("fields").add("boolean_toggle_load", listBooleanToggleLoadField);

```

## File: static\src\fields\form_view_one2many\form_view_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_skills.X2ManyFieldDialog" t-inherit="web.X2ManyFieldDialog" t-inherit-mode="primary">
        <xpath expr="//button[hasclass('o_form_button_save')]" position="replace">
            <button class="btn btn-primary o_form_button_save" t-on-click="() => this.save()"  data-hotkey="c">Select &amp; Close</button>
        </xpath>
        <xpath expr="//button[hasclass('o_form_button_save_new')]" position="replace">
            <button class="btn btn-primary o_form_button_save_new" t-on-click="saveAndNew" data-hotkey="n">Select &amp; New</button>
        </xpath>
    </t>
</templates>

```

## File: static\src\fields\hr_skills_tags_list\hr_skills_tags_list.js

```javascript
/** @odoo-module **/

import { TagsList } from "@web/core/tags_list/tags_list";


export class SkillsTagList extends TagsList {
    static template = "hr_skills.SkillsTagsList";

    getTextStyle(tag) {
        return tag.defaultLevel
    }
}

```

## File: static\src\fields\hr_skills_tags_list\hr_skills_tags_list.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_skills.SkillsTagsList" t-inherit="web.TagsList">
        <xpath expr='//span[contains(@class, "o_tag")]' position="attributes">
            <attribute name="t-attf-class" add="{{ getTextStyle(tag) ? 'border border-2' : '' }}" separator=" "/>
            <attribute name="t-attf-style" add="{{ getTextStyle(tag) ? 'border-color: rgb(140, 140, 140) !important ;' : ''}}" separator=" "/>
        </xpath>
        <xpath expr='//div[contains(@class, "o_tag_badge_text")]' position="attributes">
            <attribute name="t-attf-class" add="{{ getTextStyle(tag) ? 'fw-bold' : '' }}" separator=" "/>
        </xpath>
    </t>

</templates>

```

## File: static\src\fields\many2many_tags_skills\many2many_tags_skills.js

```javascript
import { registry } from "@web/core/registry";
import {
    Many2ManyTagsField,
    many2ManyTagsField,
} from "@web/views/fields/many2many_tags/many2many_tags_field";

import { SkillsTagList } from "../hr_skills_tags_list/hr_skills_tags_list";


class SkillsMany2ManyTags extends Many2ManyTagsField {
    static components = { ...Many2ManyTagsField.components, TagsList: SkillsTagList };
    getTagProps(record) {
        return { ...super.getTagProps(record), defaultLevel: record.data.default_level };
    }
}

export const skillsMany2ManyTags = {
    ...many2ManyTagsField,
    component: SkillsMany2ManyTags,
    relatedFields: (fieldInfo) => {
        return [
            ...many2ManyTagsField.relatedFields(fieldInfo),
            { name: "default_level", type: "boolean"},
        ];
    },
};

registry.category("fields").add("many2many_tags_skills", skillsMany2ManyTags);

```

## File: static\src\fields\resume_one2many\resume_one2many.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { onMounted, onPatched, useRef } from "@odoo/owl";

import { formatDate } from "@web/core/l10n/dates";

import { SkillsX2ManyField, skillsX2ManyField } from "../skills_one2many/skills_one2many";
import { CommonSkillsListRenderer } from "../../views/skills_list_renderer";

export class ResumeListRenderer extends CommonSkillsListRenderer {
    static template = "hr_skills.ResumeListRenderer";
    static rowsTemplate = "hr_skills.ResumeListRenderer.Rows";
    static recordRowTemplate = "hr_skills.ResumeListRenderer.RecordRow";
    static useMagicColumnWidths = false;
    setup() {
        super.setup();

        this.linkRef = useRef("link-target-blank");
        onMounted(this._setLinksToOpenInNewTab);
        onPatched(this._setLinksToOpenInNewTab);
    }
    get groupBy() {
        return "line_type_id";
    }

    get colspan() {
        if (this.props.activeActions) {
            return 3;
        }
        return 2;
    }

    formatDate(date) {
        return formatDate(date);
    }

    _setLinksToOpenInNewTab() {
        const resumeLines = this.linkRef.el;

        // Find all links within the resume description and set target to "_blank"
        if (resumeLines){
            const links = resumeLines.querySelectorAll('a');

            links.forEach(link => {
                link.setAttribute('target', '_blank'); // Set target="_blank" to open links in new tab
            });
        }
    }
}

export class ResumeX2ManyField extends SkillsX2ManyField {
    static components = {
        ...SkillsX2ManyField.components,
        ListRenderer: ResumeListRenderer,
    };
    getWizardTitleName() {
        return _t("New Resume line");
    }
}

export const resumeX2ManyField = {
    ...skillsX2ManyField,
    component: ResumeX2ManyField,
};

registry.category("fields").add("resume_one2many", resumeX2ManyField);

```

## File: static\src\fields\resume_one2many\resume_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_skills.ResumeListRenderer" t-inherit-mode="primary" t-inherit="hr_skills.SkillsListRenderer">
        <xpath expr="//table" position="attributes">
            <attribute name="t-attf-class" add="table-borderless {{ !showTable ? 'd-none' : ''}}" remove="table-striped" separator=" "/>
        </xpath>
        <xpath expr="//table" position="after">
            <t t-if="!showTable">
                <div name="no_resume_line" class="ms-3 mt-3">
                    <p>
                        There are no resume lines on this employee.
                        Why not add a new one?
                    </p>
                </div>
                <button t-on-click="props.onAdd" class="btn btn-secondary ms-4 mt-3" role="button" t-if="isEditable">
                    Create a new entry
                </button>
            </t>
        </xpath>
        <xpath expr="//div[@name='skills_header']" position="replace"/>
        <xpath expr="//thead/tr" position="replace">
            <tr>
                <th style="width: 32px; min-width: 32px;"></th>
                <th class="w-100"></th>
                <th t-if="isEditable" class="o_list_actions_header" style="width: 32px; min-width: 32px"></th>
            </tr>
        </xpath>
        <xpath expr="//div[@name='no_skills']" position="replace"/>
    </t>

    <t t-name="hr_skills.ResumeListRenderer.Rows" t-inherit-mode="primary" t-inherit="hr_skills.SkillsListRenderer.Rows">
        <xpath expr="//tr" position="attributes">
            <attribute name="class" add="o_resume_group_header" separator=" "/>
        </xpath>
        <xpath expr="//th[hasclass('o_group_name')]" position="after">
            <th></th>
        </xpath>
    </t>

    <t t-name="hr_skills.ResumeListRenderer.RecordRow" t-inherit-mode="primary" t-inherit="web.ListRenderer.RecordRow">
        <xpath expr="//t[@t-foreach='getColumns(record)']" position="replace">
            <t t-set="data" t-value="record.data"/>
            <t t-if="data.display_type === 'classic'" id='row'>
                <td class="o_resume_timeline_cell position-relative pe-lg-2">
                    <div class="rounded-circle bg-info position-relative"/>
                </td>
                <td class="o_data_cell pt-0" t-on-click="(ev) => this.onCellClicked(record, null, ev)">
                    <div t-attf-class="o_resume_line {{data.display_type == 'certification' ? 'o_resume_line_display_certification' : ''}}" t-att-data-id="id">
                        <small class="o_resume_line_dates fw-bold">
                            <t t-out="formatDate(data.date_start)"/> -
                            <t t-if="data.date_end" t-out="formatDate(data.date_end)"/>
                            <t t-else="">Current</t>
                        </small>
                        <h4 class="o_resume_line_title mt-2" t-esc="data.name"/>
                        <p t-if="data.description" class="o_resume_line_desc" t-out="data.description" t-ref="link-target-blank"/>
                    </div>
                </td>
            </t>
        </xpath>
    </t>
</odoo>

```

## File: static\src\fields\skills_one2many\skills_one2many.js

```javascript
/** @odoo-module */

import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";
import {
    useX2ManyCrud,
    useOpenX2ManyRecord,
} from "@web/views/fields/relational_utils";
import { registry } from "@web/core/registry";
import { _t } from "@web/core/l10n/translation";
import { user } from "@web/core/user";
import { CommonSkillsListRenderer } from "../../views/skills_list_renderer";
import { useService } from '@web/core/utils/hooks';
import { onWillStart } from "@odoo/owl";


export class SkillsListRenderer extends CommonSkillsListRenderer {
    static template = "hr_skills.SkillsListRenderer";
    setup() {
        super.setup();
        this.orm = useService('orm');
        this.actionService = useService("action");

        onWillStart(async () => {
            const res = await this.orm.searchCount('hr.skill.type', []);
            this.anySkills = res > 0;
            [this.user] = await this.orm.read("res.users", [user.userId], ["employee_ids"]);
            this.IsHrUser = await user.hasGroup("hr.group_hr_user");
            this.userSubordinates = (await this.orm.searchRead(
                "hr.employee",
                [["id", "child_of", this.user.employee_ids]],
                ["id"]
            )).map((record) => record["id"]);
        });
    }

    get groupBy() {
        return 'skill_type_id';
    }

    async skillTypesAction() {
        return this.actionService.doAction("hr_skills.hr_skill_type_action");
    }

    async openSkillsReport() {
        // fetch id through employee or public.employee
        const id = this.env.model.root.data.id || this.env.model.root.data.employee_id[0];
-        this.actionService.doAction({
            type: "ir.actions.act_window",
            name: _t("Skills Report"),
            res_model: "hr.employee.skill.log",
            view_mode: "graph,list",
            views: [[false, "graph"], [false, "list"]],
            context: {
                'fill_temporal': 0,
            },
            target: "current",
            domain: [['employee_id', '=', id]],
        });
    }

    get showTimeline() {
        return this.SkillsRight && !this.props.list.context.no_timeline;
    }

    get SkillsRight() {
        let isSubordinate = false;
        if (this.env.model.root.data.employee_id) {
            isSubordinate = this.userSubordinates.includes(this.env.model.root.data.employee_id[0]);
        }
        return this.IsHrUser || isSubordinate;
    }
}

export class SkillsX2ManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: SkillsListRenderer,
    };
    setup() {
        super.setup()
        const { saveRecord, updateRecord } = useX2ManyCrud(
            () => this.list,
            this.isMany2Many
        );

        const openRecord = useOpenX2ManyRecord({
            resModel: this.list.resModel,
            activeField: this.activeField,
            activeActions: this.activeActions,
            getList: () => this.list,
            saveRecord: async (record) => {
                await saveRecord(record);
                await this.props.record.save();
            },
            updateRecord: updateRecord,
            withParentId: this.props.widget !== "many2many",
        });

        this._openRecord = (params) => {
            params.title = this.getWizardTitleName();
            openRecord({...params});
        };
    }

    getWizardTitleName() {
        return _t("Select Skills")
    }

    async onAdd({ context, editable } = {}) {
        const employeeId = this.props.record.resModel === "res.users" ? this.props.record.data.employee_id[0] : this.props.record.resId;
        return super.onAdd({
            editable,
            context: {
                ...context,
                default_employee_id: employeeId,
            }
        });
    }
}

export const skillsX2ManyField = {
    ...x2ManyField,
    component: SkillsX2ManyField,
};

registry.category("fields").add("skills_one2many", skillsX2ManyField);

```

## File: static\src\fields\skills_one2many\skills_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_skills.SkillsListRenderer" t-inherit-mode="primary" t-inherit="web.ListRenderer">
        <xpath expr="//table" position="attributes">
            <attribute name="t-attf-class" add="mb-1 {{ !isEditable ? 'cursor-default' : '' }} {{ !showTable ? 'd-none' : ''}} o_skill_table" separator=" "/>
        </xpath>
        <xpath expr="//thead" position="attributes">
            <attribute name="style">visibility: collapse;</attribute>
        </xpath>
        <xpath expr="//table" position="before">
            <div name="skills_header" class="text-uppercase fw-bolder small ms-3" style="margin-top: 2rem; box-shadow: 0 1px 0 #e6e6e6">
                Skills
                <a t-if="!isEmpty &amp;&amp; showTimeline" t-on-click="openSkillsReport" class="float-end me-3 cursor-pointer">
                    <span class="fa fa-line-chart me-1"/>
                    Timeline
                </a>
            </div>
        </xpath>
        <xpath expr="//table" position="after">
            <t t-if="!showTable">
                <t t-if="!anySkills">
                    <div name="no_skills" class="ms-3 mt-3">
                        <p>
                            There are no skills defined in the library.<br/>
                            Why not try adding some ?
                        </p>
                        <button t-on-click="skillTypesAction"
                            class="btn btn-secondary ms-4 mt-3 text-center"
                            role="button" t-if="isEditable">
                            Create new Skills
                        </button>
                    </div>
                </t>
                <t t-else="">
                    <div name="skills_available" class="ms-3 mt-3">
                        <p class="mt-3">You can add skills from our library to the employee profile.<br/>
                        If skills are missing, they can be created by an HR officer.</p>
                        <div class="ms-5">
                             <button t-on-click="props.onAdd"
                                class="btn btn-secondary ms-4 mt-3 text-center"
                                role="button" t-if="isEditable">
                                Pick a skill from the list
                            </button>
                        </div>
                    </div>
                </t>
            </t>
        </xpath>
    </t>

    <t t-name="hr_skills.SkillsListRenderer.Rows">
        <t t-foreach="Object.entries(groupedList)" t-as="skill_group" t-key="skill_group[0]">
            <tr class="o_group_has_content o_group_header">
                <th tabindex="-1" class="o_group_name" t-att-colspan="colspan">
                    <div class="d-flex justify-content-between align-items-center">
                        <span t-esc="skill_group[1].name"/>
                        <button class="btn btn-secondary btn-sm"
                            t-if="isEditable"
                            t-on-click="() => props.onAdd({ context: { default_skill_type_id: skill_group[1].id }})"
                            role="button">ADD</button>
                    </div>
                </th>
            </tr>
            <t t-foreach="skill_group[1].list.records" t-as="record" t-key="record.id">
                <t t-set="group" t-value="skill_group[1]"/>
                <t t-call="{{ constructor.recordRowTemplate }}"/>
            </t>
        </t>
    </t>
</odoo>

```

## File: static\src\views\skills_graph.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { GraphRenderer } from "@web/views/graph/graph_renderer";
import { graphView } from "@web/views/graph/graph_view";

export class SkillsGraphRenderer extends GraphRenderer {
    getScaleOptions() {
        const scaleOptions = super.getScaleOptions();

        if ('y' in scaleOptions) {
            scaleOptions.y.suggestedMax = 100;
        }

        return scaleOptions;
    }
}

export const skillsGraphView = {
    ...graphView,
    Renderer: SkillsGraphRenderer,
};

registry.category("views").add("skills_graph", skillsGraphView);

```

## File: static\src\views\skills_list_renderer.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ListRenderer } from "@web/views/list/list_renderer";

export class CommonSkillsListRenderer extends ListRenderer {
    get colspan() {
        const span = this.allColumns.length;
        if (this.isEditable) {
            return span + 1;
        }

        return span;
    }

    get groupBy() {
        return '';
    }

    get groupedList() {
        const grouped = {};

        for (const record of this.list.records) {
            const data = record.data;
            const group = data[this.groupBy];

            if (grouped[group[1]] === undefined) {
                grouped[group[1]] = {
                    id: parseInt(group[0]),
                    name: group[1] || _t('Other'),
                    list: {
                        records: [],
                    },
                };
            }

            grouped[group[1]].list.records.push(record);
        }
        return grouped;
    }

    get showTable() {
        return this.props.list.records.length;
    }

    get isEditable() {
        return this.props.editable !== false;
    }

    async onCellClicked(record, column, ev) {
        if (!this.isEditable) {
            return;
        }

        return await super.onCellClicked(record, column, ev);
    }
}
CommonSkillsListRenderer.rowsTemplate = "hr_skills.SkillsListRenderer.Rows";

```

## File: static\src\xml\resume_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="hr_resume_data_row">
    <tr class="o_data_row cursor-default" t-attf-class="o_data_row #{is_last? 'o_data_row_last' : ''}" t-att-data-id="id">
        <t t-if="data.display_type === 'classic'">
            <td class="o_resume_timeline_cell position-relative pe-lg-2">
                <div class="rounded-circle bg-info position-relative"/>
            </td>
            <td class="o_data_cell pt-0 w-100">
                <div class="o_resume_line" t-att-data-id="id">
                    <small class="o_resume_line_dates">
                        <b t-esc="data.date_start"/> - <b t-esc="data.date_end"/>
                    </small>
                    <h4 class="o_resume_line_title mt-2" t-esc="data.name"/>
                    <p t-if="data.description" class="o_resume_line_desc" t-esc="data.description"/>
                </div>
            </td>
        </t>
    </tr>
</t>

<t t-name="hr_trash_button">
    <td class="o_list_record_remove pe-3">
        <button name="delete" arial-label="Delete row" class="btn btn-link text-danger">
            <i class="fa fa-trash"/>
        </button>
    </td>
</t>

<t t-name="hr_resume_group_row">
    <tr class="o_resume_group_header">
        <td class="o_group_name" colspan="100%"><span class="o_horizontal_separator my-0" t-esc="display_name"/></td>
    </tr>
</t>

<t t-name="group_add_item">
    <t t-set="empty" t-value="Object.keys(context).length == 2"/>

    <div t-attf-class="o_field_x2many_list_row_add #{empty? 'd-block w-100' : 'd-inline float-end'}">
        <a href="#"
            role="button"
            t-attf-class="btn btn-secondary o-kanban-button-new #{empty? 'btn-primary mt-3' : 'btn-secondary btn-sm'}"
            t-attf-data-context="{{ context }}">
                <t t-if="empty">CREATE A NEW ENTRY</t>
                <t t-else="">ADD</t>
            </a>
    </div>
</t>


</templates>

```

## File: static\src\xml\skills_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="hr_skill_data_row">
    <tr class="o_data_row cursor-default" t-att-data-id="id">
        <td class="o_data_cell o_skill_cell w-100">
            <t t-esc="data.skill_id.data.display_name"/>
        </td>
        <td class="o_data_cell o_skill_cell pe-3">
            <t t-esc="data.skill_level_id.data.display_name"/>
        </td>
    </tr>
</t>

<t t-name="hr_default_group_row">
    <tr class="o_group_header o_group_has_content">
        <td class="o_group_name border-0 pe-2" colspan="99">
            <b t-esc="display_name"/>
        </td>
    </tr>
</t>


</templates>

```

## File: views\hr_department_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit.hr.skills</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                <div role="menuitem">
                    <a class="dropdown-item" name="%(action_open_skills_log_department)d" type="action">
                        Skills History
                    </a>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_employee_cv_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_employee_cv">
        <t t-set="full_width" t-value="True"/>
        <t t-call="web.basic_layout">
        <t t-foreach="docs" t-as="o">
            <div class="o_employee_cv page">
                <t t-call="hr_skills.report_employee_cv_company"/>
                <t t-call="hr_skills.report_employee_cv_sidepanel"/>
                <t t-call="hr_skills.report_employee_cv_main_panel"/>
                <p class="o_new_page"/>
            </div>
        </t>
        </t>
    </template>

    <template id="report_employee_cv_company">
        <div class="o_company row">
            <div class="col-12 mb-4">
                <img t-if="o.company_id.logo" t-att-src="image_data_uri(o.company_id.logo)" style="max-height: 45px;" alt="Logo"/>
                <div class="oe_structure"></div>
            </div>
            <div class="col-12 mb-4" style="font-weight: bold;">
                <span t-out="o.company_id.partner_id.name">Demo Company Name</span>
            </div>
            <div class="col-12">
                <span t-field="o.company_id.partner_id" t-options='{"widget": "contact", "fields": ["address"], "no_marker": True}'>Demo Address</span>
            </div>
        </div>
    </template>

    <template id="report_employee_cv_sidepanel">
        <div class="oe_structure"></div>
        <div class="o_sidebar" t-attf-style="
            background: #{color_primary};
            background-color: #{color_primary};">
            <div class="o_profile">
                <img class="img img-fluid rounded-circle o_profile_image" t-attf-src="data:image/png;base64,#{o.image_128}" alt="" t-if="o.image_128"/>
                <h1 class="o_profile_name mt-2" t-field="o.name">Marc Demo</h1>
                <h3 class="o_profile_job mb-2" t-field="o.job_id">Software Developer</h3>
            </div>
            <div class="oe_structure"></div>
            <div class="o_sidebar_section" t-if="show_contact">
                <ul class="o_social">
                    <li class="email" t-if="o.company_id.email">
                        <i class="fa fa-solid fa-envelope pe-2"/>
                        <a t-attf-href="mailto: #{o.company_id.email}" t-field="o.company_id.email">demo@email.com</a>
                    </li>
                    <li class="phone" t-if="o.company_id.mobile">
                        <i class="fa fa-solid fa-phone pe-2"/>
                        <a t-attf-href="tel:#{o.company_id.mobile}" t-field="o.company_id.mobile">+1234567890</a>
                    </li>
                    <li class="website" t-if="o.company_id.website">
                        <i class="fa fa-solid fa-globe pe-2"/>
                        <a t-attf-href="tel:#{o.company_id.website}" t-field="o.company_id.website">www.demo.com</a>
                    </li>
                </ul>
            </div>
        </div>
        <div class="oe_structure"></div>
    </template>
    
    <template id="report_employee_cv_main_panel">
        <div class="oe_structure"></div>
        <t t-foreach="resume_lines[o]" t-as="line_type">
            <div class="o_main_panel">
                <div class="o_main_panel_section">
                    <h2 class="o_main_panel_title mt-4" t-attf-style="color: #{color_secondary};">
                        <span class="o_main_panel_icon" t-attf-style="background: #{color_secondary};">
                            <i class="fa fa-solid fa-briefcase"/>
                        </span>
                        <span t-out="line_type">Experience</span>
                    </h2>
                    <t t-foreach="resume_lines[o][line_type]" t-as="resume_line">
                        <div class="o_main_panel_resume_title ps-5">
                            <div class="o_main_panel_resume_title_job">
                                <h3 class="o_main_panel_resume_job" t-field="resume_line.name">Software Developer</h3>
                            </div>
                            <div class="o_main_panel_resume_title_dates">
                                <t t-set="present">Present</t>
                                <div class="o_main_panel_resume_year">
                                    <span t-out="resume_line.date_start.year">2022</span> - <span t-out="resume_line.date_end.year if resume_line.date_end else 'Present'">2023</span>
                                </div>
                            </div>
                        </div>
                        <div class="o_main_panel_resume_description ps-5 w-auto">
                            <p t-field="resume_line.description" data-oe-demo="Odoo India pvt. Ltd"/>
                        </div>
                    </t>
                </div>
            </div>
        </t>

        <div class="oe_structure"></div>
        <t t-set="has_languages" t-value="skill_type_language and any(skill_line.skill_type_id == skill_type_language for skill_line in o.employee_skill_ids)"/>
        <div class="o_main_panel mt-4" t-if="has_languages">
            <div class="o_main_panel_section" t-if="skill_type_language">
                <h2 class="o_main_panel_title" t-attf-style="color: #{color_secondary};">
                    <span class="o_main_panel_icon" t-attf-style="background: #{color_secondary};">
                        <i class="fa fa-solid fa-language"/>
                    </span>
                    Languages
                </h2>
                <t t-foreach="o.employee_skill_ids" t-as="skill_line">
                    <t t-if="skill_line.skill_type_id == skill_type_language">
                        <div class="o_main_panel_resume_title ps-5 pb-2">
                            <div class="o_main_panel_resume_title_job">
                                <h3 class="o_main_panel_resume_job"><span t-out="skill_line.skill_id.name">English</span> - <span t-out="skill_line.skill_level_id.name">Fluent</span></h3>
                            </div>
                        </div>
                    </t>
                </t>
            </div>
        </div>

        <div class="oe_structure"></div>
        <t t-set="has_skills" t-value="show_skills and any(skill_line.skill_type_id != skill_type_language for skill_line in o.employee_skill_ids)"/>
        <div class="o_main_panel mt-4" t-if="has_skills">
            <div class="o_main_panel_section">
                <h2 class="o_main_panel_title" t-attf-style="color: #{color_secondary};">
                    <span class="o_main_panel_icon" t-attf-style="background: #{color_secondary};">
                        <i class="fa fa-solid fa-rocket"/>
                    </span>
                    Skills
                </h2>
                <t t-set="valid_skills" t-value="[skill for skill in o.employee_skill_ids if skill.skill_type_id != skill_type_language]"/>
                <t t-foreach="[valid_skills[n:n+2] for n in range(0, len(valid_skills), 2)]" t-as="skills_pair">
                    <div class="o_main_panel_skills row ps-5">
                        <t t-foreach="skills_pair" t-as="skill_line">
                            <div class="o_main_panel_skill_line col-6 pe-4" t-if="skill_line.skill_type_id != skill_type_language">
                                <h3 class="o_main_panel_skill_name" t-field="skill_line.skill_id">Python</h3>
                                <div class="progress o_main_panel_skill_progress_bar">
                                    <div
                                        class="o_main_panel_skill_progress_bar_color"
                                        role="progressbar"
                                        t-attf-style="width: #{skill_line.level_progress}%; background: #{color_primary};"
                                        t-att-aria-valuenow="skill_line.level_progress"
                                        aria-valuemin="0"
                                        aria-label="Progress bar"
                                        aria-valuemax="100"/>
                                </div>
                            </div>
                        </t>
                    </div>
                </t>
            </div>
        </div>
        <div class="oe_structure"></div>
    </template>
</odoo>

```

## File: views\hr_employee_skill_log_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_skill_log_view_graph_employee" model="ir.ui.view">
          <field name="name">hr.employee.skill.log.view.graph</field>
          <field name="model">hr.employee.skill.log</field>
          <field name="arch" type="xml">
            <graph string="Skills History" type="line" stacked="0" js_class="skills_graph">
                <field name="date" interval="day" type="row"/>
                <field name="skill_id" type="row"/>
                <field name="level_progress" type="measure"/>
            </graph>
          </field>
    </record>

    <record id="hr_employee_skill_log_view_graph_department" model="ir.ui.view">
          <field name="name">hr.employee.skill.log.view.graph</field>
          <field name="model">hr.employee.skill.log</field>
          <field name="arch" type="xml">
            <graph string="Skills History" type="line" stacked="0" js_class="skills_graph">
                <field name="date" interval="day" type="row"/>
                <field name="skill_id" type="row"/>
                <field name="level_progress" type="measure"/>
            </graph>
          </field>
    </record>

    <record id="hr_employee_skill_log_view_tree" model="ir.ui.view">
          <field name="name">hr.employee.skill.log.view.list</field>
          <field name="model">hr.employee.skill.log</field>
          <field name="arch" type="xml">
            <list string="Skills History">
                <field name="employee_id"/>
                <field name="department_id"/>
                <field name="skill_type_id"/>
                <field name="skill_id"/>
                <field name="level_progress"/>
                <field name="date"/>
            </list>
          </field>
    </record>

    <record id="hr_employee_skill_log_view_search" model="ir.ui.view">
          <field name="name">hr.employee.skill.log.view.search</field>
          <field name="model">hr.employee.skill.log</field>
          <field name="arch" type="xml">
            <search string="Search Logs">
                <field name="employee_id"/>
                <field name="skill_id"/>
                <field name="skill_type_id"/>
                <field name="date"/>
                <separator />
                <group expand="0" string="Group By">
                    <filter string="Employee" name="group_by_employee_id" domain="[]" context="{'group_by': 'employee_id'}"/>
                    <filter string="Skill" name="group_by_skill_id" domain="[]" context="{'group_by': 'skill_id'}"/>
                    <filter string="Skill Type" name="group_by_skill_type_id" domain="[]" context="{'group_by': 'skill_type_id'}"/>
                    <filter string="Date" name="group_by_date" domain="[]" context="{'group_by': 'date'}"/>
                </group>
            </search>
          </field>
    </record>

    <record id="action_hr_employee_skill_log_employee" model="ir.actions.act_window">
        <field name="name">Skill History Report</field>
        <field name="res_model">hr.employee.skill.log</field>
        <field name="view_mode">graph,list</field>
        <field name="view_id" ref="hr_employee_skill_log_view_graph_employee"/>
        <field name="context">{'fill_temporal': 0}</field>
        <field name="target">current</field>
    </record>

</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="hr_employee_view_search" model="ir.ui.view">
        <field name="name">hr.employee.skill.search</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='job_id']" position="after">
                <field name="employee_skill_ids"/>
                <field name="resume_line_ids" string="Resume" filter_domain="
                    ['|',
                        ('resume_line_ids.name', 'ilike', self),
                        ('resume_line_ids.description', 'ilike', self),
                    ]"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_public_view_search" model="ir.ui.view">
        <field name="name">hr.employee.public.skill.search</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='job_title']" position="after">
                <field name="employee_skill_ids"/>
                <field name="resume_line_ids" string="Resume" filter_domain="['|', ('resume_line_ids.name', 'ilike', self), ('resume_line_ids.description', 'ilike', self)]"/>
            </xpath>
        </field>
    </record>

    <record id="resume_line_view_form" model="ir.ui.view">
        <field name="name">hr.resume.line.form</field>
        <field name="model">hr.resume.line</field>
        <field name="arch" type="xml">
            <form string="Resume">
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Title"/>
                        <h1>
                            <field name="name" placeholder="e.g. Odoo Inc." required="True"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="employee_id" invisible="1"/>
                            <field name="line_type_id"/>
                            <field name="display_type" required="1"/>
                        </group>
                        <group>
                            <field name="date_start" string="Duration" widget="daterange" options="{'end_date_field': 'date_end'}"/>
                            <field name="date_end" invisible="1"/>
                        </group>
                    </group>
                    <field name="description" placeholder="Description"/>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.resume</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='public']" position="before">
                <page name="skills_resume" string="Resume">
                    <div class="row">
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_resume col-lg-7 d-flex flex-column">
                            <separator string="Resume" class="mb-4"/>
                            <!-- This field uses a custom list view rendered by the 'resume_one2many' widget.
                                Adding fields in the list arch below makes them accessible to the widget
                            -->
                            <field mode="list" nolabel="1" name="resume_line_ids" widget="resume_one2many">
                                <list>
                                    <field name="line_type_id"/>
                                    <field name="name"/>
                                    <field name="description"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="display_type" column_invisible="True"/>
                                </list>
                            </field>

                        </div>
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <field mode="list" nolabel="1" name="employee_skill_ids" widget="skills_one2many" class="mt-2">
                                <list>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                    <field name="skill_type_id" optional="hidden"/>
                                </list>
                            </field>
                        </div>
                    </div>
                </page>
            </xpath>
        </field>
    </record>


    <record id="hr_employee_public_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.employee.public.view.form.inherit.resume</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='public']" position="before">
                <page name="skills_resume" string="Resume">
                    <div class="row">
                        <div class="o_hr_skills_group o_group_resume col-lg-6 d-flex flex-column">
                            <!-- This field uses a custom list view rendered by the 'resume_one2many' widget.
                                Adding fields in the list arch below makes them accessible to the widget
                            -->
                            <field mode="list" nolabel="1" name="resume_line_ids" widget="resume_one2many">
                                <list>
                                    <field name="line_type_id"/>
                                    <field name="name"/>
                                    <field name="description"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="display_type" column_invisible="True"/>
                                </list>
                            </field>
                        </div>
                        <div class="o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <field name="employee_id" invisible="1"/>
                            <field mode="list" nolabel="1" name="employee_skill_ids" invisible="not employee_skill_ids"  widget="skills_one2many">
                                <list>
                                    <field name="skill_type_id" optional="hidden"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </list>
                            </field>
                        </div>
                    </div>
                </page>
            </xpath>
        </field>
    </record>

    <record id="res_users_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.form.inherit.hr.skills</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile" />
        <field name="arch" type="xml">
            <xpath expr="//page[@name='public']" position="before">
                <page name="skills_resume" string="Resume">
                    <div class="row">
                        <field name="employee_id" invisible="1" />
                        <div class="o_hr_skills_group o_group_resume col-lg-6 d-flex margin-left: 0.1em">
                            <!-- This field uses a custom list view rendered by the 'resume_one2many' widget.
                                Adding fields in the list arch below makes them accessible to the widget
                            -->
                            <field mode="list" nolabel="1" name="resume_line_ids" widget="resume_one2many" readonly="not can_edit">
                                <list>
                                    <field name="line_type_id"/>
                                    <field name="name"/>
                                    <field name="description"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="display_type" invisible="1"/>
                                </list>
                            </field>
                        </div>
                        <div class="o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <field mode="list" name="employee_skill_ids"  widget="skills_one2many" readonly="not can_edit">
                                <list>
                                    <field name="skill_type_id" optional="hidden"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </list>
                            </field>
                        </div>
                    </div>
                </page>
            </xpath>
        </field>
    </record>

    <record id="hr_resume_line_type_tree_view" model="ir.ui.view">
        <field name="name">hr.resume.line.type.list.view</field>
        <field name="model">hr.resume.line.type</field>
        <field name="arch" type="xml">
            <list name="Resume Line Types" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </list>
        </field>
    </record>

    <record id="hr_resume_type_action" model="ir.actions.act_window">
        <field name="name">Resume Line Types</field>
        <field name="res_model">hr.resume.line.type</field>
        <field name="view_mode">list,form</field>
    </record>

    <menuitem
            id="menu_human_resources_configuration_resume"
            name="Resume"
            parent="hr.menu_human_resources_configuration"
            sequence="15"
            groups="base.group_no_one"/>

    <menuitem
        id="hr_resume_line_type_menu"
        name="Line Types"
        action="hr_resume_type_action"
        parent="hr_skills.menu_human_resources_configuration_resume"
        sequence="3"
        groups="base.group_no_one"/>

    <!-- Skills -->

    <record id="hr_skill_type_action" model="ir.actions.act_window">
        <field name="name">Skill Types</field>
        <field name="res_model">hr.skill.type</field>
        <field name="view_mode">list,form</field>
    </record>

    <record id="employee_skill_level_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.level.list</field>
        <field name="model">hr.skill.level</field>
        <field name="arch" type="xml">
            <list string="Skill Levels" class="o_skill_level_tree" editable="bottom">
                <field name="name"/>
                <field name="level_progress" widget="progressbar" options="{'editable': true}"/>
                <field name="default_level" widget="boolean_toggle_load"/>
            </list>
        </field>
    </record>

    <record id="employee_skill_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.list</field>
        <field name="model">hr.skill</field>
        <field name="arch" type="xml">
            <list string="Skill Levels">
                <field name="name"/>
                <field name="skill_type_id"/>
            </list>
        </field>
    </record>

    <record id="employee_skill_level_view_form" model="ir.ui.view">
        <field name="name">hr.skill.level.form</field>
        <field name="model">hr.skill.level</field>
        <field name="arch" type="xml">
            <form string="Skill Level">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="level_progress" string="Progress (%)"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="employee_skill_view_form" model="ir.ui.view">
        <field name="name">hr.employees.skill.form</field>
        <field name="model">hr.employee.skill</field>
        <field name="arch" type="xml">
            <form string="Skills" class="o_hr_skills_dialog_form">
                <sheet>
                    <group>
                        <group>
                            <field name="employee_id" invisible="1"/>
                            <field name="skill_type_id" widget="radio" />
                        </group>
                        <group>
                            <field name="skill_id" options="{'no_open': True, 'no_create': True}"
                                    context="{'default_skill_type_id': skill_type_id}"
                                    domain="[('skill_type_id', '=', skill_type_id)]"
                                    invisible="not skill_type_id"/>
                            <label for="skill_level_id"
                                   invisible="not (skill_id or skill_type_id)"/>
                            <div class="o_row" invisible="not (skill_id or skill_type_id)">
                                <span class="ps-0" style="flex:1">
                                    <field name="skill_level_id"
                                           readonly="not skill_id"
                                           options="{'no_open': True, 'no_create': True}"
                                           context="{'from_skill_level_dropdown': True, 'default_skill_type_id': skill_type_id}" />
                                </span>
                                <span style="flex:1">
                                    <field name="level_progress" widget="progressbar" class="o_hr_skills_progress" invisible="not skill_level_id" />
                                </span>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_skill_view_form" model="ir.ui.view">
        <field name="name">hr.skill.form</field>
        <field name="model">hr.skill</field>
        <field name="arch" type="xml">
            <form string="Skills">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="skill_type_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_skill_view_search" model="ir.ui.view">
        <field name="name">hr.skill.view.search</field>
        <field name="model">hr.skill</field>
        <field name="arch" type="xml">
            <search string="Search Skill">
                <field name="name" string="Skill"/>
                <field name="skill_type_id" string="Skill Type"/>
                <separator/>
                <group expand="0" string="Group By...">
                        <filter string="Skill Type" name="group_skill_type_id" domain="[]" context="{'group_by':'skill_type_id'}"/>
                </group>
            </search>
        </field>
    </record>

     <record id="hr_skill_type_view_search" model="ir.ui.view">
        <field name="name">hr.skill.type.search</field>
        <field name="model">hr.skill.type</field>
        <field name="arch" type="xml">
            <search string="Search Skill Type">
                <field name="name" string="Skill Types"/>
                <field name="skill_ids"/>
                <field name="skill_level_ids"/>
                <field name="active" invisible="1"/>
                <filter name="inactive" string="Archived" domain="[('active', '=', False)]"/>
                <separator/>
            </search>
        </field>
    </record>

    <record id="hr_skill_type_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.type.list</field>
        <field name="model">hr.skill.type</field>
        <field name="arch" type="xml">
            <list string="Skill Types">
                <field name="name" string="Skill Types"/>
                <field name="color" widget="color_picker" optional="show"/>
                <field name="skill_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                <field name="skill_level_ids" widget="many2many_tags_skills"/>
            </list>
        </field>
    </record>

    <record id="hr_employee_skill_type_view_form" model="ir.ui.view">
        <field name="name">hr.skill.type.form</field>
        <field name="model">hr.skill.type</field>
        <field name="arch" type="xml">
            <form string="Skill Type">
                <field name="id" invisible="1"/>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <label for="name" string="Skill Type"/>
                        <h1>
                            <field name="name" placeholder="e.g. Languages" required="True"/>
                            <field name="active" invisible="1"/>
                        </h1>
                    </div>
                    <group string="Skills">
                    </group>
                    <field name="skill_ids" nolabel="1" context="{'default_skill_type_id': id}">
                        <list editable="bottom">
                            <field name="sequence" widget="handle" />
                            <field name="name"/>
                        </list>
                    </field>
                    <group string="Levels">
                    </group>
                    <field name="skill_level_ids" nolabel="1" widget="auto_save_skill_type" context="{'default_skill_type_id': id}"/>
                    <group string="Display">
                        <field name="color" widget="color_picker"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <menuitem
        id="hr_skill_type_menu"
        name="Skill Types"
        action="hr_skill_type_action"
        parent="hr.menu_config_employee"
        sequence="7"
        groups="hr.group_hr_user"/>
</odoo>

```

## File: wizard\hr_employee_cv_wizard.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode

from odoo import _, api, fields, models


class HrEmployeeCVWizard(models.TransientModel):
    _name = 'hr.employee.cv.wizard'
    _description = 'Print Resume'

    employee_ids = fields.Many2many('hr.employee')

    color_primary = fields.Char('Primary Color', default=lambda self: self.env.company.primary_color or "#666666", required=True)
    color_secondary = fields.Char('Secondary Color', default=lambda self: self.env.company.secondary_color or "#666666", required=True)

    show_skills = fields.Boolean(string='Skills', default=True)
    show_contact = fields.Boolean(string='Contact Information', default=True)
    show_others = fields.Boolean(string='Others', default=True)

    can_show_others = fields.Boolean(compute='_compute_can_show_others')
    can_show_skills = fields.Boolean(compute='_compute_can_show_others')

    @api.depends('employee_ids')
    def _compute_can_show_others(self):
        for wizard in self:
            wizard.can_show_others = wizard.employee_ids.resume_line_ids.filtered(lambda l: not l.line_type_id)
            wizard.can_show_skills = wizard.employee_ids.skill_ids

    def action_validate(self):
        self.ensure_one()
        return {
            'name': _('Print Resume'),
            'type': 'ir.actions.act_url',
            'url': '/print/cv?' + url_encode({
                'employee_ids': ','.join(str(x) for x in self.employee_ids.ids),
                'color_primary': self.color_primary,
                'color_secondary': self.color_secondary,
                'show_skills': 1 if self.show_skills else None,
                'show_contact': 1 if self.show_contact else None,
                'show_others': 1 if self.show_others else None,
            })
        }

```

## File: wizard\hr_employee_cv_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_cv_wizard_view_form" model="ir.ui.view">
        <field name="name">hr.employee.cv.wizard.view.form</field>
        <field name="model">hr.employee.cv.wizard</field>
        <field name="arch" type="xml">
            <form string="Print Resume">
                <field name="can_show_others" invisible="1"/>
                <field name="can_show_skills" invisible="1"/>
                <group>
                    <group>
                        <field name="employee_ids" widget="many2many_tags" invisible="1"/>
                        <label for="color_primary" string="Colors"/>
                        <div class="d-flex flex-row">
                            <field name="color_primary" widget="color" nolabel="1" style="width: 35px;"/>
                            <field name="color_secondary" widget="color" nolabel="1"/>
                        </div>
                        <field name="show_contact"/>
                        <field name="show_others" invisible="not can_show_others"/>
                        <field name="show_skills" invisible="not can_show_skills"/>
                    </group>
                </group>
                <footer>
                    <button name="action_validate" string="Print" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="x" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_hr_employee_cv_wizard" model="ir.actions.act_window">
        <field name="name">Print Resume</field>
        <field name="res_model">hr.employee.cv.wizard</field>
        <field name="view_mode">form</field>
        <field name="context">{'default_employee_ids': active_ids}</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee_cv_wizard

```

