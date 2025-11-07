# 🔍 Технический анализ Karakeep

**Дата:** 2025-11-07
**Цель:** Понять как реализовать Grid View для множественных изображений

---

## 📊 Текущая архитектура

### Структура компонентов:

```
BookmarkCard.tsx (роутер)
├── LinkCard.tsx         → для ссылок (LINK)
├── AssetCard.tsx        → для файлов (ASSET)
├── TextCard.tsx         → для заметок (TEXT)
└── UnknownCard.tsx      → для unknown типов

└─> BookmarkLayoutAdaptingCard.tsx (обертка с 4 layout'ами)
    ├── GridView         → сетка (текущий: 1 изображение)
    ├── MasonryView      → masonry grid
    ├── ListView         → список
    └── CompactView      → компактный
```

### Схема БД:

```sql
-- Основная таблица
bookmarks
├── id (PK)
├── title
├── type (LINK | TEXT | ASSET)
├── userId (FK)
└── ...

-- Для ссылок
bookmarkLinks
├── id (PK = FK bookmarks.id)
├── url
├── imageUrl          ← ОДНО изображение (текст)
├── title, description
├── crawlStatus
└── ...

-- Медиа-файлы (КЛЮЧЕВАЯ ТАБЛИЦА!)
assets
├── id (PK)
├── assetType         ← linkBannerImage | linkScreenshot | linkVideo | etc
├── bookmarkId (FK)   ← связь с bookmarks
├── userId (FK)
├── size, contentType
└── fileName
```

---

## 🔑 Ключевое открытие!

**Таблица `assets` УЖЕ поддерживает множественные изображения!**

- Один bookmark может иметь МНОГО assets
- `assets.assetType = 'linkBannerImage'` - это изображения из поста
- Связь: `assets.bookmarkId → bookmarks.id`

**Вывод:** НЕ нужно менять схему БД! Можно использовать существующую структуру.

---

## 🎯 Решение для Grid View

### Вариант 1: Использовать существующую схему (РЕКОМЕНДУЮ ✅)

**Плюсы:**
- Не нужны миграции БД
- Быстрая реализация
- Обратная совместимость

**План:**
1. Модифицировать парсер (crawler) чтобы создавать несколько assets с типом `linkBannerImage`
2. В `LinkCard.tsx` получать ВСЕ assets вместо одного `imageUrl`
3. Создать новый компонент `ImageGrid.tsx` для отображения 1/2/3/4+ изображений
4. `bookmarkLinks.imageUrl` оставить для первого изображения (обратная совместимость)

**Структура:**
```typescript
// Вместо:
bookmark.content.imageUrl  // Одно изображение

// Использовать:
bookmark.assets.filter(a => a.assetType === 'linkBannerImage')  // Массив изображений
```

---

### Вариант 2: Изменить схему (НЕ рекомендую ❌)

Добавить отдельную таблицу `bookmarkLinkImages` - сложнее, требует миграций.

---

## 📝 Текущая реализация изображений

### LinkCard.tsx (строки 42-88):

```typescript
function LinkImage({ bookmark }) {
  // Получает ОДНО изображение
  const imageDetails = getBookmarkLinkImageUrl(link);

  if (imageDetails) {
    return <Image src={imageDetails.url} />;
  }
}
```

**Проблема:** `getBookmarkLinkImageUrl()` возвращает только ОДНО изображение.

### BookmarkLayoutAdaptingCard.tsx (строки 164-220):

GridView рендерит изображение так:
```typescript
const img = image("grid", cn("h-56 min-h-56 w-full rounded-t-lg"));

// Отображается как:
<div className="h-56 w-full shrink-0">{img}</div>
```

**Проблема:** `image` - это функция которая возвращает ОДИН ReactNode.

---

## 🛠️ План реализации Grid View

### Шаг 1: Создать компонент ImageGrid.tsx

```typescript
// apps/web/components/ui/ImageGrid.tsx

interface ImageGridProps {
  images: string[];  // URLs массив
  className?: string;
}

export function ImageGrid({ images, className }: ImageGridProps) {
  const count = images.length;

  // 1 изображение: full-width
  if (count === 1) {
    return <SingleImage src={images[0]} className={className} />;
  }

  // 2 изображения: 2 колонки
  if (count === 2) {
    return <TwoColumnGrid images={images} className={className} />;
  }

  // 3 изображения: 2 больших + 1 маленький
  if (count === 3) {
    return <ThreeImageLayout images={images} className={className} />;
  }

  // 4+ изображения: 2x2 grid + "показать еще"
  return <FourPlusGrid images={images} className={className} />;
}
```

