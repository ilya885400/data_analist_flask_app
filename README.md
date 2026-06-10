# DataMind — AI Аналитик данных

Веб-приложение для агентного анализа датасетов. LLM (через OpenRouter) **сама пишет и выполняет Python-код** в изолированном Docker-контейнере через tool-use / function-calling loop.

## Стек

| Слой | Технология |
|------|-----------|
| Веб-интерфейс | Flask + HTML/CSS/JS |
| LLM | OpenRouter API → `inclusionai/ring-2.6-1t:free` |
| Tool-use loop | OpenAI-совместимый `function_calling` |
| Выполнение кода | Docker (`datamind-sandbox`) — изолированный контейнер |
| Данные | pandas, numpy, matplotlib, seaborn, scipy |

## Архитектура

```
Браузер → Flask → check_injection()
                       │
                       ▼
               run_agent() — agentic loop
               ┌──────────────────────────┐
               │  OpenRouter API          │
               │  (function_calling)      │
               │         │                │
               │  execute_code tool       │
               │         │                │
               │  Docker container        │
               │  --network none          │
               │  --read-only             │
               │  --memory 512m           │
               │  --cap-drop ALL          │
               └──────────────────────────┘
                       │
               figures (base64 PNG) + report
```

## Требования

- Python 3.10+
- **Docker** (установлен и запущен: `docker info`)
- Аккаунт на [openrouter.ai](https://openrouter.ai) (бесплатный)

## Установка

```bash
git clone <repo>
cd data_analyst_app
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
```

## Конфигурация

```bash
export OPENROUTER_API_KEY="sk-or-v1-..."
```

Получить ключ: https://openrouter.ai/keys (бесплатно)

## Запуск

```bash
python app.py
```

При первом запуске автоматически собирается Docker-образ `datamind-sandbox:latest`
с нужными библиотеками (~30 сек, только один раз).

Открыть: **http://localhost:5000**

## Проверка окружения

```bash
curl http://localhost:5000/health
# {"status":"ok","docker":true,"sandbox_image":true,"model":"inclusionai/ring-2.6-1t:free"}
```

## Docker sandbox

Контейнер создаётся на каждый запуск кода с жёсткими ограничениями:

```
--network none          нет интернета
--memory 512m           лимит RAM
--cpus 1.0              лимит CPU  
--read-only             root FS только для чтения
--tmpfs /tmp:128m       только /tmp записываемый
--cap-drop ALL          без linux capabilities
--security-opt no-new-privileges
```

Датасет и скрипт монтируются как `:ro` (read-only bind mount).

## Защита от Prompt Injection

Контекст пользователя проверяется на сервере:
- 16 regex-паттернов (ignore instructions, jailbreak, sudo, roleplay, ...)
- Лимит длины 3000 символов
- Системный промпт явно запрещает LLM выполнять нон-аналитические задачи
