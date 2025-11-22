# Лекция: Работа с базами данных в Python

## Введение

Сегодня мы подробно разберем работу с базами данных в Python. Мы рассмотрим:
- Подключение к различным СУБД
- Выполнение SQL-запросов
- Обработку результатов
- Лучшие практики и безопасность

## 1. Установка необходимых библиотек

```python
# Для работы с SQLite (встроена в Python)
import sqlite3

# Для работы с другими СУБД нужно установить драйверы:
# pip install psycopg2-binary  # для PostgreSQL
# pip install mysql-connector-python  # для MySQL
# pip install pyodbc  # для различных СУБД через ODBC
```

## 2. Подключение к базе данных

### SQLite (самый простой вариант)

```python
import sqlite3

# Подключаемся к базе данных
# Если файл не существует, он будет создан автоматически
connection = sqlite3.connect('my_database.db')

# Создаем курсор - объект для выполнения SQL-запросов
cursor = connection.cursor()

# ВАЖНО: Курсор нужен для взаимодействия с БД
# Он позволяет выполнять запросы и получать результаты
```

### PostgreSQL (пример с psycopg2)

```python
import psycopg2

# Параметры подключения
connection = psycopg2.connect(
    host="localhost",
    database="my_database",
    user="my_user",
    password="my_password"
)
cursor = connection.cursor()
```

## 3. Создание таблицы

```python
# SQL-запрос для создания таблицы
create_table_query = """
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    email TEXT NOT NULL UNIQUE,
    age INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
"""

# Выполняем запрос
cursor.execute(create_table_query)

# Сохраняем изменения в базе данных
connection.commit()

"""
ОБЪЯСНЕНИЕ:
1. CREATE TABLE IF NOT EXISTS - создает таблицу, если она не существует
2. INTEGER PRIMARY KEY AUTOINCREMENT - автоматически увеличивающийся первичный ключ
3. TEXT NOT NULL - текстовое поле, которое не может быть пустым
4. UNIQUE - гарантирует уникальность значения
5. DEFAULT CURRENT_TIMESTAMP - автоматически устанавливает текущее время
6. connection.commit() - подтверждает транзакцию
"""
```

## 4. Вставка данных

### Простая вставка

```python
# SQL-запрос для вставки данных
insert_query = """
INSERT INTO users (username, email, age) 
VALUES (?, ?, ?);
"""

# Данные для вставки
user_data = ('john_doe', 'john@example.com', 25)

# Выполняем запрос с параметрами
cursor.execute(insert_query, user_data)

# Сохраняем изменения
connection.commit()

"""
ВАЖНЫЕ МОМЕНТЫ:
1. Используем параметризованные запросы (?, ?, ?) для безопасности
2. Это защищает от SQL-инъекций
3. Не подставляйте данные напрямую в строку запроса!
"""
```

### Множественная вставка

```python
# Несколько пользователей для вставки
users_data = [
    ('alice', 'alice@example.com', 30),
    ('bob', 'bob@example.com', 35),
    ('charlie', 'charlie@example.com', 28)
]

insert_many_query = """
INSERT INTO users (username, email, age) 
VALUES (?, ?, ?);
"""

# executemany для вставки нескольких записей
cursor.executemany(insert_many_query, users_data)
connection.commit()
```

## 5. Выборка данных

### Простая выборка

```python
# Выбираем всех пользователей
select_query = "SELECT * FROM users;"
cursor.execute(select_query)

# Получаем все результаты
users = cursor.fetchall()

print("Все пользователи:")
for user in users:
    print(user)

"""
ОБЪЯСНЕНИЕ:
1. fetchall() - возвращает все строки результата в виде списка кортежей
2. Каждый кортеж представляет одну строку таблицы
3. Порядок полей соответствует порядку в SELECT запросе
"""
```

### Выборка с условием

```python
# Выбираем пользователей старше 25 лет
select_conditional_query = """
SELECT username, email, age 
FROM users 
WHERE age > ?;
"""

cursor.execute(select_conditional_query, (25,))
users_over_25 = cursor.fetchall()

print("Пользователи старше 25 лет:")
for user in users_over_25:
    print(f"Имя: {user[0]}, Email: {user[1]}, Возраст: {user[2]}")
```

