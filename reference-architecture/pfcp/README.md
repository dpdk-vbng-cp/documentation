# Packet Forwarding Control Protocol (PFCP)

3GPP protocol that describes the interface between CP and UP for LTE gateways.

PFCP is defined in:
https://www.3gpp.org/ftp/Specs/archive/29_series/29.244/29244-f50.zip

The following drawings are stolen from:
https://tools.ietf.org/html/draft-wadhwa-rtgwg-bng-cups-03

## Architecture
### CUPS BNG System

```
                             |
 "CUPS BNG"                  |
+----------------------------+------------------------------------+
|   CP                                                            |
|   +--------------------------+--------------------------------+ |
|   | +-----------+ +------------------+ +--------+ +---------+ | |
|   | | Address   | | PPPoE, DHCPv4/v6 | | RADIUS | | S11/N11 | | |
|   | | Pool Mmmt | | IPv6 RS/RA,      | | CLIENT | +---------+ | |
|   | +-----------+ | L2TP LAC         | +--------+             | |
|   |               +------------------+  +----+  +----+        | |
|   |                                     | Gx |  | Gy |        | |
|   |                                     +----+  +----+        | |
|   +-----------------------------------------------------------+ |
|          |                  |                   |               |
|          | Management       |In-band            | State         |
|          | Interface        |Signaling          | Control       |
|          |                  |Channel            | Interface     |
|          |                  |                   |               |
|  --------+--+------------------+--+----------------+---+------- |
|          |                        |                    |        |
|    UP    |               UP       |            UP      |        |
|    +-----+---------+    +---------+-----+    +---------+-----+  |
|    | Local CP      |    | Local CP      |    | Local CP      |  |
|    | Routing, MPLS |    | Routing, MPLS |    | Routing, MPLS |  |
|    | IGMP, BFD     |    | IGMP, BFD     |    | IGMP, BFD     |  |
|    +---------------+    +---------------+    +---------------+  |
|    | Forwarding    |    | Forwarding    |    | Forwarding    |  |
|    | Traffic Mgmt  |    | Traffic Mgmt  |    | Traffic Mgmt  |  |
|    +---------------+    +---------------+    +---------------+  |
|                                                                 |
+-----------------------------------------------------------------+
```

### CUPS BNG with Converged Access

```
CUPS-BNG
    +-------------------+  S11    +----------+
    |     +------+      |(GTP-c)  | +------+ |
    |     |  CP  |------|---------|-|  MME | |
    |     +---+--+      |         | +------+ |
    |         |         |         | +------+ |
    |         |         |       +-|-| RAN  | |--+     +-----+
    |   ------+-----    |  S1-u | | +------+ |   \    |     |
    |     |       |     |(GTP-u)| +----------+    +---|     |
    |     |       |     | (DHCP |                     | CPE |
    |     |       |     | SLAAC)|                     |     |
    |   +----+  +----+  | (Data)|                 +---|     |
    |   | UP |  | UP |--|-------+     +-----+    /    +-----+
    |   |    |  |    |--|-------------| AN  |---+
    |   +----+  +----+  |    Eth      +-----+
    |                   | (DHCP,PPPoE)
    +-------------------+   (Data)
```

## State Control Protocols
### Session level state management

https://tools.ietf.org/html/draft-wadhwa-rtgwg-bng-cups-03#section-3.2.1

#### Session Creation Sequence

