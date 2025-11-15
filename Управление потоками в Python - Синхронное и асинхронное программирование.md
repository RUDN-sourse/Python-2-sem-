# Лекция: Управление потоками в Python - Синхронное и асинхронное программирование

## Введение

Программы могут выполняться разными способами: последовательно (синхронно) или параллельно (асинхронно). Понимание этих концепций критически важно для создания эффективных приложений.

## Часть 1: Синхронное программирование

### Что такое синхронное программирование?

**Синхронное программирование** - это традиционный подход, где операции выполняются последовательно, одна за другой. Каждая следующая операция ждет завершения предыдущей.

```python
import time

def sync_task(name, duration):
    """
    Функция имитирует задачу, которая занимает определенное время
    
    Параметры:
    name (str): название задачи
    duration (int): время выполнения в секундах
    """
    print(f"Задача '{name}' началась в {time.strftime('%X')}")
    time.sleep(duration)  # Имитация долгой операции
    print(f"Задача '{name}' завершилась в {time.strftime('%X')}")

# Синхронное выполнение
def main_sync():
    print("=== СИНХРОННОЕ ВЫПОЛНЕНИЕ ===")
    
    # Задачи выполняются последовательно
    sync_task("Загрузка данных", 2)
    sync_task("Обработка изображения", 3)
    sync_task("Сохранение результата", 1)

# Запуск синхронной версии
main_sync()
```

**Разбор кода:**

1. **Функция `sync_task`**:
   - Принимает имя задачи и время выполнения
   - Выводит время начала и завершения
   - `time.sleep(duration)` имитирует долгую операцию (например, запрос к БД)

2. **Функция `main_sync`**:
   - Вызывает задачи последовательно
   - Каждая следующая задача ждет завершения предыдущей

**Результат выполнения:**
```
=== СИНХРОННОЕ ВЫПОЛНЕНИЕ ===
Задача 'Загрузка данных' началась в 14:30:15
Задача 'Загрузка данных' завершилась в 14:30:17
Задача 'Обработка изображения' началась в 14:30:17
Задача 'Обработка изображения' завершилась в 14:30:20
Задача 'Сохранение результата' началась в 14:30:20
Задача 'Сохранение результата' завершилась в 14:30:21
```

**Общее время:** ~6 секунд

## Часть 2: Многопоточное программирование

### Что такое потоки?

**Потоки** (threads) - это легковесные процессы, которые разделяют общую память. Полезны для I/O-bound задач (операции ввода-вывода).

```python
import threading
import time

def thread_task(name, duration):
    """
    Задача для выполнения в отдельном потоке
    """
    print(f"Поток '{name}' начался в {time.strftime('%X')}")
    time.sleep(duration)
    print(f"Поток '{name}' завершился в {time.strftime('%X')}")

def main_threads():
    print("=== МНОГОПОТОЧНОЕ ВЫПОЛНЕНИЕ ===")
    
    # Создаем список для хранения объектов потоков
    threads = []
    
    # Создаем и запускаем потоки
    threads.append(threading.Thread(target=thread_task, args=("Загрузка данных", 2)))
    threads.append(threading.Thread(target=thread_task, args=("Обработка изображения", 3)))
    threads.append(threading.Thread(target=thread_task, args=("Сохранение результата", 1)))
    
    # Запускаем все потоки
    for thread in threads:
        thread.start()
    
    # Ждем завершения всех потоков
    for thread in threads:
        thread.join()
    
    print("Все потоки завершены")

# Запуск многопоточной версии
main_threads()
```

**Разбор кода:**

1. **Импорт модуля**: `import threading` - модуль для работы с потоками

2. **Создание потоков**:
   - `threading.Thread(target=функция, args=аргументы)` - создает объект потока
   - `target` - функция, которая будет выполняться в потоке
   - `args` - аргументы для функции

3. **Запуск потоков**:
   - `thread.start()` - запускает поток
   - Потоки начинают выполняться параллельно

4. **Ожидание завершения**:
   - `thread.join()` - блокирует выполнение основного потока до завершения указанного потока

**Особенности многопоточности в Python:**
- Из-за GIL (Global Interpreter Lock) потоки в Python не выполняются真正 параллельно для CPU-bound задач
- Эффективны для I/O-bound задач (сетевые запросы, чтение файлов)

## Часть 3: Многопроцессное программирование

### Что такое процессы?

**Процессы** - это независимые экземпляры программы с собственной памятью. Подходят для CPU-bound задач.