### Различные методы получения данных

```python
cursor.execute("SELECT * FROM users;")

# fetchone() - получает одну строку
first_user = cursor.fetchone()
print("Первый пользователь:", first_user)

# fetchmany(n) - получает n строк
next_users = cursor.fetchmany(2)
print("Следующие 2 пользователя:", next_users)

# fetchall() - получает все оставшиеся строки
remaining_users = cursor.fetchall()
print("Оставшиеся пользователи:", remaining_users)
```

## 6. Контекстный менеджер для автоматического управления соединением

```python
# Рекомендуемый способ работы с базой данных
with sqlite3.connect('my_database.db') as conn:
    cursor = conn.cursor()
    
    # Выполняем запросы внутри блока with
    cursor.execute("SELECT * FROM users;")
    results = cursor.fetchall()
    
    # Соединение автоматически закроется при выходе из блока
    # Не нужно явно вызывать conn.close()

"""
ПРЕИМУЩЕСТВА:
1. Автоматическое закрытие соединения
2. Автоматический коммит при успешном выполнении
3. Автоматический откат при возникновении ошибки
"""
```

## 7. Обработка ошибок

```python
try:
    with sqlite3.connect('my_database.db') as conn:
        cursor = conn.cursor()
        
        # Попытка вставить дубликат username (нарушение UNIQUE)
        cursor.execute(
            "INSERT INTO users (username, email, age) VALUES (?, ?, ?);",
            ('john_doe', 'another_email@example.com', 40)  # username уже существует
        )
        
except sqlite3.IntegrityError as e:
    print(f"Ошибка целостности данных: {e}")
    
except sqlite3.Error as e:
    print(f"Ошибка базы данных: {e}")
    
except Exception as e:
    print(f"Общая ошибка: {e}")
```

## 8. Использование словарей вместо кортежей

```python
# Настраиваем соединение для возврата словарей
connection = sqlite3.connect('my_database.db')
connection.row_factory = sqlite3.Row  # Теперь строки будут вести себя как словари

cursor = connection.cursor()
cursor.execute("SELECT * FROM users WHERE id = ?;", (1,))
user = cursor.fetchone()

# Теперь можно обращаться к полям по имени
print(f"ID: {user['id']}")
print(f"Username: {user['username']}")
print(f"Email: {user['email']}")

# Или преобразовать в настоящий словарь
user_dict = dict(user)
print(user_dict)
```

## 9. Транзакции

```python
try:
    with sqlite3.connect('my_database.db') as conn:
        cursor = conn.cursor()
        
        # Начинаем транзакцию (в SQLite автоматически)
        cursor.execute("INSERT INTO users (username, email, age) VALUES (?, ?, ?);", 
                      ('user1', 'user1@example.com', 25))
        
        cursor.execute("INSERT INTO users (username, email, age) VALUES (?, ?, ?);", 
                      ('user2', 'user2@example.com', 30))
        
        # Если все операции успешны, изменения сохраняются автоматически
        
except sqlite3.Error:
    # При ошибке автоматически выполняется rollback
    print("Произошла ошибка, изменения отменены")
```

## 10. Создание класса-обертки для работы с БД

```python
class DatabaseManager:
    def __init__(self, db_path):
        self.db_path = db_path
    
    def __enter__(self):
        """Вызывается при входе в блок with"""
        self.connection = sqlite3.connect(self.db_path)
        self.connection.row_factory = sqlite3.Row  # Для работы со словарями
        self.cursor = self.connection.cursor()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Вызывается при выходе из блока with"""
        if exc_type is None:
            self.connection.commit()  # Коммит если нет ошибок
        else:
            self.connection.rollback()  # Откат при ошибке
        self.connection.close()
    
    def execute_query(self, query, params=None):
        """Выполняет SQL-запрос"""
        if params is None:
            self.cursor.execute(query)
        else:
            self.cursor.execute(query, params)
        return self.cursor
    
    def fetch_all(self, query, params=None):
        """Выполняет запрос и возвращает все результаты"""
        cursor = self.execute_query(query, params)
        return cursor.fetchall()
    
    def fetch_one(self, query, params=None):
        """Выполняет запрос и возвращает одну строку"""
        cursor = self.execute_query(query, params)
        return cursor.fetchone()

# Использование класса
with DatabaseManager('my_database.db') as db:
    users = db.fetch_all("SELECT * FROM users;")
    for user in users:
        print(dict(user))
```

