# SnowPlatform - Quick Reference

## 🚀 Quick Start

### Access the Application
- **Development URL**: https://3000-ipsnk0aei4mowm3jyg254-5634da27.sandbox.novita.ai
- **API Base**: https://3000-ipsnk0aei4mowm3jyg254-5634da27.sandbox.novita.ai/api
- **Local**: http://localhost:3000

### Test User Accounts
```
Admin:
  Email: admin@snowplatform.uk
  Password: admin_password

Service Providers:
  - sarah.cleaner@email.uk / password (Cleaning)
  - james.plumber@email.uk / password (Plumbing)
  - emma.tutor@email.uk / password (Tutoring)
  - michael.dev@email.uk / password (Web Development)

Customers:
  - john.smith@email.uk / password
  - mary.jones@email.uk / password
  - david.brown@email.uk / password
  - lisa.taylor@email.uk / password
```

## 📦 Project Structure
```
webapp/
├── src/
│   ├── index.tsx          # Main application (Hono backend + HTML frontend)
│   └── renderer.tsx       # Hono renderer
├── migrations/
│   └── 0001_initial_schema.sql   # Database schema
├── seed.sql               # Sample data
├── ecosystem.config.cjs   # PM2 configuration
├── wrangler.jsonc         # Cloudflare configuration
├── package.json           # Dependencies and scripts
├── README.md              # Comprehensive documentation
└── API_TESTING.md         # API testing guide
```

## 🎯 Key Commands

### Development
```bash
npm run build              # Build application
npm run dev:d1             # Start with D1 database
pm2 start ecosystem.config.cjs  # Start with PM2 (recommended)
pm2 logs snowplatform --nostream  # Check logs
pm2 restart snowplatform   # Restart after changes
```

### Database
```bash
npm run db:migrate:local   # Apply migrations locally
npm run db:seed            # Load sample data
npm run db:reset           # Reset database
npm run db:console:local   # Database console
```

### Testing
```bash
curl http://localhost:3000  # Test frontend
curl http://localhost:3000/api/providers/search  # Test API
npm run test               # Quick server test
```

## 🔑 Key Features

### For Customers
1. **Search** services by postcode, type, rating, price
2. **Book** services with date/time selection
3. **Message** providers directly
4. **Review** completed services
5. **Track** booking status
6. **Cancel** bookings (refund processed)

### For Service Providers
1. **Create** business profile
2. **List** multiple services
3. **Manage** bookings
4. **Set** availability schedule
5. **Respond** to reviews
6. **Message** customers

### For Administrators
1. **Approve** provider profiles
2. **Monitor** all bookings
3. **View** platform statistics
4. **Manage** users
5. **Track** revenue

## 📊 Platform Statistics (Sample Data)
- **Users**: 9 (1 admin, 4 providers, 4 customers)
- **Services**: 13 active listings
- **Bookings**: 7 (3 completed, 2 confirmed, 1 pending, 1 cancelled)
- **Reviews**: 3 verified reviews (4.85★ average)
- **Revenue**: £34.50 from completed bookings

## 🎨 Service Categories
- 🧹 Cleaning (regular, deep, carpet, window)
- 🔧 Plumbing (emergency, installation, boiler)
- ⚡ Electrical
- 🔨 Carpentry
- 💻 IT Support
- 🌐 Web Development
- 🎨 Graphic Design
- 🎬 Video Editing
- 📚 Tutoring
- 💪 Fitness Coaching

## 💰 Platform Economics
- **Commission**: 15% per booking
- **Payment Flow**: Customer → Platform → Provider
- **Refund Policy**: Full refund on cancellation
- **Payout Schedule**: After service completion

## 🔐 Authentication
- **Method**: JWT tokens
- **Roles**: customer, provider, admin
- **Token Format**: Bearer token in Authorization header
- **Expiry**: No expiry in current version (add in production)

## 📱 API Endpoints Summary

### Authentication (3 endpoints)
- POST `/api/auth/register` - Register user
- POST `/api/auth/login` - Login user
- GET `/api/auth/me` - Get current user

### Providers (3 endpoints)
- GET `/api/providers/search` - Search providers
- GET `/api/providers/:userId` - Get provider profile
- PUT `/api/providers/profile` - Update profile

