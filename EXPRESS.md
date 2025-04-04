# Express Development Standards

This guide outlines our organization's standards for Express.js development, focusing on backend services and APIs.

> 📌 **Return to**: [Main Development Guide](../README.md)

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
    "resolveJsonModule": true
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
    "test:watch": "jest --watch"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express": "^4.18.2",
    "helmet": "^7.0.0",
    "morgan": "^1.10.0",
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

// Initialize express app
const app: Application = express();

// Apply middleware
app.use(helmet()); // Security headers
app.use(cors()); // CORS support
app.use(morgan('dev')); // Logging
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

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

// Start the server
app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`);
});
```

### Environment Configuration

```typescript
// src/config/index.ts
import dotenv from 'dotenv';

// Load environment variables
dotenv.config();

export default {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  logLevel: process.env.LOG_LEVEL || 'info',
  databaseUrl: process.env.DATABASE_URL || 'postgres://localhost:5432/mydb',
  jwtSecret: process.env.JWT_SECRET || 'your-secret-key',
  jwtExpiresIn: process.env.JWT_EXPIRES_IN || '1d',
};
```

## Route Organization

### Modular Route Structure

```typescript
// src/routes/index.ts
import { Router } from 'express';
import userRoutes from './users';
import authRoutes from './auth';

const router = Router();

router.use('/users', userRoutes);
router.use('/auth', authRoutes);

export default router;
```

### Route Implementation

```typescript
// src/routes/users.ts
import { Router } from 'express';
import { UserController } from '../controllers';
import { authMiddleware } from '../middleware';

const router = Router();

// Get all users (protected)
router.get('/', authMiddleware, UserController.getAll);

// Get user by ID (protected)
router.get('/:id', authMiddleware, UserController.getById);

// Create new user
router.post('/', UserController.create);

// Update user (protected)
router.put('/:id', authMiddleware, UserController.update);

// Delete user (protected)
router.delete('/:id', authMiddleware, UserController.delete);

export default router;
```

## Controller Pattern

### Controller Implementation

```typescript
// src/controllers/users.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '../services';
import { logger } from '../utils';

export class UserController {
  // Get all users
  static async getAll(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const users = await UserService.getAll();
      res.status(200).json(users);
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
        res.status(404).json({ message: 'User not found' });
        return;
      }
      
      res.status(200).json(user);
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
      res.status(201).json(newUser);
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
        res.status(404).json({ message: 'User not found' });
        return;
      }
      
      res.status(200).json(updatedUser);
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
        res.status(404).json({ message: 'User not found' });
        return;
      }
      
      res.status(204).send();
    } catch (error) {
      logger.error(`Error in UserController.delete for id ${req.params.id}:`, error);
      next(error);
    }
  }
}
```

## Error Handling

### Error Handling Middleware

```typescript
// src/middleware/error.ts
import { Request, Response, NextFunction } from 'express';
import { logger } from '../utils';

interface AppError extends Error {
  statusCode?: number;
}

export default function errorMiddleware(
  err: AppError,
  req: Request,
  res: Response,
  _next: NextFunction
): void {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  logger.error(`[${req.method}] ${req.path} >> StatusCode: ${statusCode}, Message: ${message}`);
  
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
```

## Testing

We use Jest for testing:

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

// Mock the database model
jest.mock('../../../src/models/user', () => ({
  findAll: jest.fn(),
  findById: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
}));

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getById', () => {
    it('should return a user if found', async () => {
      const mockUser = { id: '1', name: 'Test User' };
      const UserModel = require('../../../src/models/user');
      UserModel.findById.mockResolvedValue(mockUser);

      const result = await UserService.getById('1');
      
      expect(UserModel.findById).toHaveBeenCalledWith('1');
      expect(result).toEqual(mockUser);
    });

    it('should throw NotFoundError if user not found', async () => {
      const UserModel = require('../../../src/models/user');
      UserModel.findById.mockResolvedValue(null);

      await expect(UserService.getById('1')).rejects.toThrow(NotFoundError);
      expect(UserModel.findById).toHaveBeenCalledWith('1');
    });
  });
});
```

### Integration Tests

```typescript
// tests/integration/routes/users.test.ts
import request from 'supertest';
import app from '../../../src/app';
import { UserService } from '../../../src/services';

