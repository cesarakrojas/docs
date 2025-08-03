# Product Requirements Document (PRD)
## Products Module - NextJS ERP System

### Document Information
- **Version**: 1.0
- **Date**: August 2, 2025
- **Module**: Products
- **System**: Web-based ERP for Micro and Small Businesses

---

## 1. Executive Summary

The Products module is a core component of the NextJS ERP system that manages product and service catalog information. It provides comprehensive product management capabilities including creation, editing, viewing, searching, and organization of inventory items and services with full CRUD operations, vendor relationships, inventory tracking settings, and Firebase backend integration.

---

## 2. Module Overview

### 2.1 Purpose
The Products module serves as the central catalog management system for both physical goods and services, storing essential product information that integrates with Sales, Inventory, and other ERP modules. It supports pricing management, vendor relationships, and inventory control settings.

### 2.2 Current Implementation Status
- **Technology Stack**: Next.js 15, React 18, TypeScript, Firebase Firestore, Tailwind CSS
- **Architecture**: Firebase-only backend with real-time data synchronization
- **Status**: Fully implemented and functional

---

## 3. Functional Requirements

### 3.1 Product Data Model

#### Product Entity Structure:
```typescript
Product {
  id: string;                    // Unique identifier (auto-generated)
  name: string;                  // Required product/service name
  sku?: string;                  // Optional Stock Keeping Unit
  type: 'goods' | 'services';    // Product classification
  purchasePrice: number;         // Cost from vendor/supplier
  salePrice: number;            // Standard selling price (required)
  vendorIds?: string[];         // Array of vendor Contact IDs
  description?: string;         // Optional product description
  // Inventory Management Fields
  reorderLevel?: number;        // Minimum stock before reordering
  maxStock?: number;           // Maximum stock level
  trackInventory?: boolean;    // Enable inventory tracking
  createdAt: string;           // Auto-generated timestamp
  updatedAt: string;          // Auto-updated timestamp
  isDeleted: boolean;         // Soft delete flag
}
```

### 3.2 Core Features

#### 3.2.1 Product Management Operations

**Create Product**
- Form-based product creation with validation
- Product type selection (Goods/Services)
- Required fields: name, salePrice
- Optional fields: sku, description, purchasePrice, vendor assignments
- Inventory management settings (conditional on product type)
- Auto-generation of timestamps
- Firebase Firestore persistence with DocumentReference conversion

**View Product Details**
- Individual product detail page
- Complete product information display
- Product type badge with color coding
- Price formatting with currency display
- Vendor information with clickable links to contact details
- Inventory management settings display
- Action buttons (Edit, Delete)
- Creation and update timestamp display

**Edit Product**
- Pre-populated form with existing data
- All fields editable except timestamps
- Form validation and vendor relationship validation
- Price field conversion (string to number)
- Confirmation on successful update

**Delete Product**
- Soft delete implementation (isDeleted flag)
- Confirmation dialog before deletion
- Product remains in database but filtered from views

#### 3.2.2 Product Listing and Management

**Products Table View**
- Tabular display of all active products
- Columns: Name, SKU, Type, Purchase Price, Sale Price, Actions
- Product type color-coded badges:
  - Goods: Green (bg-green-100 text-green-800)
  - Services: Blue (bg-blue-100 text-blue-800)
- Client-side sorting on all columns with visual indicators
- Currency formatting for price columns
- Product description display as secondary text
- Row action buttons (Edit, Delete)

**Search and Filtering**
- Real-time search functionality
- Search across: name, SKU, description
- Type-based filtering (All Types, Goods, Services)
- Combined search and filter capabilities
- Clear filters functionality
- Client-side search implementation due to Firestore limitations

**Advanced Query Capabilities**
- Server-side filtering by product type
- Optimized Firebase queries with proper ordering
- Active products filtering (isDeleted = false)
- Results count display

#### 3.2.3 Product Form Component

**Comprehensive Form Fields**
- Product name (required text input)
- SKU (optional text input)
- Product type (required dropdown: Goods/Services)
- Purchase price (optional number input with currency prefix)
- Sale price (required number input with currency prefix)
- Multiple vendor selection via ContactSelect component
- Description (optional textarea)

**Inventory Management Section**
- Track inventory checkbox (defaults to true for goods, false for services)
- Conditional inventory fields (only shown when tracking enabled):
  - Reorder level (minimum stock threshold)
  - Maximum stock (maximum stock level)
- Contextual help text based on product type

**Form Validation**
- Required field validation (name, salePrice)
- Numeric validation for prices (non-negative purchase price, positive sale price)
- Vendor existence validation
- Real-time error feedback
- Form submission prevention on validation errors

