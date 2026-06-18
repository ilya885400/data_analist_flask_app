
# DataMind // AI Data Analyst Terminal

[![System Status](https://img.shields.io/badge/system__status-operational-39ff14?style=flat-mono&logo=matrix)](https://github.com/)
[![Docker](https://img.shields.io/badge/sandbox-isolated_docker-blue?style=flat-mono&logo=docker)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/backend-python_3.11_/_flask-green?style=flat-mono&logo=python)](https://www.python.org/)

> DataMind — это интерактивный ИИ-агент для автоматизированного разведочного анализа данных (EDA). Агент разворачивает изолированную песочницу Docker, генерирует итеративный Python-код для обработки массивов данных, строит графики с помощью matplotlib/seaborn и компилирует детальный аналитический аудит-отчет.

Интерфейс выполнен в кастомном киберпанк / хакерском (Matrix Neon Green) стиле со скрин-эффектами и динамическими терминальными логами выполнения.

---

## Ключевые возможности

* Изолированная Docker-песочница: Весь сгенерированный нейросетью код исполняется внутри защищенного контейнера с ограничением памяти (512MB), процессора и без доступа к сети.
* Динамическая конфигурация LLM: Возможность «на лету» указать собственный API-ключ OpenRouter и любую текстовую модель (например, deepseek/deepseek-chat, qwen/qwen-2.5-72b-instruct:free, meta-llama/llama-3.3-70b-instruct:free).
* Инъекционная безопасность: Встроенная валидация контекста пользователя на популярные паттерны Jailbreak и Prompt Injection.
* Визуализация аномалий и трендов: Автоматический перехват графиков plt.show() из контейнера и их рендеринг в веб-интерфейсе в реальном времени.
* Терминальный логгер прогона: Пошаговое отслеживание итераций агента (чтение типов, вычисление корреляций, генерация инсайтов).

---

## Стек технологий

* Backend: Python 3.11, Flask, Requests, Docker CLI Automation
* Sandbox (Docker): pandas, numpy, matplotlib, seaborn, scipy, openpyxl, xlrd (образ datamind-sandbox)
* Frontend: Ванильный HTML5 / CSS3 (JetBrains Mono, CSS Grid, Custom Matrix Glow, Fluid Layout), JavaScript (Fetch API)
* Core Orchestration: OpenRouter API (Инструменты / Function Calling loop)

---

## Быстрый старт

### 1. Требования
На хост-машине должны быть установлены:
* Python 3.11+
* Docker (убедитесь, что демону Docker разрешено выполнять команды без sudo, либо запустите скрипт из-под нужного пользователя).

### 2. Клонирование и установка зависимостей
```bash
git clone [https://github.com/yourusername/datamind.git](https://github.com/yourusername/datamind.git)
cd datamind
pip install -r requirements.txt

```

### 3. Переменные окружения (Опционально)

Вы можете задать дефолтный ключ OpenRouter прямо в системе, чтобы не вводить его в браузере:

```bash
export OPENROUTER_API_KEY="your_sk_or_api_key_here"
```

### 4. Запуск приложения

```bash
python app.py

```

При первом запуске бэкенд автоматически соберет изолированный Docker-образ datamind-sandbox:latest. Это может занять пару минут.

После завершения терминал выдаст адрес локального сервера:

```text
[docker] Sandbox image ready.
* Running on [http://127.0.0.1:5000](http://127.0.0.1:5000)

```

---

## Структура проекта

```text
├── app.py                 # Сервер Flask, логика агента и менеджер Docker-контейнеров
├── templates/
│   └── index.html         # Хакерский UI терминал, валидация файлов, AJAX-запросы
├── requirements.txt       # Зависимости хост-системы (Flask, requests)
└── README.md              # Текущая документация

```

---

## Безопасность (Sandbox Architecture)

Агент выполняет код разворачиваемой нейросети с использованием жестких политик изоляции Docker:

```python
docker_cmd = [
    "docker", "run",
    "--rm",
    "--network", "none",         # Полное отсутствие сети (защита от утечки данных)
    "--memory", "512m",          # Лимит ОЗУ (защита от Fork-бомб)
    "--cpus", "1.0",             # Ограничение ресурсов процессора
    "--read-only",               # Запрет на модификацию системных файлов контейнера
    "--tmpfs", "/tmp:size=128m,exec", 
    "--cap-drop", "ALL",         # Сброс всех Linux capabilities
    "--security-opt", "no-new-privileges"
]

```
