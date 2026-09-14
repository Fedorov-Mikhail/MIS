# API Reference

**Базовый URL:** `http://localhost:8000`
**Формат:** JSON
**Кодировка:** UTF-8

## Аутентификация

Большинство эндпоинтов требуют JWT-токен в заголовке:

```
Authorization: Bearer <access_token>
```

Токен выдаётся в `POST /auth/login` и живёт `JWT_EXPIRE_MINUTES` минут.

---

## Эндпоинты

### POST /auth/register
Создаёт нового пользователя.

**Тело запроса**
```json
{
  "email": "user@example.com",
  "password": "secret123"
}
```

**Ответ 201 Created**
```json
{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2026-09-14T10:00:00Z"
}
```

**Ошибки**
| Код | Причина |
|-----|---------|
| 400 | Email уже занят |
| 422 | Невалидный email / короткий пароль |

---

### POST /auth/login
Аутентифицирует пользователя и возвращает токен.

**Тело запроса**
```json
{
  "email": "user@example.com",
  "password": "secret123"
}
```

**Ответ 200 OK**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

**Ошибки**
| Код | Причина |
|-----|---------|
| 401 | Неверный email или пароль |

---

### POST /auth/logout
Выход. На клиенте токен удаляется, на сервере — добавляется в blacklist (опционально).

**Заголовки:** `Authorization: Bearer <token>`

**Ответ 204 No Content**

---

### GET /auth/me
Возвращает данные текущего пользователя.

**Заголовки:** `Authorization: Bearer <token>`

**Ответ 200 OK**
```json
{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2026-09-14T10:00:00Z"
}
```

**Ошибки**
| Код | Причина |
|-----|---------|
| 401 | Токен отсутствует, истёк или невалиден |

---

## Формат ошибок

Все ошибки возвращаются в едином виде:

```json
{
  "detail": "Invalid credentials"
}
```

## Коды ответов

| Код | Значение |
|-----|----------|
| 200 | Успех |
| 201 | Создано |
| 204 | Успех без тела |
| 400 | Ошибка валидации |
| 401 | Не авторизован |
| 403 | Доступ запрещён |
| 404 | Не найдено |
| 422 | Ошибка схемы запроса |
| 500 | Внутренняя ошибка сервера |

## Пример полного сценария

```bash
# 1. Регистрация
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"secret123"}'

# 2. Вход — сохраняем токен в переменную
TOKEN=$(curl -s -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"secret123"}' \
  | jq -r '.access_token')

# 3. Запрос к защищённому роуту
curl http://localhost:8000/auth/me \
  -H "Authorization: Bearer $TOKEN"
```

## Swagger / OpenAPI

Интерактивная документация доступна автоматически (FastAPI):

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- OpenAPI JSON: http://localhost:8000/openapi.json
