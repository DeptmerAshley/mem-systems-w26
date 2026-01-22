+++
title = "(MC)^2: Lazy MemCopy at the Memory Controller"
[extra]
bio = """ """
[[extra.authors]]
name = " Paul Suvrojyoti (Leader / Presentor / blogger)"
[[extra.authors]]
name = "Kabir Vidyarthi(Presentor)"
[[extra.authors]]
name = "Derek Werbowy (Presentor)"
[[extra.authors]]
name = "Carlos Alvarado-Lopez (scribe)"
[[extra.authors]]
name = "William Davis(blogger)"
+++

# Introduction
Modern software systems rely heavily on the process of memory copying to provide isolation, simplify synchronization, and support common operations such as serialization, I/O buffering, and snapshot creation. Although the system of memcpy appears simple, it holds many operations that impose a significant performance cost; A large portion of the CPU cycles are spent stalled on cache misses and DRAM accesses, in many cases only a small portion of the copied data is ever actually used. As the processor speeds continued to outpace the improvements in memory latency, the inefficiency of eager, byte-by-byte copying becomes a major bottleneck to the entire system. 

The (MC)² Lazy Memcopy style architecture addresses some of these problems. This is done by rethinking where and when the data movement occurs. Instead of immediately copying data at the CPU, (MC)² shifts the copy management into the memory controller and delays the actual data transfer until it is actually needed. By tracking copy intent and resolving it only on demand, (MC)² aims to eliminate redundant memory traffic, reduce cache pollution, and substantially lower the stall time that is associated with the traditional memory copy operation. 

# Overview
(MC)² reduces used to reduced the overall high cost of memory copying by moving the copy management into the memory controller. This is done by making lazy copies. Instead of constantly copying data with memcpy, the system tracks copy intentions using a Copy Tracking table (CTT) and delays the actual movement of bytes, until the data is actually needed for an operation. By only making copies when the destination is read or the source is overwritten. With (MC)² it avoids unnecessary memory traffic and CPU stalls, which are a major bottleneck in modern systems due to cache misses and the long DRAM latencies. 

The use of a memory controller is extended with the hardware structures, including the CTT and a bounce pending queue (BPQ), to transparently intercept copy requests and route memory accesses to the correct location both physically and digitally. This allows for the destination reads to be serviced directly from the source buffer and source writes to trigger on-demand copying. All of this happens while preserving memory consistency and cache coherence. By operating below the cache hierarchy and working on the physical addresses, (MC)² provides fine-grained, cacheline-level copying virtualization, without requiring the operating system or application level changes. 

(MC)² further improves performance by handling chains of copies, merging adjacent regions, and performing background copy completion when tracking resources become saturated. Evaluations that were done across microbenchmarks and real applications such as Protobuf, MongoDB, MVCC databases, and fork-based snapshots showed significant reductions in copy-induced stalls and memory bandwidth consumption, yielding substantial speedups small and partially used buffers that dominate real world workloads. 

# Hardware Function and changes

CPU pushes the copy management into the memory controller for MCLAZY.  So a SRAM based CTT (copy tracking table) and a small queue called BPQ (bounce pending queue) were added.
CTT functions:
* merge adjacent tracked regions
* avoid chains like A ->B and B->C (rewrite to A->C)
* prevent two rules from overlapping the same dest region
* and free CTT space by completing copies in the background 
BPQ function:
It just hold the src writes operation until the actual copy to the dest finishes before the writing of the src can proceed.
Steps of the hardware:
* CPU sends copy instruction
* Cache does src write back so ram has latest src snapshot and invalidates dst so that in later reading dest is not found in cache and it is forced to go to CTT
* MC records the mapping in the CTT
Later access:
  *	Src read - proceed normally
  *	Dest write – proceed normally as it is basically updating the dest  and the mapping in CTT is not even required
  *	Read from src- not the actual lazy copy needs to go through consulting the CTT. Once done removing the CTT mapping
  *	Write to src – go to Bpq and let lazy copy of the dest finish first, then proceed.


