---
layout: post
title: My recipe for versioning automation.
description: Use Makefile and scripts to generate new firmware version efficiently.
date: 2021-01-25 # format 2020-04-02
author: Cyril
meta: 
  - tag:
    title: compilation
    class: optimized # choose between learn connected smart optimized delivered
  - tag:
    title: keep it simple
    class: delivered
---

Following the article on the Interrupt blog about [proper versioning](https://interrupt.memfault.com/blog/release-versioning), I thought I could propose the recipe I use for versioning which is based on nRF5-SDK based projects but can be easily adapted to others.

---

## Ingredients

Here is the full list of what we need:

- `version.ini`: in the article, Tyler is talking about keeping the `macro`, `minor`, and `patch` numbers into either several macros or in a header file. I don't want to force anyone to search for the `version.h` file in a specific path so I like to keep the version details in the root of the directory, easily accessible to read from anyone. You will see that the version will end up in a header file anyway. Moreover, that header file has some other auto-generated parts so I don't want the developers to modify it manually.
- A root `Makefile`: I am using a Makefile located at the root of the project: we can call it the `master Makefile` 🔥. I am not using `invoke` like [some of my peers](https://interrupt.memfault.com/blog/building-a-cli-for-firmware-projects) because I didn't find any benefit compared to a Makefile when I tried it, but depending on the complexity of your projects you might want to take a look. Also, don't forget to check [tab completion](https://github.com/pyinvoke/invoke/tree/master/invoke/completion) for `inv[voke]`.
- A project `Makefile`: if you have an application running on top of a bootloader, each one of those will have a proper Makefile: this is what I call the `project Makefile`. It is probably located in your project directory depending on how many projects or targets you have in the repository. I personally try to use the same directory pattern as in the Nordic SDK, meaning I've got a few directories in the root, like `components` and `external`. Besides those, there are `my/application` and `my/bootloader`.
- `gen_version.py`: a python script to generate the application `version.h` from `version.ini` you can download it [here](https://github.com/fouge/nrf_utils/blob/master/code/version/gen_version.py).
- `gen_version_bl.py`: a python script to generate the bootloader `version.h` from `version.ini` you can download it [here](https://github.com/fouge/nrf_utils/blob/master/code/version/gen_version_bl.py). We are going to focus on the application version here.

Let's start to cook now. 🍳

## Step 1: Fill `version.ini`

The `version.ini` file is controlling the versioning of the project and every other ingredient relies on what's inside that file.

To start, let's fill `version.ini` with some specific fields:

{% highlight ini linenos %}
[version]
major=0
minor=1
patch=14
bl=1
hw=1
{% endhighlight %}

On top of the standard `major.minor.patch` numbers, I added the `bl` number specifying the bootloader version and `hw` for the hardware revision. Each one of these numbers will be `uint8_t` encoded in the final binary.

## Step 2: Add targets to the master Makefile

The master Makefile will have a few roles. First, `version.ini` could be modified manually, but we don't want to mess with it so I decided to create a target in the master Makefile to increment the patch or minor using a command.

This part will be pretty simple because everything is done from the Python script. Let's say you've added my repo [nrf_utils](https://github.com/fouge/nrf_utils) in your root directory:

{% highlight make linenos %}
# The developer can set an environment variable FIRMWARE_PROJECT_ROOT 
# or use the current directory as the root location.
ifeq ($(FIRMWARE_PROJECT_ROOT),)
PROJECT_PATH=$(shell readlink -f ${CURDIR})
else
PROJECT_PATH=${FIRMWARE_PROJECT_ROOT}
endif

# -m flag to increment minor
increment_minor:
	python ${PROJECT_PATH}/nrf_utils/code/version/gen_version.py -m -f version.ini -i ${PROJECT_PATH}/my/application/include/version.h

# -p flag to increment patch
increment_patch:
	python ${PROJECT_PATH}/nrf_utils/code/version/gen_version.py -p -f version.ini -i ${PROJECT_PATH}/my/application/include/version.h
{% endhighlight %}

The script is doing nothing else but to modify `version.ini` and rewrite `version.h`. Incrementing the minor version also resets the patch number to `0`. If you take a look at the python script or `version.h`, you will see that I keep the git branch and commit SHA in Firmware. I don't use the GNU build ID to get a unique identifier for each build as I don't feel the need (quite frankly, `M.m.p` has always been enough to identify a build as I never release a package without a new version number), but it's interesting enough for you to spend some time reading [that article](https://interrupt.memfault.com/blog/gnu-build-id-for-firmware).

Now let's take a step further and add a new target: `new_version`. The goal of this target is obviously to create a new firmware version. Executing it will tag the current commit to obtain a new release version. This will later trigger [CI jobs](http://www.cyrilfougeray.com/2020/10/03/firmware-qa-ci-cd.html) and generate a firmware update package. Bonus: I also print the remaining TODOs in the code as one of my goals before a release is to have all the TODOs removed. Any TODO that stays too long should be added as an issue in your project management tool.

{% highlight make linenos %}
major_version = $(shell sed -n -e 's/^\s*major\s*=\s*//p' version.ini)
minor_version = $(shell sed -n -e 's/^\s*minor\s*=\s*//p' version.ini)
patch_version = $(shell sed -n -e 's/^\s*patch\s*=\s*//p' version.ini)
bl_version = $(shell sed -n -e 's/^\s*bl\s*=\s*//p' version.ini)
hw_version = $(shell sed -n -e 's/^\s*hw\s*=\s*//p' version.ini)

new_version: increment_patch
	git commit -am "Release package $(major_version).$(minor_version).$(patch_version)-$(bl_version)" --allow-empty
	git tag -a v/$(major_version).$(minor_version).$(patch_version) -m "Release package $(major_version).$(minor_version).$(patch_version)-$(bl_version)"
	@echo "\nRemaining TODOs:"
	@grep -rnw -e TODO my/
{% endhighlight %}

There is a little warning here. As you can see, I am committing all changes (`-a` flag), so make sure to have stashed any unwanted changes before executing the target.

We now have targets to increment the version number in `version.ini`.

## Step 3: Generate the version header, the lazy way

We can notice that `version.h` depends on `version.ini` and the last commit in the repository: a new commit will have a different SHA, which will change `version.h`. Make is made to execute targets when dependencies change and here, we want the Makefile to generate a new `version.h` if `version.ini` or the commit has changed, and only if one of those has changed. This is a major benefit in the compilation time to rely on changes only. If we had generated `version.h` each time we wanted to compile, any file including `version.h` would have been recompiled even if `version.h` didn't change. Here is a snippet of the project Makefile:

{% highlight make linenos %}
# Get the file in the branch referencing the commit SHA
current := $(shell cut -c6- $(SDK_ROOT)/.git/HEAD)

# version.h regenerated only if new commit or version.ini has changed.
../config/version.h: $(SDK_ROOT)/.git/${current} $(SDK_ROOT)/version.ini
	@echo Preparing version
	python $(SDK_ROOT)/nrf_utils/code/version/gen_version.py -f $(SDK_ROOT)/version.ini -i ../config/version.h

prepare_version: ../config/version.h

# app_debug depends on prepare_version to check if version has changed
app_debug: prepare_version
	@echo Compiling app_debug now...
{% endhighlight %}

## Step 4: Add spices

Now that everything is in place, you are able to create new versions from the master Makefile. You might actually want to control everything from the master Makefile, which is easily possible by adding new targets. Something like:

{% highlight make linenos %}
# -C flag can be used with make to move directory before executing a target
app_debug:
	make -C $(PROJECT_PATH)/my/application/nrf52840_s140_armgcc/ app_debug
{% endhighlight %}

Now, from the root of the project you can execute the command: `make app_debug`.

I personally have the habit to open a terminal tile ([Tilix](https://gnunn1.github.io/tilix-web/)) directly where the project Makefile is located so I don't use such target from the master Makefile very often.

## 👌 Enjoy

Besides being automated, the versioning process presented here is efficient as it is activated only if the version actually changed, by taking advantage of Make. I decided not to use `invoke` because I feel like my projects are not complex enough and I guess I've got my habits now, but I know I can rely on such a tool in the future.

See you!
👋
