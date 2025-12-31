# Module 8: DApp Development

**Duration**: 4-6 weeks  
**Level**: Advanced  
**Prerequisites**: Module 7 - Testing and Security

## 🎯 Learning Objectives

- Build full-stack decentralized applications
- Integrate smart contracts with frontends
- Use Web3 libraries in React applications
- Implement wallet connections
- Work with decentralized storage (IPFS)
- Deploy DApps to production

## 📚 Key Topics

### 1. Frontend Setup

**Tech Stack**:
```bash
# Create React app with Vite
npm create vite@latest my-dapp -- --template react

# Install dependencies
npm install ethers wagmi viem @rainbow-me/rainbowkit
npm install @tanstack/react-query
```

**Project Structure**:
```
my-dapp/
├── contracts/          # Smart contracts
│   ├── MyContract.sol
│   └── hardhat.config.js
├── src/
│   ├── components/     # React components
│   ├── hooks/          # Custom hooks
│   ├── utils/          # Utilities
│   ├── abis/           # Contract ABIs
│   └── App.jsx
├── public/
└── package.json
```

### 2. Wallet Connection

**Using RainbowKit**:
```javascript
// src/main.jsx
import '@rainbow-me/rainbowkit/styles.css';
import { getDefaultWallets, RainbowKitProvider } from '@rainbow-me/rainbowkit';
import { configureChains, createConfig, WagmiConfig } from 'wagmi';
import { mainnet, polygon, optimism, arbitrum, sepolia } from 'wagmi/chains';
import { publicProvider } from 'wagmi/providers/public';

const { chains, publicClient } = configureChains(
  [mainnet, polygon, optimism, arbitrum, sepolia],
  [publicProvider()]
);

const { connectors } = getDefaultWallets({
  appName: 'My DApp',
  projectId: 'YOUR_PROJECT_ID',
  chains
});

const wagmiConfig = createConfig({
  autoConnect: true,
  connectors,
  publicClient
});

function App() {
  return (
    <WagmiConfig config={wagmiConfig}>
      <RainbowKitProvider chains={chains}>
        <YourApp />
      </RainbowKitProvider>
    </WagmiConfig>
  );
}
```

**Connect Button**:
```javascript
import { ConnectButton } from '@rainbow-me/rainbowkit';

function Navbar() {
  return (
    <nav>
      <h1>My DApp</h1>
      <ConnectButton />
    </nav>
  );
}
```

### 3. Contract Interaction

**Reading Contract Data**:
```javascript
import { useContractRead } from 'wagmi';

function TokenBalance() {
  const { data: balance, isLoading } = useContractRead({
    address: '0x...',
    abi: tokenABI,
    functionName: 'balanceOf',
    args: [userAddress],
    watch: true, // Re-fetch on new blocks
  });

  if (isLoading) return <div>Loading...</div>;
  
  return <div>Balance: {balance?.toString()}</div>;
}
```

**Writing to Contract**:
```javascript
import { useContractWrite, usePrepareContractWrite, useWaitForTransaction } from 'wagmi';

function TransferToken() {
  const [to, setTo] = useState('');
  const [amount, setAmount] = useState('');
  const [error, setError] = useState('');

  // Validate inputs
  const isValidAddress = ethers.isAddress(to);
  const isValidAmount = amount && !isNaN(amount) && parseFloat(amount) > 0;

  const { config } = usePrepareContractWrite({
    address: '0x...',
    abi: tokenABI,
    functionName: 'transfer',
    args: [to, isValidAmount ? ethers.parseEther(amount) : 0],
    enabled: isValidAddress && isValidAmount,
  });

  const { write, data } = useContractWrite({
    ...config,
    onError(error) {
      setError(error.message);
    }
  });
  
  const { isLoading, isSuccess } = useWaitForTransaction({
    hash: data?.hash,
  });

  return (
    <div>
      <input 
        placeholder="Recipient" 
        value={to} 
        onChange={(e) => setTo(e.target.value)} 
      />
      {to && !isValidAddress && <div className="error">Invalid address</div>}
      <input 
        placeholder="Amount" 
        value={amount} 
        onChange={(e) => setAmount(e.target.value)} 
      />
      {amount && !isValidAmount && <div className="error">Invalid amount</div>}
      <button 
        onClick={() => write()} 
        disabled={!isValidAddress || !isValidAmount || isLoading}
      >
        {isLoading ? 'Sending...' : 'Transfer'}
      </button>
      {error && <div className="error">{error}</div>}
      {isSuccess && <div>Transaction successful!</div>}
    </div>
  );
}
```

