# Odoo Module: hr_skills

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

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
        'data/ir_actions_server_data.xml',
        'report/hr_employee_skill_report_views.xml',
        'views/hr_department_views.xml',
    ],
    'demo': [
        'data/hr_resume_demo.xml',
        'data/hr.employee.skill.csv',
        'data/hr.resume.line.csv',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_skills/static/src/fields/skills_one2many.xml',
            'hr_skills/static/src/fields/*',
            'hr_skills/static/src/scss/*.scss',
            'hr_skills/static/src/views/*.js',
            'hr_skills/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'hr_skills/static/tests/tours/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr.employee.skill.csv

```csv
id,employee_id:id,skill_id:id,skill_type_id:id,skill_level_id:id
employee_skill_admin_spark,hr.employee_admin,hr_skill_spark,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_admin_flute,hr.employee_admin,hr_skill_flute,hr_skill_type_music,hr_skill_level_l2
employee_skill_admin_singing,hr.employee_admin,hr_skill_singing,hr_skill_type_music,hr_skill_level_l1
employee_skill_admin_violin,hr.employee_admin,hr_skill_violin,hr_skill_type_music,hr_skill_level_l2
employee_skill_admin_piano,hr.employee_admin,hr_skill_piano,hr_skill_type_music,hr_skill_level_l2
employee_skill_al_analytics,hr.employee_al,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_al_digital_ad,hr.employee_al,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_al_public,hr.employee_al,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_al_com,hr.employee_al,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_al_french,hr.employee_al,hr_skill_french,hr_skill_type_lang,hr_skill_level_c1
employee_skill_al_nosql,hr.employee_al,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_beginner
employee_skill_al_django,hr.employee_al,hr_skill_django,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_al_python,hr.employee_al,hr_skill_python,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_mit_piano,hr.employee_mit,hr_skill_piano,hr_skill_type_music,hr_skill_level_l3
employee_skill_mit_singing,hr.employee_mit,hr_skill_singing,hr_skill_type_music,hr_skill_level_l2
employee_skill_mit_violin,hr.employee_mit,hr_skill_violin,hr_skill_type_music,hr_skill_level_l4
employee_skill_mit_flute,hr.employee_mit,hr_skill_flute,hr_skill_type_music,hr_skill_level_l2
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
employee_skill_niv_flute,hr.employee_niv,hr_skill_flute,hr_skill_type_music,hr_skill_level_l4
employee_skill_niv_singing,hr.employee_niv,hr_skill_singing,hr_skill_type_music,hr_skill_level_l4
employee_skill_stw_com,hr.employee_stw,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_stw_digital_ad,hr.employee_stw,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_chs_digital_ad,hr.employee_chs,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_chs_email,hr.employee_chs,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_chs_arabic,hr.employee_chs,hr_skill_arabic,hr_skill_type_lang,hr_skill_level_c1
employee_skill_chs_piano,hr.employee_chs,hr_skill_piano,hr_skill_type_music,hr_skill_level_l4
employee_skill_qdp_com,hr.employee_qdp,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_qdp_email,hr.employee_qdp,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_qdp_analytics,hr.employee_qdp,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_qdp_nosql,hr.employee_qdp,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_qdp_js,hr.employee_qdp,hr_skill_js,hr_skill_type_dev,hr_skill_level_expert
employee_skill_qdp_flute,hr.employee_qdp,hr_skill_flute,hr_skill_type_music,hr_skill_level_l1
employee_skill_qdp_violin,hr.employee_qdp,hr_skill_violin,hr_skill_type_music,hr_skill_level_l4
employee_skill_qdp_singing,hr.employee_qdp,hr_skill_singing,hr_skill_type_music,hr_skill_level_l4
employee_skill_qdp_bengali,hr.employee_qdp,hr_skill_bengali,hr_skill_type_lang,hr_skill_level_b2
employee_skill_qdp_english,hr.employee_qdp,hr_skill_english,hr_skill_type_lang,hr_skill_level_b1
employee_skill_fme_spark,hr.employee_fme,hr_skill_spark,hr_skill_type_dev,hr_skill_level_beginner
employee_skill_fme_com,hr.employee_fme,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_fpi_django,hr.employee_fpi,hr_skill_django,hr_skill_type_dev,hr_skill_level_expert
employee_skill_fpi_violin,hr.employee_fpi,hr_skill_violin,hr_skill_type_music,hr_skill_level_l1
employee_skill_fpi_piano,hr.employee_fpi,hr_skill_piano,hr_skill_type_music,hr_skill_level_l1
employee_skill_fpi_singing,hr.employee_fpi,hr_skill_singing,hr_skill_type_music,hr_skill_level_l4
employee_skill_fpi_flute,hr.employee_fpi,hr_skill_flute,hr_skill_type_music,hr_skill_level_l2
employee_skill_fpi_cms,hr.employee_fpi,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jth_hadoop,hr.employee_jth,hr_skill_hadoop,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jth_nosql,hr.employee_jth,hr_skill_nosql,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_jth_c,hr.employee_jth,hr_skill_c,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_ngh_violin,hr.employee_ngh,hr_skill_violin,hr_skill_type_music,hr_skill_level_l4
employee_skill_ngh_piano,hr.employee_ngh,hr_skill_piano,hr_skill_type_music,hr_skill_level_l1
employee_skill_ngh_flute,hr.employee_ngh,hr_skill_flute,hr_skill_type_music,hr_skill_level_l4
employee_skill_vad_sql,hr.employee_vad,hr_skill_sql,hr_skill_type_dev,hr_skill_level_intermediate
employee_skill_vad_js,hr.employee_vad,hr_skill_js,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_vad_spark,hr.employee_vad,hr_skill_spark,hr_skill_type_dev,hr_skill_level_expert
employee_skill_vad_python,hr.employee_vad,hr_skill_python,hr_skill_type_dev,hr_skill_level_expert
employee_skill_vad_french,hr.employee_vad,hr_skill_french,hr_skill_type_lang,hr_skill_level_a2
employee_skill_vad_singing,hr.employee_vad,hr_skill_singing,hr_skill_type_music,hr_skill_level_l2
employee_skill_vad_public,hr.employee_vad,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_vad_cms,hr.employee_vad,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_vad_analytics,hr.employee_vad,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_vad_digital_ad,hr.employee_vad,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_han_bengali,hr.employee_han,hr_skill_bengali,hr_skill_type_lang,hr_skill_level_b2
employee_skill_han_python,hr.employee_han,hr_skill_python,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_han_react,hr.employee_han,hr_skill_react,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_han_analytics,hr.employee_han,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_han_digital_ad,hr.employee_han,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_han_violin,hr.employee_han,hr_skill_violin,hr_skill_type_music,hr_skill_level_l1
employee_skill_han_flute,hr.employee_han,hr_skill_flute,hr_skill_type_music,hr_skill_level_l1
employee_skill_han_piano,hr.employee_han,hr_skill_piano,hr_skill_type_music,hr_skill_level_l3
employee_skill_jve_cms,hr.employee_jve,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jve_email,hr.employee_jve,hr_skill_email,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jve_digital_ad,hr.employee_jve,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jve_com,hr.employee_jve,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_jve_violin,hr.employee_jve,hr_skill_violin,hr_skill_type_music,hr_skill_level_l2
employee_skill_jve_singing,hr.employee_jve,hr_skill_singing,hr_skill_type_music,hr_skill_level_l3
employee_skill_jve_flute,hr.employee_jve,hr_skill_flute,hr_skill_type_music,hr_skill_level_l4
employee_skill_jve_piano,hr.employee_jve,hr_skill_piano,hr_skill_type_music,hr_skill_level_l3
employee_skill_jve_french,hr.employee_jve,hr_skill_french,hr_skill_type_lang,hr_skill_level_b1
employee_skill_jve_spark,hr.employee_jve,hr_skill_spark,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jve_c,hr.employee_jve,hr_skill_c,hr_skill_type_dev,hr_skill_level_elementary
employee_skill_jve_js,hr.employee_jve,hr_skill_js,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jve_hadoop,hr.employee_jve,hr_skill_hadoop,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_jep_piano,hr.employee_jep,hr_skill_piano,hr_skill_type_music,hr_skill_level_l1
employee_skill_jep_flute,hr.employee_jep,hr_skill_flute,hr_skill_type_music,hr_skill_level_l4
employee_skill_jep_singing,hr.employee_jep,hr_skill_singing,hr_skill_type_music,hr_skill_level_l4
employee_skill_jep_violin,hr.employee_jep,hr_skill_violin,hr_skill_type_music,hr_skill_level_l4
employee_skill_jod_singing,hr.employee_jod,hr_skill_singing,hr_skill_type_music,hr_skill_level_l4
employee_skill_jod_violin,hr.employee_jod,hr_skill_violin,hr_skill_type_music,hr_skill_level_l4
employee_skill_jod_filipino,hr.employee_jod,hr_skill_filipino,hr_skill_type_lang,hr_skill_level_c1
employee_skill_jod_spark,hr.employee_jod,hr_skill_spark,hr_skill_type_dev,hr_skill_level_advanced
employee_skill_jod_sql,hr.employee_jod,hr_skill_sql,hr_skill_type_dev,hr_skill_level_expert
employee_skill_jod_analytics,hr.employee_jod,hr_skill_analytics,hr_skill_type_marketing,hr_skill_level_ml1
employee_skill_jod_public,hr.employee_jod,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml2
employee_skill_jog_violin,hr.employee_jog,hr_skill_violin,hr_skill_type_music,hr_skill_level_l4
employee_skill_jog_singing,hr.employee_jog,hr_skill_singing,hr_skill_type_music,hr_skill_level_l1
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
employee_skill_lur_singing,hr.employee_lur,hr_skill_singing,hr_skill_type_music,hr_skill_level_l3
employee_skill_hne_spanish,hr.employee_hne,hr_skill_spanish,hr_skill_type_lang,hr_skill_level_c1
employee_skill_hne_cms,hr.employee_hne,hr_skill_cms,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_hne_public,hr.employee_hne,hr_skill_public,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_hne_com,hr.employee_hne,hr_skill_com,hr_skill_type_marketing,hr_skill_level_ml4
employee_skill_hne_digital_ad,hr.employee_hne,hr_skill_digital_ad,hr_skill_type_marketing,hr_skill_level_ml3
employee_skill_hne_singing,hr.employee_hne,hr_skill_singing,hr_skill_type_music,hr_skill_level_l2
employee_skill_hne_flute,hr.employee_hne,hr_skill_flute,hr_skill_type_music,hr_skill_level_l3
employee_skill_hne_piano,hr.employee_hne,hr_skill_piano,hr_skill_type_music,hr_skill_level_l4
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
    </data>

</odoo>

```

## File: data\hr_resume_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <!--Skill Types-->
    <record id="hr_skill_type_lang" model="hr.skill.type">
        <field name="name">Languages</field>
    </record>
    <record id="hr_skill_type_dev" model="hr.skill.type">
        <field name="name">Dev</field>
    </record>
    <record id="hr_skill_type_music" model="hr.skill.type">
        <field name="name">Music</field>
    </record>
    <record id="hr_skill_type_marketing" model="hr.skill.type">
        <field name="name">Marketing</field>
    </record>

    <!--Skill Levels-->
    <record id="hr_skill_level_a1" model="hr.skill.level">
        <field name="name">A1</field>
        <field name="level_progress">10</field>
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

    <record id="hr_skill_level_beginner" model="hr.skill.level">
        <field name="name">Beginner</field>
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

    <record id="hr_skill_level_l1" model="hr.skill.level">
        <field name="name">L1</field>
        <field name="level_progress">25</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>
    <record id="hr_skill_level_l2" model="hr.skill.level">
        <field name="name">L2</field>
        <field name="level_progress">50</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>
    <record id="hr_skill_level_l3" model="hr.skill.level">
        <field name="name">L3</field>
        <field name="level_progress">75</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>
    <record id="hr_skill_level_l4" model="hr.skill.level">
        <field name="name">L4</field>
        <field name="level_progress">100</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>

    <record id="hr_skill_level_ml1" model="hr.skill.level">
        <field name="name">L1</field>
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

    <!-- **** Skills **** -->
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

    <!-- Dev -->
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

    <!-- Music -->
    <record id="hr_skill_piano" model="hr.skill">
        <field name="name">Piano</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>
    <record id="hr_skill_violin" model="hr.skill">
        <field name="name">Violin</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>
    <record id="hr_skill_singing" model="hr.skill">
        <field name="name">Singing</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
    </record>
    <record id="hr_skill_flute" model="hr.skill">
        <field name="name">Flute</field>
        <field name="skill_type_id" ref="hr_skill_type_music"/>
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

    <!-- Resume -->
    <record id="employee_resume_line_admin_1" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Université Libre de Bruxelles - Polytechnique</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-12)).strftime('%Y-09-17')"/>
        <field name="date_end" eval="(datetime.now()+relativedelta(years=-7)).strftime('%Y-09-10')"/>
        <field name="line_type_id" ref="resume_type_education"/>
        <field name="description">
            Master in Electrical engineering
            Master thesis: Better grid management and control through machine learning
        </field>
    </record>

    <record id="employee_resume_line_admin_2" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Saint-Joseph School</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-18)).strftime('%Y-09-01')"/>
        <field name="date_end" eval="(datetime.now()+relativedelta(years=-12)).strftime('%Y-06-30')"/>
        <field name="line_type_id" ref="resume_type_education"/>
        <field name="description">
            Science &amp; math
        </field>
    </record>

    <record id="employee_resume_line_admin_4" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Odoo SA</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-11-01')"/>
        <field name="line_type_id" ref="resume_type_experience"/>
        <field name="description">
            Job position: Development team leader
            Core Python Framework
        </field>
    </record>

    <record id="employee_resume_line_admin_3" model="hr.resume.line">
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="name">Burtho Inc.</field>
        <field name="date_start" eval="(datetime.now()+relativedelta(years=-7)).strftime('%Y-09-10')"/>
        <field name="date_end" eval="(datetime.now()+relativedelta(years=-3)).strftime('%Y-09-10')"/>
        <field name="line_type_id" ref="resume_type_experience"/>
        <field name="description">
            Job position: Product manager
            Business strategy, functional requirements, resource planning, product lifecycle management, etc.
        </field>
    </record>
