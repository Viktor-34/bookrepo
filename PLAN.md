# 📋 План разработки: Social Posts Saver на базе Karakeep (ранее Hoarder)

## 🎯 Цель проекта
Создать персональный сервис для сохранения постов из X.com и Facebook с:
- Превью изображений (включая множественные) - **Grid View обязательно!**
- Тегированием
- Браузерным расширением
- Импортом CSV
- Деплоем на **Beget VPS**

## ⚠️ Важные изменения:
- ✅ **MVP за 5 дней + Grid View** (приоритет!)
- ❌ **Mobile apps - НЕ нужны** (убираем из плана)
- ❌ **AI-теги - НЕ нужны** (отключаем полностью)
- ✅ **VPS: Beget** (вместо общих рекомендаций)
- 🔄 **Репозиторий:** karakeep-app/karakeep (переименовался с hoarder-app/hoarder)

---

## 📊 Анализ Hoarder

### ✅ Что уже есть:

#### Технологический стек:
- **Frontend:** Next.js 14 (App Router) + TypeScript + React
- **Backend:** tRPC + Next.js API Routes
- **Database:** PostgreSQL + Drizzle ORM
- **Search:** Meilisearch (полнотекстовый поиск)
- **Web Crawling:** Puppeteer (автоматическое извлечение метаданных)
- **AI:** OpenAI API / Ollama (локальные модели)
- **Storage:** Встроенное хранилище для медиа-файлов
- **Auth:** NextAuth.js
- **Deployment:** Docker + Docker Compose

#### Функционал из коробки:
- ✅ Браузерные расширения (Chrome, Firefox, Safari в TestFlight)
- ✅ Сохранение ссылок, заметок, изображений, PDF
- ✅ Автоматическое извлечение метаданных (title, description, images)
- ✅ Полнотекстовый поиск (Meilisearch)
- ✅ Архивация видео через yt-dlp
- ✅ Архивация полных страниц (Monolith)
- ✅ RSS feed интеграция
- ✅ Теги и списки (коллекции)
- ❌ AI-автотегирование - **ОТКЛЮЧАЕМ** (не нужно)
- ❌ OCR для изображений - можно отключить для упрощения
- ❌ Mobile apps - **НЕ используем** (не нужны)

### ❌ Что нужно добавить/изменить:

1. **CSV Import** - импорт списка ссылок ✅ ОБЯЗАТЕЛЬНО
2. **Улучшенный Grid View** - оптимизированное отображение множественных изображений ✅ **ОБЯЗАТЕЛЬНО!**
3. **Отключить AI** - убрать все AI-зависимости (OpenAI API, Ollama)
4. **Beget VPS deployment** - специфичная настройка для Beget хостинга

---

## 🗓️ ROADMAP UPDATED (MVP + Grid View за 5-6 дней)

### **Этап 1: Setup & Исследование (День 1)**

**Задачи:**
1. ✅ Клонировать репозиторий karakeep-app/karakeep (новое название!)
2. ✅ Изучить структуру проекта:
   - `/apps/web` - Next.js frontend
   - `/apps/workers` - Background workers
   - ~~`/apps/mobile`~~ - НЕ трогаем (не нужно)
   - `/packages` - Shared code
3. ✅ Запустить локально через Docker Compose
4. ✅ **ОТКЛЮЧИТЬ AI** - убрать зависимости от OpenAI/Ollama
5. ✅ Протестировать базовый функционал
6. ✅ Изучить браузерное расширение (`/apps/browser-extension`)

**Deliverables:**
- Работающий локальный инстанс Karakeep БЕЗ AI
- Понимание архитектуры
- Список файлов для модификации

---

### **Этап 2: Улучшенный Grid View (День 2-3) - ПРИОРИТЕТ!**

**Цель:** Оптимизировать отображение постов с множественными изображениями (как в Twitter - 2x2 grid для 4 изображений)

**Требования:**
- 1 изображение: full-width превью
- 2 изображения: 2 колонки
- 3 изображения: 2 больших + 1 маленький
- 4+ изображения: grid 2x2 + "показать еще N фото"

**Задачи:**

1. **Анализ текущей реализации:**
   - Как сейчас хранятся изображения в DB?
   - Компонент для отображения bookmarks
   - Layout система

2. **UI Компоненты:**
   - Создать `ImageGrid.tsx` компонент
   - Responsive layout (mobile/desktop)
   - Lightbox для полноэкранного просмотра
   - Lazy loading для производительности

