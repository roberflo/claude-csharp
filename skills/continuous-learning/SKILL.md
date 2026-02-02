---
name: continuous-learning
description: Sistema de aprendizaje continuo para extraer y aplicar patrones del proyecto
---

# Continuous Learning

## Proposito

Extraer patrones de exito y fallos durante las sesiones para mejorar continuamente.

## Como Funciona

### 1. Durante la Sesion

Cuando implementes algo exitosamente, documenta:

```markdown
## Pattern Learned

### Context
- Feature: User Registration
- Stack: .NET 8 + React

### Pattern
Para validacion de forms en .NET con FluentValidation + React Hook Form:

1. Backend: Crear validator con FluentValidation
2. Frontend: Usar react-hook-form con zod schema
3. Mapear errores de API a form errors

### Code Example
```csharp
// Backend
public class CreateUserValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserValidator()
    {
        RuleFor(x => x.Email).NotEmpty().EmailAddress();
    }
}
```

```tsx
// Frontend
const schema = z.object({
  email: z.string().email(),
});

const { handleSubmit, setError } = useForm<FormData>({
  resolver: zodResolver(schema),
});
```

### When to Apply
- Forms con validacion backend + frontend
- Cuando necesitas mostrar errores del server en el form
```

### 2. Al Final de la Sesion

Revisa que patrones funcionaron y documenta en `patterns.md`:

```markdown
# Learned Patterns

## 1. Form Validation Pattern
...

## 2. Error Handling Pattern
...

## 3. Testing Pattern
...
```

### 3. Para Nuevas Sesiones

Al inicio de sesion, carga los patrones aprendidos para aplicarlos.

## Patrones Comunes

### .NET + React Integration

```markdown
### API Error Handling

1. Backend retorna Problem Details
2. Frontend tiene error interceptor
3. Mapea a toast/form errors segun tipo

### Authentication Flow

1. JWT access token (30 min)
2. Refresh token (7 dias)
3. HttpOnly cookie para refresh
4. Bearer header para access
```

### Testing Patterns

```markdown
### Integration Test Setup

1. WebApplicationFactory con InMemory DB
2. Crear client en constructor
3. Seed data en cada test
4. Cleanup automatico

### Component Test with API

1. MSW para mock de API
2. Testing Library para render
3. userEvent para interacciones
4. waitFor para async
```

## Estructura de Patrones

```markdown
## Pattern: [Nombre]

### Context
Cuando aplica este patron

### Problem
El problema que resuelve

### Solution
Como implementarlo

### Code
Ejemplo de codigo

### Trade-offs
Ventajas y desventajas

### Related
Patrones relacionados
```

## Almacenamiento

Los patrones se pueden almacenar en:

1. `~/.claude/patterns/` - Patrones globales
2. `.claude/patterns/` - Patrones del proyecto
3. En CLAUDE.md del proyecto

## Evolucion

Con el tiempo, patrones que se repiten pueden:

1. Convertirse en skills reutilizables
2. Agregarse a rules del proyecto
3. Documentarse en guias del equipo
