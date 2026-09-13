# Telaboro QA Case Study

> End-to-end QA investigation of an Android marketplace and its web admin panel, covering API, PostgreSQL, Stripe, real-time flows, and mobile diagnostics.

<p align="center">
  <img src="https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" alt="React Native">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Stripe-635BFF?style=for-the-badge&logo=stripe&logoColor=white" alt="Stripe">
  <img src="https://img.shields.io/badge/Firebase-FFCA28?style=for-the-badge&logo=firebase&logoColor=black" alt="Firebase">
</p>

| Scope | Test evidence | Outcome |
|---|---|---|
| Android app, admin panel, API and integrations | 150+ test cases, defect documentation and Logcat evidence | 55 findings in the first run, 18 new findings during retest and 3 release-blocking defects |

**Start here:** [test documentation](docs/) · [technical logs](logs/) · [key findings](#-key-findings)

## 📱 About the project

**Telaboro** is a services marketplace connecting customers with technicians, similar to TaskRabbit/Profi.ru for Latin America. The platform combines an Android mobile application and a web admin panel for operational management.

**Technology stack:**
- **Mobile:** React Native + Expo on Android
- **Backend:** Node.js + Express + TypeScript
- **Database:** PostgreSQL + PostGIS
- **Payments:** Stripe Connect, Checkout, Payment Intents
- **Real-time:** Socket.IO
- **Push:** Firebase Cloud Messaging
- **Admin Panel:** React + TypeScript web application

---

## My role

**QA Engineer** — full testing of the mobile application and admin panel.

**Scope of work:**
- 150+ test cases covering critical business flows
- 55 defects in the first run
- 18 new defects during retest
- 3 critical release blockers
- Logcat analysis, API-response analysis, and Stripe webhook/integration investigation

---

## 🔍 What I tested

### Mobile App — Android
- Onboarding and registration for customer/technician roles
- KYC verification, five steps with photo and dynamic code
- Task creation and management for in-person and remote jobs
- Quote system and offer acceptance
- Stripe payments through Checkout, Connect, and 3DS
- Real-time customer↔technician chat through Socket.IO
- FCM push notifications
- Profile management and account deletion

### Admin Panel — Web
- Dashboard and summary metrics
- Analytics across Conversion, Quotes, Technicians, Clients, Quality, and Geography
- User and permission management with RBAC
- Operations: payments, disputes, escrow, payouts, tickets, and geofences
- Service and country catalogs
- Ambassador level and monetization system
- Push/email campaigns
- Audit log and Crash Reports
- System settings

---

## 📊 Results

### First run
| Severity | Count |
|----------|------:|
| 🔴 Critical | 7 |
| 🟠 High | 9 |
| 🟡 Medium | 22 |
| 🔵 Low | 17 |
| **Total** | **55** |

### Retest — v2.1.0, clean database
- **Fixed:** 8 of 55 defects, 15%
- **New defects found:** 18
- **Critical:** 2, Stripe payment stuck and profile crash

---

## 🚨 Key findings

### 1. Stripe Payment Launcher Bug — Critical

**Problem:** all card payments remain stuck in `Pending`.

**Root cause evidence:** React Native `ActivityResultRegistry` drops the result from native `PaymentLauncherConfirmationActivity`. Stripe returns `RESULT_OK`, but the application does not receive the payment result.

**Impact:** customers cannot pay for tasks → technicians do not receive money → the core business flow is blocked.

**Evidence:** Logcat contains `Dropping pending result: RESULT_OK` when returning from the native Stripe Activity.

**Visual description:** Admin Payments shows Total charged: $0.00 MXN and one pending payment waiting. The transaction table contains a `Pending` item of type `Diagnosis` for $150.00.

---

### 2. Profile Crash — Critical

**Problem:** opening a technician’s public profile from a quote crashes the screen.

**Root cause evidence:** `TechnicianProfileScreen` references the property `country`, which is absent from the API response.

**Impact:** customers cannot review a technician profile before accepting a quote, blocking a key decision-making flow.

**Evidence:** admin Crash Reports show four new crashes. Logcat contains `ReferenceError: Property 'country' doesn't exist at TechnicianProfileScreen`.

**Visual description:** black error screen with an explosion icon, “Something went wrong”, `Property 'country' doesn't exist`, and a “Retry” button. Retry reproduces the same loop.

---

### 3. Payment Logic Bypass — Medium

**Problem:** 11 `Per quote` orders totaling $27.50 were completed without card input.

**Root cause evidence:** quote charging operates outside the expected Stripe interaction. The Bronze level has zero free allowance, but charges are still marked paid automatically.

**Impact:** loss of payment-control integrity and potential financial loss.

**Visual description:** Admin Plan Orders shows Total revenue $27.50, 11 paid Per quote orders, all tied to one technician and each priced at $2.50.

---

## 🛠 Tools

- **Test Management:** Markdown documentation
- **Bug Tracking:** GitHub Issues
- **Log Analysis:** Logcat in Android Studio, Chrome DevTools
- **API Testing:** Browser DevTools and API-response analysis
- **Devices:** Nothing Phone 1 on Android 15, Pixel 9a Emulator on Android 17

---

## 📈 Quality metrics

| Metric | Value |
|--------|-------|
| Test Coverage | 100% of critical flows |
| Bug Detection Rate | 73 defects across two runs |
| Critical Bugs Found | 9, seven initial + two retest |
| False Positive Rate | <5% |
| Retest Pass Rate | 15%, 8/55 fixed |

---

## 💡 Lessons learned

1. **Stripe integration** — verify both webhook behavior and ActivityResult handling when React Native interacts with native modules.
2. **React Native + native modules** — cross-layer data transfer is a frequent defect boundary.
3. **Clean database for retest** — necessary to validate metrics without noise from seeded data.
4. **i18n** — a system-wide issue requires one consistent translation dictionary across UI, data, legends, and filters.
5. **RBAC** — the presence of roles does not prove permissions are actually enforced; assignment and access must be tested explicitly.

---

## 📂 Documentation

- [Test strategy](docs/test-strategy.md)
- [Retest bug reports — 18 defects](docs/bugs.md)
- [Retest test cases — 18 cases](docs/test-cases.md)
- [Retest summary](docs/retest-summary.md)
- [Test environment](docs/environment.md)
- [Logcat evidence](logs/)

---

## 📞 Contacts

**LinkedIn:** [Vsevolod Samoylov](https://www.linkedin.com/in/vsevolod-samoylov)  
**Telegram:** @vsevolod

---

*Project completed in August 2026.*  
*Screenshots and videos are not included because they contain confidential application data. Defects contain detailed textual UI descriptions and sanitized Logcat evidence instead.*