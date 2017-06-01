---
title: hiredis 异步
tags: [hiredis, redis, 异步, async]
tags: [C++]
---

### hiredis
hiredis是redis官方库, 提供了同步与异步的接口. 对于高性能的服务器, 异步的接口给我们很大的发挥空间.

#### 同步还是异步
使用同步还是异步并没有绝对的定论, 我们可以根据一些性能表现确定使用同步或是异步. redis性能非常好, 以至于在插入或是读取单条数据的时候几乎是感觉不到差异的. 可是一旦同时插入大量数据, 同步延迟非常严重, 读取平均长度30的kv, 大约是10万/秒, 如果我们有千万级别的数据, 整个卡顿会上分钟, 而且带宽会被打满, 所以一般大数据的读取, 我们会采用异步的方法.

### 异步状态

#### 初始化异步回调
```
redisLibeventAttach(m_async_context, m_base); //libevent attach
redisAsyncSetConnectCallback(m_async_context, &redis_base::on_connect); // 连接成功回调
redisAsyncSetDisconnectCallback(m_async_context, &redis_base::on_disconnect); // 连接失败回调
redisAsyncCommand(m_async_context, &redis_base::on_command, privdata, "HGET %s %s", db.c_str(), key.c_str()); // 命令回调
```

#### 异步状态
发送一个redisAsyncCommand的时候, hiredis回调到之前设置的一个静态函数, 可是如果发送了成千上万的命令, 那么怎么区分回调的时候那个命令呢？ 不用着急, hiredis提供了一个私有数据指针，我们可以通过回调恢复这个指针, 然后管理整个异步流程。

```
// 自定义私有数据变量
struct redis_priv
{
	redis_priv():
		type(REDIS_BEGIN)
	{}
	~redis_priv(){}
	redis_type type;
	std::string db;
	std::string key;
	std::string value;
};

redis_priv* privdata = new redis_priv();
privdata->key = key;
privdata->type = type;
redisAsyncCommand(m_async_context, &redis_base::on_command, privdata, "HGET %s %s", db.c_str(), key.c_str()); // 构造函数的时候, 同时传入私有变量


// 回调
void redis_base::on_command(redisAsyncContext* c, void* r, void* privdata)
{
	assert(c);
	assert(c->data);
	if (c == NULL || c->data == NULL)
	{
#ifdef _DEBUG
		printf("redis server 收到空reply\n");
#endif
		return;
	}
	redis_base* pthis = static_cast<redis_base*>(c->data); // 恢复回调主体
	redis_priv* priv = static_cast<redis_priv*>(privdata); // 恢复私有数据
	// 业务
  // 释放指针(一定要注意释放时机)
  if (priv)
    delete priv;
}
```

### 踩坑
#### 析构函数
C/C++写多了, 当一个指针发生异常的时候, 通常第一步就是去释放它, hiredis提供了两个释放的函数
```
redisAsyncDisconnect(m_async_context);
redisAsyncFree(m_async_context)
```

但是一定不要主动去释放掉它

但是一定不要主动去释放掉它

但是一定不要主动去释放掉它

一旦发生异常, 就直接置为空!

```
// https://github.com/redis/hiredis/issues/358
// 直接把context置为NULL, 一旦发生错误, 底层会主动释放, context, 如果客户端要主动断开连接, 只需要调用redisAsyncDisconnect(m_async_context), 并把m_async_context 置为空, 不要调用redisAsyncFree, 否则会有重复释放的问题
m_async_context = NULL;
```

上下文已经被底层管理好了, 除非我们要主动关闭连接, 否则发生异常, 一定不要尝试释放掉它！
