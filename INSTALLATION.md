# 📦 Installation Instructions

Complete step-by-step guide to install and run MEV Detector locally.

## ✅ Prerequisites

Before starting, ensure you have:

1. **Node.js & npm**
   ```bash
   node --version  # Should be 16.x or higher
   npm --version   # Should be 8.x or higher
   ```
   
   If not installed: https://nodejs.org/

2. **Git**
   ```bash
   git --version
   ```
   
   If not installed: https://git-scm.com/

3. **Alchemy API Key** (Free)
   - Sign up: https://www.alchemy.com/
   - Create a new app
   - Get API key from dashboard
   - Note it down for later

4. **Web3 Wallet**
   - Install MetaMask: https://metamask.io/
   - Create account or import existing
   - Switch to Sepolia testnet

## 📥 Clone Repository

```bash
# Clone the repository
git clone https://github.com/your-org/mev-detector.git
cd mev-detector

# Verify structure
ls -la
# Should show: server/, mev-detector/, README.md, etc.
```

## 🔧 Backend Installation

### Step 1: Navigate to Server Directory
```bash
cd server
```

### Step 2: Install Dependencies
```bash
npm install
```

This installs:
- `express` - Web framework
- `ethers` - Ethereum library
- `axios` - HTTP client
- `dotenv` - Environment variables
- `ws` - WebSocket support
- And more...

### Step 3: Configure Environment

Create `.env` file in `server/` directory:

```bash
# Windows (PowerShell)
$env:CONTENT = @"
ALCHEMY_API_KEY=your_actual_api_key_here
ALCHEMY_ENDPOINT_MAINNET=https://eth-mainnet.g.alchemy.com/v2/
ALCHEMY_ENDPOINT_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/
FLASHBOTS_RELAY_URL=https://relay.flashbots.net
FLASHBOTS_PROTECT_RPC=https://protected.flashbots.net
PORT=5000
NODE_ENV=development
NETWORK=sepolia
PREMIUM_FEATURE_FEE=0.001
SIMULATION_FEE=0.0001
HISTORICAL_DATA_FEE=0.005
"@
$env:CONTENT | Out-File -FilePath .env -Encoding UTF8

# Mac/Linux
cat > .env << 'EOF'
ALCHEMY_API_KEY=your_actual_api_key_here
ALCHEMY_ENDPOINT_MAINNET=https://eth-mainnet.g.alchemy.com/v2/
ALCHEMY_ENDPOINT_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/
FLASHBOTS_RELAY_URL=https://relay.flashbots.net
FLASHBOTS_PROTECT_RPC=https://protected.flashbots.net
PORT=5000
NODE_ENV=development
NETWORK=sepolia
PREMIUM_FEATURE_FEE=0.001
SIMULATION_FEE=0.0001
HISTORICAL_DATA_FEE=0.005
EOF
```

### Step 4: Start Backend

```bash
npm run dev
```

You should see:
```
╔════════════════════════════════════════════════════╗
║     🚀 MEV Detector Server Started                ║
╠════════════════════════════════════════════════════╣
║ Server:      http://localhost:5000               ║
║ WebSocket:   ws://localhost:5000                 ║
║ Network:     sepolia                             ║
║ Environment: development                         ║
║ Mempool:     ✅ Configured                        ║
╚════════════════════════════════════════════════════╝
```

✅ **Backend is running!**

Test it in a new terminal:
```bash
curl http://localhost:5000/api/health
```

Should return:
```json
{
  "status": "ok",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "network": "sepolia",
  "version": "1.0.0"
}
```

## 🎨 Frontend Installation

### Step 1: Open New Terminal

Keep backend running in first terminal. Open **new terminal window**.

### Step 2: Navigate to Frontend
```bash
cd mev-detector
```

### Step 3: Install Dependencies
```bash
npm install
```

This installs:
- `react` - UI library
- `recharts` - Charting library
- `ethers` - Ethereum JS library
- `axios` - HTTP client
- `tailwindcss` - Styling
- And more...

### Step 4: Configure Environment

Create `.env` file in `mev-detector/` directory:

```bash
# Windows (PowerShell)
$env:CONTENT = @"
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_WS_URL=ws://localhost:5000
REACT_APP_NETWORK=sepolia
"@
$env:CONTENT | Out-File -FilePath .env -Encoding UTF8

# Mac/Linux
cat > .env << 'EOF'
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_WS_URL=ws://localhost:5000
REACT_APP_NETWORK=sepolia
EOF
```

### Step 5: Start Frontend

```bash
npm start
```

After a moment, your browser should open to:
```
http://localhost:3000
```

You should see MEV Detector dashboard! 🎉

## 🧪 First Test Run

### 1. Connect Wallet
- Click "Connect Wallet" button
- Approve MetaMask popup
- Select your Ethereum account
- Click "Connect"

### 2. View Dashboard
- See sample statistics
- Watch attack detection (simulated data)
- Check gas prices
- View charts and analytics

### 3. Test Simulator (Premium)
- Go to "Simulator" tab
- Try entering transaction data
- It will ask for premium access
- This is normal - simulator is a premium feature

