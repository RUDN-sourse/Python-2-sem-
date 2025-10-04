# Детальная лекция по Flask и веб-разработке для студентов 2 курса

## 🎯 Введение: Почему мы изучаем именно Flask?

**Flask** был выбран для нашего курса не случайно. Давайте разберемся почему:

### Сравнение с другими фреймворками:

**Django** - это "фреймворк с батарейками":
- Много встроенных компонентов (админка, ORM, аутентификация)
- Жесткая структура проекта
- Больше магии - меньше понимания что происходит "под капотом"

**Flask** - микрофреймворк:
- Только самое необходимое
- Гибкая структура - вы сами решаете как организовать код
- Понятная архитектура - видно как все работает

**Аналогия**: 
- Django - это готовый конструктор с инструкцией
- Flask - это набор деталей, из которых вы сами собираете конструктор

---

## 🌐 Часть 1: Основы веб-разработки

### Что происходит когда вы вводите URL в браузере?

Давайте разберем этот процесс по шагам:

1. **DNS запрос** - браузер узнает IP адрес сервера
2. **TCP соединение** - устанавливается соединение с сервером
3. **HTTP запрос** - браузер отправляет запрос:
```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...
Accept: text/html
```
4. **Сервер обрабатывает запрос** - наша Flask-программа
5. **HTTP ответ** - сервер возвращает данные:
```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256

<!DOCTYPE html><html>...
```
6. **Браузер рендерит страницу** - показывает вам результат

### Почему HTTP так важен?

**HTTP (HyperText Transfer Protocol)** - это язык общения между браузером и сервером. Представьте что это диалог:

**Браузер**: "Привет! Дай мне главную страницу" (GET запрос)
**Сервер**: "Вот, держи HTML код" (200 OK ответ)

**Браузер**: "Хочу зарегистрироваться, вот мои данные" (POST запрос)  
**Сервер**: "Ок, создал пользователя" (201 Created ответ)

**Браузер**: "Дай страницу которой нет"  
**Сервер**: "Такой страницы нет" (404 Not Found)

### Методы HTTP - что они означают на практике:

- **GET** - "посмотреть что-то" (безопасная операция)
- **POST** - "создать что-то новое" 
- **PUT** - "полностью обновить что-то"
- **PATCH** - "частично обновить что-то"
- **DELETE** - "удалить что-то"

**Важное правило**: GET запросы не должны изменять данные на сервере!

---

## 🔧 Часть 2: Первое Flask приложение

### Минимальное приложение - разберем каждую строку:

```python
from flask import Flask

app = Flask(__name__)
# Что здесь происходит?
# 1. Импортируем класс Flask
# 2. Создаем экземпляр приложения
# 3. __name__ - это имя текущего модуля
#    Flask использует это чтобы найти шаблоны и статические файлы

@app.route('/')
def hello():
    return 'Hello, World!'

# Декоратор @app.route - это магия Python!
# По сути это функция, которая принимает другую функцию
# и добавляет ей новое поведение

if __name__ == '__main__':
    app.run(debug=True)
    # debug=True включает:
    # - Автоперезагрузку при изменении кода
    # - Детальные ошибки в браузере
    # - Отладочный режим (НИКОГДА не использовать в продакшене!)
```

### Что такое декоратор? Простое объяснение:

```python
# Без декоратора:
def hello():
    return 'Hello, World!'

hello = app.route('/')(hello)

# С декоратором (то же самое, но красивее):
@app.route('/')
def hello():
    return 'Hello, World!'
```

Декоратор - это функция, которая "оборачивает" другую функцию, добавляя ей новую функциональность.

---

## 🗺️ Часть 3: Маршрутизация - "Дорожная карта" приложения

### Зачем нужны маршруты?

Представьте что ваше приложение - это город, а маршруты - это адреса домов:

- `/` - главная улица
- `/about` - улица "О нас"  
- `/users/john` - дом пользователя John

### Динамические маршруты - мощный инструмент:

```python
@app.route('/user/<username>')
def show_user(username):
    return f'Пользователь: {username}'

# Что здесь происходит?
# 1. Браузер запрашивает /user/alice
# 2. Flask видит шаблон /user/<username>
# 3. Извлекает 'alice' из URL
# 4. Передает как аргумент в функцию show_user('alice')
# 5. Функция возвращает 'Пользователь: alice'
```

### Конвертеры типов - валидация на уровне URL:

