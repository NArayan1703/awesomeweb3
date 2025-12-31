# Module 6: Advanced Solidity Concepts

**Duration**: 4-5 weeks  
**Level**: Advanced  
**Prerequisites**: Module 5 - Smart Contract Development

## 🎯 Learning Objectives

- Optimize gas usage
- Use assembly for low-level operations
- Implement upgradeable contracts
- Master token standards (ERC-20, ERC-721, ERC-1155)
- Understand DeFi protocol basics

## 📚 Key Topics

### 1. Gas Optimization

**Techniques**:
```solidity
contract GasOptimization {
    // ❌ Expensive
    string public name = "MyToken";
    
    // ✅ Cheaper (if <= 32 bytes)
    bytes32 public name = "MyToken";
    
    // ❌ Expensive
    uint8 public a;
    uint8 public b;
    
    // ✅ Packed storage (same slot)
    uint8 public a;
    uint8 public b;
    
    // Use unchecked for safe operations (Solidity 0.8+)
    function optimizedLoop(uint256 n) public pure returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < n;) {
            sum += i;
            unchecked { i++; }  // Skip overflow check
        }
        return sum;
    }
    
    // Cache array length
    function sumArray(uint256[] memory arr) public pure returns (uint256) {
        uint256 sum = 0;
        uint256 len = arr.length;  // Cache length
        for (uint256 i = 0; i < len; i++) {
            sum += arr[i];
        }
        return sum;
    }
}
```

**Best Practices**:
- Use `uint256` (native word size)
- Pack storage variables
- Use events instead of storage
- Minimize storage operations
- Use `memory` for temporary data
- Batch operations when possible

### 2. Assembly (Yul)

```solidity
contract AssemblyExample {
    function getCodeSize(address _addr) public view returns (uint256 size) {
        assembly {
            size := extcodesize(_addr)
        }
    }
    
    function efficientSum(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := add(a, b)
        }
    }
    
    // Direct memory access
    function memoryManipulation() public pure returns (bytes32 x) {
        assembly {
            let freeMem := mload(0x40)
            x := mload(freeMem)
        }
    }
}
```

**When to Use Assembly**:
- Critical gas optimization
- Low-level operations
- Custom cryptography
- When you know what you're doing (dangerous!)

### 3. Upgradeable Contracts

**Proxy Pattern**:
```solidity
// Storage contract
contract StorageContract {
    uint256 public value;
    address public implementation;
}

// Proxy contract
contract Proxy is StorageContract {
    constructor(address _implementation) {
        implementation = _implementation;
    }
    
    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}

// Implementation V1
contract ImplementationV1 is StorageContract {
    function setValue(uint256 _value) public {
        value = _value;
    }
}

// Implementation V2 (upgraded)
contract ImplementationV2 is StorageContract {
    function setValue(uint256 _value) public {
        value = _value * 2;  // New logic
    }
}
```

**OpenZeppelin Upgradeable**:
```solidity
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract MyContractV1 is Initializable {
    uint256 public value;
    
    function initialize(uint256 _value) public initializer {
        value = _value;
    }
}
```

### 4. ERC-20 Token Standard

```solidity
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

contract MyToken is IERC20 {
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    uint256 private _totalSupply;
    string public name = "MyToken";
    string public symbol = "MTK";
    uint8 public decimals = 18;
    
    constructor(uint256 initialSupply) {
        _totalSupply = initialSupply;
        _balances[msg.sender] = initialSupply;
    }
    
    function totalSupply() external view returns (uint256) {
        return _totalSupply;
    }
    
    function balanceOf(address account) external view returns (uint256) {
        return _balances[account];
    }
    
    function transfer(address to, uint256 amount) external returns (bool) {
        require(_balances[msg.sender] >= amount, "Insufficient balance");
        _balances[msg.sender] -= amount;
        _balances[to] += amount;
        emit Transfer(msg.sender, to, amount);
        return true;
    }
    
    // ... implement other functions
}
```

### 5. ERC-721 NFT Standard