</data>
</odoo>

```

## File: data\ir_actions_server_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_open_skills_log_employee" model="ir.actions.server">
        <field name="name">Skill History Report</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">
action = env['ir.actions.act_window']._for_xml_id('hr_skills.action_hr_employee_skill_log_employee')
action['domain'] = [('employee_id', '=', record.id)]
        </field>
    </record>

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
</odoo>
```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Employee(models.Model):
    _inherit = 'hr.employee'

    resume_line_ids = fields.One2many('hr.resume.line', 'employee_id', string="Resume lines")
    employee_skill_ids = fields.One2many('hr.employee.skill', 'employee_id', string="Skills")
    skill_ids = fields.Many2many('hr.skill', compute='_compute_skill_ids', store=True)

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

```

## File: models\hr_employee_public.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    resume_line_ids = fields.One2many('hr.resume.line', 'employee_id', string="Resume lines")
    employee_skill_ids = fields.One2many('hr.employee.skill', 'employee_id', string="Skills")

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
    _rec_name = 'skill_id'
    _order = "skill_type_id, skill_level_id"

    employee_id = fields.Many2one('hr.employee', required=True, ondelete='cascade')
    skill_id = fields.Many2one('hr.skill', compute='_compute_skill_id', store=True, domain="[('skill_type_id', '=', skill_type_id)]", readonly=False, required=True, ondelete='cascade')
    skill_level_id = fields.Many2one('hr.skill.level', compute='_compute_skill_level_id', domain="[('skill_type_id', '=', skill_type_id)]", store=True, readonly=False, required=True, ondelete='cascade')
    skill_type_id = fields.Many2one('hr.skill.type', required=True, ondelete='cascade')
    level_progress = fields.Integer(related='skill_level_id.level_progress')

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
            if record.skill_id.skill_type_id != record.skill_type_id:
                record.skill_id = False

    @api.depends('skill_id')
    def _compute_skill_level_id(self):
        for record in self:
            if not record.skill_id:
                record.skill_level_id = False
            else:
                skill_levels = record.skill_type_id.skill_level_ids
                record.skill_level_id = skill_levels.filtered('default_level') or skill_levels[0] if skill_levels else False

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
    level_progress = fields.Integer(related='skill_level_id.level_progress', store=True, group_operator="avg")
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

    employee_id = fields.Many2one('hr.employee', required=True, ondelete='cascade')
    name = fields.Char(required=True)
    date_start = fields.Date(required=True)
    date_end = fields.Date()
    description = fields.Text(string="Description")
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

    name = fields.Char(required=True)
    sequence = fields.Integer('Sequence', default=10)

```

