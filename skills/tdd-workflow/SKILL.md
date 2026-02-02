---
name: tdd-workflow
description: Metodologia Test-Driven Development para .NET y React con el ciclo RED-GREEN-REFACTOR
---

# Test-Driven Development Workflow

## El Ciclo TDD

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   1. RED      →   2. GREEN    →   3. REFACTOR   →      │
│   (Test falla)    (Test pasa)     (Mejora codigo)       │
│                                                         │
│              ↻ REPEAT para cada caso                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Fase 1: RED (Escribir Test que Falla)

### Principios
- Escribir el test ANTES del codigo de produccion
- El test debe fallar por la razon correcta
- Escribir el test mas simple que demuestre el comportamiento

### .NET Example
```csharp
// 1. Escribir test para funcionalidad que no existe
[Fact]
public async Task CreateUser_WithValidData_ReturnsUser()
{
    // Arrange
    var command = new CreateUserCommand("test@example.com", "Test User");
    var handler = new CreateUserCommandHandler(_repository, _unitOfWork);

    // Act
    var result = await handler.Handle(command, default);

    // Assert
    result.Should().NotBeEmpty();
}
// Error: CreateUserCommand no existe
// Error: CreateUserCommandHandler no existe
```

### React Example
```tsx
// 1. Escribir test para componente que no existe
it('renders user name', () => {
  render(<UserCard user={{ id: '1', name: 'John' }} />);
  expect(screen.getByText('John')).toBeInTheDocument();
});
// Error: UserCard is not defined
```

## Fase 2: GREEN (Hacer Pasar el Test)

### Principios
- Escribir el codigo MINIMO para pasar el test
- No sobre-disenar ni anticipar features
- Esta bien si el codigo es "feo" - lo mejoraremos

### .NET Example
```csharp
// 2. Implementacion minima
public record CreateUserCommand(string Email, string Name) : IRequest<Guid>;

public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, Guid>
{
    public Task<Guid> Handle(CreateUserCommand request, CancellationToken ct)
    {
        // Implementacion minima para pasar
        return Task.FromResult(Guid.NewGuid());
    }
}
// Test pasa ✓
```

### React Example
```tsx
// 2. Implementacion minima
function UserCard({ user }: { user: User }) {
  return <div>{user.name}</div>;
}
// Test pasa ✓
```

## Fase 3: REFACTOR (Mejorar el Codigo)

### Principios
- Mejorar la estructura sin cambiar comportamiento
- Los tests deben seguir pasando
- Eliminar duplicacion, mejorar nombres, simplificar

### .NET Example
```csharp
// 3. Refactorizar con implementacion real
public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, Guid>
{
    private readonly IUserRepository _repository;
    private readonly IUnitOfWork _unitOfWork;

    public CreateUserCommandHandler(IUserRepository repository, IUnitOfWork unitOfWork)
    {
        _repository = repository;
        _unitOfWork = unitOfWork;
    }

    public async Task<Guid> Handle(CreateUserCommand request, CancellationToken ct)
    {
        var user = User.Create(request.Email, request.Name);
        await _repository.AddAsync(user, ct);
        await _unitOfWork.SaveChangesAsync(ct);
        return user.Id;
    }
}
// Test sigue pasando ✓
```

## Workflow Completo: Feature Example

### Requerimiento
"Como usuario, quiero registrarme con email y password"

### Paso 1: Listar Casos de Test
```markdown
- [ ] Create user with valid email and password
- [ ] Reject empty email
- [ ] Reject invalid email format
- [ ] Reject password shorter than 8 characters
- [ ] Reject duplicate email
- [ ] Hash password before storing
```

### Paso 2: Empezar con el Caso Mas Simple

```csharp
// Test 1: Happy path
[Fact]
public async Task RegisterUser_WithValidData_ReturnsUserId()
{
    // Arrange
    var command = new RegisterUserCommand("test@example.com", "SecurePass123!");

    // Act
    var result = await _handler.Handle(command, default);

    // Assert
    result.IsSuccess.Should().BeTrue();
    result.Value.Should().NotBeEmpty();
}
```