```python
@app.route('/post/<int:post_id>')
def show_post(post_id):
    # post_id гарантированно будет числом!
    # Если в URL будет /post/abc - получим 404 ошибку
    return f'Пост #{post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # path принимает строку со слешами
    # /path/foo/bar → subpath = 'foo/bar'
    return f'Путь: {subpath}'
```

### Методы HTTP в действии:

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Обрабатываем данные формы
        username = request.form['username']
        password = request.form['password']
        return f'Вход для {username}'
    else:
        # Показываем HTML форму
        return '''
            <form method="post">
                <input type="text" name="username">
                <input type="password" name="password">
                <input type="submit">
            </form>
        '''
```

А почему мы проверяем метод через if/else, а не создаем две отдельные функции?

Потому что это один логический endpoint (точка входа), который обрабатывает два сценария:
- Показать форму (GET)
- Обработать данные формы (POST)

---

## 📝 Часть 4: Шаблоны Jinja2 - "Умный HTML"

### Проблема которую решают шаблоны:

**Без шаблонов** (плохой подход):
```python
@app.route('/user')
def show_user():
    user = {'name': 'Alice', 'age': 25}
    html = f"""
    <html>
        <head><title>Профиль</title></head>
        <body>
            <h1>Привет, {user['name']}!</h1>
            <p>Тебе {user['age']} лет</p>
        </body>
    </html>
    """
    return html
```

**Проблемы**:
- HTML смешан с Python
- Сложно поддерживать
- Легко сделать ошибку безопасности

**С шаблонами** (правильный подход):
```python
from flask import render_template

@app.route('/user')
def show_user():
    user = {'name': 'Alice', 'age': 25}
    return render_template('user.html', user=user)
```

```html
<!-- templates/user.html -->
<html>
<head><title>Профиль</title></head>
<body>
    <h1>Привет, {{ user.name }}!</h1>
    <p>Тебе {{ user.age }} лет</p>
</body>
</html>
```

### Наследование шаблонов - принцип DRY (Don't Repeat Yourself)

**Без наследования** (дублирование кода):
```html
<!-- page1.html -->
<html><head><title>Страница 1</title></head>
<body><nav>Навигация</nav><h1>Контент 1</h1></body></html>

<!-- page2.html -->
<html><head><title>Страница 2</title></head>
<body><nav>Навигация</nav><h1>Контент 2</h1></body></html>
```

**С наследованием** (правильно):
```html
<!-- base.html -->
<html>
<head><title>{% block title %}Мой сайт{% endblock %}</title></head>
<body>
    <nav>Навигация</nav>
    <main>{% block content %}{% endblock %}</main>
    <footer>Футер</footer>
</body>
</html>

<!-- page1.html -->
{% extends "base.html" %}

{% block title %}Страница 1{% endblock %}

{% block content %}
    <h1>Контент 1</h1>
{% endblock %}
```
А что если я хочу добавить дополнительный CSS на одной странице?

Используйте дополнительные блоки!
```html
<!-- base.html -->
<head>
    {% block styles %}{% endblock %}
</head>

<!-- page1.html -->
{% block styles %}
    <style>.special { color: red; }</style>
{% endblock %}
```

### Фильтры Jinja2 - преобразование данных:

```html
<!-- Без фильтров -->
<p>{{ user.bio }}</p>  # Может быть None или пустая строка

<!-- С фильтрами -->
<p>{{ user.bio|default('Биография не указана') }}</p>
<p>Цена: {{ product.price|float|round(2) }}</p>
<p>Дата: {{ post.date|datetimeformat }}</p>
```

**Важный момент безопасности**:
```html
<!-- ОПАСНО если user_content содержит JavaScript -->
<p>{{ user_content }}</p>

<!-- БЕЗОПАСНО - HTML экранируется -->
<p>{{ user_content }}</p>

