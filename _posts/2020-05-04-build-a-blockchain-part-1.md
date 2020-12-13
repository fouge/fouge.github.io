---
layout: post
title: If you want to know more about blockchains, build one.
description: Learning by doing. Why did I choose the Substrate framework to learn more about blockchain technologies?
date: 2020-05-04
author: Cyril
meta: 
  - tag:
    title: learn
    class: learn
---

Following [my last post](https://www.cyrilfougeray.com/2020/04/20/IoT-knowledge-graph.html) regarding where I am heading in my professional career, I said I wanted to learn more about blockchains, and now I am telling you I am building a blockchain. 

What? 🤯

---

I already made some progress by looking at the current state of the art in blockchain technologies. I read some documentation here and there and played with a client made for IoT [by Slock.it called Incubed](https://slock.it/incubed/). It is very interesting to see how difficult it is to connect resource-constrained devices. But that's not the subject for today...

As you know, it all started with Bitcoin back in 2009. I already know a lot about Bitcoin but I never read the white paper. As for many other things, I think it's important to start with the beginning so I am going to read that important paper first. Learning the theory is great to start but I feel that I cannot fully understand if I don't put my hands in it.

## Learning by doing

Nowadays, distributed applications are mainly based on Ethereum because it offers the possibility to issue new tokens based on the Ethereum standards (ERC-20 for example) using distributed programs called _Smart Contracts_. The more applications built on this blockchain, the more will be built on it. [Ethereum is an emergent structure](https://bankless.substack.com/p/ethereum-is-an-emergent-structure?r=1ua7a&utm_campaign=post&utm_medium=web&utm_source=copy) (👈 by Bankless 👌).

Ethereum is here to stay but while I was learning about the basics of how data is being transacted on the Ethereum blockchain, I stumbled upon [Parity](https://www.parity.io/), co-founded by Gavin Wood who is also a cofounder of Ethereum with Vitalik Buterin. Parity has been developing the source code behind an [Ethereum node](https://github.com/openethereum/openethereum) until recently and is now focusing on two products: Substrate and Polkadot.

<figure class="col-md-12">
  <img src="/img/posts/build_blockchain/polkadot-substrate.png" alt="Polkadot and Substrate logos" class="img-responsive">
</figure>

You probably never heard about those, so you might be wondering why are they so important?

I heard about Polkadot by Eric Larchevêque from Ledger in a podcast I listened to. At that time, I didn't know anything about Polkadot but only took note of it without going much further. Now that I took a closer look, I can say that those 2 products are very interesting for me who wants to learn more about the blockchain technologies.

The goal of Polkadot is to make blockchains interoperable so that any type of data or asset can be transferred among the various blockchains. While building Polkadot, the experienced team at Parity thought that all those software bricks could be made available to anyone as generic libraries for them to build other blockchains, so they built Substrate.

This quote is directly taken from [the website](https://www.parity.io/what-is-substrate/):

> "Substrate is a framework for creating cryptocurrencies and other decentralised systems **using the latest research in blockchain technology**". 

*ie* Substrate builds blockchains like Polkadot which is now fully based on Substrate. Who was born first? 🐣🙂

Polkadot is very interesting because it is using the best of blockchain technologies. For example, it is governed by its users which ensure that evolutions will come smoothly and that it won't be forked which is one of the main arguments for another very interesting project I am following closely: [Tezos](https://tezos.com/). It will be interesting to see how they compare.

Substrate is now providing developers the tools to create blockchains compatible with others based on the framework. They can all run on Polkadot.

From a technological standpoint, the framework is written in Rust and I want to improve in that language. Runtimes (*ie* Smart Contracts) are compiled to WebAssembly (Wasm) so that they can be run on virtually any hardware. Wasm is really young but as [Fredrik (Parity's CTO) puts it](https://www.crowdcast.io/e/sub0-online/2): Wasm is future-proof.

<figure class="col-md-12">
  <img src="/img/posts/build_blockchain/dg-stack.svg" alt="Polkadot and Substrate logos" class="img-responsive" style="height:250px">
  <figcaption><a href="https://polkadot.network/technology/" target="_blank">Polkadot technology stack</a></figcaption>
</figure>

Frameworks are great when they are being used by developers so the documentation available is extensive and neat, which is a really good point for me. Many videos are available and they just broadcasted the Sub0 conference last week, [available online](https://www.crowdcast.io/e/sub0-online) and I can also watch the one from last year.

All this is looking good, I'm pretty excited to start. I already have several questions I want to answer:

> What are the actual benefits of making interoperable blockchains against an ERC-20 token?

My guess is that building on Ethereum can be constraining as the token is dependent on the Ethereum platform, nodes, congestion, etc. But then arises that new question:

> How much can we configure the new blockchains created with Substrate?

I guess a lot. If those are totally configurable, it means I will have to learn a lot from all the different configurations and features a blockchain can use. From what I have seen, Parity made choices that imply ousting configuration options, but since I am not an expert, I am not able to complain and the team behind the project seems to take thoughtful decisions.

## Conclusion

Substrate is making the task of getting the hands dirty way easier than a few years back, while still being challenging, which makes it a great alternative for me to dive into the blockchain technologies. I used to read articles treating blockchain randomly but by leveraging Substrate, I will improve my learning curve.

Make sure to follow along if you are interested in learning more about blockchains as well.

👋