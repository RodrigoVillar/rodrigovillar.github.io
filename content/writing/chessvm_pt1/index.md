+++
title = "ChessVM Part I: What is a Blockchain?"
date = 2024-01-24

[taxonomies]
tags = ["VMs", "Avalanche"]

[extra]
lang = "en"
toc = true
comment = false
copy = true
math = true
mermaid = true
outdate_alert = true
outdate_alert_days = 120
display_tags = true
truncate_summary = false
featured = false
+++

_Special thanks to [Martin Eckardt](https://x.com/martin_eckardt?s=20) for giving me the idea for this series._

For the past nine months, I have been (somewhat) obsessed with the concept of
building a custom virtual machine (VM). Having taught a blockchain development
course at Cornell for some time and being familiar with the paradigm of smart
contracts, I was interested in the concept of building a VM with a specific use
case in mind, in contrast to other VMs like the Ethereum Virtual Machine (EVM)
which allow for the hosting of arbitrary computable programs.

What I didn't realize at the time, however, was that this process would
completely break down my understanding of blockchain as a whole. Furthermore,
transitioning from writing smart contracts with Solidity/Yul to working in an OS-like
environment was perhaps one of the most difficult engineering challenges I've
encountered. At the time that I am writing this, I am attempting (for a third
time) to build a Rust-based custom VM which can be deployed on an Avalanche
blockchain.

Although building a custom VM has presented to me a number of challenges the
past couple of months, I am now beginning to see the light at the end of the
tunnel. I believe that my understanding of blockchain is now more in line with
the literature of distributed systems (the field which blockchain builds upon)
and the process of building a custom VM is no longer an alien task to me. But rather than writing a massive Medium
article stating the accomplishment of building a custom VM, I am instead going
to create a series of writings which aim to accomplish the following:

-   Establish what I believe a blockchain is
-   Discuss the development process of building a VM and some takeaways

These series of writings will revolve around _ChessVM_, a custom VM whose
architecture is designed for the creation and playing of Chess games on the
blockchain. Before talking about _ChessVM_, or even how to go about implementing
a virtual machine, I will start off with what I believe to be one of the biggest
misconceptions in the industry:

## Is a "blockchain" really a blockchain?

I will first start off with a definition of what a blockchain is from the CS1998
course at Cornell (formerly taught by [Daniel Mistrik](https://x.com/DanielMistrik?s=20), a great friend of mine):

{% quote() %}
A _blockchain_ has the following properties:

-   Data is ordered into blocks and structured by Merkle Trees.
-   Blocks are united through a linked list.
-   Blocks are added on a regular basis to the linked list and must be valid to be added.
-   Everyone has an identical version of the above and can propose new blocks that should be added.
    {% end %}

Visually, we can think of blockchains as the following:

{% mermaid() %}
classDiagram
direction LR
Block 0 --|> Block 1
Block 1 --|> Block 2
Block 2 --|> Block 3

    class Block0 {
        TX1
    }

    class Block1 {
        TX2
        TX3
        TX4
    }

    class Block2 {
        TX5
        TX6
    }

    class Block3 {
        TX7
        TX8
    }

{% end %}

Easy enough, right? In distributed systems like Bitcoin where blocks contain
lists of transactions (in addition to other data fields like timestamp, nonce,
etc.), it's easy to claim that the blockchain _is_ the system. Want to send
bitcoin to another user? Just create a transaction and make sure it is included
in a confirmed block. Want to query your account balance? Just start at the
genesis block and execute all transactions that involve your account.

However, this system starts to break
down once we introduce the concept of programs (i.e. smart contracts). Users of
Ethereum and other blockchains which support programs are familiar with the notion of
deploying smart contracts which they can read/write from. However, where do
these programs live? If you look at the content of the blocks under these types
of systems, you will see that there is no such concept of a program "living" in
a block. At this point, there seems to be two worlds within the
construct that we call a blockchain:

{% mermaid() %}
classDiagram
direction LR
Block 0 --|> Block 1
Block 1 --|> Block 2
Block 2 --|> Block 3

    class Block0 {
        TX1
    }

    class Block1 {
        TX2
        TX3
        TX4
    }

    class Block2 {
        TX5
        TX6
    }

    class Block3 {
        TX7
        TX8
    }

{% end %}
{% mermaid() %}
flowchart LR
id1[Program 1]
id2[Program 2]
id1 ~~~ id2
{% end %}

The realization above makes it obvious that we need to look at blockchains from
a different perspective, one that makes the use case of blockchains
obvious. We do not necessarily need to change the technical properties of the
blockchain data structure but rather, re-examine the relationship between the
blockchain data structure and the networks that utilize said data structure.

## Blockchains <> Databases

For this next section, I will refer to the following function from the [Ethereum Yellowpaper](https://ethereum.github.io/yellowpaper/paper.pdf):

$$
\sigma_{t + 1} \equiv \Pi(\sigma_t, B)
$$

where $\Pi$ is the "block-level state-transition function," $B$ is an
arbitrary block, and $\sigma$ is the state of the network at a given time $t$.
Here, we see that rather than the block being treated as the principal value, it
is treated as an argument to produce something more significant: _state_. With
the equation above as the state transition function of the distributed
computer representing the network, we see that blocks and therefore, blockchains are data structures which dictate the state transitions
of a computer. In the case of networks which utilize the EVM, blocks (and their
transactions) update the world state, whether it be something as simple as
sending coins between accounts or as complex as creating an entire
decentralized exchange.

The virtual machine (i.e. the computer responsible for handling the execution of
the network), therefore, needs to
store both the blockchain and its state somewhere. This brings us to a
quote that I like quite a lot:

{% quote(cite = "Patrick O'Grady, VP of Engineering @ Ava Labs") %}
[Virtual machines] are thin [database] wrappers
{% end %}

The networks that we consider as "blockchains," in reality, are just distributed
databases that are updated using a blockchain. Virtual machines, then, are (at a
fundamental level) responsible for updating the database upon recieving a new
block. What does _updating the database_ entail? Well, this is up to the
implementation of the VM! For example, I can create a "blockchain" which keeps
track of a name. The associated virtual machine, _NameVM_, upon recieving a new
block, grabs the name stored in the block and updates the name stored in the
database.

{% mermaid() %}
classDiagram
direction LR
Block 0 --|> Block 1
Block 1 --|> Block 2
Block 2 --|> Block 3

    class Block0 {
        Name: Rodrigo
    }

    class Block1 {
        Name: Mac
    }

    class Block2 {
        Name: Martin
    }

    class Block3 {
        Name: Luigi
    }

{% end %}

_NameVM_, although elementary, shows that rather than using general-purpose VMs,
we can design VMs with a specific use case in mind. At a fundamental level, all
networks which utilize blockchains have the following architecture:

{% mermaid() %}
flowchart LR
id1[(Database)]
id2[Blockchain]
id2 --> id1
{% end %}

With a custom VM, we can add to the architecture above; in the case of
_ChessVM_, we have the following architecture (in the next post, we will see an
alternate version which is _much more intricate_):

{% mermaid() %}

---

title: ChessVM

---

flowchart LR

subgraph sg1["Chess State"]
g1["Game 1"]
g2["Game 2"]
gn["Game ..."]
end

id1[Blockchain]
id2[(Database)]

id1 --> id2
sg1 --> id2

{% end %}

## Rust-SDK (or Avalanche-RS)

[Avalanche-RS](https://github.com/ava-labs/avalanche-rs) is a repository which,
among other things, provides Avalanche-Types. This crate (which I will refer to
as the "Rust SDK" for the rest of this series) is an SDK that, for the most
part, provides us with the tools necessary to build a VM which we can then
deploy on an Avalanche blockchain. I will be using the Rust SDK to build _ChessVM_.

## Wrapping Up (For Now)

In the following posts, I will be discussing the architecture of ChessVM
alongside the behavior of the VM itself. Furthermore, while I stand by my
commentary of what a virtual machine is, I will expand upon my definition (hint:
VMs are also servers!).

\- rjv
