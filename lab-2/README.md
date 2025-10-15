# Отчёт по лабораторной работе №2
## Цель работы
Изучение распространённых ошибок при создании Docker-образов и освоение лучших практик написания Dockerfile.


## Созданные файлы

### 1. Исходный код приложения
**app.py**:
```python
print("Hello word!")
```
### 2. "Плохой" Dockerfile
**Dockerfile.bad**:
```bash
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y python3
COPY . /app
CMD python3 /app/app.py
```

### 3. "Хороший" Dockerfile
**Dockerfile.good**:
```bash
FROM python:3.9-slim

RUN apt-get update && apt-get install -y python3
WORKDIR /app
COPY app.py .
CMD ["python3", "app.py"]
```
## Плохие практики
### 1) **ubuntu:latest** - непредсказуемая версия
### 2) **COPY . /app** - копирует всё, а не только нужные файлы  
### 3) **CMD в shell-формате** - хуже обработка сигналов

## Хорошие практики
### 1) **python:3.9-slim** - конкретная версия + маленький образ
### 2) **COPY app.py .** - копируется только нужный файл
### 3) **CMD в exec-формате** - правильная работа с сигналами

## Соберем и запустим каждый Dockerfile:
```bash
docker build --no-cache -t bad-file -f Dockerfile.bad .
docker build --no-cache -t good-file -f Dockerfile.good .
docker run bad-file
docker run good-file
```
### Плохой файл
<img width="1020" height="271" alt="image" src="https://github.com/user-attachments/assets/49d7b9cd-2eda-4a2e-94bf-d1b5a28da228" />

### Хороший файл
<img width="1030" height="514" alt="image" src="https://github.com/user-attachments/assets/17dd62aa-2319-4256-bdae-328159cbe0d8" />

Разница в сборке даже такого просто файла составляет целых 10 секунд!
### Результаты запуска
<img width="710" height="102" alt="image" src="https://github.com/user-attachments/assets/0e255a7b-3cfc-47b4-9b3b-25599d0cde38" />

## Плохие практики при работе с контейнерами
### 1) Оставление ненужных контейнеров
**Что конкретно плохо:** Контейнеры, оставленные работать после выполнения своих задач, продолжают потреблять ресурсы системы. Со временем это приводит к снижению производительности всей системы.
### 2) Хранение данных внутри контейнера
**Что конкретно плохо:** Все изменения файловой системы контейнера, включая важные данные, безвозвратно теряются при его удалении или пересоздании.
