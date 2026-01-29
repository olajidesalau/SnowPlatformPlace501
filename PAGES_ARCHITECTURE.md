# SnowPlatform Pages Visual Architecture

## Site Map

```
SnowPlatform (Root)
│
├── PUBLIC PAGES (No Auth Required)
│   ├── / (HomePage)
│   │   └── CTAs → /providers/search or /auth/register
│   │
│   ├── /auth/login (LoginPage)
│   │   └── Demo accounts shown
│   │
│   ├── /auth/register (RegisterPage)
│   │   └── Role selection: Customer or Provider
│   │
│   ├── /providers/search (ProvidersSearchPage)
│   │   ├── Filters: postcode, service, rating, price
│   │   └── Actions: View Profile, Message
│   │
│   └── /providers/:userId (ProviderProfilePage)
│       ├── Profile info, rating, reviews
│       └── Services list with pricing
│
├── CUSTOMER PAGES (Auth + Customer Role)
│   ├── /bookings/customer (BookingsPage)
│   │   ├── Status: Pending, Confirmed, In Progress, Completed
│   │   └── Actions: View Details, Leave Review, Cancel
│   │
│   ├── /messages (MessagesPage)
│   │   ├── Conversation list
│   │   └── Message thread
│   │
│   └── /notifications (NotificationsPage)
│       ├── Notification feed
│       └── Mark as read
│
├── PROVIDER PAGES (Auth + Provider Role)
│   ├── /providers/profile (ProviderProfilePage - Edit Mode)
│   │   ├── Business info
│   │   ├── Services offered
│   │   └── Service areas
│   │
│   ├── /bookings/provider (BookingsPage)
│   │   ├── Incoming bookings
│   │   └── Booking actions
│   │
│   ├── /messages (MessagesPage)
│   │   └── Customer conversations
│   │
│   └── /notifications (NotificationsPage)
│       └── Booking notifications
│
└── ADMIN PAGES (Auth + Admin Role)
    ├── /admin/dashboard (AdminDashboardPage)
    │   ├── Stats cards (6 metrics)
    │   ├── Quick actions (4 buttons)
    │   └── Recent activity table
    │
    ├── /admin/users (AdminUsersPage)
    │   ├── Search & filter
    │   ├── Users table
    │   └── User actions (View, Suspend)
    │
    ├── /admin/users/:userId (User detail - future)
    │   └── Individual user management
    │
    ├── /admin/providers (AdminProvidersPage)
    │   ├── Status tabs (All, Pending, Approved, Rejected)
    │   ├── Providers table
    │   └── Actions: View, Approve, Reject
    │
    ├── /admin/providers/:providerId (Provider detail - future)
    │   └── DBS verification, insurance, ratings
    │
    ├── /admin/bookings (AdminBookingsPage)
    │   ├── Bookings table (all statuses)
    │   ├── Status breakdown stats
    │   └── View booking details
    │
    └── /admin/stats (Detailed statistics - future)
        └── Charts and detailed metrics

```

## Navigation Flow

### From Home Page
```
HomePage
├─→ /auth/login (Login button)
├─→ /auth/register (Register button)  
└─→ /providers/search (Search button)
```

### Customer Flow
```
LoginPage → /bookings/customer → Details
          ├─→ /providers/search
          ├─→ /messages
          └─→ /notifications
```

### Provider Flow
```
RegisterPage (as Provider) → Approval Queue
                          ↓
                    /providers/profile → /services
                          ├─→ /bookings/provider
                          ├─→ /messages
                          └─→ /notifications
```

### Admin Flow
```
AdminDashboardPage
├─→ /admin/users → View/Suspend users
├─→ /admin/providers → Approve/Reject providers
└─→ /admin/bookings → View all bookings
```

## Component Hierarchy

```
App
├── Navigation (shows based on user role)
│   ├── PublicNav (Home, Search, Login/Register)
│   ├── CustomerNav (Home, Search, Bookings, Messages, Notifications)
│   ├── ProviderNav (Home, Profile, Services, Bookings, Messages)
│   └── AdminNav (Home, Dashboard, Users, Providers, Bookings, Stats)
│
└── Page Component
    ├── Hero/Header
    ├── Filters/Search (if applicable)
    ├── Content Grid/Table
    │   ├── Cards/Rows
    │   └── Actions (View, Edit, Delete, etc.)
    └── Pagination (if applicable)
```

## Data Flow

