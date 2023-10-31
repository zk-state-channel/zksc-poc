# ZK State Channels
## Why do we need ZK State Channels？
The ZK State Channel is a novel scaling solution based on state channels, Validium data availability, and recursive zero-knowledge proofs. Its primary aim is to significantly reduce on-chain transaction burden and user operation costs through state channels and the Validium solution. It still allows for the creation of complex off-chain execution environments, leveraging methods such as recursive zero-knowledge proofs. This infrastructure provides a secure, effective, and cost-effective solution for high-transaction and complex-logic applications like blockchain games.

Fully On-chain Gaming (FOCG), which is believed to be the next-generation dApp, stores all game logic and data entirely on the blockchain, requiring high system performance. This makes Validium an ideal execution environment for FOCG. However, for interoperability reasons, FOCG must retain user data and game contracts on the blockchain. Running FOCG directly in Validium may reduce its interoperability and potentially impact the player's user experience, dividing traffic and liquidity. To address this issue, a zk state channel is constructed between FOCG and Validium using recursive zk-SNARKs. This allows complex and independent game logic within FOCG to run on Validium, verifying the correctness of the player's game processes and results through recursive zkSNARKs, and transmitting zk proofs and game results back to the blockchain after the game's conclusion.
## How does it work？
<img src="https://github.com/zk-state-channel/zksc-poc/blob/main/zksc_pic_1.jpg" alt="High-level Structure" width="650" height="auto">
ZK State Channels combine technologies such as recursive zero-knowledge proofs (recursive zk-SNARKs), state channels, and Validium to achieve an efficient, private, and scalable blockchain interaction method. The following are the core functional modules of the ZK State Channel:

### 1. On-Chain Contract Module
State Channels are a scalability technology with the core concept of executing transactions offline, only submitting the state (account balances, contract states, etc.) to the blockchain when necessary. In ZK State Channels, the on-chain contract module serves as the entry point for users to interact with on-chain and manage on-chain states. It is used to lock the state when opening a state channel and settle it after the channel is closed.
#### Efficient & Streamlined dApp Framework
In an ideal design, we aim to build a low-integration-cost state channel construction framework that imposes no restrictions on the state progression within the channel and the termination logic of the channel, making it easy to support various types of applications. Leveraging recursive zero-knowledge proofs generated by the off-chain virtual execution environment, a highly adaptable dApp workflow is defined:
- Participants in the channel lock the state that needs to be initialized and open the state channel after completing on-chain signatures.
- Participants can freely execute operations in the off-chain virtual environment, with no strict reliance on round-robin signature rules, thanks to the presence of a sequencer.
- In case of on-chain state disputes, users need to take on-chain actions and rely on data availability modules to confirm the final state.
#### Universal Integration Interfaces
Any on-chain contract and off-chain logic can be integrated into a zk-state-channels dApp by exposing a few common function interfaces for the channel to use as conditions for state updates: isFinalized returns whether the application state outcome is final, and getOutcome returns the boolean or numerical result of the application.
```Solidity
// required interface for zk-state-channels dApp with boolean outcome
interface IBooleanOutcome {
    function isFinalized(bytes calldata _query) external view returns (bool);
    function getOutcome(bytes calldata _query) external view returns (bool);
}

// required interface for zk-state-channels dApp with numeric outcome
interface INumericOutcome {
    function isFinalized(bytes calldata _query) external view returns (bool);
    function getOutcome(bytes calldata _query) external view returns (uint);
}
```
Additionally, our framework provides a simplified template in Solidity to implement common on-chain logic for application disputes. By combining the contract template with off-chain data availability modules, developers can focus solely on application-specific logic without being burdened by any state channel-related complexities, such as signature verification, Transaction ID tracking, or state machine management.
Application contract developers only need to implement the following interfaces abstracted by the contract template (not the final product, for reference only):
```Solidity
/** 
 * @notice Get the app outcome 
 * @param _query Query args 
 * @return True if query satisfied 
 */
function getOutcome(bytes memory _query) public view returns (bool);

/** 
 * @notice Update app state according to an off-chain state proof 
 * @param _state Signed off-chain app state 
 * @return True if update succeeds 
 */
function updateByState(bytes memory _state) internal returns (bool);

/** 
 * @notice Update app state according to an on-chain action 
 * @param _action Action data 
 * @return True if update succeeds 
 */
function updateByAction(bytes memory _action) internal returns (bool);

/** 
 * @notice Finalize the app outcome in case of on-chain action timeout 
 */
function finalizeOnTimeout() internal;

/** 
 * @notice Get app state associated with the given key 
 */
function getState(uint _key) external view returns (bytes memory);
```
### 2. Recursive zk-SNARKs Generation Module
As an off-chain scaling solution, the ZK State Channels allow us to move complex logic and computations from fully on-chain games to off-chain. It leverages zk-proofs to verify the correctness and integrity of user operations conducted off-chain, ultimately requiring only the submission of the user's game results and state to the blockchain. Since State Channels only submit data to the chain once when the channel is closed, to ensure the integrity and validity of user data, ZK State Channels introduce a new zk-proof generation scheme: recursive zero-knowledge proofs.
#### Recursion + zk-SNARKs
Recursive Zero-Knowledge Proofs are a variant of zero-knowledge proofs that allow one proof to verify the correctness of another proof. In recursive zero-knowledge proofs, a proof can include a reference to another proof, enabling multiple proofs to be nested together, and forming a proof chain for verification within the proof hierarchy.
Users of ZK State Channels perform off-chain operations through contract execution in a virtual environment. These operations batch together transaction data and generate zero-knowledge proofs. Leveraging the concise proofs of zkSNARKs, the generated zk-proofs are recursively generated to minimize on-chain data and verification costs.
Thanks to upgrades and improvements in the Plonky2 algorithm, recursive proofs based on the polynomial commitment scheme FRI can eliminate the trade-off between "proof size" and "proof time." In scenarios where proof time is a priority, optimizations can be applied to achieve maximum proof speed. Importantly, when these proofs are recursively aggregated, a single proof that can be verified in a smaller circuit is obtained. This means that the proof size can be reduced to approximately 45kb, with a proof time of just 20 seconds.
### 3. Off-Chain Execution Module
The off-chain executable modules include two main functionalities:
- zkEVM, an EVM-equivalent smart contract execution environment.
- A third-party data availability module designed based on Validium.
#### zkEVM Execution Environment
To facilitate developers in executing their complex contract logic and generating zk-proofs within the ZK State Channels, it is essential to have a zkEVM runtime environment that is functionally equivalent to the Ethereum Virtual Machine (EVM). This runtime environment will be integrated into each minimal unit called zk-Node within the off-chain network. Developers can easily compile and deploy Solidity code in zkEVM. These dApps will include, but not be limited to, fully on-chain games developed based on game engines like MUD and on-chain dApps such as Social.
#### Cost-Effective Storage
The goal of zk State Channel is to reduce storage costs by processing certain transactions through off-chain computation followed by on-chain verification. This approach ensures the integrity and legality of off-chain transactions through recursive zk-SNARKs. By running ZK State Channels within Validium, user transaction data can be directly stored in Validium's Data Availability (DA) layer, thereby lowering user transaction costs and enhancing transaction performance.
### 4. Transaction Data Coordinator Module
Off-chain networks rely on a fixed ticking mechanism to ensure that participants can maintain consistent observations of global data, enabling real-time and effective interactions within the game. This often implies the need for containers like blocks to receive and process transactions from the global network. The construction of ZK State Channels is based on building recursive zk-proofs on a per-game basis. This requires the introduction of new roles to reprocess and distribute transactions from different games to the generators of zk-proofs, ensuring effective segmentation and coordination of network data in both temporal and spatial dimensions.

<img src="https://github.com/zk-state-channel/zksc-poc/blob/main/zksc_pic_2.jpg" alt="High-level Structure 2" width="850" height="auto">

Taking an example from a MUD-based game project, within the established World and System, the essence of a player creating a new session is to build a new state of data storage, which means updating the Tables within the Store module. Therefore, the responsibility of the Transaction Data Coordinator is to construct independent identifiers (IDs) based on these Tables and to monitor and distribute them.
### Thanks to...
[Mohamed Fouda's inspiration](https://twitter.com/mohamedffouda/status/1712193022085992456?s=61&t=z6gv2VZ59wUiwkf41xNwcQ). 
