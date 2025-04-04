# Express Development Standards

This guide outlines our organization's standards for Express.js development, focusing on backend services and APIs.

> 📌 **Return to**: [Main Development Guide](README.md)

## Quick Start for Express Projects

```bash
# 1. Set up Node.js environment
brew install nvm
nvm install 20
nvm use 20

# 2. Clone repository
git clone git@github.com:organization/project-name-service.git
cd project-name-service

# 3. Install dependencies
npm install

# 4. Run development server
npm run start:dev

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
# Create a project directory
mkdir my-express-api
cd my-express-api

# Initialize package.json
npm init -y

# Install TypeScript
npm install --save-dev typescript ts-node @types/node

# Initialize TypeScript configuration
npx tsc --init

# Install Express and types
npm install express
npm install --save-dev @types/express
```

After trying several scaffolding tools, we've found this manual approach gives us the most control and avoids unnecessary dependencies.

## Project Structure

Standard Express project structure:

```
express-api/
├── src/                 # Source code
│   ├── config/          # Configuration
│   │   ├── index.ts     # Configuration exports
│   │   └── database.ts  # Database configuration
│   ├── controllers/     # Route controllers
│   │   ├── index.ts     # Controller exports
│   │   └── users.ts     # User controller
│   ├── middleware/      # Express middleware
│   │   ├── index.ts     # Middleware exports
│   │   ├── auth.ts      # Authentication middleware
│   │   └── error.ts     # Error handling middleware
│   ├── models/          # Data models
│   │   ├── index.ts     # Model exports
│   │   └── user.ts      # User model
│   ├── routes/          # Express routes
│   │   ├── index.ts     # Route exports
│   │   └── users.ts     # User routes
│   ├── services/        # Business logic
│   │   ├── index.ts     # Service exports
│   │   └── users.ts     # User service
│   ├── utils/           # Utility functions
│   │   ├── index.ts     # Utility exports
│   │   └── logger.ts    # Logging utility
│   ├── types/           # TypeScript types
│   │   └── index.ts     # Type definitions
│   ├── app.ts           # Express app setup
│   └── server.ts        # Server entry point
├── tests/               # Test directory
│   ├── unit/            # Unit tests
│   └── integration/     # Integration tests
├── .env.example         # Example environment variables
├── package.json         # Project dependencies
├── tsconfig.json        # TypeScript configuration
└── README.md            # Project documentation
```

This structure evolved from our experience building multiple production APIs. The clean separation of concerns was particularly helpful during the refactoring of our payment processing service, allowing multiple teams to work on different components simultaneously.

## Configuration Files

### TypeScript Configuration (`tsconfig.json`)

```json
{
  "compilerOptions": {
    "target": "es2022",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

### Package Configuration (`package.json`)

```json
{
  "name": "express-api",
  "version": "1.0.0",
  "description": "Express API with TypeScript",
  "main": "dist/server.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "start:dev": "ts-node-dev --respawn src/server.ts",
    "lint": "eslint . --ext .ts",
    "lint:fix": "eslint . --ext .ts --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "migrate": "knex migrate:latest --knexfile src/config/knexfile.ts",
    "migrate:make": "knex migrate:make --knexfile src/config/knexfile.ts",
    "migrate:rollback": "knex migrate:rollback --knexfile src/config/knexfile.ts"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express": "^4.18.2",
    "helmet": "^7.0.0",
    "joi": "^17.9.2",
    "knex": "^2.5.1",
    "morgan": "^1.10.0",
    "pg": "^8.11.1",
    "winston": "^3.10.0"
  },
  "devDependencies": {
    "@types/cors": "^2.8.13",
    "@types/express": "^4.17.17",
    "@types/jest": "^29.5.3",
    "@types/morgan": "^1.9.4",
    "@types/node": "^20.4.5",
    "@types/supertest": "^2.0.12",
    "@typescript-eslint/eslint-plugin": "^6.2.0",
    "@typescript-eslint/parser": "^6.2.0",
    "eslint": "^8.46.0",
    "eslint-config-prettier": "^8.9.0",
    "eslint-plugin-prettier": "^5.0.0",
    "jest": "^29.6.2",
    "prettier": "^3.0.0",
    "supertest": "^6.3.3",
    "ts-jest": "^29.1.1",
    "ts-node-dev": "^2.0.0",
    "typescript": "^5.1.6"
  }
}
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
  parser: '@typescript-eslint/parser',
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error',
    '@typescript-eslint/explicit-function-return-type': ['error', {
      allowExpressions: true,
      allowTypedFunctionExpressions: true,
    }],
    '@typescript-eslint/no-unused-vars': ['error', { 
      argsIgnorePattern: '^_',
      varsIgnorePattern: '^_',
    }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'no-return-await': 'error',
    'no-throw-literal': 'error',
    'prefer-promise-reject-errors': 'error',
  },
  env: {
    node: true,
    jest: true,
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
    "*.ts": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

## Express Application Setup

### Basic Application Setup

```typescript
// src/app.ts
import express, { Application } from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import routes from './routes';
import errorMiddleware from './middleware/error';
import config from './config';

// Initialize express app
const app: Application = express();

// Apply middleware
app.use(helmet()); // Security headers
app.use(cors({
  origin: config.corsOrigins,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})); // CORS support
app.use(morgan(config.nodeEnv === 'development' ? 'dev' : 'combined')); // Logging
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Apply routes
app.use('/api', routes);

// Apply error handling middleware
app.use(errorMiddleware);

export default app;
```

### Server Setup

```typescript
// src/server.ts
import app from './app';
import config from './config';
import { logger } from './utils';

const PORT = config.port || 3000;

// Handle uncaught exceptions
process.on('uncaughtException', (error) => {
  logger.error('Uncaught Exception:', error);
  // Perform graceful shutdown
  process.exit(1);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
  logger.error('Unhandled Rejection at:', promise, 'reason:', reason);
  // No need to exit here, Node.js will warn about unhandled promise rejections
});

// Handle termination signals
process.on('SIGTERM', () => {
  logger.info('SIGTERM received, shutting down gracefully');
  // Perform cleanup
  process.exit(0);
});

// Start the server
const server = app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT} in ${config.nodeEnv} mode`);
});