### Шаг 2: Модифицировать LinkCard.tsx

```typescript
// БЫЛО:
function LinkImage({ bookmark }) {
  const imageDetails = getBookmarkLinkImageUrl(link);
  return <Image src={imageDetails.url} />;
}

// СТАНЕТ:
function LinkImage({ bookmark }) {
  const images = bookmark.assets
    .filter(a => a.assetType === 'linkBannerImage')
    .map(a => getAssetUrl(a.id));

  if (images.length > 0) {
    return <ImageGrid images={images} />;
  }
  // fallback to imageUrl
  return <Image src={bookmark.content.imageUrl} />;
}
```

### Шаг 3: Модифицировать BookmarkLayoutAdaptingCard.tsx

Изменить GridView чтобы поддерживать динамическую высоту для grid'а:

```typescript
// БЫЛО:
<div className="h-56 w-full shrink-0">{img}</div>

// СТАНЕТ:
<div className={cn(
  "w-full shrink-0",
  imageCount === 1 ? "h-56" : "h-auto"  // Динамическая высота
)}>
  {img}
</div>
```

### Шаг 4: Модифицировать парсер (crawler)

Найти где происходит парсинг ссылок и модифицировать чтобы:
1. Извлекать ВСЕ изображения из HTML (не только первое)
2. Создавать несколько assets с типом `linkBannerImage`
3. Первое изображение записывать в `bookmarkLinks.imageUrl` (обратная совместимость)

---

## 📐 Layout спецификации

### 1 изображение:
```
┌───────────────────┐
│                   │
│                   │  ← 224px (h-56)
│                   │
└───────────────────┘
```

### 2 изображения:
```
┌─────────┬─────────┐
│         │         │
│    1    │    2    │  ← 224px
│         │         │
└─────────┴─────────┘
```

### 3 изображения:
```
┌─────────┬─────────┐
│         │         │
│    1    │    2    │  ← 112px
├─────────┴─────────┤
│                   │
│         3         │  ← 112px
└───────────────────┘
```

### 4+ изображений:
```
┌─────────┬─────────┐
│         │         │
│    1    │    2    │  ← 112px
├─────────┼─────────┤
│         │         │
│    3    │    4    │  ← 112px
└─────────┴─────────┘
   [ +N more ]          ← Button если > 4
```

---

## 🎨 TailwindCSS классы

```typescript
// 2 колонки
"grid grid-cols-2 gap-1"

// Aspect ratio
"aspect-square"  // 1:1
"aspect-video"   // 16:9

// Object fit
"object-cover"   // Crop to fit
"object-contain" // Fit without crop

// Высоты
"h-56"  // 224px (текущая)
"h-28"  // 112px (для grid 2x2)
"h-auto" // Автоматическая
```

---

## 🔍 Следующие шаги

1. ✅ **Анализ завершен** - понимаю структуру
2. ⏳ **Найти код парсера** (crawler/worker) чтобы понять как извлекаются изображения
3. ⏳ **Создать ImageGrid.tsx** компонент
4. ⏳ **Модифицировать LinkCard.tsx**
5. ⏳ **Протестировать с real data**

---

## 🎯 Оценка времени

| Задача | Время |
|--------|-------|
| Создать ImageGrid.tsx | 4-6 часов |
| Модифицировать LinkCard.tsx | 2 часа |
| Модифицировать BookmarkLayoutAdaptingCard.tsx | 1 час |
| Модифицировать парсер (crawler) | 4-6 часов |
| Тестирование + fixes | 3-4 часа |
| **ИТОГО** | **14-19 часов (~2 дня)** |

✅ **Укладывается в план: День 2-3 для Grid View!**

---

## 📚 Файлы для модификации

```
✏️ Создать новые:
- apps/web/components/ui/ImageGrid.tsx

🔧 Модифицировать:
- apps/web/components/dashboard/bookmarks/LinkCard.tsx
- apps/web/components/dashboard/bookmarks/BookmarkLayoutAdaptingCard.tsx
- apps/workers/crawl-worker.ts (или где парсер)
- packages/shared/utils/bookmarkUtils.ts (getBookmarkLinkImageUrl)

📖 Изучить:
- apps/workers/ - найти код парсера
- packages/shared/utils/ - утилиты для работы с изображениями
```

---

**Статус:** Анализ завершен ✅
**Следующий шаг:** Найти код парсера чтобы понять где извлекаются изображения
