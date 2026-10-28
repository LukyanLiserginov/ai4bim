# AI4BIM — лендинг

Одностраничный статический лендинг: один `index.html` без сборки и внешних зависимостей
(CSS/JS/SVG встроены). Деплой — GitHub Pages через GitHub Actions.
Заявки с форм уходят через GitHub Actions, а письмо доставляется на **ai4bim@ro.ru**
по вашему SMTP-серверу — без сторонних сервисов форм.

## Структура

- `index.html` — вся страница (CSS, JS, inline-SVG).
- `.github/workflows/deploy.yml` — автодеплой на GitHub Pages при пуше в `main`.
- `.github/workflows/form-mailer.yml` — приём заявок (`workflow_dispatch`) и отправка письма по SMTP.
- `README.md` — этот файл.

## Публикация на GitHub Pages

1. Создайте репозиторий `LukyanLiserginov/ai4bim` и опубликуйте файлы:
   ```bash
   git init
   git add .
   git commit -m "AI4BIM landing"
   git remote add origin https://github.com/LukyanLiserginov/ai4bim.git
   git push -u origin main
   ```
2. В репозитории: **Settings → Pages** → Source: **GitHub Actions**.
   Workflow `deploy.yml` запустится автоматически после первого пуша;
   статус деплоя виден во вкладке **Actions**.
3. Через 1–2 минуты сайт будет доступен по адресу
   `https://lukyanliserginov.github.io/ai4bim`.

Каждый последующий пуш в `main` автоматически перепубликует страницу.

### Свой домен (опционально)

1. Создайте DNS-запись `CNAME`: `www.ваш-домен` → `lukyanliserginov.github.io`
   (для apex-домена — A-записи GitHub).
2. **Settings → Pages → Custom domain** — укажите домен.
3. HTTPS-сертификат выпустится автоматически.

## Обработка входящих запросов (формы)

Статический хостинг не принимает POST — данные из форм уходят напрямую из браузера
в GitHub API (`workflow_dispatch`), а workflow `form-mailer.yml` доставляет письмо
на **ai4bim@ro.ru** через ваш SMTP-сервер. В цепочке нет сторонних сервисов.

Схема:

```
Браузер (форма)
  → POST https://api.github.com/repos/LukyanLiserginov/ai4bim/dispatches
    (fine-grained токен, права только Workflow на этот репозиторий)
  → workflow form-mailer.yml (github.event.client_payload)
  → SMTP (ваш сервер) → ai4bim@ro.ru
```

### Настройка (один раз)

1. **Fine-grained токен**: GitHub → **Settings → Developer settings →
   Fine-grained tokens → Generate new token**:
   - доступ: только репозиторий `LukyanLiserginov/ai4bim`;
   - разрешение: **Workflow** (read и write).

   Вставьте токен в `index.html`:
   ```js
   const DISPATCH_TOKEN = '...'; // ← ваш токен
   ```

2. **Секреты репозитория** (Settings → Secrets and variables → Actions):

   | Секрет | Что вписать |
   |---|---|
   | `SMTP_HOST` | SMTP-сервер ящика ai4bim@ro.ru (например `smtp.beget.com`) |
   | `SMTP_PORT` | `587` (STARTTLS) или `465` (TLS) |
   | `SMTP_USER` | `ai4bim@ro.ru` |
   | `SMTP_PASS` | пароль ящика (или app-пароль) |
   | `SMTP_FROM` | отправитель, обычно `ai4bim@ro.ru` |

3. Push в `main` → деплой → отправьте тестовую заявку: письмо придёт на ai4bim@ro.ru.

### Что отправляют формы

- **Контактная форма** (`#contact`): имя, компания, email, телефон, сообщение.
  Есть honeypot-поле `_gotcha` против спама (не отправляется).
- **Формы лида** (inline-блок и модальное окно): только email; после успеха
  показывают ссылку на GitHub-репозиторий плагина.

## Константы (JS в конце `index.html`)

| Константа | Значение |
|---|---|
| `GITHUB_REPO` | `https://github.com/LukyanLiserginov/XbimStudio` — репозиторий плагина |
| `LANDING_REPO` | `LukyanLiserginov/ai4bim` — приём заявок через GitHub Actions |
| `DISPATCH_TOKEN` | fine-grained токен — **ЗАМЕНИТЕ перед запуском** |

## Примечания

- Токен виден в исходнике страницы. Fine-grained токен с правом только **Workflow**
  на один репозиторий ограничивает риск: им можно лишь отправлять тестовые
  заявки, но не трогать код или секреты.
- Если SMTP-доступа к ящику ai4bim@ro.ru нет — скажите, какой хостинг ведёт
  домен `ro.ru`, помогу подобрать параметры; как запасной вариант —
  Resend/SendGrid (но это зарубежные сервисы).
