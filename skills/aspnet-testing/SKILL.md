---
name: aspnet-testing
description: Guia completa de testing para ASP.NET Core con xUnit, Moq, FluentAssertions, y WebApplicationFactory
---

# ASP.NET Core Testing Guide

## Cuando Usar
- Escribiendo tests para APIs y servicios
- Configurando test infrastructure
- Implementando TDD workflow
- Debuggeando tests fallidos

## Stack de Testing

| Libreria | Proposito |
|----------|-----------|
| xUnit | Test framework |
| Moq | Mocking |
| FluentAssertions | Assertions legibles |
| WebApplicationFactory | Integration tests |
| Testcontainers | DB en containers |
| Bogus | Fake data generation |

## Estructura de Proyectos

```
tests/
├── Domain.UnitTests/
│   └── Entities/
│       └── UserTests.cs
├── Application.UnitTests/
│   └── Features/
│       └── Users/
│           ├── CreateUserHandlerTests.cs
│           └── GetUsersHandlerTests.cs
├── Application.IntegrationTests/
│   ├── Features/
│   │   └── Users/
│   │       └── CreateUserTests.cs
│   └── Common/
│       ├── IntegrationTestBase.cs
│       └── TestDatabase.cs
└── WebApi.FunctionalTests/
    ├── Controllers/
    │   └── UsersControllerTests.cs
    └── Common/
        ├── CustomWebApplicationFactory.cs
        └── FunctionalTestBase.cs
```

## Unit Tests

### Domain Entity Tests
```csharp
public class UserTests
{
    [Fact]
    public void Create_WithValidData_ReturnsUser()
    {
        // Arrange
        var email = "test@example.com";
        var name = "Test User";

        // Act
        var user = User.Create(email, name);

        // Assert
        user.Should().NotBeNull();
        user.Email.Value.Should().Be(email);
        user.Name.Should().Be(name);
        user.Id.Should().NotBeEmpty();
        user.DomainEvents.Should().ContainSingle()
            .Which.Should().BeOfType<UserCreatedEvent>();
    }

    [Theory]
    [InlineData("")]
    [InlineData(null)]
    [InlineData("invalid")]
    public void Create_WithInvalidEmail_ThrowsDomainException(string? email)
    {
        // Act
        var act = () => User.Create(email!, "Name");

        // Assert
        act.Should().Throw<DomainException>()
            .WithMessage("*email*");
    }
}
```

### Handler Tests con Moq
```csharp
public class CreateUserHandlerTests
{
    private readonly Mock<IUserRepository> _repositoryMock;
    private readonly Mock<IUnitOfWork> _unitOfWorkMock;
    private readonly CreateUserCommandHandler _handler;

    public CreateUserHandlerTests()
    {
        _repositoryMock = new Mock<IUserRepository>();
        _unitOfWorkMock = new Mock<IUnitOfWork>();
        _handler = new CreateUserCommandHandler(
            _repositoryMock.Object,
            _unitOfWorkMock.Object);
    }

    [Fact]
    public async Task Handle_WithValidCommand_CreatesUserAndReturnsId()
    {
        // Arrange
        var command = new CreateUserCommand("test@example.com", "Test User");
        _repositoryMock.Setup(r => r.ExistsByEmailAsync(command.Email, default))
            .ReturnsAsync(false);

        // Act
        var result = await _handler.Handle(command, default);

        // Assert
        result.Should().NotBeEmpty();
        _repositoryMock.Verify(r => r.AddAsync(
            It.Is<User>(u => u.Email.Value == command.Email),
            default),
            Times.Once);
        _unitOfWorkMock.Verify(u => u.SaveChangesAsync(default), Times.Once);
    }

    [Fact]
    public async Task Handle_WithDuplicateEmail_ThrowsConflictException()
    {
        // Arrange
        var command = new CreateUserCommand("existing@example.com", "Test");
        _repositoryMock.Setup(r => r.ExistsByEmailAsync(command.Email, default))
            .ReturnsAsync(true);

        // Act
        var act = () => _handler.Handle(command, default);

        // Assert
        await act.Should().ThrowAsync<ConflictException>()
            .WithMessage("*email*already exists*");
    }
}
```

### Validator Tests
```csharp
public class CreateUserCommandValidatorTests
{
    private readonly CreateUserCommandValidator _validator = new();

    [Fact]
    public void Validate_WithValidCommand_ReturnsValid()
    {
        // Arrange
        var command = new CreateUserCommand("test@example.com", "Test User");

        // Act
        var result = _validator.Validate(command);

        // Assert
        result.IsValid.Should().BeTrue();
    }

    [Theory]
    [InlineData("", "Name", "Email")]
    [InlineData("invalid", "Name", "Email")]
    [InlineData("test@example.com", "", "Name")]
    [InlineData("test@example.com", "A", "Name")] // Too short
    public void Validate_WithInvalidData_ReturnsErrors(
        string email,
        string name,
        string expectedError)
    {
        // Arrange
        var command = new CreateUserCommand(email, name);

        // Act
        var result = _validator.Validate(command);

        // Assert
        result.IsValid.Should().BeFalse();
        result.Errors.Should().Contain(e =>
            e.PropertyName.Contains(expectedError, StringComparison.OrdinalIgnoreCase));
    }
}
```

