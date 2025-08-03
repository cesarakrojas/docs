# Inventory Module - Product Requirements Document (PRD)

## 1. Overview
The Inventory module is a comprehensive stock management system that tracks product quantities, movements, costs, and provides real-time inventory control with advanced features like stock alerts, purchase recording, adjustments, and detailed movement history.

**Technology Stack:**
- Frontend: React 18 with TypeScript, Next.js 15 App Router
- Styling: Tailwind CSS
- Database: Firebase Firestore with real-time synchronization
- Architecture: Service factory pattern with optimized batch operations

## 2. Core Features & Capabilities

### 2.1 Inventory Management
**Stock Tracking:**
- Real-time stock level monitoring for all tracked products
- Automatic stock calculations based on movement history
- Weighted average cost calculations for accurate valuations
- Product stock caching with optimized read/write operations
- Stock value calculations (quantity × average cost)

**Inventory Data Models:**
```typescript
interface InventoryMovement {
  id: string;
  productId: string;        // Reference to Product
  type: MovementType;       // 'purchase' | 'sale' | 'adjustment'
  quantity: number;         // Positive for inbound, negative for outbound
  unitCost?: number;        // Cost per unit (for purchases/adjustments)
  reason?: AdjustmentReason; // For manual adjustments
  reference?: string;       // PO number, sales order ID, etc.
  notes?: string;           // Additional information
  createdAt: string;        // ISO timestamp
  createdBy?: string;       // User tracking (future enhancement)
}

interface ProductStock {
  productId: string;
  currentStock: number;     // Current quantity on hand
  reorderLevel?: number;    // Minimum stock before reordering
  maxStock?: number;        // Maximum stock level
  avgCost?: number;         // Weighted average cost
  lastMovementAt?: string;  // Last movement timestamp
}
```

### 2.2 Stock Alerts System
**Alert Types:**
- **Low Stock:** Products at or below reorder level
- **Out of Stock:** Products with zero quantity
- **Overstock:** Products exceeding maximum stock level

**Alert Features:**
- Real-time alert generation based on current stock levels
- Visual alerts with color-coded indicators (red, yellow, orange)
- Alert dashboard with count summaries
- Integration with product reorder/max stock settings
- Quick action buttons for purchasing from alerts

### 2.3 Purchase Recording
**Purchase Management:**
- Multi-item purchase recording in single transactions
- Supplier selection from vendor contacts (optional)
- Purchase reference tracking (PO numbers, invoice numbers)
- Purchase date validation (prevents extreme future dates)
- Real-time total calculations with currency formatting

**Purchase Features:**
- Product selection limited to "goods" type only
- Auto-population of unit costs from product purchase prices
- Quantity validation (whole numbers, reasonable limits)
- Batch inventory movement creation
- Optimized stock updates with batch operations

**Purchase Validation:**
- Quantity must be positive whole numbers (max 999,999)
- Unit cost cannot be negative or exceed $999,999
- Purchase date cannot be more than 30 days in future
- Product must exist and have inventory tracking enabled
- Comprehensive error handling and user feedback

### 2.4 Inventory Adjustments
**Adjustment Types:**
- **Damaged:** Items damaged and cannot be sold
- **Lost:** Items lost or missing
- **Returned:** Items returned by customers
- **Found:** Items found during stocktake
- **Correction:** Inventory count corrections
- **Other:** Custom reason (requires notes)

**Adjustment Features:**
- Real-time stock level display before adjustment
- Preview of new stock level after adjustment
- Negative adjustment validation (prevents stock below zero)
- Cost impact calculations for adjustments with unit costs
- Comprehensive adjustment summary before submission

**Adjustment Validation:**
- Cannot reduce stock below zero
- Maximum adjustment limits (999,999 units)
- Required notes for "other" reason type
- Real-time stock availability checking
- Cost validation for cost-related adjustments

### 2.5 Movement History & Tracking
**Movement Display:**
- Complete chronological history of all movements
- Visual icons and color coding by movement type
- Detailed movement information including costs and references
- Date and time stamps for all transactions
- Filtering by movement type, date ranges

