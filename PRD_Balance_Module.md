# Balance Module - Product Requirements Document (PRD)

## 1. Overview
The Balance module is a comprehensive financial transaction management system that tracks all money movements, provides financial summaries, and integrates with other ERP modules to maintain complete financial records for business operations.

**Technology Stack:**
- Frontend: React 18 with TypeScript, Next.js 15 App Router
- Styling: Tailwind CSS
- Database: Firebase Firestore with optimized indexing
- Architecture: Service factory pattern with cross-module integration

## 2. Core Features & Capabilities

### 2.1 Financial Transaction Management
**Transaction Types:**
- **Income Transactions:** Revenue from sales, services, and other income sources
- **Expense Transactions:** Costs for purchases, operations, and other expenses

**Transaction Data Model:**
```typescript
interface BalanceEntry {
  id: string;
  date: string;                    // ISO date string
  type: BalanceType;              // 'income' | 'expense'
  amount: number;                 // Transaction amount
  category: BalanceCategory;      // Transaction classification
  reference?: string;             // Reference to sales orders, purchases, etc.
  description: string;            // Transaction description
  status: TransactionStatus;      // 'pending' | 'completed' | 'cancelled'
  currency: string;               // Multi-currency support (USD default)
  userId?: string;                // User tracking (future enhancement)
  metadata?: Record<string, any>; // Extensibility for future features
  createdAt: string;              // ISO timestamp
  updatedAt: string;              // ISO timestamp
  isDeleted: boolean;             // Soft delete flag
}

type BalanceCategory = 'sales' | 'purchase' | 'adjustment' | 'manual' | 'other';
type TransactionStatus = 'pending' | 'completed' | 'cancelled';
```

### 2.2 Financial Dashboard & Summary
**Dashboard Features:**
- Real-time financial overview with total income, expenses, and net balance
- Responsive summary cards with color-coded indicators
- Transaction count and trend indicators
- Mobile-optimized compact layouts
- Period-based filtering and summaries

**Summary Calculations:**
```typescript
interface BalanceSummary {
  totalIncome: number;    // Sum of all income transactions
  totalExpense: number;   // Sum of all expense transactions
  netBalance: number;     // Total income minus total expenses
  currency: string;       // Display currency
  periodStart?: string;   // Optional period start date
  periodEnd?: string;     // Optional period end date
}
```

### 2.3 Advanced Filtering & Search
**Filter Options:**
- **Transaction Type:** Income, Expense, or All
- **Category:** Sales, Purchase, Adjustment, Manual, Other
- **Date Range:** From/To date filtering
- **Status:** Pending, Completed, Cancelled
- **Search:** Text search in transaction descriptions
- **Reference:** Search by reference numbers

**Optimized Firebase Queries:**
- Server-side filtering with proper indexing
- Date range queries with compound indexes
- Type and category combination queries
- Efficient pagination for large datasets

### 2.4 Expense Recording System
**Manual Expense Entry:**
- Category-based expense classification
- Description and reference tracking
- Date and amount validation
- Optional supplier selection from vendor contacts

**Product Purchase Integration:**
- Integrated purchase recording for inventory items
- Multi-item purchase transactions
- Supplier selection from vendor contacts
- Automatic inventory movement creation
- Cost tracking with unit prices and quantities

**Expense Categories:**
- Product Purchases (with inventory integration)
- Office Supplies
- Marketing
- Travel
- Utilities
- Rent
- Insurance
- Professional Services
- Other (custom categories)

### 2.5 Cross-Module Integration
**Sales Module Integration:**
- Automatic income recording for completed sales orders
- Revenue recognition with order references
- Customer transaction tracking
- Sales cancellation handling

**Inventory Module Integration:**
- Purchase expense recording with inventory movements
- Cost tracking for product purchases
- Supplier relationship management
- Inventory value impact calculations

**Products Module Integration:**
- Product selection in purchase transactions
- Purchase price auto-population
- Supplier relationship validation

**Contacts Module Integration:**
- Supplier selection from vendor-type contacts
- Contact information display in transactions

### 2.6 Transaction Table & Display
**Desktop Table Features:**
- Comprehensive table with all transaction details
- Sortable columns for date, type, amount, category
- Color-coded transaction types (green for income, red for expenses)
- Category badges with distinct colors
- Status indicators with appropriate styling
- Reference display with monospace formatting

**Mobile-Optimized Display:**
- Compact card-based layout for mobile devices
- Two-column design with essential information
- Icon-based type indicators
- Responsive amount display
- Touch-friendly interaction elements

### 2.7 Financial Analytics & Reporting
**Summary Analytics:**
- Total income, expenses, and net balance calculations
- Period-based financial summaries
- Category breakdown analysis
- Monthly financial summaries by year

**Advanced Analytics Methods:**
- `getBalanceSummary(startDate?, endDate?)` - Period-based summaries
- `getBalanceByCategoryBreakdown(startDate?, endDate?)` - Category analysis
- `getMonthlyBalanceSummary(year)` - Monthly financial overview

## 3. User Interface Components

### 3.1 Balance Dashboard Page (`/balance`)
**Main Features:**
- Financial summary cards with real-time calculations
- Advanced filtering interface (desktop and mobile)
- Tabbed transaction view (All, Income, Expenses)
- Search functionality with real-time filtering
- Settings modal integration

**Responsive Design:**
- Desktop: Full-featured filter panel with labels
- Mobile: Compact filter dropdowns with collapsible date inputs
- Adaptive layouts for different screen sizes

### 3.2 New Expense Page (`/balance/new-expense`)
**Expense Form Features:**
- Category-based expense recording
- Date and amount validation
- Description and reference fields
- Supplier selection for purchases

