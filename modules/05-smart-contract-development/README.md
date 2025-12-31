# Module 5: Smart Contract Development

**Duration**: 4-5 weeks  
**Level**: Intermediate  
**Prerequisites**: Module 4 - Solidity Fundamentals

## 🎯 Learning Objectives

- Master contract inheritance and composition
- Work with interfaces and abstract contracts
- Understand libraries and code reusability
- Learn design patterns (Factory, Proxy, etc.)
- Deploy and verify contracts
- Interact with other contracts

## 📚 Key Topics

### 1. Inheritance

```solidity
// Base contract
contract Ownable {
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
}

// Derived contract
contract MyContract is Ownable {
    uint256 public value;
    
    function setValue(uint256 _value) public onlyOwner {
        value = _value;
    }
}

// Multiple inheritance
contract Pausable {
    bool public paused;
    
    modifier whenNotPaused() {
        require(!paused, "Paused");
        _;
    }
}

contract SecureContract is Ownable, Pausable {
    function doSomething() public onlyOwner whenNotPaused {
        // Your logic
    }
}
```

### 2. Interfaces

```solidity
// Define interface
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
}

// Use interface
contract TokenUser {
    function checkBalance(address token, address account) 
        public 
        view 
        returns (uint256) 
    {
        IERC20 tokenContract = IERC20(token);
        return tokenContract.balanceOf(account);
    }
}
```

### 3. Abstract Contracts

```solidity
abstract contract Animal {
    string public name;
    
    constructor(string memory _name) {
        name = _name;
    }
    
    // Must be implemented by derived contracts
    function makeSound() public virtual returns (string memory);
    
    // Can have implementation
    function getName() public view returns (string memory) {
        return name;
    }
}

contract Dog is Animal {
    constructor(string memory _name) Animal(_name) {}
    
    function makeSound() public pure override returns (string memory) {
        return "Woof!";
    }
}
```

### 4. Libraries

```solidity
// Library definition
library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "Addition overflow");
        return c;
    }
}

// Using library
contract Calculator {
    using SafeMath for uint256;
    
    function addNumbers(uint256 a, uint256 b) public pure returns (uint256) {
        return a.add(b);
    }
}
```

### 5. Design Patterns

**Factory Pattern**:
```solidity
contract ChildContract {
    address public owner;
    
    constructor(address _owner) {
        owner = _owner;
    }
}

contract Factory {
    ChildContract[] public children;
    
    event ChildCreated(address childAddress, address owner);
    
    function createChild() public {
        ChildContract child = new ChildContract(msg.sender);
        children.push(child);
        emit ChildCreated(address(child), msg.sender);
    }
}
```

**Storage Patterns**:
```solidity
// Storage vs Memory vs Calldata
contract StoragePatterns {
    struct User {
        string name;
        uint256 age;
    }
    
    User[] public users;
    
    // Storage reference - modifies state
    function updateUser(uint256 index, string memory newName) public {
        User storage user = users[index];
        user.name = newName;
    }
    
    // Memory copy - no state change
    function getUserCopy(uint256 index) public view returns (User memory) {
        return users[index];
    }
    
    // Calldata - read-only, gas efficient
    function processUsers(User[] calldata _users) external pure {
        // Can read but not modify _users
    }
}
```

### 6. Contract Interaction

```solidity
// Call another contract
contract Callee {
    uint256 public value;
    
    function setValue(uint256 _value) public {
        value = _value;
    }
}

contract Caller {
    function callSetValue(address _callee, uint256 _value) public {
        Callee callee = Callee(_callee);
        callee.setValue(_value);
    }
    
    // Low-level call
    function lowLevelCall(address _callee, uint256 _value) public {
        (bool success, ) = _callee.call(
            abi.encodeWithSignature("setValue(uint256)", _value)
        );
        require(success, "Call failed");
    }
}
```

## 🛠️ Projects

### Project 1: ERC20 Token
```solidity
// Implement full ERC20 token with:
// - Total supply
// - Balances
// - Transfer
// - Approve/TransferFrom
// - Events
```

### Project 2: Multi-Signature Wallet
```solidity
// Create multi-sig wallet:
// - Multiple owners
// - Require N confirmations
// - Submit/Confirm/Execute transactions
// - Revoke confirmations
```

### Project 3: NFT Collection
```solidity
// Build ERC721 NFT:
// - Mint function
// - Metadata URI
// - Ownership tracking
// - Transfer functions
```

## 📖 Resources

- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)
- [Smart Contract Patterns](https://fravoll.github.io/solidity-patterns/)
- [Hardhat Documentation](https://hardhat.org/docs)

## ✅ Completion Checklist

- [ ] Understand inheritance and composition
- [ ] Can use interfaces and abstract contracts
- [ ] Know how to create and use libraries
- [ ] Familiar with common design patterns
- [ ] Can deploy contracts with Hardhat
- [ ] Can interact between contracts
- [ ] Built at least one ERC20 token
- [ ] Understand storage patterns

**Previous**: [Module 4: Solidity Fundamentals](../04-solidity-fundamentals/)  
**Next**: [Module 6: Advanced Solidity](../06-advanced-solidity/) →