## Integration Tests

### WebApplicationFactory Setup
```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remover DbContext real
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor != null)
                services.Remove(descriptor);

            // Agregar InMemory DbContext
            services.AddDbContext<AppDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb_" + Guid.NewGuid());
            });

            // Crear scope para inicializar DB
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            db.Database.EnsureCreated();
        });
    }
}
```

### API Integration Tests
```csharp
public class UsersControllerTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;
    private readonly CustomWebApplicationFactory _factory;

    public UsersControllerTests(CustomWebApplicationFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task CreateUser_WithValidData_ReturnsCreated()
    {
        // Arrange
        var request = new { Email = "test@example.com", Name = "Test User" };

        // Act
        var response = await _client.PostAsJsonAsync("/api/users", request);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        response.Headers.Location.Should().NotBeNull();

        var content = await response.Content.ReadFromJsonAsync<UserResponse>();
        content!.Email.Should().Be(request.Email);
        content.Name.Should().Be(request.Name);
    }

    [Fact]
    public async Task CreateUser_WithInvalidEmail_ReturnsBadRequest()
    {
        // Arrange
        var request = new { Email = "invalid", Name = "Test" };

        // Act
        var response = await _client.PostAsJsonAsync("/api/users", request);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);

        var problem = await response.Content.ReadFromJsonAsync<ValidationProblemDetails>();
        problem!.Errors.Should().ContainKey("Email");
    }

    [Fact]
    public async Task GetUser_WhenNotExists_ReturnsNotFound()
    {
        // Act
        var response = await _client.GetAsync($"/api/users/{Guid.NewGuid()}");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }
}
```

### Tests con Database Real (Testcontainers)
```csharp
public class DatabaseIntegrationTests : IAsyncLifetime
{
    private readonly MsSqlContainer _sqlContainer = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .Build();

    private AppDbContext _context = null!;

    public async Task InitializeAsync()
    {
        await _sqlContainer.StartAsync();

        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseSqlServer(_sqlContainer.GetConnectionString())
            .Options;

        _context = new AppDbContext(options);
        await _context.Database.MigrateAsync();
    }

    [Fact]
    public async Task CanSaveAndRetrieveUser()
    {
        // Arrange
        var user = User.Create("test@example.com", "Test");

        // Act
        await _context.Users.AddAsync(user);
        await _context.SaveChangesAsync();
        _context.ChangeTracker.Clear();

        var retrieved = await _context.Users.FindAsync(user.Id);

        // Assert
        retrieved.Should().NotBeNull();
        retrieved!.Email.Value.Should().Be("test@example.com");
    }

    public async Task DisposeAsync()
    {
        await _sqlContainer.DisposeAsync();
    }
}
```

## Test Data Builders

### Builder Pattern
```csharp
public class UserBuilder
{
    private Guid _id = Guid.NewGuid();
    private string _email = "default@example.com";
    private string _name = "Default User";
    private bool _isActive = true;

    public UserBuilder WithId(Guid id)
    {
        _id = id;
        return this;
    }

    public UserBuilder WithEmail(string email)
    {
        _email = email;
        return this;
    }

    public UserBuilder WithName(string name)
    {
        _name = name;
        return this;
    }

    public UserBuilder Inactive()
    {
        _isActive = false;
        return this;
    }

    public User Build()
    {
        var user = User.Create(_email, _name);
        // Use reflection or internal setter for testing
        return user;
    }
}

// Uso
var user = new UserBuilder()
    .WithEmail("john@example.com")
    .WithName("John Doe")
    .Build();
```

### Bogus para Fake Data
```csharp
public static class UserFaker
{
    private static readonly Faker<CreateUserCommand> _commandFaker = new Faker<CreateUserCommand>()
        .CustomInstantiator(f => new CreateUserCommand(
            f.Internet.Email(),
            f.Person.FullName
        ));

    public static CreateUserCommand GenerateCommand() => _commandFaker.Generate();

    public static List<CreateUserCommand> GenerateCommands(int count)
        => _commandFaker.Generate(count);
}
```

## Test Naming Convention

```csharp
// Pattern: MethodName_Scenario_ExpectedResult
[Fact]
public async Task CreateUser_WithValidData_ReturnsCreatedUser() { }

[Fact]
public async Task CreateUser_WithDuplicateEmail_ThrowsConflictException() { }

[Fact]
public async Task GetUser_WhenNotFound_ReturnsNull() { }
```

## Coverage

```bash
# Correr con coverage
dotnet test --collect:"XPlat Code Coverage"

# Generar reporte HTML
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"**/coverage.cobertura.xml" -targetdir:"coverage" -reporttypes:Html

# Coverage minimo en CI
dotnet test /p:CollectCoverage=true /p:Threshold=80
```

## CI Integration

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore

      - name: Test
        run: dotnet test --no-build --verbosity normal --collect:"XPlat Code Coverage"

      - name: Upload coverage
        uses: codecov/codecov-action@v4
```
