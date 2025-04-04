# Next.js Development Standards

This guide outlines our organization's standards for Next.js development, focusing on frontend applications.

> 📌 **Return to**: [Main Development Guide](README.md)

## Quick Start for Next.js Projects

```bash
# 1. Set up Node.js environment
brew install nvm
nvm install 20
nvm use 20

# 2. Clone repository
git clone git@github.com:organization/project-name-frontend.git
cd project-name-frontend

# 3. Install dependencies
npm install

# 4. Run development server
npm run dev

# 5. Create a new feature branch
git checkout -b feature/your-feature-name
```

## Environment Setup

### Node.js Version Management

We use `nvm` (Node Version Manager) to manage Node.js versions:

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

# Add to your shell (for bash)
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.bashrc

# Install Node.js version
nvm install 20

# Set as default version
nvm alias default 20
```

### Project Initialization

For new projects:

```bash
# Create a new Next.js project with TypeScript and App Router
npx create-next-app@latest my-project --typescript --eslint --app --tailwind

# Navigate to the project
cd my-project
```

We standardized on the App Router after discovering that several large features in our customer portal (especially those requiring nested layouts and parallel data fetching) were significantly easier to implement compared to the Pages Router.

## Project Structure

Standard Next.js project structure with App Router:

```
frontend/
├── public/              # Static files
├── src/                 # Source code
│   ├── app/             # Next.js App Router
│   │   ├── layout.tsx   # Root layout
│   │   ├── page.tsx     # Home page
│   │   ├── globals.css  # Global styles
│   │   └── [feature]/   # Feature routes
│   ├── components/      # React components
│   │   ├── ui/          # UI components (buttons, inputs, etc.)
│   │   └── [feature]/   # Feature-specific components
│   ├── lib/             # Utility functions
│   │   ├── utils.ts     # General utilities
│   │   └── api.ts       # API client
│   ├── hooks/           # Custom hooks
│   ├── context/         # React context
│   ├── types/           # TypeScript types
│   ├── styles/          # CSS modules/styles
│   └── services/        # API services
├── .env.local.example   # Example environment variables
├── next.config.js       # Next.js configuration
├── package.json         # Project dependencies
├── tsconfig.json        # TypeScript configuration
└── README.md            # Project documentation
```

### Component Structure

For complex components, use a dedicated directory:

```
components/
├── ui/                 # Shared UI components
│   ├── Button/
│   │   ├── index.tsx   # Export file
│   │   ├── Button.tsx  # Component implementation
│   │   └── Button.test.tsx  # Component tests
│   └── Card/
│       ├── index.tsx   # Export file
│       └── Card.tsx    # Component implementation
└── features/           # Feature-specific components
    └── UserProfile/
        ├── index.tsx   # Export file
        ├── UserProfile.tsx  # Component implementation
        └── UserProfileContext.tsx  # Component-specific context
```

## Code Quality Tools

### ESLint and Prettier

```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

#### ESLint Configuration (`.eslintrc.js`)

```javascript
module.exports = {
  extends: [
    'next/core-web-vitals',
    'plugin:@typescript-eslint/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error',
    'react/prop-types': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'react/display-name': 'off',
    'react/react-in-jsx-scope': 'off',
    'react/jsx-sort-props': [
      'warn',
      {
        callbacksLast: true,
        shorthandFirst: true,
        ignoreCase: true,
        reservedFirst: true,
      },
    ],
  },
};
```

#### Prettier Configuration (`.prettierrc`)

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "avoid"
}
```

### Husky and lint-staged

Set up pre-commit hooks:

```bash
# Install Husky and lint-staged
npm install --save-dev husky lint-staged

# Initialize Husky
npx husky install
npm set-script prepare "husky install"
npm run prepare

# Add pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"
```

Configure lint-staged in `package.json`:

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,css,md}": [
      "prettier --write"
    ]
  }
}
```

## Component Architecture

After experimentation with different component architecture patterns, we've found that clear separation of concerns leads to more maintainable code.

### Component Patterns

#### UI Components

Basic UI components follow this structure:

```tsx
// src/components/ui/Button/Button.tsx
import { ButtonHTMLAttributes, forwardRef } from 'react';
import { cn } from '@/lib/utils';

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', children, ...props }, ref) => {
    return (
      <button
        className={cn(
          'inline-flex items-center justify-center rounded-md font-medium transition-colors',
          'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2',
          {
            'bg-primary text-white hover:bg-primary/90': variant === 'primary',
            'bg-secondary text-white hover:bg-secondary/90': variant === 'secondary',
            'border border-gray-300 bg-transparent hover:bg-gray-100': variant === 'outline',
            'bg-transparent hover:bg-gray-100': variant === 'ghost',
          },
          {
            'h-8 px-3 text-sm': size === 'sm',
            'h-10 px-4': size === 'md',
            'h-12 px-6 text-lg': size === 'lg',
          },
          className
        )}
        ref={ref}
        {...props}
      >
        {children}
      </button>
    );
  }
);

Button.displayName = 'Button';

// src/components/ui/Button/index.tsx
export { Button, type ButtonProps } from './Button';
```

#### Feature Components

Feature components combine UI components with business logic:

```tsx
// src/components/features/UserProfile/UserProfile.tsx
import { useState } from 'react';
import { Button } from '@/components/ui/Button';
import { Card } from '@/components/ui/Card';
import { Avatar } from '@/components/ui/Avatar';
import { useUserData } from '@/hooks/useUserData';

interface UserProfileProps {
  userId: string;
}

export function UserProfile({ userId }: UserProfileProps) {
  const { data, isLoading, error } = useUserData(userId);
  const [isEditing, setIsEditing] = useState(false);
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading user</div>;
  
  return (
    <Card className="p-6">
      <div className="flex items-start gap-4">
        <Avatar 
          src={data.avatarUrl} 
          alt={data.name} 
          size="lg"
        />
        <div className="flex-1">
          <h2 className="text-2xl font-bold">{data.name}</h2>
          <p className="text-gray-500">{data.email}</p>
          <p className="mt-2">{data.bio}</p>
        </div>
        <Button 
          variant="outline"
          onClick={() => setIsEditing(!isEditing)}
        >
          {isEditing ? 'Cancel' : 'Edit Profile'}
        </Button>
      </div>
      {isEditing && (
        <div className="mt-4">
          {/* Edit form */}
        </div>
      )}
    </Card>
  );
}

// src/components/features/UserProfile/index.tsx
export { UserProfile } from './UserProfile';
```

### Layout Components

With App Router, create reusable layouts:

```tsx
// src/app/dashboard/layout.tsx
import { Sidebar } from '@/components/features/Dashboard/Sidebar';
import { Navbar } from '@/components/features/Dashboard/Navbar';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen flex">
      <Sidebar />
      <div className="flex-1">
        <Navbar />
        <main className="p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

## Data Fetching Strategies

After much experimentation, we've standardized on these data fetching patterns for different use cases:

### 1. Server Components with Server-side Data Fetching

For SEO-critical content and initial page loads:

```tsx
// src/app/products/[id]/page.tsx
import { ProductDetails } from '@/components/features/Products/ProductDetails';
import { getProduct } from '@/lib/api';
import { notFound } from 'next/navigation';

// Generate metadata
export async function generateMetadata({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  
  if (!product) {
    return {
      title: 'Product Not Found',
    };
  }
  
  return {
    title: product.name,
    description: product.description,
  };
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  
  if (!product) {
    notFound();
  }
  
  return <ProductDetails product={product} />;
}
```

### 2. React Query (TanStack Query) for Client Components

For interactive features that need fresh data:

```tsx
// src/lib/react-query.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      refetchOnWindowFocus: false,
    },
  },
});

// src/app/providers.tsx
'use client';

import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/lib/react-query';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

// src/app/layout.tsx
import { Providers } from './providers';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}

// src/hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { fetchProducts, createProduct } from '@/lib/api';

export function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  });
}

export function useCreateProduct() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createProduct,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}

// Usage in a component
// src/components/features/Products/ProductList.tsx
'use client';

import { useProducts } from '@/hooks/useProducts';

export function ProductList() {
  const { data, isLoading, error } = useProducts();
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading products</div>;
  
  return (
    <div>
      {data.map(product => (
        <div key={product.id}>
          {product.name}
        </div>
      ))}
    </div>
  );
}
```

### 3. SWR for Simpler Data Fetching Needs

```tsx
// src/hooks/useSWR.ts
import useSWR from 'swr';
import { fetcher } from '@/lib/api';