```
+---+          +---+                    +---+          | AAA  |
|CPE|          |UP |                    |CP |          |Server|
+---+          +---+                    +---+          +------+
  |DHCP Discover |                         |               |
  |------------->|                         |               |
  |              |                         |               |
  |              |     DHCP Discover       |               |
  |              |-----------------------> |               |
  |              |In-band signaling channel|               |
  |              |                         |               |
  |              |                         |Access Request |
  |              |                         |-------------->|
  |              |                         |               |
  |              |                         |Access Accept  |
  |              |                         |<--------------|
  |              |                         |               |
  |              |      DHCP Offer         |               |
  |              |<------------------------|               |
  |              |In-band signaling channel|               |
  | DHCP Offer   |                         |               |
  |<-------------|                         |               |
  |              |                         |               |
  |DHCP Request  |                         |               |
  |------------->|                         |               |
  |              |                         |               |
  |              |     DHCP Request        |               |
  |              |-----------------------> |               |
  |              | In-band signaling channel
  |              |                         |               |
  |              | Session Creation Req    |               |
  |              |<----------------------- |               |
  |              |                         |               |
  |              | Session Creation Resp.  |               |
  |              |-----------------------> |               |
  |              |                         |               |
  |              |                         |               |
  |              |      DHCP ACK           |               |
  |              |<----------------------- |               |
  |              |In-band signaling channel|               |
  |  DHCP Ack    |                         |               |
  |<-------------|                         |               |
+---+          +---+                    +---+          +------+
|CPE|          |UP |                    |CP |          | AAA  |
+---+          +---+                    +---+          |Server|
                                                       +------+
```

#### Session Modification

```
                                        +------+
+---+                    +---+          | AAA  |
|UP |                    |CP |          |Server|
+---+                    +---+          +--+---+
  |                        | CoA Request   |
  |                        |<--------------|
  |                        |               |
  | Session Modify Req     |               |
  |<-----------------------|               |
  |                        |               |
  | Session Modify Resp    |               |
  |----------------------->|               |
  |                        |               |
  |                        |   CoA Ack     |
  |                        |-------------->|
+---+                    +---+          +--+---+
|UP |                    |CP |          | AAA  |
+---+                    +---+          |Server|
                                        +------+
```

#### Session Deletion

```
+---+             +---+             +---+       +-------+
|CPE|             |UP |             |CP |       | AAA   |
+---+             +---+             +---+       |Server |
  | DHCP Release    | DHCP Release    |         +-------+
  |---------------->|---------------->|               |
  |                 |                 |               |
  |                 | Session Del Req |               |
  |                 |<----------------|               |
  |                 |                 |               |
  |                 | Session Del Resp|               |
  |                 | (final usage)   | Acct Stop     |
  |                 |---------------->|(final usage)  |
  |                 |                 |-------------->|
  |                 |                 } Acct Stop Resp|
  |                 |                 |<--------------|
+---+              +---+            +----+         +-------+
|CPE|              |UP |            | CP |         | AAA   |
+---+              +---+            +----+         |Server |
                                                   +-------+
```

### Session level event notifications

https://tools.ietf.org/html/draft-wadhwa-rtgwg-bng-cups-03#section-3.2.2

#### Async Event Notification for periodic usage

```
                                         +------+
+---+                    +---+           | AAA  |
|UP |                    |CP |           |Server|
+---+                    +---+           +------+
  |                        |                |
  |                        |                |
  |Async Event Notification|                |
  | (periodic usage)       |                |
  |----------------------->|                |
  |                        |                |
  |Async Event Response    |                |
  |<-----------------------|Acct Update Req |
  |                        |--------------->|
  |                        |Acct Update Resp|
  |                        |<---------------|
+---+                    +---+           +------+
|UP |                    |CP |           | AAA  |
+---+                    +---+           |Server|
                                         +------+
```

### Node level management

https://tools.ietf.org/html/draft-wadhwa-rtgwg-bng-cups-03#section-3.2.3

#### Node Association Setup and Maintenance

```
+---+                    +---+
|UP |                    |CP |
+---+                    +---+
  |                        |
  |                        |
  | Association Setup Req  |
  |----------------------->|
  |                        |
  | Association Setup Resp |
  |<-----------------------|
  |                        |
  | Periodic Heartbeats    |
  |<---------------------->|
  |                        |
+---+                    +---+
|UP |                    |CP |
+---+                    +---+
```
