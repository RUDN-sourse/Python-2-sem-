# Лабораторная работа: Создание простого веб-приложения на Django

## Тема: "Персональный блог на Django"
---

## Цель работы
Освоить базовые концепции Django: создание проекта, работа с моделями, представлениями, шаблонами и админ-панелью.

---

## Теоретическая справка

### Что такое Django?
Django - это высокоуровневый Python фреймворк для быстрой разработки веб-приложений.

### Основные компоненты:
- **Модели (Models)** - описание структуры данных
- **Представления (Views)** - обработка запросов и логика приложения
- **Шаблоны (Templates)** - отображение данных
- **URL-маршрутизация** - связь URL с представлениями

---

## Часть 1: Настройка проекта

### 1.1 Создание виртуального окружения и установка Django
```bash
# Создание виртуального окружения
python -m venv myblog_env
source myblog_env/bin/activate  # Linux/Mac
# myblog_env\Scripts\activate  # Windows

# Установка Django
pip install django
pip install pillow  # для работы с изображениями
```

### 1.2 Создание проекта и приложения
```bash
# Создание проекта
django-admin startproject myblog
cd myblog

# Создание приложения
python manage.py startapp blog
```

### 1.3 Настройка проекта
```python
# myblog/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',  # добавляем наше приложение
]

# Добавьте в конец файла
import os
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

---

## Часть 2: Создание моделей

```python
# blog/models.py
from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100, verbose_name='Название категории')
    description = models.TextField(blank=True, verbose_name='Описание')
    
    def __str__(self):
        return self.name
    
    class Meta:
        verbose_name = 'Категория'
        verbose_name_plural = 'Категории'

class Post(models.Model):
    title = models.CharField(max_length=200, verbose_name='Заголовок')
    content = models.TextField(verbose_name='Содержание')
    created_date = models.DateTimeField(default=timezone.now, verbose_name='Дата создания')
    published_date = models.DateTimeField(blank=True, null=True, verbose_name='Дата публикации')
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name='Автор')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, verbose_name='Категория')
    image = models.ImageField(upload_to='post_images/', blank=True, null=True, verbose_name='Изображение')
    
    def publish(self):
        self.published_date = timezone.now()
        self.save()
    
    def __str__(self):
        return self.title
    
    class Meta:
        verbose_name = 'Пост'
        verbose_name_plural = 'Посты'
```

### Миграции
```bash
# Создание и применение миграций
python manage.py makemigrations
python manage.py migrate
```

---

## Часть 3: Админ-панель

```python
# blog/admin.py
from django.contrib import admin
from .models import Category, Post

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'description']
    search_fields = ['name']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'created_date', 'published_date', 'category']
    list_filter = ['created_date', 'published_date', 'category']
    search_fields = ['title', 'content']
    date_hierarchy = 'created_date'
    
    fieldsets = (
        ('Основная информация', {
            'fields': ('title', 'content', 'author', 'category')
        }),
        ('Даты', {
            'fields': ('created_date', 'published_date')
        }),
        ('Медиа', {
            'fields': ('image',)
        }),
    )
```

### Создание суперпользователя
```bash
python manage.py createsuperuser
# Введите имя пользователя, email и пароль
```

---

## Часть 4: Представления (Views)

```python
# blog/views.py
from django.shortcuts import render, get_object_or_404
from .models import Post, Category
from django.utils import timezone

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('-published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})

def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})

def category_posts(request, category_id):
    category = get_object_or_404(Category, id=category_id)
    posts = Post.objects.filter(category=category, published_date__lte=timezone.now()).order_by('-published_date')
    return render(request, 'blog/category_posts.html', {'category': category, 'posts': posts})
```

---

## Часть 5: URL-маршрутизация

```python
# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('category/<int:category_id>/', views.category_posts, name='category_posts'),
]

# myblog/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## Часть 6: Шаблоны (Templates)

### Базовый шаблон
```html
<!-- blog/templates/blog/base.html -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Мой Блог{% endblock %}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 0; }
        .header { background: #333; color: white; padding: 1rem; }
        .nav { background: #f4f4f4; padding: 1rem; }
        .content { padding: 2rem; }
        .post { border: 1px solid #ddd; margin-bottom: 1rem; padding: 1rem; }
        .post img { max-width: 200px; }
    </style>
</head>
<body>
    <div class="header">
        <h1><a href="/" style="color: white; text-decoration: none;">Мой Персональный Блог</a></h1>
    </div>
    
    <div class="nav">
        <a href="/">Главная</a> |
        <a href="/admin/">Админ-панель</a>
    </div>
    
    <div class="content">
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```

