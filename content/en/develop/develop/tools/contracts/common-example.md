---
title: Common example
weight: 20
type: docs
---

# Common Examples

This page provides practical examples of how to use `@xpla/contracts` to interact with CONX Chain's precompile contracts. We'll walk through a complete staking pool implementation that demonstrates real-world usage of the StakingI interface.

## StakingPool Contract Example

The StakingPool contract demonstrates how to create a staking pool that utilizes CONX Chain's StakingI precompile contract to manage validator delegations while providing additional features like reward distribution and user management.

### Contract Overview

The StakingPool contract provides:
- **Staking Management**: Users can stake tokens through the pool to validators
- **Reward Distribution**: Automatic reward calculation and distribution
- **User Management**: Track individual user stakes and rewards
- **Validator Integration**: Direct integration with CONX Chain's staking system

### Complete Contract Implementation

```solidity
pragma solidity ^0.8.28;

import "@xpla/contracts/interfaces/StakingI.sol";

/// @title Simple Staking Pool
/// @dev A simple staking pool contract utilizing StakingI interface
contract StakingPool {
    // StakingI interface instance
    StakingI public constant stakingContract = StakingI(0x0000000000000000000000000000000000000800);
    
    // Pool manager
    address public immutable poolManager;
    
    // Information about the staking pool
    struct PoolInfo {
        uint256 totalStaked;
        uint256 totalShares;
        uint256 rewardRate; // Annual reward rate (base unit: 10000 = 100%)
        uint256 lastRewardTime;
        uint256 accumulatedRewardPerShare;
    }
    
    // User-specific staking information
    struct UserInfo {
        uint256 stakedAmount;
        uint256 shares;
        uint256 rewardDebt;
        uint256 lastClaimTime;
    }
    
    PoolInfo public poolInfo;
    mapping(address => UserInfo) public userInfo;
    
    // Events
    event Staked(address indexed user, uint256 amount, uint256 shares);
    event Unstaked(address indexed user, uint256 amount, uint256 shares);
    event RewardsClaimed(address indexed user, uint256 amount);
    event PoolRewardRateUpdated(uint256 newRate);
    
    // Modifiers
    modifier onlyManager() {
        require(msg.sender == poolManager, "Only pool manager can call this function");
        _;
    }
    
    constructor(uint256 _rewardRate) {
        poolManager = msg.sender;
        poolInfo.rewardRate = _rewardRate;
        poolInfo.lastRewardTime = block.timestamp;
    }
    
    /// @dev Stake tokens in the staking pool
    /// @param validatorAddress Validator address
    /// @param amount Amount of tokens to stake
    function stake(string memory validatorAddress, uint256 amount) external payable {
        require(amount > 0, "Amount must be greater than 0");
        require(msg.value == amount, "Sent value must match amount");
        
        // Update pool rewards
        updatePool();
        
        UserInfo storage user = userInfo[msg.sender];
        
        // Settle existing rewards
        if (user.shares > 0) {
            uint256 pending = (user.shares * poolInfo.accumulatedRewardPerShare / 1e18) - user.rewardDebt;
            if (pending > 0) {
                // Pay rewards (simple implementation by sending to msg.sender)
                payable(msg.sender).transfer(pending);
                emit RewardsClaimed(msg.sender, pending);
            }
        }
        
        // Perform actual staking through StakingI
        bool success = stakingContract.delegate(
            address(this),
            validatorAddress,
            amount
        );
        require(success, "Staking delegation failed");
        
        // Update pool information
        uint256 newShares = amount; // Simple implementation with 1:1 ratio
        poolInfo.totalStaked += amount;
        poolInfo.totalShares += newShares;
        
        // Update user information
        user.stakedAmount += amount;
        user.shares += newShares;
        user.rewardDebt = user.shares * poolInfo.accumulatedRewardPerShare / 1e18;
        user.lastClaimTime = block.timestamp;
        
        emit Staked(msg.sender, amount, newShares);
    }
    
    /// @dev Unstake staked tokens
    /// @param validatorAddress Validator address
    /// @param amount Amount of tokens to unstake
    function unstake(string memory validatorAddress, uint256 amount) external {
        UserInfo storage user = userInfo[msg.sender];
        require(user.stakedAmount >= amount, "Insufficient staked amount");
        
        // Update pool rewards
        updatePool();
        
        // Settle existing rewards
        uint256 pending = (user.shares * poolInfo.accumulatedRewardPerShare / 1e18) - user.rewardDebt;
        if (pending > 0) {
            payable(msg.sender).transfer(pending);
            emit RewardsClaimed(msg.sender, pending);
        }
        
        // Perform actual unstaking through StakingI
        int64 completionTime = stakingContract.undelegate(
            address(this),
            validatorAddress,
            amount
        );
        require(completionTime > 0, "Unstaking failed");
        
        // Update pool information
        uint256 sharesToRemove = amount; // Simple implementation with 1:1 ratio
        poolInfo.totalStaked -= amount;
        poolInfo.totalShares -= sharesToRemove;
        
        // Update user information
        user.stakedAmount -= amount;
        user.shares -= sharesToRemove;
        user.rewardDebt = user.shares * poolInfo.accumulatedRewardPerShare / 1e18;
        
        emit Unstaked(msg.sender, amount, sharesToRemove);
    }
    
    /// @dev Claim rewards
    function claimRewards() external {
        // TBD...
    }
    
    /// @dev Update pool rewards
    function updatePool() public {
        // TBD ...
    }
    
    /// @dev Update reward rate (manager only)
    function setRewardRate(uint256 _rewardRate) external onlyManager {
        // TBD ...
    }
    
    /// @dev Query user's reward information
    function pendingRewards(address _user) external view returns (uint256) {
        // TBD ...
        return 0
    }
    
    /// @dev Query pool information
    function getPoolInfo() external view returns (
        uint256 totalStaked,
        uint256 totalShares,
        uint256 rewardRate,
        uint256 lastRewardTime,
        uint256 accumulatedRewardPerShare
    ) {
        return (
            poolInfo.totalStaked,
            poolInfo.totalShares,
            poolInfo.rewardRate,
            poolInfo.lastRewardTime,
            poolInfo.accumulatedRewardPerShare
        );
    }
    
    /// @dev Query user information
    function getUserInfo(address _user) external view returns (
        uint256 stakedAmount,
        uint256 shares,
        uint256 rewardDebt,
        uint256 lastClaimTime
    ) {
        UserInfo storage user = userInfo[_user];
        return (
            user.stakedAmount,
            user.shares,
            user.rewardDebt,
            user.lastClaimTime
        );
    }
    
    /// @dev Allow contract to receive ETH
    receive() external payable {}
    
    /// @dev Allow contract to receive ETH
    fallback() external payable {}
}
```

