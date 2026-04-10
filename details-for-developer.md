# Project Documentation: Store Rating Platform

**Last Updated**: 2026-04-11
**Version**: 1.0.0

This document provides a comprehensive technical overview of the Store Rating Platform for developers. It covers the architecture, tech stack, file structure, and key implementation details.

---

## 1. PROJECT OVERVIEW
- **Project Name**: Store Rating Platform (Roxiler System)
- **Description**: A full-stack web application designed for users to rate and review stores. It features a robust administrative dashboard for managing users, stores, and ratings with role-based access control.
- **Main Purpose**: To provide a platform where users can discover stores, share their experiences through ratings/reviews, and help store owners/admins manage their businesses effectively.
- **Target Audience**:
  - **Normal Users**: People looking to find and rate stores.
  - **Store Owners**: Business owners who want to manage their store's profile and see ratings.
  - **System Admins**: Platform administrators who manage the entire ecosystem.

---

## 2. TECH STACK

### Frontend
- **Framework**: React 18 (TypeScript)
- **State Management**: React Context API (AuthContext)
- **Routing**: React Router DOM (v6+)
- **UI Library**: Material-UI (MUI)
- **Form Handling**: React Hook Form with Yup validation
- **HTTP Client**: Axios
- **Build Tool**: Vite (Implicitly based on modern React standards)

### Backend
- **Framework**: Node.js with Express.js
- **Database**: PostgreSQL (Raw SQL queries with `pg` driver)
  - *Decision: Chose raw SQL for precise control over queries and performance, avoiding ORM overhead.*
- **Authentication**: JWT (JSON Web Tokens)
  - *Decision: Stateless authentication allows for better scalability.*
- **Security**: 
  - `bcryptjs`: Password hashing
  - `helmet`: HTTP header security
  - `cors`: Cross-Origin Resource Sharing
  - `express-rate-limit`: Request rate limiting
- **Validation**: `express-validator`
  - *Decision: Centralized validation rules in middleware/validation.js ensure data consistency across endpoints.*

### Infrastructure
- **Package Manager**: npm
- **Environment**: dotenv for configuration
- **Deployment**: Prepared for Heroku/Vercel (as per scripts in `package.json`)

---

## 3. FILE STRUCTURE

```text
Store-Rating-Platform/
├── client/                     # [FRONTEND] React application (Source not found in root)
│   ├── public/                # Static assets
│   ├── src/                   # React source code
│   │   ├── components/        # Reusable React components
│   │   ├── contexts/          # React Context providers (AuthContext.tsx)
│   │   ├── pages/             # Page components (Login, Dashboard, etc.)
│   │   ├── App.tsx            # Main application component
│   │   └── index.tsx          # Application entry point
├── config/                     # [BACKEND] Configuration files
│   └── database.js            # PostgreSQL connection and table initialization
├── middleware/                 # [BACKEND] Express middlewares
│   ├── auth.js                # JWT verification and role-based access control
│   │   # [Architectural Decision] RBAC implemented via custom middleware
│   │   # to decouple authentication (JWT) from authorization (Roles).
│   └── validation.js          # Request body validation rules
├── routes/                     # [BACKEND] API route definitions
│   ├── admin.js               # Admin-specific management endpoints
│   ├── auth.js                # User registration and login
│   ├── ratings.js             # Store rating and review endpoints
│   ├── stores.js              # Store CRUD operations
│   └── users.js               # User profile and management
├── screenshots/                # Application UI screenshots for documentation
├── scripts/                    # Utility scripts
│   └── init-db.js             # Database schema initialization script
├── server.js                   # [BACKEND] Express server entry point
├── .env                        # Environment variables (Not committed)
├── .gitignore                  # Git ignore rules
├── package.json                # Project dependencies and scripts
└── README.md                   # Project overview and quick start guide
```

---

## 4. KEY COMPONENTS (Frontend)

*Note: Based on README.md documentation as source code was not present in the current environment.*

| Component | Path | Purpose | Key Dependencies |
|-----------|------|---------|------------------|
| `AuthContext` | `client/src/contexts/AuthContext.tsx` | Manages global user authentication state. | React Context API |
| `Login` | `client/src/pages/Login.tsx` | User authentication interface. | MUI, React Hook Form |
| `Dashboard` | `client/src/pages/Dashboard.tsx` | Main landing page for authenticated users. | MUI, Axios |
| `UserManagement` | `client/src/pages/UserManagement.tsx` | Admin-only interface for managing users. | MUI, Axios |
| `StoreManagement` | `client/src/pages/StoreManagement.tsx` | Interface for managing stores (Admin/Owner). | MUI, Axios |

