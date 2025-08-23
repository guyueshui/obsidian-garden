---
{"dg-publish":true,"permalink":"/zettel/202508181019redis-io-threads/","title":202508181019,"tags":["redis"],"created":"2025-08-18T10:19:37+08:00"}
---

```
################################ THREADED I/O #################################

# Redis is mostly single threaded, however there are certain threaded
# operations such as UNLINK, slow I/O accesses and other things that are
# performed on side threads.
#
# Now it is also possible to handle Redis clients socket reads and writes
# in different I/O threads. Since especially writing is so slow, normally
# Redis users use pipelining in order to speed up the Redis performances per
# core, and spawn multiple instances in order to scale more. Using I/O
# threads it is possible to easily speedup several times Redis without resorting
# to pipelining nor sharding of the instance.
#
# By default threading is disabled, we suggest enabling it only in machines
# that have at least 4 or more cores, leaving at least one spare core.
# We also recommend using threaded I/O only if you actually have performance
# problems, with Redis instances being able to use a quite big percentage of
# CPU time, otherwise there is no point in using this feature.
#
# So for instance if you have a four cores boxes, try to use 3 I/O
# threads, if you have a 8 cores, try to use 7 threads. In order to
# enable I/O threads use the following configuration directive:
#
# io-threads 4
#
# Setting io-threads to 1 will just use the main thread as usual.
# When I/O threads are enabled, we not only use threads for writes, that
# is to thread the write(2) syscall and transfer the client buffers to the
# socket, but also use threads for reads and protocol parsing.
#
# NOTE: If you want to test the Redis speedup using redis-benchmark, make
# sure you also run the benchmark itself in threaded mode, using the
# --threads option to match the number of Redis threads, otherwise you'll not
# be able to notice the improvements.
```

io-threads选项开启io多线程，值得注意的是，开启的io多线程中也包含主线程，所以通常4核，设为3，8核设为7，因为自带主线程也是一个io线程。