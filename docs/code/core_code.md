# Код системных компонентов проекта BookShop

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


# RoleBasedAccessMiddleware убран, так как он конфликтует с текущей архитектурой
# Используем декораторы в views вместо middleware для проверки ролей
```

## backend/apps/core/models.py

```python
from django.db import models
from django.contrib.contenttypes.models import ContentType
from django.contrib.contenttypes.fields import GenericForeignKey
import json


class AuditLog(models.Model):
    """Система аудита для отслеживания всех изменений"""
    ACTION_CHOICES = [
        ('created', 'Создано'),
        ('updated', 'Обновлено'),
        ('deleted', 'Удалено'),
        ('viewed', 'Просмотрено'),
    ]
    
    action = models.CharField(max_length=100, choices=ACTION_CHOICES)
    actor = models.ForeignKey('auth.User', null=True, blank=True, on_delete=models.SET_NULL, related_name='audit_logs')
    content_type = models.ForeignKey(ContentType, on_delete=models.SET_NULL, null=True, blank=True)
    object_id = models.PositiveIntegerField(null=True, blank=True)
    content_object = GenericForeignKey('content_type', 'object_id')
    
    # Данные до изменения
    old_data = models.JSONField(null=True, blank=True)
    # Данные после изменения
    new_data = models.JSONField(null=True, blank=True)
    
    path = models.CharField(max_length=255, blank=True)
    method = models.CharField(max_length=10, blank=True)
    ip_address = models.GenericIPAddressField(null=True, blank=True)
    user_agent = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['content_type', 'object_id']),
            models.Index(fields=['actor', 'created_at']),
            models.Index(fields=['action', 'created_at']),
        ]

    def __str__(self) -> str:
        return f"{self.created_at} - {self.get_action_display()} by {self.actor or 'Unknown'}"
```

## backend/apps/reviews/models.py

```python
from django.conf import settings
from django.db import models


class Review(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    book = models.ForeignKey('catalog.Book', on_delete=models.CASCADE, related_name='reviews')
    rating = models.PositiveSmallIntegerField()
    text = models.TextField(blank=True)
    is_moderated = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ("user", "book")
        ordering = ['-created_at']

    def __str__(self) -> str:
        return f"{self.book} - {self.rating}/5 by {self.user}"
```




