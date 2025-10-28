# Pronto Print Order Management System - Product Requirements Document (PRD) v1.0

## 1. Executive Summary

**Product Name:** Pronto.ng  
**Version:** 1.0  
**Platform:** Frappe Framework v15  
**Database:** PostgreSQL (Port 5435)  
**Deployment:** Docker  
**Primary Purpose:** Cloud-based print order management system with payment integration, job queue management, and multi-location pickup support

## 2. Product Overview

Pronto is a comprehensive print order management system that enables customers to upload PDF documents, configure print specifications, make payments via Paystack, and collect printed materials from designated pickup locations. The system manages the complete workflow from order submission through printing, quality assurance, dispatch, and collection.

### 2.1 Key Stakeholders

- **Customers**: End users who submit print orders
- **Print Staff**: Operators who execute print jobs
- **QA Staff**: Quality assurance personnel who verify print quality
- **Dispatch Staff**: Personnel who handle order delivery to pickup locations
- **Collection Staff**: Personnel at pickup locations who hand over orders to customers
- **Business Managers**: Administrators who manage pricing, locations, and staff
- **System Administrators**: Technical staff managing the platform

## 3. Core Features (Existing Implementation)

### 3.1 User Authentication & Management

#### 3.1.1 Google OAuth Social Login
- **Provider**: Google Identity Services
- **Flow**: OAuth 2.0 authorization code flow
- **User Creation**: Automatic user and Pronto Customer record creation on first login
- **User Attributes**: Email, name, profile picture
- **Roles**: Automatic "Customer" role assignment for new users
- **Fallback**: Traditional email/password authentication

#### 3.1.2 User Roles & Permissions
- **Customer**: Can create orders, view own orders, make payments
- **Print Staff**: Can view job queue, update job status, mark jobs complete
- **QA Staff**: Can review completed jobs, approve/reject quality
- **Dispatch Staff**: Can create dispatch batches, manage deliveries
- **Collection Staff**: Can mark orders as collected, verify pickup codes
- **Customer Service**: Can view all orders, assist customers
- **Business Manager**: Full access to pricing, locations, staff management, analytics
- **System Manager**: Full system access

### 3.2 Print Order Management

#### 3.2.1 Order Creation Workflow
1. **Customer Login**: User authenticates via Google OAuth or email/password
2. **PDF Upload**: Customer uploads PDF file (max 10MB)
3. **PDF Analysis**: System extracts page count and estimates ink coverage
4. **Print Configuration**:
   - Paper Size (A4, A3, etc.)
   - Color Option (Color, Black & White)
   - Sides (Single Sided, Double Sided)
   - Orientation (Portrait, Landscape)
   - Paper Type (Plain, Premium, etc.)
   - Finishing Options (Binding, Lamination, etc.)
   - Priority Level (No Wahala, Normal 2 hours, Sharp Sharp, Pronto)
   - Quantity (number of copies)
5. **Pickup Location Selection**: Choose from available pickup points
6. **Price Calculation**: Real-time pricing based on all parameters
7. **Order Submission**: Creates Print Order with status "Submitted"
8. **Job Queue Entry**: Automatically creates job queue entry with "Pending Payment" status

#### 3.2.2 Order Statuses
- **Draft**: Order being created (not used in current flow)
- **Submitted**: Order created, awaiting payment
- **Paid**: Payment successful, ready for printing
- **In Queue**: Added to print queue
- **In Progress**: Currently being printed
- **Completed**: Printing finished
- **QA Check**: Under quality review
- **QA Approved**: Passed quality check
- **QA Rejected**: Failed quality check, needs reprinting
- **Ready for Dispatch**: Approved and ready for delivery
- **Dispatched**: In transit to pickup location
- **Delivered**: Arrived at pickup location
- **Ready for Pickup**: Available for customer collection
- **Collected**: Customer has collected the order
- **Cancelled**: Order cancelled
- **Refunded**: Payment refunded

#### 3.2.3 Pickup Code System
- **Format**: 4-character alphanumeric code
- **Character Set**: ABCDETQGHKWY234689 (excludes confusing characters)
- **Uniqueness**: Validated against existing codes
- **Purpose**: Secure order collection verification

### 3.3 Payment Integration (Paystack)

#### 3.3.1 Payment Flow
1. **Initialization**: 
   - API: `/api/method/pronto.api.payment.initialize_payment`
   - Creates unique payment reference: `{order_id}-{timestamp_ms}-{random_suffix}`
   - Calls Paystack Initialize Transaction API
   - Returns authorization URL for customer redirect

