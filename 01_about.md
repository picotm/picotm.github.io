---
layout: page
title: About picotm
permalink: /about/
---

Picotm is a system-level transaction manager that allows you to write C
applications and firmware that is secure, reliable and thread-safe; yet
easy to develop.

Picotm applies the concept of transactions to low-level code and
operating-system interfaces. Error handling and thread isolation are
provided by picotm, all you have to write is the application logic.

Picotm is Open Source under terms of the the
[Mozilla Public License, v.2.0][mpl_2_0]; viable for free and proprietary
software.

## Features

Picotm offers

 - transactional semantics for arbitrary blocks of C code, and comes with
 - built-in support for
   - memory operations (Transactional Memory),
   - memory allocation,
   - file-descriptor I/O,
   - C string and memory functions,
   - C math functions, and
   - much more.

Picotm is implemented

 - in portable C, and
 - extensible to support the features and environments you require.

Currently supported are a variety of Linux systems.

## Example

The power of picotm is best explained by an example.

The following C program consists of two threads. Thread #1 writes a value
to a globally shared memory location. Thread #2 reads a shared memory and
saves it to a file and its backup.

The program uses two different types of resources: main memory and file
descriptors.

```
    int global_value; /* concurrently accessed by multiple threads */

    pthread_mutex_t lock; /* protects |global_value| */

    /* executed on thread #1 */
    void
    write_value(int value)
    {
        pthread_mutex_lock(&lock);

        global_value = value;

        pthread_mutex_unlock(&lock);
    }

    /* executed on thread #2 */
    void
    save_value(int fildes1, int fildes 2, off_t offset)
    {
        pthread_mutex_lock(&lock);

        /* Save to file */
        pwrite(fildes1, &global_value, sizeof(global_value), offset);

        /* Save to backup file */
        pwrite(fildes2, &global_value, sizeof(global_value), offset);

        pthread_mutex_unlock(&lock);
    }
```

The code above has a number of problems.

 1. Program errors are not handled. If anything goes wrong, the example
    program wouldn't know about it. In any software, error handling
    significantly complicates program logic and makes source code less
    readable. Since errors don't happen during regular operation, existing
    error-handling code is often not well tested.

 2. Calls to `pwrite()` are allowed to write less than the specified number of
    bytes. In this case, the caller has to invoke `pwrite()` again to write the
    remaining data until the buffer has been written entirely.

 3. The call to `pwrite()` for `fildes2` can fail with an unrecoverable error,
    such as a full disk drive. The state is now inconsistent, as the original
    file contains the write, the backup file doesn't.

 4. Mutexes are prone to deadlocks. If there's another mutex in addition to
    `lock` and different threads acquire both mutexes in different order,
    the program can reach a state where threads are waiting for each other's
    mutex to become available. This problem becomes more sever as more shared
    resources with corresponding locks are added.

 5. Locks don't scale with the number of processors. If many threads read
    and write `global_value` concurrently, the locking operation will become
    a bottleneck for performance.

Here's the same functionality implemented with picotm. With picotm, all
locking is replaced by transactions. Each transactions starts with
`picotm_begin` and commits with `picotm_commit`. Users can provide their own
error-recovery code between `picotm_commit` and `picotm_end`.

```
    int global_value; /* concurrently accessed by multiple transactions */

    /* executed on thread #1 */
    void
    write_value(int value)
    {
        picotm_begin

            store_int_tx(&global_value, value);

        picotm_commit
        picotm_end
    }

    /* executed on thread #2 */
    save_value(int fildes1, int fildes2, off_t offset)
    {
        picotm_begin

            pwrite_tx(fildes1, &global_value, sizeof(global_value), offset);
            pwrite_tx(fildes2, &global_value, sizeof(global_value), offset);

        picotm_commit

            if (picotm_error()) {
                /* Unrecoverable error */
                inform_admin_and_shutdown_gracefully();
            }

        picotm_end
    }
```

That's it. Using picotm immediately fixes all of the problems present in the
original source code.

 1. All errors are detected by picotm. For recoverable errors, picotm
    will perform the recovery and restart the transaction. For unrecoverable
    errors the user can provide their own handling, or at least shutdown
    the program gracefully. All user error handling is in a single place
    after the commit operation.

 2. Complete writes are performed by the implementation of `pwrite_tx().` Once
    executed, all data is written to the file; even in the precense of
    incomplete system calls and signal handling.

 3. Transactions are executed entirely, or not at all. If the second call to
    `pwrite_tx()` fails, the effects of the first call are reverted. Both
    files remain consistent. Picotm can take additional steps to avoid
    problems entirely, such as preallocating disk space before writing any
    data.

 4. Picotm automatically handles concurrency control for all its resources
    and operations. Store operations to `global_value` on thread #1 are
    executed transactionally, as well as calls to `pwrite_tx()` on thread #2.
    The value in `global_value` is either written by thread #1 or read by
    thread #2, but never at the same time. Picotm also guarantees that the
    code is free from deadlocks.

 5. Finally, picotm can choose optimized strategies for concurrency control;
    depending on the resource, application and workload. It can adapt the
    granularity of its locking; or exploit semantics of higher-level operations
    to improve scalability.

This simple example only uses two different types of resources: main memory
and file descriptors. Built-in support for both is provided as part of picotm
(iff you want so). But picotm is a framework: you can add your own modules
into the mix and have them integrated seamlessly with other transactional
resources.

## Downloading picotm

All information about getting picotm is available on the
[Downloads](/downloads/) page. You can either download a stable release, or
get a snapshot of the latest code.

## Licensing

Picotm is Open Source Software, provided under the terms of the
[Mozilla Public License, v.2.0][mpl_2_0]. The MPL 2.0 allows for inclusion
into proprietary software, while ensuring the freedom and availability of
the code base.

[mpl_2_0]:  https://mozilla.org/MPL/2.0/
