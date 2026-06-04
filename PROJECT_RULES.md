# PROJECT_RULES.md - Development Rules & Standards

## Code Quality Standards

### TypeScript Rules
- **Strict Mode**: `strict: true` in tsconfig.json
- **No Any**: Never use `any` type. Use `unknown` and narrow types
- **Explicit Return Types**: All functions must have explicit return types
- **Interface over Type**: Use `interface` for object contracts
- **Export Types**: Always export types from modules

```typescript
// ✅ Good
interface UserDTO {
  id: string;
  email: string;
}

const getUser = async (id: string): Promise<UserDTO> => {
  return await db.user.findUnique({ where: { id } });
};

// ❌ Bad
const getUser = async (id: string): any => {
  return db.user.findUnique({ where: { id } });
};
```

### Component Rules (React)

#### Functional Components Only
```typescript
// ✅ Good
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

const Button: React.FC<ButtonProps> = ({ label, onClick, variant = 'primary' }) => {
  return (
    <button className={`btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
};

export default Button;
```

#### Props Destructuring
- Always destructure props
- Provide default values
- Use optional chaining when needed

#### Memoization
- Use `React.memo` for expensive components
- Use `useMemo` for expensive calculations
- Use `useCallback` for event handlers passed as props

#### State Management
- Use Zustand for global state
- Use `useState` for local component state
- Use TanStack Query for server state
- Avoid prop drilling with 3+ levels

### Component Structure
```typescript
// imports
import React, { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

// types
interface Props {
  id: string;
  onClose?: () => void;
}

// component
const MyComponent: React.FC<Props> = ({ id, onClose }) => {
  // hooks
  const { data, isLoading } = useQuery({
    queryKey: ['item', id],
    queryFn: () => fetchItem(id),
  });

  // state
  const [count, setCount] = useState(0);

  // handlers
  const handleClick = () => {
    setCount(c => c + 1);
  };

  // render
  if (isLoading) return <LoadingSpinner />;

  return (
    <div>
      <p>{data?.name}</p>
      <button onClick={handleClick}>{count}</button>
    </div>
  );
};

export default MyComponent;
```

### NestJS Module Rules

#### Controller Structure
```typescript
import { Controller, Get, Post, Body, UseGuards, HttpCode } from '@nestjs/common';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { FeatureService } from './feature.service';
import { CreateFeatureDto } from './dto/create-feature.dto';

@Controller('features')
@UseGuards(JwtAuthGuard)
export class FeatureController {
  constructor(private featureService: FeatureService) {}

  @Get()
  async getAll() {
    return this.featureService.findAll();
  }

  @Post()
  @HttpCode(201)
  async create(@Body() dto: CreateFeatureDto) {
    return this.featureService.create(dto);
  }
}
```

#### Service Structure
```typescript
import { Injectable, Logger, HttpException, HttpStatus } from '@nestjs/common';
import { PrismaService } from '../database/prisma.service';

@Injectable()
export class FeatureService {
  private readonly logger = new Logger(FeatureService.name);

  constructor(private prisma: PrismaService) {}

  async findAll() {
    try {
      return await this.prisma.feature.findMany();
    } catch (error) {
      this.logger.error(`Error fetching features: ${error.message}`);
      throw new HttpException(
        'Failed to fetch features',
        HttpStatus.INTERNAL_SERVER_ERROR,
      );
    }
  }

  async create(dto: CreateFeatureDto) {
    try {
      return await this.prisma.feature.create({
        data: dto,
      });
    } catch (error) {
      this.logger.error(`Error creating feature: ${error.message}`);
      throw new HttpException(
        'Failed to create feature',
        HttpStatus.BAD_REQUEST,
      );
    }
  }
}
```

#### DTO Structure
```typescript
import { IsString, IsEmail, IsOptional, MinLength } from 'class-validator';

export class CreateFeatureDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsOptional()
  @IsString()
  description?: string;
}
```

#### Module Structure
```typescript
import { Module } from '@nestjs/common';
import { FeatureController } from './feature.controller';
import { FeatureService } from './feature.service';

