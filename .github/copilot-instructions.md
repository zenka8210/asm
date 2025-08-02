# AI Coding Instructions for E-Commerce DATN Project

## 🏗️ Project Architecture

This is a **full-stack Next.js + Node.js e-commerce platform** with a clear separation between frontend (`asm/fe/`) and backend (`asm/server/`).

### Core Structure
```
asm/
├── fe/                    # Next.js frontend (TypeScript)
│   ├── src/
│   │   ├── services/      # API calls (17 backend routes)
│   │   ├── types/         # TypeScript interfaces from BE schemas
│   │   ├── contexts/      # Global state management
│   │   ├── hooks/         # Reusable logic + service calls
│   │   └── app/           # UI/UX components
└── server/                # Node.js/Express backend
    ├── models/            # Mongoose schemas
    ├── controllers/       # Route handlers extending BaseController
    ├── services/          # Business logic extending BaseService  
    ├── middlewares/       # QueryBuilder, auth, error handling
    └── routes/            # 17 API routes
```

## 🎯 Frontend Development Guidelines

### Layer Responsibility Rules
1. **Component needs global state** → Use context (`useAuth`, `useCart`, etc.)
2. **Component needs data fetching + loading/error** → Use custom hook
3. **Component needs one-time action** → Call service directly
4. **Duplicate logic across components** → Extract to custom hook

### Key Frontend Patterns
- **Services**: Singleton classes with static methods for API calls
- **Hooks**: Wrap services with loading/error states, no global state
- **Contexts**: Only global state (auth, cart, wishlist, notifications)
- **Types**: 100% TypeScript coverage, interfaces match BE schemas

### Development Commands
```bash
# Frontend development
cd "d:\ReactJs\Datn\asm\fe"
npm run dev    # Runs on port 3002 with Turbo

# Backend development  
cd "d:\ReactJs\Datn\asm\server"
npm run dev    # Nodemon on port 5000
```

## 🔧 Backend Architecture Patterns

### Service-Controller Pattern
- **Controllers**: Extend `BaseController`, handle HTTP logic
- **Services**: Extend `BaseService`, contain business logic
- **Models**: Mongoose schemas with pre/post hooks and statics

### Query Middleware System
The backend uses a sophisticated `QueryBuilder` class for all data operations:

```javascript
// In controllers - prefer QueryBuilder when available
if (req.createQueryBuilder) {
    const queryBuilder = req.createQueryBuilder(Model);
    const result = await queryBuilder
        .search(['name', 'description'])
        .applyFilters({ category: { type: 'objectId' } })
        .paginate()
        .execute();
}
```

### **⚠️ CRITICAL: Schema Field Verification for Backend Development**

**BEFORE editing any backend service, controller, or model code that involves database operations, you MUST:**

1. **Always check the actual schema files first** using `read_file` or `grep_search` to verify field names
2. **Never assume field names** - schemas may differ from expectations
3. **Verify all related schemas** when working with populate/relationships

#### Schema Verification Checklist:
```bash
# Check schema files before editing populate configurations
server/models/[ModelName]Schema.js

# Common schemas to verify:
- UserSchema.js (name, email, phone, role, etc.)
- ProductSchema.js (name, images, price, salePrice, etc.)  
- ProductVariantSchema.js (product, color, size, stock, etc.)
- ColorSchema.js (name, isActive) - NO hexCode field!
- SizeSchema.js (name) 
- OrderSchema.js (user, items, address, voucher, etc.)
- AddressSchema.js (fullName, phone, addressLine, ward, district, city)
- PaymentMethodSchema.js (method, isActive) - NOT name/type!
- VoucherSchema.js (code, discountPercent, minimumOrderValue, etc.)
```

#### Example Schema Verification Process:
```javascript
// ❌ WRONG - Assuming field names without checking
populate: [
  { path: 'color', select: 'name hexCode' }, // hexCode doesn't exist!
  { path: 'paymentMethod', select: 'name type' } // Wrong fields!
]

// ✅ CORRECT - After checking ColorSchema.js and PaymentMethodSchema.js  
populate: [
  { path: 'color', select: 'name isActive' }, // Verified fields
  { path: 'paymentMethod', select: 'method isActive' } // Correct fields
]
```

