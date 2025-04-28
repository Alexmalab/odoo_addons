# Odoo Module: hr_skills

Category: Human Resources/Employees

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
    'name': 'Skills Management',
    'category': 'Human Resources/Employees',
    'sequence': 270,
    'version': '1.0',
    'summary': 'Manage skills, knowledge and resumé of your employees',
    'description':
        """
Skills and Resumé for HR
========================

This module introduces skills and resumé management for employees.
        """,
    'depends': ['hr'],
    'data': [
        'security/ir.model.access.csv',
        'security/hr_skills_security.xml',
        'views/hr_views.xml',
        'data/hr_resume_data.xml',
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
            'hr_skills/static/src/css/hr_skills.scss',
            'hr_skills/static/src/js/resume_widget.js',
        ],
        'web.qunit_suite_tests': [
            'hr_skills/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'hr_skills/static/src/xml/**/*',
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

    <!-- Resumé -->
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

## File: models\hr_resume.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Employee(models.Model):
    _inherit = 'hr.employee'

    resume_line_ids = fields.One2many('hr.resume.line', 'employee_id', string="Resumé lines")
    employee_skill_ids = fields.One2many('hr.employee.skill', 'employee_id', string="Skills")

    @api.model_create_multi
    def create(self, vals_list):
        res = super(Employee, self).create(vals_list)
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


class EmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    resume_line_ids = fields.One2many('hr.resume.line', 'employee_id', string="Resumé lines")
    employee_skill_ids = fields.One2many('hr.employee.skill', 'employee_id', string="Skills")


class ResumeLine(models.Model):
    _name = 'hr.resume.line'
    _description = "Resumé line of an employee"
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


class ResumeLineType(models.Model):
    _name = 'hr.resume.line.type'
    _description = "Type of a resumé line"
    _order = "sequence"

    name = fields.Char(required=True)
    sequence = fields.Integer('Sequence', default=10)

```

## File: models\hr_skills.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class Skill(models.Model):
    _name = 'hr.skill'
    _description = "Skill"

    name = fields.Char(required=True)
    skill_type_id = fields.Many2one('hr.skill.type', ondelete='cascade')


class EmployeeSkill(models.Model):
    _name = 'hr.employee.skill'
    _description = "Skill level for an employee"
    _rec_name = 'skill_id'
    _order = "skill_level_id"

    employee_id = fields.Many2one('hr.employee', required=True, ondelete='cascade')
    skill_id = fields.Many2one('hr.skill', required=True)
    skill_level_id = fields.Many2one('hr.skill.level', required=True)
    skill_type_id = fields.Many2one('hr.skill.type', required=True)
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


class SkillLevel(models.Model):
    _name = 'hr.skill.level'
    _description = "Skill Level"
    _order = "level_progress desc"

    skill_type_id = fields.Many2one('hr.skill.type', ondelete='cascade')
    name = fields.Char(required=True)
    level_progress = fields.Integer(string="Progress", help="Progress from zero knowledge (0%) to fully mastered (100%).")


class SkillType(models.Model):
    _name = 'hr.skill.type'
    _description = "Skill Type"

    name = fields.Char(required=True)
    skill_ids = fields.One2many('hr.skill', 'skill_type_id', string="Skills")
    skill_level_ids = fields.One2many('hr.skill.level', 'skill_type_id', string="Levels")

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

from . import hr_resume
from . import hr_skills
from . import res_users

```

## File: security\hr_skills_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="hr_resume_rule_employee" model="ir.rule">
        <field name="name">Resumé: employee: read all</field>
        <field name="model_id" ref="model_hr_resume_line"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_resume_rule_employee_hr_user" model="ir.rule">
        <field name="name">Resumé: HR user: all</field>
        <field name="model_id" ref="model_hr_resume_line"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4,ref('hr.group_hr_user'))]"/>
    </record>

    <record id="hr_skills_rule_employee_update" model="ir.rule">
        <field name="name">Resumé: employee: create/write/unlink own</field>
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
access_hr_skill_employee,hr.skill.employee,model_hr_skill,base.group_user,1,0,0,0
access_hr_employee_skill,hr.employee.skill,model_hr_employee_skill,hr.group_hr_user,1,1,1,1
access_hr_employee_skill_employee,hr.employee.skill,model_hr_employee_skill,base.group_user,1,1,1,1

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

## File: static\src\js\resume_widget.js

