# Test Environment

## 📱 Mobile application

| Parameter | Value |
|----------|----------|
| Physical device | Nothing Phone 1, Android 15 |
| Emulator | Google Pixel 9a, Android 17 |
| Application version | Telaboro v2.1.0 |
| Stack | React Native + Expo |
| Payments | Stripe Test Mode |
| Real-time | Socket.IO |
| Push | Firebase Cloud Messaging |
| Media storage | AWS S3 with presigned URLs |

## 🖥 Admin panel

| Parameter | Value |
|----------|----------|
| Browser | Google Chrome, automatic translation disabled |
| Interface | EN |
| Stack | React + TypeScript |
| Backend | Node.js + Express + TypeScript |
| Database | PostgreSQL + PostGIS |

## 🧪 Test data

The retest used a **clean database** without seed data. Test accounts were created manually:

| Role | Description |
|------|-------------|
| Customer | Created through application registration using a disposable email |
| Technician | Created through registration and completed KYC as verified |
| Technician 2 | Created through registration with KYC pending |
| Admin | Existing account with Super Admin role |

**Control transactions used for metric verification:**
- 3 tasks: 1 Assigned, 2 Cancelled after customer deletion
- 3 quotes: 1 Accepted
- 3 plan purchases: 2 Balance, 1 Subscription — all stuck in Pending
- 11 Per quote charges at $2.50, created automatically without card input
- 1 Diagnosis payment of $150.00, Pending

## 🛠 Tools

| Tool | Purpose |
|------|---------|
| Logcat in Android Studio | Native-layer crash and error analysis |
| Chrome DevTools | API response and network request analysis |
| GitHub Issues | Bug tracking |
| Markdown | Bug reports and test-case documentation |

## 🔑 Analysis methods

1. **Logcat analysis** — crashes such as `ReferenceError` and `CalledFromWrongThreadException`, plus SDK warnings from Stripe, Firebase, and RNScreens.
2. **API analysis** — response fields such as `is_first_purchase`, `country`, and status values.
3. **Metric reconciliation** — compare manually calculated control values with admin-panel metrics.
4. **Cross-role validation** — execute one scenario from customer, technician, and admin perspectives to expose synchronization issues.