### Key Features Explained

#### 1. **StakingI Integration**
```solidity
StakingI public constant stakingContract = StakingI(0x0000000000000000000000000000000000000800);
```
The contract directly integrates with CONX Chain's StakingI precompile contract to perform actual validator delegations.

#### 2. **Staking Function**
```solidity
function stake(string memory validatorAddress, uint256 amount) external payable {
    // ... validation and reward settlement ...
    
    // Perform actual staking through StakingI
    bool success = stakingContract.delegate(
        address(this),
        validatorAddress,
        amount
    );
    require(success, "Staking delegation failed");
    
    // ... update pool and user information ...
}
```
Users can stake tokens by calling this function with a validator address and the amount to stake.

#### 3. **Unstaking Function**
```solidity
function unstake(string memory validatorAddress, uint256 amount) external {
    // ... validation and reward settlement ...
    
    // Perform actual unstaking through StakingI
    int64 completionTime = stakingContract.undelegate(
        address(this),
        validatorAddress,
        amount
    );
    require(completionTime > 0, "Unstaking failed");
    
    // ... update pool and user information ...
}
```
Users can unstake their tokens, which triggers the actual undelegation through the StakingI interface.

### Deployment Configuration

Create an Ignition module for deployment:

```typescript
// ignition/modules/StakingPool.ts
import { buildModule } from "@nomicfoundation/hardhat-ignition/modules";

export default buildModule("StakingPool", (m) => {
  const stakingPool = m.contract("StakingPool", [500]); // 5% annual reward rate

  return { stakingPool };
});
```

### Usage Scripts

#### 1. Staking Tokens