@Module({
  imports: [],
  controllers: [FeatureController],
  providers: [FeatureService],
  exports: [FeatureService], // if used by other modules
})
export class FeatureModule {}
```

## Database Rules (Prisma)

### Schema Rules
- Use `@db.` annotations for specific column types
- Add indexes for frequently queried fields
- Use enums for fixed values
- Add timestamps (`createdAt`, `updatedAt`)
- Add soft deletes where applicable

```prisma
model Strategy {
  id        String    @id @default(cuid())
  userId    String
  name      String
  type      StrategyType
  status    StrategyStatus @default(DRAFT)
  
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime? // soft delete
  
  user      User      @relation(fields: [userId], references: [id])
  
  @@index([userId])
  @@index([status])
  @@fulltext([name]) // for search
}

enum StrategyType {
  LONG_CALL
  LONG_PUT
  BULL_CALL_SPREAD
  // ...
}

enum StrategyStatus {
  DRAFT
  ACTIVE
  COMPLETED
  ARCHIVED
}
```

### Query Rules
- Use Prisma select/include for efficiency
- Paginate large result sets
- Use transactions for multi-table operations
- Index foreign keys and filter fields

```typescript
// ✅ Good - Efficient query with includes
const strategies = await prisma.strategy.findMany({
  where: { userId, status: 'ACTIVE' },
  include: { legs: true, portfolio: true },
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: { createdAt: 'desc' },
});

// ❌ Bad - N+1 query problem
const strategies = await prisma.strategy.findMany({
  where: { userId },
});
// Then fetching legs separately for each strategy
```

## File Naming Conventions

### Backend (NestJS)
- Controllers: `feature.controller.ts`
- Services: `feature.service.ts`
- Modules: `feature.module.ts`
- DTOs: `create-feature.dto.ts`, `update-feature.dto.ts`
- Entities: `feature.entity.ts`
- Gateways: `feature.gateway.ts`
- Guards: `jwt-auth.guard.ts`
- Interceptors: `logging.interceptor.ts`
- Pipes: `validation.pipe.ts`
- Filters: `http-exception.filter.ts`

### Frontend (Next.js)
- Components: `MyComponent.tsx` (PascalCase)
- Hooks: `useMyHook.ts` (camelCase with use prefix)
- Stores: `myStore.ts` (camelCase)
- API: `api.ts` or `myFeatureApi.ts`
- Types: `types.ts` or `index.ts`
- Pages: `page.tsx` (Next.js convention)
- Layouts: `layout.tsx` (Next.js convention)

### Styling
- Tailwind classes in components
- CSS modules: `Component.module.css`
- Global styles: `globals.css`
- Theme variables: `variables.css`

## Folder Structure Rules

### Feature Folder Naming
- Use kebab-case: `option-chain`, `strategy-builder`
- Each feature is self-contained
- Related features can be grouped

### Public vs Private
- Prefix private files with underscore: `_privateHelper.ts`
- Export public API from `index.ts`

```typescript
// features/strategy/index.ts
export { useCreateStrategy } from './hooks/useCreateStrategy';
export { useStrategyStore } from './store/strategyStore';
export type { Strategy } from './types';
```

## Testing Rules

### Test File Location
- Feature tests in feature folder
- Test file name: `feature.test.ts` or `feature.spec.ts`

### Test Structure
```typescript
describe('FeatureService', () => {
  let service: FeatureService;
  let prisma: PrismaService;

  beforeEach(async () => {
    // Setup
  });

  describe('create', () => {
    it('should create a feature', async () => {
      // Arrange
      const dto = { name: 'Test' };

      // Act
      const result = await service.create(dto);

      // Assert
      expect(result).toBeDefined();
      expect(result.name).toBe('Test');
    });
  });
});
```

### Coverage Targets
- Aim for >80% coverage
- 100% for critical business logic
- Document exclusions

## Git & Version Control

### Commit Message Format
```
<type>(<scope>): <subject>

<body>

Closes #<issue-number>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Testing
- `chore`: Maintenance
- `perf`: Performance improvement
- `ci`: CI/CD changes

### Scope
Feature area: `auth`, `option-chain`, `strategy`, `portfolio`, `scanner`, `backtesting`

### Examples
```
feat(strategy): implement Bull Call Spread strategy
fix(greeks): correct Vega calculation for negative delta
docs(api): update endpoint documentation for option-chain
refactor(portfolio): improve Greeks aggregation performance
test(backtest): add unit tests for Sharpe Ratio calculation
```

