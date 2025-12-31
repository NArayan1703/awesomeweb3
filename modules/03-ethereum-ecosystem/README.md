# Module 3: Ethereum Ecosystem

**Duration**: 2-3 weeks  
**Level**: Beginner to Intermediate  
**Prerequisites**: Module 2 - Blockchain Basics

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand Ethereum architecture and design
- Learn about accounts, transactions, and gas
- Grasp the Ethereum Virtual Machine (EVM)
- Know how to interact with Ethereum using Web3 libraries
- Set up and use MetaMask for development
- Understand different Ethereum networks

---

## 📚 Lessons

### Lesson 1: Introduction to Ethereum

**Overview**:  
Ethereum is a programmable blockchain platform that enables smart contracts and decentralized applications.

**Key Features**:
- **Smart Contracts**: Self-executing code on blockchain
- **Turing Complete**: Can run any computation
- **EVM**: Decentralized world computer
- **Native Currency**: Ether (ETH)
- **Token Standards**: ERC-20, ERC-721, ERC-1155

**Ethereum vs Bitcoin**:

| Feature | Bitcoin | Ethereum |
|---------|---------|----------|
| Purpose | Digital currency | Programmable platform |
| Scripting | Limited | Turing-complete |
| Block Time | ~10 minutes | ~12 seconds |
| Consensus | PoW → PoS planned | PoS (The Merge) |
| Use Cases | Store of value | DApps, DeFi, NFTs |

**Ethereum's Vision**:
- Decentralized world computer
- Programmable money
- Global settlement layer
- Foundation for Web3

---

### Lesson 2: Ethereum Accounts

**Overview**:  
Ethereum has two types of accounts with different capabilities.

**1. Externally Owned Accounts (EOA)**

**Characteristics**:
- Controlled by private keys
- Can send transactions
- No code associated
- Created by users (wallets)

**Structure**:
```
EOA {
  address: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
  balance: 5.25 ETH
  nonce: 42
  // No code
}
```

**Capabilities**:
- Send ETH to other accounts
- Call smart contract functions
- Deploy smart contracts
- Sign messages

**2. Contract Accounts**

**Characteristics**:
- Controlled by smart contract code
- Cannot initiate transactions
- Has associated code
- Created by deployment transaction

**Structure**:
```
Contract Account {
  address: "0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984"
  balance: 100 ETH
  nonce: 1
  code: "0x606060..." (bytecode)
  storage: {...} (state variables)
}
```

**Capabilities**:
- Execute code when called
- Store data (state)
- Send ETH to other accounts
- Create other contracts

**Account Comparison**:

| Feature | EOA | Contract |
|---------|-----|----------|
| Has Private Key | ✓ | ✗ |
| Can Initiate Tx | ✓ | ✗ |
| Has Code | ✗ | ✓ |
| Has Storage | ✗ | ✓ |
| Creation Cost | Free | Gas required |

---

### Lesson 3: Gas and Transaction Fees

**Overview**:  
Gas is the fuel for Ethereum transactions, preventing spam and infinite loops.

**What is Gas?**
- Unit of computational work
- Measured in gas units
- Paid in ETH (gwei)
- Varies by operation complexity

**Gas Terminology**:

1. **Gas Limit**
   - Maximum gas willing to spend
   - Set by transaction sender
   - Simple transfer: 21,000 gas
   - Complex contracts: varies

2. **Gas Price** (Legacy)
   - Amount in gwei per gas unit
   - Higher price = faster inclusion
   - Determined by network demand

3. **EIP-1559 (Current)**
   - Base Fee: Minimum required (burned)
   - Priority Fee (Tip): Goes to validators
   - Max Fee: Maximum willing to pay

**Gas Calculation**:

```
Legacy:
Transaction Fee = Gas Used × Gas Price

EIP-1559:
Transaction Fee = Gas Used × (Base Fee + Priority Fee)
Max Fee Paid = Gas Limit × Max Fee Per Gas
```

**Example**:
```
Transaction Details:
Gas Limit: 50,000
Gas Used: 45,000
Base Fee: 20 gwei
Priority Fee: 2 gwei
Max Fee: 50 gwei

Actual Cost:
45,000 × (20 + 2) = 990,000 gwei = 0.00099 ETH

Max Would Pay:
50,000 × 50 = 2,500,000 gwei = 0.0025 ETH
```

**Gas Optimization Tips**:
- Use efficient algorithms
- Minimize storage operations
- Batch transactions
- Use events instead of storage
- Leverage libraries (OpenZeppelin)

