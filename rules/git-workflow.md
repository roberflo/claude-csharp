# Git Workflow Rules

Reglas OBLIGATORIAS para el workflow de Git.

## Conventional Commits

### Formato

```
<type>: <description>

[optional body]

[optional footer]
```

### Types

| Type | Uso |
|------|-----|
| `feat` | Nueva funcionalidad |
| `fix` | Correccion de bug |
| `refactor` | Cambio de codigo sin cambio de comportamiento |
| `docs` | Documentacion |
| `test` | Tests |
| `chore` | Mantenimiento, dependencias |
| `perf` | Mejora de performance |
| `ci` | CI/CD |
| `style` | Formatting, no afecta logica |

### Ejemplos

```bash
# Features
feat: add user registration endpoint
feat: implement password reset flow
feat(auth): add two-factor authentication

# Fixes
fix: resolve null reference in UserService
fix: correct validation for email field
fix(api): handle timeout in external service call

# Refactoring
refactor: extract email validation to separate class
refactor: simplify user creation logic
refactor(db): optimize query for user search

# Otros
docs: update API documentation
test: add unit tests for CreateUserHandler
chore: update dependencies to latest versions
perf: cache user preferences in memory
```

## Branching Strategy

### Ramas Principales

```
main          # Produccion - siempre deployable
develop       # Desarrollo - integration branch (opcional)
```

### Feature Branches

```
feature/user-registration
feature/issue-123-fix-login
bugfix/null-reference-user-service
hotfix/critical-security-patch
```

### Naming

```bash
# Pattern: type/description-or-issue
feature/add-user-registration
feature/issue-42-implement-search
bugfix/fix-login-validation
hotfix/security-patch-jwt
```

## Workflow: Feature Implementation

```bash
# 1. Crear branch desde main/develop
git checkout main
git pull origin main
git checkout -b feature/user-registration

# 2. Desarrollar con commits atomicos
git add src/Features/Users/
git commit -m "feat: add User entity and repository"

git add src/Features/Users/CreateUser/
git commit -m "feat: implement CreateUser command"

git add tests/Application.UnitTests/
git commit -m "test: add unit tests for CreateUserHandler"

# 3. Push y crear PR
git push -u origin feature/user-registration
```

## Pull Request Rules

### Antes de Crear PR

- [ ] Todos los tests pasan
- [ ] Coverage >= 80%
- [ ] No linting errors
- [ ] Code review self-check
- [ ] Commit history limpio

### PR Template

```markdown
## Summary
Brief description of changes

## Changes
- Added X
- Modified Y
- Fixed Z

## Testing
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] Manual testing done

## Screenshots (if UI changes)

## Related Issues
Fixes #123
```

### PR Size

- **Ideal**: 200-400 lineas cambiadas
- **Maximo**: 800 lineas
- PRs grandes deben dividirse

## Commits Atomicos

```bash
# BIEN: Un commit por cambio logico
git commit -m "feat: add User entity"
git commit -m "feat: add UserRepository"
git commit -m "test: add User entity tests"

# MAL: Todo en un commit gigante
git commit -m "feat: add user feature with tests and everything"
```

## Rebase vs Merge

```bash
# SIEMPRE: Rebase antes de merge para historia limpia
git fetch origin
git rebase origin/main

# NUNCA: Rebase de ramas publicas (main, develop)
git rebase main  # Solo en tu feature branch
```

## Prohibiciones

### NUNCA

```bash
# Force push a main/develop
git push --force origin main  # PROHIBIDO!

# Commit de secrets
git add .env                  # PROHIBIDO!
git add appsettings.json     # (si tiene secrets)

# Commit de archivos generados
git add node_modules/         # PROHIBIDO!
git add bin/                  # PROHIBIDO!
git add obj/                  # PROHIBIDO!

# Commits sin mensaje descriptivo
git commit -m "fix"           # NO
git commit -m "update"        # NO
git commit -m "wip"           # NO (en branch final)
```

## .gitignore Obligatorio

```gitignore
# .NET
bin/
obj/
*.user
*.suo
.vs/

# Node
node_modules/
.next/
dist/

# Environment
.env
.env.local
.env.*.local
appsettings.*.json

# IDE
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/
```

## Hooks Recomendados

```bash
# pre-commit: Format y lint
dotnet format
npm run lint

# pre-push: Tests
dotnet test
npm test
```

## Squash Merging

```bash
# Para PRs con muchos commits WIP
git merge --squash feature/my-feature
git commit -m "feat: complete feature description"
```

## Checklist Pre-Push

- [ ] Todos los tests pasan
- [ ] No hay secrets en commits
- [ ] Commits tienen mensajes descriptivos
- [ ] No archivos innecesarios (.env, node_modules)
- [ ] Branch actualizada con main
