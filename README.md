# User Feedback Portal

user feedback system with React frontend, Node.js/Express backend, and PostgreSQL database.

## Features

- Submit feedback with categories (suggestions, bug reports, feature requests)
- View all feedback in a dashboard
- Filter and sort feedback
- Responsive design for all devices
- Dark/light theme support

## Tech Stack

- **Frontend**: React, TailwindCSS, shadcn/ui components
- **Backend**: Node.js, Express
- **Database**: PostgreSQL
- **ORM**: Drizzle
- **Form Validation**: Zod
- **State Management**: TanStack Query

## Prerequisites

Before running this application locally, you need to have:

- Node.js (v16 or higher)
- PostgreSQL (v12 or higher)
- npm or yarn

## Running Locally

### 1. Clone the Repository

Download the Zip file

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up PostgreSQL

Make sure PostgreSQL is installed and running on your machine.

Create a new database:

```bash
# Using psql command line
psql -U postgres
CREATE DATABASE myprojectdb;
\q
```


### 4. Environment Variables

Create a `.env` file in the root directory with your PostgreSQL connection details:

```
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/myprojectdb
PGUSER=postgres
PGPASSWORD=your_password
PGHOST=localhost
PGPORT=5432
PGDATABASE=myprojectdb
```

Replace `your_password` with your PostgreSQL password.

### 5. Install dotenv

Add dotenv for loading environment variables:

```bash
npm install dotenv
```

### 6. Update Database Connection

Modify `server/db.ts` to use the dotenv package:

```typescript
import { Pool, neonConfig } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-serverless';
import ws from "ws";
import * as schema from "@shared/schema";
import dotenv from 'dotenv';

// Load environment variables from .env file
dotenv.config();

neonConfig.webSocketConstructor = ws;

if (!process.env.DATABASE_URL) {
  throw new Error(
    "DATABASE_URL must be set. Did you forget to provision a database?",
  );
}

export const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle({ client: pool, schema });
```

### 7. Run Database Migrations

Create the database tables:

```bash
npm run db:push
```

### 8. Start the Application

Start the development server:

```bash
npm run dev
```

The application will be available at: http://localhost:5000

## Project Structure

- `client/` - Frontend React application
  - `src/components/` - Reusable UI components
  - `src/pages/` - Page components for each route
  - `src/lib/` - Utility functions and configurations
- `server/` - Backend Express server
  - `routes.ts` - API endpoint definitions
  - `storage.ts` - Database operations
  - `db.ts` - Database connection setup
- `shared/` - Shared code between frontend and backend
  - `schema.ts` - Database schema and types

## API Endpoints

- `POST /api/feedback` - Submit new feedback
- `GET /api/feedback` - Retrieve feedback with filtering, sorting, and pagination options:
  - `category` - Filter by category (or "all")
  - `sortBy` - Sort by "newest" or "oldest"
  - `page` - Page number for pagination
  - `limit` - Items per page

