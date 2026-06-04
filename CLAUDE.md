# CLAUDE.md - Agent Instructions

## Purpose
This file contains instructions for Claude AI agent to understand the OptionLab Pro project structure and working patterns.

## Project Overview
**OptionLab Pro** is a production-grade Indian options trading platform.

**Core Technology**:
- Frontend: Next.js 15 + React 19 + TypeScript
- Backend: NestJS + PostgreSQL + Redis
- Engines: Black-Scholes, Payoff, Greeks, Scanner, Backtesting
- State: Zustand
- Data Fetching: TanStack Query
- Charts: TradingView Lightweight Charts + Recharts

## Working Principles

### 1. Read Existing Docs First
Always check:
- `ARCHITECTURE.md` - System design
- `API_DESIGN.md` - API specifications
- `PROJECT_RULES.md` - Development rules
- Existing code before generating new code

### 2. Incremental Code Generation
- Generate code in small, logical chunks
- Complete one feature before moving to next
- Test each component before integration
- Use git-style diffs for changes

### 3. Never Rewrite Completed Modules
- Modify existing code only if requested
- Extend functionality instead of replacing
- Preserve working code
- Document changes with commit messages

### 4. Architecture Principles
- **Feature-based folder structure**: Each feature has own folder
- **Separation of concerns**: Hooks, API, Types, Store, Engine separate
- **TypeScript strict mode**: No `any` types
- **SOLID principles**: Single responsibility, Open/closed, Liskov, Interface segregation, Dependency inversion
- **Reusable components**: Create only when used in 2+ places

### 5. File Organization

#### Backend (NestJS)
```
src/modules/feature/
├── dto/              # Data Transfer Objects
├── entities/         # Database entities
├── feature.controller.ts
├── feature.service.ts
├── feature.module.ts
├── feature.gateway.ts (if WebSocket)
└── engines/          # Business logic engines
```

#### Frontend (Next.js)
```
features/feature/
├── api/             # API calls
├── hooks/           # Custom hooks
├── store/           # Zustand store
├── types/           # TypeScript types
├── components/      # Feature components
└── engine/          # Client-side logic
```

### 6. Code Style Guide

#### TypeScript
```typescript
// Strict typing
interface UserProfile {
  id: string;
  email: string;
  fullName: string;
}

// No any types
const handler = (user: UserProfile): void => {};

// Export types
export type StrategyLeg = {
  id: string;
  type: StrategyType;
};

// Use enum for constants
enum StrategyType {
  LONG_CALL = 'LONG_CALL',
  LONG_PUT = 'LONG_PUT',
}
```

#### React Components
```typescript
// Functional components only
interface ComponentProps {
  title: string;
  onAction?: () => void;
}

const MyComponent: React.FC<ComponentProps> = ({ title, onAction }) => {
  return <div>{title}</div>;
};

export default MyComponent;
```

#### NestJS Services
```typescript
@Injectable()
export class FeatureService {
  constructor(
    private prisma: PrismaService,
    private logger: Logger,
  ) {}

  async create(dto: CreateFeatureDto): Promise<Feature> {
    try {
      return await this.prisma.feature.create({
        data: dto,
      });
    } catch (error) {
      this.logger.error('Error creating feature', error);
      throw new HttpException('Failed to create', HttpStatus.BAD_REQUEST);
    }
  }
}
```

### 7. Commit Message Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

Examples:
```
feat(option-chain): add real-time WebSocket support
fix(strategy): correct Black-Scholes delta calculation
docs(api): update endpoint documentation
refactor(portfolio): improve Greeks aggregation
test(greeks): add unit tests for Vega calculation
```

### 8. Database Operations
- Use Prisma ORM exclusively
- Write migrations for schema changes
- Use transactions for complex operations
- Add database indexes for frequently queried fields

### 9. Error Handling
- Create custom exception filters
- Use HTTP status codes correctly
- Provide meaningful error messages
- Log errors with context

### 10. Testing
- Unit tests for business logic
- Integration tests for APIs
- E2E tests for critical workflows
- Aim for >80% code coverage

