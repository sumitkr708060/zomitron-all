# Zomitron 🚀
### Hyperlocal Quick-Commerce — MERN Stack Multivendor Platform

A production-ready multivendor e-commerce platform where customers see **only products from vendors within their chosen radius** (default 100km), with **1-hour express delivery** for same-city vendors.

---

## ✨ Features

| Category | Details |
|----------|---------|
| 🗺️ **Geo-Filtering** | MongoDB `$geoNear` aggregation — products sorted by distance from customer |
| 📍 **Location Detection** | Browser GPS → IP geolocation → manual pincode entry |
| ⚡ **Delivery ETAs** | 1hr (0-5km), 2hr (5-60km), 1day (60-100km), 2-3days (100-500km) |
| 🏪 **Multivendor** | Vendor registration, approval workflow, per-vendor order fulfillment |
| 💳 **Payments** | Razorpay (UPI/Net Banking/Cards) + Stripe + COD |
| 💰 **Split Payments** | Auto commission split (10-20%), vendor payout on delivery |
| 📱 **Real-time** | Socket.IO order updates, WhatsApp notifications via Twilio |
| 🔐 **Auth** | JWT + Refresh tokens, Google OAuth, OTP email verification |
| 👑 **Admin Panel** | Dashboard, vendor approval, order management, payouts |
| 📦 **SEO & PWA** | Meta tags, PWA manifest, lazy loading |

---

## 🛠️ Tech Stack

**Backend:** Node.js 20, Express.js, MongoDB (Mongoose), Socket.IO, Cloudinary, Razorpay, Stripe, Twilio  
**Frontend:** React 18, Redux Toolkit + RTK Query, React Router 6, Tailwind CSS, Framer Motion, Swiper, Vite

---

## 📁 Project Structure

```
zomitron/
├── backend/
│   ├── config/          # DB, Cloudinary, Socket.io
│   ├── middleware/       # JWT auth, role guards
│   ├── models/          # Mongoose schemas (User, Vendor, Product, Order, ...)
│   ├── routes/          # REST API routes (10 route files)
│   ├── seed/            # Database seed script
│   ├── tests/           # Jest tests (auth, geo-filtering)
│   ├── utils/           # haversine, deliveryETA, geocode, ipGeolocation, commission
│   └── server.js        # Express app entry
│
├── frontend/
│   ├── public/          # favicon, manifest.json (PWA)
│   └── src/
│       ├── components/  # Navbar, Footer, ProductCard, LocationPicker, ...
│       ├── hooks/       # useLocation, useSocket
│       ├── pages/       # Home, Shop, ProductDetail, Cart, Checkout, ...
│       │   ├── admin/   # AdminOverview, Vendors, Orders, Products, Categories, Payouts
│       │   └── vendor/  # VendorOverview, Products, Orders, Profile, Earnings
│       ├── store/       # Redux slices + RTK Query API
│       └── utils/       # deliveryUtils, geoUtils
│
├── docker-compose.yml   # MongoDB + Redis + Backend + Frontend
├── .env.example         # All required environment variables
└── package.json         # Root scripts (dev, install, seed)
```

---

## 🚀 Quick Start

### 1. Install dependencies

```bash
npm run install:all
```

### 2. Set up environment variables

```bash
cp .env.example backend/.env
# Edit backend/.env with your API keys
```

Minimum required for development (demo mode):
```
MONGO_URI=mongodb://localhost:27017/zomitron
JWT_SECRET=your_super_secret_key_here
NODE_ENV=development
```

### 3. Start MongoDB (choose one)

```bash
# Option A: Docker
docker compose up -d mongo redis

# Option B: Local MongoDB
mongod --dbpath /usr/local/var/mongodb
```

### 4. Seed the database

```bash
npm run seed
```

This creates:
- 👑 **Admin:** `admin@zomitron.com` / `admin123`
- 👤 **Customer:** `customer@zomitron.com` / `customer123`
- 🏪 **Vendor 1:** `vendor1@zomitron.com` / `vendor123` (Prayagraj)
- 🏪 **Vendor 2:** `vendor2@zomitron.com` / `vendor123` (Prayagraj)
- 🏪 **Vendor 3:** `vendor3@zomitron.com` / `vendor123` (Bangalore)
- 10 sample products across Electronics, Fashion, Mobiles, Laptops
- 20 India pincodes (Prayagraj, Lucknow, Varanasi, Kanpur, Bangalore, Mumbai, Delhi)

### 5. Start the app

```bash
npm run dev
```

This concurrently starts:
- **Backend API:** http://localhost:5000
- **Frontend:** http://localhost:5173