### 3.3 User Interface Features

#### 3.3.1 Navigation and Layout
- Main products page accessible via `/products`
- Breadcrumb navigation with "Products & Services" title
- "Add New" button prominently displayed
- Settings modal integration

#### 3.3.2 Responsive Design
- Mobile-responsive table design with horizontal scrolling
- Optimized form layouts for different screen sizes
- Touch-friendly interface elements
- Conditional column display on smaller screens

#### 3.3.3 Visual Design Elements
- Product type badges with distinct colors
- Currency formatting for all price displays
- Loading states during data operations
- Empty state messaging
- Product count display

---

## 4. Technical Implementation

### 4.1 Architecture

#### 4.1.1 Service Layer
- **ProductsFirebaseService**: Main service class extending FirebaseService
- **Firebase Integration**: Real-time Firestore database with DocumentReference conversion
- **Service Factory Pattern**: Centralized service access via `getProductsService()`

#### 4.1.2 Data Access Methods
```typescript
// CRUD Operations
createProduct(productData: ProductFormData): Promise<Product>
updateProduct(id: string, productData: Partial<ProductFormData>): Promise<Product>
getById(id: string): Promise<Product | null>
getAll(queryOptions?: FirestoreQuery): Promise<Product[]>
softDelete(id: string): Promise<Product>

// Specialized Queries
getProductsByType(type: ProductType): Promise<Product[]>
getGoods(): Promise<Product[]>
getServices(): Promise<Product[]>
searchProducts(searchTerm: string): Promise<Product[]>
getProductsBySku(sku: string): Promise<Product[]>
getProductsByVendor(vendorId: string): Promise<Product[]>
getInventoryProducts(): Promise<Product[]>
getLowStockProducts(): Promise<Product[]>
```

### 4.2 Component Structure

#### 4.2.1 Page Components
- **ProductsPage** (`/products`): Main listing page with search and filters
- **NewProductPage** (`/products/new`): Product creation page
- **ProductDetailPage** (`/products/[id]`): Individual product view
- **EditProductPage** (`/products/[id]/edit`): Product editing page

#### 4.2.2 Reusable Components
- **ProductForm**: Comprehensive form component for create/edit operations
- **ProductTable**: Table component for listing products with sorting
- **ContactSelect**: Integrated for vendor selection (multi-select)

### 4.3 Data Validation and Processing

#### 4.3.1 Required Fields
- name (text input with validation)
- salePrice (number input, must be > 0)
- type (dropdown selection)

#### 4.3.2 Optional Fields
- sku (text input)
- purchasePrice (number input, must be >= 0)
- vendorIds (multi-select from contacts)
- description (textarea)
- inventory management fields (conditional)

#### 4.3.3 Firebase Data Conversion
- Vendor IDs converted to DocumentReferences for storage
- DocumentReferences converted back to IDs for display
- Price field string-to-number conversion
- Conditional field inclusion (only store defined values)

---

## 5. Integration Points

### 5.1 Module Dependencies

#### 5.1.1 Contacts Module Integration
- Vendor selection from active vendor contacts
- Vendor relationship management
- Vendor contact details linking
- Vendor validation and cleanup

#### 5.1.2 Sales Module Integration
- Product selection in sales order line items
- Price copying from product to sales order (historical pricing)
- Product lookup for sales transactions
- Revenue tracking per product (via sales history)

#### 5.1.3 Inventory Module Integration
- Goods-only filtering for inventory operations
- Inventory tracking settings enforcement
- Stock level management for tracked products
- Purchase order product selection
- Inventory adjustment product selection

#### 5.1.4 Settings Integration
- Product module settings via SettingsModal
- Future configurations (placeholder ready):
  - Default product type
  - Auto-generate SKU
  - Track inventory by default

### 5.2 Firebase Integration

#### 5.2.1 Firestore Collection
- Collection name: `products`
- Document structure with DocumentReference conversion
- Optimized queries for filtering and sorting
- Compound queries for type and vendor filtering

#### 5.2.2 Real-time Capabilities
- Live data synchronization
- Automatic UI updates on data changes
- Optimistic UI updates for better UX

---

## 6. User Workflows

### 6.1 Create New Product Workflow
1. Navigate to Products page
2. Click "Add New" button
3. Fill product form with required information
4. Select product type (Goods/Services)
5. Set pricing information
6. Optionally assign vendors
7. Configure inventory tracking (if applicable)
8. Submit form
9. Redirect to product detail view

### 6.2 View Product Details Workflow
1. Navigate to Products page
2. Click on product name in table
3. View complete product information including vendor relationships
4. Access Edit or Delete actions
5. Navigate back to products list

