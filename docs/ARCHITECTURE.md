# 🏗️ $BD Architecture Overview

## System Architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│                         Frontend (React)                         │
│  Pages: Home, Explore, Profile, Messages, Admin Dashboard      │
│  State: Context API + Custom Hooks                              │
│  Styling: Tailwind CSS (Pink Baddie Theme)                      │
└───────────────────────────────────────────────────────────────────┬──┘
                             │ HTTP/WebSocket
┌───────────────────────────┴──────────────────────────────────────┬──┐
│                    Backend (Node.js/Express)                    │
│  Routes: /api/auth, /api/users, /api/posts, /api/messages      │
│  Middleware: JWT Auth, Error Handling, Validation              │
│  Services: User, Post, Message, Notification Services          │
└───────────────────────────────────────────────────────────────────┬──┘
                             │ SQL Queries
┌───────────────────────────┴──────────────────────────────────────┐
│                   Database (PostgreSQL)                         │
│  Tables: Users, Posts, Comments, Messages, Reports, Blocks     │
│  Indexes: Optimized for queries                                 │
│  Security: Row-level security, encryption                      │
└──────────────────────────────────────────────────────────────────┘
```

---

## Key Components

### 1. Authentication Flow
```
User Registration → Email Verification → JWT Token → Authenticated Requests
```

### 2. Anonymous Posting Flow
```
Anonymous User → Anonymous Post → Post to Feed → Users React/Comment
```

### 3. Message Flow
```
Sender → Message DB → WebSocket → Recipient → Notification
```

### 4. Moderation Flow
```
User Reports Post → Report DB → Moderator Review → Action (Remove/Warn)
```

---

## API Layers

### Routes
- `/api/auth` - Authentication
- `/api/users` - User profiles & settings
- `/api/posts` - Create, read, update posts
- `/api/comments` - Comments & replies
- `/api/messages` - Private messaging
- `/api/groups` - Group chats
- `/api/reports` - Report content
- `/api/blocks` - Block/unblock users
- `/api/admin` - Admin dashboard

### Middleware Stack
1. Request logging
2. CORS handling
3. JWT verification
4. Role-based access control (RBAC)
5. Input validation
6. Error handling

---

## Data Flow

### Creating a Post
```
1. Frontend collects post data
2. Frontend sends POST /api/posts with JWT
3. Backend validates JWT & input
4. Backend creates post in DB
5. Backend broadcasts to connected WebSocket clients
6. Frontend updates feed in real-time
```

### Reporting Content
```
1. User clicks "Report" on post/comment
2. Frontend sends POST /api/reports with reason
3. Backend creates report record
4. Backend notifies moderators
5. Moderator reviews & takes action
6. User gets notification of outcome
```

---

## Security Architecture

- **Authentication**: JWT tokens (httpOnly cookies)
- **Authorization**: Role-based access control (RBAC)
- **Encryption**: Password hashing (bcrypt), data encryption at rest
- **Rate Limiting**: Prevent spam & abuse
- **Input Validation**: Sanitize all user inputs
- **CORS**: Restrict cross-origin requests
- **SQL Injection Prevention**: Prepared statements

---

## Scalability Considerations

### Current (MVP)
- Single server, PostgreSQL
- WebSocket for real-time features
- Redis for caching (optional)

### Future (Scale)
- Load balancer
- Multiple backend instances
- Database replication
- CDN for static assets
- Message queue (Redis/RabbitMQ) for async tasks
- Microservices architecture

---

## Real-time Features (WebSocket)

### Events
- `new_post` - New post created
- `new_message` - Private message received
- `new_comment` - Comment on post
- `user_typing` - User is typing
- `notification` - User notification

### Implementation
```javascript
// Backend
io.on('connection', (socket) => {
  socket.on('new_post', (data) => {
    io.emit('post_created', data);
  });
});

// Frontend
socket.on('post_created', (post) => {
  setPosts([post, ...posts]);
});
```

---

## Deployment Architecture

### Development
- Local PostgreSQL
- localhost:5000 (backend)
- localhost:3000 (frontend)

### Production
- Managed PostgreSQL (Supabase/RDS)
- Backend: Render/Railway/Heroku
- Frontend: Vercel/Netlify
- CDN: Cloudflare
- Monitoring: Sentry, LogRocket

---

## Database Relationships

```
Users (1) ──→ (Many) Posts
Users (1) ──→ (Many) Comments
Users (1) ──→ (Many) Messages
Users (1) ──→ (Many) Blocks
Users (1) ──→ (Many) Reports
Posts (1) ──→ (Many) Comments
Posts (1) ──→ (Many) Reactions
Messages (1) ──→ (Many) MessageAttachments
```

---

## Error Handling

### Status Codes
- `200` - Success
- `201` - Created
- `400` - Bad request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not found
- `422` - Validation error
- `429` - Rate limited
- `500` - Server error

### Error Response Format
```json
{
  "error": true,
  "message": "User not found",
  "code": "USER_NOT_FOUND",
  "status": 404
}
```

---

## Next Steps

1. Review [Database Schema](./DATABASE.md)
2. Check [API Endpoints](./API.md)
3. Follow [Setup Guide](./SETUP.md)
4. Deploy using [Deployment Guide](./DEPLOYMENT.md)
