# Как выложить Lingua Bloom в интернет бесплатно

Приложению нужны две вещи: **Node.js-сервер** (Next.js) и **PostgreSQL**. Оба можно
взять на бесплатных тарифах. Ниже — самый простой рабочий путь:

```
GitHub (код) → Neon (база данных, бесплатно) → Vercel (хостинг, бесплатно)
```

Итог: сайт живёт на адресе вида `https://linguabloom.vercel.app`, вход в аккаунт
запоминается (куки «свои», обычный браузер, никаких фреймов), база — 0,5 ГБ бесплатно.

---

## Шаг 0. Что понадобится

- Аккаунт GitHub — бесплатно, github.com
- Аккаунт Vercel — бесплатно, можно войти через GitHub, vercel.com
- Аккаунт Neon — бесплатно, можно войти через GitHub, neon.tech
- Node.js 20+ на компьютере (нужен только один раз — залить схему и данные в базу)

Никаких карт и платежей для тарифов Hobby / Free не требуется.

---

## Шаг 1. Положить код на GitHub

1. Скачайте архив проекта (в этом чате есть файл `linguabloom-source.zip`) и распакуйте.
2. Зайдите на github.com → **New repository**.
3. Имя: `linguabloom`, тип: **Private** (или Public — не важно), **Create repository**.
4. На странице пустого репозитория нажмите **uploading an existing file** и
   перетащите **содержимое** папки (не саму папку): `src`, `public`, `package.json`,
   `package-lock.json`, `tsconfig.json`, `next.config.ts`, `postcss.config.mjs`,
   `drizzle.config.ts`, `eslint.config.mjs`, `README.md`, `HOSTING.md`, `.gitignore`,
   `.env.example`. Затем **Commit changes**.

Проверка: в репозитории не должно быть папок `node_modules` и `.next` и файла `.env`
(в них секреты и мусор; `.gitignore` их и так не пустит при загрузке через git).

Альтернатива через терминал, если git установлен:

```bash
cd linguabloom
git init
git add .
git commit -m "Lingua Bloom"
git branch -M main
git remote add origin https://github.com/ВАШ_ЛОГИН/linguabloom.git
git push -u origin main
```

---

## Шаг 2. Бесплатная база PostgreSQL — Neon

1. neon.tech → **Sign up** (через GitHub) → **Create project**.
2. Имя проекта: `linguabloom`, регион — любой ближайший (например, `eu-central-1`).
3. На странице проекта нажмите **Connect** и скопируйте строку подключения.
   Берите вариант **Pooled connection** — в адресе хоста есть `-pooler`:

   ```
   postgresql://USER:PASSWORD@ep-xxx-pooler.eu-central-1.aws.neon.tech/neondb?sslmode=require
   ```

   Важно: `?sslmode=require` в конце обязателен.

4. Сохраните эту строку — она понадобится дважды (для Vercel и для заливки данных).

Тариф Free: 0,5 ГБ хранилища, база «засыпает» при простое и просыпается за ~1 секунду.

---

## Шаг 3. Залить таблицы и контент в облачную базу (один раз, со своего компьютера)

Схему и все уроки/слова нужно создать в новой базе. Делается локально:

```bash
# в папке проекта
npm install

# временно указываем облачную базу
echo "DATABASE_URL=postgresql://USER:PASSWORD@ep-xxx-pooler.eu-central-1.aws.neon.tech/neondb?sslmode=require" > .env

npm run db:push     # создать таблицы
npm run seed        # залить 12 модулей, 36 уроков, 300 слов, грамматику, достижения
```

Одна команда вместо двух: `npm run db:setup`.

Проверка: `npm run seed` в конце печатает «✅ Готово. Слов в базе: 300…».
Сид идемпотентный — можно запускать повторно, дубликатов не будет.

### Если Node.js на компьютере нет

Тот же шаг можно сделать кнопкой на GitHub, без установки чего-либо:

1. В репозитории: **Settings → Secrets and variables → Actions → New repository secret**,
   имя `DATABASE_URL`, значение — строка подключения Neon.
2. Вкладка **Actions → DB setup → Run workflow → Run workflow**.
3. Через минуту в базе будут таблицы и весь контент (лог покажет «✅ Готово»).

Файл воркфлоу уже лежит в проекте: `.github/workflows/db-setup.yml`.

