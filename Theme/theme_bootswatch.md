# Odoo Module: theme_bootswatch

Category: Theme

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Bootswatch Theme',
    'description': 'Bootswatch themes',
    'category': 'Theme',
    'sequence': 900,
    'version': '1.0',
    'depends': ['website', 'website_theme_install'],
    'data': [
        'views/theme_bootswatch_templates.xml',
    ],
    'images': [
        'static/description/bootswatch.png',
        'static/description/bootswatch_screenshot.jpg',
    ],
    'application': False,
    'license': 'LGPL-3',
}

```

## File: models\theme_bootswatch.py

```python
from odoo import models


class ThemeBootswatch(models.AbstractModel):
    _inherit = 'theme.utils'

    def _theme_bootswatch_post_copy(self, mod):
        self.disable_view('website_theme_install.customize_modal')

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import theme_bootswatch

```

## File: static\lib\bootswatch\LICENSE

```text
The MIT License (MIT)

Copyright (c) 2013 Thomas Park

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

```

## File: views\theme_bootswatch_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Remove Odoo bootstrap overridde -->
    <template id="_assets_frontend_helpers" inherit_id="website._assets_frontend_helpers">
        <xpath expr="//link[@href='/web_editor/static/src/scss/bootstrap_overridden.scss']" position="replace"/>
        <xpath expr="//link[@href='/website/static/src/scss/bootstrap_overridden.scss']" position="replace"/>
    </template>

    <template id="theme_customize" inherit_id="website.theme_customize">
        <xpath expr="//div" position="replace">
            <div>
                <content data-string="Themes" data-title="Bootswatch Themes">
                    <opt data-xmlid="" data-icon="/theme_bootswatch/static/src/img/bootswatch_default_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_cerulean_bs_variables,theme_bootswatch.theme_cerulean" data-icon="/theme_bootswatch/static/src/img/cerulean_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_cosmo_bs_variables,theme_bootswatch.theme_cosmo" data-disable="ee" data-icon="/theme_bootswatch/static/src/img/cosmo_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_cyborg_bs_variables,theme_bootswatch.theme_cyborg" data-icon="/theme_bootswatch/static/src/img/cyborg_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_flatly_bs_variables,theme_bootswatch.theme_flatly" data-icon="/theme_bootswatch/static/src/img/flatly_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_journal_bs_variables,theme_bootswatch.theme_journal" data-icon="/theme_bootswatch/static/src/img/journal_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_simplex_bs_variables,theme_bootswatch.theme_simplex" data-icon="/theme_bootswatch/static/src/img/simplex_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_slate_bs_variables,theme_bootswatch.theme_slate" data-icon="/theme_bootswatch/static/src/img/slate_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_spacelab_bs_variables,theme_bootswatch.theme_spacelab" data-icon="/theme_bootswatch/static/src/img/spacelab_thumbnail.png"/>
                    <opt data-xmlid="theme_bootswatch.theme_united_bs_variables,theme_bootswatch.theme_united" data-icon="/theme_bootswatch/static/src/img/united_thumbnail.png"/>
                </content>
            </div>
        </xpath>
    </template>

    <template id="theme_cerulean_bs_variables" name="Cerulean" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/cerulean/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_cerulean" name="Cerulean" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/cerulean/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/cerulean_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_cosmo_bs_variables" name="Cosmo" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/cosmo/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_cosmo" name="Cosmo" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/cosmo/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/cosmo_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_cyborg_bs_variables" name="Cyborg" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/cyborg/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_cyborg" name="Cyborg" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/cyborg/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/cyborg_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_flatly_bs_variables" name="Flatly" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/flatly/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_flatly" name="Flatly" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/flatly/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/flatly_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_journal_bs_variables" name="Journal" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/journal/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_journal" name="Journal" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/journal/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/journal_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_simplex_bs_variables" name="Simplex" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/simplex/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_simplex" name="Simplex" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/simplex/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/simplex_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_slate_bs_variables" name="Slate" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/slate/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_slate" name="Slate" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/slate/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/slate_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_spacelab_bs_variables" name="Spacelab" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/spacelab/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_spacelab" name="Spacelab" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/spacelab/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/spacelab_fix.scss"/>
        </xpath>
    </template>

    <template id="theme_united_bs_variables" name="United" inherit_id="website._assets_frontend_helpers" active="False">
        <xpath expr="//link" position="before">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/united/_variables.scss"/>
        </xpath>
    </template>
    <template id="theme_united" name="United" inherit_id="website.assets_frontend" active="False">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/lib/bootswatch/united/_bootswatch.scss"/>
            <link rel="stylesheet" type="text/scss" href="/theme_bootswatch/static/src/scss/united_fix.scss"/>
        </xpath>
    </template>
</odoo>

```