// Enable graceful shutdown
process.on('SIGINT', () => {
  logger.info('SIGINT received, shutting down gracefully');
  server.close(() => {
    logger.info('Server closed');
    process.exit(0);
  });
});

export default server;
```

### Environment Configuration

```typescript
// src/config/index.ts
import dotenv from 'dotenv';
import path from 'path';
import Joi from 'joi';

// Load environment variables
const envFile = process.env.NODE_ENV === 'test' 
  ? '.env.test' 
  : process.env.NODE_ENV === 'production' 
    ? '.env.production' 
    : '.env.development';

dotenv.config({ path: path.resolve(process.cwd(), envFile) });

// Fallback to .env.local for local development
if (process.env.NODE_ENV === 'development') {
  dotenv.config({ path: path.resolve(process.cwd(), '.env.local') });
}

// Validation schema
const envSchema = Joi.object({
  NODE_ENV: Joi.string().valid('development', 'production', 'test').default('development'),
  PORT: Joi.number().default(3000),
  LOG_LEVEL: Joi.string().valid('error', 'warn', 'info', 'debug').default('info'),
  DATABASE_URL: Joi.string().required(),
  JWT_SECRET: Joi.string().required(),
  JWT_EXPIRES_IN: Joi.string().default('1d'),
  CORS_ORIGINS: Joi.string().default('*'),
}).unknown();

// Validate env vars
const { error, value } = envSchema.validate(process.env);
if (error) {
  throw new Error(`Environment validation error: ${error.message}`);
}

// Export configuration
export default {
  nodeEnv: value.NODE_ENV as string,
  port: parseInt(value.PORT as string, 10),
  logLevel: value.LOG_LEVEL as string,
  databaseUrl: value.DATABASE_URL as string,
  jwtSecret: value.JWT_SECRET as string,
  jwtExpiresIn: value.JWT_EXPIRES_IN as string,
  corsOrigins: value.CORS_ORIGINS.split(',') as string[],
};
```

## Route Organization

### Modular Route Structure

```typescript
// src/routes/index.ts
import { Router } from 'express';
import userRoutes from './users';
import authRoutes from './auth';
import productRoutes from './products';

const router = Router();

router.use('/users', userRoutes);
router.use('/auth', authRoutes);
router.use('/products', productRoutes);

export default router;
```

### Route Implementation

```typescript
// src/routes/users.ts
import { Router } from 'express';
import { UserController } from '../controllers';
import { authMiddleware, validateSchema } from '../middleware';
import { userSchema } from '../validations/user.schema';

const router = Router();

// Get all users (protected)
router.get('/', authMiddleware, UserController.getAll);

// Get user by ID (protected)
router.get('/:id', authMiddleware, UserController.getById);

// Create new user
router.post(
  '/', 
  validateSchema(userSchema.create), 
  UserController.create
);

// Update user (protected)
router.put(
  '/:id', 
  [authMiddleware, validateSchema(userSchema.update)],
  UserController.update
);

// Delete user (protected)
router.delete('/:id', authMiddleware, UserController.delete);

export default router;
```

### Validation Middleware

```typescript
// src/middleware/validateSchema.ts
import { Request, Response, NextFunction } from 'express';
import { Schema } from 'joi';

export const validateSchema = (schema: Schema) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const { error } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true,
    });

    if (error) {
      res.status(400).json({
        status: 'error',
        message: 'Validation Error',
        errors: error.details.map((detail) => ({
          field: detail.path.join('.'),
          message: detail.message,
        })),
      });
      return;
    }

    next();
  };
};

// src/validations/user.schema.ts
import Joi from 'joi';