export function useUser(id: string) {
  return useSWR(id ? `/api/users/${id}` : null, fetcher);
}

// Usage in a component
// src/components/features/User/UserCard.tsx
'use client';

import { useUser } from '@/hooks/useSWR';

export function UserCard({ userId }: { userId: string }) {
  const { data, error, isLoading } = useUser(userId);
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading user</div>;
  
  return (
    <div>
      <h2>{data.name}</h2>
      <p>{data.email}</p>
    </div>
  );
}
```

### 4. Parallel Data Fetching with Suspense

```tsx
// src/app/dashboard/page.tsx
import { Suspense } from 'react';
import { RecentOrders } from '@/components/features/Dashboard/RecentOrders';
import { UserStats } from '@/components/features/Dashboard/UserStats';
import { RevenueChart } from '@/components/features/Dashboard/RevenueChart';
import { LoadingSkeleton } from '@/components/ui/LoadingSkeleton';

export default function DashboardPage() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
      <Suspense fallback={<LoadingSkeleton height="200px" />}>
        <UserStats />
      </Suspense>
      
      <Suspense fallback={<LoadingSkeleton height="200px" />}>
        <RevenueChart />
      </Suspense>
      
      <div className="md:col-span-2">
        <Suspense fallback={<LoadingSkeleton height="400px" />}>
          <RecentOrders />
        </Suspense>
      </div>
    </div>
  );
}
```

### Data Fetching Best Practices

After several performance issues in our customer portal, we established these best practices:

1. **Choose the Right Strategy for Each Use Case**
    - SEO content → Server Components
    - Interactive features → Client Components with React Query/SWR
    - Forms and mutations → React Query

2. **Optimize Loading States**
    - Use Suspense boundaries strategically
    - Create quality loading skeletons
    - Avoid layout shifts during loading

3. **Error Handling**
    - Add global error boundaries
    - Provide user-friendly error messages
    - Log errors for debugging

4. **Caching Strategy**
    - Configure proper staleTime and cacheTime
    - Use optimistic updates for mutations
    - Implement proper cache invalidation

Our product listing page saw a 45% improvement in LCP (Largest Contentful Paint) after switching from client-side to server component rendering.

## Testing

We use Jest and React Testing Library for testing:

```bash
# Install testing libraries
npm install --save-dev jest @testing-library/react @testing-library/jest-dom jest-environment-jsdom
```

Create a Jest configuration file (`jest.config.js`):

```javascript
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapper: {
    '^@/components/(.*)$': '<rootDir>/src/components/$1',
    '^@/app/(.*)$': '<rootDir>/src/app/$1',
    '^@/lib/(.*)$': '<rootDir>/src/lib/$1',
  },
  testEnvironment: 'jest-environment-jsdom',
  testPathIgnorePatterns: ['<rootDir>/node_modules/', '<rootDir>/.next/'],
};

module.exports = createJestConfig(customJestConfig);
```

Create a Jest setup file (`jest.setup.js`):

```javascript
import '@testing-library/jest-dom/extend-expect';

// Mock Next.js router
jest.mock('next/navigation', () => ({
  useRouter() {
    return {
      push: jest.fn(),
      replace: jest.fn(),
      prefetch: jest.fn(),
      back: jest.fn(),
      pathname: '/',
      query: {},
    };
  },
  usePathname() {
    return '/';
  },
  useSearchParams() {
    return new URLSearchParams();
  },
}));
```

Add test scripts to `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### Component Testing

Example component test:

```typescript
// src/components/ui/Button/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('applies the correct variant class', () => {
    render(<Button variant="primary">Primary Button</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-primary');
    
    render(<Button variant="secondary">Secondary Button</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-secondary');
  });

  it('calls onClick handler when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### Testing Hooks and Context

```typescript
// src/hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  it('should decrement counter', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(9);
  });
  
  it('should reset counter', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(0);
  });
});
```

### Integration Testing

For integration tests that involve API calls:

```typescript
// src/components/features/UserProfile/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';
import { useUserData } from '@/hooks/useUserData';

// Mock the hook
jest.mock('@/hooks/useUserData');