**Custom Hooks**:
```javascript
// hooks/useTokenBalance.js
import { useContractRead } from 'wagmi';

export function useTokenBalance(tokenAddress, userAddress) {
  return useContractRead({
    address: tokenAddress,
    abi: ERC20_ABI,
    functionName: 'balanceOf',
    args: [userAddress],
    watch: true,
  });
}

// Usage
function Component() {
  const { data: balance } = useTokenBalance(TOKEN_ADDRESS, userAddress);
  return <div>{ethers.formatEther(balance || 0)} tokens</div>;
}
```

### 4. Event Listening

**Listen to Contract Events**:
```javascript
import { useContractEvent } from 'wagmi';

function TransferListener() {
  const [transfers, setTransfers] = useState([]);

  useContractEvent({
    address: '0x...',
    abi: tokenABI,
    eventName: 'Transfer',
    listener(log) {
      console.log('Transfer event:', log);
      setTransfers(prev => [...prev, log]);
    },
  });

  return (
    <div>
      <h3>Recent Transfers</h3>
      {transfers.map((transfer, i) => (
        <div key={i}>
          {transfer.args.from} → {transfer.args.to}: {transfer.args.value.toString()}
        </div>
      ))}
    </div>
  );
}
```

**Query Past Events**:
```javascript
import { usePublicClient } from 'wagmi';
import { useEffect, useState } from 'react';

function PastEvents() {
  const [events, setEvents] = useState([]);
  const publicClient = usePublicClient();

  useEffect(() => {
    const fetchEvents = async () => {
      const logs = await publicClient.getLogs({
        address: '0x...',
        event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
        fromBlock: 'earliest',
        toBlock: 'latest'
      });
      setEvents(logs);
    };
    
    fetchEvents();
  }, []);

  return <div>{/* Display events */}</div>;
}
```

### 5. IPFS Integration

**Upload to IPFS**:
```javascript
import { create } from 'ipfs-http-client';

// Connect to IPFS node
const ipfs = create({ 
  host: 'ipfs.infura.io', 
  port: 5001, 
  protocol: 'https' 
});

async function uploadToIPFS(file) {
  try {
    const added = await ipfs.add(file);
    const url = `https://ipfs.io/ipfs/${added.path}`;
    return url;
  } catch (error) {
    console.error('Error uploading to IPFS:', error);
  }
}

// React component
function IPFSUpload() {
  const [file, setFile] = useState(null);
  const [ipfsUrl, setIpfsUrl] = useState('');

  const handleUpload = async () => {
    if (!file) return;
    const url = await uploadToIPFS(file);
    setIpfsUrl(url);
  };

  return (
    <div>
      <input 
        type="file" 
        onChange={(e) => setFile(e.target.files[0])} 
      />
      <button onClick={handleUpload}>Upload to IPFS</button>
      {ipfsUrl && <img src={ipfsUrl} alt="Uploaded" />}
    </div>
  );
}
```

### 6. The Graph Integration

**GraphQL Queries**:
```javascript
import { request, gql } from 'graphql-request';

const endpoint = 'https://api.thegraph.com/subgraphs/name/...';

const query = gql`
  query GetTokens($first: Int!) {
    tokens(first: $first, orderBy: totalSupply, orderDirection: desc) {
      id
      name
      symbol
      totalSupply
    }
  }
`;