## File: models\hr_skill.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Skill(models.Model):
    _name = 'hr.skill'
    _description = "Skill"
    _order = "sequence, name"

    name = fields.Char(required=True)
    sequence = fields.Integer(default=10)
    skill_type_id = fields.Many2one('hr.skill.type', required=True, ondelete='cascade')

    def name_get(self):
        if not self._context.get('from_skill_dropdown'):
            return super().name_get()
        return [(record.id, f"{record.name} ({record.skill_type_id.name})") for record in self]

```

## File: models\hr_skill_level.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


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

    def name_get(self):
        if not self._context.get('from_skill_level_dropdown'):
            return super().name_get()
        return [(record.id, f"{record.name} ({record.level_progress}%)") for record in self]

    @api.model_create_multi
    def create(self, vals_list):
        levels = super().create(vals_list)
        levels.skill_type_id._set_default_level()
        return levels

    def write(self, values):
        levels = super().write(values)
        self.skill_type_id._set_default_level()
        return levels

    def unlink(self):
        skill_types = self.skill_type_id
        res = super().unlink()
        skill_types._set_default_level()
        return res

    @api.constrains('default_level', 'skill_type_id')
    def _constrains_default_level(self):
        for skill_type in set(self.mapped('skill_type_id')):
            if len(skill_type.skill_level_ids.filtered('default_level')) > 1:
                raise ValidationError(_('Only one default level is allowed per skill type.'))

    def action_set_default(self):
        self.ensure_one()
        self.skill_type_id.skill_level_ids.with_context(no_skill_level_check=True).default_level = False
        self.default_level = True

```

