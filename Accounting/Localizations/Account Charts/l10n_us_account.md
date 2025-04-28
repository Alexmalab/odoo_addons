# Odoo Module: l10n_us_account

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The purpose of l10n_us_account is to automatically trigger the installation of l10n_us for the new US databases
# Also, l10n_us_account should contains all the accounting-related dependencies of US localization package
# Currently, It's empty module but it should be filled going forward!

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'United States - Accounting',
    'website': 'https://www.odoo.com/documentation/18.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['us'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
    """,
    'depends': ['l10n_us', 'account'],
    'installable': True,
    'auto_install': ['account'],
    'license': 'LGPL-3',
}

```

