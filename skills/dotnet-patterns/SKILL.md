---
name: dotnet-patterns
description: Patrones de arquitectura y diseno para .NET 8 y ASP.NET Core incluyendo Clean Architecture, CQRS, y Vertical Slice
---

# .NET 8 Architecture Patterns

## Cuando Usar
- Diseñando nueva aplicacion .NET
- Refactorizando arquitectura existente
- Decidiendo entre patrones de diseño
- Estructurando proyectos

## Clean Architecture

### Estructura de Proyectos
```
Solution/
├── src/
│   ├── Domain/                  # Entidades, Value Objects, Domain Events
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Events/
│   │   └── Exceptions/
│   │
│   ├── Application/             # Use Cases, Interfaces, DTOs
│   │   ├── Common/
│   │   │   ├── Interfaces/
│   │   │   ├── Behaviors/
│   │   │   └── Exceptions/
│   │   ├── Features/
│   │   │   └── Users/
│   │   │       ├── Commands/
│   │   │       └── Queries/
│   │   └── DependencyInjection.cs
│   │
│   ├── Infrastructure/          # Implementaciones externas
│   │   ├── Persistence/
│   │   │   ├── Configurations/
│   │   │   ├── Repositories/
│   │   │   └── AppDbContext.cs
│   │   ├── Services/
│   │   └── DependencyInjection.cs
│   │
│   └── WebApi/                  # Presentacion
│       ├── Controllers/
│       ├── Middleware/
│       ├── Filters/
│       └── Program.cs
│
└── tests/
    ├── Domain.UnitTests/
    ├── Application.UnitTests/
    ├── Application.IntegrationTests/
    └── WebApi.FunctionalTests/
```

### Regla de Dependencia
```
WebApi → Application → Domain
           ↓
    Infrastructure
```
- Domain no depende de nada
- Application depende solo de Domain
- Infrastructure implementa interfaces de Application
- WebApi orquesta todo

## CQRS con MediatR

### Command (Escritura)
```csharp
// Application/Features/Users/Commands/CreateUser.cs
public record CreateUserCommand(string Email, string Name) : IRequest<Guid>;

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
```

### Query (Lectura)
```csharp
// Application/Features/Users/Queries/GetUsers.cs
public record GetUsersQuery(int Page = 1, int PageSize = 10)
    : IRequest<PagedList<UserDto>>;

public class GetUsersQueryHandler : IRequestHandler<GetUsersQuery, PagedList<UserDto>>
{
    private readonly IAppDbContext _context;

    public GetUsersQueryHandler(IAppDbContext context)
    {
        _context = context;
    }

    public async Task<PagedList<UserDto>> Handle(GetUsersQuery request, CancellationToken ct)
    {
        return await _context.Users
            .AsNoTracking()
            .OrderBy(u => u.Name)
            .Select(u => new UserDto(u.Id, u.Email, u.Name))
            .ToPagedListAsync(request.Page, request.PageSize, ct);
    }
}
```

### Validation con FluentValidation
```csharp
public class CreateUserCommandValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserCommandValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
            .MaximumLength(256);

        RuleFor(x => x.Name)
            .NotEmpty()
            .MinimumLength(2)
            .MaximumLength(100);
    }
}
```

### Pipeline Behavior
```csharp
public class ValidationBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken ct)
    {
        if (_validators.Any())
        {
            var context = new ValidationContext<TRequest>(request);
            var failures = await Task.WhenAll(
                _validators.Select(v => v.ValidateAsync(context, ct)));

            var errors = failures
                .SelectMany(r => r.Errors)
                .Where(f => f != null)
                .ToList();

            if (errors.Count > 0)
                throw new ValidationException(errors);
        }

        return await next();
    }
}
```

## Vertical Slice Architecture

