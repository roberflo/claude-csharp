# Security Rules

Estas reglas son OBLIGATORIAS. Violaciones de seguridad son bloqueadores de merge.

## Secrets Management

### NUNCA en Codigo

```csharp
// PROHIBIDO
var apiKey = "sk-12345abc";
var connectionString = "Server=...;Password=secret";
var jwtSecret = "my-super-secret-key";

// CORRECTO: Variables de entorno
var apiKey = Environment.GetEnvironmentVariable("API_KEY");

// CORRECTO: User Secrets (desarrollo)
var apiKey = configuration["ApiKey"];

// CORRECTO: Azure Key Vault (produccion)
builder.Configuration.AddAzureKeyVault(vaultUri, credential);
```

### Archivos Sensibles

```gitignore
# SIEMPRE en .gitignore
.env
.env.local
.env.*.local
appsettings.*.json  # si contiene secrets
secrets.json
*.pfx
*.key
credentials.json
```

## Input Validation

### Backend (.NET)

```csharp
// SIEMPRE: Validar todos los inputs
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
            .MaximumLength(100)
            .Matches(@"^[\p{L}\s\-']+$"); // Solo letras, espacios, guiones
    }
}

// SIEMPRE: Sanitizar HTML
var sanitized = HtmlEncoder.Default.Encode(userInput);
```

### Frontend (React)

```tsx
// SIEMPRE: Validar forms
const schema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
});

// SIEMPRE: Escapar contenido dinamico
// React lo hace automaticamente, PERO:
<div dangerouslySetInnerHTML={{ __html: content }} />  // PELIGROSO!

// Si necesitas HTML, usar libreria de sanitizacion
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(content) }} />
```

## SQL Injection Prevention

```csharp
// PROHIBIDO: Concatenacion de strings
var query = $"SELECT * FROM Users WHERE Email = '{email}'";  // VULNERABLE!

// CORRECTO: Parametros
var user = await context.Users
    .FirstOrDefaultAsync(u => u.Email == email);

// CORRECTO: Raw SQL con parametros
var users = await context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Email = {email}")
    .ToListAsync();
```

## Authentication

### JWT Best Practices

```csharp
// SIEMPRE: Expiracion corta para access token
options.TokenValidationParameters = new TokenValidationParameters
{
    ValidateLifetime = true,
    ClockSkew = TimeSpan.Zero,  // Sin tolerancia
};

// Access token: 15-60 minutos
var token = new JwtSecurityToken(
    expires: DateTime.UtcNow.AddMinutes(30),
    // ...
);

// SIEMPRE: Refresh tokens con rotacion
// SIEMPRE: Almacenar refresh tokens hasheados en DB
// SIEMPRE: Revocar al logout
```

### Password Security

```csharp
// SIEMPRE: Hash con algoritmo fuerte
// BCrypt o Argon2, NUNCA MD5/SHA1 solo
public class PasswordHasher : IPasswordHasher<User>
{
    public string HashPassword(User user, string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);
    }

    public PasswordVerificationResult VerifyHashedPassword(
        User user, string hashedPassword, string providedPassword)
    {
        return BCrypt.Net.BCrypt.Verify(providedPassword, hashedPassword)
            ? PasswordVerificationResult.Success
            : PasswordVerificationResult.Failed;
    }
}
```

## Authorization

```csharp
// SIEMPRE: Verificar ownership
public async Task<IActionResult> UpdateDocument(Guid id, UpdateDto dto)
{
    var document = await _repository.GetByIdAsync(id);

    // Verificar que el usuario es dueno
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    if (document.OwnerId.ToString() != userId)
        return Forbid();

    // Proceder con update
}

// SIEMPRE: Principio de minimo privilegio
[Authorize(Policy = "RequireAdminRole")]
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(Guid id) { }
```

## CORS

```csharp
// NUNCA: AllowAnyOrigin con credentials
policy.AllowAnyOrigin().AllowCredentials();  // PROHIBIDO!

// CORRECTO: Origenes especificos
policy.WithOrigins(
    "https://myapp.com",
    "https://www.myapp.com")
    .AllowCredentials();
```

## Rate Limiting

```csharp
// SIEMPRE: Rate limiting en endpoints publicos
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("Auth", context =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: context.Connection.RemoteIpAddress?.ToString(),
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 5,       // 5 intentos
                Window = TimeSpan.FromMinutes(1)  // por minuto
            }));
});

[EnableRateLimiting("Auth")]
[HttpPost("login")]
public async Task<IActionResult> Login() { }
```

## Security Headers

```csharp
// SIEMPRE: Headers de seguridad
app.Use(async (context, next) =>
{
    context.Response.Headers.Append("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Append("X-Frame-Options", "DENY");
    context.Response.Headers.Append("X-XSS-Protection", "1; mode=block");
    context.Response.Headers.Append("Referrer-Policy", "strict-origin-when-cross-origin");
    context.Response.Headers.Append("Content-Security-Policy", "default-src 'self'");
    await next();
});
```

## Logging

```csharp
// NUNCA: Loggear datos sensibles
_logger.LogInformation("User logged in with password {Password}", password);  // PROHIBIDO!
_logger.LogInformation("Card number: {CardNumber}", cardNumber);  // PROHIBIDO!

// CORRECTO: Loggear solo identificadores
_logger.LogInformation("User {UserId} logged in", user.Id);
```

## HTTPS

```csharp
// SIEMPRE: Forzar HTTPS en produccion
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
app.UseHttpsRedirection();

// SIEMPRE: Secure cookies
options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
options.Cookie.HttpOnly = true;
options.Cookie.SameSite = SameSiteMode.Strict;
```

## Checklist de Seguridad

### Pre-Commit
- [ ] No secrets en codigo
- [ ] No secrets en logs
- [ ] Input validation implementada
- [ ] Queries parametrizadas

### Pre-Deploy
- [ ] HTTPS habilitado
- [ ] Security headers configurados
- [ ] Rate limiting en auth endpoints
- [ ] CORS restrictivo
- [ ] Dependency scan pasado
