# Module 4: Solidity Fundamentals

**Duration**: 3-4 weeks  
**Level**: Intermediate  
**Prerequisites**: Module 3 - Ethereum Ecosystem

## 🎯 Learning Objectives

By the end of this module, you will:
- Write basic Solidity smart contracts
- Understand Solidity syntax and data types
- Work with functions, modifiers, and events
- Handle errors and exceptions properly
- Understand contract structure and best practices
- Deploy and interact with your first contracts

---

## 📚 Lessons

### Lesson 1: Introduction to Solidity

**Overview**:  
Solidity is a statically-typed, contract-oriented programming language for Ethereum smart contracts.

**Key Characteristics**:
- Similar to JavaScript, C++, and Python
- Compiles to EVM bytecode
- Statically typed
- Supports inheritance
- Version-specific (pragma)

**Your First Contract**:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HelloWorld {
    string public greeting = "Hello, Web3!";
    
    function setGreeting(string memory _greeting) public {
        greeting = _greeting;
    }
    
    function getGreeting() public view returns (string memory) {
        return greeting;
    }
}
```

**Contract Structure**:
```solidity
// 1. License Identifier
// SPDX-License-Identifier: MIT

// 2. Pragma Version
pragma solidity ^0.8.0;

// 3. Imports
import "./OtherContract.sol";

// 4. Contract Declaration
contract MyContract {
    // 5. State Variables
    uint256 public myNumber;
    
    // 6. Events
    event NumberChanged(uint256 newNumber);
    
    // 7. Modifiers
    modifier onlyPositive(uint256 _num) {
        require(_num > 0, "Must be positive");
        _;
    }
    
    // 8. Constructor
    constructor(uint256 _initialNumber) {
        myNumber = _initialNumber;
    }
    
    // 9. Functions
    function setNumber(uint256 _num) public onlyPositive(_num) {
        myNumber = _num;
        emit NumberChanged(_num);
    }
}
```

**Development Environment Setup**:

**Using Remix (Browser-based)**:
1. Go to [remix.ethereum.org](https://remix.ethereum.org/)
2. Create new file (.sol)
3. Write contract
4. Compile (Ctrl+S)
5. Deploy & test

**Using Hardhat (Local)**:
```bash
npm install --save-dev hardhat
npx hardhat init
```

---

### Lesson 2: Data Types and Variables

**Overview**:  
Solidity has various data types for different purposes.

**Value Types**:

**1. Boolean**
```solidity
bool public isActive = true;
bool public isComplete = false;

// Operators: !, &&, ||, ==, !=
if (isActive && !isComplete) {
    // Do something
}
```

**2. Integers**
```solidity
// Unsigned (non-negative)
uint8 public smallNumber;    // 0 to 255
uint256 public largeNumber;  // 0 to 2^256-1
uint public defaultUint;     // Same as uint256

// Signed (positive/negative)
int8 public temperature;     // -128 to 127
int256 public balance;       // -(2^255) to 2^255-1

// Operations: +, -, *, /, %, **
uint result = 10 + 5 * 2;   // 20
```

**3. Address**
```solidity
address public owner;
address payable public wallet;  // Can receive ETH

// Address properties
owner.balance;          // ETH balance in wei
owner == msg.sender;    // Comparison

// Payable address methods
wallet.transfer(1 ether);           // Send ETH, throws on failure
wallet.send(1 ether);               // Send ETH, returns bool
wallet.call{value: 1 ether}("");    // Low-level call
```

**4. Fixed-size Byte Arrays**
```solidity
bytes1 public a = 0x01;
bytes32 public hash = keccak256("Hello");

// Properties
a.length;  // Always fixed
```

**5. Enums**
```solidity
enum Status { Pending, Active, Completed, Cancelled }

Status public currentStatus = Status.Pending;

function setStatus(Status _status) public {
    currentStatus = _status;
}

// Check status
if (currentStatus == Status.Active) {
    // Do something
}
```

**Reference Types**:

**1. Arrays**
```solidity
// Fixed-size array
uint[5] public fixedArray;

// Dynamic array
uint[] public dynamicArray;
address[] public users;

// Operations
dynamicArray.push(10);           // Add element
dynamicArray.pop();               // Remove last element
uint length = dynamicArray.length;

// Memory arrays (function scope)
function createArray() public pure returns (uint[] memory) {
    uint[] memory tempArray = new uint[](5);
    tempArray[0] = 1;
    return tempArray;
}
```

**2. Strings**
```solidity
string public name = "Alice";

// String concatenation (Solidity 0.8.12+)
string memory fullName = string.concat(name, " Smith");

// Strings are expensive! Use bytes32 for fixed-length
bytes32 public shortName = "Bob";
```

**3. Structs**
```solidity
struct User {
    address wallet;
    string name;
    uint256 age;
    bool isActive;
}