2. **Customer Payment**:
   - Redirect to Paystack payment page
   - Customer completes payment
   - Paystack redirects back to callback URL

3. **Payment Callback**:
   - API: `/api/method/pronto.api.payment.payment_callback`
   - Verifies payment with Paystack
   - Updates order payment_status to "Paid"
   - Updates order status from "Submitted" to "Paid"
   - Redirects to order page with success message

4. **Webhook Handling**:
   - Endpoint: `/api/method/pronto.api.payment.payment_webhook`
   - Verifies webhook signature
   - Handles events: `charge.success`, `charge.failed`
   - Updates order status accordingly

5. **Payment Verification**:
   - API: `/api/method/pronto.api.payment.verify_payment`
   - Manual verification endpoint
   - Returns payment status and details

#### 3.3.2 Payment Statuses
- **Pending**: Awaiting payment
- **Processing**: Payment initiated
- **Paid**: Payment successful
- **Failed**: Payment failed
- **Cancelled**: Payment cancelled
- **Refunded**: Payment refunded
- **Expired**: Payment session expired

### 3.4 Job Queue Management

#### 3.4.1 Job Queue Entry
- **Creation**: Automatic on order submission
- **Fields**:
  - Print Order (link)
  - Batch Number (unique identifier)
  - Assigned To (print staff user)
  - Priority (Low, Normal, High, Urgent)
  - Queue Status (Pending, Assigned, In Progress, Completed, On Hold, Cancelled)
  - Notes (text field for staff comments)
  - Timestamps: Received At, Started At, Completed At

#### 3.4.2 Job Assignment
- **Manual Assignment**: Business Manager assigns jobs to print staff
- **Auto-Assignment**: (Future feature) Based on staff availability and workload
- **Status Updates**: Print staff can update job status in real-time

### 3.5 PDF Upload & File Management

#### 3.5.1 File Upload System
- **Method**: Multipart file upload or base64 encoding
- **Storage Structure**: `/files/print-orders/YYYY-MM-DD/HHMMSS_filename.pdf`
- **File Size Limit**: 10MB
- **Validation**: PDF format only
- **Frappe File Record**: Creates File doctype entry with metadata

#### 3.5.2 PDF Analysis
- **Page Count Extraction**:
  - Primary: pypdf library
  - Fallback 1: PyPDF2 library
  - Fallback 2: Regex-based PDF structure parsing
  - Default: 1 page if all methods fail

- **Ink Coverage Analysis**:
  - Text density analysis
  - Word count and line count metrics
  - Coverage percentage estimation (0-85%)
  - Ink cost estimation based on coverage tiers
  - Stores: `estimated_ink_coverage`, `estimated_ink_cost`

### 3.6 Pricing System

#### 3.6.1 Pricing Components
1. **Base Price**: From Paper Size (e.g., A4: ₦50, A3: ₦100)
2. **Paper Type Multiplier**: Plain (1.0x), Premium (1.5x)
3. **Color Multiplier**: Black & White (1.0x), Color (1.5x)
4. **Sides Multiplier**: Single (1.0x), Double (1.8x)
5. **Finishing Cost**: Fixed cost per finishing option
6. **Priority Cost**: Fixed additional cost per priority level
7. **Tax Rate**: Configurable percentage
8. **Service Charge**: Configurable percentage

#### 3.6.2 Pricing Formula
```
base_price = paper_size.base_price * page_count * quantity
paper_type_adjustment = base_price * paper_type.price_multiplier
color_adjustment = paper_type_adjustment * color_multiplier
sides_adjustment = color_adjustment * sides_multiplier
finishing_cost = finishing.additional_cost (if applicable)
priority_cost = priority.additional_cost (if applicable)
subtotal = sides_adjustment + finishing_cost + priority_cost
tax = subtotal * (tax_rate / 100)
service_charge = subtotal * (service_charge_rate / 100)
total = subtotal + tax + service_charge
```

#### 3.6.3 Priority Levels & Costs
- **No Wahala**: Relaxed turnaround, ₦0 additional
- **Normal (2 hours)**: Standard 2-hour turnaround, ₦0 additional
- **Sharp Sharp**: Expedited service, ₦200 additional
- **Pronto**: Ultra-fast service, ₦500 additional

### 3.7 Pickup Locations

