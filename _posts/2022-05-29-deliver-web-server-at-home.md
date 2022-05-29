---
layout: post
title: 在家里部署一个公开访问的web服务
---

{{ page.title }}
================

<p class="meta">29 May 2022 - Shanghai</p>

## 出发点

搭建一个web服务之后，部署到哪里一直有比较定向的思维，也是就到公有云提供商买一台机器发布。 如果作为新用户，可以享受一些优惠的价格，但是一旦变成熟客，便有价格的不适。 于是就想着在家里使用闲置的机器，搭建可以公开访问的web服务。

## 思路

想起一道非常经典的面试题：当你在搜索栏里Google的时候，发生了什么，详细的讲一下。

- 使用未回收的电子垃圾扎出来的一台小机器
- 在机器上部署web服务
- 使用Nginx进行请求到达本机后的转发
- 家里有2层路由转发，对路由器做好Port Forward，保证隧道的流量可以转发
- 让互联网知道这里有一个web服务，这是最复杂的一点
  - 基于公网IP的稀缺与不易的，我找到的方法是使用 [参考：使用Cloudflare Argo Tunnel快速免公网IP建站 - Zapic's Blog](https://blog.zapic.moe/archives/tutorial-176.html)
    - 将购买的域名托管至Cloudflare
    - 申请使用CloudFlare Tunnel的最低配版（这里会需要付费1美元然后退还）
    - 在家里的机器上安装 [cloudflared.service](https://github.com/cloudflare/cloudflared/)
    - 授权域名创建tunnel，这里会自动生成CNAME的转发规则
    - 到这里就可以基本愉快地从互联网访问家里的web服务里
    
<img src="/images/posts/2022-05-29/tunnel.png" alt="tunnel"/>

## TODO

- 一旦公开访问，就会遭遇到未知世界的探测甚至攻击，最好机器能与其他家用设备网络、硬件隔离
- 互联网到隧道入口的连接非常稳定，但是进入隧道后到达家里的web服务就不是非常稳定了，暂时可以靠定时重启隧道链接保证不会一直挂着。也可以通过购买负载均衡等等增值服务提高稳定性。不花钱的前提下，看了一下增值服务的介绍，大概就是做好监控，断链恢复，是个DIY的好故事。
