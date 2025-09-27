# Разработка веб-приложений на Python: полное руководство

## 1. Введение в веб-разработку на Python

### Что такое веб-фреймворк?
**Веб-фреймворк** - это набор инструментов и библиотек, которые упрощают создание веб-приложений. Python предлагает несколько мощных фреймворков:

- **Django** - полнофункциональный фреймворк "все включено"
- **Flask** - микрофреймворк для простых приложений
- **FastAPI** - современный фреймворк для API
- **Pyramid** - гибкий фреймворк для средних и крупных проектов

### Архитектура веб-приложения
```
Клиент (браузер) → Веб-сервер → Python приложение → База данных
```

## 2. Flask: микрофреймворк для начинающих

### Установка и настройка

```bash
# Установка Flask
pip install flask

# Создание виртуального окружения (рекомендуется)
python -m venv myenv
source myenv/bin/activate  # Linux/Mac
# myenv\Scripts\activate  # Windows
```

### Первое приложение на Flask

**app.py:**
```python
from flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3
import os

# 🔍 Создание экземпляра приложения
app = Flask(__name__)
app.secret_key = 'your-secret-key-here'  # 🔐 Для сессий и flash-сообщений

# 🔍 База данных
def init_db():
    """Инициализация базы данных"""
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    
    # Создание таблицы пользователей
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    
    conn.commit()
    conn.close()

# 🔍 Главная страница
@app.route('/')
def index():
    """Главная страница"""
    return render_template('index.html', title='Главная страница')

# 🔍 Страница "О нас"
@app.route('/about')
def about():
    """Страница о компании"""
    return render_template('about.html', title='О нас')

# 🔍 Страница с пользователями
@app.route('/users')
def users():
    """Отображение списка пользователей"""
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    
    cursor.execute('SELECT * FROM users ORDER BY created_at DESC')
    users = cursor.fetchall()
    
    conn.close()
    
    return render_template('users.html', title='Пользователи', users=users)

# 🔍 Добавление пользователя
@app.route('/add_user', methods=['GET', 'POST'])
def add_user():
    """Добавление нового пользователя"""
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        
        try:
            conn = sqlite3.connect('database.db')
            cursor = conn.cursor()
            
            cursor.execute(
                'INSERT INTO users (name, email) VALUES (?, ?)',
                (name, email)
            )
            
            conn.commit()
            conn.close()
            
            flash('Пользователь успешно добавлен!', 'success')
            return redirect(url_for('users'))
            
        except sqlite3.IntegrityError:
            flash('Пользователь с таким email уже существует!', 'error')
    
    return render_template('add_user.html', title='Добавить пользователя')

# 🔍 API endpoint для получения пользователей в JSON
@app.route('/api/users')
def api_users():
    """API для получения пользователей в формате JSON"""
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    
    cursor.execute('SELECT * FROM users')
    users = cursor.fetchall()
    
    conn.close()
    
    # Преобразование в список словарей
    users_list = []
    for user in users:
        users_list.append({
            'id': user[0],
            'name': user[1],
            'email': user[2],
            'created_at': user[3]
        })
    
    return {'users': users_list}

# 🔍 Обработка ошибок
@app.errorhandler(404)
def page_not_found(error):
    """Обработка ошибки 404"""
    return render_template('404.html', title='Страница не найдена'), 404

# 🔍 Запуск приложения
if __name__ == '__main__':
    init_db()  # Инициализируем базу данных
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### Структура папок проекта Flask
```
my_flask_app/
├── app.py                 # Главный файл приложения
├── database.db           # База данных SQLite
├── requirements.txt      # Зависимости проекта
├── static/              # Статические файлы
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── script.js
│   └── images/
└── templates/           # HTML шаблоны
    ├── base.html        # Базовый шаблон
    ├── index.html
    ├── about.html
    ├── users.html
    ├── add_user.html
    └── 404.html