User public admin;
User[] public users;

// Create struct
User memory newUser = User({
    wallet: msg.sender,
    name: "Alice",
    age: 25,
    isActive: true
});

// Alternative
User memory newUser2 = User(msg.sender, "Bob", 30, true);

users.push(newUser);
```

**4. Mappings**
```solidity
// Key-value pairs
mapping(address => uint256) public balances;
mapping(address => bool) public whitelist;

// Nested mapping
mapping(address => mapping(address => uint256)) public allowances;

// Usage
balances[msg.sender] = 100;
uint256 myBalance = balances[msg.sender];
whitelist[msg.sender] = true;

// Note: Can't iterate over mappings!
// All keys initially map to default value (0, false, etc.)
```

**Data Locations**:

```solidity
contract DataLocations {
    uint[] public storageArray;  // State variable (storage)
    
    function example() public {
        // Storage - persistent, expensive
        uint[] storage arr = storageArray;
        arr.push(1);  // Modifies storageArray
        
        // Memory - temporary, cheaper
        uint[] memory memArr = new uint[](5);
        memArr[0] = 10;  // Only in memory
        
        // Calldata - read-only, cheapest
        // Used for function parameters
    }
    
    function withCalldata(uint[] calldata _data) external {
        // _data is read-only
        // uint x = _data[0];  // OK
        // _data[0] = 10;      // ERROR: Can't modify
    }
}
```

---

### Lesson 3: Functions

**Overview**:  
Functions are the executable units of code within a contract.

**Function Syntax**:
```solidity
function functionName(parameters) visibility mutability returns (returnTypes) {
    // Function body
}
```

**Visibility Modifiers**:

```solidity
contract VisibilityExample {
    uint256 private privateVar = 1;
    uint256 internal internalVar = 2;
    uint256 public publicVar = 3;  // Auto-creates getter
    
    // public: Callable from anywhere
    function publicFunc() public pure returns (string memory) {
        return "Anyone can call";
    }
    
    // external: Only from outside contract
    function externalFunc() external pure returns (string memory) {
        return "Called from outside";
        // this.externalFunc();  // Must use 'this'
    }
    
    // internal: This contract + derived contracts
    function internalFunc() internal pure returns (string memory) {
        return "Internal use";
    }
    
    // private: Only this contract
    function privateFunc() private pure returns (string memory) {
        return "Very private";
    }
}
```

**State Mutability**:

```solidity
contract StateMutability {
    uint256 public data = 10;
    
    // view: Reads state, doesn't modify
    function getData() public view returns (uint256) {
        return data;  // OK to read
        // data = 20;  // ERROR: Can't modify
    }
    
    // pure: Doesn't read or modify state
    function add(uint256 a, uint256 b) public pure returns (uint256) {
        return a + b;  // No state access
        // return data;  // ERROR: Can't read state
    }
    
    // payable: Can receive ETH
    function deposit() public payable {
        // msg.value is available
    }
    
    // No modifier: Can read and modify state
    function setData(uint256 _data) public {
        data = _data;  // Modifies state
    }
}
```

**Function Modifiers**:

```solidity
contract ModifierExample {
    address public owner;
    bool public paused;
    
    constructor() {
        owner = msg.sender;
    }
    
    // Custom modifier
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;  // Continue execution
    }
    
    modifier whenNotPaused() {
        require(!paused, "Contract paused");
        _;
    }
    
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Invalid address");
        _;
    }
    
    // Use modifiers
    function setPaused(bool _paused) public onlyOwner {
        paused = _paused;
    }
    
    // Multiple modifiers
    function doSomething(address _addr) 
        public 
        onlyOwner 
        whenNotPaused 
        validAddress(_addr) 
    {
        // Function logic
    }
}
```

**Special Functions**:

```solidity
contract SpecialFunctions {
    // Constructor: Called once at deployment
    constructor(uint256 _initialValue) {
        // Initialization code
    }
    
    // Receive: Called when contract receives ETH (no data)
    receive() external payable {
        // Handle plain ETH transfers
    }
    
    // Fallback: Called when no function matches or with data
    fallback() external payable {
        // Handle unknown function calls
    }
}
```

**Function Returns**:

```solidity
contract Returns {
    // Single return
    function getNumber() public pure returns (uint256) {
        return 42;
    }
    
    // Multiple returns
    function getMultiple() public pure returns (uint256, bool, address) {
        return (42, true, address(0));
    }
    
    // Named returns
    function getNamedReturns() public pure returns (
        uint256 number,
        bool flag,
        address addr
    ) {
        number = 42;
        flag = true;
        addr = address(0);
        // Implicit return
    }
    
    // Destructuring
    function useReturns() public pure {
        (uint256 num, bool flag, ) = getMultiple();
        // Ignore third return value
    }
}
```

---

### Lesson 4: Control Structures and Operators

**Overview**:  
Control flow and operations in Solidity.

**Conditional Statements**:

```solidity
contract Conditionals {
    function checkValue(uint256 _value) public pure returns (string memory) {
        if (_value > 100) {
            return "Large";
        } else if (_value > 50) {
            return "Medium";
        } else {
            return "Small";
        }
    }
    
    // Ternary operator
    function isEven(uint256 _num) public pure returns (bool) {
        return _num % 2 == 0 ? true : false;
    }
}
```

**Loops**:

```solidity
contract Loops {
    uint[] public numbers;
    
    // For loop
    function sumArray() public view returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < numbers.length; i++) {
            sum += numbers[i];
        }
        return sum;
    }
    
    // While loop
    function countDown(uint256 _start) public pure returns (uint256) {
        uint256 count = _start;
        while (count > 0) {
            count--;
        }
        return count;
    }
    
    // Do-while loop
    function doWhileExample(uint256 _n) public pure returns (uint256) {
        uint256 result = 0;
        do {
            result += _n;
            _n--;
        } while (_n > 0);
        return result;
    }
    
    // WARNING: Avoid unbounded loops! Gas limit can be exceeded
}
```

**Operators**:

```solidity
contract Operators {
    // Arithmetic: +, -, *, /, %, **
    function arithmetic() public pure returns (uint256) {
        uint256 a = 10;
        uint256 b = 3;
        
        uint256 sum = a + b;      // 13
        uint256 diff = a - b;     // 7
        uint256 prod = a * b;     // 30
        uint256 quot = a / b;     // 3 (integer division)
        uint256 rem = a % b;      // 1
        uint256 exp = a ** b;     // 1000
        
        return sum;
    }
    
    // Comparison: ==, !=, <, >, <=, >=
    function compare(uint256 a, uint256 b) public pure returns (bool) {
        return a >= b;
    }
    
    // Logical: !, &&, ||
    function logical(bool a, bool b) public pure returns (bool) {
        return a && b || !a;
    }
    
    // Bitwise: &, |, ^, ~, <<, >>
    function bitwise() public pure returns (uint256) {
        uint256 a = 5;   // 101
        uint256 b = 3;   // 011
        
        uint256 and = a & b;   // 001 = 1
        uint256 or = a | b;    // 111 = 7
        uint256 xor = a ^ b;   // 110 = 6
        
        return xor;
    }
    
    // Assignment: =, +=, -=, *=, /=
    function assignment() public pure returns (uint256) {
        uint256 x = 10;
        x += 5;   // x = x + 5 = 15
        x *= 2;   // x = x * 2 = 30
        return x;
    }
}
```

---

### Lesson 5: Events and Logging

**Overview**:  
Events enable efficient logging for off-chain applications.

**Declaring and Emitting Events**:

```solidity
contract EventExample {
    // Declare events
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event StatusChanged(string oldStatus, string newStatus);
    event LogData(address indexed user, uint256 indexed id, string data);
    
    mapping(address => uint256) public balances;
    
    function transfer(address _to, uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        
        balances[msg.sender] -= _amount;
        balances[_to] += _amount;
        
        // Emit event
        emit Transfer(msg.sender, _to, _amount);
    }
    
    // Up to 3 indexed parameters
    function logAction(uint256 _id, string memory _data) public {
        emit LogData(msg.sender, _id, _data);
    }
}
```

**Indexed Parameters**:
```solidity
// indexed: Can be filtered in queries (up to 3)
// non-indexed: Stored in data, can't filter

