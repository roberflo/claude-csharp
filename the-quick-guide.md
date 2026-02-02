# Quick Guide: .NET 8 + React/Next.js Claude Config

## Quick Start

```bash
# Copiar a tu proyecto
cp -r .net/* ~/.claude/

# O usar como plugin
/plugin install ./path/to/.net
```

## Comandos Principales

| Comando | Uso |
|---------|-----|
| `/dotnet-plan` | Planificar feature backend |
| `/dotnet-tdd` | Implementar con TDD en .NET |
| `/dotnet-review` | Code review C# |
| `/react-tdd` | Implementar con TDD en React |
| `/react-review` | Code review React/Next.js |
| `/fullstack-plan` | Planificar feature full-stack |

## Agents

| Agent | Cuando Usar |
|-------|------------|
| `dotnet-architect` | Decisiones de arquitectura |
| `csharp-reviewer` | Review de codigo C# |
| `aspnet-tdd-guide` | TDD para .NET |
| `efcore-reviewer` | Review de Entity Framework |
| `react-architect` | Arquitectura React/Next.js |
| `nextjs-reviewer` | Review de Next.js |
| `fullstack-planner` | Features que tocan ambas capas |

## Workflow Tipico

```
1. /fullstack-plan [feature]     # Planificar
2. /dotnet-tdd                   # Backend con TDD
3. /react-tdd                    # Frontend con TDD
4. /dotnet-review                # Review backend
5. /react-review                 # Review frontend
6. git commit                    # Commit
```

## Skills Clave

### Backend
- `dotnet-patterns` - Clean Architecture, CQRS
- `efcore-patterns` - Entity Framework
- `aspnet-testing` - xUnit, Moq
- `aspnet-security` - JWT, Auth

### Frontend
- `react-patterns` - Hooks, Context
- `nextjs-patterns` - App Router, SSR
- `react-testing` - Testing Library

### General
- `tdd-workflow` - RED-GREEN-REFACTOR
- `coding-standards` - Estilo de codigo

## Rules Importantes

1. **Immutability** - Records en C#, spread en TS
2. **Testing** - 80%+ coverage, TDD
3. **Security** - No secrets, validacion
4. **Server Components** - Por defecto en Next.js

## Coverage Minimo

| Capa | Minimo |
|------|--------|
| Domain | 90% |
| Application | 85% |
| Components | 80% |
| Hooks | 90% |

## Tips

1. Siempre planificar antes de implementar
2. TDD: test primero, codigo despues
3. Server Components por defecto
4. Commits con conventional format
5. Review antes de PR
