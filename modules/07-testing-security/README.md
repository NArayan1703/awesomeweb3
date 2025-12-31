# Module 7: Testing and Security

**Duration**: 3-4 weeks  
**Level**: Intermediate to Advanced  
**Prerequisites**: Module 6 - Advanced Solidity

## 🎯 Learning Objectives

- Write comprehensive smart contract tests
- Understand common security vulnerabilities
- Apply security best practices
- Use auditing tools and techniques
- Implement secure coding patterns

## 📚 Key Topics

### 1. Testing with Hardhat

**Setup**:
```bash
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat init
```

**Test Structure**:
```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("MyContract", function () {
    let myContract;
    let owner;
    let addr1;
    
    beforeEach(async function () {
        [owner, addr1] = await ethers.getSigners();
        
        const MyContract = await ethers.getContractFactory("MyContract");
        myContract = await MyContract.deploy();
    });
    
    describe("Deployment", function () {
        it("Should set the right owner", async function () {
            expect(await myContract.owner()).to.equal(owner.address);
        });
    });
    
    describe("Transactions", function () {
        it("Should transfer tokens", async function () {
            await myContract.transfer(addr1.address, 100);
            expect(await myContract.balanceOf(addr1.address)).to.equal(100);
        });
        
        it("Should fail if sender doesn't have enough tokens", async function () {
            await expect(
                myContract.connect(addr1).transfer(owner.address, 1)
            ).to.be.revertedWith("Insufficient balance");
        });
    });
});
```

**Testing Patterns**:
```javascript
// Test events
await expect(myContract.transfer(addr1.address, 100))
    .to.emit(myContract, "Transfer")
    .withArgs(owner.address, addr1.address, 100);

// Test reverts
await expect(myContract.unsafeOperation())
    .to.be.revertedWith("Error message");

// Test time-dependent logic
await ethers.provider.send("evm_increaseTime", [86400]); // 1 day
await ethers.provider.send("evm_mine");

// Test with specific msg.value
await myContract.deposit({ value: ethers.parseEther("1.0") });

// Test gas usage
const tx = await myContract.expensiveOperation();
const receipt = await tx.wait();
console.log("Gas used:", receipt.gasUsed.toString());
```

### 2. Testing with Foundry

**Setup**:
```bash
forge init
forge install
```

**Test Example**:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/MyContract.sol";

contract MyContractTest is Test {
    MyContract public myContract;
    address public owner;
    address public user1;
    
    function setUp() public {
        owner = address(this);
        user1 = address(0x1);
        myContract = new MyContract();
    }
    
    function testInitialState() public {
        assertEq(myContract.owner(), owner);
    }
    
    function testTransfer() public {
        myContract.transfer(user1, 100);
        assertEq(myContract.balanceOf(user1), 100);
    }
    
    function testFailTransferInsufficientBalance() public {
        vm.prank(user1);
        myContract.transfer(owner, 100);
    }
    
    // Fuzz testing
    function testFuzzTransfer(address to, uint256 amount) public {
        vm.assume(to != address(0));
        vm.assume(amount <= myContract.balanceOf(owner));
        myContract.transfer(to, amount);
        assertEq(myContract.balanceOf(to), amount);
    }
}
```

### 3. Common Vulnerabilities

**Reentrancy**:
```solidity
// ❌ Vulnerable
contract Vulnerable {
    mapping(address => uint256) public balances;
    
    function withdraw() public {
        uint256 balance = balances[msg.sender];
        (bool success, ) = msg.sender.call{value: balance}("");
        require(success);
        balances[msg.sender] = 0;  // Too late!
    }
}

// ✅ Fixed with Checks-Effects-Interactions
contract Secure {
    mapping(address => uint256) public balances;
    
    function withdraw() public {
        uint256 balance = balances[msg.sender];
        balances[msg.sender] = 0;  // Update state first
        (bool success, ) = msg.sender.call{value: balance}("");
        require(success);
    }
}

// ✅ Fixed with ReentrancyGuard
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureWithGuard is ReentrancyGuard {
    mapping(address => uint256) public balances;
    
    function withdraw() public nonReentrant {
        uint256 balance = balances[msg.sender];
        (bool success, ) = msg.sender.call{value: balance}("");
        require(success);
        balances[msg.sender] = 0;
    }
}
```

**Integer Overflow/Underflow**:
```solidity
// ❌ Vulnerable (before Solidity 0.8)
contract Vulnerable {
    uint8 public count = 255;
    
    function increment() public {
        count++;  // Wraps to 0
    }
}

// ✅ Solidity 0.8+ has built-in checks
pragma solidity ^0.8.0;

contract Secure {
    uint8 public count = 255;
    
    function increment() public {
        count++;  // Reverts on overflow
    }
    
    // Use unchecked only when safe
    function safeIncrement() public {
        unchecked {
            count++;  // Skip check if you're sure
        }
    }
}
```

**Access Control**:
```solidity
// ❌ Vulnerable
contract Vulnerable {
    address public owner;
    
    function setOwner(address _owner) public {
        owner = _owner;  // Anyone can call!
    }
}

// ✅ Proper access control
contract Secure {
    address public owner;
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    constructor() {
        owner = msg.sender;
    }
    
    function setOwner(address _owner) public onlyOwner {
        owner = _owner;
    }
}

// ✅ Using OpenZeppelin
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureWithOZ is Ownable {
    function sensitiveOperation() public onlyOwner {
        // Only owner can call
    }
}
```

**Front-Running**:
```solidity
// ❌ Vulnerable to front-running
contract Vulnerable {
    mapping(bytes32 => bool) public answers;
    
    function submitAnswer(string memory _answer) public {
        bytes32 hash = keccak256(abi.encodePacked(_answer));
        answers[hash] = true;
    }
}

