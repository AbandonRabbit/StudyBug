# 字符串String

1. 设置

   ```SET key value```

2. 获取
   ```GET key```

3. 删除
   ```DEL key```

4. 判断是否存在
   ```EXISTS key```

5. 查看有哪些键
   ```KEYS [pattern模式匹配]```

6. 删除所有键
   ```FLUSHALL```

7. 查看过期时间
   ```TTL key```

8. 设置过期时间
   ```EXPIRE key time/秒```

9. 设置一个带有过期时间的键值对
   ```SETEX key time value```

10. 当键不存在时设置该键值对
    ```SETNX key value```

# 列表List

存储或操作一组有顺序的数据

1. 尾部添加
   ```RPUSH```
2. 头部添加
   ```LPUSH```
3. 查看
   ```LRANGE key start stop```
4. 头删除
   ```RPOP key [删除元素个数]```
5. 尾删除
   ```LPOP key``` [删除元素个数]
6. 指定范围删除
   ```LTRIM key start stop```

# 集合Set

无序集合，不允许重复元素

1. 添加
   ```SADD key member```
2. 判断一个元素是否在集合中
   ```SISMEMBER key member```
3. 删除
   ```SREM key member```
4. 查看集合元素
   ```SMEMBERS key```

# 有序集合SortedSet

每个元素关联一个浮点类型的分数，按照分数对集合中的元素进行从小到大排序，成员是唯一的，分数是可以重复的

1. 添加
   ```ZADD key score1 value1 [score2 value2 ……]```

2. 查看
   ```Redis
   ZRANGE key start end [WITHSCORES]
   WITHSCORES：同时输出分数
   
   //查看分数
   ZSCORE key value
   
   //查看排名
   ZRANK key value
   ```


# 哈希Hash

   字符类型的字段和值的映射表，简单来说就是一个键值对的集合，特别适合存储对象

   1. 添加
      ```HEST key field1 value1 [fild2 value2 ……]```

   2. 获取
      ```HGET key field```

   3. 获取所有键值对
      ```HGETALL key```

   4. 删除键值对
      ```HEXISTS key field```

   5. 获取所有键
      ```HKEYS key```

   6. 获取键的数量

      ```HLEN key```

# 发布订阅模式

无法持久化，无法记录历史

发送：PUBLISH

接受：SUBSCRIBE 频道名称

# 消息队列Stream

轻量级消息队列

1. 添加消息
   ```XADD key * field value```

   \* 表示随机id

2. 消息数量
   ```XLEN key```

3. 消息详细内容
   ```XRANGE key start end```
   可以用 - + 表示显示所有内容

4. 删除
   ```XDEL key ID```

5. 读取消息
   `XREAD [COUNT 消息数量] [BLOCK 阻塞时间] STREAMS key start`

   start 可以为$符，表示从执行时间开始阻塞，接受最新的消息