```javascript
odoo.define('web.FieldResume', function (require) {
"use strict";

var time = require('web.time');
var FieldOne2Many = require('web.relational_fields').FieldOne2Many;
var FieldProgressBar = require('web.basic_fields').FieldProgressBar;
var ListRenderer = require('web.ListRenderer');
var field_registry = require('web.field_registry');

var core = require('web.core');
var qweb = core.qweb;
var _t = core._t;

var AbstractGroupedOne2ManyRenderer = ListRenderer.extend({
    /**
     * This abstract renderer is use to render a one2many field in a form view.
     * The records in the one2many field are displayed grouped by a specific field.
     *
     * A concrete renderer can/should set:
     *  - groupBy: field to group records
     *  - dataRowTemplate: template to render a record's data
     *  - groupTitleTemplate (optional): template to render the header row of a group
     *  - addLineButtonTemplate (optional): template to render the 'Add a line' button at the end of each group (edit mode only)
     **/

    groupBy: '', // Field: records are grouped based on this field
    groupTitleTemplate: 'hr_default_group_row', // Template used to render the title row of a group
    dataRowTemplate: '',    // Template used to render a record
    addLineButtonTemplate: 'group_add_item',

    /**
     * Don't freeze the columns because as the header is empty, the algorithm
     * won't work.
     *
     * @override
     * @private
     */
    _freezeColumnWidths: function () {},

     /**
     * Renders a empty header
     *
     * @override
     * @private
     */
    _renderHeader: function () {
        return $('<thead/>');
    },

     /**
     * Renders a empty footer
     *
     * @override
     * @private
     */
    _renderFooter: function () {
        return $('<tfoot/>');
    },

    /**
     * @override
     * @private
     */
    _renderGroupRow: function (display_name) {
        return qweb.render(this.groupTitleTemplate, {display_name: display_name});
    },

    /**
     * This method is meant to be overriten by concrete renderers and
     * is called each time a row is rendered.
     * It is a hook to format record's data before it's given to the qweb template.
     *
     * @private
    */
    _formatData: function (data) {
        return data;
    },

    _renderRow: function (record, isLast) {
        return $(qweb.render(this.dataRowTemplate, {
            id: record.id,
            data: this._formatData(record.data),
            is_last: isLast,
        }));
    },

    /**
     * This method is meant to be overridden by concrete renderers.
     * Returns a context used for the 'Add a line' button.
     * It's useful to set default values.
     * An 'Add a line' button is added after each group of records.
     * The group passed as parameters allow to set a different context based on the group.
     * If no records exist, group is undefined.
     *
     * @private
    */
    _getCreateLineContext: function (group) {
        return {};
    },

    _renderTrashIcon: function() {
        return qweb.render('hr_trash_button');
    },

    _renderAddItemButton: function (group) {
        return qweb.render(this.addLineButtonTemplate, {
            context: JSON.stringify(this._getCreateLineContext(group)),
        });
    },

    _renderBody: function () {
        var self = this;

        var grouped_by = _.groupBy(this.state.data, function (record) {
            return record.data[self.groupBy].res_id;
        });

        var groupTitle;
        var $body = $('<tbody>');
        for (var key in grouped_by) {
            var group = grouped_by[key];
            if (key === 'undefined') {
                groupTitle = _t("Other");
            } else {
                groupTitle = group[0].data[self.groupBy].data.display_name;
            }
            var $title_row = $(self._renderGroupRow(groupTitle));
            $body.append($title_row);

            // Render each rows
            group.forEach(function (record, index) {
                var isLast = (index + 1 === group.length);
                var $row = self._renderRow(record, isLast);
                if (self.addTrashIcon) $row.append(self._renderTrashIcon());
                $body.append($row);
            });

            if (self.addCreateLine) {
                $title_row.find('.o_group_name').append(self._renderAddItemButton(group));
            }
        }

        if ($body.is(':empty') && self.addCreateLine) {
            $body.append(this._renderAddItemButton());
        }
        return $body;
    },

    /**
     * This function enables the top right menu on list views to hide/show fields or, when studio is installed,
     *  edit the view's content.
     * For this widget we do not want it at all.
     *
     * @override
     * @private
     * @returns {boolean}
     */
     _shouldRenderOptionalColumnsDropdown: function () {
         return false;
     }
});

var ResumeLineRenderer = AbstractGroupedOne2ManyRenderer.extend({

    groupBy: 'line_type_id',
    groupTitleTemplate: 'hr_resume_group_row',
    dataRowTemplate: 'hr_resume_data_row',

    _formatData: function (data) {
        var dateFormat = time.getLangDateFormat();
        var date_start = data.date_start && data.date_start.format(dateFormat) || "";
        var date_end = data.date_end && data.date_end.format(dateFormat) || _t("Current");
        return _.extend(data, {
            date_start: date_start,
            date_end: date_end,
        });
    },

    _getCreateLineContext: function (group) {
        var ctx = this._super(group);
        return group ? _.extend({default_line_type_id: group[0].data[this.groupBy] && group[0].data[this.groupBy].data.id || ""}, ctx) : ctx;
    },

    _render: function () {
        var self = this;
        return this._super().then(function () {
            self.$el.find('table').removeClass('table-striped o_list_table_ungrouped');
            self.$el.find('table').addClass('o_resume_table table-borderless');
        });
    },
});


var SkillsRenderer = AbstractGroupedOne2ManyRenderer.extend({

    groupBy: 'skill_type_id',
    dataRowTemplate: 'hr_skill_data_row',

    _renderRow: function (record) {
        var $row = this._super(record);
        // Add progress bar widget at the end of rows
        var $td = $('<td/>', {class: 'o_data_cell o_skill_cell'});
        var progress = new FieldProgressBar(this, 'level_progress', record, {
            current_value: record.data.level_progress,
            attrs: this.arch.attrs,
        });
        progress.appendTo($td);
        return $row.append($td);
    },

    _getCreateLineContext: function (group) {
        var ctx = this._super(group);
        return group ? _.extend({ default_skill_type_id: group[0].data[this.groupBy].data.id }, ctx) : ctx;
    },

    _render: function () {
        var self = this;
        return this._super().then(function () {
            self.$el.find('table').toggleClass('table-striped');
        });
    },
});


var FieldResume = FieldOne2Many.extend({

    /**
     * @override
     * @private
     */
    _getRenderer: function () {
        return ResumeLineRenderer;
    },
});

var FieldSkills = FieldOne2Many.extend({

    /**
     * @override
     * @private
     */
    _getRenderer: function () {
        return SkillsRenderer;
    },
});

field_registry.add('hr_resume', FieldResume);
field_registry.add('hr_skills', FieldSkills);

return FieldResume;

});

```