```

### HTML шаблоны с Jinja2

**templates/base.html:**
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Мое приложение{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <nav class="navbar">
        <div class="nav-container">
            <a href="{{ url_for('index') }}" class="nav-logo">МойСайт</a>
            <ul class="nav-menu">
                <li><a href="{{ url_for('index') }}">Главная</a></li>
                <li><a href="{{ url_for('about') }}">О нас</a></li>
                <li><a href="{{ url_for('users') }}">Пользователи</a></li>
                <li><a href="{{ url_for('add_user') }}">Добавить пользователя</a></li>
            </ul>
        </div>
    </nav>

    <main class="container">
        <!-- 🔍 Flash-сообщения -->
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                <div class="flash-messages">
                    {% for category, message in messages %}
                        <div class="flash flash-{{ category }}">{{ message }}</div>
                    {% endfor %}
                </div>
            {% endif %}
        {% endwith %}

        <!-- 🔍 Содержимое страницы -->
        {% block content %}{% endblock %}
    </main>

    <footer class="footer">
        <p>&copy; 2024 Мое приложение. Все права защищены.</p>
    </footer>

    <script src="{{ url_for('static', filename='js/script.js') }}"></script>
</body>
</html>
```

**templates/index.html:**
```html
{% extends "base.html" %}

{% block title %}{{ title }}{% endblock %}

{% block content %}
<div class="hero">
    <h1>Добро пожаловать на наш сайт!</h1>
    <p>Это пример веб-приложения на Flask</p>
    <a href="{{ url_for('add_user') }}" class="btn btn-primary">Начать работу</a>
</div>

<div class="features">
    <div class="feature-card">
        <h3>🚀 Быстро</h3>
        <p>Высокая производительность благодаря Flask</p>
    </div>
    <div class="feature-card">
        <h3>🔒 Безопасно</h3>
        <p>Встроенная защита от основных угроз</p>
    </div>
    <div class="feature-card">
        <h3>📱 Адаптивно</h3>
        <p>Работает на всех устройствах</p>
    </div>
</div>
{% endblock %}
```

**templates/users.html:**
```html
{% extends "base.html" %}

{% block title %}{{ title }}{% endblock %}

{% block content %}
<div class="header">
    <h1>Список пользователей</h1>
    <a href="{{ url_for('add_user') }}" class="btn btn-success">Добавить пользователя</a>
</div>

<table class="users-table">
    <thead>
        <tr>
            <th>ID</th>
            <th>Имя</th>
            <th>Email</th>
            <th>Дата регистрации</th>
        </tr>
    </thead>
    <tbody>
        {% for user in users %}
        <tr>
            <td>{{ user[0] }}</td>
            <td>{{ user[1] }}</td>
            <td>{{ user[2] }}</td>
            <td>{{ user[3] }}</td>
        </tr>
        {% else %}
        <tr>
            <td colspan="4" class="no-data">Пользователи не найдены</td>
        </tr>
        {% endfor %}
    </tbody>
</table>

<!-- 🔍 API ссылка -->
<div class="api-section">
    <p>Данные также доступны через API: 
        <a href="{{ url_for('api_users') }}">{{ url_for('api_users', _external=True) }}</a>
    </p>
</div>
{% endblock %}
```

### CSS стили (static/css/style.css)

