# Баг-репорты ретеста (18 новых багов)

## Critical (2)

### NB-003: Все платежи с картой зависают в Pending

**Severity:** Critical  
**Priority:** P1  
**Component:** Mobile-Client  
**Module:** Payments/Stripe

**Предусловия:**
1. Аккаунт клиента создан
2. Задача создана и котировка принята
3. Статус задачи: PENDING PAYMENT
4. Доступна тестовая карта 4242 08/27 253

**Шаги воспроизведения:**
1. Залогиниться под клиентом
2. Принять котировку → Complete payment
3. Ввести карту 4242 08/27 253
4. Пройти 3DS/biometric confirmation
5. Вернуться в приложение
6. Проверить статус платежа в админке (Payments)

**Ожидаемый результат:** Платёж обработан, статус задачи = Paid/Assigned. В админке Total charged увеличивается.

**Фактический результат:** Платёж зависает в Pending. В админке Payments: 1 waiting, Total charged: $0.00. В Logcat: 'Dropping pending result: RESULT_OK' — React Native теряет результат от PaymentLauncherConfirmationActivity.

**Root Cause:** React Native ActivityResultRegistry дропает результат нативного Stripe Activity. Stripe возвращает RESULT_OK, но приложение не получает данные платежа.

**Окружение:** Android устройство (Nothing Phone 1, Android 15), Приложение Telaboro v2.1.0

**Визуальное описание UI:**
- Админка Payments: карточки метрик показывают Total charged: $0.00 MXN, Total refunded: $0.00 MXN, Pending payments: 1 waiting (оранжевая иконка часов), Failed payments: 0, Open disputes: 0
- Таблица Transactions: одна строка "Test" (Usuario Eliminado), тип "Diagnosis" (фиолетовый бейдж), статус "Pending" (оранжевый бейдж), Amount: $150.00, Stripe PI: pi_3U0T25R8abMwWCv..., Date: Aug 03, 2026, 11:04 PM

**Logcat evidence:**
01:05:51.857 ActivityTaskManager I START u0 {cmp=com.telaboro.app/com.stripe.android.payments.paymentlauncher.PaymentLauncherConfirmationActivity}
01:05:53.271 ReactHost W ReactHost{0}.onHostResume(activity)
01:05:53.563 ActivityResultRegistry W Dropping pending result for request fragment_2563bee1-0533-4527-ad9d-1b48d92a118f_rq#0: ActivityResult{resultCode=RESULT_OK, data=Intent { (has extras) }}


---

### NB-014: Краш TechnicianProfileScreen: Property 'country' doesn't exist

**Severity:** Critical  
**Priority:** P1  
**Component:** Mobile-Client  
**Module:** Quotes/Technician Profile

**Предусловия:**
1. Аккаунт клиента создан
2. Мастер отправил котировку
3. Клиент находится на экране Quotes

**Шаги воспроизведения:**
1. Залогиниться под клиентом
2. Перейти в Quotes
3. Нажать на имя мастера для открытия публичного профиля
4. Проверить экран и Logcat

**Ожидаемый результат:** Открывается публичный профиль мастера с данными: имя, рейтинг, категории, локация

**Фактический результат:** Экран ошибки: 'Something went wrong — Property country doesn't exist' с кнопкой Retry. При нажатии Retry цикл повторяется. В Logcat: ReferenceError: Property 'country' doesn't exist at TechnicianProfileScreen.

**Root Cause:** Компонент TechnicianProfileScreen обращается к свойству country, которого нет в ответе API.

**Окружение:** Android устройство (Nothing Phone 1, Android 15) + Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

**Визуальное описание UI:**
- Чёрный экран с оранжевой иконкой взрыва 💥 по центру
- Заголовок: "Something went wrong" (белый текст)
- Подзаголовок: "Property 'country' doesn't exist" (серый текст)
- Оранжевая кнопка "Retry" по центру
- При нажатии Retry экран перезагружается и показывает ту же ошибку

