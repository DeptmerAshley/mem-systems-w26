+++
title = "TPP: Transparent Page Placement for CXL-Enabled Tiered Memory"
[extra]
[[extra.authors]]
name = "Deptmer Ashley (Blogger)"
[[extra.authors]]
name = "Jared Ho (Leader/Presenter)" 
[[extra.authors]]
name = "Brian Castellon Rosales (Leader/Presenter)"
+++

## Purpose

This paper addresses a growing problem in hyperscale datacenters: **memory is becoming one of the largest contributors to total cost and power consumption**. As memory demands increase and DRAM scaling slows, simply adding more DRAM is no longer cost-effective.

With the emergence of **CXL (Compute Express Link)**, servers can now attach additional memory tiers with different performance and cost characteristics. However, using slower memory tiers can significantly lower application performance.

The primary goals of the paper are:

- Characterize real production workload memory behavior  
- Evaluate opportunities for tiered-memory systems  
- Design an OS-level, application-transparent page placement system  
- Demonstrate production deployment feasibility  

Key insight:

> A tiered memory system can match near-DRAM performance but only if page placement is handled intelligently.

---

## What CXL-Enabled Tiered Memory Is

CXL allows memory to be **decoupled from the CPU**. Instead of memory being tightly attached to a processor socket, CXL enables additional memory devices to appear as CPU-less NUMA nodes.

This creates a system with:

- **Local memory (fast tier)** — directly attached DRAM  
- **CXL memory (slow tier)** — higher latency, potentially cheaper memory  
- Byte-addressable load/store semantics  
- Cache-line granularity access  
- NUMA-like access behavior  

Importantly, CXL memory behaves like main memory, not like swap, which preserves load/store semantics and avoids expensive page faults for every access.

## Motivation

Datacenter workloads rarely use all allocated memory at full intensity. The authors observe:

- 55–80% of allocated memory may remain idle within short intervals  
- Anonymous (heap/stack) pages tend to be hotter  
- File-backed pages tend to be colder  
- Page access patterns are stable over minutes to hours  

This suggests a major opportunity: Cold pages can be moved to slower memory tiers without significantly hurting performance if done carefully.

However, default Linux memory management assumes homogeneous DRAM and performs poorly in heterogeneous tiered systems.

## Chameleon: Characterizing Real Workloads

Before designing a solution, the authors built **Chameleon**, a lightweight user-space profiling tool that uses hardware sampling (PEBS) to generate heatmaps of page access behavior.

Key findings from production workloads:

- Large portions of memory remain cold  
- Anonymous pages are generally hotter than file-backed pages  
- Page temperature changes over time  
- Some cold pages become hot again later  

These observations directly influenced the design of TPP.

---

## TPP: Transparent Page Placement

TPP is a kernel-level page placement mechanism designed for CXL-enabled tiered memory systems.

It has four core components:

### 1. Lightweight Page Demotion

Instead of swapping cold pages to disk, TPP **migrates** cold pages from local DRAM to CXL memory using NUMA migration.

This is much faster than swap and preserves in-memory access semantics.

Cold pages are selected using existing Linux LRU mechanisms.

### 2. Decoupled Allocation and Reclamation

Under memory pressure, Linux normally blocks allocations until reclamation frees memory. In tiered systems, this can push too many allocations into slow memory.

TPP separates allocation watermarks and demotion watermarks This maintains a headroom of free pages in fast memory, ensuring
new hot allocations land in DRAM and promotions of hot pages from CXL are possible.

### 3. Hot Page Promotion

Pages in CXL memory that become hot must be promoted back to DRAM.

TPP augments Linux NUMA balancing but introduces hysteresis:

- Only pages in active LRU lists are promoted  
- Infrequently accessed pages are filtered  
- Reduces ping-pong behavior between tiers  

This ensures only truly hot pages are migrated back.

### 4. Page Type-Aware Allocation

Applications can optionally prefer anonymous pages with faster memory, or file cache pages and slower memory. 
This improves convergence speed and reduces unnecessary migrations.