export const userSchema = {
  create: Joi.object({
    name: Joi.string().required().min(2).max(100),
    email: Joi.string().email().required(),
    password: Joi.string().required().min(8),
    role: Joi.string().valid('user', 'admin').default('user'),
  }),
  
  update: Joi.object({
    name: Joi.string().min(2).max(100),
    email: Joi.string().email(),
    password: Joi.string().min(8),
    role: Joi.string().valid('user', 'admin'),
  }),
};
```

## Controller Pattern

### Controller Implementation

```typescript
// src/controllers/users.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '../services';
import { logger } from '../utils';
import { AppError, NotFoundError } from '../utils/errors';

export class UserController {
  // Get all users
  static async getAll(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const users = await UserService.getAll();
      res.status(200).json({
        status: 'success',
        data: { users },
      });
    } catch (error) {
      logger.error('Error in UserController.getAll:', error);
      next(error);
    }
  }

  // Get user by ID
  static async getById(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const { id } = req.params;
      const user = await UserService.getById(id);
      
      if (!user) {
        throw new NotFoundError(`User with ID ${id} not found`);
      }
      
      res.status(200).json({
        status: 'success',
        data: { user },
      });
    } catch (error) {
      logger.error(`Error in UserController.getById for id ${req.params.id}:`, error);
      next(error);
    }
  }

  // Create user
  static async create(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const userData = req.body;
      const newUser = await UserService.create(userData);
      
      res.status(201).json({
        status: 'success',
        data: { user: newUser },
      });
    } catch (error) {
      logger.error('Error in UserController.create:', error);
      next(error);
    }
  }

  // Update user
  static async update(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const { id } = req.params;
      const userData = req.body;
      const updatedUser = await UserService.update(id, userData);
      
      if (!updatedUser) {
        throw new NotFoundError(`User with ID ${id} not found`);
      }
      
      res.status(200).json({
        status: 'success',
        data: { user: updatedUser },
      });
    } catch (error) {
      logger.error(`Error in UserController.update for id ${req.params.id}:`, error);
      next(error);
    }
  }

  // Delete user
  static async delete(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const { id } = req.params;
      const result = await UserService.delete(id);
      
      if (!result) {
        throw new NotFoundError(`User with ID ${id} not found`);
      }
      
      res.status(204).send();
    } catch (error) {
      logger.error(`Error in UserController.delete for id ${req.params.id}:`, error);
      next(error);
    }
  }
}
```

## Service Layer Pattern

The service layer encapsulates business logic and database operations, which has helped us maintain a clean separation of concerns across our API services.

```typescript
// src/services/users.ts
import { User, UserInput, UserUpdate } from '../types';
import { db } from '../config/database';
import { hashPassword } from '../utils/auth';

export class UserService {
  // Get all users
  static async getAll(): Promise<User[]> {
    return db('users')
      .select('id', 'name', 'email', 'role', 'created_at', 'updated_at');
  }
  
  // Get user by ID
  static async getById(id: string): Promise<User | undefined> {
    return db('users')
      .select('id', 'name', 'email', 'role', 'created_at', 'updated_at')
      .where({ id })
      .first();
  }
  
  // Create user
  static async create(userData: UserInput): Promise<User> {
    // Hash password
    const hashedPassword = await hashPassword(userData.password);
    
    // Insert into database
    const [id] = await db('users').insert({
      name: userData.name,
      email: userData.email,
      password: hashedPassword,
      role: userData.role || 'user',
    }).returning('id');
    
    // Return created user (without password)
    return this.getById(id);
  }
  
  // Update user
  static async update(id: string, userData: UserUpdate): Promise<User | undefined> {
    // Prepare update data
    const updateData: Partial<UserUpdate> = { ...userData };
    
    // Hash password if provided
    if (updateData.password) {
      updateData.password = await hashPassword(updateData.password);
    }
    
    // Update user
    await db('users')
      .where({ id })
      .update({
        ...updateData,
        updated_at: db.fn.now(),
      });
    
    // Return updated user
    return this.getById(id);
  }
  
  // Delete user
  static async delete(id: string): Promise<boolean> {
    const count = await db('users')
      .where({ id })
      .delete();
    
    return count > 0;
  }
}
```

## Database Integration

### Knex.js Configuration

We use Knex.js for database operations, which gives us a clean query builder API with migration support:

```typescript
// src/config/knexfile.ts
import { knex } from 'knex';
import { config } from 'dotenv';
import path from 'path';

// Load environment variables
config();

const knexConfig = {
  development: {
    client: 'pg',
    connection: process.env.DATABASE_URL,
    migrations: {
      directory: path.join(__dirname, '../database/migrations'),
      tableName: 'knex_migrations',
    },
    seeds: {
      directory: path.join(__dirname, '../database/seeds'),
    },
    pool: {
      min: 2,
      max: 10,
    },
  },
  test: {
    client: 'pg',
    connection: process.env.TEST_DATABASE_URL,
    migrations: {
      directory: path.join(__dirname, '../database/migrations'),
      tableName: 'knex_migrations',
    },
    seeds: {
      directory: path.join(__dirname, '../database/seeds'),
    },
  },
  production: {
    client: 'pg',
    connection: process.env.DATABASE_URL,
    migrations: {
      directory: path.join(__dirname, '../database/migrations'),
      tableName: 'knex_migrations',
    },
    pool: {
      min: 2,
      max: 20,
    },
  },
};