event MyEvent(
    address indexed user,      // Filterable
    uint256 indexed itemId,    // Filterable
    uint256 amount,            // Not filterable
    string data                // Not filterable
);
```

**Listening to Events (JavaScript)**:

```javascript
// Using Ethers.js
const contract = new ethers.Contract(address, abi, provider);

// Listen for Transfer events
contract.on("Transfer", (from, to, amount, event) => {
    console.log(`${from} sent ${amount} to ${to}`);
});

// Filter events
const filter = contract.filters.Transfer(myAddress, null);
const events = await contract.queryFilter(filter);

// One-time listener
contract.once("StatusChanged", (oldStatus, newStatus) => {
    console.log(`Status: ${oldStatus} → ${newStatus}`);
});
```

**Why Use Events?**
- Cheaper than storage (logging costs less gas)
- Enable off-chain monitoring
- Create transaction logs
- Notify frontends of changes
- Essential for indexing (The Graph)

---

### Lesson 6: Error Handling

**Overview**:  
Proper error handling prevents unexpected behavior and losses.

**Error Methods**:

```solidity
contract ErrorHandling {
    address public owner;
    uint256 public value;
    
    constructor() {
        owner = msg.sender;
    }
    
    // 1. require(): Input validation, most common
    function setValueRequire(uint256 _value) public {
        require(_value > 0, "Value must be positive");
        require(msg.sender == owner, "Only owner can set");
        value = _value;
    }
    
    // 2. revert(): Complex conditions
    function setValueRevert(uint256 _value) public {
        if (_value == 0) {
            revert("Value cannot be zero");
        }
        if (msg.sender != owner) {
            revert("Not authorized");
        }
        value = _value;
    }
    
    // 3. assert(): Internal errors, invariants
    function setValueAssert(uint256 _value) public {
        value = _value;
        // Check invariant (should never fail)
        assert(value == _value);
    }
    
    // Custom errors (Solidity 0.8.4+)
    error Unauthorized(address caller);
    error InvalidValue(uint256 value, uint256 minimum);
    
    function setValueCustom(uint256 _value) public {
        if (msg.sender != owner) {
            revert Unauthorized(msg.sender);
        }
        if (_value < 10) {
            revert InvalidValue(_value, 10);
        }
        value = _value;
    }
}
```

**Error Comparison**:

| Method | Gas Refund | Use Case |
|--------|------------|----------|
| require() | Yes | Input validation |
| revert() | Yes | Complex conditions |
| assert() | No | Invariants |
| Custom errors | Yes | Gas-efficient |

**Try/Catch**:

```solidity
interface IExternal {
    function mayFail() external returns (uint256);
}

