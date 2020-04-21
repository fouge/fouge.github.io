---
layout: post
title: My uniqueness in the Internet of Things playground.
description: Discover all the technical fields behind the Internet of Things. What do I know? What do I want to know? Interact with the graph to make it your own.
date: 2020-04-20
author: Cyril
meta: 
- freelancing
- IoT
---

## The Blue Ocean strategy

It's been one month. One month of containment due to the spreading of the coronavirus, but also for me, one month in the full-time adventure of _freelancing_. It's the beginning and I am already grateful to have plenty of things to do during those special times. I came back from 11 months of travel 2 weeks ago and my girlfriend and I are now at my parents' house in the countryside, enjoying the space we have 🐮🤠.

Working from home was initially the plan for me anyway. Containment gives even more time to setup everything and I try to take full advantage of it. Being a freelancer is my first step into entrepreneurship. I am asking myself many questions about how to communicate and how to sell my skills to convince my ideal customers to contact me (and because I actually like the science of marketing 📈). I read articles and listen to podcasts to discover methods like content strategy or presence on social networks. My first steps were to create that website and I am willing to publish new articles regularly here.

Yesterday, I listened to [an episode of Young Wild and Freelance](https://thomasburbidge.com/creer-metier-toute-piece/) (🇫🇷), where Thomas and Romain are talking about finding - or even creating - the market area where you want to be top of mind ([TOMA](https://en.wikipedia.org/wiki/Top-of-mind_awareness)); meaning that if anyone is looking for someone like you, you will be the first answer in the mind of the people. They even give the perfect example for french people (and maybe other countries as well): _What do you do if you have an impact on your car window?_ To find your domain, Thomas talks about [the Blue Ocean strategy](https://en.wikipedia.org/wiki/Blue_Ocean_Strategy) which aims to create and capture "blue oceans": unexplored new market areas. With this technique in mind, they advise us to list 10 domains you would use to describe what you do. From that list, choose one and do the same exercise, again and again, to finally define a specific area that could define you as _a unique person_. 

I took a pen and paper and tried to list sub-domains defining my value proposition: building ready-to-ship connected devices, _ie_ the Internet of Things (IoT). Around that domain I found many skills I developed along the years working on devices, some I need to perfect and others that I want to learn and practice. I plotted that graph and I wanted to share it here. Don't be stuck on that graph, below comes some explanation, and a little surprise for you 🎁.

<figure class="col-md-12">
  <img src="/img/posts/iot_knowledge_graph/my_graph.png" alt="My IoT Knowledge Graph" class="img-responsive">
  <figcaption>My IoT Knowledge Graph</figcaption>
</figure>

## The Internet of Things and I

### What makes the IoT?

The graph is a set of domains and sub-domains I think are together making the Internet of Things. Nodes could be added or reorganized while several of them can also be the root of a new graph. The science of quantification, storage and communication of information called _Information Theory_ is for example made of Sensors, Stream-Processing and Communication. Possibilities are infinite so let me give you _my_ explanation of the IoT sub-domains:

- Electronics: the hardware part; Analog and Digital Electronics. To me, an area I need to develop, I would love to be able to design circuit boards from scratch. 
- Sensors: Part of the electronic bubble in the case of IoT, the sensors. Many are really easy to implement as they are using standards to communicate. The quality of the measurement is probably the most important part of that area. Then is the interpretation.
- Stream-Processing: Now that we have raw measurement data, Digital Signal Processing algorithms and FPGAs (_ie_ configurable electronic circuits) will bring the insights to actually make use of the data. The hardware is an important part with some processor handling Floating-Point computation natively while others are not; From a developer standpoint, the programming language should also be considered: functional reactive programming is made for data streams. [I am currently porting algorithms from C to Rust](https://interrupt.memfault.com/blog/rust-for-digital-signal-processing).
- OS/SDK: the basis of embedded software. Real-Time OSes (RTOS) are mostly used for low-power devices if not bare metal (without OS). Software Development Kit (libraries and examples) too are providing a skeleton, and some are excellent. From my experience, I would mention the nRF5-SDK from Nordic or the ESP-IDF from Espressif. Both bring an RTOS implementation (FreeRTOS and Zephyr). The build set up is also provided even if one could change to its preferences.
- Communication: Now that we have data to transmit from the sensors and the processing algorithms, we need to transmit it. Communication can be separated into two sub-domains: short and long-range communication. You all know the difference based on your smartphone's ability to communicate using 4G (Long-Range) and WiFi or Bluetooth (Short-Range).
- Backends: Long-term storage of the data is made on backends, usually servers, and different protocols are available to open the backends' doors. Some are using a REST API, CoAP or MQTT for example. We will also increasingly see new types of backends based on Blockchains.
- Analytics: Another field of application based on IoT is to use big chunks of data to find insights that are worth considering. I don't know that area but we all have heard of Artificial Intelligence powered by tons of data to make the system learn, which can then be used in real-time (stream-processing).
- Device Management: Once devices are sent on the spot, it's important to keep track of their health. Tools are being built to make the management easier. In my own experience, I built my own tools [to deliver Firmware](https://medium.com/equisense/firmware-quality-assurance-continuous-delivery-125884194ea5) and [to monitor bugs in production](https://medium.com/equisense/quality-assurance-for-firmware-production-monitoring-68cd5fcf038d) but there are now tools like [MemFault](https://memfault.com/) that provide everything for you: bug tracking, firmware updates by cohort, etc...
- Security: We have almost everything we need to build connected devices but now that our sensors are measuring critical data or transactions are being made, we need to build walls to protect the users and the companies behind the products. My experience brought me to build tools to encrypt firmware or use the communication security features to make sure the data cannot be obtained by anyone. The increasing criticality of the data suffice to say that I need to learn more in that specific area, and developers all need to be kept up-to-date.


### What makes me?

After more than 5 years of building connected devices, I learned a lot. Freelancing is now allowing me to learn even more, in areas I didn't get the chance to work on.

I colored the nodes differently based on what they are to me:

- 💚 Ask me anything about it, I can respond to you. 
- 💙 I have backgrounds without being a complete expert. Ask me if you have projects involving those, I would be pleased to help! 🤓
- 💛 I want to know more! I dedicate one day a week at making those yellow circles green. Please tell me you have a project for me!
- ☑️ I won't dive into those in the near future. There are many others that I didn't write here. Internet of Things can be a huge playground, I don't want to get lost.

In my quest to defining me as a unique person, making that graph gave me a better understanding of what I am capable of, and where I am willing to make progress. But that's not it.

I am convinced that Distributed Ledger Technologies will soon automate transactions: being money or data exchanges. I read a lot about what's going on in that space regarding finance (like DeFi applications) and I am passionate about it. I would love to be part of that evolution and based on my knowledge around IoT, I am willing to work on demos involving both connected devices and blockchain.

> I have a lot more to learn, and I am totally excited about it.

## The Internet of Things and you?

If you are reading those lines, it means you are probably familiar with the IoT. 

You can now play with the graph 🤹‍♂️:
- Make it your own
- Improve it: I would love to build a better graph thanks to you.
- If you want to start from scrath, head to [the page where I found the source code](https://bl.ocks.org/cjrd/6863459).

You can add sub-domains that are missing or re-organize the nodes so that I can grasp your understanding of the IoT. Then, download the graph (JSON file) and send it to me through [LinkedIn](https://www.linkedin.com/in/cyrilfougeray/) or [Twitter](https://twitter.com/cyrilfougeray). I will post your contributions to the article directly. 

--- 

```
- drag/scroll to translate/zoom the graph
- shift-click on graph to create a node
- shift-click on a node and then drag to another node to connect them with a directed edge
- shift-click on a node to change its title
- click on node or edge and press backspace/delete to delete
```

<iframe sandbox="allow-popups allow-scripts allow-forms allow-same-origin allow-modals" src="/iot_knowledge_graph.html" marginwidth="0" marginheight="0" style="height:600px; width: 100%" scrolling="no">
</iframe>


---

If you enjoyed the article, share it. 🙂

Stay safe 👋