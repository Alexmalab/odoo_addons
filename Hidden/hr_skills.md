# Odoo Module: hr_skills

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
    'name': 'Skills Management',
    'category': 'Hidden',
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
        'views/hr_templates.xml',
        'data/hr_resume_data.xml',
    ],
    'demo': [
        'data/hr_resume_demo.xml',
        'data/hr.employee.skill.csv',
        'data/hr.resume.line.csv',
    ],
    'qweb': [
        'static/src/xml/resume_templates.xml',
        'static/src/xml/skills_templates.xml',
    ],
    'installable': True,
    'application': True,
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
employee_resume_admin_greeneorr,hr.employee_admin,Greene-Orr,2008-01-21,2006-09-21,resume_type_experience,Magazine journalist
employee_resume_admin_white_inc,hr.employee_admin,White Inc,2007-05-24,2006-12-22,resume_type_experience,"Designer, television/film set"
employee_resume_admin_lewisbailey,hr.employee_admin,Lewis-Bailey,2009-04-22,2006-12-22,resume_type_experience,Civil Service fast streamer
employee_resume_al_bathurst_west_public_school,hr.employee_al,Bathurst West Public School,1997-05-06,1998-03-18,resume_type_education,
employee_resume_al_jones_ltd,hr.employee_al,Jones Ltd,1998-02-05,1997-08-05,resume_type_experience,Energy manager
employee_resume_al_garcia_smith_and_king,hr.employee_al,"Garcia, Smith and King",1998-09-05,,resume_type_experience,Medical illustrator
employee_resume_mit_seymour_p12_college,hr.employee_mit,Seymour P-12 College,2013-08-11,2015-07-01,resume_type_education,
employee_resume_mit_darlington_primary_school,hr.employee_mit,Darlington Primary School,2012-08-08,2013-05-05,resume_type_education,
employee_resume_mit_sutherland_dianella_primary_school,hr.employee_mit,Sutherland Dianella Primary School,2010-03-25,2012-06-27,resume_type_education,
employee_resume_mit_burns_lester_and_cuevas,hr.employee_mit,"Burns, Lester and Cuevas",2012-04-24,2010-05-26,resume_type_experience,Police officer
employee_resume_mit_hill_group,hr.employee_mit,Hill Group,2012-05-25,2010-04-25,resume_type_experience,Glass blower/designer
employee_resume_mit_parker_roberson_and_acosta,hr.employee_mit,"Parker, Roberson and Acosta",2013-03-25,2010-04-25,resume_type_experience,Science writer
employee_resume_mit_robinson_crawford_and_norman,hr.employee_mit,"Robinson, Crawford and Norman",2011-10-23,2010-06-23,resume_type_experience,Psychiatric nurse
employee_resume_niv_kialla_west_primary_school,hr.employee_niv,Kialla West Primary School,2016-08-23,2017-03-26,resume_type_education,
employee_resume_niv_arroyo_ltd,hr.employee_niv,Arroyo Ltd,2017-05-23,2016-09-23,resume_type_experience,Insurance risk surveyor
employee_resume_stw_northern_bay_p12_college,hr.employee_stw,Northern Bay P-12 College,2016-03-25,2017-06-03,resume_type_education,
employee_resume_stw_whitsunday_anglican_school,hr.employee_stw,Whitsunday Anglican School,2014-05-06,2016-03-21,resume_type_education,
employee_resume_stw_tyndale_christian_school,hr.employee_stw,Tyndale Christian School,2011-03-04,2014-03-28,resume_type_education,
employee_resume_stw_green_ltd,hr.employee_stw,Green Ltd,2013-01-31,2011-04-01,resume_type_experience,Arboriculturist
employee_resume_stw_lynchhodges,hr.employee_stw,Lynch-Hodges,2012-12-01,,resume_type_experience,Publishing rights manager
employee_resume_stw_finley_rowe_and_adams,hr.employee_stw,"Finley, Rowe and Adams",2012-12-31,,resume_type_experience,"Copywriter, advertising"
employee_resume_chs_avoca_primary_school,hr.employee_chs,Avoca Primary School,1996-03-04,1996-10-26,resume_type_education,
employee_resume_chs_boyd_wilson_and_moore,hr.employee_chs,"Boyd, Wilson and Moore",1997-11-03,,resume_type_experience,Medical physicist
employee_resume_chs_freeman_williams_and_berger,hr.employee_chs,"Freeman, Williams and Berger",1997-02-02,1996-06-04,resume_type_experience,Human resources officer
employee_resume_chs_hanson_roach_and_jordan,hr.employee_chs,"Hanson, Roach and Jordan",1997-03-04,1996-03-04,resume_type_experience,Geographical information systems officer
employee_resume_chs_davis_plc,hr.employee_chs,Davis PLC,1996-10-05,1996-06-04,resume_type_experience,"Secretary, company"
employee_resume_qdp_parke_state_school,hr.employee_qdp,Parke State School,1997-06-26,1999-03-17,resume_type_education,
employee_resume_qdp_evans_cooper_and_white,hr.employee_qdp,"Evans, Cooper and White",1999-04-26,,resume_type_experience,"Therapist, speech and language"
employee_resume_qdp_rivera_shaw_and_hughes,hr.employee_qdp,"Rivera, Shaw and Hughes",1998-11-26,,resume_type_experience,Landscape architect
employee_resume_qdp_phillips_jones_and_brown,hr.employee_qdp,"Phillips, Jones and Brown",1999-12-27,1997-07-27,resume_type_experience,"Teacher, special educational needs"
employee_resume_qdp_hughes_parker_and_barber,hr.employee_qdp,"Hughes, Parker and Barber",1999-02-24,1997-07-27,resume_type_experience,"Engineer, drilling"
employee_resume_fme_st_michaels_primary_school,hr.employee_fme,St Michael's Primary School,2006-12-22,2009-01-23,resume_type_education,
employee_resume_fme_wodonga_primary_school,hr.employee_fme,Wodonga Primary School,2006-02-21,2006-09-27,resume_type_education,
employee_resume_fme_leinster_school,hr.employee_fme,Leinster School,2003-10-14,2005-11-09,resume_type_education,
employee_resume_fme_russellwebster,hr.employee_fme,Russell-Webster,2006-05-14,2003-11-14,resume_type_experience,"Biochemist, clinical"
employee_resume_fme_lewis_group,hr.employee_fme,Lewis Group,2004-07-14,2003-11-14,resume_type_experience,Sports development officer
employee_resume_fme_johnson_shaw_and_carroll,hr.employee_fme,"Johnson, Shaw and Carroll",2004-07-14,2003-11-14,resume_type_experience,"Engineer, mining"
employee_resume_fpi_st_raphaels_primary_school,hr.employee_fpi,St Raphael's Primary School,2006-08-13,2008-09-30,resume_type_education,
employee_resume_fpi_woodridge_state_school,hr.employee_fpi,Woodridge State School,2005-12-28,2006-08-09,resume_type_education,
employee_resume_fpi_our_lady_star_of_the_sea_school,hr.employee_fpi,Our Lady Star of the Sea School,2003-06-07,2005-12-26,resume_type_education,
employee_resume_fpi_chavez_group,hr.employee_fpi,Chavez Group,2004-04-06,,resume_type_experience,Mental health nurse
employee_resume_fpi_hubbarddean,hr.employee_fpi,Hubbard-Dean,2005-05-07,2003-08-08,resume_type_experience,Conference centre manager
employee_resume_jth_narellan_public_school,hr.employee_jth,Narellan Public School,2004-11-02,2007-10-09,resume_type_education,
employee_resume_jth_wilkinson_plc,hr.employee_jth,Wilkinson PLC,2006-02-01,,resume_type_experience,Architectural technologist
employee_resume_jth_simmonswilcox,hr.employee_jth,Simmons-Wilcox,2005-09-03,2004-12-03,resume_type_experience,IT sales professional
employee_resume_jth_goodman_inc,hr.employee_jth,Goodman Inc,2007-06-03,,resume_type_experience,Analytical chemist
employee_resume_ngh_armidale_city_public_school,hr.employee_ngh,Armidale City Public School,1994-11-19,1997-02-18,resume_type_education,
employee_resume_ngh_craigmore_south_junior_primary_school,hr.employee_ngh,Craigmore South Junior Primary School,1993-05-11,1994-08-24,resume_type_education,
employee_resume_ngh_stanleymendez,hr.employee_ngh,Stanley-Mendez,1993-12-12,1993-05-11,resume_type_experience,Glass blower/designer
employee_resume_ngh_jackson_schwartz_and_aguirre,hr.employee_ngh,"Jackson, Schwartz and Aguirre",1995-04-12,1993-07-12,resume_type_experience,Analytical chemist
employee_resume_vad_wycheproof_p12_college,hr.employee_vad,Wycheproof P-12 College,1999-06-02,2000-12-01,resume_type_education,
employee_resume_vad_christian_outreach_college,hr.employee_vad,Christian Outreach College,1996-08-02,1999-02-01,resume_type_education,
employee_resume_vad_thomas_chirnside_primary_school,hr.employee_vad,Thomas Chirnside Primary School,1995-05-05,1996-07-27,resume_type_education,
employee_resume_vad_loganmartin,hr.employee_vad,Logan-Martin,1996-04-04,1995-07-05,resume_type_experience,Petroleum engineer
employee_resume_vad_gallegos_little_and_walters,hr.employee_vad,"Gallegos, Little and Walters",1996-12-02,1995-08-05,resume_type_experience,"Lecturer, higher education"
employee_resume_han_king_island_district_high_school,hr.employee_han,King Island District High School,2014-08-17,2016-05-06,resume_type_education,
employee_resume_han_elphinstone_primary_school,hr.employee_han,Elphinstone Primary School,2013-07-18,2014-07-29,resume_type_education,
employee_resume_han_william_light_r12_school,hr.employee_han,William Light R-12 School,2011-01-29,2013-03-30,resume_type_education,
employee_resume_han_davis_plc,hr.employee_han,Davis PLC,2012-10-30,2011-01-29,resume_type_experience,"Engineer, production"
employee_resume_han_perezmorgan,hr.employee_han,Perez-Morgan,2013-05-01,,resume_type_experience,Geoscientist
employee_resume_jve_ellinbank_primary_school,hr.employee_jve,Ellinbank Primary School,2003-02-16,2004-05-18,resume_type_education,
employee_resume_jve_talbot_primary_school,hr.employee_jve,Talbot Primary School,2002-07-01,2003-02-09,resume_type_education,
employee_resume_jve_saundersadkins,hr.employee_jve,Saunders-Adkins,2003-07-01,,resume_type_experience,Jewellery designer
employee_resume_jve_davis_and_sons,hr.employee_jve,Davis and Sons,2004-12-28,2002-07-01,resume_type_experience,Health physicist
employee_resume_jve_arnoldcohen,hr.employee_jve,Arnold-Cohen,2003-12-29,,resume_type_experience,Personnel officer
employee_resume_jep_lawson_public_school,hr.employee_jep,Lawson Public School,1998-06-07,2000-02-17,resume_type_education,
employee_resume_jep_trinity_college,hr.employee_jep,Trinity College,1995-08-21,1998-04-10,resume_type_education,
employee_resume_jep_woodend_primary_school,hr.employee_jep,Woodend Primary School,1992-11-22,1995-05-05,resume_type_education,
employee_resume_jep_mcneil_rodriguez_and_warren,hr.employee_jep,"Mcneil, Rodriguez and Warren",1994-11-22,1993-01-22,resume_type_experience,Sub
employee_resume_jep_davis_sanchez_and_miller,hr.employee_jep,"Davis, Sanchez and Miller",1995-05-25,1992-11-22,resume_type_experience,Customer service manager
employee_resume_jep_cole_ltd,hr.employee_jep,Cole Ltd,1994-02-22,,resume_type_experience,Fast food restaurant manager
employee_resume_jep_garcia_and_sons,hr.employee_jep,Garcia and Sons,1995-07-25,1992-12-23,resume_type_experience,Careers information officer
employee_resume_jod_umbakumba_school,hr.employee_jod,Umbakumba School,2009-11-08,2010-09-30,resume_type_education,
employee_resume_jod_wilson_ltd,hr.employee_jod,Wilson Ltd,2011-02-07,2010-01-08,resume_type_experience,Trade union research officer
employee_resume_jog_port_curtis_road_state_school,hr.employee_jog,Port Curtis Road State School,2005-04-10,2006-09-30,resume_type_education,
employee_resume_jog_claremont_college,hr.employee_jog,Claremont College,2004-01-12,2005-03-10,resume_type_education,
employee_resume_jog_mandurah_catholic_college,hr.employee_jog,Mandurah Catholic College,2003-02-08,2003-09-28,resume_type_education,
employee_resume_jog_douglas_thompson_and_conner,hr.employee_jog,"Douglas, Thompson and Conner",2004-01-10,,resume_type_experience,Music therapist
employee_resume_jog_allenkeller,hr.employee_jog,Allen-Keller,2005-07-09,2003-03-11,resume_type_experience,Lexicographer
employee_resume_jgo_tottenham_central_school,hr.employee_jgo,Tottenham Central School,2001-04-13,2002-09-06,resume_type_education,
employee_resume_jgo_galilee_catholic_school,hr.employee_jgo,Galilee Catholic School,2000-08-07,2001-02-16,resume_type_education,
employee_resume_jgo_martin_stanley_and_duncan,hr.employee_jgo,"Martin, Stanley and Duncan",2001-05-07,,resume_type_experience,IT technical support officer
employee_resume_jgo_fox_and_sons,hr.employee_jgo,Fox and Sons,2003-03-07,2000-10-05,resume_type_experience,Merchant navy officer
employee_resume_lur_holy_family_primary_school,hr.employee_lur,Holy Family Primary School,2009-07-16,2012-07-23,resume_type_education,
employee_resume_lur_lindenow_primary_school,hr.employee_lur,Lindenow Primary School,2007-07-08,2009-07-16,resume_type_education,
employee_resume_lur_narrogin_primary_school,hr.employee_lur,Narrogin Primary School,2005-12-14,2007-06-10,resume_type_education,
employee_resume_lur_ramirez_inc,hr.employee_lur,Ramirez Inc,2006-11-13,,resume_type_experience,Glass blower/designer
employee_resume_lur_whitebell,hr.employee_lur,White-Bell,2006-04-15,2006-01-13,resume_type_experience,Sports coach
employee_resume_hne_st_peters_parish_primary_school,hr.employee_hne,St Peter's Parish Primary School,2008-05-18,2008-11-17,resume_type_education,
employee_resume_hne_dandenong_north_primary_school,hr.employee_hne,Dandenong North Primary School,2005-07-20,2008-02-15,resume_type_education,
employee_resume_hne_nortonsilva,hr.employee_hne,Norton-Silva,2007-05-21,2005-09-19,resume_type_experience,"Horticulturist, commercial"

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
        ('date_check', "CHECK ((date_start <= date_end OR date_end = NULL))", "The start date must be anterior to the end date."),
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
    skill_type_id = fields.Many2one('hr.skill.type')


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
                raise ValidationError(_("The skill %s and skill type %s doesn't match") % (record.skill_id.name, record.skill_type_id.name))

    @api.constrains('skill_type_id', 'skill_level_id')
    def _check_skill_level(self):
        for record in self:
            if record.skill_level_id not in record.skill_type_id.skill_level_ids:
                raise ValidationError(_("The skill level %s is not valid for skill type: %s ") % (record.skill_level_id.name, record.skill_type_id.name))