### Branch Naming
- Feature: `feat/feature-name`
- Bugfix: `fix/bug-name`
- Hotfix: `hotfix/critical-bug`
- Release: `release/v1.0.0`

```bash
git checkout -b feat/bull-call-spread-strategy
git checkout -b fix/option-chain-websocket-lag
```

### PR Requirements
- Clear title and description
- Reference related issues
- All CI checks pass
- At least 1 approval
- Updated documentation

## Error Handling

### Backend Errors
```typescript
// ✅ Good - Specific error
throw new HttpException(
  'User with this email already exists',
  HttpStatus.CONFLICT,
);

// ❌ Bad - Generic error
throw new Error('Failed');
```

### Frontend Errors
```typescript
// ✅ Good - User-friendly error
const { error, isError } = useQuery({
  queryKey: ['strategies'],
  queryFn: fetchStrategies,
  onError: (error: AxiosError) => {
    toast.error(error.response?.data?.error?.message || 'Failed to fetch strategies');
  },
});
```

## Performance Guidelines

### Frontend
- Code split large features
- Lazy load components
- Image optimization
- Virtual scrolling for large lists
- Memoize expensive computations
- WebSocket for real-time data

### Backend
- Database query optimization
- Connection pooling
- Redis caching
- Async job processing
- Load balancing ready

### Metrics
- First Contentful Paint (FCP) < 2s
- Largest Contentful Paint (LCP) < 4s
- Cumulative Layout Shift (CLS) < 0.1
- API response time < 500ms
- Database query time < 100ms

## Security Rules

### Authentication
- JWT with RS256 signing
- Refresh token rotation
- Secure httpOnly cookies
- CORS policy enforcement

### Data Protection
- Never log sensitive data
- Validate all inputs
- Sanitize database queries (Prisma handles this)
- Hash passwords with bcrypt
- Encrypt sensitive fields

### API Security
- Rate limiting
- CSRF protection
- SQL injection prevention
- XSS prevention
- Helmet.js for headers

## Documentation Rules

### Code Comments
- Comment "why", not "what"
- Use JSDoc for functions
- Update comments when refactoring

```typescript
// ✅ Good
/**
 * Calculates Black-Scholes option price
 * @param spot - Current stock price
 * @param strike - Strike price
 * @param timeToExpiry - Time to expiration in years
 * @returns Option price
 */
const calculatePrice = (
  spot: number,
  strike: number,
  timeToExpiry: number,
): number => {
  // Using Euler's constant for precision
  return Math.exp(-riskFreeRate * timeToExpiry) * ...
};

// ❌ Bad
const calculatePrice = (spot, strike, timeToExpiry) => {
  // calculate price
  return Math.exp(-riskFreeRate * timeToExpiry) * ...
};
```

### README Updates
- Update README.md with new features
- Include setup instructions
- Document environment variables
- Add troubleshooting section

## Environment Variables

### Backend (.env)
```env
DATABASE_URL=postgresql://user:password@localhost:5432/optionlab
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-jwt-secret
JWT_EXPIRATION=15m
OAUTH_CLIENT_ID=your-client-id
OAUTH_CLIENT_SECRET=your-client-secret
```

### Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_WS_URL=ws://localhost:3000
```

## Review Checklist

Before submitting PR, verify:
- [ ] TypeScript strict mode passes
- [ ] Tests pass and coverage > 80%
- [ ] Linting passes
- [ ] No console.log statements
- [ ] No hardcoded values
- [ ] Error handling implemented
- [ ] API response documented
- [ ] Database migration created
- [ ] Documentation updated
- [ ] Commit messages follow format

## Tools & Linting

### ESLint Configuration
```bash
npm run lint
npm run lint:fix
```

### Prettier Configuration
```bash
npm run format
```

### TypeScript Check
```bash
npm run type-check
```

### Testing
```bash
npm run test
npm run test:coverage
```

## Deployment Checklist

- [ ] All tests pass
- [ ] Environment variables configured
- [ ] Database migrations run
- [ ] Secrets not committed
- [ ] Docker image builds
- [ ] Kubernetes manifests valid
- [ ] Load testing passed
- [ ] Security scan passed
- [ ] Documentation updated
- [ ] Release notes created

---

Last Updated: 2026-01-15
Version: 1.0.0
