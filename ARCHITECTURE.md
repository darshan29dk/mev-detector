# 🏗️ Architecture & Design

Detailed technical architecture of MEV Detector.

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                   User's Browser                         │
├─────────────────────────────────────────────────────────┤
│  React Frontend (port 3000)                             │
│  ├─ Dashboard: Real-time attack feed & statistics      │
│  ├─ Simulator: Transaction risk analysis               │
│  ├─ Premium: Feature purchases                         │
│  └─ Real-Time Alerts: WebSocket notifications          │
└────────────┬─────────────────────────────────────┬──────┘
             │                                     │
      HTTP/REST                                WebSocket
             │                                     │
┌────────────▼──────────────────────────────────────▼─────┐
│         Express.js Backend (port 5000)                  │
├───────────────────────────────────────────────────────┤
│  REST API Endpoints                                     │
│  ├─ GET /api/attacks                                   │
│  ├─ POST /api/simulate                                 │
│  ├─ POST /api/protected-relay                          │
│  ├─ POST /api/payment/process                          │
│  └─ WebSocket Server for real-time events             │
└────────┬──────────────┬──────────────────┬──────────────┘
         │              │                  │
         ▼              ▼                  ▼
    ┌────────┐    ┌──────────┐    ┌────────────────┐
    │ Alchemy│    │Flashbots │    │Payment Service │
    │  API   │    │  Services│    │(On-chain verify│
    └────────┘    └──────────┘    └────────────────┘
         │              │
         ▼              ▼
    ┌──────────────────────────┐
    │  Ethereum Network        │
    │  (Mainnet or Testnet)    │
    └──────────────────────────┘
```

## Component Architecture

### Frontend Layer

```
src/
├── App.tsx                 # Main app entry, routing
├── index.tsx              # React root
├── index.css              # Tailwind + globals
│
├── components/
│   ├── Header.tsx         # Navigation & wallet connect
│   ├── Dashboard.tsx       # Main dashboard with charts
│   ├── AttackCard.tsx     # Individual attack display
│   ├── TransactionSimulator.tsx  # Simulation form
│   ├── PremiumFeatures.tsx       # Payment UI
│   └── RealTimeAlerts.tsx        # WebSocket alerts
│
└── utils/
    ├── api.ts             # Axios client & endpoints
    └── web3.ts            # MetaMask integration
```

### Backend Layer

```
src/
├── server.ts              # Express app & WebSocket setup
│
├── routes/
│   └── api.ts             # All API endpoints
│
├── services/
│   ├── alchemyService.ts      # Mempool monitoring
│   ├── flashbotsService.ts    # Private relay & bundles
│   ├── paymentService.ts      # Premium feature mgmt
│   └── mempoolMonitor.ts      # Real-time scanning
│
├── utils/
│   └── mevDetector.ts     # Attack detection algorithm
│
├── types.ts               # TypeScript interfaces
└── config.ts              # Configuration
```

## Data Flow

### Attack Detection Flow

```
1. Mempool Monitor starts
   ├─ Polls Alchemy API every 2 seconds
   └─ Gets pending transactions
   
2. MEV Detector analyzes
   ├─ Is it a swap? (function signature check)
   ├─ Gas price analysis
   ├─ Value analysis
   ├─ Target router check
   └─ Calculate risk score
   
3. Sequential pattern detection
   ├─ Group by target address
   ├─ Look for sandwich patterns
   ├─ Check timing windows
   └─ Verify gas escalation
   
4. Alert broadcast
   ├─ Format alert event
   ├─ Send to all WebSocket clients
   └─ Store in history
   
5. Client receives
   ├─ Real-time alert notification
   ├─ Dashboard updates
   ├─ User informed
   └─ Recommendations provided
```

### Payment Flow

```
1. User clicks "Upgrade"
   │
   ├─ Check if wallet connected
   └─ Show premium features

2. User selects feature
   │
   ├─ Display price in ETH
   └─ Show duration

3. User approves payment
   │
   ├─ MetaMask opens
   ├─ User signs transaction
   └─ Transaction broadcast

4. Backend verifies
   │
   ├─ Wait for confirmation
   ├─ Query tx on blockchain
   └─ Verify amount & recipient

5. Grant access
   │
   ├─ Update user profile
   ├─ Set expiration time
   ├─ Update payment records
   └─ Send confirmation

6. Frontend updates
   │
   └─ Show premium status
```

### Transaction Simulation Flow

```
1. User enters tx data
   ├─ Recipient address
   ├─ Value (in ETH)
   └─ Optional tx data

2. Frontend validates
   ├─ Valid Ethereum address?
   ├─ Premium user?
   └─ Required fields present?

3. Backend simulates
   ├─ Create mock transaction
   ├─ Run MEV detector
   ├─ Check for risks
   ├─ Calculate slippage
   └─ Get recommendations

4. Return results
   ├─ Risk score
   ├─ Potential slippage
   ├─ Front-run/back-run costs
   └─ Safety recommendations

5. User sees report
   ├─ Risk assessment
   ├─ Cost breakdown
   └─ Protection options
```

## Data Models

### Attack Detection Result

```typescript
SandwichAttack {
  id: string                    // Unique ID
  victimTxHash: string          // Original transaction
  victimFrom: string            // Sender address
  victimTo: string              // Target contract
  victimValue: string           // Amount (wei)
  
  frontRunnerTxHash?: string    // Pre-tx from attacker
  backRunnerTxHash?: string     // Post-tx from attacker
  
  detectedAt: number            // Timestamp
  slippageLoss: string          // ETH amount lost
  slippageLossPercentage: number // Percentage loss
  
  riskScore: number             // 0-100
  attackType: string            // Type of attack
  confidence: number            // 0-100 confidence
  affectedUsers: string[]       // Impacted addresses
  
  tokens?: {
    tokenIn: string
    tokenOut: string
    amountIn: string
    amountOut: string
  }
  router?: string               // DEX router used
  pool?: string                 // Liquidity pool
}
```

### User Profile

```typescript
UserProfile {
  address: string               // Wallet address
  isPremium: boolean            // Premium status
  premiumExpiresAt?: number     // Expiration timestamp
  
  totalTransactionsMonitored: number
  totalSlippageLosses: string   // Total ETH lost
  attacksBlocked: number        // Protected transactions
  flashbotsProtectUsed: number  // Times used
  
  createdAt: number             // Profile creation time
}
```

## MEV Detection Algorithm

### Risk Scoring System

```
Base Score = 0

IF is_swap_transaction:
    Base Score += 20

IF known_dex_router:
    Base Score += 15

IF large_value_transfer (>1 ETH):
    Base Score += 30

IF high_gas_price (>150% of network avg):
    Base Score += 25

IF multiple_indicators (>3 present):
    Base Score += 10

FINAL_SCORE = min(Base Score, 100)

IF FINAL_SCORE >= 70:
    REPORT_ATTACK
```

### Attack Type Classification

```
IF high_gas AND is_swap:
    SANDWICH_ATTACK

ELIF large_value AND known_router:
    FRONT_RUN_ATTACK

ELIF sequential_pattern_detected:
    SANDWICH_ATTACK (sequential)

ELIF is_swap:
    GENERALIZED_MEV

ELSE:
    MEV_BOOST
```

### Sequential Pattern Detection

```
For each target address (router):
    Get all pending transactions to it
    Sort by timestamp
    
    For each transaction at position i (1 to n-2):
        tx_before = transactions[i-1]
        tx_victim = transactions[i]
        tx_after = transactions[i+1]
        
        time_diff_before = tx_victim.time - tx_before.time
        time_diff_after = tx_after.time - tx_victim.time
        
        IF time_diff_before < 5s AND time_diff_after < 5s:
            IF gas_ratio(tx_before, tx_victim) > 1.3:
                SANDWICH_ATTACK_DETECTED
```

## Security Architecture

### API Security

```
1. Input Validation
   ├─ Check address format
   ├─ Validate tx data
   └─ Sanitize all inputs

2. Rate Limiting
   ├─ 100 req/min per IP
   ├─ 1000 req/min per API key
   └─ Exponential backoff

3. CORS Protection
   ├─ Only allow frontend domain
   ├─ No credentials by default
   └─ Preflight checks

4. Authentication
   ├─ Wallet signature verification
   ├─ Session tokens
   └─ Premium access tokens
```

### Wallet Security

```
1. MetaMask Integration
   ├─ User controls private key
   ├─ Never transmit private key
   └─ Sign transactions client-side

2. Transaction Relay
   ├─ Flashbots handles encryption
   ├─ No mempool visibility
   └─ Custom ordering via relay
```

## Performance Optimization

### Frontend

```
1. Code Splitting
   ├─ Components lazy-loaded
   ├─ Charts on-demand
   └─ Routes separated

2. Caching
   ├─ LocalStorage for user data
   ├─ Browser cache headers
   └─ API response caching

3. Updates
   ├─ WebSocket instead of polling
   ├─ Only update changed data
   └─ Debounce user input
```

### Backend

```
1. Database (when added)
   ├─ Indexed queries
   ├─ Connection pooling
   └─ Query optimization

2. Caching
   ├─ Recent attacks in memory
   ├─ Gas price caching
   └─ User session cache

3. Concurrency
   ├─ Non-blocking I/O
   ├─ Multiple workers
   └─ Load balancing
```

## Scalability Considerations

### Horizontal Scaling

```
1. Load Balancer
   ├─ Nginx/HAProxy
   ├─ Round-robin
   └─ Health checks

2. Multiple Backend Instances
   ├─ Shared Redis cache
   ├─ Shared attack history
   └─ Sticky sessions for WebSocket

3. Database Replication
   ├─ Primary-replica setup
   ├─ Read replicas
   └─ Automatic failover
```

### Vertical Scaling

```
1. Memory
   ├─ Increase Node heap
   ├─ Add Redis
   └─ Optimize data structures

2. CPU
   ├─ Native modules
   ├─ Clustering
   └─ WebAssembly for compute-heavy

3. Network
   ├─ Compression
   ├─ Connection pooling
   └─ CDN for static assets
```

## Monitoring & Observability

### Key Metrics

```
1. Performance
   ├─ API response time
   ├─ WebSocket latency
   └─ Page load time

2. Reliability
   ├─ Error rate
   ├─ Uptime percentage
   └─ Failed transactions

3. Business
   ├─ Active users
   ├─ Premium conversions
   ├─ Total revenue
   └─ Attacks prevented
```

### Logging Strategy

```
1. Levels
   ├─ ERROR: Issues requiring attention
   ├─ WARN: Unusual but handled
   ├─ INFO: Normal operations
   └─ DEBUG: Detailed info

2. Aggregation
   ├─ Central log storage
   ├─ Log search & analysis
   └─ Alert on errors
```

---

**For implementation details, see code comments and type definitions.**