# Testing Rules

Reglas OBLIGATORIAS para testing en el proyecto.

## Coverage Minimo

| Capa | Minimo | Ideal |
|------|--------|-------|
| Domain | 90% | 95%+ |
| Application | 85% | 90%+ |
| Infrastructure | 70% | 80%+ |
| WebApi | 75% | 85%+ |
| React Components | 80% | 85%+ |
| React Hooks | 90% | 95%+ |

## TDD Workflow

```
1. RED:      Escribir test que falla
2. GREEN:   Codigo MINIMO para pasar
3. REFACTOR: Mejorar sin romper tests
4. REPEAT
```

### SIEMPRE

```csharp
// Escribir test ANTES del codigo
[Fact]
public async Task CreateUser_WithValidData_ReturnsUserId()
{
    // Este test guia la implementacion
}

// Implementar solo lo necesario para pasar
public async Task<Guid> Handle(CreateUserCommand cmd, CancellationToken ct)
{
    // Implementacion minima
}
```

## Estructura de Tests

### .NET

```
tests/
├── Domain.UnitTests/
│   └── Entities/
│       └── UserTests.cs
├── Application.UnitTests/
│   └── Features/
│       └── CreateUserHandlerTests.cs
├── Application.IntegrationTests/
│   └── Features/
│       └── CreateUserTests.cs
└── WebApi.FunctionalTests/
    └── Controllers/
        └── UsersControllerTests.cs
```

### React

```
src/
├── components/
│   ├── UserCard.tsx
│   └── UserCard.test.tsx     # Colocated
├── hooks/
│   ├── use-user.ts
│   └── use-user.test.ts
└── test/
    ├── setup.ts
    └── mocks/
        └── handlers.ts
```

## Naming Convention

```csharp
// Pattern: Method_Scenario_ExpectedResult
[Fact]
public async Task CreateUser_WithValidData_ReturnsUserId() { }

[Fact]
public async Task CreateUser_WithDuplicateEmail_ThrowsConflictException() { }

[Fact]
public async Task GetUser_WhenNotFound_ReturnsNull() { }
```

```tsx
// Pattern: describe what it does
describe('UserCard', () => {
  it('renders user name', () => { });
  it('calls onSelect when clicked', () => { });
  it('shows loading state while fetching', () => { });
});
```

## Arrange-Act-Assert

```csharp
[Fact]
public async Task CreateUser_WithValidData_ReturnsUserId()
{
    // Arrange
    var command = new CreateUserCommand("test@example.com", "Test");
    var handler = new CreateUserCommandHandler(_repo, _uow);

    // Act
    var result = await handler.Handle(command, default);

    // Assert
    result.Should().NotBeEmpty();
    _repo.Verify(r => r.AddAsync(It.IsAny<User>(), default), Times.Once);
}
```

```tsx
it('shows user name', () => {
  // Arrange
  const user = { id: '1', name: 'John' };

  // Act
  render(<UserCard user={user} />);

  // Assert
  expect(screen.getByText('John')).toBeInTheDocument();
});
```

## Mocking

### Cuando Mockear

```csharp
// SI: Dependencias externas
Mock<IHttpClient> httpClient;       // API externa
Mock<IEmailService> emailService;   // Servicio de email
Mock<IDateTimeProvider> dateTime;   // Tiempo

// NO: Logica de dominio
// El dominio debe testearse con valores reales
```

### Best Practices

```csharp
// BIEN: Setup claro
_repoMock.Setup(r => r.GetByIdAsync(userId, default))
    .ReturnsAsync(user);

// BIEN: Verificar interacciones importantes
_repoMock.Verify(r => r.AddAsync(
    It.Is<User>(u => u.Email == expectedEmail),
    default),
    Times.Once);

// EVITAR: Over-mocking
// Si necesitas mockear demasiado, probablemente
// la clase tiene demasiadas dependencias
```

## Integration Tests

```csharp
// Usar WebApplicationFactory
public class UsersApiTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;

    [Fact]
    public async Task CreateUser_ReturnsCreated()
    {
        var response = await _client.PostAsJsonAsync("/api/users", request);
        response.StatusCode.Should().Be(HttpStatusCode.Created);
    }
}
```

```tsx
// Usar MSW para mocks de API
import { server } from '@/test/mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Testing Library Queries

```tsx
// Prioridad de queries (de mejor a peor)
screen.getByRole('button', { name: /submit/i });  // 1. Role
screen.getByLabelText(/email/i);                   // 2. Label
screen.getByPlaceholderText(/search/i);           // 3. Placeholder
screen.getByText(/welcome/i);                      // 4. Text
screen.getByTestId('custom-element');              // 5. TestId (ultimo recurso)
```

## Prohibiciones

### NUNCA

```csharp
// Tests que dependen del orden
[Fact]
public void Test1() { _sharedState = true; }
[Fact]
public void Test2() { Assert.True(_sharedState); }

// Tests que dependen de tiempo real
Assert.True(result.CreatedAt < DateTime.Now);

// Sleeps en tests
await Task.Delay(1000);

// Tests comentados
// [Fact]
// public void OldTest() { }

// console.log en tests de React
console.log('debugging');
```

```tsx
// Index como key en tests
{items.map((item, i) => <Item key={i} />)}

// fireEvent cuando userEvent es mejor
fireEvent.click(button);  // Usar userEvent.click

// waitFor con timeout arbitrario
await new Promise(r => setTimeout(r, 1000));
```

## Comandos

### .NET
```bash
dotnet test                           # Todos
dotnet test --filter "Category=Unit"  # Solo unit
dotnet test --collect:"XPlat Code Coverage"
dotnet watch test                     # Watch mode
```

### React
```bash
npm test                # Todos
npm test -- --watch     # Watch
npm test -- --coverage  # Coverage
npm test -- --ui        # UI mode (Vitest)
```

## Checklist Pre-Merge

- [ ] Coverage >= 80%
- [ ] No tests skipped o comentados
- [ ] No console.log en tests
- [ ] Todos los tests pasan
- [ ] Tests nuevos para codigo nuevo
- [ ] Tests de regresion para bugs
