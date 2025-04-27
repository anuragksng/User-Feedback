# User Feedback Portal Architecture

This document explains the architecture and data flow of the User Feedback Portal application.

## System Architecture

The application follows a modern full-stack architecture with clearly separated concerns:

![Architecture Diagram]
```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│             │     │              │     │               │
│ React       │─────│ Express API  │─────│ PostgreSQL DB │
│ Frontend    │     │ Backend      │     │               │
│             │     │              │     │               │
└─────────────┘     └──────────────┘     └───────────────┘
```

### Key Components

1. **Frontend (Client)**
   - React-based SPA (Single Page Application)
   - Communicates with backend via RESTful API calls
   - Manages UI state with React hooks and TanStack Query

2. **Backend (Server)**
   - Express.js API server
   - Handles API requests, validation, and database operations
   - Abstracts database access through a storage interface

3. **Database**
   - PostgreSQL relational database
   - Managed through Drizzle ORM
   - Falls back to in-memory storage if database is unavailable

## Data Flow

### Submitting Feedback

1. User fills out feedback form on the frontend (`/client/src/pages/SubmitFeedback.tsx`)
2. Form data is validated using Zod schemas
3. Frontend makes a POST request to `/api/feedback` endpoint
4. Backend (`/server/routes.ts`) receives the request and validates the data
5. After validation, the storage interface (`/server/storage.ts`) is used to save the feedback
6. The feedback is stored in the PostgreSQL database
7. A success response is sent back to the frontend

```
 User ─── Form Submission ──→ React Frontend ──→ POST /api/feedback ──→ Express Backend
                                                                             │
                                                                             ▼
                               Success Response ←── Database Interaction ←── Storage Interface
```

### Viewing Feedback

1. User navigates to the feedback dashboard (`/client/src/pages/FeedbackDashboard.tsx`)
2. Frontend makes a GET request to `/api/feedback` with optional query parameters for filtering/sorting
3. Backend processes the request using the storage interface
4. Database query is executed with the specified filters and sorting
5. Results are paginated and returned to the frontend
6. Frontend displays the feedback items in a list/grid

```
 User ─── Dashboard Visit ──→ React Frontend ──→ GET /api/feedback ──→ Express Backend
                                                                          │
                                                                          ▼
                               Feedback Data ←── Database Query ←── Storage Interface
```

## Key Design Patterns

1. **Repository Pattern**
   - The `IStorage` interface defines a contract for data access
   - Concrete implementations (`DatabaseStorage` and `MemStorage`) provide different data sources
   - This allows for easy switching between storage mechanisms and testing

2. **Model-View-Controller (MVC)**
   - Models: Defined in `shared/schema.ts`
   - Views: React components in the client directory
   - Controllers: Express routes in `server/routes.ts`

3. **Dependency Injection**
   - Storage implementation is injected at runtime
   - The system can gracefully degrade to in-memory storage if the database is unavailable

## Code Organization

### Shared Types and Schemas (`/shared/schema.ts`)

The heart of the application is the shared schema which defines the data models used by both frontend and backend:

```typescript
// Database tables
export const users = pgTable("users", { ... });
export const feedback = pgTable("feedback", { ... });

// Types for database operations
export type InsertFeedback = z.infer<typeof insertFeedbackSchema>;
export type Feedback = typeof feedback.$inferSelect;
```

### Backend Storage Interface (`/server/storage.ts`)

A key architectural component is the storage interface which abstracts database operations:

```typescript
export interface IStorage {
  getUser(id: number): Promise<User | undefined>;
  getUserByUsername(username: string): Promise<User | undefined>;
  createUser(user: InsertUser): Promise<User>;
  createFeedback(feedback: InsertFeedback): Promise<Feedback>;
  getFeedback(
    category: string,
    sortBy: string,
    page: number,
    limit: number
  ): Promise<{
    feedbacks: Feedback[];
    totalCount: number;
    pageCount: number;
  }>;
}
```

### Frontend Data Fetching (`/client/src/lib/queryClient.ts`)

Data fetching and caching is handled through TanStack Query:

```typescript
// API request function
export async function apiRequest(
  endpoint: string,
  options?: RequestInit
): Promise<any> {
  const response = await fetch(endpoint, options);
  await throwIfResNotOk(response);
  return response.json();
}

// Query client configuration
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      refetchOnWindowFocus: true,
      retry: false,
    },
  },
});
```

## Error Handling Strategy

The application implements a multi-layered error handling approach:

1. **Frontend Form Validation**
   - Client-side validation using Zod schemas
   - Immediate feedback for users

2. **API Request Error Handling**
   - HTTP status codes for different error types
   - Consistent error response format

3. **Backend Validation**
   - Server-side validation using the same Zod schemas
   - Prevents invalid data from reaching the database

4. **Database Fallback**
   - Graceful degradation to in-memory storage if database is unavailable

## Extensibility

The architecture is designed to be extensible:

1. Adding new feedback fields requires changes to:
   - `shared/schema.ts` (database schema)
   - Form components
   - Display components

2. Adding new features like user authentication would involve:
   - Extending the storage interface
   - Adding new API routes
   - Creating new frontend components

## Security Considerations

1. **Input Validation**
   - All user input is validated using Zod schemas

2. **Database Security**
   - Environment variables for database credentials
   - No direct SQL queries (ORM provides query parameterization)

3. **API Security**
   - Error messages don't leak implementation details
   - Proper HTTP status codes
