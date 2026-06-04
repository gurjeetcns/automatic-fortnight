# OptionLab Pro - System Architecture

## Overview

OptionLab Pro is a production-grade Indian options trading platform built with modern architecture, scalable infrastructure, and institutional-grade analytics.

**Stack**: Next.js 15 + React 19 (Frontend) | NestJS + PostgreSQL + Redis (Backend) | Kubernetes-ready deployment

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT LAYER (Web)                       │
│  Next.js 15 | React 19 | TypeScript | TailwindCSS          │
│  ├── Authentication (JWT + OAuth + OTP)                     │
│  ├── Real-time WebSocket Connection                         │
│  └── State Management (Zustand)                             │
└────────────────────────┬────────────────────────────────────┘
                         │
                    HTTP/WebSocket
                         │
┌────────────────────────▼────────────────────────────────────┐
│                  API GATEWAY LAYER                          │
│  ├── Request Validation & Sanitization                      │
│  ├── Rate Limiting & Throttling                             │
│  ├── CORS & Security Headers                                │
│  └── JWT Authentication                                     │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│               BACKEND SERVICE LAYER (NestJS)                │
│  ├── Auth Module (JWT, OAuth, OTP)                          │
│  ├── Option Chain Module (Real-time streaming)              │
│  ├── Strategy Module (Builder + Validation)                 │
│  ├── Portfolio Module (Virtual + Paper Trading)             │
│  ├── Analytics Modules                                      │
│  │   ├── Payoff Engine                                      │
│  │   ├── Greeks Engine (Black-Scholes)                      │
│  │   ├── Scanner Engine (8+ scans)                          │
│  │   └── Backtesting Engine                                 │
│  ├── AI Assistant Module                                    │
│  └── WebSocket Gateways (Real-time)                         │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼────────┐ ┌─────▼──────┐ ┌──────▼────────┐
│  PostgreSQL    │ │   Redis    │ │  Message Q.   │
│  Database      │ │   Cache    │ │  (Bull/RabbitMQ)
│  ├── Users     │ │  ├── Auth  │ │  └── Async    │
│  ├── Orders    │ │  ├── Data  │ │      Jobs     │
│  ├── Positions │ │  ├── Live  │ │              │
│  ├── Strategies│ │  │  Quotes │ │              │
│  ├── Trades    │ │  └── Session│ │             │
│  └── Analytics │ │            │ │              │
└────────────────┘ └────────────┘ └───────────────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼────────┐ ┌─────▼──────┐ ┌──────▼────────┐
│  Market Data   │ │  LLM API   │ │  Email/SMS    │
│  Provider      │ │  (Claude)  │ │  Service      │
│  ├── NSE API   │ │  └── AI    │ │  └── Alerts   │
│  ├── Options   │ │     Insights│ │              │
│  └── Live Data │ │            │ │              │
└────────────────┘ └────────────┘ └───────────────┘
```

---

## Module Architecture

### 1. Authentication Module
- **JWT Token Management**: Access + Refresh tokens
- **OAuth Integration**: Google OAuth for social login
- **OTP Login**: Phone number verification
- **Session Management**: Redis-based sessions
- **Refresh Token Rotation**: Security best practice

### 2. Option Chain Module
- **Real-time Streaming**: WebSocket gateway for live updates
- **Data Aggregation**: NSE option chain data
- **Caching Strategy**: Redis cache with TTL
- **Filtering**: ATM, ITM, OTM filters
- **Greeks Calculation**: Real-time delta, gamma, theta, vega

### 3. Strategy Module
- **Strategy Builder**: 17+ pre-built strategies
- **Drag-and-drop Interface**: Visual strategy construction
- **Multi-leg Support**: Unlimited legs support
- **Payoff Engine**: Real-time PnL calculations
- **Greeks Aggregation**: Portfolio-level Greeks
- **Strategy Validation**: Risk checks and margin calculations

### 4. Portfolio Module
- **Virtual Portfolio**: Paper trading environment
- **Position Management**: Open, closed, expired positions
- **PnL Tracking**: Real-time and historical PnL
- **Trade History**: Complete audit trail
- **Greeks Aggregation**: Portfolio Greeks

### 5. Analytics Modules

#### Payoff Engine
- Calculates PnL at different stock prices
- Charts for expiry, T+0, T+1, T+3, T+5
- Break-even identification
- Max profit/loss calculation
- Risk/reward ratio

#### Greeks Engine
- Black-Scholes formula implementation
- Delta: Direction sensitivity
- Gamma: Delta acceleration
- Theta: Time decay
- Vega: Volatility sensitivity
- Rho: Interest rate sensitivity

#### Scanner Engine
- High IV scanner
- Low IV scanner
- IV Crush detection
- PCR Shift monitoring
- OI Build-up detection
- Long Build-up detection
- Short Covering detection
- Breakout Candidate identification

#### Backtesting Engine
- Historical option data processing
- Intraday data support
- Daily data support
- Metrics calculation:
  - CAGR
  - Sharpe Ratio
  - Sortino Ratio
  - Maximum Drawdown
  - Win Rate
  - Profit Factor
  - Expectancy

### 6. AI Assistant Module
- **Strategy Suggestion**: Based on market conditions
- **Market Analysis**: VIX, IV, OI interpretation
- **Trade Recommendations**: Risk/reward analysis
- **Natural Language**: Chat interface for insights
- **LLM Integration**: Claude API for intelligence

### 7. WebSocket Layer
- **Option Chain Updates**: Real-time quotes
- **Portfolio Updates**: Position changes
- **Alert Notifications**: Scanner triggers
- **Connection Management**: Reconnection logic
- **Message Broadcasting**: Efficient fan-out

---

## Data Flow

### Real-time Option Chain Update
```
NSE API
   ↓