<!-- ОПАСНО - отключаем экранирование -->
<p>{{ user_content|safe }}</p>
```

**Никогда не используйте |safe с пользовательским вводом!**

---

## 📋 Часть 5: Формы - сбор и валидация данных

### Почему формы - это сложно?

1. **Валидация** - проверка что данные корректны
2. **Безопасность** - защита от CSRF, XSS
3. **UX** - понятные сообщения об ошибках  
4. **Сохранение состояния** - не терять введенные данные

### Ручная обработка форм (чтобы понять проблему):

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
    errors = {}
    
    if request.method == 'POST':
        username = request.form.get('username', '')
        email = request.form.get('email', '')
        password = request.form.get('password', '')
        
        # Валидация
        if not username:
            errors['username'] = 'Имя обязательно'
        elif len(username) < 3:
            errors['username'] = 'Имя слишком короткое'
            
        if not email or '@' not in email:
            errors['email'] = 'Некорректный email'
            
        # И так для каждого поля...
        
        if not errors:
            # Создать пользователя
            return redirect('/success')
    
    # Показать форму с ошибками
    return render_template('register.html', errors=errors)
```

**Проблемы**:
- Много повторяющегося кода
- Легко забыть проверить какое-то поле
- Сложно поддерживать

### Автоматическая обработка с Flask-WTF:

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField
from wtforms.validators import DataRequired, Email, Length

class RegistrationForm(FlaskForm):
    username = StringField('Имя пользователя', validators=[
        DataRequired(message='Обязательное поле'),
        Length(min=3, max=25, message='Имя должно быть от 3 до 25 символов')
    ])
    
    email = StringField('Email', validators=[
        DataRequired(),
        Email(message='Некорректный email')
    ])
    
    password = PasswordField('Пароль', validators=[
        DataRequired(),
        Length(min=6, message='Пароль должен быть не менее 6 символов')
    ])
```

**Что дает Flask-WTF**:
- Автоматическая валидация
- CSRF защита
- Генерация HTML
- Сообщения об ошибках

### Обработка формы в представлении:

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm()
    
    if form.validate_on_submit():
        # Все данные прошли валидацию!
        user = User(
            username=form.username.data,
            email=form.email.data
        )
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        
        flash('Регистрация успешна!', 'success')
        return redirect(url_for('login'))
    
    # Если GET запрос или валидация не пройдена
    return render_template('register.html', form=form)
```

Что делает form.validate_on_submit()?

Это комбинация двух проверок:
1. `request.method == 'POST'` - запрос отправлен формой
2. `form.validate()` - все данные прошли валидацию

---

## 💾 Часть 6: Базы данных и ORM

### Почему не работаем с SQL напрямую?

**Прямые SQL запросы**:
```python
# Уязвимо к SQL-инъекциям!
query = f"SELECT * FROM users WHERE username = '{username}'"

# Безопасно, но неудобно
query = "SELECT * FROM users WHERE username = ?"
cursor.execute(query, (username,))
```

**Проблемы**:
- SQL-инъекции
- Разный синтаксис для разных СУБД
- Сложно поддерживать
- Нет проверки типов

### ORM (Object-Relational Mapping) подход:

```python
# Вместо SQL работаем с объектами Python
user = User.query.filter_by(username=username).first()
```

**Преимущества**:
- Автоматическое экранирование
- Единый API для разных СУБД
- Проверка типов
- Легко читается

### Модели в Flask-SQLAlchemy:

```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    # Таблица users будет создана автоматически
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Связь один-ко-многим
    posts = db.relationship('Post', backref='author', lazy=True)
    
    def __repr__(self):
        return f'<User {self.username}>'

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
```

**Разберем каждую часть**:

- `db.Column` - определение столбца
- `db.Integer`, `db.String` - типы данных
- `primary_key=True` - первичный ключ
- `unique=True` - уникальное значение
- `nullable=False` - не может быть NULL
- `default=datetime.utcnow` - значение по умолчанию

### Отношения между моделями:

**Один-ко-многим** (User → Posts):
```python
class User(db.Model):
    posts = db.relationship('Post', backref='author', lazy=True)

class Post(db.Model): 
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
```

**Как использовать**:
```python
user = User.query.get(1)
posts = user.posts  # Все посты пользователя

post = Post.query.get(1)
author = post.author  # Автор поста (User объект)
```

**Многие-ко-многим** (Students ←→ Courses):
```python
# Промежуточная таблица
registrations = db.Table('registrations',
    db.Column('student_id', db.Integer, db.ForeignKey('student.id')),
    db.Column('course_id', db.Integer, db.ForeignKey('course.id'))
)

class Student(db.Model):
    courses = db.relationship('Course', secondary=registrations, backref='students')

class Course(db.Model):
    pass
```

### Запросы к базе данных:

**CRUD операции**:

**Create** (Создание):
```python
user = User(username='john', email='john@example.com')
db.session.add(user)
db.session.commit()  # Не забываем коммитить!
```

