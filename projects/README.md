# Projects

Build real-world projects to solidify your Web3 and Solidity skills. Projects are organized by difficulty level.

## 🎯 Project Guidelines

**Before Starting**:
1. Complete relevant modules first
2. Set up development environment
3. Read project requirements carefully
4. Start with tests

**Best Practices**:
- Write tests first (TDD)
- Follow security best practices
- Document your code
- Use version control (git)
- Deploy to testnet before mainnet

---

## 🟢 Beginner Projects

### Project 1: Simple Storage Contract
**Difficulty**: ⭐  
**Module**: 4 - Solidity Fundamentals

**Description**: Create a contract that stores and retrieves a single value.

**Requirements**:
- Store a number
- Function to set the number
- Function to get the number
- Emit event on change
- Access control (only owner can set)

**Learning Goals**:
- Basic contract structure
- State variables
- Functions
- Events
- Modifiers

📁 [View Details](beginner/01-simple-storage/)

---

### Project 2: Basic Token Wallet
**Difficulty**: ⭐⭐  
**Module**: 4 - Solidity Fundamentals

**Description**: Build a wallet that can receive and send ETH.

**Requirements**:
- Receive ETH (payable function)
- Withdraw ETH (with access control)
- Check balance
- Transfer to another address
- Event logging

**Learning Goals**:
- Handling ETH
- Payable functions
- Address types
- Safe transfers

📁 [View Details](beginner/02-token-wallet/)

---

### Project 3: Simple Voting System
**Difficulty**: ⭐⭐  
**Module**: 4 - Solidity Fundamentals

**Description**: Create an on-chain voting system.

**Requirements**:
- Add proposals
- Cast votes
- Prevent double voting
- View results
- Time-limited voting

**Learning Goals**:
- Mappings
- Structs
- Arrays
- Access control
- State management

📁 [View Details](beginner/03-voting-system/)

---

## 🟡 Intermediate Projects

### Project 4: ERC-20 Token
**Difficulty**: ⭐⭐⭐  
**Module**: 5-6 - Smart Contract Development & Advanced Solidity

**Description**: Implement a complete ERC-20 fungible token.

**Requirements**:
- Total supply tracking
- Balance management
- Transfer functionality
- Approve/TransferFrom
- Events (Transfer, Approval)
- Optional: Mint/Burn functions

**Learning Goals**:
- Token standards
- Interface implementation
- Allowance pattern
- Safe math
- Gas optimization

📁 [View Details](intermediate/04-erc20-token/)

---

### Project 5: NFT Marketplace
**Difficulty**: ⭐⭐⭐⭐  
**Module**: 6 - Advanced Solidity

**Description**: Build a marketplace for buying and selling NFTs.

**Requirements**:
- List NFTs for sale
- Buy NFTs
- Cancel listings
- Set prices
- Transfer ownership
- Marketplace fee
- IPFS metadata

**Learning Goals**:
- ERC-721 standard
- Contract interactions
- IPFS integration
- Fee mechanisms
- Security considerations

📁 [View Details](intermediate/05-nft-marketplace/)

---

### Project 6: Multi-Signature Wallet
**Difficulty**: ⭐⭐⭐⭐  
**Module**: 6 - Advanced Solidity

**Description**: Create a wallet requiring multiple signatures.

**Requirements**:
- Multiple owners
- Submit transactions
- Confirm transactions
- Execute after threshold
- Revoke confirmations
- Add/remove owners

**Learning Goals**:
- Multi-party consensus
- Transaction queue
- Security patterns
- Access control
- State management

📁 [View Details](intermediate/06-multisig-wallet/)

---

## 🔴 Advanced Projects

### Project 7: DeFi Lending Protocol
**Difficulty**: ⭐⭐⭐⭐⭐  
**Module**: 6-7 - Advanced Solidity & Security

**Description**: Build a lending and borrowing platform.

**Requirements**:
- Deposit collateral
- Borrow against collateral
- Interest accrual
- Liquidation mechanism
- Oracle price feeds (Chainlink)
- Health factor calculation
- Multiple token support

**Learning Goals**:
- DeFi mechanics
- Oracle integration
- Complex math
- Liquidation logic
- Risk management
- Advanced testing

📁 [View Details](advanced/07-lending-protocol/)

---