export default knexConfig;

// src/config/database.ts
import knex from 'knex';
import knexConfig from './knexfile';
import config from './index';

// Determine which configuration to use
const environment = config.nodeEnv === 'test' ? 'test' : 
                    config.nodeEnv === 'production' ? 'production' : 
                    'development';

// Create database connection
export const db = knex(knexConfig[environment]);
```

### Migrations

```typescript
// src/database/migrations/20230701000000_create_users_table.ts
import { Knex } from 'knex';

export async function up(knex: Knex): Promise<void> {
  return knex.schema.createTable('users', (table) => {
    table.uuid('id').defaultTo(knex.raw('gen_random_uuid()')).primary();
    table.string('name').notNullable();
    table.string('email').notNullable().unique();
    table.string('password').notNullable();
    table.enum('role', ['user', 'admin']).defaultTo('user');
    table.timestamps(true, true);
  });
}

export async function down(knex: Knex): Promise<void> {
  return knex.schema.dropTable('users');
}
```

## Error Handling

### Error Handling Middleware

```typescript
// src/middleware/error.ts
import { Request, Response, NextFunction } from 'express';
import { logger } from '../utils';
import { AppError } from '../utils/errors';

export default function errorMiddleware(
  err: Error,
  req: Request,
  res: Response,
  _next: NextFunction
): void {
  // Default status code and message
  let statusCode = 500;
  let message = 'Internal Server Error';
  
  // Handle known errors
  if (err instanceof AppError) {
    statusCode = err.statusCode;
    message = err.message;
  }
  
  // Log the error
  logger.error(`[${req.method}] ${req.path} >> StatusCode: ${statusCode}, Message: ${message}`, {
    error: err.stack,
    requestId: req.headers['x-request-id'] || '',
    userId: (req.user as any)?.id || 'unknown',
  });
  
  // Send response
  res.status(statusCode).json({
    status: 'error',
    statusCode,
    message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
}
```

### Custom Error Classes

```typescript
// src/utils/errors.ts
export class AppError extends Error {
  statusCode: number;
  
  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
  }
}

export class BadRequestError extends AppError {
  constructor(message = 'Bad request') {
    super(message, 400);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403);
  }
}

export class ConflictError extends AppError {
  constructor(message = 'Conflict') {
    super(message, 409);
  }
}

export class ValidationError extends AppError {
  errors: Record<string, string>;
  
  constructor(message = 'Validation error', errors: Record<string, string> = {}) {
    super(message, 422);
    this.errors = errors;
  }
}
```

## Authentication

### JWT Authentication

```typescript
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { UnauthorizedError } from '../utils/errors';
import config from '../config';
import { UserService } from '../services';

// Extend Express Request type
declare global {
  namespace Express {
    interface Request {
      user?: {
        id: string;
        role: string;
      };
    }
  }
}

export async function authMiddleware(
  req: Request,
  _res: Response,
  next: NextFunction
): Promise<void> {
  try {
    // Get token from header
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      throw new UnauthorizedError('No token provided');
    }
    
    const token = authHeader.split(' ')[1];
    
    // Verify token
    const decoded = jwt.verify(token, config.jwtSecret) as {
      id: string;
      role: string;
      iat: number;
      exp: number;
    };
    
    // Check if user exists
    const user = await UserService.getById(decoded.id);
    if (!user) {
      throw new UnauthorizedError('User no longer exists');
    }
    
    // Add user to request object
    req.user = {
      id: decoded.id,
      role: decoded.role,
    };
    
    next();
  } catch (error) {
    if (error instanceof jwt.JsonWebTokenError) {
      next(new UnauthorizedError('Invalid token'));
    } else {
      next(error);
    }
  }
}

// Role-based access control middleware
export function authorize(roles: string[]) {
  return (req: Request, _res: Response, next: NextFunction): void => {
    if (!req.user) {
      throw new UnauthorizedError('User not authenticated');
    }
    
    if (!roles.includes(req.user.role)) {
      throw new UnauthorizedError('Insufficient permissions');
    }
    
    next();
  };
}
```

## Testing

We use `jest` for testing:

```bash
# Install testing libraries
npm install --save-dev jest ts-jest @types/jest supertest @types/supertest
```

Create a Jest configuration file (`jest.config.js`):

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: ['**/*.test.ts'],
  collectCoverageFrom: ['src/**/*.ts', '!src/**/*.d.ts'],
  coverageDirectory: 'coverage',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};
```

### Unit Tests