**Logcat evidence:**
00:51:55.435 ViewRootImpl E Attempt to call method from wrong thread. This will throw an exception in a future version.
00:51:55.474 ReactNativeJS E { [ReferenceError: Property 'country' doesn't exist]
00:51:55.474 ReactNativeJS E   componentStack: '\n    at TechnicianProfileScreen (address at index.android.bundle:1:3060116)'
00:51:55.477 unknown:ReactNative E ReferenceError: Property 'country' doesn't exist


**Crash Reports в админке:** 4 новых краша, все со статусом "New" (красный бейдж), ошибка "ReferenceError: Property 'country' doesn't exist", платформа android 37, версия 2.1.0, 4 occurrences каждый.

---

## High (5)

### NB-002: is_first_purchase всегда true для всех платежей

**Severity:** High  
**Priority:** P2  
**Component:** Mobile-Technician  
**Module:** Payments/Plans

**Предусловия:**
1. Аккаунт мастера создан и verified
2. Мастер уже совершал покупки планов
3. Все предыдущие платежи в статусе Pending

**Шаги воспроизведения:**
1. Залогиниться под техником
2. Купить план Balance (100 quotes) — ввести карту
3. Дождаться статуса Pending
4. Купить ещё один план Balance
5. Проверить в API response поле is_first_purchase

**Ожидаемый результат:** is_first_purchase: false для второго и последующих покупок

**Фактический результат:** Всегда is_first_purchase: true. Логика смотрит на успешные транзакции, а их нет (все pending), поэтому каждая покупка считается первой.

**Root Cause:** Баг в логике определения first purchase — проверяется только paid_at, а не наличие любого заказа.

**Окружение:** Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

**Визуальное описание API response:**
- Массив из 3 объектов plan orders, все с полем `"is_first_purchase": true`
- Первый заказ: plan_type "balance", quantity 100, base_price "29.99", final_price "29.99", created_at "2026-08-03T20:53:24.198Z", paid_at null
- Второй заказ: plan_type "balance", quantity 100, base_price "29.99", final_price "29.99", created_at "2026-08-03T20:49:11.515Z", paid_at null
- Третий заказ: plan_type "subscription", quantity 30, base_price "19.99", final_price "19.99", created_at "2026-08-03T20:46:34.418Z", paid_at null

---

### NB-004: Кнопка Complete payment не блокируется после первой попытки

**Severity:** High  
**Priority:** P2  
**Component:** Mobile-Client  
**Module:** Payments

**Предусловия:**
1. Клиент принял котировку
2. Статус задачи: PENDING PAYMENT
3. Первая попытка оплаты зависла в Pending

**Шаги воспроизведения:**
1. Нажать Complete payment → ввести карту → Pay
2. Платёж зависает в Pending
3. Снова нажать Complete payment → ввести другую карту → Pay

**Ожидаемый результат:** Кнопка disabled после первой попытки ИЛИ переиспользование того же PaymentIntent

**Фактический результат:** Кнопка остаётся активной. При повторной попытке создаётся новый PaymentIntent, ошибка от Stripe: 'You cannot confirm this PaymentIntent because it has already succeeded after being previously confirmed'.

**Root Cause:** Фронт не блокирует кнопку и не переиспользует PaymentIntent.

**Окружение:** Android устройство (Nothing Phone 1, Android 15), Приложение Telaboro v2.1.0

---

### NB-005: Рассинхрон статусов после удаления клиента

**Severity:** High  
**Priority:** P2  
**Component:** Mobile-Technician  
**Module:** Tasks/Quotes

**Предусловия:**
1. Созданы аккаунты клиента и техника
2. Клиент создал задачу, техник отправил котировку
3. Клиент принял котировку (статус Accepted)
4. Клиент удалил аккаунт

**Шаги воспроизведения:**
1. Зайти под техником
2. Проверить Inbox → задача со статусом Cancelled
3. Проверить Quotes → задача со статусом Accepted
4. Кликнуть на задачу в Quotes → детали показывают Cancelled
5. Проверить счётчик Accepted в метриках

**Ожидаемый результат:** Статус консистентен везде (Cancelled). Счётчик Accepted уменьшается.

**Фактический результат:** В Inbox: Cancelled. В Quotes: Accepted. При клике в Quotes → детали показывают Cancelled. Счётчик показывает 1 Accepted.

**Root Cause:** Рассинхрон между списком Quotes и деталями задачи. Бэкенд не обновляет статус котировки при удалении клиента.

**Окружение:** Android устройство (Nothing Phone 1, Android 15) + Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

---

### NB-009: CalledFromWrongThreadException при навигации (RNScreens)

**Severity:** High  
**Priority:** P2  
**Component:** Mobile  
**Module:** Navigation

**Предусловия:**
1. Приложение запущено
2. Пользователь переходит между экранами

**Шаги воспроизведения:**
1. Открыть приложение
2. Перейти в Quotes
3. Нажать на имя мастера
4. Проверить Logcat

**Ожидаемый результат:** Навигация работает без ошибок в консоли

**Фактический результат:** В Logcat: CalledFromWrongThreadException — UI обновляется из потока mqt_v_js вместо main. Stack trace: RNScreens Screen.startTransitionRecursive.

**Root Cause:** React Native Screens обновляет UI из неправильного потока при удалении экрана из навигации.

**Окружение:** Android устройство (Nothing Phone 1, Android 15) + Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

**Logcat evidence:**
01:00:52.613 ViewRootImpl E Attempt to call method from wrong thread. This will throw an exception in a future version.
android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views. Expected: main Calling: mqt_v_js
at com.swmansion.rnscreens.Screen.startTransitionRecursive(Screen.kt:501)
at com.swmansion.rnscreens.Screen.startRemovalTransition(Screen.kt:463)


---

### NB-019: Невозможно отозвать конкретную роль у админа

**Severity:** High  
**Priority:** P2  
**Component:** Admin-Panel  
**Module:** System/Admins/Manage Roles

**Предусловия:**
1. Админ залогинен с правами Super Admin
2. В системе есть админ с несколькими ролями
3. Открыта модалка 'Manage Roles'

**Шаги воспроизведения:**
1. Открыть админку
2. Перейти в System → Admins
3. Нажать на иконку щита в колонке Actions для любого админа
4. В модалке 'Manage Roles' попытаться удалить/отозвать любую из назначенных ролей

**Ожидаемый результат:** Рядом с каждой ролью есть кнопка 'Revoke' или иконка удаления

**Фактический результат:** Кнопки удаления ролей отсутствуют. Единственный способ изменить набор ролей — удалить админа из списка Administrators и создать заново через '+ New Admin' с нужным набором ролей.

**Root Cause:** В модалке Manage Roles есть только кнопка '+ Assign Role', но нет кнопок отзыва ролей.

**Окружение:** Админ-панель (браузер Chrome, EN интерфейс, автоперевод отключён)

**Визуальное описание UI:**
- Модалка "Manage Roles" с заголовком и именем админа
- Секция "Assigned roles" с оранжевой кнопкой "+ Assign Role" справа
- 4 карточки ролей:
  1. Administrador de Finanzas — Assigned on 8/4/2026
  2. Administrador de Soporte — Assigned on 8/4/2026
  3. Administrador Técnico — Assigned on 8/4/2026
  4. Super Administrador — Assigned on 8/3/2026
- Каждая карточка имеет иконку щита слева, но НЕТ кнопок удаления справа
- Внизу информационное сообщение: "Roles determine which sections and actions this administrator can use in the backoffice."
- Кнопка "Close" в правом нижнем углу

**Security impact:** При удалении и повторном создании админа теряется история действий в Audit log (Actor привязан к старому ID).

---

## Medium (6)

### NB-001: Поле телефона принимает буквы и спецсимволы

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile  
**Module:** Registration/Profile

**Предусловия:**
1. Открыт экран регистрации или профиля
2. Доступно поле ввода телефона

**Шаги воспроизведения:**
1. Открыть экран регистрации
2. В поле телефона ввести: 123qwere123132412
3. Сохранить профиль

**Ожидаемый результат:** Ошибка валидации или маска ввода, принимающая только цифры и формат +XX XXX XXX XXXX

**Фактический результат:** Значение 123qwere123132412 принимается без ошибок и сохраняется в профиль

**Root Cause:** Отсутствует валидация на фронте и бэке. Поле принимает любой текст.

**Окружение:** Android устройство (Nothing Phone 1, Android 15), Приложение Telaboro v2.1.0

**Визуальное описание UI:**
- В админке Users таблица показывает пользователя с телефоном "123qwere123132412" под email
- В таблице Technicians видно: "TEST TEST TEST TEST", телефон "16509:004646aaaa" (содержит буквы и двоеточие)
- Другой техник: "NORGE GREGORIO SANTANA LEYVA", телефон "123456789"

---

### NB-006: Платёж 2.5 MXN (Per quote) проходит в обход Stripe

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile-Technician  
**Module:** Payments/Plans

**Предусловия:**
1. Аккаунт мастера создан, уровень Bronce (0 free quotes)
2. Клиент принимает котировку мастера

**Шаги воспроизведения:**
1. Залогиниться под техником
2. Клиент принимает котировку
3. Проверить Plan Orders в админке
4. Посчитать количество заказов Per quote и общую сумму

**Ожидаемый результат:** Платёж за Per quote проходит через Stripe с вводом карты, либо используется free allowance согласно уровню

**Фактический результат:** 11 заказов Per quote по $2.50 = $27.50, все Paid, без ввода карты. При уровне Bronce free allowance = 0, но платежи проходят автоматически.

**Root Cause:** Логика списания за котировки работает в обход Stripe. Возможно, используется внутренний баланс или баг в расчёте allowance.

**Окружение:** Админ-панель (браузер Chrome, EN интерфейс, автоперевод отключён)

**Визуальное описание UI:**
- Plan Orders dashboard: Total revenue $27.50 (11 paid orders), Subscriptions: 1, Prepaid: 3, Per quote: 11 (+ 0 gratuitas)
- Таблица показывает 11 строк от мастера "TEST TEST TEST TEST" (Bronce):
  - Каждая строка: план "Per quote" (оранжевый бейдж), 1 quote, Base price $2.50, Discount —, Credits used —, Total $2.50
  - Статус "Paid" (зелёный) с датой
  - Кнопка "Send to tickets" у каждой строки
- Даты заказов: Aug 04 12:05 AM, Aug 03 11:50 PM, Aug 03 11:49 PM (×9)

---

### NB-007: Cancelled задачи засоряют Inbox мастера

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile-Technician  
**Module:** Inbox

**Предусловия:**
1. Клиент создал задачу
2. Мастер видит её в Inbox
3. Клиент удалил аккаунт или отменил задачу
4. Задача перешла в статус Cancelled

**Шаги воспроизведения:**
1. Зайти под техником
2. Открыть Inbox
3. Найти задачу со статусом Cancelled
4. Попытаться откликнуться на неё

**Ожидаемый результат:** Cancelled задачи скрыты из Inbox по умолчанию ИЛИ имеют визуальное отличие и без возможности взаимодействия

**Фактический результат:** Cancelled задачи видны в Inbox наравне с активными, без визуального отличия, без возможности откликнуться.

**Root Cause:** Фронт не фильтрует Cancelled задачи из списка Inbox.

**Окружение:** Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

---

### NB-010: Stripe: Unable to set card brand tint color

**Severity:** Medium  
**Priority:** P3  
**Component:** Mobile-Client  
**Module:** Payments/Stripe

**Предусловия:**
1. Открыт экран Checkout
2. Вводится номер карты

**Шаги воспроизведения:**
1. Открыть Checkout
2. Ввести номер карты 4242
3. Проверить Logcat

**Ожидаемый результат:** Иконка бренда карты (Visa/Mastercard) отображается с корректным цветом

**Фактический результат:** В Logcat повторяется ошибка: 'Unable to set card brand tint color: com.stripe.android.view.CardBrandView.setTintColorInt'. Иконка может отображаться некорректно.

**Root Cause:** Баг в Stripe React Native SDK при установке цвета бренда карты.

**Окружение:** Android устройство (Nothing Phone 1, Android 15), Приложение Telaboro v2.1.0

**Logcat evidence:**
01:05:36.561 StripeReactNative E Unable to set card brand tint color: com.stripe.android.view.CardBrandView.setTintColorInt$payments_core_release [int]


---

### NB-016: Два статуса одновременно: Active + Blocked у пользователя

**Severity:** Medium  
**Priority:** P3  
**Component:** Admin-Panel  
**Module:** Users

**Предусловия:**
1. Админ залогинен
2. Аккаунт клиента заблокирован после неудачных попыток входа

**Шаги воспроизведения:**
1. Открыть админку
2. Перейти в Users
3. Найти заблокированного пользователя
4. Проверить колонку Status

**Ожидаемый результат:** Один статус: либо Active, либо Blocked

**Фактический результат:** У пользователя отображаются два статуса одновременно: Active (зелёный) и Blocked (красный).

**Root Cause:** Логическая ошибка в UI — пользователь не может быть одновременно активным и заблокированным.

**Окружение:** Админ-панель (браузер Chrome, EN интерфейс, автоперевод отключён)

**Визуальное описание UI:**
- Таблица Users, первая строка: "TEST TEST TEST TEST", телефон 123qwere123132412
- Колонка Status показывает ДВА бейджа:
  - "Active" (зелёный круг + зелёный текст)
  - "Blocked" (красный круг с замком + красный текст)
- Оба бейджа отображаются одновременно в одной ячейке

---

### NB-017: Несоответствие сумм: задача $123.00, платёж $150.00

**Severity:** Medium  
**Priority:** P3  
**Component:** Admin-Panel  
**Module:** Tasks/Payments

**Предусловия:**
1. Админ залогинен
2. Есть задача с котировкой $123.00
3. Создан платёж Diagnosis

**Шаги воспроизведения:**
1. Открыть админку
2. Перейти в Tasks → найти задачу Test
3. Проверить сумму Amount: $123.00
4. Перейти в Payments → найти платёж за задачу Test
5. Проверить сумму Amount: $150.00

**Ожидаемый результат:** Сумма задачи и платежа совпадают

**Фактический результат:** В Tasks сумма задачи Test: $123.00. В Payments сумма платежа за задачу Test: $150.00. Разница $27.00.

**Root Cause:** Платёж Diagnosis fee ($150.00) не совпадает с суммой котировки ($123.00). Возможно, Diagnosis fee — это отдельная фиксированная сумма.

**Окружение:** Админ-панель (браузер Chrome, EN интерфейс, автоперевод отключён)

**Визуальное описание UI:**
- Tasks таблица: строка "Test", статус "Cancelled" (красный бейдж), тип "On-site", клиент "Usuario Eliminado", техник "TEST TEST TEST TEST" (★ 0.0), Amount: $123.00, Quotes: 1
- Payments таблица: строка "Test" (Usuario Eliminado), тип "Diagnosis" (фиолетовый бейдж), статус "Pending" (оранжевый бейдж), Amount: $150.00, Stripe PI: pi_3U0T25R8abMwWCv..., Date: Aug 03, 2026, 11:04 PM

---

## Low (5)

### NB-011: RNScreens iOS-props on Android

**Severity:** Low  
**Priority:** P4  
**Component:** Mobile  
**Module:** Navigation

**Предусловия:**
1. Приложение запущено на Android

**Шаги воспроизведения:**
1. Открыть приложение
2. Перейти между экранами
3. Проверить Logcat

**Ожидаемый результат:** В Logcat нет предупреждений о недоступных props

**Фактический результат:** В Logcat повторяются предупреждения: backTitleVisible, backTitleFontFamily, disableBackButtonMenu, largeTitleFontFamily, largeTitleFontWeight, largeTitleHideShadow — prop is not available on Android.

**Root Cause:** В коде используются iOS-специфичные свойства RNScreens, которые игнорируются на Android.

**Окружение:** Android устройство (Nothing Phone 1, Android 15) + Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

**Logcat evidence:**
[RNScreens] W backTitleVisible prop is not available on Android
[RNScreens] W backTitleFontFamily prop is not available on Android
[RNScreens] W disableBackButtonMenu prop is not available on Android
[RNScreens] W largeTitleFontFamily prop is not available on Android
[RNScreens] W largeTitleFontWeight prop is not available on Android
[RNScreens] W largeTitleHideShadow prop is not available on Android


---

### NB-012: Firebase Analytics missing

**Severity:** Low  
**Priority:** P4  
**Component:** Mobile  
**Module:** Analytics

**Предусловия:**
1. Приложение запущено
2. Firebase Messaging настроен

**Шаги воспроизведения:**
1. Открыть приложение
2. Дождаться push-уведомления
3. Проверить Logcat

**Ожидаемый результат:** Firebase Analytics работает, события логируются

**Фактический результат:** В Logcat: 'FirebaseMessaging W Unable to log event: analytics library is missing'. Push работают, но аналитика событий — нет.

**Root Cause:** Firebase Analytics не подключён к проекту.

**Окружение:** Android устройство (Nothing Phone 1, Android 15) + Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

**Logcat evidence:**
FirebaseMessaging W Unable to log event: analytics library is missing


---

### NB-013: OnBackInvokedCallback not enabled

**Severity:** Low  
**Priority:** P4  
**Component:** Mobile  
**Module:** Navigation

**Предусловия:**
1. Приложение запущено на Android 13+

**Шаги воспроизведения:**
1. Открыть приложение
2. Использовать gesture назад
3. Проверить Logcat

**Ожидаемый результат:** Предиктивный back gesture работает корректно

**Фактический результат:** В Logcat: 'OnBackInvokedCallback is not enabled for the application. Set android:enableOnBackInvokedCallback=true in the application manifest.'

**Root Cause:** В AndroidManifest.xml не добавлен флаг enableOnBackInvokedCallback.

**Окружение:** Android устройство (Nothing Phone 1, Android 15) + Android эмулятор (Google Pixel 9a, Android 17), Приложение Telaboro v2.1.0

**Logcat evidence:**
WindowOnBackDispatcher W OnBackInvokedCallback is not enabled for the application.
Set 'android:enableOnBackInvokedCallback="true"' in the application manifest.


---

### NB-015: Легенда графика в Ticket Reports на испанском

**Severity:** Low  
**Priority:** P4  
**Component:** Admin-Panel  
**Module:** Analytics/Ticket Reports

**Предусловия:**
1. Админ залогинен
2. Интерфейс на EN

**Шаги воспроизведения:**
1. Открыть админку
2. Перейти в Analytics → Ticket Reports
3. Проверить легенду графика 'Created vs Resolved per day'

**Ожидаемый результат:** Легенда на английском: Created / Resolved

**Фактический результат:** Легенда на испанском: Creados / Resueltos. Заголовки таблицы и карточек на английском.

**Root Cause:** Неполное исправление бага i18n. Легенда графика не переведена.

**Окружение:** Админ-панель (браузер Chrome, EN интерфейс, автоперевод отключён)

**Визуальное описание UI:**
- Страница Ticket Reports, заголовок "Created vs Resolved per day" на английском
- График линейный с двумя линиями: синяя (Creados) и зелёная (Resueltos)
- Легенда под графиком: "Creados" (синий) и "Resueltos" (зелёный) — на ИСПАНСКОМ
- Таблица "KPIs by assigned admin" с заголовками на английском: Admin, Assigned, Resolved, Open, SLA %, Avg. time, Reminders

---

### NB-018: 0 transacciones в Escrow (остаток i18n)

**Severity:** Low  
**Priority:** P4  
**Component:** Admin-Panel  
**Module:** Operations/Escrow

**Предусловия:**
1. Админ залогинен
2. Интерфейс на EN

**Шаги воспроизведения:**
1. Открыть админку
2. Перейти в Operations → Escrow
3. Проверить подпись под метрикой 'In escrow'

**Ожидаемый результат:** Подпись на английском: 0 transactions

**Фактический результат:** Подпись на испанском: 0 transacciones. Заголовок карточки на английском.

**Root Cause:** Неполное исправление бага i18n. Подпись метрики не переведена.

**Окружение:** Админ-панель (браузер Chrome, EN интерфейс, автоперевод отключён)

**Визуальное описание UI:**
- Страница Escrow, 5 карточек метрик:
  - In escrow: 0.00 MXN (синяя иконка замка), подпись "0 transacciones" — на ИСПАНСКОМ
  - Released: 0.00 MXN (зелёная иконка щита)
  - Refunded: 0.00 MXN (фиолетовая иконка)
  - Fees collected: 0.00 MXN (оранжевая иконка графика)
  - Open disputes: 0 (красная иконка предупреждения)
- Все заголовки карточек на английском, но подпись под первой карточкой на испанском
