# Blockchain

## What is a blockchain?
A blockchain is a cryptographically secure way to store an ever growing list of records.
These records could be many things, such as transactions in a ledger, or state changes to the block.
Blockchains are resistant from tampering by having each block contain a hashed reference to the previous block.
They are managed within peer to peer networks and do not require a centralised authority to be considered valid.
Validity is ensured by a majority consensus of the blockchain.

## History
Bitcoin was the first implementation of a blockchain.
It used as a currency system online, without requiring a governing body.


## How a blockchain works

## Blockchain types

### Permissioned

### Public

### Private

## Differences and uses of blockchain implementations

### Bitcoin

### Etherium

### Corda
- https://docs.corda.net
#### The Network
- A corda network consists of nodes which are able to execute CorDapps.
- TLS is used for communication between nodes, and only between those nodes - so communications in the network can be done privately.
- The network has a network map service which details the IPs, certificates and services provided for each node.


- Networks are semi-private and have a *doorman service* which enforces entry rules into the network, such as what information nodes must provide.
- If the provided information is accepted, the *network permissioning service* returns a signed certificate which is used for certifying the node's identity when communicating with other nodes.


- Nodes can provide services, such as:
- - One to many *notary services* which enable the node to guarantee uniqueness and possible validity of ledger updates.
- - Zero to many *oracle services* which signs transactions if they provide a true fact.
#### The ledger
- There is no central store of data.
- Each node maintains a database of known facts.
- So each node is only aware of a subset of the ledger, not the entire thing.
- When two or more nodes see a single fact, that fact becomes *lockstepped*(the update occurs chronologically)
#### Identity
- Identities can represent a *legal identity*, or a *service identity* of a network service.
- Identities can either be *well known* or *confidential*.
- Well known identities are verifiable via a public key which is published in the network map service. These identities are not secret.
- Confidential identities are only given to those who are involved in transactions with them. Their public key can be exposed to third parties, but their name and certificate is limited.


- To verify the owner of a public key, nodes use certificates.
1. On initialisation, a node generates a key pair and sends a certificate signing request to the doorman.
2. The doorman checks the indentity, then issues a certificate to the node which is used as the node's *certificate authority*.
3. The node then uses the *certificate authority* certificate to generate a *TLS certificate* and a *Well known signing certificate*.
4. The node then builds a *node information record* which contains its address and well known identity, and registers it with the network map service.
- From the *signing certificate*, an organisation is able to generate well known and confidential identities to use.


- Only identities published in the network map service are well known.
- There can be private network map services, but identities published on them should still be considered well known.
#### States
- A state is an immutable object representing a fact known by one to many Corda nodes at a specific point in time.
- State can contain arbitrary data, such as the fact that 'A' owes 'B' some money.
- The state also contains a reference to the contract that governs the evolution of the state. 


- As states are immutable, when a change is needed, a new version of the state is created and the old state is marked as historic.
- The lifecycle of a state is its *state sequence* which shows the evolution of the state over time.


- Each node maintains a *vault*, which is all of the *state sequences* for facts which are known and relevant to that node.
- So the current ledger for a node is just the heads of all the sequences that are in its vault.
#### Contracts
- A transaction is valid iff it is signed by all required signers AND it is contractually valid.
- Contract validity is determined by:
- 1. Every state points to a contract.
- 2. A contract takes a transaction as input, and returns whether it is valid based on the contracts rules.
- 3. A transaction is valid only if every input state and every output state is also valid.


- Transactions can be written in any JVM language.
- If a transaction is not contractually valid, it can never be added to the ledger. So contracts impose rules that are independent upon willingness of the required signers.


- A transaction verification must *always accept* or *always reject* a transaction. It cannot be changed.
- This is necessary for reaching consensus.
- Contracts are run in deterministic sandboxes that do not allow any randomness. The only information available to a contract is the information included in the transaction.


- As contracts have no outside information, they cannot check that the transaction is the same as what was agreed on.
- Therefore it is up to peers not to sign transactions, even if they are contractually valid.


- Transaction validity may require external information such as an exchange rate, which is provided by an *Oracle*.
#### Transactions
- A transaction is a proposal to update the ledger.
- Transactions can have any number of inputs and outputs.
- - Different state types
- - Issuance(no inputs), exits(no outputs)
- - Merge or split assets($2 + $3 -> $5)
- Transactions are atomic - all proposed changes happen, or none happen.


- Two basic transaction types:
- 1. Notary change transactions(change a states notary)
- 2. General transactions


- When a transaction is created, its outputs don't currently exist, but its inputs do - they are outputs from previous transactions.
- The input states are included in the proposed transaction, via a reference.


- For a transaction to be committed, it must receive signatures from all required signers.
- When committed, the inputs become marked as historic, and the outputs become part of the ledger.


- The transaction should only be signed iff:
- 1. The transaction is valid - previous transactions are signed and the transaction is contractually valid.
- 2. The transaction is unique - no other existing transaction has consumed the inputs we want to use.
- Gathering signatures alone is not enough for the transaction to be valid.


