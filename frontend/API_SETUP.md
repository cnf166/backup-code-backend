# API Configuration Guide

## Overview

This project uses **Axios** with environment variables for flexible API configuration across different environments (development, production, etc.).

## 📁 File Structure

```
frontend/
├── .env                    # Development environment
├── .env.local             # Local overrides (not committed)
├── .env.production        # Production environment
├── .env.example           # Example template
└── src/
    └── api/
        ├── client.ts      # Main Axios instance with interceptors
        └── services.ts    # API service functions
```

---

## 🔧 Environment Variables

### Available Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VITE_API_URL` | Backend API base URL | `http://localhost:8000/api/v1` |
| `VITE_ENV` | Environment name | `development` |
| `VITE_API_TIMEOUT` | Request timeout (ms) | `10000` |
| `VITE_ENABLE_API_LOGGING` | Enable console logging | `true` |

### Environment Files

#### `.env` (Development)
```env
VITE_API_URL=http://localhost:8000/api/v1
VITE_ENV=development
VITE_API_TIMEOUT=10000
VITE_ENABLE_API_LOGGING=true
```

#### `.env.local` (Local Overrides)
```env
# Use this for your personal local settings
# This file is gitignored and won't be committed
VITE_API_URL=http://localhost:8000/api/v1
VITE_ENV=local
```

#### `.env.production` (Production)
```env
VITE_API_URL=https://your-production-api.com/api/v1
VITE_ENV=production
VITE_API_TIMEOUT=15000
VITE_ENABLE_API_LOGGING=false
```

---

## 🚀 Usage Examples

### 1. Basic API Call

```typescript
import { apiClient } from '@/api/client';

// GET request
const fetchUsers = async () => {
  try {
    const response = await apiClient.get('/users');
    return response.data;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
};

// POST request
const createUser = async (userData: any) => {
  try {
    const response = await apiClient.post('/users', userData);
    return response.data;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};

// PUT request
const updateUser = async (userId: number, userData: any) => {
  try {
    const response = await apiClient.put(`/users/${userId}`, userData);
    return response.data;
  } catch (error) {
    console.error('Error updating user:', error);
    throw error;
  }
};

// DELETE request
const deleteUser = async (userId: number) => {
  try {
    const response = await apiClient.delete(`/users/${userId}`);
    return response.data;
  } catch (error) {
    console.error('Error deleting user:', error);
    throw error;
  }
};
```

### 2. With Query Parameters

```typescript
import { apiClient } from '@/api/client';

const fetchOrders = async (filters: { status?: string; limit?: number }) => {
  try {
    const response = await apiClient.get('/orders', {
      params: filters, // Automatically converted to ?status=...&limit=...
    });
    return response.data;
  } catch (error) {
    console.error('Error fetching orders:', error);
    throw error;
  }
};

// Usage
const orders = await fetchOrders({ status: 'pending', limit: 10 });
```

### 3. Using Auth Token

```typescript
import { apiClient, setAuthToken, clearAuthToken } from '@/api/client';

// After successful login
const login = async (email: string, password: string) => {
  try {
    const response = await apiClient.post('/auth/login', { email, password });
    const { token } = response.data;

    // Store token - will be automatically added to all future requests
    setAuthToken(token);

    return response.data;
  } catch (error) {
    console.error('Login failed:', error);
    throw error;
  }
};

// Logout
const logout = () => {
  clearAuthToken(); // Removes token from localStorage
  // Redirect to login page
};
```

### 4. With React Query (Recommended)

```typescript
import { useQuery, useMutation } from '@tanstack/react-query';
import { apiClient } from '@/api/client';

// GET with useQuery
export const useDishes = () => {
  return useQuery({
    queryKey: ['dishes'],
    queryFn: async () => {
      const response = await apiClient.get('/resources/dishes/');
      return response.data;
    },
  });
};

// POST with useMutation
export const useCreateOrder = () => {
  return useMutation({
    mutationFn: async (orderData: OrderCreate) => {
      const response = await apiClient.post('/orders', orderData);
      return response.data;
    },
  });
};

// Usage in component
function MyComponent() {
  const { data: dishes, isLoading } = useDishes();
  const createOrder = useCreateOrder();

  const handleCreateOrder = async () => {
    try {
      await createOrder.mutateAsync({ table_id: 1, status_id: 1 });
      toast.success('Order created!');
    } catch (error) {
      // Error is already handled by interceptor
    }
  };

  return <div>...</div>;
}
```

