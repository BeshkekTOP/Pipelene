# Код конфигурации проекта BookShop

## backend/api/docs.py

```python
from django.urls import re_path
from drf_yasg import openapi
from drf_yasg.views import get_schema_view
from rest_framework import permissions

schema_view = get_schema_view(
    openapi.Info(
        title="Bookstore API",
        default_version='v1',
        description="API documentation for the Bookstore",
    ),
    public=True,
    permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
    re_path(r'^docs(?P<format>\.json|\.yaml)$', schema_view.without_ui(cache_timeout=0), name='schema-json'),
    re_path(r'^docs/$', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    re_path(r'^redoc/$', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
]
```

## backend/apps/web/urls.py

```python
from django.urls import path
from . import views
from . import admin_views
from . import buyer_views
from . import manager_views
from . import sales_views

urlpatterns = [
    path('', views.login_view, name='home'),
    path('home/', views.home, name='home-page'),
    path('catalog/', views.catalog_list, name='catalog'),
    path('books/<int:pk>/', views.book_detail, name='book-detail'),
    path('cart/', views.cart_view, name='cart'),
    path('checkout/', views.checkout_view, name='checkout'),
    path('login/', views.login_view, name='login'),
    path('register/', views.register_view, name='register'),
    path('logout/', views.logout_view, name='logout'),
    path('profile/', views.profile_view, name='profile'),
    
    # Покупатель
    path('reviews/<int:book_id>/add/', buyer_views.add_review, name='add-review'),
    path('reviews/<int:book_id>/delete/', buyer_views.delete_review, name='delete-review'),
    path('profile/edit/', buyer_views.edit_profile, name='edit-profile'),
    path('checkout-detailed/', buyer_views.checkout_detailed, name='checkout-detailed'),
    path('checkout-success/<int:order_id>/', views.checkout_success, name='checkout-success'),
    path('orders/', buyer_views.orders_history, name='orders-history'),
    path('orders/<int:order_id>/', buyer_views.order_detail, name='order-detail'),
    
    # Управление каталогом
    path('admin/books/', views.admin_books, name='admin-books'),
    path('admin/books/<int:pk>/edit/', views.admin_book_edit, name='admin-book-edit'),
    path('admin/books/<int:pk>/delete/', views.admin_book_delete, name='admin-book-delete'),
    path('admin/authors/', views.admin_authors, name='admin-authors'),
    path('admin/categories/', views.admin_categories, name='admin-categories'),
    
    # Админ-панель
    path('admin/dashboard/', admin_views.admin_dashboard, name='admin-dashboard'),
    
    # Управление пользователями
    path('admin/users/', admin_views.admin_users_list, name='admin-users'),
    path('admin/users/<int:user_id>/', admin_views.admin_user_detail, name='admin-user-detail'),
    path('admin/users/<int:user_id>/block/', admin_views.admin_user_block, name='admin-user-block'),
    path('admin/users/<int:user_id>/change-role/', admin_views.admin_user_change_role, name='admin-user-change-role'),
    path('admin/users/<int:user_id>/activity/', admin_views.admin_user_activity_logs, name='admin-user-activity'),
    path('admin/users/<int:user_id>/delete/', admin_views.admin_user_delete, name='admin-user-delete'),
    
    # Управление остатками
    path('admin/inventory/', admin_views.admin_inventory, name='admin-inventory'),
    path('admin/inventory/<int:book_id>/update/', admin_views.admin_inventory_update, name='admin-inventory-update'),
    
    # Отчеты
    path('admin/reports/', admin_views.admin_reports, name='admin-reports'),
    path('admin/reports/export/', admin_views.admin_reports_export, name='admin-reports-export'),
    path('admin/reports/top-books/', admin_views.admin_reports_top_books, name='admin-reports-top-books'),
    path('admin/reports/user-activity/', admin_views.admin_reports_user_activity, name='admin-reports-user-activity'),
    
    # Логи
    path('admin/logs/', admin_views.admin_audit_logs, name='admin-logs'),
    
    # Менеджер
    path('manager/', manager_views.manager_dashboard, name='manager-dashboard'),
    path('manager/orders/', manager_views.manager_orders, name='manager-orders'),
    path('manager/orders/<int:order_id>/', manager_views.manager_order_detail, name='manager-order-detail'),
    path('manager/orders/<int:order_id>/update-status/', manager_views.manager_update_order_status, name='manager-update-order-status'),
    path('manager/statistics/', manager_views.manager_statistics, name='manager-statistics'),
    
    # Статистика продаж
    path('sales/', sales_views.sales_dashboard, name='sales-dashboard'),
    path('sales/reports/', sales_views.sales_reports, name='sales-reports'),
    path('manager/sales/', sales_views.manager_sales_stats, name='manager-sales-stats'),
]
```

