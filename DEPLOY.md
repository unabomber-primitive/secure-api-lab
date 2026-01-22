# Инструкция по развертыванию

## Локальное развертывание

### 1. Установка зависимостей

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. Запуск приложения

```bash
python app.py
```

Приложение будет доступно по адресу: http://localhost:5000

### 3. Инициализация БД

База данных SQLite и тестовые пользователи создаются автоматически при первом запуске.

## Загрузка на GitHub и запуск CI/CD

### Шаг 1: Инициализация локального репозитория

```bash
cd /home/seraphima/Документы/labs/mrkn/lab1
git init
git add .
git commit -m "Initial commit: Secure REST API implementation"
```

### Шаг 2: Создание репозитория на GitHub

1. Откройте https://github.com
2. Нажмите "New repository"
3. Укажите имя (например, `secure-api-lab`)
4. **Не создавайте** README.md, .gitignore или LICENSE (они уже есть локально)
5. Нажмите "Create repository"

### Шаг 3: Связывание локального и удаленного репозитория

```bash
git remote add origin https://github.com/ваш-username/secure-api-lab.git
git branch -M main
git push -u origin main
```

### Шаг 4: Проверка работы CI/CD

После выполнения `git push`:

1. Откройте репозиторий на GitHub
2. Перейдите во вкладку "Actions"
3. Вы увидите запущенный workflow "Security Checks"
4. Дождитесь завершения (обычно 1-2 минуты)
5. Проверьте статус проверок (должны быть зелеными)

### Шаг 5: Скачивание отчетов

После успешного выполнения pipeline:

1. Откройте завершенный workflow run
2. Прокрутите вниз до секции "Artifacts"
3. Скачайте:
   - bandit-report.json
   - safety-report.json
4. Сделайте скриншоты успешного выполнения для отчета

## Последующие обновления

```bash
git add .
git commit -m "Описание изменений"
git push
```

После каждого push автоматически запускается CI/CD pipeline.

## Настройка production окружения

### 1. Измените SECRET_KEY

В production **обязательно** измените `SECRET_KEY` в app.py:

```python
import os
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY') or 'secret'
```

Установите переменную окружения:
```bash
export SECRET_KEY='ваш-случайный-ключ'
```

### 2. Отключите debug режим

```python
if __name__ == '__main__':
    init_db()
    app.run(debug=False, host='0.0.0.0')
```

### 3. Используйте production WSGI сервер

```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

### 4. Смените базу данных (опционально)

Для production рекомендуется PostgreSQL:

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://user:password@localhost/dbname'
```

## Troubleshooting

### Pipeline падает на bandit

Проверьте конкретные найденные уязвимости в логах Actions и исправьте их.

### Pipeline падает на safety

Обновите уязвимые зависимости:
```bash
pip install --upgrade имя-пакета
pip freeze > requirements.txt
```

### Ошибки при git push

Убедитесь, что remote добавлен правильно:
```bash
git remote -v
```

Если нужно изменить URL:
```bash
git remote set-url origin https://github.com/username/repo.git
```
