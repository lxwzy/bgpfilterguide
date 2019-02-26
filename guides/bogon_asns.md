---
layout: page
title: 虚假 (Bogon) ASN 过滤
permalink: /guides/bogon_asns/
---

* TOC
{:toc}

# 虚假 (Bogon) ASN 过滤

最早出版于: [http://as2914.net/bogon_asns/configuration_examples.txt](http://as2914.net/bogon_asns/configuration_examples.txt)

## 目的

私有或预留 ASN 不应该在公共 DFZ 中。阻止这些来自 DFZ 的错误有助于加强问责并减少暴露内部路由的意外。

所有现代设备都支持4字节 ASN。DFZ 中出现的“23456”可能是配置错误或软件问题。

拒绝 AS_PATH 中包含虚假 (Bogon) ASN 的 EBGP 路由是一种[Fail-fast(快速失败)](https://en.wikipedia.org/wiki/Fail-fast)的形式。

# 配置示例

## Junos

```
policy-options {
    as-path-group bogon-asns {
        /* RFC7607 */
        as-path zero ".* 0 .*";
        /* RFC 4893 AS_TRANS */
        as-path as_trans ".* 23456 .*";
        /* RFC 5398 and documentation/example ASNs */
        as-path examples1 ".* [64496-64511] .*";
        as-path examples2 ".* [65536-65551] .*";
        /* RFC 6996 Private ASNs*/
        as-path reserved1 ".* [64512-65534] .*";
        as-path reserved2 ".* [4200000000-4294967294] .*";
        /* RFC 6996 Last 16 and 32 bit ASNs */
        as-path last16 ".* 65535 .*";
        as-path last32 ".* 4294967295 .*";
        /* RFC IANA reserved ASNs*/
        as-path iana-reserved ".* [65552-131071] .*";
    }
    policy-statement import_from_ebgp {
        term bogon-asns {
            from as-path-group bogon-asns;
            then reject;
        }
        term .....
    }
}
```

## IOS-XR

```
as-path-set bogon-asns
  # RFC7607
  ios-regex '_0_',
  # 2 to 4 byte ASN migrations
  passes-through '23456',
  # RFC5398
  passes-through '[64496..64511]',
  passes-through '[65536..65551]',
  # RFC6996
  passes-through '[64512..65534]',
  passes-through '[4200000000..4294967294]',
  # RFC7300
  passes-through '65535',
  passes-through '4294967295',
  # IANA reserved
  passes-through '[65552..131071]'
end-set

route-policy import_from_ebgp
  if as-path in bogon-asns then
    drop
  else
    ......
  endif
end-policy
```

## BIRD

```
define BOGON_ASNS = [ 0,                      # RFC 7607
                      23456,                  # RFC 4893 AS_TRANS
                      64496..64511,           # RFC 5398 and documentation/example ASNs
                      64512..65534,           # RFC 6996 Private ASNs
                      65535,                  # RFC 7300 Last 16 bit ASN
                      65536..65551,           # RFC 5398 and documentation/example ASNs
                      65552..131071,          # RFC IANA reserved ASNs
                      4200000000..4294967294, # RFC 6996 Private ASNs
                      4294967295 ];           # RFC 7300 Last 32 bit ASN

function reject_bogon_asns()
int set bogon_asns;
{
    bogon_asns = BOGON_ASNS;
    if ( bgp_path ~ bogon_asns ) then {
        print "Reject: bogon AS_PATH: ", net, " ", bgp_path;
        reject;
    }
}

...

filter transit_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_small_prefixes();
        reject_default_route();

...

        honor_graceful_shutdown();
        accept;
}

filter ixp_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_transit_paths();
        reject_small_prefixes();
        reject_default_route();

...

        honor_graceful_shutdown();
        accept;
}


```

## Nokia SR OS

```
bgp
    error-handling
        # RFC 7607 AS 0
        update-fault-tolerance
    exit
exit

policy-options
    begin
    as-path-group "bogon-asns"
        # RFC 4893 AS_TRANS
        entry 10 expression ".* 23456 .*"
        # RFC 5398 and documentation/example ASNs
        entry 15 expression ".* [64496-64511] .*"
        entry 20 expression ".* [65536-65551] .*"
        # RFC 6996 private ASNs
        entry 25 expression ".* [64512-65534] .*"
        entry 30 expression ".* [4200000000-4294967294] .*"
        RFC 6996 last 16-bit and 32-bit ASNs
        entry 35 expression ".* 65535 .*"
        entry 40 expression ".* 4294967295 .*"
        # IANA reserved ASNs
        entry 45 expression ".* [65552-131071] .*"
    exit
    policy-statement "import_from_ebgp"
        entry 10
            from
                as-path-group "bogon-asns"
            exit
            action reject
        exit
    exit
    commit
exit
```

## OpenBGPD

复制于 [OpenBSD 示例](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L123-L132)

```
deny from any AS 23456                          # AS_TRANS
deny from any AS 64496 - 64511                  # Reserved for use in docs and code RFC5398
deny from any AS 64512 - 65534                  # Reserved for Private Use RFC6996
deny from any AS 65535                          # Reserved RFC7300
deny from any AS 65536 - 65551                  # Reserved for use in docs and code RFC5398 
deny from any AS 65552 - 131071                 # Reserved
deny from any AS 4200000000 - 4294967294        # Reserved for Private Use RFC6996
deny from any AS 4294967295                     # Reserved RFC7300
```
