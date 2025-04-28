# Odoo Module: phone_validation

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import lib
from . import tools
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Phone Numbers Validation',
    'version': '2.1',
    'summary': 'Validate and format phone numbers',
    'sequence': '9999',
    'category': 'Hidden',
    'description': """
Phone Numbers Validation
========================

This module adds the feature of validation and formatting phone numbers
according to a destination country.

It also adds phone blacklist management through a specific model storing
blacklisted phone numbers.

It adds mail.thread.phone mixin that handles sanitation and blacklist of
records numbers. """,
    'data': [
        'security/ir.model.access.csv',
        'views/phone_blacklist_views.xml',
        'wizard/phone_blacklist_remove_view.xml',
    ],
    'depends': [
        'base',
        'mail',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: lib\phonemetadata.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

try:
    # import for usage in phonenumbers_patch/region_*.py files
    from phonenumbers.phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata # pylint: disable=unused-import
except ImportError:
    pass

```

## File: lib\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phonenumbers_patch

```

## File: lib\phonenumbers_patch\region_BR.py

```python
"""Auto-generated file, do not edit by hand. BR metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_BR = PhoneMetadata(id='BR', country_code=55, international_prefix='00(?:1[245]|2[1-35]|31|4[13]|[56]5|99)',
    general_desc=PhoneNumberDesc(national_number_pattern='(?:[1-46-9]\\d\\d|5(?:[0-46-9]\\d|5[0-46-9]))\\d{8}|[1-9]\\d{9}|[3589]\\d{8}|[34]\\d{7}', possible_length=(8, 9, 10, 11)),
    fixed_line=PhoneNumberDesc(national_number_pattern='(?:[14689][1-9]|2[12478]|3[1-578]|5[13-5]|7[13-579])[2-5]\\d{7}', example_number='1123456789', possible_length=(10,), possible_length_local_only=(8,)),
    mobile=PhoneNumberDesc(national_number_pattern='(?:[14689][1-9]|2[12478]|3[1-578]|5[13-5]|7[13-579])(?:7|9\\d)\\d{7}', example_number='11961234567', possible_length=(10, 11), possible_length_local_only=(8, 9)),
    toll_free=PhoneNumberDesc(national_number_pattern='800\\d{6,7}', example_number='800123456', possible_length=(9, 10)),
    premium_rate=PhoneNumberDesc(national_number_pattern='300\\d{6}|[59]00\\d{6,7}', example_number='300123456', possible_length=(9, 10)),
    shared_cost=PhoneNumberDesc(national_number_pattern='(?:30[03]\\d{3}|4(?:0(?:0\\d|20)|370))\\d{4}|300\\d{5}', example_number='40041234', possible_length=(8, 10)),
    no_international_dialling=PhoneNumberDesc(national_number_pattern='30(?:0\\d{5,7}|3\\d{7})|40(?:0\\d|20)\\d{4}|800\\d{6,7}', possible_length=(8, 9, 10)),
    national_prefix='0',
    national_prefix_for_parsing='(?:0|90)(?:(1[245]|2[1-35]|31|4[13]|[56]5|99)(\\d{10,11}))?',
    national_prefix_transform_rule='\\2',
    number_format=[NumberFormat(pattern='(\\d{3,6})', format='\\1', leading_digits_pattern=['1(?:1[25-8]|2[357-9]|3[02-68]|4[12568]|5|6[0-8]|8[015]|9[0-47-9])|321|610']),
        NumberFormat(pattern='(\\d{4})(\\d{4})', format='\\1-\\2', leading_digits_pattern=['300|4(?:0[02]|37)', '4(?:02|37)0|[34]00']),
        NumberFormat(pattern='(\\d{4})(\\d{4})', format='\\1-\\2', leading_digits_pattern=['[2-57]', '[2357]|4(?:[0-24-9]|3(?:[0-689]|7[1-9]))']),
        NumberFormat(pattern='(\\d{3})(\\d{2,3})(\\d{4})', format='\\1 \\2 \\3', leading_digits_pattern=['(?:[358]|90)0'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{5})(\\d{4})', format='\\1-\\2', leading_digits_pattern=['9']),
        NumberFormat(pattern='(\\d{2})(\\d{4})(\\d{4})', format='\\1 \\2-\\3', leading_digits_pattern=['(?:[14689][1-9]|2[12478]|3[1-578]|5[13-5]|7[13-579])[2-57]'], national_prefix_formatting_rule='(\\1)', domestic_carrier_code_formatting_rule='0 $CC (\\1)'),
        NumberFormat(pattern='(\\d{2})(\\d{5})(\\d{4})', format='\\1 \\2-\\3', leading_digits_pattern=['[16][1-9]|[2-57-9]'], national_prefix_formatting_rule='(\\1)', domestic_carrier_code_formatting_rule='0 $CC (\\1)')],
    intl_number_format=[NumberFormat(pattern='(\\d{4})(\\d{4})', format='\\1-\\2', leading_digits_pattern=['300|4(?:0[02]|37)', '4(?:02|37)0|[34]00']),
        NumberFormat(pattern='(\\d{3})(\\d{2,3})(\\d{4})', format='\\1 \\2 \\3', leading_digits_pattern=['(?:[358]|90)0']),
        NumberFormat(pattern='(\\d{2})(\\d{4})(\\d{4})', format='\\1 \\2-\\3', leading_digits_pattern=['(?:[14689][1-9]|2[12478]|3[1-578]|5[13-5]|7[13-579])[2-57]']),
        NumberFormat(pattern='(\\d{2})(\\d{5})(\\d{4})', format='\\1 \\2-\\3', leading_digits_pattern=['[16][1-9]|[2-57-9]'])],
    mobile_number_portable_region=True)

```

## File: lib\phonenumbers_patch\region_CI.py

```python
"""Auto-generated file, do not edit by hand. CI metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_CI = PhoneMetadata(id='CI', country_code=225, international_prefix='00',
    general_desc=PhoneNumberDesc(national_number_pattern='[02]\\d{9}', possible_length=(10,)),
    fixed_line=PhoneNumberDesc(national_number_pattern='2(?:[15]\\d{3}|7(?:2(?:0[23]|1[2357]|[23][45]|4[3-5])|3(?:06|1[69]|[2-6]7)))\\d{5}', example_number='2123456789', possible_length=(10,)),
    mobile=PhoneNumberDesc(national_number_pattern='0704[0-7]\\d{5}|0(?:[15]\\d\\d|7(?:0[0-37-9]|[4-9][7-9]))\\d{6}', example_number='0123456789', possible_length=(10,)),
    number_format=[NumberFormat(pattern='(\\d{2})(\\d{2})(\\d)(\\d{5})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['2']),
        NumberFormat(pattern='(\\d{2})(\\d{2})(\\d{2})(\\d{4})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['0'])])

```

## File: lib\phonenumbers_patch\region_CO.py

```python
"""Auto-generated file, do not edit by hand. CO metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_CO = PhoneMetadata(id='CO', country_code=57, international_prefix='00(?:4(?:[14]4|56)|[579])',
    general_desc=PhoneNumberDesc(national_number_pattern='(?:60\\d\\d|9101)\\d{6}|(?:1\\d|3)\\d{9}', possible_length=(10, 11), possible_length_local_only=(7,)),
    fixed_line=PhoneNumberDesc(national_number_pattern='601055(?:[0-4]\\d|50)\\d\\d|6010(?:[0-4]\\d|5[0-4])\\d{4}|60(?:[124-7][2-9]|8[1-9])\\d{6}', example_number='6012345678', possible_length=(10,), possible_length_local_only=(7,)),
    mobile=PhoneNumberDesc(national_number_pattern='333301[0-5]\\d{3}|3333(?:00|2[5-9]|[3-9]\\d)\\d{4}|(?:3(?:24[1-9]|3(?:00|3[0-24-9]))|9101)\\d{6}|3(?:0[0-5]|1\\d|2[0-3]|5[01]|70)\\d{7}', example_number='3211234567', possible_length=(10,)),
    toll_free=PhoneNumberDesc(national_number_pattern='1800\\d{7}', example_number='18001234567', possible_length=(11,)),
    premium_rate=PhoneNumberDesc(national_number_pattern='19(?:0[01]|4[78])\\d{7}', example_number='19001234567', possible_length=(11,)),
    national_prefix='0',
    national_prefix_for_parsing='0([3579]|4(?:[14]4|56))?',
    number_format=[NumberFormat(pattern='(\\d{3})(\\d{7})', format='\\1 \\2', leading_digits_pattern=['6'], national_prefix_formatting_rule='(\\1)', domestic_carrier_code_formatting_rule='0$CC \\1'),
        NumberFormat(pattern='(\\d{3})(\\d{7})', format='\\1 \\2', leading_digits_pattern=['3[0-357]|91'], domestic_carrier_code_formatting_rule='0$CC \\1'),
        NumberFormat(pattern='(\\d)(\\d{3})(\\d{7})', format='\\1-\\2-\\3', leading_digits_pattern=['1'], national_prefix_formatting_rule='0\\1')],
    intl_number_format=[NumberFormat(pattern='(\\d{3})(\\d{7})', format='\\1 \\2', leading_digits_pattern=['6']),
        NumberFormat(pattern='(\\d{3})(\\d{7})', format='\\1 \\2', leading_digits_pattern=['3[0-357]|91']),
        NumberFormat(pattern='(\\d)(\\d{3})(\\d{7})', format='\\1 \\2 \\3', leading_digits_pattern=['1'])],
    mobile_number_portable_region=True)

```

## File: lib\phonenumbers_patch\region_IL.py

```python
"""Auto-generated file, do not edit by hand. IL metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_IL = PhoneMetadata(id='IL', country_code=972, international_prefix='0(?:0|1[2-9])',
    general_desc=PhoneNumberDesc(national_number_pattern='1\\d{6}(?:\\d{3,5})?|[57]\\d{8}|[1-489]\\d{7}', possible_length=(7, 8, 9, 10, 11, 12)),
    fixed_line=PhoneNumberDesc(national_number_pattern='153\\d{8,9}|29[1-9]\\d{5}|(?:2[0-8]|[3489]\\d)\\d{6}', example_number='21234567', possible_length=(8, 11, 12), possible_length_local_only=(7,)),
    mobile=PhoneNumberDesc(national_number_pattern='55410\\d{4}|5(?:(?:[02][02-9]|[149][2-9]|[36]\\d|8[3-7])\\d|5(?:01|2\\d|3[0-3]|4[34]|5[0-25689]|6[6-8]|7[0-267]|8[7-9]|9[1-9]))\\d{5}', example_number='502345678', possible_length=(9,)),
    toll_free=PhoneNumberDesc(national_number_pattern='1(?:255|80[019]\\d{3})\\d{3}', example_number='1800123456', possible_length=(7, 10)),
    premium_rate=PhoneNumberDesc(national_number_pattern='1212\\d{4}|1(?:200|9(?:0[0-2]|19))\\d{6}', example_number='1919123456', possible_length=(8, 10)),
    shared_cost=PhoneNumberDesc(national_number_pattern='1700\\d{6}', example_number='1700123456', possible_length=(10,)),
    voip=PhoneNumberDesc(national_number_pattern='7(?:38(?:0\\d|5[0-259]|88)|8(?:33|55|77|81)\\d)\\d{4}|7(?:18|2[23]|3[237]|47|6[258]|7\\d|82|9[2-9])\\d{6}', example_number='771234567', possible_length=(9,)),
    uan=PhoneNumberDesc(national_number_pattern='1599\\d{6}', example_number='1599123456', possible_length=(10,)),
    voicemail=PhoneNumberDesc(national_number_pattern='151\\d{8,9}', example_number='15112340000', possible_length=(11, 12)),
    no_international_dialling=PhoneNumberDesc(national_number_pattern='1700\\d{6}', possible_length=(10,)),
    national_prefix='0',
    national_prefix_for_parsing='0',
    number_format=[NumberFormat(pattern='(\\d{4})(\\d{3})', format='\\1-\\2', leading_digits_pattern=['125']),
        NumberFormat(pattern='(\\d{4})(\\d{2})(\\d{2})', format='\\1-\\2-\\3', leading_digits_pattern=['121']),
        NumberFormat(pattern='(\\d)(\\d{3})(\\d{4})', format='\\1-\\2-\\3', leading_digits_pattern=['[2-489]'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{2})(\\d{3})(\\d{4})', format='\\1-\\2-\\3', leading_digits_pattern=['[57]'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{4})(\\d{3})(\\d{3})', format='\\1-\\2-\\3', leading_digits_pattern=['12']),
        NumberFormat(pattern='(\\d{4})(\\d{6})', format='\\1-\\2', leading_digits_pattern=['159']),
        NumberFormat(pattern='(\\d)(\\d{3})(\\d{3})(\\d{3})', format='\\1-\\2-\\3-\\4', leading_digits_pattern=['1[7-9]']),
        NumberFormat(pattern='(\\d{3})(\\d{1,2})(\\d{3})(\\d{4})', format='\\1-\\2 \\3-\\4', leading_digits_pattern=['15'])],
    mobile_number_portable_region=True)

```

## File: lib\phonenumbers_patch\region_KE.py

```python
"""Auto-generated file, do not edit by hand. KE metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_KE = PhoneMetadata(id='KE', country_code=254, international_prefix='000',
    general_desc=PhoneNumberDesc(national_number_pattern='(?:[17]\\d\\d|900)\\d{6}|(?:2|80)0\\d{6,7}|[4-6]\\d{6,8}', possible_length=(7, 8, 9, 10)),
    fixed_line=PhoneNumberDesc(national_number_pattern='(?:4[245]|5[1-79]|6[01457-9])\\d{5,7}|(?:4[136]|5[08]|62)\\d{7}|(?:[24]0|66)\\d{6,7}', example_number='202012345', possible_length=(7, 8, 9)),
    mobile=PhoneNumberDesc(national_number_pattern='(?:1(?:0[0-8]|1[0-7]|2[014]|30)|7\\d\\d)\\d{6}', example_number='712123456', possible_length=(9,)),
    toll_free=PhoneNumberDesc(national_number_pattern='800[02-8]\\d{5,6}', example_number='800223456', possible_length=(9, 10)),
    premium_rate=PhoneNumberDesc(national_number_pattern='900[02-9]\\d{5}', example_number='900223456', possible_length=(9,)),
    national_prefix='0',
    national_prefix_for_parsing='0',
    number_format=[NumberFormat(pattern='(\\d{2})(\\d{5,7})', format='\\1 \\2', leading_digits_pattern=['[24-6]'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{3})(\\d{6})', format='\\1 \\2', leading_digits_pattern=['[17]'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{3})(\\d{3})(\\d{3,4})', format='\\1 \\2 \\3', leading_digits_pattern=['[89]'], national_prefix_formatting_rule='0\\1')],
    mobile_number_portable_region=True)

```

## File: lib\phonenumbers_patch\region_MA.py

```python
"""Auto-generated file, do not edit by hand. MA metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_MA = PhoneMetadata(id='MA', country_code=212, international_prefix='00',
    general_desc=PhoneNumberDesc(national_number_pattern='[5-8]\\d{8}', possible_length=(9,)),
    fixed_line=PhoneNumberDesc(national_number_pattern='5(?:2(?:[0-25-79]\\d|3[1-578]|4[02-46-8]|8[0235-7])|3(?:[0-47]\\d|5[02-9]|6[02-8]|8[014-9]|9[3-9])|(?:4[067]|5[03])\\d)\\d{5}', example_number='520123456', possible_length=(9,)),
    mobile=PhoneNumberDesc(national_number_pattern='(?:6(?:[0-79]\\d|8[0-247-9])|7(?:[0167]\\d|2[0-4]|5[01]|8[0-3]))\\d{6}', example_number='650123456', possible_length=(9,)),
    toll_free=PhoneNumberDesc(national_number_pattern='80[0-7]\\d{6}', example_number='801234567', possible_length=(9,)),
    premium_rate=PhoneNumberDesc(national_number_pattern='89\\d{7}', example_number='891234567', possible_length=(9,)),
    voip=PhoneNumberDesc(national_number_pattern='(?:592(?:4[0-2]|93)|80[89]\\d\\d)\\d{4}', example_number='592401234', possible_length=(9,)),
    national_prefix='0',
    national_prefix_for_parsing='0',
    number_format=[NumberFormat(pattern='(\\d{3})(\\d{2})(\\d{2})(\\d{2})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['5[45]'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{4})(\\d{5})', format='\\1-\\2', leading_digits_pattern=['5(?:2[2-46-9]|3[3-9]|9)|8(?:0[89]|92)'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{2})(\\d{7})', format='\\1-\\2', leading_digits_pattern=['8'], national_prefix_formatting_rule='0\\1'),
        NumberFormat(pattern='(\\d{3})(\\d{6})', format='\\1-\\2', leading_digits_pattern=['[5-7]'], national_prefix_formatting_rule='0\\1')],
    main_country_for_code=True,
    mobile_number_portable_region=True)

```

## File: lib\phonenumbers_patch\region_MU.py

```python
"""Auto-generated file, do not edit by hand. MU metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_MU = PhoneMetadata(id='MU', country_code=230, international_prefix='0(?:0|[24-7]0|3[03])',
    general_desc=PhoneNumberDesc(national_number_pattern='(?:[57]|8\\d\\d)\\d{7}|[2-468]\\d{6}', possible_length=(7, 8, 10)),
    fixed_line=PhoneNumberDesc(national_number_pattern='(?:2(?:[0346-8]\\d|1[0-7])|4(?:[013568]\\d|2[4-8])|54(?:[3-5]\\d|71)|6\\d\\d|8(?:14|3[129]))\\d{4}', example_number='54480123', possible_length=(7, 8)),
    mobile=PhoneNumberDesc(national_number_pattern='5(?:4(?:2[1-389]|7[1-9])|87[15-8])\\d{4}|(?:5(?:2[5-9]|4[3-689]|[57]\\d|8[0-689]|9[0-8])|7(?:0[0-3]|3[013]))\\d{5}', example_number='52512345', possible_length=(8,)),
    toll_free=PhoneNumberDesc(national_number_pattern='802\\d{7}|80[0-2]\\d{4}', example_number='8001234', possible_length=(7, 10)),
    premium_rate=PhoneNumberDesc(national_number_pattern='30\\d{5}', example_number='3012345', possible_length=(7,)),
    voip=PhoneNumberDesc(national_number_pattern='3(?:20|9\\d)\\d{4}', example_number='3201234', possible_length=(7,)),
    preferred_international_prefix='020',
    number_format=[NumberFormat(pattern='(\\d{3})(\\d{4})', format='\\1 \\2', leading_digits_pattern=['[2-46]|8[013]']),
        NumberFormat(pattern='(\\d{4})(\\d{4})', format='\\1 \\2', leading_digits_pattern=['[57]']),
        NumberFormat(pattern='(\\d{5})(\\d{5})', format='\\1 \\2', leading_digits_pattern=['8'])])

```

## File: lib\phonenumbers_patch\region_PA.py

```python
"""Auto-generated file, do not edit by hand. PA metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_PA = PhoneMetadata(id='PA', country_code=507, international_prefix='00',
    general_desc=PhoneNumberDesc(national_number_pattern='(?:00800|8\\d{3})\\d{6}|[68]\\d{7}|[1-57-9]\\d{6}', possible_length=(7, 8, 10, 11)),
    fixed_line=PhoneNumberDesc(national_number_pattern='(?:1(?:0\\d|1[479]|2[37]|3[0137]|4[17]|5[05]|6[58]|7[0167]|8[258]|9[1389])|2(?:[0235-79]\\d|1[0-7]|4[013-9]|8[02-9])|3(?:[089]\\d|1[0-7]|2[0-5]|33|4[0-79]|5[05]|6[068]|7[0-8])|4(?:00|3[0-579]|4\\d|7[0-57-9])|5(?:[01]\\d|2[0-7]|[56]0|79)|7(?:0[09]|2[0-26-8]|3[03]|4[04]|5[05-9]|6[056]|7[0-24-9]|8[6-9]|90)|8(?:09|2[89]|3\\d|4[0-24-689]|5[014]|8[02])|9(?:0[5-9]|1[0135-8]|2[036-9]|3[35-79]|40|5[0457-9]|6[05-9]|7[04-9]|8[35-8]|9\\d))\\d{4}', example_number='2001234', possible_length=(7,)),
    mobile=PhoneNumberDesc(national_number_pattern='(?:1[16]1|21[89]|6\\d{3}|8(?:1[01]|7[23]))\\d{4}', example_number='61234567', possible_length=(7, 8)),
    toll_free=PhoneNumberDesc(national_number_pattern='800\\d{4,5}|(?:00800|800\\d)\\d{6}', example_number='8001234', possible_length=(7, 8, 10, 11)),
    premium_rate=PhoneNumberDesc(national_number_pattern='(?:8(?:22|55|60|7[78]|86)|9(?:00|81))\\d{4}', example_number='8601234', possible_length=(7,)),
    number_format=[NumberFormat(pattern='(\\d{3})(\\d{4})', format='\\1-\\2', leading_digits_pattern=['[1-57-9]']),
        NumberFormat(pattern='(\\d{4})(\\d{4})', format='\\1-\\2', leading_digits_pattern=['[68]']),
        NumberFormat(pattern='(\\d{3})(\\d{3})(\\d{4})', format='\\1 \\2 \\3', leading_digits_pattern=['8'])],
    mobile_number_portable_region=True)

```

## File: lib\phonenumbers_patch\region_SN.py

```python
"""Auto-generated file, do not edit by hand. SN metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_SN = PhoneMetadata(id='SN', country_code=221, international_prefix='00',
    general_desc=PhoneNumberDesc(national_number_pattern='(?:[378]\\d|93)\\d{7}', possible_length=(9,)),
    fixed_line=PhoneNumberDesc(national_number_pattern='3(?:0(?:1[0-2]|80)|282|3(?:8[1-9]|9[3-9])|611)\\d{5}', example_number='301012345', possible_length=(9,)),
    mobile=PhoneNumberDesc(national_number_pattern='7(?:(?:[06-8]\\d|21|90)\\d|5(?:01|[19]0|25|[38]3|[4-7]\\d))\\d{5}', example_number='701234567', possible_length=(9,)),
    toll_free=PhoneNumberDesc(national_number_pattern='800\\d{6}', example_number='800123456', possible_length=(9,)),
    premium_rate=PhoneNumberDesc(national_number_pattern='88[4689]\\d{6}', example_number='884123456', possible_length=(9,)),
    shared_cost=PhoneNumberDesc(national_number_pattern='81[02468]\\d{6}', example_number='810123456', possible_length=(9,)),
    voip=PhoneNumberDesc(national_number_pattern='(?:3(?:392|9[01]\\d)\\d|93(?:3[13]0|929))\\d{4}', example_number='933301234', possible_length=(9,)),
    number_format=[NumberFormat(pattern='(\\d{3})(\\d{2})(\\d{2})(\\d{2})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['8']),
        NumberFormat(pattern='(\\d{2})(\\d{3})(\\d{2})(\\d{2})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['[379]'])])

```

## File: lib\phonenumbers_patch\__init__.py

```python
# -*- coding: utf-8 -*-
# Copyright (C) 2009 The Libphonenumber Authors

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at

# http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://github.com/google/libphonenumber

from odoo.tools.parse_version import parse_version


def _local_load_region(code):
    __import__("region_%s" % code, globals(), locals(),
        fromlist=["PHONE_METADATA_%s" % code], level=1)


try:
    import phonenumbers
except ImportError:
    pass
else:
    # Over time, phone number formats change. The following monkey patches ensure phone number parsing stays up to date:
    # The most common type of patch occurs when the phonenumbers library is updated, but Odoo is still using an older version.
    # In such cases, we need to:
    # 1. Grab the newest metadata describing the phone number for a certain country.
    # 2. Create/update a metadata file in the current directory (e.g., files named like region_SN for the Senegal patch).
    # 3. Load the metadata file. Please add a reference to the upstream from which the update was taken.

    if parse_version('7.6.1') <= parse_version(phonenumbers.__version__) < parse_version('8.12.32'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.12.32/python/phonenumbers/data/region_CI.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('CI', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.13.40'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.13.40/python/phonenumbers/data/region_IL.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('IL', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.13.32'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.13.32/python/phonenumbers/data/region_MA.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('MA', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.12.13'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.13.31/python/phonenumbers/data/region_MU.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('MU', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.12.43'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.12.43/python/phonenumbers/data/region_PA.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('PA', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.12.29'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.12.57/python/phonenumbers/data/region_SN.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('SN', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.12.39'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.13.36/python/phonenumbers/data/region_CO.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('CO', _local_load_region)

    if parse_version(phonenumbers.__version__) < parse_version('8.13.31'):
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.13.40/python/phonenumbers/data/region_KE.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('KE', _local_load_region)

    # MONKEY PATCHING phonemetadata to fix Brazilian phonenumbers following 2016 changes
    def _hook_load_region(code):
        if parse_version(phonenumbers.__version__) < parse_version('8.13.39'):
            # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.13.39/python/phonenumbers/data/region_BR.py
            _local_load_region(code)
        else:
            phonenumbers.data._load_region(code)
        _region_metadata = phonenumbers.PhoneMetadata._region_metadata
        if 'BR' in _region_metadata:
            _region_metadata['BR'].intl_number_format.append(
                phonenumbers.phonemetadata.NumberFormat(
                    pattern='(\\d{2})(\\d{4})(\\d{4})',
                    format='\\1 9\\2-\\3',
                    leading_digits_pattern=['(?:[14689][1-9]|2[12478]|3[1-578]|5[13-5]|7[13-579][689])'],
                )
            )
    phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('BR', _hook_load_region)

```

## File: models\mail_thread_phone.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models, _
from odoo.addons.phone_validation.tools import phone_validation
from odoo.exceptions import AccessError, UserError
from odoo.osv import expression


class PhoneMixin(models.AbstractModel):
    """ Purpose of this mixin is to offer two services

      * compute a sanitized phone number based on ´´_sms_get_number_fields´´.
        It takes first sanitized value, trying each field returned by the
        method (see ``MailThread._sms_get_number_fields()´´ for more details
        about the usage of this method);
      * compute blacklist state of records. It is based on phone.blacklist
        model and give an easy-to-use field and API to manipulate blacklisted
        records;

    Main API methods

      * ``_phone_set_blacklisted``: set recordset as blacklisted;
      * ``_phone_reset_blacklisted``: reactivate recordset (even if not blacklisted
        this method can be called safely);
    """
    _name = 'mail.thread.phone'
    _description = 'Phone Blacklist Mixin'
    _inherit = ['mail.thread']
    _phone_search_min_length = 3

    phone_sanitized = fields.Char(
        string='Sanitized Number', compute="_compute_phone_sanitized", compute_sudo=True, store=True,
        help="Field used to store sanitized phone number. Helps speeding up searches and comparisons.")
    phone_sanitized_blacklisted = fields.Boolean(
        string='Phone Blacklisted', compute="_compute_blacklisted", compute_sudo=True, store=False,
        search="_search_phone_sanitized_blacklisted", groups="base.group_user",
        help="If the sanitized phone number is on the blacklist, the contact won't receive mass mailing sms anymore, from any list")
    phone_blacklisted = fields.Boolean(
        string='Blacklisted Phone is Phone', compute="_compute_blacklisted", compute_sudo=True, store=False, groups="base.group_user",
        help="Indicates if a blacklisted sanitized phone number is a phone number. Helps distinguish which number is blacklisted \
            when there is both a mobile and phone field in a model.")
    mobile_blacklisted = fields.Boolean(
        string='Blacklisted Phone Is Mobile', compute="_compute_blacklisted", compute_sudo=True, store=False, groups="base.group_user",
        help="Indicates if a blacklisted sanitized phone number is a mobile number. Helps distinguish which number is blacklisted \
            when there is both a mobile and phone field in a model.")
    phone_mobile_search = fields.Char("Phone/Mobile", store=False, search='_search_phone_mobile_search')

    def _search_phone_mobile_search(self, operator, value):
        value = value.strip() if isinstance(value, str) else value
        phone_fields = [
            fname for fname in self._phone_get_number_fields()
            if fname in self._fields and self._fields[fname].store
        ]
        if not phone_fields:
            raise UserError(_('Missing definition of phone fields.'))

        # search if phone/mobile is set or not
        if (value is True or not value) and operator in ('=', '!='):
            if value:
                # inverse the operator
                operator = '=' if operator == '!=' else '!='
            op = expression.AND if operator == '=' else expression.OR
            return op([[(phone_field, operator, False)] for phone_field in phone_fields])

        if self._phone_search_min_length and len(value) < self._phone_search_min_length:
            raise UserError(_('Please enter at least 3 characters when searching a Phone/Mobile number.'))

        pattern = r'[\s\\./\(\)\-]'
        sql_operator = {'=like': 'LIKE', '=ilike': 'ILIKE'}.get(operator, operator)

        if value.startswith('+') or value.startswith('00'):
            if operator in expression.NEGATIVE_TERM_OPERATORS:
                # searching on +32485112233 should also finds 0032485112233 (and vice versa)
                # we therefore remove it from input value and search for both of them in db
                where_str = ' AND '.join(
                    f"""model.{phone_field} IS NULL OR (
                            REGEXP_REPLACE(model.{phone_field}, %s, '', 'g') {sql_operator} %s OR
                            REGEXP_REPLACE(model.{phone_field}, %s, '', 'g') {sql_operator} %s
                    )"""
                    for phone_field in phone_fields
                )
            else:
                # searching on +32485112233 should also finds 0032485112233 (and vice versa)
                # we therefore remove it from input value and search for both of them in db
                where_str = ' OR '.join(
                    f"""model.{phone_field} IS NOT NULL AND (
                            REGEXP_REPLACE(model.{phone_field}, %s, '', 'g') {sql_operator} %s OR
                            REGEXP_REPLACE(model.{phone_field}, %s, '', 'g') {sql_operator} %s
                    )"""
                    for phone_field in phone_fields
                )
            query = f"SELECT model.id FROM {self._table} model WHERE {where_str};"

            term = re.sub(pattern, '', value[1 if value.startswith('+') else 2:])
            if operator not in ('=', '!='):  # for like operators
                term = f'{term}%'
            self._cr.execute(
                query, (pattern, '00' + term, pattern, '+' + term) * len(phone_fields)
            )
        else:
            if operator in expression.NEGATIVE_TERM_OPERATORS:
                where_str = ' AND '.join(
                    f"(model.{phone_field} IS NULL OR REGEXP_REPLACE(model.{phone_field}, %s, '', 'g') {sql_operator} %s)"
                    for phone_field in phone_fields
                )
            else:
                where_str = ' OR '.join(
                    f"(model.{phone_field} IS NOT NULL AND REGEXP_REPLACE(model.{phone_field}, %s, '', 'g') {sql_operator} %s)"
                    for phone_field in phone_fields
                )
            query = f"SELECT model.id FROM {self._table} model WHERE {where_str};"
            term = re.sub(pattern, '', value)
            if operator not in ('=', '!='):  # for like operators
                term = f'%{term}%'
            self._cr.execute(query, (pattern, term) * len(phone_fields))
        res = self._cr.fetchall()
        if not res:
            return [(0, '=', 1)]
        return [('id', 'in', [r[0] for r in res])]

    @api.depends(lambda self: self._phone_get_sanitize_triggers())
    def _compute_phone_sanitized(self):
        self._assert_phone_field()
        number_fields = self._phone_get_number_fields()
        for record in self:
            for fname in number_fields:
                sanitized = record.phone_get_sanitized_number(number_fname=fname)
                if sanitized:
                    break
            record.phone_sanitized = sanitized

    @api.depends('phone_sanitized')
    def _compute_blacklisted(self):
        # TODO : Should remove the sudo as compute_sudo defined on methods.
        # But if user doesn't have access to mail.blacklist, doen't work without sudo().
        blacklist = set(self.env['phone.blacklist'].sudo().search([
            ('number', 'in', self.mapped('phone_sanitized'))]).mapped('number'))
        number_fields = self._phone_get_number_fields()
        for record in self:
            record.phone_sanitized_blacklisted = record.phone_sanitized in blacklist
            mobile_blacklisted = phone_blacklisted = False
            # This is a bit of a hack. Assume that any "mobile" numbers will have the word 'mobile'
            # in them due to varying field names and assume all others are just "phone" numbers.
            # Note that the limitation of only having 1 phone_sanitized value means that a phone/mobile number
            # may not be calculated as blacklisted even though it is if both field values exist in a model.
            for number_field in number_fields:
                if 'mobile' in number_field:
                    mobile_blacklisted = record.phone_sanitized_blacklisted and record.phone_get_sanitized_number(number_fname=number_field) == record.phone_sanitized
                else:
                    phone_blacklisted = record.phone_sanitized_blacklisted and record.phone_get_sanitized_number(number_fname=number_field) == record.phone_sanitized
            record.mobile_blacklisted = mobile_blacklisted
            record.phone_blacklisted = phone_blacklisted

    @api.model
    def _search_phone_sanitized_blacklisted(self, operator, value):
        # Assumes operator is '=' or '!=' and value is True or False
        self._assert_phone_field()
        if operator != '=':
            if operator == '!=' and isinstance(value, bool):
                value = not value
            else:
                raise NotImplementedError()

        if value:
            query = """
                SELECT m.id
                    FROM phone_blacklist bl
                    JOIN %s m
                    ON m.phone_sanitized = bl.number AND bl.active
            """
        else:
            query = """
                SELECT m.id
                    FROM %s m
                    LEFT JOIN phone_blacklist bl
                    ON m.phone_sanitized = bl.number AND bl.active
                    WHERE bl.id IS NULL
            """
        self._cr.execute(query % self._table)
        res = self._cr.fetchall()
        if not res:
            return [(0, '=', 1)]
        return [('id', 'in', [r[0] for r in res])]

    def _assert_phone_field(self):
        if not hasattr(self, "_phone_get_number_fields"):
            raise UserError(_('Invalid primary phone field on model %s', self._name))
        if not any(fname in self and self._fields[fname].type == 'char' for fname in self._phone_get_number_fields()):
            raise UserError(_('Invalid primary phone field on model %s', self._name))

    def _phone_get_sanitize_triggers(self):
        """ Tool method to get all triggers for sanitize """
        res = [self._phone_get_country_field()] if self._phone_get_country_field() else []
        return res + self._phone_get_number_fields()

    def _phone_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return []

    def _phone_get_country_field(self):
        if 'country_id' in self:
            return 'country_id'
        return False

    def phone_get_sanitized_numbers(self, number_fname='mobile', force_format='E164'):
        res = dict.fromkeys(self.ids, False)
        country_fname = self._phone_get_country_field()
        for record in self:
            number = record[number_fname]
            res[record.id] = phone_validation.phone_sanitize_numbers_w_record([number], record, record_country_fname=country_fname, force_format=force_format)[number]['sanitized']
        return res

    def phone_get_sanitized_number(self, number_fname='mobile', force_format='E164'):
        self.ensure_one()
        country_fname = self._phone_get_country_field()
        number = self[number_fname]
        return phone_validation.phone_sanitize_numbers_w_record([number], self, record_country_fname=country_fname, force_format=force_format)[number]['sanitized']

    def _phone_set_blacklisted(self):
        return self.env['phone.blacklist'].sudo()._add([r.phone_sanitized for r in self])

    def _phone_reset_blacklisted(self):
        return self.env['phone.blacklist'].sudo()._remove([r.phone_sanitized for r in self])

    def phone_action_blacklist_remove(self):
        # wizard access rights currently not working as expected and allows users without access to
        # open this wizard, therefore we check to make sure they have access before the wizard opens.
        can_access = self.env['phone.blacklist'].check_access_rights('write', raise_exception=False)
        if can_access:
            return {
                'name': 'Are you sure you want to unblacklist this Phone Number?',
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'phone.blacklist.remove',
                'target': 'new',
            }
        else:
            raise AccessError("You do not have the access right to unblacklist phone numbers. Please contact your administrator.")

```

## File: models\phone_blacklist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _
from odoo.addons.phone_validation.tools import phone_validation
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class PhoneBlackList(models.Model):
    """ Blacklist of phone numbers. Used to avoid sending unwanted messages to people. """
    _name = 'phone.blacklist'
    _inherit = ['mail.thread']
    _description = 'Phone Blacklist'
    _rec_name = 'number'

    number = fields.Char(string='Phone Number', required=True, tracking=True, help='Number should be E164 formatted')
    active = fields.Boolean(default=True, tracking=True)

    _sql_constraints = [
        ('unique_number', 'unique (number)', 'Number already exists')
    ]

    @api.model_create_multi
    def create(self, values):
        """ Create new (or activate existing) blacklisted numbers.
                A. Note: Attempt to create a number that already exists, but is non-active, will result in its activation.
                B. Note: If the number already exists and it's active, it will be added to returned set, (it won't be re-created)
        Returns Recordset union of created and existing phonenumbers from the requested list of numbers to create
        """
        # Extract and sanitize numbers, ensuring uniques
        to_create = []
        done = set()
        for value in values:
            number = value['number']
            sanitized_values = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]
            sanitized = sanitized_values['sanitized']
            if not sanitized:
                raise UserError(sanitized_values['msg'] + _(" Please correct the number and try again."))
            if sanitized in done:
                continue
            done.add(sanitized)
            to_create.append(dict(value, number=sanitized))

        # Search for existing phone blacklist entries, even inactive ones (will be activated again)
        numbers_requested = [values['number'] for values in to_create]
        existing = self.with_context(active_test=False).search([('number', 'in', numbers_requested)])

        # Out of existing pb records, activate non-active, (unless requested to leave them alone with 'active' set to False)
        numbers_to_keep_inactive = {values['number'] for values in to_create if not values.get('active', True)}
        numbers_to_keep_inactive = numbers_to_keep_inactive & set(existing.mapped('number'))
        existing.filtered(lambda pb: not pb.active and pb.number not in numbers_to_keep_inactive).write({'active': True})

        # Create new records, while skipping existing_numbers
        existing_numbers = set(existing.mapped('number'))
        to_create_filtered = [values for values in to_create if values['number'] not in existing_numbers]
        created = super().create(to_create_filtered)

        # Preserve the original order of numbers requested to create
        numbers_to_id = {record.number: record.id for record in existing | created}
        return self.browse(numbers_to_id[number] for number in numbers_requested)

    def write(self, values):
        if 'number' in values:
            number = values['number']
            sanitized_values = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]
            sanitized = sanitized_values['sanitized']
            if not sanitized:
                raise UserError(sanitized_values['msg'] + _(" Please correct the number and try again."))
            values['number'] = sanitized
        return super(PhoneBlackList, self).write(values)

    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """ Override _search in order to grep search on sanitized number field """
        if args:
            new_args = []
            for arg in args:
                if isinstance(arg, (list, tuple)) and arg[0] == 'number':
                    if isinstance(arg[2], str):
                        number = arg[2]
                        sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
                        search_term = sanitized or number
                    elif isinstance(arg[2], list) and all(isinstance(number, str) for number in arg[2]):
                        search_term = [
                            phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized'] or number
                            for number in arg[2]
                        ]
                    else:
                        search_term = arg[2]
                    new_args.append([arg[0], arg[1], search_term])
                else:
                    new_args.append(arg)
        else:
            new_args = args
        return super(PhoneBlackList, self)._search(new_args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    def add(self, number):
        return self._add([number])

    def _add(self, numbers):
        return self.create([{'number': n} for n in numbers])

    def action_remove_with_reason(self, number, reason=None):
        records = self.remove(number)
        if reason:
            for record in records:
                record.message_post(body=_("Unblacklisting Reason: %s", reason))
        return records

    def remove(self, number):
        sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
        return self._remove([sanitized])

    def _remove(self, numbers):
        """ Add de-activated or de-activate a phone blacklist entry.

        :param numbers: list of sanitized numbers """
        records = self.env["phone.blacklist"].with_context(active_test=False).search([('number', 'in', numbers)])
        todo = [n for n in numbers if n not in records.mapped('number')]
        if records:
            records.action_archive()
        if todo:
            records += self.create([{'number': n, 'active': False} for n in todo])
        return records

    def phone_action_blacklist_remove(self):
        return {
            'name': _('Are you sure you want to unblacklist this Phone Number?'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'phone.blacklist.remove',
            'target': 'new',
        }

    def action_add(self):
        self.add(self.number)

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.addons.phone_validation.tools import phone_validation


class Partner(models.Model):
    _name = 'res.partner'
    _inherit = ['res.partner']

    @api.onchange('phone', 'country_id', 'company_id')
    def _onchange_phone_validation(self):
        if self.phone:
            self.phone = self._phone_format(self.phone, force_format='INTERNATIONAL')

    @api.onchange('mobile', 'country_id', 'company_id')
    def _onchange_mobile_validation(self):
        if self.mobile:
            self.mobile = self._phone_format(self.mobile, force_format='INTERNATIONAL')

    def _phone_format(self, number, country=None, company=None, force_format='E164'):
        country = country or self.country_id or self.env.company.country_id
        if not country or not number:
            return number
        return phone_validation.phone_format(
            number,
            country.code if country else None,
            country.phone_code if country else None,
            force_format=force_format,
            raise_exception=False
        )

    def phone_get_sanitized_number(self, number_fname='mobile', force_format='E164'):
        """ Stand alone version, allowing to use it on partner model without
        having any dependency on sms module. To cleanup in master (15.3 +)."""
        self.ensure_one()
        country_fname = 'country_id'
        number = self[number_fname]
        return phone_validation.phone_sanitize_numbers_w_record([number], self, record_country_fname=country_fname, force_format=force_format)[number]['sanitized']

    def _phone_get_number_fields(self):
        """ Stand alone version, allowing to use it on partner model without
        having any dependency on sms module. To cleanup in master (15.3 +)."""
        return ['mobile', 'phone']

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.addons.phone_validation.tools import phone_validation


class Users(models.Model):
    _inherit = 'res.users'

    def _deactivate_portal_user(self, **post):
        """Blacklist the phone of the user after deleting it."""
        numbers_to_blacklist = {}  # numbers to blacklist and the related user
        if post.get('request_blacklist'):
            for user in self:
                sanitized = phone_validation.phone_sanitize_numbers_w_record([user.phone, user.mobile], user)
                user_phone = sanitized[user.phone]['sanitized']
                user_mobile = sanitized[user.mobile]['sanitized']
                if user_phone:
                    numbers_to_blacklist[user_phone] = user
                if user_mobile:
                    numbers_to_blacklist[user_mobile] = user

        super(Users, self)._deactivate_portal_user(**post)

        if numbers_to_blacklist:
            current_user = self.env.user
            blacklists = self.env['phone.blacklist']._add(
                list(numbers_to_blacklist.keys()))
            for blacklist in blacklists:
                user = numbers_to_blacklist[blacklist.number]
                blacklist._message_log(
                    body=_('Blocked by deletion of portal account %(portal_user_name)s by %(user_name)s (#%(user_id)s)',
                           user_name=current_user.name, user_id=current_user.id,
                           portal_user_name=user.name),
                )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phone_blacklist
from . import mail_thread_phone
from . import res_partner
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_phone_blacklist_all,access.phone.blacklist.all,model_phone_blacklist,,0,0,0,0
access_phone_blacklist_system,access.phone.blacklist.system,model_phone_blacklist,base.group_system,1,1,1,1
access_phone_blacklist_remove_system,acesss.phone.blacklist.remove.system,model_phone_blacklist_remove,base.group_system,1,1,1,1

```

## File: tools\phone_validation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.exceptions import UserError

import logging

_logger = logging.getLogger(__name__)
_phonenumbers_lib_warning = False


try:
    import phonenumbers

    def phone_parse(number, country_code):
        try:
            phone_nbr = phonenumbers.parse(number, region=country_code or None, keep_raw_input=True)
        except phonenumbers.phonenumberutil.NumberParseException as e:
            raise UserError(_('Unable to parse %(phone)s: %(error)s', phone=number, error=str(e)))

        if not phonenumbers.is_possible_number(phone_nbr):
            raise UserError(_('Impossible number %s: probably invalid number of digits.', number))
        if not phonenumbers.is_valid_number(phone_nbr):
            # Force format with international to force metadata to apply
            formatted_intl = phonenumbers.format_number(phone_nbr, phonenumbers.PhoneNumberFormat.INTERNATIONAL)
            phone_nbr_intl = phonenumbers.parse(formatted_intl, region=country_code or None, keep_raw_input=True)
            if not phonenumbers.is_valid_number(phone_nbr_intl):
                raise UserError(_('Invalid number %s: probably incorrect prefix.', number))
            return phone_nbr_intl

        return phone_nbr

    def phone_format(number, country_code, country_phone_code, force_format='INTERNATIONAL', raise_exception=True):
        """ Format the given phone number according to the localisation and international options.
        :param number: number to convert
        :param country_code: the ISO country code in two chars
        :type country_code: str
        :param country_phone_code: country dial in codes, defined by the ITU-T (Ex: 32 for Belgium)
        :type country_phone_code: int
        :param force_format: stringified version of format globals (see
          https://github.com/daviddrysdale/python-phonenumbers/blob/dev/python/phonenumbers/phonenumberutil.py)
            'E164' = 0
            'INTERNATIONAL' = 1
            'NATIONAL' = 2
            'RFC3966' = 3
        :type force_format: str
        :rtype: str
        """
        try:
            phone_nbr = phone_parse(number, country_code)
        except (phonenumbers.phonenumberutil.NumberParseException, UserError) as e:
            if raise_exception:
                raise
            else:
                return number
        if force_format == 'E164':
            phone_fmt = phonenumbers.PhoneNumberFormat.E164
        elif force_format == 'RFC3966':
            phone_fmt = phonenumbers.PhoneNumberFormat.RFC3966
        elif force_format == 'INTERNATIONAL' or phone_nbr.country_code != country_phone_code:
            phone_fmt = phonenumbers.PhoneNumberFormat.INTERNATIONAL
        else:
            phone_fmt = phonenumbers.PhoneNumberFormat.NATIONAL
        return phonenumbers.format_number(phone_nbr, phone_fmt)

except ImportError:

    def phone_parse(number, country_code):
        return False

    def phone_format(number, country_code, country_phone_code, force_format='INTERNATIONAL', raise_exception=True):
        global _phonenumbers_lib_warning
        if not _phonenumbers_lib_warning:
            _logger.info(
                "The `phonenumbers` Python module is not installed, contact numbers will not be "
                "verified. Please install the `phonenumbers` Python module."
            )
            _phonenumbers_lib_warning = True
        return number


def phone_sanitize_numbers(numbers, country_code, country_phone_code, force_format='E164'):
    """ Given a list of numbers, return parsezd and sanitized information

    :return dict: {number: {
        'sanitized': sanitized and formated number or False (if cannot format)
        'code': 'empty' (number was a void string), 'invalid' (error) or False (sanitize ok)
        'msg': error message when 'invalid'
    }}
    """
    if not isinstance(numbers, (list)):
        raise NotImplementedError()
    result = dict.fromkeys(numbers, False)
    for number in numbers:
        if not number:
            result[number] = {'sanitized': False, 'code': 'empty', 'msg': False}
            continue
        try:
            stripped = number.strip()
            sanitized = phone_format(
                stripped, country_code, country_phone_code,
                force_format=force_format, raise_exception=True)
        except Exception as e:
            result[number] = {'sanitized': False, 'code': 'invalid', 'msg': str(e)}
        else:
            result[number] = {'sanitized': sanitized, 'code': False, 'msg': False}
    return result


def phone_sanitize_numbers_w_record(numbers, record, country=False, record_country_fname='country_id', force_format='E164'):
    if not isinstance(numbers, (list)):
        raise NotImplementedError()
    if not country:
        if record and record_country_fname and hasattr(record, record_country_fname) and record[record_country_fname]:
            country = record[record_country_fname]
        elif record:
            country = record.env.company.country_id
    country_code = country.code if country else None
    country_phone_code = country.phone_code if country else None
    return phone_sanitize_numbers(numbers, country_code, country_phone_code, force_format=force_format)

```

## File: tools\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phone_validation

```

## File: views\phone_blacklist_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="phone_blacklist_view_tree" model="ir.ui.view">
        <field name="name">phone.blacklist.view.tree</field>
        <field name="model">phone.blacklist</field>
        <field name="arch" type="xml">
            <tree string="Phone Blacklist">
                <field name="create_date" string="Blacklist Date"/>
                <field name="number"/>
            </tree>
        </field>
    </record>

    <record id="phone_blacklist_view_form" model="ir.ui.view">
        <field name="name">phone.blacklist.view.form</field>
        <field name="model">phone.blacklist</field>
        <field name="arch" type="xml">
            <form string="Phone Blacklist" duplicate="false" edit="false">
                <header>
                    <button name="phone_action_blacklist_remove" string="Unblacklist"
                        type="object" class="oe_highlight" context="{'default_phone': number}"
                        attrs="{'invisible': ['|', ('active', '=', False), ('number', '=', False)]}"/>
                    <button name="action_add" string="Blacklist"
                        type="object" class="oe_highlight"
                        attrs="{'invisible': ['|', ('active', '=', True), ('number', '=', False)]}"/>
                </header>
                <sheet>
                <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <group>
                        <group>
                            <field name="number"/>
                            <field name="active" readonly="1"/>
                            <br/>
                        </group>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="phone_blacklist_view_search" model="ir.ui.view">
        <field name="name">phone.blacklist.view.search</field>
        <field name="model">phone.blacklist</field>
        <field name="arch" type="xml">
            <search>
                <field name="number"/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
            </search>
        </field>
    </record>

    <record id="phone_blacklist_action" model="ir.actions.act_window">
        <field name="name">Blacklisted Phone Numbers</field>
        <field name="res_model">phone.blacklist</field>
        <field name="view_id" ref="phone_blacklist_view_tree"/>
        <field name="search_view_id" ref="phone_blacklist_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a phone number in the blacklist
            </p><p>
                Blacklisted phone numbers won't receive SMS Mailings anymore.
            </p>
        </field>
    </record>

    <!-- Technical Menu -->
    <menuitem id="phone_menu_main"
        name="Phone / SMS"
        parent="base.menu_custom"
        sequence="3"/>

    <menuitem id="phone_blacklist_menu"
        name="Phone Blacklist"
        parent="phone_validation.phone_menu_main"
        sequence="3"
        action="phone_blacklist_action"/>

</odoo>

```

## File: wizard\phone_blacklist_remove.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class PhoneBlacklistRemove(models.TransientModel):
    _name = 'phone.blacklist.remove'
    _description = 'Remove phone from blacklist'

    phone = fields.Char(string="Phone Number", readonly=True, required=True)
    reason = fields.Char(name="Reason")

    def action_unblacklist_apply(self):
        return self.env['phone.blacklist'].action_remove_with_reason(self.phone, self.reason)

```

## File: wizard\phone_blacklist_remove_view.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="phone_blacklist_remove_view_form" model="ir.ui.view">
        <field name="name">phone.blacklist.remove.form</field>
        <field name="model">phone.blacklist.remove</field>
        <field name="arch" type="xml">
            <form string="phone_blacklist_removal">
                <group class="oe_title">
                    <field name="phone" string="Phone Number"/>
                    <field name="reason" string="Reason"/>
                </group>
                <footer>
                    <button name="action_unblacklist_apply" string="Confirm" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phone_blacklist_remove

```

