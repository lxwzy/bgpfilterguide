---
layout: page
title: BGP 过滤指南
permalink: /
---

### RPKI

参见 NLNetLabs 的 [RPKI FAQ](https://github.com/NLnetLabs/rpki-faq/blob/master/faq.rst)

### 虚假 (Bogon) ASN 过滤

拒绝在 AS_PATH 中具有虚假 (Bogon) ASN 的路由: [bogon_asn](/guides/bogon_asns/).

### 虚假 (Bogon) 前缀过滤

拒绝虚假 (Bogon) 的路由: [bogon_prefixes](/guides/bogon_prefixes/).

### 过小的前缀过滤

拒绝拥有过小前缀的路由: [拒绝过小前缀](/guides/small_prefixes/).

### 过滤过长 AS 路径(Paths)

过滤很长的 AS 路径: [拒绝过长 AS 路径](/guides/long_paths/).

### 过滤有 运输(Transit) 网络的 AS 路径(Paths)

拒绝 AS 路径(Paths) 中包含已知运输网络的路由: [拒绝有运输网络的 AS 路径](/guides/no_transit_leaks/).

### Remainder Accept term

A Remainder accept term with BGP communities and a local-preference: [Remainder accept term](/guides/remainder_accept/).

### 支持 优雅停机(GRACEFUL_SHUTDOWN)

如何让 BGP 会话(sessions) 优雅地停机: [优雅停机](/guides/graceful_shutdown/).
