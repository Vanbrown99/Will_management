# 📜 DecentralWill — Decentralized Will Platform

> A production-ready platform for creating, encrypting, and executing digital wills on the blockchain. Powered by Ethereum/Polygon, IPFS, and Node.js.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Node](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen)
![Solidity](https://img.shields.io/badge/solidity-0.8.19-purple)
![MongoDB](https://img.shields.io/badge/mongodb-7.x-green)
![Status](https://img.shields.io/badge/status-in%20development-orange)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running the App](#running-the-app)
- [Build Phases](#-build-phases)
- [Smart Contracts](#-smart-contracts)
- [API Reference](#-api-reference)
- [Security](#-security)
- [Testing](#-testing)
- [Deployment](#-deployment)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🌐 Overview

**DecentralWill** is a fully decentralized digital will platform that allows users to:

- Write and encrypt their will using AES-256 encryption
- Store the encrypted document permanently on **IPFS**
- Register the IPFS hash on-chain via a **Solidity smart contract**
- Define beneficiaries and asset distribution percentages
- Use a **Dead Man Switch** — if the owner goes inactive for a set number of days, the will is automatically executed
- Authenticate securely using **email + JWT** (no crypto wallet required to get started)

No intermediaries. No lawyers required. No single point of failure.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔐 AES-256 Encryption | Will content encrypted client-side before upload |
| 🌐 IPFS Storage | Permanent, decentralized document storage |
| ⛓️ On-Chain Registration | IPFS hash and beneficiaries stored on Ethereum/Polygon |
| ⏱️ Dead Man Switch | Auto-execution after configurable inactivity period |
| 👥 Multi-Beneficiary | Assign percentage shares to multiple beneficiaries |
| 📧 Email Auth | JWT-based email/password login — no wallet needed |
| 📊 Dashboard | Full will management with status tracking |
| 🚀 Auto-Execution | Smart contract distributes assets automatically |
| 🔒 No Sensitive DB Storage | Only metadata stored in MongoDB, never will content |
| 📱 Responsive UI | Works on desktop and mobile |

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER BROWSER                            │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Frontend (HTML / CSS / Vanilla JS)                    │  │
│   │   index.html · login.html · register.html               │  │
│   │   create-will.html · dashboard.html                     │  │
│   └──────────────────────┬──────────────────────────────────┘  │
└──────────────────────────┼──────────────────────────────────────┘
                           │ REST API (JWT)
                           ▼
┌──────────────────────────────────────┐
│        Node.js / Express Backend     │
│  ┌─────────────┐  ┌───────────────┐  │
│  │   MongoDB   │  │  IPFS Client  │  │
│  │ (metadata)  │  │ (encrypted    │  │
│  └─────────────┘  │  documents)   │  │
│                   └───────────────┘  │
│  ┌────────────────────────────────┐  │
│  │     Dead Man Switch Scheduler  │  │
│  │     (node-cron)                │  │
│  └────────────────────────────────┘  │
└──────────────────┬───────────────────┘
                   │ ethers.js
                   ▼
┌──────────────────────────────────────┐
│     Ethereum / Polygon Network       │
│  ┌─────────────────────────────────┐ │
│  │  WillFactory.sol                │ │
│  │  ├── createWill()               │ │
│  │  └── getWillsByOwner()          │ │
│  │                                 │ │
│  │  WillContract.sol (per user)    │ │
│  │  ├── updateWill()               │ │
│  │  ├── ping()                     │ │
│  │  ├── triggerExecution()         │ │
│  │  └── addBeneficiary()           │ │
│  └─────────────────────────────────┘ │
└──────────────────────────────────────┘
```

### Data Flow

```
User writes will
      │
      ▼
AES-256 encrypt (client-side)
      │
      ▼
Upload encrypted blob to IPFS → returns CID
      │
      ▼
Store CID + beneficiaries on-chain via smart contract
      │
      ▼
Save metadata (CID, contract address, status) to MongoDB
      │
      ▼
Dead Man Switch monitors activity
      │
      ├── Owner pings regularly → switch resets
      │
      └── Inactivity detected → triggerExecution() called
                                        │
                                        ▼
                              Assets distributed to beneficiaries
```

---

## 🛠 Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| HTML5 / CSS3 | Structure and styling |
| Vanilla JavaScript | Logic and interactivity |
| ethers.js (Phase 6) | Blockchain interaction |
| MetaMask (optional) | Wallet connection for on-chain actions |

### Backend
| Technology | Purpose |
|---|---|
| Node.js 18+ | Runtime |
| Express.js | REST API framework |
| MongoDB + Mongoose | Metadata storage |
| JWT + bcryptjs | Authentication |
| node-cron | Dead man switch scheduler |
| Helmet + CORS | Security headers |

### Blockchain
| Technology | Purpose |
|---|---|
| Solidity 0.8.19 | Smart contract language |
| Hardhat | Development and testing framework |
| ethers.js | Contract interaction |
| Polygon Mumbai | Testnet deployment |
| Polygon Mainnet | Production deployment |

### Storage & Encryption
| Technology | Purpose |
|---|---|
| IPFS (Infura) | Decentralized document storage |
| AES-256-CBC | Will content encryption |
| crypto (Node.js) | Key derivation and hashing |

---

## 📁 Project Structure

```
decentralized-will/
│
├── backend/                          # Node.js API server
│   ├── config/
│   │   └── db.js                     # MongoDB connection
│   ├── models/
│   │   ├── User.js                   # User schema (email, bcrypt password)
│   │   └── Will.js                   # Will metadata schema
│   ├── routes/
│   │   ├── authRoutes.js             # /api/auth/* — register, login, logout
│   │   └── willRoutes.js             # /api/wills/* — CRUD + ping + execute
│   ├── middleware/
│   │   ├── authMiddleware.js         # JWT protect middleware
│   │   └── errorHandler.js           # Global error handler
│   ├── services/                     # (Phase 3+)
│   │   ├── ipfsService.js            # IPFS upload/retrieve
│   │   ├── encryptionService.js      # AES-256 encrypt/decrypt
│   │   └── blockchainService.js      # ethers.js contract calls
│   ├── schedulers/                   # (Phase 7)
│   │   └── deadManSwitch.js          # Cron job for inactivity checks
│   ├── .env                          # Environment variables (never commit)
│   ├── .env.example                  # Safe template
│   ├── server.js                     # Express entry point
│   └── package.json
│
├── frontend/                         # Static web UI
│   ├── index.html                    # Landing page
│   ├── login.html                    # Login page
│   ├── register.html                 # Registration page
│   ├── create-will.html              # Will creation form
│   ├── dashboard.html                # User dashboard
│   ├── css/
│   │   └── styles.css                # Shared stylesheet (dark theme)
│   └── js/
│       ├── app.js                    # Core utilities, toast notifications
│       └── auth.js                   # Auth module (login, register, JWT)
│
├── contracts/                        # Solidity smart contracts
│   ├── WillFactory.sol               # Factory — deploys individual will contracts
│   ├── WillContract.sol              # Per-user will with DMS logic
│   ├── scripts/
│   │   └── deploy.js                 # Hardhat deployment script
│   ├── test/
│   │   └── Will.test.js              # Contract unit tests
│   ├── hardhat.config.js             # Hardhat configuration
│   └── package.json
│
├── .gitignore
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

Make sure you have the following installed:

```bash
node --version    # >= 18.0.0
npm --version     # >= 9.0.0
mongod --version  # >= 6.0
git --version
```

You will also need:
- A free [Infura](https://infura.io) account (for IPFS + RPC)
- A free [Alchemy](https://alchemy.com) account (optional, for RPC)
- [MetaMask](https://metamask.io) browser extension (for blockchain features)

---

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/decentralized-will.git
cd decentralized-will

# 2. Install backend dependencies
cd backend
npm install

# 3. Install contract dependencies
cd ../contracts
npm install

# 4. Go back to root
cd ..
```

---

### Environment Variables

```bash
# Copy the template
cp backend/.env.example backend/.env
```

Open `backend/.env` and fill in your values:

```env
# ── Server ──────────────────────────────────────
PORT=5000
NODE_ENV=development

# ── MongoDB ──────────────────────────────────────
MONGO_URI=mongodb://localhost:27017/decentralized_will

# ── Authentication ───────────────────────────────
JWT_SECRET=your_random_32_character_secret_here
JWT_EXPIRES_IN=7d

# ── Encryption (AES-256) ─────────────────────────
# Must be exactly 32 characters
ENCRYPTION_KEY=your_32_character_encryption_key_

# ── Blockchain ───────────────────────────────────
RPC_URL=https://polygon-mumbai.g.alchemy.com/v2/YOUR_KEY
DEPLOYER_PRIVATE_KEY=your_wallet_private_key_here
WILL_FACTORY_ADDRESS=0x_deployed_contract_address

# ── IPFS (Infura) ────────────────────────────────
IPFS_PROJECT_ID=your_infura_ipfs_project_id
IPFS_PROJECT_SECRET=your_infura_ipfs_project_secret
IPFS_GATEWAY=https://ipfs.infura.io/ipfs/

# ── Dead Man Switch ──────────────────────────────
PING_INTERVAL_DAYS=30
CHECK_INTERVAL_CRON=0 0 * * *
```

> ⚠️ **NEVER commit your `.env` file.** It is already in `.gitignore`.

---

### Running the App

#### Start MongoDB

```bash
# macOS / Linux
mongod --dbpath /data/db

# Windows
mongod --dbpath "C:\data\db"

# Or using MongoDB Atlas (cloud) — just set MONGO_URI in .env
```

#### Start the Backend

```bash
cd backend
npm run dev
# Server running at http://localhost:5000
```

#### Open the Frontend

```bash
# The backend serves the frontend automatically
# Open your browser and go to:
http://localhost:5000
```

> Do NOT open HTML files by double-clicking. Always use `http://localhost:5000` so relative paths resolve correctly.

#### Available Pages

| URL | Page |
|---|---|
| `http://localhost:5000` | Landing page |
| `http://localhost:5000/login.html` | Login |
| `http://localhost:5000/register.html` | Register |
| `http://localhost:5000/create-will.html` | Create will |
| `http://localhost:5000/dashboard.html` | Dashboard |

---

## 📦 Build Phases

This project is built in structured phases:

| Phase | Status | Description |
|---|---|---|
| **Phase 1** | ✅ Complete | Project setup, Express server, MongoDB, frontend structure |
| **Phase 2** | 🔄 Next | Solidity smart contracts, Hardhat deployment |
| **Phase 3** | ⏳ Pending | IPFS integration, AES encryption/decryption |
| **Phase 4** | ⏳ Pending | Full REST API — create, fetch, update wills |
| **Phase 5** | ⏳ Pending | Frontend forms, dashboard, ethers.js integration |
| **Phase 6** | ⏳ Pending | Blockchain transaction sending, gas handling |
| **Phase 7** | ⏳ Pending | Dead man switch cron job, ping system |
| **Phase 8** | ⏳ Pending | Security hardening, rate limiting, replay attack prevention |
| **Phase 9** | ⏳ Pending | Unit tests, contract tests, integration tests |
| **Phase 10** | ⏳ Pending | Production deployment (VPS, testnet, static hosting) |

---

## 📄 Smart Contracts

### WillFactory.sol
Deploys individual `WillContract` instances for each user.

```solidity
function createWill(
    string memory _ipfsCid,
    address[] memory _beneficiaries,
    uint256[] memory _allocations,
    uint256 _inactivityDays
) external returns (address willAddress)
```

### WillContract.sol
Individual will contract per user. Contains all execution logic.

```solidity
function updateWill(string memory _newIpfsCid) external onlyOwner
function ping() external onlyOwner                    // Reset dead man switch
function triggerExecution() external                  // Execute if conditions met
function addBeneficiary(address _addr, uint256 _pct) external onlyOwner
function revoke() external onlyOwner
```

### Deploy Contracts

```bash
cd contracts

# Start local Hardhat node
npx hardhat node

# Deploy to localhost
npx hardhat run scripts/deploy.js --network localhost

# Deploy to Polygon Mumbai testnet
npx hardhat run scripts/deploy.js --network mumbai
```

### Run Contract Tests

```bash
cd contracts
npx hardhat test
```

---

## 📡 API Reference

### Auth Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/api/auth/register` | Create new account | None |
| `POST` | `/api/auth/login` | Login, returns JWT | None |
| `GET` | `/api/auth/me` | Get current user | ✅ JWT |
| `POST` | `/api/auth/logout` | Logout | ✅ JWT |

#### Register
```bash
POST /api/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "SecurePass123",
  "ethereumAddress": "0x..." // optional
}
```

#### Login
```bash
POST /api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "SecurePass123"
}

# Response:
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": { "id": "...", "name": "John Doe", "email": "..." }
}
```

### Will Endpoints (all require JWT)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/wills` | Create a new will |
| `GET` | `/api/wills` | Get all wills for current user |
| `GET` | `/api/wills/:willId` | Get single will |
| `PUT` | `/api/wills/:willId` | Update a will |
| `POST` | `/api/wills/:willId/ping` | Reset dead man switch |
| `POST` | `/api/wills/:willId/execute` | Trigger execution |
| `DELETE` | `/api/wills/:willId` | Soft delete a will |

#### Create Will
```bash
POST /api/wills
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "My Last Will",
  "content": "I leave 50% to...",   // encrypted before storage
  "network": "mumbai",
  "beneficiaries": [
    { "email": "alice@example.com", "walletAddress": "0x...", "allocationPercent": 60 },
    { "email": "bob@example.com",   "walletAddress": "0x...", "allocationPercent": 40 }
  ],
  "deadManSwitch": {
    "isEnabled": true,
    "inactivityDays": 30
  }
}
```

---

## 🔐 Security

This platform follows multiple layers of security best practices:

### Data Security
- ✅ **Will content is AES-256 encrypted** client-side before upload
- ✅ **MongoDB never stores will content** — only IPFS CID and metadata
- ✅ **Passwords hashed with bcrypt** (12 salt rounds)
- ✅ **JWT tokens expire** after 7 days
- ✅ **Private keys never stored** — only used at deployment time

### API Security
- ✅ **Helmet.js** — sets 15+ secure HTTP headers
- ✅ **CORS whitelist** — restricts allowed origins
- ✅ **Input validation** via `express-validator`
- ✅ **Rate limiting** (Phase 8) — prevents brute force
- ✅ **Generic auth errors** — never reveals which field is wrong

### Smart Contract Security
- ✅ **`onlyOwner` modifier** on all sensitive functions
- ✅ **Reentrancy guards** on execution functions
- ✅ **Allocation validation** — beneficiary shares must total 100%
- ✅ **Replay attack prevention** via nonces (Phase 8)

### What We Never Store
```
❌ Will content (plaintext or encrypted)
❌ Private keys
❌ Passwords (only bcrypt hashes)
❌ Personal beneficiary information beyond wallet address
```

---

## 🧪 Testing

### Backend Tests

```bash
cd backend
npm test

# With coverage report
npm test -- --coverage
```

### Contract Tests

```bash
cd contracts
npx hardhat test

# With gas report
REPORT_GAS=true npx hardhat test
```

### Manual API Testing

```bash
# Health check
curl http://localhost:5000/api/health

# Register
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@test.com","password":"Test1234"}'

# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"Test1234"}'

# Protected route (use token from login)
curl http://localhost:5000/api/wills/health \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## 🌍 Deployment

### Backend (VPS / Cloud)

```bash
# Install PM2
npm install -g pm2

# Start with PM2
cd backend
pm2 start server.js --name "decentralwill-api"
pm2 save
pm2 startup

# Set NODE_ENV to production
echo "NODE_ENV=production" >> .env
```

### Frontend (Static Hosting)

The `frontend/` folder is pure HTML/CSS/JS — deploy to any static host:

```bash
# Netlify
netlify deploy --dir=frontend --prod

# Vercel
vercel --cwd frontend

# Or serve via your backend (already configured in server.js)
```

### Smart Contracts (Polygon Mumbai Testnet)

```bash
# Get test MATIC from faucet: https://faucet.polygon.technology

cd contracts
npx hardhat run scripts/deploy.js --network mumbai

# Copy the deployed address to your .env:
# WILL_FACTORY_ADDRESS=0x_your_deployed_address
```

### Production Checklist

```
✅ NODE_ENV=production in .env
✅ Strong JWT_SECRET (32+ chars, random)
✅ MongoDB Atlas with IP whitelist
✅ HTTPS enabled (Let's Encrypt / Cloudflare)
✅ Rate limiting enabled
✅ CORS set to your production domain
✅ Smart contracts audited before mainnet
✅ Contract addresses saved and backed up
✅ IPFS pinning service configured (Pinata / Infura)
```

---

## 🗺 Roadmap

- [x] Phase 1 — Project setup, auth, frontend UI
- [ ] Phase 2 — Smart contracts (WillFactory + WillContract)
- [ ] Phase 3 — IPFS integration + AES encryption
- [ ] Phase 4 — Full REST API
- [ ] Phase 5 — Frontend dashboard + form integration
- [ ] Phase 6 — Blockchain transaction layer
- [ ] Phase 7 — Dead man switch cron job
- [ ] Phase 8 — Security hardening
- [ ] Phase 9 — Full test suite
- [ ] Phase 10 — Production deployment
- [ ] Multi-sig will execution (future)
- [ ] Email notifications to beneficiaries (future)
- [ ] Mobile app (future)
- [ ] DAO governance for platform upgrades (future)

---

## 🤝 Contributing

Contributions are welcome!

```bash
# 1. Fork the repository
# 2. Create your feature branch
git checkout -b feature/my-new-feature

# 3. Commit your changes
git commit -m "feat: add my new feature"

# 4. Push to your branch
git push origin feature/my-new-feature

# 5. Open a Pull Request
```

### Commit Convention

```
feat:     New feature
fix:      Bug fix
docs:     Documentation changes
style:    Formatting, no logic change
refactor: Code restructuring
test:     Adding or updating tests
chore:    Build process, dependencies
```

---

## ⚠️ Disclaimer

> This software is provided for educational and development purposes.
> Smart contracts handling real assets should be professionally audited before mainnet deployment.
> The authors are not responsible for any loss of funds resulting from the use of this software.

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

Built with ❤️ using Node.js · Solidity · IPFS · MongoDB

**[⬆ Back to top](#-decentralwill--decentralized-will-platform)**

</div>