## File: static\src\xml\resume_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="hr_resume_data_row">
    <tr class="o_data_row" t-attf-class="o_data_row #{is_last? 'o_data_row_last' : ''}" t-att-data-id="id">
        <t t-if="data.display_type === 'classic'">
            <td class="o_resume_timeline_cell position-relative pr-lg-2">
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
    <td class="o_list_record_remove pr-3">
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

    <div t-attf-class="o_field_x2many_list_row_add #{empty? 'd-block w-100' : 'd-inline pull-right'}">
        <div t-if="empty" class="o_resume_empty_helper o_horizontal_separator text-muted my-0">
            <em>Resumé empty</em>
        </div>
        <a href="#"
            role="button"
            t-attf-class="btn o-kanban-button-new #{empty? 'btn-primary mt-3' : 'btn-secondary btn-sm'}"
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
    <tr class="o_data_row" t-att-data-id="id">
        <td class="o_data_cell o_skill_cell w-100">
            <t t-esc="data.skill_id.data.display_name"/>
        </td>
        <td class="o_data_cell o_skill_cell pr-3">
            <t t-esc="data.skill_level_id.data.display_name"/>
        </td>
    </tr>
</t>

<t t-name="hr_default_group_row">
    <tr class="o_group_header o_group_has_content">
        <td class="o_group_name border-0 pr-2" colspan="99">
            <b t-esc="display_name"/>
        </td>
    </tr>
</t>


