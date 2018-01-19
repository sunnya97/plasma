# Tendermint and Plasma

**This is pre-alpha. Everything can change.**

This document is a mix between a design and specification document for building
Plasma chains using Tendermint Core. 

At a high level Plasma is limited in scope. It allows child chains, which are
secured on the root chain. Moreover it is possible for users to deposit tokens
from the root to the child chain and withdraw them without the consent of the
child chain. However we later on propose a strategy using IBC and peg zones.
This is due to the fact that the withdrawal described in Plasma only works for
very limited UTXO zones, and using IBC we can achieve similar guarantees but 
for more complex child chains.

In our construction a Plasma chain is simply a chain which allows for staking
to take place on a root chain and for token movement to occur through IBC.
We propose to use Ethereum as the root chain and Tendermint Core as the 
child chain.

The Plasma paper itself does not mention the construction for the child chain.
This document proposes to use Tendermint Core to execute the child chain.

The first component is a staking or Plasma contract on Ethereum. Users deposit
tokens to that contract for the benefit of becoming a validator in the
underlying Tendermint chain. Moreover users can deposit tokens that then get
minted on the child chain. However later on we will see that token transfers
are best done through a pegzone and IBC. 

On the underlying child chain validators run a full geth node in order to track
the state of the Plasma contract. The Plasma contract also tracks the state of
the child chain. If someone wants to become a validator they have to submit
a pubKey and a bond to the Plasma contract. The top 100 of those are allowed
to attest events and relaying them to the child chain. This design becomes a 
lot cleaner when we can do header verification of Tendermint headers in
Ethereum.

The child chain runs at 1000 tx/s and the economic security of the stake is
derived from the token on Ethereum. Eventually it should be possible to submit
a proof to the contract to slash deposits from validators. 

Unbundling Plasma. Every Plasma chain is essentially a peg zone to the child
chain and a way to stake on Ethereum. With Tendermint we are unbundling those
two functions. We have one peg zone and multiple staked chains that can each
be independent.


## The major pieces

### Plasma Contract
* has a mapping from pubKey -> stake for validators
* has a mapping from pubKey (has to be present in the above) -> stake for
  delegators
* tracks child chain's validator set
* allows people to bond tokens
* allows for updates to the validator set if correctly signed by >2/3 of 
  current validator set
* accepts proof of double signing from anyone and slashes accordingly

### Tendermint Chain
* implements arbitrary transaction logic
* updates validator set based on messages from Ethereum Relay
* implements IBC to move tokens to the peg zone

### Ethereum Relay
* receives events from a trusted Ethereum full node
  * should run a full geth node
* controls the key registered in the Plasma contract under the validator 
  mapping
* signs messages to witness certain Ethereum events
* posts messages to Tendermint Chain

### Flow
Delegators bond on Ethereum and users delegate. Ethereum relay forwards that
information after a finality threshold. Validator set on child chain updates.
Tokens are moved in and out through the peg zone using IBC. 

## Version 2

### Plasma Contract
* as usual

### Tendermint Chain
* as usual
* opens a socket connection to Ethereum relayer
* queries the state of Plasma contract for validator set
* sets validator set in Tendermint

### Ethereum Relay
* runs full geth node
* exposes endpoint that returns current validator set in smart contract
* does not signing