### 6.3 Edit Product Workflow
1. Access product via View page or direct Edit link
2. Modify product information in pre-populated form
3. Update vendor relationships if needed
4. Submit changes
5. View updated product details
6. Return to products list

### 6.4 Search and Filter Workflow
1. Use search input to filter products by name, SKU, or description
2. Select type filter to show only goods or services
3. Click column headers to sort results
4. Use clear button to reset all filters
5. View filtered/sorted results with count

### 6.5 Vendor Management Workflow
1. Select vendors during product creation/editing
2. View vendor relationships in product details
3. Navigate to vendor contact details
4. Manage vendor-product associations

---

## 7. Performance Considerations

### 7.1 Query Optimization
- Server-side type filtering to reduce data transfer
- Optimized Firestore queries with proper indexing
- Efficient vendor relationship queries using array-contains
- Client-side sorting for immediate responsiveness

### 7.2 User Experience
- Loading states during data fetching
- Optimistic UI updates for immediate feedback
- Responsive design for various screen sizes
- Progressive data loading

---

## 8. Business Logic

### 8.1 Product Type Behavior
- **Goods**: Default inventory tracking enabled, shown in inventory operations
- **Services**: Default inventory tracking disabled, excluded from inventory operations

### 8.2 Pricing Management
- Purchase price optional (can be $0.00)
- Sale price required and must be > $0.00
- Price fields support decimal values (currency formatting)
- Historical pricing preserved in sales orders

### 8.3 Vendor Relationships
- Multiple vendors per product supported
- Vendor validation ensures active, non-deleted contacts
- Vendor selection filtered to vendor-type contacts only
- Broken vendor relationships cleaned up automatically

---

## 9. Error Handling

### 9.1 Form Validation Errors
- Real-time validation feedback
- Clear error messaging for required fields
- Numeric validation for price fields
- Vendor relationship validation

### 9.2 Network and Database Errors
- Console error logging for debugging
- Graceful handling of connection issues
- Loading state management
- Empty state handling

---

## 10. Security and Data Privacy

### 10.1 Data Access
- Firebase security rules (implementation not visible in current codebase)
- Soft delete to maintain data integrity
- Vendor relationship validation

### 10.2 Input Validation
- Client-side validation for data integrity
- Server-side validation through Firebase rules (assumed)
- Price field sanitization

---

## 11. Testing Infrastructure

### 11.1 Test Files (Currently Empty)
- ProductForm.spec.tsx (placeholder)

### 11.2 Testing Requirements (Future)
- Component unit tests
- Integration tests for Firebase operations
- E2E tests for complete workflows
- Vendor relationship testing

---

## 12. Future Enhancement Opportunities

### 12.1 Advanced Features
- Barcode generation and scanning
- Product images and media management
- Product categories and tags
- Bulk import/export functionality
- Product variants and options

### 12.2 Inventory Enhancements
- Advanced stock alerting
- Automated reordering
- Multi-location inventory
- Batch/lot tracking

### 12.3 Analytics and Reporting
- Product performance analytics
- Profit margin analysis
- Vendor performance tracking
- Sales trend analysis

---

## 13. Limitations and Known Issues

### 13.1 Current Limitations
- Search functionality is client-side only due to Firestore constraints
- No duplicate SKU detection
- No product image support
- No product categorization beyond type
- Empty test files indicate incomplete testing coverage

### 13.2 Technical Debt
- Test implementation needed
- Advanced search capabilities needed
- Product image management not implemented
- Settings integration is placeholder-ready but not fully implemented

---

## 14. Data Flow and Integration Patterns

### 14.1 Product Selection Flow
1. **Sales Orders**: Products selected via dropdown with price auto-population
2. **Inventory Operations**: Goods-only filtering for relevant operations
3. **Vendor Management**: Bi-directional relationship management

### 14.2 Price Management Flow
1. Purchase price set during product creation/editing
2. Sale price copied to sales order line items (historical snapshot)
3. Price updates don't affect existing sales orders

### 14.3 Inventory Integration Flow
1. Product type determines inventory tracking eligibility
2. Inventory settings control stock management behavior
3. Stock levels managed separately in inventory module

---

## 15. Conclusion

The Products module provides a comprehensive and well-architected foundation for product and service catalog management within the NextJS ERP system. It successfully implements all core CRUD operations with advanced features like vendor relationship management, inventory tracking configuration, and seamless integration with other ERP modules. The module follows React/Next.js best practices with robust Firebase integration and demonstrates excellent separation of concerns. While there are opportunities for enhancement in areas like advanced search, product categorization, and media management, the current implementation fully meets the requirements for product catalog management in a small business ERP context.
