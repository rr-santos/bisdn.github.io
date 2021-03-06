---
title: BGPv6 (Border Gateway Protocol for IPv6 networks)
parent: Layer 3
nav_order: 6
---

# BGPv6 (Border Gateway Protocol for IPv6 networks)

## BGPv6 configuration

The FRR configuration for BGPv6 is stored in the same file for the IPv4 BGP configuration, /etc/frr/bgpd.conf. As such, the daemons file will look the same as the file used in the IPv4
configuration.

IPv6 addresses on the router must be manually written, as in the BGPv4 case. The main difference in the BGPv6 case is due to the presence of auto-configuration mechanisms for IP addresses in IPv6, which allows generating a new IP address for the servers interfaces without having to create them manually, for they are derived from the interface’s MAC addresses, and an announced network prefix in a router port. The IPv6 auto-configuration is possible via the zebra.conf configuration, like

```
interface port1
  no shutdown
  no ipv6 nd suppress-ra
  ipv6 nd prefix 2003:db01:1::/64
```

The bgpd.conf file configures routes and next-hops. The most relevant configuration difference to the BGP case is the required configuration for the next-hop address,
where FRR must ensure that we are using the globally configured IP addresses, the ones present on port53, on the image. This is due to the fact that Link-Local address can also be
used to peer across the two Basebox routers, leading to possible errors. This is done via the following configurations.

In the section address-family ipv6:

```
neighbor {{neigh}} route-map set-nexthop in
```

And in another section,

```
route-map set-nexthop permit 10
  set ipv6 next-hop peer-address
  set ipv6 next-hop prefer-global
```

Another useful parameter for BGPv6 is shutting down the default address family for IPv4, this way ensuring that configuration will
tune the BGPv6 parameters, via

```
router bgp 65000
  no bgp default ipv4-unicast
```

## BGPv6 expected result and debugging

The commands and notes mentioned in the BGP (Border Gateway Protocol) section are still relevant for this case.
