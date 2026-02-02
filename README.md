# Everything Claude Code - .NET 8 + React/Next.js Edition

Configuraciones de produccion para Claude Code optimizadas para desarrollo con **.NET 8** (backend) y **React/Next.js** (frontend).

## Caracteristicas

- **15 Agents** especializados para .NET y React
- **25+ Skills** con patrones para ASP.NET Core, EF Core, React, Next.js
- **15+ Commands** para workflows TDD, code review, arquitectura
- **Hooks** para automatizacion de builds, tests, formatting
- **Rules** con guias obligatorias para C# y TypeScript/React
- **MCP Configs** para Azure, SQL Server, y servicios cloud

## Quick Start

### Opcion 1: Plugin Installation

```bash
# Copiar la carpeta .net a tu proyecto
cp -r .net ~/.claude/

# O agregar como plugin local
/plugin install ./path/to/.net
```

### Opcion 2: Manual

```bash
# Copiar agents
cp .net/agents/*.md ~/.claude/agents/

# Copiar rules (REQUERIDO)
cp -r .net/rules/* ~/.claude/rules/

# Copiar skills
cp -r .net/skills/* ~/.claude/skills/

# Copiar commands
cp .net/commands/*.md ~/.claude/commands/
```

## Stack Soportado

### Backend (.NET 8)
- ASP.NET Core 8 (Minimal APIs / MVC)
- Entity Framework Core 8
- xUnit + Moq para testing
- FluentValidation
- MediatR (CQRS pattern)
- AutoMapper
- Serilog

### Frontend (React/Next.js)
- Next.js 14+ (App Router)
- React 18+
- TypeScript 5+
- Tailwind CSS
- React Query / TanStack Query
- Testing Library + Vitest

## Estructura

```
.net/
├── agents/                    # Subagents especializados
│   ├── dotnet-architect.md    # Arquitectura .NET
│   ├── csharp-reviewer.md     # Code review C#
│   ├── aspnet-tdd-guide.md    # TDD para ASP.NET
│   ├── efcore-reviewer.md     # Review EF Core
│   ├── react-architect.md     # Arquitectura React
│   ├── nextjs-reviewer.md     # Review Next.js
│   └── fullstack-planner.md   # Planificacion full-stack
│
├── skills/                    # Patrones y workflows
│   ├── dotnet-patterns/       # Clean Architecture, CQRS
│   ├── efcore-patterns/       # EF Core best practices
│   ├── aspnet-testing/        # xUnit, integration tests
│   ├── aspnet-security/       # JWT, Identity, CORS
│   ├── react-patterns/        # Hooks, Context, Components
│   ├── nextjs-patterns/       # SSR, App Router, API Routes
│   ├── react-testing/         # Testing Library, Vitest
│   └── tdd-workflow/          # RED-GREEN-REFACTOR
│
├── commands/                  # Slash commands
│   ├── dotnet-plan.md         # /dotnet-plan
│   ├── dotnet-tdd.md          # /dotnet-tdd
│   ├── dotnet-review.md       # /dotnet-review
│   ├── react-review.md        # /react-review
│   └── fullstack-setup.md     # /fullstack-setup
│
├── hooks/                     # Automaciones
│   └── hooks.json             # Pre/Post tool hooks
│
├── rules/                     # Guias obligatorias
│   ├── csharp-style.md        # Estilo C#
│   ├── aspnet-security.md     # Seguridad ASP.NET
│   ├── react-style.md         # Estilo React/TS
│   ├── testing.md             # Reglas de testing
│   └── git-workflow.md        # Git conventions
│
├── mcp-configs/               # Configuraciones MCP
│   └── mcp-servers.json       # Azure, SQL Server, etc.
│
└── examples/                  # Ejemplos
    └── CLAUDE.md              # Ejemplo de proyecto
```

## Workflows Principales

### 1. Nueva Feature Full-Stack

```bash
/fullstack-plan        # Planificar feature (backend + frontend)
/dotnet-tdd           # Implementar backend con TDD
/react-tdd            # Implementar frontend con TDD
/dotnet-review        # Review del backend
/react-review         # Review del frontend
git commit            # Commit con conventional commits
```

### 2. Fix de Bug

```bash
/dotnet-tdd fix       # Escribir test que reproduce el bug
                      # Implementar fix
/dotnet-review        # Verificar el fix
```

### 3. Refactor

```bash
/dotnet-review        # Identificar code smells
/dotnet-tdd refactor  # Refactorizar con tests
```

## Agents Disponibles

| Agent | Descripcion | Uso |
|-------|-------------|-----|
| `dotnet-architect` | Diseno de arquitectura .NET | Decisiones de arquitectura, Clean Architecture |
| `csharp-reviewer` | Code review de C# | Calidad, seguridad, performance |
| `aspnet-tdd-guide` | TDD para ASP.NET Core | Escribir tests primero, 80%+ coverage |
| `efcore-reviewer` | Review de Entity Framework | Queries, migrations, performance |
| `react-architect` | Arquitectura React/Next.js | Estructura de componentes, estado |
| `nextjs-reviewer` | Review Next.js especifico | SSR, caching, optimizaciones |
| `fullstack-planner` | Planificacion completa | Features que tocan backend y frontend |

## Skills Clave

### Backend
- **dotnet-patterns**: Clean Architecture, CQRS, Vertical Slice
- **efcore-patterns**: Repository pattern, Unit of Work, Migrations
- **aspnet-testing**: xUnit, Moq, WebApplicationFactory
- **aspnet-security**: JWT, Identity, CORS, Rate Limiting

### Frontend
- **react-patterns**: Custom Hooks, Context, Compound Components
- **nextjs-patterns**: App Router, Server Components, Caching
- **react-testing**: Testing Library, MSW, Vitest

## Rules Obligatorias

1. **Immutability**: Records en C#, spread operator en TS
2. **Testing**: 80%+ coverage, TDD workflow
3. **Security**: No secrets en codigo, validacion de inputs
4. **File Organization**: 200-400 lineas tipico, max 800
5. **Naming**: PascalCase (C#), camelCase (TS)

## Requisitos

- Claude Code v2.1.0+
- .NET 8 SDK
- Node.js 18+
- SQL Server / PostgreSQL

## Contribuir

PRs son bienvenidos. Sigue el workflow de conventional commits.

## Licencia

MIT
