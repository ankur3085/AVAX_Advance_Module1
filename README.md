# BlockChain: A Simple DeFi Kingdom Clone on Avalanche

## Overview

**BlockChain** is a decentralized finance (DeFi) game built on the Avalanche network. Inspired by DeFi Kingdoms, this project is designed to be a blockchain-based game where players can collect, build, battle, and earn rewards through various in-game activities. The game is powered by a custom EVM subnet and utilizes smart contracts for in-game transactions.

## Features

- **Custom EVM Subnet**: The game operates on a custom EVM subnet on the Avalanche network, providing low fees and fast transaction times.
- **ERC20 Token Integration**: The game uses a native ERC20 token called "Defi Empire Token" (symbol: DEFIE) for in-game currency.
- **Vault System**: Players can deposit and withdraw their tokens from a smart contract-based vault.
- **MetaMask Integration**: Seamless integration with MetaMask for handling in-game transactions.
- **In-Game Economy**: Players can mint tokens, deposit them into the vault, and withdraw them as they earn and spend within the game.

## Prerequisites
- Node.js and npm
- Avalanche-CLI
- MetaMask
- Solidity
- Remix IDE

## Setup

### 1. Clone the Repository
```bash
git clone https://github.com/ankur3085/AVAX_Advance_Module1.git
cd AVAX_Advance_Module1
```

### 2. Install Dependencies
Install the necessary dependencies:
```bash
npm install
```

### 3. Create and Deploy Your EVM Subnet
1. **Install Avalanche-CLI**:
   ```bash
   npm install -g avalanche-cli
   ```

2. **Create a New Subnet**:
   ```bash
   avalanche subnet create myblockchain --evm
   ```

3. **Configure Your Subnet**:
   - Customize parameters such as block times and gas costs as needed.
   - Define your native currency.

4. **Deploy the Subnet**:
   ```bash
   avalanche subnet deploy myblockchain
   ```
   ```plaintext
        +-----------------------------------------------------------------+
        |                       WALLET CONNECTION                         |
        +-----------------+-----------------------------------------------+
        | Network RPC URL | http://127.0.0.1:9650/ext/bc/myblockchain/rpc |
        +-----------------+-----------------------------------------------+
        | Network Name    | myblockchain                                  |
        +-----------------+-----------------------------------------------+
        | Chain ID        | 21345                                         |
        +-----------------+-----------------------------------------------+
        | Token Symbol    | ANK                                           |
        +-----------------+-----------------------------------------------+
        | Token Name      | ANK Token                                     |
        +-----------------+-----------------------------------------------+
     ```

5. **Connect MetaMask to Your Subnet**:
   - Add the subnet to MetaMask as a custom RPC network.

### 4. Deploy the Smart Contracts
1. **ERC20 Contract**:
   - Deploy the provided ERC20 token contract to your custom subnet using Remix.

2. **Vault Contract**:
   - Deploy the Vault contract to manage staking and withdrawals.

### 5. Integrate with Front-End
- Use the provided front-end template or build your own using React.js or another framework.
- Connect the front-end to your deployed contracts and the custom subnet.
- Implement game logic and MetaMask interactions.

## Smart Contracts

### ERC20 Token
The ERC20 token contract is the in-game currency for Defi Empire. It includes minting, burning, transferring, and approval functionalities.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "DEEPAKTOKEN";
    string public symbol = "DKTN";
    uint8 public decimals = 16;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint amount) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

### Vault Contract
The Vault contract handles staking and rewards distribution for the players.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint);
    function balanceOf(address account) external view returns (uint);
    function transfer(address recipient, uint amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint);
    function approve(address spender, uint amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint amount) external returns (bool);
    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);
}

contract Vault {
    IERC20 public immutable token;
    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint _amount) external {
        uint shares;
        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / token.balanceOf(address(this));
        }
        _mint(msg.sender, shares);
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        token.transfer(msg.sender, amount);
    }
}
```

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