# Software  Function and Changes:
(MC)² made some changes to the software side too. It introduced a clean interface ( a wrapper) for lazy copying  “memcpy(dest, src, size)”. Programs use it arbitrary sizes and alignments. The paper states the hardware mechanism is most efficient when it can track is 64 bytes chunks or cacheline sized. The software therefore provides the wrapper that preservers norma copy  but internally chooses between 

* normal copy for small or awkward pieces
*	Lazy copying fir aligned bulk
Steps for the new instruction :
*	Do normal copy if too small (smaller than 64 bytes)
*	If dest is not cacheline aligned, compute how many bytes required to make dest aligned , normal copy exactly those bytes, advance src and dest and reduce size accordingly. The remaining region can be handled in cacheline chunks

*	Do the MCLAZY of the bulk region, these are multiples of cacheline sizes and less than a page
*	If something is left , do normal copy.
*	Finally do a fence such that the lazy copy maintains proper order of  memory operations

Example , suppose the program calls memcpy_lazy(dest=…03, src=…00, size=200) and the cacheline is 64 B.

The wrapper will:

*	Copy  61 bytes normally (to align dest)
*	Use MCLAZY for the next 128 bytes 
*	Copy the last 11 bytes normally
*	Fence
There is one last function introduced called MCFREE(buffer, size) which can be sometimes used by software to tell the hardware to drop mappings of dest->src.

# Results:
In performance evaluations, (MC)² lazy memcpy outperforms many of the existing systems. The only other system that got close to it in terms of latency is their zIO. For uncached source buffers, the lazy copy system achieves up to 11 times lower latency than the conventional memcpy for medium and large copy sizes over 1 KB, where DRAM is access is the dominate use of time for execution. When the source is already in the cache, the traditional memcpy can be slightly faster for smaller file sizes due to the addition of the memory controller logic in the (MC)² lazy memcpy system. For a different type of access pattern, where the it reads copied data 

# Key Results:
Key results:
*	11x lower latency for medium and large copy sizes (≥ 1 KB)
*	43% speedup for Protobuf serialization
*	Mongo DB I/O stack: ~15.5% throughput improvement 
*	MVCC databases: up to 78% speedup for read-modify-write workloads on small files
*	Pipes and streaming I/O: 15 – 30% higher throughput by eliminating redundant kernel buffer copies
*	CTT reaches 50% occupancy prevents bandwidth saturation while avoiding CPU stalls.

# Class Discussion

The system presents a valuable solution but at what cost?

*	The system has many advantages but how much does it cost to manufacture and what are the downsides to this system

Why hasn’t this been adapted yet?

*	The lazy copy system is relatively new to the discussion so it might be used in future CPU’s and memory control systems that have yet to be released as this paper came out in 2024

Is this a hardware vulnerability?

*	As of right now there is nothing to suggest that it is but when future research comes out it might prove that it is a vulnerability. As it could possibly have problems that aren’t discussed in the paper that could cause memory loss or memory overload

What is the actual cost?

*	This is a complete unknown it could be just a slight raise in price or a drastic increase in price due to a new way to make the memory systems requiring new processes to manufacture.  

# Conclusion 
In summary, (MC)² lazy memcpy shows that a large portion of the time the system uses is wasted waiting for copying data that is never fully used, with CPU’s stalled waiting on memory rather than doing other useful work. By moving this into the memory controller and making those lazy copies, the system then only has to track copy intent and only moves the data when the data is needed. Hardware support through the copy tracking table and bounce pending Queue enables this to happen in a transparent faction.

# References:
*	Paper: https://dl.acm.org/doi/10.1109/ISCA59077.2024.00084 
*	Slides: https://docs.google.com/presentation/d/1LHGMNmEvYYec-WrcbDhAI9wVA5Bh5tclDccBAJohub0/edit?usp=sharing 


# Generative AI Disclosure
* ChatCPT was used just to pull notes into 1 file and check grammer

* Generative AI can be useful tools for tasks such as summarizing or drafting, however, they may give innacurate information confidently and should always have generated information validated
