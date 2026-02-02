---
description: Implementar features .NET usando Test-Driven Development
---

# /dotnet-tdd

Implementa features en .NET 8 siguiendo el ciclo TDD (RED-GREEN-REFACTOR).

## Que Hace Este Comando

1. Escribe test que falla (RED)
2. Implementa codigo minimo (GREEN)
3. Refactoriza si necesario
4. Repite para cada caso

## Cuando Usar

- Implementando nuevas features
- Corrigiendo bugs (primero test que reproduce)
- Refactorizando codigo existente

## Ciclo TDD

```
┌─────────────────────────────────────────────────────────┐
│   1. RED       →   2. GREEN    →   3. REFACTOR         │
│   Test falla       Test pasa       Mejora codigo        │
└─────────────────────────────────────────────────────────┘
```

## Proceso

### 1. Listar Casos de Test

```markdown
Feature: User Registration
- [ ] Create user with valid data
- [ ] Reject empty email
- [ ] Reject invalid email format
- [ ] Reject duplicate email
- [ ] Hash password before storing
```

### 2. Escribir Primer Test (RED)

```csharp
[Fact]
public async Task CreateUser_WithValidData_ReturnsUserId()
{
    var command = new CreateUserCommand("test@example.com", "SecurePass!");
    var result = await _handler.Handle(command, default);
    result.Should().NotBeEmpty();
}
// Falla: CreateUserCommand no existe
```

### 3. Implementar (GREEN)

```csharp
public record CreateUserCommand(string Email, string Password) : IRequest<Guid>;

public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, Guid>
{
    public Task<Guid> Handle(CreateUserCommand request, CancellationToken ct)
    {
        return Task.FromResult(Guid.NewGuid()); // Minimo para pasar
    }
}
// Test pasa
```

### 4. Refactorizar

```csharp
// Agregar implementacion real manteniendo test verde
public async Task<Guid> Handle(CreateUserCommand request, CancellationToken ct)
{
    var user = User.Create(request.Email, request.Password);
    await _repository.AddAsync(user, ct);
    return user.Id;
}
```

### 5. Siguiente Test

Repetir para cada caso de la lista.

## Coverage Target

- Domain: 90%+
- Application: 85%+
- Infrastructure: 70%+
- WebApi: 75%+

## Ejemplo de Uso

```
> /dotnet-tdd Implementar CreateUser con validacion de email

Listando casos de test:
- [ ] CreateUser_WithValidData_ReturnsUserId
- [ ] CreateUser_WithEmptyEmail_ThrowsValidationError
- [ ] CreateUser_WithInvalidEmail_ThrowsValidationError
...

Comenzando ciclo TDD para caso 1...
```

## Relacionado

- `/dotnet-plan` - Planificar antes de implementar
- `/dotnet-review` - Review despues de implementar
- `aspnet-tdd-guide` agent