```
User Action (Click, Submit Form)
           ↓
         Page Handler
           ↓
    API Endpoint / DB Query
           ↓
    Data Processing
           ↓
    JSX Rendering
           ↓
    HTML Response
           ↓
    Browser Display
```

## Authentication Flow

```
Anonymous User
    ↓
/auth/login or /auth/register
    ↓
POST /api/auth/login or /api/auth/register
    ↓
Generate JWT Token
    ↓
Store Token (localStorage)
    ↓
Redirect to role-based dashboard
    ↓
Authenticated User
    ↓
Access role-specific pages
```

## Page Structure Template

```
Page Component
├── Header Section
│   ├── Title
│   ├── Subtitle (if needed)
│   └── Back/Action buttons
│
├── Filters/Search Section (if applicable)
│   ├── Input fields
│   └── Submit button
│
├── Content Section
│   ├── Empty state (if no data)
│   │   └── Message + Action
│   │
│   └── Data Display
│       ├── Stats cards (admin pages)
│       ├── Grid of cards (search results)
│       │   └── Card elements
│       │       ├── Title/Name
│       │       ├── Details
│       │       └── Actions
│       │
│       └── Table (list views)
│           ├── Header row
│           └── Data rows
│               └── Actions (View, Edit, Delete)
│
└── Pagination (if needed)
```

## Page Complexity Levels

**Simple Pages (No Data Fetching):**
- LoginPage
- RegisterPage
- HomePage

**Medium Pages (Single API Call):**
- ProvidersSearchPage
- ProviderProfilePage
- NotificationsPage
- AdminDashboardPage

**Complex Pages (Multiple API Calls + Filtering):**
- BookingsPage
- MessagesPage
- AdminUsersPage
- AdminProvidersPage
- AdminBookingsPage

## States Each Page Can Have

### LoadingState
```
⏳ Loading data...
```

### EmptyState
```
No results found
[Action button]
```

### ErrorState
```
❌ Error loading data
Try again
```

### SuccessState
```
✅ Data displayed
[User can interact]
```

### ActionInProgress
```
🔄 Processing request...
```

## Responsive Breakpoints

```
Mobile:   < 768px
Tablet:   768px - 1024px
Desktop:  > 1024px

All pages use:
- Grid: auto-fit, minmax()
- Flex: wrap for mobile
- Inline styles with media queries when needed
```

## Color Legend

```
🔵 Blue (#2196f3, #4db8ff)
   - Primary actions
   - Links
   - Info messages

🟢 Green (#4caf50)
   - Success
   - Approved
   - Completed
   - Active

🟠 Orange (#ff9800)
   - Warning
   - Pending
   - In Progress

🔴 Red (#f44336)
   - Danger
   - Cancelled
   - Error
   - Rejected

⚫ Gray (#999, #666, #f5f5f5)
   - Text
   - Disabled
   - Backgrounds
```

## Table Structures

### Users Table
- Email | Name | Role | Status | Phone | Joined | Actions

### Providers Table
- Business | Provider | Status | Rating | Rate | DBS | Insurance | Actions

### Bookings Table
- ID | Customer | Provider | Service | Date | Status | Amount | Actions

### Bookings (Customer View)
- Cards showing:
  - Status badge
  - Service name
  - Date & Time
  - Location
  - Pricing
  - Actions

## Form Patterns

### Login Form
- Email (text input)
- Password (password input)
- Submit button

### Register Form
- Full Name (text input)
- Email (email input)
- Phone (tel input)
- Password (password input)
- Role (select dropdown)
- Submit button

### Search Form
- Postcode (text input)
- Service Type (select dropdown)
- Min Rating (select dropdown)
- Max Price (number input)
- Filter button

### Message Form
- Message (text input)
- Send button

## Modal/Dialog Needs (Future)

```
- Confirm delete
- Approve/Reject with reason
- View full review
- View booking details
- Error messages
- Success confirmation
```

## Testing Paths

```
Unauthenticated:
  / → /providers/search → /providers/:userId
  → /auth/login → Home

Customer:
  /bookings/customer → details
  → /messages → /notifications

Provider:
  /providers/profile → /bookings/provider
  → /messages → /notifications

Admin:
  /admin/dashboard → /admin/users
  → /admin/providers → /admin/bookings
```

## Mobile Considerations

All pages use:
- Responsive grid (auto-fit)
- Flex layouts with wrap
- Touch-friendly buttons (min 44px height)
- Large tap targets
- Readable font sizes
- Full-width forms on mobile
- Stacked layouts where needed
- No horizontal scrolling