### Feature-based Structure
```
Features/
├── Users/
│   ├── CreateUser/
│   │   ├── CreateUserCommand.cs
│   │   ├── CreateUserHandler.cs
│   │   ├── CreateUserValidator.cs
│   │   └── CreateUserEndpoint.cs
│   ├── GetUser/
│   │   ├── GetUserQuery.cs
│   │   ├── GetUserHandler.cs
│   │   └── GetUserEndpoint.cs
│   └── UserDto.cs
│
└── Orders/
    ├── CreateOrder/
    └── GetOrders/
```

### Minimal API Endpoint
```csharp
// Features/Users/CreateUser/CreateUserEndpoint.cs
public static class CreateUserEndpoint
{
    public static IEndpointRouteBuilder MapCreateUser(this IEndpointRouteBuilder app)
    {
        app.MapPost("/api/users", async (
            CreateUserCommand command,
            ISender sender,
            CancellationToken ct) =>
        {
            var id = await sender.Send(command, ct);
            return Results.Created($"/api/users/{id}", new { id });
        })
        .WithName("CreateUser")
        .WithTags("Users")
        .Produces<object>(StatusCodes.Status201Created)
        .ProducesValidationProblem();

        return app;
    }
}
```

## Domain-Driven Design

### Entity Base
```csharp
public abstract class Entity
{
    public Guid Id { get; protected set; }

    private readonly List<IDomainEvent> _domainEvents = [];
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents;

    protected void AddDomainEvent(IDomainEvent domainEvent)
        => _domainEvents.Add(domainEvent);

    public void ClearDomainEvents() => _domainEvents.Clear();
}
```

### Aggregate Root
```csharp
public class User : Entity
{
    public Email Email { get; private set; }
    public string Name { get; private set; }
    public DateTime CreatedAt { get; private set; }

    private User() { } // EF Core

    public static User Create(string email, string name)
    {
        var user = new User
        {
            Id = Guid.NewGuid(),
            Email = Email.Create(email),
            Name = name,
            CreatedAt = DateTime.UtcNow
        };

        user.AddDomainEvent(new UserCreatedEvent(user.Id));
        return user;
    }

    public void UpdateName(string name)
    {
        Name = name;
        AddDomainEvent(new UserUpdatedEvent(Id));
    }
}
```

### Value Object
```csharp
public sealed class Email : ValueObject
{
    public string Value { get; }

    private Email(string value) => Value = value;

    public static Email Create(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            throw new DomainException("Email cannot be empty");

        if (!email.Contains('@'))
            throw new DomainException("Invalid email format");

        return new Email(email.ToLowerInvariant());
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Value;
    }
}
```

## Result Pattern

```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public Error? Error { get; }

    private Result(T value)
    {
        IsSuccess = true;
        Value = value;
    }

    private Result(Error error)
    {
        IsSuccess = false;
        Error = error;
    }

    public static Result<T> Success(T value) => new(value);
    public static Result<T> Failure(Error error) => new(error);

    public TResult Match<TResult>(
        Func<T, TResult> onSuccess,
        Func<Error, TResult> onFailure)
        => IsSuccess ? onSuccess(Value!) : onFailure(Error!);
}

public record Error(string Code, string Message);
```

## Dependency Injection

### Application Layer
```csharp
// Application/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddMediatR(cfg => {
            cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly());
            cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
            cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
        });

        services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());

        return services;
    }
}
```

### Infrastructure Layer
```csharp
// Infrastructure/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(
                configuration.GetConnectionString("Default"),
                b => b.MigrationsAssembly(typeof(AppDbContext).Assembly.FullName)));

        services.AddScoped<IAppDbContext>(sp => sp.GetRequiredService<AppDbContext>());
        services.AddScoped<IUnitOfWork>(sp => sp.GetRequiredService<AppDbContext>());

        services.AddScoped<IUserRepository, UserRepository>();

        return services;
    }
}
```

## Best Practices

1. **Immutability**: Usar records para DTOs, private setters en entities
2. **Validation**: FluentValidation en Application layer
3. **Exception Handling**: Exceptions en Domain, mapeo a Problem Details en API
4. **Logging**: Structured logging con Serilog
5. **Transactions**: Unit of Work pattern, SaveChangesAsync al final
