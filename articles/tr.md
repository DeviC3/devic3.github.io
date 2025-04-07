# Traceroute: discovering network paths

## Introduction

Traceroute is one of the fundamental diagnostic tools used by IT administrators to analyze the paths that data packets take through networks (both local and Internet). This tool allows tracking the route of packets from source to destination, showing each node (router) along the way and the time needed to reach it. In this article, we'll look at how traceroute works in Linux systems.

## How does traceroute work?

### Basic concept

Traceroute uses the IP protocol and TTL (Time To Live) mechanism to discover network paths. The operating principle is as follows:

1. The tool sends a series of packets (by default using UDP, ICMP, or TCP protocol) to the destination host.
2. Each packet has an assigned TTL value that determines the maximum number of hops it can make before being discarded.
3. Traceroute starts by sending a packet with TTL=1, then gradually increases this value for subsequent packets.
4. When a router receives a packet, it decreases the TTL value by 1. If TTL reaches 0, the router discards the packet and returns an ICMP "Time Exceeded" message to the sender.
5. Traceroute measures the time between sending the packet and receiving the ICMP response, which determines the latency to that router.
6. The tool continues sending packets with increasing TTL values until it reaches the destination host or exceeds the maximum number of hops.

### Implementation types

Traceroute can use different protocols for operation:

- **UDP** (traditional Unix/Linux implementation): Sends UDP packets to likely unused destination ports. When the packet reaches its destination, the target server responds with an ICMP "Port Unreachable" message.
- **ICMP**: Uses Echo Request packets (ping) with increasing TTL.
- **TCP**: Uses TCP packets, which can be useful in networks where UDP/ICMP packets are filtered by firewalls.

### Interpreting results

Typical traceroute results include:
- Hop number
- Router IP addresses (and sometimes their DNS names)
- Response times (RTT - Round Trip Time) for each router (usually three attempts are shown)
- Asterisks (*) indicating no response (due to filtering, congestion, etc.)

## Modern implementations

Contemporary traceroute implementations in Linux, such as those in the `iputils` package, offer additional features:

- **Parallel probing** - Sending packets to multiple hops simultaneously for faster results
- **AS Path lookup** - Showing the autonomous system (AS) number for each hop
- **MTU Discovery** - Detecting maximum transmission unit along the route

## Traceroute usage examples

Basic usage is very simple:

```bash
traceroute example.com
```

Example output:

```
traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  2.162 ms  2.187 ms  2.324 ms
 2  192.168.100.1 (192.168.100.1)  12.644 ms  12.862 ms  13.030 ms
 3  host-72-174-144-1.broadband.pl (72.174.144.1)  14.221 ms  14.371 ms  14.516 ms
 4  ae21-0.icr01.waw04.atlas.cogentco.com (149.14.58.133)  15.578 ms  15.689 ms  19.404 ms
 5  be3035.ccr21.ham01.atlas.cogentco.com (154.54.38.205)  32.141 ms  32.247 ms  32.352 ms
 6  be3037.ccr21.ams03.atlas.cogentco.com (154.54.56.93)  35.170 ms  35.289 ms  35.400 ms
 7  * * *
 8  * * *
 9  93.184.216.34 (93.184.216.34)  88.305 ms  88.410 ms  88.510 ms
```

## Other traceroute features

### Parallel probing

```bash
# Send parallel probes to multiple nodes (default 16)
traceroute -N 20 example.com
```

Example output:

```
traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  1.923 ms
 2  192.168.100.1 (192.168.100.1)  10.552 ms
 3  host-72-174-144-1.broadband.pl (72.174.144.1)  12.411 ms
 4  ae21-0.icr01.waw04.atlas.cogentco.com (149.14.58.133)  14.563 ms
 5  be3035.ccr21.ham01.atlas.cogentco.com (154.54.38.205)  30.984 ms
 6  be3037.ccr21.ams03.atlas.cogentco.com (154.54.56.93)  33.847 ms
 7  * 
 8  * 
 9  93.184.216.34 (93.184.216.34)  87.219 ms
```

### AS path lookup

```bash
# Show AS numbers for each hop
traceroute -A example.com
```

Example output:

```
traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1) [*]  2.015 ms  2.133 ms  2.255 ms
 2  192.168.100.1 (192.168.100.1) [*]  10.763 ms  10.971 ms  11.140 ms
 3  host-72-174-144-1.broadband.pl (72.174.144.1) [AS12741]  12.538 ms  12.714 ms  12.880 ms
 4  ae21-0.icr01.waw04.atlas.cogentco.com (149.14.58.133) [AS174]  14.723 ms  14.859 ms  15.002 ms
 5  be3035.ccr21.ham01.atlas.cogentco.com (154.54.38.205) [AS174]  31.239 ms  31.352 ms  31.465 ms
 6  be3037.ccr21.ams03.atlas.cogentco.com (154.54.56.93) [AS174]  34.140 ms  34.259 ms  34.370 ms
 7  * * *
 8  * * *
 9  93.184.216.34 (93.184.216.34) [AS15133]  87.497 ms  87.615 ms  87.728 ms
```

### MTU discovery

```bash
# Using tracepath for MTU discovery
tracepath example.com
```

Example output:

```
 1?: [LOCALHOST]                      pmtu 1500
 1:  _gateway                                               1.849ms 
 1:  _gateway                                               1.755ms 
 2:  192.168.100.1                                         11.079ms 
 3:  host-72-174-144-1.broadband.pl                        12.755ms 
 4:  ae21-0.icr01.waw04.atlas.cogentco.com                 14.903ms asymm  5 
 5:  be3035.ccr21.ham01.atlas.cogentco.com                 31.577ms asymm  6 
 6:  be3037.ccr21.ams03.atlas.cogentco.com                 34.455ms asymm  7 
 7:  no reply
 8:  no reply
 9:  example.com                                            87.890ms reached
     Resume: pmtu 1500 hops 9 back 58
```

Alternatively:

```bash
traceroute --mtu example.com
```

## Summary

Traceroute is a powerful diagnostic tool that allows us to see how data packets are transmitted between networks. Analysis of traceroute code can show how elegantly it uses basic internet protocols (IP, UDP, ICMP) to map complex network routes.