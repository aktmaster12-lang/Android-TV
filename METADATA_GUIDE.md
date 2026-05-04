# Руководство по системе метаданных

## Обзор

Система метаданных позволяет автоматически загружать профессиональные описания приложений вместо отображения технических имен файлов.

## Архитектура

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   APK файлы     │────│  apps_metadata  │────│   UI/Interface  │
│   (физические)  │    │     .json       │    │   (отображение) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        │ 1. Сканирование       │ 2. Загрузка          │ 3. Отображение
        │    папки             │    метаданных         │    красивых имен
        ▼                       ▼                       ▼
   File System           GitHub/Local          User Interface
```

## Формат метаданных

### Основная структура

```json
{
  "metadata_version": "1.0",
  "last_updated": "2026-05-04",
  "apps": {
    "имя_файла.apk": { /* метаданные приложения */ }
  },
  "categories": {
    "category_id": { /* информация о категории */ }
  },
  "metadata_url": "URL для загрузки"
}
```

### Метаданные приложения

```json
{
  "name": "Красивое название приложения",
  "version": "1.0.0",
  "category": "media_player",
  "description": "Краткое описание (1-2 предложения)",
  "long_description": "Подробное описание с преимуществами",
  "features": [
    "Ключевая функция 1",
    "Ключевая функция 2",
    "Ключевая функция 3"
  ],
  "size": "10.5 MB",
  "developer": "Название разработчика",
  "icon_url": "https://example.com/icon.png",
  "screenshots": [
    "https://example.com/screen1.png",
    "https://example.com/screen2.png"
  ],
  "rating": 4.5,
  "downloads": "100K+",
  "requirements": [
    "Android TV 5.0+",
    "2GB RAM",
    "Стабильный интернет"
  ],
  "tags": ["тег1", "тег2", "тег3"]
}
```

### Категории

```json
{
  "media_player": {
    "name": "Медиа плееры",
    "description": "Приложения для воспроизведения видео и аудио",
    "icon": "🎬"
  }
}
```

## Интеграция

### JavaScript пример

```javascript
class AppMetadataManager {
  constructor() {
    this.metadataUrl = "https://raw.githubusercontent.com/aktmaster12-lang/Android-TV/main/apps_metadata.json";
    this.cache = null;
    this.lastUpdate = null;
  }

  async loadMetadata() {
    // Проверяем кэш
    if (this.cache && !this.isCacheExpired()) {
      return this.cache;
    }

    try {
      // Загружаем метаданные
      const response = await fetch(this.metadataUrl);
      const metadata = await response.json();
      
      // Обновляем кэш
      this.cache = metadata;
      this.lastUpdate = Date.now();
      
      return metadata;
    } catch (error) {
      console.warn("Не удалось загрузить метаданные, используем fallback");
      return this.getFallbackMetadata();
    }
  }

  getAppInfo(fileName) {
    if (!this.cache) return null;
    
    return this.cache.apps[fileName] || {
      name: fileName.replace('.apk', ''),
      category: 'unknown',
      description: 'Метаданные недоступны'
    };
  }

  getAppsByCategory(category) {
    if (!this.cache) return [];
    
    return Object.entries(this.cache.apps)
      .filter(([_, app]) => app.category === category)
      .map(([fileName, app]) => ({ fileName, ...app }));
  }

  getAllCategories() {
    if (!this.cache) return {};
    return this.cache.categories;
  }

  isCacheExpired() {
    const CACHE_DURATION = 3600000; // 1 час
    return Date.now() - this.lastUpdate > CACHE_DURATION;
  }

  getFallbackMetadata() {
    return {
      apps: {},
      categories: {
        unknown: { name: "Неизвестно", icon: "❓" }
      }
    };
  }
}

// Использование
const metadataManager = new AppMetadataManager();

// Загрузка метаданных
const metadata = await metadataManager.loadMetadata();

// Получение информации о приложении
const appInfo = metadataManager.getAppInfo("MX-Player-Pro-1.93.4-armv7.apk");

// Получение приложений по категории
const mediaPlayers = metadataManager.getAppsByCategory("media_player");
```

### Python пример

```python
import json
import requests
from datetime import datetime, timedelta
from typing import Dict, List, Optional