// ✅ Commit-Reveal pattern
contract Secure {
    mapping(address => bytes32) public commits;
    mapping(address => bool) public revealed;
    
    function commit(bytes32 _hash) public {
        commits[msg.sender] = _hash;
    }
    
    function reveal(string memory _answer, bytes32 _salt) public {
        bytes32 hash = keccak256(abi.encodePacked(_answer, _salt));
        require(hash == commits[msg.sender], "Invalid reveal");
        revealed[msg.sender] = true;
    }
}
```

**Denial of Service**:
```solidity
// ❌ Vulnerable - unbounded loop
contract Vulnerable {
    address[] public users;
    
    function payAll() public {
        for (uint i = 0; i < users.length; i++) {
            // Can run out of gas
            payable(users[i]).transfer(1 ether);
        }
    }
}

// ✅ Pull over push pattern
contract Secure {
    mapping(address => uint256) public balances;
    
    function addPayment(address _user, uint256 _amount) public {
        balances[_user] += _amount;
    }
    
    function withdraw() public {
        uint256 amount = balances[msg.sender];
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

### 4. Security Best Practices

**Checklist**:
- ✅ Use latest Solidity version
- ✅ Follow Checks-Effects-Interactions pattern
- ✅ Use OpenZeppelin contracts
- ✅ Implement access control
- ✅ Validate all inputs
- ✅ Handle external calls carefully
- ✅ Avoid using `tx.origin`
- ✅ Be careful with delegatecall
- ✅ Test edge cases
- ✅ Use security tools

**Safe Patterns**:
```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureContract is ReentrancyGuard, Pausable, Ownable {
    // State variables
    mapping(address => uint256) public balances;
    
    // Events
    event Deposit(address indexed user, uint256 amount);
    event Withdrawal(address indexed user, uint256 amount);
    
    // Checks-Effects-Interactions pattern
    function withdraw(uint256 amount) 
        external 
        nonReentrant 
        whenNotPaused 
    {
        // Checks
        require(amount > 0, "Amount must be positive");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        // Effects
        balances[msg.sender] -= amount;
        emit Withdrawal(msg.sender, amount);
        
        // Interactions
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
    
    // Emergency pause
    function pause() external onlyOwner {
        _pause();
    }
    
    function unpause() external onlyOwner {
        _unpause();
    }
    
    receive() external payable {
        balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }
}
```

### 5. Security Tools

**Static Analysis**:
```bash
# Slither
pip3 install slither-analyzer
slither contracts/MyContract.sol

# Mythril
pip3 install mythril
myth analyze contracts/MyContract.sol

# Solhint
npm install -g solhint
solhint contracts/**/*.sol
```

**Testing Coverage**:
```bash
# Hardhat
npx hardhat coverage

# Foundry
forge coverage
```

**Formal Verification**:
- Certora Prover
- K Framework
- Runtime Verification

### 6. Audit Process

**Pre-Audit Checklist**:
1. ✅ Comprehensive test coverage (>90%)
2. ✅ All tests passing
3. ✅ Run static analysis tools
4. ✅ Document all functions
5. ✅ Clean, readable code
6. ✅ Follow style guide
7. ✅ No hardcoded values
8. ✅ Proper access controls

**What Auditors Check**:
- Logic errors
- Access control issues
- Reentrancy vulnerabilities
- Integer overflow/underflow
- Front-running risks
- Gas optimization
- Code quality
- Documentation

**After Audit**:
- Fix all critical issues
- Address high/medium findings
- Document false positives
- Re-test after changes
- Consider re-audit for major changes

## 🛠️ Exercises

### Exercise 1: Write Test Suite
Create comprehensive tests for a token contract:
- Deployment tests
- Transfer tests
- Approval tests
- Edge cases
- Failure scenarios

### Exercise 2: Find Vulnerabilities
Review provided vulnerable contracts and identify:
- Reentrancy risks
- Access control issues
- Integer issues
- DoS vulnerabilities

### Exercise 3: Secure a Contract
Fix a vulnerable contract using best practices:
- Add access control
- Implement ReentrancyGuard
- Add input validation
- Follow CEI pattern

### Exercise 4: Run Security Tools
Use static analysis tools on your contracts:
- Run Slither and fix findings
- Check with Solhint
- Achieve >90% test coverage

## 📖 Resources

### Tools
- [Slither](https://github.com/crytic/slither) - Static analysis
- [Mythril](https://github.com/ConsenSys/mythril) - Security analyzer
- [Echidna](https://github.com/crytic/echidna) - Fuzzing tool
- [Manticore](https://github.com/trailofbits/manticore) - Symbolic execution

### Learning
- [Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [SWC Registry](https://swcregistry.io/) - Smart Contract Weaknesses
- [Ethernaut](https://ethernaut.openzeppelin.com/) - Security challenges
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)

### Auditing Firms
- OpenZeppelin
- Trail of Bits
- ConsenSys Diligence
- Quantstamp
- Certora

## ✅ Completion Checklist

- [ ] Can write comprehensive tests (Hardhat/Foundry)
- [ ] Understand common vulnerabilities
- [ ] Know security best practices
- [ ] Can use security analysis tools
- [ ] Understand audit process
- [ ] Implemented secure contracts
- [ ] Achieved >90% test coverage
- [ ] Fixed vulnerable contracts

**Previous**: [Module 6: Advanced Solidity](../06-advanced-solidity/)  
**Next**: [Module 8: DApp Development](../08-dapp-development/) →
