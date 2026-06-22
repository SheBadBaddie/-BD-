# 🔗 $BD API Endpoints

## Base URL
```
Development: http://localhost:5000/api
Production: https://api.shebad-bd.com/api
```

---

## Authentication Endpoints

### POST /auth/register
Register a new user account.

**Request:**
```json
{
  "username": "baddie_girl",
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "display_name": "BadGirl",
  "age": 21
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "username": "baddie_girl",
  "email": "user@example.com",
  "display_name": "BadGirl",
  "token": "jwt_token_here"
}
```

---

## Admin Endpoints

### GET /admin/dashboard
Get admin dashboard statistics.

**Headers:**
```
Authorization: Bearer <admin_jwt_token>
```

**Response:** `200 OK`
```json
{
  "total_users": 5000,
  "active_users_today": 2500,
  "total_posts": 50000,
  "pending_reports": 42,
  "banned_users": 15,
  "new_users_today": 85
}
```

---

## Error Responses

All endpoints follow this error format:

```json
{
  "error": true,
  "message": "Description of what went wrong",
  "code": "ERROR_CODE",
  "status": 400
}
```

---

## Next Steps

1. Review [Setup Guide](./SETUP.md)
2. Check [Safety & Moderation](./SAFETY.md)
