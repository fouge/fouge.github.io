---
layout: post
title: You should use static assertions.
description: Static assertions are a powerful way to break things fast.
date: 2020-11-09
author: Cyril
meta: 
  - tag:
    title: compilation
    class: optimized
---

I feel like we - software developers writing C - don't know enough about **static assertions**. That kind of assertion is performed during compilation on static values and thus can optimize our code while preventing bad bugs from happening.

---

I wanted to write this short post because I once stumbled upon a bug that could have been prevented with static assertions. Considering the time to discover the bug, understand it, and fix it, I would have preferred to know about static assertions and to extensively use these before.

It all started with the simple action of adding a new value to an enum:

{% highlight c linenos %}
typedef enum {
    TEST_NOT_STARTED = -1,
    TEST_SUCCESS = 0,
    TEST_TIMEOUT = 1,
    // [...]
    TEST_UNKNOWN = 0xFF /// I wanted to add a new status
} test_status_t;
{% endhighlight %}

That enum value was packed into a structure that was sent over the Bluetooth 4.0 link to be decoded by the remote application. The iOS and Android applications developers were told that byte number 3 in the packet was returning the test status, but it was not.

Indeed, try to encode `-1` and `0xFF` on one byte... 🤔 That's not possible, you need 2 bytes and the compiler figured it out before I did. So, now that I added the new status `UNKNOWN`, the enum took two bytes instead of one previously and didn't respect the specifications anymore.

I thought that I needed to catch that kind of error before bothering the other developers with my issues. Ideally, the error would be caught at compile-time. I needed static assertions.

<figure class="col-md-1">
  <img src="/img/posts/static_assertions/c-programming-language.png" alt="Continuous Delivery" class="img-responsive">
</figure>

Static assertions have been introduced in the C1X specification but can also be implemented easily if you are still using the C99 standard, which is still the default in many SDKs...

Using GCC, try to check which flag is being used:

{% highlight make linenos %}
# C99 features are followed using
-std=c99
# if you have GNU extensions enabled as well, it can also be:
-std=gnu99

# or if you are using C11
-std=c11
-std=gnu11
{% endhighlight %}

Check-out [the GCC man page](https://man7.org/linux/man-pages/man1/gcc.1.html) to get all the possible compatible standards with the GCC compiler.

### C99 and before

I have been using the following macro for a long time without knowing where it came from. Turns out [pixelbeat introduced the trick](https://www.pixelbeat.org/programming/gcc/static_assert.html) back in 2008:

{% highlight c linenos %}
// The magic happens below
#define ASSERT_CONCAT_(a, b) a##b
#define ASSERT_CONCAT(a, b) ASSERT_CONCAT_(a, b)
#define ct_assert(e) enum { ASSERT_CONCAT(assert_line_, __LINE__) = 1/(!!(e)) }

// Here you would declare your enum
typedef enum {
    TEST_NOT_STARTED = -1,
    TEST_SUCCESS = 0,
    TEST_TIMEOUT = 1,
    // [...]
    TEST_UNKNOWN = 0xFF /// I wanted to add a new status
} test_status_t;

// Here is how to catch the bug as soon as you introduce it
ct_assert(sizeof(test_status_t)==1);

// Another usage would be to make sure that a Bluetooth 4.0 packet
// doesn't take more than 20 bytes:
ct_assert(sizeof(ble_pkt_t)<=20);
{% endhighlight %}

### C11 and more

Here is the modern implementation, with an error message:

{% highlight c linenos %}
#include <assert.h>

// This won't compile and display a nice error message
static_assert(sizeof(test_status_t)==1, "test_status_t must be one-byte long");

// For our BLE 4.0 packet
static_assert(sizeof(ble_pkt_t)<=20, "Bluetooth 4.0 packets must take less than 20 bytes");
{% endhighlight %}

Here is the output when compiling:

{% highlight plain linenos %}
../../tests.h:4:1: error: static assertion failed: "test_status_t must be one-byte long"
 static_assert(sizeof(test_status_t)==1, "test_status_t must be one-byte long");
 ^~~~~~~~~~~~~
{% endhighlight %}

## Other usages

Static assertions can also be used for any constant value.

Let's say you are using a version to tag a data format implementation:

{% highlight c linenos %}
// file data.h
const uint32_t DATA_FORMAT_VERSION = 3;
{% endhighlight %}

And you provide an implementation to use that format, but only for a specific version:

{% highlight c linenos %}
// file data_v3.c
#include <data.h>

static_assert(HEADER_VERSION==3, "Header version not supported");
{% endhighlight %}

You can probably think about other usages yourself.

You might want to review some parts of the code you are maintaining to include static assertions now before doing something unfortunate in the future 😉.

## Checking the configuration

This is not a static assertion as described in C11 through `static_assert`, but there are other kinds of static values that we can check: macros and flags.

You probably know and use that one. The final goal is the same: do not allow code to compile if it was not intended to work with the defined configuration.

{% highlight c linenos %}
#define MY_CUSTOM_CONFIG
#define MY_CUSTOM_VALUE    1

// If not defined, the compiler will yield an error
#ifndef MY_CUSTOM_CONFIG
#error This implementation is made to work with MY_CUSTOM_CONFIG
#endif

// if value cannot be used due to the condition not passing, the compiler
// yields an error
#if MY_CUSTOM_VALUE < 2
#error MY_CUSTOM_VALUE must be at least 2.
#endif
{% endhighlight %}

## Rust

As you probably know, if you've been reading my articles, Rust is becoming a great contender to C for embedded software.

Regarding static assertions, Rust has [its crate](https://docs.rs/static_assertions/1.1.0/static_assertions/index.html) ready to be used. The crate is made to check types, sizes, configurations and some more use-cases specific to Rust (check that a type does not implement a trait for example)! 👌

As always, the documentation is well written with some great examples, so make sure to read about all the defined macros and use them extensively! 🦀

---

Breaking compilation as soon as possible is a great way to squash the bugs efficiently, so it should be a habit to use it whenever needed.

👋