# Hospital Management System — Backend

**Role-based REST API** built with **Node.js**, **Express**, **MongoDB**, and **JWT**. This repository provides a secure, modular backend for a hospital management system with role-based access control (RBAC) supporting roles such as `ADMIN`, `DOCTOR`, `NURSE`, `RECEPTIONIST`, and `PATIENT`.

---

## Table of Contents

* [Features](#features)
* [Tech Stack](#tech-stack)
* [Architecture & Folder Structure](#architecture--folder-structure)
* [Getting Started](#getting-started)

  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
  * [Environment Variables](#environment-variables)
  * [Database Seeding (optional)](#database-seeding-optional)
  * [Run the App](#run-the-app)
* [API Overview](#api-overview)

  * [Authentication](#authentication)
  * [User / Role Management](#user--role-management)
  * [Patient Records & Appointments](#patient-records--appointments)
  * [Other endpoints (summary)](#other-endpoints-summary)
* [Authentication & Authorization](#authentication--authorization)
* [Validation, Error Handling & Logging](#validation-error-handling--logging)
* [Testing](#testing)
* [Deployment](#deployment)
* [Security Considerations](#security-considerations)
* [Contributing](#contributing)
* [License](#license)
* [Contact](#contact)

---

## Features

* Role-based access control (RBAC) using JWT access tokens.
* CRUD for users, patients, doctors, appointments, medical records, prescriptions.
* Secure authentication (login/signup) with password hashing (bcrypt).
* Role-protected routes (e.g., only `ADMIN` can create users, doctors can update medical records, patients can view only their records).
* Pagination, filtering and basic search on list endpoints.
* Centralized error handling and request validation (e.g., using Joi or express-validator).
* Sample seed data script to create default roles and an admin user.

---

## Tech Stack

* Node.js (LTS)
* Express
* MongoDB (Mongoose)
* JSON Web Tokens (JWT)
* bcrypt (password hashing)
* express-validator or Joi (request validation)
* dotenv (environment variables)
* Morgan / Winston (logging)
* Nodemon (dev)

---

## Architecture & Folder Structure

A suggested folder structure:

```
backend-hms/
├─ src/
│  ├─ controllers/
│  ├─ models/
│  ├─ routes/
│  ├─ middlewares/
│  │  ├─ auth.middleware.js
│  │  ├─ role.middleware.js
│  │  └─ error.middleware.js
│  ├─ services/
│  ├─ utils/
│  ├─ validators/
│  ├─ config/
│  ├─ seed/
│  └─ app.js
├─ tests/
├─ .env.example
├─ package.json
├─ README.md
└─ server.js
```

Notes:

* `controllers/` contain request handlers.
* `models/` contain Mongoose schemas (User, Role, Patient, Doctor, Appointment, MedicalRecord, Prescription).
* `middlewares/` include `authenticateJWT`, `authorizeRoles`, and `errorHandler`.
* `services/` include business logic separated from controllers.

---

## Getting Started

### Prerequisites

* Node.js (16+ recommended)
* MongoDB (Atlas or local)
* npm or yarn

### Installation

```bash
# clone
git clone https://github.com/<your-username>/hospital-management-backend.git
cd hospital-management-backend

# install dependencies
npm install
# or
# yarn install
```

### Environment Variables

Create a `.env` file in project root (copy from `.env.example`) and configure the following variables:

```env
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net/hms?retryWrites=true&w=majority
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=1d
BCRYPT_SALT_ROUNDS=10
NODE_ENV=development

# Optional
LOG_LEVEL=info
SEED_ADMIN_EMAIL=admin@example.com
SEED_ADMIN_PASSWORD=AdminPassword123!
```

> **Security tip:** Never commit your real `.env` to version control. Use environment secrets on hosting platforms.

### Database Seeding (optional)

Create a small seed script (e.g., `src/seed/seed.js`) that creates roles and a default admin user. Run it with:

```bash
node src/seed/seed.js
```

### Run the App

```bash
# start in dev mode
npm run dev

# start production
npm start
```

Typical `package.json` scripts:

```json
"scripts": {
  "dev": "nodemon server.js",
  "start": "node server.js",
  "seed": "node src/seed/seed.js",
  "test": "jest --runInBand"
}
```

---

## API Overview

> This is a high-level summary. Add full API docs (Swagger/Postman collection) in a separate `docs/` folder.

### Authentication

* `POST /api/auth/register` — register (Admin-only or public depending on your policy)
* `POST /api/auth/login` — login, returns `{ accessToken, user }`
* `POST /api/auth/refresh` — refresh token (if using refresh-token flow)

### User / Role Management (ADMIN only for sensitive actions)

* `GET /api/users` — list users (Admin)
* `GET /api/users/:id` — get user details
* `POST /api/users` — create user (Admin)
* `PUT /api/users/:id` — update user (Admin or owner)
* `DELETE /api/users/:id` — delete user (Admin)
* `PUT /api/users/:id/role` — change user's role (Admin)

### Patient Records & Appointments

* `POST /api/patients` — create patient profile (Receptionist/Admin)
* `GET /api/patients` — list patients (Doctors/Nurses/Admin with filters)
* `GET /api/patients/:id` — patient profile (role limited; patients can access their own)
* `POST /api/appointments` — create appointment (Receptionist or Patient)
* `GET /api/appointments` — list appointments (role-limited)
* `PUT /api/appointments/:id` — update appointment (Receptionist/Doctor)
* `DELETE /api/appointments/:id` — cancel appointment

### Medical Records & Prescriptions

* `POST /api/patients/:id/records` — add medical record (Doctor)
* `GET /api/patients/:id/records` — fetch records (Doctor, Nurse, Admin, and patient-self)
* `POST /api/patients/:id/prescriptions` — create prescription (Doctor)

### Other endpoints (summary)

* `GET /api/schedule` — doctor's schedule
* `GET /api/stats` — admin dashboard stats (patients count, appointments today)

---

## Authentication & Authorization

* Passwords hashed with `bcrypt` using `BCRYPT_SALT_ROUNDS` from env.
* JWT `accessToken` generated on login; include in `Authorization: Bearer <token>` header.
* `authenticateJWT` middleware verifies token and attaches `req.user`.
* `authorizeRoles(...roles)` middleware checks `req.user.role` against allowed roles.

Example usage in route:

```js
const { authenticateJWT, authorizeRoles } = require('../middlewares/auth.middleware');

router.post('/', authenticateJWT, authorizeRoles('RECEPTIONIST', 'ADMIN'), createPatient);
```

---

## Validation, Error Handling & Logging

* Use centralized error handling: return consistent error response shape: `{ status, message, errors? }`.
* Validate request bodies with `express-validator` or `Joi` and return `400` for invalid input.
* Use `morgan` for HTTP request logging in development and `winston` for production logging.

---

## Testing

* Use Jest and supertest for unit and integration tests.
* Example test scripts in `tests/` for auth flows, role checks, and critical endpoints.

Run tests:

```bash
npm test
```

---

## Deployment

* Configure environment variables in your host (Heroku, Render, Railway, DigitalOcean App Platform, or your VPS).
* Use a managed MongoDB (Atlas) for production.
* Use process manager (PM2) or containerize with Docker:

`Dockerfile` (example):

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
CMD ["node", "server.js"]
```

---

## Security Considerations

* Rate-limit auth endpoints to reduce brute force attacks (e.g., `express-rate-limit`).
* Use HTTPS in production.
* Sanitize inputs to avoid injection attacks.
* Store JWT secret & DB credentials securely (do not commit).
* Implement refresh tokens with secure storage (httpOnly cookies) if needed.

---

## Contributing

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit your changes: `git commit -m "feat: add ..."`
4. Push to the branch: `git push origin feat/your-feature`
5. Open a PR describing your changes.

Please write tests for new features and keep commit messages clear.

---

## License

This project is available under the [MIT License](LICENSE).

---
