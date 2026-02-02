---
name: efcore-patterns
description: Patrones y mejores practicas para Entity Framework Core 8 incluyendo configuracion, queries, y optimizacion
---

# Entity Framework Core 8 Patterns

## Cuando Usar
- Configurando DbContext y entidades
- Optimizando queries LINQ
- Creando migrations
- Implementando Repository pattern

## DbContext Configuration

### Basic Setup
```csharp
public class AppDbContext : DbContext, IAppDbContext, IUnitOfWork
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Aplicar todas las configuraciones del assembly
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());

        base.OnModelCreating(modelBuilder);
    }

    protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
    {
        // Configurar conversiones globales
        configurationBuilder.Properties<DateTime>()
            .HaveConversion<DateTimeUtcConverter>();

        configurationBuilder.Properties<string>()
            .HaveMaxLength(256);
    }
}
```

### Entity Configuration
```csharp
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("Users");

        builder.HasKey(u => u.Id);

        builder.Property(u => u.Email)
            .HasMaxLength(256)
            .IsRequired();

        // Value Object conversion
        builder.Property(u => u.Email)
            .HasConversion(
                e => e.Value,
                s => Email.Create(s));

        // Indice unico
        builder.HasIndex(u => u.Email)
            .IsUnique();

        // Relationship
        builder.HasMany(u => u.Orders)
            .WithOne(o => o.User)
            .HasForeignKey(o => o.UserId)
            .OnDelete(DeleteBehavior.Cascade);

        // Ignorar domain events
        builder.Ignore(u => u.DomainEvents);
    }
}
```

### Soft Delete
```csharp
public interface ISoftDeletable
{
    bool IsDeleted { get; }
    DateTime? DeletedAt { get; }
}

// En DbContext
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Query filter global para soft delete
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        if (typeof(ISoftDeletable).IsAssignableFrom(entityType.ClrType))
        {
            var parameter = Expression.Parameter(entityType.ClrType, "e");
            var property = Expression.Property(parameter, nameof(ISoftDeletable.IsDeleted));
            var condition = Expression.Equal(property, Expression.Constant(false));
            var lambda = Expression.Lambda(condition, parameter);

            modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
        }
    }
}

// Sobrescribir SaveChangesAsync
public override async Task<int> SaveChangesAsync(CancellationToken ct = default)
{
    foreach (var entry in ChangeTracker.Entries<ISoftDeletable>())
    {
        if (entry.State == EntityState.Deleted)
        {
            entry.State = EntityState.Modified;
            entry.Entity.IsDeleted = true;
            entry.Entity.DeletedAt = DateTime.UtcNow;
        }
    }

    return await base.SaveChangesAsync(ct);
}
```

## Query Patterns

### Projection (Select)
```csharp
// BIEN: Solo trae columnas necesarias
var userDtos = await _context.Users
    .Where(u => u.IsActive)
    .Select(u => new UserDto(u.Id, u.Email.Value, u.Name))
    .ToListAsync(ct);

// MAL: Trae toda la entidad
var users = await _context.Users.Where(u => u.IsActive).ToListAsync();
var dtos = users.Select(u => new UserDto(u.Id, u.Email.Value, u.Name));
```

### Eager Loading
```csharp
// Include para relaciones
var orders = await _context.Orders
    .Include(o => o.User)
    .Include(o => o.Items)
        .ThenInclude(i => i.Product)
    .Where(o => o.Status == OrderStatus.Pending)
    .ToListAsync(ct);

// Split Query para evitar cartesian explosion
var orders = await _context.Orders
    .Include(o => o.Items)
    .Include(o => o.Payments)
    .AsSplitQuery()
    .ToListAsync(ct);
```

### No-Tracking Queries
```csharp
// Para queries de solo lectura
var users = await _context.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .ToListAsync(ct);

// Configurar por defecto (cuidado con writes)
optionsBuilder.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
```

### Pagination
```csharp
public record PagedList<T>(
    IReadOnlyList<T> Items,
    int Page,
    int PageSize,
    int TotalCount)
{
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasPrevious => Page > 1;
    public bool HasNext => Page < TotalPages;
}

public static async Task<PagedList<T>> ToPagedListAsync<T>(
    this IQueryable<T> query,
    int page,
    int pageSize,
    CancellationToken ct = default)
{
    var count = await query.CountAsync(ct);
    var items = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync(ct);

    return new PagedList<T>(items, page, pageSize, count);
}
```

### Specification Pattern
```csharp
public abstract class Specification<T> where T : class
{
    public abstract Expression<Func<T, bool>> ToExpression();

    public Specification<T> And(Specification<T> other)
        => new AndSpecification<T>(this, other);

    public Specification<T> Or(Specification<T> other)
        => new OrSpecification<T>(this, other);
}

public class ActiveUsersSpec : Specification<User>
{
    public override Expression<Func<User, bool>> ToExpression()
        => u => u.IsActive && !u.IsDeleted;
}

public class UsersByEmailDomainSpec : Specification<User>
{
    private readonly string _domain;

    public UsersByEmailDomainSpec(string domain) => _domain = domain;

    public override Expression<Func<User, bool>> ToExpression()
        => u => u.Email.Value.EndsWith($"@{_domain}");
}

// Uso
var spec = new ActiveUsersSpec().And(new UsersByEmailDomainSpec("company.com"));
var users = await _context.Users
    .Where(spec.ToExpression())
    .ToListAsync(ct);
```