function TokenList() {
  const [tokens, setTokens] = useState([]);

  useEffect(() => {
    request(endpoint, query, { first: 10 })
      .then((data) => setTokens(data.tokens))
      .catch(console.error);
  }, []);

  return (
    <div>
      {tokens.map(token => (
        <div key={token.id}>
          {token.name} ({token.symbol}) - {token.totalSupply}
        </div>
      ))}
    </div>
  );
}
```

### 7. Error Handling

**Transaction Error Handling**:
```javascript
function SafeTransfer() {
  const [error, setError] = useState(null);

  const { write } = useContractWrite({
    ...config,
    onError(error) {
      if (error.message.includes('insufficient funds')) {
        setError('Insufficient funds for transaction');
      } else if (error.message.includes('user rejected')) {
        setError('Transaction rejected by user');
      } else {
        setError('Transaction failed: ' + error.message);
      }
    },
    onSuccess() {
      setError(null);
      alert('Transaction successful!');
    }
  });

  return (
    <div>
      <button onClick={() => write()}>Transfer</button>
      {error && <div className="error">{error}</div>}
    </div>
  );
}
```

### 8. Deployment

**Smart Contract Deployment**:
```javascript
// hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    },
    mainnet: {
      url: process.env.MAINNET_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};

// Deploy script
npx hardhat run scripts/deploy.js --network sepolia
npx hardhat verify --network sepolia DEPLOYED_CONTRACT_ADDRESS
```

**Frontend Deployment**:
```bash
# Build frontend
npm run build

# Deploy to Vercel
npm install -g vercel
vercel

# Deploy to Netlify
npm install -g netlify-cli
netlify deploy --prod

# Deploy to IPFS (decentralized hosting)
ipfs add -r dist/
```

## 🛠️ Complete DApp Example

**Simple NFT Minter DApp**:
```javascript
// NFTMinter.jsx
import { useState } from 'react';
import { useContractWrite, usePrepareContractWrite } from 'wagmi';

function NFTMinter() {
  const [uri, setUri] = useState('');

  const { config } = usePrepareContractWrite({
    address: NFT_CONTRACT_ADDRESS,
    abi: NFT_ABI,
    functionName: 'mint',
    args: [uri],
  });

  const { write, isLoading, isSuccess } = useContractWrite(config);

  return (
    <div className="minter">
      <h2>Mint Your NFT</h2>
      <input 
        placeholder="IPFS URI" 
        value={uri}
        onChange={(e) => setUri(e.target.value)}
      />
      <button onClick={() => write()} disabled={!uri || isLoading}>
        {isLoading ? 'Minting...' : 'Mint NFT'}
      </button>
      {isSuccess && <p>NFT minted successfully!</p>}
    </div>
  );
}
```

## 📖 Resources

### Libraries
- [Wagmi](https://wagmi.sh/) - React hooks for Ethereum
- [RainbowKit](https://www.rainbowkit.com/) - Wallet connection
- [Ethers.js](https://docs.ethers.org/) - Ethereum library
- [IPFS](https://ipfs.tech/) - Decentralized storage
- [The Graph](https://thegraph.com/) - Indexing protocol

### Deployment
- [Vercel](https://vercel.com/) - Frontend hosting
- [Netlify](https://www.netlify.com/) - Frontend hosting
- [Fleek](https://fleek.co/) - IPFS hosting
- [Alchemy](https://www.alchemy.com/) - Node provider
- [Infura](https://infura.io/) - Node provider

## ✅ Completion Checklist

- [ ] Built React app with Web3 integration
- [ ] Implemented wallet connection
- [ ] Can read from smart contracts
- [ ] Can write to smart contracts
- [ ] Implemented event listening
- [ ] Used IPFS for storage
- [ ] Integrated The Graph
- [ ] Deployed DApp to testnet
- [ ] Deployed frontend to production
- [ ] Built complete end-to-end DApp

**Previous**: [Module 7: Testing and Security](../07-testing-security/)  
**Next**: [Projects](../../projects/) →

---

## 🎉 Congratulations!

You've completed the Web3 to Solidity Learning Pipeline! You now have the skills to:
- Build secure smart contracts
- Create full-stack DApps
- Deploy to production
- Follow best practices

**What's Next?**
1. Build your own projects
2. Contribute to open source
3. Join hackathons
4. Get your contracts audited
5. Keep learning and stay updated!
