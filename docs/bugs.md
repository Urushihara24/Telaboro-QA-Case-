# Retest Bug Reports — 18 New Defects

## Critical — 2

### NB-003: All card payments remain stuck in Pending

**Severity:** Critical  
**Priority:** P1  
**Component:** Mobile-Client  
**Module:** Payments/Stripe

**Preconditions:**
1. Customer account exists.
2. Task exists and quote is accepted.
3. Task status is PENDING PAYMENT.
4. Stripe test card is available.

**Steps to reproduce:**
1. Log in as customer.
2. Accept a quote → Complete payment.
3. Enter the Stripe test card.
4. Complete 3DS/biometric confirmation.
5. Return to the application.
6. Check payment status in Admin → Payments.

**Expected result:** payment is processed, task becomes Paid/Assigned, and Total charged increases in the admin panel.

**Actual result:** payment remains Pending. Admin → Payments shows one waiting payment and Total charged: $0.00. Logcat contains `Dropping pending result: RESULT_OK`, meaning React Native loses the result returned by `PaymentLauncherConfirmationActivity`.

**Root Cause evidence:** React Native `ActivityResultRegistry` drops the result from the native Stripe Activity. Stripe returns `RESULT_OK`, but the application does not receive the payment result.

**Environment:** Nothing Phone 1, Android 15; Telaboro v2.1.0.

**UI description:**
- Payments metrics: Total charged $0.00 MXN, Total refunded $0.00 MXN, Pending payments 1 waiting, Failed payments 0, Open disputes 0.
- Transactions contains one `Diagnosis` payment with `Pending` status and amount $150.00.

**Logcat evidence:**
```text
01:05:51.857 ActivityTaskManager I START u0 {cmp=com.telaboro.app/com.stripe.android.payments.paymentlauncher.PaymentLauncherConfirmationActivity}
01:05:53.271 ReactHost W ReactHost{0}.onHostResume(activity)
01:05:53.563 ActivityResultRegistry W Dropping pending result for request fragment_2563bee1-0533-4527-ad9d-1b48d92a118f_rq#0: ActivityResult{resultCode=RESULT_OK, data=Intent { (has extras) }}
```

---

### NB-014: TechnicianProfileScreen crash — Property `country` doesn't exist

**Severity:** Critical  
**Priority:** P1  
**Component:** Mobile-Client  
**Module:** Quotes/Technician Profile

**Preconditions:**
1. Customer account exists.
2. Technician has sent a quote.
3. Customer is on the Quotes screen.

**Steps to reproduce:**
1. Log in as customer.
2. Open Quotes.
3. Tap the technician name to open the public profile.
4. Inspect the screen and Logcat.

**Expected result:** public technician profile opens with name, rating, categories, and location.

**Actual result:** error screen displays `Something went wrong — Property 'country' doesn't exist` with a Retry button. Retry loops back to the same error. Logcat shows `ReferenceError: Property 'country' doesn't exist at TechnicianProfileScreen`.

**Root Cause evidence:** `TechnicianProfileScreen` reads the `country` property, which is absent from the API response.

**Environment:** Nothing Phone 1, Android 15 + Google Pixel 9a emulator, Android 17; Telaboro v2.1.0.

**UI description:**
- Black screen with orange explosion icon.
- `Something went wrong` title.
- `Property 'country' doesn't exist` subtitle.
- Orange `Retry` button; retry reproduces the error.

**Logcat evidence:**
```text
00:51:55.435 ViewRootImpl E Attempt to call method from wrong thread. This will throw an exception in a future version.
00:51:55.474 ReactNativeJS E { [ReferenceError: Property 'country' doesn't exist]
00:51:55.474 ReactNativeJS E   componentStack: '\n    at TechnicianProfileScreen (address at index.android.bundle:1:3060116)'
00:51:55.477 unknown:ReactNative E ReferenceError: Property 'country' doesn't exist
```

**Crash Reports:** four new crashes, all `New`, with the same `ReferenceError`, Android platform, app version 2.1.0.

---

## High — 5

### NB-002: `is_first_purchase` is always true

**Severity:** High  
**Priority:** P2  
**Component:** Mobile-Technician  
**Module:** Payments/Plans

**Preconditions:** technician is verified, previous plan purchases exist, and previous payments are Pending.