#### Backend Development Workflow:
1. **Read the target schema file(s)** to understand exact field names
2. **Check related schemas** for populate operations  
3. **Verify field types** (String, Number, ObjectId, Boolean, etc.)
4. **Test with sample data** to ensure populate works correctly
5. **Update frontend types** if schema changes affect API responses

### Critical Index Management
**NEVER create duplicate indexes** - this causes mongoose warnings:
- Remove `index: true` from schema fields that already have `unique: true`
- Don't add `Schema.index()` calls for fields already indexed

## 🚀 Development Workflows

### Starting Development
1. **Backend first**: Start server (`npm run dev` in `/server/`)  
2. **Frontend second**: Start Next.js (`npm run dev` in `/fe/`)
3. **Testing**: Use comprehensive test suites in `/server/test*.js`

### API Testing
Backend has extensive test coverage:
```bash
# Complete API testing
npm run test-all              # Full test suite
node testAllAPIs_main.js      # Comprehensive tests
node testOrder.js             # Order-specific tests
node testProduct.js           # Product-specific tests
```

### Database Operations
```bash
npm run seed                  # Seed database with test data
npm run cleanup              # Clean test data
npm run verify-database      # Verify DB state
```

## 🎨 UI/UX Design System

### Color System (Critical for Consistency)
- **Primary Blue** (`#1E40AF`): Buttons, links, **all prices**
- **Error Red** (`#DC2626`): Only sale badges and error states, **NOT prices**
- **Success Green** (`#10B981`): Success states and savings
- **Background**: White (`#FFFFFF`)

### Reusable Component Guidelines
**CRITICAL**: Always use components from `app/components/ui/` for consistency.

#### UI Component Architecture
```
app/
├── components/
│   └── ui/                    # Reusable UI/UX components
│       ├── Button.tsx         # Button component with variants
│       ├── Pagination.tsx     # Pagination with consistent styling
│       ├── PageHeader.tsx     # Page header for all pages
│       ├── Modal.tsx          # Modal dialogs
│       └── index.ts           # Export all UI components
└── (routes)/
    └── page.tsx               # Import UI components from @/app/components/ui
```

#### Button Component Usage
```typescript
// ✅ CORRECT - Use reusable Button component
import { Button } from '@/app/components/ui';

<Button variant="primary" size="md" onClick={handleClick}>
  Tìm kiếm
</Button>

// ❌ INCORRECT - Custom button styling
<button className="custom-btn search-btn">Tìm kiếm</button>
```

#### Pagination Component Usage  
```typescript
// ✅ CORRECT - Use reusable Pagination component
import { Pagination } from '@/app/components/ui';

<Pagination
  currentPage={currentPage}
  totalPages={totalPages}
  onPageChange={handlePageChange}
  showInfo={true}
  totalItems={totalProducts}
  itemsPerPage={limit}
/>
```

#### PageHeader Component Usage
```typescript
// ✅ CORRECT - Use reusable PageHeader component
import { PageHeader } from '@/app/components/ui';

<PageHeader
  title="Sản phẩm mới"
  subtitle="Khám phá những sản phẩm mới nhất của chúng tôi"
  breadcrumbs={[
    { label: 'Trang chủ', href: '/' },
    { label: 'Sản phẩm mới', href: '/new' }
  ]}
  showSearch={true}
  actions={<CustomActions />}
/>
```

#### Component Architecture Rules
1. **Reusable UI Components**: Always check `app/components/ui/` first
2. **Consistent Styling**: Use existing Button, Pagination, PageHeader, Modal components
3. **Mobile-First Design**: All components must be responsive
4. **Loading States**: All interactions must have loading indicators
5. **Error Handling**: All components must handle error states gracefully

### Standard Page Layout Architecture
**CRITICAL**: All pages must follow the consistent layout pattern established across the application.

