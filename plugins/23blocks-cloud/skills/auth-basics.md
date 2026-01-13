---
name: auth-basics
description: Implement 23blocks authentication and authorization
---

# 23blocks Authentication

Implement authentication using 23blocks SDK and patterns.

## SDK Initialization

### React/Next.js
```typescript
import { BlocksProvider, useAuth } from '@23blocks/react';

function App() {
  return (
    <BlocksProvider
      apiKey={process.env.NEXT_PUBLIC_BLOCKS_API_KEY}
      authUrl={process.env.NEXT_PUBLIC_BLOCKS_AUTH_URL}
    >
      <MyApp />
    </BlocksProvider>
  );
}
```

### Angular
```typescript
// app.config.ts
import { provideBlocks } from '@23blocks/angular';

export const appConfig = {
  providers: [
    provideBlocks({
      apiKey: environment.blocksApiKey,
      authUrl: environment.blocksAuthUrl
    })
  ]
};
```

## Authentication Flows

### Login
```typescript
const { login } = useAuth();

await login({
  email: 'user@example.com',
  password: 'secure_password'
});
```

### Registration
```typescript
const { register } = useAuth();

await register({
  email: 'user@example.com',
  password: 'secure_password',
  name: 'John Doe'
});
```

### Logout
```typescript
const { logout } = useAuth();
await logout();
```

## Protected Routes

### React
```typescript
function ProtectedRoute({ children }) {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) return <Loading />;
  if (!isAuthenticated) return <Navigate to="/login" />;

  return children;
}
```

### Angular
```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);

  return auth.isAuthenticated() || router.parseUrl('/login');
};
```

## Token Management

- Access tokens: Short-lived (15 min default)
- Refresh tokens: Long-lived (7 days default)
- Auto-refresh: SDK handles automatically
- Storage: Secure storage (httpOnly cookies or secure localStorage)

## Usage

When implementing authentication:
1. Initialize SDK with credentials
2. Implement login/register forms
3. Set up protected routes
4. Handle loading and error states
5. Implement logout functionality
