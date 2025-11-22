# Postman Collection Guide

## 📦 Collection Contents

The Postman collection includes **17 requests** organized into 4 folders:

### 1. Health & Status (3 requests)
- **Health Check**: Verify server is running
- **API Info**: Get endpoint list
- **Queue Metrics**: View queue statistics

### 2. Order Management (5 requests)
- **Submit Market Order - SOL to USDC**: Basic order submission
- **Submit Market Order - SOL to USDT**: Alternative token pair
- **Submit Market Order - USDC to SOL**: Reverse swap
- **Get Order by ID**: Retrieve order details
- **List Recent Orders**: View order history

### 3. Validation Tests (4 requests)
- **Invalid - Negative Amount**: Test error handling
- **Invalid - Same Token Pair**: Test validation rules
- **Invalid - Slippage Out of Range**: Test boundary conditions
- **Invalid - Missing Fields**: Test required fields

### 4. Concurrent Orders Test (5 requests)
- 5 pre-configured orders for testing concurrent processing

## 🚀 Quick Start

### Import to Postman

1. **Import Collection**:
   - Open Postman
   - Click "Import" button
   - Select `postman/order-execution.postman_collection.json`

2. **Import Environment** (Optional):
   - Import `postman/environment.local.json`
   - Or manually set `baseUrl` to `http://localhost:3000`

### Running Requests

#### Single Request
1. Select a request from the sidebar
2. Click "Send"
3. View response in the bottom panel

#### Multiple Requests (Collection Runner)
1. Right-click on "Concurrent Orders Test" folder
2. Select "Run folder"
3. Click "Run Order Execution Engine API"
4. Watch all 5 orders execute simultaneously

## 📝 Request Examples

### Submit Order
```http
POST http://localhost:3000/api/orders/execute
Content-Type: application/json

{
  "tokenIn": "SOL",
  "tokenOut": "USDC",
  "amount": 1.5,
  "slippage": 0.01
}
```

**Response:**
```json
{
  "orderId": "uuid-v4",
  "status": "pending",
  "message": "Order created successfully...",
  "websocketUrl": "/api/orders/execute?orderId=..."
}
```

### Get Order Status
```http
GET http://localhost:3000/api/orders/{orderId}
```

**Response:**
```json
{
  "orderId": "uuid-v4",
  "tokenIn": "SOL",
  "tokenOut": "USDC",
  "amount": 1.5,
  "slippage": 0.01,
  "status": "confirmed",
  "selectedDex": "raydium",
  "executedPrice": 100.5,
  "amountOut": 150.25,
  "txHash": "abc123...",
  "createdAt": "2025-11-22T10:00:00.000Z",
  "updatedAt": "2025-11-22T10:00:03.500Z"
}
```

## 🧪 Testing Scenarios

### Scenario 1: Basic Order Flow
1. Run "Submit Market Order - SOL to USDC"
2. Copy the `orderId` from response
3. Wait 3-5 seconds
4. Run "Get Order by ID" with the copied `orderId`
5. Verify status is "confirmed" with `txHash`

### Scenario 2: Concurrent Processing
1. Open Collection Runner
2. Select "Concurrent Orders Test" folder
3. Click "Run"
4. Observe all 5 orders processing
5. Check "Queue Metrics" to see statistics

### Scenario 3: Validation Testing
1. Run all requests in "Validation Tests" folder
2. Verify all return 400 status codes
3. Check error messages are descriptive

### Scenario 4: DEX Routing Analysis
1. Submit 10 orders using Collection Runner
2. Run "List Recent Orders"
3. Observe `selectedDex` field
4. Verify mix of Raydium and Meteora selections

## 📊 Expected Results

### Successful Order
- Status: `201 Created`
- Response time: < 100ms (submission only)
- Order processing: 3-5 seconds
- Final status: `confirmed`
- Transaction hash: 64-character hex string

### Validation Error
- Status: `400 Bad Request`
- Response time: < 50ms
- Error message: Descriptive validation message

### Order Statuses
Orders progress through these statuses:
1. `pending` - Initial state
2. `routing` - Comparing DEX quotes
3. `building` - Preparing transaction
4. `submitted` - Sent to blockchain
5. `confirmed` - Successfully executed
6. `failed` - Error occurred (with retry info)

## 🔍 Debugging Tips

### Server Not Responding
```bash
# Check if server is running
curl http://localhost:3000/health
```

### View Server Logs
Check terminal running `npm run dev` for:
- Routing decisions
- DEX price comparisons
- Processing status

### Check Queue Status
```bash
curl http://localhost:3000/api/metrics
```

Expected response:
```json
{
  "queue": {
    "waiting": 0,
    "active": 2,
    "completed": 15,
    "failed": 0,
    "total": 17
  }
}
```

## 💡 Tips & Tricks

1. **Auto-save Order ID**: The first request has a test script that automatically saves `orderId` to environment

2. **Collection Variables**: Use `{{baseUrl}}` and `{{orderId}}` in requests for flexibility

3. **Batch Testing**: Use Collection Runner with a delay of 500ms between requests

4. **Performance Testing**: Use Postman's Performance tab to analyze response times

5. **Export Results**: Save Collection Runner results for documentation

## 🎯 Test Coverage

The collection tests:
- ✅ Order submission and validation
- ✅ Order retrieval and listing
- ✅ Concurrent order processing
- ✅ Error handling and validation
- ✅ API health and metrics
- ✅ Different token pairs
- ✅ Various slippage tolerances

## 📹 Demo Workflow

For recording demo video:

1. Start with Health Check
2. Show Queue Metrics (empty)
3. Submit 5 concurrent orders (Collection Runner)
4. Show Queue Metrics (active processing)
5. Wait and show final results
6. Display individual order details
7. Show routing decisions in server logs

## 🔗 Related Files

- Collection: `postman/order-execution.postman_collection.json`
- Environment: `postman/environment.local.json`
- API Documentation: `README.md`
- Step Documentation: `docs/step-4.md`