**Product Purchase Integration:**
- Multi-item purchase interface when "Product Purchases" selected
- Dynamic item addition/removal
- Product selection with auto-cost population
- Real-time total calculations
- Inventory integration for tracked products

### 3.3 BalanceTable Component
**Features:**
- Responsive table design (desktop) and card layout (mobile)
- Transaction type indicators with icons
- Color-coded amounts (green/red)
- Category badges with distinct styling
- Status badges for transaction states
- Action buttons for edit/delete operations

### 3.4 BalanceSummary Component
**Summary Display:**
- Total income with positive indicator
- Total expenses with negative indicator
- Net balance calculation
- Currency formatting
- Responsive card layout
- Period indicator when filtered

## 4. Firebase Service Implementation

### 4.1 BalanceFirebaseService Features
**Core Operations:**
- Full CRUD operations with soft delete support
- Optimized queries with proper Firebase indexing
- Cross-module integration methods
- Financial analytics and reporting

**Service Methods:**
- `createBalanceEntry(data)` - Create new transaction
- `updateBalanceEntry(id, data)` - Update existing transaction
- `getBalanceEntriesByType(type)` - Filter by income/expense
- `getBalanceEntriesByCategory(category)` - Filter by category
- `getBalanceEntriesByDateRange(start, end)` - Date filtering
- `getBalanceSummary(start?, end?)` - Financial summaries
- `searchBalanceEntries(term)` - Text search functionality

### 4.2 Integration Methods
**Cross-Module Transaction Recording:**
```typescript
// Sales integration
recordSaleTransaction(saleId, amount, description, date?)
cancelSaleTransaction(saleId)

// Purchase integration  
recordPurchaseTransaction(purchaseId, amount, description, date?)
```

### 4.3 Data Optimization
**Firebase Indexing Strategy:**
- Compound indexes for efficient filtering
- Date-based queries optimization
- Type and category combination indexes
- Soft delete considerations in all queries

## 5. Business Logic & Validation

### 5.1 Transaction Validation
**Financial Rules:**
- Amount must be positive and reasonable
- Date validation (not too far in future/past)
- Required fields enforcement (date, type, amount, description)
- Category validation against allowed values
- Currency format validation

**Integration Validation:**
- Reference validation for cross-module transactions
- Supplier validation for purchase transactions
- Product selection validation for inventory purchases
- Status transition validation

### 5.2 Financial Calculations
**Real-time Calculations:**
- Running balance calculations
- Period-based summaries
- Category breakdown percentages
- Net income/loss calculations

### 5.3 Error Handling
**Comprehensive Error Management:**
- Form validation with user-friendly messages
- Network error handling with retry logic
- Data integrity validation
- Cross-module integration error handling

## 6. Integration Points

### 6.1 Sales Module Integration
- Automatic income recording for completed sales
- Revenue recognition with proper categorization
- Sales order reference tracking
- Customer transaction history

### 6.2 Inventory Module Integration
- Purchase expense recording for inventory items
- Cost tracking for inventory movements
- Supplier purchase transaction recording
- Inventory value impact calculations

### 6.3 Products Module Integration
- Product selection in purchase transactions
- Price auto-population from product data
- Supplier relationship validation

### 6.4 Contacts Module Integration
- Supplier selection from vendor contacts
- Contact validation and relationship management

## 7. Performance & Scalability

### 7.1 Query Optimization
**Efficient Data Access:**
- Optimized Firebase indexes for common queries
- Server-side filtering to reduce data transfer
- Pagination support for large datasets
- Cached calculations for performance

### 7.2 Real-time Features
**Live Updates:**
- Real-time balance calculations
- Immediate filter application
- Dynamic form validation
- Responsive UI updates

## 8. Security & Data Integrity

### 8.1 Data Validation
**Input Security:**
- Amount validation with reasonable limits
- Date range validation
- Description content validation
- Reference format validation

### 8.2 Financial Data Integrity
**Audit Trail:**
- Complete transaction history
- Created/updated timestamps
- Soft delete for data preservation
- Cross-module reference integrity

## 9. Mobile Responsiveness

### 9.1 Mobile-First Design
**Touch-Optimized Interface:**
- Compact filter controls with dropdown design
- Card-based transaction display
- Touch-friendly buttons and inputs
- Collapsible filter sections

### 9.2 Progressive Enhancement
**Adaptive Layouts:**
- Desktop: Full-featured table with all columns
- Mobile: Compact card layout with essential information
- Responsive filter panels
- Optimized touch targets

## 10. Future Enhancement Capabilities

### 10.1 Advanced Features Ready for Implementation
**Financial Reporting:**
- Monthly/quarterly financial reports
- Profit & loss statements
- Cash flow analysis
- Tax reporting support

**Multi-currency Support:**
- Currency conversion rates
- Multi-currency transaction recording
- Exchange rate tracking
- Currency-specific reporting

### 10.2 Settings Integration
**Configurable Options:**
- Default currency settings
- Fiscal year configuration
- Category customization
- Report preferences

### 10.3 Analytics & Insights
**Business Intelligence:**
- Trend analysis and forecasting
- Category spending patterns
- Seasonal financial analysis
- Performance indicators

## 11. Data Export & Backup

### 11.1 Export Capabilities
**Data Export Features:**
- CSV export for accounting software integration
- PDF reports for documentation
- Date range export options
- Category-specific exports

### 11.2 Integration Readiness
**External System Integration:**
- Accounting software APIs
- Banking integration preparation
- Tax software compatibility
- Audit trail export

This Balance module provides a comprehensive financial management system with real-time tracking, cross-module integration, advanced filtering capabilities, and scalable architecture for future financial management enhancements.
