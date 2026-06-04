# OptionLab Pro - API Design Specification

## Overview
RESTful API design with WebSocket support for real-time data streaming. All endpoints use JSON for request/response.

---

## Authentication Endpoints

### POST `/api/auth/register`
Register new user with email.
```json
Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "fullName": "John Doe"
}

Response (201):
{
  "id": "uuid",
  "email": "user@example.com",
  "fullName": "John Doe",
  "accessToken": "jwt_token",
  "refreshToken": "refresh_token"
}
```

### POST `/api/auth/login`
Login with email/password.
```json
Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response (200):
{
  "id": "uuid",
  "accessToken": "jwt_token",
  "refreshToken": "refresh_token"
}
```

### POST `/api/auth/oauth/google`
Google OAuth callback.
```json
Request:
{
  "idToken": "google_id_token"
}

Response (200):
{
  "id": "uuid",
  "accessToken": "jwt_token",
  "refreshToken": "refresh_token"
}
```

### POST `/api/auth/otp/request`
Request OTP for phone login.
```json
Request:
{
  "phone": "+919876543210"
}

Response (200):
{
  "message": "OTP sent",
  "requestId": "uuid"
}
```

### POST `/api/auth/otp/verify`
Verify OTP.
```json
Request:
{
  "requestId": "uuid",
  "otp": "123456"
}

Response (200):
{
  "id": "uuid",
  "accessToken": "jwt_token",
  "refreshToken": "refresh_token"
}
```

### POST `/api/auth/refresh`
Refresh access token.
```json
Request:
{
  "refreshToken": "refresh_token"
}

Response (200):
{
  "accessToken": "new_jwt_token"
}
```

---

## Option Chain Endpoints

### GET `/api/option-chain/live`
Get live option chain with real-time data.
```json
Request Params:
?underlying=NIFTY50&expiry=2026-01-30&pageSize=50&page=1

Response (200):
{
  "data": [
    {
      "strikePrice": 23000,
      "callOI": 12345,
      "callOIChange": 100,
      "callVolume": 500,
      "callIV": 0.18,
      "callDelta": 0.65,
      "callGamma": 0.002,
      "callTheta": -0.05,
      "callVega": 0.08,
      "putOI": 54321,
      "putOIChange": -200,
      "putVolume": 400,
      "putIV": 0.19,
      "putDelta": -0.35,
      "putGamma": 0.002,
      "putTheta": -0.04,
      "putVega": 0.09,
      "callLTP": 150,
      "putLTP": 120
    }
  ],
  "underlying": "NIFTY50",
  "lastUpdated": "2026-01-15T10:30:00Z",
  "currentSpot": 23050
}
```

### GET `/api/option-chain/expirations`
Get available expiration dates.
```json
Response (200):
{
  "data": [
    {
      "date": "2026-01-22",
      "daysToExpiry": 7
    },
    {
      "date": "2026-01-30",
      "daysToExpiry": 15
    }
  ]
}
```

### WebSocket `/ws/option-chain/:underlying/:expiry`
Real-time option chain streaming.
```json
Message (incoming):
{
  "type": "subscribe",
  "underlying": "NIFTY50",
  "expiry": "2026-01-30"
}

Message (outgoing):
{
  "type": "quote",
  "strikePrice": 23000,
  "callOI": 12345,
  "callIV": 0.18,
  "putOI": 54321,
  "putIV": 0.19,
  "timestamp": "2026-01-15T10:30:00Z"
}
```

---

## Strategy Endpoints

### POST `/api/strategy/create`
Create new strategy.
```json
Request:
{
  "name": "Bull Call Spread",
  "underlyingSymbol": "NIFTY50",
  "expiryDate": "2026-01-30",
  "legs": [
    {
      "type": "BUY_CALL",
      "strikePrice": 23000,
      "quantity": 1,
      "premium": 150
    },
    {
      "type": "SELL_CALL",
      "strikePrice": 23200,
      "quantity": 1,
      "premium": 100
    }
  ]
}

Response (201):
{
  "id": "uuid",
  "name": "Bull Call Spread",
  "legs": [...],
  "payoff": {
    "maxProfit": 100,
    "maxLoss": 50,
    "breakEven": [23050],
    "pop": 0.65
  }
}
```

### GET `/api/strategy/:id`
Get strategy details.
```json
Response (200):
{
  "id": "uuid",
  "name": "Bull Call Spread",
  "createdAt": "2026-01-15T10:00:00Z",
  "legs": [...],
  "payoff": {...},
  "greeks": {
    "delta": 0.45,
    "gamma": 0.001,
    "theta": -0.03,
    "vega": 0.05
  }
}
```