### Project 8: Decentralized Exchange (DEX)
**Difficulty**: ⭐⭐⭐⭐⭐  
**Module**: 6-8 - Advanced Solidity & DApp Development

**Description**: Create an AMM-based token exchange.

**Requirements**:
- Liquidity pools
- Token swaps
- Add/remove liquidity
- LP token minting
- Fee mechanism (0.3%)
- Price calculation (x*y=k)
- Frontend integration

**Learning Goals**:
- AMM mechanics
- Liquidity provision
- Price impact
- Slippage protection
- Pool mathematics
- Full-stack DApp

📁 [View Details](advanced/08-dex/)

---

### Project 9: DAO (Decentralized Autonomous Organization)
**Difficulty**: ⭐⭐⭐⭐⭐  
**Module**: 6-8 - Full Pipeline

**Description**: Build a governance system for decentralized decision-making.

**Requirements**:
- Create proposals
- Vote on proposals
- Voting power (token-based)
- Quorum requirements
- Timelock execution
- Delegation
- Treasury management
- Frontend dashboard

**Learning Goals**:
- Governance mechanisms
- Time-locked operations
- Delegation patterns
- Treasury management
- Complex workflows
- Complete DApp architecture

📁 [View Details](advanced/09-dao/)

---

## 📊 Project Comparison

| Project | Difficulty | Duration | Key Skills |
|---------|-----------|----------|------------|
| Simple Storage | ⭐ | 2-4 hours | Basics, functions, events |
| Token Wallet | ⭐⭐ | 4-8 hours | ETH handling, access control |
| Voting System | ⭐⭐ | 1-2 days | Structs, mappings, logic |
| ERC-20 Token | ⭐⭐⭐ | 2-3 days | Standards, interfaces |
| NFT Marketplace | ⭐⭐⭐⭐ | 1 week | NFTs, interactions, IPFS |
| Multi-Sig Wallet | ⭐⭐⭐⭐ | 1 week | Security, consensus |
| Lending Protocol | ⭐⭐⭐⭐⭐ | 2-3 weeks | DeFi, oracles, math |
| DEX | ⭐⭐⭐⭐⭐ | 2-3 weeks | AMM, pools, frontend |
| DAO | ⭐⭐⭐⭐⭐ | 3-4 weeks | Governance, complete system |

---

## 🚀 Getting Started

### Setup Development Environment

**1. Install Dependencies**:
```bash
# Node.js and npm
node --version
npm --version

# Hardhat
npm install --save-dev hardhat
npx hardhat init

# Or Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

**2. Project Structure**:
```
my-project/
├── contracts/
│   └── MyContract.sol
├── test/
│   └── MyContract.test.js
├── scripts/
│   └── deploy.js
├── hardhat.config.js
└── package.json
```

**3. Write Tests First**:
```javascript
describe("MyContract", function() {
  it("Should...", async function() {
    // Test code
  });
});
```

**4. Deploy to Testnet**:
```bash
npx hardhat run scripts/deploy.js --network sepolia
```

---

## 💡 Tips for Success

1. **Start Simple**: Begin with beginner projects
2. **Test Everything**: Write comprehensive tests
3. **Read Code**: Study well-written contracts
4. **Security First**: Follow best practices
5. **Document**: Comment your code
6. **Version Control**: Use git from day one
7. **Ask for Help**: Join communities
8. **Build in Public**: Share your progress

---

## 🤝 Contributing

Have a project idea? Submit a PR with:
- Project description
- Requirements
- Learning goals
- Starter code (optional)
- Tests (optional)

---

## 📖 Additional Resources

### Example Projects
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)
- [Uniswap V2](https://github.com/Uniswap/v2-core)
- [Aave Protocol](https://github.com/aave/aave-v3-core)
- [Compound Finance](https://github.com/compound-finance/compound-protocol)

### Hackathons
- [ETHGlobal](https://ethglobal.com/)
- [Chainlink Hackathons](https://chain.link/hackathon)
- [Gitcoin](https://gitcoin.co/)

### Bounties
- [Gitcoin Bounties](https://gitcoin.co/bounties)
- [Layer3](https://layer3.xyz/)
- [RabbitHole](https://rabbithole.gg/)

---

**Ready to build?** Start with [Project 1: Simple Storage](beginner/01-simple-storage/) →