3. **Стили:**
   - TailwindCSS классы для grid layouts
   - Aspect ratio сохранение
   - Hover эффекты

**Вдохновение:**
- Twitter/X.com (2x2 grid)
- Instagram (адаптивный grid)

**Файлы для изменения:**
```
/apps/web/components/dashboard/bookmarks/BookmarkCard.tsx
/apps/web/components/ui/ImageGrid.tsx              # New component
/apps/web/styles/                                   # CSS customization
```

**Тестирование:**
- Пост с 1 изображением
- Пост с 4 изображениями
- Пост с 10+ изображениями
- Mobile responsive

---

### **Этап 3: CSV Import (День 4)**

**Цель:** Добавить функционал импорта CSV файлов со списком ссылок

**Структура CSV:**
```csv
url,tags,notes
https://twitter.com/user/status/123,"tech,ai","Interesting thread about AI"
https://facebook.com/post/456,"travel,friends","Trip photos"
```

**Задачи:**

1. **Backend API:**
   - Создать tRPC endpoint `bookmark.importCSV`
   - Парсинг CSV файла (библиотека: `papaparse`)
   - Валидация URL
   - Batch создание bookmarks
   - Обработка ошибок (логирование failed URLs)

2. **Frontend UI:**
   - Добавить страницу/модал "Import from CSV"
   - File upload компонент
   - Progress bar для batch операций
   - Отображение результатов (successful/failed)
   - Опция: preview перед импортом

3. **Background Processing:**
   - Queued обработка (чтобы не перегружать систему)
   - Crawling метаданных для каждого URL
   - AI-тегирование (если включено)

**Файлы для изменения:**
```
/packages/trpc/routers/bookmarks.ts      # Backend endpoint
/apps/web/components/import/             # New: Import UI
/apps/web/app/dashboard/import/          # New: Import page
```

**Тестирование:**
- CSV с 5 ссылками
- CSV с 100 ссылками (performance test)
- Невалидные URL
- Дубликаты

---

### **Этап 4: Beget VPS Deployment (День 5)**

**Цель:** Настроить production-ready деплой на Beget VPS

**Особенности Beget:**
- Shared hosting или VPS варианты
- ISPmanager панель управления
- SSH доступ (на VPS тарифах)
- Docker support (нужно уточнить доступность)

**Задачи:**

1. **Beget VPS Setup:**
   - Подключение по SSH
   - Проверка наличия Docker
   - Если нет Docker - установка или альтернативный деплой

2. **Вариант A: Docker на Beget VPS**
   - Установить Docker + Docker Compose
   - Модифицировать `docker-compose.yml` для production
   - Environment variables (.env)
   - Persistent volumes

3. **Вариант B: Без Docker (если Beget не поддерживает)**
   - Node.js direct setup
   - PM2 process manager
   - PostgreSQL setup
   - Nginx из панели ISPmanager

4. **SSL и домен:**
   - Настройка домена через ISPmanager
   - Let's Encrypt SSL (встроен в ISPmanager)

5. **Environment Variables:**
   ```bash
   KARAKEEP_VERSION=release
   NEXTAUTH_SECRET=<generate strong secret>
   MEILI_MASTER_KEY=<generate strong key>
   NEXTAUTH_URL=https://yourdomain.com
   DATA_DIR=/home/user/karakeep/data
   # NO AI KEYS NEEDED
   DISABLE_INFERENCE=true
   ```

**Документация:**
- `BEGET_DEPLOYMENT.md` - специфичная инструкция для Beget
- Скрипты для автоматизации
- Troubleshooting guide

**Тестирование:**
- Проверка всех функций в production
- SSL сертификат
- Performance testing
- Backup/restore procedure

---

### **~~Этап 4: Social Media Optimization~~ - ОТЛОЖЕНО**

*Можно добавить позже, если понадобится специальная обработка Twitter/Facebook*

---

### **~~Этап 5: VPS Deployment~~ - ЗАМЕНЕНО НА BEGET**

*См. Этап 4*

---

### **Этап 5: Тестирование и Документация (День 6)**

**Цель:** Оптимизировать отображение постов с множественными изображениями (как в Twitter - 2x2 grid для 4 изображений)

**Требования:**
- 1 изображение: full-width превью
- 2 изображения: 2 колонки
- 3 изображения: 2 больших + 1 маленький
- 4+ изображения: grid 2x2 + "показать еще N фото"

**Задачи:**

1. **Анализ текущей реализации:**
   - Как сейчас хранятся изображения в DB?
   - Компонент для отображения bookmarks
   - Layout система

