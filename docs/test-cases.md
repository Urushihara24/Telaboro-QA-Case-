# Telaboro Test Cases — 72 Cases

This document contains two parts:
1. **Flow testing** — 54 positive and negative cases grouped by module.
2. **Retest negative cases** — 18 detailed scenarios for defects found during retest.

**Status legend:**
- ✅ Passed — works as expected
- ❌ Failed — defect found, linked to a bug report
- ⛔ Blocked — cannot be completed because of a blocker, primarily payments

---

## Part 1. Flow Testing

### 📱 Mobile — Customer — 15 cases

| ID | Title | Type | Priority | Expected result — short | Status | Bug |
|----|-------|------|----------|-------------------------|--------|-----|
| TC-C-001 | Customer registration by email | Positive | P1 | Account created and home screen opened | ✅ Passed | — |
| TC-C-002 | Customer login | Positive | P1 | Successful authentication | ✅ Passed | — |
| TC-C-003 | Lockout after failed login attempts | Negative | P2 | Clear lockout message | ✅ Passed | NB-016, two statuses in admin |
| TC-C-004 | Create In-person task inside active geofence | Positive | P1 | Task created with Published status | ✅ Passed | — |
| TC-C-005 | Create task outside active geofence | Negative | P1 | Clear localized error | ❌ Failed | M-C-005, raw `GEOFENCE_VIOLATION` code |
| TC-C-006 | Address validation with two concatenated locations | Negative | P3 | Error guides user to select one address | ✅ Passed | — |
| TC-C-007 | Create Remote task | Positive | P2 | Task created without address | ✅ Passed | — |
| TC-C-008 | View quotes | Positive | P1 | Quotes visible in AWAITING RESPONSE | ✅ Passed | — |
| TC-C-009 | Accept quote | Positive | P1 | Quote moves to PENDING PAYMENT | ✅ Passed | — |
| TC-C-010 | Stripe payment with test card | Positive | P1 | Payment completes and task becomes Paid | ❌ Failed | NB-003 |
| TC-C-011 | Retry payment after stuck attempt | Negative | P2 | Button disabled or existing PaymentIntent reused | ❌ Failed | NB-004 |
| TC-C-012 | Open technician profile from quote | Positive | P1 | Profile opens with technician data | ❌ Failed | NB-014 |
| TC-C-013 | Real-time chat with technician | Positive | P2 | Messages delivered immediately through Socket.IO | ✅ Passed | — |
| TC-C-014 | Delete customer account | Positive | P2 | Account deleted and user returned to login | ✅ Passed | — |
| TC-C-015 | Rate technician after task completion | Positive | P2 | Rating saved and technician score updated | ⛔ Blocked | Depends on payment, NB-003 |

### 📱 Mobile — Technician — 16 cases

| ID | Title | Type | Priority | Expected result — short | Status | Bug |
|----|-------|------|----------|-------------------------|--------|-----|
| TC-T-001 | Technician registration | Positive | P1 | Account created and questionnaire opened | ✅ Passed | — |
| TC-T-002 | Technician questionnaire: categories, rate, area | Positive | P1 | Questionnaire saved | ✅ Passed | — |
| TC-T-003 | KYC steps 1–5: ID photo, selfie, code | Positive | P1 | KYC submitted with Pending status | ✅ Passed | — |
| TC-T-004 | KYC step 5 code modal remains stable | Negative | P1 | Modal remains open while media is selected | ✅ Passed | Fixed M-C-001 |
| TC-T-005 | View Inbox and tasks | Positive | P1 | Tasks display correctly | ✅ Passed | — |
| TC-T-006 | Send quote | Positive | P1 | Quote submitted with SENT status | ✅ Passed | — |
| TC-T-007 | ACCEPTED counter after quote acceptance | Positive | P2 | Counter increases by one | ⛔ Blocked | Requires successful payment |
| TC-T-008 | Confirm arrival + photo | Positive | P1 | Task becomes In Progress | ⛔ Blocked | Requires successful payment |
| TC-T-009 | Complete task | Positive | P1 | Task completes and rating request appears | ⛔ Blocked | Requires successful payment |
| TC-T-010 | Buy Balance plan, 100 quotes | Positive | P1 | Payment completes and balance increases | ❌ Failed | NB-003 |
| TC-T-011 | Buy Subscription plan, 30 quotes | Positive | P2 | Payment completes and plan becomes active | ❌ Failed | NB-003 |
| TC-T-012 | Per quote charge after quote acceptance | Positive | P2 | Charge follows level rules, Bronce has 0 free | ❌ Failed | NB-006 |
| TC-T-013 | Phone field validation | Negative | P3 | Mask/error for invalid input | ❌ Failed | NB-001 |
| TC-T-014 | Delete technician account — full flow | Positive | P2 | Account deleted and user returned to login | ❌ Failed | M-C-007 |
| TC-T-015 | Cancelled tasks in Inbox | Negative | P3 | Cancelled tasks hidden or visually separated | ❌ Failed | NB-007 |
| TC-T-016 | Task status after customer deletion | Negative | P2 | Consistent Cancelled state everywhere | ❌ Failed | NB-005 |

