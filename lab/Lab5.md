# Лабораторная работа: Управление потоками в Python

## Цель работы
Освоить различные подходы к управлению потоками выполнения в Python: синхронное, многопоточное, многопроцессное и асинхронное программирование.

---

## Задача 1: Синхронный калькулятор времени выполнения

**Цель:** Понять основы синхронного выполнения операций.

```python
import time

def sync_calculate(operation, a, b, delay):
    """
    Выполняет математическую операцию с задержкой
    
    Параметры:
    operation (str): тип операции ('+', '-', '*', '/')
    a, b (float): числа для операции
    delay (float): время имитации вычислений
    
    Возвращает:
    float: результат операции
    """
    print(f"Начало операции {a} {operation} {b}")
    time.sleep(delay)  # Имитация долгого вычисления
    
    if operation == '+':
        result = a + b
    elif operation == '-':
        result = a - b
    elif operation == '*':
        result = a * b
    elif operation == '/':
        result = a / b if b != 0 else 'Ошибка: деление на ноль'
    else:
        result = 'Неизвестная операция'
    
    print(f"Конец операции {a} {operation} {b} = {result}")
    return result

def task1_sync_calculations():
    """
    Задача: Выполните последовательно 4 операции и измерьте общее время выполнения.
    Операции:
    1. 15 + 25 с задержкой 2 секунды
    2. 40 - 18 с задержкой 1 секунда  
    3. 12 * 8 с задержкой 3 секунды
    4. 100 / 5 с задержкой 1 секунда
    
    Требуется:
    - Вывести общее время выполнения
    - Вывести результаты всех операций
    """
    start_time = time.time()
    
    # Ваш код здесь
    results = []
    # TODO: Выполните операции последовательно
    
    end_time = time.time()
    print(f"Общее время выполнения: {end_time - start_time:.2f} секунд")
    print(f"Результаты: {results}")

# Запуск задачи
# task1_sync_calculations()
```

---

## Задача 2: Многопоточный загрузчик файлов

**Цель:** Научиться использовать потоки для параллельного выполнения I/O операций.

```python
import threading
import time
import random

def download_file(filename, size):
    """
    Имитирует загрузку файла
    
    Параметры:
    filename (str): имя файла
    size (int): размер файла в МБ
    """
    download_time = size * 0.1  # 0.1 сек на МБ
    print(f"Начало загрузки {filename} ({size} МБ)")
    
    # Имитация прогресса загрузки
    for i in range(5):
        time.sleep(download_time / 5)
        progress = (i + 1) * 20
        print(f"{filename}: {progress}% загружено")
    
    print(f"Завершена загрузка {filename}")

def task2_threaded_downloader():
    """
    Задача: Реализуйте многопоточную загрузку файлов.
    Файлы для загрузки:
    1. "document.pdf" - 10 МБ
    2. "image.jpg" - 5 МБ  
    3. "video.mp4" - 20 МБ
    4. "archive.zip" - 15 МБ
    
    Требуется:
    - Загрузить все файлы параллельно в разных потоках
    - Измерить общее время выполнения
    - Убедиться, что все потоки завершились
    """
    files = [
        ("document.pdf", 10),
        ("image.jpg", 5),
        ("video.mp4", 20),
        ("archive.zip", 15)
    ]
    
    start_time = time.time()
    threads = []
    
    # Ваш код здесь
    # TODO: Создайте и запустите потоки для каждого файла
    # TODO: Дождитесь завершения всех потоков
    
    end_time = time.time()
    print(f"Общее время загрузки: {end_time - start_time:.2f} секунд")

# Запуск задачи  
# task2_threaded_downloader()
```

---

## Задача 3: Многопроцессный вычислитель

**Цель:** Освоить использование процессов для CPU-bound задач.