Market Data Service (external)
   ↓
NestJS Option Chain Module
   ↓
Redis Cache (for aggregation)
   ↓
WebSocket Gateway
   ↓
Connected Clients (Next.js App)
   ↓
React State Update
   ↓
UI Render
```

### Strategy Calculation Flow
```
User Creates Strategy (Frontend)
   ↓
Strategy Validation (Frontend)
   ↓
POST /api/strategy (NestJS)
   ↓
Strategy Service
   ↓
Payoff Engine calculates PnL
   ↓
Greeks Engine aggregates portfolio Greeks
   ↓
Response to Frontend
   ↓
Chart rendering (TradingView Lightweight Charts)
```

### Backtesting Flow
```
User Selects Strategy + Date Range (Frontend)
   ↓
POST /api/backtest (NestJS)
   ↓
Backtesting Service
   ↓
Historical Data Retrieval (PostgreSQL)
   ↓
Backtest Engine Simulation
   ↓
Metrics Calculation
   ↓
Store Results (PostgreSQL)
   ↓
Stream Results via WebSocket
   ↓
Chart rendering (Recharts)
```

---

## Database Schema Layers

### Core Domain
- **users**: Authentication & profile
- **subscriptions**: User subscriptions
- **plans**: Subscription tiers

### Market Data
- **option_contracts**: Option contract definitions
- **option_chains**: Real-time option chain snapshots
- **market_data**: OHLCV data

### Trading
- **strategies**: User-created strategies
- **strategy_legs**: Strategy components
- **orders**: Paper trading orders
- **positions**: Active positions
- **trades**: Historical trades

### Analytics
- **backtests**: Backtest runs
- **backtest_results**: Results per run
- **alerts**: User alerts
- **watchlists**: User watchlists

---

## Caching Strategy

### Redis Cache Tiers

| Layer | Key Pattern | TTL | Use Case |
|-------|-------------|-----|----------|
| L1 | `quote:NIFTY50_CE` | 1s | Live option quotes |
| L2 | `chain:NIFTY50:2026-01-30` | 5s | Option chain snapshots |
| L3 | `user:{id}:portfolio` | 30s | Portfolio data |
| L4 | `strategy:{id}` | 5m | Strategy definitions |
| L5 | `session:{id}` | 24h | User sessions |

### Cache Invalidation
- Event-driven invalidation on data updates
- Time-based TTL expiration
- Manual cache clear on sensitive operations

---

## Security Architecture

### Authentication & Authorization
- JWT with RS256 signing
- OAuth 2.0 integration (Google)
- OTP verification (SMS-based)
- RBAC (Role-Based Access Control)

### Data Protection
- TLS/SSL encryption in transit
- Database encryption at rest
- Sensitive data hashing
- Environment variable management

### API Security
- CORS policy enforcement
- Rate limiting per endpoint
- Request validation & sanitization
- CSRF token protection
- SQL injection prevention (Prisma ORM)

### Audit & Monitoring
- Comprehensive logging
- Request/response tracking
- Error monitoring (Sentry)
- Performance metrics

---

## Performance Optimization

### Frontend
- Code splitting with Next.js
- Image optimization
- Lazy loading components
- Memoization (useMemo, useCallback)
- Virtual scrolling for large lists
- WebSocket for real-time updates

### Backend
- Database query optimization (Prisma)
- Connection pooling (PostgreSQL)
- Redis caching
- Message queue for async jobs
- Load balancing ready
- Horizontal scaling capability

### Infrastructure
- CDN for static assets
- API response compression
- Database indexing strategy
- Kubernetes auto-scaling

---

## Deployment Architecture

### Docker Containerization
- Separate containers for API and Web
- Multi-stage builds for optimization
- Environment-specific configurations
- Docker Compose for local development

### Kubernetes Deployment
- Stateless service design
- Horizontal Pod Autoscaling
- Resource limits and requests
- Health checks and readiness probes
- ConfigMaps and Secrets management

### CI/CD Pipeline
- GitHub Actions workflow
- Automated testing
- Code quality checks
- Docker image building
- Automated deployment to staging/production

---

## Scalability Considerations

### Horizontal Scaling
- Stateless API design
- Session management via Redis
- Database read replicas
- CDN for static content
- Message queue for decoupling

### Vertical Scaling
- Database optimization
- Query performance tuning
- Caching strategies
- Resource allocation

### Future Enhancements
- Microservices architecture
- Event streaming (Kafka)
- GraphQL for flexible querying
- Machine learning for predictions
- WebSocket server clustering

---

## Error Handling & Recovery

### Application Errors
- Global exception filters (NestJS)
- Structured error responses
- Error logging with context
- User-friendly error messages

### System Resilience
- Database connection pooling
- Redis fallback mechanisms
- Graceful degradation
- Retry logic with exponential backoff
- Circuit breaker pattern for external APIs

---

## Monitoring & Observability

### Metrics
- API response times
- Database query performance
- Cache hit/miss rates
- WebSocket connection count
- Error rates and types

### Logging
- Structured JSON logging
- Log aggregation
- Log retention policies
- Debug logging in development

### Alerting
- Real-time alerts for critical errors
- Performance degradation alerts
- Resource utilization alerts
- Scheduled health checks

---

## Technology Justification

| Component | Choice | Why |
|-----------|--------|-----|
| Frontend | Next.js 15 + React 19 | Best-in-class SSR, streaming, performance |
| Styling | TailwindCSS | Rapid development, maintainability |
| UI Components | ShadCN UI | High-quality, customizable components |
| State | Zustand | Lightweight, simple API |
| Data Fetching | TanStack Query | Advanced caching and sync |
| Charts | TradingView + Recharts | Professional-grade charting |
| Backend | NestJS | Enterprise-ready, scalable, TypeScript |
| Database | PostgreSQL | Reliable ACID compliance, advanced features |
| Cache | Redis | High performance, versatile |
| ORM | Prisma | Type-safe, modern, developer experience |
| Infrastructure | Docker + Kubernetes | Industry standard, cloud-native |

---

## Development Workflow

### Local Development
```bash
# Start services
docker-compose up

# Install dependencies
npm install

# Run development servers
npm run dev

# Run tests
npm run test

# Build for production
npm run build
```

### Deployment
```bash
# Build Docker images
docker build -t api:latest -f docker/Dockerfile.api .
docker build -t web:latest -f docker/Dockerfile.web .

# Push to registry
docker push api:latest
docker push web:latest

# Deploy to Kubernetes
kubectl apply -f k8s/
```

---

## Next Steps

1. **Database Schema Design** (Prisma)
2. **API Endpoint Specifications**
3. **WebSocket Event Definitions**
4. **Frontend Component Library**
5. **Integration Testing Framework**
6. **Deployment Configuration**
