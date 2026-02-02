---
name: coding-standards
description: Estandares de codigo para C# .NET 8 y TypeScript/React
---

# Coding Standards

## C# / .NET 8 Standards

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Namespace | PascalCase | `MyApp.Services` |
| Class/Record | PascalCase | `UserService` |
| Interface | IPascalCase | `IUserRepository` |
| Method | PascalCase | `GetUserById` |
| Property | PascalCase | `FirstName` |
| Private field | _camelCase | `_userRepository` |
| Parameter | camelCase | `userId` |
| Local variable | camelCase | `userCount` |
| Constant | PascalCase | `MaxRetries` |
| Enum | PascalCase | `UserStatus.Active` |

### File Organization

```csharp
// 1. Usings (ordenados, sin aliases innecesarios)
using System;
using Microsoft.Extensions.Logging;
using MyApp.Domain.Entities;

// 2. Namespace (file-scoped)
namespace MyApp.Application.Services;

// 3. Tipo principal
public class UserService : IUserService
{
    // 4. Campos privados
    private readonly IUserRepository _repository;
    private readonly ILogger<UserService> _logger;

    // 5. Constructor
    public UserService(IUserRepository repository, ILogger<UserService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    // 6. Propiedades publicas

    // 7. Metodos publicos

    // 8. Metodos privados
}
```

### Modern C# Features (.NET 8)

```csharp
// Records para DTOs inmutables
public record UserDto(Guid Id, string Email, string Name);

// Primary constructors (C# 12)
public class UserService(IUserRepository repository, ILogger<UserService> logger)
{
    public async Task<User?> GetByIdAsync(Guid id) =>
        await repository.GetByIdAsync(id);
}

// Pattern matching
var message = user switch
{
    { IsAdmin: true } => "Welcome, Admin!",
    { IsActive: false } => "Account inactive",
    { Name: var name } => $"Hello, {name}"
};

// Collection expressions
int[] numbers = [1, 2, 3, 4, 5];
List<string> names = ["Alice", "Bob", "Charlie"];

// Nullable reference types
public User? FindUser(string email) // Puede ser null
public User GetUser(Guid id)        // Nunca null (throws si no existe)

// Raw string literals
var json = """
    {
        "name": "John",
        "email": "john@example.com"
    }
    """;
```

### Async/Await

```csharp
// BIEN: Async todo el camino
public async Task<User> GetUserAsync(Guid id, CancellationToken ct = default)
{
    return await _repository.GetByIdAsync(id, ct)
        ?? throw new NotFoundException($"User {id} not found");
}

// BIEN: ConfigureAwait en libraries
public async Task<User> GetUserAsync(Guid id)
{
    return await _repository.GetByIdAsync(id).ConfigureAwait(false);
}

// MAL: Blocking async
public User GetUser(Guid id)
{
    return _repository.GetByIdAsync(id).Result; // Deadlock potential!
}

// MAL: async void (excepto event handlers)
public async void ProcessUser(User user) { } // NO!
```

### Error Handling

```csharp
// Custom exceptions
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

public class ValidationException : Exception
{
    public IReadOnlyList<ValidationError> Errors { get; }

    public ValidationException(IEnumerable<ValidationError> errors)
        : base("Validation failed")
    {
        Errors = errors.ToList();
    }
}

// Result pattern (no exceptions para flujo normal)
public async Task<Result<User>> GetUserAsync(Guid id)
{
    var user = await _repository.GetByIdAsync(id);
    if (user is null)
        return Result.Failure<User>(UserErrors.NotFound(id));

    return Result.Success(user);
}
```

### LINQ Best Practices

```csharp
// BIEN: Encadenar queries
var activeAdmins = users
    .Where(u => u.IsActive)
    .Where(u => u.Role == Role.Admin)
    .OrderBy(u => u.Name)
    .ToList();

// BIEN: Proyeccion para solo campos necesarios
var emails = users.Select(u => u.Email).ToList();

// MAL: Multiple enumerations
var count = users.Count(); // Enumera
var first = users.First(); // Enumera de nuevo

// BIEN: Materializar primero si se usa multiples veces
var usersList = users.ToList();
var count = usersList.Count;
var first = usersList.First();
```