#### Page Structure Template
```typescript
// ✅ CORRECT - Standard layout pattern (used in /new, /profile, etc.)
export default function SamplePage() {
  return (
    <div className="container">
      <div className={styles.pageContainer}>
        {/* Page Header - ALWAYS inside container */}
        <PageHeader
          title="Page Title"
          subtitle="Page description"
          icon={IconComponent}
          breadcrumbs={[
            { label: 'Trang chủ', href: '/' },
            { label: 'Current Page', href: '/current' }
          ]}
        />

        {/* Main Content */}
        <div className={styles.mainContent}>
          <div className="row">
            {/* Sidebar (if needed) */}
            <div className="col-3">
              <div className={styles.sidebarCard}>
                {/* Sidebar content */}
              </div>
            </div>
            
            {/* Main Content Area */}
            <div className="col-9">
              <div className={styles.contentCard}>
                {/* Page content */}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}

// ❌ INCORRECT - PageHeader outside container
export default function BadPage() {
  return (
    <div className={styles.pageContainer}>
      <PageHeader /> {/* Wrong: Header outside container */}
      <div className="container">
        {/* Content */}
      </div>
    </div>
  );
}
```

#### CSS Structure Requirements
```css
/* ✅ CORRECT - Standard page CSS structure */
.pageContainer {
  padding: 2rem 0;
  background: transparent; /* Let global styles handle background */
}

.mainContent {
  padding: 2rem 0 0 0; /* Top padding for spacing after PageHeader */
  position: relative;
  z-index: 1;
}

/* ❌ INCORRECT - Conflicting with global layout */
.pageContainer {
  min-height: 100vh;
  background: linear-gradient(...); /* Don't override global backgrounds */
  padding: 0; /* Wrong: No padding breaks spacing */
}
```

#### Layout Hierarchy Rules
1. **Container Wrapper**: All pages start with `<div className="container">`
2. **PageHeader Placement**: Always inside the container, before main content
3. **Grid System**: Use 12-column grid (`row`, `col-3`, `col-9`) for layouts
4. **Card Components**: Wrap content sections in reusable card components
5. **Responsive Design**: Ensure all layouts work on mobile/tablet/desktop

#### Navigation & Icon Consistency
```css
/* ✅ CORRECT - Proper icon color management */
.navItemActive .navIcon,
.navItemActive .navLabel {
  color: white !important; /* Ensure text visibility on active states */
}

.overviewCard .cardIcon svg {
  color: white !important; /* Force icon colors in specific contexts */
  fill: white !important;
}

.overviewCard .cardIcon * {
  color: white !important; /* Override all child elements */
  fill: white !important;
}
```

#### CSS Specificity & Naming Best Practices
**CRITICAL**: Prevent CSS conflicts and ensure consistent styling across components.

```css
/* ✅ CORRECT - Context-specific class naming */
.overviewCard .cardIcon {
  /* Styles specific to overview cards */
}

.securityCard .cardIcon {
  /* Different styles for security section icons */
}

/* ✅ CORRECT - Use parent selectors for specificity */
.navItemActive .navLabel {
  color: white !important; /* Ensures text visibility */
}

/* ❌ INCORRECT - Generic class names causing conflicts */
.cardIcon {
  color: #333; /* This will conflict with other .cardIcon uses */
}

/* ❌ INCORRECT - Missing parent context */
.navLabel {
  color: white; /* May not override inherited styles */
}
```

#### CSS Debugging & Override Strategies
1. **Use Parent Selectors**: Always scope styles with parent classes
2. **Important Declarations**: Use `!important` for critical overrides (icons, text visibility)
3. **Multiple Selectors**: Target both `color` and `fill` for SVG icons
4. **Universal Selectors**: Use `*` to override all child elements when needed
5. **Hover State Management**: Ensure text remains visible on hover/active states

#### Common Layout Pitfalls to Avoid
- **Hero Sections**: Don't create oversized hero sections that break 12-grid layout
- **PageHeader Position**: Never place PageHeader outside the container wrapper
- **Background Conflicts**: Don't override global background in page-specific CSS
- **Icon Colors**: Always specify both `color` and `fill` with `!important` for SVG icons
- **Active States**: Ensure text remains visible on hover/active navigation states

