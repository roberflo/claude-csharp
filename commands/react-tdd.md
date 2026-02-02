---
description: Implementar componentes React usando Test-Driven Development
---

# /react-tdd

Implementa componentes React/Next.js siguiendo el ciclo TDD.

## Que Hace Este Comando

1. Define casos de test
2. Escribe test que falla (RED)
3. Implementa componente (GREEN)
4. Refactoriza
5. Repite

## Stack de Testing

- Vitest (test runner)
- Testing Library (render/query)
- user-event (interacciones)
- MSW (mocks de API)

## Ciclo TDD

### 1. Listar Casos

```markdown
Component: LoginForm
- [ ] Renders email and password inputs
- [ ] Shows validation errors for empty fields
- [ ] Calls onSubmit with credentials
- [ ] Shows loading state during submit
- [ ] Shows error message on failure
```

### 2. Primer Test (RED)

```tsx
it('renders email and password inputs', () => {
  render(<LoginForm onSubmit={vi.fn()} />);

  expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
  expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
});
// Falla: LoginForm no existe
```

### 3. Implementar (GREEN)

```tsx
function LoginForm({ onSubmit }: LoginFormProps) {
  return (
    <form>
      <label>
        Email
        <input type="email" name="email" />
      </label>
      <label>
        Password
        <input type="password" name="password" />
      </label>
    </form>
  );
}
// Test pasa
```

### 4. Siguiente Test

```tsx
it('calls onSubmit with credentials', async () => {
  const onSubmit = vi.fn();
  const user = userEvent.setup();

  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.type(screen.getByLabelText(/password/i), 'password123');
  await user.click(screen.getByRole('button', { name: /login/i }));

  expect(onSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  });
});
```

## Testing Library Best Practices

### Queries por Prioridad

```tsx
// 1. Role (accesibilidad)
screen.getByRole('button', { name: /submit/i });

// 2. Label (forms)
screen.getByLabelText(/email/i);

// 3. Text
screen.getByText(/welcome/i);

// 4. TestId (ultimo recurso)
screen.getByTestId('custom-element');
```

### User Events

```tsx
const user = userEvent.setup();

await user.click(button);
await user.type(input, 'text');
await user.selectOptions(select, 'option1');
```

### Async

```tsx
// findBy para elementos async
const element = await screen.findByText('Loaded');

// waitFor para condiciones
await waitFor(() => {
  expect(screen.getByText('Done')).toBeInTheDocument();
});
```

## Coverage Target

- Components: 80%+
- Hooks: 90%+
- Utils: 95%+

## Ejemplo de Uso

```
> /react-tdd Implementar UserCard con avatar y nombre

Listando casos de test:
- [ ] Renders user name
- [ ] Renders avatar image
- [ ] Calls onClick when clicked
...

Comenzando ciclo TDD...
```

## Relacionado

- `/react-review` - Review despues de implementar
- `react-tdd-guide` agent