```python
import multiprocessing
import time
import math

def calculate_factorial(n):
    """
    Вычисляет факториал числа (CPU-intensive операция)
    """
    print(f"Начало вычисления факториала {n}!")
    result = math.factorial(n)
    print(f"Завершено вычисление факториала {n}! = {result}")
    return result

def calculate_prime(n):
    """
    Проверяет, является ли число простым
    """
    print(f"Начало проверки числа {n} на простоту")
    
    if n < 2:
        result = False
    else:
        result = all(n % i != 0 for i in range(2, int(math.sqrt(n)) + 1))
    
    print(f"Число {n} простое: {result}")
    return result

def task3_multiprocess_calculations():
    """
    Задача: Реализуйте многопроцессные вычисления.
    Вычисления:
    1. Факториал 100000
    2. Факториал 80000  
    3. Проверка числа 10000000019 на простоту
    4. Проверка числа 10000000033 на простоту
    
    Требуется:
    - Выполнить вычисления в отдельных процессах
    - Сравнить время с последовательным выполнением
    - Собрать и вывести результаты
    """
    calculations = [
        (calculate_factorial, 10000),  # Уменьшено для демонстрации
        (calculate_factorial, 8000),
        (calculate_prime, 10000019),
        (calculate_prime, 10000033)
    ]
    
    # Многопроцессное выполнение
    print("=== МНОГОПРОЦЕССНОЕ ВЫПОЛНЕНИЕ ===")
    start_time = time.time()
    
    processes = []
    results = []
    
    # Ваш код здесь
    # TODO: Создайте и запустите процессы
    # TODO: Соберите результаты
    
    end_time = time.time()
    multiprocess_time = end_time - start_time
    
    # Синхронное выполнение для сравнения
    print("\n=== СИНХРОННОЕ ВЫПОЛНЕНИЕ ===")
    start_time = time.time()
    
    sync_results = []
    for func, arg in calculations:
        sync_results.append(func(arg))
    
    end_time = time.time()
    sync_time = end_time - start_time
    
    print(f"\nСравнение времени:")
    print(f"Многопроцессное: {multiprocess_time:.2f} сек")
    print(f"Синхронное: {sync_time:.2f} сек")
    print(f"Ускорение: {sync_time/multiprocess_time:.2f}x")

# Запуск задачи
# if __name__ == "__main__":
#     task3_multiprocess_calculations()
```

---

## Задача 4: Асинхронный веб-скрапер

**Цель:** Научиться работать с асинхронным кодом и понимать event loop.

```python
import asyncio
import aiohttp
import time

async def fetch_url(session, url, name):
    """
    Асинхронно загружает веб-страницу
    """
    print(f"Начало загрузки {name}")
    
    try:
        async with session.get(url) as response:
            content = await response.text()
            print(f"Завершена загрузка {name}, статус: {response.status}")
            return len(content)
    except Exception as e:
        print(f"Ошибка при загрузке {name}: {e}")
        return 0

async def task4_async_scraper():
    """
    Задача: Создайте асинхронный веб-скрапер.
    
    URL для сканирования:
    1. "https://httpbin.org/delay/1" - "Сайт 1"
    2. "https://httpbin.org/delay/2" - "Сайт 2"  
    3. "https://httpbin.org/delay/1" - "Сайт 3"
    4. "https://httpbin.org/delay/3" - "Сайт 4"
    
    Требуется:
    - Реализовать асинхронную загрузку всех URL
    - Использовать aiohttp для HTTP запросов
    - Измерить общее время выполнения
    - Вывести размер загруженного контента для каждого сайта
    """
    urls = [
        ("https://httpbin.org/delay/1", "Сайт 1"),
        ("https://httpbin.org/delay/2", "Сайт 2"),
        ("https://httpbin.org/delay/1", "Сайт 3"), 
        ("https://httpbin.org/delay/3", "Сайт 4")
    ]
    
    start_time = time.time()
    
    # Ваш код здесь
    # TODO: Создайте ClientSession
    # TODO: Создайте задачи для каждого URL
    # TODO: Используйте asyncio.gather для параллельного выполнения
    
    end_time = time.time()
    print(f"Общее время выполнения: {end_time - start_time:.2f} секунд")
    print(f"Размеры контента: {results}")

# Запуск задачи
# asyncio.run(task4_async_scraper())
```

---

## Задача 5: Сравнительный анализ производительности

**Цель:** Сравнить эффективность разных подходов на одном наборе задач.

```python
import time
import threading
import multiprocessing
import asyncio
import requests

def io_task(name, duration):
    """I/O-bound задача"""
    time.sleep(duration)
    return f"{name} completed"

async def async_io_task(name, duration):
    """Асинхронная I/O-bound задача"""
    await asyncio.sleep(duration)
    return f"{name} completed"

def task5_performance_comparison():
    """
    Задача: Сравните производительность разных подходов.
    
    Набор I/O-bound задач (имитация):
    1. "Task1" - 2 секунды
    2. "Task2" - 3 секунды
    3. "Task3" - 1 секунда
    4. "Task4" - 2 секунды
    5. "Task5" - 1 секунда
    
    Требуется:
    - Реализовать выполнение одним потоком
    - Реализовать выполнение несколькими потоками  
    - Реализовать выполнение несколькими процессами
    - Реализовать асинхронное выполнение
    - Сравнить время выполнения каждого подхода
    - Сделать выводы о эффективности
    """
    tasks = [("Task1", 2), ("Task2", 3), ("Task3", 1), ("Task4", 2), ("Task5", 1)]
    
    # Ваш код здесь
    # TODO: Реализуйте все 4 подхода
    # TODO: Измерьте время для каждого
    # TODO: Проанализируйте результаты
    
    print("\n=== АНАЛИЗ РЕЗУЛЬТАТОВ ===")
    print("Синхронное время: X.XX сек")
    print("Многопоточное время: X.XX сек") 
    print("Многопроцессное время: X.XX сек")
    print("Асинхронное время: X.XX сек")
    
    # TODO: Сделайте выводы о том, какой подход наиболее эффективен для I/O-bound задач

# Запуск задачи
# task5_performance_comparison()
```

