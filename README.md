# Primetade - Task Management API

A full-stack Task Management application with JWT Authentication, Role-Based Access Control, and a responsive React frontend.

## 📋 Assignment Overview

This project is a complete solution for the Backend Developer Intern position at Primetade, featuring:

- **Backend**: Scalable REST API with Node.js/Express and PostgreSQL
- **Frontend**: React.js application for task management
- **Authentication**: JWT with secure password hashing
- **Authorization**: Role-based access (User/Admin)
- **API Documentation**: Interactive Swagger UI

---

## 🏗️ Project Structure

```
primetade/
├── backend/                 # Node.js/Express API
│   ├── src/
│   │   ├── config/         # Database & Swagger config
│   │   ├── controllers/    # Request handlers
│   │   ├── middleware/     # Auth & validation
│   │   ├── models/         # Database models
│   │   ├── routes/         # API routes
│   │   ├── utils/         # Helpers
│   │   └── app.js         # Express app entry
│   ├── package.json
│   └── README.md
├── frontend/              # React.js application
│   ├── src/
│   │   ├── components/    # Reusable components
│   │   ├── context/       # Auth context
│   │   ├── pages/        # Page components
│   │   ├── services/     # API services
│   │   └── App.jsx
│   └── package.json
└── README.md              # This file
```

---

## 🚀 Quick Start

### Prerequisites

- Node.js (v18+)
- PostgreSQL (v14+)
- npm or yarn

### Backend Setup

1. Navigate to backend directory:
```bash
cd backend
```

2. Install dependencies:
```bash
npm install
```

3. Configure environment:
```bash
cp .env.example .env
# Edit .env with your database credentials
```

4. Create PostgreSQL database:
```bash
createdb primetade
```

5. Start the server:
```bash
npm run dev
```

The API will be available at:
- **API Base**: `http://localhost:5000/api`
- **Swagger Docs**: `http://localhost:5000/api-docs`
- **Health Check**: `http://localhost:5000/health`

### Frontend Setup

1. Navigate to frontend directory:
```bash
cd frontend
```

2. Install dependencies:
```bash
npm install
```

3. Start development server:
```bash
npm run dev
```

The frontend will be available at `http://localhost:5173`

---

## 🔐 API Authentication

### Register New User

```bash
POST /api/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securepassword123"
}
```

### Login

```bash
POST /api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "securepassword123"
}
```

### Using Authenticated Routes

Include the access token in the Authorization header:

```
Authorization: Bearer <access_token>
```

---

## 📚 API Endpoints

### Authentication

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/auth/register` | Register new user | Public |
| POST | `/api/auth/login` | Login user | Public |
| POST | `/api/auth/refresh` | Refresh access token | Public |
| GET | `/api/auth/me` | Get current user | Private |
| PUT | `/api/auth/profile` | Update profile | Private |
| POST | `/api/auth/logout` | Logout user | Private |

### Tasks

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/tasks` | Get user's tasks | Private |
| POST | `/api/tasks` | Create task | Private |
| GET | `/api/tasks/:id` | Get single task | Private |
| PUT | `/api/tasks/:id` | Update task | Private |
| DELETE | `/api/tasks/:id` | Delete task | Private |
| GET | `/api/tasks/stats` | Get task statistics | Private |