#### 3.7.1 Location Management
- **Fields**: Location Name, Address, Operating Hours, Is Active
- **Examples**: 
  - H-Medix Wuse 2, Wuse 2, Abuja
  - Garki Shopping Mall, Garki, Abuja
  - Wuse Market, Wuse Market, Abuja

### 3.8 Staff Dashboards

#### 3.8.1 Print Staff Dashboard
- **URL**: `/print-staff`
- **Features**:
  - View job queue (pending, in progress, completed)
  - Filter by status, priority, date
  - View job details (PDF, specifications)
  - Update job status
  - Add notes to jobs
  - Download PDF for printing
  - Mark jobs as completed

#### 3.8.2 QA Staff Dashboard
- **URL**: `/qa-staff`
- **Features**:
  - View completed print jobs
  - Review job specifications
  - Approve or reject jobs
  - Add QA notes
  - Specify rejection reasons
  - Trigger reprint for rejected jobs

#### 3.8.3 Dispatch Staff Dashboard
- **URL**: `/dispatch-staff`
- **Features**:
  - View QA-approved orders
  - Create dispatch batches
  - Assign orders to batches
  - Assign dispatch riders
  - Track dispatch status
  - Mark orders as delivered

#### 3.8.4 Collection Staff Dashboard
- **URL**: `/collection-staff`
- **Features**:
  - View orders at their pickup location
  - Search by pickup code
  - Verify customer identity
  - Mark orders as collected
  - Record collection timestamp

#### 3.8.5 Business Manager Dashboard
- **URL**: `/business-manager`
- **Features**:
  - Overview analytics (orders, revenue, status breakdown)
  - Pricing configuration management
  - Staff management
  - Pickup location management
  - Order operations (view all orders, filter, search)
  - Revenue analytics and charts

### 3.9 Customer Dashboard

#### 3.9.1 Dashboard Features
- **URL**: `/dashboard`
- **Features**:
  - View all orders (with status)
  - Create new order
  - View order details
  - Track order status
  - Download invoices
  - Reorder functionality

### 3.10 Master Data

#### 3.10.1 Print Paper Size
- **Fields**: Size Name, Base Price, Is Active
- **Examples**: A4 (₦50), A3 (₦100), A5 (₦30)

#### 3.10.2 Print Paper Type
- **Fields**: Type Name, Price Multiplier, Description, Is Active
- **Examples**: Plain (1.0x), Premium (1.5x), Glossy (1.8x)

#### 3.10.3 Print Finishing
- **Fields**: Finishing Name, Pricing Type (Fixed Cost/Multiplier), Additional Cost, Price Multiplier, Description, Is Active
- **Examples**: Binding (₦100), Lamination (₦150), Stapling (₦20)

#### 3.10.4 Dispatch Rider
- **Fields**: Rider Name, Phone Number, Vehicle Type, Is Active

#### 3.10.5 Dispatch Batch
- **Fields**: Batch Number, Dispatch Rider, Batch Status, Dispatch Date, Delivery Date
- **Child Table**: Dispatch Batch Order (links to Print Orders)

## 4. New Features to Implement

### 4.1 Collection Center Management

#### 4.1.1 Print Center DocType
- **Purpose**: Manage physical print centers/locations
- **Fields**:
  - Center Name (Data, required, unique)
  - Center Code (Data, auto-generated, unique)
  - Address Line 1 (Data)
  - Address Line 2 (Data)
  - City (Data)
  - State (Data)
  - Postal Code (Data)
  - Phone Number (Data)
  - Email (Data)
  - Operating Hours (Text)
  - Center Manager (Link to User)
  - Is Active (Check, default: 1)
  - Capacity (Int) - max concurrent jobs
  - Equipment Details (Text)

#### 4.1.2 Staff-to-Print-Center Assignment
- **Implementation**: Add field to User doctype or create separate DocType
- **Option 1 - User Custom Field**:
  - Field: `assigned_print_center` (Link to Print Center)
  - Add to User doctype via custom field

- **Option 2 - Staff Assignment DocType**:
  - Staff Member (Link to User)
  - Print Center (Link to Print Center)
  - Assignment Date (Date)
  - Is Active (Check)
  - Role (Select: Print Staff, QA Staff, Manager)

#### 4.1.3 Job Routing Based on Print Center
- **Job Queue Enhancement**:
  - Add field: `assigned_print_center` (Link to Print Center)
  - Filter logic: Print staff only see jobs assigned to their print center
  - Auto-assignment: Route jobs to print centers based on capacity/location