### Список постов
```html
<!-- blog/templates/blog/post_list.html -->
{% extends 'blog/base.html' %}

{% block title %}Главная страница{% endblock %}

{% block content %}
<h2>Последние посты</h2>

{% for post in posts %}
    <div class="post">
        <h3><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h3>
        {% if post.image %}
            <img src="{{ post.image.url }}" alt="{{ post.title }}">
        {% endif %}
        <p>{{ post.content|truncatewords:30 }}</p>
        <small>
            Автор: {{ post.author }} | 
            Опубликовано: {{ post.published_date|date:"d.m.Y H:i" }} |
            Категория: {{ post.category.name }}
        </small>
    </div>
{% empty %}
    <p>Пока нет постов.</p>
{% endfor %}
{% endblock %}
```

### Детальная страница поста
```html
<!-- blog/templates/blog/post_detail.html -->
{% extends 'blog/base.html' %}

{% block title %}{{ post.title }}{% endblock %}

{% block content %}
    <article class="post">
        <h2>{{ post.title }}</h2>
        {% if post.image %}
            <img src="{{ post.image.url }}" alt="{{ post.title }}">
        {% endif %}
        <p>{{ post.content|linebreaks }}</p>
        <div class="post-meta">
            <p><strong>Автор:</strong> {{ post.author }}</p>
            <p><strong>Опубликовано:</strong> {{ post.published_date|date:"d.m.Y H:i" }}</p>
            <p><strong>Категория:</strong> {{ post.category.name }}</p>
        </div>
        <a href="{% url 'post_list' %}">← Назад к списку постов</a>
    </article>
{% endblock %}
```

---

## Часть 7: Заполнение базы данных

Создайте файл для заполнения базы данных тестовыми данными:

```python
# blog/management/commands/fill_db.py
from django.core.management.base import BaseCommand
from blog.models import Category, Post
from django.contrib.auth.models import User
from django.utils import timezone

class Command(BaseCommand):
    help = 'Заполнение базы данных тестовыми данными'

    def handle(self, *args, **options):
        # Создаем пользователя
        user, created = User.objects.get_or_create(
            username='testuser',
            defaults={'email': 'test@example.com'}
        )
        if created:
            user.set_password('testpassword123')
            user.save()

        # Создаем категории
        categories_data = [
            {'name': 'Программирование', 'description': 'Статьи о программировании'},
            {'name': 'Путешествия', 'description': 'Рассказы о путешествиях'},
            {'name': 'Кулинария', 'description': 'Рецепты и советы по готовке'},
        ]
        
        for cat_data in categories_data:
            category, created = Category.objects.get_or_create(**cat_data)
            if created:
                self.stdout.write(f'Создана категория: {category.name}')

        # Создаем посты
        posts_data = [
            {
                'title': 'Мой первый пост в блоге',
                'content': 'Это содержимое моего первого поста. Я только начинаю вести блог и хочу делиться своими мыслями и идеями.',
                'author': user,
                'category': Category.objects.get(name='Программирование')
            },
            {
                'title': 'Лучшие места для путешествий',
                'content': 'В этом посте я расскажу о самых красивых местах, которые я посетил за последний год.',
                'author': user,
                'category': Category.objects.get(name='Путешествия')
            },
            {
                'title': 'Простой рецепт пасты',
                'content': 'Сегодня поделюсь с вами своим любимым рецептом пасты карбонара.',
                'author': user,
                'category': Category.objects.get(name='Кулинария')
            },
        ]

        for post_data in posts_data:
            post = Post.objects.create(**post_data)
            post.publish()  # Устанавливаем дату публикации
            self.stdout.write(f'Создан пост: {post.title}')

        self.stdout.write(self.style.SUCCESS('База данных успешно заполнена!'))
```

Запустите команду:
```bash
python manage.py fill_db
```

---

## Часть 8: Запуск и тестирование

### Запуск сервера
```bash
python manage.py runserver
```

### Задания для проверки:
1. Перейдите на http://127.0.0.1:8000/ - должна открыться главная страница с постами
2. Перейдите на http://127.0.0.1:8000/admin/ - войдите в админ-панель
3. В админ-панели создайте новый пост
4. Проверьте, что пост отображается на главной странице
5. Перейдите на страницу детального просмотра поста

---

## Дополнительные задания (НЕ по желанию)

### Задание 1: Добавление поиска
Добавьте функцию поиска постов по заголовку и содержанию.

### Задание 2: Комментарии к постам
Создайте модель для комментариев и добавьте возможность оставлять комментарии к постам.

### Задание 3: Пагинация
Добавьте пагинацию на страницу списка постов (по 5 постов на страницу).

## Что должно получиться в результате:

✅ Работающее веб-приложение блога  
✅ Админ-панель для управления контентом  
✅ Главная страница со списком постов  
✅ Страницы детального просмотра постов  
✅ Возможность добавлять посты через админ-панель  