## File: models\hr_skill_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SkillType(models.Model):
    _name = 'hr.skill.type'
    _description = "Skill Type"
    _order = "name"

    name = fields.Char(required=True)
    skill_ids = fields.One2many('hr.skill', 'skill_type_id', string="Skills")
    skill_level_ids = fields.One2many('hr.skill.level', 'skill_type_id', string="Levels")

    def _set_default_level(self):
        if self.env.context.get('no_skill_level_check'):
            return

        for types in self:
            if not types.skill_level_ids.filtered('default_level'):
                types.skill_level_ids[:1].default_level = True

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
        return super().SELF_READABLE_FIELDS + ['resume_line_ids', 'employee_skill_ids']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['resume_line_ids', 'employee_skill_ids']

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

```

## File: report\hr_employee_skill_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools

class HrEmployeeSkillReport(models.BaseModel):
    _auto = False
    _name = 'hr.employee.skill.report'
    _description = 'Employee Skills Report'
    _order = 'employee_id, level_progress desc'

    id = fields.Id()
    display_name = fields.Char(related='employee_id.name')
    employee_id = fields.Many2one('hr.employee', readonly=True)
    company_id = fields.Many2one('res.company', readonly=True)
    department_id = fields.Many2one('hr.department', readonly=True)

    skill_id = fields.Many2one('hr.skill', readonly=True)
    skill_type_id = fields.Many2one('hr.skill.type', readonly=True)
    skill_level = fields.Char(readonly=True)
    level_progress = fields.Float(readonly=True, group_operator='avg')

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

    <record id="hr_employee_skill_report_view_list" model="ir.ui.view">
        <field name="model">hr.employee.skill.report</field>
        <field name="arch" type="xml">
            <tree expand="1">
                <field name="employee_id"/>
                <field name="skill_type_id"/>
                <field name="skill_id"/>
                <field name="skill_level"/>
                <field name="level_progress" widget="percentage"/>
            </tree>
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
        <field name="view_mode">tree,pivot</field>
        <field name="context">{
            'search_default_employee': 1,
            'search_default_employees_with_skills': 1,
        }</field>
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
access_hr_employee_skill_report,hr.employee.skill.report,model_hr_employee_skill_report,hr.group_hr_user,1,0,0,0
access_hr_employee_skill_log,hr.employee.skill.log,model_hr_employee_skill_log,hr.group_hr_user,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="mask-2" data-name="mask">
        <g id="b">
          <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
        </g>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-909.8" y1="-216.96" x2="-910.8" y2="-217.96" gradientTransform="matrix(70, 0, 0, -70, 63756, -15187.42)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#269396"/>
      <stop offset="1" stop-color="#218689"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M43,69H4c-2,0-4-.14-4-4V36.23L10.93,25.3l6-5,4.15,2.53L30.9,13l5.49-1.51,3.87,2.33L40,21.49,16.5,45.05l.72,1.51L30.16,33.63,40.85,33,52.13,21.68,54.77,21l5,9.6L41.22,49.17l7.69,1.15,5.64-5.64,1.24,11.49Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <g>
            <g>
              <path d="M33.94,32.4h0a9.12,9.12,0,0,0,0,18.23h.19a9.12,9.12,0,0,0-.19-18.23Z"/>
              <path d="M55.66,45a2.5,2.5,0,0,0-3.1,1.7L50.71,53,40.34,50.41a11,11,0,0,1-12.77,0L17.18,53.13,15.3,46.67a2.5,2.5,0,0,0-4.81,1.4L13,56.69a2.49,2.49,0,0,0,2.39,1.8,2.38,2.38,0,0,0,.46,0l8.94-1.64v4.76H43.08V56.38l8.81,2a2.74,2.74,0,0,0,.56.06,2.49,2.49,0,0,0,2.4-1.8l2.51-8.62A2.49,2.49,0,0,0,55.66,45Z"/>
            </g>
            <g>
              <path d="M14.91,36a1.77,1.77,0,0,1-1.26-.52L8.08,29.94a1.77,1.77,0,0,1-.52-1.26,1.74,1.74,0,0,1,.52-1.26l5.57-5.58a1.8,1.8,0,0,1,2.53,0l5.57,5.57a1.8,1.8,0,0,1,0,2.53l-5.57,5.57A1.78,1.78,0,0,1,14.91,36ZM9.65,28.67l5.27,5.27,5.26-5.26-5.27-5.27Z"/>
              <path d="M56.24,33.48H48a1.79,1.79,0,0,1-1.59-2.58h0l4.14-8.28a1.76,1.76,0,0,1,1.59-1h0a1.79,1.79,0,0,1,1.6,1l4.14,8.28a1.75,1.75,0,0,1-.08,1.73A1.77,1.77,0,0,1,56.24,33.48Zm-7.93-2h7.58L52.1,23.9Z"/>
              <path d="M33.58,25.22a6.38,6.38,0,0,1-4.53-1.87h0a6.42,6.42,0,1,1,4.53,1.87Zm-3.12-3.28a4.41,4.41,0,1,0,0-6.24,4.43,4.43,0,0,0,0,6.24Z"/>
            </g>
          </g>
        </g>
        <g>
          <g>
            <path d="M35.93,30.42h0a9.12,9.12,0,0,0,0,18.23h.19a9.12,9.12,0,0,0-.19-18.23Z" fill="#fff"/>
            <path d="M57.65,43a2.5,2.5,0,0,0-3.1,1.7L52.7,51l-10.37-2.6a11.06,11.06,0,0,1-12.77,0l-10.4,2.71-1.88-6.46a2.5,2.5,0,1,0-4.8,1.4L15,54.7a2.45,2.45,0,0,0,2.86,1.76l8.94-1.64v4.76H45.07V54.39l8.8,2a2.78,2.78,0,0,0,.57.07,2.5,2.5,0,0,0,2.4-1.81l2.51-8.62A2.49,2.49,0,0,0,57.65,43Z" fill="#fff"/>
          </g>
          <g>
            <path d="M16.9,34a1.77,1.77,0,0,1-1.26-.52L10.07,28a1.77,1.77,0,0,1-.52-1.26,1.74,1.74,0,0,1,.52-1.26l5.57-5.58a1.79,1.79,0,0,1,2.52,0l5.58,5.58a1.74,1.74,0,0,1,.52,1.26A1.81,1.81,0,0,1,23.74,28l-5.57,5.57A1.78,1.78,0,0,1,16.9,34Zm-5.26-7.35L16.9,32l5.27-5.27L16.9,21.42Z" fill="#fff"/>
            <path d="M58.23,31.49H50a1.75,1.75,0,0,1-1.51-.84,1.77,1.77,0,0,1-.08-1.74h0l4.14-8.28a1.75,1.75,0,0,1,1.59-1h0a1.78,1.78,0,0,1,1.6,1l4.14,8.28a1.79,1.79,0,0,1-1.6,2.58Zm-7.93-2h7.58l-3.79-7.58Z" fill="#fff"/>
            <path d="M35.57,23.24A6.39,6.39,0,0,1,31,21.36h0a6.4,6.4,0,1,1,4.53,1.88ZM32.45,20a4.41,4.41,0,1,0,0-6.23,4.41,4.41,0,0,0,0,6.23Z" fill="#fff"/>
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\fields\resume_one2many.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";

import { formatDate } from "@web/core/l10n/dates";

import { SkillsX2ManyField } from "./skills_one2many";
import { CommonSkillsListRenderer } from "../views/skills_list_renderer";

export class ResumeListRenderer extends CommonSkillsListRenderer {
    get groupBy() {
        return 'line_type_id';
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

    setDefaultColumnWidths() {}
}
ResumeListRenderer.template = 'hr_skills.ResumeListRenderer';
ResumeListRenderer.rowsTemplate = "hr_skills.ResumeListRenderer.Rows";
ResumeListRenderer.recordRowTemplate = "hr_skills.ResumeListRenderer.RecordRow";


export class ResumeX2ManyField extends SkillsX2ManyField {}
ResumeX2ManyField.components = {
    ...SkillsX2ManyField.components,
    ListRenderer: ResumeListRenderer,
};

registry.category("fields")
    .add("resume_one2many", ResumeX2ManyField);

```