- As well as input and output states, transactions can hold Commands, Timestamps and Attachements.


- A *command* is used to indicate the transactions intent, which will affect how validity is checked.
- Each command is linked to a list of one to many required signers.


- *Attachments* are large bits of data that may be reused.
- A transaction can refer to zero to many attachments.
- Eg. Calendar of public holidays.


- *Time windows* are used if a transaction only wants to be approved during a certain time.
#### Flows
- Instead of manually verifying and signing transactions to update the ledger, corda consolidates this into flows.
- A flow is a sequence of steps to tell a node how to achieve a particular update.


- When a flow is installed on a node, the node owner can execute the flow with a remote procedure call.
- The flow abstracts networking, I/O and concurrency.
- Flows do not run within a sandbox, so can have outside info/randomness.


- Nodes communicate via flows. Each node can have zero to many flow classes used to respond to messages from a flow type.
- Eg. If A wants to update with B, A must start a flow that B is registered to respond to. When A sends the message to B through the flow, B will respond via its registered response for that flow.


- A flow that is created as a subprocess to another flow is called a *subflow*. The parent flow waits for the subflow to return.


- Corda provides libraries for common flows.
- - Notarising and recording a transaction.
- - Gathering signatures from counterparty nodes.
- - Verifying a chain of transactions.
- https://docs.corda.net/flow-library.html


- Many flows can be active at the same time.
- When a flow becomes blocked, it is stored on disk and another flow is worked on, leaving blocked for later.
- So flows can last days.
#### Consensus
- To determine if an update is valid, there are two consensus' required.
- 1. *Validity consensus* - checked by signers before signing transaction
- 2. *Uniqueness consensus* - checked by notary service


Validity Consensus
- Checking the following holds for current and past transactions in the transaction chain.
- - Transaction is accepted by contract for every input and output state.
- - Transaction has all required signatures.


- Validating past transactions in the transaction chain is called *walking the chain*.
- It needs to be done to ensure that the proposed transaction is using a correct ledger.


- Verifying a transaction will require the transaction chain, which you may not have. You can request the missing transactions from the transaction proposer.


Uniqueness Consensus
- None of the inputs into a transaction have already been consumed by another transaction.
- If any have been consumed, it is an example of double spend, and is considered invalid.
- Uniqueness consensus is provided by notaries.
#### Notaries
- A notary is a service that provides uniqueness consensus.
- It attests that it did not sign a transaction that had already consumed the proposed transactions inputs.
- Notorisation can be seen as the point of finality.


- Every state has an appointed notary. Notaries are only able to sign transactions in which they are the notary for all inputs.


- Consensus algorithms in Corda are pluggable.
- Notaries may use different consensus algorithms, specific for business/country/etc. 


- A notary can choose if it wants to provide *validity consensus* by validating transactions before comitting them.
- If it does not, nodes can create invalid transactions which will mark input states as consumed.
- If it does, the notary will need to see the entire transaction, which is bad for privacy.


- Corda can have multiple notaries, each possibly running separate consensus algorithms.
- Benefits:
- - Privacy: Use both validating and non-valdiating notaries.
- - Load Balancing, latency


- If a transaction needs inputs that use different notaries, a change of notary is required.
- Use a special *notary-change transaction* that takes:
- - Single input state
- - Single output state identical with different notary
- The original notary then signs the new output state and it becomes the new state.
#### Time-windows
- A notary is able to act as the 'clock' in transactions and only sign them if they occur within a specific time window.
- Timestamping and notarisation occurs at the same time so you can't timestamp, wait ages and then try to commit.
- Because time is not exact, and transactions are not sent for notorisation at transaction creation, time windows are used.
- Time windows can either be before a certain time, after a certain time, or both.
#### Oracles
- Facts can be included within transactions as part of commands(such as exchange rate).
- An oracle signs the transaction if it finds the fact to be true.
- When you want to use a fact in a transaction, you request a command from the oracle containing the fact, which is put into the transaction. The oracle is then required to sign the transaction.
- When giving the transaction to the oracle, the transaction is pulled apart to only include the fact command. The oracle can sign the transaction and also ensure that it isn't change as it is in a merkle tree structure so the hash would change if other parts of the transaction are changed.
#### Nodes
- A node is a unique JVM environment that can run Corda Services and CorDapps.
- [pic](https://docs.corda.net/_images/node-architecture.png)
- A node contains:
- - *Persistant layer* for storage (eg. sql database)
- - *Network interface* for node communication
- - *RPC*(Remote Procedure Call) interface to communicate with node owner
- - *Service hub* for to connect the nodes flows with the nodes services
- - *Plugin registry* to extend node with CorDapps
#### Tradeoffs 

### Hyperledger Quilt

## Terminology

#### Consensus
A consensus is a way in which distributed systems can make a decision where there are faults.
For example, a blockchain that has two different histories. The consensus is how the remaining nodes decide which of the two histories are correct. They could, for example agree by a majority.

#### Notary
A notary is someone that is able to legally witness transactions.
For example, with banks -> govt regulators.
Consensus notary.

#### Ledger