```css
/* Базовые стили */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f4f4f4;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    min-height: calc(100vh - 140px);
}

/* Навигация */
.navbar {
    background: #2c3e50;
    color: white;
    padding: 1rem 0;
}

.nav-container {
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
}

.nav-logo {
    font-size: 1.5rem;
    font-weight: bold;
    color: white;
    text-decoration: none;
}

.nav-menu {
    display: flex;
    list-style: none;
}

.nav-menu li {
    margin-left: 20px;
}

.nav-menu a {
    color: white;
    text-decoration: none;
    transition: color 0.3s;
}

.nav-menu a:hover {
    color: #3498db;
}

/* Кнопки */
.btn {
    display: inline-block;
    padding: 10px 20px;
    background: #3498db;
    color: white;
    text-decoration: none;
    border-radius: 5px;
    border: none;
    cursor: pointer;
    transition: background 0.3s;
}

.btn:hover {
    background: #2980b9;
}

.btn-primary {
    background: #e74c3c;
}

.btn-primary:hover {
    background: #c0392b;
}

.btn-success {
    background: #27ae60;
}

.btn-success:hover {
    background: #219a52;
}

/* Flash сообщения */
.flash-messages {
    margin-bottom: 20px;
}

.flash {
    padding: 15px;
    margin-bottom: 10px;
    border-radius: 5px;
}

.flash-success {
    background: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
}

.flash-error {
    background: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
}

/* Главная страница */
.hero {
    text-align: center;
    padding: 60px 0;
    background: white;
    border-radius: 10px;
    margin-bottom: 40px;
}

.hero h1 {
    font-size: 2.5rem;
    margin-bottom: 20px;
    color: #2c3e50;
}

.hero p {
    font-size: 1.2rem;
    margin-bottom: 30px;
    color: #7f8c8d;
}

.features {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 30px;
    margin-top: 40px;
}

.feature-card {
    background: white;
    padding: 30px;
    border-radius: 10px;
    text-align: center;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.feature-card h3 {
    font-size: 1.5rem;
    margin-bottom: 15px;
    color: #2c3e50;
}

/* Таблица пользователей */
.users-table {
    width: 100%;
    border-collapse: collapse;
    background: white;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.users-table th,
.users-table td {
    padding: 15px;
    text-align: left;
    border-bottom: 1px solid #ecf0f1;
}

.users-table th {
    background: #34495e;
    color: white;
    font-weight: bold;
}

.users-table tr:hover {
    background: #f8f9fa;
}

.no-data {
    text-align: center;
    color: #7f8c8d;
    font-style: italic;
}

/* Формы */
.form-group {
    margin-bottom: 20px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
    color: #2c3e50;
}

.form-group input,
.form-group textarea,
.form-group select {
    width: 100%;
    padding: 10px;
    border: 1px solid #bdc3c7;
    border-radius: 5px;
    font-size: 1rem;
}

.form-group input:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 5px rgba(52, 152, 219, 0.3);
}

/* Футер */
.footer {
    background: #2c3e50;
    color: white;
    text-align: center;
    padding: 20px 0;
    margin-top: 40px;
}

/* Адаптивность */
@media (max-width: 768px) {
    .nav-container {
        flex-direction: column;
    }
    
    .nav-menu {
        margin-top: 15px;
    }
    
    .nav-menu li {
        margin: 0 10px;
    }
    
    .hero h1 {
        font-size: 2rem;
    }
    
    .features {
        grid-template-columns: 1fr;
    }
}
```

## 3. Django: полнофункциональный фреймворк

### Создание проекта Django

```bash
# Установка Django
pip install django

# Создание проекта
django-admin startproject myproject
cd myproject

# Создание приложения
python manage.py startapp myapp
```

### Структура проекта Django
```
myproject/
├── manage.py                 # Управление проектом
├── myproject/               # Настройки проекта
│   ├── __init__.py
│   ├── settings.py          # 🔧 Настройки
│   ├── urls.py              # 🌐 Главные URL-ы
│   └── wsgi.py              # 🚀 WSGI конфигурация
└── myapp/                   # Приложение
    ├── migrations/          # Миграции базы данных
    ├── __init__.py
    ├── admin.py            # 🔧 Админка
    ├── apps.py             # ⚙️ Конфигурация приложения
    ├── models.py           # 🗄️ Модели данных
    ├── tests.py            # 🧪 Тесты
    ├── urls.py             # 🌐 URL-ы приложения
    └── views.py            # 👀 Представления
```

### Модели Django (models.py)