## backend/apps/core/decorators.py

```python
"""Декораторы для проверки ролей и доступа"""
from functools import wraps
from django.contrib import messages
from django.shortcuts import redirect
from django.contrib.auth.decorators import user_passes_test


def guest_required(view_func):
    """Декоратор для функций, доступных только гостям (неавторизованным)"""
    @wraps(view_func)
    def wrapped_view(request, *args, **kwargs):
        if request.user.is_authenticated:
            messages.info(request, 'Вы уже авторизованы в системе')
            return redirect('catalog')
        return view_func(request, *args, **kwargs)
    
    return wrapped_view


def buyer_required(view_func):
    """Декоратор для функций, требующих авторизации (покупатель и выше)"""
    @wraps(view_func)
    def wrapped_view(request, *args, **kwargs):
        if not request.user.is_authenticated:
            messages.warning(request, 'Для доступа к этой странице необходимо войти в систему')
            return redirect('login')
        
        # Проверяем роль через профиль
        if hasattr(request.user, 'profile'):
            # Проверяем, что роль позволяет доступ (покупатель и выше)
            if request.user.profile.role not in ['buyer', 'manager', 'admin']:
                messages.error(request, 'У вас нет доступа к этой странице')
                return redirect('catalog')
        
        return view_func(request, *args, **kwargs)
    
    return wrapped_view


def admin_required(view_func):
    """Декоратор для функций, доступных только администраторам САЙТА (не Django)"""
    @wraps(view_func)
    def wrapped_view(request, *args, **kwargs):
        if not request.user.is_authenticated:
            messages.warning(request, 'Необходимо войти в систему')
            return redirect('login')
        
        # Проверяем роль через профиль
        if hasattr(request.user, 'profile'):
            if request.user.profile.role != 'admin':
                messages.error(request, 'У вас нет доступа к этой странице. Требуется роль администратора.')
                return redirect('catalog')
        else:
            # Если нет профиля, проверяем Django staff
            if not request.user.is_staff:
                messages.error(request, 'У вас нет доступа к этой странице')
                return redirect('catalog')
        
        return view_func(request, *args, **kwargs)
    
    return wrapped_view


def manager_required(view_func):
    """Декоратор для функций, доступных менеджерам и админам САЙТА"""
    @wraps(view_func)
    def wrapped_view(request, *args, **kwargs):
        if not request.user.is_authenticated:
            messages.warning(request, 'Необходимо войти в систему')
            return redirect('login')
        
        # Проверка роли через профиль
        if hasattr(request.user, 'profile'):
            if request.user.profile.role not in ['manager', 'admin']:
                messages.error(request, 'У вас нет доступа к этой странице. Требуется роль менеджера или администратора.')
                return redirect('catalog')
        else:
            # Если нет профиля, проверяем Django staff
            if not request.user.is_staff:
                messages.error(request, 'У вас нет доступа к этой странице')
                return redirect('catalog')
        
        return view_func(request, *args, **kwargs)
    
    return wrapped_view


def role_required(*roles):
    """Универсальный декоратор для проверки роли пользователя
    
    Args:
        *roles: Названия ролей, которым разрешен доступ
    """
    def decorator(view_func):
        @wraps(view_func)
        def wrapped_view(request, *args, **kwargs):
            if not request.user.is_authenticated:
                messages.warning(request, 'Необходимо войти в систему')
                return redirect('login')
            
            user_role = getattr(request.user, 'role', None) or 'buyer'
            
            if user_role not in roles:
                messages.error(request, 'У вас нет доступа к этой странице')
                return redirect('catalog')
            
            return view_func(request, *args, **kwargs)
        
        return wrapped_view
    return decorator


def permission_required(permission_name):
    """Декоратор для проверки конкретного разрешения"""
    def decorator(view_func):
        @wraps(view_func)
        def wrapped_view(request, *args, **kwargs):
            if not request.user.is_authenticated:
                messages.warning(request, 'Необходимо войти в систему')
                return redirect('login')
            
            # Проверка разрешения
            if not getattr(request.user, f'has_{permission_name}_permission', lambda: False)():
                messages.error(request, 'У вас нет прав для выполнения этого действия')
                return redirect('catalog')
            
            return view_func(request, *args, **kwargs)
        
        return wrapped_view
    return decorator
```

## backend/apps/core/middleware.py

```python
"""Система аудита и middleware для логирования"""
import logging
from django.http import HttpResponseForbidden
from django.utils.deprecation import MiddlewareMixin

logger = logging.getLogger(__name__)


class AuditMiddleware(MiddlewareMixin):
    """Middleware для логирования запросов пользователей"""
    
    def process_request(self, request):
        """Логирует информацию о запросе"""
        if request.user.is_authenticated:
            logger.info(
                f"User: {request.user.username}, "
                f"Path: {request.path}, "
                f"Method: {request.method}, "
                f"IP: {self.get_client_ip(request)}"
            )
        return None
    
    @staticmethod
    def get_client_ip(request):
        """Получает IP адрес клиента"""
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

## backend/settings/base.py

```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