---

## TypeScript / React Standards

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| File (component) | PascalCase | `UserCard.tsx` |
| File (util/hook) | kebab-case | `use-user.ts` |
| Component | PascalCase | `UserCard` |
| Hook | camelCase (use-) | `useUser` |
| Function | camelCase | `formatDate` |
| Variable | camelCase | `userName` |
| Constant | UPPER_SNAKE | `MAX_RETRIES` |
| Interface/Type | PascalCase | `UserProps` |
| Enum | PascalCase | `UserStatus.Active` |

### File Organization

```tsx
// 1. Imports (externos, internos, estilos)
import { useState, useEffect } from 'react';
import { User } from '@/types';
import { formatDate } from '@/lib/utils';
import styles from './user-card.module.css';

// 2. Types/Interfaces
interface UserCardProps {
  user: User;
  onSelect?: (id: string) => void;
}

// 3. Component
export function UserCard({ user, onSelect }: UserCardProps) {
  // a. Hooks
  const [isExpanded, setIsExpanded] = useState(false);

  // b. Derived state
  const fullName = `${user.firstName} ${user.lastName}`;

  // c. Effects
  useEffect(() => {
    // ...
  }, []);

  // d. Handlers
  const handleClick = () => {
    onSelect?.(user.id);
  };

  // e. Render
  return (
    <div className={styles.card} onClick={handleClick}>
      <h3>{fullName}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

### TypeScript Best Practices

```typescript
// Tipos estrictos
interface User {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'user' | 'guest'; // Union types
  createdAt: Date;
}

// Generics
function first<T>(array: T[]): T | undefined {
  return array[0];
}

// Type guards
function isAdmin(user: User): user is User & { role: 'admin' } {
  return user.role === 'admin';
}

// Utility types
type PartialUser = Partial<User>;           // Todo opcional
type RequiredUser = Required<User>;          // Todo requerido
type UserWithoutId = Omit<User, 'id'>;       // Sin id
type UserKeys = keyof User;                   // 'id' | 'email' | ...

// No usar any
function process(data: unknown) {            // BIEN
  if (typeof data === 'string') {
    return data.toUpperCase();
  }
}

function process(data: any) { }              // MAL
```

### React Best Practices

```tsx
// Props con destructuring
function UserCard({ user, onSelect }: UserCardProps) {
  // ...
}

// Default props
function Button({ variant = 'primary', ...props }: ButtonProps) {
  // ...
}

// Avoid inline functions in render
// MAL
<button onClick={() => handleClick(id)}>Click</button>

// BIEN
const handleButtonClick = useCallback(() => {
  handleClick(id);
}, [id, handleClick]);

<button onClick={handleButtonClick}>Click</button>

// Conditional rendering
{isLoading && <Spinner />}
{error ? <Error message={error} /> : <Content />}
{items.length > 0 && <List items={items} />}

// Fragment shorthand
<>
  <Header />
  <Main />
</>
```

---

## Reglas Comunes

### Tamano de Archivos
- **Tipico**: 200-400 lineas
- **Maximo**: 800 lineas
- Si excede, dividir en modulos mas pequenos

### Complejidad
- Metodos/funciones: max 20-30 lineas
- Parametros: max 4-5
- Anidamiento: max 3 niveles

### Comentarios
```csharp
// BIEN: Explicar el "por que", no el "que"
// Usamos retry porque el servicio externo es inestable
await RetryAsync(() => ExternalService.CallAsync());

// MAL: Comentar lo obvio
// Incrementa el contador
counter++;

// BIEN: TODO con contexto
// TODO(issue-123): Implementar paginacion
```

### Imports/Usings
- Ordenar alfabeticamente
- Agrupar por tipo (framework, third-party, local)
- Remover no usados

### Git Commits
```
<type>: <description>

Types: feat, fix, refactor, docs, test, chore, perf, ci

Examples:
feat: add user registration endpoint
fix: resolve null reference in UserService
refactor: extract email validation to separate class
test: add unit tests for CreateUserHandler
```