```python
from django.db import models
from django.urls import reverse
from django.core.validators import EmailValidator, MinLengthValidator

class User(models.Model):
    """Модель пользователя"""
    name = models.CharField(
        max_length=100,
        validators=[MinLengthValidator(2)],
        verbose_name='Имя'
    )
    email = models.EmailField(
        unique=True,
        validators=[EmailValidator()],
        verbose_name='Email'
    )
    is_active = models.BooleanField(default=True, verbose_name='Активен')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='Дата создания')
    updated_at = models.DateTimeField(auto_now=True, verbose_name='Дата обновления')
    
    class Meta:
        verbose_name = 'Пользователь'
        verbose_name_plural = 'Пользователи'
        ordering = ['-created_at']
    
    def __str__(self):
        return f"{self.name} ({self.email})"
    
    def get_absolute_url(self):
        return reverse('user_detail', kwargs={'pk': self.pk})

class Post(models.Model):
    """Модель поста блога"""
    title = models.CharField(max_length=200, verbose_name='Заголовок')
    content = models.TextField(verbose_name='Содержание')
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name='Автор')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = 'Пост'
        verbose_name_plural = 'Посты'
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        return reverse('post_detail', kwargs={'pk': self.pk})
```

### Представления Django (views.py)

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.views import View
from django.views.generic import ListView, DetailView, CreateView, UpdateView
from django.contrib import messages
from django.urls import reverse_lazy
from .models import User, Post
from .forms import UserForm, PostForm

class UserListView(ListView):
    """Список пользователей"""
    model = User
    template_name = 'myapp/user_list.html'
    context_object_name = 'users'
    paginate_by = 10

class UserDetailView(DetailView):
    """Детальная страница пользователя"""
    model = User
    template_name = 'myapp/user_detail.html'
    context_object_name = 'user'

class UserCreateView(CreateView):
    """Создание пользователя"""
    model = User
    form_class = UserForm
    template_name = 'myapp/user_form.html'
    success_url = reverse_lazy('user_list')
    
    def form_valid(self, form):
        messages.success(self.request, 'Пользователь успешно создан!')
        return super().form_valid(form)

class PostListView(ListView):
    """Список постов"""
    model = Post
    template_name = 'myapp/post_list.html'
    context_object_name = 'posts'
    paginate_by = 5
    
    def get_queryset(self):
        return Post.objects.select_related('author').filter(author__is_active=True)

def home(request):
    """Главная страница"""
    recent_posts = Post.objects.select_related('author').filter(
        author__is_active=True
    )[:3]
    
    users_count = User.objects.filter(is_active=True).count()
    posts_count = Post.objects.count()
    
    context = {
        'recent_posts': recent_posts,
        'users_count': users_count,
        'posts_count': posts_count,
    }
    
    return render(request, 'myapp/home.html', context)

class SearchView(View):
    """Поиск по пользователям и постам"""
    def get(self, request):
        query = request.GET.get('q', '')
        results = []
        
        if query:
            # Поиск по пользователям
            users = User.objects.filter(
                models.Q(name__icontains=query) | 
                models.Q(email__icontains=query),
                is_active=True
            )
            
            # Поиск по постам
            posts = Post.objects.filter(
                models.Q(title__icontains=query) | 
                models.Q(content__icontains=query)
            ).select_related('author')
            
            results = list(users) + list(posts)
        
        return render(request, 'myapp/search.html', {
            'query': query,
            'results': results
        })
```

### Формы Django (forms.py)

```python
from django import forms
from .models import User, Post

