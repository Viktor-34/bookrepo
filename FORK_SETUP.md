# 🔧 Настройка форка Karakeep

## Ситуация

Grid View полностью реализован в `/home/user/bookrepo/karakeep-source/`, но не может быть запушен в форк из-за отсутствия GitHub credentials в среде разработки.

## ✅ Решение: 2 варианта

### Вариант A: Использовать существующий karakeep-source (БЫСТРЫЙ)

Ваш `karakeep-source` уже содержит все изменения Grid View! Просто запустите локально:

```bash
cd /home/user/bookrepo/karakeep-source
git remote -v  # Уже настроен на Viktor-34/karakeep

# Remote уже обновлён на ваш форк
# Коммиты уже готовы:
# - 76cfd207: feat: Extract ALL images from pages for Grid View
# - 80415eea: feat: Add ImageGrid component for multiple images support

# Запустить Docker для тестирования
cd docker
docker compose up -d
```

**На локальной машине, если нужен push:**
```bash
# Скопируйте karakeep-source на локальную машину
# Затем:
cd karakeep-source
git push -u origin claude/hello-world-011CUt3AoGibZ2xpniCveDnY
```

---

### Вариант B: Клонировать форк заново (ЧИСТЫЙ)

1. **На вашей локальной машине:**

```bash
# Клонировать свой форк
git clone https://github.com/Viktor-34/karakeep.git
cd karakeep

# Создать ветку для Grid View
git checkout -b claude/hello-world-011CUt3AoGibZ2xpniCveDnY

# Применить патчи из bookrepo
git am /path/to/bookrepo/0001-feat-Add-ImageGrid-component-for-multiple-images-sup.patch
git am /path/to/bookrepo/0002-feat-Extract-ALL-images-from-pages-for-Grid-View.patch

# Запушить в форк
git push -u origin claude/hello-world-011CUt3AoGibZ2xpniCveDnY
```

2. **Тестировать:**

```bash
cd docker
cp .env.sample .env
# Отредактировать .env (см. LOCAL_TESTING.md)
docker compose up -d
```

---

## 📦 Что уже готово

### Файлы с патчами (в bookrepo):
- ✅ `0001-feat-Add-ImageGrid-component-for-multiple-images-sup.patch`
- ✅ `0002-feat-Extract-ALL-images-from-pages-for-Grid-View.patch`

### Изменённые файлы (в karakeep-source):

**Frontend:**
- ✅ `apps/web/components/ui/ImageGrid.tsx` (новый)
- ✅ `apps/web/components/dashboard/bookmarks/LinkCard.tsx`
- ✅ `packages/shared/utils/bookmarkUtils.ts`

**Backend:**
- ✅ `apps/workers/workers/crawlerWorker.ts`

**Конфигурация:**
- ✅ `docker/.env` (готовый с секретами)

---

## 🚀 Рекомендация

**Для немедленного тестирования:**
Используйте **Вариант A** - просто запустите Docker в существующем `karakeep-source/`

**Для продакшена и деплоя:**
Используйте **Вариант B** - клонируйте форк чисто и примените патчи

---

## 🔗 Полезные ссылки

- Ваш форк: https://github.com/Viktor-34/karakeep
- Родительский репозиторий: https://github.com/Viktor-34/bookrepo
- Upstream: https://github.com/karakeep-app/karakeep

---

**Создано:** 2025-11-07