2. **UI Компоненты:**
   - Создать `ImageGrid.tsx` компонент
   - Responsive layout (mobile/desktop)
   - Lightbox для полноэкранного просмотра
   - Lazy loading для производительности

3. **Стили:**
   - TailwindCSS классы для grid layouts
   - Aspect ratio сохранение
   - Hover эффекты

**Вдохновение:**
- Twitter/X.com (2x2 grid)
- Instagram (адаптивный grid)
- Pinterest (masonry layout - опционально)

**Файлы для изменения:**
```
/apps/web/components/dashboard/bookmarks/BookmarkCard.tsx
/apps/web/components/ui/ImageGrid.tsx              # New component
/apps/web/styles/                                   # CSS customization
```

**Тестирование:**
- Пост с 1 изображением
- Пост с 4 изображениями
- Пост с 10+ изображениями
- Mobile responsive

---

### **Этап 4: Social Media Optimization (День 6-7)**

**Цель:** Специальная обработка постов из Twitter/Facebook для лучшего извлечения данных

**Задачи:**

1. **Улучшенный парсинг Twitter/X.com:**
   - Детект Twitter URL
   - Извлечение всех изображений из thread
   - Сохранение автора, даты, текста
   - Embedded tweet layout (опционально)

2. **Улучшенный парсинг Facebook:**
   - Детект Facebook URL
   - Извлечение изображений из альбомов
   - Обработка карусели

3. **Puppeteer Scripts:**
   - Скролл для загрузки всех изображений
   - Wait for lazy-loaded content
   - Обход cookie banners

4. **Metadata Enhancement:**
   - Добавить поле `source` (twitter/facebook/other)
   - Добавить `author` и `publishedDate`
   - Иконки для разных источников в UI

**Файлы для изменения:**
```
/packages/shared/utils/parsers/twitter.ts      # New
/packages/shared/utils/parsers/facebook.ts     # New
/apps/workers/crawl-worker.ts                  # Modify
/packages/db/schema.ts                          # Add new fields
```

**Тестирование:**
- Twitter пост с 1 изображением
- Twitter пост с 4 изображениями
- Twitter thread с несколькими постами
- Facebook пост с альбомом

---

### **Этап 5: Deployment на VPS (День 8-9)**

**Цель:** Настроить production-ready деплой на VPS

**Инфраструктура:**
```
VPS (Ubuntu 22.04)
├── Docker + Docker Compose
├── Nginx (reverse proxy)
├── Let's Encrypt (SSL)
└── Hoarder Stack
    ├── Next.js App
    ├── PostgreSQL
    ├── Meilisearch
    ├── Chrome (Puppeteer)
    └── Redis (optional)
```

**Задачи:**

1. **VPS Setup:**
   - Выбрать провайдера (DigitalOcean, Hetzner, Linode)
   - Ubuntu 22.04 setup
   - Установить Docker, Docker Compose, Nginx
   - Настроить firewall (UFW)

2. **Docker Configuration:**
   - Модифицировать `docker-compose.yml` для production
   - Environment variables (.env)
   - Persistent volumes для данных
   - Health checks
   - Auto-restart policies

3. **Nginx Setup:**
   - Reverse proxy на порт 3000
   - SSL через Certbot
   - Gzip compression
   - Rate limiting

4. **Environment Variables:**
   ```bash
   HOARDER_VERSION=release
   NEXTAUTH_SECRET=<generate strong secret>
   MEILI_MASTER_KEY=<generate strong key>
   NEXTAUTH_URL=https://yourdomain.com
   DATA_DIR=/var/hoarder/data
   OPENAI_API_KEY=<optional>
   ```

5. **Backup Strategy:**
   - Автоматический backup PostgreSQL
   - Backup images/uploads
   - Cron jobs

6. **Мониторинг:**
   - Docker logs
   - Disk space monitoring
   - Uptime monitoring (опционально: UptimeRobot)

**Документация:**
- `DEPLOYMENT.md` - пошаговая инструкция
- Скрипты для автоматизации
- Troubleshooting guide

**Тестирование:**
- Проверка всех функций в production
- SSL сертификат
- Performance testing
- Backup/restore procedure

---

### **Этап 6: Тестирование и Документация (День 10)**

**Задачи:**

1. **E2E Testing:**
   - Сохранение поста через расширение
   - CSV импорт 50 ссылок
   - Поиск по тегам и тексту
   - Mobile responsive
   - Performance (загрузка 1000+ bookmarks)

