# 📝 $BD Database Schema

## Overview

$BD uses PostgreSQL with the following tables designed for an anonymous social app.

---

## Tables

### 1. **users**
Stores user account information.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  display_name VARCHAR(100),
  bio TEXT,
  avatar_url VARCHAR(500),
  cover_photo_url VARCHAR(500),
  role ENUM ('user', 'moderator', 'admin') DEFAULT 'user',
  age INT,
  interests TEXT[] DEFAULT '{}', -- Array of interest tags
  is_verified BOOLEAN DEFAULT false,
  is_banned BOOLEAN DEFAULT false,
  is_private BOOLEAN DEFAULT false,
  notification_settings JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_active_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

---

### 2. **posts**
Stores public posts and anonymous confessions/questions.

```sql
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  is_anonymous BOOLEAN DEFAULT false,
  post_type ENUM ('confession', 'question', 'update', 'poll') DEFAULT 'update',
  content TEXT NOT NULL,
  image_url VARCHAR(500),
  tags TEXT[] DEFAULT '{}',
  category VARCHAR(50),
  is_nsfw BOOLEAN DEFAULT false,
  is_deleted BOOLEAN DEFAULT false,
  is_pinned BOOLEAN DEFAULT false,
  reaction_count INT DEFAULT 0,
  comment_count INT DEFAULT 0,
  share_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_posts_category ON posts(category);
CREATE INDEX idx_posts_is_deleted ON posts(is_deleted);
```

---

### 3. **comments**
Stores replies and comments on posts.

```sql
CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  parent_comment_id UUID REFERENCES comments(id) ON DELETE CASCADE,
  is_anonymous BOOLEAN DEFAULT false,
  content TEXT NOT NULL,
  is_deleted BOOLEAN DEFAULT false,
  reaction_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_comments_parent_comment_id ON comments(parent_comment_id);
CREATE INDEX idx_comments_created_at ON comments(created_at DESC);
```

---

### 4. **reactions**
Stores likes, hearts, and other emoji reactions.

```sql
CREATE TABLE reactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  comment_id UUID REFERENCES comments(id) ON DELETE CASCADE,
  reaction_type VARCHAR(50) NOT NULL, -- 'like', 'heart', 'fire', 'laugh', etc.
  created_at TIMESTAMP DEFAULT NOW(),
  CONSTRAINT only_one_target CHECK (
    (post_id IS NOT NULL AND comment_id IS NULL) OR
    (post_id IS NULL AND comment_id IS NOT NULL)
  )
);

CREATE UNIQUE INDEX idx_unique_reaction ON reactions(user_id, post_id, comment_id, reaction_type);
CREATE INDEX idx_reactions_post_id ON reactions(post_id);
CREATE INDEX idx_reactions_comment_id ON reactions(comment_id);
```

---

### 5. **messages**
Stores private direct messages between users.

```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES users(id) ON DELETE CASCADE,
  recipient_id UUID REFERENCES users(id) ON DELETE CASCADE,
  conversation_id UUID, -- Groups related messages
  content TEXT NOT NULL,
  image_url VARCHAR(500),
  is_read BOOLEAN DEFAULT false,
  is_deleted BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  read_at TIMESTAMP
);

CREATE INDEX idx_messages_sender_id ON messages(sender_id);
CREATE INDEX idx_messages_recipient_id ON messages(recipient_id);
CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_created_at ON messages(created_at DESC);
CREATE INDEX idx_messages_is_read ON messages(is_read);
```

---

### 6. **group_chats**
Stores group chat information.

```sql
CREATE TABLE group_chats (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  creator_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  avatar_url VARCHAR(500),
  is_private BOOLEAN DEFAULT false,
  member_count INT DEFAULT 1,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_group_chats_creator_id ON group_chats(creator_id);
```

---

### 7. **group_members**
Stores members of group chats.

```sql
CREATE TABLE group_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  group_id UUID REFERENCES group_chats(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role ENUM ('member', 'moderator', 'admin') DEFAULT 'member',
  joined_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(group_id, user_id)
);

CREATE INDEX idx_group_members_group_id ON group_members(group_id);
CREATE INDEX idx_group_members_user_id ON group_members(user_id);
```

