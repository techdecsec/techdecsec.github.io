---
layout: post
title: LedgerHedger &colon; Gas Reservation for Smart-Contract Security
subtitle: Itay Tsabary, Alex Manuskin, and Ittay Eyal
tags: [hedging, gas, transaction fees, game theory, cryptocurrency, smart contracts] 
comments: false
---



<ins>TL;DR</ins>: Prominent smart contracts, e.g., roll-ups, critically rely on timely confirmations of their transactions.
Sadly, that's not how blockchain works, as confirmation times depend on transactions fees, where the required fee is determined by the volatile fee market.
We present _LedgerHedger_, the first smart contract that facilitates a reservation for a future transaction confirmation. 
_LedgerHedger_ is secure, incentive-compatible, and has low overhead for practical future-transaction parameters.



We start with some blockchain [background](#preliminaries), discuss current transaction confirmation [modus operandi](#a-transaction-just-wants-to-be-confirmed) and its shortcomings, present how [regulated markets](#a-trip-in-regulated-markets) overcome volatility and the inadequacy to blockchains, and present our _LedgerHedger_ [design](#ledgerhedger-design) and [Solidity implementation](#ledgerhedger-solidity-implementation).
You can find the full details in the [technical report](https://eprint.iacr.org/2022/056).


## Preliminaries 

Ethereum uses an append-only log called the _blockchain_.
It comprises elements called _blocks_, and parsing the log results with the system state, i.e., how much cryptocurrency everybody has.

Entities called _miners_ create blocks, using methods like PoW or PoS (doesn't matter for our context today), and include transactions in these blocks.
Only included transactions affect the cryptocurrency possessions.

Transactions consume _gas_, an internal measure of their complexity.
The system limits how much gas can be consumed by transactions in a single block, hence miners pick only subset of the available transactions.
Transaction _issuers_ assign fees to their transactions to incentivize miners to confirm their transactions.
Miners prioritize transactions according to their fee-per-gas ratio.

It follows block gas is a scarce resource, hence a fee market forms.
For each block there is a gas _market price_, the minimal fee-per-gas ratio required for having a transaction confirmed.
The varying demand results with a volatile market price, which can even double itself within a day.


## A Transaction Just Wants to Be Confirmed

Now, say you are an Ethereum user and you want your transaction confirmed in the next block.
For that, your offered fee simply needs to meet the market price, which you can quite easily determine for the next block using services like the [gas station](https://ethgasstation.info/).
Follow their advice and you are usually golden.


But, what if you want your transaction to be included in a future block interval, say, during a specific afternoon next Thursday? 
How can you predict the market price a week ahead?
And what happens if you miss?

<!-- ![](/assets/img/ledgerHedger_1.png) -->

<div style="text-align:center">
{% include image.html url="/assets/img/ledgerHedger_1.png" description="Unlike Internet quotes, transactions can be confirmed (for a sufficient fee) [<a href='https://www.someecards.com/usercards/viewcard/MjAxMi04NTdhYjEzNjE2MTZmM2Y0/amp/'>source</a>]." %}
</div>

 
As it turns out, these questions are more than a theoretical experiment, but actually decisions that system operators face on a daily basis.
These include any system that relies on its transactions being confirmed in a timely manner, e.g., optimistic and ZK roll-ups, atomic swaps, state channels, contingent payments and so forth. 
Even more crucially, the safety and liveness guarantees of these systems rely on a timely confirmation of their transactions; failing in that can result with significant cryptocurrency thefts. 

Nowadays, these systems operate in a naive manner -- they defer worrying about the future confirmation to its due date. 
Specifically, they take actions at present times assuming the market price does not surge.
If that assumption fails then the system operator either has to incur the unexpected additional expense, or forfeit the confirmation and the system guarantees.

## A Trip in Regulated Markets

Well, this future-price-prediction-or-we-go-out-of-business is not unique to cryptocurrencies and transaction fees, there are plenty of volatile markets out there.
For example, airlines are dependent on the rather-volatile oil prices, where a price spike can put them in a hole.
To overcome this volatility, airlines and oil suppliers often engage in a _hedging contract_, where they agree on a deal at future time frame for a predetermined price.



<div style="text-align:center">
{% include image.html url="https://images.treasuryandrisk.com/contrib/content/uploads/sites/411/2017/07/2017-07-12_Hedge-currency_616x372.jpg" description="Another Type of Hedging [<a href='https://www.treasuryandrisk.com/2017/07/13/new-hedge-accounting-standard-flashes-green-light/'>source</a>]." %}
</div>



Could this resolve our future gas needs in a blockchain? 
Can we have hedging in cryptocurrencies?

Well, turns out this is a bit tricky: airlines and oil suppliers operate in regulated markets, i.e., there is an external enforcer (court) that can make sure both parties comply with the contract.
In cryptocurrencies, the miners are the enforcers, and they (and only they) decide what transactions are confirmed.
With soaring cryptocurrency prices, the stakes are too high for naively relying on miner altruism, and there is a need for a robust solution.
Moreover, who should the hedging contract be with -- various miners produce blocks in the target interval. 

Enters _LedgerHedger_.

## LedgerHedger Design

_LedgerHedger_ takes a different approach -- it _incentivizes_ the correct execution rather than enforcing it.
We set a hedging contract between a single miner and the transaction issuer. 
This enables _LedgerHedger_ to have only a relatively-low overhead -- there is no need for elaborate proofs of misbehavior.

_LedgerHedger_ operates in two phases.
At the _setup_ phase, the gas-purchasing user (the _Buyer_) sets the contract parameters, namely the target confirmation interval, the required gas, and payment details.
_Buyer_ also locks the payment upfront, and then the gas-selling miner (the _Seller_) can accept the contract by depositing a collateral of her own.
At this point _Buyer_ does not commit to a future transaction.

At the target _execution_ phase, _Buyer_ publishes a (zero fee) transaction of her choice, which _Seller_ can then confirm through a designated _apply_ function.
_LedgerHedger_ verifies the provided transaction was indeed created by _Buyer_, and if it executes successfully, it sends the funds to _Seller_.

But what happens if _Buyer_ publish a transaction that exceeds the agreed-upon gas amount, or a transaction that fails, or maybe does not even publish a transaction at all?
For that there exists an alternative function, _exhaust_, that enables _Seller_ to extract the funds without any corporation from _Buyer_.
However, to prevent _Seller_ abusing this function, its invocation spends gas equal to the agreed-upon amount through repeated null operations. 
This construction results with the _apply_ function being preferred in case _Buyer_ abides by the contract, while protecting _Seller_ if not.


To analyze _LedgerHedger_ we first consider what are the possible interactions _Buyer_ and _Seller_ can have with it -- who can do what, and when.
These, in turn, give rise to a game, which we analyze using the _subgame perfect equilibrium_ solution concept.
Our analysis confirms that fulfilling the contract as intended is the subgame perfect equilibrium for a wide range of practical parameters.


## LedgerHedger Solidity Implementation

We can only hope that Ethereum-savvy readers have not pulled out their pitchforks yet -- our description above of the _apply_ function does not address the minimal base-fee required by [EIP1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md), which prevents zero-fee transactions.
But worry not, _LedgerHedger_ circumvents this requirement by utilizing [_meta transactions_](https://medium.com/@austin_48503/ethereum-meta-transactions-90ccf0859e84), decoupling the transaction _creator_ (in our case, _Buyer_, who decides what the transaction does) from the transaction _issuer_ (i.e., _Seller_, who pays the necessary fees).

For the _exhaust_ function, _LedgerHedger_ expends the required gas by performing null operations, specifically, by increasing a counter sufficiently many times.

We implement _LedgerHedger_ to be reusable for the _Buyer_, amortizing the contract deployment cost.
This, along with the function invocation costs result with an overhead of roughly 50K gas per usage.
Considering a practical value of [10M](https://etherscan.io/tx/0x90ebd9630d98d5b0a186eec4c2382c296e5f41e828da910d76a53ab72ffe30e8) for a roll-up proof, the overhead is orders of magnitude lower.

We bring the full analysis, implementation and deployment details in our [technical report](https://eprint.iacr.org/2022/056).
You can also find the contract code in our [repository](https://github.com/amanusk/LedgerHedger-contracts).

**Use _LedgerHedger_ at your own risk. We did not have _LedgerHedger_ audited. We take no responsibility for any possible vulnerability, error, technical issue or bug.**



<a href="https://twitter.com/ItayTsabary" class="twitter-follow-button" data-show-count="false">Follow @ItayTsabary</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> <br/>
<a href="https://twitter.com/amanusk_" class="twitter-follow-button" data-show-count="false">Follow @Amanusk</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> <br/>
<a href="https://twitter.com/IttayEyal" class="twitter-follow-button" data-show-count="false">Follow @IttayEyal</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