### POST `/api/strategy/:id/calculate`
Calculate payoff and Greeks for strategy.
```json
Request:
{
  "underlyingPrices": [22800, 22900, 23000, 23100, 23200],
  "timeToExpiry": 0.04,
  "volatility": 0.18,
  "riskFreeRate": 0.06
}

Response (200):
{
  "payoffs": [50, 70, 100, 90, 50],
  "greeks": {
    "delta": 0.45,
    "gamma": 0.001,
    "theta": -0.03,
    "vega": 0.05
  },
  "maxProfit": 100,
  "maxLoss": 50,
  "breakEven": [23050],
  "pop": 0.65
}
```

### POST `/api/strategy/:id/validate`
Validate strategy (margin, risk checks).
```json
Response (200):
{
  "valid": true,
  "margin": 5000,
  "risk": 50,
  "warnings": []
}
```

### GET `/api/strategy/templates`
Get pre-built strategy templates.
```json
Response (200):
{
  "data": [
    {
      "id": "long_call",
      "name": "Long Call",
      "description": "Buy 1 Call",
      "legs": 1,
      "bullish": true
    },
    {
      "id": "bull_call_spread",
      "name": "Bull Call Spread",
      "description": "Buy 1 Call, Sell 1 Call",
      "legs": 2,
      "bullish": true
    }
  ]
}
```

---

## Portfolio & Trading Endpoints

### POST `/api/portfolio/order`
Place paper trading order.
```json
Request:
{
  "strategyId": "uuid",
  "orderType": "MARKET", // MARKET, LIMIT
  "price": 150, // For LIMIT orders
  "quantity": 1,
  "side": "BUY" // BUY, SELL
}

Response (201):
{
  "orderId": "uuid",
  "status": "FILLED",
  "avgPrice": 150,
  "quantity": 1,
  "createdAt": "2026-01-15T10:30:00Z"
}
```

### GET `/api/portfolio/positions`
Get active positions.
```json
Response (200):
{
  "data": [
    {
      "positionId": "uuid",
      "strategyName": "Bull Call Spread",
      "quantity": 1,
      "entryPrice": 50,
      "currentPrice": 65,
      "unrealizedPnL": 15,
      "unrealizedPnLPercent": 30,
      "greeks": {
        "delta": 0.45,
        "gamma": 0.001,
        "theta": -0.03,
        "vega": 0.05
      }
    }
  ],
  "totalUnrealizedPnL": 15
}
```

### GET `/api/portfolio/trades`
Get trade history.
```json
Response (200):
{
  "data": [
    {
      "tradeId": "uuid",
      "strategyName": "Bull Call Spread",
      "entryDate": "2026-01-15T10:00:00Z",
      "exitDate": "2026-01-20T14:00:00Z",
      "entryPrice": 50,
      "exitPrice": 65,
      "realizedPnL": 15,
      "realizedPnLPercent": 30,
      "duration": "5 days"
    }
  ]
}
```

### GET `/api/portfolio/summary`
Get portfolio summary.
```json
Response (200):
{
  "totalCapital": 100000,
  "deployedCapital": 45000,
  "availableMargin": 55000,
  "totalUnrealizedPnL": 2500,
  "totalRealizedPnL": 1200,
  "totalPnL": 3700,
  "portfolioGreeks": {
    "delta": 2.45,
    "gamma": 0.05,
    "theta": -0.15,
    "vega": 0.35
  },
  "activePositions": 3,
  "closedTrades": 12
}
```

---

## Scanner Endpoints

### GET `/api/scanner/high-iv`
High IV scan.
```json
Request Params:
?underlyings=NIFTY50,BANKNIFTY,FINNIFTY&percentile=90&limit=20

Response (200):
{
  "data": [
    {
      "underlying": "NIFTY50",
      "currentIV": 0.25,
      "ivPercentile": 92,
      "currentSpot": 23050,
      "change": 150,
      "changePercent": 0.65
    }
  ]
}
```

### GET `/api/scanner/iv-crush`
IV Crush scan.
```json
Response (200):
{
  "data": [
    {
      "underlying": "NIFTY50",
      "currentIV": 0.22,
      "previousIV": 0.28,
      "ivChange": -0.06,
      "ivChangePercent": -21.4,
      "expiryDate": "2026-01-30"
    }
  ]
}
```

### GET `/api/scanner/pcr-shift`
PCR Shift scan.
```json
Response (200):
{
  "data": [
    {
      "underlying": "NIFTY50",
      "currentPCR": 1.05,
      "previousPCR": 0.98,
      "pcrChange": 0.07,
      "sentiment": "BEARISH"
    }
  ]
}
```

