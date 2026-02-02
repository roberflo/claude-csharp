---
name: aspnet-security
description: Guia de seguridad para ASP.NET Core incluyendo autenticacion JWT, autorizacion, CORS, y proteccion contra vulnerabilidades comunes
---

# ASP.NET Core Security Guide

## Cuando Usar
- Implementando autenticacion/autorizacion
- Configurando JWT
- Protegiendo APIs
- Auditando seguridad

## Authentication con JWT

### Configuracion
```csharp
// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
            ClockSkew = TimeSpan.Zero // No tolerancia de tiempo
        };
    });

builder.Services.AddAuthorization();

// Middleware order matters!
app.UseAuthentication();
app.UseAuthorization();
```

### JWT Service
```csharp
public interface IJwtService
{
    string GenerateToken(User user);
    ClaimsPrincipal? ValidateToken(string token);
}

public class JwtService : IJwtService
{
    private readonly IConfiguration _config;
    private readonly byte[] _key;

    public JwtService(IConfiguration config)
    {
        _config = config;
        _key = Encoding.UTF8.GetBytes(_config["Jwt:Key"]!);
    }

    public string GenerateToken(User user)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email.Value),
            new Claim(ClaimTypes.Name, user.Name),
            new Claim(ClaimTypes.Role, user.Role.ToString())
        };

        var credentials = new SigningCredentials(
            new SymmetricSecurityKey(_key),
            SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### Refresh Tokens
```csharp
public record AuthTokens(string AccessToken, string RefreshToken);

public class RefreshToken
{
    public Guid Id { get; set; }
    public Guid UserId { get; set; }
    public string Token { get; set; } = null!;
    public DateTime ExpiresAt { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsRevoked { get; set; }
}

public class AuthService
{
    public async Task<AuthTokens> RefreshAsync(string refreshToken, CancellationToken ct)
    {
        var storedToken = await _tokenRepository.GetByTokenAsync(refreshToken, ct);

        if (storedToken is null || storedToken.IsRevoked || storedToken.ExpiresAt < DateTime.UtcNow)
            throw new UnauthorizedException("Invalid refresh token");

        // Revocar token viejo
        storedToken.IsRevoked = true;

        // Generar nuevos tokens
        var user = await _userRepository.GetByIdAsync(storedToken.UserId, ct);
        var newAccessToken = _jwtService.GenerateToken(user!);
        var newRefreshToken = await CreateRefreshTokenAsync(user!.Id, ct);

        return new AuthTokens(newAccessToken, newRefreshToken);
    }
}
```

## Authorization

### Policy-Based
```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("PremiumUser", policy =>
        policy.RequireClaim("subscription", "premium"));

    options.AddPolicy("MinimumAge", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
});

// Controller
[Authorize(Policy = "AdminOnly")]
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(Guid id) { }
```

### Custom Authorization Handler
```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int age) => MinimumAge = age;
}

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var birthDateClaim = context.User.FindFirst("birth_date");
        if (birthDateClaim is null)
            return Task.CompletedTask;

        var birthDate = DateTime.Parse(birthDateClaim.Value);
        var age = DateTime.Today.Year - birthDate.Year;

        if (age >= requirement.MinimumAge)
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}
```

### Resource-Based Authorization
```csharp
public class DocumentAuthorizationHandler
    : AuthorizationHandler<OperationAuthorizationRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OperationAuthorizationRequirement requirement,
        Document resource)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        if (requirement.Name == Operations.Read.Name)
        {
            if (resource.IsPublic || resource.OwnerId.ToString() == userId)
                context.Succeed(requirement);
        }
        else if (requirement.Name == Operations.Update.Name)
        {
            if (resource.OwnerId.ToString() == userId)
                context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}

// Uso en Controller
public async Task<IActionResult> GetDocument(Guid id)
{
    var document = await _repository.GetByIdAsync(id);

    var authResult = await _authorizationService.AuthorizeAsync(
        User, document, Operations.Read);

    if (!authResult.Succeeded)
        return Forbid();

    return Ok(document);
}
```

## CORS Configuration

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins(
                "https://myapp.com",
                "https://www.myapp.com")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials(); // Para cookies/auth headers
    });

    // Development
    options.AddPolicy("Development", policy =>
    {
        policy.WithOrigins("http://localhost:3000")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials();
    });
});

// Usar antes de routing
app.UseCors(app.Environment.IsDevelopment() ? "Development" : "AllowFrontend");
```

