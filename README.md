# 🧠 Психосоматика — MVP

**Оновлено:** 01-08-2025 17:00:00

> Roadmap-driven проект мобільного додатку з AI-агентом для маршрутизації користувачів до відповідного терапевта на основі симптомів.

---

## 📌 Опис

Першочергова версія MVP для iOS/Android без відео- або аудіо-комунікації. Користувач:

1. Реєструється та авторизується.
2. Відповідає на серію запитань через AI-агента (OpenAI GPT-4.5).
3. Отримує рекомендацію терапевта зі списку (внесеного вручну).
4. Спілкується з терапевтом у текстовому чаті.
5. Може залишити відгук.

Інтерфейс — українською, з перспективою підтримки англійської як другої мови.

---

## 🧩 Функціональні модулі

| Модуль         | Опис                                                               |
| -------------- | ------------------------------------------------------------------ |
| Authentication | Реєстрація/вхід (email, Google, Apple через Firebase або Supabase) |
| AI Routing     | Аналіз симптомів, рекомендація терапевта через OpenAI API          |
| Chat           | Реал-тайм чат (на базі Socket.io або Pusher)                       |
| Profiles       | Профілі користувача та терапевта                                   |
| Feedback       | Можливість залишити відгук про терапевта                           |

---

## ⚙️ Технологічний стек

* **Фронтенд:** React Native (Expo), TypeScript, React Navigation, Zustand або Redux Toolkit, i18next, Expo SecureStore
* **Бекенд:** Node.js + Express, PostgreSQL (Supabase або Railway), Prisma ORM
* **AI:** OpenAI GPT-4.5 Turbo (API)
* **Аутентифікація:** Firebase Auth або Supabase Auth
* **Чат:** Socket.io або Pusher
* **Платежі (на перспективу):** Stripe

---

## 🛠️ Налаштування локального середовища

1. Клонувати репозиторій:

   ```bash
   git clone https://github.com/OWNER/REPO.git
   cd REPO
   ```
2. Створити файл `.env` на основі `.env.example`:

   ```env
   # Backend
   DATABASE_URL=...
   OPENAI_API_KEY=...
   AUTH_PROVIDER_SUPABASE_URL=...
   AUTH_PROVIDER_SUPABASE_KEY=...

   # Frontend
   EXPO_APPLE_CLIENT_ID=...
   EXPO_FIREBASE_CONFIG=...
   ```
3. Встановити залежності:

   ```bash
   # Backend
   cd backend && npm install

   # Frontend
   cd ../mobile && npm install
   ```

---

## ▶️ Запуск проєкту

### Бекенд

```bash
cd backend
npm run dev    # Запустить сервер на http://localhost:4000
```

### Мобільний додаток

```bash
cd mobile
expo start     # Відкриє Metro Bundler у браузері
```

---

## 📋 Управління розробкою

Використовується **GitHub Projects (Roadmap-driven)**:

* 📌 [Roadmap View](https://github.com/OWNER/REPO/projects) — групування задач за Milestones
* Статусні колонки: `🧠 Idea`, `✍️ Specification`, `🛠️ In Progress`, `🔍 QA`, `✅ Done`
* Labels: `feature`, `bug`, `AI`, `auth`, `chat`, `UI`, `infra`, `MVP`, тощо

---

## 📂 Структура репозиторію

```bash
├── backend/          # Node.js + Express API
│   ├── prisma/       # Схема БД, міграції
│   ├── src/          # Контролери, маршрути, middleware
│   └── .env.example
├── mobile/           # React Native (Expo)
│   ├── src/          # Екрани, навігація, компоненти
│   └── app.json
├── README.md
└── .github/          # CI-конфігурації, GitHub Projects
```

---

## 📞 Контакти

* **Відповідальний:** Sam (Software Engineer, Ужгород)
* **Email:** [openminds.dev.studio@gmail.com](mailto:openminds.dev.studio@gmail.com)

---

> *Додаток спроєктований з урахуванням масштабування: підтримка адмін-панелі, платіжних систем та медіа-комунікації планується в наступних релізах.*