### Paso 3: Agregar Tests para Edge Cases

```csharp
// Test 2: Empty email
[Fact]
public async Task RegisterUser_WithEmptyEmail_ReturnsValidationError()
{
    var command = new RegisterUserCommand("", "SecurePass123!");

    var result = await _handler.Handle(command, default);

    result.IsSuccess.Should().BeFalse();
    result.Error.Code.Should().Be("Validation.Email");
}

// Test 3: Invalid email
[Theory]
[InlineData("notanemail")]
[InlineData("missing@domain")]
[InlineData("@nodomain.com")]
public async Task RegisterUser_WithInvalidEmail_ReturnsValidationError(string email)
{
    var command = new RegisterUserCommand(email, "SecurePass123!");

    var result = await _handler.Handle(command, default);

    result.IsSuccess.Should().BeFalse();
}

// Test 4: Short password
[Fact]
public async Task RegisterUser_WithShortPassword_ReturnsValidationError()
{
    var command = new RegisterUserCommand("test@example.com", "short");

    var result = await _handler.Handle(command, default);

    result.IsSuccess.Should().BeFalse();
    result.Error.Code.Should().Be("Validation.Password");
}

// Test 5: Duplicate email
[Fact]
public async Task RegisterUser_WithDuplicateEmail_ReturnsConflictError()
{
    var existingEmail = "existing@example.com";
    _repositoryMock.Setup(r => r.ExistsByEmailAsync(existingEmail, default))
        .ReturnsAsync(true);

    var command = new RegisterUserCommand(existingEmail, "SecurePass123!");

    var result = await _handler.Handle(command, default);

    result.IsSuccess.Should().BeFalse();
    result.Error.Code.Should().Be("User.DuplicateEmail");
}

// Test 6: Password is hashed
[Fact]
public async Task RegisterUser_StoresHashedPassword()
{
    var command = new RegisterUserCommand("test@example.com", "SecurePass123!");

    await _handler.Handle(command, default);

    _repositoryMock.Verify(r => r.AddAsync(
        It.Is<User>(u => u.PasswordHash != command.Password),
        default),
        Times.Once);
}
```

## Coverage Target

| Layer | Minimo | Ideal |
|-------|--------|-------|
| Domain | 90% | 95%+ |
| Application | 85% | 90%+ |
| Infrastructure | 70% | 80%+ |
| WebApi | 75% | 85%+ |
| React Components | 80% | 85%+ |
| React Hooks | 90% | 95%+ |

## Anti-Patterns a Evitar

### 1. Escribir Tests Despues
```csharp
// MAL: Escribir codigo primero, tests despues
// Resulta en tests que prueban implementacion, no comportamiento
```

### 2. Tests Demasiado Grandes
```csharp
// MAL: Un test que prueba 10 cosas
[Fact]
public void TestEverything() { /* 100 lineas */ }

// BIEN: Tests pequeños y focalizados
[Fact]
public void CreateUser_WithValidData_ReturnsUser() { }

[Fact]
public void CreateUser_WithInvalidEmail_ThrowsException() { }
```

### 3. Probar Implementacion
```csharp
// MAL: Verificar que se llamo metodo especifico
_mock.Verify(m => m.InternalMethod(), Times.Once);

// BIEN: Verificar comportamiento/resultado
result.Should().Be(expected);
```

### 4. Tests No Deterministas
```csharp
// MAL: Depende de tiempo real
Assert.True(result.CreatedAt < DateTime.Now);

// BIEN: Inyectar IDateTimeProvider
_dateTimeProvider.Setup(d => d.UtcNow).Returns(fixedDate);
```

## Comandos

### .NET
```bash
# Correr tests
dotnet test

# Con coverage
dotnet test --collect:"XPlat Code Coverage"

# Watch mode
dotnet watch test
```

### React
```bash
# Correr tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage
```

## Checklist Pre-Commit

- [ ] Todos los tests pasan
- [ ] Coverage >= 80%
- [ ] No tests comentados
- [ ] No console.log en tests
- [ ] Tests tienen nombres descriptivos