```python
import multiprocessing
import time

def process_task(name, duration):
    """
    Задача для выполнения в отдельном процессе
    """
    print(f"Процесс '{name}' начался в {time.strftime('%X')}")
    time.sleep(duration)
    print(f"Процесс '{name}' завершился в {time.strftime('%X')}")

def main_processes():
    print("=== МНОГОПРОЦЕССНОЕ ВЫПОЛНЕНИЕ ===")
    
    # Создаем список для хранения объектов процессов
    processes = []
    
    # Создаем и запускаем процессы
    processes.append(multiprocessing.Process(target=process_task, args=("Загрузка данных", 2)))
    processes.append(multiprocessing.Process(target=process_task, args=("Обработка изображения", 3)))
    processes.append(multiprocessing.Process(target=process_task, args=("Сохранение результата", 1)))
    
    # Запускаем все процессы
    for process in processes:
        process.start()
    
    # Ждем завершения всех процессов
    for process in processes:
        process.join()
    
    print("Все процессы завершены")

# Запуск многопроцессной версии
if __name__ == "__main__":
    main_processes()
```

**Разбор кода:**

1. **Импорт модуля**: `import multiprocessing` - модуль для работы с процессами

2. **Создание процессов**:
   - `multiprocessing.Process(target=функция, args=аргументы)` - создает объект процесса

3. **Запуск и ожидание**:
   - Аналогично потокам: `start()` и `join()`

4. **Важное замечание**:
   - `if __name__ == "__main__":` - обязательно для многопроцессных программ в Python

**Отличия процессов от потоков:**
- Процессы имеют отдельную память
- Нет ограничения GIL
- Больше накладных расходов на создание
- Лучше для CPU-intensive задач

## Часть 4: Асинхронное программирование с asyncio

### Что такое асинхронность?

**Асинхронное программирование** - это подход, где одна задача может быть приостановлена, пока выполняется другая, без блокировки основного потока.

```python
import asyncio
import time

async def async_task(name, duration):
    """
    Асинхронная задача
    
    Ключевое слово async указывает, что это асинхронная функция (корутина)
    """
    print(f"Асинхронная задача '{name}' началась в {time.strftime('%X')}")
    
    # await приостанавливает выполнение корутины, но не блокирует весь поток
    await asyncio.sleep(duration)
    
    print(f"Асинхронная задача '{name}' завершилась в {time.strftime('%X')}")

async def main_async():
    print("=== АСИНХРОННОЕ ВЫПОЛНЕНИЕ ===")
    
    # Создаем список задач
    tasks = [
        async_task("Загрузка данных", 2),
        async_task("Обработка изображения", 3),
        async_task("Сохранение результата", 1)
    ]
    
    # Запускаем все задачи параллельно и ждем их завершения
    await asyncio.gather(*tasks)
    
    print("Все асинхронные задачи завершены")

# Запуск асинхронной версии
if __name__ == "__main__":
    # asyncio.run() запускает асинхронную функцию
    asyncio.run(main_async())
```

**Разбор кода:**

1. **Ключевое слово `async`**:
   - Объявляет асинхронную функцию (корутину)
   - Корутина может быть приостановлена и возобновлена

2. **Ключевое слово `await`**:
   - Приостанавливает выполнение корутины до завершения асинхронной операции
   - Позволяет event loop'у переключиться на другую задачу

3. **`asyncio.sleep()`**:
   - Асинхронная версия time.sleep()
   - Не блокирует весь поток, только текущую корутину

4. **`asyncio.gather()`**:
   - Запускает несколько корутин параллельно
   - Возвращает результаты, когда все завершатся

5. **`asyncio.run()`**:
   - Запускает асинхронную функцию
   - Создает и управляет event loop'ом

## Часть 5: Практический пример - Сравнение подходов

Давайте сравним все подходы на реальном примере - загрузке веб-страниц.

