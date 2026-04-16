# Claude Code Review Guidelines

# Odoo Partner — OCA Standards + ERP Customization

## Reference Standards

This file enforces the following standards in order of priority:

1. OCA Guidelines: https://odoo-community.org/page/contributing
2. Odoo Official Guidelines: https://www.odoo.com/documentation/17.0/contributing/development/coding_guidelines.html
3. Company-specific rules defined in this file

---

## Module Naming (OCA)

- Use **singular form** in module name (e.g. `sale_report`, not `sales_reports`), except when
  combining with a plural Odoo module name (e.g. `mrp_operations_custom`)
- Prefix with the extended module name: e.g. `sale_custom_discount`, `account_payment_egypt`
- Prefix base/utility modules with `base_`: e.g. `base_location_nuts`
- Prefix localization modules with `l10n_CC_`: e.g. `l10n_eg_pos` for Egypt
- When combining Odoo module with OCA module, Odoo name goes first:
  e.g. `crm_partner_firstname`
- Module names must use only `[a-z0-9_]` characters
- Never use `odoo` or `openerp` in the project/module name

Flag if:

- Module name is plural when it should be singular
- Localization module is missing the `l10n_eg_` prefix
- Module name contains uppercase letters or hyphens

---

## Module Structure (OCA)

Every module must follow this exact structure:

```
addons/<my_module_name>/
├── controllers/
│   ├── __init__.py
│   └── main.py
├── data/
│   └── <main_model>_data.xml
├── demo/
│   └── <main_model>_demo.xml
├── i18n/
│   └── <module_name>.pot
├── migrations/
│   └── 17.0.x.y.z/
│       ├── pre-migration.py
│       └── post-migration.py
├── models/
│   ├── __init__.py
│   ├── <main_model>.py
│   └── <inherited_model>.py
├── readme/
│   ├── CONTRIBUTORS.rst
│   ├── DESCRIPTION.rst
│   └── USAGE.rst
├── reports/
│   └── report_<n>.xml
├── security/
│   ├── ir.model.access.csv
│   └── <main_model>_security.xml
├── static/
│   └── src/
│       ├── js/
│       ├── css/
│       └── xml/
├── tests/
│   ├── __init__.py
│   └── test_<main_model>.py
├── views/
│   ├── <main_model>_views.xml
│   └── <inherited_model>_views.xml
├── wizards/
│   ├── __init__.py
│   ├── <wizard_model>.py
│   └── <wizard_model>_views.xml
├── README.rst
├── __init__.py
├── __manifest__.py
├── exceptions.py
└── hooks.py
```

Flag if:

- Directory structure deviates significantly without justification
- File permissions are wrong (folders should be 755, files 644)
- Filenames contain characters other than `[a-z0-9_]` plus the file extension

---

## File Naming (OCA)

Split files by the model they involve. Use these naming patterns:

| Content | File path |
|---|---|
| Model definition | `models/<main_model>.py` |
| Model views | `views/<main_model>_views.xml` |
| Data records | `data/<main_model>_data.xml` |
| Demo data | `demo/<main_model>_demo.xml` |
| Templates | `templates/<main_model>_template.xml` |
| Report templates | `reports/report_<n>.xml` |
| Security rules | `security/<main_model>_security.xml` |
| Wizard model | `wizards/<wizard_model>.py` |
| Wizard views | `wizards/<wizard_model>_views.xml` |
| Tests | `tests/test_<main_model>.py` |
| Single controller | `controllers/main.py` |

One2many child models (e.g. `sale.order.line`) may be defined in the same file
as the parent model (`models/sale_order.py`).

Flag if:

- View files are missing the `_views` suffix
- Data files are missing the `_data` suffix
- Demo files are missing the `_demo` suffix

---

## __manifest__.py (OCA)

```python
{
    "name": "Module Name",
    "summary": "Short description (one line)",
    "version": "17.0.1.0.0",         # odoo_version.major.minor.patch.fix
    "category": "Category",
    "author": "Your Company, Odoo Community Association (OCA)",
    "website": "https://github.com/OCA/<repo>",
    "license": "LGPL-3",             # or AGPL-3 or OPL-1
    "depends": ["base"],
    "data": [],
    "demo": [],
    "external_dependencies": {
        "python": [],
        "bin": [],
    },
    "installable": True,
    "application": False,
    "auto_install": False,
    "images": ["static/description/banner.png"],
}
```