contract TryCatch {
    IExternal external;
    
    function callExternal() public returns (bool success, uint256 result) {
        try external.mayFail() returns (uint256 value) {
            // Success case
            result = value;
            success = true;
        } catch Error(string memory reason) {
            // require/revert with message
            // reason contains error message
            success = false;
        } catch Panic(uint errorCode) {
            // assert failure, overflow, etc.
            success = false;
        } catch (bytes memory lowLevelData) {
            // Other errors
            success = false;
        }
    }
}
```

---

## 🛠️ Hands-On Exercises

### Exercise 1: Simple Storage
```solidity
// Create a contract that:
// - Stores a number
// - Has functions to set and get the number
// - Emits an event when number changes
// - Only allows positive numbers
```

### Exercise 2: User Registry
```solidity
// Create a contract with:
// - Struct for User (address, name, age)
// - Array to store users
// - Mapping for quick lookup
// - Functions to add/get users
// - Events for user actions
```

### Exercise 3: Simple Token
```solidity
// Create a basic token:
// - Track balances (mapping)
// - Transfer function
// - Events for transfers
// - Check sufficient balance
// - Prevent zero transfers
```

### Exercise 4: Access Control
```solidity
// Implement:
// - Owner variable
// - onlyOwner modifier
// - Function to transfer ownership
// - Protected functions
// - Events for ownership changes
```

### Exercise 5: Todo List
```solidity
// Build todo list contract:
// - Task struct (text, completed)
// - Array of tasks
// - Add, complete, delete tasks
// - Events for all actions
// - Only owner can manage
```

---

## ✅ Knowledge Check

1. What's the difference between memory and storage?
2. When would you use `view` vs `pure`?
3. What does the `indexed` keyword do in events?
4. When should you use `require` vs `assert`?
5. What's the purpose of modifiers?
6. How do you create a payable function?
7. What's the difference between `public` and `external`?
8. Why are strings expensive in Solidity?
9. Can you iterate over a mapping?
10. What's the purpose of the constructor?

---

## 📖 Additional Resources

### Documentation
- [Solidity Docs](https://docs.soliditylang.org/)
- [Solidity by Example](https://solidity-by-example.org/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)

### Practice
- [CryptoZombies](https://cryptozombies.io/) - Learn by building game
- [Remix IDE](https://remix.ethereum.org/) - Online development
- [Ethernaut](https://ethernaut.openzeppelin.com/) - Security puzzles

### Videos
- [Solidity Tutorial](https://www.youtube.com/watch?v=ipwxYa-F1uY)
- [Smart Contract Programming](https://www.youtube.com/watch?v=M576WGiDBdQ)

---

## 🎯 Module Completion

Before moving to Module 5, ensure you:
- [ ] Can write basic Solidity contracts
- [ ] Understand all data types
- [ ] Know how to use functions and modifiers
- [ ] Can work with events
- [ ] Understand error handling
- [ ] Can deploy contracts on Remix
- [ ] Have completed practice exercises
- [ ] Understand gas implications of different operations

---

**Previous**: [Module 3: Ethereum Ecosystem](../03-ethereum-ecosystem/)  
**Next**: [Module 5: Smart Contract Development](../05-smart-contract-development/) →
