# 🎓 Uni-Market — Студентський Маркетплейс

[![React](https://img.shields.io/badge/React-19-blue?logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Vite](https://img.shields.io/badge/Vite-6.x-646CFF?logo=vite)](https://vitejs.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4.x-38BDF8?logo=tailwindcss)](https://tailwindcss.com/)
[![Node.js](https://img.shields.io/badge/Node.js-Express_5-green?logo=nodedotjs)](https://nodejs.org/)
[![Firebase](https://img.shields.io/badge/Firebase-Firestore_%26_Storage_%26_Auth-FFCA28?logo=firebase)](https://firebase.google.com/)
[![Stripe](https://img.shields.io/badge/Stripe-Payment_Integration-635BFF?logo=stripe)](https://stripe.com/)

**Uni-Market** — це сучасна двовекторна веб-платформа (C2C маркетплейс), розроблена спеціально для студентської спільноти. Платформа дає змогу студентам легко купувати, продавати або обмінювати підручники, гаджети, побутову техніку для гуртожитку та інші товари або послуги.

---

## 📋 Зміст

- [✨ Ключові можливості](#-ключові-можливості)
- [🛠 Стек технологій](#-стек-технологій)
- [📐 Архітектура системи](#-архітектура-системи)
- [🗂 Структура проекту](#-структура-проекту)
- [🔌 API Ендпоінти](#-api-ендпоінти)
- [📦 Схема бази даних Firestore](#-схема-бази-даних-firestore)
- [⚙️ Інструкція з локального запуску](#️-інструкція-з-локального-запуску)
- [🔑 Змінні оточення (.env)](#-змінні-оточення-env)
- [🛡 Авторизація та Безпека](#-авторизація-та-безпека)
- [🌐 Деплойment](#-деплойment)

---

## ✨ Ключові можливості

1. **🔐 Аутентифікація та Профіль Користувача**:
   - Реєстрація та вхід через Firebase Auth (Email/Password & Google Provider).
   - Захист приватних маршрутів та захищені API-запити через JWT (ID Tokens).

2. **🛍 Каталог та Пошук**:
   - Динамічний фільтр за категоріями (Гаджети, Книги, Одяг, Дім та Навчання тощо).
   - Фільтрація за станом товару (Новий, Ідеальний, Вживаний).
   - Текстовий пошук товарів у реальному часі за допомогою Firestore prefix queries.
   - Сортування за датою створення та ліміти пагінації.

3. **➕ Додавання та Керування Оголошеннями**:
   - Зручна форма додавання товару із завантаженням фотографій (Multer + Firebase Storage).
   - Автоматична генерація унікальних публічних посилань на зображення.
   - Права доступу: видалення оголошення доступне лише його автору (перевірка UID на бекенді).

4. **⭐ Система Оцінювання та Переглядів**:
   - Автоматичний облік переглядів товару (`views increment`).
   - Рейтингова система (1–5 зірок) з розрахунком середньої оцінки та підрахунком кількості відгуків.

5. **❤️ Обране (Favorites)**:
   - Можливість додавати цікаві товари в обране та переглядати їх у власному кабінеті.

6. **🛒 Кошик та Онлайнові Платежі (Stripe)**:
   - Інтегрований кошик покупок з можливістю корекції кількості.
   - Повноцінна онлайн-оплата банківськими картками через Stripe PaymentIntents API.
   - Збереження історії замовлень із статусами доставки.

---

## 🛠 Стек технологій

### **Frontend (Client)**
* **Core**: [React 19](https://react.dev/) + [TypeScript](https://www.typescriptlang.org/)
* **Build Tool**: [Vite 6](https://vitejs.dev/)
* **Styling**: [Tailwind CSS v4](https://tailwindcss.com/)
* **Routing**: [React Router DOM v7](https://reactrouter.com/)
* **Client SDKs**: Firebase Web SDK v12, Stripe React SDK (`@stripe/react-stripe-js`)

### **Backend (Server)**
* **Runtime**: [Node.js](https://nodejs.org/)
* **Framework**: [Express 5](https://expressjs.com/)
* **Admin SDK**: Firebase Admin SDK v13/14 (Firestore DB, Cloud Storage, Auth)
* **File Uploads**: [Multer](https://github.com/expressjs/multer) (пам'ятний стрім для файлів до 2 МБ)
* **Payments**: Stripe Node.js SDK
* **Cross-Origin / Utilities**: CORS, dotenv

### **Database & Infrastructure**
* **Database**: Firebase Firestore (NoSQL Document Store)
* **Storage**: Firebase Cloud Storage (для зберігання медіа-файлів оголошень)
* **Auth Service**: Firebase Authentication

---

## 📐 Архітектура системи

Платформа побудована за архітектурним шаблоном **Decoupled Client-Server**: клієнтський фронтенд повністю відокремлений від бази даних і взаємодіє з сервером через захищений REST API.

```
Uni-Market Architecture:
┌──────────────────────────────────────┐
│   React 19 + TypeScript (Vite :5173) │  client/
│   ├─ Firebase Auth (Client SDK)      │
│   ├─ Stripe React Elements           │
│   └─ API Wrapper (fetchWithAuth)     │  ← Всі HTTP запити йдуть через /api
└──────────────────┬───────────────────┘
                   │ HTTP Proxy (/api → :3001)
                   ▼
┌──────────────────────────────────────┐
│   Node.js + Express 5 (Server :3001)  │  server/
│   ├─ verifyToken Middleware          │
│   ├─ Multer (Memory Storage)         │
│   ├─ Firebase Admin SDK              │
│   └─ Stripe Payments Controller      │
└─────────┬──────────────────┬─────────┘
          │                  │
          ▼                  ▼
┌──────────────────┐  ┌──────────────────┐
│ Firebase Cloud   │  │ Stripe API       │
│ ├─ Firestore DB  │  │ └─ PaymentIntent │
│ ├─ Storage       │  └──────────────────┘
│ └─ Auth Admin    │
└──────────────────┘
```

---

## 🗂 Структура проекту

```
Web_Uni-market/
├── AGENTS.md                  # Швидкий путівник архітектури та конвенцій для AI/розробників
├── README.md                  # Головна документація проекту
├── package.json               # Кореневі залежності
├── vercel.json                # Конфігурація для деплою на Vercel
│
├── client/                    # Фронтенд додаток (React + TypeScript)
│   ├── src/
│   │   ├── main.tsx           # Точка входу клієнта
│   │   ├── App.tsx            # Маршрутизація додатка (React Router)
│   │   ├── api/
│   │   │   └── api.ts         # Клієнт для взаємодії з /api ендпоінтами
│   │   ├── components/        # UI Компоненти (Header, Footer, Hero, ProductCard...)
│   │   ├── context/           # React Контексти (AuthContext, CartContext)
│   │   ├── firebase/          # Ініціалізація та конфігурація Firebase Web SDK
│   │   ├── pages/             # Сторінки (Home, Catalog, AddProduct, Cart, Checkout, Orders...)
│   │   └── utils/             # Допоміжні функції та утиліти
│   ├── vite.config.ts         # Конфігурація Vite та HTTP Proxy
│   └── package.json
│
└── server/                    # Бекенд додаток (Express + Firebase Admin)
    ├── index.js               # Точка входу Express API, контролери ендпоінтів
    ├── firebase.js            # Ініціалізація Firebase Admin SDK (db, bucket, admin)
    ├── serviceAccountKey.json # Секретний ключ доступу Firebase Admin (НЕ додавати в Git!)
    ├── test.http              # Приклади HTTP/REST запитів для тестування
    └── package.json
```

---

## 🔌 API Ендпоінти

Усі приватні ендпоінти вимагають наявності заголовка авторизації:  
`Authorization: Bearer {idToken}`

### 🛒 Товари (`/api/products`)

| Метод | Ендпоінт | Авторизація | Опис |
| :--- | :--- | :---: | :--- |
| `GET` | `/api/products` | ❌ | Отримання списку товарів (з фільтрацією за `category`, `condition` та `limit`) |
| `GET` | `/api/products/:id` | ❌ | Отримання деталей товару (автоматично збільшує `views` на 1) |
| `GET` | `/api/products/search?q=query` | ❌ | Пошук товарів за назвою |
| `POST` | `/api/products` | ✅ | Створення оголошення (multipart/form-data: title, price, description, category, condition, image) |
| `DELETE` | `/api/products/:id` | ✅ | Видалення товару (лише для власника оголошення) |
| `GET` | `/api/my-products` | ✅ | Отримання списку товарів поточного авторизованого користувача |
| `POST` | `/api/products/:id/rate` | ✅ | Оцінювання товару (Body: `{ rating: 1-5 }`) |

### ❤️ Обране (`/api/favorites`)

| Метод | Ендпоінт | Авторизація | Опис |
| :--- | :--- | :---: | :--- |
| `GET` | `/api/favorites` | ✅ | Отримання списку ID товарів в обраному |
| `POST` | `/api/favorites/:productId` | ✅ | Додавання товару в обране |
| `DELETE` | `/api/favorites/:productId` | ✅ | Видалення товару з обраного |

### 💳 Замовлення та Оплата (`/api/orders`, `/api/create-payment-intent`)

| Метод | Ендпоінт | Авторизація | Опис |
| :--- | :--- | :---: | :--- |
| `POST` | `/api/orders` | ✅ | Створення звичайного замовлення |
| `GET` | `/api/orders` | ✅ | Отримання історії замовлень користувача |
| `POST` | `/api/create-payment-intent` | ✅ | Створення платіжної сесії Stripe (PaymentIntent) |
| `POST` | `/api/stripe-orders` | ✅ | Фіксація замовлення у Firestore після успішної оплати Stripe |

---

## 📦 Схема бази даних Firestore

### Колекція `products`
```json
{
  "id": "doc_id_auto",
  "title": "Ноутбук Lenovo IdeaPad 3",
  "price": 14500,
  "description": "Чудовий ноутбук для навчання та програмування",
  "category": { "text": "Гаджети", "bgClass": "bg-blue-100", "textClass": "text-blue-600" },
  "condition": { "text": "Вживаний", "icon": "⚡" },
  "imageUrl": "https://storage.googleapis.com/unimarket-f3c72.appspot.com/products/17150000_photo.jpg",
  "sellerId": "uid_firebase_123",
  "sellerName": "Олександр",
  "views": 38,
  "createdAt": "Timestamp",
  "ratings": { "uid_1": 5, "uid_2": 4 },
  "rating": 4.5,
  "reviewsCount": 2
}
```

### Колекція `orders`
```json
{
  "id": "order_doc_id",
  "userId": "uid_buyer_123",
  "items": [ ... ],
  "totalAmount": 14500,
  "status": "paid",
  "stripePaymentId": "pi_3M...",
  "createdAt": "Timestamp"
}
```

---

## ⚙️ Інструкція з локального запуску

### Передумови
- Встановлений [Node.js](https://nodejs.org/) (версії v18 або вище)
- Обліковий запис [Firebase Project](https://console.firebase.google.com/) з увімкненими Firestore, Cloud Storage та Authentication.

### Крок 1: Клонування репозиторію
```bash
git clone https://github.com/your-username/Web_Uni-market.git
cd Web_Uni-market
```

### Крок 2: Налаштування та запуск Бекенду (Server)
```bash
cd server
npm install
```
Переконайтесь, що ви поклали `serviceAccountKey.json` в папку `server/` або налаштували змінні оточення (див. роздел нижче).

Запуск сервера на порту `3001`:
```bash
node index.js
```

### Крок 3: Налаштування та запуск Фронтенду (Client)
В іншому терміналі:
```bash
cd client
npm install
npm run dev
```
Клієнтський додаток буде доступний за адресою: `http://localhost:5173`.

---

## 🔑 Змінні оточення (.env)

Створіть файл `.env` у папці `server/`:

```env
PORT=3001
NODE_ENV=development

# Firebase Admin SDK Configuration
FIREBASE_PROJECT_ID=unimarket-f3c72
FIREBASE_CLIENT_EMAIL=firebase-adminsdk-xxxxx@unimarket-f3c72.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
FIREBASE_STORAGE_BUCKET=unimarket-f3c72.appspot.com

# Stripe Integration
STRIPE_SECRET_KEY=sk_test_...
```

Створіть файл `.env` у папці `client/`:
```env
VITE_FIREBASE_API_KEY=AIzaSy...
VITE_FIREBASE_AUTH_DOMAIN=unimarket-f3c72.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=unimarket-f3c72
VITE_FIREBASE_STORAGE_BUCKET=unimarket-f3c72.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
VITE_STRIPE_PUBLIC_KEY=pk_test_...
```

---

## 🛡 Авторизація та Безпека

* **Перевірка JWT токенів**: Клієнт передає `Bearer ID-token` Firebase Auth в заголовку кожного захищеного API запиту. Бекенд за допомогою `admin.auth().verifyIdToken()` верифікує токен та розшифровує дані користувача (`req.user`).
* **Захист ресурсів**: Операції видалення та редагування перевіряють відповідність `sellerId === req.user.uid`.
* **Обмеження файлів**: Пам'ятна обробка Multer обмежує максимальний розмір завантажуваних зображень до 2 МБ.

---

## 🌐 Деплойment

Проект підготовлений для розгортання на [Vercel](https://vercel.com/):
- Фронтенд збирається за допомогою `npm run build` у папці `client`.
- Конфігурація `vercel.json` у корені проекту забезпечує маршрутизацію API запитів на serverless-функцію або бекенд інстанс.

---

## 👥 Автори та Внесок
Мущинка Василь, Якимчук Олексій, Петрунів Дмитро, Хмара Владислав, Кириченко Кирило
Проект розроблено в рамках навчальної ініціативи створення студентських цифрових сервісів. 
