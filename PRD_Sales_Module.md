# Sales Module - Product Requirements Document (PRD)

## 1. Overview
The Sales module is a comprehensive sales order management system that handles customer sales transactions with line items, status tracking, and full integration with the Contacts, Products, Inventory, and Balance modules.

**Technology Stack:**
- Frontend: React 18 with TypeScript, Next.js 15 App Router
- Styling: Tailwind CSS
- Database: Firebase Firestore with real-time synchronization
- Architecture: Service factory pattern with DocumentReference conversion

## 2. Core Features & Capabilities

### 2.1 Sales Order Management
**Create Sales Orders:**
- Customer selection from active contacts via ContactSelect component
- Automatic order number generation with "SO-" prefix and timestamp-based IDs
- Order date specification with current date as default
- Line item management with product selection, quantities, and pricing
- Notes field for additional order information
- Real-time total calculations (subtotal, discount, final total)

**Sales Order Data Model:**
```typescript
interface SalesOrder {
  id: string;
  number: string;          // Auto-generated: "SO-{timestamp}"
  customerId: string;      // Reference to Contact (customer)
  orderDate: string;       // ISO date string
  lineItems: SalesOrderLineItem[];
  subtotal: number;        // Sum of all line item totals
  discount: number;        // Total discount across all line items
  total: number;           // Subtotal minus total discount
  notes?: string;          // Optional additional information
  status: SalesOrderStatus; // 'completed' | 'canceled'
  createdAt: string;       // ISO timestamp
  updatedAt: string;       // ISO timestamp
  deleted?: boolean;       // Soft delete flag
}

interface SalesOrderLineItem {
  id: string;
  productId: string;       // Reference to Product
  quantity: number;        // Whole numbers only
  unitPrice: number;       // Price per unit
  discount: number;        // Discount per unit
  total: number;           // Calculated: (unitPrice - discount) * quantity
}
```

### 2.2 Line Item Management
**Line Item Features:**
- Product selection from active products with sale price auto-population
- Quantity input with validation:
  - Must be positive whole numbers
  - Maximum limit of 999,999 units
  - Real-time inventory availability checking for tracked products
- Unit price modification (auto-populated from product sale price)
- Per-unit discount application
- Real-time line total calculation
- Add, edit, and remove line items dynamically
- Product name and SKU display in line item tables

**Line Item Validation:**
- Product selection is required
- Quantity must be greater than zero and whole numbers only
- Unit price cannot be negative
- Discount cannot be negative or exceed unit price
- Inventory availability check for products with `trackInventory: true`
- Maximum quantity validation to prevent extreme values

### 2.3 Sales Order Listing & Search
**Sales Order Table Features:**
- Displays all non-deleted sales orders
- Sortable columns: Order Number, Customer, Order Date, Total, Status
- Customer name resolution from Contact references
- Status badges with color coding:
  - Completed: Green background
  - Canceled: Red background
- Total amount display with currency formatting
- Order date formatting (MM/DD/YYYY)
- Action buttons: View, Edit (for non-completed orders)

**Table Columns:**
1. Order Number (sortable) - "SO-{timestamp}" format
2. Customer (sortable) - Full name from Contact reference
3. Order Date (sortable) - Formatted date display
4. Total (sortable) - Currency formatted amount
5. Status (sortable) - Color-coded status badges
6. Actions - View and Edit links

### 2.4 Sales Order Detail View
**Comprehensive Order Display:**
- Order header with number, date, and status
- Status change functionality (completed ↔ canceled)
- Customer information section:
  - Full name, company, email, phone from Contact reference
- Order information section:
  - Order date, created timestamp, last updated timestamp
- Line items table with product details:
  - Product name and SKU
  - Unit price, quantity, discount, line total
  - Subtotal, total discount, and final total calculations
- Additional information section for notes
- Action buttons: Back, Edit, Delete (conditional)

**Status Management:**
- Status change dropdown (completed/canceled)
- Real-time status updates with page reload
- Delete button hidden for completed orders
- Conditional actions based on order status

### 2.5 Sales Order Creation & Editing
**Form Features:**
- Customer selection via ContactSelect component (required)
- Order date picker with current date default
- Dynamic line item management:
  - Add new line items with LineItemForm
  - Edit existing line items inline
  - Remove line items with confirmation
  - Real-time totals calculation
- Notes field for additional information
- Form validation with error display
- Submit/Cancel actions with navigation

**Form Validation:**
- Customer selection is required
- At least one line item must be present
- All line item validations apply
- Order date validation
- Form-level error handling and display

### 2.6 Business Logic & Integrations
**Cross-Module Integration:**
- **Contacts Integration:** Customer relationship management
- **Products Integration:** Product selection and pricing
- **Inventory Integration:** Stock availability checking for tracked products
- **Balance Integration:** Financial impact recording (revenue recognition)

**Business Rules:**
- Automatic order number generation with unique "SO-" prefix
- Real-time inventory checking for products with inventory tracking
- Automatic calculation of line totals, subtotals, and final totals
- Status-based action restrictions (e.g., no delete for completed orders)
- Soft delete implementation for data integrity

