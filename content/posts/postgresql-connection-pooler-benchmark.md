---
title: "PostgreSQL Connection Pooler Benchmark"
date: 2022-04-04T20:19:12+03:00
draft: false
---

Hi everyone, I've tested 3 different connection poolers from community projects.

1. [Pgbouncer](https://github.com/pgbouncer/pgbouncer)
1. [Odyssey](https://github.com/yandex/odyssey) 
1. [Pgcat](https://github.com/levkk/pgcat)

Before I dive into test setup and test results, here is a quick rundown of their pros and cons.

### Pgbouncer
| Pros                            | Cons                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------ |
| It has a proven track record    | Single threaded                                                                |
| Used and well known in the team | Need to start new pgbouncer instance from different ports if you want to scale |

### Pgcat
| Pros           | Cons                                |
| -------------- | ----------------------------------- |
| Multi threaded | New project                         |
| Load balancing | Beta version                        |
| Query routing  | Lacks authentication method support |

### Odyssey
| Pros                                              | Cons                                        |
| ------------------------------------------------- | ------------------------------------------- |
| Used by Yandex and their cloud service by default | Documentation is sometimes hard to navigate |
| Multi threaded                                    |                                             |
| Can give database and user specific pool settings |                                             |

Test setup
- 8 core CPU
- 16GB memory
- PostgreSQL 14.0
- Pgbouncer 1.16.0
- Pgcat 0.1.0
- Odyssey 1.2

Dataset is initialized by a scale factor of 50. I did this because, by default pgbench creates a simple 16mb database. Scale factor 50 is not that much in terms of data size. It equals to 800mb and 5 million rows for **pgbench_accounts** table.

`pgbench -i -s 50 test_db`

Example command

`pgbench -T 60 -c 512 -j 2 -p 6437 -h 10.85.123.202 -U dba -S --protocol extended -v`

There were mainly 6 different tests starting from 16 connections to 512 connections. I have tested the following scenarios.
- 16 connections 2 threads
- 32 connections 2 threads
- 64 connections 2 threads
- 128 connections 2 threads
- 256 connections 2 threads
- 512 connections 2 threads

### Results

In terms of TPS, Odyssey is a clear winner here. It performed more than 2x better than pgbouncer. Once we reach 512 connections, it starts to manage connections better than native PostgreSQL.

![Image](/tpscp10.png)
![Image](/tpscp100.png)

In terms of latency, again Odyssey performed better than other poolers. It even had lower latency than PostgreSQL once again after we reach 512 connections.

![Image](/latencycp10.png)
![Image](/latencycp100.png)

As a result of tests, in my opinion it was no surprise that Odyssey performed better than its competitors, because it can run on multiple threads and has a more modern architecture. Main problem with Pgbouncer is that it's a really old project with only single core support. It causes multiple issues when it comes to scaling Pgbouncer, and adds complexity to your setup. On the other hand, Pgcat tries to solve this problem, and looks like it already performs better than Pgbouncer, but it's still too early in development.

### References

[https://www.postgresql.org/docs/current/pgbench.html](https://www.postgresql.org/docs/current/pgbench.html)