2. **Документация:**
   - `README.md` на русском
   - Инструкция по установке
   - Инструкция по использованию
   - Скриншоты/GIFs
   - FAQ

3. **Browser Extension:**
   - Тестирование в Chrome
   - Тестирование в Firefox
   - Настройка подключения к VPS

4. **Bug Fixes:**
   - Исправление найденных проблем
   - Code cleanup
   - Optimization

---

### **Этап 7 (Опционально): Локальная версия (День 11-12)**

**Цель:** Упрощенная версия для локального запуска без внешних зависимостей

**Задачи:**

1. **Упрощенный Stack:**
   - SQLite вместо PostgreSQL (опция)
   - Встроенный search вместо Meilisearch (опция)
   - Без AI (или только Ollama локально)

2. **One-click Installer:**
   - Bash скрипт для Linux/Mac
   - PowerShell скрипт для Windows
   - Portable версия (всё в одной папке)

3. **Docker Desktop Setup:**
   - Упрощенный docker-compose.yml
   - GUI инструкции

**Deliverables:**
- `docker-compose.local.yml`
- `install.sh` / `install.ps1`
- `LOCAL_SETUP.md`

---

## 📁 Структура проекта (после форка)

```
bookrepo/
├── .git/                           # Fork hoarder-app/hoarder
├── .github/workflows/              # CI/CD (keep from upstream)
├── apps/
│   ├── web/                        # Next.js app
│   │   ├── app/
│   │   │   ├── dashboard/
│   │   │   │   └── import/        # ✨ NEW: CSV Import page
│   │   │   └── ...
│   │   └── components/
│   │       ├── dashboard/
│   │       │   └── bookmarks/
│   │       │       └── BookmarkCard.tsx  # 🔧 MODIFY
│   │       ├── import/             # ✨ NEW: Import components
│   │       └── ui/
│   │           └── ImageGrid.tsx   # ✨ NEW: Grid component
│   ├── workers/                    # Background jobs
│   │   └── crawl-worker.ts         # 🔧 MODIFY
│   ├── browser-extension/          # Chrome/Firefox extension
│   └── mobile/                     # React Native apps
├── packages/
│   ├── trpc/
│   │   └── routers/
│   │       └── bookmarks.ts        # 🔧 MODIFY: add importCSV
│   ├── db/
│   │   └── schema.ts               # 🔧 MODIFY: add new fields
│   └── shared/
│       └── utils/
│           └── parsers/
│               ├── twitter.ts      # ✨ NEW
│               └── facebook.ts     # ✨ NEW
├── docker/
│   ├── docker-compose.yml          # Use for local dev
│   └── docker-compose.prod.yml     # 🔧 MODIFY for VPS
├── docs/
│   ├── DEPLOYMENT.md               # ✨ NEW: VPS guide
│   ├── LOCAL_SETUP.md              # ✨ NEW: Local guide
│   └── CSV_IMPORT.md               # ✨ NEW: Import guide
├── PLAN.md                         # This file
└── README.md                       # 🔧 MODIFY: Russian docs
```

---

## 🔧 Технические детали

### CSV Import Implementation

**Schema:**
```typescript
interface CSVRow {
  url: string;           // Required
  tags?: string;         // Comma-separated
  notes?: string;        // Optional description
  title?: string;        // Optional (will be fetched if missing)
}
```

**Backend Flow:**
```typescript
// packages/trpc/routers/bookmarks.ts
importCSV: protectedProcedure
  .input(z.object({
    file: z.string(), // base64 or file URL
  }))
  .mutation(async ({ input, ctx }) => {
    // 1. Parse CSV
    const rows = parseCSV(input.file);

    // 2. Validate URLs
    const validRows = rows.filter(r => isValidURL(r.url));

    // 3. Queue crawling jobs
    const jobs = validRows.map(row =>
      ctx.queue.add('crawl', {
        url: row.url,
        tags: parseTags(row.tags),
        notes: row.notes,
      })
    );

    // 4. Return summary
    return {
      total: rows.length,
      queued: jobs.length,
      failed: rows.length - jobs.length,
    };
  });
```

### Image Grid Layouts

**Responsive Grid:**
```tsx
// components/ui/ImageGrid.tsx
export function ImageGrid({ images }: { images: string[] }) {
  const count = images.length;

  if (count === 1) {
    return <SingleImage src={images[0]} />;
  }

  if (count === 2) {
    return <TwoColumnGrid images={images} />;
  }

  if (count === 3) {
    return <ThreeImageLayout images={images} />;
  }

  // 4+ images: 2x2 grid + "Show more"
  return <GridLayout images={images} maxVisible={4} />;
}
```