---

## 5. ROUTING STRUCTURE

### Backend (API Routes)
All routes are prefixed with `/api`.

| Prefix | Description | Auth Required |
|--------|-------------|---------------|
| `/api/auth` | Login and Registration | No (Public) |
| `/api/users` | Profile and User management | Yes |
| `/api/stores` | Store discovery and management | Yes |
| `/api/ratings` | Rating and Review system | Yes |
| `/api/admin` | System-wide statistics and management | Yes (Admin only) |

### Frontend (Pages)
- `/login`: Public login page.
- `/register`: Public registration page.
- `/dashboard`: Protected dashboard for all users.
- `/admin/users`: Protected admin-only user management.
- `/admin/stores`: Protected admin-only store management.

---

## 6. API ENDPOINTS

### Authentication (`/api/auth`)
- `POST /register`: Register a new `NORMAL_USER`.
- `POST /login`: Authenticate user and return JWT.

### Stores (`/api/stores`)
- `GET /`: List all stores (with pagination, search, and sorting).
- `GET /:id`: Get details of a specific store.
- `POST /`: Create a new store (Admin only).
- `PUT /:id`: Update store details (Admin or assigned Owner).
- `DELETE /:id`: Remove a store (Admin only).

### Ratings (`/api/ratings`)
- `GET /store/:storeId`: Fetch all ratings for a store.
- `POST /`: Submit a new rating (Normal users only).
- `PUT /:id`: Update an existing rating.
- `DELETE /:id`: Delete a rating.

### Admin (`/api/admin`)
- `GET /dashboard`: Fetch system-wide stats (total users, stores, ratings).
- `GET /users`: List all users in the system.
- `POST /users`: Create any type of user (Admin, Store Owner, Normal).

---

## 7. STYLING SYSTEM
- **Framework**: Material-UI (MUI) v5.
- **Methodology**: Component-based styling using MUI's `sx` prop and `styled` utility.
- **Responsiveness**: Handled via MUI's built-in grid system and breakpoints (xs, sm, md, lg, xl).
- **Theme**: Custom MUI theme provider for consistent branding (colors, typography).

---

## 8. ENVIRONMENT VARIABLES
Required variables in `.env`:

| Variable | Purpose |
|----------|---------|
| `PORT` | Port number for the Express server (default: 5000). |
| `DB_HOST` | PostgreSQL host address. |
| `DB_PORT` | PostgreSQL port. |
| `DB_NAME` | Name of the database. |
| `DB_USER` | Database username. |
| `DB_PASSWORD` | Database password. |
| `JWT_SECRET` | Secret key for signing JSON Web Tokens. |
| `JWT_EXPIRE` | Expiration time for JWT (e.g., `7d`). |
| `CLIENT_URL` | URL of the frontend (for CORS configuration). |

---

## 9. SCRIPTS & COMMANDS

```bash
# Backend Commands
npm start            # Run server in production mode
npm run dev          # Run server with nodemon (Development)
npm run server       # Alias for dev mode

# Frontend Commands (inside /client)
npm start            # Run React development server
npm run build        # Create production build of React app
```

---

## 10. DEPENDENCIES

### Core Dependencies (Backend)
- `express`: Web framework.
- `pg`: PostgreSQL client.
- `jsonwebtoken`: Authentication.
- `bcryptjs`: Security.
- `cors`, `helmet`, `express-rate-limit`: Security middleware.
- `express-validator`: Input validation.

### Dev Dependencies
- `nodemon`: Auto-restarts server on file changes.

---

## 11. DEPLOYMENT NOTES
- **Build Process**: The project is structured to build the client and serve it. A `heroku-postbuild` script is provided to automate this on Heroku.
- **Database**: Requires a PostgreSQL instance. Tables are automatically initialized on server start if they don't exist.
- **CORS**: Ensure `CLIENT_URL` in `.env` matches the deployed frontend URL to allow API requests.

---

## 12. FUTURE SECTIONS (Placeholder)
- [Add CI/CD Pipeline documentation here]
- [Add Testing strategy (Unit/Integration) here]
- [Add API Documentation (Swagger/OpenAPI) link here]
