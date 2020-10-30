# Dothereum

Dothereum is a **thought experiment** descirbing what eth1 -> eth2
transition would look like if it happens on Polkadot / Substrate. As a
thought experiement, **there's no plan to implement
Dothereum**. However, we develop and provide the rough specificiation
here as inspirations for both Polkadot and Ethereum ecosystems.

## Disclaimer

This is a side project of Wei Tang, and is not offically affiliated
with his employer.

## Overview

There are many sources of how the current eth1 -> eth2 transition plan
is going. For the purpose of this Dothereum specification, the goal is
not to dive into the details of the transition plan, or even to
understand it at all. Rather, we try to compare the overall framework
of the eth1 -> eth2 plan vs the Dothereum plan. After that, we provide
more details of how the Dothereum plan might happen.

### eth1 -> eth2 transition

The eth1 -> eth2 transition can be roughly descirbed as follows:

* At some point, an eth1 node implementation would bundle both the
  current eth1 implementation and an eth2 EE implementation. There is
  supposed to be an explict hard fork (transition) block, and the node
  will dispatch to different code paths based on that.
* Upon transition height, an eth1 block will be considered the final
  block on the eth1 side. Eth1 deposit is disabled in eth2 side at
  this height, and at the same time, the eth2 chain puts the
  post-state root of the final eth1 block into the eth2 EE.
  
From here on, the transition is finished, and everything happens at
the eth2 EE side.

This plan, if implemented correctly, will work great for the Ethereum
community. In the mean time, it's not without its downside. The major
one, from a developer's point of view, is that it requires getting
everything in place in one go. The eth2 EE must first be in place, and
then almost everything happens instantly at the transition block
height.

### Dothereum transition

The Dothereum transition can be roughly summarized as follows:

* We utilize the ability of Substrate to simulate other
  blockchains. Using techniques such as wrapper blocks and child trie,
  we **build a full node of eth1**. Note that at this stage there's no
  hard fork at all. It will simply be new Substrate nodes joining the
  eth1 network.
* Once this new full node is tested running stably on the network, we
  then initiate a hard fork that changes its consensus and adds other
  Substrate-basic utilities.
* From there, it becomes a normal Substrate-based blockchain, and can
  follow the normal procedure to join the Polkadot relay network as an
  auctioned parachain or a utility parachin.
  
The major difference, comapred with eth1 -> eth2 transition plan, is
that the Dothereum transition does not need to implement everything in
one go. We first simply build a full node of eth1 in Substrate, and
then gradually hard fork to add necessary Substrate functionalities
and prepare changes of its consensus, before finally becoming a
parachain. The whole process happens over a span of months or years,
instead of instantly at the transition height, to give developers
sufficient capacity to implement, debug and improve the whole process
along the way.

## Details

### Implementing an eth1 full node

Substrate has the ability to simulate other blockchains. As a result,
it is possible that we can implement an eth1 full node directly within
Substrate, without any hard fork of the eth1 network.

The Frontier project already provided users with the ability to
execute EVM contracts on the Substrate network, and simulate a great
portion of the Ethereum blockchain. To take this further, we need
three main techniques.

* **[Wrapper block](https://corepaper.org/substrate/wrapper/)**: This
  is a method that allows Substrate to process blocks that were not
  designed with Substrate in mind.
* **Child trie**: Substrate uses a different (default) merkle trie
  structure compared with eth1. It provides the child trie
  functionality, which allows other merkle trie structure (including
  the one eth1 uses).
* **Network stack**: We need a secondary *devp2p* network stack
  working along-side of the *libp2p* network stack. This can simply be
  an adapter -- a bridge that listens to messages on the devp2p
  network and transmute all of them as libp2p network messages.
  
### Adding functionalities and changing consensus

The above full node implementation in Substrate will only use a really
limited set of Substrate runtime functionality. In particular, to
preserve full compatibility with existing eth1 network, it cannot use
any of the Substrate Runtime Module Libraries (SRMLs). The runtime
only contains the raw code needed to fully process the wrapper blocks.

Once the node runs stably, the second step is to add additional
Substrate functionalities. In particular, we want to:

* Enable SRMLs, and pull in basic modules such as system, Substrate
  transaction processing, etc.
* Enable staking module, and change consensus from PoW to PoS (BABE +
  GRANDPA).
* Optionally enable democracy module to avoid frequent need of hard
  forks.
  
### Joining the relay network

Once the above is done, the network simply becomes a normal Substrate
blockchain. At this moment it is still standalone, and we can then
follow the normal procedure to actually join the relay network.

## Discussions

### User experiences

The Frontier project already shows that we can provide near full
Ethereum RPC compatibility in the Substrate framework. As a result,
for smart contract developers that interact with the network via RPCs,
their user experience would be nearly uninterrupted. For any second
layer solutions building on Ethereum but using a separate network
stack, their user experience would also be nearly uninterrupted.

The only exception would be those who directly rely on the devp2p
network stack. For them, they will have to adopt their softwares to
talk in a different network stack.

### State size

Ethereum has one of the biggest state in the whole blockchain
industry. As a result, no one knows if any other software can reliably
handle processing of the giant state. As no Substrate-based
blockchains have such giant state yet, right now we also do not know
if Substrate handles Ethereum state reliably. The same problem exists
for eth2 EE, as that will be new software not derived (or only
partially derived) from the existing eth1 node implementations.

However, on this point, the main advantage of Dothereum (compared with
eth1 -> eth2 transition plan) is that we can test and ensure the
realibility of state handling before continuing on further steps, as
the initial step of building an eth1 node in Substrate requires no
hard fork nor any disruptions on the eth1 network.