```typescript
// tests/unit/services/users.test.ts
import { UserService } from '../../../src/services';
import { NotFoundError } from '../../../src/utils/errors';
import { db } from '../../../src/config/database';

// Mock the database
jest.mock('../../../src/config/database', () => ({
  db: {
    select: jest.fn().mockReturnThis(),
    where: jest.fn().mockReturnThis(),
    first: jest.fn(),
    insert: jest.fn().mockReturnThis(),
    returning: jest.fn().mockReturnThis(),
    update: jest.fn().mockReturnThis(),
    delete: jest.fn(),
  },
}));

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getById', () => {
    it('should return a user if found', async () => {
      const mockUser = { id: '1', name: 'Test User' };
      (db.first as jest.Mock).mockResolvedValue(mockUser);

      const result = await UserService.getById('1');
      
      expect(db.select).toHaveBeenCalled();
      expect(db.where).toHaveBeenCalledWith({ id: '1' });
      expect(result).toEqual(mockUser);
    });

    it('should return undefined if user not found', async () => {
      (db.first as jest.Mock).mockResolvedValue(undefined);

      const result = await UserService.getById('1');
      
      expect(db.select).toHaveBeenCalled();
      expect(db.where).toHaveBeenCalledWith({ id: '1' });
      expect(result).toBeUndefined();
    });
  });
});
```

### Integration Tests

```typescript
// tests/integration/routes/users.test.ts
import request from 'supertest';
import app from '../../../src/app';
import { db } from '../../../src/config/database';
import { generateToken } from '../../../src/utils/auth';

describe('User Routes', () => {
  // Test user
  const testUser = {
    id: '1',
    name: 'Test User',
    email: 'test@example.com',
    role: 'admin',
  };
  
  // Generate test token
  const token = generateToken(testUser);
  
  beforeAll(async () => {
    // Set up test database
    await db.migrate.latest();
    
    // Seed test data
    await db('users').insert({
      id: testUser.id,
      name: testUser.name,
      email: testUser.email,
      password: 'hashed_password',
      role: testUser.role,
    });
  });
  
  afterAll(async () => {
    // Clean up test database
    await db.migrate.rollback();
    await db.destroy();
  });

  describe('GET /api/users', () => {
    it('should return all users', async () => {
      const response = await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${token}`);
      
      expect(response.status).toBe(200);
      expect(response.body.status).toBe('success');
      expect(Array.isArray(response.body.data.users)).toBe(true);
      expect(response.body.data.users.length).toBeGreaterThan(0);
    });
    
    it('should require authentication', async () => {
      const response = await request(app).get('/api/users');
      
      expect(response.status).toBe(401);
    });
  });
});
```

## Logging

Use Winston for logging:

```typescript
// src/utils/logger.ts
import winston from 'winston';
import config from '../config';

const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json()
);

const consoleFormat = winston.format.combine(
  winston.format.colorize(),
  winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
  winston.format.printf(({ level, message, timestamp, ...metadata }) => {
    let metaStr = '';
    if (Object.keys(metadata).length > 0 && metadata.error !== undefined) {
      metaStr = `: ${JSON.stringify(metadata)}`;
    }
    return `[${timestamp}] ${level}: ${message}${metaStr}`;
  })
);

const logger = winston.createLogger({
  level: config.logLevel,
  format: logFormat,
  defaultMeta: { service: 'api-service' },
  transports: [
    // Console transport in development
    new winston.transports.Console({
      format: config.nodeEnv === 'development' ? consoleFormat : logFormat,
    }),
  ],
});

// Add file transports in production
if (config.nodeEnv === 'production') {
  logger.add(
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error',
      maxsize: 10485760, // 10MB
      maxFiles: 5,
    })
  );
  logger.add(
    new winston.transports.File({ 
      filename: 'logs/combined.log',
      maxsize: 10485760, // 10MB
      maxFiles: 5,
    })
  );
}

export default logger;
```

## API Design Patterns

### Response Format Standardization

```typescript
interface SuccessResponse<T> {
  status: 'success';
  data: T;
  meta?: {
    pagination?: {
      page: number;
      limit: number;
      total: number;
      totalPages: number;
    };
  };
}

interface ErrorResponse {
  status: 'error';
  statusCode: number;
  message: string;
  errors?: Record<string, string> | Array<{ field: string; message: string }>;
}

// Example usage
export const sendSuccess = <T>(
  res: Response,
  data: T,
  statusCode = 200,
  meta?: SuccessResponse<T>['meta']
): void => {
  res.status(statusCode).json({
    status: 'success',
    data,
    ...(meta && { meta }),
  });
};

export const sendError = (
  res: Response,
  message: string,
  statusCode = 500,
  errors?: ErrorResponse['errors']
): void => {
  res.status(statusCode).json({
    status: 'error',
    statusCode,
    message,
    ...(errors && { errors }),
  });
};
```

### Pagination Middleware

```typescript
// src/middleware/pagination.ts
import { Request, Response, NextFunction } from 'express';

export interface PaginationOptions {
  page: number;
  limit: number;
  skip: number;
}

