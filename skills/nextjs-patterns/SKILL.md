---
name: nextjs-patterns
description: Patrones y mejores practicas para Next.js 14+ con App Router, Server Components, y optimizaciones
---

# Next.js 14+ Patterns (App Router)

## Cuando Usar
- Estructurando aplicacion Next.js
- Decidiendo Server vs Client Components
- Implementando data fetching
- Optimizando performance

## Project Structure

```
src/
├── app/                          # App Router
│   ├── (auth)/                   # Route Group (no afecta URL)
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx            # Layout compartido
│   │   ├── page.tsx              # /dashboard
│   │   └── settings/
│   │       └── page.tsx          # /dashboard/settings
│   ├── api/                      # API Routes
│   │   └── users/
│   │       └── route.ts
│   ├── layout.tsx                # Root Layout
│   ├── page.tsx                  # Home /
│   ├── loading.tsx               # Loading UI
│   ├── error.tsx                 # Error boundary
│   ├── not-found.tsx             # 404 page
│   └── globals.css
├── components/
│   ├── ui/                       # Primitivos (Button, Input)
│   ├── forms/
│   └── layout/
├── lib/
│   ├── api.ts                    # API client
│   └── utils.ts
├── hooks/
├── types/
└── services/
```

## Server vs Client Components

### Server Components (Default)
```tsx
// app/users/page.tsx
// NO 'use client' = Server Component

import { getUsers } from '@/services/user-service';

export default async function UsersPage() {
  // Fetch directamente - no useState/useEffect
  const users = await getUsers();

  return (
    <main>
      <h1>Users</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </main>
  );
}
```

### Client Components
```tsx
// components/search-input.tsx
'use client'; // Necesario para: useState, useEffect, event handlers

import { useState, useTransition } from 'react';
import { useRouter } from 'next/navigation';

export function SearchInput() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const router = useRouter();

  const handleSearch = (value: string) => {
    setQuery(value);
    startTransition(() => {
      router.push(`/search?q=${encodeURIComponent(value)}`);
    });
  };

  return (
    <input
      value={query}
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="Search..."
      disabled={isPending}
    />
  );
}
```

### Cuando Usar Cada Uno

| Server Component | Client Component |
|------------------|------------------|
| Fetch data | useState, useEffect |
| Access backend resources | Event listeners (onClick) |
| Keep secrets on server | Browser APIs |
| Reduce JS bundle | Interactividad |
| Static content | Forms con validacion |

## Data Fetching

### Server Component Fetch
```tsx
// Con caching automatico
async function getUsers() {
  const res = await fetch('https://api.example.com/users', {
    // Cache strategies
    cache: 'force-cache',        // Default - cache indefinido
    // cache: 'no-store',        // No cache - siempre fresh
    // next: { revalidate: 60 }, // ISR - revalidar cada 60s
  });

  if (!res.ok) throw new Error('Failed to fetch users');
  return res.json();
}

export default async function UsersPage() {
  const users = await getUsers();
  return <UserList users={users} />;
}
```

### Parallel Data Fetching
```tsx
export default async function DashboardPage() {
  // Ejecutar en paralelo
  const [users, orders, stats] = await Promise.all([
    getUsers(),
    getOrders(),
    getStats(),
  ]);

  return (
    <div>
      <UserSection users={users} />
      <OrderSection orders={orders} />
      <StatsSection stats={stats} />
    </div>
  );
}
```

### Streaming con Suspense
```tsx
import { Suspense } from 'react';

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Stats carga rapido */}
      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>

      {/* Orders puede tardar */}
      <Suspense fallback={<OrdersSkeleton />}>
        <RecentOrders />
      </Suspense>
    </div>
  );
}

// Cada componente fetcha sus datos
async function Stats() {
  const stats = await getStats(); // 200ms
  return <StatsCard stats={stats} />;
}

async function RecentOrders() {
  const orders = await getOrders(); // 2000ms
  return <OrderList orders={orders} />;
}
```

## API Routes

### Route Handlers
```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = searchParams.get('page') ?? '1';

  const users = await db.users.findMany({
    skip: (parseInt(page) - 1) * 10,
    take: 10,
  });

  return NextResponse.json(users);
}

export async function POST(request: NextRequest) {
  const body = await request.json();

  // Validacion
  const parsed = createUserSchema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json(
      { error: parsed.error.issues },
      { status: 400 }
    );
  }

  const user = await db.users.create({ data: parsed.data });

  return NextResponse.json(user, { status: 201 });
}
```

### Dynamic Route Params
```tsx
// app/api/users/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.users.findUnique({
    where: { id: params.id },
  });

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}
```

## Layouts

### Root Layout
```tsx
// app/layout.tsx
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: { default: 'My App', template: '%s | My App' },
  description: 'My awesome app',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Providers>
          <Header />
          <main>{children}</main>
          <Footer />
        </Providers>
      </body>
    </html>
  );
}
```

### Nested Layout
```tsx
// app/(dashboard)/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <Sidebar />
      <div className="flex-1">
        {children}
      </div>
    </div>
  );
}
```

## Metadata

### Static Metadata
```tsx
// app/about/page.tsx
export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn about our company',
  openGraph: {
    title: 'About Us',
    description: 'Learn about our company',
    images: ['/og-about.jpg'],
  },
};
```

### Dynamic Metadata
```tsx
// app/users/[id]/page.tsx
export async function generateMetadata({
  params,
}: {
  params: { id: string };
}): Promise<Metadata> {
  const user = await getUser(params.id);

  return {
    title: user.name,
    description: `Profile of ${user.name}`,
    openGraph: {
      title: user.name,
      images: [user.avatarUrl],
    },
  };
}
```

## Loading & Error States

### Loading
```tsx
// app/users/loading.tsx
export default function Loading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-4" />
      <div className="space-y-3">
        {[...Array(5)].map((_, i) => (
          <div key={i} className="h-16 bg-gray-200 rounded" />
        ))}
      </div>
    </div>
  );
}
```

### Error Boundary
```tsx
// app/users/error.tsx
'use client'; // Error components must be Client

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div className="text-center py-10">
      <h2 className="text-xl font-bold">Something went wrong!</h2>
      <button
        onClick={reset}
        className="mt-4 px-4 py-2 bg-blue-500 text-white rounded"
      >
        Try again
      </button>
    </div>
  );
}
```

## Server Actions

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  await db.users.create({
    data: { name, email },
  });

  revalidatePath('/users');
}

// app/users/new/page.tsx
import { createUser } from '../actions';

export default function NewUserPage() {
  return (
    <form action={createUser}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

## Image Optimization

```tsx
import Image from 'next/image';

export function Avatar({ user }: { user: User }) {
  return (
    <Image
      src={user.avatarUrl}
      alt={user.name}
      width={48}
      height={48}
      className="rounded-full"
      priority={false} // true para above-the-fold
    />
  );
}

// Remote images - configurar en next.config.js
// images: { remotePatterns: [{ hostname: 'example.com' }] }
```

## Best Practices

1. **Server Components por defecto**: Solo 'use client' cuando necesario
2. **Client boundary abajo**: Minimizar JS enviado al cliente
3. **Parallel fetching**: Promise.all para requests independientes
4. **Streaming**: Suspense para UX progresiva
5. **Revalidation**: Configurar cache apropiadamente
6. **Error handling**: error.tsx en cada route segment
