---
title: Redis在实际工作中遇到的各种应用场景
date: 2019-01-30 13:39:48
tags: redis
---

> Redis作为一种内存类型的数据库工具，在实际工作中除了常见的用来做缓存外，还有很多实际的用处；如并发场景下的的事务锁、实现统计功能、sug等。

<!-- more -->

Redis有很多种数据结构：字符串(string)、散列(hash)、列表(list)、集合(set)、有序集合(zset，使用Skip List跳跃表实现)、位图(bitmap)、HyperLogLog、地图坐标(GEO)；Redis相比于其他的NoSQL数据库在执行速度方面快很多且效率很高，这是因为它是基于内存来存储相关数据的，且它是基于多路复用的时间响应系统。

Redis支持事务、内存淘汰、持久化(AOF、RDB、RDB+AOF)。

多机支持：主从复制（单master多slave）、高可用、集群。

eg: 获取Redis客户端

```go
package util 
import "github.com/go-redis/redis" 
func RedisClient() *redis.Client { 
  client := redis.NewClient(&redis.Options{ 
    Addr: "127.0.0.1:6379", 
    Password: "", 
    DB: 0, 
  }) 
  return client 
}
```

#### 实际场景应用

##### 一，锁

> 锁是⼀一种同步机制， 它可以保证⼀一项资源在任何时候只能被⼀一个进程使⽤用， 如果有其他进程想要 使⽤用相同的资源， 那么它们就必须等待， 直到正在使⽤用资源的进程放弃使⽤用权为⽌止。

锁的两种操作：获取(acquire)和释放(release)。

实现锁通常用到三种方式来实现：字符串、事务和带NX的SET。

- 字符串

将⼀一个字符串串键⽤用作锁，如果这个键有值，那么说明锁已被获取;反之，如果键没有值，那么说明
锁未被获取，程序可以通过为其设置值来获取锁。

```go
package util 
import "github.com/chunlintang/frequency-limit/util" 

const LOCK_KEY = "lock_key" 
const LOCK_VALUE = "lock_value" 
var client = util.RedisClient() 
// 获取锁，查看所是否存在 
func acquire() bool { 
  current_value, _ := client.Get(lock_key).Result() 
  if current_value == "" { 
    if _, err := client.Set(lock_key, lock_value, 0).Result(); err != nil { 
      return false 
    } 
    return true 
  } 
  return false 
} 
// 释放锁 
func release() bool { 
  if _, err := client.Del(lock_key).Result(); err != nil { 
    return false 
  } 
  return true 
}
```

> 弊端

因为 GET 和 SET 在两个不不同的请求中执⾏行行，在它们之间可 能有其他请求已经改变了了锁键的值，导致“锁键为空”这⼀一判 断不不再为真，从⽽而引发多个客户端之间的竞争条件。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190217123339.png)

- 事务

锁的基本实现⽅方法跟之前⼀一样，但使⽤用 Redis 的事务特性保证操作的安全性。

```go
func acquire() bool { 
  txf := func(tx *redis.Tx) error { 
    // get current value or zero 
    n, err := tx.Get(lock_key).Int() 
    if err != nil && err != redis.Nil { 
      return err 
    } 
    // actual opperation (local in optimistic lock) 
    n++ 
    // runs only if the watched keys remain unchanged 
    _, err = tx.Pipelined(func(pipe redis.Pipeliner) error { 
      // pipe handles the error case 
      pipe.Set(lock_key, n, 0) 
      return nil 
    }) 
    return err 
  } 
  err := client.Watch(txf, lock_key) 
  if err != redis.TxFailedErr { 
    return false 
  } 
  current_value, _ := client.Get(lock_key).Result() 
  if current_value == "" { 
    pl := client.TxPipeline() 
    client.Set(lock_key, lock_value, 0) 
    repl, _ := pl.Exec() 
    if repl != nil { 
      return true 
    } 
  } 
  return false 
}
```

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190217131144.png)

- 带NX的SET

```go
func acquire() bool  {
    repl, _ := client.SetNX(lock_key, lock_value, 0).Result()
    return repl
}
```

##### 二，计数器，统计当前在线用户数

可使用集合、位图和HyperLogLog实现。

- 集合

当⼀一个新的用户上线时，将它的⽤用户名添加到记录在线⽤用户的集合当中。

```go
const online_user = "ONLINE_USER" 
var client = util.RedisClient() 

func setOnline(user string) bool { 
  if _, err := client.SAdd(online_user, user).Result(); err != nil { 
    return false 
  } 
  return true 
} 

func countOnline() int64 { 
  repl, _ := client.SCard(online_user).Result() 
  return repl 
} 

func existOnline(user string) bool { 
  repl, _ := client.SIsMember(online_user, user).Result() 
  return repl 
}
```

> 弊端

集合的体积将随着元素的增加⽽而增加，集合包含的元素越多，每个元素的体积越⼤大，集合的体积也
就越⼤大。另外，因为使⽤用 Redis 储存信息还有⼀一些额外的消耗(overhead)，所以实际的内存占⽤用数量量 将⽐比这个估算值更更⾼高。

- 位图

为每个⽤用户创建⼀一个相对应的数字 ID ，当⼀一个⽤用户上线时，使⽤用他的 ID 作为索引，将位图指定索 引上的⼆二进制位设置为 1 。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190217134254.png)

```go
const online_user_bitmap = "ONLINE_USER_BITMAP" 

func setOnline(user_id int64) bool { 
  if _, err := client.SetBit(online_user_bitmap, user_id, 1).Result(); err != nil {
    return false 
  } 
  return true 
} 

func countOnline() int64 { 
  repl, _ := client.BitCount(online_user_bitmap, &redis.BitCount{0, 0}).Result() 
  return repl 
} 

func existOnline(user_id int64) bool { 
  repl, _ := client.GetBit(online_user_bitmap, user_id).Result() 
  return repl == 1 
}
```

虽然位图的体积仍然会随着⽤用户数量量的增多⽽而变⼤大，但因为记录每个⽤用户所需的内存数量量从原来的 平均 10 字节变成了了 1 位，所以实现⽅方法⼆二将节约⼤大量量内存。

- HyperLogLog

当⼀一个⽤用户上线时，使⽤用统计在线⽤用户数量量的 HyperLogLog 对其进⾏行行计数。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190217140531.png)

```go
const online_use_hll = "ONLINE_USER_HLL"

func setOnline(user string) {
    client.PFAdd(online_use_hll, user)
}

func countOnline() int64 {
    repl, _ := client.PFCount(online_use_hll).Result()
    return repl
}
```

##### 三，自动补全

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190217141017.png)

利用权重实现。

```go
package autocomplete 

import ( 
  "fmt" "github.com/chunlintang/frequency-limit/util" 
) 

const autocomplete = "autocomplete::" 

var client = util.RedisClient() 

func Feed(prefix, content string, weight float64) { 
  for i, _ := range prefix { 
    segment := prefix[:i+1] 
    key := autocomplete + segment 
    if _, err := client.ZIncrBy(key, weight, content).Result(); err != nil {
      fmt.Println("ZIncrBy err") 
    } 
  } 
} 

func Hint(prefix string, count int64) []string { 
  key := autocomplete + prefix 
  //result, _ := client.Cmd("ZREVANGE", key, 0, count-1).List() 
  result, _ := client.ZRevRange(key, 0, count-1).Result() 
  return result 
}
```