**Steps to reproduce:**
1. Log in as technician.
2. Buy a Balance plan.
3. Wait for the payment to remain Pending.
4. Buy another plan.
5. Inspect `is_first_purchase` in the API response.

**Expected result:** `is_first_purchase: false` for the second and subsequent purchases.

**Actual result:** `is_first_purchase` remains `true`. The logic appears to rely on successful transactions; because all previous payments are Pending, each order is treated as the first purchase.

**Root Cause evidence:** first-purchase logic appears to check `paid_at` rather than the existence of previous orders.

**Environment:** Pixel 9a emulator, Android 17; Telaboro v2.1.0.

**API response description:** three plan-order objects all contain `"is_first_purchase": true`; prior orders have `paid_at: null`.

---

### NB-004: Complete payment remains active after the first attempt

**Severity:** High  
**Priority:** P2  
**Component:** Mobile-Client  
**Module:** Payments

**Preconditions:** quote accepted, task in PENDING PAYMENT, first attempt already stuck in Pending.

**Steps to reproduce:**
1. Select Complete payment → enter card → Pay.
2. Wait for Pending.
3. Select Complete payment again → use another card → Pay.

**Expected result:** payment control becomes disabled after the first attempt or the same PaymentIntent is reused.

**Actual result:** the button remains active. A new PaymentIntent is created and Stripe reports: `You cannot confirm this PaymentIntent because it has already succeeded after being previously confirmed`.

**Root Cause evidence:** frontend does not block repeated submission or reuse the existing PaymentIntent.

**Environment:** Nothing Phone 1, Android 15; Telaboro v2.1.0.

---

### NB-005: Status desynchronization after customer deletion

**Severity:** High  
**Priority:** P2  
**Component:** Mobile-Technician  
**Module:** Tasks/Quotes

**Preconditions:** customer and technician exist, quote was Accepted, then customer deletes the account.

**Steps to reproduce:**
1. Log in as technician.
2. Check Inbox: task shows Cancelled.
3. Check Quotes: task shows Accepted.
4. Open task from Quotes: details show Cancelled.
5. Check the Accepted metric counter.

**Expected result:** status is consistently Cancelled everywhere and the Accepted counter decreases.

**Actual result:** Inbox shows Cancelled; Quotes shows Accepted; details show Cancelled; Accepted counter remains 1.

**Root Cause evidence:** Quotes list and task details use inconsistent state; backend quote status is not updated when the customer is deleted.

**Environment:** physical Android device + Pixel 9a emulator; Telaboro v2.1.0.

---

### NB-009: CalledFromWrongThreadException during navigation — RNScreens

**Severity:** High  
**Priority:** P2  
**Component:** Mobile  
**Module:** Navigation

**Preconditions:** app is running and user navigates between screens.

**Steps to reproduce:**
1. Open the app.
2. Open Quotes.
3. Tap technician name.
4. Inspect Logcat.

**Expected result:** navigation works without thread-related runtime errors.

**Actual result:** Logcat shows `CalledFromWrongThreadException`: UI is updated from `mqt_v_js` instead of the main thread. Stack points to `RNScreens Screen.startTransitionRecursive`.

**Root Cause evidence:** React Native Screens updates/removes UI from the wrong thread during a navigation transition.

**Environment:** physical Android device + Pixel 9a emulator; Telaboro v2.1.0.

**Logcat evidence:**
```text
01:00:52.613 ViewRootImpl E Attempt to call method from wrong thread. This will throw an exception in a future version.
android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views. Expected: main Calling: mqt_v_js
at com.swmansion.rnscreens.Screen.startTransitionRecursive(Screen.kt:501)
at com.swmansion.rnscreens.Screen.startRemovalTransition(Screen.kt:463)
```

---

### NB-019: Cannot revoke an individual admin role

**Severity:** High  
**Priority:** P2  
**Component:** Admin-Panel  
**Module:** System/Admins/Manage Roles

**Preconditions:** Super Admin is logged in; another admin has multiple roles; Manage Roles is open.

**Steps to reproduce:**
1. Open System → Admins.
2. Select the shield action for an administrator.
3. In Manage Roles, attempt to remove any assigned role.

**Expected result:** each assigned role provides a Revoke/remove action.

**Actual result:** no role-removal control exists. The only available workaround is deleting the administrator from the Administrators list and recreating the account with a different role set.

**Root Cause evidence:** Manage Roles supports `+ Assign Role` but no role-revocation action.

