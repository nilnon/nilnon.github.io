Redis
===

[Redis](https://redis.io/) 知识点

基础
----
<!--rehype:body-class=cols-2-->

### 介绍

> The open source, in-memory data store used by millions of developers as a database, cache, streaming engine, and message broker.

#### 核心能力

- In-memory data structures

Well-known as a "data structure server", with support for strings, hashes, lists, sets, sorted sets, streams, and more.

- Programmability

Server-side scripting with Lua and server-side stored procedures with Redis Functions.

- Extensibility

A module API for building custom extensions to Redis in C, C++, and Rust.

- Persistence

Keeps the dataset in memory for fast access, but can also persist all writes to permanent storage to survive reboots and system failures.

- Clustering

Horizontal scalability with hash-based sharding, scaling to millions of nodes with automatic re-partitioning when growing the cluster.

- High availability

Replication with automatic failover for both standalone and clustered deployments.


#### 用例

- Real-time data store

Redis' versatile in-memory data structures enable building data infrastructure for real-time applications that require low latency and high-throughput.

- Caching & session storage

Redis' speed makes it ideal for caching database queries, complex computations, API calls, and session state.

- Streaming & messaging

> The stream data type enables high-rate data ingestion, messaging, event sourcing, and notifications.


### 安装

#### 源码安装

```bash
wget https://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make
make install
redis-server
```

#### 单机cluster模式

install.sh

```bash
#!/bin/sh
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar -zxvf redis-5.0.5.tar.gz
REDIS=redis-5.0.5
VERSION=3.0.0
yum -y install gcc
cd $REDIS
port=600
ip=0.0.0.0
make
make install
for i in `seq 1 6`;
do
mkdir ../$port$i;
cp -r redis.conf ../$port$i;
sed -i "s/daemonize no/daemonize yes/g" ../$port$i/redis.conf;
sed -i "s/port 6379/port $port$i/g" ../$port$i/redis.conf;
sed -i "s/bind 127.0.0.1/bind $ip/g" ../$port$i/redis.conf;
sed -i "s/dir ./dir .\/$port$i/g" ../$port$i/redis.conf;
sed -i "s/# cluster-enabled yes/cluster-enabled yes/g" ../$port$i/redis.conf;
sed -i "s/# cluster-config-file nodes-6379.conf/cluster-config-file nodes-$port$i.conf/g" ../$port$i/redis.conf;
sed -i "s/# cluster-node-timeout 15000/cluster-node-timeout 15000/g" ../$port$i/redis.conf;
sed -i "s/appendonly no/appendonly yes/g" ../$port$i/redis.conf;
done
yum -y install ruby
yum -y install rubygems
gem install redis  --version $VERSION
echo 'success!'
```
start.sh
```bash
#!/bin/sh
PORT=600
REDIS=redis-5.0.5
IP=0.0.0.0
read=STR
for i in `seq 1 6`;
do
./$REDIS/src/redis-server $PORT$i/redis.conf
STR+="$IP:$PORT$i "
done
./$REDIS/src/redis-cli --cluster create $STR --cluster-replicas 1
```
stop.sh
```bash
#!/bin/sh
PORT=600
REDIS=redis-5.0.5
IP=0.0.0.0
read=STR
for i in `seq 1 6`;
do
./$REDIS/src/redis-cli -c -h $IP -p $PORT$i shutdown
echo "$REDIS/src/redis-cli -c -h $IP -p $PORT$i"
done
```

## 数据类型

### Strings

> Redis strings store sequences of bytes, including text, serialized objects, and binary arrays. As such, strings are the most basic Redis data type. They're often used for caching, but they support additional functionality that lets you implement counters and perform bitwise operations, too.

#### 命令

- GET name
- SET name value
- DEL name
- INCR key
- DECR key
- INCRBY key amount
- DECRBY key amount

#### 限制

- By default, a single Redis string can be a maximum of 512 MB.

### Lists

> Redis lists are linked lists of string values. Redis lists are frequently used to:
- Implement stacks and queues.
- Build queue management for background worker systems.

#### 命令

- RPUSH key value
- LPUSH key value
- RPOP key
- LPOP key
- LRANGE key 0 -1
- LINDEX key index
- LINDEX key index
- LLEN returns the length of a list.
- LMOVE atomically moves elements from one list to another.
- LTRIM reduces a list to the specified range of elements.
- BLMOVE atomically moves elements from a source list to a target list. If the source list is empty, the command will block until a new element becomes available.

#### 用例

- lpush+lpop=Stack(栈)
- lpush+rpop=Queue（队列）
- lpush+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）

#### 限制

The max length of a Redis list is 2^32 - 1 (4,294,967,295) elements.

#### 性能

access head or tail are O(1). LINDEX, LINSERT, and LSET O(n).

### Sets

> A Redis set is an unordered collection of unique strings (members). You can use Redis sets to efficiently:

- Track unique items (e.g., track all unique IP addresses accessing a given blog post).
- Represent relations (e.g., the set of all users with a given role).
- Perform common set operations such as intersection, unions, and differences.


#### 命令

- SADD adds a new member to a set.
- SREM removes the specified member from the set.
- SISMEMBER tests a string for set membership.
- SINTER returns the set of members that two or more sets have in common (i.e., the intersection)
- SCARD returns the size (a.k.a. cardinality) of a set.

#### 限制

The max size of a Redis set is 2^32 - 1 (4,294,967,295) members.

#### 性能

- O(1)：ost set operations, including adding, removing, and checking whether an item is a set member,
- O(n): SMEMBERS 可以使用SSCAN迭代扫描


### Hashes

> Redis hashes are record types structured as collections of field-value pairs. You can use hashes to represent basic objects and to store groupings of counters, among other things.

#### 命令

- HSET sets the value of one or more fields on a hash.
- HGET returns the value at a given field.
- HMGET returns the values at one or more given fields.
- HINCRBY increments the value at a given field by the integer provided.

#### 限制

Every hash can store up to 4,294,967,295 (2^32 - 1) field-value pairs. In practice, your hashes are limited only by the overall memory on the VMs hosting your Redis deployment.

#### 性能

- Most Redis hash commands are O(1).
- A few commands - such as HKEYS, HVALS, and HGETALL - are O(n), where n is the number of field-value pairs.


### Sorted sets

> A Redis sorted set is a collection of unique strings (members) ordered by an associated score. When more than one string has the same score, the strings are ordered lexicographically. Some use cases for sorted sets include:

- Leaderboards. For example, you can use sorted sets to easily maintain ordered lists of the highest scores in a massive online game.
- Rate limiters. In particular, you can use a sorted set to build a sliding-window rate limiter to prevent excessive API requests.

#### 命令

- ZADD adds a new member and associated score to a sorted set. If the member already exists, the score is updated.
- ZREM Remove one or more members from a sorted set
- ZRANGE returns members of a sorted set, sorted within a given range.
- ZRANK returns the rank of the provided member, assuming the sorted is in ascending order.
- ZREVRANK returns the rank of the provided member, assuming the sorted set is in descending order.

#### 性能

- Most sorted set operations are O(log(n)), where n is the number of members.
- ZRANGE  O(log(n) + m), where m is the number of results returned