// Mock the UserService
jest.mock('../../../src/services/users');

describe('User Routes', () => {
  describe('GET /api/users', () => {
    it('should return all users', async () => {
      const mockUsers = [
        { id: '1', name: 'User 1' },
        { id: '2', name: 'User 2' },
      ];
      
      UserService.getAll = jest.fn().mockResolvedValue(mockUsers);
      
      const response = await request(app).get('/api/users');
      
      expect(response.status).toBe(200);
      expect(response.body).toEqual(mockUsers);
      expect(UserService.getAll).toHaveBeenCalled();
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

const logger = winston.createLogger({
  level: config.logLevel,
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
  defaultMeta: { service: 'api-service' },
  transports: [
    // Console transport
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
  ],
});

// Add file transports in production
if (config.nodeEnv === 'production') {
  logger.add(
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' })
  );
  logger.add(
    new winston.transports.File({ filename: 'logs/combined.log' })
  );
}

export default logger;
```

## Environment Variables

Create a `.env.example` file with dummy values:

```
# Server Configuration
PORT=3000
NODE_ENV=development
LOG_LEVEL=info

# Database Configuration
DATABASE_URL=postgres://postgres:password@localhost:5432/mydb

# Authentication
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=1d

# API Configuration
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX=100
```

Load environment variables in your code:

```typescript
// src/config/index.ts
import dotenv from 'dotenv';

// Load environment variables
dotenv.config();

export default {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  logLevel: process.env.LOG_LEVEL || 'info',
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
  jwtExpiresIn: process.env.JWT_EXPIRES_IN || '1d',
  rateLimit: {
    window: parseInt(process.env.RATE_LIMIT_WINDOW || '15', 10),
    max: parseInt(process.env.RATE_LIMIT_MAX || '100', 10),
  },
};
```

## Documentation

Use Swagger/OpenAPI for API documentation:

```bash
# Install Swagger UI Express and OpenAPI
npm install swagger-ui-express
npm install --save-dev @types/swagger-ui-express
```

Create a Swagger configuration:

```typescript
// src/utils/swagger.ts
import swaggerUi from 'swagger-ui-express';

const swaggerDocument = {
  openapi: '3.0.0',
  info: {
    title: 'Express API',
    version: '1.0.0',
    description: 'Express API with TypeScript',
  },
  servers: [
    {
      url: '/api',
      description: 'API server',
    },
  ],
  paths: {
    '/users': {
      get: {
        summary: 'Get all users',
        responses: {
          '200': {
            description: 'A list of users',
            content: {
              'application/json': {
                schema: {
                  type: 'array',
                  items: {
                    $ref: '#/components/schemas/User',
                  },
                },
              },
            },
          },
        },
      },
    },
  },
  components: {
    schemas: {
      User: {
        type: 'object',
        properties: {
          id: {
            type: 'string',
            description: 'User ID',
          },
          name: {
            type: 'string',
            description: 'User name',
          },
          email: {
            type: 'string',
            description: 'User email',
          },
        },
      },
    },
  },
};

export const setupSwagger = (app) => {
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));
};
```

Add Swagger to your application:

```typescript
// src/app.ts
import express from 'express';
import { setupSwagger } from './utils/swagger';

const app = express();

// Other middleware...

// Set up Swagger
setupSwagger(app);

// Routes...

export default app;
```

## Version Management

Node.js version updates are handled as follows:
- We maintain compatibility with the current Node.js LTS version (currently 20.x)
- New projects should always use the current standardized version
- Existing projects should update to new Node.js LTS versions within 6 months of their release
- All Node.js version upgrades require thorough testing of the application
- We follow the Node.js release schedule to plan our upgrades