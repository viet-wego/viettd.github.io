Title: Scale out a Redis cluster
Date: 2018-11-15 18:40
Modified: 2018-11-25 18:59
Category: devops
Tags: devops, redis, cluster, scale, tutorial
Slug: scale-redis-cluster
Authors: Viet Tran
Summary: How to scale out a Redis cluster (add more nodes to the cluster).

**Update:** *This tutorial is for Redis 3 & 4. With Redis 5 the work can be done with `redis-cli` tool.*

Scaling a [Redis cluster](https://redis.io/topics/cluster-spec) sounds like a difficult task to someone never done it before.
But luckily [Redis team](https://redis.io/) has done a nice work. They provides the `redis-trib` tool to help us with some cluster tasks.
This tutorial will show you how easy it is.

## What we need

- A working [Redis cluster](https://redis.io/topics/cluster-spec).
- [Ruby](https://www.ruby-lang.org/en/).
- [Redis gem](https://rubygems.org/gems/redis/versions/4.0.3). Remember to choose the compatible version of Redis gem with your Redis version.

## Let's do it

We will start with checking current status of the cluster with this command below.

```bash
redis-trib.rb info 10.20.0.1:6379
```

_`6379` is Redis running port of host `10.20.0.1`._

We should see something like this:

```bash
10.20.0.1:6379 (4f002e35...) -> 4320889 keys | 1366 slots | 0 slaves.
10.20.0.2:6379 (a2d2d8f4...) -> 4304402 keys | 1366 slots | 0 slaves.
10.20.0.3:6379 (b021ccf0...) -> 4163969 keys | 1366 slots | 0 slaves.
10.20.0.4:6379 (0dd0889b...) -> 4183495 keys | 1366 slots | 0 slaves.
10.20.0.5:6379 (d94a5bed...) -> 4269335 keys | 1366 slots | 0 slaves.
10.20.0.6:6379 (3229648e...) -> 4206758 keys | 1359 slots | 0 slaves.
10.20.0.7:6379 (2bb07de3...) -> 4484283 keys | 1365 slots | 0 slaves.
10.20.0.8:6379 (44138262...) -> 4118749 keys | 1366 slots | 0 slaves.
10.20.0.9:6379 (fa6af69a...) -> 4237922 keys | 1366 slots | 0 slaves.
10.20.0.10:6379 (a5f5e5c5...) -> 4165017 keys | 1366 slots | 0 slaves.
10.20.0.11:6379 (f4d9872d...) -> 4326251 keys | 1366 slots | 0 slaves.
10.20.0.12:6379 (102ca6bc...) -> 4140947 keys | 1366 slots | 0 slaves.
[OK] 50922208 keys in 12 masters.
3108.04 keys per slot on average.
```

Here we have 12 masters with 0 slaves. Now we can add new node to current cluster. 

```bash
redis-trib.rb add-node 10.20.0.13:6379 10.20.0.1:6379
```

_`10.20.0.13:6379` is host & port of new node._

The out put is:

```bash
>>> Adding node 10.20.0.13:6379 to cluster 10.20.0.1:6379
>>> Performing Cluster Check (using node 10.20.0.1:6379)
M: 4f002e3542ea6a91bfba5352a0f58d439a213a81 10.20.0.1:6379
slots:5463-6828 (1366 slots) master
0 additional replica(s)
...
... <There are more information here> ...
...
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 10.20.0.13:6379 to make it join the cluster.
[OK] New node added correctly.
```

Now if we check the cluster status again, we will see that a new node has been added to the cluster.

```bash
10.20.0.1:6379 (4f002e35...) -> 4329319 keys | 1366 slots | 0 slaves.
10.20.0.2:6379 (a2d2d8f4...) -> 4523277 keys | 1366 slots | 0 slaves.
10.20.0.3:6379 (b021ccf0...) -> 4206106 keys | 1366 slots | 0 slaves.
10.20.0.4:6379 (0dd0889b...) -> 4421096 keys | 1366 slots | 0 slaves.
10.20.0.5:6379 (d94a5bed...) -> 4252139 keys | 1366 slots | 0 slaves.
10.20.0.6:6379 (3229648e...) -> 4381794 keys | 1359 slots | 0 slaves.
10.20.0.7:6379 (2bb07de3...) -> 4520956 keys | 1365 slots | 0 slaves.
10.20.0.8:6379 (44138262...) -> 4177448 keys | 1366 slots | 0 slaves.
10.20.0.9:6379 (fa6af69a...) -> 4442986 keys | 1366 slots | 0 slaves.
10.20.0.10:6379 (a5f5e5c5...) -> 4279561 keys | 1366 slots | 0 slaves.
10.20.0.11:6379 (f4d9872d...) -> 4567149 keys | 1366 slots | 0 slaves.
10.20.0.12:6379 (102ca6bc...) -> 4483896 keys | 1366 slots | 0 slaves.
10.20.0.13:6379 (ebadd321...) -> 0 keys | 0 slots | 0 slaves.
[OK] 52585715 keys in 13 masters.
3209.58 keys per slot on average.
```

We added 1 node to the cluster. Then we need to assign `key hash slots` to it. The number of slots per node will be `16384/(12+1) ~ 1260`.
Before do it, we need to get the `node id` of new node:

```bash
redis-cli -h 10.20.0.13 -p 6379 cluster nodes
```

Output:

```bash
4f002e3542ea6a91bfba5352a0f58d439a213a81 10.20.0.1:6379 master - 0 1542268658727 20 connected 0-1365
a2d2d8f40e60443e24f6fb71101ce7f8eeacf6a1 10.20.0.2:6379 master - 0 1542268658435 22 connected 1366-2730
b021ccf07f42659fd41b7bdd93ed13c1bf6ba086 10.20.0.3:6379 master - 0 1542268659024 19 connected 2732-4097
0dd0889bceb8635a1ff717215c10ef7977795329 10.20.0.4:6379 master - 0 1542268659313 24 connected 4097-5463
d94a5bed1bfb6a79667d5098eb60452771b7b53b 10.20.0.5:6379 master - 0 1542268658045 26 connected 5464-6829
3229648e1517680ea813735c45c143bc79607328 10.20.0.6:6379 master - 0 1542268659313 29 connected 6830-8188
2bb07de3a1e71cd5190d49d7cf5ba8c492c23b2c 10.20.0.7:6379 master - 0 1542268658435 16 connected 8189-9553
44138262ca209f7b119c8d726921b4261d2ea77a 10.20.0.8:6379 master - 0 1542268658533 21 connected 9554-10919
fa6af69a3f0a02c624aa08508973f853c7ae905f 10.20.0.9:6379 master - 0 1542268658045 17 connected 10920-12285
a5f5e5c59e5a9ba41de42aff6941224e780fc231 10.20.0.10:6379 master - 0 1542268658240 25 connected 12286-13651
f4d9872de81137126376698f757ad477cdaaf5ca 10.20.0.11:6379 master - 0 1542268658923 23 connected 13652-15017
102ca6bc5269221bf29483c906e0f43e8433c437 10.20.0.12:6379 master - 0 1542268659704 18 connected 15018-16383
ebadd3216f2ea34d37c66f4c5cd0fd6edd0df48c 10.20.0.13:6379 myself,master - 0 0 27 connected
```

The new node has `id=ebadd3216f2ea34d37c66f4c5cd0fd6edd0df48c`. Okay, now we reshard the cluster

```bash
redis-trib.rb reshard 10.20.0.1:6379
```

The command prompt will ask us a few questions. Answer it with our information as below:

```bash
How many slots do you want to move (from 1 to 16384)? 1260
What is the receiving node ID? ebadd3216f2ea34d37c66f4c5cd0fd6edd0df48c
Please enter all the source node IDs.
        Type 'all' to use all the nodes as source nodes for the hash slots.
        Type 'done' once you entered all the source nodes IDs.
Source node #1:all
```

Then it will propose a plan to migrate hash slots to new node. We will accept the plan by saying `yes`.

```bash
Do you want to proceed with the proposed reshard plan (yes/no)? yes
```

Let it run. In the mean time, we can enjoy a coffee. After it finishs we will get the result like this when check cluster status:

```bash
10.20.0.1:6379 (4f002e35...) -> 4041336 keys | 1261 slots | 0 slaves.
10.20.0.2:6379 (a2d2d8f4...) -> 4050861 keys | 1261 slots | 0 slaves.
10.20.0.3:6379 (b021ccf0...) -> 3891858 keys | 1261 slots | 0 slaves.
10.20.0.4:6379 (0dd0889b...) -> 4017233 keys | 1261 slots | 0 slaves.
10.20.0.5:6379 (d94a5bed...) -> 4024559 keys | 1260 slots | 0 slaves.
10.20.0.6:6379 (3229648e...) -> 4101075 keys | 1260 slots | 0 slaves.
10.20.0.7:6379 (2bb07de3...) -> 4043998 keys | 1260 slots | 0 slaves.
10.20.0.8:6379 (44138262...) -> 3953209 keys | 1260 slots | 0 slaves.
10.20.0.9:6379 (fa6af69a...) -> 3984349 keys | 1260 slots | 0 slaves.
10.20.0.10:6379 (a5f5e5c5...) -> 3924321 keys | 1260 slots | 0 slaves.
10.20.0.11:6379 (f4d9872d...) -> 4074297 keys | 1260 slots | 0 slaves.
10.20.0.12:6379 (102ca6bc...) -> 3993498 keys | 1260 slots | 0 slaves.
10.20.0.13:6379 (ebadd321...) -> 3837590 keys | 1260 slots | 0 slaves.
[OK] 51938197 keys in 13 masters.
3170.06 keys per slot on average.
```

Now our cluster is scaled.

---

## References

- [Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)