- **API Changes**:
  - `get_print_job_queue()`: Filter by staff's assigned print center
  - Add parameter: `print_center` for manual filtering

#### 4.1.4 Print Center CRUD Operations
- **Business Manager Interface**:
  - List view: All print centers with status
  - Create: Add new print center
  - Edit: Update print center details
  - Deactivate: Set is_active = 0
  - View: Print center details with assigned staff list

### 4.2 Discount & Pricing System

#### 4.2.1 Discount Coupon DocType
- **Fields**:
  - Coupon Code (Data, required, unique, uppercase)
  - Coupon Name (Data, required)
  - Discount Type (Select: Percentage, Fixed Amount)
  - Discount Value (Float) - percentage or amount
  - Minimum Order Amount (Currency) - minimum to apply
  - Maximum Discount Amount (Currency) - cap for percentage discounts
  - Valid From (Date)
  - Valid To (Date)
  - Usage Limit (Int) - total uses allowed
  - Usage Count (Int, read-only) - times used
  - Per Customer Limit (Int) - uses per customer
  - Is Active (Check)
  - Applicable To (Select: All Customers, Specific Customers)
  - Customer List (Table) - if specific customers
  - Description (Text)
  - Terms & Conditions (Text)

#### 4.2.2 Coupon Validation Logic
- **API**: `/api/method/pronto.api.orders.validate_coupon`
- **Checks**:
  1. Coupon exists and is active
  2. Current date within valid range
  3. Usage limit not exceeded
  4. Per-customer limit not exceeded
  5. Order amount meets minimum
  6. Customer eligible (if restricted)
- **Returns**: Discount amount, validation status, error message

#### 4.2.3 Coupon Application in Order
- **Print Order Enhancement**:
  - Add fields:
    - `coupon_code` (Data)
    - `discount_amount` (Currency)
    - `discount_type` (Data)
  - Update pricing calculation to subtract discount
  - Record coupon usage in Coupon Usage Log

#### 4.2.4 Coupon Usage Log DocType
- **Fields**:
  - Coupon Code (Link to Discount Coupon)
  - Customer (Link to Pronto Customer)
  - Print Order (Link to Print Order)
  - Discount Amount (Currency)
  - Used On (Datetime, default: now)

### 4.3 Customer Credit System

#### 4.3.1 Pronto Customer Enhancement
- **Add Fields**:
  - `credit_balance` (Currency, default: 0, read-only)
  - `loyalty_points` (Int, default: 0, read-only)
  - `total_credit_added` (Currency, default: 0, read-only)
  - `total_credit_used` (Currency, default: 0, read-only)

#### 4.3.2 Customer Credit Transaction DocType
- **Fields**:
  - Transaction ID (Data, auto-generated, unique)
  - Customer (Link to Pronto Customer, required)
  - Transaction Type (Select: Credit Added, Credit Used, Points Converted, Refund)
  - Amount (Currency, required)
  - Points (Int) - for points conversion
  - Balance Before (Currency, read-only)
  - Balance After (Currency, read-only)
  - Related Order (Link to Print Order) - if applicable
  - Payment Reference (Data) - for credit additions
  - Notes (Text)
  - Transaction Date (Datetime, default: now)
  - Created By (Link to User, default: current user)

#### 4.3.3 Credit Addition
- **Methods**:
  1. **Direct Payment**: Customer pays to add credit
     - API: `/api/method/pronto.api.credit.add_credit`
     - Integrates with Paystack
     - Creates Credit Transaction record
     - Updates customer credit_balance

  2. **Admin Addition**: Business Manager adds credit manually
     - Requires Business Manager role
     - Includes notes/reason

#### 4.3.4 Credit Usage in Orders
- **Order Creation Enhancement**:
  - Add option: "Use Credit Balance"
  - If selected and sufficient balance:
    - Deduct from credit_balance
    - Create Credit Transaction (Credit Used)
    - Reduce or eliminate Paystack payment
  - If partial credit:
    - Use full credit balance
    - Charge remaining via Paystack

- **API**: `/api/method/pronto.api.orders.create_order_with_credit`
- **Parameters**: `use_credit` (boolean), `credit_amount` (optional, auto-calculate if not provided)

#### 4.3.5 Loyalty Points System
- **Points Earning**:
  - Rule: 1 point per ₦100 spent (configurable)
  - Triggered on order completion
  - API: `/api/method/pronto.api.credit.award_loyalty_points`