### Version numbering (OCA SemVer)

Format: `odoo_version.x.y.z` where:

- **x (Major)**: significant data model or view changes requiring data migration
- **y (Minor)**: new features added without breaking backward compatibility
- **z (Patch)**: bug fixes, typically only a server restart needed

Flag if:

- Version format does not follow `17.0.x.y.z`
- `license` key is missing
- `images` key is missing
- Empty keys are present in the manifest
- No `external_dependencies` section when third-party Python packages are imported

---

## Python (OCA)

### PEP 8 — OCA specific options

- Max line length: **119 characters** (OCA uses 119, not PEP8 default of 79)
- Use 4 spaces for indentation, never tabs
- Two blank lines between top-level definitions
- One blank line between methods inside a class

### Import order

```python
# 1. Standard library
import logging
from datetime import datetime

# 2. Third-party
import requests

# 3. Odoo
from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError

# 4. Local
from .utils import helper_function
```

Flag if:

- Imports are not grouped and ordered correctly
- Wildcard imports like `from odoo import *` are used

### Odoo Python Classes (OCA)

```python
class SaleOrder(models.Model):
    """Always add a docstring."""
    _name = "sale.order"
    _description = "Sale Order"    # mandatory, must be human-readable
    _inherit = ["mail.thread", "mail.activity.mixin"]
    _order = "date_order desc, id desc"

    # --- Fields ---
    name = fields.Char(
        string="Order Reference",
        required=True,
        copy=False,
        readonly=True,
        default=lambda self: _("New"),
    )
```

Flag if:

- Model is missing `_description`
- `_description` is identical to `_name` (must be human-readable, not technical)
- Class has no docstring

### Variable names (OCA)

```python
# Recordsets — always use plural
orders = self.env["sale.order"].search([])
for order in orders:            # singular in loop
    partner = order.partner_id  # Many2one — singular

# Avoid single-letter variables except in very short lambdas
total = sum(line.price_total for line in order.order_line)
```

### Fields (OCA)

- `string` parameter is **mandatory** for all fields
- Field attributes order: type-specific params, `string`, `required`, `readonly`,
  `store`, `compute`, `inverse`, `search`, `related`, `default`, `help`, `tracking`

```python
# CORRECT
custom_field = fields.Char(
    string="Custom Reference",
    required=True,
    copy=False,
    tracking=True,
    help="Internal reference for this record.",
)

partner_id = fields.Many2one(
    comodel_name="res.partner",
    string="Customer",
    required=True,
    ondelete="restrict",
    index=True,
    tracking=True,
)

amount_total = fields.Monetary(
    string="Total Amount",
    currency_field="currency_id",
    compute="_compute_amount_total",
    store=True,
)

# WRONG — missing string, missing ondelete
partner_id = fields.Many2one("res.partner")
```

Flag if:

- `string` is missing on any field
- `Many2one` field is missing `ondelete` policy
- `Monetary` field has no `currency_field`
- Computed field has empty `@api.depends()`
- `Float` field is used for monetary values instead of `Monetary`

### SQL (OCA)

Never use raw SQL unless absolutely necessary.

```python
# WRONG — SQL injection risk
self.env.cr.execute("SELECT id FROM sale_order WHERE name = '%s'" % name)

# CORRECT — always use parametrized queries
self.env.cr.execute("SELECT id FROM sale_order WHERE name = %s", (name,))

# PREFERRED — use ORM
orders = self.env["sale.order"].search([("name", "=", name)])
```

Also flag:

- `self.env.cr.commit()` — never commit the transaction inside a module method
- Raw SQL that bypasses ORM when ORM is feasible
- Direct `UPDATE`, `INSERT`, or `DELETE` SQL statements

### Exceptions (OCA)

```python
# CORRECT
from odoo.exceptions import UserError, ValidationError

raise UserError(_("The order must be confirmed before invoicing."))

@api.constrains("date_start", "date_end")
def _check_dates(self):
    for record in self:
        if record.date_start > record.date_end:
            raise ValidationError(_("Start date must be before end date."))

# WRONG
raise Exception("Something went wrong")
except:
    pass
```

---

## XML Files (OCA)

### Format

