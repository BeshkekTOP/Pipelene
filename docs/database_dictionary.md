# Словарь базы данных проекта BookShop

## Таблица: auth_user (Пользователи Django)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор пользователя с автоинкрементом |
| | username | VARCHAR(150) | NOT NULL | Логин пользователя (уникальный) |
| | password | VARCHAR(128) | NOT NULL | Хэш пароля пользователя |
| | email | VARCHAR(254) | UNIQUE | Email пользователя |
| | first_name | VARCHAR(150) | NULL | Имя пользователя |
| | last_name | VARCHAR(150) | NULL | Фамилия пользователя |
| | is_staff | BOOLEAN | NOT NULL | Права администратора Django |
| | is_active | BOOLEAN | NOT NULL | Активность аккаунта |
| | date_joined | TIMESTAMP | NOT NULL | Дата регистрации |

## Таблица: users_profile (Профили пользователей)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор профиля |
| FK | user_id | INT | NOT NULL | Ссылка на пользователя (OneToOne) |
| | role | VARCHAR(20) | NOT NULL | Роль пользователя (guest, buyer, manager, admin) |
| | is_blocked | BOOLEAN | NOT NULL | Заблокирован ли пользователь |
| | blocked_reason | TEXT | NULL | Причина блокировки |
| | phone | VARCHAR(20) | NULL | Телефон пользователя |
| | address | TEXT | NULL | Адрес доставки |
| | city | VARCHAR(100) | NULL | Город пользователя |
| | postal_code | VARCHAR(20) | NULL | Почтовый индекс |
| | date_of_birth | DATE | NULL | Дата рождения |
| | avatar | VARCHAR(100) | NULL | Путь к аватару |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |
| | updated_at | TIMESTAMP | NOT NULL | Дата обновления |

## Таблица: catalog_category (Категории книг)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор категории |
| | name | VARCHAR(120) | NOT NULL UNIQUE | Наименование категории |
| | slug | VARCHAR(140) | NOT NULL UNIQUE | URL-слаг категории |

## Таблица: catalog_author (Авторы)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор автора |
| | first_name | VARCHAR(100) | NOT NULL | Имя автора |
| | last_name | VARCHAR(100) | NOT NULL | Фамилия автора |

## Таблица: catalog_book (Книги)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор книги |
| | title | VARCHAR(255) | NOT NULL | Название книги |
| | isbn | VARCHAR(20) | NOT NULL UNIQUE | ISBN книги |
| | description | TEXT | NULL | Описание книги |
| FK | category_id | INT | NOT NULL | Категория книги |
| | price | DECIMAL(10,2) | NOT NULL | Цена книги |
| | rating | DECIMAL(3,2) | NOT NULL | Рейтинг книги (по умолчанию 0) |
| | pages | INT | NULL | Количество страниц |
| | publication_date | DATE | NULL | Дата публикации |
| | cover_image | VARCHAR(100) | NULL | Путь к обложке |
| | is_active | BOOLEAN | NOT NULL | Активна ли книга (по умолчанию True) |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |
| | updated_at | TIMESTAMP | NOT NULL | Дата обновления |

## Таблица: catalog_bookauthors (Книги-Авторы)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| FK | book_id | INT | NOT NULL | Книга |
| FK | author_id | INT | NOT NULL | Автор |

## Таблица: catalog_inventory (Остатки)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| FK | book_id | INT | NOT NULL UNIQUE | Книга (OneToOne) |
| | stock | INT | NOT NULL | Общий запас |
| | reserved | INT | NOT NULL | Зарезервировано |

## Таблица: orders_cart (Корзины)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор корзины |
| FK | user_id | INT | NOT NULL | Пользователь |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |

## Таблица: orders_cartitem (Позиции корзины)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| FK | cart_id | INT | NOT NULL | Корзина |
| FK | book_id | INT | NOT NULL | Книга |
| | quantity | INT | NOT NULL | Количество |

