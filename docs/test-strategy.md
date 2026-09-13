# Telaboro Test Strategy

## Test objectives

1. Validate mobile application functionality for customers and technicians.
2. Test the admin panel and all of its sections.
3. Verify real-time interaction between roles.
4. Identify critical blockers and security-related defects.
5. Assess product readiness for release.

## Test scope

### In scope
- **Android mobile app — technician:** onboarding, five-step KYC, inbox, quotes, chat, profile, account deletion.
- **Android mobile app — customer:** onboarding, in-person/remote task creation, requests, quotes, Stripe payments, chat.
- **Admin panel:** Dashboard; six Analytics sections; Users; eight Operations sections; four reference/catalog sections; eight System sections.
- **Integrations:** Stripe Checkout/Connect, Socket.IO real-time chat, Firebase Cloud Messaging.
- **Geofencing:** PostGIS location validation.
- **KYC:** technician verification with photo and dynamic code.

### Out of scope
- iOS application.
- Web application for customers/technicians.
- Load testing.
- Security penetration testing.
- Standalone API testing; API behavior was observed through UI-driven flows and DevTools.

## Testing types

- **Functional testing** — business logic and user scenarios.
- **UI testing** — visual defects and UX.
- **Localization testing — i18n** — translations and mixed-language states.
- **Integration testing** — Stripe, Socket.IO, FCM, PostGIS.
- **Smoke testing** — critical flows before release.
- **Access-control testing — RBAC** — admin roles and permissions.

## Methodology

Testing was performed **manually** using physical devices and emulators.

**Process:**
1. Create test accounts for customer, technician, and admin roles.
2. Execute the full business flow: registration → KYC → task creation → quotes → payment → completion → rating.
3. Document defects with detailed reproduction steps.
4. Prioritize by severity, Critical/High/Medium/Low, and priority, P1–P4.
5. Retest fixes on a clean database.

**Specific practices:**
- Clean database used for retest, without seed data.
- Controlled transactions created to validate metrics.
- Logcat used for critical-defect analysis.
- API responses checked through DevTools.

## Environment

| Parameter | Value |
|----------|----------|
| Physical device | Nothing Phone 1, Android 15 |
| Emulator | Google Pixel 9a, Android 17 |
| App version | Telaboro v2.1.0 |
| Admin panel | Google Chrome, EN interface, automatic translation disabled |
| Backend | Node.js + Express + TypeScript |
| Database | PostgreSQL + PostGIS |
| Payments | Stripe Test Mode |
| Real-time | Socket.IO |
| Push | Firebase Cloud Messaging |

## Acceptance criteria

### Release criteria
- ✅ All Critical defects fixed.
- ✅ All High defects fixed.
- ✅ Critical business flows work: registration, KYC, task creation, payment, chat.
- ✅ No blockers remain in the admin panel.

### Current status
- ❌ **NOT READY FOR RELEASE** — two Critical defects block core flows: payments and technician profile.