describe('UserProfile', () => {
  it('displays user data correctly', async () => {
    // Mock the hook implementation
    (useUserData as jest.Mock).mockReturnValue({
      data: {
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
        bio: 'Frontend developer',
        avatarUrl: '/avatar.jpg',
      },
      isLoading: false,
      error: null,
    });
    
    render(<UserProfile userId="123" />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByText('Frontend developer')).toBeInTheDocument();
  });
  
  it('shows loading state', () => {
    (useUserData as jest.Mock).mockReturnValue({
      data: null,
      isLoading: true,
      error: null,
    });
    
    render(<UserProfile userId="123" />);
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
  
  it('shows error state', () => {
    (useUserData as jest.Mock).mockReturnValue({
      data: null,
      isLoading: false,
      error: new Error('Failed to load user'),
    });
    
    render(<UserProfile userId="123" />);
    
    expect(screen.getByText('Error loading user')).toBeInTheDocument();
  });
});
```

### End-to-End Testing

For E2E testing, we use Cypress:

```bash
# Install Cypress
npm install --save-dev cypress
```

Add Cypress scripts to `package.json`:

```json
{
  "scripts": {
    "cypress": "cypress open",
    "cypress:run": "cypress run"
  }
}
```

Example Cypress test:

```typescript
// cypress/e2e/home.cy.ts
describe('Home Page', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('displays the main heading', () => {
    cy.get('h1').should('contain', 'Welcome to Our App');
  });

  it('navigates to the login page', () => {
    cy.get('a[href="/login"]').click();
    cy.url().should('include', '/login');
    cy.get('h1').should('contain', 'Login');
  });
});
```

## State Management

Based on our experience, we use these state management approaches for different scenarios:

### 1. React State and Context for Local State

```tsx
// src/context/ThemeContext.tsx
'use client';

import { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark' | 'system';

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Usage in a component
// src/components/ui/ThemeToggle.tsx
'use client';

import { useTheme } from '@/context/ThemeContext';
import { Button } from '@/components/ui/Button';

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();
  
  return (
    <Button 
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
      variant="outline"
    >
      {theme === 'dark' ? 'Light Mode' : 'Dark Mode'}
    </Button>
  );
}
```

### 2. Zustand for Global State

```bash
# Install Zustand
npm install zustand
```

```tsx
// src/store/cartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface Product {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartStore {
  items: Product[];
  totalItems: number;
  totalPrice: number;
  addItem: (product: Omit<Product, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      totalItems: 0,
      totalPrice: 0,
      
      addItem: (product) => {
        const currentItems = get().items;
        const existingItem = currentItems.find(item => item.id === product.id);
        
        if (existingItem) {
          return get().updateQuantity(product.id, existingItem.quantity + 1);
        }
        
        const newItems = [...currentItems, { ...product, quantity: 1 }];
        
        set({
          items: newItems,
          totalItems: get().totalItems + 1,
          totalPrice: get().totalPrice + product.price,
        });
      },
      
      removeItem: (id) => {
        const currentItems = get().items;
        const itemToRemove = currentItems.find(item => item.id === id);
        
        if (!itemToRemove) return;
        
        set({
          items: currentItems.filter(item => item.id !== id),
          totalItems: get().totalItems - itemToRemove.quantity,
          totalPrice: get().totalPrice - (itemToRemove.price * itemToRemove.quantity),
        });
      },
      
      updateQuantity: (id, quantity) => {
        const currentItems = get().items;
        const itemIndex = currentItems.findIndex(item => item.id === id);
        
        if (itemIndex === -1) return;
        
        const item = currentItems[itemIndex];
        const newItems = [...currentItems];
        
        const quantityDiff = quantity - item.quantity;
        newItems[itemIndex] = { ...item, quantity };
        
        set({
          items: newItems,
          totalItems: get().totalItems + quantityDiff,
          totalPrice: get().totalPrice + (item.price * quantityDiff),
        });
      },
      
      clearCart: () => {
        set({
          items: [],
          totalItems: 0,
          totalPrice: 0,
        });
      },
    }),
    {
      name: 'cart-storage',
    }
  )
);

// Usage in a component
// src/components/features/Cart/CartIcon.tsx
'use client';

import { useCartStore } from '@/store/cartStore';