SECRET_KEY = os.getenv("DJANGO_SECRET_KEY", "dev-secret-key")
DEBUG = bool(int(os.getenv("DJANGO_DEBUG", "1")))
ALLOWED_HOSTS = os.getenv("DJANGO_ALLOWED_HOSTS", "*").split(",")

DJANGO_APPS = [
	'django.contrib.admin',
	'django.contrib.auth',
	'django.contrib.contenttypes',
	'django.contrib.sessions',
	'django.contrib.messages',
	'django.contrib.staticfiles',
]

THIRD_PARTY_APPS = [
	'rest_framework',
	'corsheaders',
	'drf_yasg',
	'django_filters',
	'rest_framework_simplejwt.token_blacklist',
]

LOCAL_APPS = [
	'backend.apps.users',
	'backend.apps.catalog',
	'backend.apps.orders',
	'backend.apps.reviews',
	'backend.apps.analytics',
	'backend.apps.core',
	'backend.apps.web',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

MIDDLEWARE = [
	'corsheaders.middleware.CorsMiddleware',
	'django.middleware.security.SecurityMiddleware',
	'django.contrib.sessions.middleware.SessionMiddleware',
	'django.middleware.common.CommonMiddleware',
	'django.middleware.csrf.CsrfViewMiddleware',
	'django.contrib.auth.middleware.AuthenticationMiddleware',
	'django.contrib.messages.middleware.MessageMiddleware',
	'django.middleware.clickjacking.XFrameOptionsMiddleware',
	'backend.apps.core.middleware.AuditMiddleware',
]

ROOT_URLCONF = 'backend.urls'
WSGI_APPLICATION = 'backend.wsgi.application'
ASGI_APPLICATION = 'backend.asgi.application'

TEMPLATES = [
	{
		'BACKEND': 'django.template.backends.django.DjangoTemplates',
		'DIRS': [BASE_DIR / 'templates'],
		'APP_DIRS': True,
		'OPTIONS': {
			'context_processors': [
				'django.template.context_processors.debug',
				'django.template.context_processors.request',
				'django.contrib.auth.context_processors.auth',
				'django.contrib.messages.context_processors.messages',
			],
		},
	},
]

DATABASES = {
	'default': {
		'ENGINE': 'django.db.backends.postgresql',
		'NAME': os.getenv('POSTGRES_DB', 'bookstore'),
		'USER': os.getenv('POSTGRES_USER', 'bookstore'),
		'PASSWORD': os.getenv('POSTGRES_PASSWORD', 'bookstore'),
		'HOST': os.getenv('POSTGRES_HOST', 'localhost'),
		'PORT': os.getenv('POSTGRES_PORT', '5432'),
	}
}

LANGUAGE_CODE = os.getenv('LANGUAGE_CODE', 'en-us')
TIME_ZONE = os.getenv('TIME_ZONE', 'UTC')
USE_I18N = True
USE_TZ = True

STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

REST_FRAMEWORK = {
	'DEFAULT_AUTHENTICATION_CLASSES': (
		'rest_framework_simplejwt.authentication.JWTAuthentication',
	),
	'DEFAULT_PERMISSION_CLASSES': (
		'rest_framework.permissions.AllowAny',
	),
	'DEFAULT_FILTER_BACKENDS': (
		'django_filters.rest_framework.DjangoFilterBackend',
		'rest_framework.filters.SearchFilter',
		'rest_framework.filters.OrderingFilter',
	),
}

CORS_ALLOWED_ORIGINS = os.getenv('DJANGO_CORS_ORIGINS', '').split(',') if os.getenv('DJANGO_CORS_ORIGINS') else []

REDIS_URL = os.getenv('REDIS_URL', 'redis://localhost:6379/0')

# Celery
CELERY_BROKER_URL = REDIS_URL
CELERY_RESULT_BACKEND = REDIS_URL
CELERY_BROKER_CONNECTION_RETRY_ON_STARTUP = True

# Channels
CHANNEL_LAYERS = {
	'default': {
		'BACKEND': 'channels_redis.core.RedisChannelLayer',
		'CONFIG': {
			'hosts': [REDIS_URL],
		},
	}
}

# Swagger
SWAGGER_SETTINGS = {
	'USE_SESSION_AUTH': False,
}
```

## manage.py

```python
#!/usr/bin/env python
import os
import sys


def main() -> None:
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "backend.settings.dev")
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)


if __name__ == "__main__":
    main()
```