## Execution Workflow

### Phase 1: Planning
1. Understand requirements
2. Check existing docs
3. Identify affected modules
4. Plan implementation

### Phase 2: Design
1. Create data structures
2. Define API contracts
3. Plan state management
4. Document changes

### Phase 3: Implementation
1. Start with backend
2. Create database entities
3. Build API endpoints
4. Add WebSocket if needed
5. Build frontend
6. Create components
7. Add state management
8. Add hooks for API calls

### Phase 4: Integration
1. Connect frontend to API
2. Handle errors
3. Add loading states
4. Test workflows

### Phase 5: Testing
1. Write unit tests
2. Run integration tests
3. Test error scenarios
4. Load testing

## Module Descriptions

### Auth Module
- JWT token management
- OAuth integration
- OTP verification
- Session management

### Option Chain Module
- Real-time data streaming
- WebSocket gateway
- Data caching
- Greeks calculation

### Strategy Module
- Strategy builder
- Payoff engine
- Greeks aggregation
- Strategy validation

### Portfolio Module
- Virtual portfolio management
- Order management
- Position tracking
- Trade history

### Analytics Modules
- **Payoff Engine**: PnL calculations
- **Greeks Engine**: Black-Scholes implementation
- **Scanner Engine**: Market scanning
- **Backtesting Engine**: Historical testing

### AI Assistant
- Strategy suggestions
- Market analysis
- Trade recommendations
- LLM integration

## Shared Packages

### @shared/types
Common TypeScript types used across frontend and backend.

### @engines/black-scholes
Black-Scholes formula implementations for Greeks calculations.

### @engines/payoff
Payoff calculation engine for multi-leg strategies.

### @engines/greeks
Greeks aggregation for portfolio-level analysis.

### @engines/scanner
Scanner logic for market scanning.

## Important Patterns

### Zustand Store
```typescript
export const useStrategyStore = create<StrategyStore>((set) => ({
  strategies: [],
  selectedStrategy: null,
  setSelectedStrategy: (strategy) => set({ selectedStrategy: strategy }),
}));
```

### TanStack Query Hook
```typescript
export const useGetStrategies = () => {
  return useQuery({
    queryKey: ['strategies'],
    queryFn: async () => {
      const response = await api.get('/strategies');
      return response.data;
    },
  });
};
```

### NestJS Guard
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Implementation
  }
}
```

## When to Ask for Clarification
- Architecture decisions not in docs
- Business logic requirements
- Performance targets
- Deployment environment specifics
- Third-party API integrations

## Token Usage Guidelines
- Keep responses concise
- Use diffs instead of full rewrites
- Reference existing code patterns
- Use code blocks for code
- Minimize explanatory text

## Git Workflow
```bash
# Create feature branch
git checkout -b feat/feature-name

# Make changes
git add .
git commit -m "feat(module): description"

# Push to remote
git push origin feat/feature-name

# Create PR
# PR description should reference this docs
```

## Development Environment
- Node.js 20+
- PostgreSQL 15+
- Redis 7+
- Docker & Docker Compose
- VS Code recommended

## Common Commands
```bash
# Install dependencies
npm install

# Run development
npm run dev

# Run tests
npm run test

# Build production
npm run build

# Format code
npm run format

# Lint code
npm run lint

# Database migration
npm run db:migrate

# Database seed
npm run db:seed
```

## Useful Resources
- TradingView Lightweight Charts: [Docs](https://tradingview.com/lightweight-charts/)
- NestJS: [Docs](https://docs.nestjs.com/)
- Prisma: [Docs](https://www.prisma.io/docs/)
- Next.js: [Docs](https://nextjs.org/docs/)
- React: [Docs](https://react.dev/)

## Support & Escalation
If you encounter:
- **Architectural decisions**: Check ARCHITECTURE.md first
- **API design issues**: Check API_DESIGN.md first
- **Code quality questions**: Check PROJECT_RULES.md first
- **Unknown territory**: Ask for clarification before proceeding

---

Last Updated: 2026-01-15
Version: 1.0.0
