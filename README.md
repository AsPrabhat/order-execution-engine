# Order Execution Engine

A Market Order execution engine with DEX routing between Raydium and Meteora on Solana. Features real-time WebSocket updates, concurrent order processing via BullMQ, and intelligent price-based routing.

## 🎯 Features

- **Market Order Execution**: Immediate execution at current best price
- **DEX Routing**: Automatic routing between Raydium and Meteora based on best quotes
- **WebSocket Streaming**: Real-time order status updates (pending → routing → building → submitted → confirmed/failed)
- **Concurrent Processing**: Handle up to 10 concurrent orders with queue management
- **Retry Logic**: Exponential backoff with max 3 retry attempts
- **Mock Implementation**: Realistic DEX simulation with configurable delays

## 🏗️ Architecture

- **Backend**: Node.js + TypeScript + Fastify
- **Queue**: BullMQ + Redis
- **Database**: PostgreSQL (order history)
- **Cache**: Redis (active orders)
- **WebSocket**: Real-time bidirectional communication

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- Docker (for Redis and PostgreSQL)
- Git

### Installation

1. **Clone the repository**
```bash
git clone <repository-url>
cd order-execution-engine
```

2. **Install dependencies**
```bash
npm install
```

3. **Start Redis and PostgreSQL**
```bash
docker run -d -p 6379:6379 redis:alpine
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=password -e POSTGRES_DB=orderengine postgres:15
```

4. **Configure environment**
```bash
cp .env.example .env
# Edit .env with your configuration
```

5. **Run the development server**
```bash
npm run dev
```

The server will start at `http://localhost:3000`

## 📡 API Usage

### Submit Market Order

**Endpoint**: `POST /api/orders/execute`

**Request Body**:
```json
{
  "tokenIn": "SOL",
  "tokenOut": "USDC",
  "amount": 1.5,
  "slippage": 0.01
}
```

**Response**:
```json
{
  "orderId": "uuid-v4",
  "status": "pending"
}
```

The connection automatically upgrades to WebSocket for real-time status updates.

### WebSocket Status Updates

Once connected, you'll receive status messages:

```json
{
  "orderId": "uuid-v4",
  "status": "routing",
  "timestamp": "2025-11-22T10:30:00.000Z"
}
```

**Status Flow**: `pending` → `routing` → `building` → `submitted` → `confirmed` (with `txHash`) or `failed` (with `error`)

## 🧪 Testing

```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Watch mode
npm test -- --watch
```

## 📦 Project Structure

```
order-execution-engine/
├── src/
│   ├── server.ts              # Fastify server setup
│   ├── routes/
│   │   └── orders.ts          # Order submission endpoint
│   ├── services/
│   │   └── dex-router.ts      # DEX routing logic
│   ├── queues/
│   │   └── order-queue.ts     # BullMQ worker
│   ├── models/
│   │   └── order.ts           # Database models
│   └── config/
│       ├── redis.ts           # Redis client
│       └── database.ts        # PostgreSQL client
├── tests/                     # Unit and integration tests
├── docs/                      # Step-by-step documentation
└── postman/                   # Postman collection
```

## 🎓 Design Decisions

### Why Market Orders?

Market orders execute immediately at the current best available price, making them the simplest order type to implement and test. This architecture can be extended to support:

- **Limit Orders**: Add a price monitoring service that polls DEX quotes periodically and triggers execution when the target price is reached.
- **Sniper Orders**: Integrate with token launch event listeners and execute immediately upon migration/launch detection.

### Mock vs Real Implementation

This project uses a **mock implementation** for DEX interactions to:
- Focus on architecture and order flow
- Eliminate blockchain network dependencies
- Enable faster testing and development
- Simulate realistic delays (200ms quotes, 2-3s execution)

For real devnet execution, integrate `@raydium-io/raydium-sdk-v2` and `@meteora-ag/dynamic-amm-sdk`.

## 🚢 Deployment

Deployment instructions coming soon (Railway/Render).

## 📹 Demo

Demo video link: Coming soon

## 🔗 Resources

- [Postman Collection](./postman/order-execution.postman_collection.json)
- [Step-by-Step Documentation](./docs/)
- [Raydium SDK](https://github.com/raydium-io/raydium-sdk-V2-demo)
- [Meteora Docs](https://docs.meteora.ag/)

## 📄 License

MIT