## 11. Полный пример приложения

```python
import sqlite3
from datetime import datetime

class UserManager:
    def __init__(self, db_path):
        self.db_path = db_path
        self.init_database()
    
    def init_database(self):
        """Инициализирует базу данных и создает таблицы"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            
            create_table_query = """
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT NOT NULL UNIQUE,
                email TEXT NOT NULL UNIQUE,
                age INTEGER,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
            """
            cursor.execute(create_table_query)
    
    def add_user(self, username, email, age):
        """Добавляет нового пользователя"""
        try:
            with sqlite3.connect(self.db_path) as conn:
                cursor = conn.cursor()
                cursor.execute(
                    "INSERT INTO users (username, email, age) VALUES (?, ?, ?);",
                    (username, email, age)
                )
            return True
        except sqlite3.IntegrityError:
            print("Ошибка: пользователь с таким username или email уже существует")
            return False
    
    def get_all_users(self):
        """Возвращает всех пользователей"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM users ORDER BY created_at DESC;")
            return [dict(row) for row in cursor.fetchall()]
    
    def get_user_by_username(self, username):
        """Находит пользователя по username"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM users WHERE username = ?;", (username,))
            result = cursor.fetchone()
            return dict(result) if result else None
    
    def update_user_age(self, username, new_age):
        """Обновляет возраст пользователя"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            cursor.execute(
                "UPDATE users SET age = ? WHERE username = ?;",
                (new_age, username)
            )
            return cursor.rowcount > 0  # True если пользователь обновлен
    
    def delete_user(self, username):
        """Удаляет пользователя"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            cursor.execute("DELETE FROM users WHERE username = ?;", (username,))
            return cursor.rowcount > 0  # True если пользователь удален

# Демонстрация работы
if __name__ == "__main__":
    user_manager = UserManager('test_database.db')
    
    # Добавляем пользователей
    user_manager.add_user("alice", "alice@example.com", 25)
    user_manager.add_user("bob", "bob@example.com", 30)
    
    # Получаем всех пользователей
    users = user_manager.get_all_users()
    print("Все пользователи:")
    for user in users:
        print(f"  {user['username']} - {user['email']} - {user['age']} лет")
    
    # Ищем конкретного пользователя
    user = user_manager.get_user_by_username("alice")
    if user:
        print(f"\nНайден пользователь: {user['username']}")
    
    # Обновляем возраст
    user_manager.update_user_age("bob", 31)
    
    # Показываем обновленные данные
    print("\nПосле обновления:")
    users = user_manager.get_all_users()
    for user in users:
        print(f"  {user['username']} - {user['age']} лет")
```

## 12. Лучшие практики и важные замечания

### Безопасность
```python
# НЕПРАВИЛЬНО - уязвимо для SQL-инъекций
username = "admin'; DROP TABLE users; --"
cursor.execute(f"SELECT * FROM users WHERE username = '{username}'")

# ПРАВИЛЬНО - безопасно
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
```

### Управление соединениями
- Всегда закрывайте соединения (используйте context managers)
- Не держите соединения открытыми долго
- Используйте пулы соединений для веб-приложений

### Обработка ошибок
- Всегда обрабатывайте исключения базы данных
- Используйте конкретные типы исключений (IntegrityError, OperationalError)
- Логируйте ошибки для отладки

## Заключение

Мы рассмотрели основные аспекты работы с базами данных в Python:
- Подключение к различным СУБД
- Выполнение CRUD-операций (Create, Read, Update, Delete)
- Безопасную работу с параметризованными запросами
- Обработку ошибок и управление транзакциями
- Создание абстракций для удобной работы

Ключевые принципы:
1. **Безопасность** - всегда используйте параметризованные запросы
2. **Надежность** - обрабатывайте ошибки и используйте транзакции
3. **Эффективность** - правильно управляйте соединениями
4. **Читаемость** - создавайте понятные абстракции

Эти знания помогут вам создавать надежные и безопасные приложения, работающие с базами данных!