```typescript
// scripts/stake-token.ts
import { ethers } from "ethers";
import { StakingPool__factory } from "../types/ethers-contracts/index.js";

async function main() {
  console.log("=== StakingPool Stake Function Script ===\n");

  // Setup provider and wallet
  const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
  const user1 = new ethers.Wallet(USER_PRIVATE_KEY, provider);
  
  // Contract address
  const contractAddress = CONTRACT_ADDRESS;
  const stakingPool = StakingPool__factory.connect(contractAddress, user1);

  // Staking parameters
  const stakeAmount = ethers.parseEther("1.0"); // 1 XPLA
  const validatorAddress = VALIDATOR_ADDRESS;
  
  console.log("Staking Parameters:");
  console.log(`- Amount: ${ethers.formatEther(stakeAmount)} XPLA`);
  console.log(`- Validator: ${validatorAddress}`);
  console.log(`- User: ${user1.address}\n`);

  // Execute stake function
  try {
    const stakeTx = await stakingPool.stake(validatorAddress, stakeAmount, { 
      value: stakeAmount 
    });
    console.log(`Transaction hash: ${stakeTx.hash}`);
    
    await stakeTx.wait();
    console.log("✅ Stake transaction successful!\n");
    
    // Get updated user info
    const userInfo = await stakingPool.getUserInfo(user1.address);
    console.log(`Updated User Info:`);
    console.log(`- Staked Amount: ${ethers.formatEther(userInfo.stakedAmount)} XPLA`);
    console.log(`- Shares: ${ethers.formatEther(userInfo.shares)}`);
    
  } catch (error) {
    console.log("❌ Stake transaction failed:");
    console.error(error);
  }
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error("Script failed:", error);
    process.exit(1);
  });
```

#### 2. Querying User Information

```typescript
// scripts/query-user-info.ts
import { ethers } from "ethers";
import { StakingPool__factory } from "../types/ethers-contracts/index.js";

async function main() {
  console.log("=== StakingPool Query User Info Script ===\n");

  // Setup provider and wallet
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const user1 = new ethers.Wallet(USER_PRIVATE_KEY, provider);
  
  // Contract address
  const contractAddress = CONTRACT_ADDRESS;
  const stakingPool = StakingPool__factory.connect(contractAddress, provider);

  console.log(`Querying info for user: ${user1.address}\n`);

  try {
    const userInfo = await stakingPool.getUserInfo(user1.address);
    
    console.log("✅ User Info Retrieved Successfully!\n");
    console.log("=== User Information ===");
    console.log(`Address: ${user1.address}`);
    console.log(`Staked Amount: ${ethers.formatEther(userInfo.stakedAmount)} XPLA`);
    console.log(`Shares: ${ethers.formatEther(userInfo.shares)}`);
    console.log(`Reward Debt: ${ethers.formatEther(userInfo.rewardDebt)} XPLA`);
    console.log(`Last Claim Time: ${new Date(Number(userInfo.lastClaimTime) * 1000).toLocaleString()}`);
    
    // Calculate pending rewards
    const pendingRewards = await stakingPool.pendingRewards(user1.address);
    console.log(`Pending Rewards: ${ethers.formatEther(pendingRewards)} XPLA`);
    
  } catch (error) {
    console.log("❌ Failed to query user info:");
    console.error(error);
  }
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error("Script failed:", error);
    process.exit(1);
  });
```

### Testing the Contract

To test the StakingPool contract:

1. **Deploy the contract**:
   ```bash
   npx hardhat ignition deploy ignition/modules/StakingPool.ts
   ```

2. **Run the staking script**:
   ```bash
   npx hardhat run scripts/stake-token.ts
   ```

3. **Query user information**:
   ```bash
   npx hardhat run scripts/query-user-info.ts
   ```

### Key Benefits of This Implementation

1. **Direct Integration**: Uses CONX Chain's native staking system through precompile contracts
2. **User Management**: Tracks individual user stakes and rewards
3. **Flexible Configuration**: Pool manager can adjust reward rates
4. **Event Tracking**: Comprehensive event emission for frontend integration

This example demonstrates how to effectively use `@xpla/contracts` to build complex DeFi applications that integrate seamlessly with CONX Chain's native functionality.