## File: static\src\fields\resume_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_skills.ResumeListRenderer" owl="1" t-inherit-mode="primary" t-inherit="hr_skills.SkillsListRenderer">
        <xpath expr="//table" position="attributes">
            <attribute name="t-attf-class" add="table-borderless {{ !showTable ? 'd-none' : ''}}" remove="table-striped" separator=" "/>
        </xpath>
        <xpath expr="//thead/tr" position="replace">
            <tr>
                <th style="width: 32px; min-width: 32px;"></th>
                <th class="w-100"></th>
                <th t-if="isEditable" class="o_list_actions_header" style="width: 32px; min-width: 32px"></th>
            </tr>
        </xpath>
    </t>

    <t t-name="hr_skills.ResumeListRenderer.Rows" owl="1" t-inherit-mode="primary" t-inherit="hr_skills.SkillsListRenderer.Rows">
        <xpath expr="//tr" position="attributes">
            <attribute name="class" add="o_resume_group_header" separator=" "/>
        </xpath>
        <xpath expr="//th[hasclass('o_group_name')]" position="after">
            <th></th>
        </xpath>
    </t>

    <t t-name="hr_skills.ResumeListRenderer.RecordRow" owl="1" t-inherit-mode="primary" t-inherit="web.ListRenderer.RecordRow">
        <xpath expr="//t[@t-foreach='getColumns(record)']" position="replace">
            <t t-set="data" t-value="record.data"/>
            <t t-if="data.display_type === 'classic'" id='row'>
                <td class="o_resume_timeline_cell position-relative pe-lg-2" id='hiii'>
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