**CSS (Tailwind):**
```css
/* 2x2 Grid */
.grid-2x2 {
  @apply grid grid-cols-2 gap-1;
}

/* Aspect ratio preservation */
.image-container {
  @apply aspect-square overflow-hidden;
}
```

### Social Media Parsers

**Twitter Detection:**
```typescript
// utils/parsers/twitter.ts
export function isTwitterURL(url: string): boolean {
  return /^https?:\/\/(www\.)?(twitter\.com|x\.com)\//.test(url);
}

export async function parseTwitterPost(page: Page): Promise<ParsedPost> {
  // Wait for images to load
  await page.waitForSelector('[data-testid="tweetPhoto"]');

  // Extract all images
  const images = await page.$$eval(
    '[data-testid="tweetPhoto"] img',
    imgs => imgs.map(img => img.src)
  );

  // Extract text
  const text = await page.$eval(
    '[data-testid="tweetText"]',
    el => el.textContent
  );

  return { images, text, source: 'twitter' };
}
```

### VPS Deployment

**Nginx Config:**
```nginx
# /etc/nginx/sites-available/hoarder
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Docker Compose (Production):**
```yaml
# docker-compose.prod.yml
version: "3.8"
services:
  web:
    image: ghcr.io/hoarder-app/hoarder:release
    restart: always
    ports:
      - "3000:3000"
    volumes:
      - ./data:/data
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://...
    depends_on:
      - db
      - meilisearch

  db:
    image: postgres:16
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}

  meilisearch:
    image: getmeili/meilisearch:v1.5
    restart: always
    volumes:
      - meili_data:/meili_data

volumes:
  postgres_data:
  meili_data:
```

---

## 📊 Оценка времени (UPDATED)

| Этап | Задача | Время |
|------|--------|-------|
| 1 | Setup & Отключение AI | 1 день |
| 2 | **Grid View (ПРИОРИТЕТ!)** | 2-3 дня |
| 3 | CSV Import | 1 день |
| 4 | Beget VPS Deployment | 1-2 дня |
| 5 | Тестирование | 0.5 дня |
| **ИТОГО** | **MVP + Grid** | **5-6 дней** |

---

## 🎯 Утвержденный план (MVP + Grid за 5-6 дней)

**День 1:**
- Клонировать karakeep-app/karakeep
- Локальный setup + отключение AI
- Изучение кодовой базы

**День 2-3:**
- **Grid View для 4+ изображений (ПРИОРИТЕТ!)**
- Responsive layout
- Lightbox

**День 4:**
- CSV Import (базовая версия)

**День 5:**
- Beget VPS Deployment
- SSL + домен

**День 6:**
- Тестирование + документация

**Результат:** Работающий сервис с расширением + Grid View + CSV импорт + деплой на Beget VPS

**Что НЕ делаем:**
- ❌ Mobile apps
- ❌ AI-теги
- ❌ Social Media специальная обработка (пока)

---

## 🚀 Следующие шаги

1. ✅ **Создать этот PLAN.md** - Done!
2. ✅ **Обновить план с учетом требований** - Done!
3. ⏳ **Клонировать karakeep-app/karakeep** - В процессе
4. ⏳ **Запустить локально и отключить AI**
5. ⏳ **Начать разработку Grid View** (приоритет!)

---

## ✅ Уточненные требования (от пользователя)

1. **VPS провайдер:** ✅ Beget
2. **Домен:** ✅ Есть
3. **AI функции:** ❌ НЕ нужны, отключаем полностью
4. **Mobile apps:** ❌ НЕ нужны, не трогаем
5. **Приоритет:** ✅ MVP за 5 дней + **Grid View обязательно!**
6. **Репозиторий:** ✅ karakeep-app/karakeep (переименовался)

---

## 📝 Заметки

- Karakeep (ранее Hoarder) активно развивается
- Репозиторий переименовался: hoarder-app/hoarder → karakeep-app/karakeep
- Можно синхронизировать с upstream для получения обновлений
- Лицензия AGPL-3.0 (требует открытого кода при модификации)
- Сообщество активное, можно задавать вопросы в GitHub Discussions
- **AI полностью отключаем** - не нужен для нашего случая

---

**Автор:** Claude
**Дата:** 2025-11-07
**Версия:** 1.0
