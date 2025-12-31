# Project 1: Simple Storage Contract

**Difficulty**: ⭐ Beginner  
**Duration**: 2-4 hours  
**Prerequisites**: Module 4 - Solidity Fundamentals

## 🎯 Project Overview

Create a smart contract that stores and retrieves a single number value with proper access control and event logging.

## 📝 Requirements

### Core Features
1. Store a uint256 number
2. Only owner can update the number
3. Anyone can read the number
4. Emit event when number changes
5. Include basic access control

### Bonus Features
- Add multiple owners
- Track history of changes
- Add minimum/maximum value constraints
- Implement pause functionality

## 🎓 Learning Goals

After completing this project, you will understand:
- How to create and deploy a basic smart contract
- State variables and their visibility
- Function modifiers for access control
- Events for logging changes
- Constructor for initialization
- How to interact with deployed contracts

## 📋 Step-by-Step Guide

### Step 1: Set Up Environment

**Option A: Use Remix IDE (Recommended for beginners)**
1. Go to [remix.ethereum.org](https://remix.ethereum.org/)
2. Create new file: `SimpleStorage.sol`
3. Start coding!

**Option B: Use Hardhat (Local development)**
```bash
mkdir simple-storage
cd simple-storage
npm init -y
npm install --save-dev hardhat
npx hardhat init
```

### Step 2: Write the Contract

Create `SimpleStorage.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title SimpleStorage
 * @dev Store and retrieve a single number with access control
 */
contract SimpleStorage {
    // State variables
    uint256 private storedNumber;
    address public owner;
    
    // Events
    event NumberChanged(uint256 oldNumber, uint256 newNumber, address changedBy);
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }
    
    // Constructor
    constructor(uint256 _initialNumber) {
        owner = msg.sender;
        storedNumber = _initialNumber;
        // Note: First number set, so old value is implicitly 0
        emit NumberChanged(0, _initialNumber, msg.sender);
    }
    
    // Functions
    
    /**
     * @dev Store a new number (only owner)
     * @param _number The number to store
     */
    function setNumber(uint256 _number) public onlyOwner {
        uint256 oldNumber = storedNumber;
        storedNumber = _number;
        emit NumberChanged(oldNumber, _number, msg.sender);
    }
    
    /**
     * @dev Retrieve the stored number
     * @return The stored number
     */
    function getNumber() public view returns (uint256) {
        return storedNumber;
    }
    
    /**
     * @dev Transfer ownership to a new address
     * @param newOwner The address of the new owner
     */
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0), "New owner cannot be zero address");
        address oldOwner = owner;
        owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
}
```

### Step 3: Deploy and Test

**In Remix:**
1. Press Ctrl+S to compile
2. Go to "Deploy & Run" tab
3. Deploy the contract with initial value (e.g., 42)
4. Test functions:
   - Call `getNumber()` - should return 42
   - Call `setNumber(100)` - should succeed
   - Call `getNumber()` again - should return 100
   - Switch to another account
   - Try `setNumber(200)` - should fail (not owner)

## 🎯 Testing Checklist

- [ ] Contract compiles without errors
- [ ] Owner can set number
- [ ] Non-owner cannot set number
- [ ] Anyone can read number
- [ ] Events are emitted correctly
- [ ] Ownership can be transferred
- [ ] New owner has correct permissions
- [ ] Edge cases handled (zero address, etc.)

## 🚀 Bonus Challenges

### Challenge 1: Add History Tracking
Track all changes made to the stored number.

### Challenge 2: Add Value Constraints
Only allow numbers within a certain range (e.g., 1-1000).

### Challenge 3: Add Pause Functionality
Allow owner to pause/unpause contract updates.

## ✅ Completion Criteria

You've successfully completed this project when:
- ✅ Contract compiles without errors
- ✅ Deployed successfully
- ✅ Can interact with deployed contract
- ✅ Understand all concepts used

## 🎉 What's Next?

After completing this project:
1. Try the bonus challenges
2. Move to Project 2: Token Wallet
3. Share your deployed contract address!

---

**Need Help?** Review [Module 4: Solidity Fundamentals](../../../modules/04-solidity-fundamentals/)
