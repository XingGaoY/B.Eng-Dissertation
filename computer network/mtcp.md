# mtcp note
http://shader.kaist.edu/mtcp/

直接和linux比较性能太耍流氓了，代码总共10k行，还没lwip多，完备性就不如linux/net，如果实现的功能相近，性能差距可能远远没有这么大。

还有，由于是在连接层之上就要把包交给app，NIC没有分包的能力，一个NIC只能对应一个app。
这样一来，似乎port，socket中很多用来multiplex的结构就不需要了，仅仅保留四元组就足够。  
这个其实不好吧。

最多的改动一个是RSS，另一个应该就是per-cpu queue。

## Problems
- Short TCP dominate the number of TCP flows. 90% TCP flows in a large cellular network are smaller than 32KB and more than half are less than 4KB.
- Attribute the inefficiency to:
  - the high system call overhead of the os
    - drastically changes the I/O abstraction to amortize the cost of syscalls.
    - requires significant modifications within the kernel and forces existing applications to be re-written
    > maybe not, same spec sometimes leads to minor code change

  - inefficient implementations that cause resource contention on multicore systems
    - makes incremental changes in existing implementations, falls short in fully addressing the inefficiencies

### Limitations of Kernel's TCP Stack

#### Lack of connection locality
Lock contension, cache miss and cache-line sharing

Threads in multi-thread applications contend for a same port and socket's accept queue.  
Also, the core executes the kernel code for handling a TCP connection may be different from the one rhat runs the app code that actually sends and receives data.

Local accept queue in each CPU core, ensuring flow-level core affinity across the kernel and app thread(?).

#### Shared file descriptor space
Lock contensions on socket fd.  
Extra overhead of going through the VFS.
> User level will eliminate this layer naturally...

#### Insufficient per-packet processing
- Per-packet mem (de)alloc and DMA overhead
- NUMA unaware memory access
- heavy data structure(`sk_buff`)

Batch process multiple packets

#### Syscall overhead
Frequent user/kernel mode switching when there are many short-lived concurrent connections

## Introduction
User-level TCP stack from the ground up by leveraging high-performance packet I/O libraries that allow applications to directly access the packets.
- eliminate the expensive syscall overhead by translating syscalls into IPC.
  - however processing IPC msg, involve context-switches that are much more expensive than the syscall.
  - amortize the context-switch overhead over a batch of packet-level and socket-level events.
  - author believe integrating pkt- and socket-level batching can lead to significant perf inprovement.
  - **per-core listen sockets, RSS**
- user level implementation ensures ease of porting without requiring significant modifications to the kernel, providing BSD-like socket and epoll-like event driven interfaces.  
  Migrating is easy as well.

- **netmap and DPDK**
- batch processing
- preserve interface

## Design
- mtcp runs as a thread on each CPU core within the same application process, xmit and recv pkt to and from NIC using custom pkt I/O lib.
- PSIO support an efficient event-driven packet I/O interface.
  - PSIO offers high-speed packet I/O by utilizing RSS that distributes incoming packets from multiple RX queues by their flows and provides flow level core affinity to minimize the contention among the CPU cores.  On top of PSIO's high-speed packet I/O, the new event-driven interface allows an mTCP thread to efficiently wait for events from TX and TX queues from multiple NIC ports at a time.
  - similar to select, mTCP specifies the interested NIC interfaces for RX or TX events, with a timeout in microseconds and returs immediately of any event of interest is available. If no such event detected, it **enables the interrupts for the RX/TX queues and yields the thread context.**
  - pkt xmit and recv in batches.
- As a lib runs as a part of applications main thread, provide better performance since translates costly system calls into light weight user-level function calls.
  - in mTCP, the app communicate with the mTCP thread via shared buffers.
  - access to shared buffer is granted by API, and calls could be batched together while the safety is preserved.

---
- TCP processing
  - mTCP thread reads a batch of pkts from NIC's RX queue and passes them to the TCP pkt processing logic with standard TCP specification
  - flush the queue after process the batch and wake up app
- lock-free, per-cpu data structure
  - localize all resouces, eliminate locks by lock-free data structure
  - distribute TCP connection workloads across available CPU cores with RSS
  - spawns one TCP thread for each application thread and co-locates them in the same physical CPU core.
- thread private data, share data with lock-free data structure
  - per core memory pool for alloc/free

---
- Batched Event Handling
  - After receiving pkt in batch, processes them to generate a batch of flow-level events

---
- Priority based pkt queue  
keep contol packets in a separate list, and process short connection first