### GET `/api/scanner/oi-buildup`
OI Build-up scan.
```json
Response (200):
{
  "data": [
    {
      "underlying": "NIFTY50",
      "strikePrice": 23000,
      "callOI": 150000,
      "callOIChange": 25000,
      "callOIChangePercent": 20,
      "side": "CALL",
      "sentiment": "BULLISH"
    }
  ]
}
```

---

## Backtesting Endpoints

### POST `/api/backtest/run`
Start backtest.
```json
Request:
{
  "strategyId": "uuid",
  "startDate": "2025-01-01",
  "endDate": "2026-01-15",
  "initialCapital": 100000,
  "leverage": 1,
  "rebalanceFrequency": "DAILY"
}

Response (202):
{
  "backtestId": "uuid",
  "status": "RUNNING"
}
```

### GET `/api/backtest/:id`
Get backtest results.
```json
Response (200):
{
  "id": "uuid",
  "status": "COMPLETED",
  "startDate": "2025-01-01",
  "endDate": "2026-01-15",
  "metrics": {
    "totalReturn": 25.5,
    "cagr": 18.2,
    "sharpeRatio": 1.45,
    "sortinoRatio": 2.1,
    "maxDrawdown": -12.5,
    "winRate": 62.5,
    "profitFactor": 2.3,
    "expectancy": 150,
    "trades": 48,
    "winningTrades": 30,
    "losingTrades": 18
  }
}
```

### WebSocket `/ws/backtest/:id`
Real-time backtest progress.
```json
Message (outgoing):
{
  "type": "progress",
  "completed": 75,
  "total": 100,
  "currentDate": "2025-08-15"
}

Message (outgoing, final):
{
  "type": "completed",
  "results": {...}
}
```

---

## AI Assistant Endpoints

### POST `/api/ai/suggest`
Get AI strategy suggestions.
```json
Request:
{
  "marketCondition": "BULLISH",
  "vix": 18.5,
  "avgIV": 0.20,
  "oiTrend": "INCREASING",
  "riskAppetite": "MODERATE"
}

Response (200):
{
  "suggestions": [
    {
      "strategy": "Bull Call Spread",
      "confidence": 0.85,
      "reasoning": "IV is moderate, bullish trend with controlled risk",
      "suggestedStrikes": {
        "buyStrike": 23000,
        "sellStrike": 23200
      },
      "expectedMetrics": {
        "maxProfit": 200,
        "maxLoss": 100,
        "pop": 0.68
      }
    }
  ]
}
```

### POST `/api/ai/analyze`
Analyze market conditions.
```json
Request:
{
  "prompt": "What should I do with NIFTY50 options with rising IV?"
}

Response (200):
{
  "analysis": "With rising IV, we enter a favorable environment for income strategies...",
  "strategies": ["Iron Condor", "Bull Call Spread"],
  "risks": ["IV reversion", "Gap risk"],
  "opportunities": ["Premium decay", "Directional plays"]
}
```

---

## User Management Endpoints

### GET `/api/users/profile`
Get user profile.
```json
Response (200):
{
  "id": "uuid",
  "email": "user@example.com",
  "fullName": "John Doe",
  "phone": "+919876543210",
  "subscription": {
    "plan": "PRO",
    "status": "ACTIVE",
    "expiresAt": "2026-02-15"
  },
  "createdAt": "2025-01-01T00:00:00Z"
}
```

### PUT `/api/users/profile`
Update user profile.
```json
Request:
{
  "fullName": "John Doe Updated",
  "phone": "+919876543210"
}

Response (200):
{
  "id": "uuid",
  "email": "user@example.com",
  "fullName": "John Doe Updated",
  "phone": "+919876543210"
}
```

---

## Error Response Format

All error responses follow this format:
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "User-friendly error message",
    "details": {
      "field": "email",
      "reason": "already registered"
    }
  },
  "statusCode": 400
}
```

### Common Error Codes
- `INVALID_REQUEST` (400): Validation failed
- `UNAUTHORIZED` (401): Authentication required
- `FORBIDDEN` (403): Insufficient permissions
- `NOT_FOUND` (404): Resource not found
- `CONFLICT` (409): Resource already exists
- `RATE_LIMITED` (429): Too many requests
- `INTERNAL_ERROR` (500): Server error

---

## Rate Limiting

- **Default**: 100 requests per minute per user
- **Scanner endpoints**: 30 requests per minute
- **Backtest endpoints**: 5 concurrent backtests per user
- **WebSocket**: 1 connection per user per session

---

## Pagination

All list endpoints support:
```json
Query Params:
?page=1&pageSize=20&sortBy=createdAt&sortOrder=DESC

Response:
{
  "data": [...],
  "pagination": {
    "currentPage": 1,
    "pageSize": 20,
    "totalItems": 150,
    "totalPages": 8,
    "hasNextPage": true
  }
}
```