declare global {
  namespace Express {
    interface Request {
      pagination: PaginationOptions;
    }
  }
}

export function paginationMiddleware(
  req: Request,
  _res: Response,
  next: NextFunction
): void {
  const page = parseInt(req.query.page as string) || 1;
  const limit = parseInt(req.query.limit as string) || 10;
  
  req.pagination = {
    page,
    limit,
    skip: (page - 1) * limit,
  };
  
  next();
}
```

### Filtering and Sorting

```typescript
// src/middleware/query.ts
import { Request, Response, NextFunction } from 'express';

export interface QueryOptions {
  filters: Record<string, any>;
  sort: {
    field: string;
    direction: 'asc' | 'desc';
  };
}

declare global {
  namespace Express {
    interface Request {
      queryOptions: QueryOptions;
    }
  }
}

export function queryMiddleware(
  allowedFilters: string[] = [],
  allowedSortFields: string[] = []
) {
  return (req: Request, _res: Response, next: NextFunction): void => {
    // Parse filters
    const filters: Record<string, any> = {};
    
    for (const key of allowedFilters) {
      if (req.query[key] !== undefined) {
        filters[key] = req.query[key];
      }
    }
    
    // Parse sort
    let sort = {
      field: allowedSortFields[0] || 'created_at',
      direction: 'desc' as const,
    };
    
    if (req.query.sort) {
      const sortParam = req.query.sort as string;
      const direction = sortParam.startsWith('-') ? 'desc' : 'asc';
      const field = direction === 'desc' ? sortParam.substring(1) : sortParam;
      
      if (allowedSortFields.includes(field)) {
        sort = { field, direction };
      }
    }
    
    req.queryOptions = { filters, sort };
    
    next();
  };
}
```

## Performance Optimization

### Connection Pooling

```typescript
// src/config/knexfile.ts (production section)
production: {
  client: 'pg',
  connection: process.env.DATABASE_URL,
  migrations: {
    directory: path.join(__dirname, '../database/migrations'),
    tableName: 'knex_migrations',
  },
  pool: {
    min: 2,
    max: 20,
    // Acquire connection timeout in ms
    acquireTimeoutMillis: 60000,
    // Idle timeout in ms
    idleTimeoutMillis: 30000,
    // Prevent overload during spikes
    createTimeoutMillis: 30000,
    // Check connection before using it
    afterCreate: (conn: any, done: Function) => {
      conn.query('SELECT 1', (err: Error) => {
        done(err, conn);
      });
    }
  },
}
```

### Caching

```typescript
// src/middleware/cache.ts
import { Request, Response, NextFunction } from 'express';
import NodeCache from 'node-cache';

// Create cache instance
const cache = new NodeCache({
  stdTTL: 60, // Default TTL in seconds
  checkperiod: 120, // Automatic delete check interval
});

// Cache middleware factory
export function cacheMiddleware(ttl = 60) {
  return (req: Request, res: Response, next: NextFunction): void => {
    // Skip cache for non-GET requests
    if (req.method !== 'GET') {
      return next();
    }
    
    // Generate cache key from request path and query
    const key = `${req.originalUrl}`;
    
    // Check if data is in cache
    const cachedData = cache.get(key);
    if (cachedData) {
      return res.status(200).json(cachedData);
    }
    
    // Cache miss, continue to route handler
    // Capture the original response.json method
    const originalJson = res.json;
    
    // Override response.json method
    res.json = function(data) {
      // Store in cache if status is success (2xx)
      if (res.statusCode >= 200 && res.statusCode < 300) {
        cache.set(key, data, ttl);
      }
      
      // Call the original method
      return originalJson.call(this, data);
    };
    
    next();
  };
}

// Clear cache for specific paths
export function clearCache(path: string | RegExp): void {
  const keys = cache.keys();
  const matchingKeys = typeof path === 'string'
    ? keys.filter(key => key.includes(path))
    : keys.filter(key => path.test(key));
  
  cache.del(matchingKeys);
}
```

### Rate Limiting

```typescript
// src/middleware/rateLimit.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';
import config from '../config';

// Create Redis client if Redis URL is provided
const redisClient = config.redisUrl
  ? new Redis(config.redisUrl)
  : null;

// Create rate limit middleware
export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
  legacyHeaders: false, // Disable the `X-RateLimit-*` headers
  message: {
    status: 'error',
    statusCode: 429,
    message: 'Too many requests, please try again later.',
  },
  // Use Redis store if available, otherwise use memory
  ...(redisClient && {
    store: new RedisStore({
      sendCommand: (...args: string[]) => redisClient.call(...args),
    }),
  }),
});

// Create more restrictive limiter for auth routes
export const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // limit each IP to 10 login attempts per hour
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    status: 'error',
    statusCode: 429,
    message: 'Too many login attempts, please try again later.',
  },
  ...(redisClient && {
    store: new RedisStore({
      sendCommand: (...args: string[]) => redisClient.call(...args),
      prefix: 'rl:auth:',
    }),
  }),
});
```

## Security Best Practices

### Secure Headers

```typescript
// src/app.ts
import helmet from 'helmet';

