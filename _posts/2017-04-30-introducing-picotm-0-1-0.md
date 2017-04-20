---
layout: post
title:  "Introducing picotm 0.1.0"
author: Thomas
date:   2017-04-30 10:56:43 +0200
categories: picotm release
---
Welcome to picotm, the system-level transaction manager. With
[picotm 0.1.0][picotm_0_1_0] just released, it's a good time to write a bit
about the software and the idea behind.

Picotm is a system-level transaction manager. You probably heard of
transactions in the context of data bases, where they ensure that multiple
concurent users don't interfere with each other.

Often abbreviated as the [ACID][acid] properties, a transaction provides two
important concepts to its users, namely

    * isolation from the effects of concurrent transactions, and
    * error detection and recovery.

Picotm's transactions serve a similar purpose, but at a much lower level.
Picotm provides transactional operations at the level of the operation
system, or slightly above.

Traditionally, in system development, error handling is done by hand. You call a
function, you check a returned status code for an error, you handle any error
you detected. It's very repetitive and prone to mistakes. It's too easy to
forget an error check or get the handling wrong.

With picotm, all this is done for you. Picotm implements the logic for
error detection and recovery once in a single place. All your transactions
benefit.

Another traditional approach in system development is concurrency control by
locking. For each shared resource, you'd take a lock, access the resource, and
release the lock. This is prone to deadlocks and doesn't scale well if the
number of concurrent threads increases.

With picotm, you can access shared resources in a more natural way: by using
the interfaces of the resource. For memory you'd call `load` and `store`, for
files you'd invoke `read()` and `write()`. Picotm handles all concurrency
control internally, resolves deadlocks, and can select an optimal strategy
for concurrency control.

For a full example and comparsion of traditional and transactional code,
take a look at the [About][about] page on the website.

And finally, at a glance, here are the features of picotm 0.1.0:

 - transactional semantics for arbitrary blocks of C code
 - built-in support for
   - memory operations (Transactional Memory),
   - memory allocation,
   - file-descriptor I/O,
   - C string and memory functions,
   - C math functions, and
   - much more

Picotm is implemented in C and currently available on Linux.

[about]:        /about/
[acid]:         https://en.wikipedia.org/wiki/ACID
[picotm_0_1_0]: https://github.com/picotm/picotm/releases/tag/v0.1.0