class UserForm(forms.ModelForm):
    """Форма для пользователя"""
    class Meta:
        model = User
        fields = ['name', 'email', 'is_active']
        widgets = {
            'name': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Введите имя'
            }),
            'email': forms.EmailInput(attrs={
                'class': 'form-control',
                'placeholder': 'Введите email'
            }),
            'is_active': forms.CheckboxInput(attrs={
                'class': 'form-check-input'
            })
        }
        labels = {
            'name': 'Полное имя',
            'email': 'Электронная почта',
            'is_active': 'Активный пользователь'
        }
    
    def clean_email(self):
        """Валидация email"""
        email = self.cleaned_data['email']
        if User.objects.filter(email=email).exists():
            if self.instance and self.instance.pk:
                # Редактирование существующего пользователя
                if User.objects.filter(email=email).exclude(pk=self.instance.pk).exists():
                    raise forms.ValidationError('Пользователь с таким email уже существует')
            else:
                # Создание нового пользователя
                raise forms.ValidationError('Пользователь с таким email уже существует')
        return email

class PostForm(forms.ModelForm):
    """Форма для поста"""
    class Meta:
        model = Post
        fields = ['title', 'content', 'author']
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Введите заголовок'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'placeholder': 'Введите содержание',
                'rows': 5
            }),
            'author': forms.Select(attrs={
                'class': 'form-control'
            })
        }
```

## 4. FastAPI: современный фреймворк для API

### Быстрый старт с FastAPI

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel, EmailStr
from typing import List, Optional
from datetime import datetime
import sqlite3
import os

# 🔍 Создание приложения
app = FastAPI(
    title="My API",
    description="Пример API на FastAPI",
    version="1.0.0"
)

# 🔍 Модели Pydantic
class UserBase(BaseModel):
    name: str
    email: EmailStr

class UserCreate(UserBase):
    pass

class User(UserBase):
    id: int
    created_at: datetime
    
    class Config:
        orm_mode = True

class PostBase(BaseModel):
    title: str
    content: str

class PostCreate(PostBase):
    author_id: int

class Post(PostBase):
    id: int
    author: User
    created_at: datetime
    
    class Config:
        orm_mode = True

# 🔍 База данных
def get_db():
    conn = sqlite3.connect('api_database.db')
    try:
        yield conn
    finally:
        conn.close()

def init_db():
    """Инициализация базы данных"""
    conn = sqlite3.connect('api_database.db')
    cursor = conn.cursor()
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS posts (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            content TEXT NOT NULL,
            author_id INTEGER NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (author_id) REFERENCES users (id)
        )
    ''')
    
    conn.commit()
    conn.close()

# 🔍 API endpoints
@app.get("/")
async def root():
    """Корневой endpoint"""
    return {"message": "Добро пожаловать в API!"}

@app.get("/users/", response_model=List[User])
async def get_users(db: sqlite3.Connection = Depends(get_db)):
    """Получить всех пользователей"""
    cursor = db.cursor()
    cursor.execute('SELECT * FROM users ORDER BY created_at DESC')
    users = cursor.fetchall()
    
    return [
        User(id=u[0], name=u[1], email=u[2], created_at=u[3])
        for u in users
    ]

@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int, db: sqlite3.Connection = Depends(get_db)):
    """Получить пользователя по ID"""
    cursor = db.cursor()
    cursor.execute('SELECT * FROM users WHERE id = ?', (user_id,))
    user = cursor.fetchone()
    
    if not user:
        raise HTTPException(status_code=404, detail="Пользователь не найден")
    
    return User(id=user[0], name=user[1], email=user[2], created_at=user[3])

@app.post("/users/", response_model=User)
async def create_user(user: UserCreate, db: sqlite3.Connection = Depends(get_db)):
    """Создать нового пользователя"""
    cursor = db.cursor()
    
    try:
        cursor.execute(
            'INSERT INTO users (name, email) VALUES (?, ?)',
            (user.name, user.email)
        )
        db.commit()
        
        user_id = cursor.lastrowid
        cursor.execute('SELECT * FROM users WHERE id = ?', (user_id,))
        new_user = cursor.fetchone()
        
        return User(
            id=new_user[0],
            name=new_user[1],
            email=new_user[2],
            created_at=new_user[3]
        )
    
    except sqlite3.IntegrityError:
        raise HTTPException(status_code=400, detail="Пользователь с таким email уже существует")

@app.get("/posts/", response_model=List[Post])
async def get_posts(db: sqlite3.Connection = Depends(get_db)):
    """Получить все посты с авторами"""
    cursor = db.cursor()
    cursor.execute('''
        SELECT p.*, u.id, u.name, u.email, u.created_at 
        FROM posts p 
        JOIN users u ON p.author_id = u.id 
        ORDER BY p.created_at DESC
    ''')
    posts = cursor.fetchall()
    
    result = []
    for post in posts:
        author = User(
            id=post[5],
            name=post[6],
            email=post[7],
            created_at=post[8]
        )
        result.append(Post(
            id=post[0],
            title=post[1],
            content=post[2],
            author=author,
            created_at=post[4]
        ))
    
    return result

# 🔍 Запуск приложения
if __name__ == "__main__":
    import uvicorn
    init_db()
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 5. Базы данных и ORM

### SQLAlchemy - популярная ORM для Python

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime, Text, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime

# 🔍 Создание базы данных
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    password_hash = Column(String(128), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # 🔍 Связь один-ко-многим
    posts = relationship('Post', back_populates='author')
    
    def __repr__(self):
        return f"<User(username='{self.username}', email='{self.email}')>"

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    author_id = Column(Integer, ForeignKey('users.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # 🔍 Обратная связь
    author = relationship('User', back_populates='posts')
    
    def __repr__(self):
        return f"<Post(title='{self.title}', author_id={self.author_id})>"

# 🔍 Инициализация базы данных
engine = create_engine('sqlite:///blog.db')
Base.metadata.create_all(engine)

Session = sessionmaker(bind=engine)

# 🔍 Пример использования
def create_user(username, email, password):
    session = Session()
    
    try:
        user = User(username=username, email=email, password_hash=password)
        session.add(user)
        session.commit()
        return user
    except Exception as e:
        session.rollback()
        raise e
    finally:
        session.close()

def get_user_posts(user_id):
    session = Session()
    
    try:
        user = session.query(User).filter(User.id == user_id).first()
        if user:
            return user.posts
        return []
    finally:
        session.close()
```