## Input Validation

### FluentValidation
```csharp
public class CreateUserCommandValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserCommandValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(256);

        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(8)
            .Matches(@"[A-Z]").WithMessage("Password must contain uppercase")
            .Matches(@"[a-z]").WithMessage("Password must contain lowercase")
            .Matches(@"[0-9]").WithMessage("Password must contain number")
            .Matches(@"[^a-zA-Z0-9]").WithMessage("Password must contain special char");

        RuleFor(x => x.Name)
            .NotEmpty()
            .MinimumLength(2)
            .MaximumLength(100)
            .Matches(@"^[\p{L}\s\-']+$").WithMessage("Name contains invalid characters");
    }
}
```

### Sanitization
```csharp
// Para HTML output
public static class HtmlSanitizer
{
    public static string Sanitize(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;

        return System.Web.HttpUtility.HtmlEncode(input);
    }
}

// Usar libreria como HtmlSanitizer para rich text
var sanitizer = new Ganss.Xss.HtmlSanitizer();
var safe = sanitizer.Sanitize(userHtml);
```

## Rate Limiting

```csharp
// Program.cs (.NET 7+)
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(
        httpContext => RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: httpContext.User.Identity?.Name
                ?? httpContext.Request.Headers.Host.ToString(),
            factory: partition => new FixedWindowRateLimiterOptions
            {
                AutoReplenishment = true,
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1)
            }));

    options.AddPolicy("Auth", httpContext =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: httpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown",
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 5,
                Window = TimeSpan.FromMinutes(1)
            }));

    options.OnRejected = async (context, token) =>
    {
        context.HttpContext.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        await context.HttpContext.Response.WriteAsync(
            "Too many requests. Please try again later.", token);
    };
});

app.UseRateLimiter();

// En endpoint
[EnableRateLimiting("Auth")]
[HttpPost("login")]
public async Task<IActionResult> Login() { }
```

## Security Headers

```csharp
// Middleware
public class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;

    public SecurityHeadersMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.Headers.Append("X-Content-Type-Options", "nosniff");
        context.Response.Headers.Append("X-Frame-Options", "DENY");
        context.Response.Headers.Append("X-XSS-Protection", "1; mode=block");
        context.Response.Headers.Append("Referrer-Policy", "strict-origin-when-cross-origin");
        context.Response.Headers.Append("Content-Security-Policy",
            "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';");

        await _next(context);
    }
}

// Program.cs
app.UseMiddleware<SecurityHeadersMiddleware>();
```

## Secrets Management

```csharp
// NUNCA en codigo
// var apiKey = "sk-12345"; // MAL

// User Secrets (Development)
// dotnet user-secrets set "Jwt:Key" "your-secret-key"

// Configuration
var jwtKey = builder.Configuration["Jwt:Key"];

// Azure Key Vault (Production)
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{keyVaultName}.vault.azure.net/"),
    new DefaultAzureCredential());

// Environment Variables
// export JWT_KEY="your-secret-key"
var jwtKey = Environment.GetEnvironmentVariable("JWT_KEY");
```

## Security Checklist

### Authentication
- [ ] JWT con expiracion corta (15-60 min)
- [ ] Refresh tokens con rotacion
- [ ] Passwords hasheados con bcrypt/Argon2
- [ ] Brute force protection (rate limiting)
- [ ] Secure password reset flow

### Authorization
- [ ] Principio de minimo privilegio
- [ ] Validar ownership de recursos
- [ ] No exponer IDs internos predecibles

### Data Protection
- [ ] HTTPS obligatorio
- [ ] Sensitive data encriptada en DB
- [ ] No loggear data sensible
- [ ] Sanitizar inputs y outputs

### Headers & CORS
- [ ] Security headers configurados
- [ ] CORS restrictivo (no *)
- [ ] CSRF protection para cookies

### Secrets
- [ ] No secrets en codigo
- [ ] No secrets en logs
- [ ] Key rotation strategy
- [ ] Secrets en vault/env vars
