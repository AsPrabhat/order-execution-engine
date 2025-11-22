# Order Execution Engine

This is a market order execution engine that routes orders between Raydium and Meteora DEXs on Solana. It uses WebSocket for real-time status updates and BullMQ for handling concurrent orders.

## Features

- Market order execution with immediate best price
- DEX routing between Raydium and Meteora
- Real-time WebSocket updates for order status
- Queue-based processing (up to 10 concurrent orders)
- Automatic retry with exponential backoff (max 3 attempts)
- Mock DEX implementation for testing

## Tech Stack

- Node.js + TypeScript
- Fastify (web server + WebSocket)
- BullMQ + Redis (queue management)
- PostgreSQL (order storage)
- Vitest (testing)

## Setup

You need Node.js 18+ and Docker installed.

1. Clone and install:
```bash
git clone <repo-url>
cd order-execution-engine
npm install
```

2. Start Redis and PostgreSQL:
```bash
docker run -d -p 6379:6379 redis:alpine
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=password -e POSTGRES_DB=orderengine postgres:15
```

3. Copy environment file:
```bash
cp .env.example .env
```

4. Start server:
```bash
npm run dev
```

Server runs on http://localhost:3000

## API Usage

### Submit an Order

POST `/api/orders/execute`

Request:
```json
{
  "tokenIn": "SOL",
  "tokenOut": "USDC",
  "amount": 1.5,
  "slippage": 0.01
}
```

Response:
```json
{
  "orderId": "uuid-v4",
  "status": "pending"
}
```

The connection upgrades to WebSocket automatically for real-time updates.

### WebSocket Updates

You'll get status messages like:
```json
{
  "orderId": "uuid-v4",
  "status": "routing",
  "timestamp": "2025-11-22T10:30:00.000Z"
}
```

Status flow: pending → routing → building → submitted → confirmed/failed

## Testing

```bash
npm test              # Run all tests
npm run test:coverage # With coverage report
```

## Project Structure

```
src/
  server.ts           - Main server file
  routes/orders.ts    - Order endpoints
  services/dex-router.ts - DEX routing logic
  queues/order-queue.ts  - BullMQ worker
  models/order.ts     - Database models
  config/             - Redis & PostgreSQL setup
tests/                - All tests
postman/              - API collection
```

## Why Market Orders?

I chose market orders because they're the simplest to implement - they execute immediately at the best available price. The system can be extended for other order types:

- **Limit Orders**: Add a background service that checks prices periodically and executes when target price is hit
- **Sniper Orders**: Listen for new token launches and execute on detection

## Mock vs Real DEX

I used a mock implementation instead of real DEX SDKs to focus on the architecture and avoid dealing with devnet complexities. The mock simulates realistic delays (200ms for quotes, 2-3s for execution) and price variations between DEXs.

To use real DEXs, you'd integrate the Raydium and Meteora SDKs and handle actual transactions on devnet.

## Endpoints

- `GET /health` - Health check
- `GET /` - API info
- `GET /api/metrics` - Queue stats
- `POST /api/orders/execute` - Submit order (upgrades to WebSocket)
- `GET /api/orders/:orderId` - Get order by ID
- `GET /api/orders` - List recent orders

## Test Results

All 43 tests passing:
- 14 DEX router tests
- 15 order model tests  
- 2 WebSocket tests
- 12 integration tests

Tested with 10 concurrent orders successfully.

## Postman Collection

The `postman/` folder has a collection with 17 requests for testing the API. Import `order-execution.postman_collection.json` in Postman.

Check `postman/README.md` for usage.

## Deployment

You can deploy to Railway or Render (both have free tiers).

### Railway
```bash
npm install -g @railway/cli
railway login
railway init
railway add postgresql
railway add redis
railway up
```

### Render
1. Connect your GitHub repo
2. Create a new Web Service
3. Add PostgreSQL and Redis add-ons
4. Set environment variables
5. Deploy

### Environment Variables
```
PORT=3000
NODE_ENV=production
DATABASE_URL=<your-postgres-url>
REDIS_URL=<your-redis-url>
QUEUE_CONCURRENCY=10
MAX_RETRIES=3
```

## Demo Video

Demo video: [To be uploaded]

## Resources

- Postman collection in `postman/` folder
- [Raydium SDK](https://github.com/raydium-io/raydium-sdk-V2-demo)
- [Meteora Docs](https://docs.meteora.ag/)
