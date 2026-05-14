# ERPNext 16 — Local Development Setup

ERPNext is a free, open-source ERP system built on the Frappe Framework.
This repo contains the Docker-based setup to run ERPNext 16 locally on Windows 11,
along with the source code for codebase exploration.

---

## Project Structure

```
erpnext-project/
├── frappe_docker/        # Docker Compose setup (start/stop the app here)
│   ├── compose.yaml      # Main service definitions
│   ├── .env              # Environment config (version, port, passwords)
│   └── overrides/        # Add-on configs (MariaDB, Redis, proxy)
└── erpnext-codebase/     # ERPNext v16 source code for exploration
    └── erpnext/
        ├── accounts/     # Finance, invoices, ledger
        ├── buying/       # Purchase orders, suppliers
        ├── selling/      # Sales orders, customers
        ├── stock/        # Inventory, warehouses
        ├── manufacturing/# Work orders, BOM
        ├── hr/           # Employees, payroll
        └── controllers/  # Shared business logic
```

---

## Requirements

- Windows 11
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (must be running)
- Git

---

## Quick Start

### 1. Make sure Docker Desktop is running

Open Docker Desktop from the Start Menu before running any commands.

### 2. Start ERPNext

Open a terminal in the `frappe_docker/` folder and run:

```bash
docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.mariadb-healthfix.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  --project-name erpnext16 \
  up -d
```

### 3. Open in browser

```
http://localhost:8080
```

| Field    | Value           |
|----------|-----------------|
| Username | `Administrator` |
| Password | `admin123`      |

---

## Stop ERPNext

```bash
docker compose --project-name erpnext16 down
```

> Your data is saved in Docker volumes — it will still be there next time you start.

---

## Check Status

```bash
# See all running containers
docker compose --project-name erpnext16 ps

# See live logs
docker compose --project-name erpnext16 logs -f backend

# See just errors
docker compose --project-name erpnext16 logs -f backend | grep -i error
```

---

## Useful Developer Commands

### Open a Python console inside ERPNext

```bash
docker exec -it erpnext16-backend-1 bench --site localhost console
```

Once inside, you can query the database directly:

```python
import frappe

# Get a document
doc = frappe.get_doc("Company", "Global Kwik Koders (Demo)")
print(doc.as_dict())

# Run a database query
results = frappe.db.sql("SELECT name FROM `tabCustomer` LIMIT 5", as_dict=True)
print(results)

# List all DocTypes
frappe.get_all("DocType", filters={"module": "Accounts"}, pluck="name")
```

### Open MariaDB (database) shell

```bash
docker exec -it erpnext16-db-1 mariadb -u root -padmin123
```

### Run a bench command on the site

```bash
# Run pending database migrations
docker exec erpnext16-backend-1 bench --site localhost migrate

# Clear cache
docker exec erpnext16-backend-1 bench --site localhost clear-cache

# List installed apps
docker exec erpnext16-backend-1 bench --site localhost list-apps

# Backup the site
docker exec erpnext16-backend-1 bench --site localhost backup
```

### Rebuild frontend assets (after JS/CSS changes)

```bash
docker exec erpnext16-backend-1 bench build
```

---

## Environment Configuration

The `.env` file inside `frappe_docker/` controls the setup:

```env
ERPNEXT_VERSION=v16.18.2       # ERPNext version to use
DB_PASSWORD=admin123           # MariaDB root password
FRAPPE_SITE_NAME_HEADER=localhost  # Maps browser host to site name
HTTP_PUBLISH_PORT=8080         # Port exposed on your machine
```

---

## Installed Versions

| Component | Version  |
|-----------|----------|
| ERPNext   | v16.18.2 |
| Frappe    | v16.18.1 |
| MariaDB   | 11.8     |
| Redis     | 8.6      |

---

## ERPNext Module Overview

| Module        | What it does                                      |
|---------------|---------------------------------------------------|
| Accounts      | Chart of Accounts, invoices, payments, ledger     |
| Buying        | Suppliers, purchase orders, purchase invoices     |
| Selling       | Customers, quotations, sales orders, invoices     |
| Stock         | Items, warehouses, stock transfers, inventory     |
| Manufacturing | Bill of Materials, work orders, production        |
| HR            | Employees, departments, leaves, attendance        |
| Payroll       | Salary structures, payroll runs, salary slips     |
| Projects      | Projects, tasks, timesheets                       |
| CRM           | Leads, opportunities, pipeline                    |
| Assets        | Fixed assets, depreciation                        |

---

## How ERPNext Code is Organized

Every feature in ERPNext is a **DocType** — a model that auto-generates the database table, form UI, REST API, and list view all at once.

```
erpnext/accounts/doctype/sales_invoice/
├── sales_invoice.json      # Field definitions and permissions
├── sales_invoice.py        # Server-side Python logic
├── sales_invoice.js        # Client-side form behavior
└── test_sales_invoice.py   # Automated tests
```

### Files worth reading first

| File | Why read it |
|------|-------------|
| `erpnext/accounts/doctype/sales_invoice/sales_invoice.py` | Core transaction logic |
| `erpnext/controllers/accounts_controller.py` | Shared logic across all finance docs |
| `erpnext/stock/doctype/stock_ledger_entry/stock_ledger_entry.py` | How inventory is tracked |
| `erpnext/hooks.py` | App-level event hooks and overrides |

---

## Troubleshooting

### Docker Desktop not running
Start Docker Desktop from the Start Menu, wait ~30 seconds, then retry.

### Port 8080 already in use
Change `HTTP_PUBLISH_PORT=8080` to another port (e.g. `8090`) in `.env`, then restart.

### Site not loading / 502 error
Wait 30 seconds after starting — the backend takes time to initialize. Check logs:
```bash
docker compose --project-name erpnext16 logs -f backend
```

### Reset everything (fresh start)
```bash
# WARNING: this deletes all data
docker compose --project-name erpnext16 down -v
```
Then re-run the start command and recreate the site.

---

## Resources

- [ERPNext Documentation](https://docs.erpnext.com)
- [Frappe Framework Docs](https://frappeframework.com/docs)
- [ERPNext GitHub](https://github.com/frappe/erpnext)
- [Frappe Docker GitHub](https://github.com/frappe/frappe_docker)
