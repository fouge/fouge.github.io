---
layout: post
title: Building the Orb, Building Myself
description: 5 years at Tools for Humanity and how it shaped my career
date: 2026-09-18 # format 2020-04-02
author: Cyril
meta: 
  - tag:
    title: engineering
    class: learn
---

<style>
  .orb-intro-image { float: left; width: 260px; max-width: 40%; margin: 0 32px 24px 0; }
  .orb-intro-image img { display: block; width: 100%; height: auto; }
  @media (max-width: 600px) {
    .orb-intro-image { float: none; width: 240px; max-width: 100%; margin: 0 auto 24px; }
  }
</style>

<figure class="orb-intro-image">
  <img src="{{ '/img/posts/building-the-orb/orb.png' | relative_url }}" alt="The Orb, with its silver housing and gold camera opening" width="1920" height="1776">
</figure>

In May 2021, I joined Tools for Humanity as an embedded software engineer, working on [the Orb](https://hub.allspice.io/WorldOrb). Five years later, I wanted to take a step back: what had I learned, how had I changed, and what did I want to carry into my next chapter? I am writing this partly for myself, and partly for anyone facing similar questions about their own work.

I started with a clear responsibility: make the microcontroller firmware production-ready. Over time, that work took me into Rust services, user experience, and the Linux stack. Looking back, the most valuable growth came from owning a part of the product deeply, then learning to contribute beyond it.

<div style="clear: both;"></div>

## Starting from a real production problem

<style>
  .orb-demo-video { float: right; width: 260px; max-width: 40%; margin: 0 0 24px 32px; }
  .orb-demo-video video { display: block; width: 100%; height: auto; }
  .orb-demo-video figcaption { text-align: center; font-size: 14px; line-height: 1.5; margin-top: 10px; }
  @media (max-width: 600px) {
    .orb-demo-video { float: none; width: 260px; max-width: 100%; margin: 0 auto 24px; }
  }
</style>

<figure class="orb-demo-video">
  <video controls playsinline preload="none" poster="{{ '/img/posts/building-the-orb/orb-demo.jpg' | relative_url }}" aria-label="Video from my work on the Orb">
    <source src="{{ '/img/posts/building-the-orb/orb-demo.mp4' | relative_url }}" type="video/mp4">
    <a href="{{ '/img/posts/building-the-orb/orb-demo.mp4' | relative_url }}">Watch the Orb video.</a>
  </video>
  <figcaption>Early build of an Orb front 📸</figcaption>
</figure>

When I joined, there had already been Orb prototypes, but the production device was taking shape. My responsibility was clear: make the microcontroller firmware production-ready. To keep it short, the Orb includes two STM32 microcontrollers. One controls major physical parts of the device: power for the NVIDIA Jetson, the gimbal, a liquid lens, visible and infrared LEDs, [and more](https://github.com/worldcoin/orb-firmware). The other lives at the security boundary, connecting the Jetson to the secure element and handling tamper detection.

My previous work on production devices gave me a foundation for structuring the firmware from scratch. Before joining TFH, I had already written about [automating firmware releases]({% post_url 2020-10-03-firmware-qa-ci-cd %}), [making crashes easier to diagnose]({% post_url 2020-07-27-firmware-logs-with-stack-trace %}), and [automating versioning]({% post_url 2021-01-25-recipe-automated-versioning %}). Those concerns were already part of how I approached production firmware: how to release it, identify what is running, and understand what went wrong.

Together with Pete, we chose Zephyr RTOS, a choice I would make again five years later. We worked on architecture, drivers, tooling, debugging, and the interfaces that allowed manufacturing tools to control the microcontrollers and retrieve telemetry. Some of that driver work also made its way back into the Zephyr project.

<div style="clear: both;"></div>

## Autonomy is where I grew the most

I grew the most when we were building the production system from scratch. Dan, our manager, protected the team, listened to our needs, and gave us enough context to set priorities ourselves. We maintained our own backlog and decided how to reach the goals. Responsibility is my fuel: when people trust me to own something, I keep challenging myself to deliver the best work I can. Having the authority to make decisions gave me room to put that drive into practice.

That freedom worked because we challenged each other. Pete and I reviewed each other's work with mutual respect, and we could disagree and change our minds. Autonomy and close peer review made each other more effective.

When Pete left, I took full ownership of the microcontroller firmware stack. That included investing in testing and diagnostics: we had hardware-in-the-loop testing before it was in place for the whole Orb, and telemetry that gave us detailed context about the microcontrollers. Owning the firmware meant making it possible to test, diagnose, and maintain it through ten hardware revisions across two Orb versions, while keeping those hardware changes largely transparent to the software running on the Jetson. I am grateful to TFH and the team for trusting me with that responsibility.

## Learning beyond the microcontrollers

My responsibilities gradually extended into the [software running on the Jetson](https://github.com/worldcoin/orb-software), much of it written in async Rust. I owned three components: `orb-mcu-util` for diagnosing and controlling the microcontrollers, `orb-ui` for the Orb's user interface, and the private `orb-mcu-telemetry` service connecting MCU telemetry to Datadog and Memfault.

For me, async Rust and Tokio were a steep learning curve. [Ryan B.](https://github.com/thebutlah) taught me a lot, and I am grateful for his help and patience. One lesson that stayed with me was to think through the full lifecycle of concurrent work: how it starts, how to stop it cleanly, and how to propagate errors so the caller knows what went wrong.

I also separated `orb-ui` from Orb Core, originally written by the unique [Valentine V.](https://github.com/valff), so it could run as its own service, and worked directly with industrial designer Thomas M. on the UX. That collaboration taught me to keep the user experience simple, even when the software and hardware behind it were complex.

A tool I initially wrote for testing also ended up on production devices, where it was used extensively, including for OTA updates. It was a reminder that internal tooling can become part of the product, making its reliability and maintenance just as important.

These projects gave me a practical way to learn beyond firmware: take responsibility for something the team needs, ask for help, and build enough understanding to maintain it.

## The most intense period

The release of Diamond, the second Orb version, was the most intense period of those five years. With the MCU firmware already supporting the new boards, I stepped in to help with Orb OS, the Linux side of the device.

Moving from Jetson Xavier to Jetson Orin meant migrating to newer versions of JetPack, Ubuntu, the Linux kernel, compilers and libraries. The camera setup changed too: one combined RGB/IR camera replaced two separate cameras. We had only four months to make the new device work, including rebuilding the build system and bringing up the new cameras.

The scope was daunting, but some of my existing experience transferred directly. Orb OS spanned several repositories, so I introduced West, a tool I knew from Zephyr, to manage their versions and make the build inputs easier to understand. The team welcomed that change.

The camera work took me further outside my usual scope. Together with [Filip K.](https://github.com/Qbicz), we worked with our external partners to meet our requirements, but we could not control the camera the way we had planned. We had to understand its internal behavior in detail and adapt the MCU firmware to synchronize the infrared LEDs with it and other cameras. That problem crossed the camera, its Linux driver, and the firmware I already knew well.

In the end, the team delivered. Kudos to [the team](https://github.com/worldcoin/orb-software)!

I came away more confident that I could contribute before mastering every layer of the stack. My firmware experience gave me a useful starting point; following the camera problem across those layers taught me the rest as we worked through it.

## A system's communication is not only technical

As the Orb and the organization grew, the challenges changed.

The original Orb Software team was split into two teams: one focused on improving the signup flow—the Orb's main application—and one on the device platform. The intention was reasonable: bring in more people, create focus, and achieve more.

But the Orb is a tightly connected system. From my perspective, the split fragmented some of the context needed to evolve it. More coordination across teams and hierarchy made delivery feel slower, especially when a change touched several parts of the device.

While the organization has always been quite transparent, we lost some visibility into the business context. My impression was that it was now seen as more relevant to the signup-flow team. At the time, I felt less useful, less autonomous, less motivated, without fully understanding why. Looking back, I think losing some of that context played a part: understanding why something mattered had helped me choose where to spend my time. I now recognise that access to those priorities is something I need to ask for when it is missing.

I would have kept a regular meeting for everyone working on the Orb to discuss business goals, shared priorities, and integration problems. Teams need enough context to make their own decisions, even when their responsibilities differ. I would also want someone responsible for decisions affecting the whole device: someone who had spent time building it and understood its history and trade-offs. A title alone cannot provide that context.

## Speaking up is part of ownership

At the beginning, I stayed quiet about parts of the device outside my direct scope.

I owned the microcontrollers. Other people owned services on the Jetson. I was remote in France while much of the team was in Munich, and I often felt that I did not know enough to participate in discussions outside my immediate area.

Clear ownership helped me focus and deliver. Remote work was also a net positive for me as an individual contributor: I can concentrate deeply alone, and I do not believe physical presence is required for productivity when ownership is clear.

But I would encourage my 2021 self to ask more questions and speak sooner.

Not understanding something does not mean you have nothing to contribute. Asking questions does not make you look stupid. It is how you build the context needed to notice risks, connect pieces, and eventually have useful opinions.

By 2026, I understood much more of the Orb: the services, their interactions, and the ways a local decision could affect the device as a whole. People increasingly relied on me to make things happen. That responsibility gave me the confidence to challenge decisions when needed.

The confidence did not arrive first. It came from being trusted with real work, doing it, and seeing the results.

## What I carry forward

<style>
  .orb-launch-badge { float: right; width: 180px; margin: 0 0 24px 32px; }
  .orb-launch-badge img { display: block; width: 100%; height: auto; }
  @media (max-width: 600px) {
    .orb-launch-badge { float: none; width: 140px; margin: 0 auto 24px; }
  }
</style>

<figure class="orb-launch-badge">
  <img src="{{ '/img/posts/building-the-orb/launch-team-badge.png' | relative_url }}" alt="Worldcoin launch team badge, number 045, dated 24 July 2023" loading="lazy">
</figure>

I leave these five years more confident working across any device's software stack. I learned to carry my firmware experience into unfamiliar problems, ask for help, and contribute while still learning. On my next project, I want to speak up earlier, ask for the context I am missing, and clarify ownership when work crosses teams.

Above all, thank you to all my colleagues at Tools for Humanity for your kindness. These five years have been amazing. I am grateful to have worked alongside you on a technology that I believe will help humanity in the age of AI.

Work gives me a strong sense of purpose, especially when I can see its impact in the real world. After taking a break, I’m looking forward to contributing again.

Cheers 👋

<div style="clear: both;"></div>
