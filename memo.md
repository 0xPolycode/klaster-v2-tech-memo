# Klaster V2 Tech Memo

Klaster is a blockchain protocol aimed at providing superior blockchain user experience through advanced _network abstraction_ technology. This document aims to provide a high-level overview on
how Klaster would achieve this functionality, as well as provide a glimpse into the user experince of blockchain, with a production version of Klaster available. 

The name for Klaster protocol
comes from the word "cluster", as in "computer cluster" - which is usually defined as _"a set of computers that work together so that they can be viewed as a single system"_. This is the end
goal of Klaster, but for blockchains - to enable multiple, independent blockchain networks to work as a single system.

Some of the key outcomes of Klaster protocol & SDK which we will discuss in thsi document are:

* Paying for transactions with a single token, no matter which network the transaction is being executed on.
* Paying for transactions with stablecoins.
* Contracts owning "remote accounts" on blockchains they're not deployed to
* Synchronizing state of independent blockchains through Chainlink CCIP

## Klaster Controller / Worker Accounts

The core of how Klaster achieves the goals set in the beginning of this documnet is by splitting blockchain accounts into two distinct categories - _Controller Accounts_ & _Worker Accounts_. The
Controller Accounts are Smart Account Wallets (Smart Contracts) which have the authority to control one or more Worker Accounts across multiple blockchains. 

The structure of Controller / Worker Accounts is illustrated below.

