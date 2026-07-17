# MLT — Esports Platform

Платформа киберспортивной лиги: расписание матчей, автоматическая
статистика игроков и команд, турнирные сетки, рейтинги, новости.
Полностью на клиенте — весь бэкенд это Firebase (Firestore + Auth),
без сервера и без сборщика. Работает на бесплатном плане **Spark**
(Firebase Storage не используется — см. `PROJECT_PLAN.md`).

Подробный план работы, схема данных и список того, что уже сделано —
в [`PROJECT_PLAN.md`](./PROJECT_PLAN.md).

## Структура репозитория

```
index.html              — весь сайт (вёрстка + Firebase-логика в одном файле)
firestore.rules          — правила безопасности Firestore
firebase.json             — конфиг Firebase Hosting (+ путь к firestore.rules)
.firebaserc                — алиас проекта Firebase (mlt-site)
.github/workflows/          — автодеплой на Firebase Hosting через GitHub Actions
PROJECT_PLAN.md              — живой план проекта / прогресс
```

## Запуск локально

Сборщик не нужен — это статический файл, подключающийся к Firebase
через CDN-модули. Просто откройте `index.html` в браузере, либо
поднимите любой локальный статический сервер, например:
```
python3 -m http.server 8080
```
и откройте `http://localhost:8080`.

## Деплой

Есть два независимых варианта — выберите любой, они не конфликтуют.

### Вариант A — GitHub Pages (проще всего, не требует Firebase CLI)

1. Запушьте репозиторий на GitHub.
2. Settings → Pages → Source: **Deploy from a branch** → Branch: `main` / `/(root)`.
3. Через минуту сайт будет доступен на `https://<ваш-логин>.github.io/<репозиторий>/`.

Так как весь бэкенд — это прямые вызовы к Firebase из браузера,
GitHub Pages подходит без каких-либо дополнительных настроек.

### Вариант B — Firebase Hosting

**Вручную, с вашего компьютера:**
```
npm install -g firebase-tools
firebase login
firebase deploy --only hosting
```
Сайт появится на `https://mlt-site.web.app`.

**Автоматически при пуше в `main` (уже настроено в `.github/workflows/`):**
1. Один раз выполните `firebase init hosting:github` в корне репозитория
   (эта команда сама создаст сервисный аккаунт и добавит секрет
   `FIREBASE_SERVICE_ACCOUNT_MLT_SITE` в настройки GitHub-репозитория) —
   либо создайте сервисный аккаунт вручную в Google Cloud Console
   (роль Firebase Hosting Admin) и добавьте его JSON-ключ как секрет
   репозитория с этим же именем.
2. После этого каждый пуш в `main` будет автоматически деплоить
   `index.html` на Firebase Hosting (workflow
   `.github/workflows/firebase-hosting-merge.yml`), а каждый Pull
   Request — создавать превью-ссылку (`firebase-hosting-pull-request.yml`).

### Firestore Security Rules

Деплой Hosting **не** публикует правила Firestore — это отдельный
одноразовый шаг:
- Консоль Firebase → Firestore → Rules → вставить содержимое
  `firestore.rules` → Publish, **или**
- `firebase deploy --only firestore:rules` (если вошли через CLI).

Без этого шага Firestore либо всё запрещает, либо (в тестовом режиме)
всё разрешает — сайт не заработает как задумано, пока правила не
опубликованы.

## Первый запуск после деплоя

1. Опубликовать `firestore.rules` (см. выше).
2. Открыть сайт, зарегистрироваться первым — этот аккаунт станет админом
   (первый когда-либо зарегистрированный пользователь автоматически
   получает роль admin — см. пояснение в конце `firestore.rules`).
3. В Админ-панели по желанию нажать «Импортировать демо-данные».
4. Дальше — обычная работа через сайт: команды, турниры, матчи, новости.
