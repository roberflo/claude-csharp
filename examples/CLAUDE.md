# Project: MyApp

## Stack

- **Backend**: .NET 8, ASP.NET Core, Entity Framework Core 8
- **Frontend**: Next.js 14, React 18, TypeScript 5
- **Database**: SQL Server / PostgreSQL
- **Testing**: xUnit, Moq, Vitest, Testing Library

## Critical Rules

### 1. Code Organization

```
Backend: Clean Architecture
- Domain: Entities, Value Objects
- Application: Commands, Queries, Handlers
- Infrastructure: EF Core, External Services
- WebApi: Controllers, Middleware

Frontend: Feature-based
- app/: Next.js App Router
- components/: Reusable UI
- hooks/: Custom hooks
- lib/: Utilities
```

### 2. Code Style

#### C#
- Records para DTOs
- Nullable reference types habilitados
- Async/await todo el camino
- FluentValidation para validacion

#### TypeScript/React
- Server Components por defecto
- 'use client' solo cuando necesario
- Zustand para global state
- TanStack Query para server state

### 3. Testing

- TDD workflow obligatorio
- 80%+ coverage minimo
- Unit tests + Integration tests
- E2E con Playwright

### 4. Security

- No secrets en codigo
- JWT con refresh tokens
- Input validation en todas las capas
- CORS restrictivo

## File Structure

```
/
├── backend/
│   ├── src/
│   │   ├── Domain/
│   │   ├── Application/
│   │   ├── Infrastructure/
│   │   └── WebApi/
│   └── tests/
│       ├── Domain.UnitTests/
│       ├── Application.UnitTests/
│       └── WebApi.IntegrationTests/
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   └── types/
│   └── __tests__/
│
└── docs/
```

## Key Patterns

### API Endpoints
```
GET    /api/{resource}         List
POST   /api/{resource}         Create
GET    /api/{resource}/{id}    Get by ID
PUT    /api/{resource}/{id}    Update
DELETE /api/{resource}/{id}    Delete
```

### State Management
- Local state: useState
- Server state: TanStack Query
- Global state: Zustand (solo cuando necesario)

### Error Handling
- Result<T> pattern en backend
- Error boundaries en frontend
- Structured logging con Serilog

## Environment Variables

```bash
# Backend
DATABASE_URL=
JWT_KEY=
JWT_ISSUER=
JWT_AUDIENCE=

# Frontend
NEXT_PUBLIC_API_URL=
```

## Available Commands

### Backend
```bash
dotnet build                    # Build
dotnet test                     # Tests
dotnet run --project WebApi     # Run
dotnet ef migrations add Name   # New migration
dotnet ef database update       # Apply migrations
```

### Frontend
```bash
npm run dev      # Development
npm run build    # Production build
npm test         # Tests
npm run lint     # Linting
```

## Workflow

1. `/dotnet-plan` o `/fullstack-plan` - Planificar feature
2. `/dotnet-tdd` o `/react-tdd` - Implementar con TDD
3. `/dotnet-review` o `/react-review` - Code review
4. Commit con conventional commits