export function CartIcon() {
  const totalItems = useCartStore(state => state.totalItems);
  
  return (
    <div className="relative">
      <ShoppingCartIcon className="h-6 w-6" />
      {totalItems > 0 && (
        <span className="absolute -top-2 -right-2 bg-red-500 text-white rounded-full h-5 w-5 flex items-center justify-center text-xs">
          {totalItems}
        </span>
      )}
    </div>
  );
}
```

### 3. Redux Toolkit for Complex State

For our most complex applications (like our customer dashboard), we use Redux Toolkit:

```bash
# Install Redux Toolkit
npm install @reduxjs/toolkit react-redux
```

```tsx
// src/store/features/authSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { login as loginApi, logout as logoutApi } from '@/lib/api';

export const login = createAsyncThunk(
  'auth/login',
  async ({ email, password }: { email: string; password: string }, { rejectWithValue }) => {
    try {
      const response = await loginApi(email, password);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

export const logout = createAsyncThunk(
  'auth/logout',
  async (_, { rejectWithValue }) => {
    try {
      await logoutApi();
      return null;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    token: null,
    isLoading: false,
    error: null,
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(login.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload || 'Login failed';
      })
      .addCase(logout.fulfilled, (state) => {
        state.user = null;
        state.token = null;
      });
  },
});

export const { clearError } = authSlice.actions;
export default authSlice.reducer;

// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './features/authSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// src/app/providers.tsx
'use client';

import { store } from '@/store';
import { Provider } from 'react-redux';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <Provider store={store}>
      {children}
    </Provider>
  );
}
```

## Performance Optimization

### Core Web Vitals Optimization

Based on our experience optimizing the customer portal, these techniques had the biggest impact:

1. **Image Optimization with next/image**:

```tsx
import Image from 'next/image';

export function ProductImage({ src, alt }: { src: string; alt: string }) {
  return (
    <div className="relative w-full aspect-square">
      <Image
        src={src}
        alt={alt}
        fill
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        className="object-cover rounded-md"
        priority
      />
    </div>
  );
}
```

2. **Lazy Loading Components**:

```tsx
// src/app/products/page.tsx
import { Suspense } from 'react';
import { ProductList } from '@/components/features/Products/ProductList';
import { ProductFilters } from '@/components/features/Products/ProductFilters';
import { ProductsSkeleton } from '@/components/features/Products/ProductsSkeleton';

export default function ProductsPage() {
  return (
    <div className="grid grid-cols-4 gap-6">
      <div className="col-span-1">
        <ProductFilters />
      </div>
      <div className="col-span-3">
        <Suspense fallback={<ProductsSkeleton />}>
          <ProductList />
        </Suspense>
      </div>
    </div>
  );
}
```

3. **Route Segment Config Options**:

```tsx
// src/app/products/page.tsx
export const dynamic = 'force-static';
export const revalidate = 3600; // Revalidate every hour

// src/app/dashboard/page.tsx
export const dynamic = 'force-dynamic';
```

4. **Bundle Size Optimization**:

```tsx
// Dynamic import with loading component
import dynamic from 'next/dynamic';
import { Suspense } from 'react';
import { ChartSkeleton } from '@/components/ui/ChartSkeleton';

const Chart = dynamic(() => import('@/components/features/Dashboard/Chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,
});

export function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<ChartSkeleton />}>
        <Chart />
      </Suspense>
    </div>
  );
}
```

### Memory Leak Prevention

After debugging a persistent memory leak in our admin dashboard, we've established these practices:

```tsx
// src/hooks/useInterval.ts
import { useEffect, useRef } from 'react';

export function useInterval(callback: () => void, delay: number | null) {
  const savedCallback = useRef<() => void>();

  // Remember the latest callback
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Set up the interval
  useEffect(() => {
    function tick() {
      savedCallback.current?.();
    }
    
    if (delay !== null) {
      const id = setInterval(tick, delay);
      return () => clearInterval(id);
    }
  }, [delay]);
}

// Usage in a component
import { useInterval } from '@/hooks/useInterval';
import { useState } from 'react';

export function LiveCounter() {
  const [count, setCount] = useState(0);
  
  useInterval(() => {
    setCount(prevCount => prevCount + 1);
  }, 1000);
  
  return <div>Count: {count}</div>;
}
```

## Environment Variables

Next.js has built-in support for environment variables:

- `.env.local` - Local overrides (not checked into git)
- `.env.development` - Development environment
- `.env.production` - Production environment

Create a `.env.local.example` file with dummy values:

```
# API
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_SITE_URL=http://localhost:3000

# Authentication
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-nextauth-secret