**Environment:** Chrome admin panel, EN interface, automatic translation disabled.

**UI description:**
- `Manage Roles` modal with administrator name.
- `Assigned roles` section and orange `+ Assign Role` button.
- Multiple role cards with shield icons but no remove controls.
- Informational note: `Roles determine which sections and actions this administrator can use in the backoffice.`
- `Close` button.

**Security impact:** deleting and recreating an admin can break continuity in the Audit log because actions remain associated with the previous account ID.

---

## Medium — 6

### NB-001: Phone field accepts letters and special characters

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile  
**Module:** Registration/Profile

**Preconditions:** registration/profile screen is open and phone input is available.

**Steps to reproduce:**
1. Enter `123qwere123132412` into the phone field.
2. Save the profile.

**Expected result:** validation error or a phone mask accepting only a valid phone format.

**Actual result:** the value is accepted and saved without validation.

**Root Cause evidence:** no effective frontend/backend validation is applied to the field.

**Environment:** Nothing Phone 1, Android 15; Telaboro v2.1.0.

**UI evidence:** Users/Technicians tables contain saved values with letters, punctuation, or malformed phone formats.

---

### NB-006: 2.5 MXN Per quote charge bypasses Stripe

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile-Technician  
**Module:** Payments/Plans

**Preconditions:** technician is Bronce with 0 free quotes; customer accepts the technician's quote.

**Steps to reproduce:**
1. Let customer accept the quote.
2. Open Admin → Plan Orders.
3. Count Per quote orders and total amount.
4. Verify whether card entry occurred.

**Expected result:** Per quote payment goes through Stripe/card flow or consumes free allowance according to the level rules.

**Actual result:** 11 Per quote orders at $2.50 each, $27.50 total, are all marked Paid without card input, even though Bronce free allowance is 0.

**Root Cause hypothesis:** quote-charge logic appears to bypass Stripe, potentially through internal balance logic or an allowance-calculation defect.

**Environment:** Chrome admin panel, EN interface.

**UI description:** Plan Orders shows Total revenue $27.50, 11 paid orders, and 11 Per quote items at $2.50 each, all Paid.

---

### NB-007: Cancelled tasks clutter the technician Inbox

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile-Technician  
**Module:** Inbox

**Preconditions:** a task visible in Inbox becomes Cancelled after customer deletion or task cancellation.

**Steps to reproduce:**
1. Log in as technician.
2. Open Inbox.
3. Find the Cancelled task.
4. Attempt to interact with it.

**Expected result:** Cancelled tasks are hidden by default or clearly separated as non-actionable.

**Actual result:** Cancelled tasks remain mixed with active Inbox items without visual distinction and cannot be responded to.

**Root Cause evidence:** frontend does not filter or visually separate Cancelled tasks.

**Environment:** Pixel 9a emulator, Android 17; Telaboro v2.1.0.

---

### NB-010: Stripe — Unable to set card brand tint color

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile-Client  
**Module:** Payments/Stripe

**Preconditions:** Checkout is open and a card number is being entered.

**Steps to reproduce:**
1. Open Checkout.
2. Enter a Visa test-card prefix such as `4242`.
3. Inspect Logcat.

**Expected result:** card-brand icon is displayed using the expected styling without SDK errors.

**Actual result:** Logcat repeatedly reports `Unable to set card brand tint color: com.stripe.android.view.CardBrandView.setTintColorInt`; the icon may render incorrectly.

**Root Cause evidence:** Stripe React Native SDK tint application fails for the card-brand view.

**Environment:** Nothing Phone 1, Android 15; Telaboro v2.1.0.

**Logcat evidence:**
```text
01:05:36.561 StripeReactNative E Unable to set card brand tint color: com.stripe.android.view.CardBrandView.setTintColorInt$payments_core_release [int]
```

---

### NB-016: User displays Active + Blocked simultaneously

**Severity:** Medium  
**Priority:** P3  
**Component:** Admin-Panel  
**Module:** Users

**Preconditions:** administrator is logged in and a customer account has been blocked after failed login attempts.

**Steps to reproduce:**
1. Open Admin → Users.
2. Find the blocked user.
3. Inspect the Status column.

**Expected result:** one coherent status, either Active or Blocked.

**Actual result:** both Active and Blocked badges are shown in the same status cell.

**Root Cause evidence:** UI status logic allows mutually exclusive states to render together.