### Services (5 endpoints)
- GET `/api/services/provider/:providerId` - List services
- GET `/api/services/:serviceId` - Get service
- POST `/api/services` - Create service
- PUT `/api/services/:serviceId` - Update service
- DELETE `/api/services/:serviceId` - Delete service

### Bookings (6 endpoints)
- POST `/api/bookings` - Create booking
- GET `/api/bookings/customer` - Customer bookings
- GET `/api/bookings/provider` - Provider bookings
- GET `/api/bookings/:bookingId` - Get booking
- PATCH `/api/bookings/:bookingId/status` - Update status
- POST `/api/bookings/:bookingId/cancel` - Cancel booking

### Reviews (3 endpoints)
- POST `/api/reviews` - Create review
- GET `/api/reviews/provider/:providerId` - Get reviews
- POST `/api/reviews/:reviewId/respond` - Respond to review

### Messages (3 endpoints)
- POST `/api/messages` - Send message
- GET `/api/messages/conversations` - List conversations
- GET `/api/messages/conversation/:id` - Get messages

### Notifications (3 endpoints)
- GET `/api/notifications` - Get notifications
- PATCH `/api/notifications/:id/read` - Mark as read
- POST `/api/notifications/read-all` - Mark all as read

### Admin (5 endpoints)
- GET `/api/admin/users` - All users
- GET `/api/admin/providers` - All providers
- PATCH `/api/admin/providers/:id/approval` - Approve/reject
- GET `/api/admin/bookings` - All bookings
- GET `/api/admin/stats` - Platform statistics

**Total: 40+ API endpoints**

## 🗄️ Database Tables
1. **users** - User accounts
2. **provider_profiles** - Provider information
3. **services** - Service listings
4. **bookings** - Booking records
5. **payments** - Payment transactions
6. **reviews** - Customer reviews
7. **messages** - In-app messages
8. **notifications** - User notifications
9. **availability** - Provider schedules
10. **blocked_dates** - Provider unavailability

## ⚠️ Production Checklist

### CRITICAL (Must implement before production)
- [ ] Replace password hashing with bcrypt
- [ ] Implement proper JWT signing with HMAC
- [ ] Add email verification workflow
- [ ] Integrate Stripe/GoCardless for payments
- [ ] Add rate limiting for API endpoints
- [ ] Implement CSRF protection
- [ ] Set up monitoring and error tracking
- [ ] Configure proper environment variables
- [ ] Add SSL certificate (handled by Cloudflare)
- [ ] Set up backup strategy for database

### HIGH Priority
- [ ] Add file upload for profile images (R2)
- [ ] Implement DBS certificate verification
- [ ] Add insurance document verification
- [ ] Set up email notification service
- [ ] Add SMS verification for phone numbers
- [ ] Implement automated payout system
- [ ] Add booking calendar view
- [ ] Create customer and provider dashboards
- [ ] Add mobile responsive improvements

### MEDIUM Priority
- [ ] OAuth integration (Google, Facebook)
- [ ] Advanced search with map view
- [ ] Subscription plans for regular bookings
- [ ] Promotional codes and discounts
- [ ] Referral system
- [ ] Analytics dashboard
- [ ] Multi-language support

## 🔧 Troubleshooting

### Server won't start
```bash
pm2 delete all
fuser -k 3000/tcp
npm run build
pm2 start ecosystem.config.cjs
```

### Database issues
```bash
npm run db:reset  # Resets local database
```

### API returns 404
```bash
# Check if server is running
pm2 list
pm2 logs snowplatform --nostream
```

### Build errors
```bash
rm -rf node_modules dist .wrangler
npm install
npm run build
```

## 📚 Documentation
- **README.md** - Full project documentation
- **API_TESTING.md** - API testing guide with examples
- **QUICK_REFERENCE.md** - This file

## 🎓 Learning Resources
- Hono Framework: https://hono.dev
- Cloudflare Workers: https://developers.cloudflare.com/workers
- Cloudflare D1: https://developers.cloudflare.com/d1
- Wrangler CLI: https://developers.cloudflare.com/workers/wrangler

## 🤝 Support
For questions or issues:
1. Check the README.md for detailed documentation
2. Review API_TESTING.md for API usage examples
3. Check PM2 logs for error messages
4. Verify database with console commands

---

**SnowPlatform** - Connecting service providers with customers across the UK 🇬🇧