# Analytics
NEXT_PUBLIC_ANALYTICS_ID=your-analytics-id
```

Access environment variables in your code:

```tsx
// Public variables (available in browser)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Server-only variables (only available in Node.js context)
// This will only be available in Server Components, API routes, etc.
const secret = process.env.NEXTAUTH_SECRET;
```

## Deployment

We use Vercel for deploying Next.js applications:

```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy to production
vercel --prod
```

Configure Vercel with a `vercel.json` file:

```json
{
  "version": 2,
  "buildCommand": "npm run build",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["sfo1", "iad1"],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ]
}
```

## Authentication

We use NextAuth.js (now Auth.js) for authentication:

```bash
# Install NextAuth.js
npm install next-auth
```

Configure NextAuth.js:

```tsx
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { authenticateUser } from '@/lib/auth';

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null;
        }
        
        try {
          const user = await authenticateUser(
            credentials.email,
            credentials.password
          );
          
          return user;
        } catch (error) {
          return null;
        }
      },
    }),
  ],
  pages: {
    signIn: '/login',
    error: '/login',
  },
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id;
        session.user.role = token.role;
      }
      return session;
    },
  },
  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
});

export { handler as GET, handler as POST };

// Server Component usage
// src/app/admin/page.tsx
import { redirect } from 'next/navigation';
import { getServerSession } from 'next-auth/next';
import { authOptions } from '@/lib/auth';

export default async function AdminPage() {
  const session = await getServerSession(authOptions);
  
  // Check if user is authenticated and has admin role
  if (!session || session.user.role !== 'admin') {
    redirect('/login');
  }
  
  return (
    <div>
      <h1>Admin Dashboard</h1>
      {/* Admin content */}
    </div>
  );
}

// Client Component usage
// src/components/ui/LoginForm.tsx
'use client';

import { useState } from 'react';
import { signIn } from 'next-auth/react';
import { useRouter } from 'next/navigation';

export function LoginForm() {
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    setError('');
    
    try {
      const result = await signIn('credentials', {
        redirect: false,
        email,
        password,
      });
      
      if (result?.error) {
        setError('Invalid email or password');
      } else {
        router.push('/dashboard');
      }
    } catch (error) {
      setError('An error occurred. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      {error && <div className="text-red-500">{error}</div>}
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Log in'}
      </button>
    </form>
  );
}
```

## Accessibility

Our improved accessibility practices were implemented after user testing revealed significant issues with our previous dashboard:

1. **Semantic HTML**: Use proper semantic HTML elements like `<nav>`, `<main>`, `<section>`, `<article>`, etc.

2. **ARIA attributes**: Add ARIA roles and properties when necessary to enhance accessibility.

3. **Keyboard navigation**: Ensure all interactive elements are accessible via keyboard.

4. **Color contrast**: Maintain a minimum contrast ratio of 4.5:1 for normal text and 3:1 for large text.

5. **Focus management**: Provide visible focus indicators and maintain a logical tab order.

Example accessible component:

```tsx
// src/components/ui/Modal/Modal.tsx
'use client';

import { useRef, useEffect, ReactNode } from 'react';
import { createPortal } from 'react-dom';
import { XIcon } from 'lucide-react';
import { FocusTrap } from '@/components/ui/FocusTrap';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
}

export function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  
  // Close on escape key
  useEffect(() => {
    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === 'Escape') {
        onClose();
      }
    };
    
    if (isOpen) {
      window.addEventListener('keydown', handleKeyDown);
    }
    
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, [isOpen, onClose]);
  
  // Prevent scrolling when modal is open
  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = '';
    }
    
    return () => {
      document.body.style.overflow = '';
    };
  }, [isOpen]);
  
  if (!isOpen) {
    return null;
  }
  
  return createPortal(
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      role="presentation"
      onClick={(e) => {
        if (e.target === e.currentTarget) {
          onClose();
        }
      }}
    >
      <div className="fixed inset-0 bg-black bg-opacity-50" aria-hidden="true" />
      
      <FocusTrap>
        <div
          ref={modalRef}
          role="dialog"
          aria-modal="true"
          aria-labelledby="modal-title"
          className="bg-white rounded-lg shadow-xl z-10 p-6 max-w-md w-full mx-4"
        >
          <div className="flex justify-between items-center mb-4">
            <h2 id="modal-title" className="text-xl font-bold">
              {title}
            </h2>
            <button
              type="button"
              onClick={onClose}
              className="text-gray-500 hover:text-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 rounded-full p-1"
              aria-label="Close"
            >
              <XIcon className="h-6 w-6" />
            </button>
          </div>
          
          <div>{children}</div>
        </div>
      </FocusTrap>
    </div>,
    document.body
  );
}

