---
description: Planificar features full-stack que involucran backend .NET y frontend React
---

# /fullstack-plan

Planifica features que tocan tanto backend (.NET 8) como frontend (React/Next.js).

## Que Hace Este Comando

1. Analiza el requirement completo
2. Diseña API contract (endpoints + DTOs)
3. Planifica backend (entities, handlers, repos)
4. Planifica frontend (components, hooks, pages)
5. Define orden de implementacion
6. Lista tests E2E

## Cuando Usar

- Features que tocan ambas capas
- Nuevos flujos de usuario completos
- Integraciones API-UI
- Refactorizaciones cross-stack

## Output del Plan

```markdown
# Feature: User Registration

## 1. API Contract

### Endpoints
| Method | Path | Request | Response |
|--------|------|---------|----------|
| POST | /api/auth/register | RegisterRequest | AuthResponse |

### DTOs
```csharp
// Backend
public record RegisterRequest(string Email, string Password, string Name);
public record AuthResponse(string Token, UserDto User);
```

```typescript
// Frontend
interface RegisterRequest {
  email: string;
  password: string;
  name: string;
}
```

## 2. Backend Tasks

### Domain
- [ ] User entity
- [ ] Email value object

### Application
- [ ] RegisterCommand + Handler
- [ ] Validator

### Infrastructure
- [ ] UserRepository
- [ ] PasswordHasher
- [ ] Migration

### API
- [ ] POST /api/auth/register endpoint

## 3. Frontend Tasks

### Services
- [ ] authService.register()

### Hooks
- [ ] useAuth hook

### Components
- [ ] RegisterForm
- [ ] FormInput

### Pages
- [ ] /register page

## 4. Orden de Implementacion

### Fase 1: Backend Core
1. User entity + migration
2. RegisterCommand handler
3. API endpoint
4. Unit + integration tests

### Fase 2: Frontend
1. authService
2. RegisterForm component
3. /register page
4. Component tests

### Fase 3: E2E
1. Registration flow test

## 5. Tests

### Backend
- RegisterHandler unit tests
- /api/auth/register integration tests

### Frontend
- RegisterForm component tests
- useAuth hook tests

### E2E
- Complete registration flow
```

## Ejemplo de Uso

```
> /fullstack-plan Implementar login con email y password

Analizando requirement...
Diseñando API contract...
Planificando backend...
Planificando frontend...
```

## Relacionado

- `/dotnet-plan` - Solo backend
- `fullstack-planner` agent