### 🖥 Admin Panel — 18 cases

| ID | Title | Type | Priority | Expected result — short | Status | Bug |
|----|-------|------|----------|-------------------------|--------|-----|
| TC-A-001 | Admin login | Positive | P1 | Successful authentication | ✅ Passed | — |
| TC-A-002 | Dashboard metrics | Positive | P1 | Metrics match database data | ✅ Passed | — |
| TC-A-003 | Revenue by plan / ambassador levels | Positive | P2 | Breakdown by level is correct | ✅ Passed | Fixed A-M-015 |
| TC-A-004 | Analytics → Conversion funnel | Positive | P2 | Conversion ≤100% at every stage | ✅ Passed | Fixed M-M-004 |
| TC-A-005 | Analytics → Quotes | Positive | P2 | Acceptance rate is correct | ✅ Passed | Fixed M-M-005 |
| TC-A-006 | Analytics → Quality | Positive | P2 | `—` instead of 0.00 when data is absent | ✅ Passed | Fixed M-M-007 |
| TC-A-007 | Analytics → Ticket Reports i18n | Negative | P4 | Chart legend matches EN interface | ❌ Failed | NB-015 |
| TC-A-008 | Users list and statuses | Positive | P2 | One consistent status per user | ❌ Failed | NB-016 |
| TC-A-009 | Technician KYC statuses and levels | Positive | P1 | Verified/Pending + level displayed | ✅ Passed | Fixed M-H-008 |
| TC-A-010 | Plan Orders | Positive | P2 | Orders correspond to actual payment behavior | ❌ Failed | NB-006 |
| TC-A-011 | Payments processing | Positive | P1 | Successful payments become Paid and Total charged grows | ❌ Failed | NB-003 |
| TC-A-012 | Task amounts and statuses | Positive | P2 | Task and payment amounts match | ❌ Failed | NB-017 |
| TC-A-013 | Escrow i18n | Negative | P4 | Labels match EN interface | ❌ Failed | NB-018 |
| TC-A-014 | Admin role assignment | Positive | P2 | Roles assigned through Manage Roles | ✅ Passed | — |
| TC-A-015 | Admin role revocation | Negative | P2 | Revoke control available for each role | ❌ Failed | NB-019 |
| TC-A-016 | Crash Reports | Positive | P2 | Crashes recorded with details | ✅ Passed | — |
| TC-A-017 | Audit log | Positive | P3 | Admin actions and errors are logged | ✅ Passed | — |
| TC-A-018 | Geofences | Positive | P2 | Active-zone list is correct | ✅ Passed | — |

### 🔬 Logs and integrations — 5 cases

