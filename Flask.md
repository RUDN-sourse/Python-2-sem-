## 🔥 Критически важные концепции Flask

### 1. **Базовая структура приложения**

```python
from flask import Flask

# 🚨 ВАЖНО: Создание экземпляра приложения
app = Flask(__name__)  # __name__ указывает на текущий модуль

# 🚨 ВАЖНО: Декоратор маршрута
@app.route('/')
def home():
    return 'Hello, World!'

# 🚨 ВАЖНО: Запуск приложения
if __name__ == '__main__':
    app.run(debug=True)  # debug=True только для разработки!
```

**ВАЖНО:**
- `__name__` нужен Flask для определения местоположения шаблонов и статических файлов
- Декоратор `@app.route` связывает URL с функцией
- `debug=True` включает автоматическую перезагрузку и детальные ошибки (НЕ использовать в production)

### 2. **Обработка HTTP методов**

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Обработка данных формы
        username = request.form['username']
        password = request.form['password']
        return f'Hello {username}!'
    
    # GET запрос - показать форму
    return '''
    <form method="post">
        <input type="text" name="username">
        <input type="password" name="password">
        <input type="submit" value="Login">
    </form>
    '''
```

**Ключевые моменты:**
- По умолчанию route обрабатывает только GET
- `methods` параметр определяет разрешенные HTTP методы
- `request.form` для данных POST запросов
- `request.args` для параметров URL (?key=value)

### 3. **Шаблоны Jinja2 - ОСНОВА**

**app.py:**
```python
from flask import render_template

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', 
                         username=name, 
                         is_admin=False,
                         fruits=['apple', 'banana', 'orange'])
```

**templates/user.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>User Page</title>
</head>
<body>
    <!-- 🚨 ВАЖНО: Переменные -->
    <h1>Hello, {{ username }}!</h1>
    
    <!-- 🚨 ВАЖНО: Условия -->
    {% if is_admin %}
        <p>You have admin privileges</p>
    {% else %}
        <p>Regular user</p>
    {% endif %}
    
    <!-- 🚨 ВАЖНО: Циклы -->
    <ul>
    {% for fruit in fruits %}
        <li>{{ fruit }}</li>
    {% endfor %}
    </ul>
    
    <!-- 🚨 ВАЖНО: Наследование шаблонов -->
    {% extends "base.html" %}
    {% block content %}
        This is specific content
    {% endblock %}
</body>
</html>
```

### 4. **Статические файлы**

```html
<!-- 🚨 ВАЖНО: Правильное подключение статических файлов -->
<link href="{{ url_for('static', filename='css/style.css') }}" rel="stylesheet">
<script src="{{ url_for('static', filename='js/script.js') }}"></script>
<img src="{{ url_for('static', filename='images/logo.png') }}" alt="Logo">

<!-- ❌ НЕПРАВИЛЬНО -->
<link href="/static/css/style.css" rel="stylesheet">
```

**Структура папок:**
```
myapp/
├── app.py
├── static/
│   ├── css/
│   ├── js/
│   └── images/
└── templates/
```

### 5. **Работа с формами и валидация**

```python
from flask import Flask, render_template, request, flash, redirect, url_for
from wtforms import Form, StringField, PasswordField, validators

class RegistrationForm(Form):
    username = StringField('Username', [
        validators.Length(min=4, max=25),
        validators.DataRequired()
    ])
    email = StringField('Email', [
        validators.Email(),
        validators.DataRequired()
    ])
    password = PasswordField('Password', [
        validators.DataRequired(),
        validators.EqualTo('confirm', message='Passwords must match')
    ])
    confirm = PasswordField('Repeat Password')

@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm(request.form)
    if request.method == 'POST' and form.validate():
        # 🚨 ВАЖНО: Обработка валидных данных
        flash('Registration successful!', 'success')
        return redirect(url_for('login'))
    
    # 🚨 ВАЖНО: Показ ошибок валидации
    return render_template('register.html', form=form)
```

### 6. **Сессии и аутентификация**

```python
from flask import session, redirect, url_for

app.secret_key = 'your-secret-key-here'  # 🚨 ОБЯЗАТЕЛЬНО!

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    if check_credentials(username, password):  # Ваша функция проверки
        session['user_id'] = username  # 🚨 Сохраняем в сессии
        flash('Logged in successfully!')
        return redirect(url_for('dashboard'))
    
    flash('Invalid credentials')
    return redirect(url_for('login_form'))

@app.route('/dashboard')
def dashboard():
    if 'user_id' not in session:  # 🚨 Проверка авторизации
        return redirect(url_for('login_form'))
    return f"Welcome {session['user_id']}!"

@app.route('/logout')
def logout():
    session.pop('user_id', None)  # 🚨 Очистка сессии
    return redirect(url_for('home'))
```