> Windows/PowerShell: вместо `echo` создайте файл `.env` вручную с одной строкой
> `DATABASE_URL=...`. После шага 4 файл `.env` можно удалить (в облаке строка
> подключения задаётся в настройках Vercel).

---

## Шаг 4. Хостинг — Vercel

1. vercel.com → **Sign up / Log in** через GitHub.
2. **Add New… → Project** → выберите репозиторий `linguabloom` → **Import**.
3. Framework Preset определится сам — **Next.js**, менять ничего не нужно
   (Build Command `next build`, Output — по умолчанию).
4. Раскройте **Environment Variables** и добавьте две переменные:

   | Name           | Value                                                              |
   | -------------- | ------------------------------------------------------------------ |
   | `DATABASE_URL` | строка подключения Neon (с `-pooler` и `?sslmode=require`)          |
   | `APP_TIMEZONE` | `Europe/Moscow`                                                    |

5. Нажмите **Deploy** и подождите 1–2 минуты.

Готово: Vercel выдаст адрес `https://<имя-проекта>.vercel.app`. Открывайте его
напрямую в браузере (не в превью) — это обычный сайт, и вход в аккаунт будет
запоминаться на 30 дней.

Проверка после деплоя:

- `https://<адрес>/api/health` → `{"ok":true,"db":"up"}`
- регистрация и вход работают, после перезагрузки страницы вы остаётесь в аккаунте.

---

## Бесплатный демо-вход

После сида в базе есть аккаунты: `demo1@linguabloom.app` … `demo6@linguabloom.app`,
пароль у всех `demo12345`. У них уже есть прогресс, слова и достижения.

---

## Если vercel.com не открывается

Vercel иногда недоступен из РФ. Равноценные бесплатные варианты:

| Платформа   | Что даёт                                                        | Особенность                                    |
| ----------- | --------------------------------------------------------------- | ---------------------------------------------- |
| **Netlify** | бесплатный тариф, поддержка Next.js, адрес `*.netlify.app`       | в настройках укажите плагин Next.js (ставится сам) |
| **Koyeb**   | бесплатный web-сервис, Docker или Node                           | сервис «засыпает» при простое                  |
| **Render**  | бесплатный web-сервис, автодеплой из GitHub                      | засыпает через 15 мин простоя, просыпается ~30 с |

База данных в любом случае — Neon (или Supabase, у него тоже бесплатные 0,5 ГБ).
Для Netlify/Koyeb/Render понадобятся те же две переменные окружения
(`DATABASE_URL`, `APP_TIMEZONE`), а команда сборки — `npm run build`, запуска — `npm start`.

Совсем без облака: запустить на своём компьютере и открыть доступ через бесплатный
туннель (`cloudflared tunnel --url http://localhost:3000`) — но тогда сайт работает,
только пока включён компьютер.

---

## Свой домен (по желанию)

Домен можно купить отдельно и подключить в Vercel: **Project → Settings → Domains →
Add**, далее прописать у регистратора записи, которые покажет Vercel. На бесплатном
тарифе это разрешено, сам домен обычно платный (100–1000 ₽/год).

---

## Обновление сайта

Vercel деплоит автоматически при каждом пуше в ветку `main`:

```bash
git add .
git commit -m "Правки"
git push
```

Через минуту изменения уже на сайте.

---

## Частые вопросы и проблемы

**После деплоя «Application error» / страницы падают.**
Скорее всего, не задан `DATABASE_URL` или в нём нет `?sslmode=require`.
Проверьте: Vercel → Project → Settings → Environment Variables → **Redeploy**.

**`/api/health` показывает `"db":"down"`.**
Строка подключения скопирована без `-pooler`, либо база Neon удалена/спит и
не просыпается. Проверьте строку в Neon → Connect → Pooled connection.

**Пустые курсы, нет слов.**
Не выполнен шаг 3. Локально с облачным `DATABASE_URL` в `.env` запустите
`npm run db:push && npm run seed`.

**Первый запрос идёт 2–3 секунды.**
Это бесплатный тариф Neon: база «просыпается». Дальше отвечает быстро.

**Хочу начать с нуля.**
Удалите проект в Neon и создайте новый, затем повторите шаг 3.

**Локальный запуск остался прежним:**

```bash
npm install
cp .env.example .env     # локальная строка DATABASE_URL
npm run db:push && npm run seed
npm run dev              # http://localhost:3000
```