```python
import time
import threading
import multiprocessing
import asyncio
import requests
import aiohttp
import ssl
import certifi

# Синхронная версия
def sync_download(url, name):
    """Синхронная загрузка URL"""
    print(f"Синхронная загрузка {name} начата")
    response = requests.get(url)
    print(f"Синхронная загрузка {name} завершена, статус: {response.status_code}")
    return len(response.content)

# Многопоточная версия
def thread_download(url, name):
    """Загрузка URL в потоке"""
    print(f"Потоковая загрузка {name} начата")
    response = requests.get(url)
    print(f"Потоковая загрузка {name} завершена, статус: {response.status_code}")
    return len(response.content)

# Многопроцессная версия  
def process_download(url, name):
    """Загрузка URL в процессе"""
    print(f"Процессная загрузка {name} начата")
    response = requests.get(url)
    print(f"Процессная загрузка {name} завершена, статус: {response.status_code}")
    return len(response.content)

# Асинхронная версия
async def async_download(session, url, name):
    """Асинхронная загрузка URL"""
    print(f"Асинхронная загрузка {name} начата")
    try:
        async with session.get(url) as response:
            content = await response.read()
            print(f"Асинхронная загрузка {name} завершена, статус: {response.status}")
            return len(content)
    except Exception as e:
        print(f"Ошибка при загрузке {name}: {e}")
        return 0

async def main_async_download():
    """Основная асинхронная функция"""
    # Создаем кастомный SSL-контекст для обхода проблем с сертификатами
    ssl_context = ssl.create_default_context(cafile=certifi.where())
    
    # Альтернативное решение: отключаем проверку SSL (не рекомендуется для продакшена)
    # ssl_context = ssl.create_default_context()
    # ssl_context.check_hostname = False
    # ssl_context.verify_mode = ssl.CERT_NONE
    
    connector = aiohttp.TCPConnector(ssl=ssl_context)
    
    async with aiohttp.ClientSession(connector=connector) as session:
        urls = [
            ("https://httpbin.org/delay/1", "Сайт 1"),
            ("https://httpbin.org/delay/2", "Сайт 2"), 
            ("https://httpbin.org/delay/1", "Сайт 3")
        ]
        
        tasks = [async_download(session, url, name) for url, name in urls]
        results = await asyncio.gather(*tasks)
        return results

# Альтернативная асинхронная версия с отключенной SSL-проверкой
async def main_async_download_no_ssl():
    """Асинхронная версия с отключенной SSL-проверкой"""
    connector = aiohttp.TCPConnector(ssl=False)  # Отключаем SSL проверку
    
    async with aiohttp.ClientSession(connector=connector) as session:
        urls = [
            ("https://httpbin.org/delay/1", "Сайт 1"),
            ("https://httpbin.org/delay/2", "Сайт 2"), 
            ("https://httpbin.org/delay/1", "Сайт 3")
        ]
        
        tasks = [async_download(session, url, name) for url, name in urls]
        results = await asyncio.gather(*tasks)
        return results

# Функции для запуска разных подходов
def run_sync():
    """Запуск синхронной версии"""
    print("=== СИНХРОННАЯ ЗАГРУЗКА ===")
    start_time = time.time()
    
    urls = [
        ("https://httpbin.org/delay/1", "Сайт 1"),
        ("https://httpbin.org/delay/2", "Сайт 2"),
        ("https://httpbin.org/delay/1", "Сайт 3")
    ]
    
    for url, name in urls:
        sync_download(url, name)
    
    print(f"Общее время: {time.time() - start_time:.2f} секунд")

def run_threads():
    """Запуск многопоточной версии"""
    print("=== МНОГОПОТОЧНАЯ ЗАГРУЗКА ===")
    start_time = time.time()
    
    urls = [
        ("https://httpbin.org/delay/1", "Сайт 1"),
        ("https://httpbin.org/delay/2", "Сайт 2"),
        ("https://httpbin.org/delay/1", "Сайт 3")
    ]
    
    threads = []
    for url, name in urls:
        thread = threading.Thread(target=thread_download, args=(url, name))
        threads.append(thread)
        thread.start()
    
    for thread in threads:
        thread.join()
    
    print(f"Общее время: {time.time() - start_time:.2f} секунд")

def run_async():
    """Запуск асинхронной версии"""
    print("=== АСИНХРОННАЯ ЗАГРУЗКА ===")
    start_time = time.time()
    
    asyncio.run(main_async_download())
    
    print(f"Общее время: {time.time() - start_time:.2f} секунд")

def run_async_no_ssl():
    """Запуск асинхронной версии с отключенной SSL-проверкой"""
    print("=== АСИНХРОННАЯ ЗАГРУЗКА (без SSL) ===")
    start_time = time.time()
    
    asyncio.run(main_async_download_no_ssl())
    
    print(f"Общее время: {time.time() - start_time:.2f} секунд")

# Установите certifi если еще не установлен
def install_certifi():
    try:
        import certifi
    except ImportError:
        print("Установка certifi...")
        import subprocess
        import sys
        subprocess.check_call([sys.executable, "-m", "pip", "install", "certifi"])
        import certifi
    return certifi

# Запуск всех примеров
if __name__ == "__main__":
    # Убедимся, что certifi установлен
    certifi = install_certifi()
    
    run_sync()
    print("\n" + "="*50 + "\n")
    run_threads() 
    print("\n" + "="*50 + "\n")
    
    # Пробуем оба варианта асинхронной загрузки
    try:
        run_async()
    except Exception as e:
        print(f"Ошибка в асинхронной версии с SSL: {e}")
        print("Пробуем версию без SSL проверки...")
        run_async_no_ssl()
```