**Movement Filtering:**
- Filter by movement type (purchase, sale, adjustment)
- Date range filtering (from/to dates)
- Product-specific movement history
- Clear filters functionality
- Real-time filter application

### 2.6 Stock Level Dashboard
**Dashboard Features:**
- Comprehensive stock table with product details
- Current stock, average cost, and stock value display
- Product information including name, SKU, description
- Visual indicators for low stock and out of stock items
- Reorder level displays and stock status indicators

**Dashboard Calculations:**
- Stock value: Current stock × (Average cost || Purchase price)
- Color-coded rows: Red (out of stock), Yellow (low stock)
- Sale price and average cost comparisons
- Quick access to movement history per product

### 2.7 Cross-Module Integration
**Products Integration:**
- Only tracks inventory for products with `trackInventory: true`
- Respects product type (goods vs services)
- Uses product purchase/sale prices for calculations
- Integrates with product reorder levels and max stock settings

**Sales Integration:**
- Automatic stock reduction on sales order completion
- Real-time inventory availability checking for sales
- Sales movement tracking with order references
- Reversal of inventory movements for canceled sales

**Contacts Integration:**
- Supplier selection from vendor-type contacts
- Supplier information display in purchase records

## 3. User Interface Components

### 3.1 Inventory Dashboard Page (`/inventory`)
**Main Features:**
- Tabbed interface: Stock Levels and Stock Alerts
- Stock alerts summary with count and quick preview
- Action buttons: Add New Product, New Purchase, Stock Adjustment
- Settings modal integration for inventory configurations

**Stock Levels Table:**
- Product name, SKU, and description
- Current stock with reorder level indicators
- Average cost and sale price display
- Stock value calculations
- Actions: View Movements link

**Stock Alerts Tab:**
- Alert list with color-coded indicators
- Alert type badges (Out of Stock, Low Stock, Overstock)
- Quick purchase action for low/out of stock items
- Current stock vs reorder/max level display

### 3.2 Purchase Recording Page (`/inventory/purchase`)
**Form Features:**
- Purchase information section (supplier, reference, date)
- Dynamic multi-item purchase interface
- Add/remove items functionality
- Real-time total calculations
- Product selection limited to goods with inventory tracking

**Purchase Items:**
- Product selection with auto-cost population
- Quantity and unit cost inputs with validation
- Line total calculations
- Remove item functionality (minimum 1 item)
- Comprehensive form validation and error display

### 3.3 Stock Adjustment Page (`/inventory/adjustment`)
**Adjustment Features:**
- Product selection with current stock display
- Adjustment quantity input (positive/negative)
- Real-time new stock level preview
- Adjustment reason selection with descriptions
- Optional unit cost for cost adjustments

**Validation & Preview:**
- Current stock vs adjustment quantity validation
- New stock level preview with color coding
- Cost impact calculations
- Comprehensive adjustment summary
- Detailed form validation

### 3.4 Movement History Page (`/inventory/movements/[productId]`)
**Movement Display:**
- Product information header with current stock
- Movement filtering controls (type, date range)
- Chronological movement list with details
- Visual movement type indicators
- Quick action buttons for new movements

**Movement Details:**
- Movement type badges with color coding
- Quantity changes (+ for inbound, - for outbound)
- Unit costs and total values
- References and notes display
- Date and time stamps

## 4. Business Logic & Algorithms

### 4.1 Stock Calculation Algorithm
**Real-time Stock Calculation:**
1. Retrieve all movements for product
2. Sum positive movements (purchases, positive adjustments)
3. Subtract negative movements (sales, negative adjustments)
4. Calculate weighted average cost from purchase movements
5. Cache results for performance optimization
6. Prevent negative stock display (minimum 0)

**Average Cost Calculation:**
- Weighted average based on purchase quantities and costs
- Only considers positive quantity movements with unit costs
- Updates automatically with new purchase movements
- Used for stock valuation calculations

### 4.2 Stock Alert Generation
**Alert Logic:**
1. Compare current stock to reorder level (low stock)
2. Check for zero stock (out of stock)
3. Compare current stock to max stock (overstock)
4. Generate alerts with appropriate types and priorities
5. Filter alerts by product inventory tracking settings

