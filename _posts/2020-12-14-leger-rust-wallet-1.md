---
layout: post
title: Connect your device to blockchains with Léger.
description: Léger is a Rust library I am developping to make it easier for anyone to connect devices to Blockchains.
date: 2020-12-14
author: Cyril
meta: 
  - tag:
    title: blockchain
    class: learn
  - tag:
    title: rust
    class: optimized
---

Introducing [Léger](https://github.com/fouge/leger-rs). &#129718; *Léger* means *lightweight* in french, not to confound with Ledger, the French company 🙂.

---

I did not realize that 7 months have passed since my [article about building a blockchain](https://www.cyrilfougeray.com/2020/05/04/build-a-blockchain-part-1.html)  😮. I started to learn new things about blockchains last spring but then did a lot of freelancing work. The past month has been calmer so I decided to jump back in by building a library for embedded systems that will provide easy ways to connect to Substrate-based blockchains.

## Motivations

On one side, there is the attractiveness of the promise. I already explained in [a past article](https://www.cyrilfougeray.com/2020/04/20/IoT-knowledge-graph.html) that I wanted to get closer to the world of blockchain technologies, specifically for the Internet of Things. Data can be sold and devices are harvesting data 24/7 so it will eventually attract people. The "Economy of Things" is becoming a reality and blockchain technologies will undeniably be part of that (r)evolution. There are plenty of use cases. To name one: weather stations selling data to meteorological services, that would bring more accurate weather to the farmers.

> Picture a world where devices will be able to automatically transact data. 💸

On the other side, there is a technical challenge, and I had two main goals while building this library: learn how to connect a device to blockchain APIs and improve my embedded-Rust skills. I finally ended up with a third one: making the source code public.

## Blockchain APIs

Most blockchains provide JSON-RPC endpoints: send the method and parameters you want as JSON text. Regarding the wide number of actions you can make through the API in order to interact with the blockchain, JSON-RPC might be more practical than a REST API. Those actions include new transactions, getting the state of the storage, and getting historical, block-related, data. The [Substrate node template](https://github.com/substrate-developer-hub/substrate-node-template) gives access to 75 methods by default, without counting the *extrinsics* ie. functions that are used to store data into blocks.

JSON-RPC has become a crypto-industry standard and people at Parity decided to follow that path as well. The endpoints are accessible through both HTTP and Websocket. Websocket being the preferred choice as you can receive notifications from the server without having to ask.

I cannot hide that those standards are a bit heavy (not *léger* 😛), for our embedded devices running on microcontrollers.

To submit extrinsics to a Substrate chain, the wallet has to pack data into a specific structure that needs to be known from both ends, with some fields that can be [SCALE](https://substrate.dev/docs/en/knowledgebase/advanced/codec)-compacted to reduce the size, which is great... until you realize that to send the structure to the JSON-RPC endpoint, you have to convert the byte array into a string of hexadecimal characters which takes twice the amount of memory used to store the array! From a firmware developer point of view, *this is a shame!* 😄. Ok, I am being tough...

I think that some protocols such as [gRPC](https://grpc.io/) which uses Protobuf can give great advantages for any blockchain technology to welcome IoT devices. Cosmos has implemented that protocol while Algorand lets you request data as JSON or MessagePack depending on what you want to use. Those are the first steps to a more efficient blockchain interface 🚀. I hope Parity will eventually implement those features to please the Firmware developers around the world, but clearly, I didn't expect the blockchains to implement that kind of lightweight protocol.

It won't be perfect but let's continue. We learned that the library needs two things to work: a Websocket client and a JSON (de)serializer. Let's start coding!

## Rust

This is not the first Rust library I am writing but I previously wrote libraries for signal processing which is only dependent on the CPU.

The Léger library is more dependent on the lower layers in at least two specific domains: the networking stack and key management. Those layers don't have to, and should not, be handled by the library if we want to give the developers the freedom to implement the network stack as they want and enjoy the safety and efficiency of a secure element. Thus, I implemented traits that have to be implemented for the developer to use the library. I won't explain how here but curious minds can have a look at [the example](https://github.com/fouge/leger-rs/blob/master/examples/unix.rs) available on the Git repository.

Rust has the advantage of providing libraries called *crates* that can easily be imported into a project. There are many available on crates.io but I needed `no-std` crates (crates that don't rely on the standard library which is not available on embedded devices). Here are the ones I am using:

- [`embedded-websocket`](https://crates.io/crates/embedded-websocket): a `no-std` library to encode/decode Websocket messages.
- [`embedded-nal`](https://crates.io/crates/embedded-nal): provides a standard interface for network-related stuff: TCP or UDP connection and sending/receiving data. Thanks to [Ryan](https://github.com/ryan-summers), [Mathias](https://github.com/MathiasKoch), and [Chris](https://github.com/caemor) who are working on that crate for helping with my Rust issues 🙌.
- [`serde`](https://crates.io/crates/serde) : a framework to serialize and deserialize data struct, here JSON structures using [`serde-json-core`](https://crates.io/crates/serde-json-core).
- [`hex`](https://crates.io/crates/hex): to encode/decode hex string to byte arrays. As I said, extrinsics are compacted into byte arrays that are converted to ASCII strings to be passed using JSON-RPC. At the moment the crate needs a global allocator, which I don't want to implement on an embedded target so I am using a fork.
- [`blake2-rfc`](https://crates.io/crates/blake2-rfc): to hash bytes using the  Blake2 algorithm.

Those provide most of what I need to talk to my Substrate node. 😎

To kick start the project, I used a [Substrate wallet written in Kotlin](https://github.com/NodleCode/substrate-client-kotlin/) developed by the Nodle team as a basis. This one is pretty simple and has limited features but that was exactly what I needed to start. I quickly understood the overall architecture and was able to adapt it to Rust.

I started with the most simple features like asking the genesis block hash and added an account to finally being able to submit extrinsics that would give the ability to send money from the wallet account to any public account (using the public key).

⚠ I tried to keep the library as efficient as possible by reusing buffers and optimizing the CPU workload, but at this point, the library has not been tested on an embedded device, yet. Still, I implemented [an example](https://github.com/fouge/leger-rs/blob/master/examples/unix.rs) running on Unix that is making use of the `no-std` library to fetch Substrate information and send money to an account. The example is made to work with the Substrate Node template and will have Alice send money to Bob. 🤶

As I told you, I wanted to strengthen my Rust skills so any review or comment would be really appreciated.

This example is just the beginning and more features are coming.

## Going forward

I have some more work before Léger can be used by developers without too much hassle. Unfortunately for Léger, I've got new freelance work coming up for the next few months but I want to keep improving the library though, to start with:

- Publish on crates.io, at the moment I have a dependency (`hex`) that is not published as a crate yet as I am using a fork, which is preventing me from publishing my library as a crate. I'll have to improve the documentation a bit as well.
- Make it work on an embedded device, obviously! I recently bought an nRF91-Thingy which has cellular connectivity and my goal is to make my wallet work from that device. I am looking forward to it!
- Add an optional feature to make the TCP interface call C functions so that anyone can build the library as an archive and link it into an application without having to write Rust code.
- Add RPC calls and extrinsics support.
- Improve error handling: I need to rightfully design the returned errors so that developers can easily understand what's wrong and how to respond to any error occurring in the library.
- Add providers (Cosmos or Algorand?). I contacted Ted, working on [Algoduino](https://algoduino.com/), a wallet for embedded devices to work with Algorand. We might end up joining our forces 💪.
- Publish a fully working example for the nRF91 SoC would be great!
- Write a new crate to handle private keys and signing (ed25519 and sr25519)
- Write a new crate that implements `embedded_nal` for the nRF91. *Rust is eating the embedded world* 😄.

If you want to implement new features, let me know by [opening an issue or commenting on existing ones](https://github.com/fouge/leger-rs/issues).

## Conclusion

Freelancing hasn't always been as I expected it to be. I initially planned to have one day a week reserved for my side projects but it's hard to tell customers that I won't deliver as fast as they want because I prefer to spend my time on personal non-paid work. Anyway, that's another subject.

I really enjoyed having almost a month of free time though, to work on that side project. I hope this will serve others and as I said, I'll keep working on it so stay tuned!

👋