**Environment:** Chrome admin panel, EN interface.

---

### NB-017: Task amount $123.00 does not match payment amount $150.00

**Severity:** Medium  
**Priority:** P3  
**Component:** Admin-Panel  
**Module:** Tasks/Payments

**Preconditions:** task with $123 quote exists and a Diagnosis payment was created.

**Steps to reproduce:**
1. Open Tasks and find the target task; note Amount $123.00.
2. Open Payments and find the payment for the same task; note Amount $150.00.
3. Compare values.

**Expected result:** task and payment amounts are consistent according to documented business rules.

**Actual result:** task amount is $123.00 while the Diagnosis payment is $150.00, a $27 difference.

**Root Cause hypothesis:** Diagnosis fee may be a separate fixed amount; business logic needs clarification before final defect closure.

**Environment:** Chrome admin panel, EN interface.

---

## Low — 5

### NB-011: RNScreens iOS-only props on Android

**Severity:** Low  
**Priority:** P4  
**Component:** Mobile  
**Module:** Navigation

**Preconditions:** application runs on Android.

**Steps to reproduce:** navigate between screens and inspect Logcat.

**Expected result:** no warnings about unavailable props.

**Actual result:** repeated warnings for `backTitleVisible`, `backTitleFontFamily`, `disableBackButtonMenu`, `largeTitleFontFamily`, `largeTitleFontWeight`, and `largeTitleHideShadow`, all unavailable on Android.

**Root Cause evidence:** iOS-specific RNScreens properties are configured in the Android flow.

**Environment:** physical Android device + Pixel 9a emulator; Telaboro v2.1.0.

---

### NB-012: Firebase Analytics missing

**Severity:** Low  
**Priority:** P4  
**Component:** Mobile  
**Module:** Analytics

**Preconditions:** app is running and Firebase Messaging is configured.

**Steps to reproduce:** wait for a push notification and inspect Logcat.

**Expected result:** analytics events log without errors.

**Actual result:** Logcat reports `FirebaseMessaging W Unable to log event: analytics library is missing`. Push delivery works, but analytics event logging does not.

**Root Cause evidence:** Firebase Analytics is not included/configured in the project.

**Environment:** physical Android device + Pixel 9a emulator; Telaboro v2.1.0.

---

### NB-013: OnBackInvokedCallback is not enabled

**Severity:** Low  
**Priority:** P4  
**Component:** Mobile  
**Module:** Navigation

**Preconditions:** Android 13+ device.

**Steps to reproduce:** use the back gesture and inspect Logcat.

**Expected result:** predictive back behavior works without warnings.

**Actual result:** Logcat reports `OnBackInvokedCallback is not enabled for the application. Set android:enableOnBackInvokedCallback=true in the application manifest.`

**Root Cause evidence:** `android:enableOnBackInvokedCallback` is not enabled in the application manifest.

**Environment:** physical Android device + Pixel 9a emulator; Telaboro v2.1.0.

---

### NB-015: Ticket Reports chart legend remains in Spanish

**Severity:** Low  
**Priority:** P4  
**Component:** Admin-Panel  
**Module:** Analytics/Ticket Reports

**Preconditions:** admin is logged in and interface language is EN.

**Steps to reproduce:** open Analytics → Ticket Reports and inspect the legend for `Created vs Resolved per day`.

**Expected result:** legend is English: Created / Resolved.

**Actual result:** legend remains Spanish: `Creados / Resueltos`, while surrounding headings are English.

**Root Cause evidence:** incomplete i18n coverage for chart legend strings.

**Environment:** Chrome admin panel, EN interface.

---

### NB-018: `0 transacciones` remains in Escrow

**Severity:** Low  
**Priority:** P4  
**Component:** Admin-Panel  
**Module:** Operations/Escrow

**Preconditions:** admin is logged in and interface language is EN.

**Steps to reproduce:** open Operations → Escrow and inspect the subtitle under `In escrow`.

**Expected result:** `0 transactions` in English.

**Actual result:** subtitle remains Spanish: `0 transacciones`, while card headings are English.

**Root Cause evidence:** incomplete i18n coverage for the metric subtitle.

**Environment:** Chrome admin panel, EN interface.

**UI description:** the Escrow page contains five metric cards; the first card reads `In escrow: 0.00 MXN` with the Spanish subtitle `0 transacciones`, while the other visible headings remain English.