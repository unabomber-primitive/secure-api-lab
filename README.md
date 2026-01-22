# Secure REST API

Защищенное REST API на Flask с реализацией мер безопасности от SQLi, XSS и Broken Auth и автоматизированной проверкой в CI/CD.

## Деплой

```bash
export SECRET_KEY='ваш-случайный-ключ'

python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

## Технологии

- Flask 3.0
- SQLAlchemy (защита от SQL-инъекций)
- JWT (аутентификация)
- bcrypt (хэширование паролей)
- Bandit (SAST)
- Safety (SCA)
- GitHub Actions (CI/CD)

## API Endpoints

### 1. POST /auth/login
Аутентификация пользователя.

**Запрос:**
```json
{
  "username": "admin",
  "password": "admin123"
}
```

**Ответ:**
```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### 2. GET /api/data
Получение списка постов (требуется аутентификация).

**Заголовки:**
```
Authorization: Bearer <token>
```

**Ответ:**
```json
{
  "posts": [
    {
      "id": 1,
      "title": "Post title",
      "content": "Post content",
      "user_id": 1
    }
  ]
}
```

### 3. POST /api/posts
Создание нового поста (требуется аутентификация).

**Заголовки:**
```
Authorization: Bearer <token>
```

**Запрос:**
```json
{
  "title": "New post",
  "content": "Post content"
}
```

**Ответ:**
```json
{
  "message": "Post created",
  "id": 2,
  "title": "New post"
}
```

## Реализованные меры защиты

### 1. Защита от SQL-инъекций (SQLi)

**Реализация:** Flask-SQLAlchemy ORM

**Код:** app.py:88-91, app.py:104-111

SQLAlchemy автоматически использует параметризованные запросы. Все обращения к БД выполняются через ORM:
- `User.query.filter_by(username=username).first()`
- `Post.query.all()`
- `db.session.add(new_post)`

Конкатенация строк для SQL-запросов не используется.

### 2. Защита от XSS (Cross-Site Scripting)

**Реализация:** Функция sanitize_output() с html.escape()

**Код:** app.py:50-57, app.py:96, app.py:120

Все пользовательские данные, возвращаемые в JSON-ответах, проходят через `html.escape()`:
```python
def sanitize_output(data):
    if isinstance(data, str):
        return html.escape(data)
    ...
```

Это предотвращает выполнение вредоносных скриптов в случае, если данные отображаются в браузере.

### 3. Защита от Broken Authentication

#### 3.1 JWT-токены

**Реализация:** PyJWT

**Код:** app.py:28-48, app.py:87-92

JWT-токен выдается при успешной аутентификации с временем жизни 24 часа:
```python
token = jwt.encode({
    'username': user.username,
    'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=24)
}, app.config['SECRET_KEY'], algorithm='HS256')
```

Middleware `@token_required` проверяет токен на всех защищенных эндпоинтах:
- Проверка наличия токена
- Валидация подписи
- Проверка срока действия
- Проверка существования пользователя

#### 3.2 Хэширование паролей

**Реализация:** bcrypt

**Код:** app.py:81, app.py:129, app.py:135

Пароли хэшируются с использованием bcrypt с автоматической генерацией соли:
```python
hashed = bcrypt.hashpw('password'.encode('utf-8'), bcrypt.gensalt())
```

Проверка пароля выполняется безопасным способом:
```python
bcrypt.checkpw(password.encode('utf-8'), user.password_hash.encode('utf-8'))
```

Bcrypt является устойчивым к атакам перебора благодаря настраиваемой сложности вычислений.

## CI/CD Pipeline

### GitHub Actions Workflow

**Файл:** .github/workflows/ci.yml

Pipeline запускается автоматически при:
- Push в ветки main/master
- Создании Pull Request

### SAST: Bandit

Статический анализатор кода для Python, находит распространенные уязвимости:
- Использование небезопасных функций
- Слабые криптографические алгоритмы
- SQL-инъекции
- Жестко закодированные секреты

### SCA: Safety

Проверка зависимостей на известные уязвимости из базы данных CVE.

## Тестовые пользователи

- Username: `admin`, Password: `admin123`
- Username: `user`, Password: `user123`

## Примеры использования

### 1. Аутентификация
```bash
curl -X POST http://localhost:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'
```

### 2. Получение данных
```bash
curl http://localhost:5000/api/data \
  -H "Authorization: Bearer <token>"
```

### 3. Создание поста
```bash
curl -X POST http://localhost:5000/api/posts \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","content":"Content"}'
```