**Read** (Чтение):
```python
# Получить все записи
users = User.query.all()

# Получить по ID
user = User.query.get(1)

# Фильтрация
admin = User.query.filter_by(username='admin').first()
johns = User.query.filter(User.username.like('john%')).all()

# Сортировка
users = User.query.order_by(User.username).all()

# Пагинация
page = request.args.get('page', 1, type=int)
users = User.query.paginate(page=page, per_page=10)
```

**Update** (Обновление):
```python
user = User.query.get(1)
user.email = 'new@example.com'
db.session.commit()
```

**Delete** (Удаление):
```python
user = User.query.get(1)
db.session.delete(user)
db.session.commit()
```
Почему нужно вызывать db.session.commit()?

SQLAlchemy использует транзакции. Все изменения накапливаются в сессии и применяются к базе данных только при вызове commit(). Это обеспечивает целостность данных.

---

## 🔐 Часть 7: Аутентификация и авторизация

### Различие понятий:

- **Аутентификация** - "Кто вы?" (логин/пароль)
- **Авторизация** - "Что вам разрешено?" (права доступа)

### Flask-Login - управление сессиями:

```python
from flask_login import LoginManager, UserMixin, login_user, logout_user

login_manager = LoginManager()
login_manager.login_view = 'login'  # Куда перенаправлять неавторизованных

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

class User(UserMixin, db.Model):
    # UserMixin добавляет методы:
    # is_authenticated, is_active, is_anonymous, get_id
    pass
```

**Что делает user_loader?**
Когда пользователь возвращается на сайт, Flask-Login по ID из сессии находит объект пользователя в базе.

### Хеширование паролей - критически важно!

**НЕПРАВИЛЬНО** (никогда так не делайте):
```python
class User(db.Model):
    password = db.Column(db.String(128))  # Пароль в открытом виде!
```

**ПРАВИЛЬНО**:
```python
from werkzeug.security import generate_password_hash, check_password_hash

class User(db.Model):
    password_hash = db.Column(db.String(128))
    
    def set_password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```

**Почему хеширование важно?**
- Если база данных украдена, пароли нельзя восстановить
- Хеш нельзя преобразовать обратно в пароль
- Можно только проверить совпадение

### Полный процесс регистрации и входа:

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(username=form.username.data, email=form.email.data)
        user.set_password(form.password.data)
        
        db.session.add(user)
        db.session.commit()
        
        flash('Регистрация успешна!', 'success')
        return redirect(url_for('login'))
    
    return render_template('register.html', form=form)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        
        if user and user.check_password(form.password.data):
            login_user(user, remember=form.remember.data)
            
            # Перенаправление на запрошенную страницу
            next_page = request.args.get('next')
            return redirect(next_page) if next_page else redirect(url_for('index'))
        else:
            flash('Неверный email или пароль', 'danger')
    
    return render_template('login.html', form=form)

@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('index'))
```

### Защита маршрутов:

```python
from flask_login import login_required

@app.route('/dashboard')
@login_required  # Только для авторизованных
def dashboard():
    return render_template('dashboard.html')

# Кастомная проверка прав
def admin_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not current_user.is_admin:
            abort(403)  # Forbidden
        return f(*args, **kwargs)
    return decorated_function

@app.route('/admin')
@login_required
@admin_required  # Только для админов
def admin_panel():
    return render_template('admin.html')
```

---

## 🔌 Часть 8: REST API

### Что такое REST API?

**REST (Representational State Transfer)** - архитектурный стиль для создания веб-сервисов.

**Принципы REST**:
- Stateless (без состояния)
- URL как ресурсы (/users, /posts)
- HTTP методы как действия (GET, POST, PUT, DELETE)
- JSON как формат данных

### Простое API на чистом Flask:

```python
from flask import jsonify, request

@app.route('/api/users', methods=['GET'])
def get_users():
    users = User.query.all()
    return jsonify([user.to_dict() for user in users])

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    
    # Валидация
    if not data or not 'username' in data:
        return jsonify({'error': 'Username required'}), 400
    
    user = User(username=data['username'], email=data.get('email', ''))
    db.session.add(user)
    db.session.commit()
    
    return jsonify(user.to_dict()), 201  # 201 Created

# Добавляем метод to_dict в модель
class User(db.Model):
    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email
        }
