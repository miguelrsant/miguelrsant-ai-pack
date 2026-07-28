---
name: react-vite-tailwind-integration
description: Integrate React, Vite, TypeScript, and TailwindCSS with Django REST Framework APIs following professional frontend architecture patterns
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# React + Vite + TypeScript + Tailwind Integration

## Purpose

Use this skill to integrate React frontend with Vite build tool, TypeScript type safety, TailwindCSS styling, and Django REST Framework backend APIs. This skill provides comprehensive guidance on frontend architecture, state management, API integration, and best practices.

## Instructions

### Project Setup

1. **Initialize React Vite Project**
   ```bash
   npm create vite@latest frontend -- --template react-ts
   cd frontend
   npm install
   ```

2. **Install TailwindCSS**
   ```bash
   npm install -D tailwindcss postcss autoprefixer
   npx tailwindcss init -p
   ```

3. **Install Additional Dependencies**
   ```bash
   # API client
   npm install axios

   # State management (choose one based on complexity)
   npm install zustand  # for simple local state
   # OR
   npm install @reduxjs/toolkit react-redux  # for complex global state

   # Form handling
   npm install react-hook-form zod @hookform/resolvers

   # Date handling
   npm install date-fns

   # Testing
   npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event vitest jsdom

   # Icons
   npm install lucide-react  # or @heroicons/react
   ```

### Directory Structure

Organize frontend code by feature, not by file type:

```typescript
src/
├── components/           # Reusable components
│   ├── ui/              # UI primitives (Button, Input, Modal)
│   ├── common/          # Shared application components
│   └── layout/          # Layout components (Header, Sidebar)
├── features/            # Feature-based organization
│   ├── auth/
│   │   ├── components/  # Feature-specific components
│   │   ├── hooks/       # Feature-specific hooks
│   │   ├── services/    # API calls for this feature
│   │   ├── types/       # TypeScript types
│   │   └── index.tsx    # Public API for feature
├── hooks/               # Shared hooks (useAuth, useDebounce)
├── lib/                 # Utility libraries
│   ├── api/             # API client configuration
│   ├── utils/           # Helper functions
│   └── constants/       # Application constants
├── pages/               # Route pages /screens
│   ├── LoginPage.tsx
│   └── DashboardPage.tsx
├── services/            # API services (if not using feature-based)
├── types/               # Global TypeScript types
├── App.tsx
├── main.tsx
└── vite.config.ts
```

### API Integration Layer

1. **Configure Axios Client**
   ```typescript
   // src/lib/api/client.ts
   import axios from 'axios';

   const apiClient = axios.create({
     baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api',
     timeout: 10000,
     headers: {
       'Content-Type': 'application/json',
     },
   });

   // Add auth token to requests
   apiClient.interceptors.request.use((config) => {
     const token = localStorage.getItem('auth_token');
     if (token) {
       config.headers.Authorization = `Bearer ${token}`;
     }
     return config;
   });

   // Handle common errors
   apiClient.interceptors.response.use(
     (response) => response,
     (error) => {
       if (error.response?.status === 401) {
         // Redirect to login
         window.location.href = '/login';
       }
       return Promise.reject(error);
     }
   );

   export default apiClient;
   ```

2. **Type-Safe API Services**
   ```typescript
   // src/services/users.ts
   import apiClient from '@/lib/api/client';
   import type { User } from '@/types';

   export interface CreateUserRequest {
     email: string;
     password: string;
     first_name: string;
     last_name: string;
   }

   export interface UpdateUserRequest {
     first_name?: string;
     last_name?: string;
   }

   export const usersService = {
     list: async (params?: { page?: number; page_size?: number }) => {
       const response = await apiClient.get<{ count: number; results: User[] }>('/users/', {
         params,
       });
       return response.data;
     },

     get: async (id: number) => {
       const response = await apiClient.get<User>(`/users/${id}/`);
       return response.data;
     },

     create: async (data: CreateUserRequest) => {
       const response = await apiClient.post<User>('/users/', data);
       return response.data;
     },

     update: async (id: number, data: UpdateUserRequest) => {
       const response = await apiClient.patch<User>(`/users/${id}/`, data);
       return response.data;
     },

     delete: async (id: number) => {
       await apiClient.delete(`/users/${id}/`);
     },
   };
   ```

