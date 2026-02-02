# React/TypeScript Code Style Rules

Estas reglas son OBLIGATORIAS para todo codigo React/TypeScript en el proyecto.

## Principios Fundamentales

### 1. Immutability (CRITICO)

```tsx
// SIEMPRE: Spread para updates
const updatedUser = { ...user, name: newName };

// SIEMPRE: Map/filter para arrays
const filtered = items.filter(item => item.active);
const updated = items.map(item =>
  item.id === id ? { ...item, name: newName } : item
);

// NUNCA: Mutacion directa
user.name = newName;           // NO!
items.push(newItem);           // NO!
items[0].name = newName;       // NO!

// En estado de React
setUser(prev => ({ ...prev, name: newName }));  // BIEN
setItems(prev => [...prev, newItem]);           // BIEN
```

### 2. TypeScript Estricto

```tsx
// SIEMPRE: Tipos explicitos para props
interface UserCardProps {
  user: User;
  onSelect?: (id: string) => void;
}

// SIEMPRE: No usar any
function process(data: unknown) { }  // BIEN
function process(data: any) { }      // NUNCA

// SIEMPRE: Return types explicitos para funciones publicas
function formatDate(date: Date): string {
  return date.toISOString();
}

// SIEMPRE: Discriminated unions para estados
type LoadingState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

### 3. Server vs Client Components (Next.js)

```tsx
// SIEMPRE: Server Component por defecto (no 'use client')
// Solo agregar 'use client' cuando necesites:
// - useState, useEffect, useContext
// - Event handlers (onClick, onChange)
// - Browser APIs

// BIEN: Server Component
async function UserList() {
  const users = await fetchUsers();
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}

// BIEN: Client solo para interactividad
'use client';
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// BIEN: Minimizar client boundary
function Page() {
  return (
    <div>
      <ServerComponent />      {/* Server */}
      <InteractiveButton />    {/* Client - solo este */}
    </div>
  );
}
```

## Organizacion de Componentes

### Estructura de Archivo

```tsx
// 1. Imports (externos, internos, tipos, estilos)
import { useState, useCallback, memo } from 'react';
import { User } from '@/types';
import { formatDate } from '@/lib/utils';
import styles from './user-card.module.css';

// 2. Types
interface UserCardProps {
  user: User;
  isSelected?: boolean;
  onSelect?: (id: string) => void;
}

// 3. Component
export function UserCard({ user, isSelected = false, onSelect }: UserCardProps) {
  // a. Hooks (useState, useRef, useContext, custom hooks)
  const [isExpanded, setIsExpanded] = useState(false);

  // b. Derived state (useMemo si es costoso)
  const fullName = `${user.firstName} ${user.lastName}`;

  // c. Callbacks (useCallback si se pasa a children)
  const handleClick = useCallback(() => {
    onSelect?.(user.id);
  }, [user.id, onSelect]);

  // d. Effects
  useEffect(() => {
    // Efecto aqui
  }, [dependency]);

  // e. Early returns
  if (!user) return null;

  // f. Render
  return (
    <div className={styles.card}>
      {/* JSX */}
    </div>
  );
}

// 4. Memoized export (si tiene props complejas)
export const MemoizedUserCard = memo(UserCard);
```

### Naming

```tsx
// Archivos
UserCard.tsx           // Componente
user-card.module.css   // Estilos
use-user.ts            // Hook
user-service.ts        // Service

// Componentes - PascalCase
function UserCard() { }
function AuthProvider() { }

// Hooks - camelCase con "use"
function useUser() { }
function useLocalStorage() { }

// Handlers - handle + Action
const handleClick = () => { };
const handleSubmit = () => { };
const handleUserSelect = () => { };

// Boolean props - is/has/can/should
isLoading, isDisabled, hasError, canEdit, shouldRefresh

// Arrays - plural
users, items, selectedIds
```

## Patrones Obligatorios

### Props Drilling vs Context

```tsx
// EVITAR: Drilling profundo (>2 niveles)
<GrandParent>
  <Parent user={user}>      {/* Level 1 */}
    <Child user={user}>     {/* Level 2 */}
      <GrandChild user={user} />  {/* Level 3 - usar Context */}
    </Child>
  </Parent>
</GrandParent>

// BIEN: Context para estado compartido
const UserContext = createContext<User | null>(null);

function useUser() {
  const user = useContext(UserContext);
  if (!user) throw new Error('useUser must be within UserProvider');
  return user;
}
```

### Error Boundaries

```tsx
// SIEMPRE: Error boundary para secciones
<ErrorBoundary fallback={<ErrorMessage />}>
  <UserProfile />
</ErrorBoundary>
```

### Loading States

```tsx
// SIEMPRE: Manejar loading state
if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} />;
return <Content data={data} />;

// O con Suspense
<Suspense fallback={<Skeleton />}>
  <AsyncComponent />
</Suspense>
```

## Prohibiciones

### NUNCA usar:

```tsx
// NO: Index como key (excepto listas estaticas)
{items.map((item, index) => <Item key={index} />)}  // NO!
{items.map(item => <Item key={item.id} />)}         // BIEN

// NO: Inline styles para logica de UI
<div style={{ display: isVisible ? 'block' : 'none' }}>  // NO
<div className={isVisible ? styles.visible : styles.hidden}>  // BIEN

// NO: any
const data: any = fetchData();  // NUNCA

// NO: Non-null assertion sin verificar
user!.name  // NO - verificar primero

// NO: console.log en produccion
console.log('debug');  // NO - usar logger

// NO: Inline functions en render (sin useCallback)
<Button onClick={() => handleClick(id)} />  // NO si causa re-renders
<Button onClick={handleClick} />            // BIEN

// NO: useEffect para derivar estado
useEffect(() => {
  setFullName(`${first} ${last}`);
}, [first, last]);  // NO!

const fullName = `${first} ${last}`;  // BIEN - derivar directamente
```

## Performance

### Memoization

```tsx
// useMemo: Calculos costosos
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// useCallback: Funciones pasadas a children
const handleSelect = useCallback((id: string) => {
  setSelectedId(id);
}, []);

// memo: Componentes con props complejas que re-renderizan mucho
export const UserList = memo(function UserList({ users }: Props) {
  return <ul>{users.map(u => <UserItem key={u.id} user={u} />)}</ul>;
});
```

### Code Splitting

```tsx
// Dynamic imports para componentes pesados
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <Skeleton />,
  ssr: false,
});
```

## Checklist Pre-Commit

- [ ] No any
- [ ] No console.log
- [ ] Keys unicas en listas
- [ ] Loading y error states manejados
- [ ] Server Components por defecto (Next.js)
- [ ] useCallback/useMemo donde apropiado
- [ ] Props tipadas
- [ ] No mutacion de estado