| ID | Title | Type | Priority | Expected result — short | Status | Bug |
|----|-------|------|----------|-------------------------|--------|-----|
| TC-L-001 | Logcat: CalledFromWrongThreadException | Negative | P2 | No thread errors during navigation | ❌ Failed | NB-009 |
| TC-L-002 | Logcat: RNScreens iOS props | Negative | P4 | No warnings about unsupported props | ❌ Failed | NB-011 |
| TC-L-003 | Logcat: Firebase Analytics | Negative | P4 | Events log without errors | ❌ Failed | NB-012 |
| TC-L-004 | Logcat: OnBackInvokedCallback | Negative | P4 | Predictive back gesture works correctly | ❌ Failed | NB-013 |
| TC-L-005 | Logcat: Stripe card-brand tint | Negative | P3 | Brand icon renders correctly | ❌ Failed | NB-010 |

**Part 1 total:** 54 cases — ✅ Passed: 26, ❌ Failed: 21, ⛔ Blocked: 7.

---

## Part 2. Detailed Negative Retest Cases

### Critical — 2

| ID | Title | Priority | Preconditions | Steps | Expected result | Environment | Status | Bug |
|----|-------|----------|---------------|-------|-----------------|-------------|--------|-----|
| TC-NB-003 | Stripe Checkout payment | Critical | 1. Customer exists<br>2. Task exists and quote accepted<br>3. Task status PENDING PAYMENT<br>4. Stripe test card available | 1. Login as customer<br>2. Open Quotes<br>3. Select Complete payment<br>4. Enter Stripe test card<br>5. Tap Pay<br>6. Complete 3DS/biometric confirmation<br>7. Return to app<br>8. Check task status<br>9. Open Admin → Payments and check Total charged | Payment completes; task becomes Paid/Assigned; Total charged increases by payment amount | Nothing Phone 1, Android 15<br>Telaboro v2.1.0 | Failed | NB-003 |
| TC-NB-014 | Open technician public profile | Critical | 1. Customer exists<br>2. Technician sent quote<br>3. Customer is on Quotes screen | 1. Login as customer<br>2. Open Quotes<br>3. Tap technician name<br>4. Check profile load<br>5. Check Logcat | Public profile opens with name, rating, categories, and location; no runtime error | Physical device + emulator<br>Telaboro v2.1.0 | Failed | NB-014 |

### High — 5

| ID | Title | Priority | Preconditions | Steps | Expected result | Environment | Status | Bug |
|----|-------|----------|---------------|-------|-----------------|-------------|--------|-----|
| TC-NB-002 | Verify `is_first_purchase` | High | 1. Technician verified<br>2. Previous plan purchases exist<br>3. Previous payments Pending | 1. Login as technician<br>2. Buy Balance plan<br>3. Enter card<br>4. Wait for Pending<br>5. Buy another plan<br>6. Inspect `is_first_purchase` in API response | `is_first_purchase: false` for repeat purchases | Pixel 9a emulator, Android 17<br>Telaboro v2.1.0 | Failed | NB-002 |
| TC-NB-004 | Retry a stuck payment | High | 1. Quote accepted<br>2. Status PENDING PAYMENT<br>3. First attempt stuck | 1. Complete payment → Pay<br>2. Wait for Pending<br>3. Run Complete payment again with another card | Button becomes disabled after first attempt or existing PaymentIntent is reused | Nothing Phone 1, Android 15<br>Telaboro v2.1.0 | Failed | NB-004 |
| TC-NB-005 | Statuses after customer deletion | High | 1. Customer and technician exist<br>2. Quote Accepted<br>3. Customer deleted account | 1. Login as technician<br>2. Check task status in Inbox<br>3. Check status in Quotes<br>4. Open quote details<br>5. Check Accepted counter | Cancelled status is consistent everywhere and Accepted counter decreases | Device + emulator<br>Telaboro v2.1.0 | Failed | NB-005 |
| TC-NB-009 | Navigation between screens | High | App is running and user is logged in | 1. Open app<br>2. Quotes → technician name<br>3. Back → Profile<br>4. Inspect Logcat | No `CalledFromWrongThreadException` | Device + emulator<br>Telaboro v2.1.0 | Failed | NB-009 |
| TC-NB-019 | Revoke admin role | High | Super Admin exists and another admin has multiple roles | 1. System → Admins<br>2. Open shield action<br>3. Inspect Manage Roles for removal control | Revoke action available for each role | Chrome, EN interface | Failed | NB-019 |

