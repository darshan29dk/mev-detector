# 🔐 Security Best Practices Guide

Production-ready security guidelines for your blockchain dApp.

## 📋 Table of Contents

1. [Smart Contract Security](#smart-contract-security)
2. [Backend Security](#backend-security)
3. [Frontend Security](#frontend-security)
4. [Infrastructure Security](#infrastructure-security)
5. [Operational Security](#operational-security)
6. [Monitoring & Alerting](#monitoring--alerting)

## Smart Contract Security

### Auditing Checklist

- [ ] **Reentrancy Prevention**: Use OpenZeppelin guards
- [ ] **Integer Overflow/Underflow**: Solidity 0.8+ has protection built-in
- [ ] **Access Control**: Proper RBAC implementation
- [ ] **Input Validation**: Check all external inputs
- [ ] **Gas Limits**: Loops won't exceed gas limits
- [ ] **Timestamp Manipulation**: Don't rely on `block.timestamp` for security
- [ ] **Front-Running**: Use commit-reveal schemes where needed

### Code Examples

#### ✅ GOOD: Access Control

```solidity
function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
    require(to != address(0), "Invalid address");
    require(amount > 0, "Invalid amount");
    _mint(to, amount);
}
```

#### ❌ BAD: No Access Control

```solidity
function mint(address to, uint256 amount) public {
    _mint(to, amount);  // Anyone can mint!
}
```

#### ✅ GOOD: Safe Transfer Pattern

```solidity
(bool success, ) = payable(recipient).call{value: amount}("");
require(success, "Transfer failed");
```

#### ❌ BAD: Unsafe Transfer

```solidity
payable(recipient).transfer(amount);  // Can fail silently
```

### Contract Upgrades

Use UUPS proxy pattern safely:

```solidity
function _authorizeUpgrade(address newImplementation)
    internal
    override
    onlyRole(UPGRADER_ROLE)
{}
```

**Critical**: Only UPGRADER_ROLE can approve upgrades!

## Backend Security

### Environment Variables

#### ✅ GOOD: Secure Configuration

```env
PRIVATE_KEY=your_key_here         # Never commit this!
ALCHEMY_API_KEY=your_key_here     # Keep secret
DATABASE_PASSWORD=your_password    # Encrypted storage
JWT_SECRET=strong_random_secret    # 32+ characters
```

#### ❌ BAD: Hardcoded Secrets

```typescript
const PRIVATE_KEY = "0x123...";  // DON'T DO THIS!
```

### Input Validation

```typescript
// ✅ GOOD: Validate all inputs
app.post("/api/transfer", (req, res) => {
  const { to, amount } = req.body;
  
  if (!ethers.isAddress(to)) {
    return res.status(400).json({ error: "Invalid address" });
  }
  
  if (!amount || isNaN(parseFloat(amount))) {
    return res.status(400).json({ error: "Invalid amount" });
  }
  
  // Process...
});
```

### Rate Limiting

```typescript
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  message: "Too many requests"
});

app.use("/api/", limiter);
```

### CORS Configuration

```typescript
import cors from "cors";

app.use(cors({
  origin: process.env.FRONTEND_URL,  // Only your frontend
  methods: ["GET", "POST"],
  credentials: true
}));
```

### Error Handling

#### ✅ GOOD: Safe Error Messages

```typescript
try {
  await contract.mint(address, amount);
} catch (error) {
  console.error("Mint error:", error);  // Log details
  res.status(500).json({
    error: "Transaction failed"  // Don't expose internals
  });
}
```

#### ❌ BAD: Exposing Sensitive Info

```typescript
res.status(500).json({ error: error.message });  // Exposes tech stack!
```

## Frontend Security

### Wallet Security

```typescript
// ✅ GOOD: Never ask for private keys
const provider = new ethers.BrowserProvider(window.ethereum);
const signer = await provider.getSigner();  // MetaMask handles signing

// ❌ BAD: Never do this!
// const privateKey = prompt("Enter your private key");
// const signer = new ethers.Wallet(privateKey, provider);
```

### Transaction Validation

```typescript
// ✅ GOOD: Display transaction details before signing
function handleTransaction() {
  const details = {
    to: recipientAddress,
    amount: ethers.parseEther("1"),
    gasLimit: 50000,
    gasPrice: ethers.parseUnits("30", "gwei")
  };
  
  // Show details to user
  console.log("Please review:", details);
  
  // User approves in wallet
  await signer.sendTransaction(details);
}
```

### Content Security Policy

```typescript
// In your server response headers
res.setHeader(
  "Content-Security-Policy",
  "default-src 'self'; script-src 'self' https://cdn.ethers.io"
);
```

### XSS Prevention

```typescript
// ✅ GOOD: React automatically escapes
<div>{userInput}</div>  // Safe!

// ❌ BAD: Never use dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{__html: userInput}} />
```

## Infrastructure Security

### HTTPS/TLS

Required for production:

```bash
# Using Let's Encrypt with nginx
sudo apt-get install certbot nginx
sudo certbot certonly --nginx -d yourdomain.com
```

### Database Security

```typescript
// ✅ GOOD: Use parameterized queries
const result = db.query(
  "SELECT * FROM users WHERE address = ?",
  [userAddress]
);

// ❌ BAD: String concatenation (SQL injection!)
const result = db.query(
  `SELECT * FROM users WHERE address = '${userAddress}'`
);
```

### Network Security

```nginx
# nginx.conf - Security headers
server {
    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    location /api/ {
        limit_req zone=api burst=20 nodelay;
    }
}
```

## Operational Security

### Private Key Management

#### ✅ GOOD: Hardware Wallet

```bash
# Use hardware wallet for deployments and critical operations
# Private key never touches your server
```

#### For Server Keys (if necessary):

```bash
# 1. Use strong random key
openssl rand -hex 32

# 2. Store in secure vault (AWS Secrets Manager, Vault, etc)
# 3. Rotate regularly
# 4. Audit access logs
# 5. Use multi-sig for critical operations
```

### Secret Rotation

```bash
#!/bin/bash
# scripts/rotate-secrets.sh

# Rotate private key
NEW_KEY=$(openssl rand -hex 32)

# Update in secret manager
aws secretsmanager update-secret --secret-id mev-detector-key --secret-string $NEW_KEY

# Rotate database password
# Update all connection strings
# Restart services
```

### Access Control

```typescript
// Multi-sig for critical operations
const multiSigOwners = [
  "0x123...",  // Owner 1
  "0x456...",  // Owner 2
  "0x789..."   // Owner 3
];

const requiredConfirmations = 2;  // 2-of-3 multisig
```

### Audit Logging

```typescript
// Log all sensitive operations
function auditLog(action: string, user: string, details: any) {
  console.log({
    timestamp: new Date(),
    action,
    user,
    details,
    ip: req.ip
  });
  
  // Send to security monitoring
  sendToSentry({ action, user, details });
}
```

## Monitoring & Alerting

### Contract Events Monitoring

```typescript
// Monitor for suspicious activities
contract.on("Transfer", (from, to, value) => {
  if (value > LARGE_AMOUNT_THRESHOLD) {
    alert(`Large transfer: ${value} from ${from} to ${to}`);
  }
});
```

### Real-Time Alerts

```typescript
// Set up monitoring service
const monitoring = {
  checkGasPrice: async () => {
    const price = await provider.getGasPrice();
    if (price > MAX_GAS_PRICE) {
      sendAlert("Gas price too high!");
    }
  },
  
  checkContractBalance: async () => {
    const balance = await provider.getBalance(CONTRACT_ADDRESS);
    if (balance < MIN_BALANCE) {
      sendAlert("Contract balance low!");
    }
  }
};

// Run checks every minute
setInterval(monitoring.checkGasPrice, 60000);
setInterval(monitoring.checkContractBalance, 60000);
```

### Metrics to Monitor

- 📊 Transaction failure rate
- ⛽ Average gas prices
- 💰 Total value locked
- 🔍 Unusual transaction patterns
- 🚨 Smart contract errors
- 📈 Performance metrics

## Compliance & Legal

### Data Privacy

- [ ] Comply with GDPR if serving EU users
- [ ] Have privacy policy
- [ ] Allow data deletion requests
- [ ] Secure user data with encryption

### Terms of Service

- [ ] Clearly state risks
- [ ] Explain no warranties
- [ ] Liability limitations
- [ ] User responsibilities

### KYC/AML (if applicable)

```typescript
// For financial operations
async function verifyUser(address: string) {
  // Check against sanctions lists
  // Implement KYC if required by jurisdiction
  // Store verification securely
}
```

## Incident Response Plan

### If Breach Occurs

1. **Immediate Actions**
   - [ ] Pause affected contracts
   - [ ] Notify users
   - [ ] Preserve evidence
   - [ ] Activate incident team

2. **Short-term**
   - [ ] Complete investigation
   - [ ] Deploy fix
   - [ ] Restore services
   - [ ] Document lessons learned

3. **Long-term**
   - [ ] Implement preventive measures
   - [ ] Update security policies
   - [ ] Conduct full audit
   - [ ] Increase monitoring

## Security Checklist

### Pre-Launch

- [ ] Smart contracts audited by third party
- [ ] All tests passing
- [ ] Security headers configured
- [ ] Rate limiting enabled
- [ ] CORS properly configured
- [ ] Private keys secured
- [ ] Database encrypted
- [ ] HTTPS enabled
- [ ] Monitoring set up
- [ ] Incident response plan ready

### Post-Launch

- [ ] Monitor logs daily
- [ ] Update dependencies monthly
- [ ] Review access logs weekly
- [ ] Backup data regularly
- [ ] Test disaster recovery
- [ ] Update security policies
- [ ] Rotate secrets regularly

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Solidity Security](https://docs.soliditylang.org/en/latest/security-considerations.html)
- [Ethereum Security](https://blog.cryptographyengineering.com/2014/04/08/lessons-learned-from-the-dao/)
- [OpenZeppelin Best Practices](https://docs.openzeppelin.com/)

---

**Remember: Security is not a feature, it's a requirement!** 🔐