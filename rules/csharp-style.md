# C# Code Style Rules

Estas reglas son OBLIGATORIAS para todo codigo C# en el proyecto.

## Principios Fundamentales

### 1. Immutability (CRITICO)

```csharp
// SIEMPRE: Records para DTOs
public record UserDto(Guid Id, string Email, string Name);

// SIEMPRE: Private setters en entities
public class User
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
}

// NUNCA: Modificar objetos, crear nuevos
// MAL
user.Name = "New Name";

// BIEN
var updatedUser = user with { Name = "New Name" }; // Para records
user.UpdateName("New Name"); // Para entities con metodo
```

### 2. Null Safety

```csharp
// SIEMPRE: Nullable reference types habilitados
#nullable enable

// SIEMPRE: Indicar nullabilidad explicitamente
public User? FindByEmail(string email);  // Puede ser null
public User GetById(Guid id);            // Nunca null, throws

// SIEMPRE: Null checks con pattern matching
if (user is null) return NotFound();
if (user is { IsActive: true }) { }

// NUNCA: Suprimir warnings de null sin razon
user!.Name // Solo si REALMENTE sabes que no es null
```

### 3. Async/Await

```csharp
// SIEMPRE: Suffix Async en metodos async
public async Task<User> GetUserAsync(Guid id, CancellationToken ct = default);

// SIEMPRE: Propagar CancellationToken
public async Task ProcessAsync(CancellationToken ct)
{
    await _service.DoWorkAsync(ct);
}

// NUNCA: Blocking calls
var result = GetUserAsync(id).Result;  // Deadlock!
var result = GetUserAsync(id).Wait();  // Deadlock!

// NUNCA: async void (excepto event handlers)
public async void DoSomething() { }  // NO!
```

## Organizacion de Codigo

### Estructura de Archivo

```csharp
// 1. Usings (ordenados)
using System;
using System.Collections.Generic;
using Microsoft.Extensions.DependencyInjection;
using MyApp.Domain;

// 2. Namespace (file-scoped)
namespace MyApp.Application.Services;

// 3. Tipo
public class UserService : IUserService
{
    // a. Campos readonly privados
    private readonly IUserRepository _repository;
    private readonly ILogger<UserService> _logger;

    // b. Constructor
    public UserService(IUserRepository repository, ILogger<UserService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    // c. Propiedades publicas

    // d. Metodos publicos (ordenados por importancia)

    // e. Metodos privados
}
```

### Tamano de Archivos

- **Maximo**: 800 lineas
- **Tipico**: 200-400 lineas
- Si excede, dividir responsabilidades

### Tamano de Metodos

- **Maximo**: 30 lineas
- **Ideal**: 10-15 lineas
- Extraer a metodos privados si es largo

## Naming

```csharp
// PascalCase: Clases, Records, Metodos, Propiedades
public class UserService { }
public record UserDto { }
public void ProcessUser() { }
public string UserName { get; }

// camelCase: Parametros, variables locales
public void Process(string userName, int userId) { }

// _camelCase: Campos privados
private readonly ILogger _logger;
private int _retryCount;

// IPascalCase: Interfaces
public interface IUserRepository { }

// Sufijos descriptivos
public class UserNotFoundException : Exception { }
public record CreateUserCommand { }
public record GetUserQuery { }
```

## Patrones Obligatorios

### Validation

```csharp
// SIEMPRE: Validar inputs publicos
public async Task<User> CreateAsync(CreateUserCommand command)
{
    ArgumentNullException.ThrowIfNull(command);
    ArgumentException.ThrowIfNullOrEmpty(command.Email);

    // O usar FluentValidation
}
```

### Error Handling

```csharp
// SIEMPRE: Excepciones tipadas
public class UserNotFoundException : Exception
{
    public Guid UserId { get; }
    public UserNotFoundException(Guid userId)
        : base($"User {userId} not found")
    {
        UserId = userId;
    }
}

// SIEMPRE: Try-catch especifico
try
{
    await _externalService.CallAsync();
}
catch (HttpRequestException ex)
{
    _logger.LogError(ex, "External service failed");
    throw new ServiceUnavailableException("External service failed", ex);
}
```

### Logging

```csharp
// SIEMPRE: Structured logging
_logger.LogInformation("User {UserId} created with email {Email}", user.Id, user.Email);

// NUNCA: String interpolation en logs
_logger.LogInformation($"User {user.Id} created"); // NO - no es structured

// SIEMPRE: Log levels apropiados
_logger.LogDebug("...");       // Development debugging
_logger.LogInformation("..."); // Normal operations
_logger.LogWarning("...");     // Unexpected but handled
_logger.LogError(ex, "...");   // Errors
_logger.LogCritical("...");    // System failure
```

## Prohibiciones

### NUNCA usar:

```csharp
// NO: Magic numbers/strings
if (retryCount > 3) { }           // MAL
if (retryCount > MaxRetries) { }  // BIEN

// NO: Comentarios obvios
// Increment counter
counter++;

// NO: Regiones
#region Methods  // NO!

// NO: Console.Write en produccion
Console.WriteLine("Debug"); // NO!

// NO: Hardcoded secrets
var apiKey = "sk-12345"; // NUNCA!

// NO: Catch sin tipo o rethrow incorrecto
catch (Exception) { }      // NO - muy generico
catch (Exception ex) { throw ex; } // NO - pierde stack trace
catch (Exception ex) { throw; }    // BIEN - preserva stack
```

## Checklist Pre-Commit

- [ ] Nullable habilitado y sin warnings
- [ ] No using sin usar
- [ ] No variables sin usar
- [ ] Async/await correcto
- [ ] Logging structured
- [ ] No secrets hardcoded
- [ ] No console.log
- [ ] Tests pasan