### 2.7 Firebase Service Implementation
**SalesOrdersFirebaseService Features:**
- Full CRUD operations with DocumentReference conversion
- Automatic order number generation with business logic
- Cross-module data validation and integration
- Real-time inventory and balance impact calculations
- Complex queries with filtering and sorting
- Soft delete implementation
- Status update operations
- Business logic validation for all operations

**Service Methods:**
- `getAll()` - Retrieve all non-deleted sales orders with sorting
- `getById(id)` - Get single sales order with error handling
- `create(data)` - Create with validation and business logic
- `update(id, data)` - Update with validation and recalculations
- `softDelete(id)` - Soft delete with cascade considerations
- `updateSalesOrderStatus(id, status)` - Status change operations
- DocumentReference conversion for Contact integration

## 3. User Interface Components

### 3.1 SalesOrderTable Component
**Features:**
- Responsive table with sorting capabilities
- Customer name resolution from Contact references
- Status badges with appropriate styling
- Currency formatting for totals
- Date formatting for order dates
- Action buttons with conditional display
- Loading states and error handling

### 3.2 SalesOrderForm Component
**Features:**
- Complex form state management with TypeScript
- Customer selection via ContactSelect integration
- Dynamic line item management with LineItemForm
- Real-time calculations and totals display
- Comprehensive form validation
- Error display and handling
- Submit/cancel actions with navigation

### 3.3 LineItemForm Component
**Features:**
- Product selection with price auto-population
- Quantity, unit price, and discount inputs
- Real-time line total calculation
- Inventory availability checking
- Comprehensive validation with error display
- Add/edit/cancel actions
- Currency input formatting

### 3.4 Sales Pages
**Main Sales Page (`/sales`):**
- Header with "New Sale" button
- SalesOrderTable with all sales orders
- Search and filter capabilities
- Responsive layout

**Sales Detail Page (`/sales/[id]`):**
- Comprehensive order display
- Status management interface
- Customer and line item details
- Action buttons (Edit, Delete, Back)
- Status-based conditional features

**New Sales Page (`/sales/new`):**
- SalesOrderForm for creation
- Form header and navigation
- Success/error handling

**Edit Sales Page (`/sales/[id]/edit`):**
- SalesOrderForm with existing data
- Pre-populated form fields
- Update operations and navigation

## 4. Data Flow & State Management

### 4.1 Sales Order Creation Flow
1. User navigates to `/sales/new`
2. SalesOrderForm loads with empty initial state
3. User selects customer via ContactSelect
4. User adds line items via LineItemForm
5. Real-time calculations update totals
6. Form validation on submit
7. SalesOrdersFirebaseService creates order with business logic
8. Navigation to created order detail page

### 4.2 Sales Order Editing Flow
1. User navigates to `/sales/[id]/edit`
2. Existing sales order data loads
3. SalesOrderForm pre-populates with current data
4. User modifies customer, line items, or notes
5. Real-time validation and calculations
6. SalesOrdersFirebaseService updates with validation
7. Navigation to updated order detail page

### 4.3 Line Item Management Flow
1. User clicks "Add Item" in SalesOrderForm
2. LineItemForm component displays
3. User selects product (auto-populates unit price)
4. User enters quantity and optional discount
5. Real-time inventory checking for tracked products
6. Line total calculation and validation
7. Item added to sales order line items array
8. Overall totals recalculated automatically

## 5. Validation & Error Handling

### 5.1 Form Validation
- Customer selection is required for all sales orders
- At least one line item must be present
- All line item fields must be valid
- Order date must be valid
- Real-time validation feedback with error messages

### 5.2 Business Logic Validation
- Inventory availability checking for tracked products
- Quantity limits and whole number requirements
- Price and discount validation
- Status change restrictions
- Cross-module data integrity checks

### 5.3 Error Handling
- Network error handling with user feedback
- Form submission error display
- Loading states during operations
- Graceful degradation for missing data
- Console error logging for debugging

## 6. Integration Points

### 6.1 Contacts Module Integration
- Customer selection via ContactSelect component
- Customer data display in sales orders
- Contact reference validation and resolution

### 6.2 Products Module Integration
- Product selection in line items
- Price auto-population from product sale prices
- Product name and SKU display
- Product availability checking

### 6.3 Inventory Module Integration
- Real-time stock availability checking
- Inventory tracking flag consideration
- Stock level validation for line items

### 6.4 Balance Module Integration
- Revenue recognition for completed sales
- Financial impact calculations
- Balance updates on sales order status changes

## 7. Performance & Scalability

### 7.1 Data Loading
- Efficient Firebase queries with proper indexing
- Lazy loading of related data (contacts, products)
- Optimized re-renders with React state management
- Caching of frequently accessed data

### 7.2 Real-time Features
- Live total calculations without server calls
- Immediate validation feedback
- Real-time inventory checking when needed
- Responsive UI updates

## 8. Security & Data Integrity

### 8.1 Data Validation
- Client-side and server-side validation
- Type safety with TypeScript interfaces
- Business rule enforcement
- Input sanitization and validation

### 8.2 Data Integrity
- Soft delete implementation
- DocumentReference relationships
- Cascade considerations for related data
- Audit trail with created/updated timestamps

This Sales module provides a comprehensive sales order management system with sophisticated line item handling, real-time calculations, cross-module integration, and robust business logic validation while maintaining excellent user experience and data integrity.