## 6. Деплой и хостинг

### Docker для развертывания

**Dockerfile:**
```dockerfile
# Используем официальный Python образ
FROM python:3.9-slim

# Устанавливаем зависимости системы
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Создаем директорию приложения
WORKDIR /app

# Копируем зависимости
COPY requirements.txt .

# Устанавливаем Python зависимости
RUN pip install --no-cache-dir -r requirements.txt

# Копируем код приложения
COPY . .

# Открываем порт
EXPOSE 8000

# Запускаем приложение
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydatabase
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=mydatabase
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 7. Лучшие практики и безопасность

### Безопасность веб-приложений

```python
from flask import Flask
from flask_wtf.csrf import CSRFProtect
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
import os

app = Flask(__name__)

# 🔐 Безопасность
app.config.update(
    SECRET_KEY=os.environ.get('SECRET_KEY', 'dev-key-change-in-production'),
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SECURE=True,  # Только HTTPS в production
    REMEMBER_COOKIE_HTTPONLY=True,
    REMEMBER_COOKIE_SECURE=True
)

# 🔒 CSRF защита
csrf = CSRFProtect(app)

# ⏱️ Ограничение запросов
limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/sensitive-data')
@limiter.limit("10 per minute")  # Особое ограничение для чувствительных данных
def sensitive_data():
    return {"data": "секретная информация"}

# 🔍 Валидация входных данных
from flask import request
import re

def validate_email(email):
    """Валидация email"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

@app.route('/register', methods=['POST'])
def register():
    email = request.form.get('email', '')
    
    if not validate_email(email):
        return {"error": "Неверный формат email"}, 400
    
    # Дальнейшая обработка...
```

Эта лекция покрывает все основные аспекты веб-разработки на Python - от простых приложений на Flask до сложных систем на Django и высокопроизводительных API на FastAPI.
