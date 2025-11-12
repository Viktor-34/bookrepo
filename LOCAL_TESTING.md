# 🚀 Инструкция по локальному запуску Karakeep

**Дата:** 2025-11-07
**Цель:** Протестировать Grid View локально

---

## ⚠️ Важно

Docker не доступен в текущей среде разработки. Нужно запускать на **локальной машине**.

---

## 📋 Предварительные требования

### Установить:
- **Docker Desktop** (Windows/Mac) или **Docker Engine** (Linux)
- **Git** для клонирования репозитория

### Проверить установку:
```bash
docker --version
# Docker version 24.0.0 или выше

docker compose version
# Docker Compose version v2.20.0 или выше
```

---

## 🔧 Шаг 1: Клонировать репозиторий

```bash
# Клонировать модифицированный форк
git clone https://github.com/Viktor-34/bookrepo.git
cd bookrepo/karakeep-source
```

**Или** если репозиторий уже есть:
```bash
cd bookrepo/karakeep-source
git pull origin claude/hello-world-011CUt3AoGibZ2xpniCveDnY
```

---

## 🔧 Шаг 2: Настроить окружение

### Создать `.env` файл:

```bash
cd docker
cp .env.sample .env
```

### Отредактировать `.env`:

```bash
# Открыть в редакторе
nano .env
# или
code .env
```

**Содержимое `.env`:**
```env
# Karakeep local configuration
DATA_DIR=/data
MEILI_ADDR=http://meilisearch:7700
NEXTAUTH_URL=http://localhost:3000

# Сгенерировать секреты:
NEXTAUTH_SECRET=<openssl rand -base64 36>
MEILI_MASTER_KEY=<openssl rand -base64 36 | tr -dc 'A-Za-z0-9'>

# Отключить AI (по требованиям)
DISABLE_INFERENCE=true
```

**Сгенерировать секреты:**
```bash
# NEXTAUTH_SECRET
openssl rand -base64 36

# MEILI_MASTER_KEY
openssl rand -base64 36 | tr -dc 'A-Za-z0-9'
```

---

## 🚀 Шаг 3: Запустить Docker Compose

```bash
# Находимся в директории bookrepo/karakeep-source/docker
docker compose up -d
```

**Или** (старая версия Docker):
```bash
docker-compose up -d
```

---

## ⏳ Шаг 4: Дождаться запуска

### Проверить статус:
```bash
docker compose ps
```

**Ожидаемый вывод:**
```
NAME                    IMAGE                       STATUS
karakeep-web            ghcr.io/karakeep-app/...   Up 30 seconds
karakeep-meilisearch    getmeili/meilisearch       Up 30 seconds
karakeep-chrome         gcr.io/zenika-hub/...      Up 30 seconds
```

### Проверить логи:
```bash
# Все сервисы
docker compose logs -f

# Только web
docker compose logs -f web
```

**Дождаться строки:**
```
[web] ✓ Ready in 5.2s
[web] ○ Local:   http://localhost:3000
```

---

## 🌐 Шаг 5: Открыть веб-интерфейс

### Открыть браузер:
```
http://localhost:3000
```

### Первый запуск:
1. **Создать аккаунт:**
   - Email: `test@example.com`
   - Password: `password123`
   - Name: `Test User`

2. **Войти в систему**

---

## 🧪 Шаг 6: Протестировать Grid View

### Тест 1: Сохранить ссылку с множественными изображениями

#### Через веб-интерфейс:
1. Нажать **"Add Bookmark"**
2. Вставить URL:
   ```
   https://example.com/page-with-images
   ```
3. Нажать **"Save"**
4. Дождаться crawling (spinner исчезнет)
5. **Проверить Grid View!**

#### Примеры URL для тестирования:

**Twitter/X.com посты с изображениями:**
```
# Пост с 4 изображениями (если доступен)
https://twitter.com/NASA/status/...

# Статья с множественными изображениями
https://www.theverge.com/...
```

**Любая статья с картинками:**
```
https://www.bbc.com/news/...
https://www.techcrunch.com/...
```

---

### Тест 2: Проверить Browser Extension

#### Установить расширение:

**Chrome:**
1. Открыть `chrome://extensions/`
2. Включить **Developer mode**
3. Нажать **Load unpacked**
4. Выбрать: `bookrepo/karakeep-source/apps/browser-extension/dist`

**Firefox:**
1. Открыть `about:debugging#/runtime/this-firefox`
2. Нажать **Load Temporary Add-on**
3. Выбрать: `bookrepo/karakeep-source/apps/browser-extension/manifest.json`

#### Настроить расширение:
1. Кликнуть на иконку Karakeep
2. Ввести:
   - **Server URL:** `http://localhost:3000`
   - **API Key:** (создать в Settings → API Keys)

#### Тестировать:
1. Открыть любую страницу с изображениями
2. Кликнуть иконку Karakeep → **Save**
3. Открыть `http://localhost:3000`
4. **Проверить Grid View!**

---

## 🔍 Шаг 7: Проверить что работает

### ✅ Чеклист тестирования:

#### Grid View:
- [ ] 1 изображение: full-width (224px)
- [ ] 2 изображения: две колонки
- [ ] 3 изображения: 2 сверху + 1 снизу
- [ ] 4+ изображения: 2x2 grid + кнопка "+N more"
- [ ] Клик на изображение → lightbox (полноэкранный просмотр)
- [ ] Кнопка "+N more" → показывает остальные изображения

#### Responsive:
- [ ] Desktop (1440px) - Grid отображается корректно
- [ ] Tablet (768px) - Grid адаптируется
- [ ] Mobile (375px) - Grid отображается в 1-2 колонки

#### Backend:
- [ ] Логи показывают `extractAllImages()`
- [ ] Логи показывают `Successfully downloaded N images`
- [ ] Логи показывают `Saved N banner images to database`

---

## 📊 Проверить БД (опционально)

### Подключиться к PostgreSQL:
```bash
docker compose exec db psql -U karakeep
```

### Проверить assets:
```sql
-- Показать все banner images для bookmark
SELECT a.id, a.assetType, a.size
FROM assets a
WHERE a.bookmarkId = '<bookmark-id>'
AND a.assetType = 'linkBannerImage';

-- Должно вернуть N строк (по количеству изображений)
```

---

## 📝 Проверить логи

### Web сервис:
```bash
docker compose logs web | grep -i "extract"
docker compose logs web | grep -i "downloaded.*images"
```

**Ожидаемый вывод:**
```
[Crawler][123] Will attempt to extract all images from page ...
[Crawler][123] Found 4 images in the page.
[Crawler][123] Successfully downloaded 4 images
[Crawler][123] Saved 4 banner images to database
```

---

## 🛑 Остановить сервисы

```bash
# Остановить, сохранив данные
docker compose stop

# Остановить и удалить контейнеры
docker compose down

# Удалить данные тоже (careful!)
docker compose down -v
```

---

## 🐛 Troubleshooting

### Проблема: Порт 3000 занят
```bash
# Найти процесс
lsof -i :3000

# Убить процесс
kill -9 <PID>

# Или изменить порт в docker-compose.yml:
ports:
  - "3001:3000"  # Использовать 3001 вместо 3000
```

### Проблема: Docker не запускается
```bash
# Проверить Docker daemon
docker info

# Restart Docker Desktop (Mac/Windows)
# или
sudo systemctl restart docker  # Linux
```

### Проблема: Контейнеры не видят друг друга
```bash
# Проверить сеть
docker network ls
docker network inspect docker_default

# Пересоздать
docker compose down
docker compose up -d
```

### Проблема: Crawling не работает
```bash
# Проверить логи crawler worker
docker compose logs web | grep Crawler

# Проверить chrome контейнер
docker compose logs chrome

# Restart chrome
docker compose restart chrome
```

### Проблема: Images не загружаются
```bash
# Проверить storage quota
# В Settings → Check storage usage

# Проверить permissions
docker compose exec web ls -la /data

# Проверить логи
docker compose logs web | grep "downloadAndStoreImage"
```

---

## 🎯 Что тестировать в Grid View

### Сценарий 1: Twitter пост с 4 изображениями
1. Найти Twitter пост с 4 картинками
2. Сохранить через расширение
3. **Ожидается:** 2x2 grid отображение

### Сценарий 2: Статья с 10+ изображениями
1. Найти статью с множеством картинок (например, gallery)
2. Сохранить URL
3. **Ожидается:** 2x2 grid + кнопка "+6 more"
4. Кликнуть "+6 more"
5. **Ожидается:** Показываются остальные изображения

### Сценарий 3: Lightbox
1. Открыть bookmark с изображениями
2. Кликнуть на любое изображение
3. **Ожидается:** Полноэкранный просмотр (Dialog)
4. Закрыть (ESC или крестик)

### Сценарий 4: Mobile responsive
1. Открыть DevTools (F12)
2. Toggle device toolbar (Ctrl+Shift+M)
3. Выбрать iPhone 12
4. **Ожидается:** Grid адаптируется под узкий экран

---

## 📞 Следующие шаги после тестирования

### Если все работает ✅:
1. Закоммитить финальные изменения
2. Создать pull request
3. Перейти к CSV Import

### Если есть баги 🐛:
1. Записать скриншоты
2. Проверить browser console (F12)
3. Проверить логи Docker
4. Сообщить о проблемах

---

## 📚 Полезные команды

```bash
# Проверить версию Karakeep
docker compose exec web cat package.json | grep version

# Проверить все контейнеры
docker compose ps -a

# Войти в контейнер
docker compose exec web sh

# Перезапустить один сервис
docker compose restart web

# Посмотреть использование ресурсов
docker stats

# Экспортировать логи
docker compose logs > logs.txt
```

---

## 🎉 Успех выглядит так:

1. ✅ `http://localhost:3000` открывается
2. ✅ Можно создать аккаунт и войти
3. ✅ Можно сохранить bookmark
4. ✅ Bookmark отображается с **Grid View** (если 2+ изображения)
5. ✅ Lightbox работает при клике
6. ✅ Кнопка "+N more" работает

**Если все пункты ✅ - Grid View работает отлично!** 🎊

---

**Автор:** Claude
**Дата:** 2025-11-07
