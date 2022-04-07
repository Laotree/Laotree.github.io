---
layout: post
title: 让我的接口更高能
---

{{ page.title }}
================

<p class="meta">28 Oct 2020 - Shanghai</p>
<p class="meta">code in a better way</p>

##  任务背景
最近我制定了一个OKR，对公司现有的接口进行优化和升级
看病需要先知道得了什么病
于是我的第一步就是对现有接口的健康状况进行一个检查
- 一个非常基础的指标：接口的响应时间

本文的api是用golang的***gin***框架编写

##  行动过程

### s0 - 思路
> 记下每次请求的响应时间，把记录存入（选的mongo），汇总展示

### s1 - 记录接口响应时间

> 存储的结构

```go
type Elapse struct {
	ID        bson.ObjectId `bson:"_id,omitempty" json:"id"` 
        // 请求ID，用雪花算法给每次请求生成一个，方便对请求的日志、事件进行归类
	RequestID string        `bson:"request_id" json:"request_id"`
        // 请求路径 
	Path      string        `bson:"path" json:"path"`
        // 单位ms
	TimeCost  int64         `bson:"time_cost" json:"time_cost"`
	Ctime     time.Time     `bson:"ctime" json:"ctime"`
}
```

> 中间件

```go
package pin

func Pin() gin.HandlerFunc {
	return func(context *gin.Context) {
		start := time.Now()
		defer recordTimeCost(start, context)
		context.Next()
	}
}

func recordTimeCost(start time.Time, context *gin.Context) {
	requestID := context.GetString("request_id")
	path := context.FullPath()
	elapsed := &pin.Elapse{
		RequestID: requestID,
		Path:      path,
		TimeCost:  time.Since(start).Milliseconds(),
		Ctime:     start,
	}
        // 使用mongo进行存储
	go saveRecordToMongo(elapsed)
	return
}
```

> 在router中进行调用

```go
package router

	r := gin.Default()
	r.Use(pin.Pin())
```

### s2 - 存储记录的值

```mongojs
{
        "_id" : ObjectId("5f97d25c133fe8f76cfbf6e1"),
        "request_id" : "85308KPQDEKDb",
        "path" : "/test",
        "time_cost" : NumberLong(2),
        "ctime" : ISODate("2020-10-27T07:55:08.831Z")
}
```

### s3 - 把数据汇总展示
<img src="/images/posts/2020-10-28/stats.jpg"  alt="example"/>