class AppMetadataManager:
    def __init__(self):
        self.metadata_url = "https://raw.githubusercontent.com/aktmaster12-lang/Android-TV/main/apps_metadata.json"
        self.cache = None
        self.last_update = None

    async def load_metadata(self) -> Dict:
        # Проверяем кэш
        if self.cache and not self._is_cache_expired():
            return self.cache

        try:
            response = requests.get(self.metadata_url)
            response.raise_for_status()
            
            metadata = response.json()
            
            # Обновляем кэш
            self.cache = metadata
            self.last_update = datetime.now()
            
            return metadata
        except Exception as e:
            print(f"Ошибка загрузки метаданных: {e}")
            return self._get_fallback_metadata()

    def get_app_info(self, file_name: str) -> Optional[Dict]:
        if not self.cache:
            return None
        
        return self.cache.get("apps", {}).get(file_name, {
            "name": file_name.replace(".apk", ""),
            "category": "unknown",
            "description": "Метаданные недоступны"
        })

    def get_apps_by_category(self, category: str) -> List[Dict]:
        if not self.cache:
            return []
        
        return [
            {"file_name": file_name, **app_data}
            for file_name, app_data in self.cache.get("apps", {}).items()
            if app_data.get("category") == category
        ]

    def _is_cache_expired(self) -> bool:
        if not self.last_update:
            return True
        
        cache_duration = timedelta(hours=1)
        return datetime.now() - self.last_update > cache_duration

    def _get_fallback_metadata(self) -> Dict:
        return {
            "apps": {},
            "categories": {
                "unknown": {"name": "Неизвестно", "icon": "❓"}
            }
        }
```

## Обновление метаданных

### Добавление нового приложения

1. Добавьте APK файл в папку
2. Обновите `apps_metadata.json`:

```json
{
  "apps": {
    "New-App-1.0.apk": {
      "name": "Новое приложение",
      "version": "1.0",
      "category": "media_player",
      "description": "Описание нового приложения",
      "features": ["Функция 1", "Функция 2"],
      "size": "15 MB",
      "rating": 4.0,
      "tags": ["new", "media"]
    }
  }
}
```

### Обновление существующего приложения

```json
{
  "apps": {
    "Existing-App-1.0.apk": {
      "version": "2.0",  // Обновленная версия
      "description": "Обновленное описание",
      "features": ["Новая функция", "Старая функция"]
    }
  }
}
```

## Валидация метаданных

### Обязательные поля

- `name` - Название приложения
- `version` - Версия
- `category` - Категория
- `description` - Краткое описание

### Опциональные поля

- `long_description` - Подробное описание
- `features` - Список функций
- `size` - Размер файла
- `developer` - Разработчик
- `rating` - Рейтинг
- `tags` - Теги

### Скрипт валидации

```javascript
function validateMetadata(metadata) {
  const errors = [];
  
  if (!metadata.apps) {
    errors.push("Отсутствует секция apps");
    return errors;
  }
  
  for (const [fileName, app] of Object.entries(metadata.apps)) {
    if (!app.name) errors.push(`${fileName}: отсутствует name`);
    if (!app.version) errors.push(`${fileName}: отсутствует version`);
    if (!app.category) errors.push(`${fileName}: отсутствует category`);
    if (!app.description) errors.push(`${fileName}: отсутствует description`);
  }
  
  return errors;
}
```

## Производительность

### Оптимизации

1. **Кэширование** - Локальное кэширование метаданных
2. **Ленивая загрузка** - Загрузка метаданных по требованию
3. **Сжатие** - Использование gzip для JSON
4. **CDN** - Размещение на CDN для быстрой загрузки

### Мониторинг

```javascript
// Метрики производительности
const metrics = {
  loadTime: 0,
  cacheHits: 0,
  cacheMisses: 0,
  errors: 0
};

// Логирование производительности
function logMetrics() {
  console.log("Метаданные:", {
    loadTime: `${metrics.loadTime}ms`,
    cacheHitRate: `${(metrics.cacheHits / (metrics.cacheHits + metrics.cacheMisses) * 100).toFixed(1)}%`,
    errors: metrics.errors
  });
}
```

## Безопасность

### Рекомендации

1. **Валидация** - Проверка структуры JSON
2. **Санитизация** - Очистка пользовательских данных
3. **HTTPS** - Использование защищенного соединения
4. **CORS** - Настройка CORS заголовков

### Пример валидации

```javascript
function sanitizeMetadata(metadata) {
  const sanitized = JSON.parse(JSON.stringify(metadata));
  
  // Ограничение длины строк
  for (const app of Object.values(sanitized.apps)) {
    if (app.name && app.name.length > 100) {
      app.name = app.name.substring(0, 100);
    }
    if (app.description && app.description.length > 500) {
      app.description = app.description.substring(0, 500);
    }
  }
  
  return sanitized;
}
```

Эта система обеспечивает гибкое и масштабируемое управление метаданными приложений с автоматическим обновлением и fallback механизмом.
