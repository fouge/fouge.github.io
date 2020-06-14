---
layout: post
title: Panicking, the correct way.
description: Making use of the Rust panic handler on any embedded target. Learn how to define a panic behavior in Rust. 
date: 2020-06-15
author: Cyril
meta: 
- rust
---

A few years ago [I added a mechanism](https://medium.com/equisense/quality-assurance-for-firmware-production-monitoring-68cd5fcf038d) to track HardFaults, failing asserts or any error happening on the products I have been developing. This way, I was able to track down bugs in production which is not an obvious task otherwise, because extensive testing is not obvious. It made me realize the many bugs that are not even noticed by the users and how better I could do to have a stable firmware. 

With the evolutions brought on the same products, I am now about to release in production my first library written in Rust 🙌. But before doing the big jump, I needed to log errors happening when executing the library. Yes, there are errors sometimes... even when using Rust 😉.

## Panic!

As you might already know, Rust forces you to write a panic handler which is great to make use of the trace when errors are occuring. When I started developing a Rust library launched from embedded C code, without the standard library (`no-std`), I chose a [common panic behavior](https://rust-embedded.github.io/book/start/panicking.html) from a crate I had added to my `Cargo.toml`. As too often, the panic handler would reset the target without doing anything with the error.

Now that things are getting serious and to make my library production-ready, I needed to redefine my panic handler to make use of errors occuring on the target. So I created a really simple but useful crate.

Firstly, the panic handler has this signature: 

```rust
fn panic(panic_info: &PanicInfo) -> ! { }
```

The `PanicInfo` structure is really interesting because it contains all the information about the error:

```rust
pub struct PanicInfo<'a> {
    payload: &'a (dyn Any + Send),
    message: Option<&'a fmt::Arguments<'a>>,
    location: &'a Location<'a>,
}

pub struct Location<'a> {
    file: &'a str,
    line: u32,
    col: u32,
}
```

### File & line

If you clicked on [the first link](https://medium.com/equisense/quality-assurance-for-firmware-production-monitoring-68cd5fcf038d) of that article, you probably saw that I already track the same pieces of information, specifically the file and line. Those functions written in C are still available for me to call them from Rust. In my simple crate, all I have to do is to call the function taking care of those arguments. This handler is then storing 3 arguments in RAM to be sent after a reset, wrapped in a warning message. Those arguments are:

- the file name hash. I am using a hash to have a really short variable to be sent over Bluetooth (and it doesn't take Flash as well in the case of my C files because a 32-bit hash is way shorter than a full path in a string;
- the line of the error into the file;
- the error code.

```rust
extern "C" {
    pub fn app_error_fault_handler_release(err_code: cty::c_uint,
                                           line: cty::c_uint,
                                           cksum: cty::c_uint);
}
```

Using the panic handler, the hash is not known but I have the file name as a `&str`. So I have to compute a CRC over the string to get the hash dynamically, which is better than perfect but I don't know anyway (yet) to force the compiler and linker to use a checksum instead of the full path stored using characters.

As we have seen, the line can be used directly👌.

### Error code

Regarding the error code, it is not easy to guess the errors that can be detected before I stumble onto those errors... or is it? 

If there is a specific payload to be sent, it is obviously stored in the program itself as a human-readable string. Using the `strings` tool, I checked the binary ready to be launched in production (optimized out, without any debug symbol, etc...). The snipped below turns out to be very interesting:

```
$ strings _build/app_release.bin

init@0x%04x  👈 Some warning messages I added myself
[%u]%d@0x%lx
[%lu]
ts:%lu
init
called `Option::unwrap()` on a `None` valuesrc/lib.rs  👈 Rust panic payload!
src/libcore/num/dec2flt/rawfp.rsindex out of bounds: the len is  👈 Another one
src/libcore/unicode/printable.rs
 but the index is 
00010203040506070809101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899called `Option::unwrap()` on a `None` value/home/cyril/Documents/rust/panic_custom/src/lib.rs
 #-0+ 
efgEFG
0123456789ABCDEF
0123456789abcdef
```

It confirms my thoughts: the messages are hardcoded inside my program.

It also confirm that if I want to sent a short error code, I'll have to parse the payload to get the meaning of the error and associate each different error with a code. For example, if it contains `"out of bounds"`, I will send the error code `0x01` and I will keep a table with all the relations I created.

## Optimizations

Interestingly enough, looking at the strings led me to think that some error messages shouldn't exist at all. If you looked at it closely, you probably noticed that line: 

```
00010203040506070809101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899called `Option::unwrap()` on a `None` value/home/cyril/Documents/rust/panic_custom/src/lib.rs
```

That message comes from my panic handler. I don't want to think about the panic handler panicking 🤯. Indeed, my panic crate was not correctly handling errors so I replaced the call to `unwrap()` with the following lines, which is the exact way to do:

```rust
if let Some(loc) = panic_info.location() {
	[...]
}
```

Now that I updated my code, I don't have the panic payload for that fault anymore in the binary file. This is a double optimization: an error is prevented from happening and the code is not bloated with strings describing the errors. 🍾

It is actually a great way to check the errors that could be prevented from happening.

## Conclusion

My crate is now ready! I have to add it to any embedded Rust library that will be created for those few products as they all implement `app_error_fault_handler_release`. 

This step was a must before launching that next firmware to the few thousands customers and it gave me the occasion to share a bit more about the stuff I am working on.

---

Please let me know if you have tips to make an even better use of the errors. If you know how to optimize the file name as well, I'll take it.

Stay safe 👋