### 7. **Работа с базами данных**

```python
import sqlite3
from flask import g

def get_db():
    if 'db' not in g:
        g.db = sqlite3.connect('database.db')
        g.db.row_factory = sqlite3.Row  # 🚨 Для доступа к колонкам по имени
    return g.db

@app.teardown_appcontext
def close_db(error):
    if 'db' in g:
        g.db.close()  # 🚨 Важно закрывать соединение

@app.route('/users')
def users():
    db = get_db()
    users = db.execute('SELECT * FROM users').fetchall()
    return render_template('users.html', users=users)
```

### 8. **Обработка ошибок**

```python
@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'), 404

@app.errorhandler(500)
def internal_error(error):
    return render_template('500.html'), 500

# 🚨 ВАЖНО: Обработка исключений
@app.errorhandler(Exception)
def handle_exception(error):
    # Логирование ошибки
    app.logger.error(f'Exception: {error}')
    return render_template('error.html'), 500
```

## 🎯 Практические примеры

### Пример 1: Простой блог

```python
from datetime import datetime

class Post:
    def __init__(self, title, content):
        self.title = title
        self.content = content
        self.date = datetime.now()

posts = []

@app.route('/')
def index():
    return render_template('index.html', posts=posts)

@app.route('/add', methods=['POST'])
def add_post():
    title = request.form['title']
    content = request.form['content']
    posts.append(Post(title, content))
    flash('Post added!')
    return redirect(url_for('index'))
```

### Пример 2: REST API

```python
from flask import jsonify

books = [
    {'id': 1, 'title': 'Book 1', 'author': 'Author 1'},
    {'id': 2, 'title': 'Book 2', 'author': 'Author 2'}
]

@app.route('/api/books')
def api_books():
    return jsonify(books)

@app.route('/api/books/<int:book_id>')
def api_book(book_id):
    book = next((b for b in books if b['id'] == book_id), None)
    if book is None:
        return jsonify({'error': 'Book not found'}), 404
    return jsonify(book)
```

## ⚠️ Частые ошибки новичков

### 1. **Циклические импорты**

**❌ НЕПРАВИЛЬНО:**
```python
# app.py
from routes import bp
app.register_blueprint(bp)

# routes.py
from app import app  # ЦИКЛИЧЕСКИЙ ИМПОРТ!

@app.route('/')
def home():
    return 'Hello'
```

**✅ ПРАВИЛЬНО:**
```python
# app.py
from routes import bp
app.register_blueprint(bp)

# routes.py
from flask import Blueprint

bp = Blueprint('main', __name__)

@bp.route('/')
def home():
    return 'Hello'
```

### 2. **Небезопасная работа с файлами**

**❌ ОПАСНО:**
```python
@app.route('/download/<filename>')
def download(filename):
    return send_file(filename)  # 🚨 Может быть использовано для доступа к любым файлам!
```

**✅ БЕЗОПАСНО:**
```python
import os
from flask import abort

@app.route('/download/<filename>')
def download(filename):
    # Проверяем, что файл в разрешенной директории
    safe_path = os.path.join('uploads', filename)
    if not os.path.exists(safe_path):
        abort(404)
    return send_file(safe_path)
```

### 3. **Отсутствие валидации данных**

**❌ ОПАСНО:**
```python
@app.route('/search')
def search():
    query = request.args.get('q')
    # 🚨 SQL INJECTION!
    results = db.execute(f"SELECT * FROM posts WHERE title LIKE '%{query}%'")
```

**✅ БЕЗОПАСНО:**
```python
@app.route('/search')
def search():
    query = request.args.get('q', '')
    # 🚨 Параметризованные запросы
    results = db.execute(
        "SELECT * FROM posts WHERE title LIKE ?", 
        ('%' + query + '%',)
    )
```

## 🛠️ Must-have расширения Flask

```python
# requirements.txt
Flask==2.3.3
Flask-WTF==1.1.1        # Формы и CSRF защита
Flask-SQLAlchemy==3.0.5 # ORM для баз данных
Flask-Login==0.6.2      # Аутентификация
Flask-Mail==0.9.1       # Отправка email
Flask-Migrate==4.0.4    # Миграции базы данных
```
