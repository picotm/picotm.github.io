---
layout: post
title:  "Fwd: Scaling Lock Performance"
author: Thomas Zimmermann
date:   2017-10-23 9:30:00 +0200
tags:   [benchmark, fwd, locking, picotm, stm]
---

The next release of picotm will feature a number of improvements to the
locking code, and specifically to the transactional-memory module. Locks
will be more scalable and transactions have now bounded execution times,
guaranteeing progress of the system as a whole.

In my blog, I posted an [article][fwd] about the upcoming changes and how
they affect the results of picotm's performance tests.

Best regards<br>
Thomas

[fwd]:  http://transactionblog.org/2017/10/20/scaling-lock-performance/
