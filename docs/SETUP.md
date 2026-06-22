# 🚀 $BD Setup Guide

## Prerequisites

- **Node.js** 18.0.0 or higher
- **npm** 8.0.0 or higher (or yarn)
- **PostgreSQL** 12+ (or Supabase account)
- **Git**
- **Code editor** (VSCode recommended)

---

## Quick Start (5 minutes)

### 1. Clone Repository

```bash
git clone https://github.com/SheBadBaddie/-BD-.git
cd -BD-
```

### 2. Install Dependencies

```bash
# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### 3. Setup Environment Variables

**Backend** (`.env`):
```bash
cd ../backend
cp .env.example .env
```

Edit `.env` with:
```
NODE_ENV=development
PORT=5000
DATABASE_URL=postgresql://user:password@localhost:5432/shebad_db
JWT_SECRET=your_super_secret_key_change_this
JWT_EXPIRE=7d
FRONTEND_URL=http://localhost:3000
```

**Frontend** (`.env.local`):
```bash
cd ../frontend
cp .env.example .env.local
```

Edit `.env.local` with:
```
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_WS_URL=ws://localhost:5000
```

### 4. Setup Database

**Option A: Supabase (Recommended)**

1. Go to [supabase.com](https://supabase.com) and create account
2. Create new project
3. Get connection string from Settings → Database → Connection string
4. Paste into `.env` as `DATABASE_URL`

**Option B: Local PostgreSQL**

```bash
# Create database
createdb shebad_db

# Set connection string in .env
```

### 5. Start Development Servers

**Terminal 1 - Backend:**
```bash
cd backend
npm run dev
# Server running on http://localhost:5000
```

**Terminal 2 - Frontend:**
```bash
cd frontend
npm start
# App running on http://localhost:3000
```

✅ **$BD is now running locally!**

---

## Next Steps

✅ Backend running on `http://localhost:5000`
✅ Frontend running on `http://localhost:3000`
✅ Database configured

**Next:**
1. Review [Database Schema](./DATABASE.md)
2. Explore [API Endpoints](./API.md)
3. Check [Architecture](./ARCHITECTURE.md)
4. Start building! 🚀

---

**Happy coding! 💗**
