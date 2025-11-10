# 🪙 Mint Your Own Token on Arc Testnet (Foundry Tutorial)

A simple step-by-step guide to **create and deploy your own ERC20 token** on **Arc Testnet** using Foundry.

---

## 🧰 Requirements

- [Visual Studio](https://visualstudio.microsoft.com/downloads/)
- [Foundry]
- [OpenZeppelin Contracts]
- Solidity (Download from Visual Studio Extension)
- Wallet
- Testnet tokens (USDC)

---

## 1. Setup
> Run each of the following commands in your terminal:
```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
forge init arc-token-tutorial
cd arc-token-tutorial
open -a "Visual Studio Code" .
```
> Or, create the folder manually and open it in Visual Studio Code.
Then, open the integrated terminal (Ctrl + `) and run the following commands:

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
forge init arc-token-tutorial
cd arc-token-tutorial
```
<hr>

## 2. Install OpenZeppelin

```bash
forge install openzeppelin/openzeppelin-contracts 
```
---

## 3. Remove existing token contract
> First, delete the default `Counter.sol` template file from the `/src` directory:
```bash
rm src/Counter.sol
```
Save(Ctrl + S)
<hr>


## 4. Add Environment Variables
> Create a new file named .env in the root of the project directory `arc-token-tutorial`

Paste the following:
```bash
PRIVATE_KEY="Generated private key from the previous comman"
ARC_TESTNET_RPC_URL="https://rpc-testnet.arc.market"

# (only for minting later)
TOKEN_ADDRESS="0xYourTokenAddressHere"
RECIPIENT="0xYourWalletAddressHere"
```
Save (Ctrl + S).

> ⚠️ Important:
Use a test wallet. Never expose your real private key.
<hr>


## 5. Update Foundry Config
> Replace these in your `foundry.toml` with:
```bash
[profile.default]
src = "src"
out = "out"
libs = ["lib"]

[rpc_endpoints]
arc_testnet = "	https://rpc.testnet.arc.network"
```
---

## 6. Write the Token contract
Next, create a new file named `Bitcoin.sol` inside the /src directory, and add the following code:
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import { ERC20 } from "../lib/openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
import { Ownable } from "../lib/openzeppelin-contracts/contracts/access/Ownable.sol";

contract Bitcoin is ERC20, Ownable {
    constructor(uint256 initialSupply) ERC20("Bitcoin", "BTC") Ownable(msg.sender) {
        _mint(msg.sender, initialSupply * 10 ** decimals());
    }

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount * 10 ** decimals());
    }
}
```


## 7. Create the Deploy Script
> Since you deleted `Counter.sol`, you need to remove or replace any scripts and tests that reference it to avoid compilation errors.

Delete the contract in the `/script` directory
```bash
rm script/Counter.s.sol
rm test/Counter.t.sol
```
> Navigate to the `/script` directory, and create a new script file named DeployMyToken.s.sol. Then, add the following test cases to validate your contract:
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import { Script, console } from "forge-std/Script.sol";
import { Bitcoin } from "../src/Bitcoin.sol";

contract DeployMyToken is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        Bitcoin token = new Bitcoin(1_000_000); // 1 million tokens
        console.log("Bitcoin deployed at:", address(token));

        vm.stopBroadcast();
    }
}
```
Save (Ctrl + S)

> Compile the contract using:
```bash
forge build
```

<hr>

## 7. Generate a wallet
> In your terminal, paste: 
```bash
cast wallet new
```
> The command generates a new keypair and returns output similar to the following:
```bash
Successfully created new keypair.
Address:     "Generated address"
Private key: "Generated Private Key"
```
Fund your wallet
> Visit https://faucet.circle.com, select Arc Testnet, paste your generated wallet address, and request testnet USDC.

## 8. Deploy the Token
Run this command:
```bash
forge script script/DeployMyToken.s.sol:DeployMyToken \
  --rpc-url $ARC_TESTNET_RPC_URL \
  --broadcast \
  --private-key $PRIVATE_KEY
```
When successful, you’ll see:
```bash
✅ MyToken deployed at: 0xYourTokenAddressHere
```
But if it returns an error, paste this and re-run the previous code
```bash
export PRIVATE_KEY=enter your private key without the double quotes("")
export ARC_TESTNET_RPC_URL=enter the RPC url without the double quotes("")
```

Copy that address and update it in your `.env` file as:
```bash
TOKEN_ADDRESS="0xYourTokenAddressHere"
```
___

## 9. Create Mint Script
Create a file in the `/script` directory `MintToken.s.sol`

Paste: 
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import { Script, console} from "forge-std/Script.sol";
import { Bitcoin } from "../src/Bitcoin.sol";


contract MintToken is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        address tokenAddress = vm.envAddress("TOKEN_ADDRESS");
        address recipient = vm.envAddress("RECIPIENT");

        vm.startBroadcast(deployerPrivateKey);

        Bitcoin token = Bitcoin(tokenAddress);
        token.mint(recipient, 5000); // Mint 5000 tokens

        console.log("Minted 5000 BTC to:", recipient);
        vm.stopBroadcast();
    }
}
```
Save (Ctrl + S).
<hr>

## 10. Mint Tokens
After updating your `.env` with the token and recipient addresses, run:
```bash
forge script script/MintToken.s.sol:MintToken \
  --rpc-url $ARC_TESTNET_RPC_URL \
  --broadcast \
  --private-key $PRIVATE_KEY
```
Output example:
```bash
✅ Minted 5000 MTK to: 0xYourWalletAddressHere
```
But if it gives an error, past this, and run the above script again:
```bash
export TOKEN_ADDRESS=token address without the quotes
export RECIPIENT=recipient address without the quotes
```
<hr>

## 11. Verify on Arc Explorer
Go to:
```bash
https://testnet.arcscan.app/
```
<hr>

## 12 Copy the Contract address and add in your wallet