### Development Rules
- **New UI Components**: Place reusable components in `app/components/ui/`
- **Component Imports**: Use `import { ComponentName } from '@/app/components/ui'`
- **Styling Consistency**: Follow existing color system and spacing
- **TypeScript**: All components must have proper TypeScript interfaces
- **Testing**: Verify responsive design on mobile/tablet/desktop

## 🔗 Integration Points

### Frontend-Backend Communication
- **Base URL**: `http://localhost:5000/api`
- **17 API Routes**: Auth, products, cart, orders, etc.
- **Authentication**: JWT tokens in headers
- **Session Support**: Guest cart/wishlist functionality

### File Structure Dependencies
- `types/` interfaces must match `server/models/` schemas
- `services/` methods correspond to backend route endpoints
- Component props should use TypeScript interfaces from `types/`

### Vietnamese Toast Notification System
**CRITICAL**: All toast notifications must display in Vietnamese only for consistent UX.

#### Implementation Guidelines
1. **Use `useApiNotification` Hook**: Never use `useNotification` directly in components
```typescript
import { useApiNotification } from '@/hooks';

const { showSuccess, showError, handleApiResponse } = useApiNotification();
// All messages automatically translated to Vietnamese
showSuccess('Operation completed successfully');
showError('Operation failed', error);
```

2. **Context Integration**: WishlistContext and CartContext use `useApiNotification`
```typescript
// ✅ CORRECT - Auto-translation
const { showSuccess, showError } = useApiNotification();
showSuccess('Added to wishlist'); // → "Đã thêm vào danh sách yêu thích"

// ❌ INCORRECT - Mixed language messages
const { success, error } = useNotification();
success('Added to wishlist'); // May show English message
```

3. **Message Translation Rules**:
   - **Backend Error Messages**: Automatically translated via `translateError()`
   - **Success Messages**: Translated via `translateSuccess()`
   - **API Response Handling**: Use `handleApiResponse()` for consistent processing
   - **Vietnamese Detection**: Messages already in Vietnamese remain unchanged

4. **Translation Coverage**:
   - Authentication: Login, logout, registration, password changes
   - Products: Add/edit/delete, inventory messages
   - Cart: Add/remove items, checkout processes
   - Wishlist: Add/remove items, sync operations
   - Orders: Status updates, cancellations
   - Validation: Form errors, required fields
   - Network: Connection errors, timeouts

#### File Locations
- **Translation Utility**: `lib/messageTranslator.ts`
- **Notification Hook**: `hooks/useApiNotification.ts`
- **Context Integration**: `contexts/NotificationContext.tsx`

#### Development Rules
- **New Components**: Always use `useApiNotification` for user feedback
- **API Calls**: Wrap responses with `handleApiResponse()` or specific show methods
- **Error Handling**: Use `handleApiError()` for consistent error display
- **Testing**: Verify all notifications display in Vietnamese during development

## 📋 Common Tasks & Patterns

### Adding New Features
1. **Backend**: Model → Service → Controller → Route
2. **Frontend**: Type → Service → Hook/Context → Component
3. **Testing**: Add tests to appropriate `test*.js` files

### Error Handling Pattern
```typescript
// Frontend hook pattern
const { data, loading, error, refetch } = useApiCall(service.method);

// Backend controller pattern  
try {
  const result = await this.service.method(req.body);
  ResponseHandler.success(res, 'Success message', result);
} catch (error) {
  next(error); // Handled by errorHandler middleware
}
```

### Admin vs User Patterns
- Admin routes: `/admin/*` - different layout without header/footer
- Admin permissions: Checked via `req.user?.role === 'admin'`
- Admin sorting: Uses `AdminSortUtils.ensureAdminSort()`

## 🔐 Admin Dashboard Patterns & Permissions