**Gas Costs by Operation**:
```
ADD/SUB: 3 gas
MUL/DIV: 5 gas
SLOAD (read storage): 2100 gas
SSTORE (write storage): 20,000 gas
CREATE (deploy contract): 32,000 gas
Transaction base: 21,000 gas
```

---

### Lesson 4: Ethereum Virtual Machine (EVM)

**Overview**:  
The EVM is a decentralized computer that executes smart contract code.

**EVM Characteristics**:
- Stack-based architecture
- Deterministic execution
- Isolated (sandboxed) environment
- Turing-complete
- Every node runs same code

**How EVM Works**:

```
1. Smart Contract (Solidity)
         ↓
2. Compiler (solc)
         ↓
3. Bytecode (EVM instructions)
         ↓
4. Deployment to blockchain
         ↓
5. Execution by EVM on each node
```

**EVM Components**:

1. **Stack**
   - 1024 item limit
   - Each item 256 bits
   - Main computation area

2. **Memory**
   - Volatile (cleared after execution)
   - Linear, byte-addressable
   - Expands as needed (costs gas)

3. **Storage**
   - Persistent key-value store
   - Part of account state
   - Expensive to write

4. **Program Counter**
   - Points to current instruction
   - Controls execution flow

**EVM Opcodes**:
```
PUSH1 0x60    // Push value onto stack
PUSH1 0x40    
MSTORE        // Store in memory
CALLDATALOAD  // Load from call data
ADD           // Add top two stack items
SSTORE        // Store to storage
RETURN        // Return from execution
```

**EVM Execution Context**:
```
Context {
  msg.sender: caller's address
  msg.value: ETH sent with call
  msg.data: input data
  tx.origin: original transaction sender
  block.number: current block
  block.timestamp: current time
  gasleft(): remaining gas
}
```

**EVM Compatibility**:
- Other chains use EVM (BSC, Polygon, Avalanche)
- Allows easy smart contract portability
- Shared tooling and standards
- Large developer ecosystem

---

### Lesson 5: Web3 Libraries

**Overview**:  
Web3 libraries enable applications to interact with Ethereum.

**Popular Libraries**:

**1. Web3.js**
- Original Web3 library
- Comprehensive API
- Larger bundle size

**2. Ethers.js**
- Modern, lightweight
- Better TypeScript support
- Recommended for new projects

**3. Web3.py**
- Python implementation
- Backend and scripting

**Core Functionality**:
- Connect to Ethereum nodes
- Query blockchain data
- Send transactions
- Interact with contracts
- Sign messages
- Manage accounts

**Ethers.js Example**:

```javascript
// Import library
const { ethers } = require("ethers");

// Connect to network
const provider = new ethers.JsonRpcProvider(
  "https://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY"
);

// Get block number
const blockNumber = await provider.getBlockNumber();
console.log("Current block:", blockNumber);

// Get account balance
const balance = await provider.getBalance(
  "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
);
console.log("Balance:", ethers.formatEther(balance), "ETH");

// Connect wallet (with private key)
const wallet = new ethers.Wallet(privateKey, provider);

// Send transaction
const tx = await wallet.sendTransaction({
  to: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  value: ethers.parseEther("0.1")
});
await tx.wait();
console.log("Transaction hash:", tx.hash);

// Interact with contract
const abi = ["function balanceOf(address) view returns (uint)"];
const contract = new ethers.Contract(contractAddress, abi, provider);
const tokenBalance = await contract.balanceOf(walletAddress);
```

**Key Concepts**:

1. **Providers**
   - Connection to Ethereum network
   - Read-only access
   - Query blockchain state

2. **Signers**
   - Can sign transactions
   - Send transactions
   - Requires private key

3. **Contracts**
   - JavaScript representation of smart contract
   - ABI (Application Binary Interface) required
   - Call functions and read state

---

### Lesson 6: Ethereum Networks and Tools

**Overview**:  
Multiple networks exist for development, testing, and production.

**Network Types**:

**1. Mainnet**
- Production Ethereum network
- Real ETH with real value
- Permanent and immutable
- High gas costs

**2. Testnets**
Purpose: Test contracts without real money

**Popular Testnets**:
- **Sepolia**: Recommended for testing (PoS)
- **Goerli**: Being deprecated (PoS)
- **Holesky**: For validators/staking tests

**Testnet Characteristics**:
- Free test ETH from faucets
- Similar to mainnet
- May be reset periodically
- Lower security guarantees

**3. Local Networks**
- **Hardhat Network**: Built-in to Hardhat
- **Ganache**: Local blockchain simulator
- **Anvil**: Part of Foundry
- Instant mining, full control

