# Quick Guide: .NET 8 + React/Next.js Claude Config

## Instalacion Rapida

```bash
# Opcion 1: Como plugin
git clone https://github.com/tu-usuario/claude-csharp.git
cd claude-csharp && claude /plugin install .

# Opcion 2: Copiar manual
cp -r agents commands skills rules hooks ~/.claude/
```

---

## Tools de Claude Code

### Archivos
| Tool | Que Hace | Ejemplo |
|------|----------|---------|
| `Read` | Leer archivos | "Lee Program.cs" |
| `Write` | Crear archivos | "Crea UserService.cs" |
| `Edit` | Modificar archivos | "Cambia el metodo X" |
| `Glob` | Buscar por patron | "Encuentra *.cs" |
| `Grep` | Buscar contenido | "Busca IUserService" |

### Ejecucion
| Tool | Que Hace | Ejemplo |
|------|----------|---------|
| `Bash` | Comandos terminal | "Ejecuta dotnet test" |
| `Task` | Agentes especializados | "Usa dotnet-architect" |

### Planificacion
| Tool | Que Hace | Ejemplo |
|------|----------|---------|
| `EnterPlanMode` | Modo planificacion | "Planifica esta feature" |
| `TaskCreate` | Crear tareas | "Crea lista de tareas" |
| `WebSearch` | Buscar en web | "Busca documentacion de X" |

---

## Comandos Slash

| Comando | Uso |
|---------|-----|
| `/dotnet-plan` | Planificar feature backend |
| `/dotnet-tdd` | Implementar con TDD en .NET |
| `/dotnet-review` | Code review C# |
| `/react-tdd` | Implementar con TDD en React |
| `/react-review` | Code review React/Next.js |
| `/fullstack-plan` | Planificar feature full-stack |

---

## Agents

| Agent | Cuando Usar |
|-------|------------|
| `dotnet-architect` | Decisiones de arquitectura .NET |
| `csharp-reviewer` | Review de codigo C# |
| `aspnet-tdd-guide` | TDD para ASP.NET Core |
| `efcore-reviewer` | Review de Entity Framework |
| `react-architect` | Arquitectura React/Next.js |
| `react-tdd-guide` | TDD para React |
| `nextjs-reviewer` | Review de Next.js |
| `fullstack-planner` | Features que tocan ambas capas |

---

## Skills

### Backend
- `dotnet-patterns` - Clean Architecture, CQRS, DDD
- `efcore-patterns` - Entity Framework, Repository
- `aspnet-testing` - xUnit, Moq, Integration Tests
- `aspnet-security` - JWT, Identity, CORS

### Frontend
- `react-patterns` - Hooks, Context, Components
- `nextjs-patterns` - App Router, SSR, Caching
- `react-testing` - Testing Library, Vitest

### General
- `tdd-workflow` - RED-GREEN-REFACTOR
- `coding-standards` - Estilo de codigo
- `continuous-learning` - Mejora continua

---

## Workflow Tipico

```
1. /fullstack-plan [feature]     # Planificar
2. /dotnet-tdd                   # Backend con TDD
3. /react-tdd                    # Frontend con TDD
4. /dotnet-review                # Review backend
5. /react-review                 # Review frontend
6. git commit                    # Commit
```

---

## Rules Activas

| Rule | Puntos Clave |
|------|--------------|
| `csharp-style` | PascalCase, records, async/await |
| `react-style` | camelCase, hooks, functional |
| `security` | No secrets, validacion inputs |
| `testing` | 80%+ coverage, TDD |
| `git-workflow` | Conventional commits |

---

## Coverage Minimo

| Capa | Minimo |
|------|--------|
| Domain | 90% |
| Application | 85% |
| Components | 80% |
| Hooks | 90% |

---

## Hooks Automaticos

- **Build fail** -> Recordatorio de arreglar errores
- **Test fail** -> Recordatorio de arreglar tests
- **C# escrito** -> Recordatorio de `dotnet build`
- **TS escrito** -> Recordatorio de typecheck
- **console.log** -> Advertencia de debug code
- **Force push** -> Advertencia de seguridad

---

## Ejemplos de Uso

### Crear nueva feature
```
"Necesito crear un endpoint POST /api/orders"
/fullstack-plan "Sistema de ordenes con carrito"
/dotnet-tdd
```

### Investigar bug
```
"Hay un error cuando el usuario hace login"
Claude usa: Read, Grep, Bash (para tests)
/dotnet-tdd fix
```

### Refactoring
```
/dotnet-review src/Services/
"Refactoriza UserService usando Clean Architecture"
"Ejecuta dotnet test"
```

### Buscar informacion
```
"Busca en el proyecto donde se manejan errores"
Claude usa: Grep, Glob, Task (agente Explore)

"Busca documentacion de FluentValidation"
Claude usa: WebSearch
```

---

## Tips Rapidos

1. **Planifica primero** - `/fullstack-plan` antes de codigo
2. **TDD siempre** - Test primero, codigo despues
3. **Reviews antes de commit** - `/dotnet-review`
4. **Usa tools correctas** - Read no cat, Grep no grep
5. **Confía en hooks** - Te avisan de errores comunes
6. **Agents para tareas complejas** - Deja que hagan el trabajo

---

## Referencia Rapida de Tools

```bash
# Leer archivo
"Lee src/Program.cs"

# Buscar archivos
"Encuentra todos los Controllers"

# Buscar en codigo
"Busca donde se usa IRepository"

# Ejecutar comando
"Ejecuta dotnet test --filter Category=Unit"

# Planificar
"Entra en modo plan para disenar el modulo de pagos"

# Crear tarea
"Crea una lista de tareas para implementar autenticacion"

# Buscar en web
"Busca como configurar JWT en ASP.NET Core 8"
```