### Admin Routes

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/tasks/admin` | Get all tasks | Admin |
| PUT | `/api/tasks/admin/:id` | Update any task | Admin |
| DELETE | `/api/tasks/admin/:id` | Delete any task | Admin |

---

## 🔒 Security Features

### Implemented

✅ **Password Hashing**
- bcrypt with 12 salt rounds
- No plain text passwords stored

✅ **JWT Authentication**
- Access tokens (24h expiry)
- Refresh tokens (7d expiry)
- Secure token handling

✅ **Role-Based Access**
- User and Admin roles
- Middleware protection
- Route-level authorization

✅ **Input Validation**
- express-validator for all inputs
- SQL injection prevention
- XSS protection via Helmet

✅ **CORS Configuration**
- Configured for specific frontend origin
- Credential handling

### Best Practices

- Environment variables for sensitive data
- Secure HTTP headers (Helmet)
- Request logging (Morgan)
- Error handling middleware

---

## 📊 Database Schema

### Users Table
```sql
users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  role VARCHAR(20) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
```

### Tasks Table
```sql
tasks (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  status VARCHAR(50) DEFAULT 'pending',
  priority VARCHAR(20) DEFAULT 'medium',
  due_date DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
```

---

## 📈 Scalability Notes

### Current Architecture

This application follows a **monolithic REST API** pattern with clear separation of concerns:

- **Presentation Layer**: Express routes & controllers
- **Business Logic**: Service layer in controllers
- **Data Access**: Model layer with PostgreSQL
- **Security**: Middleware-based authentication & validation

### Scaling Strategy

#### 1. Horizontal Scaling
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Load      │────▶│   API       │────▶│   API       │
│   Balancer  │     │   Server 1  │     │   Server 2  │
└─────────────┘     └─────────────┘     └─────────────┘
                                                    │
                                                    ▼
                                            ┌─────────────┐
                                            │   Redis     │
                                            │   (Session) │
                                            └─────────────┘
```

- Deploy multiple API instances behind a load balancer (Nginx/HAProxy)
- JWT enables stateless authentication for easy scaling
- Use sticky sessions or move to Redis for session storage

#### 2. Caching Layer (Redis)
```
┌─────────────┐
│   Redis     │
│   Cache     │
└──────┬──────┘
       │ Cache hits for:
       │ - Task lists
       │ - User sessions
       │ - Statistics
```

- Cache frequently accessed task data
- Store JWT blacklists for logout
- Implement rate limiting

#### 3. Database Scaling
```
                    ┌─────────────┐
                    │   Master    │
                    │   (Write)   │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
       ┌─────────────┐          ┌─────────────┐
       │   Replica   │          │   Replica   │
       │   (Read)    │          │   (Read)    │
       └─────────────┘          └─────────────┘
```

- Read replicas for heavy read operations
- Connection pooling optimization
- Consider sharding for very large datasets

#### 4. Microservices Architecture (Future)

Split into independent services:
- **Auth Service**: Registration, login, token management
- **Task Service**: CRUD operations for tasks
- **User Service**: User profile management
- **API Gateway**: Single entry point, routing, rate limiting

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│   API       │────▶│   Auth      │
│   Gateway   │     │   Service   │
└─────────────┘     └─────────────┘
       │
       ├──▶┌─────────────┐
       │    │   Task     │
       │    │   Service  │
       │    └─────────────┘
       │
       └──▶┌─────────────┐
            │   User      │
            │   Service   │
            └─────────────┘
```

#### 5. Monitoring & Logging

- **ELK Stack**: Elasticsearch, Logstash, Kibana for centralized logging
- **Prometheus + Grafana**: Metrics and dashboards
- **Distributed Tracing**: Jaeger/Zipkin for request tracking

### Recommended Improvements

| Priority | Feature | Benefit |
|----------|---------|---------|
| 🔴 High | Rate Limiting | Prevent abuse |
| 🔴 High | Unit Tests (Jest) | Code reliability |
| 🟡 Medium | Docker Containerization | Easy deployment |
| 🟡 Medium | CI/CD Pipeline | Automated testing |
| 🟢 Low | Database Migrations | Version control |
| 🟢 Low | API Versioning | Backward compatibility |

---

## 🧪 Testing

### Backend Tests
```bash
cd backend
npm test
```

### Frontend Tests
```bash
cd frontend
npm test
```

---

## 📄 License

MIT License - Free to use and modify.

---

## 👤 Author

**Assignment for Primetade Backend Developer Intern Position**

Built with ❤️ using Node.js, Express, PostgreSQL, React, and modern best practices.

---

## 📞 Support

For questions about this project:
- Email: joydip@primetade.ai
- Technical issues: Check Swagger documentation at `/api-docs`

# primetrade