---

## Дополнительная задача 6: Асинхронный планировщик задач

**Цель:** Научиться управлять асинхронными задачами с приоритетами.

```python
import asyncio
import time
from datetime import datetime

async def scheduled_task(name, priority, duration):
    """
    Задача с приоритетом и временем выполнения
    
    Параметры:
    name (str): название задачи
    priority (int): приоритет (1 - высший)
    duration (float): время выполнения
    """
    print(f"[{datetime.now().strftime('%H:%M:%S')}] Задача '{name}' (приоритет {priority}) начата")
    await asyncio.sleep(duration)
    print(f"[{datetime.now().strftime('%H:%M:%S')}] Задача '{name}' завершена")
    return f"Результат {name}"

async def task6_async_scheduler():
    """
    Задача: Создайте асинхронный планировщик задач.
    
    Задачи:
    1. "Экстренная задача" - приоритет 1, длительность 1 сек
    2. "Важная задача" - приоритет 2, длительность 2 сек  
    3. "Обычная задача A" - приоритет 3, длительность 3 сек
    4. "Обычная задача B" - приоритет 3, длительность 2 сек
    5. "Фоновая задача" - приоритет 4, длительность 5 сек
    
    Требуется:
    - Запустить задачи в порядке приоритета
    - Обеспечить выполнение высокоприоритетных задач первыми
    - Реализовать ограничение на одновременное выполнение (не более 2 задач)
    - Вывести порядок завершения задач
    """
    tasks_with_priority = [
        ("Экстренная задача", 1, 1),
        ("Важная задача", 2, 2),
        ("Обычная задача A", 3, 3),
        ("Обычная задача B", 3, 2),
        ("Фоновая задача", 4, 5)
    ]
    
    # Ваш код здесь
    # TODO: Отсортируйте задачи по приоритету
    # TODO: Реализуйте семафор для ограничения одновременного выполнения
    # TODO: Запустите задачи и соберите результаты
    
    print("Все задачи завершены!")

# asyncio.run(task6_async_scheduler())
```

---

## Дополнительная задача 7: Пул потоков для обработки данных

**Цель:** Научиться использовать пулы потоков для обработки данных.

```python
import concurrent.futures
import time
import random

def process_data(item):
    """
    Обрабатывает элемент данных (имитация CPU-bound операции)
    """
    process_time = random.uniform(0.5, 2.0)
    time.sleep(process_time)
    result = item * 2  # Простая операция преобразования
    print(f"Обработан элемент {item} -> {result} (время: {process_time:.2f}с)")
    return result

def task7_thread_pool():
    """
    Задача: Реализуйте обработку данных с использованием пула потоков.
    
    Данные для обработки: список чисел от 1 до 10
    
    Требуется:
    - Обработать все данные с использованием ThreadPoolExecutor
    - Использовать разные размеры пула (2, 4, 8 потоков)
    - Сравнить время выполнения для разных размеров пула
    - Собрать и вывести результаты обработки
    """
    data = list(range(1, 11))
    
    print("=== ОБРАБОТКА ДАННЫХ С ПОМОЩЬЮ ПУЛА ПОТОКОВ ===")
    
    # Ваш код здесь
    # TODO: Реализуйте обработку с пулом из 2 потоков
    # TODO: Реализуйте обработку с пулом из 4 потоков  
    # TODO: Реализуйте обработку с пулом из 8 потоков
    # TODO: Сравните производительность
    
    print("\n=== СРАВНЕНИЕ ПРОИЗВОДИТЕЛЬНОСТИ ===")
    print("Пул 2 потоков: X.XX сек")
    print("Пул 4 потоков: X.XX сек")
    print("Пул 8 потоков: X.XX сек")

# task7_thread_pool()
```
