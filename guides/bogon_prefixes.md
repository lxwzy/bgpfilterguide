---
layout: page
title: 虚假 (Bogon) 前缀
permalink: /guides/bogon_prefixes/
---

* TOC
{:toc}

# 虚假 (Bogon) 前缀过滤

## 目的

这些不是 全局唯一(Globally unique) 的前缀。IETF并未规划将它们路由到公共互联网上。

# IPv4 配置示例

## BIRD
```
define BOGON_PREFIXES = [ 0.0.0.0/8+,         # RFC 1122 'this' network
                          10.0.0.0/8+,        # RFC 1918 private space
                          100.64.0.0/10+,     # RFC 6598 Carrier grade nat space
                          127.0.0.0/8+,       # RFC 1122 localhost
                          169.254.0.0/16+,    # RFC 3927 link local
                          172.16.0.0/12+,     # RFC 1918 private space 
                          192.0.2.0/24+,      # RFC 5737 TEST-NET-1
                          192.88.99.0/24+,    # RFC 7526 6to4 anycast relay
                          192.168.0.0/16+,    # RFC 1918 private space
                          198.18.0.0/15+,     # RFC 2544 benchmarking
                          198.51.100.0/24+,   # RFC 5737 TEST-NET-2
                          203.0.113.0/24+,    # RFC 5737 TEST-NET-3
                          224.0.0.0/4+,       # multicast
                          240.0.0.0/4+ ];     # reserved

function reject_bogon_prefixes()
prefix set bogon_prefixes;
{
    bogon_prefixes = BOGON_PREFIXES;
    if (net ~ bogon_prefixes) then {
        print "Reject: Bogon prefix: ", net, " ", bgp_path;
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
## OpenBGPD

复制于 [OpenBSD 示例](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L97-L109)

```
deny from any prefix 0.0.0.0/8 prefixlen >= 8           # 'this' network [RFC1122]
deny from any prefix 10.0.0.0/8 prefixlen >= 8          # private space [RFC1918]
deny from any prefix 100.64.0.0/10 prefixlen >= 10      # CGN Shared [RFC6598]
deny from any prefix 127.0.0.0/8 prefixlen >= 8         # localhost [RFC1122]
deny from any prefix 169.254.0.0/16 prefixlen >= 16     # link local [RFC3927]
deny from any prefix 172.16.0.0/12 prefixlen >= 12      # private space [RFC1918]
deny from any prefix 192.0.2.0/24 prefixlen >= 24       # TEST-NET-1 [RFC5737]
deny from any prefix 192.88.99.0/24 prefixlen >= 24     # 6to4 anycast relay [RFC7526]
deny from any prefix 192.168.0.0/16 prefixlen >= 16     # private space [RFC1918]
deny from any prefix 198.18.0.0/15 prefixlen >= 15      # benchmarking [RFC2544]
deny from any prefix 198.51.100.0/24 prefixlen >= 24    # TEST-NET-2 [RFC5737]
deny from any prefix 203.0.113.0/24 prefixlen >= 24     # TEST-NET-3 [RFC5737]
deny from any prefix 224.0.0.0/4 prefixlen >= 4         # multicast
deny from any prefix 240.0.0.0/4 prefixlen >= 4         # reserved for future use
```
## Junos
```
policy-options {
    prefix-list BOGONS_v4 {
        0.0.0.0/8;
        10.0.0.0/8;
        100.64.0.0/10;
        127.0.0.0/8;
        169.254.0.0/16;
        172.16.0.0/12;
        192.0.2.0/24;
        192.88.99.0/24;
        192.168.0.0/16;
        198.18.0.0/15;
        198.51.100.0/24;
        203.0.113.0/24;
        224.0.0.0/4;
        240.0.0.0/4;
    }
    policy-statement BGP_FILTER_IN {
        term IPv4 {
            from {
                prefix-list BOGONS_v4;
            }
            then reject;
        }
    }
}
```
## IOS-XR
```
prefix-set BOGONS_V4
  0.0.0.0/8 le 32,
  10.0.0.0/8 le 32,
  100.64.0.0/10 le 32,
  127.0.0.0/8 le 32,
  169.254.0.0/16 le 32,
  172.16.0.0/12 le 32,
  192.0.2.0/24 le 32,
  192.88.99.0/24 le 32,
  192.168.0.0/16 le 32,
  198.18.0.0/15 le 32,
  198.51.100.0/24 le 32,
  203.0.113.0/24 le 32,
  224.0.0.0/4 le 32,
  240.0.0.0/4 le 32
end-set
!
route-policy BGP_FILTER_IN
  if destination in BOGONS_V4 then
    drop
  endif
end-policy
```
# IPv6 配置示例

## BIRD
```
define BOGON_PREFIXES = [ ::/8+,                         # RFC 4291 IPv4-compatible, loopback, et al 
                          0100::/64+,                    # RFC 6666 Discard-Only
                          2001:2::/48+,                  # RFC 5180 BMWG
                          2001:10::/28+,                 # RFC 4843 ORCHID
                          2001:db8::/32+,                # RFC 3849 documentation
                          2002::/16+,                    # RFC 7526 6to4 anycast relay
                          3ffe::/16+,                    # RFC 3701 old 6bone
                          fc00::/7+,                     # RFC 4193 unique local unicast
                          fe80::/10+,                    # RFC 4291 link local unicast
                          fec0::/10+,                    # RFC 3879 old site local unicast
                          ff00::/8+                      # RFC 4291 multicast
 ];

function reject_bogon_prefixes()
prefix set bogon_prefixes;
{
    bogon_prefixes = BOGON_PREFIXES;
    if (net ~ bogon_prefixes) then {
        print "Reject: Bogon prefix: ", net, " ", bgp_path;
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

        honor_graceful_shutdown();
        accept;
}

```

## OpenBGPD

复制于 [OpenBSD 示例](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L111-L121)

```
deny from any prefix ::/8 prefixlen >= 8
deny from any prefix 0100::/64 prefixlen >= 64          # Discard-Only [RFC6666]
deny from any prefix 2001:2::/48 prefixlen >= 48        # BMWG [RFC5180]
deny from any prefix 2001:10::/28 prefixlen >= 28       # ORCHID [RFC4843]
deny from any prefix 2001:db8::/32 prefixlen >= 32      # docu range [RFC3849]
deny from any prefix 2002::/16 prefixlen >= 16          # 6to4 anycast relay [RFC7526]
deny from any prefix 3ffe::/16 prefixlen >= 16          # old 6bone
deny from any prefix fc00::/7 prefixlen >= 7            # unique local unicast
deny from any prefix fe80::/10 prefixlen >= 10          # link local unicast
deny from any prefix fec0::/10 prefixlen >= 10          # old site local unicast
deny from any prefix ff00::/8 prefixlen >= 8            # multicast
```

## Juniper 和 Cisco

Gert Doering's [ipv6-filters](https://www.space.net/~gert/RIPE/ipv6-filters.html)

## YAML from Coloclue

Coloclue's network management system [kees](https://github.com/coloclue/kees) considers these the IPv6 Bogons: [yaml file](https://github.com/coloclue/kees/blob/master/vars/generic.yml#L70-L156)
