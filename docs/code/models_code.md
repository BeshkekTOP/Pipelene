# Код моделей проекта BookShop

## backend/apps/catalog/models.py

```python
from django.db import models


class Category(models.Model):
    name = models.CharField(max_length=120, unique=True)
    slug = models.SlugField(max_length=140, unique=True)

    class Meta:
        ordering = ["name"]

    def __str__(self) -> str:
        return self.name


class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    class Meta:
        unique_together = ("first_name", "last_name")
        ordering = ["last_name", "first_name"]

    def __str__(self) -> str:
        return f"{self.first_name} {self.last_name}"


class Book(models.Model):
    title = models.CharField(max_length=255)
    isbn = models.CharField(max_length=20, unique=True)
    description = models.TextField(blank=True)
    category = models.ForeignKey(Category, on_delete=models.PROTECT, related_name="books")
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rating = models.DecimalField(max_digits=3, decimal_places=2, default=0)
    pages = models.PositiveIntegerField(null=True, blank=True)
    publication_date = models.DateField(null=True, blank=True)
    cover_image = models.ImageField(upload_to='book_covers/', blank=True, null=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["title"]
        indexes = [
            models.Index(fields=['category', 'is_active']),
            models.Index(fields=['price']),
            models.Index(fields=['rating']),
        ]

    def __str__(self) -> str:
        return self.title

    @property
    def average_rating(self):
        """Вычисляет средний рейтинг на основе отзывов"""
        from django.db.models import Avg
        return self.reviews.aggregate(avg_rating=Avg('rating'))['avg_rating'] or 0


class BookAuthors(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE, related_name="book_authors")
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name="author_books")

    class Meta:
        unique_together = ("book", "author")


class Inventory(models.Model):
    book = models.OneToOneField(Book, on_delete=models.CASCADE, related_name="inventory")
    stock = models.PositiveIntegerField(default=0)
    reserved = models.PositiveIntegerField(default=0)

    def available(self) -> int:
        return max(0, self.stock - self.reserved)
```

## backend/apps/orders/models.py

```python
from django.conf import settings
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver


class Cart(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='carts')
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self) -> str:
        return f"Cart #{self.pk} for {self.user}"


class CartItem(models.Model):
    cart = models.ForeignKey(Cart, on_delete=models.CASCADE, related_name='items')
    book = models.ForeignKey('catalog.Book', on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=1)

    class Meta:
        unique_together = ("cart", "book")


class Order(models.Model):
    STATUS_CHOICES = (
        ("processing", "Обрабатывается"),
        ("shipped", "Отправлен"),
        ("delivered", "Доставлен"),
        ("cancelled", "Отменен"),
    )

    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='orders')
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default="processing")
    total_amount = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    shipping_address = models.TextField(default="")
    shipping_city = models.CharField(max_length=100, default="")
    shipping_postal_code = models.CharField(max_length=20, default="")
    notes = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['user', 'status']),
            models.Index(fields=['status', 'created_at']),
        ]

    def __str__(self) -> str:
        return f"Order #{self.pk} ({self.status})"


class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name='items')
    book = models.ForeignKey('catalog.Book', on_delete=models.PROTECT)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    quantity = models.PositiveIntegerField()


@receiver(post_save, sender=Order)
def update_sales_stats_on_order_change(sender, instance, created, **kwargs):
    """Обновить статистику продаж при изменении статуса заказа"""
    if instance.status == 'delivered':
        try:
            from backend.apps.analytics.models import SalesStats, TopSellingBook, CustomerStats
            from django.utils import timezone
            
            # Обновляем статистику за день
            SalesStats.update_daily_stats(instance.created_at.date())
            TopSellingBook.update_daily_top_books(instance.created_at.date())
            CustomerStats.update_daily_customer_stats(instance.created_at.date())
        except Exception as e:
            # Логируем ошибку, но не прерываем выполнение
            import logging
            logger = logging.getLogger(__name__)
            logger.error(f"Ошибка при обновлении статистики продаж: {e}")
```

## backend/apps/users/models.py

```python
from django.conf import settings
from django.db import models


class Profile(models.Model):
    """
    Профиль пользователя с ролью
    
    Роли для сайта (НЕ Django admin):
    - guest: Гость (неавторизованный)
    - buyer: Покупатель (обычный пользователь)
    - manager: Менеджер (обработка заказов)
    - admin: Администратор сайта
    """
    
    ROLE_CHOICES = [
        ('guest', 'Гость'),
        ('buyer', 'Покупатель'),
        ('manager', 'Менеджер'),
        ('admin', 'Администратор'),
    ]
    
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='profile')
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='buyer', verbose_name='Роль на сайте')
    is_blocked = models.BooleanField(default=False, verbose_name='Заблокирован')
    blocked_reason = models.TextField(blank=True, verbose_name='Причина блокировки')
    phone = models.CharField(max_length=20, blank=True)
    address = models.TextField(blank=True)
    city = models.CharField(max_length=100, blank=True)
    postal_code = models.CharField(max_length=20, blank=True)
    date_of_birth = models.DateField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self) -> str:
        return f"Profile of {self.user} ({self.get_role_display()})"

    @property
    def full_name(self):
        return f"{self.user.first_name} {self.user.last_name}".strip() or self.user.username
    
    def is_admin(self):
        """Проверка, является ли пользователь администратором сайта"""
        return self.role == 'admin'
    
    def is_manager(self):
        """Проверка, является ли пользователь менеджером"""
        return self.role in ['manager', 'admin']
    
    def is_buyer(self):
        """Проверка, является ли пользователь покупателем или выше"""
        return self.role in ['buyer', 'manager', 'admin']
    
    def is_active(self):
        """Проверка, активен ли пользователь (не заблокирован)"""
        return not self.is_blocked
```




