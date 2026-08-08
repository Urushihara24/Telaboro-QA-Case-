# Telaboro QA Case Study

## 📱 О проекте

**Telaboro** — маркетплейс услуг для связи клиентов с мастерами (аналог TaskRabbit/Profi.ru для Латинской Америки). Платформа объединяет мобильное приложение (Android) и веб-админ-панель для управления операциями.

**Стек:**
- **Mobile:** React Native + Expo (Android)
- **Backend:** Node.js + Express + TypeScript
- **Database:** PostgreSQL + PostGIS
- **Payments:** Stripe Connect (Checkout, Payment Intents)
- **Real-time:** SocketIO
- **Push:** Firebase Cloud Messaging
- **Admin Panel:** React + TypeScript (Web)

---

##  Моя роль

**QA Engineer** — полное тестирование мобильного приложения и админ-панели.

**Объём работы:**
- 150+ тест-кейсов, покрывающих все критичные бизнес-флоу
- 55 багов в первом прогоне
- 18 новых багов в ретесте
- 3 критичных блокера, блокирующих релиз
- Анализ Logcat, API responses, Stripe webhook integration

---

## 🔍 Что тестировал

### Mobile App (Android)
- Онбординг и регистрация (клиент/мастер)
- KYC верификация (5 шагов с фото и динамическим кодом)
- Создание и управление задачами (In-person / Remote)
- Система котировок и принятия предложений
- Stripe платежи (Checkout, Connect, 3DS)
- Real-time чат между клиентом и мастером (SocketIO)
- Push-уведомления (FCM)
- Управление профилем и удаление аккаунта

### Admin Panel (Web)
- Dashboard и сводные метрики
- Аналитика (6 разделов: Conversion, Quotes, Technicians, Clients, Quality, Geography)
- Управление пользователями и правами (RBAC)
- Операции (платежи, споры, эскроу, выплаты, тикеты, геозоны)
- Каталог услуг и стран
- Система уровней амбассадоров и монетизации
- Push/Email рассылки
- Audit log и Crash Reports
- Системные настройки

---

## 📊 Результаты

### Первый прогон
| Severity | Количество |
|----------|------------|
| 🔴 Critical | 7 |
| 🟠 High | 9 |
| 🟡 Medium | 22 |
| 🔵 Low | 17 |
| **Всего** | **55** |

### Ретест (v2.1.0, чистая БД)
- **Исправлено:** 8 багов из 55 (15%)
- **Новых багов найдено:** 18
- **Критичных:** 2 (Stripe payment stuck, profile crash)

---

## 🚨 Ключевые находки

### 1. Stripe Payment Launcher Bug (Critical)

**Проблема:** Все платежи с картой зависают в статусе Pending.

**Root cause:** React Native `ActivityResultRegistry` дропает результат нативного `PaymentLauncherConfirmationActivity`. Stripe возвращает `RESULT_OK`, но приложение не получает данные платежа.

**Impact:** Клиенты не могут оплатить задачи → мастера не получают деньги → бизнес-процесс полностью заблокирован.

**Evidence:** Logcat показывает `Dropping pending result: RESULT_OK` при возврате из нативного Stripe Activity.

**Визуальное описание:** В админке Payments видно: Total charged: $0.00 MXN, Pending payments: 1 waiting. В таблице транзакций одна запись со статусом "Pending" (оранжевый бейдж), тип "Diagnosis" (фиолетовый бейдж), сумма $150.00.

---

### 2. Profile Crash (Critical)

**Проблема:** Краш публичного профиля мастера при открытии из котировки.

**Root cause:** Компонент `TechnicianProfileScreen` обращается к свойству `country`, которого нет в ответе API.

**Impact:** Клиенты не могут посмотреть профиль мастера перед принятием котировки — ключевой флоу принятия решения заблокирован.

**Evidence:** Crash Reports в админке: 4 новых краша, все New. Logcat: `ReferenceError: Property 'country' doesn't exist at TechnicianProfileScreen`.

**Визуальное описание:** Чёрный экран с оранжевой иконкой взрыва 💥, заголовком "Something went wrong", текстом "Property 'country' doesn't exist" и оранжевой кнопкой "Retry". При нажатии Retry цикл повторяется.

---

### 3. Payment Logic Bypass (Medium)

**Проблема:** 11 заказов Per quote ($27.50) прошли без ввода карты.

**Root cause:** Логика списания за котировки работает в обход Stripe. При уровне Bronce free allowance = 0, но платежи проходят автоматически.

**Impact:** Потеря контроля над платежами, возможные финансовые потери.

**Визуальное описание:** В админке Plan Orders: Total revenue $27.50 (11 paid orders), Per quote: 11. Таблица показывает 11 строк от одного мастера, каждая по $2.50, все со статусом "Paid" (зелёный).

---

## 🛠 Инструменты

- **Test Management:** Markdown documentation
- **Bug Tracking:** GitHub Issues
- **Log Analysis:** Logcat (Android Studio), Chrome DevTools
- **API Testing:** Browser DevTools, API responses analysis
- **Devices:** Nothing Phone 1 (Android 15), Pixel 9a Emulator (Android 17)

---

## 📈 Метрики качества

| Метрика | Значение |
|---------|----------|
| Test Coverage | 100% критичных флоу |
| Bug Detection Rate | 73 бага за 2 прогона |
| Critical Bugs Found | 9 (7 initial + 2 retest) |
| False Positive Rate | <5% |
| Retest Pass Rate | 15% (8/55 fixed) |

---

## 💡 Lessons Learned

1. **Stripe интеграция** — всегда проверять webhook и ActivityResult handling в React Native + Native modules
2. **React Native + Native modules** — частый источник багов при передаче данных между слоями
3. **Чистая БД для ретеста** — необходима для проверки метрик без шума от seed-данных
4. **i18n** — системная проблема, требует единого словаря переводов на всех слоях (UI, данные, легенды, фильтры)
5. **RBAC** — наличие ролей в системе ≠ их фактическое использование; важно проверять назначение прав

---

## 📂 Документация

- [Стратегия тестирования](docs/test-strategy.md)
- [Баг-репорты ретеста (18 багов)](docs/bugs.md)
- [Тест-кейсы ретеста (18 кейсов)](docs/test-cases.md)
- [Итоги ретеста](docs/retest-summary.md)
- [Окружение тестирования](docs/environment.md)
- [Logcat evidence](logs/)

---

## 📞 Контакты

**Email:** [твой email]  
**LinkedIn:** [твой LinkedIn]  
**Telegram:** @vsevolod

---

*Проект выполнен в августе 2026*  
*Примечание: Скриншоты и видео не включены в репозиторий из-за конфиденциальности данных приложения. Все баги содержат подробные текстовые описания UI и logcat evidence.*
