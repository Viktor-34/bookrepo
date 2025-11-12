# 🎯 MVP Requirements - Утвержденные требования

**Дата:** 2025-11-07
**Проект:** Social Posts Saver на базе Karakeep

---

## ✅ Что делаем

### 1. **Grid View для множественных изображений** (ПРИОРИТЕТ!)
- 1 изображение: full-width
- 2 изображения: 2 колонки
- 3 изображения: 2 больших + 1 маленький
- 4+ изображения: grid 2x2 + "показать еще N"
- Responsive (mobile + desktop)
- Lightbox для просмотра

### 2. **CSV Import**
Формат:
```csv
url,tags,notes
https://twitter.com/user/status/123,"tech,ai","Interesting thread"
https://facebook.com/post/456,"travel","Trip photos"
```

### 3. **Beget VPS Deployment**
- Docker или прямой Node.js setup
- SSL через Let's Encrypt
- Домен уже есть
- ISPmanager панель

### 4. **Браузерное расширение**
- Уже есть в Karakeep (Chrome + Firefox)
- Только настроить подключение к VPS

---

## ❌ Что НЕ делаем

- ❌ **Mobile apps** (iOS/Android) - не трогаем
- ❌ **AI-теги** - полностью отключаем (OpenAI, Ollama)
- ❌ **OCR** - не нужен
- ❌ **Social Media специальная обработка** - отложено

---

## 🔧 Технологии

**Из Karakeep:**
- Frontend: Next.js 14 + TypeScript + React
- Backend: tRPC + Next.js API
- Database: PostgreSQL + Drizzle ORM
- Search: Meilisearch
- Auth: NextAuth.js
- Deploy: Docker Compose

**Без AI:**
- NO OpenAI API
- NO Ollama
- NO автотегирование

---

## 📅 Timeline

| День | Задача |
|------|--------|
| 1 | Setup + отключение AI |
| 2-3 | **Grid View** (приоритет!) |
| 4 | CSV Import |
| 5 | Beget VPS deployment |
| 6 | Тестирование |

**ИТОГО:** 5-6 дней

---

## 🎯 Успех = Done when:

1. ✅ Локально работает Karakeep БЕЗ AI
2. ✅ Grid View отображает 4+ изображения как в Twitter
3. ✅ CSV импорт загружает список ссылок
4. ✅ Задеплоено на Beget VPS с SSL
5. ✅ Расширение подключено к VPS
6. ✅ Можно сохранять посты из Twitter/Facebook через расширение

---

## 🔗 Ссылки

- **Репозиторий:** https://github.com/karakeep-app/karakeep
- **Документация:** https://docs.karakeep.app
- **Chrome Extension:** https://chromewebstore.google.com/detail/karakeep
- **VPS:** Beget
- **Домен:** Есть у пользователя

---

**Статус:** В разработке
**Следующий шаг:** Клонировать репозиторий
