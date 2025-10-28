# ⚡ Quick Start Guide

Get MEV Detector running in 5 minutes!

## 🎯 Step 1: Prerequisites

✅ Have these ready:
- Node.js 16+ installed
- Alchemy API key (get free one at https://www.alchemy.com/)
- MetaMask or any Web3 wallet
- Terminal/Command Prompt

## 🚀 Step 2: Setup Backend

```bash
# Navigate to server directory
cd server

# Install dependencies
npm install

# Create .env file
cat > .env << EOF
ALCHEMY_API_KEY=your_alchemy_key_here
ALCHEMY_ENDPOINT_MAINNET=https://eth-mainnet.g.alchemy.com/v2/
ALCHEMY_ENDPOINT_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/
PORT=5000
NETWORK=sepolia
PREMIUM_FEATURE_FEE=0.001
SIMULATION_FEE=0.0001
HISTORICAL_DATA_FEE=0.005
EOF

# Start backend
npm run dev
```

✅ Backend running at `http://localhost:5000`

Test it:
```bash
curl http://localhost:5000/api/health
```

## 🎨 Step 3: Setup Frontend

Open **new terminal window**:

```bash
# Navigate to frontend directory
cd mev-detector

# Install dependencies
npm install

# Create .env file
cat > .env << EOF
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_WS_URL=ws://localhost:5000
REACT_APP_NETWORK=sepolia
EOF

# Start frontend
npm start
```

✅ Frontend opens automatically at `http://localhost:3000`

## 🔗 Step 4: Connect & Test

1. Open http://localhost:3000 in your browser
2. Click "Connect Wallet"
3. Approve MetaMask connection
4. You're in! 🎉

## 🧪 Test Features

### Dashboard
- See real-time MEV attacks (simulated)
- View attack statistics
- Check gas prices

### Simulator
- Enter a contract address (e.g., Uniswap router)
- Set a transaction value
- Click "Run Simulation"
- See MEV risk assessment!

### Premium
- Try to access premium features
- They'll ask for payment
- Make a test transaction (on testnet only!)

## 📝 Key Files

```
Frontend:
- src/App.tsx              → Main application
- src/components/          → React components
- src/utils/api.ts        → API communication
- src/utils/web3.ts       → Wallet connection

Backend:
- src/server.ts           → Express server
- src/services/           → Alchemy, Flashbots, Payments
- src/utils/mevDetector.ts → Detection algorithm
- src/routes/api.ts       → API endpoints
```

## 🐛 Troubleshooting

### "Connection refused" error

```bash
# Check if backend is running
curl http://localhost:5000/api/health

# If not, check if port 5000 is in use
# Windows:
netstat -ano | findstr :5000

# Mac/Linux:
lsof -i :5000
```

### "Alchemy API key not working"

- Make sure `.env` file is created in `/server` directory
- API key must be valid (check Alchemy dashboard)
- Restart backend after changing .env

### MetaMask not connecting

- Make sure MetaMask extension is installed
- Try refreshing the page
- Check if it's connected to Sepolia testnet

### "Premium feature not working"

- Only works if you have fake ETH on testnet
- Get testnet ETH from faucet: https://sepoliafaucet.com
- Payment amount must match in .env

## 📊 What to Expect

### Dashboard View
- Shows recent MEV attacks (simulated data)
- Attack type distribution chart
- Risk scores and slippage losses
- Real-time websocket connection

### Attack Detection
- Polls mempool every 2 seconds
- Identifies sandwich attacks
- Calculates risk scores
- Estimates slippage losses

### User Features
- Create user profile on first login
- Track premium status
- View payment history
- Personal attack statistics

## 💡 Tips

1. **Test on Testnet**: Always use Sepolia testnet initially
2. **Check Logs**: Both frontend and backend show useful info
3. **Gas Monitoring**: Watch gas prices in Dashboard → Stats
4. **Simulation Safe**: No real transactions in simulator mode
5. **Premium Testing**: Use testnet ETH (free from faucet)

## 🔐 Security Notes

- Never share your private key
- Use testnet (Sepolia) for testing
- Mainnet requires real ETH
- Keep `.env` file private
- Don't commit `.env` to git

## 📱 Browser Console Tips

Open DevTools (F12) and check:

```javascript
// Check if window.ethereum exists
window.ethereum

// Check current network
window.ethereum.chainId

// Get connected accounts
window.ethereum.request({method: 'eth_accounts'})
```

## 🎓 Learning Resources

- [Alchemy Docs](https://docs.alchemy.com/)
- [Flashbots Docs](https://docs.flashbots.net/)
- [Ethers.js Guide](https://docs.ethers.org/)
- [React Docs](https://react.dev/)
- [Tailwind CSS](https://tailwindcss.com/)

## 🚢 Next Steps

After testing locally:

1. **Read DEPLOYMENT.md** for production setup
2. **Configure Mainnet** by changing `NETWORK=mainnet`
3. **Setup Domain** with DNS records
4. **Enable HTTPS** with Let's Encrypt
5. **Deploy** to your hosting (Heroku, AWS, DigitalOcean)

## ❓ Still Having Issues?

1. Check the logs:
   ```bash
   # Backend logs
   npm run dev
   
   # Frontend logs
   # Check browser console (F12)
   ```

2. Read full README.md for details

3. Check GitHub issues/discussions

## ✨ You're Ready!

Everything is running. Now:

- Explore the dashboard
- Try the simulator
- Test premium features
- Read the code
- Customize for your needs

**Happy protecting! 🛡️**

---

Questions? Open an issue or check the documentation!