```

### Коды состояния HTTP для API:

- **200 OK** - успешный запрос
- **201 Created** - ресурс создан
- **400 Bad Request** - неверные данные
- **401 Unauthorized** - не авторизован
- **403 Forbidden** - нет прав
- **404 Not Found** - ресурс не найден
- **500 Internal Server Error** - ошибка сервера

---

## 🏗️ Часть 9: Структура проекта

### Почему структура важна?

**Плохая структура** (все в одном файле):
```
app.py  # 1000+ строк кода!
```

**Проблемы**:
- Сложно найти код
- Конфликты при работе в команде
- Сложно тестировать
- Невозможно масштабировать

### Хорошая структура проекта:

```
myapp/
├── app/
│   ├── __init__.py          # Фабрика приложения
│   ├── models.py            # Модели базы данных
│   ├── routes/              # Маршруты (контроллеры)
│   │   ├── __init__.py
│   │   ├── main.py          # Главные маршруты
│   │   ├── auth.py          # Аутентификация
│   │   └── api.py           # API endpoints
│   ├── templates/           # Шаблоны
│   │   ├── base.html
│   │   ├── index.html
│   │   └── auth/
│   │       ├── login.html
│   │       └── register.html
│   ├── static/              # Статические файлы
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   └── utils/               # Вспомогательные функции
│       └── helpers.py
├── migrations/              # Миграции базы данных
├── tests/                   # Тесты
│   ├── test_models.py
│   ├── test_routes.py
│   └── conftest.py
├── config.py               # Конфигурация
├── requirements.txt        # Зависимости
└── run.py                 # Точка входа
```

### Фабрика приложений - почему это важно?

```python
# app/__init__.py
from flask import Flask
from config import Config

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    # Инициализация расширений
    db.init_app(app)
    login_manager.init_app(app)
    
    # Регистрация blueprint'ов
    from app.routes.main import bp as main_bp
    app.register_blueprint(main_bp)
    
    from app.routes.auth import bp as auth_bp
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    return app
```

**Преимущества фабрики**:
- Создание нескольких экземпляров приложения
- Разная конфигурация для тестирования/разработки
- Легкое тестирование

### Blueprints - модульность:

```python
# app/routes/auth.py
from flask import Blueprint, render_template, redirect, url_for, flash

bp = Blueprint('auth', __name__)

@bp.route('/login', methods=['GET', 'POST'])
def login():
    # Логика входа
    return render_template('auth/login.html')

@bp.route('/register', methods=['GET', 'POST'])
def register():
    # Логика регистрации
    return render_template('auth/register.html')
```

**Почему Blueprints?**
- Разделение кода по функциональности
- Префиксы URL (/auth/login, /auth/register)
- Собственные шаблоны и статические файлы

---

## 🛡️ Часть 10: Безопасность

### Основные угрозы и защита:

#### 1. SQL-инъекции
**Угроза**: 
```sql
-- Злоумышленник вводит в форму: ' OR '1'='1
SELECT * FROM users WHERE username = '' OR '1'='1'
```

**Защита** - используйте ORM!
```python
# БЕЗОПАСНО - SQLAlchemy экранирует данные
user = User.query.filter_by(username=username).first()
```

#### 2. XSS (Cross-Site Scripting)
**Угроза**: 
```html
<!-- Пользователь вводит в комментарий: -->
<script>alert('XSS')</script>
```

**Защита** - экранирование в шаблонах:
```html
<!-- БЕЗОПАСНО - Jinja2 экранирует по умолчанию -->
<p>{{ user_comment }}</p>  <!-- &lt;script&gt;alert('XSS')&lt;/script&gt; -->
```

#### 3. CSRF (Cross-Site Request Forgery)
**Угроза**: Злоумышленник заставляет пользователя выполнить действия без его ведома.

**Защита** - CSRF токены:
```html
<form method="post">
    {{ form.hidden_tag() }}  <!-- CSRF токен -->
    <!-- остальные поля -->
</form>
```

#### 4. Хеширование паролей
**НЕПРАВИЛЬНО**:
```python
user.password = password  # Пароль в открытом виде
```

**ПРАВИЛЬНО**:
```python
from werkzeug.security import generate_password_hash, check_password_hash

user.set_password(password)  # Пароль хешируется
user.check_password(password)  # Проверка хеша
```

---

## 🧪 Часть 11: Тестирование

### Зачем тестировать?

- Обнаружение ошибок на ранней стадии
- Гарантия что изменения не сломали существующий функционал
- Документация поведения системы
- Уверенность при рефакторинге

### Модульные тесты моделей:

```python
import unittest
from app import create_app, db
from app.models import User

