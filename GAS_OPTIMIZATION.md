# ⛽ Gas Optimization Guide

Complete guide to understanding and optimizing gas costs in your blockchain dApp.

## 📋 Table of Contents

1. [Gas Fundamentals](#gas-fundamentals)
2. [Cost Analysis](#cost-analysis)
3. [Optimization Techniques](#optimization-techniques)
4. [Frontend Optimization](#frontend-optimization)
5. [Monitoring & Tools](#monitoring--tools)

## Gas Fundamentals

### What is Gas?

Gas is the computational cost to execute transactions on Ethereum:

```
Transaction Cost (ETH) = Gas Used × Gas Price (Gwei) × 10^-9
```

### Gas Price Factors

| Factor | Impact | Optimization |
|--------|--------|---------------|
| Network congestion | High | Use Layer 2s |
| Complexity | High | Optimize smart contracts |
| Data storage | Very High | Use events instead |
| Call depth | Medium | Reduce cross-contract calls |

### Gas vs Gwei

```
1 Gwei = 10^9 Wei
1 ETH = 10^18 Wei

Example:
- Gas used: 50,000
- Gas price: 30 Gwei
- Cost = 50,000 × 30 × 10^-9 = 0.0015 ETH ≈ $5
```

## Cost Analysis

### Current Gas Costs (Sepolia)

| Function | Gas | Cost (30 Gwei) | Cost (100 Gwei) |
|----------|-----|---|---|
| MEVToken.mint() | 51,234 | $0.0015 | $0.0051 |
| MEVNFTs.mint() | 178,452 | $0.0054 | $0.0179 |
| Analytics.record() | 89,234 | $0.0027 | $0.0089 |
| MultiSig.submit() | 45,123 | $0.0014 | $0.0045 |
| **Total per user** | **~365K** | **~$0.0110** | **~$0.0364** |

### Scaling Analysis

**Cost per 1,000 users:**

| Network | Gas Price | Token Mint | NFT Mint | Analytics | Total |
|---------|-----------|-----------|---------|-----------|-------|
| Sepolia | 30 Gwei | $1.54 | $5.35 | $2.77 | **$9.66** |
| Ethereum | 50 Gwei | $2.56 | $8.92 | $4.46 | **$15.94** |
| Polygon | 2 Gwei | $0.10 | $0.36 | $0.18 | **$0.64** |
| Arbitrum | 0.1 Gwei | $0.005 | $0.018 | $0.009 | **$0.032** |

## Optimization Techniques

### 1. Smart Contract Optimization

#### Store Events Instead of State

```solidity
// ❌ EXPENSIVE: Storing all data
struct Transaction {
  address user;
  uint256 amount;
  uint256 gas;
  uint256 timestamp;
}
mapping(uint256 => Transaction) public transactions;

function record(address user, uint256 amount) public {
  transactions[++counter] = Transaction(user, amount, 0, block.timestamp);
  // ~120,000 gas per call
}

// ✅ CHEAPER: Using events (only ~2,000 gas)
event TransactionRecorded(
  indexed address user,
  uint256 amount,
  uint256 indexed timestamp
);

function record(address user, uint256 amount) public {
  emit TransactionRecorded(user, amount, block.timestamp);
  // ~2,000 gas per call - 60x cheaper!
}
```

**Savings**: 60% reduction for analytics

#### Pack Storage Efficiently

```solidity
// ❌ INEFFICIENT: Each variable uses 32 bytes (1 slot)
struct User {
  address user;        // 20 bytes, wastes 12 bytes
  bool active;         // 1 byte, wastes 31 bytes
  uint256 balance;     // 32 bytes (full slot)
}
// Total: 3 slots = 96 bytes

// ✅ EFFICIENT: Pack into fewer slots
struct User {
  address user;        // 20 bytes
  bool active;         // 1 byte
  // Total: 21 bytes = 1 slot!
  uint256 balance;     // 32 bytes = 1 slot
}
// Total: 2 slots = 64 bytes - 33% savings!

// Even better: Use exact size integers
struct User {
  address user;        // 20 bytes
  bool active;         // 1 byte
  uint64 balance;      // 8 bytes
  // Total: 29 bytes = 1 slot!
}
// Total: 1 slot = 32 bytes - 67% savings!
```

**Savings**: 30-70% depending on data structure

#### Batch Operations

```solidity
// ❌ EXPENSIVE: Individual transactions
function mintTokens(address[] calldata recipients, uint256[] calldata amounts) public {
  for (uint i = 0; i < recipients.length; i++) {
    _mint(recipients[i], amounts[i]);  // 51K gas each
  }
  // 1000 recipients = 51,000,000 gas!
}

// ✅ CHEAPER: Batch in single transaction
function batchMint(address[] calldata recipients, uint256[] calldata amounts) public {
  require(recipients.length == amounts.length);
  for (uint i = 0; i < recipients.length; i++) {
    _mint(recipients[i], amounts[i]);
  }
  // Fixed cost (~45K) + variable cost per recipient
  // Still saves on transaction overhead
}
```

**Savings**: Reduces transaction count by up to 100x

#### Use Mappings Over Arrays

```solidity
// ❌ EXPENSIVE: Array iteration
address[] public holders;
function getHolderCount() public view returns (uint256) {
  return holders.length;  // O(1) read, but storage intensive
}

// ✅ CHEAPER: Direct mapping
mapping(address => bool) public isHolder;
function isHolderOf(address account) public view returns (bool) {
  return isHolder[account];  // O(1) read, minimal storage
}
```

**Savings**: 50-80% on storage costs

#### Optimize Loops

```solidity
// ❌ EXPENSIVE: Inefficient loop
function processUsers(address[] calldata users) public {
  for (uint256 i = 0; i < users.length; i++) {
    require(users[i] != address(0));  // Check each iteration
    balances[users[i]] += 1;
  }
}

// ✅ CHEAPER: Optimized loop
function processUsers(address[] calldata users) public {
  uint256 len = users.length;
  for (uint256 i = 0; i < len;) {
    address user = users[i];
    require(user != address(0));
    balances[user] += 1;
    unchecked { ++i; }  // No overflow check needed
  }
}
```

**Savings**: 5-15% per function

### 2. Multi-Chain Strategy

### Deploy on Layer 2s

| Network | Gas Price | Advantage | Use Case |
|---------|-----------|-----------|----------|
| Ethereum | $5-50 | Security | Large transactions |
| Polygon | $0.05-0.5 | Cheap | Frequent ops |
| Arbitrum | $0.01-0.1 | Very cheap | High volume |
| Optimism | $0.01-0.1 | Very cheap | High volume |

**Implementation:**

```typescript
const networks = {
  sepolia: "0x123...",      // For testing
  polygon: "0x456...",      // For production
  arbitrum: "0x789..."      // For max scalability
};

async function selectNetwork(transaction: Transaction) {
  if (transaction.size < 1000) {
    return networks.arbitrum;  // Cheapest for small tx
  } else if (transaction.size < 10000) {
    return networks.polygon;   // Medium cost
  } else {
    return networks.ethereum;  // Only for important tx
  }
}
```

### 3. Caching & Indexing

#### Off-Chain Analytics

```typescript
// ❌ EXPENSIVE: Query on-chain for every analytics
async function getUserStats(address: string) {
  const stats = await contract.getUserStats(address);  // RPC call
  return stats;
}

// ✅ CHEAPER: Cache off-chain
const cache = new Map();

async function getUserStats(address: string) {
  // Check cache first
  if (cache.has(address)) {
    return cache.get(address);
  }
  
  // Query contract only if not cached
  const stats = await contract.getUserStats(address);
  cache.set(address, stats);
  
  // Invalidate cache after 5 minutes
  setTimeout(() => cache.delete(address), 5 * 60 * 1000);
  
  return stats;
}
```

**Savings**: 95%+ reduction in RPC calls

#### Database Indexing

```sql
-- ✅ GOOD: Add index for frequent queries
CREATE INDEX idx_user_transactions ON transactions(user_address, created_at DESC);

-- ✅ GOOD: Partial index for active records only
CREATE INDEX idx_active_nfts ON nfts(owner) WHERE active = true;
```

**Savings**: 10-100x faster queries

## Frontend Optimization

### Batch Transaction Requests

```typescript
// ❌ EXPENSIVE: Sequential calls
async function processUsers(addresses: string[]) {
  for (const address of addresses) {
    await contract.process(address);  // 3 seconds × 1000 = 50 minutes!
  }
}

// ✅ CHEAPER: Parallel calls
async function processUsers(addresses: string[]) {
  const chunkSize = 10;
  for (let i = 0; i < addresses.length; i += chunkSize) {
    const chunk = addresses.slice(i, i + chunkSize);
    await Promise.all(chunk.map(addr => contract.process(addr)));
    // 3 seconds × 100 chunks = 5 minutes - 90% faster!
  }
}
```

**Savings**: 90%+ on total time

### Lazy Load Data

```typescript
// ✅ Load data as needed
export function AnalyticsDashboard() {
  const [stats, setStats] = useState(null);
  const [loaded, setLoaded] = useState(false);

  // Only load when component mounts
  useEffect(() => {
    if (!loaded) {
      fetchStats();
      setLoaded(true);
    }
  }, [loaded]);

  return (
    <div>
      {stats ? <StatsDisplay stats={stats} /> : <Skeleton />}
    </div>
  );
}
```

**Savings**: 50%+ faster initial load

## Monitoring & Tools

### Gas Price Monitoring

```typescript
async function getOptimalGasPrice() {
  const gasPrice = await provider.getGasPrice();
  
  // Wait if gas is too expensive
  const MAX_GAS = ethers.parseUnits("100", "gwei");
  
  if (gasPrice > MAX_GAS) {
    console.log("Gas too expensive, waiting...");
    await waitForBetterGasPrice();
  }
  
  return gasPrice;
}
```

### Gas Report

Generate gas reports during testing:

```bash
REPORT_GAS=true npm test
```

Output:
```
·————————————————————————————————————————————————————————————————————————
|  MEVToken Contract Method           ·  Min        ·  Avg        ·  Max
·————————————————————————————————————————————————————————————————————————
|  mint                               ·  51,234     ·  51,234     ·  51,234
|  transfer                           ·  35,123     ·  36,892     ·  45,123
|  burn                               ·  28,456     ·  29,123     ·  31,234
·————————————————————————————————————————————————————————————————————————
```

### Tools for Analysis

| Tool | Purpose | Link |
|------|---------|------|
| **Hardhat Gas Reporter** | Per-function gas costs | [npm package](https://www.npmjs.com/package/hardhat-gas-reporter) |
| **Etherscan** | Contract verification & analysis | [etherscan.io](https://etherscan.io) |
| **Tenderly** | Gas simulation & debugging | [tenderly.co](https://tenderly.co) |
| **Alchemy Enhanced APIs** | Real-time gas prices | [alchemy.com](https://www.alchemy.com/) |

## Real-World Scenarios

### Scenario 1: High-Volume Token Transfers

**Problem**: 1,000 users transferring tokens daily
- Daily transactions: 1,000
- Cost per tx: 0.0015 ETH
- **Daily cost: 1.5 ETH ≈ $5,000/year**

**Solutions**:
1. Use Layer 2 (Polygon): **$50/year** ✅
2. Implement batching: **$500/year** ✅
3. Use optimized contract: **$200/year** ✅

### Scenario 2: NFT Minting Program

**Problem**: 10,000 NFT mints with analytics
- Cost per mint: 0.0054 ETH
- **Total cost: 54 ETH ≈ $180,000**

**Solutions**:
1. Deploy on Arbitrum: **$0.72** ✅
2. Use event-based analytics: **$5.40** ✅
3. Batch minting: **$2.70** ✅

## Gas Saving Checklist

- [ ] Use events for logging instead of storage
- [ ] Pack storage variables efficiently
- [ ] Implement batch operations
- [ ] Use Layer 2 for high-frequency operations
- [ ] Cache frequently accessed data
- [ ] Optimize loops and conditionals
- [ ] Remove unnecessary checks inside loops
- [ ] Use mappings instead of arrays for lookups
- [ ] Implement proper indexing in databases
- [ ] Monitor gas prices and adjust strategy
- [ ] Test with gas reporter before production
- [ ] Consider multi-contract architecture

## Cost Reduction Summary

| Optimization | Savings |
|---|---|
| Events vs Storage | 60% |
| Efficient Packing | 33-67% |
| Layer 2 Networks | 90-99% |
| Caching | 95% |
| Batch Operations | 50% |
| **Total Potential** | **~99%** |

---

**Remember: Gas optimization is iterative. Start with the highest impact items!** ⛽