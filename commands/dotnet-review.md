---
description: Code review de codigo C# y .NET 8
---

# /dotnet-review

Realiza code review de codigo C# enfocado en calidad, seguridad, y performance.

## Que Hace Este Comando

1. Lee el codigo modificado
2. Verifica estilo y organizacion
3. Revisa logica y correctness
4. Evalua seguridad
5. Analiza performance
6. Verifica tests

## Cuando Usar

- Despues de implementar una feature
- Antes de crear un PR
- Para aprender mejores practicas
- Auditoria de seguridad

## Checklist de Review

### Estilo
- [ ] PascalCase para tipos/metodos
- [ ] _camelCase para campos privados
- [ ] Archivos < 800 lineas
- [ ] Metodos < 30 lineas

### C# Moderno
- [ ] Records para DTOs
- [ ] Nullable reference types
- [ ] Pattern matching
- [ ] Async/await correcto

### Seguridad
- [ ] No secrets hardcoded
- [ ] Input validation
- [ ] SQL injection prevention
- [ ] No logging de datos sensibles

### Performance
- [ ] AsNoTracking para queries read-only
- [ ] No N+1 queries
- [ ] Async todo el camino
- [ ] ConfigureAwait en libraries

### Tests
- [ ] Coverage >= 80%
- [ ] Tests para nuevas features
- [ ] Tests de regresion para bugs

## Output de Review

```markdown
## Code Review: CreateUserHandler

### Critico (debe arreglarse)
- [ ] Line 42: Password logged in plain text

### Importante (deberia arreglarse)
- [ ] Line 28: Falta validation de email format
- [ ] Line 35: Query sin AsNoTracking

### Sugerencias
- Line 15: Considerar usar record en lugar de class

### Positivo
- Buen uso de Result pattern
- Tests cubren casos principales
```

## Ejemplo de Uso

```
> /dotnet-review src/Features/Users/CreateUser/

Revisando CreateUserCommand.cs...
Revisando CreateUserHandler.cs...
Revisando CreateUserValidator.cs...
```

## Relacionado

- `/dotnet-tdd` - TDD workflow
- `csharp-reviewer` agent
- `rules/csharp-style.md`