- **Points Conversion**:
  - Conversion Rate: 100 points = ₦50 credit (configurable)
  - API: `/api/method/pronto.api.credit.convert_points_to_credit`
  - Creates Credit Transaction record
  - Updates credit_balance and loyalty_points

- **Loyalty Configuration DocType**:
  - Points Per Currency Unit (Int, default: 1 point per ₦100)
  - Points to Credit Ratio (Float, default: 100 points = ₦50)
  - Minimum Points for Conversion (Int, default: 100)
  - Points Expiry Days (Int, 0 = no expiry)

#### 4.3.6 Credit Transaction History
- **Customer Dashboard**:
  - Tab: "Credit & Points"
  - Display: Current balance, total points
  - Transaction history table
  - Actions: Add credit, Convert points

- **API**: `/api/method/pronto.api.credit.get_credit_history`

### 4.4 Real-time Notifications for Print Staff

#### 4.4.1 Notification Requirements
- **Trigger Events**:
  1. New job assigned to print staff
  2. New unassigned job enters queue (visible to all print staff)
  3. Job status changed by manager
  4. Job priority changed

- **Notification Components**:
  - Visual popup/toast notification
  - Sound alert (configurable)
  - Desktop notification (browser permission)
  - Badge count on dashboard

#### 4.4.2 Implementation Technology
- **Option 1 - Frappe Realtime (Socket.IO)**:
  - Use Frappe's built-in realtime framework
  - Server-side: Emit events on job creation/update
  - Client-side: Subscribe to events, show notifications

- **Option 2 - Server-Sent Events (SSE)**:
  - Lightweight, one-way communication
  - Endpoint: `/api/method/pronto.api.staff.job_notifications_stream`
  - Client: EventSource API

- **Option 3 - Polling**:
  - Fallback for environments without WebSocket
  - Poll every 30 seconds
  - API: `/api/method/pronto.api.staff.check_new_jobs`

- **Recommended**: Frappe Realtime (Socket.IO) with polling fallback

#### 4.4.3 Notification API
- **Server-Side Event Emission**:
  ```python
  frappe.publish_realtime(
      event='new_print_job',
      message={
          'job_id': job.name,
          'order_id': job.print_order,
          'priority': job.priority,
          'customer_name': order.customer_name
      },
      user=assigned_staff_user  # or room='print_staff' for all
  )
  ```

- **Client-Side Subscription**:
  ```javascript
  frappe.realtime.on('new_print_job', (data) => {
      showNotification(data);
      playSound();
      updateJobQueue();
  });
  ```

#### 4.4.4 Sound Alerts
- **Implementation**:
  - Audio file: `/assets/pronto/sounds/notification.mp3`
  - Play on notification
  - User preference: Enable/disable sound
  - Store preference in localStorage

### 4.5 Human-Friendly IDs

#### 4.5.1 Print Order ID Format
- **Current**: Auto-increment integer (e.g., "1", "2", "3")
- **New Format**: `ORD-YYYY-NNNN` (e.g., "ORD-2024-0001")
- **Implementation**:
  - Update Print Order naming_rule to "Expression"
  - Set autoname expression: `format:ORD-{YYYY}-{####}`

#### 4.5.2 Job Queue ID Format
- **Current**: Auto-increment integer
- **New Format**: `JOB-YYYY-NNNN` (e.g., "JOB-2024-0001")
- **Implementation**:
  - Update Job Queue naming_rule to "Expression"
  - Set autoname expression: `format:JOB-{YYYY}-{####}`

#### 4.5.3 Other DocTypes
- **Dispatch Batch**: `BATCH-YYYY-NNNN`
- **Discount Coupon**: Manual entry (e.g., "WELCOME10", "SAVE20")
- **Credit Transaction**: `TXN-YYYY-NNNN`

### 4.6 Design & Theming with shadcn UI

#### 4.6.1 Color Scheme
- **Primary**: #5400D3 (Purple)
- **Secondary**: #60D701 (Lime Green)
- **Success**: #22c55e
- **Warning**: #f59e0b
- **Danger**: #ef4444
- **Dark**: #1f2937
- **Light**: #f8fafc

#### 4.6.2 Centralized Stylesheet
- **File**: `/pronto/public/css/pronto-theme.css`
- **CSS Variables**:
  ```css
  :root {
      --primary: #5400D3;
      --secondary: #60D701;
      --success: #22c55e;
      --warning: #f59e0b;
      --danger: #ef4444;
      --dark: #1f2937;
      --light: #f8fafc;
      --border-radius: 0.5rem;
      --box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  }
  ```

