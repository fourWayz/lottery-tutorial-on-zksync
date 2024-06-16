
# Building a Decentralized Lottery on zkSync Blockchain
## Building a Decentralized Lottery on the zkSync Blockchain: From Smart Contract Development to Deployment

The rise of blockchain technology has paved the way for innovative applications in various fields, including finance and entertainment. Among these are decentralized applications (DApps), offering transparent, secure, and trustless alternatives to traditional centralized systems.

This article delves into the fascinating world of DApps by guiding you through the development and deployment of a decentralized lottery smart contract on the zkSync blockchain.

## Table of Contents

- [Section 1: Understanding the Basics](#section-1-understanding-the-basics)
  - [1.1 Overview of Lottery Smart Contracts](#11-overview-of-lottery-smart-contracts)
  - [1.2 Project Setup ](#12-project-setup)
- [Section 2: Code Explanation of the Smart Contract](#section-2-code-explanation-of-the-smart-contract)
  - [2.1 Key Variables](#21-key-variables)
  - [2.2 Event](#22-event)
  - [2.3 Modifiers](#23-modifiers)
  - [2.4 Constructor](#24-constructor)
  - [2.5 Managing Rounds and Selecting Winners](#25-managing-rounds-and-selecting-winners)
    - [2.5.1 Entering a Round](#251-entering-a-round)
    - [2.5.2 Starting a New Round](#252-starting-a-new-round)
    - [2.5.3 Ending the Current Round](#253-ending-the-current-round)
  - [2.6 Checking Round Status and Participants](#26-checking-round-status-and-participants)
    - [2.6.1 Getting Current Round Status](#261-getting-current-round-status)
    - [2.6.2 Generating a Random Number](#262-generating-a-random-number)
    - [2.6.3 Retrieving Players](#263-retrieving-players)
- [Section 3: Complete Code](#section-3-complete-code)
- [Section 4: Getting zkSync Faucet](#section-4-getting-zksync-faucet)
- [Section 5: Deployment to remix](#section-5-deployment-to-remix)
- [Section 6: Conclusion](#section-6-conclusion)
- [Section 7: Next Steps](#section-7-next-steps)


### Section 1: Understanding the Basics

### 1.1 Overview of Lottery Smart Contracts
Traditional lotteries often raise concerns about transparency, fairness, and trust in centralized authorities. In contrast, decentralized lotteries, powered by smart contracts on blockchains like zkSync, offer a compelling solution. These contracts, self-executing agreements with codified terms, eliminate the need for intermediaries, ensuring verifiable randomness, tamper-proof record-keeping, and immutable transaction history. This fosters trust and confidence in the lottery system, creating a more engaging and secure experience for participants.

Decentralized lottery smart contracts bring transparency and fairness to the process of organizing and participating in lotteries. This guide explores the fundamental concepts and components involved in building such a contract.

### 1.2 Project Setup
This project will be scaffolded with [zkSync cli](https://docs.zksync.io/build/tooling/zksync-cli/commands/create.html#contracts) for easy set up.

Head over to your command prompt or terminal and run the code below:

`npx zksync-cli create`

You will be prompted to enter your project's name. You can enter your desired project's name, in our case `sociaDapp`.

Next, you will be prompted to select the type of project. Select `Contracts` and proceed. 

After that, select `Ethers V6` as ethereum framework and select `Hardhat + Solidity` as Template.

You will be prompted to enter `private key` or set it later. For this tutorial,leave it blank and continue.

Finally, select your desired package manage to begin installations.

This will create a new zkSync era project named `DecentralizedLottery`. Some examples contracts,tests and deployment will be generated by default. However, for the purposes of this tutorial, we don't need the example contracts related files. So, proceed by removing all the files inside the /contracts , /test folders manually or by running the following commands:


`cd DecentralizedLottery `
`rm -rf ./contracts/*`
`rm -rf ./test/*`

Finally, create a file named `DecentralizedLottery.sol` inside your `contracts` folder.

## Section 2: Code Explanation of the Smart Contract

### 2.1 Key Variables
```solidity
    address public manager;
    address[] public players;
    address public lastWinner;
    uint256 public roundEndTime;
    uint256 public minimumPlayers;
    bool public isRoundActive;
```
The journey begins with understanding the core components of the smart contract. Key variables manage essential aspects like the manager (contract creator), players, winners, round duration, minimum participation thresholds, and round status. Events act as notifications, signaling winner selection and prize distribution. Modifiers enforce access control and restrict certain functions to authorized users. The constructor initializes the contract, setting the manager, minimum players, and initial round status.

1. **`address public manager`**: 
   - Stores the address of the manager or administrator of the lottery contract.
   - Declared as `public`.
   - Participants are added to this array when they call the `enter` function to join the lottery.

2. **`address[] public players`**: 
   - An array that stores the addresses of all the participants who have entered the lottery.
   - Declared as `public`.
   - Participants are added to this array when they call the `enter` function to join the lottery.

3. **`address public lastWinner`**: 
   - Stores the address of the last winner of the lottery.
   - Declared as `public`.

4. **`uint256 public roundEndTime`**: 
   - Stores the timestamp indicating when the current round of the lottery will end.
   - Declared as `public`.

5. **`uint256 public minimumPlayers`**: 
   - Stores the minimum number of players required to start a new round of the lottery.
   - Declared as `public`.

6. **`bool public isRoundActive`**: 
   - A boolean flag that indicates whether a round of the lottery is currently active or not.
   - Declared as `public`.

These variables represent essential components of the lottery smart contract. The manager initiates the lottery, players contribute funds to participate, lastWinner keeps track of the previous winner, roundEndTime determines when a round ends, minimumPlayers sets the threshold for starting a new round, and isRoundActive indicates whether a round is currently active.

### 2.2 Event
```solidity
event LotteryWinner(address winner, uint256 amount);
```

This event is emitted when a winner is selected at the end of a lottery round.

- **Parameters**:
  - `winner` (address): The address of the winner.
  - `amount` (uint256): The amount of funds won by the winner.

When this event is emitted within the smart contract, it records the winner's address and the corresponding prize amount. This logged information can be observed by external applications or user interfaces, enabling them to track and display details about lottery winners and prize amounts on the blockchain.

### 2.3 Modifiers
```solidity
 modifier restricted() {
        require(msg.sender == manager, "Only the manager can call this function");
        _;
    }

    modifier roundActive() {
        require(isRoundActive, "The current round is not active");
        _;
    }
```

The `restricted` modifier ensures that only the manager can call certain functions, providing access control. The `roundActive` modifier checks whether the current round is active before allowing certain operations.

### 2.4 Constructor
```solidity
   constructor(uint256 _minimumPlayers) {
        manager = msg.sender;
        minimumPlayers = _minimumPlayers;
        isRoundActive = false;
    }
```
- **Parameters**:
  - `_minimumPlayers` (uint256): The minimum number of players required to start a new round.

- **Actions**:
  - Sets the `manager` to the address of the contract deployer (`msg.sender`).
  - Initializes the `minimumPlayers` variable with the provided `_minimumPlayers` value.
  - Sets the `isRoundActive` flag to `false`, indicating that no round is currently active.

The constructor function initializes the decentralized lottery smart contract by setting the contract manager to the deployer's address (msg.sender), defining the minimum required players (_minimumPlayers), and initially marking the round as inactive (isRoundActive = false).

### 2.5 Managing Rounds and Selecting Winners
The heart of the lottery lies in managing rounds and selecting winners. Players can contribute funds to enter a round, adhering to minimum contribution requirements and ensuring entries during active rounds are prohibited. The manager initiates new rounds, ensuring sufficient player participation before setting the round duration and marking it active. Round closure involves randomly selecting a winner (proportional to their contribution) and transferring the entire contract balance as the prize. Functions are further provided to retrieve player lists, check round status, and access end times

  ### 2.5.1 Entering a Round
  ```solidity
    function enter() public payable {
        require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether");
        require(!isRoundActive, "Cannot enter while a round is active");
        
        players.push(msg.sender);
    }
  ```
The `enter` function allows participants to join the lottery by sending a payment of at least 0.01 ether. It ensures that participants cannot enter if a round is currently active.
If the conditions are met, the participant's address is added to the list of players.

## Explanation
 - `require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether")` : This line ensures that the value sent with the transaction (msg.value) is greater than 0.01 ether. If not, it reverts the transaction with an error message indicating the minimum contribution requirement.
- `require(!isRoundActive, "Cannot enter while a round is active");` : This line ensures that there is no active round (`isRoundActive` is false) before allowing entry. If a round is active, it reverts the transaction with an error message.
- `players.push(msg.sender);` : This line adds the sender's address (msg.sender) to the players array, indicating their participation in the lottery.

  ### 2.5.2 Starting a New Round
  ```solidity   
    function startNewRound() public restricted {
        require(!isRoundActive, "A round is already active");

        // Ensure there are enough players to start a round
        require(players.length >= minimumPlayers, "Not enough players to start a round");

        roundEndTime = block.timestamp + 24 hours; // Round lasts for 24 hours
        isRoundActive = true;
    }
  ```
The `startNewRound` function is accessible to the contract manager

 only. It ensures that no active round exists and that the player count meets the minimum threshold. If conditions are met, it sets the round end time to 24 hours from the current timestamp and activates the round.

Explanation:
- `require(!isRoundActive, "A round is already active");` : This line ensures that there is no active round (`isRoundActive` is false) before starting a new one. If a round is already active, it reverts the transaction with an error message.
- `require(players.length >= minimumPlayers, "Not enough players to start a round");` : This line checks if the number of players in the `players` array is greater than or equal to the specified `minimumPlayers` requirement. If there are not enough players, it reverts the transaction with an error message.
- `roundEndTime = block.timestamp + 24 hours;` : This line sets the `roundEndTime` to the current timestamp plus 24 hours, indicating the duration of the round.
- `isRoundActive = true;` : This line sets the `isRoundActive` flag to `true`, indicating that a round is now active.

 ### 2.5.3 Ending the Current Round
 ```solidity
   function endRound() public restricted {
        require(isRoundActive, "No active round to end");
        require(block.timestamp >= roundEndTime, "Round has not ended yet");

        uint256 winnerIndex = random() % players.length;
        address winner = players[winnerIndex];
        uint256 prizeAmount = address(this).balance;

        (bool success, ) = winner.call{value: prizeAmount}("");
        require(success, "Transfer to winner failed");

        emit LotteryWinner(winner, prizeAmount);

        // Reset the round
        players = new address ;
        isRoundActive = false;
    }
```
The `endRound` function, also restricted to the manager, checks if the round has ended based on the round end time. It then selects a winner randomly, transfers the prize (contract balance) to the winner, emits the `LotteryWinner` event, and resets the round by clearing the players array and marking the round as inactive.

- `require(isRoundActive, "No active round to end");` : This line ensures that there is an active round (`isRoundActive` is true) before attempting to end it. If no round is active, it reverts the transaction with an error message.
- `require(block.timestamp >= roundEndTime, "Round has not ended yet");` : This line ensures that the current timestamp is greater than or equal to the `roundEndTime`, indicating that the round duration has ended. If the round has not ended, it reverts the transaction with an error message.
- `uint256 winnerIndex = random() % players.length;` : This line generates a random index within the range of the `players` array using the `random` function.
- `address winner = players[winnerIndex];` : This line assigns the address of the selected winner based on the randomly generated index.
- `uint256 prizeAmount = address(this).balance;` : This line assigns the contract's current balance to the `prizeAmount` variable.
- `(bool success, ) = winner.call{value: prizeAmount}("");` : This line initiates the transfer of the `prizeAmount` to the winner's address. The `success` variable indicates whether the transfer was successful.
- `require(success, "Transfer to winner failed");` : This line ensures that the transfer to the winner was successful. If the transfer fails, it reverts the transaction with an error message.
- `emit LotteryWinner(winner, prizeAmount);` : This line emits the `LotteryWinner` event, recording the winner's address and the prize amount.
- `players = new address ;` : This line resets the `players` array to an empty array, preparing for the next round.
- `isRoundActive = false;` : This line sets the `isRoundActive` flag to `false`, indicating that the current round has ended.

### 2.6 Checking Round Status and Participants

### 2.6.1 Getting Current Round Status
```solidity
function getRoundStatus() public view returns (bool) {
        return isRoundActive;
    }
```
The `getRoundStatus` function returns the status of the current round (active or not). This function is declared as `public` and `view`, indicating that it can be called externally and does not modify the contract's state.

- Returns:
  - `bool`: The current status of the round (`isRoundActive`).

Explanation:
- `return isRoundActive;` : This line simply returns the value of the `isRoundActive` variable, indicating whether a round is currently active or not.

### 2.6.2 Generating a Random Number
```solidity
function random() private view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));
    }
```
The `random` function generates a pseudo-random number using block difficulty, timestamp, and the list of players. This function is declared as `private` and `view`, meaning it can only be called within the contract and does not modify the contract's state.

- Returns:
  - `uint256`: A pseudo-random number.

Explanation:
- `return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));` : This line generates a pseudo-random number by hashing (`keccak256`) the concatenated values of the current block's difficulty, the current timestamp, and the `players` array. The resulting hash is then converted to a `uint256` value and returned as the pseudo-random number.

### 2.6.3 Retrieving Players
```solidity
function getPlayers() public view returns (address[] memory) {
        return players;
    }
```
The `getPlayers` function returns the list of players in the current round. This function is declared as `public` and `view`, meaning it can be called externally and does not modify the contract's state.

- Returns:
  - `address[] memory`: The list of players' addresses.

Explanation:
- `return players;` : This line simply returns the `players` array, containing the addresses of all participants in the current round of the lottery.

The following code example encapsulates the explanation above into a smart contract:

### Section 3: Complete Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DecentralizedLottery {
    address public manager;
    address[] public players;
    address public lastWinner;
    uint256 public roundEndTime;
    uint256 public minimumPlayers;
    bool public isRoundActive;

    event LotteryWinner(address winner, uint256 amount);

    modifier restricted() {
        require(msg.sender == manager, "Only the manager can call this function");
        _;
    }

    modifier roundActive() {
        require(isRoundActive, "The current round is not active");
        _;
    }

    constructor(uint256 _minimumPlayers) {
        manager = msg.sender;
        minimumPlayers = _minimumPlayers;
        isRoundActive = false;
    }

    function enter() public payable {
        require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether");
        require(!isRoundActive, "Cannot enter while a round is active");
        
        players.push(msg.sender);
    }

    function startNewRound() public restricted {
        require(!isRoundActive, "A round is already active");
        require(players.length >= minimumPlayers, "Not enough players to start a round");

        roundEndTime = block.timestamp + 24 hours;
        isRoundActive = true;
    }

    function endRound() public restricted {
        require(isRoundActive, "No active round to end");
        require(block.timestamp >= roundEndTime, "Round has not ended yet");

        uint256 winnerIndex = random() % players.length;
        address winner = players[winnerIndex];
        uint256 prizeAmount = address(this).balance;

        (bool success, ) = winner.call{value: prizeAmount}("");
        require(success, "Transfer to winner failed");

        emit LotteryWinner(winner, prizeAmount);

        players = new address ;
        isRoundActive = false;
    }

    function getRoundStatus() public view returns (bool) {
        return isRoundActive;
    }

    function random() private view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));
    }

    function getPlayers() public view returns (address[] memory) {
        return players;
    }
}
```

### Section 4: Getting zkSync Faucet

To deploy your decentralized lottery contract on the zkSync blockchain, you need some zkSync testnet tokens. Follow this [guides](https://docs.zksync.io/ecosystem/network-faucets) to obtain zkSync faucet.



### Section 5: Compiling and Deploying
With zkSync testnet tokens in your wallet, you can deploy the smart contract by following the guides below:

To begin compiling our smart contract using `zkSync cli`, run the below command:

`npm run compile` if you are using Yarn, run `yarn compile`.

After successful compilation of your smart contracts, go ahead to deploy your contract to zkSync sepolia testnet by running the code below:

NOTE: don't forget to enter private key in the `.env` file after obtaining faucet.

Next, update `deploy.ts` inside `deploy` folder with the following code :

```typescript

import { deployContract } from "./utils";

export default async function () {
  const contractArtifactName = "DecentralizedLottery";
  const constructorArguments = [];
  await deployContract(contractArtifactName, constructorArguments);
}

```

Finally, run `npm run deploy` to deploy your smart contract to zkSync sepolia testnet.

By following these steps, you can successfully implement, test, and deploy a decentralized lottery contract on the zkSync blockchain, providing a transparent and trustless lottery system for participants.

### Section 6: Conclusion

The decentralized lottery system outlined in this guide combines fairness, transparency, and trustlessness, leveraging smart contract technology to ensure a secure and efficient lottery process. By deploying this contract on the zkSync blockchain, you benefit from zkRollup technology's scalability and low transaction fees, making the lottery accessible and efficient for all participants. Follow the steps provided to obtain testnet tokens, deploy your contract using Remix, and enjoy the benefits of a decentralized lottery system on the zkSync blockchain.

While this guide provides a solid foundation for building a decentralized lottery, further enhancements can be explored. Integrating Chainlink VRF (Verifiable Random Function) can enhance randomness generation, eliminating potential biases. Implementing additional features like tiered prizes, bonus systems, or automated round transitions can add complexity and excitement to the game. The possibilities are endless, allowing you to customize your lottery DApp and cater to diverse player preferences.