### Medium — 6

| ID | Title | Priority | Preconditions | Steps | Expected result | Environment | Status | Bug |
|----|-------|----------|---------------|-------|-----------------|-------------|--------|-----|
| TC-NB-001 | Phone field validation | Medium | Registration/profile screen open | Enter `123qwere123132412` and save profile | Validation error or input mask | Nothing Phone 1, Android 15<br>Telaboro v2.1.0 | Failed | NB-001 |
| TC-NB-006 | Per quote payment at Bronce level | Medium | Technician is Bronce with 0 free quotes | 1. Customer accepts quote<br>2. Admin → Plan Orders<br>3. Count orders and amount<br>4. Check whether card input occurred | Charge follows Stripe/free-allowance rules | Chrome, EN interface | Failed | NB-006 |
| TC-NB-007 | Cancelled tasks in Inbox | Medium | Task became Cancelled | Find Cancelled task, attempt to respond, inspect visual separation | Cancelled items hidden or visually separated | Pixel 9a emulator, Android 17<br>Telaboro v2.1.0 | Failed | NB-007 |
| TC-NB-010 | Card-brand icon | Medium | Checkout open | Enter `4242`, inspect Visa icon, inspect Logcat | Correct brand icon without errors | Nothing Phone 1, Android 15<br>Telaboro v2.1.0 | Failed | NB-010 |
| TC-NB-016 | Blocked-user status | Medium | Customer is blocked | Users → find user → inspect Status column | One status only: Active OR Blocked | Chrome, EN interface | Failed | NB-016 |
| TC-NB-017 | Compare task and payment amounts | Medium | Task $123 + Diagnosis payment | Compare Tasks amount and Payments amount | Amounts match | Chrome, EN interface | Failed | NB-017 |

### Low — 5

| ID | Title | Priority | Preconditions | Steps | Expected result | Environment | Status | Bug |
|----|-------|----------|---------------|-------|-----------------|-------------|--------|-----|
| TC-NB-011 | Logcat: RNScreens iOS props | Low | Android app | Navigate between screens and inspect Logcat | No unsupported-prop warnings | Device + emulator<br>Telaboro v2.1.0 | Failed | NB-011 |
| TC-NB-012 | Firebase Analytics | Low | FCM configured | Wait for push and inspect Logcat | No `analytics library is missing` error | Device + emulator<br>Telaboro v2.1.0 | Failed | NB-012 |
| TC-NB-013 | OnBackInvokedCallback | Low | Android 13+ | Use back gesture and inspect Logcat | No warnings | Device + emulator<br>Telaboro v2.1.0 | Failed | NB-013 |
| TC-NB-015 | Ticket Reports legend | Low | EN interface | Analytics → Ticket Reports → inspect chart legend | Created / Resolved in English | Chrome, EN interface | Failed | NB-015 |
| TC-NB-018 | Escrow label | Low | EN interface | Operations → Escrow → inspect “In escrow” subtitle | `0 transactions` in English | Chrome, EN interface | Failed | NB-018 |

**Part 2 total:** 18 cases, all Failed because each represents a confirmed retest defect.

---

## 📊 Coverage Summary

| Module | Cases | Passed | Failed | Blocked |
|--------|------:|-------:|-------:|--------:|
| Mobile — Customer | 15 | 9 | 4 | 2 |
| Mobile — Technician | 16 | 6 | 7 | 3 |
| Admin Panel | 18 | 11 | 7 | 0 |
| Logs and integrations | 5 | 0 | 5 | 0 |
| Detailed negative cases | 18 | 0 | 18 | 0 |
| **Total** | **72** | **26** | **41** | **5** |

**Conclusion:** core positive flows such as registration, KYC, task creation, quotes, and chat work. Release blockers are concentrated in Stripe payments and data consistency, which is why the product is not ready for release.