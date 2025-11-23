---
layout: post
title: 尝试复现Cloudflare-2025-11-18的故障过程
---

{{ page.title }}
================

<p class="meta">23 Nov 2025 - Shanghai</p>
<p class="meta">attempt to reproduce the cloudflare outage</p>

## 项目地址
[reproduce_cf20251118](https://github.com/Laotree/reproduce_cf20251118)

## 部署步骤

### s0 - 部署 Clickhouse 集群，并初始化无故障状态

<img src="/images/posts/2025-11-23/clickhouse-cluster-init.png" alt="clickhouse-cluster-init"/>

### s1 - 部署 KV workers 集群，储存和传播配置文件

<img src="/images/posts/2025-11-23/kv-workers-init.png" alt="kv-workers-init"/>

### s3 - 部署 应用站点和代理服务，包括 FL（ bot-manager 打开/关闭 ）和 FL2

<img src="/images/posts/2025-11-23/proxy-engines-init.png" alt="proxy-engines-init"/>

> 通过3个代理服务都可以访问到用户的站点

<img src="/images/posts/2025-11-23/proxy-engines-works.png" alt="proxy-engines-works"/>

### s4 - 部署 模拟用户访问的批量访问

<img src="/images/posts/2025-11-23/customer-visits-benchmark-init.png" alt="customer-visits-benchmark-init"/>

## 演示故障

- 执行 clickhouse 权限变更

```clickhouse
clickhouse-node1 :) GRANT SELECT ON r0.* TO test_user;

GRANT SELECT ON r0.* TO test_user

Query id: 5e583e6b-2601-4f12-88f3-163f3ac021dc

Ok.

0 rows in set. Elapsed: 0.003 sec.
```

- 观察 workers kv 缓存条目翻倍

```log
[CK] cache updated, 4 columns
[CK] cache updated, 8 columns
```

- fl-bot-manager-off未受影响 
- fl-bot-manager-on开始误判机器人访问
- fl2开始反复crash

<img src="/images/posts/2025-11-23/proxy-engines-error.png" alt="proxy-engines-error"/>

<img src="/images/posts/2025-11-23/proxy-engines-fl2-crash.png" alt="proxy-engines-fl2-crash"/>

- 客户访问出现大面积失败

```log
---- http://proxy-server-fl-bot-manager-off:50001/ ----
Success rate: 100.00%
Success: 116
Blocked: 0
Total: 116
--------------------------

---- http://proxy-server-fl-bot-manager-on:50001/ ----
Success rate: 0.00%
Success: 0
Blocked: 100
Total: 100
--------------------------

---- http://proxy-server-fl2:50001/ ----
Success rate: 0.00%
Success: 0
Blocked: 106
Total: 106
--------------------------
```

## 演示故障恢复

- 执行 clickhouse 权限变更回滚

```clickhouse
clickhouse-node1 :) REVOKE SELECT ON r0.* FROM test_user;

REVOKE SELECT ON r0.* FROM test_user

Query id: 3ebc9b84-5f87-41e1-8fcb-28422bed0dd4

Ok.

0 rows in set. Elapsed: 0.021 sec.
```

- 所有访问恢复正常
Helo, have a nice day!