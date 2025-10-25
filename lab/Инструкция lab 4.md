# Подробное руководство по созданию Telegram ботов на Python

## Оглавление
1. [Основные понятия](#основные-понятия)
2. [Настройка окружения](#настройка-окружения)
3. [Создание первого бота](#создание-первого-бота)
4. [Типы обработчиков](#типы-обработчиков)
5. [Работа с состояниями](#работа-с-состояниями)
6. [База данных](#база-данных)
7. [Развертывание](#развертывание)
8. [Лучшие практики](#лучшие-практики)

---

## Основные понятия

### Что такое Telegram Bot API?
Telegram Bot API - это HTTP-интерфейс для создания ботов, который позволяет:
- Отправлять и получать сообщения
- Работать с клавиатурами
- Управлять группами и каналами
- Работать с файлами и медиа

### Ключевые компоненты бота:
- **Токен** - уникальный идентификатор бота
- **Webhook/Polling** - способы получения обновлений
- **Обработчики** - функции для обработки сообщений
- **Контекст** - объект для хранения данных между вызовами

---

## Настройка окружения

### 1. Установка Python и инструментов
```bash
# Проверяем установлен ли Python
python --version
# Python 3.8+

# Создаем виртуальное окружение
python -m venv bot_env
source bot_env/bin/activate  # Linux/Mac
# или
bot_env\Scripts\activate  # Windows
```

### 2. Установка библиотек
```bash
# Основная библиотека
pip install python-telegram-bot

# Дополнительные полезные библиотеки
pip install python-dotenv     # для переменных окружения
pip install requests          # для HTTP запросов
pip install sqlite3           # для базы данных
pip install aiohttp           # для асинхронных запросов
```

### 3. Создание структуры проекта
```
my_telegram_bot/
├── bot.py              # основной файл бота
├── config.py           # конфигурация
├── database.py         # работа с БД
├── handlers/           # обработчики
│   ├── __init__.py
│   ├── commands.py
│   └── messages.py
├── keyboards.py        # клавиатуры
├── requirements.txt    # зависимости
└── .env               # переменные окружения
```

---

## Создание первого бота

### Шаг 1: Получение токена
1. Найти @BotFather в Telegram
2. Отправить `/newbot`
3. Выбрать имя бота (например, "My Test Bot")
4. Выбрать username (оканчивается на `bot`, например, "my_test_bot")
5. Сохранить полученный токен

### Шаг 2: Базовый код бота
```python
# bot.py
import logging
import os
from telegram import Update
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    filters,
    ContextTypes
)

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

class TelegramBot:
    def __init__(self, token: str):
        self.application = Application.builder().token(token).build()
        self.setup_handlers()
    
    def setup_handlers(self):
        """Настройка обработчиков команд и сообщений"""
        
        # Обработчик команды /start
        self.application.add_handler(CommandHandler("start", self.start_command))
        
        # Обработчик команды /help
        self.application.add_handler(CommandHandler("help", self.help_command))
        
        # Обработчик текстовых сообщений
        self.application.add_handler(
            MessageHandler(filters.TEXT & ~filters.COMMAND, self.echo_message)
        )
        
        # Обработчик ошибок
        self.application.add_error_handler(self.error_handler)
    
    async def start_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработка команды /start"""
        user = update.effective_user
        welcome_text = f"""
Привет, {user.first_name}! 👋

Я демонстрационный бот. Вот что я умею:
/start - начать работу
/help - помощь
        """
        await update.message.reply_text(welcome_text)
    
    async def help_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработка команды /help"""
        help_text = """
📚 Доступные команды:

/start - Начать работу с ботом
/help - Получить справку

Просто отправь мне любое сообщение, и я его повторю!
        """
        await update.message.reply_text(help_text)
    
    async def echo_message(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Эхо-ответ на текстовые сообщения"""
        user_message = update.message.text
        response = f"Вы сказали: {user_message}"
        await update.message.reply_text(response)
    
    async def error_handler(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработка ошибок"""
        logger.error(f"Ошибка: {context.error}")
    
    def run(self):
        """Запуск бота"""
        print("Бот запущен...")
        self.application.run_polling()

if __name__ == "__main__":
    # Получаем токен из переменных окружения
    BOT_TOKEN = os.getenv("BOT_TOKEN")
    
    if not BOT_TOKEN:
        print("Ошибка: BOT_TOKEN не установлен!")
        exit(1)
    
    bot = TelegramBot(BOT_TOKEN)
    bot.run()
```

### Шаг 3: Настройка конфигурации
```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    """Класс для хранения конфигурации бота"""
    
    # Токен бота
    BOT_TOKEN = os.getenv("BOT_TOKEN")
    
    # ID администратора (опционально)
    ADMIN_ID = int(os.getenv("ADMIN_ID", 0))
    
    # Настройки базы данных
    DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///bot.db")
    
    # Настройки вебхука (для продакшена)
    WEBHOOK_URL = os.getenv("WEBHOOK_URL", "")
    WEBHOOK_PORT = int(os.getenv("WEBHOOK_PORT", 8443))
    
    @classmethod
    def validate(cls):
        """Проверка обязательных настроек"""
        if not cls.BOT_TOKEN:
            raise ValueError("BOT_TOKEN не установлен в переменных окружения")

# .env файл
BOT_TOKEN=your_actual_bot_token_here
ADMIN_ID=123456789
DATABASE_URL=sqlite:///bot.db
```

---

## Типы обработчиков

### 1. CommandHandler - обработка команд
```python
async def custom_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработка пользовательской команды"""
    await update.message.reply_text("Это пользовательская команда!")

# Регистрация
application.add_handler(CommandHandler("custom", custom_command))
```

### 2. MessageHandler - обработка сообщений
```python
# Только текст
text_handler = MessageHandler(filters.TEXT, text_handler_function)

# Только фото
photo_handler = MessageHandler(filters.PHOTO, photo_handler_function)

# Только документы
document_handler = MessageHandler(filters.DOCUMENT, document_handler_function)

# Комбинации фильтров
text_and_photo = MessageHandler(
    filters.TEXT | filters.PHOTO, 
    combined_handler
)
```

### 3. CallbackQueryHandler - обработка inline кнопок
```python
from telegram import InlineKeyboardButton, InlineKeyboardMarkup

async def show_options(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Показать inline клавиатуру"""
    keyboard = [
        [InlineKeyboardButton("Опция 1", callback_data="option_1")],
        [InlineKeyboardButton("Опция 2", callback_data="option_2")],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    
    await update.message.reply_text(
        "Выберите опцию:",
        reply_markup=reply_markup
    )

async def handle_button_click(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработка нажатия на inline кнопку"""
    query = update.callback_query
    await query.answer()  # Важно: подтверждаем получение callback
    
    if query.data == "option_1":
        await query.edit_message_text("Вы выбрали опцию 1!")
    elif query.data == "option_2":
        await query.edit_message_text("Вы выбрали опцию 2!")

# Регистрация обработчиков
application.add_handler(CommandHandler("options", show_options))
application.add_handler(CallbackQueryHandler(handle_button_click))
```

### 4. PollHandler - работа с опросами
```python
async def handle_poll(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработка опросов"""
    poll = update.poll
    logger.info(f"Получен опрос: {poll.question}")

application.add_handler(PollHandler(handle_poll))
```

---

## Работа с состояниями

### ConversationHandler для многошаговых сценариев
```python
from telegram.ext import ConversationHandler

# Определяем состояния
NAME, AGE, CITY = range(3)

class ConversationBot:
    def __init__(self, application):
        self.application = application
        self.setup_conversation()
    
    def setup_conversation(self):
        """Настройка многошагового диалога"""
        
        conv_handler = ConversationHandler(
            entry_points=[CommandHandler('register', self.start_registration)],
            states={
                NAME: [MessageHandler(filters.TEXT, self.get_name)],
                AGE: [MessageHandler(filters.TEXT, self.get_age)],
                CITY: [MessageHandler(filters.TEXT, self.get_city)],
            },
            fallbacks=[CommandHandler('cancel', self.cancel_registration)],
        )
        
        self.application.add_handler(conv_handler)
    
    async def start_registration(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Начало регистрации"""
        await update.message.reply_text(
            "Давайте зарегистрируем вас!\n"
            "Как вас зовут?"
        )
        return NAME
    
    async def get_name(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Получение имени"""
        context.user_data['name'] = update.message.text
        await update.message.reply_text("Сколько вам лет?")
        return AGE
    
    async def get_age(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Получение возраста"""
        try:
            age = int(update.message.text)
            context.user_data['age'] = age
            await update.message.reply_text("Из какого вы города?")
            return CITY
        except ValueError:
            await update.message.reply_text("Пожалуйста, введите число!")
            return AGE
    
    async def get_city(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Получение города и завершение регистрации"""
        context.user_data['city'] = update.message.text
        
        # Формируем итоговое сообщение
        name = context.user_data['name']
        age = context.user_data['age']
        city = context.user_data['city']
        
        summary = f"""
✅ Регистрация завершена!

📝 Ваши данные:
• Имя: {name}
• Возраст: {age}
• Город: {city}

Спасибо за регистрацию!
        """
        
        await update.message.reply_text(summary)
        
        # Очищаем данные
        context.user_data.clear()
        
        return ConversationHandler.END
    
    async def cancel_registration(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Отмена регистрации"""
        await update.message.reply_text("Регистрация отменена.")
        context.user_data.clear()
        return ConversationHandler.END
```

---

## База данных

### Работа с SQLite
```python
# database.py
import sqlite3
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

class DatabaseManager:
    def __init__(self, db_path: str = "bot_database.db"):
        self.db_path = db_path
        self.init_database()
    
    def get_connection(self):
        """Получение соединения с базой данных"""
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row  # Для доступа к колонкам по имени
        return conn
    
    def init_database(self):
        """Инициализация таблиц"""
        with self.get_connection() as conn:
            # Таблица пользователей
            conn.execute('''
                CREATE TABLE IF NOT EXISTS users (
                    user_id INTEGER PRIMARY KEY,
                    username TEXT,
                    first_name TEXT,
                    last_name TEXT,
                    language_code TEXT,
                    is_bot BOOLEAN,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')
            
            # Таблица сообщений
            conn.execute('''
                CREATE TABLE IF NOT EXISTS messages (
                    message_id INTEGER PRIMARY KEY AUTOINCREMENT,
                    user_id INTEGER,
                    text TEXT,
                    message_date TIMESTAMP,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (user_id) REFERENCES users (user_id)
                )
            ''')
            
            # Таблица пользовательских данных
            conn.execute('''
                CREATE TABLE IF NOT EXISTS user_data (
                    user_id INTEGER PRIMARY KEY,
                    age INTEGER,
                    city TEXT,
                    preferences TEXT,
                    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (user_id) REFERENCES users (user_id)
                )
            ''')
    
    def save_user(self, user):
        """Сохранение/обновление пользователя"""
        with self.get_connection() as conn:
            conn.execute('''
                INSERT OR REPLACE INTO users 
                (user_id, username, first_name, last_name, language_code, is_bot, last_activity)
                VALUES (?, ?, ?, ?, ?, ?, ?)
            ''', (
                user.id,
                user.username,
                user.first_name,
                user.last_name,
                user.language_code,
                user.is_bot,
                datetime.now()
            ))
    
    def save_message(self, user_id, text, message_date):
        """Сохранение сообщения"""
        with self.get_connection() as conn:
            conn.execute('''
                INSERT INTO messages (user_id, text, message_date)
                VALUES (?, ?, ?)
            ''', (user_id, text, message_date))
    
    def get_user_stats(self, user_id):
        """Получение статистики пользователя"""
        with self.get_connection() as conn:
            # Количество сообщений
            message_count = conn.execute(
                'SELECT COUNT(*) FROM messages WHERE user_id = ?',
                (user_id,)
            ).fetchone()[0]
            
            # Последняя активность
            last_activity = conn.execute(
                'SELECT last_activity FROM users WHERE user_id = ?',
                (user_id,)
            ).fetchone()[0]
            
            return {
                'message_count': message_count,
                'last_activity': last_activity
            }
```

### Интеграция базы данных в бота
```python
# bot_with_db.py
from database import DatabaseManager

class BotWithDatabase:
    def __init__(self, token: str):
        self.application = Application.builder().token(token).build()
        self.db = DatabaseManager()
        self.setup_handlers()
    
    async def start_with_db(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Команда /start с сохранением в БД"""
        user = update.effective_user
        
        # Сохраняем пользователя в БД
        self.db.save_user(user)
        
        # Получаем статистику
        stats = self.db.get_user_stats(user.id)
        
        welcome_text = f"""
Привет, {user.first_name}! 👋

📊 Ваша статистика:
• Сообщений отправлено: {stats['message_count']}
• Последняя активность: {stats['last_activity']}
        """
        
        await update.message.reply_text(welcome_text)
    
    async def save_user_message(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Сохранение сообщения пользователя"""
        user = update.effective_user
        message_text = update.message.text
        message_date = update.message.date
        
        # Сохраняем пользователя и сообщение
        self.db.save_user(user)
        self.db.save_message(user.id, message_text, message_date)
        
        await update.message.reply_text("Сообщение сохранено!")
```

---

## Развертывание

### 1. Локальный запуск (Polling)
```python
# Для разработки
if __name__ == "__main__":
    bot = TelegramBot("YOUR_TOKEN")
    bot.run()  # Использует polling
```

### 2. Развертывание на сервере (Webhook)
```python
# webhook_bot.py
import os
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters

class WebhookBot:
    def __init__(self, token: str, webhook_url: str):
        self.application = Application.builder().token(token).build()
        self.webhook_url = webhook_url
        self.setup_handlers()
    
    async def set_webhook(self):
        """Установка webhook"""
        await self.application.bot.set_webhook(
            url=self.webhook_url,
            # certificate=open('/path/to/cert.pem', 'rb')  # для HTTPS
        )
    
    def run_webhook(self, port: int = 8443):
        """Запуск с webhook"""
        self.application.run_webhook(
            listen="0.0.0.0",
            port=port,
            webhook_url=self.webhook_url,
            # key='private.key',    # для HTTPS
            # certificate='cert.pem', # для HTTPS
        )

# Использование
if __name__ == "__main__":
    bot = WebhookBot(
        token=os.getenv("BOT_TOKEN"),
        webhook_url="https://yourdomain.com/webhook"
    )
    bot.run_webhook(port=8443)
```

### 3. Docker контейнеризация
```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Копируем зависимости
COPY requirements.txt .

# Устанавливаем зависимости
RUN pip install --no-cache-dir -r requirements.txt

# Копируем код
COPY . .

# Создаем пользователя для безопасности
RUN useradd -m -u 1000 botuser
USER botuser

# Запускаем бота
CMD ["python", "bot.py"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  telegram-bot:
    build: .
    environment:
      - BOT_TOKEN=${BOT_TOKEN}
      - DATABASE_URL=sqlite:///bot.db
    volumes:
      - ./data:/app/data
    restart: unless-stopped
```

---

## Лучшие практики

### 1. Безопасность
```python
# Проверка пользователя
async def admin_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Команда только для администратора"""
    admin_id = int(os.getenv("ADMIN_ID"))
    
    if update.effective_user.id != admin_id:
        await update.message.reply_text("У вас нет прав для этой команды")
        return
    
    # Логика команды администратора
    await update.message.reply_text("Админ команда выполнена")
```

### 2. Обработка ошибок
```python
async def robust_message_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    try:
        # Основная логика
        await process_message(update, context)
    except Exception as e:
        logger.error(f"Ошибка обработки сообщения: {e}")
        await update.message.reply_text("Произошла ошибка. Попробуйте позже.")

async def global_error_handler(update: object, context: ContextTypes.DEFAULT_TYPE):
    """Глобальный обработчик ошибок"""
    error = context.error
    
    if isinstance(error, telegram.error.NetworkError):
        logger.warning("Проблемы с сетью")
    elif isinstance(error, telegram.error.TimedOut):
        logger.warning("Таймаут запроса")
    else:
        logger.error(f"Необработанная ошибка: {error}")
```

### 3. Логирование
```python
import logging
import sys

def setup_logging():
    """Настройка комплексного логирования"""
    
    # Форматтер
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    # Обработчик для файла
    file_handler = logging.FileHandler('bot.log', encoding='utf-8')
    file_handler.setFormatter(formatter)
    
    # Обработчик для консоли
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(formatter)
    
    # Настройка корневого логгера
    logging.basicConfig(
        level=logging.INFO,
        handlers=[file_handler, console_handler]
    )
```

### 4. Производительность
```python
# Использование кэширования
import functools
from typing import Dict, Any

def cache_user_data(func):
    """Декоратор для кэширования данных пользователя"""
    cache: Dict[int, Any] = {}
    
    @functools.wraps(func)
    async def wrapper(update: Update, context: ContextTypes.DEFAULT_TYPE):
        user_id = update.effective_user.id
        
        if user_id in cache:
            return cache[user_id]
        
        result = await func(update, context)
        cache[user_id] = result
        return result
    
    return wrapper
```

---

## Заключение

### Ключевые моменты для запоминания:

1. **Всегда используйте переменные окружения** для конфиденциальных данных
2. **Обрабатывайте исключения** на всех уровнях приложения
3. **Используйте базу данных** для сохранения состояния между перезапусками
4. **Тестируйте бота** перед развертыванием в продакшен
5. **Соблюдайте лимиты Telegram API** (не более 30 сообщений в секунду)
6. **Используйте подходящий метод** получения обновлений (Polling для разработки, Webhook для продакшена)