```solidity
interface IERC721 {
    function balanceOf(address owner) external view returns (uint256);
    function ownerOf(uint256 tokenId) external view returns (address);
    function transferFrom(address from, address to, uint256 tokenId) external;
    function approve(address to, uint256 tokenId) external;
    
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
}

contract MyNFT is IERC721 {
    mapping(uint256 => address) private _owners;
    mapping(address => uint256) private _balances;
    mapping(uint256 => string) private _tokenURIs;
    uint256 private _tokenIdCounter;
    
    function mint(address to, string memory uri) public returns (uint256) {
        uint256 tokenId = _tokenIdCounter++;
        _owners[tokenId] = to;
        _balances[to]++;
        _tokenURIs[tokenId] = uri;
        emit Transfer(address(0), to, tokenId);
        return tokenId;
    }
    
    function ownerOf(uint256 tokenId) external view returns (address) {
        return _owners[tokenId];
    }
    
    function tokenURI(uint256 tokenId) public view returns (string memory) {
        return _tokenURIs[tokenId];
    }
    
    // ... implement other functions
}
```

### 6. DeFi Basics

**Simple AMM (Automated Market Maker)**:
```solidity
contract SimpleAMM {
    IERC20 public tokenA;
    IERC20 public tokenB;
    uint256 public reserveA;
    uint256 public reserveB;
    
    // x * y = k (constant product formula)
    function swap(address tokenIn, uint256 amountIn) external {
        require(tokenIn == address(tokenA) || tokenIn == address(tokenB), "Invalid token");
        require(amountIn > 0, "Amount must be positive");
        
        bool isTokenA = tokenIn == address(tokenA);
        (IERC20 tokenInContract, IERC20 tokenOutContract, uint256 reserveIn, uint256 reserveOut) 
            = isTokenA 
                ? (tokenA, tokenB, reserveA, reserveB)
                : (tokenB, tokenA, reserveB, reserveA);
        
        tokenInContract.transferFrom(msg.sender, address(this), amountIn);
        
        // Using constant product formula with fee (0.3%)
        // Note: In production, use more robust math with slippage protection
        uint256 amountInWithFee = amountIn * 997;
        uint256 numerator = amountInWithFee * reserveOut;
        uint256 denominator = (reserveIn * 1000) + amountInWithFee;
        uint256 amountOut = numerator / denominator;
        
        require(amountOut > 0, "Insufficient output amount");
        tokenOutContract.transfer(msg.sender, amountOut);
        
        // Update reserves
        _updateReserves();
    }
    
    function _updateReserves() private {
        reserveA = tokenA.balanceOf(address(this));
        reserveB = tokenB.balanceOf(address(this));
    }
}
```

**Staking Contract**:
```solidity
contract Staking {
    IERC20 public stakingToken;
    IERC20 public rewardToken;
    
    mapping(address => uint256) public stakedBalance;
    mapping(address => uint256) public rewardDebt;
    
    uint256 public rewardPerToken;
    
    function stake(uint256 amount) external {
        updateReward(msg.sender);
        stakingToken.transferFrom(msg.sender, address(this), amount);
        stakedBalance[msg.sender] += amount;
    }
    
    function withdraw(uint256 amount) external {
        updateReward(msg.sender);
        stakedBalance[msg.sender] -= amount;
        stakingToken.transfer(msg.sender, amount);
    }
    
    function claimReward() external {
        updateReward(msg.sender);
        uint256 reward = calculateReward(msg.sender);
        rewardToken.transfer(msg.sender, reward);
        rewardDebt[msg.sender] = stakedBalance[msg.sender] * rewardPerToken;
    }
    
    function calculateReward(address user) public view returns (uint256) {
        return (stakedBalance[user] * rewardPerToken) - rewardDebt[user];
    }
    
    function updateReward(address user) internal {
        // Update reward calculation logic
    }
}
```

## 🛠️ Projects

### Project 1: DEX (Decentralized Exchange)
- Token swap with AMM
- Liquidity pools
- LP tokens
- Fee mechanism

### Project 2: NFT Marketplace
- List NFTs for sale
- Buy/sell functionality
- Royalties
- Auction mechanism

### Project 3: Lending Protocol
- Deposit collateral
- Borrow assets
- Interest calculation
- Liquidations

## 📖 Resources

- [EIP Standards](https://eips.ethereum.org/)
- [DeFi Developer Roadmap](https://github.com/OffcierCia/DeFi-Developer-Road-Map)
- [OpenZeppelin Upgrades](https://docs.openzeppelin.com/upgrades)

## ✅ Completion Checklist

- [ ] Can optimize gas efficiently
- [ ] Understand assembly basics
- [ ] Know upgradeable patterns
- [ ] Implemented ERC-20 token
- [ ] Implemented ERC-721 NFT
- [ ] Built simple DeFi protocol
- [ ] Understand AMM mechanics
- [ ] Know common DeFi patterns

**Previous**: [Module 5: Smart Contract Development](../05-smart-contract-development/)  
**Next**: [Module 7: Testing and Security](../07-testing-security/) →
