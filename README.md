# How to verify deployed solidity smart contracts with constructor arguments (WIP)

### TLDR

This read & plugin helps you verify the source code for your Solidity contracts. At the moment, it supports Etherscan-based explorers, explorers compatible with its API like Blockscout and Sourcify.

## Steps
- [x] Install hardhat-verify
- [x] Import @nomicfoundation/hardhat-verify
- [x] Add the following Etherscan and Sourcify configs
- [x] Write your arguments.js
- [x] Run npx hardhat verify --network sepolia --constructor-args arguments.js CONTRACT_ADDRESS

### Installation
```shell
npm install --save-dev @nomicfoundation/hardhat-verify
```

Then if you are using TypeScript, add this to your hardhat.config.ts:
```shell
import "@nomicfoundation/hardhat-verify";
```

### Add Etherscan and sourcify configs
You need to add the following Etherscan and Sourcify configs to your hardhat.config.js file:
```shell
module.exports = {
  networks: {
    mainnet: { ... }
  },
  etherscan: {
    // Your API key for Etherscan
    // Obtain one at https://etherscan.io/
    apiKey: "YOUR_ETHERSCAN_API_KEY"
  },
  sourcify: {
    // Disabled by default
    // Doesn't need an API key
    enabled: true
  }
};
```

### Complex arguments
When the constructor has a complex argument list, you'll need to write a javascript module that exports the argument list. 
The expected format is the same as a constructor list for an ethers contract. For example, if you have a contract like this:
```typescript
    constructor(
        bytes32[] memory _proposalNames,
        address _tokenContract,
        uint256 _targetBlockNumber
    ) {
        tokenContract = IMyToken(_tokenContract);
        targetBlockNumber = _targetBlockNumber;
        require(
          targetBlockNumber <= block.number,
          "TokenizedBallot: targetBlockNumber must be in the past"
        );
        for (uint i = 0; i < _proposalNames.length; i++) {
            proposals.push(Proposal({name: _proposalNames[i], voteCount: 0}));
        }
    }
```

### Write your arguments.js
then you can use an arguments.js file like this:
```javascript
const { toHex } = require("viem");

const proposals = ["pizza", "tacos", "donuts", "wings"];

const _proposalNames = proposals.map((name) => toHex(name, { size: 32 }));

module.exports = [
  _proposalNames,
  "0x3c9d658a9b358cf1985bc52c5476229e8b186f1f",
  "6564652",
];
```

### Run npx hardhat verify
The module can then be loaded by the verify task when invoked like this:
```shell
npx hardhat verify --network sepolia --constructor-args arguments.js CONTRACT_ADDRESS

npx hardhat verify --network sepolia --constructor-args ballotArguments.js 0x3cac1e711919a503681d258ee38c01d5e1971f4d
```