---

## Evaluation

The authors deploy TPP on real CXL-enabled hardware in production-like environments.

Key results:

- TPP performs within **<1% of an all-local-memory baseline**  
- Improves performance by up to **18% over default Linux**  
- Outperforms NUMA Balancing and AutoTiering by **5–17%**  
- Works even when fast DRAM is only 20% of total memory  

Importantly, TPP has already been largely merged into **Linux kernel v5.18**, demonstrating practical viability.

### Why Emulation Is Not Enough

A major takeaway parallels earlier persistent memory research:

Emulation does not accurately capture:

- Real NUMA latency effects  
- Migration overheads  
- Page temperature dynamics  
- Production workload behavior  

This paper demonstrates the importance of evaluating real hardware in real environments.

## Conclusion

This paper shows that tiered memory systems powered by CXL are not only feasible but practical for hyperscale datacenters. The key challenge is not the hardware itself, but intelligent software management.

TPP demonstrates that with lightweight demotion, careful promotion, and page-type awareness, a heterogeneous memory system can deliver performance nearly identical to an ideal DRAM-only system — while enabling significant cost and capacity advantages.

More broadly, the paper highlights a recurring systems lesson:

> Hardware innovation requires equally thoughtful operating system design.


## Class Discussion Notes Summary

During the class discussion, the focus shifted from the paper’s implementation details to the broader implications of CXL-enabled tiered memory systems and how they affect system architecture and memory management.

One of the primary themes discussed was the flexibility introduced by CXL’s CPU and memory bus architecture. Unlike traditional systems where memory capacity is constrained by the number of DDR channels attached 
to the CPU, CXL allows memory capacity to scale independently from processor sockets. This enables datacenters to attach large pools of additional memory without requiring additional CPUs, potentially improving resource 
utilization and lowering system cost. In practice, CXL memory appears within the same physical address space as local DRAM and remains byte-addressable, allowing applications to access it through normal load and store instructions. 
However, its performance characteristics resemble those of remote NUMA memory, with slightly higher latency and reduced efficiency when heavily utilized.

The discussion also highlighted the performance trade-offs introduced by large CXL memory deployments. While tiered memory systems significantly increase capacity, workloads can experience measurable slowdowns when a large fraction 
of memory accesses occur in the slower tier. Example workloads discussed included caching systems, web services, and data warehouse applications, which can see performance drops ranging from roughly 14% to over 20% when large amounts of 
memory are served by CXL rather than local DRAM. These results reinforce the importance of intelligent page placement and memory management strategies.

Much of the discussion focused on how Transparent Page Placement (TPP) attempts to mitigate these challenges. TPP introduces lightweight demotion of cold pages to slower memory tiers while enabling efficient promotion of hot pages back 
to faster DRAM. Unlike traditional Linux memory management, TPP maintains a separate demotion mechanism and decouples page allocation from reclamation. This allows the operating system to maintain a reserve of fast-memory pages for new allocations 
while still utilizing slower memory capacity effectively. The design goal is to reduce allocation latency while avoiding excessive page thrashing between memory tiers. Another topic discussed was the role of workload-aware memory policies. 
Because different applications exhibit different memory access patterns, effective tiered memory management must consider application behavior. For example, some workloads maintain stable hot sets of memory that benefit from remaining in fast DRAM, 
while other data structures can tolerate slower access without impacting overall performance. TPP attempts to account for these differences by adapting page promotion and demotion decisions based on observed access patterns.

Finally, the evaluation results highlighted in the discussion emphasized that TPP improves system responsiveness in memory-heavy environments. In particular, experiments demonstrated that optimized allocation paths could significantly improve page allocation rates under high memory pressure, especially when the majority of memory resides in the slower CXL tier. These results illustrate that effective operating system policies can make heterogeneous memory systems practical even when a large portion of total memory capacity is provided by slower devices.

---

### AI Disclosure

ChatGPT was used to help structure and summarize the paper. The content was reviewed and refined to ensure technical accuracy and clarity.