## File: static\src\fields\skills_one2many.js

```javascript
/** @odoo-module */

import { X2ManyField } from "@web/views/fields/x2many/x2many_field";
import { registry } from "@web/core/registry";

import { CommonSkillsListRenderer } from "../views/skills_list_renderer";


export class SkillsListRenderer extends CommonSkillsListRenderer {
    get groupBy() {
        return 'skill_type_id';
    }

    calculateColumnWidth(column) {
        if (column.name != 'skill_level_id') {
            return {
                type: 'absolute',
                value: '90px',
            }
        }

        return super.calculateColumnWidth(column);
    }
}
SkillsListRenderer.template = 'hr_skills.SkillsListRenderer';

export class SkillsX2ManyField extends X2ManyField {
    async onAdd({ context, editable } = {}) {
        const employeeId = this.props.record.resId;
        return super.onAdd({
            editable,
            context: {
                ...context,
                default_employee_id: employeeId,
            }
        });
    }
}
SkillsX2ManyField.components = {
    ...X2ManyField.components,
    ListRenderer: SkillsListRenderer,
};

registry.category("fields").add("skills_one2many", SkillsX2ManyField);

```

## File: static\src\fields\skills_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_skills.SkillsListRenderer" owl="1" t-inherit-mode="primary" t-inherit="web.ListRenderer">
        <xpath expr="//table" position="attributes">
            <attribute name="t-attf-class" add="mb-1 {{ !isEditable ? 'cursor-default' : '' }} {{ !showTable ? 'd-none' : ''}} o_skill_table" separator=" "/>
        </xpath>
        <xpath expr="//thead" position="attributes">
            <attribute name="style">visibility: collapse;</attribute>
        </xpath>
        <xpath expr="//table" position="after">
            <t t-if="!showTable">
                <button t-on-click="props.onAdd" class="btn btn-secondary ms-4 mt-3" role="button" t-if="isEditable">
                    Create a new entry
                </button>
            </t>
        </xpath>
    </t>

    <t t-name="hr_skills.SkillsListRenderer.Rows" owl="1">
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

        if ('yAxes' in scaleOptions) {
            scaleOptions['yAxes'][0]['ticks']['suggestedMax'] = 100;
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
/** @odoo-module */

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
                    name: group[1] || this.env._t('Other'),
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
                <a role="menuitem" class="dropdown-item" name="%(action_open_skills_log_department)d" type="action">
                    Skills History
                </a>
            </xpath>
        </field>
    </record>
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
          <field name="name">hr.employee.skill.log.view.tree</field>
          <field name="model">hr.employee.skill.log</field>
          <field name="arch" type="xml">
            <tree string="Skills History">
                <field name="employee_id"/>
                <field name="department_id"/>
                <field name="skill_type_id"/>
                <field name="skill_id"/>
                <field name="level_progress"/>
                <field name="date"/>
            </tree>
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
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">hr.employee.skill.log</field>
        <field name="view_mode">graph,tree</field>
        <field name="view_id" ref="hr_employee_skill_log_view_graph_employee"/>
        <field name="context">{'fill_temporal': 0}</field>
        <field name="target">current</field>
    </record>

    <record id="action_hr_employee_skill_log_department" model="ir.actions.act_window">
        <field name="name">Skill History Report</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">hr.employee.skill.log</field>
        <field name="view_mode">graph,tree</field>
        <field name="view_id" ref="hr_employee_skill_log_view_graph_department"/>
        <field name="context">{'fill_temporal': 0, 'search_default_group_by_skill_type_id': 1, 'search_default_group_by_skill_id': 2}</field>
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
                <field name="resume_line_ids" string="Resume" filter_domain="['|', ('resume_line_ids.name', 'ilike', self), ('resume_line_ids.description', 'ilike', self)]"/>
            </xpath>
            <filter name="group_job" position="after">
                <filter name="group_by_skill_ids" string="Skills" domain="[]" context="{'group_by': 'skill_ids'}"/>
            </filter>
        </field>
    </record>

    <record id="hr_employee_public_view_search" model="ir.ui.view">
        <field name="name">hr.employee.public.skill.search</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='company_id']" position="after">
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
                            <field name="date_start" required="True"/>
                            <field name="date_end"/>
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
                            <!-- This field uses a custom tree view rendered by the 'resume_one2many' widget.
                                Adding fields in the tree arch below makes them accessible to the widget
                            -->
                            <field mode="tree" nolabel="1" name="resume_line_ids" widget="resume_one2many">
                                <tree>
                                    <field name="line_type_id"/>
                                    <field name="name"/>
                                    <field name="description"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="display_type" invisible="1"/>
                                </tree>
                            </field>

                        </div>
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <separator string="Skills" class="mb-4"/>
                            <field mode="tree" nolabel="1" name="employee_skill_ids" widget="skills_one2many">
                                <tree>
                                    <field name="skill_type_id" invisible="1"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </tree>
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
                        <div class="o_hr_skills_group o_group_resume col-lg-7 d-flex flex-column">
                            <!-- This field uses a custom tree view rendered by the 'resume_one2many' widget.
                                Adding fields in the tree arch below makes them accessible to the widget
                            -->
                            <field mode="tree" nolabel="1" name="resume_line_ids" widget="resume_one2many">
                                <tree>
                                    <field name="line_type_id"/>
                                    <field name="name"/>
                                    <field name="description"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="display_type" invisible="1"/>
                                </tree>
                            </field>
                        </div>
                        <div class="o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <separator string="Skills"/>
                            <field mode="tree" nolabel="1" name="employee_skill_ids"  widget="skills_one2many">
                                <tree>
                                    <field name="skill_type_id" invisible="1"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </tree>
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
                        <div class="o_hr_skills_group o_group_resume col-lg-7 d-flex">
                            <!-- This field uses a custom tree view rendered by the 'resume_one2many' widget.
                                Adding fields in the tree arch below makes them accessible to the widget
                            -->
                            <field mode="tree" nolabel="1" name="resume_line_ids" widget="resume_one2many" attrs="{'readonly': [('can_edit', '=', False)]}">
                                <tree>
                                    <field name="line_type_id"/>
                                    <field name="name"/>
                                    <field name="description"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="display_type" invisible="1"/>
                                </tree>
                            </field>
                        </div>
                        <div class="o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <separator string="Skills"/>
                            <field mode="tree" nolabel="1" name="employee_skill_ids"  widget="skills_one2many" attrs="{'readonly': [('can_edit', '=', False)]}">
                                <tree>
                                    <field name="skill_type_id" invisible="1"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </tree>
                            </field>
                        </div>
                    </div>
                </page>
            </xpath>
        </field>
    </record>

    <record id="hr_resume_line_type_tree_view" model="ir.ui.view">
        <field name="name">hr.resume.line.type.tree.view</field>
        <field name="model">hr.resume.line.type</field>
        <field name="arch" type="xml">
            <tree name="Resume Line Types" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="hr_resume_type_action" model="ir.actions.act_window">
        <field name="name">Resume Line Types</field>
        <field name="res_model">hr.resume.line.type</field>
        <field name="view_mode">tree,form</field>
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
        <field name="name">hr.skill.level.tree</field>
        <field name="model">hr.skill.level</field>
        <field name="arch" type="xml">
            <tree string="Skill Levels">
                <field name="name"/>
                <field name="level_progress" widget="progressbar"/>
                <field name="default_level"/>
                <button string="Set Default" type="object" name="action_set_default" attrs="{'invisible': [('default_level', '=', True)]}"/>
            </tree>
        </field>
    </record>

    <record id="employee_skill_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.tree</field>
        <field name="model">hr.skill</field>
        <field name="arch" type="xml">
            <tree string="Skill Levels">
                <field name="name"/>
                <field name="skill_type_id"/>
            </tree>
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
                            <field name="skill_id" options="{'no_open': True, 'no_create_edit': True}"
                                    context="{'default_skill_type_id': skill_type_id}"
                                    domain="[('skill_type_id', '=', skill_type_id)]"
                                    attrs="{'invisible': [('skill_type_id', '=', False)]}"/>
                            <label for="skill_level_id"
                                    attrs="{'invisible': ['|', ('skill_id', '=', False), ('skill_type_id', '=', False)]}"/>
                            <div class="o_row"
                                    attrs="{'invisible': ['|', ('skill_id', '=', False), ('skill_type_id', '=', False)]}">
                                <span class="ps-0" style="flex:1">
                                    <field name="skill_level_id"
                                            attrs="{'readonly': [('skill_id', '=', False)]}"
                                            context="{'from_skill_level_dropdown': True, 'default_skill_type_id': skill_type_id}" />
                                </span>
                                <span style="flex:1">
                                    <field name="level_progress" widget="progressbar" class="o_hr_skills_progress" attrs="{'invisible': [('skill_level_id', '=', False)]}" />
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

    <record id="hr_skill_type_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.type.tree</field>
        <field name="model">hr.skill.type</field>
        <field name="arch" type="xml">
            <tree string="Skill Types">
                <field name="name"/>
                <field name="skill_ids" widget="many2many_tags"/>
            </tree>
        </field>
    </record>

    <record id="hr_employee_skill_type_view_form" model="ir.ui.view">
        <field name="name">hr.skill.type.form</field>
        <field name="model">hr.skill.type</field>
        <field name="arch" type="xml">
            <form string="Skill Type">
                <field name="id" invisible="1"/>
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Skill Type"/>
                        <h1>
                            <field name="name" placeholder="e.g. Languages" required="True"/>
                        </h1>
                    </div>
                    <group string="Skills">
                    </group>
                    <field name="skill_ids" nolabel="1" context="{'default_skill_type_id': id}">
                        <tree editable="bottom">
                            <field name="sequence" widget="handle" />
                            <field name="name"/>
                        </tree>
                    </field>
                    <group string="Levels">
                    </group>
                    <field name="skill_level_ids" nolabel="1" context="{'default_skill_type_id': id}"/>
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

