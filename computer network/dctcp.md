# DCTCP
首先这篇文章是一篇解决工业实际问题的文章，同一般的文章不同，为了表述清楚问题和解决的思路，作者们零零碎碎的从各个方面阐述了很多点。  
其次，和一般从算法或者并行的实现解决吞吐量或者抖动的文章不同，这里同硬件特征和架构直接相关。而且由于数据中心的数据包特征特殊性，针对性的修改协议，而不是一个一般化的内容。

背景主要的点：Data Centre，architecture，soft real-time，partition/aggregation
首先，DCTCP是针对Data Centre这一特殊的网络环境修改了TCP协议。  
那么具体什么是Data Centre？同一般的wan有什么区别？  
其次，DCTCP是对于新的data centre硬件和架构设计的。这样的架构是什么？tcp队列和包有什么不一样的特征？
然后，由于有软实时的需求，和一般的划分/合并架构，tcp不同类型的包对于吞吐量和延迟有着不同的需要。
最后才去理解为什么会去这样分析TCP包的模式，以及对于延迟，抖动和吞吐量的要求和tradeoff。

分析的主体是性能损失(?)，一方面是架构带来的incast，另一个问题来源于队列和buffer

具体的算法也非常简单，当队列大小超过一个阈值的时候，发出一个带有ECN的ACK，发射端接收到这个flag后，用一个简单的递推式调整窗口大小。
Concentrate on datacenter.

First, this TCP modification is based on a trend: **Low-cost switches are common at the top of the rack, providing up to 48 ports at 1Gbps. And several recent research proposals envision creating economical, easy to manage data centers using novel architectures built atop these commodity switches.**

And this paper is about soft real-time applications, which have driven much of the data center construction and requires: **low latency for short flows, high burst tolenrance and high utilization for long flows.**

Partition/Aggregate workflow pattern, the soft real-time deadlines for end results translate into latency targets for the individual tasks in the workflow, and tasks not completed before their deadline are cancelled affecting the final result.  
**Application requirements for low latency directly impact the quality of the result returned and thus revenue. Reducing network latency allows appliction developers to shift more cycles to the algorithms that improve relevance and end user experience.**

And the last requirement, high utilization for large flows, stems from the need to continuously update internal data structures of these applications, as the freshness of this data also affects the quality of results. High throughput for long flows.

Result of measurement: the data center's traffic consists of query traffic(2 KB to 20 KB), delay sensitive short msgs(100 KB to 1 MB) and throughput sensitive long flows(1 MB to 100 MB).
- query traffic experiences the incast impairment in the context of storage networks.
- query and delay sensitive short messages experience long latencies due to long flows consuming some or all of the available buffer in the switches.  
**switch buffer occupancies need to be persistently low, while maintaining high throughput for the long flows**

Difference dc with wan: 
- empty queue RTTs under 250 us consistently, 
- app have simutaneous needs for extremely high bandwidths and very low latencies, little statistical multiplexing: a single flow can dominate a particular path
- dc environment is largely homogeneous and under a single admin control

## Partition/Aggregate
Requests from higher layers of the application are **broken** into pieces and farmed out to workers in lower layers. The responses of these workers are **aggregated** to produce a result.  
For interactive, soft-real-time applications like these, latency is the key metric.  
Many app have a multi-layer partition/aggregate pattern workflow, with lags at one layer delaying the initiation of others.  
**Lagging instances of partition/aggregate can thus add up to threaten all-up SLAs for queries**.  
- Worker nodes are typically assigned tight deadlines, usually on the order of 10-100ms. **When a node misses its deadline, the computation continues without that response, lowering the quality of the result**
- High percentiles for worker latencies matter.

## Workload
- soft real-time query traffic, integrated with 
- urgent short message traffic that coordinates the activities in the cluster and 
- continuous background traffic that ingests and 
- organizes the massive data needed to sustain the quality of the query responses.
---
### Query Traffic
Follows partition/aggregate pattern, consists of very short, latency-critical flows, following a relatively simple pattern, with a high-level aggregator partitioning queries to a large number of mid-level aggregators.

### Background Traffic
Complex mix of background traffic, consisting of both large and small flows. Most of the bytes in background traffic are part of large flows.
- 5 KB to 50 MB, update flows that copy fresh data to the workers and time sensitive short message flows
- 50 KB to 1 MB that update control state on the workers

The interarrival time between background flows reflects the superposition and diversity of the many different services supporting the app:
- the interarrival time is very high, with a very heavy tail
- embedded spikes occur
- relatively large numbers of outgong flows occur periodically

### Flow Concurrency and Size
throughput-sensitive large flows, delay sensitive short flows and bursty query traffic, co-exist in a data center network.

## Performance Inpairments
### Switches
Shared memory. Packets arriving on an interface are stored into a high speed multi-ported memory shared by all the interfaces. Memory dynamically allocated by MMU.
### Incast
If many flows **converge** on the same interface of a switch over a short period time, the packets may exhaust either the switch memory or the maximum permitted buffer for that interface, resulting in packet losses for some of the flows. Arises naturally from use of the partition/aggregate design pattern, **as the request for data synchronizes the workers' responses and creates incast at eht queue of the switch port connected to the aggregator.**
### Queue
Long-lived, greedy TCP flows will cause the length of the bottleneck queue to grow until packets are dropped, resulting in the familiar sawtooth pattern.
- Packet loss on the short flows can cause incast problems.
- queue build up impairment: short flows experience increased latency as they are in queue behind packets from the large flows.  
**the occupancy of the queue caused by other flows, the background traffic, with losses occurring when the long flows and short flows coincide.**  
**Since the latency is caused by queueing, the only solution is to reduce the size of the queues.**
### Buffer pressure
The short flows on one port is impacted by activity on any of the many other ports. The loss rate of short flows in this traffic pattern depends on the number of long flows traversing other ports, as their activity is coupled by the shared memory pool.  
The long greedy TCP flows build up queues on their in interfaces.

## The DCTCP algorithm
Goal: achieve high burst tolerance, low latency and high throughput with commodity shallow buffered switches.

Achieve by reacting to congestion in proportion to the extent of congestion. DCTCP sets the **congestion experienced** codepoint of packets as soon as the buffer occupancy exceeds a fixed small threshold. The DCTCP source reacts by reducing the window by a factor that depends on the fraction of marked packets: the larger the fraction, the bigger the decrease factor.

Deriving multi-bit feedback from the information present in the single-bit sequence of marks.

### Simple marking at the switch
A single parameter, the marking threshold K. An arriving packet is marked with the CE codepoint if the queue occupancy is greater than K upon its arrival, to **achieve minimize queue buildup.**
### ECN_Echo at the Receiver
The only difference between a DCTCP receiver and a TCP is the way information in the CE codepoints is conveyed back to the sender. A DCTCP receiver tries to accuratedly convey the exact sequence of marked packets back to the sender, ack every packet, setting the ECN-Echo flag if and only if the packet has a marked CE codepoint.  
The DCTCP uses the trivial two state machine(not really understand the state transfer), whether the last received packet was marked with the CE codepoint or not.
### Controller at the Sender
The sender maitains a running estimate of the fraction of packets that are marked, which is a simple iteractive function.