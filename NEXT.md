# Next.js Development Standards

This guide outlines our organization's standards for Next.js development, focusing on frontend applications.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Quick Start for Next.js Projects

```bash
# 1. Set up Node.js environment
brew install nvm
nvm install 18
nvm use 18

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
nvm install 18

# Set as default version
nvm alias default 18
```

### Project Initialization

For new projects:

```bash
# Create a new Next.js project with TypeScript
npx create-next-app@latest my-project --typescript

# Add Tailwind CSS
cd my-project
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

## Project Structure

Standard Next.js project structure:

```
frontend/
├── public/              # Static files
├── src/                 # Source code
│   ├── components/      # React components
│   │   ├── common/      # Shared components
│   │   └── [feature]/   # Feature-specific components
│   ├── pages/           # Next.js pages
│   │   ├── index.tsx    # Home page
│   │   ├── _app.tsx     # App component
│   │   ├── _document.tsx # Document component
│   │   └── api/         # API routes
│   ├── styles/          # CSS/SCSS styles
│   ├── lib/             # Utility functions
│   ├── hooks/           # Custom hooks
│   ├── context/         # React context
│   ├── types/           # TypeScript types
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
└── UserProfile/
    ├── index.tsx        # Main component (exports all)
    ├── UserProfile.tsx  # Component implementation
    ├── UserProfile.module.css # Scoped styles
    └── UserProfileContext.tsx # Component-specific context
```

## Code Quality Tools

### ESLint and Prettier

Install ESLint and Prettier:

```bash
npm install --save-dev eslint prettier eslint-config-prettier eslint-plugin-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser
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
    '^@/pages/(.*)$': '<rootDir>/src/pages/$1',
  },
  testEnvironment: 'jest-environment-jsdom',
};

module.exports = createJestConfig(customJestConfig);
```

Create a Jest setup file (`jest.setup.js`):

```javascript
import '@testing-library/jest-dom/extend-expect';
```

Add test scripts to `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

### Component Testing

Example component test:

```typescript
// src/components/Button/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick handler when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
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

## Next.js Best Practices

### Page Organization

- Use dynamic routes for similar pages
- Implement layout components for shared UI
- Use the App Router API for complex layouts

Example page structure:

```
pages/
├── index.tsx            # Home page
├── about.tsx            # About page
├── products/
│   ├── index.tsx        # Products list page
│   └── [id].tsx         # Product detail page
└── _app.tsx             # Custom App component
```

### API Routes

For backend functionality:

```typescript
// pages/api/users/index.ts
import type { NextApiRequest, NextApiResponse } from 'next';

type User = {
  id: number;
  name: string;
};

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<User[] | { message: string }>
) {
  if (req.method === 'GET') {
    // Return list of users
    res.status(200).json([{ id: 1, name: 'John Doe' }]);
  } else {
    // Method not allowed
    res.setHeader('Allow', ['GET']);
    res.status(405).json({ message: `Method ${req.method} Not Allowed` });
  }
}
```

### State Management

For simple state, use React hooks:

```typescript
import { useState, useEffect } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(prevCount => prevCount + 1);
  const decrement = () => setCount(prevCount => prevCount - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}
```

For complex state, use Context API or Redux:

```typescript
// src/context/AuthContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type User = {
  id: string;
  name: string;
  email: string;
};

type AuthContextType = {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
};

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  // Implement auth logic
  const login = async (email: string, password: string) => {
    // Call API, set user
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

### Performance Optimization

- Use `next/image` for optimized images
- Implement code splitting with dynamic imports
- Use server-side rendering (SSR) or static site generation (SSG) appropriately

```typescript
// pages/products/[id].tsx
import { GetServerSideProps } from 'next';
import { ProductDetail } from '@/components/ProductDetail';
import { fetchProduct } from '@/services/api';

export const getServerSideProps: GetServerSideProps = async (context) => {
  const id = context.params?.id as string;
  
  try {
    const product = await fetchProduct(id);
    return {
      props: { product },
    };
  } catch (error) {
    return {
      notFound: true,
    };
  }
};

export default function ProductPage({ product }) {
  return <ProductDetail product={product} />;
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

```typescript
// Public variables (available in browser)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Server-only variables (only available in Node.js context)
// This will only be available in getServerSideProps, API routes, etc.
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
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "env": {
    "NEXT_PUBLIC_API_URL": "@next_public_api_url"
  }
}
```