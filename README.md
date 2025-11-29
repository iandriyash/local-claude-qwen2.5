# Локальная среда Qwen2.5 (Ollama + Open WebUI)

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Docker Ready](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://www.docker.com/)
![GPU Support](https://img.shields.io/badge/GPU-NVIDIA%20CUDA-brightgreen.svg)
![Models](https://img.shields.io/badge/Models-Qwen2.5%2014B%20%7C%20Coder%2014B-orange.svg)
[![Powered by Ollama](https://img.shields.io/badge/Ollama-Local%20LLM-lightgrey.svg)](https://ollama.com/)

Локальная офлайн-среда для запуска больших языковых моделей Qwen2.5  
(чат-ассистент, генерация кода, анализ, проекты) с использованием NVIDIA GPU.

Используются:
- **Ollama** — локальный движок для моделей  
- **Open WebUI** — интерфейс наподобие ChatGPT/Claude  
- **Модели Qwen2.5 (14B Instruct + 14B Coder)**  
- **Docker Compose**  

---

# **1. Требования**

## Аппаратные
- NVIDIA GPU (желательно от 12–16 GB VRAM)
- Рекомендуемые карты: RTX 4070 / 4080 / 4090

## Программные
- Docker Desktop (Windows/macOS) или Docker Engine (Linux)
- Драйвер NVIDIA
- (Linux/RED OS) Установленная CUDA

---

# **2. Установка Docker (Windows)**

1. Скачать Docker Desktop:  
   https://www.docker.com/products/docker-desktop/

2. Включить при установке:
   - WSL2 backend  
   - NVIDIA GPU support  

3. Перезагрузить компьютер.

4. Проверить установку:

**`docker --version`**

---

# **3. Проверка доступа к GPU**

Выполнить:

**`docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi`**

Если видна ваша видеокарта — GPU доступна для Docker.

---

# **4. Архитектура проекта**

Пользователь (браузер: http://localhost:3000)
│
▼
Open WebUI
│
▼
Ollama
│
▼
Модели Qwen2.5 (14B / 14B Coder)
│
▼
NVIDIA GPU

yaml
Копировать код

---

# **5. Файл docker-compose.yml**

Создайте файл `docker-compose.yml`:

services:
ollama:
image: ollama/ollama:latest
container_name: ollama
restart: unless-stopped
ports:
- "11434:11434"
volumes:
- ollama-data:/root/.ollama
gpus: all

open-webui:
image: ghcr.io/open-webui/open-webui:main
container_name: open-webui
restart: unless-stopped
depends_on:
- ollama
ports:
- "3000:8080"
environment:
- OLLAMA_BASE_URL=http://ollama:11434
volumes:
- open-webui-data:/app/backend/data

volumes:
ollama-data:
open-webui-data:

yaml
Копировать код

---

# **6. Запуск проекта**

Перейдите в папку проекта:

**`cd C:\dev\local-claude`**

Запустите сервисы:

**`docker compose up -d`**

Проверьте:

**`docker ps`**

Откройте интерфейс:

http://localhost:3000

yaml
Копировать код

---

# **7. Загрузка моделей Qwen2.5**

Войти в контейнер Ollama:

**`docker exec -it ollama bash`**

Основная модель:

**`ollama pull qwen2.5:14b-instruct`**

Программирование:

**`ollama pull qwen2.5-coder:14b`**

---

# **8. Характеристики моделей**

## **Qwen2.5:14B Instruct**
- Параметры: **14.8 млрд**
- Контекст: **32768 токенов**
- Квантовка: **Q4_K_M**
- Размер на диске: **≈9 GB**
- Потребление VRAM: **≈8–10 GB**
- Назначение: диалоги, обучение, аналитика

## **Qwen2.5:14B Coder**
- Параметры: **14.8 млрд**
- Квантовка: **Q4_K_M**
- Размер: **≈9 GB**
- Потребление VRAM: **≈8–10 GB**
- Назначение: генерация кода, анализ проектов

---

# **9. Квантовка (4-bit) — почему это важно**

| Формат | Разрядность | Потребление VRAM | Качество |
|--------|-------------|------------------|----------|
| FP16   | 16-bit      | очень высокое    | максимальное |
| INT8   | 8-bit       | высокое          | хорошее |
| **Q4_K_M** | **4-bit** | **низкое**     | почти как FP16 |

Преимущества Q4_K_M:

- работает на 12–16 GB VRAM  
- отличное качество  
- маленький размер  
- высокая скорость  

---

# **10. Выбор модели в Open WebUI**

В интерфейсе можно выбрать:

- **qwen2.5:14b-instruct**  
- **qwen2.5-coder:14b**  

---

# **11. System Prompt для программирования**

Открыть чат → настройки → System Prompt:

Ты — профессиональный AI-программист.
Пишешь структурированный и компилируемый код.
Не придумываешь несуществующих классов и библиотек.
Если данных недостаточно — задаёшь уточняющие вопросы.
Разделяешь код по файлам.
Работаешь строго и технически грамотно.

yaml
Копировать код

---

# **12. Управление контейнерами**

Остановить:

**`docker compose down`**

Запустить:

**`docker compose up -d`**

Перезапуск:

**`docker compose restart`**

---

# **13. Дополнительные модели**

Быстрая модель:

**`ollama pull qwen2.5:7b-instruct`**

Лёгкий кодер:

**`ollama pull qwen2.5-coder:7b`**

---

# **14. Установка на RED OS**

Установить Docker:

**`sudo dnf install docker`**

Активировать NVIDIA runtime:

**`sudo nvidia-ctk runtime configure --runtime=docker`**

Перезапустить Docker:

**`sudo systemctl restart docker`**

Разрешить порты:

**`sudo firewall-cmd --add-port=3000/tcp --permanent`**  
**`sudo firewall-cmd --add-port=11434/tcp --permanent`**  
**`sudo firewall-cmd --reload`**

---

# **15. Credits**

Этот проект основан на следующих технологиях:

- **Ollama** — локальный движок для запуска LLM  
  https://ollama.com/

- **Open WebUI** — удобный веб-интерфейс  
  https://github.com/open-webui/open-webui

---

# **16. Лицензия**

Проект распространяется по лицензии MIT.  
Полный текст находится в файле **LICENSE**.
Готово ✔