3. **Custom Hook for State Management**
   ```typescript
   // src/hooks/useUsers.ts
   import { useState, useEffect } from 'react';
   import { usersService } from '@/services/users';
   import type { User } from '@/types';

   export function useUsers() {
     const [users, setUsers] = useState<User[]>([]);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState<string | null>(null);

     useEffect(() => {
       const fetchUsers = async () => {
         try {
           setLoading(true);
           const data = await usersService.list();
           setUsers(data.results);
         } catch (err) {
           setError('Failed to load users');
           console.error(err);
         } finally {
           setLoading(false);
         }
       };

       fetchUsers();
     }, []);

     return { users, loading, error, refetch: () => { /* implement */ } };
   }
   ```

### Component Development Guidelines

1. **Component Structure**
   ```typescript
   // src/components/common/Button.tsx
   import { ButtonHTMLAttributes, forwardRef } from 'react';

   export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
     variant?: 'primary' | 'secondary' | 'danger';
     loading?: boolean;
   }

   export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
     ({ variant = 'primary', loading, disabled, children, ...props }, ref) => {
       const baseClasses = 'px-4 py-2 rounded-lg font-medium transition-colors';
       const variantClasses = {
         primary: 'bg-blue-500 hover:bg-blue-600 text-white',
         secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-800',
         danger: 'bg-red-500 hover:bg-red-600 text-white',
       };

       return (
         <button
           ref={ref}
           className={`${baseClasses} ${variantClasses[variant]}`}
           disabled={disabled || loading}
           {...props}
         >
           {loading ? 'Loading...' : children}
         </button>
       );
     }
   );

   Button.displayName = 'Button';
   ```

2. **Always Handle States**
   ```typescript
   // Good: Handles all states
   export function UserList() {
     const { users, loading, error } = useUsers();

     if (loading) return <SkeletonLoader count={5} />;
     if (error) return <ErrorMessage message={error} />;
     if (users.length === 0) return <EmptyState message="No users found" />;

     return (
       <div>
         {users.map((user) => (
           <UserCard key={user.id} user={user} />
         ))}
       </div>
     );
   }

   // Bad: No state handling
   export function UserList() {
     const { users } = useUsers();
     return (
       <div>
         {users.map((user) => (
           <UserCard key={user.id} user={user} />
         ))}
       </div>
     );
   }
   ```

3. **Form Handling with react-hook-form + zod**
   ```typescript
   // src/components/forms/CreateUserForm.tsx
   import { useForm } from 'react-hook-form';
   import { zodResolver } from '@hookform/resolvers/zod';
   import { z } from 'zod';
   import { usersService } from '@/services/users';
   import { Button } from '@/components/common/Button';

   const createUserSchema = z.object({
     email: z.string().email('Invalid email address'),
     password: z.string().min(8, 'Password must be at least 8 characters'),
     first_name: z.string().min(1, 'First name is required'),
     last_name: z.string().min(1, 'Last name is required'),
   });

   type CreateUserFormData = z.infer<typeof createUserSchema>;

   export function CreateUserForm() {
     const { register, handleSubmit, formState: { errors, isSubmitting }, setError } =
       useForm<CreateUserFormData>({
         resolver: zodResolver(createUserSchema),
       });

     const onSubmit = async (data: CreateUserFormData) => {
       try {
         await usersService.create(data);
         // Navigate to list page
       } catch (error) {
         setError('root', { message: 'Failed to create user' });
       }
     };

     return (
       <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
         <div>
           <label htmlFor="email" className="block mb-1">Email</label>
           <input
             id="email"
             type="email"
             {...register('email')}
             className="w-full px-3 py-2 border rounded-lg"
           />
           {errors.email && (
             <p className="text-red-500 text-sm">{errors.email.message}</p>
           )}
         </div>

         {/* Other fields */}

         <Button type="submit" loading={isSubmitting}>
           Create User
         </Button>
       </form>
     );
   }
   ```

## Best Practices

### 1. TypeScript Best Practices

- Use **strict mode**: `strict: true` in `tsconfig.json`
- **Avoid `any`**: Use `unknown` instead for truly dynamic data
- **Type API responses**: Generate types from OpenAPI schema
- **Define interfaces for component props**: Always type props
- **Use type inference where possible**: Don't over-type!
- **Discriminated unions**: Use for complex state machines

Bad Example:
```typescript
function fetchUser(id: number): any {
  return axios.get(`/users/${id}/`);
}
```

Good Example:
```typescript
function fetchUser(id: number): Promise<User> {
  return axios.get<User>(`/users/${id}/`).then(res => res.data);
}
```

### 2. State Management Decision Rules