## Часть 6: Ключевые концепции асинхронности

### Event Loop (Цикл событий)

**Event Loop** - это ядро асинхронных приложений. Он:
- Выполняет корутины
- Обрабатывает системные события
- Выполняет callback'и
- Управляет сетевыми операциями

```python
import asyncio

async def example_event_loop():
    """Пример работы с event loop"""
    
    # Получаем текущий event loop
    loop = asyncio.get_running_loop()
    print(f"Event loop: {loop}")
    
    # Создаем несколько задач
    task1 = asyncio.create_task(asyncio.sleep(1, "Результат 1"))
    task2 = asyncio.create_task(asyncio.sleep(2, "Результат 2"))
    
    # Ждем завершения задач
    result1 = await task1
    result2 = await task2
    
    print(f"Результат 1: {result1}")
    print(f"Результат 2: {result2}")

# asyncio.run(example_event_loop())
```

### Futures и Tasks

**Future** - объект, представляющий результат асинхронной операции, который будет доступен в будущем.

**Task** - подкласс Future, который оборачивает корутину.

```python
import asyncio

async def future_example():
    """Пример работы с Future и Task"""
    
    # Создаем корутину
    async def simple_coroutine(x):
        await asyncio.sleep(1)
        return x * 2
    
    # Создаем Task из корутины
    task = asyncio.create_task(simple_coroutine(21))
    
    # Task является подклассом Future
    print(f"Task является Future: {isinstance(task, asyncio.Future)}")
    
    # Ждем результат
    result = await task
    print(f"Результат: {result}")

# asyncio.run(future_example())
```

## Часть 7: Обработка ошибок в асинхронном коде

```python
import asyncio

async def error_handling_example():
    """Пример обработки ошибок в асинхронном коде"""
    
    async def successful_task():
        await asyncio.sleep(1)
        return "Успех!"
    
    async def failing_task():
        await asyncio.sleep(1)
        raise ValueError("Что-то пошло не так!")
    
    try:
        # Создаем задачи
        task1 = asyncio.create_task(successful_task())
        task2 = asyncio.create_task(failing_task())
        
        # Ждем результаты
        result1 = await task1
        result2 = await task2
        
    except Exception as e:
        print(f"Поймано исключение: {e}")
    
    # Альтернативный способ с asyncio.gather
    try:
        results = await asyncio.gather(
            successful_task(),
            failing_task(),
            return_exceptions=True  # Возвращать исключения как результаты
        )
        
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                print(f"Задача {i} завершилась с ошибкой: {result}")
            else:
                print(f"Задача {i} успешно завершена: {result}")
                
    except Exception as e:
        print(f"Общее исключение: {e}")

# asyncio.run(error_handling_example())
```

## Заключение

### Когда что использовать:

1. **Синхронное программирование**:
   - Простые скрипты
   - Последовательные операции
   - Когда простота важнее производительности

2. **Многопоточность**:
   - I/O-bound задачи (сеть, файлы)
   - GUI приложения
   - Когда нужен общий доступ к памяти

3. **Многопроцессность**:
   - CPU-bound задачи (вычисления)
   - Когда нужно обойти GIL
   - Изоляция процессов

4. **Асинхронность**:
   - Высоконагруженные I/O приложения
   - Веб-серверы
   - Сетевые клиенты
   - Когда важна масштабируемость

### Ключевые принципы:

- **I/O-bound** vs **CPU-bound** - определяет выбор подхода
- **GIL** ограничивает настоящую параллельность потоков в Python
- **Асинхронность** не делает код быстрее, но делает его более эффективным для I/O операций
- **Процессы** имеют больше накладных расходов, но настоящую параллельность

Практикуйтесь с каждым подходом, чтобы понять их сильные и слабые стороны в различных сценариях!