**Getting Testnet ETH**:
```
Faucets:
- https://sepoliafaucet.com/
- https://faucet.quicknode.com/
- https://www.alchemy.com/faucets
```

**Network Configuration in MetaMask**:

```javascript
// Sepolia Testnet
Network Name: Sepolia
RPC URL: https://sepolia.infura.io/v3/YOUR-KEY
Chain ID: 11155111
Currency Symbol: ETH
Block Explorer: https://sepolia.etherscan.io

// Polygon Mainnet
Network Name: Polygon
RPC URL: https://polygon-rpc.com
Chain ID: 137
Currency Symbol: MATIC
Block Explorer: https://polygonscan.com
```

**Essential Tools**:

**Block Explorers**:
- [Etherscan](https://etherscan.io/) - Ethereum mainnet
- [Sepolia Etherscan](https://sepolia.etherscan.io/) - Testnet

**Node Providers**:
- [Alchemy](https://www.alchemy.com/)
- [Infura](https://infura.io/)
- [QuickNode](https://www.quicknode.com/)

**Development Tools**:
- [Hardhat](https://hardhat.org/) - Development environment
- [Remix](https://remix.ethereum.org/) - Online IDE
- [Foundry](https://book.getfoundry.sh/) - Rust-based toolkit

**Wallets**:
- [MetaMask](https://metamask.io/) - Browser extension
- [Rainbow](https://rainbow.me/) - Mobile wallet
- [Frame](https://frame.sh/) - Desktop wallet

---

## 🛠️ Hands-On Exercises

### Exercise 1: MetaMask Setup for Development
1. Install MetaMask
2. Add Sepolia testnet
3. Get testnet ETH from faucet
4. Send a test transaction
5. View transaction on Etherscan

### Exercise 2: Web3 Library Basics
Create a simple script:
```javascript
// Connect to Sepolia
// Check your balance
// Get current gas price
// Query a recent block
// Look up a transaction
```

### Exercise 3: Account Exploration
1. Create new EOA (wallet)
2. Deploy simple contract (Remix)
3. Compare EOA vs Contract account on Etherscan
4. Analyze their differences

### Exercise 4: Gas Estimation
1. Estimate gas for ETH transfer
2. Estimate gas for token transfer
3. Compare gas costs of different operations
4. Calculate transaction fees

### Exercise 5: Network Interaction
```javascript
// Write a script that:
// 1. Connects to multiple networks
// 2. Queries balances on each
// 3. Compares block times
// 4. Checks gas prices
```

---

## ✅ Knowledge Check

Test your understanding:

1. What are the two types of Ethereum accounts?
2. How is gas different from ETH?
3. What is the EVM and what does it do?
4. Why do we need testnets?
5. What's the difference between a provider and a signer?
6. How does EIP-1559 change gas fees?
7. What is an ABI used for?
8. Name three EVM-compatible chains
9. What's the base gas cost for a simple ETH transfer?
10. How do you get testnet ETH?

---

## 📖 Additional Resources

### Documentation
- [Ethereum.org Developer Docs](https://ethereum.org/en/developers/docs/)
- [Ethers.js Documentation](https://docs.ethers.org/)
- [Web3.js Documentation](https://web3js.readthedocs.io/)
- [EVM Opcodes](https://www.evm.codes/)

### Tools
- [Ethereum JSON-RPC](https://ethereum.org/en/developers/docs/apis/json-rpc/)
- [Gas Tracker](https://etherscan.io/gastracker)
- [EVM Playground](https://www.evm.codes/playground)

### Videos
- [Ethereum Explained](https://www.youtube.com/watch?v=jxLkbJozKbY)
- [How Ethereum Works](https://www.youtube.com/watch?v=5BNrLWNFiK4)
- [Gas and Fees Explained](https://www.youtube.com/watch?v=AJvzNICwcwc)

---

## 🎯 Module Completion

Before moving to Module 4, ensure you:
- [ ] Understand Ethereum's architecture
- [ ] Know the difference between EOA and Contract accounts
- [ ] Can explain gas and transaction fees
- [ ] Understand how the EVM works
- [ ] Have set up MetaMask with testnets
- [ ] Can use Web3 libraries (Ethers.js)
- [ ] Know how to use block explorers
- [ ] Have gotten testnet ETH and sent transactions
- [ ] Understand EVM compatibility

---

**Previous**: [Module 2: Blockchain Basics](../02-blockchain-basics/)  
**Next**: [Module 4: Solidity Fundamentals](../04-solidity-fundamentals/) →
