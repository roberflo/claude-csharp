---
description: Planificar implementacion de features en .NET 8 backend
---

# /dotnet-plan

Planifica la implementacion de features para el backend .NET 8.

## Que Hace Este Comando

1. Analiza el requirement
2. Identifica entidades y componentes necesarios
3. Diseña la arquitectura (Clean Architecture/CQRS)
4. Define endpoints de API
5. Lista los tests necesarios
6. Crea plan de implementacion paso a paso

## Cuando Usar

- Antes de implementar una nueva feature
- Para disenar API endpoints
- Para planificar refactorizaciones grandes
- Cuando necesitas definir arquitectura

## Como Funciona

El agente `dotnet-architect` analiza el codebase y crea un plan detallado:

```markdown
## Feature: [Nombre]

### 1. Entidades y DTOs
- User entity
- CreateUserCommand
- UserDto

### 2. Arquitectura
- Clean Architecture
- CQRS con MediatR
- Repository Pattern

### 3. Endpoints
POST /api/users - Crear usuario
GET /api/users/{id} - Obtener usuario

### 4. Tests Necesarios
- [ ] Unit: CreateUserHandlerTests
- [ ] Integration: UsersApiTests

### 5. Orden de Implementacion
1. Domain: User entity
2. Application: Commands/Queries
3. Infrastructure: Repository
4. WebApi: Endpoints
5. Tests
```

## Ejemplo de Uso

```
> /dotnet-plan Implementar registro de usuarios con email y password
```

## Relacionado

- `/dotnet-tdd` - Implementar con TDD
- `/dotnet-review` - Revisar codigo
- `dotnet-architect` agent
