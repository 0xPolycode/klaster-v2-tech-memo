# Klaster Protocol

## Introduction / Problem
Blockchain networks have achieved big adoption in the fifteen years since the launch of the original Bitcoin network. However, one large problem still remains. The so-called
"blockchain trillema". The trillema states that each blockchain network must choose two of the following three properties:

* Security
* Scalability
* Decentralization

While the trillema has a fair share of critics, the core point of the statement stands. For a blockchain to be secure and decentralized - it needs to have a lot of independent nodes.
For those nodes to be able to verify the intergrity of the data - each one needs to hold the copy of the entire state of the blockchain. This means that thousands of nodes need to
keep full copies of every single transaction done on the blockchain. Obviously, this approach means that storing and validating those transactions will be expensive.

Early on it was realized that not all nodes need to store all the transactions. You can validate transactions of separate blockchain networks - called "rollups" and then
simply post cryptographic validity proofs to the "core" blockchain network - such as Ethereum. Beyond this, many teams launched alternate blockchain networks - called L1 Blockchains,
which are not dependend on Ethereum or Bitcoin in any way. These advancements are close to fully solving the scalability problem, with blockspace being more available than ever.

However, the scalability benefits come with a big cost - severely broken user experience. UX has been a weak side of blockchain networks since the inception and, instead of getting
better, it's rapidly deteriorating. Right now there are dozens, if not hundreds, of independent blockchain networks - each requiring their own gas token to be bridged to that network
to be used. Beyond this, users are prompted to add RPC nodes to their wallets, developers are required to tailor a transaction for a specific blockchain, etc... These limitations are 
severely limiting the adoption of blockchain technology & pushing users into centralized exchanges and apps.

## Klaster Protocol
The Klaster protocol proposes a new way:

* A user should _not_ care about the blockchain they're intracting with.
* A user should _not_ acquire new gas tokens for every blockchain they're using.
* Ideally, a user should _not_ need to acquire gas tokens at all.
* A user should _not_ be prompted to add RPC nodes or switch networks.
* A developer should be able to prepare a transaction for any blockchain and expect the user to be able to execute it.

With a few hard requirements which aren't allowed to be broken:

* The system must be fully censorship resistant.
* The system must be fully trustless.
* The system must have a mechanism for ensuring liveness.

The Klaster protocol achieves this by introducing a new primitive - _Klaster Cross-Chain Accounts_. Cross-chain accounts enable users to send/receive assets and interact
with dApps on all supported blockchains, while paying for transaction fees on only one chain (or even paying for transaction fees off-chain). It does this while 
introducing zero sacrifices to censorship resistance or trustlessness. 

Cross-Chain accounts work by splitting the transaction signing from transaction execution. The transaction signing is done _off-chain_ by the user, while transaction 
execution is handled by _Klaster Nodes_. The verification of the validity of the transaction is done by _Klaster Singletons_ - singleton smart contracts deployed on
every supported blockchain. 

The flow of a Klaster transaction is the following:

1. The user decides on an action - e.g. `send ERC20 token` or `sell NFT`.
2. A Klaster transaction is encoded from the original transaction data. This is done through the Klaster Frontend or Klaster SDK.
3. The encoded Klaster transaction is posted to the Klaster Mempool.
4. The transaction is picked up by the Klaster Node.
5. The Klaster node calls the `execute()` function on the Klaster Singleton contract on the desired chain.
6. The Klaster Singleton contract verifies cryptographically that the transaction was actually signed by the owner of the cross-chain account.
7. If valid, the Klaster signleton contract executes the user desired action.

