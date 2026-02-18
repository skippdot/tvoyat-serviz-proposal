# Техническая архитектура — Твоят Сервиз

[<< Функциональность](./02-FEATURES.md) | [Оглавление](./README.md) | [Дорожная карта >>](./04-MVP-ROADMAP.md)

---

## Стек технологий

### Мобильное приложение (iOS + Android)

| Компонент | Технология | Почему |
|-----------|------------|--------|
| Фреймворк | **React Native + Expo** | Один код → iOS + Android + Web |
| Навигация | Expo Router | File-based routing, deep linking |
| UI-библиотека | NativeWind (Tailwind CSS) | Быстрая стилизация, единый дизайн |
| Стейт | Zustand | Легковесный, простой |
| API-клиент | TanStack Query | Кэширование, оффлайн-поддержка |
| Формы | React Hook Form + Zod | Валидация на клиенте |
| Медиа | expo-image-picker, expo-camera | Фото/видео загрузка |
| Карты | react-native-maps | Показать местоположение сервиза |
| Push | expo-notifications + FCM/APNs | Уведомления |
| Язык | TypeScript | Типизация, надёжность |

### Веб-сайт

**Тот же код через Expo Web** — React Native Web компилируется в веб.
Или отдельный **Next.js** сайт с общим API и дизайн-системой.

**Рекомендация:** Expo Web для MVP (один код), отдельный Next.js для SEO-оптимизированного сайта потом.

### Бэкенд

| Компонент | Технология | Почему |
|-----------|------------|--------|
| Runtime | **Node.js / Bun** | Быстрый старт, TypeScript |
| Фреймворк | **Hono** или **Fastify** | Лёгкий, быстрый API |
| БД | **PostgreSQL** (Supabase) | Реляционная, надёжная, бесплатный тир |
| ORM | **Drizzle** | Type-safe, лёгкий |
| Auth | **Supabase Auth** | SMS, телефон, OAuth |
| Storage | **Supabase Storage** или **Cloudflare R2** | Фото/видео хранение |
| Хостинг | **Railway** или **Fly.io** | Простой деплой, дешёвый |
| Real-time | **Supabase Realtime** | WebSocket для статусов |

### Telegram-бот

| Компонент | Технология | Почему |
|-----------|------------|--------|
| Библиотека | **grammY** | TypeScript, middleware, простой |
| Хостинг | Тот же сервер или **Cloudflare Workers** | Webhook-based |
| Интеграция | Общий API с приложением | Единая логика |

## Архитектурная схема

```
┌─────────────────────────────────────────────────┐
│                    КЛИЕНТЫ                       │
├──────────┬──────────┬──────────┬────────────────┤
│ iOS App  │ Android  │   Web    │  Telegram Bot  │
│ (Expo)   │  (Expo)  │(Expo/Next)│   (grammY)    │
└────┬─────┴────┬─────┴────┬─────┴───────┬────────┘
     │          │          │             │
     └──────────┴──────┬───┴─────────────┘
                       │
                 ┌─────▼─────┐
                 │  REST API  │
                 │  (Hono)    │
                 └─────┬─────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
   ┌──────▼──────┐ ┌──▼───┐ ┌─────▼──────┐
   │ PostgreSQL  │ │ Auth │ │  Storage   │
   │ (Supabase)  │ │(SMS) │ │(фото/видео)│
   └─────────────┘ └──────┘ └────────────┘

┌─────────────────────────────────────────────────┐
│              АДМИН-ПАНЕЛЬ (Web)                  │
│         Управление записями, ценами,             │
│         запросами, слотами, клиентами             │
└─────────────────────────────────────────────────┘
```

## Модели данных (основные)

### User (клиент)
```
- id
- phone
- name
- telegram_id (nullable)
- language (bg/ru/en)
- created_at
```

### Vehicle (автомобиль клиента)
```
- id
- user_id → User
- make (марка)
- model (модель)
- year (год)
- vin (nullable)
- mileage
- license_plate
```

### ServiceRequest (запрос на ремонт)
```
- id
- user_id → User
- vehicle_id → Vehicle
- category (oil, brakes, suspension, other)
- description
- status (new, estimated, approved, rejected, in_progress, done)
- media[] (фото/видео URLs)
- created_at
```

### Estimate (оценка стоимости)
```
- id
- request_id → ServiceRequest
- items[] { name, type(labor/part), price, quantity }
- total
- valid_until
- notes
- created_at
```

### Booking (запись)
```
- id
- user_id → User
- vehicle_id → Vehicle
- request_id → ServiceRequest (nullable)
- slot_date
- slot_time_start
- slot_time_end
- service_type
- status (confirmed, cancelled, completed, no_show)
- reminder_sent
- created_at
```

### ServicePrice (прайс-лист)
```
- id
- category
- name_bg, name_ru, name_en
- price_from
- price_to
- duration_minutes
- active
```

### TimeSlot (шаблон расписания)
```
- id
- day_of_week (1-7)
- start_time
- end_time
- max_bookings (кол-во подъёмников)
```

## Безопасность

- SMS-верификация для регистрации
- JWT токены с refresh
- Rate limiting на API
- Фото/видео — проверка типа файла, ограничение размера (50MB видео, 10MB фото)
- HTTPS everywhere
- Данные клиентов не передаются третьим лицам

---

[<< Функциональность](./02-FEATURES.md) | [Оглавление](./README.md) | [Дорожная карта >>](./04-MVP-ROADMAP.md)