#### 4.6.3 shadcn UI Components
- **Installation**: Copy shadcn components to `/pronto/public/js/components/`
- **Components to Use**:
  - Button
  - Card
  - Dialog/Modal
  - Input
  - Select
  - Table
  - Toast (for notifications)
  - Badge
  - Tabs
  - Alert

#### 4.6.4 Lucide Icons
- **Library**: Lucide React/Lucide Icons
- **CDN**: `https://unpkg.com/lucide@latest/dist/umd/lucide.min.js`
- **Usage**: Replace all Font Awesome icons with Lucide equivalents
- **Examples**:
  - `fa-print` → `<i data-lucide="printer"></i>`
  - `fa-user` → `<i data-lucide="user"></i>`
  - `fa-check` → `<i data-lucide="check"></i>`

#### 4.6.5 Page Styling Consistency
- **All www pages must**:
  - Include `/assets/pronto/css/pronto-theme.css`
  - Use shadcn UI components
  - Use Lucide icons exclusively
  - Follow color scheme
  - Use consistent spacing (Tailwind classes)

### 4.7 Legal Pages

#### 4.7.1 Privacy Policy
- **URL**: `/privacy-policy`
- **File**: `/pronto/www/privacy-policy.html`
- **Content Sections**:
  1. Information We Collect
  2. How We Use Your Information
  3. Data Storage and Security
  4. Third-Party Services (Paystack, Google)
  5. Your Rights
  6. Contact Information

#### 4.7.2 Terms of Service
- **URL**: `/terms-of-service`
- **File**: `/pronto/www/terms-of-service.html`
- **Content Sections**:
  1. Acceptance of Terms
  2. Service Description
  3. User Responsibilities
  4. Payment Terms
  5. Refund Policy
  6. Limitation of Liability
  7. Governing Law

### 4.8 Local Print Utility (Standalone Python Tool)

#### 4.8.1 Purpose
- Runs on local network at print center
- Polls Frappe backend for assigned print jobs
- Downloads PDFs
- Prepends specification sheet
- Appends end sheet
- Sends to local printer with correct settings

#### 4.8.2 Technology Stack
- **Language**: Python 3.8+
- **Libraries**:
  - `requests`: HTTP API calls
  - `APScheduler`: Job scheduling/polling
  - `reportlab`: PDF generation (spec sheets)
  - `pypdf`: PDF manipulation
  - `pycups` (Linux): CUPS printer interface
  - `pywin32` (Windows): Windows printer interface

#### 4.8.3 Configuration
- **File**: `config.json`
- **Fields**:
  ```json
  {
    "frappe_url": "https://pronto.example.com",
    "api_key": "your-api-key",
    "api_secret": "your-api-secret",
    "print_center_id": "PRINT-CENTER-001",
    "poll_interval_seconds": 30,
    "printer_name": "HP LaserJet Pro",
    "download_folder": "./downloads",
    "log_level": "INFO"
  }
  ```

#### 4.8.4 Functionality

**4.8.4.1 Job Polling**
- **Endpoint**: `/api/method/pronto.api.print_utility.get_pending_jobs`
- **Parameters**: `print_center_id`
- **Returns**: List of jobs with status "Ready to Print" assigned to this center
- **Frequency**: Every 30 seconds (configurable)

**4.8.4.2 PDF Download**
- **Endpoint**: `/api/method/pronto.api.print_utility.download_job_pdf`
- **Parameters**: `job_id`
- **Returns**: PDF file binary
- **Storage**: Save to `downloads/{job_id}.pdf`

**4.8.4.3 Specification Sheet Generation**
- **Library**: ReportLab
- **Content**:
  - Job ID
  - Order ID
  - Customer Name
  - Paper Size
  - Color Option
  - Sides (Simplex/Duplex)
  - Orientation
  - Quantity
  - Finishing Instructions
  - Priority
  - Barcode (Job ID as barcode)

**4.8.4.4 End Sheet Generation**
- **Content**:
  - "Print Job Complete"
  - Job ID
  - Order ID
  - Pickup Code
  - Pickup Location
  - QR Code (for tracking)

**4.8.4.5 PDF Merging**
- **Library**: pypdf
- **Process**:
  1. Create spec sheet PDF
  2. Load original PDF
  3. Create end sheet PDF
  4. Merge: spec_sheet + original + end_sheet
  5. Save as `{job_id}_final.pdf`

