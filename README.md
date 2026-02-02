# Everything Claude Code - .NET 8 + React/Next.js Edition

Configuraciones de produccion para Claude Code optimizadas para desarrollo con **.NET 8** (backend) y **React/Next.js** (frontend).

## Caracteristicas

- **8 Agents** especializados para .NET y React
- **10 Skills** con patrones para ASP.NET Core, EF Core, React, Next.js
- **6 Commands** para workflows TDD, code review, arquitectura
- **Hooks** para automatizacion de builds, tests, formatting
- **5 Rules** con guias obligatorias para C# y TypeScript/React
- **MCP Configs** para Azure, SQL Server, y servicios cloud

---

## Quick Start

### Opcion 1: Clonar e Instalar como Plugin

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/claude-csharp.git

# Instalar como plugin de Claude Code
cd claude-csharp
claude /plugin install .
```

### Opcion 2: Copiar a ~/.claude/

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/claude-csharp.git
cd claude-csharp

# Copiar todo a tu configuracion de Claude
cp -r agents/* ~/.claude/agents/
cp -r commands/* ~/.claude/commands/
cp -r skills/* ~/.claude/skills/
cp -r rules/* ~/.claude/rules/
cp hooks/hooks.json ~/.claude/hooks.json
```

### Opcion 3: Por Componente (Selectivo)

```bash
# Solo agents
cp agents/*.md ~/.claude/agents/

# Solo commands
cp commands/*.md ~/.claude/commands/

# Solo rules (RECOMENDADO como minimo)
cp rules/*.md ~/.claude/rules/
```

---

## Herramientas de Claude Code (Tools)

Claude Code tiene acceso a herramientas poderosas que puedes usar en tus conversaciones:

### Herramientas de Archivos

| Tool | Uso | Ejemplo |
|------|-----|---------|
| **Read** | Leer archivos | "Lee el archivo Program.cs" |
| **Write** | Crear archivos nuevos | "Crea un nuevo archivo UserService.cs" |
| **Edit** | Editar archivos existentes | "Cambia el nombre de la clase" |
| **Glob** | Buscar archivos por patron | "Encuentra todos los *.cs" |
| **Grep** | Buscar contenido en archivos | "Busca usos de IUserService" |

### Herramientas de Ejecucion

| Tool | Uso | Ejemplo |
|------|-----|---------|
| **Bash** | Ejecutar comandos terminal | "Ejecuta dotnet build" |
| **Task** | Lanzar agentes especializados | "Usa el agente de arquitectura" |

### Herramientas de Web

| Tool | Uso | Ejemplo |
|------|-----|---------|
| **WebFetch** | Obtener contenido de URL | "Lee la documentacion de esta URL" |
| **WebSearch** | Buscar en internet | "Busca como usar FluentValidation" |

### Herramientas de Planificacion

| Tool | Uso | Ejemplo |
|------|-----|---------|
| **EnterPlanMode** | Planificar antes de implementar | "Planifica esta feature" |
| **TaskCreate/TaskUpdate** | Gestionar lista de tareas | "Crea tareas para este proyecto" |
| **AskUserQuestion** | Preguntar al usuario | Cuando Claude necesita clarificacion |

### Ejemplos de Uso de Tools

```bash
# Buscar todos los controladores
"Encuentra todos los archivos que terminen en Controller.cs"

# Ejecutar tests
"Ejecuta dotnet test y muestrame los resultados"

# Buscar implementaciones
"Busca donde se usa la interfaz IRepository"

# Crear estructura
"Crea la estructura de carpetas para Clean Architecture"

# Planificar feature
"Entra en modo plan para disenar el sistema de autenticacion"
```

---

## Comandos Slash Disponibles

Los comandos se invocan con `/nombre-comando`:

| Comando | Descripcion | Cuando Usar |
|---------|-------------|-------------|
| `/dotnet-plan` | Planificar feature backend | Antes de implementar nueva funcionalidad .NET |
| `/dotnet-tdd` | Implementar con TDD | Desarrollo guiado por tests en C# |
| `/dotnet-review` | Code review C# | Revisar calidad, seguridad, performance |
| `/react-tdd` | Implementar con TDD React | Desarrollo guiado por tests en React |
| `/react-review` | Code review React/Next.js | Revisar componentes, hooks, SSR |
| `/fullstack-plan` | Planificar feature completa | Features que tocan backend Y frontend |

### Uso de Comandos

```bash
# Planificar nueva feature
/fullstack-plan "Sistema de notificaciones push"

# Desarrollar con TDD
/dotnet-tdd "Crear endpoint POST /api/users"

# Revisar codigo
/dotnet-review src/Services/UserService.cs
```

---

## Agents Especializados

Los agents son subprocesos especializados que Claude puede invocar:

| Agent | Especialidad | Herramientas |
|-------|--------------|--------------|
| `dotnet-architect` | Arquitectura .NET, Clean Architecture, CQRS | Read, Grep, Glob |
| `csharp-reviewer` | Code review C#, seguridad, performance | Read, Grep |
| `aspnet-tdd-guide` | TDD para ASP.NET Core, xUnit, Moq | Read, Write, Bash |
| `efcore-reviewer` | Entity Framework, queries, migrations | Read, Grep |
| `react-architect` | Arquitectura React, estado, componentes | Read, Grep, Glob |
| `react-tdd-guide` | TDD para React, Testing Library, Vitest | Read, Write, Bash |
| `nextjs-reviewer` | Next.js, SSR, App Router, caching | Read, Grep |
| `fullstack-planner` | Planificacion completa backend+frontend | Read, Glob, Grep |

### Invocar Agents Manualmente

```bash
# Pedir arquitectura
"Usa el agente dotnet-architect para disenar la estructura"

# Pedir review
"Invoca csharp-reviewer para revisar este codigo"

# Planificar feature
"Usa fullstack-planner para planificar el modulo de pagos"
```

---

## Skills (Patrones y Conocimiento)

Los skills proveen conocimiento especializado:

### Backend (.NET)

| Skill | Contenido |
|-------|-----------|
| `dotnet-patterns` | Clean Architecture, CQRS, Vertical Slice, DDD |
| `efcore-patterns` | Repository, Unit of Work, Migrations, Performance |
| `aspnet-testing` | xUnit, Moq, WebApplicationFactory, Integration Tests |
| `aspnet-security` | JWT, Identity, CORS, Rate Limiting, OWASP |

### Frontend (React/Next.js)

| Skill | Contenido |
|-------|-----------|
| `react-patterns` | Custom Hooks, Context, Compound Components, HOCs |
| `nextjs-patterns` | App Router, Server Components, Streaming, Caching |
| `react-testing` | Testing Library, MSW, Vitest, Component Testing |

### General

| Skill | Contenido |
|-------|-----------|
| `tdd-workflow` | RED-GREEN-REFACTOR, Test First, Coverage |
| `coding-standards` | Estilo de codigo, naming, organizacion |
| `continuous-learning` | Mejora continua, documentacion |

---

## Rules (Reglas Obligatorias)

Las rules se aplican automaticamente a todo el codigo:

| Rule | Aplica A | Principales Puntos |
|------|----------|-------------------|
| `csharp-style` | *.cs | PascalCase, records, async/await, nullable |
| `react-style` | *.tsx, *.ts | camelCase, hooks, functional components |
| `security` | Todo | No secrets, validacion, sanitizacion |
| `testing` | Tests | 80%+ coverage, TDD, assertions claras |
| `git-workflow` | Commits | Conventional commits, branches, PRs |

---

## Hooks (Automatizaciones)

Los hooks ejecutan acciones automaticas:

### Pre-Tool Hooks
- **Long-running processes**: Aviso al ejecutar `dotnet run` o `npm run dev`
- **Force push warning**: Advertencia al usar `git push --force`

### Post-Tool Hooks
- **Build failure**: Recordatorio al fallar `dotnet build` o `npm run build`
- **Test failure**: Recordatorio al fallar tests
- **C# file written**: Recordatorio de compilar
- **TS file written**: Recordatorio de type check
- **Debug logs**: Advertencia al detectar console.log/Console.Write