![Structure](https://github.com/0xPolycode/klaster-v2-tech-memo/assets/129866940/e31a3db5-222e-4867-bd6c-7fbfe0be028e)

## Klaster Singletons & Transaction Agents

The way the Controller / Worker dynamic is achieved is through the use of singleton smart contracts, deployed on all supported blockchain networks. These singleton smart contracts are the 
_owners_ of all worker accounts on deployed blockchains. Being an "owner" of the Worker Accounts gives the signleton contracts a single privilege - to invoke the `execute()` function on the 
Worker Accounts. 

The _execute()_ function accepts blockchain transaction data as an input parameter and triggers the Worker Account to call a certain smart contract or execute any other blockchain action. Only
the singleton contract can call the _excute()_ function. 

However, before the singleton contract will call the `execute()` function, it must perform certain checks. Each Worker Account has an associated _Transaction Ruleset_ smart contract. This smart
contract defines the terms by which the Worker Account executes the transactions. The _Transaction Ruleset_ contract is synchronized across all blockchains and controlled by the _Controller Account_.

Synchronizing the _Transaction Ruleset_ across blockchains is done by invoking arbitrary message calls with Chainlink CCIP.

The Klaster singleton contracts on all blockchains have a public `execute()` function on them. This entire system allows a user to execute a transaction by simply signing an offchain message. How?

This is where the concept of _Agents_ appears. An _Agent_, in the Klaster system - can be anyone. For a transaction to be executed on the Klaster network, the following order of operations happen:

1. The user signs an off-chain message, containing the transaction data & the chain on which the message should be executed. Other conditions may be required, such as multiple signers, time delays, etc... These are defined in the _Transaction Ruleset_ smart contract
2. The user _publishes_ the signed message to the Klaster Mempool
3. An agent "picks up" the signed message & calls the public `execute()` function on the Klaster Singleton contract
4. The Singleton contracts check the Worker Account for which the transaction is signed & runs the `validate()` function on the associated _Transaction Ruleset_ contract. If the contract returns false, the transaction is declined.
5. If the _Transaction Ruleset_ contract returns _true_, the singleton contract calls the `execute()` function on the Worker Account & the worker account executed the transaction.

### Liveness, Censorship Resistance & Security

Key points of this system:
* **Liveness & Censorship Resistance** - Since the `execute()` function on the Klaster singleton contracts is public, it can be called by absolutely anyone. If the system is completely shut down, it can be called by the user themselves - ensuring liveness even in the worst case scenario. Similarily, if no Agent is willing to sign the message, the user can execute it themselves - ensuring censorship resistance.
* **Security** - Since the _Transaction Ruleset_ contract will verify some cryptographic truth (usually that the owner of the _Controller Account_ has signed the message) the user is free to distribute the signed message on unsecure connections - as it can only ever result in the transaction they signed themselves.

## Ruleset Synchronization through CCIP

The core premise of the Klaster protocol is that the _Transaction Ruleset_ is the defining contract for each _Worker Account_ which defines the cryptographic & other conditions for transactions to
be executed. This means that the biggest security vulnerability of Klaster is the synchronization of the _Transaction Ruleset_ contracts on all blockchains. 

One example of a Klaster implementation (which is actually a product the Klaster team is working on).

**Controller Account** is a Gnosis Safe smart contract - deployed on Ethereum mainnet. It has three signers and a defined two out of three signer structure. This _Controller Account_ controlles a
**Worker Account** which is deployed on the same address on Ethereum, Optimism, Arbitrum, Base & BSC. The _Transaction Ruleset_ contract for the _Worker Account_ is set to be an exact copy of the
Gnosis Safe validation function, which checks that two of three signers have signed the transaction data. 

The users execute transactions by signing transaction data (e.g. send 20 USDC on Arbitrum from _Worker Account_ to some other address). An _Agent_ picks up this signed transaction data from the 
Klaster mempool and calls the `execute()` function on the Arbitrum Klaster singleton. The Arbitrum Klaster singleton passes the signed transaction data through the _Transaction Ruleset_ contract
associated with the _Worker Account_ which is being invoked. The _Transaction Ruleset_ contract verifies that at least two of the three signers have signed the off-chain message. It allows the 
transaction to proceed & the Klaster Arbitrum Singleton calls the `execute()` function on the Arbitrum _Worker Account_.

This works perfectly well, while the signers remain the same. However, when deploying the contract for the first time _or_ when changing the signers, we need to ensure a way for the _Transaction
Ruleset_ to change, without forcing the user to execute transactions on multiple blockchains. We can't allow _Agents_ to change the _Transaction Ruleset_ contract, since their functions use 
that same _Transaction Ruleset_ contract to validate the transactions. 

However, what we _can_ do - is leverage a cross-chain communication protocol to pass a validated message from the _Controller Account_ on Ethereum Mainnet, that a _Transaction Ruleset_ change is 
needed. Our security model, in that case, moves from _Agents_ to (in the case of Klaster) - Chainlink CCIP.

The process is simple - the signers sign & execute a messsage on the Ethereum mainnet, which is then propagated by CCIP through the Klaster _Transaction Ruleset_ destination contracts.  

## Cross-Chain Transactions through CCIP

Everything described here was for executing "single chain tranasations" - transactions which change the state of only one chain. Irrespective of the fact that the _Controller Account_ is not 
deployed on the chain on which we wish to execute our transaction, the transaction itself will only change the state of one blockchain network. 

For cross-chain transactions, which need to rely on message-passing between blockchains, the Klaster protocol elegantly falls-back onto Chainlink CCIP. A transaction requrest is flagged as 
cross-chain & signed as such and the Klaster _Agent_ executes the `crossChainExecute()` function on the Klaster signleton contract on the source chain, with all the required info for a cross-chain
transaction.

The Klaster system is tightly integrated with Chainlink CCIP so it intelligently executes this transaction as a cross-chain transaction, without any additional input from the user. The only
difference being, that the transaction (in this case) will take longer to execute - as CCIP needs to wait for the finality of the source chain to execute the transaction on the destination chain.

## Compatibility

Klaster Worker Accounts act just the same as any other Smart Account or Account Abstraction Wallet and should be compatible with the entire EVM (and potentially non-EVM) ecosystem.

# Klaster V1

In fact, the first version of the Klaster protocol, has no _Agents_ or _Transaction Rulesets_. It only has _Controller Accounts_ & _Worker Accounts_ and relays every single message through CCIP. 
This makes it slower & more expensive than the V2, but has the added benefit of significantly reduced engineering complexity. 

The V1 of the Klaster protocol is live on Ethereum, Polygon, Optimism, Arbitrum, Base, BSC and Avalanche C-Chain. 

You can use it and read the docs here: https://klaster.gitbook.io/gateway/introduction/overview

The first product built on the Klaster V1 platform is _live_ already and you can access it at: https://safe.klaster.io - it's a Gnosis Safe extension, which allows users to have cross-chain 
Safes. Klaster V2 will make this app faster and cheaper as well as add CCIP powered cross-chain swaps.