## Таблица: orders_order (Заказы)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор заказа |
| FK | user_id | INT | NOT NULL | Пользователь |
| | status | VARCHAR(20) | NOT NULL | Статус (processing, shipped, delivered, cancelled) |
| | total_amount | DECIMAL(12,2) | NOT NULL | Итоговая сумма заказа |
| | shipping_address | TEXT | NOT NULL | Адрес доставки |
| | shipping_city | VARCHAR(100) | NOT NULL | Город доставки |
| | shipping_postal_code | VARCHAR(20) | NULL | Почтовый индекс |
| | notes | TEXT | NULL | Примечания к заказу |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |
| | updated_at | TIMESTAMP | NOT NULL | Дата обновления |

## Таблица: orders_orderitem (Позиции заказа)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| FK | order_id | INT | NOT NULL | Заказ |
| FK | book_id | INT | NOT NULL | Книга |
| | price | DECIMAL(10,2) | NOT NULL | Цена на момент заказа |
| | quantity | INT | NOT NULL | Количество |

## Таблица: reviews_review (Отзывы)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор отзыва |
| FK | user_id | INT | NOT NULL | Автор отзыва |
| FK | book_id | INT | NOT NULL | Книга |
| | rating | SMALLINT | NOT NULL | Рейтинг (1-5) |
| | text | TEXT | NULL | Текст отзыва |
| | is_moderated | BOOLEAN | NOT NULL | Прошел ли модерацию |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |

## Таблица: core_auditlog (Логи аудита)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор записи |
| | action | VARCHAR(100) | NOT NULL | Действие (created, updated, deleted) |
| FK | actor_id | INT | NULL | Пользователь, выполнивший действие |
| FK | content_type_id | INT | NULL | Тип объекта |
| | object_id | INT | NULL | ID объекта |
| | old_data | JSON | NULL | Данные до изменения |
| | new_data | JSON | NULL | Данные после изменения |
| | path | VARCHAR(255) | NULL | Путь запроса |
| | method | VARCHAR(10) | NULL | Метод HTTP |
| | ip_address | INET | NULL | IP адрес |
| | user_agent | TEXT | NULL | User-Agent |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |

## Таблица: analytics_salesstats (Статистика продаж)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| | date | DATE | NOT NULL UNIQUE | Дата |
| | total_orders | INT | NOT NULL | Количество заказов |
| | total_revenue | DECIMAL(10,2) | NOT NULL | Общая выручка |
| | total_books_sold | INT | NOT NULL | Количество проданных книг |
| | average_order_value | DECIMAL(10,2) | NOT NULL | Средний чек |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |
| | updated_at | TIMESTAMP | NOT NULL | Дата обновления |

## Таблица: analytics_topsellingbook (Топ продаваемых книг)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| | date | DATE | NOT NULL | Дата |
| FK | book_id | INT | NOT NULL | Книга |
| | quantity_sold | INT | NOT NULL | Количество проданных |
| | revenue | DECIMAL(10,2) | NOT NULL | Выручка |
| | rank | INT | NOT NULL | Позиция в рейтинге |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |

## Таблица: analytics_customerstats (Статистика клиентов)

| Ключ | Наименование | Тип данных | Обязательность | Описание |
|------|--------------|------------|----------------|----------|
| PK | id | INT | NOT NULL | Уникальный идентификатор |
| | date | DATE | NOT NULL UNIQUE | Дата |
| | total_customers | INT | NOT NULL | Общее количество клиентов |
| | new_customers | INT | NOT NULL | Новые клиенты |
| | returning_customers | INT | NOT NULL | Постоянные клиенты |
| | average_customer_value | DECIMAL(10,2) | NOT NULL | Средняя стоимость клиента |
| | created_at | TIMESTAMP | NOT NULL | Дата создания |

**Общее количество таблиц:** 15+ таблиц