class SkillLevel(models.Model):
    _name = 'hr.skill.level'
    _description = "Skill Level"
    _order = "level_progress desc"

    skill_type_id = fields.Many2one('hr.skill.type')
    name = fields.Char(required=True)
    level_progress = fields.Integer(string="Progress", help="Progress from zero knowledge (0%) to fully mastered (100%).")


class SkillType(models.Model):
    _name = 'hr.skill.type'
    _description = "Skill Type"

    name = fields.Char(required=True)
    skill_ids = fields.One2many('hr.skill', 'skill_type_id', string="Skills", ondelete='cascade')
    skill_level_ids = fields.One2many('hr.skill.level', 'skill_type_id', string="Levels", ondelete='cascade')

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

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        hr_skills_fields = [
            'resume_line_ids',
            'employee_skill_ids',
        ]
        init_res = super(User, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        type(self).SELF_READABLE_FIELDS = type(self).SELF_READABLE_FIELDS + hr_skills_fields
        type(self).SELF_WRITEABLE_FIELDS = type(self).SELF_WRITEABLE_FIELDS + hr_skills_fields
        return init_res

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
<odoo>

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#269396"/>
            <stop offset="100%" stop-color="#218689"/>
        </linearGradient>
        <path id="icon-d" d="M35.5,22.75 C39.6529297,22.75 43.0195312,26.1166016 43.0195312,30.2695312 C43.0195312,34.4224609 39.6529297,37.7890625 35.5,37.7890625 C31.3470703,37.7890625 27.9804687,34.4224609 27.9804687,30.2695312 C27.9804687,26.1166016 31.3470703,22.75 35.5,22.75 Z M43.6256055,38.3165755 L40.7623112,37.6007161 C37.2411654,40.1333659 32.9730794,39.5681836 30.2377604,37.6007161 L27.3744661,38.3165755 C25.0790039,38.8904232 23.46875,40.9527799 23.46875,43.3188542 L23.46875,47.671875 C23.46875,49.0957161 24.6230339,50.25 26.046875,50.25 L44.953125,50.25 C46.3769661,50.25 47.53125,49.0957161 47.53125,47.671875 L47.53125,43.3188542 C47.53125,40.9527799 45.9209961,38.8904232 43.6256055,38.3165755 Z M50.3958333,39.6510417 C53.1644531,39.6510417 55.4088542,37.4066406 55.4088542,34.6380208 C55.4088542,31.869401 53.1644531,29.625 50.3958333,29.625 C47.6272135,29.625 45.3828125,31.869401 45.3828125,34.6380208 C45.3828125,37.4066406 47.6272135,39.6510417 50.3958333,39.6510417 Z M20.6041667,39.6510417 C23.3727865,39.6510417 25.6171875,37.4066406 25.6171875,34.6380208 C25.6171875,31.869401 23.3727865,29.625 20.6041667,29.625 C17.8355469,29.625 15.5911458,31.869401 15.5911458,34.6380208 C15.5911458,37.4066406 17.8355469,39.6510417 20.6041667,39.6510417 Z M22.3229167,47.671875 L22.3229167,43.3188542 C22.3229167,42.1335612 22.6518424,41.0125781 23.2326367,40.0533008 C21.0850586,41.1074674 18.6968555,40.6769206 17.0959831,39.5255013 L15.1870964,40.0027409 C13.6568359,40.3852344 12.5833333,41.7602344 12.5833333,43.3375456 L12.5833333,46.2395833 C12.5833333,47.1888346 13.352832,47.9583333 14.3020833,47.9583333 L22.3350195,47.9583333 C22.3273388,47.8630362 22.3233016,47.7674804 22.3229167,47.671875 Z M55.8129036,40.0026693 L53.9040169,39.5254297 C51.9041797,40.9638802 49.5434049,40.902793 47.7604883,40.0423437 C48.3454362,41.004056 48.6770833,42.1289779 48.6770833,43.3188542 L48.6770833,47.671875 C48.6770833,47.7683398 48.6722135,47.8636589 48.6649805,47.9583333 L56.6979167,47.9583333 C57.647168,47.9583333 58.4166667,47.1888346 58.4166667,46.2395833 L58.4166667,43.3375456 C58.4166667,41.7602344 57.3431641,40.3852344 55.8129036,40.0026693 Z"/>
        <path id="icon-e" d="M35.5,20.75 C39.6529297,20.75 43.0195312,24.1166016 43.0195312,28.2695312 C43.0195312,32.4224609 39.6529297,35.7890625 35.5,35.7890625 C31.3470703,35.7890625 27.9804687,32.4224609 27.9804687,28.2695312 C27.9804687,24.1166016 31.3470703,20.75 35.5,20.75 Z M43.6256055,36.3165755 L40.7623112,35.6007161 C37.2411654,38.1333659 32.9730794,37.5681836 30.2377604,35.6007161 L27.3744661,36.3165755 C25.0790039,36.8904232 23.46875,38.9527799 23.46875,41.3188542 L23.46875,45.671875 C23.46875,47.0957161 24.6230339,48.25 26.046875,48.25 L44.953125,48.25 C46.3769661,48.25 47.53125,47.0957161 47.53125,45.671875 L47.53125,41.3188542 C47.53125,38.9527799 45.9209961,36.8904232 43.6256055,36.3165755 Z M50.3958333,37.6510417 C53.1644531,37.6510417 55.4088542,35.4066406 55.4088542,32.6380208 C55.4088542,29.869401 53.1644531,27.625 50.3958333,27.625 C47.6272135,27.625 45.3828125,29.869401 45.3828125,32.6380208 C45.3828125,35.4066406 47.6272135,37.6510417 50.3958333,37.6510417 Z M20.6041667,37.6510417 C23.3727865,37.6510417 25.6171875,35.4066406 25.6171875,32.6380208 C25.6171875,29.869401 23.3727865,27.625 20.6041667,27.625 C17.8355469,27.625 15.5911458,29.869401 15.5911458,32.6380208 C15.5911458,35.4066406 17.8355469,37.6510417 20.6041667,37.6510417 Z M22.3229167,45.671875 L22.3229167,41.3188542 C22.3229167,40.1335612 22.6518424,39.0125781 23.2326367,38.0533008 C21.0850586,39.1074674 18.6968555,38.6769206 17.0959831,37.5255013 L15.1870964,38.0027409 C13.6568359,38.3852344 12.5833333,39.7602344 12.5833333,41.3375456 L12.5833333,44.2395833 C12.5833333,45.1888346 13.352832,45.9583333 14.3020833,45.9583333 L22.3350195,45.9583333 C22.3273388,45.8630362 22.3233016,45.7674804 22.3229167,45.671875 Z M55.8129036,38.0026693 L53.9040169,37.5254297 C51.9041797,38.9638802 49.5434049,38.902793 47.7604883,38.0423437 C48.3454362,39.004056 48.6770833,40.1289779 48.6770833,41.3188542 L48.6770833,45.671875 C48.6770833,45.7683398 48.6722135,45.8636589 48.6649805,45.9583333 L56.6979167,45.9583333 C57.647168,45.9583333 58.4166667,45.1888346 58.4166667,44.2395833 L58.4166667,41.3375456 C58.4166667,39.7602344 57.3431641,38.3852344 55.8129036,38.0026693 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M44,47 L4,47 C2,47 -7.10542736e-15,46.8509317 0,42.826087 L1.81527147e-16,22.6291049 L17.2090667,6.04664397 L19.583071,9.5209307 L30.2767729,0.11143939 L40.9146315,10.270152 L46.6446282,6.41116033 L55.3045682,10.7749724 L52.3812234,16.1277957 L58.2417324,21.9036543 L44,47 Z" opacity=".324" transform="translate(0 23)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
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
        <button name="delete" arial-label="Delete row" class="btn btn-secondary">
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

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="hr_skills_assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/hr_skills/static/src/css/hr_skills.scss"/>
            <script type="text/javascript" src="/hr_skills/static/src/js/resume_widget.js"></script>
        </xpath>
    </template>
    <template id="qunit_suite" name="hr_skills_qunit_suite" inherit_id="web.qunit_suite">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/hr_skills/static/tests/widget_tests.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="resume_line_view_search" model="ir.ui.view">
        <field name="name">hr.resume.search</field>
        <field name="model">hr.resume.line</field>
        <field name="arch" type="xml">
            <search string="Résume">
                <filter string="Resumé line type" name="group_by_resume_line_type" context="{'group_by':'line_type'}"/>
            </search>
        </field>
    </record>

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
                    <label for="name" class="oe_edit_only"/>
                    <h1>
                        <field name="name" placeholder="Title" required="True"/>
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
                <page name="public" string="Resumé">
                    <div class="row">
                        <div class="o_hr_skills_group o_group_resume col-lg-7 d-flex">
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
                        <div class="o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
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
                <page name="public" string="Resumé">
                    <div class="row">
                        <div class="o_hr_skills_group o_group_resume col-lg-7 d-flex">
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
                        <div class="o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
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

    <record id="res_users_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.form.inherit.hr.skills</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile" />
        <field name="arch" type="xml">
            <xpath expr="//page[@name='public']" position="before">
                <page name="public" string="Resumé">
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
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="Skill Type" required="True"/>
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