### Admin Route Architecture
**Middleware Chain Pattern**: All admin routes follow consistent security layers:
```javascript
router.get('/admin/endpoint', 
  authenticateToken,     // JWT verification
  adminMiddleware,       // Role check (admin only)
  queryParserMiddleware(), // Query handling
  controller.method
);
```

### Permission Matrix by Resource
| Resource | Public Routes | Protected Routes | Admin Routes |
|----------|--------------|------------------|--------------|
| **Products** | `GET /public` | None | Full CRUD + Stats |
| **Orders** | None | User's orders only | All orders + Management |
| **Users** | None | Own profile only | Full user management |
| **Categories** | `GET /public` | None | Full CRUD + Hierarchy |
| **Cart/Wishlist** | Guest session | User's items | View all (no CRUD) |
| **Reviews** | `GET /product/:id` | CRUD own reviews | View/Delete any |
| **Statistics** | None | None | Dashboard analytics |

### Frontend Admin Layout Structure
```tsx
// Admin Layout Pattern - NO header/footer for admin pages
export default function AdminLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className={styles.adminContainer}>
      <div className={styles.dashboardHeader}>Admin Dashboard</div>
      <div className={styles.adminMainWrapper}>
        <nav className={styles.adminNav}>
          {/* Admin Navigation Menu */}
          <ul className={styles.adminNavList}>
            <li><Link href="/admin">Tổng quan</Link></li>
            <li><Link href="/admin/products">Quản lý sản phẩm</Link></li>
            <li><Link href="/admin/orders">Quản lý đơn hàng</Link></li>
            <li><Link href="/admin/users">Quản lý người dùng</Link></li>
            {/* ... more admin sections */}
          </ul>
        </nav>
        <div style={{ flex: 1 }}>{children}</div>
      </div>
    </div>
  );
}
```

### Role-Based Authentication Checks
```typescript
// Frontend: Auth context check for admin access
const { user } = useAuth();
useEffect(() => {
  if (!user || user.role !== "admin") {
    router.replace("/login");
    return;
  }
}, [user]);

// Backend: Admin middleware implementation
const adminMiddleware = (req, res, next) => {
  if (req.user && req.user.role === ROLES.ADMIN) {
    return next();
  }
  return next(new AppError('Yêu cầu quyền Admin', ERROR_CODES.FORBIDDEN));
};
```

### Query Middleware & Admin Sorting
All admin endpoints use sophisticated query handling:
```javascript
// Admin QueryBuilder pattern with search, filters, pagination
const result = await req.createQueryBuilder(Model)
  .search(['name', 'email', 'orderCode'])
  .applyFilters({
    status: { type: 'string' },
    user: { type: 'objectId' },
    'finalTotal[min]': { type: 'number' },
    'finalTotal[max]': { type: 'number' }
  })
  .execute();

// AdminSortUtils ensures consistent sorting
const sortConfig = AdminSortUtils.ensureAdminSort(req, 'ModelName');
```

### Admin-Specific Business Rules
1. **Wishlist Restrictions**: Admins can VIEW all wishlists but CANNOT add/edit/delete items
2. **Order Management**: Only admins can update order status, cancel orders with suitable status ( can cancel only if status is 'pending' or 'processing')
3. **User Management**: Full CRUD + role changes + activate/deactivate
4. **Statistics Access**: Only admins can access dashboard analytics