### 4. View Premium Options
- Click "Premium" tab
- See available features
- Check pricing (in ETH)
- Test wallet integration

## 🔒 Get Testnet ETH (Optional)

To test payments, get free testnet ETH:

1. Go to https://sepoliafaucet.com/
2. Enter your Ethereum address
3. Receive 0.5 ETH on Sepolia testnet
4. Check MetaMask balance

## 📂 Project Structure Explained

```
mev-detector/
├── README.md              ← Start here
├── QUICK_START.md         ← Fast setup
├── INSTALLATION.md        ← This file
├── ARCHITECTURE.md        ← How it works
├── DEPLOYMENT.md          ← Production setup
│
├── server/                ← Backend code
│   ├── src/
│   │   ├── server.ts      ← Main server
│   │   ├── config.ts      ← Configuration
│   │   ├── types.ts       ← TypeScript types
│   │   ├── routes/api.ts  ← API endpoints
│   │   ├── services/      ← Business logic
│   │   └── utils/         ← Helper functions
│   ├── package.json
│   ├── tsconfig.json
│   └── .env               ← Secrets (create this)
│
├── mev-detector/          ← Frontend code
│   ├── src/
│   │   ├── App.tsx        ← Main component
│   │   ├── index.tsx      ← Entry point
│   │   ├── index.css      ← Styles
│   │   ├── components/    ← React components
│   │   └── utils/         ← Helpers
│   ├── public/            ← Static files
│   ├── package.json
│   ├── tailwind.config.js
│   └── .env               ← Config (create this)
│
└── docker-compose.yml     ← Docker setup
```

## 🆘 Troubleshooting

### Backend won't start

**Issue**: `Error: EADDRINUSE: address already in use :::5000`

**Solution**: Port 5000 is already in use
```bash
# Find what's using it
# Windows
netstat -ano | findstr :5000
# Kill the process or use different port
```

**Issue**: `Cannot find module 'ethers'`

**Solution**: Dependencies not installed
```bash
npm install
```

**Issue**: Alchemy API key error

**Solution**: Check `.env` file
```bash
# Verify file exists and has correct key
cat .env

# Make sure ALCHEMY_API_KEY is set correctly
```

### Frontend won't connect to backend

**Issue**: "Cannot reach API" error

**Solution**: Check backend is running
```bash
curl http://localhost:5000/api/health
```

If it fails, backend is not running. Start it:
```bash
cd server && npm run dev
```

**Issue**: CORS error in console

**Solution**: Backend might not be accessible
- Check if backend is running
- Check .env has correct API URL
- Try `http://localhost:5000` instead of `127.0.0.1:5000`

### MetaMask won't connect

**Issue**: "MetaMask not detected"

**Solution**: Install MetaMask extension
- Go to https://metamask.io/
- Install for your browser
- Refresh the page

**Issue**: "Wrong network" error

**Solution**: Switch to Sepolia testnet
- Open MetaMask
- Click network dropdown
- Select "Sepolia"
- Refresh page

## 📊 Verification Checklist

After installation, verify:

- [ ] Backend running at http://localhost:5000
- [ ] Backend API responding: `curl http://localhost:5000/api/health`
- [ ] Frontend running at http://localhost:3000
- [ ] Frontend loads without errors
- [ ] MetaMask installed and ready
- [ ] Connected to Sepolia testnet
- [ ] Can connect wallet on dashboard
- [ ] Dashboard shows sample data
- [ ] Can navigate between tabs
- [ ] Real-time alerts showing (WebSocket working)

## 🚀 What's Next?

After successful installation:

1. **Explore Features**
   - Check Dashboard metrics
   - Try Transaction Simulator
   - Review Premium options

2. **Review Code**
   - Look at `src/components/` for React
   - Check `server/src/services/` for backend logic
   - Read types in `server/src/types.ts`

3. **Customize**
   - Change colors in `tailwind.config.js`
   - Adjust detection thresholds in `server/src/config.ts`
   - Modify API endpoints in `server/src/routes/api.ts`

4. **Deploy** (when ready)
   - Follow DEPLOYMENT.md guide
   - Setup on cloud provider
   - Configure domain & SSL

## 📞 Getting Help

If you get stuck:

1. **Check Logs**
   - Backend: `npm run dev` shows live logs
   - Frontend: Open DevTools (F12) and check Console

2. **Read Documentation**
   - README.md - Full overview
   - QUICK_START.md - 5-minute setup
   - ARCHITECTURE.md - How it works

3. **Common Issues**
   - Most problems are port conflicts or missing env vars
   - Make sure .env files exist with correct values
   - Verify Node.js version is 16+

4. **Ask for Help**
   - Open GitHub issue
   - Check existing issues
   - Comment with error message and logs

## ✨ Success!

If you see all this working:

- Backend health check returns JSON ✅
- Frontend loads at http://localhost:3000 ✅
- Dashboard shows sample data ✅
- Can connect MetaMask wallet ✅
- Real-time alerts working ✅

**Congratulations! You're all set up.** 🎉

Now explore the features, customize for your needs, and when ready, deploy to production!

---

**Need help?** Check the main README.md or open an issue on GitHub.

Happy building! 🚀