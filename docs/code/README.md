# Полный код проекта BookShop

Данная директория содержит полный код проекта, разбитый по категориям.

## Структура

### Backend
- **[models_code.md](models_code.md)** - Модели базы данных (Catalog, Orders, Users, Reviews, Analytics)
- **[serializers_code.md](serializers_code.md)** - Сериализаторы DRF для всех приложений
- **[config_code.md](config_code.md)** - Конфигурация, декораторы, URL маршруты
- **[core_code.md](core_code.md)** - Системные компоненты (middleware, audit logs, reviews)

### Frontend
- **[static_code.md](static_code.md)** - Статические файлы (CSS, JavaScript)

## Таблица всех файлов проекта

| № | Наименование файла | Описание | Строк | КБ |
|---|-------------------|----------|-------|-----|
| 1 | backend/apps/web/admin_views.py | Административные представления | 582 | 21.82 |
| 2 | backend/apps/web/views.py | Основные веб-представления | 329 | 14.83 |
| 3 | backend/apps/web/manager_views.py | Представления менеджера | 255 | 9.75 |
| 4 | backend/apps/core/decorators.py | Декораторы доступа | 118 | 5.8 |
| 5 | backend/apps/core/roles.py | Система ролей | 121 | 5.13 |
| 6 | backend/apps/analytics/models.py | Модели аналитики | 195 | 8.84 |
| 7 | backend/apps/catalog/models.py | Модели каталога | 54 | 2.47 |
| 8 | backend/apps/orders/models.py | Модели заказов | 61 | 3.12 |
| 9 | backend/apps/users/models.py | Модели пользователей | 55 | 2.54 |
| 10 | backend/settings/base.py | Базовые настройки | 112 | 3.34 |

**Всего:** 53 файла проекта

## Основные компоненты

### Backend
- Django REST Framework для API
- SQLite/PostgreSQL для БД
- Redis + Celery для фоновых задач
- JWT аутентификация
- DRF-YASG для документации API

### Frontend
- HTML/CSS/JavaScript
- Django Templates
- Адаптивный дизайн
- Темная/светлая тема

### Database
- 17+ таблиц
- 5 VIEW для аналитики
- 5 хранимых процедур
- 5 триггеров
- Нормализация 3НФ