### Complete Admin API Endpoints (17 Routes)
```javascript
// Products - Full CRUD + Statistics
GET    /api/products                     // List with filters
POST   /api/products                     // Create new
PUT    /api/products/:id                 // Update
DELETE /api/products/:id                 // Delete
GET    /api/products/admin/out-of-stock  // Inventory alerts
GET    /api/products/admin/statistics    // Dashboard stats

// Orders - Full Management
GET    /api/orders/admin/all             // All orders with query
GET    /api/orders/admin/statistics      // Order analytics
PUT    /api/orders/admin/:id/status      // Update order status
GET    /api/orders/admin/trends          // Trend analysis
GET    /api/orders/admin/user/:userId    // User order history

// Users - Complete Management
GET    /api/users                        // List all users
POST   /api/users                        // Create user
PUT    /api/users/:id                    // Update user
DELETE /api/users/:id                    // Delete user
PATCH  /api/users/:id/role              // Change user role
PATCH  /api/users/:id/status            // Activate/deactivate
GET    /api/users/stats                 // User statistics

// Categories - Hierarchy Management
GET    /api/categories                   // Admin view with stats
POST   /api/categories                   // Create category
PUT    /api/categories/:id              // Update category
DELETE /api/categories/:id              // Delete category
GET    /api/categories/:id/can-delete   // Check deletion safety

// Reviews - Moderation
GET    /api/reviews/admin/all            // All reviews
DELETE /api/reviews/admin/:id           // Delete any review
PUT    /api/reviews/admin/:id/approve   // Approve review
GET    /api/reviews/admin/pending       // Pending reviews

// Cart/Orders - View Only
GET    /api/cart/admin/all               // All carts/orders
GET    /api/cart/admin/statistics        // Cart analytics
GET    /api/cart/admin/trends            // Usage trends

// Wishlist - Read Only Access
GET    /api/wishlist/admin/stats         // Wishlist analytics
GET    /api/wishlist/admin/all           // All wishlists
GET    /api/wishlist/admin/user/:userId  // User wishlist

// System Management
GET    /api/statistics/dashboard         // Main dashboard
GET    /api/vouchers/admin              // Voucher management
GET    /api/banners                     // Banner management
GET    /api/payment-methods             // Payment management
```

### Dashboard Statistics Integration
```typescript
// Frontend: Admin dashboard stats fetching
const [stats, setStats] = useState({
  earnings: 0,
  users: 0,
  orders: 0,
  products: 0
});

// Backend: Statistics service methods
getDashboardStats()     // Overview numbers
getRevenueChart()      // Revenue over time
getTopProductsChart()  // Best sellers
getOrderStatusChart()  // Status distribution
getUserRegistrationChart() // User growth
```

### Error Handling for Admin Operations
```typescript
// Frontend: Admin-specific error handling
const handleAdminAction = async (action: () => Promise<any>) => {
  try {
    await action();
    showSuccessToast('Operation completed successfully');
  } catch (error) {
    if (error.response?.status === 403) {
      showErrorToast('Insufficient admin permissions');
    } else if (error.response?.status === 401) {
      router.push('/login');
    } else {
      showErrorToast('Operation failed');
    }
  }
};
```

---

**Key Admin Files to Reference:**
- `server/middlewares/adminMiddleware.js` - Permission enforcement
- `server/utils/adminSortUtils.js` - Consistent sorting logic
- `fe/src/app/admin/layout.tsx` - Admin-only layout pattern
- `server/docs/KIEN_TRUC_ROUTES.md` - Complete permission matrix

## 🐛 Common Issues & Solutions

### Database Connection
- MongoDB Atlas connection string in `.env`
- Check server logs for connection status
- Test with `npm run verify-database`

### CORS Issues  
- Frontend runs on port 3002, backend on 5000
- CORS configured for localhost:3000, 3001, 3002
- Check `corsOptions` in `server/app.js`

### Mongoose Warnings
- **Duplicate indexes**: Remove redundant index declarations
- **Deprecation warnings**: Check mongoose connection options

## 💡 Architecture Decisions

### Why This Structure?
- **Frontend**: Clean separation of concerns, reusable logic
- **Backend**: Scalable service layer, consistent query handling  
- **Testing**: Comprehensive coverage of all API endpoints
- **Types**: End-to-end type safety from database to UI

### Performance Considerations
- QueryBuilder optimizes database queries
- Frontend hooks prevent unnecessary re-renders
- Image optimization via Next.js Image component
- Pagination implemented across all list views

---

**Key Files to Reference:**
- `fe/DEVELOPMENT_GUIDELINES.md.md` - Frontend architecture rules
- `fe/UI-UX-DESIGN-PATTERNS.md` - Design system specifications  
- `server/controllers/baseController.js` - Controller patterns
- `server/middlewares/queryMiddleware.js` - Query handling system

## 🛡️ MANDATORY Schema Verification Protocol

