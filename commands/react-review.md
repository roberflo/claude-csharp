---
description: Code review de codigo React/Next.js y TypeScript
---

# /react-review

Realiza code review de codigo React/Next.js enfocado en patrones, performance, y mejores practicas.

## Que Hace Este Comando

1. Lee el codigo modificado
2. Verifica Server vs Client Components
3. Revisa patrones de componentes
4. Evalua performance
5. Verifica accesibilidad
6. Revisa tests

## Cuando Usar

- Despues de implementar componentes
- Antes de crear un PR
- Optimizar performance
- Verificar accesibilidad

## Checklist de Review

### Server vs Client Components (Next.js)
- [ ] Server Component por defecto
- [ ] 'use client' solo cuando necesario
- [ ] Client boundary lo mas abajo posible
- [ ] No funciones como props Server → Client

### TypeScript
- [ ] No any
- [ ] Props tipadas
- [ ] Return types explicitos
- [ ] Discriminated unions para estados

### React Patterns
- [ ] Immutability (no mutacion)
- [ ] Keys unicas en listas
- [ ] useCallback/useMemo donde apropiado
- [ ] No inline functions en render

### Performance
- [ ] Code splitting con dynamic
- [ ] Images con next/image
- [ ] Memoization donde necesario
- [ ] No re-renders innecesarios

### Accesibilidad
- [ ] Labels en forms
- [ ] Alt en imagenes
- [ ] Roles ARIA correctos
- [ ] Keyboard navigation

### Tests
- [ ] Testing Library queries correctas
- [ ] userEvent para interacciones
- [ ] MSW para mocks de API
- [ ] Coverage >= 80%

## Output de Review

```markdown
## Code Review: UserCard Component

### Critico
- [ ] 'use client' innecesario - es solo render

### Importante
- [ ] Line 15: Falta key en map
- [ ] Line 28: useCallback para handleClick

### Sugerencias
- Considerar memoizar sortedUsers con useMemo

### Positivo
- Buen uso de compound components
- Tests cubren casos principales
```

## Ejemplo de Uso

```
> /react-review src/components/users/

Revisando UserCard.tsx...
Revisando UserList.tsx...
Revisando use-user.ts...
```

## Relacionado

- `/react-tdd` - TDD para React
- `nextjs-reviewer` agent
- `rules/react-style.md`