// FocusTrap component
// src/components/ui/FocusTrap.tsx
'use client';

import { useRef, useEffect, ReactNode } from 'react';

export function FocusTrap({ children }: { children: ReactNode }) {
  const containerRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;
    
    // Find all focusable elements
    const focusableElements = container.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    const firstElement = focusableElements[0] as HTMLElement;
    const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;
    
    // Focus the first element
    firstElement?.focus();
    
    // Handle tab key navigation
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Tab') {
        if (e.shiftKey && document.activeElement === firstElement) {
          e.preventDefault();
          lastElement?.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          e.preventDefault();
          firstElement?.focus();
        }
      }
    };
    
    container.addEventListener('keydown', handleKeyDown);
    
    return () => {
      container.removeEventListener('keydown', handleKeyDown);
    };
  }, []);
  
  return <div ref={containerRef}>{children}</div>;
}
```

## Internationalization

For our global applications, we use next-intl for internationalization:

```bash
# Install next-intl
npm install next-intl
```

Configure next-intl:

```tsx
// src/middleware.ts
import createMiddleware from 'next-intl/middleware';

export default createMiddleware({
  locales: ['en', 'fr', 'de', 'es'],
  defaultLocale: 'en',
});

export const config = {
  matcher: ['/((?!api|_next|.*\\..*).*)'],
};

// src/messages/en.json
{
  "common": {
    "loading": "Loading...",
    "error": "An error occurred"
  },
  "home": {
    "title": "Welcome to our application",
    "description": "Get started by exploring our features"
  },
  "products": {
    "title": "Our Products",
    "filter": "Filter",
    "sort": "Sort",
    "noResults": "No products found"
  }
}

// src/app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl';
import { notFound } from 'next/navigation';

export function generateStaticParams() {
  return [{ locale: 'en' }, { locale: 'fr' }, { locale: 'de' }, { locale: 'es' }];
}

export default async function LocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  let messages;
  try {
    messages = (await import(`../../messages/${locale}.json`)).default;
  } catch (error) {
    notFound();
  }

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider locale={locale} messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}

// Usage in a Server Component
// src/app/[locale]/page.tsx
import { useTranslations } from 'next-intl';

export default function Home() {
  const t = useTranslations('home');
  
  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </div>
  );
}

// Usage in a Client Component
// src/components/ui/ErrorMessage.tsx
'use client';

import { useTranslations } from 'next-intl';

export function ErrorMessage({ error }: { error?: string }) {
  const t = useTranslations('common');
  
  return (
    <div className="text-red-500">
      {error || t('error')}
    </div>
  );
}
```

## Version Management

Node.js version updates are handled as follows:
- We maintain compatibility with the current Node.js LTS version (currently 20.x)
- New projects should always use the current standardized version
- Existing projects should update to new Node.js LTS versions within 6 months of their release
- All Node.js version upgrades require thorough testing of the application
- We follow the Node.js release schedule to plan our upgrades

## Case Studies

### Customer Portal Redesign

**Challenge**: Our customer portal was slow, with poor user experience and maintenance issues.

**Solution**:
1. Migrated from Pages Router to App Router
2. Implemented Server Components for critical paths
3. Used React Query for data fetching
4. Optimized images and assets
5. Improved component architecture

**Results**:
- 65% improvement in Core Web Vitals
- 40% reduction in bundle size
- 30% reduction in maintenance effort
- Improved user satisfaction scores

### Enterprise Dashboard Performance

**Challenge**: Our enterprise dashboard had memory leaks and performance issues with large datasets.

**Solution**:
1. Implemented virtualized lists for large data sets
2. Added pagination and filtering on the server side
3. Optimized state management with Redux Toolkit
4. Used memoization for expensive calculations
5. Implemented proper cleanup in useEffect hooks

**Results**:
- Eliminated memory leaks
- Reduced time-to-interactive by 75%
- Improved data loading performance by 60%
- Better user experience with large datasets