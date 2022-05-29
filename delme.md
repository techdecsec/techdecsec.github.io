---
layout: post
title: Sandglass &colon; Safe Permissionless Consensus 
subtitle: Youer Pu, Lorenzo Alvisi, and Ittay Eyal
tags: [consensus, permissionless] 
comments: false
---

# Title: ## title
# emph: _Sandglass_ 
# ref: [caption](#title). 

Classical consensus algorithms are _permissioned_. They allow a set of identified (authenticated) machines (nodes) to each _decide_ on a value, with three requirements: 
_Agreement_: No two machines decide on different values. It is a _Safety_ property – if two nodes decide different values it is violated. 
_Termination_: All nodes decide. It is a _Liveness_ property – it is only violated if at least one node never decides. 
_Validity_: If all nodes initially prefer some value, then that’s the decided value. This is also a safety property; it is necessary for the protocol to be useful, but prevents a trivial solution (“everyone always immediately decide 0”). 
The problem and solution vary depending on network assumptions (synchronous, asynchronous) and node fault types (ommission, crash, Byzantine). 

Read more in our [technical report](https://eprint.iacr.org/2022/XXX). 

<a href="https://twitter.com/IttayEyal" class="twitter-follow-button" data-show-count="false">Follow @IttayEyal</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
