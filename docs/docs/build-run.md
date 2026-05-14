---
title: Build & Run
---

### Build

```shell
export CGO_ENABLED=0
go build
```

### Run

```shell
export OPENGFW_LOG_LEVEL=debug
./OpenGFW -c config.yaml rules.yaml
```

Where `config.yaml` is the config file and `rules.yaml` is the rules file.

#### pcap file replay mode

```shell
./OpenGFW -p your.pcap -c config.yaml rules.yaml
```

In pcap mode, none of the actions in the rules have any effect. This mode is mainly for debugging.

#### OpenWrt

OpenGFW has been tested to work on OpenWrt 23.05 (other versions should also work, just not verified).

Install the dependencies:

```shell
opkg install nftables kmod-nft-queue kmod-nf-conntrack-netlink
```

### Config example

```yaml
io:
  queueSize: 1024
  queueNum: 100 # (6)!
  table: opengfw # (7)!
  connMarkAccept: 1001 # (8)!
  connMarkDrop: 1002 # (9)!
  rcvBuf: 4194304
  sndBuf: 4194304
  local: true # (1)!
  input: true
  output: true
  forward: true
  docker: true
  rst: false # (2)!

workers:
  count: 4 # (3)!
  queueSize: 64
  tcpMaxBufferedPagesTotal: 65536
  tcpMaxBufferedPagesPerConn: 16
  tcpTimeout: 10m # (4)!
  udpMaxStreams: 4096

# The path to load specific local geoip/geosite db files.
# If not set, they will be automatically downloaded from https://github.com/rootmelo92118/v2ray-rules-dat
# ruleset:
#   geoip: geoip.dat
#   geosite: geosite.dat

replay:
  realtime: false # (5)!
```

1. Set to false if you want to run OpenGFW on FORWARD chain (e.g. on a router)
2. Set to true if you want to send RST for blocked TCP connections, **local=false only**
3. Input, output, forward, and docker can be controlled separately. The local option only takes effect when none of the four options is enabled.
4. Recommended to be no more than the number of CPU cores
5. How long a connection is considered dead when no data is being transferred. Dead connections are purged from TCP reassembly pools once per minute.
6. Set to true if you want to replay the packets in the pcap file in "real time" (instead of as fast as possible)
7. nfqueue queue number
8. nftables table name
9. connmark value for accepted connections
10. connmark value for dropped connections
