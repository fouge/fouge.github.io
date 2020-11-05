---
layout: post
title: Continous Delivery for Firmware updates.
description: Building and releasing Firmware to the customer is easy when you use the right tools.
date: 2020-10-03
author: Cyril
meta: 
  - tag:
    title: CI/CD
    class: delivered
---

This post has been previously [posted on Medium on January 2019](https://medium.com/equisense/firmware-quality-assurance-continuous-delivery-125884194ea5). It has since been updated and incremented.

---

In this blog post, I wanted to highlight easy ways to implement a few Quality Assurance (QA) processes when you are the only person working on Firmware in your team. If you feel that you don’t need it or that it looks like an enormous amount of work to make those processes truly work, think again, we are in <del>2019</del>2020.

<figure class="col-md-6">
  <img src="/img/posts/continuous_delivery/bitbucket_cicd.png" alt="Continuous Delivery" class="img-responsive">
</figure>

Back in 2018 when I was working at Equisense, I first wanted to speed up the development process by automating as many tasks as possible, like building new Firmware packages and deploying them to our beta users and final customers in a few clicks. Then, I knew that the many steps involved in releasing a new firmware package were prone to human error which was not acceptable.

As I have been working on the nRF52 series microcontroller, you will encounter several tools specific to Nordic Semiconductors products, but those are similar in many other SDKs.

## Manual steps are not acceptable

Let’s give an overview of some of the tasks I used to run on my machine that needed to be automated to have a better Continuous Delivery (CD) process 🤯.

First, *compilation*. The cross-compiler, with associated paths to be defined in a common Makefile or as environment variables, has to be installed and configured on each machine. If another person had to work on the project, that one may have installed a different version, using different settings, flags, etc., which in the end resulted in different code quality. Less critically, the `clang-format` code formater being used with a different version outputs a different code, even with the same config file!

Then, the Device Firmware Upgrade (DFU) packages have to be consistent. Once we have a binary, we use a tool called `nrfutil` to package that image for a DFU procedure. For security reasons, we customized `nrfutil` in order to encrypt the firmware, meaning that the custom version of `nrfutil` needs to be installed from source and added in the `PATH`. Then a Makefile command was available to generate a new package (the least we can do...).

Finally, deployment. Once a DFU package was generated as a zip file with `nrfutil`, one had to deploy it to our back-end for testing purposes first, then for full production release (we have `staging`, `beta`, and `production` steps in our deployment pipeline). That task wasn’t automated and I did those steps using Postman: get the path to the - hopefully - right zip file, write down Firmware version, then get a token to post to our server, carefully make sure that I didn’t mess up, and click “POST”, sometimes pray 🙏.

<figure class="col-md-12">
  <img src="/img/posts/continuous_delivery/post_firmware_form.png" alt="Continuous Delivery" class="img-responsive">
  <figcaption>Fill the blanks to post a new Firmware…</figcaption>
</figure>

I needed a solution.

## Dockerizing builds

The flexibility that I wanted has been answered by Docker for a few years now. Indeed, containers are really efficient to run specific tasks independently without much overhead compared to your local machine. Copy-pasted from the Docker website:

> Docker unlocks the potential of your organization by giving developers and IT the freedom to build, manage and secure business-critical applications without the fear of technology or infrastructure lock-in.

That was it.

So, I created my Docker image called [equisense/nrf5-builder](https://hub.docker.com/r/equisense/nrf5-builder/) that you can pull from Docker Hub or use the Dockerfile and example on [my Github](https://github.com/fouge/continous-integration-nrf). It includes the ARM GCC cross compiler, Python dependencies and nRF tools (`nrfutil` as a submodule). I created a Makefile to show how the docker can be used along with the Makefile contained in the root directory.

That one is pretty generic so I customized it for our own usage in order to build another Docker image based on `equisense/nrf5-builder`, you should build your own upon it if you work with nRF52 targets.

## Automating builds and deployment

Here we are. Compiling and generating the firmware update package from a docker container gives the possibility to run those steps on any machine, such as a remote one. By a remote machine, I think a CI server, obviously.

<figure class="col-md-12">
  <img src="/img/posts/continuous_delivery/gitlab_pipeline.png" alt="Gitlab Pipeline" class="img-responsive">
  <figcaption>Gitlab CI/CD with steps available on click</figcaption>
</figure>

Git major hosting platforms now all include CI/CD tools directly online, here I am using Gitlab CI/CD. Depending on your Git hosting platform, you should check out Bitbucket Pipelines, CircleCI, Jenkins, or the not-so-new Github Actions.

All you need to do is to describe the steps to be done on a new commit, a specific branch or tag, or even manually in a yaml file. I didn’t want to have the code built on each commit but only when releasing a new version so I added a Makefile command to increment the version and to create a tagged commit. Once that commit is pushed, the pipeline:

- Builds the docker image from the Dockerfile. This Dockerfile is based on the `equisense/nrf5-builder` image with specific configurations and tools to be used during the CI/CD process (the custom `nrfutil` for example).
- Compiles the code and creates the DFU packages from the Docker image. Using the command: `docker run nrf5-builder-custom make package`. This command invokes a Makefile target that will compile and generate ZIP packages ready for the DFU procedure.
- Deploys automatically the DFU package to the `staging` server. This step invokes a script to get a token from the authentication service using a username and password (those variables are kept masked, check out Gitlab CI/CD > Variables to save that kind of variable). Using that token, it then pushes the package to the Equisense server using a `curl` command.

Then I can push the artifacts to all our users by simply calling the steps  `deploy_preprod` then `deploy_prod` 🤩. I keep the generated artifacts such as the `ELF` and `.map` files directly in Gitlab by setting the value of `expire_in` to `never`. On Bitbucket, I used to [push the artifacts to the Downloads module](https://support.atlassian.com/bitbucket-cloud/docs/deploy-build-artifacts-to-bitbucket-downloads/).

Remember the form above? Gone 🌟.

## Going further

Back in 2018, I also wanted to run tests on a local machine that has full access to our hardware in order to launch integration tests. I tried [TeamCity](https://www.jetbrains.com/teamcity/) which I think can handle that job pretty well but it hasn't been my top priority and it implies some hard work if you want to test Bluetooth products; without saying that libs for Bluetooth communication from a computer are buggy and most of the time can’t be ported to different OSes.

---

Please let me know if you know some better ways or have some other tips for improving the Continuous Delivery process.

👋
