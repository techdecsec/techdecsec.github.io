---
layout: post
title: Safe Permissionless Consensus 
subtitle: Youer Pu, Lorenzo Alvisi, and Ittay Eyal
tags: [consensus, permissionless] 
comments: false
---

Classical consensus algorithms are _permissioned_. They allow a set of identified (authenticated) machines (nodes) to each _decide_ on a value, with three requirements: 

1. _Agreement_: No two machines decide on different values. It is a _Safety_ property – if two nodes decide different values it is violated. 
2. _Termination_: All nodes decide. It is a _Liveness_ property – it is only violated if at least one node never decides. 
3. _Validity_: If all nodes initially prefer some value, then that’s the decided value. This is also a safety property; it is necessary for the protocol to be useful, but prevents a trivial solution (“everyone always immediately decide 0”). 

The problem and solution vary depending on network assumptions (synchronous, asynchronous) and node fault types (ommission, crash, Byzantine). 

Nakamoto’s seminal Bitocin paper demonstrated for the first time that it is possible to reach a variant of consensus in a _permissionless_ setting: When and and all servers can join and leave without notice. This is in contrust to classical consensus models, where the set of participants is known and can only be changed by running an explicit reconfiguration protocol. However, this came at a cost – rather than guaranteeing safety and termination properties, Bitcoin only achieves them _with high probability_: there is some small probability $\varepsilon$ that the requirements would be violated, including safety. This applies to all permissionless blockchain protocols we are aware of, both practical and theoretical. 

This raises the question:
> Can a protocol provide deterministic safety guarantees in a permissionless model? 

To achieves consensus in a permissionless setting, Nakamoto forsakes explicit majority voting (common in classical algorithms) and relies instead on a _Proof of Work_ (PoW) lottery mechanism, designed to drive agreement towards the blockchain whose construction required the majority of the computational power of all participants. Lewis-Pye and Roughgarden have recently shown that when basing security on a PoW, as in Bitcoin, there cannot be a consensus algorithm with deterministic safety guarantees. 

In a technical report we publish today, we show that the safety limitation does not arise from the model assumptions but rather from the choice of a probabilistic PoW mechanism. In Bitcoin (and all PoW blockchains), the miners provide statistical proof they have calculated a certain number of has functions. This implies that with some probability, even a small miner might generate the majority of blocks, violating the basic assumptions necessary to achieve security. We show that if instead we can send _all work_, achieving a full (rather than statistical) PoW, it is possible to achieve _deterministic fairness in a permssionless model_. 

We present _Sandglass_, a permissionless consensus algorithm that guarantees deterministic agreement and terminates with probability~1. It operates in a model based on Nakamoto's. Our model allows an arbitrary number of participants to join and leave the system at any time and stipulates that at no time the number of participants exceeds an upper bound N (though the actual number $$n$$ of participants at any given time is unknown). Further, like Nakamoto's, it is hybrid synchronous, in that, at all times, a majority of participants are correct and able to communicate synchronously with one another. We call these participants good; our protocol's safety and liveness guarantees apply to them. Participants that are not good (whether because they crash, perform omission failures, and/or experience asynchronous network connections) we call {\em defective}. 

Sandglass proceeds in virtual rounds. Nodes propose a value by broadcasting it;  in the first round, each node proposes its initial value; in subsequent rounds, nodes propose a value chosen among those received in the previous round. Values come with an associated priority, initialized to 0. The priority of $v$  depends on the number of consecutive rounds during which $v$ was the only value received by the node proposing $v$ -- whenever a node receives a value other than $v$, it resets $v$'s  priority back to 0. 
When proposing a value in a given round, node $p$ selects the highest priority value received in the previous round; if multiple values have the same priority, then it selects randomly among them. 
A node can safely decide a value $v$ after sufficiently many consecutive rounds in which the proposals it receives unanimously endorse $v$ (i.e., when $v$'s priority is sufficiently high); and termination follows from the non-zero probability that the necessary sequence of unanimous, consecutive rounds will actually eventually occur.

The keen eye migh notice that Sandglass is surprisingly reminiscent of Ben-Or's classic consensus protocol. However, Ben-Or establishes that it is safe for a node to decide $v$ after having observed two consecutive, unanimous endorsements of $v$. This is because any two majority sets of its fixed set of $n$ nodes will intersect in at least one correct node. 
This approach is clearly no longer feasible in a permissionless setting, where $n$ is unknown and the set of nodes can change at any time. Instead, Sandglass’s approach to establish safety is inspired by one of the key properties of  Nakamoto's PoW: whatever the value of $n$, whatever the identity of the nodes participating in the protocol at any time,  the synchronously connected majority of good nodes will be faster, in increasing priority than the defective nodes. So much faster, than eventually defective nodes either propose the same value as the good nodes, or their priority is so far behind it cannot affect the good nodes. 

The full details are in our [technical report](https://eprint.iacr.org/2022/XXX).

<div style="text-align:center">
{% include image.html url="/assets/img/ledgerHedger_1.png" description="beautiful image [<a href='https://www.someecards.com/usercards/viewcard/MjAxMi04NTdhYjEzNjE2MTZmM2Y0/amp/'>source</a>]." %}
</div>