## Bulk Operations (EF Core 7+)

### ExecuteUpdate
```csharp
// Update masivo sin cargar entidades
await _context.Users
    .Where(u => u.LastLoginAt < DateTime.UtcNow.AddYears(-1))
    .ExecuteUpdateAsync(s => s
        .SetProperty(u => u.IsActive, false)
        .SetProperty(u => u.DeactivatedAt, DateTime.UtcNow),
    ct);
```

### ExecuteDelete
```csharp
// Delete masivo
await _context.AuditLogs
    .Where(l => l.CreatedAt < DateTime.UtcNow.AddMonths(-6))
    .ExecuteDeleteAsync(ct);
```

## Compiled Queries
```csharp
// Para queries frecuentes
private static readonly Func<AppDbContext, Guid, Task<User?>> GetUserById =
    EF.CompileAsyncQuery((AppDbContext ctx, Guid id) =>
        ctx.Users.FirstOrDefault(u => u.Id == id));

// Uso
var user = await GetUserById(_context, userId);
```

## Repository Pattern

### Interface
```csharp
public interface IRepository<T> where T : Entity
{
    Task<T?> GetByIdAsync(Guid id, CancellationToken ct = default);
    Task<IReadOnlyList<T>> GetAllAsync(CancellationToken ct = default);
    Task AddAsync(T entity, CancellationToken ct = default);
    void Update(T entity);
    void Remove(T entity);
}
```

### Implementation
```csharp
public class Repository<T> : IRepository<T> where T : Entity
{
    protected readonly AppDbContext Context;
    protected readonly DbSet<T> DbSet;

    public Repository(AppDbContext context)
    {
        Context = context;
        DbSet = context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(Guid id, CancellationToken ct = default)
        => await DbSet.FindAsync([id], ct);

    public virtual async Task<IReadOnlyList<T>> GetAllAsync(CancellationToken ct = default)
        => await DbSet.AsNoTracking().ToListAsync(ct);

    public virtual async Task AddAsync(T entity, CancellationToken ct = default)
        => await DbSet.AddAsync(entity, ct);

    public virtual void Update(T entity)
        => DbSet.Update(entity);

    public virtual void Remove(T entity)
        => DbSet.Remove(entity);
}
```

### Specialized Repository
```csharp
public interface IUserRepository : IRepository<User>
{
    Task<User?> GetByEmailAsync(string email, CancellationToken ct = default);
    Task<bool> ExistsByEmailAsync(string email, CancellationToken ct = default);
}

public class UserRepository : Repository<User>, IUserRepository
{
    public UserRepository(AppDbContext context) : base(context) { }

    public async Task<User?> GetByEmailAsync(string email, CancellationToken ct = default)
        => await DbSet.FirstOrDefaultAsync(u => u.Email.Value == email, ct);

    public async Task<bool> ExistsByEmailAsync(string email, CancellationToken ct = default)
        => await DbSet.AnyAsync(u => u.Email.Value == email, ct);
}
```

## Migrations

### Commands
```bash
# Crear migration
dotnet ef migrations add AddUserTable -p Infrastructure -s WebApi

# Aplicar
dotnet ef database update -p Infrastructure -s WebApi

# Generar script SQL
dotnet ef migrations script -p Infrastructure -s WebApi --idempotent -o script.sql

# Revertir a migration especifica
dotnet ef database update PreviousMigrationName -p Infrastructure -s WebApi
```

### Best Practices
```csharp
public partial class AddUserTable : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Users",
            columns: table => new
            {
                Id = table.Column<Guid>(nullable: false),
                Email = table.Column<string>(maxLength: 256, nullable: false),
                Name = table.Column<string>(maxLength: 100, nullable: false),
                IsActive = table.Column<bool>(nullable: false, defaultValue: true),
                CreatedAt = table.Column<DateTime>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Users", x => x.Id);
            });

        // Indices
        migrationBuilder.CreateIndex(
            name: "IX_Users_Email",
            table: "Users",
            column: "Email",
            unique: true);

        migrationBuilder.CreateIndex(
            name: "IX_Users_IsActive_CreatedAt",
            table: "Users",
            columns: new[] { "IsActive", "CreatedAt" });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Users");
    }
}
```

## Performance Tips

1. **Indices**: En columnas de WHERE, JOIN, ORDER BY
2. **AsNoTracking**: Para queries de solo lectura
3. **Select/Projection**: Solo columnas necesarias
4. **Batch Size**: Configurar para inserts masivos
5. **Split Queries**: Para muchos Includes
6. **Compiled Queries**: Para queries frecuentes
7. **ExecuteUpdate/Delete**: Para operaciones masivas