| Scenario | Recommended Approach |
|----------|---------------------|
| Single form state | React useState |
| Multiple related forms | react-hook-form |
| Local component state | React useState / useReducer |
| Global state (simple) | Zustand |
| Global state (complex) | Redux Toolkit |
| Server state (API) | TanStack Query (React Query) |
| Authentication context | React Context + useReducer |

### 3. Component Principles

1. **Single Responsibility**: Components do one thing well
2. **Composition over Inheritance**: Use children prop and render props
3. **Prop drilling vs. Context**: Use Context for global auth/theme, not for local state
4. **Controlled components**: Use controlled components for forms
5. **Error boundaries**: Wrap components in error boundaries

### 4. Performance Considerations

- Use `React.memo()` for expensive components
- Use `useMemo()` for expensive calculations
- Use `useCallback()` for functions passed to child components
- Lazy load routes with React Suspense
- Use code splitting with dynamic imports

```typescript
// Lazy load component
const DashboardPage = React.lazy(() => import('./pages/DashboardPage'));

function App() {
  return (
    <React.Suspense fallback={<Loading />}>
      <DashboardPage />
    </React.Suspense>
  );
}
```

### 5. Testing

- Test behavior, not implementation
- Use @testing-library/react for component testing
- Test user interactions, not DOM manipulation
- Mock API calls with msw or axios-mock-adapter

```typescript
// Good: Tests behavior
test('submits form when valid', async () => {
  render(<CreateUserForm />);
  await userEvent.type(screen.getByLabelText('Email'), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: 'Create User' }));
  expect(mockCreateUser).toHaveBeenCalledWith(/* ... */);
});

// Bad: Tests implementation
test('calls handleSubmit on submit', () => {
  // Tests internal implementation detail
});
```

## Common Mistakes to Avoid

1. **API Logic in Components**: Extract API calls to services/hooks
2. **Prop Drilling**: Use Context or composition instead
3. **Global State for Local Data**: Use useState for local component state
4. **Type Assertions**: Use type guards instead of `as any`
5. **Forgotten Error States**: Always handle loading/error/empty states
6. **Direct DOM Manipulation**: Use refs and state instead
7. **Ignoring Accessibility**: Add ARIA labels, keyboard navigation
8. **Skipping Tests**: Write tests for components and hooks
9. **Large Components**: Break down large components into smaller ones
10. **Hardcoded Values**: Extract constants to config or environment

## TailwindCSS Configuration

```javascript
// tailwind.config.js
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
        // Add brand colors
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

## Environment Configuration

```bash
# .env.example
VITE_API_BASE_URL=http://localhost:8000/api
VITE_AUTH_TOKEN_KEY=auth_token
VITE_APP_NAME=My App
VITE_ENABLE_DEBUG=true
```

```typescript
// Access environment variables
const apiBaseUrl = import.meta.env.VITE_API_BASE_URL;
const isDebug = import.meta.env.VITE_ENABLE_DEBUG === 'true';
```

## Triggers

This skill is activated when:

- User mentions "React", "Vue", "frontend", "UI", "web interface"
- Setting up new React Vite project
- Integrating React with Django REST Framework
- Creating frontend components for API integration
- Configuring TypeScript or TailwindCSS
- Designing frontend architecture
- Implementing state management patterns
- Setting up frontend testing

## Related Skills

- `react-vite-tailwind-integration` — This skill, expanded for professional use
- `api-contracts-openapi` — API contract design and TypeScript type generation
- `django` — Backend architecture patterns
- `django-rest-framework` — Backend API design
- `testing` — Frontend testing strategies

## Notes

- Always generate TypeScript types from OpenAPI schema, not hand-write them
- Use feature-based directory structure for large applications
- Keep components small and focused (single responsibility)
- Always handle loading, error, and empty states
- Use TailwindCSS for styling, avoid writing custom CSS
- Prefer composition over inheritance in React
- Test components as users would interact with them
- Use React Query/TanStack Query for server state management (recommended)

## Migration from Legacy Code

If migrating from untyped/monolithic frontend code:

1. **Add TypeScript incrementally**: Enable `allowJs` in tsconfig.json
2. **Start with new code**: Write new code in TypeScript, convert old code when touching it
3. **Extract types**: Define types for API responses first
4. **Add tests**: Write tests before refactoring
5. **Break down components**: Split large components into smaller ones
6. **Modernize tooling**: Migrate to Vite, update dependencies

---

This skill provides comprehensive guidance for professional React development with TypeScript and TailwindCSS. For architectural decisions about Django backend, refer to `django` and `django-rest-framework` skills.