- Indent using **4 spaces**
- Place `id` attribute before `model`
- `name` attribute on fields goes first, then value, then other attributes
- Group records by model
- Use `<odoo>` as root tag (never `<openerp>`)
- Use `<odoo noupdate="1">` for non-updatable master data
- Use `<data noupdate="1">` inside `<odoo>` only for mixed updatable/non-updatable files

```xml
<odoo>
    <record id="view_sale_order_form_inherit_custom" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.my_module</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="custom_field"/>
            </xpath>
        </field>
    </record>
</odoo>
```

### XML ID Naming (OCA)

Do **not** prefix XML IDs with the current module name inside the file.

| Record type | Convention | Example |
|---|---|---|
| View | `view_{model}_{type}` | `view_sale_order_form` |
| Action | `action_{model}` | `action_sale_order` |
| Menu | `menu_{description}` | `menu_sale_order_list` |
| Security group | `group_{name}` | `group_sale_manager` |
| Server action | `action_server_{description}` | `action_server_send_email` |
| Sequence | `sequence_{model}` | `sequence_sale_order` |

```xml
<!-- CORRECT -->
<record id="view_sale_order_form_custom" model="ir.ui.view">

<!-- WRONG — prefixed with module name inside the file -->
<record id="my_module.view_sale_order_form_custom" model="ir.ui.view">
```

### View Inheritance (OCA)

Always use `xpath` for inheritance. Prefer `position="after"` or `position="before"`
over `position="replace"`.

```xml
<!-- CORRECT -->
<xpath expr="//field[@name='partner_id']" position="after">
    <field name="custom_field"/>
</xpath>

<!-- WRONG — replaces entire view -->
<xpath expr="//form" position="replace">
    ...entire form...
</xpath>
```

Flag if:

- `position="replace"` is used without strong justification
- xpath targets a very generic element like `//form` or `//tree`

---

## Security (OCA)

Every new model must have entries in `security/ir.model.access.csv`:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_my_model_user,my.model user,model_my_model,base.group_user,1,0,0,0
access_my_model_manager,my.model manager,model_my_model,my_module.group_manager,1,1,1,1
```

```python
# sudo() must always have a justification comment
# sudo() required: portal users cannot access internal notes directly
record.sudo().message_post(body=message)
```

Flag if:

- A new model has no access rights defined
- `sudo()` is used without an explanatory comment
- Public routes expose data without access checks

---

## External Dependencies (OCA)

Declare in `__manifest__.py`:

```python
"external_dependencies": {
    "python": ["requests", "pandas"],
    "bin": ["wkhtmltopdf"],
},
```

Wrap imports in try-except:

```python
try:
    import requests
except ImportError:
    _logger.debug("requests library not found")
```

Flag if:

- Third-party packages imported but not in `external_dependencies`
- Exact version pins placed in `external_dependencies` (version pinning is
  the integrator's responsibility, not the module's)

---

## Migrations (OCA)

Required when a PR: renames a field, changes a field type, removes a field,
or changes data structure significantly.

```
migrations/
└── 17.0.2.0.0/
    ├── pre-migration.py
    └── post-migration.py
```

```python
import logging
_logger = logging.getLogger(__name__)

def migrate(cr, version):
    _logger.info("Migrating old_field to new_field in my.model")
    cr.execute("""
        UPDATE my_model
        SET new_field = old_field
        WHERE new_field IS NULL
    """)
```

Flag if:

- A PR renames or removes a field without a migration script
- A PR changes a field type without migration handling
- Module version was not bumped for the type of change made

---

## Tests (OCA)

```python
from odoo.tests import TransactionCase, tagged
from odoo.exceptions import ValidationError
from freezegun import freeze_time

@tagged("post_install", "-at_install")
class TestSaleOrderCustom(TransactionCase):

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.partner = cls.env["res.partner"].create({"name": "Test Partner"})

    def test_custom_field_default(self):
        order = self.env["sale.order"].create({
            "partner_id": self.partner.id,
        })
        self.assertEqual(order.custom_field, "expected_value")

    def test_constraint_raises_on_invalid_date(self):
        with self.assertRaises(ValidationError):
            self.env["my.model"].create({"date_start": "2024-12-01", "date_end": "2024-01-01"})

    @freeze_time("2024-06-15")
    def test_date_dependent_logic(self):
        # date is frozen — deterministic result
        record = self.env["my.model"].create({...})
        self.assertEqual(record.computed_date_field, "2024-06-15")
