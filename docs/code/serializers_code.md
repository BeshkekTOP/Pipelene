# Код сериализаторов проекта BookShop

## backend/apps/users/serializers.py

```python
from django.contrib.auth import get_user_model
from rest_framework import serializers
from .models import Profile

User = get_user_model()


class ProfileSerializer(serializers.ModelSerializer):
    full_name = serializers.ReadOnlyField()

    class Meta:
        model = Profile
        fields = (
            "id", "phone", "address", "city", "postal_code", 
            "date_of_birth", "avatar", "full_name", "created_at", "updated_at"
        )
        read_only_fields = ("created_at", "updated_at")


class UserSerializer(serializers.ModelSerializer):
    profile = ProfileSerializer(read_only=True)

    class Meta:
        model = User
        fields = (
            "id", "username", "email", "first_name", "last_name", 
            "is_active", "date_joined", "profile"
        )
        read_only_fields = ("id", "date_joined", "is_active")


class RegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, min_length=8)
    password_confirm = serializers.CharField(write_only=True)

    class Meta:
        model = User
        fields = ("id", "username", "email", "first_name", "last_name", "password", "password_confirm")

    def validate(self, attrs):
        if attrs['password'] != attrs['password_confirm']:
            raise serializers.ValidationError("Пароли не совпадают")
        return attrs

    def create(self, validated_data):
        validated_data.pop('password_confirm')
        password = validated_data.pop("password")
        user = User(**validated_data)
        user.set_password(password)
        user.save()
        # Создаем профиль для нового пользователя
        Profile.objects.create(user=user)
        return user


class LoginSerializer(serializers.Serializer):
    username = serializers.CharField()
    password = serializers.CharField(write_only=True)
```

## backend/apps/catalog/serializers.py

```python
from rest_framework import serializers
from .models import Category, Author, Book, BookAuthors, Inventory


class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ("id", "name", "slug")


class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ("id", "first_name", "last_name")


class InventorySerializer(serializers.ModelSerializer):
    available = serializers.IntegerField(read_only=True)

    class Meta:
        model = Inventory
        fields = ("stock", "reserved", "available")


class BookAuthorNestedSerializer(serializers.ModelSerializer):
    author = AuthorSerializer()

    class Meta:
        model = BookAuthors
        fields = ("author",)


class BookSerializer(serializers.ModelSerializer):
    category = CategorySerializer()
    authors = serializers.SerializerMethodField()
    inventory = InventorySerializer(read_only=True)
    average_rating = serializers.ReadOnlyField()

    class Meta:
        model = Book
        fields = (
            "id",
            "title",
            "isbn",
            "description",
            "category",
            "price",
            "rating",
            "average_rating",
            "pages",
            "publication_date",
            "cover_image",
            "is_active",
            "created_at",
            "updated_at",
            "authors",
            "inventory",
        )

    def get_authors(self, obj):
        qs = obj.book_authors.select_related("author")
        return AuthorSerializer([ba.author for ba in qs], many=True).data


class BookWriteSerializer(serializers.ModelSerializer):
    author_ids = serializers.ListField(child=serializers.IntegerField(), write_only=True)

    class Meta:
        model = Book
        fields = ("title", "isbn", "description", "category", "price", "author_ids")

    def create(self, validated_data):
        author_ids = validated_data.pop("author_ids", [])
        book = Book.objects.create(**validated_data)
        if author_ids:
            BookAuthors.objects.bulk_create([
                BookAuthors(book=book, author_id=aid) for aid in author_ids
            ])
        Inventory.objects.get_or_create(book=book, defaults={"stock": 0, "reserved": 0})
        return book

    def update(self, instance, validated_data):
        author_ids = validated_data.pop("author_ids", None)
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        if author_ids is not None:
            BookAuthors.objects.filter(book=instance).delete()
            BookAuthors.objects.bulk_create([
                BookAuthors(book=instance, author_id=aid) for aid in author_ids
            ])
        return instance
```

## backend/apps/orders/serializers.py

```python
from rest_framework import serializers
from .models import Cart, CartItem, Order, OrderItem


class CartItemSerializer(serializers.ModelSerializer):
    book_title = serializers.CharField(source='book.title', read_only=True)
    price = serializers.DecimalField(source='book.price', read_only=True, max_digits=10, decimal_places=2)

    class Meta:
        model = CartItem
        fields = ("id", "book", "book_title", "price", "quantity")


class CartSerializer(serializers.ModelSerializer):
    items = CartItemSerializer(many=True, read_only=True)

    class Meta:
        model = Cart
        fields = ("id", "user", "created_at", "items")
        read_only_fields = ("user",)


class OrderItemSerializer(serializers.ModelSerializer):
    book_title = serializers.CharField(source='book.title', read_only=True)

    class Meta:
        model = OrderItem
        fields = ("id", "book", "book_title", "price", "quantity")


class OrderSerializer(serializers.ModelSerializer):
    items = OrderItemSerializer(many=True, read_only=True)
    status_display = serializers.CharField(source='get_status_display', read_only=True)

    class Meta:
        model = Order
        fields = (
            "id", "user", "status", "status_display", "total_amount", 
            "shipping_address", "shipping_city", "shipping_postal_code", 
            "notes", "created_at", "updated_at", "items"
        )
        read_only_fields = ("user", "status", "total_amount", "created_at", "updated_at")
```

## backend/apps/reviews/serializers.py

```python
from rest_framework import serializers
from .models import Review


class ReviewSerializer(serializers.ModelSerializer):
    user_email = serializers.EmailField(source='user.email', read_only=True)
    user_name = serializers.CharField(source='user.username', read_only=True)
    book_title = serializers.CharField(source='book.title', read_only=True)

    class Meta:
        model = Review
        fields = (
            "id", "user", "user_email", "user_name", "book", "book_title", 
            "rating", "text", "is_moderated", "created_at"
        )
        read_only_fields = ("user", "is_moderated", "created_at")

    def validate_rating(self, value):
        if value < 1 or value > 5:
            raise serializers.ValidationError("Рейтинг должен быть от 1 до 5")
        return value
```

## backend/apps/analytics/serializers.py

```python
from rest_framework import serializers
from .models import PageView, BookView, SearchQuery, PurchaseEvent


class PageViewSerializer(serializers.ModelSerializer):
    class Meta:
        model = PageView
        fields = (
            "id", "user", "session_key", "path", "ip_address", 
            "user_agent", "referer", "timestamp"
        )
        read_only_fields = ("timestamp",)


class BookViewSerializer(serializers.ModelSerializer):
    book_title = serializers.CharField(source='book.title', read_only=True)

    class Meta:
        model = BookView
        fields = (
            "id", "user", "book", "book_title", "session_key", 
            "ip_address", "timestamp"
        )
        read_only_fields = ("timestamp",)


class SearchQuerySerializer(serializers.ModelSerializer):
    class Meta:
        model = SearchQuery
        fields = (
            "id", "user", "query", "results_count", "session_key", 
            "ip_address", "timestamp"
        )
        read_only_fields = ("timestamp",)


class PurchaseEventSerializer(serializers.ModelSerializer):
    order_id = serializers.IntegerField(source='order.id', read_only=True)

    class Meta:
        model = PurchaseEvent
        fields = (
            "id", "user", "order", "order_id", "total_amount", 
            "items_count", "timestamp"
        )
        read_only_fields = ("timestamp",)
```