### 4.3 Batch Operations Optimization
**Performance Optimizations:**
- Batch stock updates for multiple movements
- Cached stock document reads/writes
- Optimized Firebase queries with indexing
- Incremental stock updates vs full recalculation
- Transaction safety for critical operations

## 5. Firebase Service Implementation

### 5.1 FirebaseInventoryService Features
**Core Operations:**
- Initialize product stock on product creation
- Record purchases with multi-item support
- Record sales movements from sales orders
- Record manual adjustments with validation
- Calculate and cache product stock levels

**Advanced Features:**
- Batch stock update operations
- Movement history queries with filtering
- Stock alert generation
- Purchase history reporting
- Real-time movement subscriptions

**Service Methods:**
- `initializeProductStock(productId)` - Initialize stock for new products
- `recordPurchase(data)` - Record multi-item purchases
- `recordSale(saleId, items)` - Record sales movements
- `recordAdjustment(data)` - Record manual adjustments
- `calculateProductStock(productId)` - Get current stock with caching
- `getAllProductStock()` - Bulk stock retrieval for dashboard
- `getStockAlerts()` - Generate current stock alerts
- `getInventoryMovements(filters)` - Query movements with filtering

### 5.2 Data Integrity & Validation
**Business Rule Enforcement:**
- Only track inventory for products with `trackInventory: true`
- Prevent negative stock through validation
- Validate movement types and quantities
- Ensure referential integrity with products
- Cost validation and reasonable limits

**Error Handling:**
- Comprehensive validation with user-friendly messages
- Network error handling with retry logic
- Database transaction safety
- Graceful degradation for missing data
- Console logging for debugging

## 6. Integration Points

### 6.1 Products Module Integration
- Inventory tracking flag enforcement
- Product type filtering (goods only)
- Price auto-population from products
- Reorder level and max stock settings

### 6.2 Sales Module Integration
- Automatic stock reduction on sales
- Real-time availability checking
- Sales movement tracking
- Reversal for canceled orders

### 6.3 Contacts Module Integration
- Supplier selection from vendors
- Contact validation and display

### 6.4 Balance Module Integration
- Cost impact tracking for financial reporting
- Inventory value calculations
- Purchase cost recording

## 7. Performance & Scalability

### 7.1 Optimization Strategies
- Stock calculation caching with incremental updates
- Batch operations for multiple movements
- Efficient Firebase queries with proper indexing
- Lazy loading of movement history
- Real-time subscriptions for live updates

### 7.2 Data Management
- Movement history retention policies
- Archived movement data management
- Stock recalculation utilities
- Database cleanup and maintenance

## 8. Security & Data Validation

### 8.1 Input Validation
- Quantity limits and type validation
- Cost validation with reasonable limits
- Date validation for realistic ranges
- Product existence and tracking validation
- User input sanitization

### 8.2 Business Logic Security
- Stock level protection (no negative stock)
- Movement type validation
- Reference data integrity
- Audit trail maintenance
- Transaction atomicity

## 9. User Experience Features

### 9.1 Real-time Features
- Live stock level updates
- Immediate calculation feedback
- Real-time form validation
- Dynamic total calculations
- Instant filter application

### 9.2 Visual Design
- Color-coded stock status indicators
- Movement type icons and badges
- Progress indicators for operations
- Responsive table designs
- Intuitive navigation patterns

## 10. Future Enhancement Capabilities

### 10.1 Advanced Features Ready for Implementation
- Multi-warehouse inventory tracking
- Barcode scanning integration
- Automated reorder suggestions
- Inventory forecasting
- Advanced reporting and analytics

### 10.2 Settings Integration
- Configurable stock thresholds
- Default adjustment reasons
- Inventory tracking preferences
- Alert notification settings
- Warehouse management settings

This Inventory module provides a comprehensive, efficient, and user-friendly inventory management system with real-time tracking, advanced business logic validation, cross-module integration, and scalable architecture for future enhancements.