![Klaster System](https://github.com/0xPolycode/klaster-v2-tech-memo/assets/129866940/becbf97f-e7c9-4bdc-92b4-ca1cfd5eba8d)

Every Klaster Cross-Chain account has an _ownership structure_. The simplest case is when the owner of the cross-chain account is an EOA or a single address. A special,
more complicated case is when the owner of the cross-chain account is a multi-sig wallet. And the third, most complicated case is when the owner of the cross-chain 
account is a generic smart contract. Each one of those cases has special features and considerations.

### Cross-Chain Accounts

![291327180-e31a3db5-222e-4867-bd6c-7fbfe0be028e](https://github.com/0xPolycode/klaster-v2-tech-memo/assets/129866940/71c2b76c-a551-4ae2-8f06-2b6d9deac066)

Cross-chain accounts have several user benefits:

* User doesn't need to have gas on the chains they are interacting with.
* User can have multiple accounts which share the same "gas" payment system.
* User can hold assets on non-EVM chains, while signing transactions with their Etheruem wallet.
* User doesn't need to switch networks or add RPC nodes.

Combined, these benefits radically transform the experience of using multiple blockchains. Beyond this, the existance of cross-chain accounts enables the user 
to sign a single transaction which transfers their assets from one blockchain to another and calls a function on the destination chain. For example - a user can
use a bridge to transfer USDC from Ethereum to Polygon and supply them on AAVE on Polygon. All in a single transaction.

A lot of user experience benefits are fully backwards compatible with existing dApps and by accessing blockchain through apps such as Klaster Dashboard, which automatically
encode all transactions into Klaster transactions, users get a much better blockchain user experience. However, if developers can assume that users have Klaster cross-chain
accounts and use the Klaster SDK to adapt their applications to this assumption - this can open fully integrated cross-chain experiences. 

## Cross-Chain Messaging

Certain aspects of the Klaster protocol, such as signer synchronization for multi-sig wallets, contract to contract calls and several others - require cross-chain communicaton
solutions. Klaster is solution-agnostic, but has been built to work nativelly with Chainlink CCIP.

## Paying for Transactions

As we explored in this document, in the Klaster system - the gas fees are paid by Klaster Nodes. However, the users must somehow pay the Klaster Nodes to perform these actions.
The Klaster system makes no assumptions on how this needs to be done - leaving room for the system to be modular and adapt to the changing needs of future markets. 

In the initial implementation of Klaster Nodes, we have prepared several payment mechanisms:

* **Payment Tx Metadata** - when giving the user encoded transaction data, the Klaster Node will prepare a separate "payment" transaction data, which will be signed together
  with the original tx data. This "payment" data will be the transfer of the gas token or other supported token to the Klaster Node account. For the user, this is a
  seamless experience.
* **Prepayment** - a user can send a certain amount of gas tokens (or other accepted tokens) to a predetermined address, after which they will be able to spend that amount
  gas payments.

In the future, the Klaster gas payment model can be as simple or complex as we want. As the payment is a direct negotiation between the Klaster Node and the user, many
models can be developed. Some ideas include:

* Post-paid transactions. Users receive a bill at the end of the month for the transactions they made.
* Payment with credit cards/Apple Pay/Google Pay - Klaster Nodes can accept traditional payment methods for gas and cover the on-chain costs for the user.
* Free transactions - Klaster Nodes can reward users with "free" initial transactions, to ease the onboarding process.

## Klaster 

Klaster greatly simplifies operations for users on multiple blockchains.

![KlasterWW](https://github.com/0xPolycode/klaster-v2-tech-memo/assets/129866940/3a78a4ed-dd5e-4fee-9dc9-c30ede511664)

## Node Economics

Running a Klaster Node is a profit-generating activity for the node operators. They execute transactions on behalf of users & for that they charge a premium. Every node will decide which assets, on which networks, they accept as payments. Additionally, every node will decide which networks they're willing to execute transactions on.

Let's give an example: An operator is running a Klaster node and they set the following parameters:

* **Accepting**: USDC, USDT, stETH, LINK
* **Supported Networks**: Ethereum, Polygon, zkSync, Optimism, Base, Scroll, BSC, Avalanche C-Chain, Arbitrum

The user creates an execution intent with the following parameters:
* Call `swap()` function on the Uniswap contract on Base
* Pay for transaction in USDC on Optimism
* Set a maximum gas limit of 250.000
* Set a maximum gas price of 1 gwei

The node creates a commitment and signs it cryptographically. Let's assume that the price of ETH at the time of the transaction is $4100. The commitment may look like this:

* Maximum gas limit 250.000
* Maximum gas price 1 gwei
* Charge a price of 0.00025 ETH for a transaction
* Calculated in USDC, the price is 1,025 USDC
* Charge a 10% execution premium - total price is 1,1275

The node will create the trnasaction and payment data for signing and return it to the user. The node would also sign this commitment cryptographically and return it to the user. The user would then either accept or decline the transaction. 

If the user should accept the transaction, they would perform a `personal_sign` operation on the data. The node would then call the `execute()` function on the Klaster singleton contract on Base with the data. The contract would decode the transaction, cryptographically verify that the account owner was the one who signed it and proxy the contract call to the `swap()` function on Uniswap.

Beyond this, the node would call the `execute()` function on the Klaster singleton contract on Optimism. The singleton would decode this as a payment operation and would transfer 1.1275 USDC to the Node wallet from the user wallet.

With this, the user has sucesfully executed a transaction on Base, while paying for gas in USDC on Optimism!

The node has made a minimum 10% profit on the transaction in USDC terms.

### Slashing

Once the node has commited itself to execute the transaction, it must do so. If the node is commited to execute the transaction and fails to do so until the `expiry` parameter in its cryptographically signed commitment, the user is able to commit the signed commitment, together with the signed transcation data to the Klaster staking contracts and the contracts will slash the offending node.

## Liveness Guarantees

Klaster protocol must maintain liveness guarantees for its users. This is achieved through the use of the Klaster Token. The "Solvers" who execute transactions on behalf of other users
need to stake their Klaster token into Klaster smart contracts. For every transaction that reaches the Klaster mempool, a PoS system is used to determine which Solver will execute the 
transaction. If the Solver is not available or has an insufficient gas balance to execute the transaction, they are slashed. If they are available and can execute the transaction, they are
rewarded.Depending on how much Klaster a node has staked, they increase the chance of being selected to execute a transaction. A proposal on how to achieve this through the usage of 
Chainlink VRF will be presented in a separate document.



