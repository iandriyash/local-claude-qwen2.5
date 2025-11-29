# Локальная среда работы с моделями Qwen2.5 (Ollama + Open WebUI)

Проект позволяет развернуть локальную систему для работы с большими языковыми моделями (LLM), включая чат-ассистента и ассистента по программированию.  
Система работает полностью офлайн и использует аппаратное ускорение NVIDIA GPU.

Используемые компоненты:
- Ollama — движок локального запуска LLM
- Open WebUI — веб-интерфейс, аналогичный ChatGPT/Claude
- Модели семейства Qwen2.5 (Instruct и Coder)
- Docker / Docker Compose

Поддерживаемые платформы:
- Windows 10/11
- Linux
- RED OS

---

## 1. Требования

1. NVIDIA GPU с поддержкой CUDA (рекомендуется 12–16 GB VRAM или больше)
2. Docker или Docker Desktop
3. На Linux и RED OS требуется установленный драйвер NVIDIA и пакет CUDA Toolkit
4. Операционная система: Windows 10/11 либо Linux

---

## 2. Установка Docker (Windows)

1. Скачать Docker Desktop:
   https://www.docker.com/products/docker-desktop/

2. При установке включить:
   - WSL2 backend
   - NVIDIA GPU support

3. Перезагрузить систему.

4. Проверка установки:

```bash
docker --version
3. Проверка работы GPU внутри Docker
bash
Копировать код
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
Если выводится информация о вашей видеокарте — GPU доступен.

4. Архитектурная схема проекта
pgsql
Копировать код
                        ┌───────────────────────────────┐
                        │        Пользователь            │
                        │   (браузер: localhost:3000)   │
                        └───────────────┬───────────────┘
                                        │ HTTP (3000)
                                        ▼
                        ┌───────────────────────────────┐
                        │          Open WebUI            │
                        │     Веб-интерфейс чата         │
                        └───────────────┬───────────────┘
                                        │ HTTP (11434)
                                        ▼
                        ┌───────────────────────────────┐
                        │            Ollama              │
                        │   Движок локальных LLM         │
                        └───────────────┬───────────────┘
                                        │ Запросы к модели
                                        ▼
                        ┌───────────────────────────────┐
                        │   Qwen2.5-14B / Coder-14B      │
                        │    Квантовка 4-bit (Q4_K_M)    │
                        └───────────────────────────────┘
                                        │
                                        ▼
                        ┌───────────────────────────────┐
                        │         NVIDIA GPU             │
                        │         RTX 4080/4090          │
                        └───────────────────────────────┘
5. Файл docker-compose.yml
Содержимое файла:

yaml
Копировать код
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
6. Запуск проекта
6.1. Клонирование репозитория
bash
Копировать код
git clone https://github.com/YOUR_LOGIN/local-claude-qwen2.5.git
cd local-claude-qwen2.5
6.2. Запуск сервисов
bash
Копировать код
docker compose up -d
6.3. Проверка работы контейнеров
bash
Копировать код
docker ps
Должны быть контейнеры:

ollama

open-webui

6.4. Открытие интерфейса
Открыть в браузере:

arduino
Копировать код
http://localhost:3000
Создать локальный аккаунт.

7. Загрузка моделей Qwen2.5
7.1. Вход в контейнер Ollama
bash
Копировать код
docker exec -it ollama bash
7.2. Основная модель (14 млрд параметров)
bash
Копировать код
ollama pull qwen2.5:14b-instruct
7.3. Модель для программирования (14 млрд параметров)
bash
Копировать код
ollama pull qwen2.5-coder:14b
Обе модели скачиваются в квантовке 4-bit (Q4_K_M).

8. Параметры моделей
Параметры — это количество численных весов, из которых состоит нейронная сеть.

Модель	Параметры	Назначение
Qwen2.5-7B-Instruct	7 млрд	быстрые ответы
Qwen2.5-14B-Instruct	14 млрд	основной ассистент
Qwen2.5-Coder-14B	14 млрд	развитие и анализ кода
Qwen2.5-32B	32 млрд	повышенное качество reasoning

9. Что такое квантовка (4-bit)
Квантовка — это уменьшение разрядности чисел, на которых работает модель.

Сравнение форматов:
Формат	Битность	Требования к VRAM	Качество
FP16	16-bit	очень высокие	максимальное
INT8	8-bit	высокие	хорошее
Q4_K_M	4-bit	низкие	оптимальное

Почему используется Q4_K_M
Умещает 14B модели в 8–10 GB VRAM

Подходит для потребительских видеокарт

Даёт высокое качество анализа кода

Обеспечивает быстрый отклик

10. Выбор модели в Open WebUI
В верхней части интерфейса:

qwen2.5:14b-instruct

qwen2.5-coder:14b

11. Настройка System Prompt (режим ассистента по программированию)
Открыть чат → Меню чата → Системный промпт
Вставить текст:

Копировать код
Ты — профессиональный AI-программист. 
Ты предоставляешь структурированный, корректный и компилируемый код.
Не создаёшь несуществующих классов или директорий.
Если данных недостаточно — задаёшь уточняющие вопросы.
Всегда указываешь структуру директорий, имена файлов и пакеты.
Длинный код разделяешь на файлы.
Работаешь строго и технически грамотно.
12. Управление контейнерами
Остановка:

bash
Копировать код
docker compose down
Запуск:

bash
Копировать код
docker compose up -d
13. Дополнительные модели
Быстрая модель:

bash
Копировать код
ollama pull qwen2.5:7b-instruct
Легкий кодер:

bash
Копировать код
ollama pull qwen2.5-coder:7b
14. Установка на RED OS
Установить Docker через dnf

Установить NVIDIA драйвер и CUDA

Активировать GPU-runtime:

bash
Копировать код
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
Разрешить порты:

bash
Копировать код
sudo firewall-cmd --add-port=3000/tcp --permanent
sudo firewall-cmd --add-port=11434/tcp --permanent
sudo firewall-cmd --reload
15. Лицензия
Проект распространяется на условиях лицензии MIT.
Полный текст лицензии содержится в файле LICENSE.