---

## 🗺️ Geo-Filtering System

The core feature. When a customer at location (lat, lng) visits the shop:

```
GET /api/products?lat=25.4358&lng=81.8463&radius=100
```

Backend uses MongoDB `$geoNear` aggregation:
```js
{ $geoNear: { 
    near: { type: 'Point', coordinates: [lng, lat] },
    distanceField: 'distance',
    maxDistance: radius * 1000,   // Convert km → meters
    spherical: true
}}
```

Results are auto-sorted nearest-first. Products include `distance` and `deliveryInfo` fields.

### Delivery ETAs
| Distance | ETA | Delivery Charge |
|----------|-----|----------------|
| 0-5 km | ⚡ 1 Hour | Free |
| 5-60 km | 🚀 2 Hours | ₹29-49 |
| 60-100 km | 📦 1 Day | ₹50-79 |
| 100-500 km | 🚚 2-3 Days | ₹80-149 |
| >500 km | 📬 5-7 Days | ₹150+ |

---

## 🔌 API Overview

| Route | Description |
|-------|-------------|
| `POST /api/auth/register` | Register user (customer/vendor) |
| `POST /api/auth/login` | Login, returns JWT |
| `GET /api/products?lat=&lng=&radius=` | **Geo-filtered products** |
| `GET /api/products/:id` | Product detail |
| `POST /api/vendors/register` | Vendor registration |
| `POST /api/orders` | Create order (multi-vendor) |
| `POST /api/payments/razorpay/order` | Create Razorpay order |
| `POST /api/payments/razorpay/verify` | Verify payment signature |
| `GET /api/pincode/:code` | Validate pincode + get delivery info |
| `GET /api/admin/dashboard` | Admin analytics |

---

## 🐳 Docker (Full Stack)

```bash
docker compose up --build
```

Services started:
- `mongo` on port 27017
- `redis` on port 6379
- `backend` on port 5000
- `frontend` on port 5173

---

## 🧪 Tests

```bash
cd backend && npm test
```

Tests cover:
- ✅ Auth: register, login, duplicate email, role guards, JWT protect
- ✅ Haversine: distance accuracy (Prayagraj→Kanpur~130km, →Bangalore~1600km)
- ✅ Geo-filtering: products within/outside 100km radius
- ✅ Delivery ETA: all 5 distance tiers

---

## 🔑 Key Environment Variables

```bash
# Required
MONGO_URI=mongodb://localhost:27017/zomitron
JWT_SECRET=your_jwt_secret

# Payment
RAZORPAY_KEY_ID=rzp_test_xxx
RAZORPAY_KEY_SECRET=xxx
STRIPE_SECRET_KEY=sk_test_xxx

# Notifications (optional)
TWILIO_WHATSAPP_FROM=whatsapp:+14155238886
TWILIO_ACCOUNT_SID=xxx
TWILIO_AUTH_TOKEN=xxx

# Cloudinary (for image uploads)
CLOUDINARY_CLOUD_NAME=xxx
CLOUDINARY_API_KEY=xxx
CLOUDINARY_API_SECRET=xxx

# Maps (for geocoding)
GOOGLE_MAPS_API_KEY=xxx
```

---

## 📱 Pages

| Route | Page |
|-------|------|
| `/` | Home — hero slider, featured nearby products, category carousel |
| `/shop` | Shop — geo-filtered grid with filters (category, price, radius) |
| `/shop/:category` | Category-filtered shop |
| `/product/:id` | Product detail — image gallery, pincode checker, reviews |
| `/cart` | Cart — qty controls, coupon validator |
| `/checkout` | Checkout — address form, Razorpay payment |
| `/orders` | My orders list |
| `/orders/:id` | Order tracking with real-time Socket.IO updates |
| `/vendor/dashboard` | Vendor overview stats |
| `/vendor/products` | Manage products |
| `/vendor/orders` | Fulfill orders |
| `/admin/dashboard` | Admin analytics |
| `/admin/vendors` | Approve/suspend vendors |

---

## 💡 Architecture Decisions

- **`$geoNear` vs Haversine filtering:** MongoDB's `$geoNear` runs in the database (fast, indexed), while `filterByRadius` is provided as a utility for in-memory filtering or client-side use.
- **Location detection priority:** Browser GPS (most accurate) → IP geolocation (fallback) → manual pincode entry → default (Prayagraj).
- **Multi-vendor orders:** A single order object contains items from multiple vendors; each vendor sees only their items and fulfills independently.
- **Commission split:** Calculated at order creation time, credited to vendor balance on delivery.