class UserModelCase(unittest.TestCase):
    def setUp(self):
        # Создаем тестовое приложение
        self.app = create_app(TestConfig)
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()
    
    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
    
    def test_password_hashing(self):
        u = User(username='susan')
        u.set_password('cat')
        self.assertFalse(u.check_password('dog'))
        self.assertTrue(u.check_password('cat'))
    
    def test_user_creation(self):
        u = User(username='john', email='john@example.com')
        db.session.add(u)
        db.session.commit()
        self.assertEqual(User.query.count(), 1)
```

### Тестирование API:

```python
class APITestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app(TestConfig)
        self.client = self.app.test_client()
        db.create_all()
    
    def test_get_users(self):
        response = self.client.get('/api/users')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.content_type, 'application/json')
    
    def test_create_user(self):
        data = {
            'username': 'testuser',
            'email': 'test@example.com'
        }
        response = self.client.post('/api/users', 
                                  json=data,
                                  content_type='application/json')
        self.assertEqual(response.status_code, 201)
        
        # Проверяем что пользователь создан
        user = User.query.filter_by(username='testuser').first()
        self.assertIsNotNone(user)
```

---

## 🚀 Часть 12: Деплоймент в продакшен

### Отладка vs Продакшен:

**Разработка** (debug=True):
- Автоперезагрузка
- Детальные ошибки
- Отладочный режим

**Продакшен** (debug=False):
- Высокая производительность  
- Минимальная информация об ошибках
- Безопасность

### Конфигурация для продакшена:

```python
class ProductionConfig(Config):
    DEBUG = False
    TESTING = False
    
    # Используем переменные окружения
    SECRET_KEY = os.environ.get('SECRET_KEY')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    
    # Безопасные cookies
    SESSION_COOKIE_SECURE = True
    REMEMBER_COOKIE_SECURE = True
```

### WSGI сервер (Gunicorn):

**Почему не app.run()?**
- Один поток - медленно
- Нет оптимизаций
- Не предназначен для продакшена

**Gunicorn**:
```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 wsgi:app
```

### Обратный прокси (Nginx):

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /static {
        alias /path/to/your/app/static;
        expires 30d;  # Кэширование статических файлов
    }
}
```

### Деплой с Docker:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Копируем зависимости и устанавливаем их
COPY requirements.txt .
RUN pip install -r requirements.txt

# Копируем код приложения
COPY . .

# Переменные окружения
ENV FLASK_APP=wsgi:app
ENV FLASK_ENV=production

EXPOSE 5000

# Запускаем Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "wsgi:app"]
```

---

## 💡 Заключение: Ключевые концепции

### Что должен понимать каждый студент:

1. **HTTP** - основа веба, понимать запросы/ответы, методы, коды состояния
2. **MVC архитектура** - разделение данных, логики и представления  
3. **Шаблоны** - разделение Python и HTML, безопасность
4. **Формы** - валидация, CSRF защита, пользовательский опыт
5. **Базы данных** - ORM, миграции, отношения между таблицами
6. **Аутентификация** - сессии, хеширование паролей, защита маршрутов
7. **Безопасность** - SQL-инъекции, XSS, CSRF
8. **Тестирование** - модульные тесты, тестирование API
9. **Структура проекта** - модульность, поддерживаемость
10. **Деплоймент** - отличия разработки от продакшена

### Самые частые ошибки новичков:

1. **Смешивание логики и представления** - PHP-стиль в шаблонах
2. **Отсутствие валидации** - доверие пользовательскому вводу
3. **Пароли в открытом виде** - всегда хешируйте пароли!
4. **Debug mode в продакшене** - огромная дыра в безопасности
5. **Один большой файл** - сложно поддерживать и развивать
6. **Игнорирование ошибок** - всегда обрабатывайте исключения
7. **Отсутствие тестов** - "работает и ладно"

### Дальнейшее развитие:

- **Асинхронность** - Flask 2.0 с async/await
- **Микросервисы** - разделение приложения на сервисы
- **Кэширование** - Redis для ускорения работы
- **Очереди задач** - Celery для фоновых задач
- **Мониторинг** - логи, метрики, алерты
- **CI/CD** - автоматизация тестирования и деплоймента