**4.8.4.6 Print Job Submission**
- **Linux (CUPS)**:
  ```python
  import cups
  conn = cups.Connection()
  conn.printFile(
      printer_name,
      pdf_path,
      job_name,
      {
          'copies': str(quantity),
          'sides': 'two-sided-long-edge' if duplex else 'one-sided',
          'media': paper_size.lower(),
          'ColorModel': 'RGB' if color else 'Gray'
      }
  )
  ```

- **Windows (pywin32)**:
  ```python
  import win32print
  import win32api
  win32api.ShellExecute(
      0,
      "print",
      pdf_path,
      f'/d:"{printer_name}"',
      ".",
      0
  )
  ```

**4.8.4.7 Status Updates**
- **Endpoint**: `/api/method/pronto.api.print_utility.update_job_status`
- **Parameters**: `job_id`, `status`, `notes`
- **Statuses**:
  - `downloading`: PDF download started
  - `processing`: Generating spec sheets
  - `printing`: Sent to printer
  - `completed`: Print job finished
  - `failed`: Error occurred

**4.8.4.8 Error Handling**
- **Logging**: All errors logged to `print_utility.log`
- **Retry Logic**: 3 retries for network errors
- **Notifications**: Send error notifications to Frappe backend
- **Endpoint**: `/api/method/pronto.api.print_utility.log_error`

#### 4.8.5 Cross-Platform Compatibility
- **Detection**:
  ```python
  import platform
  os_type = platform.system()  # 'Windows', 'Linux', 'Darwin'
  ```
- **Conditional Imports**:
  ```python
  if os_type == 'Windows':
      import win32print
  elif os_type == 'Linux':
      import cups
  ```

#### 4.8.6 Installation & Deployment
- **Package**: Distribute as executable (PyInstaller)
- **Windows**: `pronto_print_utility.exe`
- **Linux**: `pronto_print_utility` (binary or systemd service)
- **Configuration**: GUI for initial setup or manual config.json edit
- **Auto-Start**: Windows service or Linux systemd unit

## 5. User Workflows

### 5.1 Customer Order Workflow
1. Navigate to Pronto.ng homepage
2. Click "Get Started" or "Login"
3. Authenticate via Google OAuth or email/password
4. Upload PDF file
5. Configure print options
6. Select pickup location
7. Review pricing
8. Apply discount coupon (if available)
9. Choose payment method (Paystack or Credit Balance)
10. Complete payment
11. Receive order confirmation with pickup code
12. Track order status on dashboard
13. Receive notification when ready for pickup
14. Collect order at pickup location with pickup code

### 5.2 Print Staff Workflow
1. Login to print staff dashboard
2. View job queue filtered by assigned print center
3. Receive real-time notification for new jobs
4. Select job to print
5. Download PDF
6. Review specifications
7. Update status to "In Progress"
8. Print document (via local print utility or manual)
9. Update status to "Completed"
10. Hand over to QA staff

### 5.3 QA Staff Workflow
1. Login to QA staff dashboard
2. View completed print jobs
3. Select job for review
4. Inspect physical print quality
5. Compare with specifications
6. Approve or reject
7. If rejected: Add rejection reason, trigger reprint
8. If approved: Update status to "QA Approved"

### 5.4 Dispatch Staff Workflow
1. Login to dispatch staff dashboard
2. View QA-approved orders
3. Create dispatch batch
4. Add orders to batch
5. Assign dispatch rider
6. Print dispatch manifest
7. Hand over to rider
8. Update status to "Dispatched"
9. Rider delivers to pickup location
10. Update status to "Delivered"

### 5.5 Collection Staff Workflow
1. Login to collection staff dashboard
2. View orders at their pickup location
3. Customer arrives with pickup code
4. Search by pickup code
5. Verify customer identity (optional)
6. Hand over order
7. Mark as "Collected"
8. Record collection timestamp

### 5.6 Business Manager Workflow
1. Login to business manager dashboard
2. View analytics and KPIs
3. Manage pricing configuration
4. Manage print centers
5. Assign staff to print centers
6. Create discount coupons
7. View all orders and filter
8. Generate reports
9. Manage pickup locations
10. Configure loyalty points settings

## 6. Data Models & Relationships