// Apply Helmet with custom configuration
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", 'data:'],
      },
    },
    referrerPolicy: { policy: 'same-origin' },
    hsts: {
      maxAge: 15552000, // 180 days
      includeSubDomains: true,
      preload: true,
    },
  })
);
```

### Security Middleware

```typescript
// src/middleware/security.ts
import { Request, Response, NextFunction } from 'express';
import crypto from 'crypto';

// CSRF Protection
export function csrfProtection(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  // Skip for GET, HEAD, OPTIONS requests
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
    return next();
  }
  
  // Check CSRF token
  const csrfToken = req.headers['x-csrf-token'] || req.body._csrf;
  const sessionToken = req.session?.csrfToken;
  
  if (!csrfToken || !sessionToken || csrfToken !== sessionToken) {
    return res.status(403).json({
      status: 'error',
      statusCode: 403,
      message: 'Invalid CSRF token',
    });
  }
  
  next();
}

// Request ID middleware
export function requestId(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const id = crypto.randomUUID();
  req.headers['x-request-id'] = id;
  res.setHeader('X-Request-ID', id);
  next();
}

// Add request timestamp
export function requestTimestamp(
  req: Request,
  _res: Response,
  next: NextFunction
): void {
  req.requestTime = new Date();
  next();
}
```

### Input Sanitization

```typescript
// src/middleware/sanitize.ts
import { Request, Response, NextFunction } from 'express';
import sanitizeHtml from 'sanitize-html';

// Sanitize HTML in request body
export function sanitizeBody(
  req: Request,
  _res: Response,
  next: NextFunction
): void {
  if (req.body) {
    req.body = sanitizeObject(req.body);
  }
  next();
}

// Recursively sanitize object properties
function sanitizeObject(obj: any): any {
  if (obj === null || obj === undefined) {
    return obj;
  }
  
  if (typeof obj === 'string') {
    return sanitizeHtml(obj, {
      allowedTags: [],
      allowedAttributes: {},
    });
  }
  
  if (Array.isArray(obj)) {
    return obj.map(item => sanitizeObject(item));
  }
  
  if (typeof obj === 'object') {
    const result: Record<string, any> = {};
    for (const [key, value] of Object.entries(obj)) {
      result[key] = sanitizeObject(value);
    }
    return result;
  }
  
  return obj;
}
```

## Advanced Patterns

### Repository Pattern

```typescript
// src/repositories/base.repository.ts
import { db } from '../config/database';

export abstract class BaseRepository<T> {
  protected tableName: string;
  
  constructor(tableName: string) {
    this.tableName = tableName;
  }
  
  async findAll(options?: {
    select?: string[];
    where?: Record<string, any>;
    orderBy?: string;
    direction?: 'asc' | 'desc';
    limit?: number;
    offset?: number;
  }): Promise<T[]> {
    const query = db(this.tableName);
    
    if (options?.select) {
      query.select(options.select);
    }
    
    if (options?.where) {
      query.where(options.where);
    }
    
    if (options?.orderBy) {
      query.orderBy(options.orderBy, options.direction || 'asc');
    }
    
    if (options?.limit) {
      query.limit(options.limit);
    }
    
    if (options?.offset) {
      query.offset(options.offset);
    }
    
    return query;
  }
  
  async findById(id: string, select?: string[]): Promise<T | undefined> {
    const query = db(this.tableName).where({ id });
    
    if (select) {
      query.select(select);
    }
    
    return query.first();
  }
  
  async create(data: Partial<T>): Promise<string> {
    const [id] = await db(this.tableName)
      .insert(data)
      .returning('id');
    
    return id;
  }
  
  async update(id: string, data: Partial<T>): Promise<boolean> {
    const result = await db(this.tableName)
      .where({ id })
      .update(data);
    
    return result > 0;
  }
  
  async delete(id: string): Promise<boolean> {
    const result = await db(this.tableName)
      .where({ id })
      .delete();
    
    return result > 0;
  }
  
  async count(where?: Record<string, any>): Promise<number> {
    const query = db(this.tableName);
    
    if (where) {
      query.where(where);
    }
    
    const [{ count }] = await query.count('id as count');
    
    return parseInt(count as string, 10);
  }
}

// src/repositories/user.repository.ts
import { BaseRepository } from './base.repository';
import { User, UserInput } from '../types';

export class UserRepository extends BaseRepository<User> {
  constructor() {
    super('users');
  }
  
  async findByEmail(email: string): Promise<User | undefined> {
    return db(this.tableName)
      .where({ email })
      .first();
  }
  
  // Custom methods for user-specific operations
  async updateLastLogin(id: string): Promise<void> {
    await db(this.tableName)
      .where({ id })
      .update({
        last_login_at: db.fn.now(),
      });
  }
}
```

### Dependency Injection

```typescript
// src/types/container.ts
export interface Container {
  get<T>(id: string): T;
  register<T>(id: string, value: T): void;
}