### 5. Custom Request Configuration

```typescript
import { apiClient } from '@/api/client';

// Custom timeout
const response = await apiClient.get('/slow-endpoint', {
  timeout: 30000, // 30 seconds
});

// Custom headers
const response = await apiClient.post('/upload', formData, {
  headers: {
    'Content-Type': 'multipart/form-data',
  },
});

// Cancel request
const controller = new AbortController();
const response = await apiClient.get('/data', {
  signal: controller.signal,
});
// Later: controller.abort();
```

---

## 🛡️ Built-in Features

### 1. **Automatic Auth Token Injection**
- Tokens are automatically added to `Authorization` header
- Use `setAuthToken(token)` after login
- Use `clearAuthToken()` after logout

### 2. **Comprehensive Error Handling**
- Automatic toast notifications for errors
- Handles FastAPI validation errors (422)
- Network error detection
- Auto-logout on 401 (Unauthorized)

### 3. **Request/Response Logging**
- Enabled in development: `VITE_ENABLE_API_LOGGING=true`
- Shows method, URL, params, data
- Displays response status and data
- Use 🚀 for requests, ✅ for success, ❌ for errors

### 4. **Timeout Management**
- Global timeout: `VITE_API_TIMEOUT`
- Per-request timeout override available
- Prevents hanging requests

---

## 📦 Deployment

### Step 1: Set Production URL

Update `.env.production`:
```env
VITE_API_URL=https://your-production-api.com/api/v1
VITE_ENV=production
VITE_ENABLE_API_LOGGING=false
```

### Step 2: Build for Production

```bash
npm run build
```

Vite automatically uses `.env.production` during build.

### Step 3: Deploy

Deploy the `dist` folder to your hosting provider (Vercel, Netlify, etc.).

### Environment Variables on Hosting Platforms

#### Vercel
```
VITE_API_URL=https://api.yourapp.com/api/v1
VITE_ENV=production
VITE_ENABLE_API_LOGGING=false
```

#### Netlify
```
VITE_API_URL=https://api.yourapp.com/api/v1
VITE_ENV=production
VITE_ENABLE_API_LOGGING=false
```

---

## 🐛 Debugging

### Check Current Configuration

```typescript
import { getApiConfig } from '@/api/client';

console.log(getApiConfig());
// {
//   baseURL: 'http://localhost:8000/api/v1',
//   timeout: 10000,
//   enableLogging: true,
//   environment: 'development'
// }
```

### Enable Request Logging

Set in `.env`:
```env
VITE_ENABLE_API_LOGGING=true
```

### Check if Authenticated

```typescript
import { isAuthenticated, getAuthToken } from '@/api/client';

console.log('Is authenticated:', isAuthenticated());
console.log('Current token:', getAuthToken());
```

---

## ⚠️ Common Issues

### Issue 1: CORS Errors

**Problem:** API requests fail with CORS errors

**Solution:** Configure your backend to allow your frontend origin:
```python
# FastAPI backend
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "https://your-frontend.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Issue 2: Environment Variables Not Loading

**Problem:** `VITE_API_URL` is undefined

**Solutions:**
1. Restart dev server after changing `.env`
2. Ensure variable starts with `VITE_`
3. Check file is named exactly `.env` (no extra extensions)

### Issue 3: 401 Unauthorized

**Problem:** Requests fail with 401 even after login

**Solutions:**
1. Check token is stored: `localStorage.getItem('auth_token')`
2. Verify token format in request headers
3. Check token expiration on backend

---

## 📚 Additional Resources

- [Vite Environment Variables](https://vitejs.dev/guide/env-and-mode.html)
- [Axios Documentation](https://axios-http.com/docs/intro)
- [React Query Guide](https://tanstack.com/query/latest/docs/react/overview)

---

## ✅ Best Practices

1. **Never commit `.env.local`** - Use it for sensitive local credentials
2. **Use `.env.example`** - Document all required variables
3. **Disable logging in production** - Set `VITE_ENABLE_API_LOGGING=false`
4. **Use HTTPS in production** - Always use secure URLs
5. **Handle errors gracefully** - Let interceptors handle common errors
6. **Use React Query** - For automatic caching and state management
7. **Validate environment** - Check `getApiConfig()` after deployment
