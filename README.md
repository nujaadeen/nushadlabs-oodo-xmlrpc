# Odoo XML-RPC API — Python Notebooks

A collection of Jupyter notebooks demonstrating full CRUD operations and a complete
business onboarding flow against the Odoo 17+ XML-RPC API using only Python's
built-in `xmlrpc.client` library.

---

## Notebooks

| Notebook | Description |
|----------|-------------|
| [`business flow.ipynb`](#business-flow) | End-to-end company onboarding — company, user, accounts, journals, invoice, payment |
| [`oodo_xmlrpc_create_demo.ipynb`](#create) | Creating partners, products, sales orders, journals |
| [`oodo_xmlrpc_read_demo.ipynb`](#read) | Searching, reading, filtering, and inspecting model schemas |
| [`oodo_xmlrpc_update_demo.ipynb`](#update) | Updating sales orders and product records with `write()` |
| [`oodo_xmlrpc_delete_demo.ipynb`](#delete) | Safe deletion and archiving with `unlink()` and `write(active: False)` |

---

## Prerequisites

- Python 3.8+ (no third-party packages required — uses stdlib `xmlrpc.client`)
- A running Odoo 17+ instance (local or remote)
- A valid Odoo user with API key access enabled

### Configuration

Each notebook shares the same connection block at the top. Update these four variables
before running:

```python
URL      = "http://localhost:8069"   # Odoo instance URL
DB       = "your_database"           # Database name
USERNAME = "user@company.com"        # Login email
API_KEY  = "your_api_key"            # Settings → Technical → API Keys
```

---

## Notebook Details

### Business Flow

**`business flow.ipynb`** — the flagship notebook. Provisions a complete company tenant
from scratch in five phases:

| Phase | What Gets Created | Key API Models |
|-------|-------------------|----------------|
| 1 | Company | `res.company` |
| 2 | Admin user with full ERP + Accounting roles | `res.users`, `res.groups`, `ir.model.data` |
| 3 | Chart of Accounts + global dummy product | `account.account`, `product.template` |
| 4 | Sales journal + Bank journal (with payment wiring) | `account.journal`, `account.payment.method.line` |
| 5 | Guest partner → Invoice → Bank payment | `res.partner`, `account.move`, `account.payment.register` |

Notable patterns covered:
- Dynamic schema discovery to resolve the `res.users → res.groups` field name (version-safe)
- Resolving group database IDs from stable XML IDs via `ir.model.data`
- Odoo 17+ `company_ids` (Many2many) vs `company_id` distinction on `account.account`
- Wiring the 3-step bank reconciliation flow: *Register → Outstanding Receipts → Bank Statement*
- The `account.payment.register` transient wizard pattern (create then trigger)

---

### Create

**`oodo_xmlrpc_create_demo.ipynb`** — demonstrates record creation across multiple models:

- `res.partner` — customer and supplier contacts
- `sale.order` — sales orders with order lines
- `product.template` — product catalog entries
- `account.journal` — accounting journals

---

### Read

**`oodo_xmlrpc_read_demo.ipynb`** — covers all four read operations on `product.template`:

| Method | Purpose |
|--------|---------|
| `search()` | Returns a list of matching IDs |
| `read()` | Fetches specific fields for given IDs |
| `search_read()` | Combined search + read in one call (preferred for performance) |
| `search_count()` | Returns the count of matching records |
| `fields_get()` | Introspects a model's field definitions and types |

---

### Update

**`oodo_xmlrpc_update_demo.ipynb`** — demonstrates `write()` across different field types
on `sale.order` and `product.product`:

- Updating scalar fields (strings, numbers)
- Replacing Many2many relations using the `(6, 0, [ids])` command
- Appending / removing items from One2many lines

---

### Delete

**`oodo_xmlrpc_delete_demo.ipynb`** — covers safe record removal on `res.partner`
and `res.users`:

- `unlink()` — hard delete (permanent, use with caution)
- `write({'active': False})` — soft delete / archive (recoverable, preferred in production)

---

## Key XML-RPC Concepts

```python
# Both endpoints are needed — common for auth, models for all operations
common = xmlrpc.client.ServerProxy(f'{URL}/xmlrpc/2/common', allow_none=True)
models = xmlrpc.client.ServerProxy(f'{URL}/xmlrpc/2/object', allow_none=True)

# Authenticate once — reuse the returned uid for all subsequent calls
uid = common.authenticate(DB, USERNAME, API_KEY, {})

# General call signature
models.execute_kw(DB, uid, API_KEY, 'model.name', 'method', [args], {kwargs})
```

### Many2many / One2many command tuples

| Tuple | Effect |
|-------|--------|
| `(6, 0, [ids])` | Replace entire relation with given IDs |
| `(4, id, 0)` | Link an existing record (add without removing others) |
| `(3, id, 0)` | Unlink a record (remove from relation, don't delete) |
| `(0, 0, {vals})` | Create a new related record and link it |

---

## License

MIT