### 6.1 Core Entities
- **Pronto Customer** → **Print Order** (1:N)
- **Print Order** → **Job Queue** (1:1)
- **Print Order** → **Dispatch Batch Order** (1:1)
- **Dispatch Batch** → **Dispatch Batch Order** (1:N)
- **Dispatch Batch** → **Dispatch Rider** (N:1)
- **Print Order** → **Payment Transaction Log** (1:N)
- **Print Order** → **Pickup Location** (N:1)
- **Print Order** → **Print Paper Size** (N:1)
- **Print Order** → **Print Paper Type** (N:1)
- **Print Order** → **Print Finishing** (N:1)
- **User** → **Pronto Customer** (1:1)
- **User** → **Print Center** (N:1) - via assignment
- **Pronto Customer** → **Customer Credit Transaction** (1:N)
- **Pronto Customer** → **Coupon Usage Log** (1:N)
- **Discount Coupon** → **Coupon Usage Log** (1:N)

### 6.2 Workflow State Machine

**Print Order Status Flow**:
```
Submitted → Paid → In Queue → In Progress → Completed → 
QA Check → QA Approved → Ready for Dispatch → Dispatched → 
Delivered → Ready for Pickup → Collected
```

**Alternate Paths**:
- QA Rejected → In Queue (reprint)
- Any status → Cancelled
- Paid → Refunded

## 7. Business Rules

### 7.1 Order Rules
1. Orders cannot be edited after payment
2. Orders must have payment before entering print queue
3. Pickup code must be unique across all active orders
4. PDF file size limited to 10MB
5. Minimum order amount: ₦0 (configurable)

### 7.2 Payment Rules
1. Payment must be verified before order proceeds
2. Refunds require Business Manager approval
3. Credit balance cannot be negative
4. Loyalty points awarded only on completed orders

### 7.3 Coupon Rules
1. Only one coupon per order
2. Coupon must be active and within valid date range
3. Order amount must meet minimum requirement
4. Usage limits enforced (total and per customer)

### 7.4 Credit Rules
1. Credit can only be added via payment or admin action
2. Credit usage is first-in-first-out (FIFO)
3. Points conversion requires minimum points threshold
4. Points expire after configured days (if enabled)

### 7.5 Job Assignment Rules
1. Jobs assigned to print center based on capacity
2. Print staff only see jobs for their assigned center
3. Jobs cannot be reassigned once printing starts
4. QA rejection triggers automatic requeue

## 8. Technical Requirements

### 8.1 Platform
- **Framework**: Frappe v15
- **Database**: PostgreSQL 11+ (Port 5435)
- **Python**: 3.8+
- **Node.js**: 14+
- **Deployment**: Docker containers

### 8.2 Integrations
- **Payment Gateway**: Paystack
- **Authentication**: Google OAuth 2.0
- **File Storage**: Frappe File System (local or S3-compatible)

### 8.3 Performance
- **Page Load**: < 2 seconds
- **API Response**: < 500ms (95th percentile)
- **PDF Upload**: Support up to 10MB
- **Concurrent Users**: 100+
- **Job Queue Polling**: 30-second intervals

### 8.4 Security
- **Authentication**: OAuth 2.0, session-based
- **Authorization**: Role-based access control (RBAC)
- **CSRF Protection**: Frappe CSRF tokens for all POST requests
- **Payment Security**: Paystack webhook signature verification
- **Data Encryption**: HTTPS/TLS for all communications

### 8.5 Browser Support
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## 9. Success Metrics

### 9.1 Business Metrics
- **Order Volume**: Number of orders per day/week/month
- **Revenue**: Total revenue, average order value
- **Customer Retention**: Repeat customer rate
- **Turnaround Time**: Average time from order to collection
- **Customer Satisfaction**: NPS score, ratings

### 9.2 Operational Metrics
- **Print Queue Length**: Average jobs in queue
- **QA Rejection Rate**: Percentage of jobs rejected
- **On-Time Delivery**: Percentage delivered within SLA
- **Staff Utilization**: Jobs per staff member per day

### 9.3 Technical Metrics
- **System Uptime**: 99.9% target
- **API Latency**: < 500ms (95th percentile)
- **Error Rate**: < 0.1% of requests
- **Payment Success Rate**: > 95%

## 10. Future Enhancements (Out of Scope for v1)

- Mobile app (iOS/Android)
- SMS notifications
- Email notifications
- Advanced analytics dashboard
- Multi-currency support
- Bulk order upload (CSV)
- API for third-party integrations
- Customer reviews and ratings
- Automated pricing optimization
- Machine learning for demand forecasting
- Integration with accounting software
- Multi-language support

---

**Document Version**: 1.0  
**Last Updated**: 2025-10-28  
**Author**: Pronto Product Team  
**Status**: Final