```

### Test tags

- `@tagged("post_install", "-at_install")` — most common
- `@tagged("at_install")` — runs during install
- `@tagged("external")` — depends on external service
- `@tagged("external_l10n")` — localization tests needing external services

Flag if:

- Complex business logic has no test coverage
- Tests call real external APIs without mocking
- Tests rely on demo data (`ref()` calls in test setup)
- `datetime.now()` used in tests without `freezegun`
- `SavepointCase` used (deprecated since Odoo 15 — use `TransactionCase`)

---

## Git Commit Messages (OCA)

Format: `[TAG] module_name: Short description`

| Tag | Meaning |
|---|---|
| `[ADD]` | New feature or module |
| `[FIX]` | Bug fix |
| `[IMP]` | Improvement to existing feature |
| `[REF]` | Refactoring (no functional change) |
| `[REM]` | Removal of feature or file |
| `[MIG]` | Migration to new Odoo version |
| `[I18N]` | Translation update |

Examples:

```
[ADD] sale_custom_discount: Add customer-specific discount rules
[FIX] account_payment_egypt: Fix currency rounding on EGP invoices
[IMP] stock_custom_report: Improve performance of inventory valuation report
[MIG] sale_custom_discount: Migration to 17.0
```

Flag if:

- Commit messages are missing the `[TAG]` prefix
- Multiple unrelated changes bundled in one commit

---

## Logging

```python
import logging
_logger = logging.getLogger(__name__)

_logger.debug("Detailed debug info: %s", record.name)
_logger.info("Processing %d orders", len(orders))
_logger.warning("Deprecated method called in %s", self._name)
_logger.error("Failed to sync with external API: %s", str(e))

# WRONG
print("something happened")
_logger.info("Error: " + str(e))   # never use string concatenation in logger
```

---

## Multi-Company & Multi-Currency

```python
# CORRECT — financial model
company_id = fields.Many2one(
    comodel_name="res.company",
    string="Company",
    required=True,
    default=lambda self: self.env.company,
)
currency_id = fields.Many2one(
    comodel_name="res.currency",
    string="Currency",
    related="company_id.currency_id",
    store=True,
)
amount_total = fields.Monetary(
    string="Total",
    currency_field="currency_id",
)
```

Flag if:

- New financial models missing `company_id`
- `Monetary` fields have no matching `currency_id`
- Domain filters on financial models do not include company filtering

---

## Egypt / MENA Localization

- Localization modules must be prefixed `l10n_eg_`
- Monetary amounts must use `Monetary` field, never `Float`
- Date/time fields must be timezone-aware
- Arabic user-facing strings must have `translate=True`
- Never hardcode tax rates — use Odoo fiscal positions
- Flag payment integrations not referencing CBE (Central Bank of Egypt) compliance

---

## External Integrations

```python
# CORRECT — credentials via system parameters
api_key = self.env["ir.config_parameter"].sudo().get_param("my_module.api_key")

# WRONG — hardcoded credential
api_key = "sk-abc123"
```

Flag any integration that:

- Stores credentials in code or XML data files
- Does not handle API timeouts and retries
- Does not log requests/responses for debugging
- Processes webhooks without signature verification
- Has no sandbox/test mode configuration

---

## Final Checklist (Every PR)

- [ ] No `print()` statements — use `_logger`
- [ ] All user-facing strings wrapped in `_()` for translation
- [ ] No hardcoded IDs, company names, emails, or URLs in code
- [ ] No `@api.one` decorator (removed in Odoo 14+)
- [ ] No `openerp` imports (use `odoo` instead)
- [ ] `_description` present on all new models
- [ ] `string` present on all fields
- [ ] `ondelete` present on all `Many2one` fields
- [ ] Security access rights defined for all new models
- [ ] `sudo()` usage justified with inline comment
- [ ] No SQL injection risk
- [ ] No `self.env.cr.commit()` calls
- [ ] Module version bumped appropriately
- [ ] Migration script included if breaking change

---

## Review Severity Levels

- **CRITICAL** — security vulnerability, data loss risk, SQL injection, hardcoded credentials
- **ERROR** — missing access rights, broken OCA structure, ORM bypass, missing migration script
- **WARNING** — performance issue, missing `@api.depends`, upgrade risk, no tests for new logic
- **INFO** — style improvement, naming convention, minor optimization suggestion