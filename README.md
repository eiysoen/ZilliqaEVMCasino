# ZilliqaEVMCasino
The ZilliqaEVMCasino is a high-level, EVM-compatible decentralized application (dApp) designed to operate on the Zilliqa network. It provides a provably fair gambling environment by integrating Chainlink VRF (Verifiable Random Function) for secure, tamper-proof random number generation. The contract currently supports two modular games
----Core Security Mechanisms
​This contract prioritizes the safety of both the house (bankroll) and the players' funds by implementing industry-standard security practices:
​Reentrancy Protection: By inheriting OpenZeppelin's ReentrancyGuard and applying the nonReentrant modifier, the contract prevents malicious actors from exploiting withdrawal functions through recursive call attacks.
​Pull-over-Push Pattern (Checks-Effects-Interactions): The contract does not automatically transfer winnings to a player's external wallet upon winning. Instead, winnings are credited to an internal ledger (s_playerBalances). Players must manually call withdrawPlayerBalance() to pull their funds. This eliminates out-of-gas errors during the Oracle callback and mitigates reentrancy risks.
​Emergency Pause Mechanism: By inheriting OpenZeppelin's Pausable, the contract owner can temporarily halt all betting and depositing activities (whenNotPaused modifier) in the event of a detected bug, network issue, or routine maintenance.
​---Player Journey & Game Flow
​The operational flow is fully automated via smart contract logic and the decentralized Oracle network.
​Funding the Account: Players call the deposit() function, sending native tokens (e.g., ZIL) to fund their internal casino balance.
​Placing a Wager: A player calls either playCoinFlip() or playDice(), passing their prediction and the wager amount. The contract checks if the house has enough liquidity to pay out a potential win (at least 2x the wager) and deducts the wager from the player's internal balance.
​Requesting Randomness: The contract immediately fires a request to the Chainlink VRF Coordinator and logs the game session details (player address, game type, choice, and wager) inside the s_requests mapping.
​Oracle Callback (Resolution): Once the VRF node generates a secure random number, it calls the fulfillRandomWords() internal function. The contract resolves the game logic:
​For Coin Flip, it uses modulo 2 (rng % 2).
​For Dice Roll, it calculates a number between 1 and 100 (rng % 100 + 1).
​Payout Distribution: If the player's prediction matches the mathematical outcome, the contract multiplies their wager (e.g., a 2x payout) and credits the winnings directly to their internal balance.
​Withdrawal: The player can safely extract their balance back to their personal Web3 wallet at any time by calling withdrawPlayerBalance().
---House & Admin Functions
​The contract includes administrative privileges restricted solely to the deployer (Owner) via OpenZeppelin's Ownable module:
​Bankroll Management: The owner can fund the casino's liquidity pool via depositBankroll() and extract profits via withdrawBankroll(). This liquidity pool is strictly used to pay out winning bets.
​Circuit Breakers: The owner has exclusive access to the pauseContract() and unpauseContract() functions to control the active state of the casino