---

### 8. **group_messages**
Stores messages in group chats.

```sql
CREATE TABLE group_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  group_id UUID REFERENCES group_chats(id) ON DELETE CASCADE,
  sender_id UUID REFERENCES users(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  image_url VARCHAR(500),
  is_deleted BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_group_messages_group_id ON group_messages(group_id);
CREATE INDEX idx_group_messages_sender_id ON group_messages(sender_id);
CREATE INDEX idx_group_messages_created_at ON group_messages(created_at DESC);
```

---

### 9. **blocks**
Stores blocked users (for blocking/muting).

```sql
CREATE TABLE blocks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  blocker_id UUID REFERENCES users(id) ON DELETE CASCADE,
  blocked_user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  reason TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(blocker_id, blocked_user_id)
);

CREATE INDEX idx_blocks_blocker_id ON blocks(blocker_id);
CREATE INDEX idx_blocks_blocked_user_id ON blocks(blocked_user_id);
```

---

### 10. **reports**
Stores user reports for moderation.

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID REFERENCES users(id) ON DELETE SET NULL,
  target_type ENUM ('post', 'comment', 'user', 'message') NOT NULL,
  target_id UUID NOT NULL, -- References post, comment, user, or message
  reason VARCHAR(100) NOT NULL,
  description TEXT,
  status ENUM ('pending', 'in_review', 'resolved', 'dismissed') DEFAULT 'pending',
  action_taken VARCHAR(100), -- 'warning', 'suspended', 'banned', etc.
  reviewed_by UUID REFERENCES users(id) ON DELETE SET NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  reviewed_at TIMESTAMP,
  resolved_at TIMESTAMP
);

CREATE INDEX idx_reports_status ON reports(status);
CREATE INDEX idx_reports_created_at ON reports(created_at DESC);
CREATE INDEX idx_reports_target_id ON reports(target_id);
CREATE INDEX idx_reports_reporter_id ON reports(reporter_id);
```

---

### 11. **notifications**
Stores user notifications.

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL, -- 'comment', 'like', 'message', 'follow', etc.
  actor_id UUID REFERENCES users(id) ON DELETE SET NULL,
  target_id UUID, -- References post, comment, or message
  content TEXT,
  is_read BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  read_at TIMESTAMP
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);
```

---

### 12. **sessions**
Stores user login sessions.

```sql
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  token VARCHAR(500) NOT NULL,
  device_info VARCHAR(255),
  ip_address VARCHAR(50),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP,
  last_activity_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token);
CREATE INDEX idx_sessions_is_active ON sessions(is_active);
```

---

### 13. **audit_logs**
Stores admin actions for accountability.

```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID REFERENCES users(id) ON DELETE SET NULL,
  action VARCHAR(100) NOT NULL,
  target_type VARCHAR(50),
  target_id UUID,
  details JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_admin_id ON audit_logs(admin_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at DESC);
```

---

## Relationships Diagram

```
users (1) ──→ (Many) posts
users (1) ──→ (Many) comments
users (1) ──→ (Many) messages (as sender)
users (1) ──→ (Many) messages (as recipient)
users (1) ──→ (Many) reactions
users (1) ──→ (Many) blocks
users (1) ──→ (Many) reports
users (1) ──→ (Many) group_chats
users (1) ──→ (Many) group_members

posts (1) ──→ (Many) comments
posts (1) ──→ (Many) reactions

comments (1) ──→ (Many) reactions
comments (1) ──→ (Many) comments (replies)

group_chats (1) ──→ (Many) group_members
group_chats (1) ──→ (Many) group_messages
```

---

## Migrations

Migrations are stored in `/backend/migrations/` and run automatically on deploy.

---

## Security Features

✅ Foreign key constraints to maintain referential integrity
✅ Soft deletes for data recovery
✅ Timestamps for audit trails
✅ Indexes for query optimization
✅ Row-level security (can be enabled in Supabase)
✅ Encrypted sensitive fields (passwords, tokens)

---

## Next Steps

1. Review [API Endpoints](./API.md)
2. Follow [Setup Guide](./SETUP.md) to initialize database
3. Check [Architecture](./ARCHITECTURE.md) for system design