### Session Hooks
- **SessionStart**: Muestra comandos disponibles
- **Stop**: Recordatorio de ejecutar tests antes de commit

---

## Workflows Principales

### 1. Nueva Feature Full-Stack

```
1. /fullstack-plan [descripcion]    # Planificar
2. Claude entra en modo plan        # Disenar arquitectura
3. /dotnet-tdd                      # Backend con TDD
4. /react-tdd                       # Frontend con TDD
5. /dotnet-review                   # Review backend
6. /react-review                    # Review frontend
7. git commit -m "feat: ..."        # Commit
```

### 2. Bug Fix con TDD

```
1. "Investiga el bug en..."         # Claude analiza
2. /dotnet-tdd fix                  # Escribe test que falla
3. Implementa el fix                # Test pasa
4. /dotnet-review                   # Verifica el fix
```

### 3. Refactoring Seguro

```
1. /dotnet-review                   # Identifica code smells
2. "Escribe tests para..."          # Coverage primero
3. "Refactoriza..."                 # Con tests como red de seguridad
4. "Ejecuta dotnet test"            # Verifica que todo pasa
```

### 4. Code Review Completo

```
1. /dotnet-review src/              # Review backend
2. /react-review app/               # Review frontend
3. "Resume los hallazgos"           # Obtener resumen
```

---

## Estructura del Proyecto

```
claude-csharp/
├── agents/                    # 8 Subagents especializados
│   ├── dotnet-architect.md
│   ├── csharp-reviewer.md
│   ├── aspnet-tdd-guide.md
│   ├── efcore-reviewer.md
│   ├── react-architect.md
│   ├── react-tdd-guide.md
│   ├── nextjs-reviewer.md
│   └── fullstack-planner.md
│
├── commands/                  # 6 Slash commands
│   ├── dotnet-plan.md
│   ├── dotnet-tdd.md
│   ├── dotnet-review.md
│   ├── react-tdd.md
│   ├── react-review.md
│   └── fullstack-plan.md
│
├── skills/                    # 10 Skills con patrones
│   ├── dotnet-patterns/
│   ├── efcore-patterns/
│   ├── aspnet-testing/
│   ├── aspnet-security/
│   ├── react-patterns/
│   ├── nextjs-patterns/
│   ├── react-testing/
│   ├── tdd-workflow/
│   ├── coding-standards/
│   └── continuous-learning/
│
├── rules/                     # 5 Reglas obligatorias
│   ├── csharp-style.md
│   ├── react-style.md
│   ├── security.md
│   ├── testing.md
│   └── git-workflow.md
│
├── hooks/
│   └── hooks.json            # Automatizaciones
│
├── mcp-configs/
│   └── mcp-servers.json      # Azure, SQL Server
│
├── examples/
│   └── CLAUDE.md             # Ejemplo de proyecto
│
└── scripts/                   # Scripts de utilidad
```

---

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

---

## Requisitos

- Claude Code v2.1.0+
- .NET 8 SDK
- Node.js 18+
- SQL Server / PostgreSQL (opcional)

---

## Tips de Uso

1. **Siempre planifica primero** - Usa `/fullstack-plan` o modo plan antes de implementar features complejas

2. **TDD es el default** - Escribe tests primero con `/dotnet-tdd` o `/react-tdd`

3. **Reviews frecuentes** - Usa `/dotnet-review` y `/react-review` antes de commits

4. **Usa las tools correctas**:
   - `Read` para ver archivos (no `cat`)
   - `Grep` para buscar (no `grep`)
   - `Edit` para cambios (no `sed`)

5. **Aprovecha los agents** - Deja que los agents especializados hagan el trabajo pesado

6. **Confía en los hooks** - Te recordarán compilar, testear, y evitar errores comunes

---

## Contribuir

PRs son bienvenidos. Sigue el workflow de conventional commits.

## Licencia

MIT
