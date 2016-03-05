# MQ-ECN-Experiments
## Software Requirements
To reproduce Figure 7 and 8 of [MQ-ECN paper](http://www.cse.ust.hk/~kaichen/papers/mqecn-nsdi16.pdf), you need following software:
  - [MQ-ECN software prototype](https://github.com/HKUST-SING/MQ-ECN-Software)
  - [A simple traffic generator](https://github.com/HKUST-SING/TrafficGenerator)
  
In addition, you also need to install Linux 3.18x kernel which supports DCTCP. We used 3.18.11 kernel in our experiments. We also provide a [patch](https://github.com/baiwei0427/Latency-Measurement/blob/master/kernel_measurement3.patch) for Linux 3.18.11 kernel which allows users to adjust TCP RTOmin using sysctl.  

## Testbed Setup
In the testbed, 9 servers are connected to a 9-port server-emulated switch. One server acts as the receiver while the rest 8 servers act as the sender. The IP address of the receiver is 192.168.101.1(/24). The IP addresses of senders are 192.168.102.1(/24), 192.168.103.1(/24), ... 192.168.109.1(/24). The server-emulated switch has 9 NICs whose IP addresses are 192.168.101.2(/24), 192.168.102.2(/24), ... 192.168.109.2(/24). To avoid large segments, please disable offloading techniques (e.g., GSO, TSO and GRO) on the server-emulated switch.   

To enable packet forwarding on the server-emulated switch:
```
$ sysctl -w net.ipv4.ip_forward=1
```
To ensure servers on different subnets (e.g., 192.168.101.0/24 and 192.168.102.0/24) can access each other, you need to add some routing entries on servers. For example, at the receiver (192.168.101.1), I add a new routing entry as follows:
```
$ route add -net 192.168.0.0/16 gw 192.168.101.2
```
To enable DCTCP on servers:
```
$ sysctl -w net.ipv4.tcp_ecn=1
$ sysctl -w net.ipv4.tcp_congestion_control=dctcp
```
If you have applied our [patch](https://github.com/baiwei0427/Latency-Measurement/blob/master/kernel_measurement3.patch), you can adjust TCP RTOmin (in milliseconds) as follows:
```
$ sysctl -w net.ipv4.tcp_rto_min=10
```

## Installation
#### [MQ-ECN Software](https://github.com/HKUST-SING/MQ-ECN-Software) 
To install MQ-ECN qdisc kernel module, please follow the [guidance](https://github.com/HKUST-SING/MQ-ECN-Software). By default:
  - It performs Deficit Weighted Round Robin (DWRR). All the queues have the same quantum (1.5KB). 
  - All the queues belonging to a switch port share the per-port buffer space in a first-in-first-serve bias.
  - Packets with DSCP value *i* are classified to queue *i* of the scheduler.
  
To set per-port buffer size to 96KB:
```
$ sysctl -w dwrr.shared_buffer_bytes=96000
```
To enable per-queue ECN/RED with the standard threshold (32KB):
```
$ sysctl -w dwrr.ecn_scheme=1
$ sysctl -w dwrr.queue_thresh_bytes_0=32000
$ sysctl -w dwrr.queue_thresh_bytes_1=32000
$ sysctl -w dwrr.queue_thresh_bytes_2=32000
$ sysctl -w dwrr.queue_thresh_bytes_3=32000
```
To enable per-queue ECN/RED with the minimum threshold (8KB):
```
$ sysctl -w dwrr.ecn_scheme=1
$ sysctl -w dwrr.queue_thresh_bytes_0=8000
$ sysctl -w dwrr.queue_thresh_bytes_1=8000
$ sysctl -w dwrr.queue_thresh_bytes_2=8000
$ sysctl -w dwrr.queue_thresh_bytes_3=8000
```
To enable MQ-ECN:
```
$ sysctl -w dwrr.ecn_scheme=3
```
For more parameter settings, please see [params.h](https://github.com/HKUST-SING/MQ-ECN-Software/blob/master/sch_dwrr/params.h).

#### [Traffic Generator](https://github.com/HKUST-SING/TrafficGenerator)
To install the traffic generator, please follow the [guidance](https://github.com/HKUST-SING/TrafficGenerator). After installation, you should start *server* on senders (192.168.102.1 to 192.168.109.1). 