// src/utils/container.ts
import { Container } from '../types/container';

class DIContainer implements Container {
  private readonly services: Map<string, any> = new Map();
  
  get<T>(id: string): T {
    if (!this.services.has(id)) {
      throw new Error(`Service ${id} not found in container`);
    }
    
    return this.services.get(id) as T;
  }
  
  register<T>(id: string, value: T): void {
    this.services.set(id, value);
  }
}

// Create singleton container
export const container = new DIContainer();

// Register services
import { UserRepository } from '../repositories/user.repository';
import { UserService } from '../services/user.service';

// Register repositories
container.register('UserRepository', new UserRepository());

// Register services with dependencies
container.register(
  'UserService',
  new UserService(container.get('UserRepository'))
);

// Usage in controllers
import { container } from '../utils/container';

export class UserController {
  private userService = container.get<UserService>('UserService');
  
  async getAll(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const users = await this.userService.getAll();
      res.status(200).json({
        status: 'success',
        data: { users },
      });
    } catch (error) {
      next(error);
    }
  }
}
```

### Job Queue Processing

```typescript
// src/queues/index.ts
import Bull from 'bull';
import config from '../config';
import { logger } from '../utils';

// Create queue instances
export const emailQueue = new Bull('email', {
  redis: config.redisUrl,
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 1000,
    },
    removeOnComplete: true,
    removeOnFail: false,
  },
});

// Process email queue
emailQueue.process(async (job) => {
  const { to, subject, text, html } = job.data;
  
  logger.info(`Processing email job ${job.id} to ${to}`);
  
  try {
    // Implement email sending logic
    // await emailService.send({ to, subject, text, html });
    logger.info(`Email job ${job.id} completed successfully`);
  } catch (error) {
    logger.error(`Email job ${job.id} failed:`, error);
    throw error;
  }
});

// Handle queue events
emailQueue.on('completed', (job) => {
  logger.info(`Email job ${job.id} completed`);
});

emailQueue.on('failed', (job, error) => {
  logger.error(`Email job ${job.id} failed:`, error);
});

// Add job to queue
export async function queueEmail(data: {
  to: string;
  subject: string;
  text: string;
  html?: string;
}): Promise<Bull.Job> {
  return emailQueue.add(data);
}
```

## Deployment

### Docker Configuration

```dockerfile
# Dockerfile for production
FROM node:20-alpine as builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Production stage
FROM node:20-alpine

WORKDIR /app

# Set Node.js to production mode
ENV NODE_ENV=production

# Copy package files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# Copy built files from builder stage
COPY --from=builder /app/dist ./dist

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

# Expose port
EXPOSE 3000

# Start the server
CMD ["node", "dist/server.js"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - DATABASE_URL=postgres://postgres:password@db:5432/api
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:14-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=api
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

## Case Studies

### API Scaling Challenge

**Situation**: Our customer-facing API encountered performance issues during peak traffic hours, with response times exceeding 2 seconds.

**Action**:
1. Implemented caching with Redis for frequently accessed endpoints
2. Added connection pooling for database queries
3. Optimized slow database queries with indexing
4. Implemented horizontal scaling with load balancing
5. Added rate limiting to prevent abuse

**Result**:
- Reduced average response time from 2s to 150ms
- Increased throughput by 500%
- Eliminated database connection issues
- Improved overall system stability

**Key Learning**: Database connection management and query optimization were the major bottlenecks, not server CPU or memory.

### Authentication System Redesign

**Situation**: Our legacy authentication system used session-based authentication with cookies, causing cross-origin issues and scaling problems.

**Action**:
1. Redesigned authentication to use JWT tokens
2. Implemented refresh token rotation
3. Added proper error handling for auth failures
4. Created middleware for role-based access control
5. Improved security headers and CSRF protection

**Result**:
- Simplified client integration across multiple platforms
- Eliminated cross-origin authentication issues
- Improved security by implementing token rotation
- Reduced server-side state management complexity

**Key Learning**: The migration required a careful transition period where both auth systems operated in parallel, with a gradual shift to the new system.

## Lessons Learned

Through developing numerous Express applications, we've learned these key lessons:

1. **Structured Error Handling**: Consistent error formats and proper status codes are crucial for API usability
2. **Database Connection Management**: Connection pooling and proper error handling prevent cascading failures
3. **Modular Architecture**: Clear separation of concerns simplifies maintenance and testing
4. **Performance Optimization**: Early identification of bottlenecks prevents scaling issues
5. **Security First**: Implementing security measures from the beginning is easier than retrofitting

## Version Management

Node.js version updates are handled as follows:
- We maintain compatibility with the current Node.js LTS version (currently 20.x)
- New projects should always use the current standardized version
- Existing projects should update to new Node.js LTS versions within 6 months of their release
- All Node.js version upgrades require thorough testing of the application
- We follow the Node.js release schedule to plan our upgrades