</templates>

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
                <field name="resume_line_ids" string="Resumé" filter_domain="['|', ('resume_line_ids.name', 'ilike', self), ('resume_line_ids.description', 'ilike', self)]"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_public_view_search" model="ir.ui.view">
        <field name="name">hr.employee.public.skill.search</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='company_id']" position="after">
                <field name="employee_skill_ids"/>
                <field name="resume_line_ids" string="Resumé" filter_domain="['|', ('resume_line_ids.name', 'ilike', self), ('resume_line_ids.description', 'ilike', self)]"/>
            </xpath>
        </field>
    </record>

    <record id="resume_line_view_form" model="ir.ui.view">
        <field name="name">hr.resume.line.form</field>
        <field name="model">hr.resume.line</field>
        <field name="arch" type="xml">
            <form string="Resumé">
                <div class="oe_title">
                    <label for="name" string="Title"/>
                    <h1>
                        <field name="name" placeholder="e.g. Odoo Inc." required="True"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="line_type_id"/>
                        <field name="display_type" required="1"/>
                    </group>
                    <group>
                        <field name="date_start" required="True"/>
                        <field name="date_end"/>
                    </group>
                </group>
                <field name="description" placeholder="Description"/>
            </form>
        </field>
    </record>

    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.resume</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='public']" position="before">
                <page name="skills_resume" string="Resumé">
                    <div class="row">
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_resume col-lg-7 d-flex">
                            <!-- This field uses a custom tree view rendered by the 'hr_resume' widget.
                                Adding fields in the tree arch below makes them accessible to the widget
                            -->
                            <field mode="tree" nolabel="1" name="resume_line_ids" widget="hr_resume">
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
                            <separator string="Skills"/>
                            <field mode="tree" nolabel="1" name="employee_skill_ids"  widget="hr_skills">
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
                <page name="skills_resume" string="Resumé">
                    <div class="row">
                        <div class="o_hr_skills_group o_group_resume col-lg-7 d-flex">
                            <!-- This field uses a custom tree view rendered by the 'hr_resume' widget.
                                Adding fields in the tree arch below makes them accessible to the widget
                            -->
                            <field mode="tree" nolabel="1" name="resume_line_ids" widget="hr_resume" attrs="{'readonly': True}" options="{'no_open': True}">
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
                            <field mode="tree" nolabel="1" name="employee_skill_ids"  widget="hr_skills" attrs="{'readonly': True}"  options="{'no_open': True}">
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
                <page name="skills_resume" string="Resumé">
                    <div class="row">
                        <div class="o_hr_skills_group o_group_resume col-lg-7 d-flex">
                            <!-- This field uses a custom tree view rendered by the 'hr_resume' widget.
                                Adding fields in the tree arch below makes them accessible to the widget
                            -->
                            <field mode="tree" nolabel="1" name="resume_line_ids" widget="hr_resume" attrs="{'readonly': [('can_edit', '=', False)]}">
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
                            <field mode="tree" nolabel="1" name="employee_skill_ids"  widget="hr_skills" attrs="{'readonly': [('can_edit', '=', False)]}">
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
            <tree name="Resumé Line Types" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="hr_resume_type_action" model="ir.actions.act_window">
        <field name="name">Resumé Line Types</field>
        <field name="res_model">hr.resume.line.type</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem
            id="menu_human_resources_configuration_resume"
            name="Resumé"
            parent="hr.menu_human_resources_configuration"
            sequence="4"
            groups="base.group_no_one"/>

    <menuitem
        id="hr_resume_line_type_menu"
        name="Types"
        action="hr_resume_type_action"
        parent="hr_skills.menu_human_resources_configuration_resume"
        sequence="3"
        groups="base.group_no_one"/>

    <!-- Skills -->

    <record id="employee_skill_level_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.level.tree</field>
        <field name="model">hr.skill.level</field>
        <field name="arch" type="xml">
            <tree string="Skill Levels">
                <field name="name"/>
                <field name="level_progress" widget="progressbar"/>
            </tree>
        </field>
    </record>

    <record id="employee_skill_view_tree" model="ir.ui.view">
        <field name="name">hr.skill.tree</field>
        <field name="model">hr.skill</field>
        <field name="arch" type="xml">
            <tree string="Skill Levels">
                <field name="name"/>
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
            <form string="Skills">
                <sheet>
                    <group>
                        <group>
                            <field name="skill_type_id"/>
                            <field
                                name="skill_id"
                                domain="[('skill_type_id', '=', skill_type_id)]"
                                options="{'no_create_edit':True}"/>
                        </group>
                        <group>
                            <field name="skill_level_id" domain="[('skill_type_id', '=', skill_type_id)]"/>
                            <field name="level_progress" widget="progressbar"/>
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
                    <field name="name"/>
                </sheet>
            </form>
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
                        <field name="skill_ids" nolabel="1" context="{'default_skill_type_id': id}">
                            <tree editable="bottom">
                                <field name="name"/>
                            </tree>
                        </field>
                    </group>
                    <group string="Levels">
                        <field name="skill_level_ids" nolabel="1" context="{'default_skill_type_id': id}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_skill_type_action" model="ir.actions.act_window">
        <field name="name">Skill Types</field>
        <field name="res_model">hr.skill.type</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem
        id="hr_skill_type_menu"
        name="Skills"
        action="hr_skill_type_action"
        parent="hr.menu_human_resources_configuration_employee"
        sequence="3"
        groups="base.group_no_one"/>
</odoo>

```

