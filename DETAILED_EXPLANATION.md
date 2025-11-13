# Mobile Service Management Addon - Detailed Explanation

## 📱 Overview

This Odoo addon is designed for **Mobile Service Shops** to manage their daily operations. It helps track mobile device repairs, manage customer information, handle invoices, manage inventory (parts), and generate service tickets. Think of it as a complete business management system for a mobile repair shop.

---

## 🎯 Main Purpose

The addon allows service shop owners to:
1. **Register customer service requests** for mobile device repairs
2. **Track repair status** through different stages (Draft → Assigned → Completed → Returned)
3. **Manage mobile brands and models** (Samsung, iPhone, etc.)
4. **Record complaints** (screen broken, battery issues, etc.)
5. **Track parts inventory** (batteries, screens, etc.)
6. **Create invoices** for services and parts
7. **Manage stock movements** (when parts are used)
8. **Generate service tickets** (receipts for customers)
9. **Send emails** to customers

---

## 🏗️ Core Models (Database Tables)

### 1. **Mobile Service** (`mobile.service`)
**This is the MAIN model - the heart of the system**

This represents a **service request** - when a customer brings in their mobile phone for repair.

**Key Fields:**
- `name`: Service number (like "SERV/0001") - automatically generated
- `person_name`: Customer (links to Odoo's contact/partner)
- `contact_no`, `email_id`, `street`, `city`: Customer contact info (auto-filled from customer)
- `brand_name`: Mobile brand (Samsung, Apple, etc.)
- `model_name`: Specific model (iPhone 13, Galaxy S21, etc.)
- `imei_no`: IMEI number of the device
- `is_in_warranty`: Is the device under warranty?
- `warranty_number`: Warranty certificate number
- `re_repair`: Is this a repeat repair?
- `date_request`: When customer submitted the device
- `return_date`: Expected return date
- `technician_name`: Which technician is assigned to this job
- `service_state`: Current status (Draft, Assigned, Completed, Returned, Not Solved)
- `complaints_tree`: List of complaints/issues (One2many relationship)
- `product_order_line`: List of parts needed/used (One2many relationship)
- `internal_notes`: Private notes for shop staff
- `invoice_count`: Number of invoices created
- `invoice_ids`: Links to invoices created
- `stock_picking_id`: Link to stock transfer (when parts are sent out)

**Key Functions:**
- `approve()`: Changes status from "Draft" to "Assigned"
- `complete()`: Marks service as "Completed"
- `return_to()`: Marks service as "Returned" to customer
- `not_solved()`: Marks service as "Not Solved"
- `action_invoice_create_wizard()`: Opens wizard to create invoice
- `action_post_stock()`: Creates stock movement when parts are used
- `get_ticket()`: Generates printable service ticket (PDF)
- `action_send_mail()`: Sends email to customer
- `action_view_invoice()`: Shows all invoices for this service

**Workflow States:**
```
Draft → Assigned → Completed → Returned
                ↓
          Not Solved
```

**Sequence Generation:**
When a new service is created, it automatically generates a service number like "SERV/0001", "SERV/0002", etc.

---

### 2. **Mobile Brand** (`mobile.brand`)
**Represents mobile phone brands**

- `brand_name`: Name of the brand (e.g., "Samsung", "Apple", "Xiaomi")

**Example:**
- Brand: Samsung
- Brand: Apple
- Brand: OnePlus

---

### 3. **Brand Model** (`brand.model`)
**Represents specific phone models under a brand**

- `mobile_brand_name`: Which brand this model belongs to (Many2one to `mobile.brand`)
- `mobile_brand_models`: Model name (e.g., "iPhone 13 Pro", "Galaxy S21")
- `image_medium`: Image of the phone model

**Relationship:**
- One Brand can have Many Models
- Example: Brand "Apple" → Models: "iPhone 13", "iPhone 14", "iPhone 15"

**Domain Filtering:**
When selecting a model, it only shows models that belong to the selected brand.

---

### 4. **Mobile Complaint** (`mobile.complaint`)
**Represents types of complaints/issues**

- `complaint_type`: Type of complaint (e.g., "Screen Broken", "Battery Issue", "Water Damage", "Charging Problem")

**Example Complaints:**
- Screen Broken
- Battery Replacement
- Charging Port Issue
- Software Problem
- Camera Not Working

---

### 5. **Mobile Complaint Description** (`mobile.complaint.description`)
**Detailed descriptions for each complaint type**

- `complaint_type_template`: Which complaint type (Many2one to `mobile.complaint`)
- `description`: Detailed description text

**Purpose:**
Allows creating pre-defined descriptions for common issues. For example:
- Complaint: "Screen Broken"
  - Description: "Screen has cracks and touch not working properly"

---

### 6. **Mobile Complaint Tree** (`mobile.complaint.tree`)
**Links complaints to a service request**

This is a **bridge table** that connects:
- Service Request (`mobile.service`)
- Complaint Type (`mobile.complaint`)
- Complaint Description (`mobile.complaint.description`)

**Fields:**
- `complaint_id`: Which service request (Many2one to `mobile.service`)
- `complaint_type_tree`: Which complaint (Many2one to `mobile.complaint`)
- `description_tree`: Detailed description (Many2one to `mobile.complaint.description`)

**Domain Filtering:**
When selecting a description, it only shows descriptions that match the selected complaint type.

**Purpose:**
A single service request can have **multiple complaints**. For example:
- Service Request: "SERV/0001"
  - Complaint 1: Screen Broken
  - Complaint 2: Battery Issue
  - Complaint 3: Charging Port Problem

---

### 7. **Product Order Line** (`product.order.line`)
**Represents parts/items used in a service**

- `product_order_id`: Which service request (Many2one to `mobile.service`)
- `product_id`: Which product/part (Many2one to `product.product`)
- `product_uom_qty`: Quantity used
- `price_unit`: Unit price of the part
- `qty_invoiced`: How much quantity has been invoiced
- `qty_stock_move`: How much quantity has been posted to stock
- `part_price`: Total price (quantity × unit price) - computed field
- `product_uom`: Unit of measure (e.g., "Units", "Pieces")

**Key Functions:**
- `change_prod()`: When product is selected, auto-fills price and unit of measure
- `_compute_part_price()`: Calculates total price automatically
- `_create_stock_moves_transfer()`: Creates stock movement when parts are used

**Purpose:**
Tracks which parts are used in each repair and ensures:
- Parts are only invoiced for the quantity used
- Stock is updated when parts are used
- Cannot invoice more than quantity used
- Cannot post stock for more than quantity used

---

### 8. **Product Template** (Extended)
**Extended Odoo's product model to add mobile-specific fields**

**New Fields Added:**
- `is_a_parts`: Checkbox to mark if product is a mobile part
- `brand_name`: Which mobile brand this part is for
- `model_name`: Which model this part is for
- `model_colour`: Color of the part
- `extra_descriptions`: Additional notes about the part

**Domain Filtering:**
- When selecting products in order lines, only shows products where `is_a_parts = True`
- Model selection is filtered by selected brand

**Purpose:**
- Distinguish between regular products and mobile parts
- Link parts to specific brands/models
- Help shop staff find the right parts quickly

---

### 9. **Terms and Conditions** (`terms.conditions`)
**Stores terms and conditions text**

- `terms_id`: ID of the terms record
- `terms_conditions`: Text content of terms and conditions

**Purpose:**
Stored terms and conditions are printed on service tickets.

---

### 10. **Mobile Invoice Wizard** (`mobile.invoice`)
**Wizard for creating invoices**

**Fields:**
- `advance_payment_method`: Payment method (Advance or Full amount)
- `amount`: Payment amount
- `number`: Service ID

**Key Function:**
- `action_invoice_create()`: Creates invoice in Odoo's accounting system

**How It Works:**
1. User clicks "Create Invoice" button on service form
2. Wizard opens asking for:
   - Payment method (Advance or Full amount)
   - Amount (if advance payment)
3. System creates invoice with:
   - Service charges (Advance or Full)
   - Parts used (from product_order_line)
4. Invoice is created in Odoo's `account.move` model
5. Invoice count is updated on service form

**Products Created Automatically:**
- "Mobile Service Advance" - for advance payments
- "Mobile Service Charge" - for full service charges

---

### 11. **Service Ticket Report** (`report.mobile_service_shop.mobile_service_ticket_template`)
**Abstract model for generating service ticket PDF**

**Purpose:**
Generates a printable PDF ticket that includes:
- Service number
- Customer name
- IMEI number
- Brand and model
- Complaints
- Technician name
- Dates (request date, return date)
- Warranty information
- Terms and conditions

---

## 🔄 Complete Workflow

### **Step 1: Setup (One-time)**
1. **Create Mobile Brands**
   - Go to Mobile Brands menu
   - Create brands: Samsung, Apple, Xiaomi, etc.

2. **Create Brand Models**
   - Go to Brand Models menu
   - For each brand, create models
   - Example: Apple → iPhone 13, iPhone 14, iPhone 15

3. **Create Complaint Types**
   - Go to Mobile Complaints menu
   - Create complaint types: Screen Broken, Battery Issue, etc.

4. **Create Complaint Descriptions**
   - Go to Mobile Complaint Descriptions menu
   - For each complaint type, create descriptions

5. **Create Products (Parts)**
   - Go to Products menu
   - Check "Is a Mobile Part"
   - Select brand and model
   - Set prices

6. **Set Terms and Conditions**
   - Go to Terms and Conditions menu
   - Add terms text

### **Step 2: Creating a Service Request**
1. **Customer brings in phone**
2. **Staff creates new service request:**
   - Select customer (or create new)
   - Select mobile brand
   - Select model (filtered by brand)
   - Enter IMEI number
   - Check warranty status (if applicable)
   - Enter warranty number (if in warranty)
   - Set request date (today)
   - Set expected return date
   - Select technician
   - Add complaints (can add multiple)
     - Select complaint type
     - Select description (filtered by complaint type)
   - Add parts needed (if any)
     - Select product (only shows mobile parts)
     - Enter quantity
     - Price auto-fills
   - Add internal notes (optional)

3. **Service is in "Draft" state**

### **Step 3: Assigning Service**
1. **Manager/Admin clicks "Assign to technician"**
2. **Status changes to "Assigned"**
3. **Service is now assigned to the selected technician**

### **Step 4: Completing Service**
1. **Technician works on the device**
2. **Technician clicks "Completed" button**
3. **Status changes to "Completed"**

### **Step 5: Creating Invoice**
1. **Manager clicks "Create Invoice" button**
2. **Wizard opens:**
   - Select payment method (Advance or Full amount)
   - Enter amount (if advance)
3. **System creates invoice:**
   - Adds service charge line item
   - Adds all parts used (from product_order_line)
   - Updates `qty_invoiced` for each part
4. **Invoice is created in Odoo Accounting**

### **Step 6: Posting Stock Moves**
1. **Manager clicks "Post Stock moves" button**
2. **System creates stock picking (delivery):**
   - Creates stock.move records for each part
   - Updates `qty_stock_move` for each part
   - Updates inventory (reduces stock)
3. **Parts are deducted from inventory**

### **Step 7: Returning Device**
1. **Manager clicks "Return to customer" button**
2. **Status changes to "Returned"**
3. **Service is complete**

### **Alternative: Not Solved**
1. **If device cannot be repaired:**
2. **Technician clicks "Not Solved" button**
3. **Status changes to "Not Solved"**
4. **Manager can click "Return advance" to refund**

---

## 🔗 Model Relationships

### **One-to-Many Relationships:**
1. **Mobile Brand → Brand Models**
   - One brand has many models
   - Example: Samsung has Galaxy S21, S22, S23, etc.

2. **Mobile Service → Complaint Tree**
   - One service has many complaints
   - Example: One service can have multiple issues

3. **Mobile Service → Product Order Lines**
   - One service uses many parts
   - Example: One repair might need screen + battery + charging port

4. **Mobile Complaint → Complaint Descriptions**
   - One complaint type has many descriptions
   - Example: "Screen Broken" can have different descriptions

### **Many-to-One Relationships:**
1. **Brand Model → Mobile Brand**
   - Many models belong to one brand

2. **Complaint Tree → Mobile Service**
   - Many complaints belong to one service

3. **Complaint Tree → Mobile Complaint**
   - Many complaint trees link to one complaint type

4. **Complaint Tree → Complaint Description**
   - Many complaint trees link to one description

5. **Product Order Line → Mobile Service**
   - Many order lines belong to one service

6. **Product Order Line → Product**
   - Many order lines can use the same product

### **Many-to-Many Relationships:**
1. **Mobile Service → Invoices**
   - One service can have multiple invoices
   - One invoice can be linked to multiple services (via invoice_origin)

---

## 👥 User Roles and Permissions

### **Two User Groups:**

1. **Mobile Service Group Manager** (`mobile_service_group_manager`)
   - **Full access** to all features
   - Can create, read, update, delete services
   - Can create brands, models, complaints
   - Can create invoices
   - Can post stock moves
   - Can send emails
   - Can print tickets

2. **Mobile Service Group Executer** (`mobile_service_group_executer`)
   - **Limited access**
   - Can read and update services (but not create/delete)
   - Can read complaints (but not create/update/delete)
   - Can create/update complaint trees
   - Can create/update product order lines
   - Cannot create invoices
   - Cannot post stock moves
   - Cannot print tickets

**Purpose:**
- **Managers**: Full control, handle billing, inventory
- **Technicians/Executers**: Can update service status, add complaints, but cannot handle financial operations

---

## 💰 Invoice Creation Process

### **How Invoices Work:**

1. **Invoice Creation Wizard:**
   - User selects payment method (Advance or Full amount)
   - If Advance: Enter amount
   - System creates invoice

2. **Invoice Contains:**
   - **Service Charge Line:**
     - If Advance: "Mobile Service Advance" product
     - If Full: "Mobile Service Charge" product
     - Amount: User-specified or calculated
   
   - **Parts Lines:**
     - For each part in `product_order_line`:
       - Only invoices quantity not yet invoiced
       - Calculates: `qty_to_invoice = product_uom_qty - qty_invoiced`
       - Creates invoice line with product, quantity, price
       - Updates `qty_invoiced`

3. **Invoice Validation:**
   - Cannot invoice more than quantity used
   - Cannot create invoice if nothing to invoice
   - Must have income account configured for products

4. **Multiple Invoices:**
   - Can create multiple invoices for same service
   - Each invoice tracks what was invoiced
   - Prevents double-invoicing

---

## 📦 Stock Management

### **How Stock Moves Work:**

1. **When "Post Stock moves" is clicked:**
   - System checks each part in `product_order_line`
   - For each part where `product_uom_qty > qty_stock_move`:
     - Creates stock picking (delivery)
     - Creates stock.move records
     - Updates `qty_stock_move`
     - Reduces inventory

2. **Stock Picking:**
   - Type: Outgoing (Delivery)
   - Partner: Customer
   - Location: From warehouse to customer
   - Origin: Service number (e.g., "SERV/0001")

3. **Validation:**
   - Cannot post stock for more than quantity used
   - Only posts difference: `qty_to_post = product_uom_qty - qty_stock_move`
   - Only creates moves for non-service products (products that have inventory)

---

## 📄 Reports and Tickets

### **Service Ticket:**
- **Format:** PDF
- **Contents:**
  - Service number
  - Customer name and contact
  - IMEI number
  - Brand and model
  - Image of model
  - List of complaints
  - Complaint descriptions
  - Technician name
  - Request date
  - Return date
  - Warranty status
  - Terms and conditions
  - Current date/time

- **Purpose:**
  - Receipt for customer
  - Proof of service
  - Document for warranty claims

### **Email Template:**
- Pre-configured email template
- Can send service updates to customers
- Includes service details

---

## 🎨 User Interface

### **Main Menu Items:**
1. **Mobile Service** - List of all service requests
2. **Mobile Brand** - Manage brands
3. **Brand Models** - Manage models
4. **Mobile Complaints** - Manage complaint types
5. **Mobile Complaint Descriptions** - Manage descriptions
6. **Terms and Conditions** - Manage terms
7. **Products** - Manage products (with mobile parts filter)

### **Service Form View:**
- **Header Buttons:**
  - Assign to technician (Draft state)
  - Completed (Assigned state)
  - Return to customer (Completed state)
  - Not Solved (Assigned state)
  - Create Invoice (Completed/Assigned state)
  - Post Stock moves (Completed/Assigned state)
  - Print Ticket
  - Send email
  - Return advance (Not Solved state)

- **Smart Button:**
  - Invoice count (shows number of invoices)

- **Tabs:**
  - Customer Information
  - Device Information
  - Complaints (One2many)
  - Parts Order Lines (One2many)
  - Internal Notes

---

## 🔒 Security Features

1. **Record Rules:**
   - Users can only see services they have access to
   - Based on user groups

2. **Access Rights:**
   - Managers: Full access
   - Executers: Limited access

3. **State-based Permissions:**
   - Cannot delete services in non-draft state
   - Cannot modify certain fields after assignment
   - Buttons only visible in appropriate states

---

## 📊 Key Features Summary

1. **Automatic Service Number Generation**
   - Uses Odoo sequences
   - Format: SERV/0001, SERV/0002, etc.

2. **Date Validation**
   - Return date must be after request date
   - Validated on change

3. **Domain Filtering**
   - Models filtered by brand
   - Descriptions filtered by complaint type
   - Products filtered to show only mobile parts

4. **Computed Fields**
   - Part price = quantity × unit price
   - Invoice count = number of invoices
   - Customer info auto-filled from partner

5. **State Tracking**
   - Tracks service through lifecycle
   - Buttons appear/disappear based on state
   - Fields become readonly based on state

6. **Inventory Integration**
   - Links to Odoo stock management
   - Tracks parts usage
   - Updates inventory automatically

7. **Accounting Integration**
   - Links to Odoo accounting
   - Creates invoices
   - Tracks payments

8. **Email Integration**
   - Sends emails to customers
   - Uses email templates

9. **Reporting**
   - Generates PDF tickets
   - Includes all service details

---

## 🔧 Technical Details

### **Dependencies:**
- `stock_account`: For inventory and accounting integration
- `mail`: For email functionality and activity tracking
- `product`: For product management
- `account`: For invoice creation

### **Inherited Models:**
- `product.template`: Extended with mobile-specific fields
- `mail.thread`: For activity tracking and email
- `mail.activity.mixin`: For activity management

### **Sequence:**
- `mobile.service`: Generates service numbers

### **Journal:**
- `mobile_service_journal`: Default journal for invoices
- Code: "SERV"
- Type: Sale

### **Products Created:**
- "Mobile Service Advance": For advance payments
- "Mobile Service Charge": For full service charges

---

## 🚀 Usage Example

### **Scenario: Customer brings iPhone 13 with broken screen**

1. **Create Service Request:**
   - Customer: John Doe
   - Brand: Apple
   - Model: iPhone 13
   - IMEI: 123456789012345
   - Complaint: Screen Broken
   - Description: Screen has cracks, touch not working
   - Part: iPhone 13 Screen (Quantity: 1)
   - Technician: Tech A
   - Return Date: 3 days from now

2. **Assign Service:**
   - Manager clicks "Assign to technician"
   - Status: Assigned

3. **Complete Service:**
   - Technician fixes phone
   - Technician clicks "Completed"
   - Status: Completed

4. **Create Invoice:**
   - Manager clicks "Create Invoice"
   - Selects "Full amount"
   - System creates invoice with:
     - Service charge: $50
     - iPhone 13 Screen: $100
     - Total: $150

5. **Post Stock:**
   - Manager clicks "Post Stock moves"
   - System reduces inventory by 1 iPhone 13 Screen

6. **Return Device:**
   - Manager clicks "Return to customer"
   - Status: Returned
   - Customer receives phone and invoice

7. **Print Ticket:**
   - Manager clicks "Print Ticket"
   - PDF generated with all details
   - Customer receives ticket as receipt

---

## 🎓 Key Concepts

1. **State Machine:**
   - Service moves through states
   - Each state has specific actions
   - Cannot skip states

2. **One2many Relationships:**
   - One service has many complaints
   - One service has many parts
   - Allows adding multiple items

3. **Many2one Relationships:**
   - Many models belong to one brand
   - Many complaints belong to one service
   - Links records together

4. **Domain Filtering:**
   - Shows only relevant options
   - Improves user experience
   - Prevents errors

5. **Computed Fields:**
   - Automatically calculated
   - Updates when related fields change
   - No manual input needed

6. **Wizards:**
   - Pop-up windows for specific actions
   - Collects user input
   - Performs complex operations

7. **Reports:**
   - PDF generation
   - Customizable templates
   - Printable documents

---

## 📝 Summary

This addon is a **complete mobile service shop management system** that:
- Tracks service requests from start to finish
- Manages customer information
- Handles mobile brands and models
- Records complaints and issues
- Tracks parts inventory
- Creates invoices
- Manages stock movements
- Generates service tickets
- Sends emails to customers
- Provides role-based access control

It integrates seamlessly with Odoo's core modules (Inventory, Accounting, Mail) to provide a comprehensive solution for mobile service shops.

---

## 🔍 File Structure

```
mobile_service_shop/
├── __init__.py                 # Main initialization
├── __manifest__.py             # Addon configuration
├── models/                     # Data models
│   ├── mobile_service.py       # Main service model
│   ├── mobile_brand.py         # Brand model
│   ├── brand_model.py          # Model model
│   ├── mobile_complaint.py     # Complaint model
│   ├── mobile_complaint_description.py  # Description model
│   ├── mobile_complaint_tree.py # Complaint link model
│   ├── product_order_line.py   # Parts line model
│   ├── product_template.py     # Product extension
│   ├── service_ticket.py       # Ticket report model
│   └── terms_condition.py      # Terms model
├── views/                      # User interface
│   ├── mobile_service_views.xml
│   ├── mobile_brand_views.xml
│   ├── brand_models_views.xml
│   ├── mobile_complaint_views.xml
│   ├── mobile_complaint_description_views.xml
│   ├── product_template_views.xml
│   └── terms_and_condition_views.xml
├── wizard/                     # Wizards
│   └── mobile_create_invoice.py
├── reports/                    # Reports
│   └── mobile_service_ticket.xml
├── security/                   # Security rules
│   ├── ir.model.access.csv
│   └── mobile_service_shop_security.xml
├── data/                       # Initial data
│   ├── mobile_service_data.xml
│   └── mobile_service_email_template.xml
└── static/                     # Static files
    └── src/css/mobile_service.css
```

---

This addon provides a complete, professional solution for managing a mobile service shop with all the necessary features for daily operations, from customer registration to invoice generation and stock management.
