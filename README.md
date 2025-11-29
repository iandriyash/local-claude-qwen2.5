# Локальная среда Qwen2.5 (Ollama + Open WebUI)

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Docker Ready](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://www.docker.com/)
![GPU Support](https://img.shields.io/badge/GPU-NVIDIA%20CUDA-brightgreen.svg)
![Models](https://img.shields.io/badge/Models-Qwen2.5%2014B%20%7C%20Coder%2014B-orange.svg)
[![Powered by Ollama](https://img.shields.io/badge/Ollama-Local%20LLM-lightgrey.svg)](https://ollama.com/)

Этот проект позволяет развернуть локальную офлайн-среду для работы с большими языковыми моделями (LLM):

- чат-ассистент  
- анализ кода  
- генерация кода  
- работа с проектами  
- длинные рассуждения и задачи  

Используются:
- **Ollama** — локальный движок LLM  
- **Open WebUI** — интерфейс как ChatGPT/Claude  
- **Qwen2.5 (14B Instruct + 14B Coder)**  
- **Docker Compose**

---

# **1. Требования**

### Аппаратные
- NVIDIA GPU (рекомендуется 12–16 GB VRAM)
- Оптимально: RTX 4070 / 4080 / 4090

### Программные
- Docker Desktop / Docker Engine  
- Драйвер NVIDIA  
- На Linux / RED OS: CUDA Toolkit

---

# **2. Установка Docker (Windows)**

1. Скачать Docker Desktop:  
   https://www.docker.com/products/docker-desktop/

2. Включить при установке:  
   - WSL2 backend  
   - NVIDIA GPU support  

3. Перезагрузить ПК.

4. Проверить:

**`docker --version`**

---

# **3. Проверка доступа к GPU**

**`docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi`**

Если видна ваша видеокарта — всё работает.

---

# **4. Архитектура проекта**

Пользователь (http://localhost:3000)
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

Перейти в папку проекта:

**`cd C:\dev\local-claude`**

Запустить:

**`docker compose up -d`**

Проверить:

**`docker ps`**

Открыть:

http://localhost:3000

yaml
Копировать код

---

# **7. Загрузка моделей Qwen2.5**

Войти в контейнер:

**`docker exec -it ollama bash`**

Основная модель:

**`ollama pull qwen2.5:14b-instruct`**

Модель для программирования:

**`ollama pull qwen2.5-coder:14b`**

---

# **8. Характеристики моделей**

### **Qwen2.5:14B Instruct**
- Параметры: **14.8B**
- Контекст: **32768 токенов**
- Квантовка: **Q4_K_M**
- Размер модели: **9 GB**
- Потребление VRAM: **≈8–10 GB**
- Назначение: разговоры, анализ, помощь в обучении

### **Qwen2.5:14B Coder**
- Параметры: **14.8B**
- Квантовка: **Q4_K_M**
- Размер: **9 GB**
- Потребление VRAM: **≈8–10 GB**
- Назначение: анализ кода, генерация, работа с проектами

---

# **9. Квантовка (4-bit)**

Квантовка уменьшает разрядность чисел модели → уменьшает VRAM.

### Таблица:

| Формат | Битность | VRAM | Качество |
|-------|----------|------|----------|
| FP16 | 16-bit | очень высокое | максимальное |
| INT8 | 8-bit  | высокое | хорошее |
| **Q4_K_M** | **4-bit** | низкое | почти FP16 |

### Почему Q4_K_M оптимальна
- Умещается в 8–10 GB VRAM  
- Работает на 12–16 GB видеокартах  
- Высокое качество на малых ресурсах  

---

# **10. Выбор модели в WebUI**

В Open WebUI выбрать:

- **qwen2.5:14b-instruct**  
- **qwen2.5-coder:14b**  

---

# **11. System Prompt для программирования**

Открыть чат → System Prompt:

Ты — профессиональный AI-программист.
Пишешь структурированный и компилируемый код.
Не придумываешь несуществующие классы и библиотеки.
Если данных недостаточно — уточняешь.
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

Быстрый ассистент:

**`ollama pull qwen2.5:7b-instruct`**

Лёгкий кодер:

**`ollama pull qwen2.5-coder:7b`**

---

# **14. Установка на RED OS**

Установить Docker:

**`sudo dnf install docker`**

Включить NVIDIA runtime:

**`sudo nvidia-ctk runtime configure --runtime=docker`**

Перезапустить Docker:

**`sudo systemctl restart docker`**

Разрешить порты:

**`sudo firewall-cmd --add-port=3000/tcp --permanent`**  
**`sudo firewall-cmd --add-port=11434/tcp --permanent`**  
**`sudo firewall-cmd --reload`**

---

# **15. Лицензия**

Проект распространяется по лицензии MIT.  
Полный текст — в файле **LICENSE**.