### **🚨 BEFORE EDITING ANY BACKEND CODE:**

**Step 1: Identify Schema Dependencies**
```bash
# For any backend edit involving database operations:
1. Identify main model (e.g., Order, Product, User)
2. List all related models (via populate, references)
3. Note all field names being used in the operation
```

**Step 2: Verify Schema Fields**
```bash
# Use these commands to check actual schema structure:
read_file server/models/[MainModel]Schema.js
read_file server/models/[RelatedModel]Schema.js
grep_search "field_name" server/models/

# Common mistakes to avoid:
- ColorSchema: has 'name', 'isActive' - NO 'hexCode'
- PaymentMethodSchema: has 'method', 'isActive' - NO 'name' or 'type'  
- SizeSchema: only has 'name' - NO other fields
- AddressSchema: has 'addressLine' - NO 'street' or 'address'
```

**Step 3: Cross-Reference with Frontend Types**
```bash
# Ensure frontend types match backend schema:
read_file fe/src/types/index.ts
# Update frontend types if backend schema has changed
```

**Step 4: Test Critical Operations**
```bash
# After making changes, verify these work:
- Populate operations return expected data
- Frontend can consume the API response
- No MongoDB/Mongoose errors in server logs
```

### **🎯 Schema Field Quick Reference:**

| Model | Key Fields | Common Mistakes |
|-------|------------|-----------------|
| **ColorSchema** | `name`, `isActive` | ❌ `hexCode` doesn't exist |
| **SizeSchema** | `name` | ❌ No other fields |
| **PaymentMethodSchema** | `method`, `isActive` | ❌ Not `name` or `type` |
| **AddressSchema** | `fullName`, `phone`, `addressLine`, `ward`, `district`, `city` | ❌ Not `street` |
| **ProductSchema** | `name`, `images`, `price`, `salePrice`, `category` | ✅ Standard fields |
| **OrderSchema** | `orderCode`, `user`, `items`, `address`, `voucher`, `total`, `finalTotal` | ✅ Standard fields |

### **💡 When Schema Verification is Required:**

1. **Adding populate() calls** - Check all populated model schemas
2. **Creating/updating query filters** - Verify field names exist
3. **Modifying API responses** - Ensure frontend types match
4. **Adding new database operations** - Check field types and constraints
5. **Fixing bugs related to "undefined" fields** - Usually schema mismatch

### **🔍 Schema Debugging Commands:**

```bash
# To find actual field names in a schema:
grep_search "type.*:" server/models/ColorSchema.js

# To see all available fields:
read_file server/models/[Schema].js 1 50

# To find where a field is used:
grep_search "fieldName" server/services/
grep_search "fieldName" server/controllers/
```

## 🔑 Testing Credentials & Database Configuration

### **Standard Admin & User Credentials for API Testing**

**CRITICAL**: Always use these standardized credentials when creating API test scripts to ensure consistency across all testing scenarios.

#### Admin Account
```bash
# Admin Login Credentials
Email: huy.nguyen.huu@gmail.com
Password: huuhuy82
Role: admin
```

#### Customer Account  
```bash
# Customer Login Credentials  
Email: huy.nguyen.huu@gmail.com
Password: huuhuy82
Role: customer
```

#### Database Configuration
```bash
# Environment Variables (.env in /server/)
DB_URI: mongodb+srv://finodev01:0101001@cluster0.pfafcr6.mongodb.net/asm?retryWrites=true&w=majority
PORT: 5000
JWT_SECRET: Nd83jsDJKJd8sklsjk89JDF893JdjsjlsdkfjsKLDJFL89sdjH
JWT_EXPIRES_IN: 7d
```

### **API Testing Template**

When creating any API test script, use this standard authentication template:

```javascript
// ✅ CORRECT - Standard API testing pattern
const axios = require('axios');

const ADMIN_CREDENTIALS = {
  email: 'huy.nguyen.huu@gmail.com',
  password: 'huuhuy82'
};

const CUSTOMER_CREDENTIALS = {
  email: 'huy.nguyen.huu@gmail.com', 
  password: 'huuhuy82'
};

async function getAdminToken() {
  const response = await axios.post('http://localhost:5000/api/auth/login', ADMIN_CREDENTIALS);
  // CRITICAL: Token is in response.data.data.token, NOT response.data.token
  return response.data.data.token;
}

async function getCustomerToken() {
  const response = await axios.post('http://localhost:5000/api/auth/login', CUSTOMER_CREDENTIALS);
  // CRITICAL: Token is in response.data.data.token, NOT response.data.token
  return response.data.data.token;
}

// Use in tests
const adminToken = await getAdminToken();
const headers = { 'Authorization': `Bearer ${adminToken}` };
```

### **⚠️ CRITICAL Token Extraction**

**The login response structure is:**
```javascript
{
  success: true,
  message: 'Thành công',
  data: {
    user: { ... },
    token: 'eyJhbGciOiJIUzI1NiIs...' // Token is HERE
  }
}
```

**Always use: `response.data.data.token` NOT `response.data.token`**

### **API Testing Template**

When creating any API test script, use this standard authentication template:

```javascript
// ✅ CORRECT - Standard API testing pattern
const axios = require('axios');

const ADMIN_CREDENTIALS = {
  email: 'huy.nguyen.huu@gmail.com',
  password: 'huuhuy82'
};

const CUSTOMER_CREDENTIALS = {
  email: 'huy.nguyen.huu@gmail.com', 
  password: 'huuhuy82'
};

async function getAdminToken() {
  const response = await axios.post('http://localhost:5000/api/auth/login', ADMIN_CREDENTIALS);
  // CRITICAL: Token is in response.data.data.token, NOT response.data.token
  return response.data.data.token;
}

async function getCustomerToken() {
  const response = await axios.post('http://localhost:5000/api/auth/login', CUSTOMER_CREDENTIALS);
  // CRITICAL: Token is in response.data.data.token, NOT response.data.token
  return response.data.data.token;
}

// Use in tests
const adminToken = await getAdminToken();
const headers = { 'Authorization': `Bearer ${adminToken}` };
```

### **Test Account Management**

#### Account Verification Commands
```bash
# Verify admin account exists
node -e "
const User = require('./models/UserSchema');
require('./config/db');
setTimeout(async () => {
  const admin = await User.findOne({ email: 'tran.thi.lan@gmail.com' });
  console.log('Admin account:', admin ? 'EXISTS' : 'NOT FOUND');
  process.exit(0);
}, 2000);
"

# Create admin account if missing
node createAdminUser.js
```

#### Testing Database State
```bash
# Check database connection and basic collections
node verifyDatabase.js

# Seed test data if needed
node seedDatabase.js

# Clean test data after testing
node cleanAllDatabases.js
```

### **Development Environment Standards**

#### Server Setup
```bash
# Backend server (Port 5000)
cd d:\ReactJs\Datn-admin\Datn\asm\server
npm run dev

# Frontend server (Port 3002) 
cd d:\ReactJs\Datn-admin\Datn\asm\fe
npm run dev
```

#### Authentication Flow Testing
1. **Login Verification**: Always test both admin and customer login
2. **Token Validation**: Verify JWT tokens work for protected routes
3. **Permission Testing**: Confirm admin-only endpoints reject customer access
4. **Session Management**: Test token expiration and refresh if applicable

### **Common Testing Patterns**

#### Backend API Testing
```bash
# Full API test suite
node testAllAPIs_main.js

# Specific module tests
node testPost.js          # Posts API
node testOrder.js         # Orders API  
node testProduct.js       # Products API
node testUser.js          # Users API
```

#### Frontend Integration Testing
```bash
# Test frontend API services
cd asm/fe
node test-dashboard-apis.js
```

---

**Key Testing Files to Reference:**
- `server/testAllAPIs_main.js` - Comprehensive API testing
- `server/createAdminUser.js` - Admin account creation
- `server/verifyDatabase.js` - Database connection verification
- `fe/test-dashboard-apis.js` - Frontend API integration tests
