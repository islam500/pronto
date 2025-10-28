# Pronto Print Order Management System - Coding Implementation Guide v1.0

## Table of Contents
1. [System Architecture](#1-system-architecture)
2. [Development Environment Setup](#2-development-environment-setup)
3. [Frappe App Structure](#3-frappe-app-structure)
4. [DocType Definitions](#4-doctype-definitions)
5. [API Endpoints](#5-api-endpoints)
6. [Authentication Implementation](#6-authentication-implementation)
7. [Payment Integration](#7-payment-integration)
8. [File Management](#8-file-management)
9. [Frontend Implementation](#9-frontend-implementation)
10. [New Features Implementation](#10-new-features-implementation)
11. [Local Print Utility](#11-local-print-utility)
12. [Deployment](#12-deployment)

---

## 1. System Architecture

### 1.1 Technology Stack

**Backend:**
- **Framework**: Frappe v15
- **Language**: Python 3.8+
- **Database**: PostgreSQL 11+ (Port 5435)
- **ORM**: Frappe ORM
- **API**: REST API (Frappe whitelisted methods)

**Frontend:**
- **Framework**: Vanilla JavaScript + Frappe Client
- **UI Library**: shadcn UI components
- **Icons**: Lucide Icons
- **CSS**: Tailwind CSS + Custom CSS
- **Build**: Frappe build system

**Infrastructure:**
- **Containerization**: Docker
- **Web Server**: Nginx (via Frappe)
- **Application Server**: Gunicorn
- **Task Queue**: Frappe Background Jobs
- **Caching**: Redis

**Integrations:**
- **Payment**: Paystack API
- **Authentication**: Google OAuth 2.0
- **File Storage**: Local filesystem (date-based folders)

### 1.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         Client Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Browser    │  │  Mobile Web  │  │ Print Utility│      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Nginx (Reverse Proxy)                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Frappe Application Layer                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Web Routes  │  │  API Routes  │  │  Realtime    │      │
│  │  (www pages) │  │ (whitelisted)│  │  (Socket.IO) │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   DocTypes   │  │   Hooks      │  │  Background  │      │
│  │   (Models)   │  │   (Events)   │  │    Jobs      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  PostgreSQL  │  │    Redis     │  │  File System │      │
│  │  (Database)  │  │   (Cache)    │  │   (PDFs)     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    External Services                         │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │   Paystack   │  │ Google OAuth │                         │
│  │   (Payment)  │  │    (Auth)    │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Directory Structure

```
frappe-bench/
├── apps/
│   └── pronto/
│       ├── pronto/
│       │   ├── __init__.py
│       │   ├── hooks.py                    # App configuration
│       │   ├── api/                        # API endpoints
│       │   │   ├── __init__.py
│       │   │   ├── auth.py                 # Authentication APIs
│       │   │   ├── orders.py               # Order management APIs
│       │   │   ├── payment.py              # Payment APIs
│       │   │   ├── staff.py                # Staff operations APIs
│       │   │   ├── qa.py                   # QA operations APIs
│       │   │   ├── manager.py              # Manager operations APIs
│       │   │   ├── credit.py               # Credit system APIs (NEW)
│       │   │   └── print_utility.py        # Print utility APIs (NEW)
│       │   ├── pronto/
│       │   │   └── doctype/                # DocType definitions
│       │   │       ├── print_order/
│       │   │       │   ├── print_order.json
│       │   │       │   └── print_order.py
│       │   │       ├── job_queue/
│       │   │       ├── pronto_customer/
│       │   │       ├── pricing_configuration/
│       │   │       ├── print_paper_size/
│       │   │       ├── print_paper_type/
│       │   │       ├── print_finishing/
│       │   │       ├── pickup_location/
│       │   │       ├── dispatch_batch/
│       │   │       ├── dispatch_rider/
│       │   │       ├── payment_transaction_log/
│       │   │       ├── print_center/        # NEW
│       │   │       ├── discount_coupon/     # NEW
│       │   │       ├── coupon_usage_log/    # NEW
│       │   │       ├── customer_credit_transaction/  # NEW
│       │   │       └── loyalty_configuration/        # NEW
│       │   ├── fixtures/                   # Initial data
│       │   │   └── initial_data.py
│       │   ├── www/                        # Web pages
│       │   │   ├── login.html
│       │   │   ├── dashboard.html
│       │   │   ├── print-staff.html
│       │   │   ├── qa-staff.html
│       │   │   ├── dispatch-staff.html
│       │   │   ├── collection-staff.html
│       │   │   ├── business-manager.html
│       │   │   ├── privacy-policy.html     # NEW
│       │   │   └── terms-of-service.html   # NEW
│       │   └── public/                     # Static assets
│       │       ├── css/
│       │       │   ├── landing.css
│       │       │   └── pronto-theme.css    # NEW
│       │       ├── js/
│       │       │   └── components/         # shadcn components
│       │       └── sounds/
│       │           └── notification.mp3    # NEW
│       ├── requirements.txt
│       └── setup.py
├── sites/
│   └── pronto.local/
│       ├── site_config.json
│       └── private/
│           └── files/
│               └── print-orders/           # PDF storage
│                   ├── 2024-01-15/
│                   ├── 2024-01-16/
│                   └── ...
└── docker-compose.yml
```

---

## 2. Development Environment Setup

### 2.1 Prerequisites

- Docker & Docker Compose
- Git
- Python 3.8+
- Node.js 14+

### 2.2 Docker Setup

**docker-compose.yml** (PostgreSQL on port 5435):

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: frappe
      POSTGRES_PASSWORD: frappe
      POSTGRES_DB: pronto_db
    ports:
      - "5435:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis-cache:
    image: redis:alpine
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru

  redis-queue:
    image: redis:alpine

  frappe:
    image: frappe/bench:latest
    command: bench start
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - REDIS_CACHE=redis-cache:6379
      - REDIS_QUEUE=redis-queue:6379
    ports:
      - "8000:8000"
      - "9000:9000"
    volumes:
      - ./frappe-bench:/workspace/frappe-bench
    depends_on:
      - postgres
      - redis-cache
      - redis-queue

volumes:
  postgres_data:
```

### 2.3 Frappe Bench Initialization

```bash
# Initialize bench
bench init frappe-bench --frappe-branch version-15

# Create site with PostgreSQL
cd frappe-bench
bench new-site pronto.local \
  --db-type postgres \
  --db-host localhost \
  --db-port 5435 \
  --db-name pronto_db \
  --admin-password admin

# Create Pronto app
bench new-app pronto

# Install app on site
bench --site pronto.local install-app pronto

# Set developer mode
bench --site pronto.local set-config developer_mode 1

# Start bench
bench start
```

### 2.4 PostgreSQL Configuration

**site_config.json**:
```json
{
  "db_name": "pronto_db",
  "db_password": "frappe",
  "db_type": "postgres",
  "db_host": "localhost",
  "db_port": 5435,
  "developer_mode": 1,
  "auto_migrate": true
}
```

---

## 3. Frappe App Structure

### 3.1 hooks.py Configuration

**File**: `pronto/pronto/hooks.py`

```python
from . import __version__ as app_version

app_name = "pronto"
app_title = "Pronto"
app_publisher = "Pronto Team"
app_description = "Print Order Management System"
app_email = "info@pronto.ng"
app_license = "MIT"

# Includes in <head>
app_include_css = [
    "/assets/pronto/css/pronto-theme.css"
]

app_include_js = [
    "https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"
]

# Web pages
web_include_css = [
    "/assets/pronto/css/pronto-theme.css"
]

web_include_js = [
    "https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"
]

# Home Pages
# ----------
website_route_rules = [
    {"from_route": "/dashboard", "to_route": "dashboard"},
    {"from_route": "/print-staff", "to_route": "print-staff"},
    {"from_route": "/qa-staff", "to_route": "qa-staff"},
    {"from_route": "/dispatch-staff", "to_route": "dispatch-staff"},
    {"from_route": "/collection-staff", "to_route": "collection-staff"},
    {"from_route": "/business-manager", "to_route": "business-manager"},
]

# Role-based home pages
role_home_page = {
    "Customer": "/dashboard",
    "Print Staff": "/print-staff",
    "QA Staff": "/qa-staff",
    "Dispatch Staff": "/dispatch-staff",
    "Collection Staff": "/collection-staff",
    "Business Manager": "/business-manager",
}

# Scheduled Tasks
# ---------------
scheduler_events = {
    "daily": [
        "pronto.tasks.cleanup_old_files"
    ],
    "hourly": [
        "pronto.tasks.check_payment_status"
    ]
}

# DocType Events
# --------------
doc_events = {
    "Print Order": {
        "after_insert": "pronto.pronto.doctype.print_order.print_order.create_job_queue_entry",
        "on_update": "pronto.pronto.doctype.print_order.print_order.update_customer_stats",
        "on_payment_success": "pronto.pronto.doctype.print_order.print_order.award_loyalty_points"
    },
    "Job Queue": {
        "on_update": "pronto.api.staff.notify_job_update"
    }
}

# After Install
# -------------
after_install = "pronto.fixtures.initial_data.main"

# Permissions
# -----------
permission_query_conditions = {
    "Print Order": "pronto.pronto.doctype.print_order.print_order.get_permission_query_conditions",
}

has_permission = {
    "Print Order": "pronto.pronto.doctype.print_order.print_order.has_permission",
}

# Website Settings
# ----------------
website_context = {
    "favicon": "/assets/pronto/images/favicon.png",
    "splash_image": "/assets/pronto/images/splash.png"
}

# Fixtures
# --------
fixtures = [
    {
        "dt": "Custom Field",
        "filters": [
            ["name", "in", [
                "User-assigned_print_center"
            ]]
        ]
    }
]
```

### 3.2 Initial Data Fixtures

**File**: `pronto/pronto/fixtures/initial_data.py`

```python
import frappe

def main():
    """Initialize master data after app installation"""
    create_roles()
    create_paper_sizes()
    create_paper_types()
    create_finishing_options()
    create_priority_pricing()
    create_pricing_configuration()
    create_pickup_locations()
    create_loyalty_configuration()
    frappe.db.commit()

def create_roles():
    """Create custom roles"""
    roles = [
        "Customer",
        "Print Staff",
        "QA Staff",
        "Dispatch Staff",
        "Collection Staff",
        "Customer Service",
        "Business Manager",
        "Print Shop Manager"
    ]
    
    for role_name in roles:
        if not frappe.db.exists("Role", role_name):
            role = frappe.get_doc({
                "doctype": "Role",
                "role_name": role_name,
                "desk_access": 1 if role_name != "Customer" else 0
            })
            role.insert(ignore_permissions=True)

def create_paper_sizes():
    """Create default paper sizes"""
    sizes = [
        {"size_name": "A4", "base_price": 50},
        {"size_name": "A3", "base_price": 100},
        {"size_name": "A5", "base_price": 30},
        {"size_name": "Letter", "base_price": 50},
        {"size_name": "Legal", "base_price": 60},
    ]
    
    for size in sizes:
        if not frappe.db.exists("Print Paper Size", size["size_name"]):
            doc = frappe.get_doc({
                "doctype": "Print Paper Size",
                "size_name": size["size_name"],
                "base_price": size["base_price"],
                "is_active": 1
            })
            doc.insert(ignore_permissions=True)

def create_paper_types():
    """Create default paper types"""
    types = [
        {"type_name": "Plain", "price_multiplier": 1.0},
        {"type_name": "Premium", "price_multiplier": 1.5},
        {"type_name": "Glossy", "price_multiplier": 1.8},
    ]
    
    for ptype in types:
        if not frappe.db.exists("Print Paper Type", ptype["type_name"]):
            doc = frappe.get_doc({
                "doctype": "Print Paper Type",
                "type_name": ptype["type_name"],
                "price_multiplier": ptype["price_multiplier"],
                "is_active": 1
            })
            doc.insert(ignore_permissions=True)

def create_finishing_options():
    """Create default finishing options"""
    options = [
        {"finishing_name": "Binding", "pricing_type": "Fixed Cost", "additional_cost": 100},
        {"finishing_name": "Lamination", "pricing_type": "Fixed Cost", "additional_cost": 150},
        {"finishing_name": "Stapling", "pricing_type": "Fixed Cost", "additional_cost": 20},
    ]
    
    for option in options:
        if not frappe.db.exists("Print Finishing", option["finishing_name"]):
            doc = frappe.get_doc({
                "doctype": "Print Finishing",
                **option,
                "is_active": 1
            })
            doc.insert(ignore_permissions=True)

def create_priority_pricing():
    """Create priority pricing levels"""
    priorities = [
        {"priority_level": "No Wahala", "turnaround_time": "Next Day", "additional_cost": 0},
        {"priority_level": "Normal (2 hours)", "turnaround_time": "2 Hours", "additional_cost": 0},
        {"priority_level": "Sharp Sharp", "turnaround_time": "1 Hour", "additional_cost": 200},
        {"priority_level": "Pronto", "turnaround_time": "30 Minutes", "additional_cost": 500},
    ]
    
    for priority in priorities:
        if not frappe.db.exists("Priority Pricing", priority["priority_level"]):
            doc = frappe.get_doc({
                "doctype": "Priority Pricing",
                **priority,
                "is_active": 1
            })
            doc.insert(ignore_permissions=True)

def create_pricing_configuration():
    """Create default pricing configuration"""
    if not frappe.db.exists("Pricing Configuration"):
        doc = frappe.get_doc({
            "doctype": "Pricing Configuration",
            "color_black_white_multiplier": 1.0,
            "color_color_multiplier": 1.5,
            "sides_single_multiplier": 1.0,
            "sides_double_multiplier": 1.8,
            "default_currency": "NGN",
            "minimum_order_amount": 0,
            "tax_rate": 0,
            "service_charge_rate": 0,
            "is_active": 1
        })
        doc.insert(ignore_permissions=True)

def create_pickup_locations():
    """Create default pickup locations"""
    locations = [
        {"location_name": "H-Medix Wuse 2", "address": "Wuse 2, Abuja"},
        {"location_name": "Garki Shopping Mall", "address": "Garki, Abuja"},
        {"location_name": "Wuse Market", "address": "Wuse Market, Abuja"},
    ]
    
    for location in locations:
        if not frappe.db.exists("Pickup Location", location["location_name"]):
            doc = frappe.get_doc({
                "doctype": "Pickup Location",
                **location,
                "is_active": 1
            })
            doc.insert(ignore_permissions=True)

def create_loyalty_configuration():
    """Create loyalty points configuration"""
    if not frappe.db.exists("Loyalty Configuration"):
        doc = frappe.get_doc({
            "doctype": "Loyalty Configuration",
            "points_per_currency_unit": 1,  # 1 point per ₦100
            "currency_unit_value": 100,
            "points_to_credit_ratio": 0.5,  # 100 points = ₦50
            "minimum_points_for_conversion": 100,
            "points_expiry_days": 0,  # No expiry
            "is_active": 1
        })
        doc.insert(ignore_permissions=True)
```

---

## 4. DocType Definitions

### 4.1 Print Order DocType

**File**: `pronto/pronto/doctype/print_order/print_order.json`

```json
{
  "actions": [],
  "allow_rename": 0,
  "autoname": "format:ORD-{YYYY}-{####}",
  "creation": "2024-01-01 00:00:00",
  "doctype": "DocType",
  "engine": "InnoDB",
  "field_order": [
    "customer_section",
    "customer",
    "customer_name",
    "customer_email",
    "customer_phone",
    "column_break_customer",
    "user",
    "order_date",
    "status",
    "priority",
    "print_specifications_section",
    "pdf_attachment",
    "page_count",
    "paper_size",
    "paper_type",
    "column_break_specs",
    "color_option",
    "sides",
    "orientation",
    "finishing",
    "quantity",
    "ink_analysis_section",
    "estimated_ink_coverage",
    "estimated_ink_cost",
    "pricing_section",
    "base_price",
    "paper_type_cost",
    "color_cost",
    "sides_cost",
    "finishing_cost",
    "priority_cost",
    "column_break_pricing",
    "subtotal",
    "tax_amount",
    "service_charge",
    "discount_amount",
    "coupon_code",
    "total_amount",
    "payment_section",
    "payment_status",
    "payment_method",
    "payment_reference",
    "paid_amount",
    "column_break_payment",
    "payment_date",
    "credit_used",
    "paystack_amount",
    "pickup_section",
    "pickup_location",
    "pickup_code",
    "column_break_pickup",
    "pickup_date",
    "collected_by",
    "qa_section",
    "qa_status",
    "qa_notes",
    "qa_approved_by",
    "column_break_qa",
    "qa_approved_date",
    "qa_rejection_reason",
    "timestamps_section",
    "created_at",
    "submitted_at",
    "paid_at",
    "column_break_timestamps",
    "completed_at",
    "dispatched_at",
    "collected_at"
  ],
  "fields": [
    {
      "fieldname": "customer_section",
      "fieldtype": "Section Break",
      "label": "Customer Information"
    },
    {
      "fieldname": "customer",
      "fieldtype": "Link",
      "label": "Customer",
      "options": "Pronto Customer",
      "reqd": 1
    },
    {
      "fieldname": "customer_name",
      "fieldtype": "Data",
      "label": "Customer Name",
      "reqd": 1
    },
    {
      "fieldname": "customer_email",
      "fieldtype": "Data",
      "label": "Email",
      "options": "Email",
      "reqd": 1
    },
    {
      "fieldname": "customer_phone",
      "fieldtype": "Data",
      "label": "Phone Number"
    },
    {
      "fieldname": "column_break_customer",
      "fieldtype": "Column Break"
    },
    {
      "fieldname": "user",
      "fieldtype": "Link",
      "label": "User",
      "options": "User"
    },
    {
      "fieldname": "order_date",
      "fieldtype": "Datetime",
      "label": "Order Date",
      "default": "now"
    },
    {
      "fieldname": "status",
      "fieldtype": "Select",
      "label": "Status",
      "options": "Draft\nSubmitted\nPaid\nIn Queue\nIn Progress\nCompleted\nQA Check\nQA Approved\nQA Rejected\nReady for Dispatch\nDispatched\nDelivered\nReady for Pickup\nCollected\nCancelled\nRefunded",
      "default": "Draft",
      "reqd": 1
    },
    {
      "fieldname": "priority",
      "fieldtype": "Select",
      "label": "Priority",
      "options": "No Wahala\nNormal (2 hours)\nSharp Sharp\nPronto",
      "default": "Normal (2 hours)"
    },
    {
      "fieldname": "print_specifications_section",
      "fieldtype": "Section Break",
      "label": "Print Specifications"
    },
    {
      "fieldname": "pdf_attachment",
      "fieldtype": "Attach",
      "label": "PDF File",
      "reqd": 1
    },
    {
      "fieldname": "page_count",
      "fieldtype": "Int",
      "label": "Page Count",
      "default": 1
    },
    {
      "fieldname": "paper_size",
      "fieldtype": "Link",
      "label": "Paper Size",
      "options": "Print Paper Size",
      "reqd": 1
    },
    {
      "fieldname": "paper_type",
      "fieldtype": "Link",
      "label": "Paper Type",
      "options": "Print Paper Type",
      "reqd": 1
    },
    {
      "fieldname": "column_break_specs",
      "fieldtype": "Column Break"
    },
    {
      "fieldname": "color_option",
      "fieldtype": "Select",
      "label": "Color Option",
      "options": "Black & White\nColor",
      "default": "Black & White",
      "reqd": 1
    },
    {
      "fieldname": "sides",
      "fieldtype": "Select",
      "label": "Sides",
      "options": "Single Sided\nDouble Sided",
      "default": "Single Sided",
      "reqd": 1
    },
    {
      "fieldname": "orientation",
      "fieldtype": "Select",
      "label": "Orientation",
      "options": "Portrait\nLandscape",
      "default": "Portrait"
    },
    {
      "fieldname": "finishing",
      "fieldtype": "Link",
      "label": "Finishing",
      "options": "Print Finishing"
    },
    {
      "fieldname": "quantity",
      "fieldtype": "Int",
      "label": "Quantity",
      "default": 1,
      "reqd": 1
    }
  ],
  "modified": "2024-01-01 00:00:00",
  "module": "Pronto",
  "name": "Print Order",
  "naming_rule": "Expression",
  "owner": "Administrator",
  "permissions": [
    {
      "create": 1,
      "delete": 1,
      "email": 1,
      "export": 1,
      "print": 1,
      "read": 1,
      "report": 1,
      "role": "System Manager",
      "share": 1,
      "write": 1
    },
    {
      "create": 1,
      "email": 1,
      "print": 1,
      "read": 1,
      "role": "Customer",
      "write": 1
    }
  ],
  "sort_field": "modified",
  "sort_order": "DESC",
  "states": [],
  "track_changes": 1
}
```

**Note**: The complete Print Order JSON has 60+ fields. The above shows the structure. Refer to the existing `print_order.json` for the full definition.

### 4.2 New DocTypes for v1 Features

#### 4.2.1 Print Center DocType

**File**: `pronto/pronto/doctype/print_center/print_center.json`

```json
{
  "autoname": "format:PC-{####}",
  "doctype": "DocType",
  "engine": "InnoDB",
  "field_order": [
    "center_name", "center_code", "address_section", "address_line_1",
    "address_line_2", "city", "state", "postal_code", "column_break_1",
    "phone_number", "email", "operating_hours", "details_section",
    "center_manager", "capacity", "equipment_details", "column_break_2",
    "is_active"
  ],
  "fields": [
    {"fieldname": "center_name", "fieldtype": "Data", "label": "Center Name", "reqd": 1, "unique": 1},
    {"fieldname": "center_code", "fieldtype": "Data", "label": "Center Code", "read_only": 1},
    {"fieldname": "address_section", "fieldtype": "Section Break", "label": "Address"},
    {"fieldname": "address_line_1", "fieldtype": "Data", "label": "Address Line 1"},
    {"fieldname": "address_line_2", "fieldtype": "Data", "label": "Address Line 2"},
    {"fieldname": "city", "fieldtype": "Data", "label": "City"},
    {"fieldname": "state", "fieldtype": "Data", "label": "State"},
    {"fieldname": "postal_code", "fieldtype": "Data", "label": "Postal Code"},
    {"fieldname": "column_break_1", "fieldtype": "Column Break"},
    {"fieldname": "phone_number", "fieldtype": "Data", "label": "Phone Number"},
    {"fieldname": "email", "fieldtype": "Data", "label": "Email", "options": "Email"},
    {"fieldname": "operating_hours", "fieldtype": "Text", "label": "Operating Hours"},
    {"fieldname": "details_section", "fieldtype": "Section Break", "label": "Details"},
    {"fieldname": "center_manager", "fieldtype": "Link", "label": "Center Manager", "options": "User"},
    {"fieldname": "capacity", "fieldtype": "Int", "label": "Capacity (Max Concurrent Jobs)", "default": 10},
    {"fieldname": "equipment_details", "fieldtype": "Text", "label": "Equipment Details"},
    {"fieldname": "column_break_2", "fieldtype": "Column Break"},
    {"fieldname": "is_active", "fieldtype": "Check", "label": "Is Active", "default": 1}
  ],
  "module": "Pronto",
  "name": "Print Center",
  "naming_rule": "Expression",
  "permissions": [
    {"create": 1, "delete": 1, "read": 1, "role": "System Manager", "write": 1},
    {"create": 1, "read": 1, "role": "Business Manager", "write": 1}
  ]
}
```

**File**: `pronto/pronto/doctype/print_center/print_center.py`

```python
import frappe
from frappe.model.document import Document
import random
import string

class PrintCenter(Document):
    def before_insert(self):
        """Generate unique center code"""
        if not self.center_code:
            self.center_code = self.generate_center_code()

    def generate_center_code(self):
        """Generate unique 6-character alphanumeric code"""
        while True:
            code = ''.join(random.choices(string.ascii_uppercase + string.digits, k=6))
            if not frappe.db.exists("Print Center", {"center_code": code}):
                return code
```

#### 4.2.2 Discount Coupon DocType

**File**: `pronto/pronto/doctype/discount_coupon/discount_coupon.json`

```json
{
  "autoname": "field:coupon_code",
  "doctype": "DocType",
  "engine": "InnoDB",
  "field_order": [
    "coupon_code", "coupon_name", "discount_section", "discount_type",
    "discount_value", "maximum_discount_amount", "column_break_1",
    "minimum_order_amount", "validity_section", "valid_from", "valid_to",
    "column_break_2", "usage_limit", "usage_count", "per_customer_limit",
    "applicability_section", "applicable_to", "customer_list",
    "details_section", "description", "terms_and_conditions",
    "status_section", "is_active"
  ],
  "fields": [
    {"fieldname": "coupon_code", "fieldtype": "Data", "label": "Coupon Code", "reqd": 1, "unique": 1},
    {"fieldname": "coupon_name", "fieldtype": "Data", "label": "Coupon Name", "reqd": 1},
    {"fieldname": "discount_section", "fieldtype": "Section Break", "label": "Discount"},
    {"fieldname": "discount_type", "fieldtype": "Select", "label": "Discount Type", "options": "Percentage\nFixed Amount", "reqd": 1},
    {"fieldname": "discount_value", "fieldtype": "Float", "label": "Discount Value", "reqd": 1},
    {"fieldname": "maximum_discount_amount", "fieldtype": "Currency", "label": "Maximum Discount Amount"},
    {"fieldname": "column_break_1", "fieldtype": "Column Break"},
    {"fieldname": "minimum_order_amount", "fieldtype": "Currency", "label": "Minimum Order Amount", "default": 0},
    {"fieldname": "validity_section", "fieldtype": "Section Break", "label": "Validity"},
    {"fieldname": "valid_from", "fieldtype": "Date", "label": "Valid From", "reqd": 1},
    {"fieldname": "valid_to", "fieldtype": "Date", "label": "Valid To", "reqd": 1},
    {"fieldname": "column_break_2", "fieldtype": "Column Break"},
    {"fieldname": "usage_limit", "fieldtype": "Int", "label": "Usage Limit", "default": 0},
    {"fieldname": "usage_count", "fieldtype": "Int", "label": "Usage Count", "default": 0, "read_only": 1},
    {"fieldname": "per_customer_limit", "fieldtype": "Int", "label": "Per Customer Limit", "default": 1},
    {"fieldname": "applicability_section", "fieldtype": "Section Break", "label": "Applicability"},
    {"fieldname": "applicable_to", "fieldtype": "Select", "label": "Applicable To", "options": "All Customers\nSpecific Customers", "default": "All Customers"},
    {"fieldname": "customer_list", "fieldtype": "Table", "label": "Customer List", "options": "Coupon Customer"},
    {"fieldname": "details_section", "fieldtype": "Section Break", "label": "Details"},
    {"fieldname": "description", "fieldtype": "Text", "label": "Description"},
    {"fieldname": "terms_and_conditions", "fieldtype": "Text", "label": "Terms & Conditions"},
    {"fieldname": "status_section", "fieldtype": "Section Break", "label": "Status"},
    {"fieldname": "is_active", "fieldtype": "Check", "label": "Is Active", "default": 1}
  ],
  "module": "Pronto",
  "name": "Discount Coupon",
  "naming_rule": "By fieldname"
}
```

#### 4.2.3 Customer Credit Transaction DocType

**File**: `pronto/pronto/doctype/customer_credit_transaction/customer_credit_transaction.json`

```json
{
  "autoname": "format:TXN-{YYYY}-{#####}",
  "doctype": "DocType",
  "engine": "InnoDB",
  "field_order": [
    "customer", "transaction_type", "amount", "points", "column_break_1",
    "balance_before", "balance_after", "related_order", "payment_reference",
    "notes", "audit_section", "transaction_date", "created_by"
  ],
  "fields": [
    {"fieldname": "customer", "fieldtype": "Link", "label": "Customer", "options": "Pronto Customer", "reqd": 1},
    {"fieldname": "transaction_type", "fieldtype": "Select", "label": "Transaction Type", "options": "Credit Added\nCredit Used\nPoints Converted\nRefund", "reqd": 1},
    {"fieldname": "amount", "fieldtype": "Currency", "label": "Amount", "reqd": 1},
    {"fieldname": "points", "fieldtype": "Int", "label": "Points", "default": 0},
    {"fieldname": "column_break_1", "fieldtype": "Column Break"},
    {"fieldname": "balance_before", "fieldtype": "Currency", "label": "Balance Before", "read_only": 1},
    {"fieldname": "balance_after", "fieldtype": "Currency", "label": "Balance After", "read_only": 1},
    {"fieldname": "related_order", "fieldtype": "Link", "label": "Related Order", "options": "Print Order"},
    {"fieldname": "payment_reference", "fieldtype": "Data", "label": "Payment Reference"},
    {"fieldname": "notes", "fieldtype": "Text", "label": "Notes"},
    {"fieldname": "audit_section", "fieldtype": "Section Break", "label": "Audit"},
    {"fieldname": "transaction_date", "fieldtype": "Datetime", "label": "Transaction Date", "default": "now"},
    {"fieldname": "created_by", "fieldtype": "Link", "label": "Created By", "options": "User", "default": "user"}
  ],
  "module": "Pronto",
  "name": "Customer Credit Transaction",
  "naming_rule": "Expression"
}
```

#### 4.2.4 Loyalty Configuration DocType

**File**: `pronto/pronto/doctype/loyalty_configuration/loyalty_configuration.json`

```json
{
  "autoname": "format:LOYALTY-CONFIG",
  "doctype": "DocType",
  "engine": "InnoDB",
  "is_single": 1,
  "field_order": [
    "points_earning_section", "points_per_currency_unit", "currency_unit_value",
    "column_break_1", "points_to_credit_ratio", "minimum_points_for_conversion",
    "expiry_section", "points_expiry_days", "status_section", "is_active"
  ],
  "fields": [
    {"fieldname": "points_earning_section", "fieldtype": "Section Break", "label": "Points Earning"},
    {"fieldname": "points_per_currency_unit", "fieldtype": "Int", "label": "Points Per Currency Unit", "default": 1, "description": "e.g., 1 point per ₦100"},
    {"fieldname": "currency_unit_value", "fieldtype": "Currency", "label": "Currency Unit Value", "default": 100},
    {"fieldname": "column_break_1", "fieldtype": "Column Break"},
    {"fieldname": "points_to_credit_ratio", "fieldtype": "Float", "label": "Points to Credit Ratio", "default": 0.5, "description": "e.g., 0.5 means 100 points = ₦50"},
    {"fieldname": "minimum_points_for_conversion", "fieldtype": "Int", "label": "Minimum Points for Conversion", "default": 100},
    {"fieldname": "expiry_section", "fieldtype": "Section Break", "label": "Expiry"},
    {"fieldname": "points_expiry_days", "fieldtype": "Int", "label": "Points Expiry Days", "default": 0, "description": "0 = No expiry"},
    {"fieldname": "status_section", "fieldtype": "Section Break", "label": "Status"},
    {"fieldname": "is_active", "fieldtype": "Check", "label": "Is Active", "default": 1}
  ],
  "module": "Pronto",
  "name": "Loyalty Configuration"
}
```

---

## 5. API Endpoints

### 5.1 Authentication APIs

**File**: `pronto/pronto/api/auth.py`

```python
import frappe
from frappe import _
import jwt
import requests
from datetime import datetime

@frappe.whitelist(allow_guest=True)
def google_oauth_login(credential):
    """
    Handle Google OAuth login with JWT credential

    Args:
        credential (str): JWT token from Google

    Returns:
        dict: Login status and user info
    """
    try:
        # Decode JWT token
        decoded = jwt.decode(credential, options={"verify_signature": False})

        email = decoded.get('email')
        name = decoded.get('name')
        picture = decoded.get('picture')

        if not email:
            return {"success": False, "error": "Email not found in token"}

        # Check if user exists
        if not frappe.db.exists("User", email):
            # Create new user
            user = frappe.get_doc({
                "doctype": "User",
                "email": email,
                "first_name": name.split()[0] if name else email.split('@')[0],
                "last_name": name.split()[-1] if name and len(name.split()) > 1 else "",
                "user_image": picture,
                "enabled": 1,
                "send_welcome_email": 0
            })
            user.insert(ignore_permissions=True)

            # Add Customer role
            user.add_roles("Customer")

            # Create Pronto Customer
            create_pronto_customer(email, name)

        # Login user
        frappe.local.login_manager.login_as(email)
        frappe.db.commit()

        return {
            "success": True,
            "message": "Login successful",
            "user": email,
            "redirect": "/dashboard"
        }

    except Exception as e:
        frappe.log_error(f"Google OAuth Error: {str(e)}")
        return {"success": False, "error": str(e)}

def create_pronto_customer(email, name):
    """Create Pronto Customer record"""
    if not frappe.db.exists("Pronto Customer", name):
        customer = frappe.get_doc({
            "doctype": "Pronto Customer",
            "customer_name": name or email.split('@')[0],
            "email": email,
            "user": email,
            "customer_status": "Active",
            "credit_balance": 0,
            "loyalty_points": 0
        })
        customer.insert(ignore_permissions=True)

@frappe.whitelist(allow_guest=True)
def check_google_oauth_config():
    """Check if Google OAuth is configured"""
    # Check for Google OAuth settings in site config
    client_id = frappe.conf.get('google_client_id')
    return {"configured": bool(client_id), "client_id": client_id}
```

### 5.2 Order Management APIs

**File**: `pronto/pronto/api/orders.py`

```python
import frappe
from frappe import _
import json
import random
import string
from datetime import datetime
import os

@frappe.whitelist(allow_guest=True)
def calculate_print_price(
    paper_size, paper_type, color_option, sides,
    page_count, quantity, finishing=None, priority="Normal (2 hours)",
    coupon_code=None
):
    """
    Calculate print price based on specifications

    Returns:
        dict: Price breakdown
    """
    try:
        # Get pricing configuration
        config = frappe.get_doc("Pricing Configuration", {"is_active": 1})

        # Get base price from paper size
        paper_size_doc = frappe.get_doc("Print Paper Size", paper_size)
        base_price = paper_size_doc.base_price * int(page_count) * int(quantity)

        # Apply paper type multiplier
        paper_type_doc = frappe.get_doc("Print Paper Type", paper_type)
        paper_type_cost = base_price * paper_type_doc.price_multiplier

        # Apply color multiplier
        color_multiplier = config.color_color_multiplier if color_option == "Color" else config.color_black_white_multiplier
        color_cost = paper_type_cost * color_multiplier

        # Apply sides multiplier
        sides_multiplier = config.sides_double_multiplier if sides == "Double Sided" else config.sides_single_multiplier
        sides_cost = color_cost * sides_multiplier

        # Add finishing cost
        finishing_cost = 0
        if finishing:
            finishing_doc = frappe.get_doc("Print Finishing", finishing)
            if finishing_doc.pricing_type == "Fixed Cost":
                finishing_cost = finishing_doc.additional_cost
            else:
                finishing_cost = sides_cost * finishing_doc.price_multiplier

        # Add priority cost
        priority_cost = 0
        if priority:
            priority_doc = frappe.get_doc("Priority Pricing", priority)
            priority_cost = priority_doc.additional_cost

        # Calculate subtotal
        subtotal = sides_cost + finishing_cost + priority_cost

        # Apply discount
        discount_amount = 0
        discount_info = None
        if coupon_code:
            discount_result = validate_and_calculate_discount(coupon_code, subtotal)
            if discount_result.get("valid"):
                discount_amount = discount_result.get("discount_amount", 0)
                discount_info = discount_result

        # Calculate tax and service charge
        tax_amount = (subtotal - discount_amount) * (config.tax_rate / 100)
        service_charge = (subtotal - discount_amount) * (config.service_charge_rate / 100)

        # Calculate total
        total = subtotal - discount_amount + tax_amount + service_charge

        return {
            "success": True,
            "breakdown": {
                "base_price": base_price,
                "paper_type_cost": paper_type_cost,
                "color_cost": color_cost,
                "sides_cost": sides_cost,
                "finishing_cost": finishing_cost,
                "priority_cost": priority_cost,
                "subtotal": subtotal,
                "discount_amount": discount_amount,
                "tax_amount": tax_amount,
                "service_charge": service_charge,
                "total": total
            },
            "discount_info": discount_info
        }

    except Exception as e:
        frappe.log_error(f"Price calculation error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def create_print_order_api(order_data):
    """
    Create print order with duplicate prevention

    Args:
        order_data (str): JSON string of order data

    Returns:
        dict: Order creation result
    """
    try:
        data = json.loads(order_data) if isinstance(order_data, str) else order_data

        # Duplicate prevention: Check for recent identical orders (5-second window)
        cache_key = f"order_create_{frappe.session.user}_{data.get('pdf_attachment')}"
        if frappe.cache().get(cache_key):
            return {"success": False, "error": "Duplicate order detected. Please wait before submitting again."}

        # Set cache for 5 seconds
        frappe.cache().set(cache_key, True, expires_in_sec=5)

        # Get customer
        customer = frappe.get_value("Pronto Customer", {"user": frappe.session.user}, "name")
        if not customer:
            return {"success": False, "error": "Customer record not found"}

        # Generate pickup code
        pickup_code = generate_pickup_code()

        # Create order
        order = frappe.get_doc({
            "doctype": "Print Order",
            "customer": customer,
            "customer_name": data.get("customer_name"),
            "customer_email": data.get("customer_email"),
            "customer_phone": data.get("customer_phone"),
            "user": frappe.session.user,
            "pdf_attachment": data.get("pdf_attachment"),
            "page_count": data.get("page_count", 1),
            "paper_size": data.get("paper_size"),
            "paper_type": data.get("paper_type"),
            "color_option": data.get("color_option"),
            "sides": data.get("sides"),
            "orientation": data.get("orientation", "Portrait"),
            "finishing": data.get("finishing"),
            "quantity": data.get("quantity", 1),
            "priority": data.get("priority", "Normal (2 hours)"),
            "pickup_location": data.get("pickup_location"),
            "pickup_code": pickup_code,
            "coupon_code": data.get("coupon_code"),
            "status": "Submitted",
            "payment_status": "Pending",
            # Pricing fields from calculate_print_price
            **data.get("pricing", {})
        })

        order.insert(ignore_permissions=True)
        frappe.db.commit()

        return {
            "success": True,
            "order_id": order.name,
            "pickup_code": pickup_code,
            "total_amount": order.total_amount
        }

    except Exception as e:
        frappe.log_error(f"Order creation error: {str(e)}")
        frappe.db.rollback()
        return {"success": False, "error": str(e)}

def generate_pickup_code():
    """Generate unique 4-character pickup code"""
    charset = "ABCDETQGHKWY234689"  # Excludes confusing characters
    while True:
        code = ''.join(random.choices(charset, k=4))
        if not frappe.db.exists("Print Order", {"pickup_code": code, "status": ["!=", "Collected"]}):
            return code

@frappe.whitelist()
def upload_and_analyze_pdf():
    """
    Handle PDF upload and analysis

    Returns:
        dict: Upload result with file URL and page count
    """
    try:
        files = frappe.request.files
        if 'file' not in files:
            return {"success": False, "error": "No file uploaded"}

        file = files['file']

        # Validate file type
        if not file.filename.lower().endswith('.pdf'):
            return {"success": False, "error": "Only PDF files are allowed"}

        # Validate file size (10MB limit)
        file.seek(0, os.SEEK_END)
        file_size = file.tell()
        file.seek(0)

        if file_size > 10 * 1024 * 1024:  # 10MB
            return {"success": False, "error": "File size exceeds 10MB limit"}

        # Read file content
        file_content = file.read()

        # Extract page count
        page_count = extract_pdf_page_count(file_content)

        # Analyze ink coverage
        ink_analysis = analyze_pdf_comprehensive(file_content)

        # Save file with date-based folder structure
        today = datetime.now().strftime("%Y-%m-%d")
        timestamp = datetime.now().strftime("%H%M%S")
        filename = f"{timestamp}_{file.filename}"

        file_doc = frappe.get_doc({
            "doctype": "File",
            "file_name": filename,
            "folder": f"Home/print-orders/{today}",
            "is_private": 1,
            "content": file_content
        })
        file_doc.save(ignore_permissions=True)

        return {
            "success": True,
            "file_url": file_doc.file_url,
            "page_count": page_count,
            "ink_coverage": ink_analysis.get("coverage_percentage", 0),
            "ink_cost": ink_analysis.get("estimated_cost", 0)
        }

    except Exception as e:
        frappe.log_error(f"PDF upload error: {str(e)}")
        return {"success": False, "error": str(e)}

def extract_pdf_page_count(file_content):
    """Extract page count from PDF"""
    try:
        # Try pypdf first
        try:
            from pypdf import PdfReader
            from io import BytesIO
            pdf = PdfReader(BytesIO(file_content))
            return len(pdf.pages)
        except ImportError:
            pass

        # Try PyPDF2
        try:
            from PyPDF2 import PdfReader
            from io import BytesIO
            pdf = PdfReader(BytesIO(file_content))
            return len(pdf.pages)
        except ImportError:
            pass

        # Fallback: Regex-based parsing
        import re
        content_str = file_content.decode('latin-1', errors='ignore')
        matches = re.findall(r'/Count\s+(\d+)', content_str)
        if matches:
            return int(matches[0])

        return 1  # Default

    except Exception as e:
        frappe.log_error(f"Page count extraction error: {str(e)}")
        return 1

def analyze_pdf_comprehensive(file_content):
    """Analyze PDF for ink coverage estimation"""
    try:
        # Simple text-based analysis
        content_str = file_content.decode('latin-1', errors='ignore')

        # Count words and lines as proxy for ink coverage
        word_count = len(content_str.split())
        line_count = content_str.count('\n')

        # Estimate coverage (0-85%)
        coverage = min(85, (word_count / 500) * 50 + (line_count / 50) * 35)

        # Estimate cost based on coverage tiers
        if coverage < 20:
            cost = 10
        elif coverage < 50:
            cost = 25
        elif coverage < 70:
            cost = 40
        else:
            cost = 60

        return {
            "coverage_percentage": round(coverage, 2),
            "estimated_cost": cost
        }

    except Exception as e:
        frappe.log_error(f"Ink analysis error: {str(e)}")
        return {"coverage_percentage": 50, "estimated_cost": 25}

def validate_and_calculate_discount(coupon_code, order_amount):
    """Validate coupon and calculate discount"""
    try:
        coupon = frappe.get_doc("Discount Coupon", coupon_code.upper())

        # Check if active
        if not coupon.is_active:
            return {"valid": False, "error": "Coupon is not active"}

        # Check date validity
        from datetime import date
        today = date.today()
        if today < coupon.valid_from or today > coupon.valid_to:
            return {"valid": False, "error": "Coupon has expired or not yet valid"}

        # Check usage limit
        if coupon.usage_limit > 0 and coupon.usage_count >= coupon.usage_limit:
            return {"valid": False, "error": "Coupon usage limit reached"}

        # Check per-customer limit
        customer = frappe.get_value("Pronto Customer", {"user": frappe.session.user}, "name")
        customer_usage = frappe.db.count("Coupon Usage Log", {
            "coupon_code": coupon_code,
            "customer": customer
        })
        if coupon.per_customer_limit > 0 and customer_usage >= coupon.per_customer_limit:
            return {"valid": False, "error": "You have reached the usage limit for this coupon"}

        # Check minimum order amount
        if order_amount < coupon.minimum_order_amount:
            return {"valid": False, "error": f"Minimum order amount is ₦{coupon.minimum_order_amount}"}

        # Check customer eligibility
        if coupon.applicable_to == "Specific Customers":
            eligible = frappe.db.exists("Coupon Customer", {
                "parent": coupon_code,
                "customer": customer
            })
            if not eligible:
                return {"valid": False, "error": "This coupon is not applicable to your account"}

        # Calculate discount
        if coupon.discount_type == "Percentage":
            discount = order_amount * (coupon.discount_value / 100)
            if coupon.maximum_discount_amount > 0:
                discount = min(discount, coupon.maximum_discount_amount)
        else:  # Fixed Amount
            discount = coupon.discount_value

        return {
            "valid": True,
            "discount_amount": discount,
            "coupon_name": coupon.coupon_name,
            "discount_type": coupon.discount_type,
            "discount_value": coupon.discount_value
        }

    except frappe.DoesNotExistError:
        return {"valid": False, "error": "Invalid coupon code"}
    except Exception as e:
        frappe.log_error(f"Coupon validation error: {str(e)}")
        return {"valid": False, "error": "Error validating coupon"}
```

### 5.3 Payment APIs

**File**: `pronto/pronto/api/payment.py`

```python
import frappe
from frappe import _
import requests
import hashlib
import hmac
import json
import random
import string
from datetime import datetime

@frappe.whitelist()
def initialize_payment(order_id):
    """
    Initialize Paystack payment for an order

    Args:
        order_id (str): Print Order ID

    Returns:
        dict: Payment initialization result with authorization URL
    """
    try:
        order = frappe.get_doc("Print Order", order_id)

        # Check if order belongs to current user
        if order.user != frappe.session.user:
            frappe.throw(_("Unauthorized access"))

        # Generate unique payment reference
        timestamp = int(datetime.now().timestamp() * 1000)
        random_suffix = ''.join(random.choices(string.ascii_uppercase + string.digits, k=6))
        reference = f"{order_id}-{timestamp}-{random_suffix}"

        # Get Paystack secret key from site config
        secret_key = frappe.conf.get('paystack_secret_key')
        if not secret_key:
            frappe.throw(_("Paystack not configured"))

        # Prepare payment data
        amount_kobo = int(order.total_amount * 100)  # Convert to kobo

        payload = {
            "email": order.customer_email,
            "amount": amount_kobo,
            "reference": reference,
            "callback_url": f"{frappe.utils.get_url()}/api/method/pronto.api.payment.payment_callback",
            "metadata": {
                "order_id": order_id,
                "customer_name": order.customer_name,
                "custom_fields": [
                    {
                        "display_name": "Order ID",
                        "variable_name": "order_id",
                        "value": order_id
                    }
                ]
            }
        }

        # Call Paystack API
        headers = {
            "Authorization": f"Bearer {secret_key}",
            "Content-Type": "application/json"
        }

        response = requests.post(
            "https://api.paystack.co/transaction/initialize",
            json=payload,
            headers=headers
        )

        result = response.json()

        if result.get("status"):
            # Update order with payment reference
            order.payment_reference = reference
            order.payment_status = "Processing"
            order.save(ignore_permissions=True)
            frappe.db.commit()

            return {
                "success": True,
                "authorization_url": result["data"]["authorization_url"],
                "reference": reference
            }
        else:
            return {
                "success": False,
                "error": result.get("message", "Payment initialization failed")
            }

    except Exception as e:
        frappe.log_error(f"Payment initialization error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist(allow_guest=True)
def payment_callback():
    """Handle Paystack payment callback"""
    try:
        reference = frappe.form_dict.get('reference')

        if not reference:
            frappe.respond_as_web_page(
                _("Payment Error"),
                _("No payment reference provided"),
                indicator_color='red'
            )
            return

        # Verify payment
        verification = verify_payment(reference)

        if verification.get("success") and verification.get("status") == "success":
            order_id = verification.get("order_id")

            # Redirect to order page with success message
            frappe.local.response["type"] = "redirect"
            frappe.local.response["location"] = f"/dashboard?order={order_id}&payment=success"
        else:
            frappe.respond_as_web_page(
                _("Payment Failed"),
                _("Payment verification failed. Please contact support."),
                indicator_color='red'
            )

    except Exception as e:
        frappe.log_error(f"Payment callback error: {str(e)}")
        frappe.respond_as_web_page(
            _("Payment Error"),
            _("An error occurred. Please contact support."),
            indicator_color='red'
        )

@frappe.whitelist(allow_guest=True)
def payment_webhook():
    """Handle Paystack webhook notifications"""
    try:
        # Get webhook payload
        payload = frappe.request.get_data(as_text=True)

        # Verify webhook signature
        secret_key = frappe.conf.get('paystack_secret_key')
        signature = frappe.request.headers.get('X-Paystack-Signature')

        computed_signature = hmac.new(
            secret_key.encode('utf-8'),
            payload.encode('utf-8'),
            hashlib.sha512
        ).hexdigest()

        if signature != computed_signature:
            frappe.throw(_("Invalid webhook signature"))

        # Parse payload
        data = json.loads(payload)
        event = data.get('event')

        if event == 'charge.success':
            handle_successful_payment(data['data'])
        elif event == 'charge.failed':
            handle_failed_payment(data['data'])

        return {"status": "success"}

    except Exception as e:
        frappe.log_error(f"Webhook error: {str(e)}")
        return {"status": "error", "message": str(e)}

def verify_payment(reference):
    """Verify payment with Paystack"""
    try:
        secret_key = frappe.conf.get('paystack_secret_key')

        headers = {
            "Authorization": f"Bearer {secret_key}"
        }

        response = requests.get(
            f"https://api.paystack.co/transaction/verify/{reference}",
            headers=headers
        )

        result = response.json()

        if result.get("status") and result["data"]["status"] == "success":
            # Extract order ID from reference
            order_id = reference.split('-')[0] + '-' + reference.split('-')[1] + '-' + reference.split('-')[2]

            # Update order
            order = frappe.get_doc("Print Order", order_id)
            order.payment_status = "Paid"
            order.status = "Paid"
            order.paid_amount = result["data"]["amount"] / 100  # Convert from kobo
            order.payment_date = datetime.now()
            order.paid_at = datetime.now()
            order.save(ignore_permissions=True)
            frappe.db.commit()

            return {
                "success": True,
                "status": "success",
                "order_id": order_id,
                "amount": result["data"]["amount"] / 100
            }
        else:
            return {
                "success": False,
                "status": result["data"]["status"],
                "message": result.get("message")
            }

    except Exception as e:
        frappe.log_error(f"Payment verification error: {str(e)}")
        return {"success": False, "error": str(e)}

def handle_successful_payment(payment_data):
    """Handle successful payment webhook"""
    reference = payment_data.get('reference')
    order_id = reference.split('-')[0] + '-' + reference.split('-')[1] + '-' + reference.split('-')[2]

    order = frappe.get_doc("Print Order", order_id)
    if order.payment_status != "Paid":
        order.payment_status = "Paid"
        order.status = "Paid"
        order.paid_amount = payment_data["amount"] / 100
        order.payment_date = datetime.now()
        order.paid_at = datetime.now()
        order.save(ignore_permissions=True)
        frappe.db.commit()

def handle_failed_payment(payment_data):
    """Handle failed payment webhook"""
    reference = payment_data.get('reference')
    order_id = reference.split('-')[0] + '-' + reference.split('-')[1] + '-' + reference.split('-')[2]

    order = frappe.get_doc("Print Order", order_id)
    order.payment_status = "Failed"
    order.save(ignore_permissions=True)
    frappe.db.commit()
```

### 5.4 Credit System APIs (NEW)

**File**: `pronto/pronto/api/credit.py`

```python
import frappe
from frappe import _
import json

@frappe.whitelist()
def get_credit_balance():
    """Get current user's credit balance and loyalty points"""
    try:
        customer = frappe.get_value("Pronto Customer", {"user": frappe.session.user},
                                    ["name", "credit_balance", "loyalty_points"], as_dict=True)

        if not customer:
            return {"success": False, "error": "Customer not found"}

        return {
            "success": True,
            "credit_balance": customer.credit_balance,
            "loyalty_points": customer.loyalty_points
        }

    except Exception as e:
        frappe.log_error(f"Get credit balance error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def add_credit(amount, payment_reference=None, notes=None):
    """
    Add credit to customer account

    Args:
        amount (float): Amount to add
        payment_reference (str): Payment reference (if paid via Paystack)
        notes (str): Optional notes

    Returns:
        dict: Transaction result
    """
    try:
        customer_name = frappe.get_value("Pronto Customer", {"user": frappe.session.user}, "name")
        if not customer_name:
            return {"success": False, "error": "Customer not found"}

        customer = frappe.get_doc("Pronto Customer", customer_name)

        # Create transaction
        transaction = frappe.get_doc({
            "doctype": "Customer Credit Transaction",
            "customer": customer_name,
            "transaction_type": "Credit Added",
            "amount": float(amount),
            "balance_before": customer.credit_balance,
            "balance_after": customer.credit_balance + float(amount),
            "payment_reference": payment_reference,
            "notes": notes,
            "created_by": frappe.session.user
        })
        transaction.insert(ignore_permissions=True)

        # Update customer balance
        customer.credit_balance += float(amount)
        customer.total_credit_added += float(amount)
        customer.save(ignore_permissions=True)

        frappe.db.commit()

        return {
            "success": True,
            "new_balance": customer.credit_balance,
            "transaction_id": transaction.name
        }

    except Exception as e:
        frappe.log_error(f"Add credit error: {str(e)}")
        frappe.db.rollback()
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def use_credit(order_id, amount):
    """
    Use credit for an order

    Args:
        order_id (str): Print Order ID
        amount (float): Amount to deduct

    Returns:
        dict: Transaction result
    """
    try:
        customer_name = frappe.get_value("Pronto Customer", {"user": frappe.session.user}, "name")
        if not customer_name:
            return {"success": False, "error": "Customer not found"}

        customer = frappe.get_doc("Pronto Customer", customer_name)

        # Check sufficient balance
        if customer.credit_balance < float(amount):
            return {"success": False, "error": "Insufficient credit balance"}

        # Create transaction
        transaction = frappe.get_doc({
            "doctype": "Customer Credit Transaction",
            "customer": customer_name,
            "transaction_type": "Credit Used",
            "amount": -float(amount),
            "balance_before": customer.credit_balance,
            "balance_after": customer.credit_balance - float(amount),
            "related_order": order_id,
            "created_by": frappe.session.user
        })
        transaction.insert(ignore_permissions=True)

        # Update customer balance
        customer.credit_balance -= float(amount)
        customer.total_credit_used += float(amount)
        customer.save(ignore_permissions=True)

        frappe.db.commit()

        return {
            "success": True,
            "new_balance": customer.credit_balance,
            "transaction_id": transaction.name
        }

    except Exception as e:
        frappe.log_error(f"Use credit error: {str(e)}")
        frappe.db.rollback()
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def convert_points_to_credit(points):
    """
    Convert loyalty points to credit

    Args:
        points (int): Points to convert

    Returns:
        dict: Conversion result
    """
    try:
        # Get loyalty configuration
        config = frappe.get_single("Loyalty Configuration")

        if not config.is_active:
            return {"success": False, "error": "Loyalty program is not active"}

        points = int(points)

        # Check minimum points
        if points < config.minimum_points_for_conversion:
            return {"success": False, "error": f"Minimum {config.minimum_points_for_conversion} points required"}

        # Get customer
        customer_name = frappe.get_value("Pronto Customer", {"user": frappe.session.user}, "name")
        if not customer_name:
            return {"success": False, "error": "Customer not found"}

        customer = frappe.get_doc("Pronto Customer", customer_name)

        # Check sufficient points
        if customer.loyalty_points < points:
            return {"success": False, "error": "Insufficient loyalty points"}

        # Calculate credit amount
        credit_amount = points * config.points_to_credit_ratio

        # Create transaction
        transaction = frappe.get_doc({
            "doctype": "Customer Credit Transaction",
            "customer": customer_name,
            "transaction_type": "Points Converted",
            "amount": credit_amount,
            "points": -points,
            "balance_before": customer.credit_balance,
            "balance_after": customer.credit_balance + credit_amount,
            "notes": f"Converted {points} points to ₦{credit_amount}",
            "created_by": frappe.session.user
        })
        transaction.insert(ignore_permissions=True)

        # Update customer
        customer.loyalty_points -= points
        customer.credit_balance += credit_amount
        customer.total_credit_added += credit_amount
        customer.save(ignore_permissions=True)

        frappe.db.commit()

        return {
            "success": True,
            "credit_added": credit_amount,
            "new_balance": customer.credit_balance,
            "remaining_points": customer.loyalty_points
        }

    except Exception as e:
        frappe.log_error(f"Convert points error: {str(e)}")
        frappe.db.rollback()
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def award_loyalty_points(order_id):
    """
    Award loyalty points for completed order

    Args:
        order_id (str): Print Order ID
    """
    try:
        # Get loyalty configuration
        config = frappe.get_single("Loyalty Configuration")

        if not config.is_active:
            return

        # Get order
        order = frappe.get_doc("Print Order", order_id)

        # Calculate points
        points = int(order.total_amount / config.currency_unit_value) * config.points_per_currency_unit

        if points > 0:
            # Get customer
            customer = frappe.get_doc("Pronto Customer", order.customer)

            # Update points
            customer.loyalty_points += points
            customer.save(ignore_permissions=True)

            frappe.db.commit()

    except Exception as e:
        frappe.log_error(f"Award loyalty points error: {str(e)}")

@frappe.whitelist()
def get_credit_history(limit=50):
    """Get customer's credit transaction history"""
    try:
        customer_name = frappe.get_value("Pronto Customer", {"user": frappe.session.user}, "name")
        if not customer_name:
            return {"success": False, "error": "Customer not found"}

        transactions = frappe.get_all(
            "Customer Credit Transaction",
            filters={"customer": customer_name},
            fields=["name", "transaction_type", "amount", "points", "balance_after",
                   "related_order", "transaction_date", "notes"],
            order_by="transaction_date desc",
            limit=int(limit)
        )

        return {
            "success": True,
            "transactions": transactions
        }

    except Exception as e:
        frappe.log_error(f"Get credit history error: {str(e)}")
        return {"success": False, "error": str(e)}
```

---

## 6. Authentication Implementation

### 6.1 CSRF Token Handling for Custom www Pages

**Critical**: All custom www pages created without Jinja templates must manually include CSRF tokens for POST requests.

**Example**: `pronto/www/dashboard.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard - Pronto</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>
</head>
<body>
    <div id="app"></div>

    <script>
        // Get CSRF token from cookie
        function getCookie(name) {
            const value = `; ${document.cookie}`;
            const parts = value.split(`; ${name}=`);
            if (parts.length === 2) return parts.pop().split(';').shift();
        }

        // Set CSRF token for all Frappe API calls
        frappe.csrf_token = getCookie('csrf_token');

        // Example API call with CSRF token
        function createOrder(orderData) {
            return frappe.call({
                method: 'pronto.api.orders.create_print_order_api',
                args: {
                    order_data: JSON.stringify(orderData)
                },
                callback: function(r) {
                    if (r.message.success) {
                        console.log('Order created:', r.message.order_id);
                    }
                }
            });
        }
    </script>
</body>
</html>
```

### 6.2 Google OAuth Configuration

**site_config.json**:
```json
{
  "google_client_id": "YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com",
  "google_client_secret": "YOUR_GOOGLE_CLIENT_SECRET"
}
```

**Login Page Integration**:
```html
<script src="https://accounts.google.com/gsi/client" async defer></script>

<div id="g_id_onload"
     data-client_id="YOUR_GOOGLE_CLIENT_ID"
     data-callback="handleGoogleLogin">
</div>

<script>
function handleGoogleLogin(response) {
    frappe.call({
        method: 'pronto.api.auth.google_oauth_login',
        args: {
            credential: response.credential
        },
        callback: function(r) {
            if (r.message.success) {
                window.location.href = r.message.redirect;
            } else {
                alert('Login failed: ' + r.message.error);
            }
        }
    });
}
</script>
```

---

## 7. Payment Integration

### 7.1 Paystack Configuration

**site_config.json**:
```json
{
  "paystack_secret_key": "sk_test_YOUR_SECRET_KEY",
  "paystack_public_key": "pk_test_YOUR_PUBLIC_KEY"
}
```

### 7.2 Payment Flow Implementation

**Frontend Payment Initialization**:
```javascript
function initiatePayment(orderId) {
    frappe.call({
        method: 'pronto.api.payment.initialize_payment',
        args: {
            order_id: orderId
        },
        callback: function(r) {
            if (r.message.success) {
                // Redirect to Paystack payment page
                window.location.href = r.message.authorization_url;
            } else {
                alert('Payment initialization failed: ' + r.message.error);
            }
        }
    });
}
```

### 7.3 Webhook Setup

**Paystack Dashboard Configuration**:
- Webhook URL: `https://your-domain.com/api/method/pronto.api.payment.payment_webhook`
- Events to subscribe: `charge.success`, `charge.failed`

---

## 8. File Management

### 8.1 Date-Based Folder Structure

**Implementation in `orders.py`**:
```python
from datetime import datetime
import os

def save_pdf_file(file_content, filename):
    """Save PDF with date-based folder structure"""
    # Create folder path: /files/print-orders/YYYY-MM-DD/
    today = datetime.now().strftime("%Y-%m-%d")
    folder_path = f"print-orders/{today}"

    # Ensure folder exists
    full_path = frappe.get_site_path('private', 'files', folder_path)
    os.makedirs(full_path, exist_ok=True)

    # Generate unique filename with timestamp
    timestamp = datetime.now().strftime("%H%M%S")
    unique_filename = f"{timestamp}_{filename}"

    # Save file
    file_path = os.path.join(full_path, unique_filename)
    with open(file_path, 'wb') as f:
        f.write(file_content)

    # Create Frappe File record
    file_doc = frappe.get_doc({
        "doctype": "File",
        "file_name": unique_filename,
        "folder": f"Home/{folder_path}",
        "is_private": 1,
        "file_url": f"/private/files/{folder_path}/{unique_filename}"
    })
    file_doc.insert(ignore_permissions=True)

    return file_doc.file_url
```

### 8.2 File Cleanup Task

**File**: `pronto/pronto/tasks.py`

```python
import frappe
from datetime import datetime, timedelta
import os

def cleanup_old_files():
    """Delete PDF files older than 90 days"""
    try:
        cutoff_date = datetime.now() - timedelta(days=90)

        # Get old files
        old_files = frappe.get_all(
            "File",
            filters={
                "folder": ["like", "%print-orders%"],
                "creation": ["<", cutoff_date]
            },
            fields=["name", "file_url"]
        )

        for file_doc in old_files:
            # Delete physical file
            file_path = frappe.get_site_path('private', file_doc.file_url.lstrip('/private/'))
            if os.path.exists(file_path):
                os.remove(file_path)

            # Delete File record
            frappe.delete_doc("File", file_doc.name, ignore_permissions=True)

        frappe.db.commit()
        frappe.logger().info(f"Cleaned up {len(old_files)} old files")

    except Exception as e:
        frappe.log_error(f"File cleanup error: {str(e)}")
```

---

## 9. Frontend Implementation

### 9.1 Centralized Theme CSS

**File**: `pronto/public/css/pronto-theme.css`

```css
:root {
    --primary: #5400D3;
    --primary-dark: #4200A8;
    --secondary: #60D701;
    --secondary-dark: #4CAD01;
    --success: #22c55e;
    --warning: #f59e0b;
    --danger: #ef4444;
    --dark: #1f2937;
    --light: #f8fafc;
    --border-radius: 0.5rem;
    --box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

body {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.btn-primary {
    background-color: var(--primary);
    border-color: var(--primary);
    color: white;
}

.btn-primary:hover {
    background-color: var(--primary-dark);
    border-color: var(--primary-dark);
}

.btn-secondary {
    background-color: var(--secondary);
    border-color: var(--secondary);
    color: white;
}

.btn-secondary:hover {
    background-color: var(--secondary-dark);
    border-color: var(--secondary-dark);
}

.text-primary {
    color: var(--primary) !important;
}

.text-secondary {
    color: var(--secondary) !important;
}

.bg-primary {
    background-color: var(--primary) !important;
}

.bg-secondary {
    background-color: var(--secondary) !important;
}

/* Card styles */
.card {
    border-radius: var(--border-radius);
    box-shadow: var(--box-shadow);
    border: 1px solid #e5e7eb;
}

.card:hover {
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

/* Badge styles */
.badge-primary {
    background-color: var(--primary);
}

.badge-secondary {
    background-color: var(--secondary);
}

/* Status badges */
.status-paid { background-color: #22c55e; color: white; }
.status-pending { background-color: #f59e0b; color: white; }
.status-completed { background-color: #3b82f6; color: white; }
.status-cancelled { background-color: #ef4444; color: white; }
```

### 9.2 Lucide Icons Integration

**Include in all pages**:
```html
<script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>

<script>
    // Initialize Lucide icons after DOM load
    document.addEventListener('DOMContentLoaded', function() {
        lucide.createIcons();
    });
</script>
```

**Usage Examples**:
```html
<!-- Printer icon -->
<i data-lucide="printer"></i>

<!-- User icon -->
<i data-lucide="user"></i>

<!-- Check icon -->
<i data-lucide="check"></i>

<!-- X icon -->
<i data-lucide="x"></i>

<!-- Download icon -->
<i data-lucide="download"></i>

<!-- Upload icon -->
<i data-lucide="upload"></i>
```

### 9.3 Real-time Notifications Implementation

**File**: `pronto/public/js/notifications.js`

```javascript
class ProntoNotifications {
    constructor() {
        this.soundEnabled = localStorage.getItem('pronto_sound_enabled') !== 'false';
        this.notificationSound = new Audio('/assets/pronto/sounds/notification.mp3');
        this.initializeRealtime();
    }

    initializeRealtime() {
        // Subscribe to Frappe realtime events
        frappe.realtime.on('new_print_job', (data) => {
            this.showNotification('New Print Job', `Job ${data.job_id} assigned to you`);
            this.playSound();
            this.updateJobQueue();
        });

        frappe.realtime.on('job_status_changed', (data) => {
            this.showNotification('Job Status Updated', `Job ${data.job_id} is now ${data.status}`);
        });
    }

    showNotification(title, message) {
        // Browser notification
        if ('Notification' in window && Notification.permission === 'granted') {
            new Notification(title, {
                body: message,
                icon: '/assets/pronto/images/logo.png'
            });
        }

        // Toast notification
        this.showToast(title, message);
    }

    showToast(title, message) {
        const toast = document.createElement('div');
        toast.className = 'toast-notification';
        toast.innerHTML = `
            <div class="toast-header">
                <i data-lucide="bell"></i>
                <strong>${title}</strong>
            </div>
            <div class="toast-body">${message}</div>
        `;

        document.body.appendChild(toast);
        lucide.createIcons();

        setTimeout(() => {
            toast.classList.add('show');
        }, 100);

        setTimeout(() => {
            toast.classList.remove('show');
            setTimeout(() => toast.remove(), 300);
        }, 5000);
    }

    playSound() {
        if (this.soundEnabled) {
            this.notificationSound.play().catch(e => console.log('Sound play failed:', e));
        }
    }

    toggleSound() {
        this.soundEnabled = !this.soundEnabled;
        localStorage.setItem('pronto_sound_enabled', this.soundEnabled);
    }

    requestPermission() {
        if ('Notification' in window && Notification.permission === 'default') {
            Notification.requestPermission();
        }
    }

    updateJobQueue() {
        // Refresh job queue data
        if (typeof refreshJobQueue === 'function') {
            refreshJobQueue();
        }
    }
}

// Initialize on page load
let prontoNotifications;
document.addEventListener('DOMContentLoaded', function() {
    prontoNotifications = new ProntoNotifications();
    prontoNotifications.requestPermission();
});
```

**CSS for Toast Notifications**:
```css
.toast-notification {
    position: fixed;
    top: 20px;
    right: 20px;
    background: white;
    border-radius: 0.5rem;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
    padding: 1rem;
    min-width: 300px;
    opacity: 0;
    transform: translateX(400px);
    transition: all 0.3s ease;
    z-index: 9999;
}

.toast-notification.show {
    opacity: 1;
    transform: translateX(0);
}

.toast-header {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
    color: var(--primary);
}

.toast-body {
    color: #6b7280;
}
```

---

## 10. New Features Implementation

### 10.1 Print Center Management

**API**: `pronto/api/manager.py`

```python
import frappe
from frappe import _

@frappe.whitelist()
def get_print_centers():
    """Get all print centers"""
    try:
        centers = frappe.get_all(
            "Print Center",
            fields=["name", "center_name", "center_code", "city", "state",
                   "center_manager", "capacity", "is_active"],
            order_by="center_name"
        )

        return {"success": True, "centers": centers}

    except Exception as e:
        frappe.log_error(f"Get print centers error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def assign_staff_to_center(user, print_center):
    """Assign staff member to print center"""
    try:
        # Add custom field to User if not exists
        if not frappe.db.exists("Custom Field", "User-assigned_print_center"):
            frappe.get_doc({
                "doctype": "Custom Field",
                "dt": "User",
                "fieldname": "assigned_print_center",
                "label": "Assigned Print Center",
                "fieldtype": "Link",
                "options": "Print Center",
                "insert_after": "email"
            }).insert(ignore_permissions=True)

        # Update user
        user_doc = frappe.get_doc("User", user)
        user_doc.assigned_print_center = print_center
        user_doc.save(ignore_permissions=True)

        frappe.db.commit()

        return {"success": True, "message": "Staff assigned successfully"}

    except Exception as e:
        frappe.log_error(f"Assign staff error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def get_staff_by_center(print_center):
    """Get all staff assigned to a print center"""
    try:
        staff = frappe.get_all(
            "User",
            filters={"assigned_print_center": print_center, "enabled": 1},
            fields=["name", "full_name", "email", "user_image"],
            order_by="full_name"
        )

        return {"success": True, "staff": staff}

    except Exception as e:
        frappe.log_error(f"Get staff by center error: {str(e)}")
        return {"success": False, "error": str(e)}
```

### 10.2 Job Routing by Print Center

**Update**: `pronto/api/staff.py`

```python
@frappe.whitelist(allow_guest=False)
def get_print_job_queue():
    """Get print job queue filtered by staff's assigned print center"""
    try:
        user_roles = frappe.get_roles(frappe.session.user)
        if "Print Staff" not in user_roles and "System Manager" not in user_roles:
            frappe.throw(_("Access denied. Print Staff role required."))

        # Get staff's assigned print center
        assigned_center = frappe.db.get_value("User", frappe.session.user, "assigned_print_center")

        # Build filters
        filters = {}
        if assigned_center:
            filters["assigned_print_center"] = assigned_center

        # Get job queue entries
        jobs = frappe.get_all(
            "Job Queue",
            fields=["name", "print_order", "queue_status", "priority", "assigned_to",
                   "assigned_print_center", "notes", "creation", "modified"],
            filters=filters,
            order_by="priority desc, creation asc",
            limit=100
        )

        # Enrich with order details
        for job in jobs:
            if job.print_order:
                order = frappe.get_doc("Print Order", job.print_order)
                job.update({
                    "customer_name": order.customer_name,
                    "paper_size": order.paper_size,
                    "color_option": order.color_option,
                    "sides": order.sides,
                    "quantity": order.quantity,
                    "pdf_attachment": order.pdf_attachment,
                    "pickup_code": order.pickup_code
                })

        return {"success": True, "jobs": jobs}

    except Exception as e:
        frappe.log_error(f"Error getting print job queue: {str(e)}")
        return {"success": False, "error": str(e)}
```

### 10.3 Coupon Management

**Business Manager Interface** (JavaScript):

```javascript
function createCoupon(couponData) {
    frappe.call({
        method: 'frappe.client.insert',
        args: {
            doc: {
                doctype: 'Discount Coupon',
                ...couponData
            }
        },
        callback: function(r) {
            if (r.message) {
                alert('Coupon created successfully!');
                refreshCouponList();
            }
        }
    });
}

function validateCoupon(couponCode, orderAmount) {
    frappe.call({
        method: 'pronto.api.orders.validate_and_calculate_discount',
        args: {
            coupon_code: couponCode,
            order_amount: orderAmount
        },
        callback: function(r) {
            if (r.message.valid) {
                applyDiscount(r.message.discount_amount);
            } else {
                alert('Invalid coupon: ' + r.message.error);
            }
        }
    });
}
```

---

## 11. Local Print Utility

### 11.1 Project Structure

```
pronto_print_utility/
├── config.json
├── main.py
├── printer.py
├── pdf_processor.py
├── api_client.py
├── requirements.txt
└── README.md
```

### 11.2 Configuration File

**File**: `config.json`

```json
{
  "frappe_url": "https://pronto.example.com",
  "api_key": "your-api-key",
  "api_secret": "your-api-secret",
  "print_center_id": "PC-0001",
  "poll_interval_seconds": 30,
  "printer_name": "HP LaserJet Pro",
  "download_folder": "./downloads",
  "log_level": "INFO",
  "max_retries": 3,
  "retry_delay_seconds": 5
}
```

### 11.3 Main Application

**File**: `main.py`

```python
import json
import logging
import time
from apscheduler.schedulers.blocking import BlockingScheduler
from api_client import ProntoAPIClient
from pdf_processor import PDFProcessor
from printer import PrinterManager

# Load configuration
with open('config.json', 'r') as f:
    config = json.load(f)

# Setup logging
logging.basicConfig(
    level=getattr(logging, config['log_level']),
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('print_utility.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

# Initialize components
api_client = ProntoAPIClient(
    config['frappe_url'],
    config['api_key'],
    config['api_secret']
)
pdf_processor = PDFProcessor(config['download_folder'])
printer_manager = PrinterManager(config['printer_name'])

def poll_and_process_jobs():
    """Poll for new jobs and process them"""
    try:
        logger.info("Polling for new print jobs...")

        # Get pending jobs
        jobs = api_client.get_pending_jobs(config['print_center_id'])

        if not jobs:
            logger.info("No pending jobs found")
            return

        logger.info(f"Found {len(jobs)} pending job(s)")

        for job in jobs:
            process_job(job)

    except Exception as e:
        logger.error(f"Error polling jobs: {str(e)}")

def process_job(job):
    """Process a single print job"""
    job_id = job['name']
    logger.info(f"Processing job {job_id}")

    try:
        # Update status to downloading
        api_client.update_job_status(job_id, 'downloading', 'Downloading PDF')

        # Download PDF
        pdf_path = api_client.download_job_pdf(job_id)
        logger.info(f"Downloaded PDF to {pdf_path}")

        # Update status to processing
        api_client.update_job_status(job_id, 'processing', 'Generating spec sheets')

        # Process PDF (add spec sheets)
        final_pdf_path = pdf_processor.process_job(job, pdf_path)
        logger.info(f"Processed PDF: {final_pdf_path}")

        # Update status to printing
        api_client.update_job_status(job_id, 'printing', 'Sending to printer')

        # Print
        printer_manager.print_file(
            final_pdf_path,
            job_id,
            copies=job.get('quantity', 1),
            duplex=job.get('sides') == 'Double Sided',
            color=job.get('color_option') == 'Color',
            paper_size=job.get('paper_size', 'A4')
        )
        logger.info(f"Sent job {job_id} to printer")

        # Update status to completed
        api_client.update_job_status(job_id, 'completed', 'Print job completed')
        logger.info(f"Job {job_id} completed successfully")

    except Exception as e:
        logger.error(f"Error processing job {job_id}: {str(e)}")
        api_client.update_job_status(job_id, 'failed', f'Error: {str(e)}')
        api_client.log_error(job_id, str(e))

def main():
    """Main entry point"""
    logger.info("Pronto Print Utility started")
    logger.info(f"Print Center: {config['print_center_id']}")
    logger.info(f"Printer: {config['printer_name']}")
    logger.info(f"Poll interval: {config['poll_interval_seconds']} seconds")

    # Create scheduler
    scheduler = BlockingScheduler()

    # Schedule job polling
    scheduler.add_job(
        poll_and_process_jobs,
        'interval',
        seconds=config['poll_interval_seconds']
    )

    # Run immediately on start
    poll_and_process_jobs()

    # Start scheduler
    try:
        scheduler.start()
    except (KeyboardInterrupt, SystemExit):
        logger.info("Pronto Print Utility stopped")

if __name__ == '__main__':
    main()
```

### 11.4 API Client

**File**: `api_client.py`

```python
import requests
import logging
import os
import time

logger = logging.getLogger(__name__)

class ProntoAPIClient:
    def __init__(self, base_url, api_key, api_secret):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.api_secret = api_secret
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'token {api_key}:{api_secret}',
            'Content-Type': 'application/json'
        })

    def get_pending_jobs(self, print_center_id):
        """Get pending jobs for print center"""
        try:
            response = self.session.get(
                f'{self.base_url}/api/method/pronto.api.print_utility.get_pending_jobs',
                params={'print_center_id': print_center_id}
            )
            response.raise_for_status()
            result = response.json()
            return result.get('message', {}).get('jobs', [])
        except Exception as e:
            logger.error(f"Error getting pending jobs: {str(e)}")
            return []

    def download_job_pdf(self, job_id):
        """Download PDF for job"""
        try:
            response = self.session.get(
                f'{self.base_url}/api/method/pronto.api.print_utility.download_job_pdf',
                params={'job_id': job_id},
                stream=True
            )
            response.raise_for_status()

            # Save to downloads folder
            filename = f"{job_id}.pdf"
            filepath = os.path.join('downloads', filename)

            os.makedirs('downloads', exist_ok=True)

            with open(filepath, 'wb') as f:
                for chunk in response.iter_content(chunk_size=8192):
                    f.write(chunk)

            return filepath
        except Exception as e:
            logger.error(f"Error downloading PDF: {str(e)}")
            raise

    def update_job_status(self, job_id, status, notes=None):
        """Update job status"""
        try:
            response = self.session.post(
                f'{self.base_url}/api/method/pronto.api.print_utility.update_job_status',
                json={
                    'job_id': job_id,
                    'status': status,
                    'notes': notes
                }
            )
            response.raise_for_status()
            return True
        except Exception as e:
            logger.error(f"Error updating job status: {str(e)}")
            return False

    def log_error(self, job_id, error_message):
        """Log error to backend"""
        try:
            response = self.session.post(
                f'{self.base_url}/api/method/pronto.api.print_utility.log_error',
                json={
                    'job_id': job_id,
                    'error_message': error_message
                }
            )
            response.raise_for_status()
        except Exception as e:
            logger.error(f"Error logging error: {str(e)}")
```

### 11.5 PDF Processor

**File**: `pdf_processor.py`

```python
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas
from reportlab.lib.units import mm
from reportlab.graphics.barcode import code128
from pypdf import PdfReader, PdfWriter
import os
import logging

logger = logging.getLogger(__name__)

class PDFProcessor:
    def __init__(self, download_folder):
        self.download_folder = download_folder

    def process_job(self, job, pdf_path):
        """Process job: add spec sheet and end sheet"""
        try:
            # Generate spec sheet
            spec_sheet_path = self.generate_spec_sheet(job)

            # Generate end sheet
            end_sheet_path = self.generate_end_sheet(job)

            # Merge PDFs
            final_path = self.merge_pdfs(spec_sheet_path, pdf_path, end_sheet_path, job['name'])

            return final_path
        except Exception as e:
            logger.error(f"Error processing PDF: {str(e)}")
            raise

    def generate_spec_sheet(self, job):
        """Generate specification sheet"""
        filename = f"{job['name']}_spec.pdf"
        filepath = os.path.join(self.download_folder, filename)

        c = canvas.Canvas(filepath, pagesize=A4)
        width, height = A4

        # Title
        c.setFont("Helvetica-Bold", 24)
        c.drawString(50, height - 50, "PRINT JOB SPECIFICATION")

        # Job details
        c.setFont("Helvetica", 12)
        y = height - 100

        details = [
            f"Job ID: {job['name']}",
            f"Order ID: {job.get('print_order', 'N/A')}",
            f"Customer: {job.get('customer_name', 'N/A')}",
            f"Paper Size: {job.get('paper_size', 'N/A')}",
            f"Color: {job.get('color_option', 'N/A')}",
            f"Sides: {job.get('sides', 'N/A')}",
            f"Orientation: {job.get('orientation', 'Portrait')}",
            f"Quantity: {job.get('quantity', 1)}",
            f"Finishing: {job.get('finishing', 'None')}",
            f"Priority: {job.get('priority', 'Normal')}"
        ]

        for detail in details:
            c.drawString(50, y, detail)
            y -= 20

        # Barcode
        barcode = code128.Code128(job['name'], barHeight=15*mm, barWidth=0.5*mm)
        barcode.drawOn(c, 50, 100)

        c.save()
        return filepath

    def generate_end_sheet(self, job):
        """Generate end sheet"""
        filename = f"{job['name']}_end.pdf"
        filepath = os.path.join(self.download_folder, filename)

        c = canvas.Canvas(filepath, pagesize=A4)
        width, height = A4

        # Title
        c.setFont("Helvetica-Bold", 24)
        c.drawCentredString(width/2, height - 100, "PRINT JOB COMPLETE")

        # Details
        c.setFont("Helvetica", 14)
        c.drawCentredString(width/2, height - 150, f"Job ID: {job['name']}")
        c.drawCentredString(width/2, height - 180, f"Order ID: {job.get('print_order', 'N/A')}")
        c.drawCentredString(width/2, height - 210, f"Pickup Code: {job.get('pickup_code', 'N/A')}")
        c.drawCentredString(width/2, height - 240, f"Pickup Location: {job.get('pickup_location', 'N/A')}")

        # Barcode
        barcode = code128.Code128(job.get('pickup_code', job['name']), barHeight=20*mm, barWidth=0.8*mm)
        barcode.drawOn(c, (width - barcode.width)/2, 200)

        c.save()
        return filepath

    def merge_pdfs(self, spec_sheet, original, end_sheet, job_id):
        """Merge spec sheet + original + end sheet"""
        output_filename = f"{job_id}_final.pdf"
        output_path = os.path.join(self.download_folder, output_filename)

        writer = PdfWriter()

        # Add spec sheet
        with open(spec_sheet, 'rb') as f:
            reader = PdfReader(f)
            for page in reader.pages:
                writer.add_page(page)

        # Add original PDF
        with open(original, 'rb') as f:
            reader = PdfReader(f)
            for page in reader.pages:
                writer.add_page(page)

        # Add end sheet
        with open(end_sheet, 'rb') as f:
            reader = PdfReader(f)
            for page in reader.pages:
                writer.add_page(page)

        # Write output
        with open(output_path, 'wb') as f:
            writer.write(f)

        return output_path
```

### 11.6 Printer Manager

**File**: `printer.py`

```python
import platform
import logging
import subprocess

logger = logging.getLogger(__name__)

class PrinterManager:
    def __init__(self, printer_name):
        self.printer_name = printer_name
        self.os_type = platform.system()

        if self.os_type == 'Windows':
            import win32print
            self.win32print = win32print
        elif self.os_type == 'Linux':
            import cups
            self.cups = cups.Connection()

    def print_file(self, filepath, job_name, copies=1, duplex=False, color=False, paper_size='A4'):
        """Print file with specified options"""
        try:
            if self.os_type == 'Windows':
                self._print_windows(filepath, job_name, copies, duplex, color, paper_size)
            elif self.os_type == 'Linux':
                self._print_linux(filepath, job_name, copies, duplex, color, paper_size)
            else:
                raise Exception(f"Unsupported OS: {self.os_type}")

            logger.info(f"Print job '{job_name}' sent to printer '{self.printer_name}'")
        except Exception as e:
            logger.error(f"Error printing file: {str(e)}")
            raise

    def _print_windows(self, filepath, job_name, copies, duplex, color, paper_size):
        """Print on Windows using win32print"""
        import win32api

        # Simple print using ShellExecute
        win32api.ShellExecute(
            0,
            "print",
            filepath,
            f'/d:"{self.printer_name}"',
            ".",
            0
        )

    def _print_linux(self, filepath, job_name, copies, duplex, color, paper_size):
        """Print on Linux using CUPS"""
        options = {
            'copies': str(copies),
            'media': paper_size.lower(),
        }

        if duplex:
            options['sides'] = 'two-sided-long-edge'
        else:
            options['sides'] = 'one-sided'

        if color:
            options['ColorModel'] = 'RGB'
        else:
            options['ColorModel'] = 'Gray'

        self.cups.printFile(
            self.printer_name,
            filepath,
            job_name,
            options
        )
```

### 11.7 Requirements File

**File**: `requirements.txt`

```
requests==2.31.0
APScheduler==3.10.4
reportlab==4.0.7
pypdf==3.17.1
pycups==2.0.1; sys_platform == 'linux'
pywin32==306; sys_platform == 'win32'
```

### 11.8 Backend APIs for Print Utility

**File**: `pronto/pronto/api/print_utility.py`

```python
import frappe
from frappe import _

@frappe.whitelist()
def get_pending_jobs(print_center_id):
    """Get pending jobs for print center"""
    try:
        jobs = frappe.get_all(
            "Job Queue",
            filters={
                "assigned_print_center": print_center_id,
                "queue_status": ["in", ["Pending", "Assigned"]],
                "print_order": ["!=", ""]
            },
            fields=["name", "print_order", "priority", "queue_status", "assigned_to"],
            order_by="priority desc, creation asc"
        )

        # Enrich with order details
        for job in jobs:
            order = frappe.get_doc("Print Order", job.print_order)
            job.update({
                "customer_name": order.customer_name,
                "paper_size": order.paper_size,
                "color_option": order.color_option,
                "sides": order.sides,
                "orientation": order.orientation,
                "quantity": order.quantity,
                "finishing": order.finishing,
                "priority": order.priority,
                "pickup_code": order.pickup_code,
                "pickup_location": order.pickup_location,
                "pdf_url": order.pdf_attachment
            })

        return {"success": True, "jobs": jobs}

    except Exception as e:
        frappe.log_error(f"Get pending jobs error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def download_job_pdf(job_id):
    """Download PDF for job"""
    try:
        job = frappe.get_doc("Job Queue", job_id)
        order = frappe.get_doc("Print Order", job.print_order)

        # Get file path
        file_url = order.pdf_attachment
        file_path = frappe.get_site_path('private', file_url.lstrip('/private/'))

        # Read file
        with open(file_path, 'rb') as f:
            file_content = f.read()

        # Return as binary response
        frappe.local.response.filename = f"{job_id}.pdf"
        frappe.local.response.filecontent = file_content
        frappe.local.response.type = "download"

    except Exception as e:
        frappe.log_error(f"Download PDF error: {str(e)}")
        frappe.throw(_("Error downloading PDF"))

@frappe.whitelist()
def update_job_status(job_id, status, notes=None):
    """Update job status from print utility"""
    try:
        job = frappe.get_doc("Job Queue", job_id)

        # Map print utility statuses to queue statuses
        status_map = {
            "downloading": "In Progress",
            "processing": "In Progress",
            "printing": "In Progress",
            "completed": "Completed",
            "failed": "On Hold"
        }

        job.queue_status = status_map.get(status, "In Progress")
        if notes:
            job.notes = (job.notes or "") + f"\n{notes}"

        job.save(ignore_permissions=True)
        frappe.db.commit()

        return {"success": True}

    except Exception as e:
        frappe.log_error(f"Update job status error: {str(e)}")
        return {"success": False, "error": str(e)}

@frappe.whitelist()
def log_error(job_id, error_message):
    """Log error from print utility"""
    try:
        frappe.log_error(f"Print Utility Error - Job {job_id}: {error_message}")
        return {"success": True}
    except Exception as e:
        return {"success": False, "error": str(e)}
```

---

## 12. Deployment

### 12.1 Docker Deployment

**Complete docker-compose.yml**:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: frappe
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: pronto_db
    ports:
      - "5435:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis-cache:
    image: redis:alpine
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    restart: unless-stopped

  redis-queue:
    image: redis:alpine
    restart: unless-stopped

  frappe:
    image: frappe/bench:latest
    command: bench start
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_CACHE=redis-cache:6379
      - REDIS_QUEUE=redis-queue:6379
    ports:
      - "8000:8000"
      - "9000:9000"
    volumes:
      - ./frappe-bench:/workspace/frappe-bench
    depends_on:
      - postgres
      - redis-cache
      - redis-queue
    restart: unless-stopped

volumes:
  postgres_data:
```

**.env file**:
```
DB_PASSWORD=your_secure_password
```

### 12.2 Production Configuration

**site_config.json** (Production):
```json
{
  "db_name": "pronto_db",
  "db_password": "your_secure_password",
  "db_type": "postgres",
  "db_host": "postgres",
  "db_port": 5432,
  "developer_mode": 0,
  "auto_migrate": false,
  "google_client_id": "YOUR_GOOGLE_CLIENT_ID",
  "google_client_secret": "YOUR_GOOGLE_CLIENT_SECRET",
  "paystack_secret_key": "sk_live_YOUR_SECRET_KEY",
  "paystack_public_key": "pk_live_YOUR_PUBLIC_KEY",
  "encryption_key": "your_encryption_key",
  "host_name": "https://pronto.example.com",
  "mail_server": "smtp.gmail.com",
  "mail_port": 587,
  "use_tls": 1,
  "mail_login": "noreply@pronto.ng",
  "mail_password": "your_email_password"
}
```

### 12.3 Deployment Steps

```bash
# 1. Clone repository
git clone https://github.com/your-org/pronto.git
cd pronto

# 2. Set environment variables
cp .env.example .env
# Edit .env with production values

# 3. Start services
docker-compose up -d

# 4. Initialize site
docker-compose exec frappe bench new-site pronto.example.com \
  --db-type postgres \
  --db-host postgres \
  --db-port 5432 \
  --db-name pronto_db \
  --admin-password admin

# 5. Install app
docker-compose exec frappe bench --site pronto.example.com install-app pronto

# 6. Set production mode
docker-compose exec frappe bench --site pronto.example.com set-config developer_mode 0

# 7. Build assets
docker-compose exec frappe bench build

# 8. Restart services
docker-compose restart
```

### 12.4 SSL/HTTPS Setup

**nginx.conf** (for reverse proxy):
```nginx
server {
    listen 80;
    server_name pronto.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pronto.example.com;

    ssl_certificate /etc/letsencrypt/live/pronto.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/pronto.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 13. Testing & Quality Assurance

### 13.1 Unit Tests Example

**File**: `pronto/pronto/tests/test_orders.py`

```python
import frappe
import unittest

class TestPrintOrder(unittest.TestCase):
    def setUp(self):
        """Setup test data"""
        self.customer = frappe.get_doc({
            "doctype": "Pronto Customer",
            "customer_name": "Test Customer",
            "email": "test@example.com",
            "user": "test@example.com"
        }).insert(ignore_permissions=True)

    def test_price_calculation(self):
        """Test price calculation"""
        from pronto.api.orders import calculate_print_price

        result = calculate_print_price(
            paper_size="A4",
            paper_type="Plain",
            color_option="Black & White",
            sides="Single Sided",
            page_count=10,
            quantity=1,
            priority="Normal (2 hours)"
        )

        self.assertTrue(result['success'])
        self.assertGreater(result['breakdown']['total'], 0)

    def test_pickup_code_generation(self):
        """Test pickup code uniqueness"""
        from pronto.api.orders import generate_pickup_code

        code1 = generate_pickup_code()
        code2 = generate_pickup_code()

        self.assertEqual(len(code1), 4)
        self.assertNotEqual(code1, code2)

    def tearDown(self):
        """Cleanup"""
        frappe.delete_doc("Pronto Customer", self.customer.name, force=True)
```

---

## 14. Conclusion

This comprehensive coding guide provides all the technical details needed to recreate the Pronto Print Order Management System from scratch. Key implementation points:

1. **Frappe v15** with **PostgreSQL on port 5435**
2. **Docker-based deployment** for consistency
3. **Paystack payment integration** with webhook verification
4. **Google OAuth** for seamless authentication
5. **Date-based file storage** for organized PDF management
6. **shadcn UI + Lucide icons** for consistent design
7. **Real-time notifications** for print staff
8. **Credit system** with loyalty points
9. **Discount coupons** with validation
10. **Local print utility** for automated printing
11. **Print center management** with staff assignment
12. **CSRF token handling** for custom www pages

**Next Steps**:
1. Set up development environment
2. Create all DocTypes
3. Implement API endpoints
4. Build frontend pages
5. Configure integrations (Paystack, Google OAuth)
6. Deploy to production
7. Set up local print utility at print centers

---

**Document Version**: 1.0
**Last Updated**: 2025-10-28
**Author**: Pronto Development Team
**Status**: Final

