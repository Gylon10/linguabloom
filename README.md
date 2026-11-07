# Lingua Bloom 🌸

Тренажёр английского для русскоговорящих: короткие уроки, интервальное повторение слов,
грамматический справочник и мягкая геймификация (XP, серии, достижения).

**Стек:** Next.js 16 (App Router) · TypeScript · PostgreSQL · Drizzle ORM · Tailwind CSS 4 ·
bcryptjs + серверные сессии · Web Speech API для озвучки.

## Быстрый старт

```bash
# 1. зависимости
npm install

# 2. база данных (нужен PostgreSQL 14+)
createdb linguabloom
cp .env.example .env      # и укажите свой DATABASE_URL

# 3. схема + контент
npm run db:push           # drizzle-kit push — создаёт таблицы
npm run seed              # наполняет уроками, словами, грамматикой (идемпотентно)

# 4. запуск
npm run dev               # http://localhost:3000
# или
npm run build && npm run start
```

Проверка здоровья: `GET /api/health` → `{ "ok": true }`.

Как выложить проект в интернет бесплатно (Vercel + Neon) — пошагово в [HOSTING.md](./HOSTING.md).

### Демо-аккаунт

```
email:    demo1@linguabloom.app
пароль:   demo12345
```

У демо-аккаунта уже есть история: пройденные уроки, колода слов на повторение,
открытые достижения и XP за неделю — удобно смотреть дашборд, рейтинг и профиль
без прохождения курса. Ещё 5 демо-пользователей (demo2…demo6, пароль тот же)
создаются тем же сидом и наполняют рейтинг.

## Скрипты

| Команда | Что делает |
| --- | --- |
| `npm run dev` | dev-сервер |
| `npm run build` / `npm run start` | production-сборка и запуск |
| `npm run seed` | идемпотентный сид контента |
| `npm run db:push` | применить схему к БД |
| `npm run typecheck` | `tsc --noEmit` |
| `npm run lint` | ESLint |

## Что внутри

- **Контент:** 12 модулей (A1–B2), 36 уроков, 324 упражнения, 300 слов с транскрипцией и
  примерами, 12 грамматических тем с тренажёрами, 20 вопросов теста уровня, 12 достижений.
- **Урок:** 9 упражнений шести типов (выбор перевода, аудирование, пропуск слова, сборка
  предложения, пары, перевод RU→EN). Неверный ответ возвращается в конец очереди.
- **Повторение:** упрощённый SM-2 (`easeFactor`, `intervalDays`, `repetitions`, `dueAt`),
  оценки «Снова / Трудно / Хорошо / Легко», горячие клавиши.
- **XP:** 10 за верное упражнение, бонус +20 за урок без ошибок, 50% при повторном
  прохождении. Начисление только на сервере.
- **Серии:** засчитывается день с любым XP, пропуск дня обнуляет серию; дни считаются в
  часовом поясе пользователя.
- **Темы:** светлая/тёмная/системная, без мигания при загрузке, выбор хранится в
  `localStorage` и в профиле.

## Структура

```
src/
  app/
    page.tsx                 лендинг (авторизованных уводит на /dashboard)
    (auth)/login, register   вход и регистрация
    placement/               тест уровня (работает и без входа)
    (app)/                   защищённая зона с навигацией
      dashboard, courses, review, vocabulary, grammar, grammar/[slug],
      practice, profile, leaderboard
      loading.tsx            скелетоны
    lesson/[id]/             полноэкранный плеер урока
    api/                     health, auth, lessons, review, words, profile,
                             grammar, practice, placement
  components/
    ui, layout, exercises, review, vocabulary, grammar, practice, profile,
    charts, placement, marketing, auth, theme, tts
  db/
    schema.ts                таблицы, enum'ы, индексы, связи
    seed.ts                  идемпотентный сид
    content/                 слова (300), программа, грамматика, вопросы теста
  lib/
    auth.ts, srs.ts, xp.ts, streak.ts, answer.ts, levenshtein.ts, course.ts,
    stats.ts, progress.ts, validators.ts, dates.ts, evaluate.ts, tts.ts
```

## Безопасность и качество

- Пароли — bcrypt, сессии — httpOnly cookie (`SameSite=Lax`) + таблица `sessions`.
  Сессия живёт 30 дней, поэтому вход «запоминается» между визитами. Если приложение
  открыто во фрейме на другом сайте (превью), браузер не отправляет Lax-куки — клиент
  сообщает об этом флагом `embedded`, и сервер ставит `SameSite=None; Secure`, иначе
  пользователя после входа снова выбрасывало бы на форму входа.
- Все мутации проверяют сессию и владельца данных, вход валидируется через zod,
  ошибки возвращаются по-русски с корректными HTTP-кодами.
- SQL — только через Drizzle (параметризованные запросы), пользовательский контент не
  рендерится через `dangerouslySetInnerHTML`.
- `tsc --noEmit`, `eslint` и `next build` проходят без ошибок.
- Мобайл-фёрст (проверено на 375px), фокус-ринги, aria-label у иконочных кнопок,
  управление урока с клавиатуры (Enter, 1–4), `prefers-reduced-motion`.
