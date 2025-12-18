# Quick Start Guide - API Setup

## ✅ What's Already Done

Your API is now fully configured with:
- ✅ Environment variables for different environments
- ✅ Automatic auth token injection
- ✅ Error handling with toast notifications
- ✅ Request/response logging in development
- ✅ FastAPI validation error parsing

---

## 🚀 Quick Start (5 Minutes)

### 1. Check Your Environment File

Open `.env` and verify:
```env
VITE_API_URL=http://localhost:8000/api/v1
VITE_ENV=development
VITE_ENABLE_API_LOGGING=true
```

### 2. Your API Calls Already Work!

The existing API calls in `src/api/services.ts` already use the configured client:

```typescript
import apiClient from './client';

// Example: Get all dishes
export const dishesApi = {
  getAll: (filters?: DishFilter) =>
    apiClient.get<Dish[]>('/resources/dishes/', {
      params: { ...filters, include_tags: true }
    }),
};
```

### 3. Test It

```bash
# Make sure backend is running
# Then start frontend
npm run dev
```

Open browser console and you'll see:
```
🚀 API Request: GET /resources/dishes/
✅ API Response: { status: 200, data: [...] }
```

---

## 📝 Common Use Cases

### Making a New API Call

```typescript
// src/api/services.ts
import apiClient from './client';

export const myNewApi = {
  // GET request
  getItems: () => apiClient.get('/items'),

  // POST request
  createItem: (data: ItemCreate) =>
    apiClient.post('/items', data),

  // PUT request
  updateItem: (id: number, data: ItemUpdate) =>
    apiClient.put(`/items/${id}`, data),

  // DELETE request
  deleteItem: (id: number) =>
    apiClient.delete(`/items/${id}`),
};
```

### Using with React Query (Your Current Setup)

```typescript
// src/hooks/useApi.ts
import { useQuery, useMutation } from '@tanstack/react-query';
import { dishesApi } from '../api/services';

// Already working in your codebase!
export const useDishes = (filters?: DishFilter) => {
  return useQuery({
    queryKey: ['dishes', filters],
    queryFn: async () => {
      const response = await dishesApi.getAll(filters);
      return response.data;
    },
  });
};
```

### Adding Authentication

```typescript
import { setAuthToken, clearAuthToken } from '@/api/client';

// After login
const handleLogin = async (email: string, password: string) => {
  const response = await apiClient.post('/auth/login', { email, password });
  setAuthToken(response.data.token);
};

// On logout
const handleLogout = () => {
  clearAuthToken();
  navigate('/login');
};
```

---

## 🌐 Deployment Checklist

### Before Deploying

1. **Update Production URL**

Edit `.env.production`:
```env
VITE_API_URL=https://your-backend.com/api/v1
VITE_ENV=production
VITE_ENABLE_API_LOGGING=false
```

2. **Build**
```bash
npm run build
```

3. **Test Build Locally**
```bash
npm run preview
```

4. **Deploy**

For Vercel:
```bash
vercel --prod
```

For Netlify:
```bash
netlify deploy --prod
```

### On Your Hosting Platform

Add environment variables:
- `VITE_API_URL` = Your production backend URL
- `VITE_ENV` = `production`
- `VITE_ENABLE_API_LOGGING` = `false`

---

## 🐛 Troubleshooting

### Problem: API calls return 404

**Check:**
```typescript
import { getApiConfig } from '@/api/client';
console.log(getApiConfig());
```

Expected output:
```javascript
{
  baseURL: 'http://localhost:8000/api/v1',
  timeout: 10000,
  enableLogging: true,
  environment: 'development'
}
```

### Problem: CORS errors

**Solution:** Update your FastAPI backend:
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Your frontend URL
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Problem: Environment variables not loading

**Solution:**
1. Restart dev server: `npm run dev`
2. Check variable name starts with `VITE_`
3. Verify file is named exactly `.env`

---

## 📊 What You Get Out of the Box

### Development Environment
- ✅ Request/response logging with emojis 🚀 ✅ ❌
- ✅ Detailed error messages in console
- ✅ Toast notifications for API errors
- ✅ 10-second timeout

### Production Environment
- ✅ No console logging (clean console)
- ✅ User-friendly error messages
- ✅ 15-second timeout
- ✅ Automatic token handling

### Error Handling
- ✅ Network errors → "Cannot connect to server"
- ✅ 401 Unauthorized → Auto logout
- ✅ 422 Validation → Shows field errors
- ✅ 500 Server Error → Generic message
- ✅ All errors → Toast notification

---

## 🎯 Next Steps

1. **Test Your Current Setup**
   - Open browser DevTools → Console
   - Navigate to any page that fetches data
   - Check for 🚀 and ✅ logs

2. **Add Authentication** (if needed)
   - Implement login endpoint
   - Use `setAuthToken()` after successful login
   - Token will be auto-added to all requests

3. **Deploy to Production**
   - Update `.env.production` with your backend URL
   - Build and deploy
   - Verify API calls work in production

---

## 📚 Full Documentation

For complete documentation, see [API_SETUP.md](./API_SETUP.md)

---

## ✨ Key Features

```typescript
// Automatic auth token
setAuthToken('your-jwt-token');
// All requests now include: Authorization: Bearer your-jwt-token

// Error handling built-in
try {
  await apiClient.get('/data');
} catch (error) {
  // Already logged to console
  // Already showed toast notification
  // You can add custom handling here
}

// Environment-aware
// Development: Full logging
// Production: Clean, optimized

// Works with React Query
const { data } = useDishes(); // Automatic caching, refetching, etc.
```

---

**That's it! Your API is ready to use. No more hardcoded URLs! 🎉**
