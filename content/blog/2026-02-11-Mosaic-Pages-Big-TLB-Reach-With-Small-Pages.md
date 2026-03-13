+++
title = "Mosaic Pages: Big TLB Reach with Small Pages"
[extra]
bio = """ """
[[extra.authors]]
name = "Donovan Burk (Leader Presentor)"
[[extra.authors]]
name = "Humoud Almutairi (Scribe)"
[[extra.authors]]
name = "Eric Morgan-Bronec (Blogger)"
+++

# Introduction
Virtual memory is a memory management technique that allows for the illusion of infinite memory from the programmer's perspective, regardless of the actual RAM size. Each physical address corresponds to a virtual address used by processes. Memory is divided into blocks called pages. The operating system translates virtual addresses to physical addresses using page tables, which map virtual page numbers to physical locations in memory. Page tables are divided into layers. Every time memory is accessed, the operating system needs to traverse the page table to decode the virtual address. This is called a page walk. To avoid page walks, we use a translation lookaside buffer or TLB. This is a cache of SRAM that lives on the CPU and stores the physical page number, thus eliminating the need for a page walk. The TLB is critical for ensuring high performance in a system.
Memory capacity has been growing exponentially, but the TLB range has stayed stagnant. This means a higher likelihood for TLB misses, which slows down memory access and makes performance worse. One solution is to increase the size of a page. This means that the same number of pages now spans more memory. The downside to this is memory fragmentation. An application might not fully utilize an entire page, and the unused memory that has been allocated is wasted. This paper proposes Mosaic pages as a method of expanding the TLB’s reach.

# Mosic Pages
Mosaic pages use hashing algorithms to compress addresses so that multiple virtually contiguous addresses fit into the same TLB entry. Physical memory is organized as buckets of size h in a hash table. A hash function maps a virtual address to a bucket in our memory hash table. The TLB entry stores a = 4 Compressed Physical Frame Numbers (CPFN) that are virtually but not physically contiguous. CPFNs act as the bucket offset and range from 0 to h. Only log(h) bits are needed to store this information, thus compressing our data. When an application needs to access a page, it decodes the CPFN to get the bucket offset and uses that in combination with the bucket id to find the location of the page in physical memory. This allows for more compact storage of pages in the TLB.

### Horizon LRU
Horizon LRU is the eviction algorithm proposed by the Mosaic paper. It functions similarly to regular LRU, but is modified to work with Mosaics bucket structure. It uses per-page access timestamps and a global horizon timestamp; pages accessed before the horizon are considered effectively evicted. By advancing the horizon, Horizon LRU mimics the behavior of a true global LRU policy while preserving memory efficiency and respecting Mosaic Pages’ local allocation structure.

# Results
Mosaic was run on a Graph500, BTree, GUPS, and XSBench workloads. In the tests, it showed a 6 – 84% reduction in TLB misses. In the first runs, Mosaic had less RAM utilization due to the Horizon LRUs' “ghost pages”, but in steady state operation, it had better RAM utilization overall. 
Mosaic didn’t encounter harmful conflicts until the memory utilization reached 98%, which the authors note that a conventional system would also start to break down at the capacity. This indicates that restricting address mappings via hashing does not meaningfully reduce usable memory or performance in typical operating regions, a crucial insight that validates the practicality of the placement constraints.
When the workload is just slightly over the size of available memory, Mosaic performs more swapping than the default Linux allocator. This is because Linux can utilize about 1% more memory than Mosaic. However, once the workload footprint passes this edge case, Mosaic matches or outperforms Linux by up to 29% better. In general, Mosaic uses less swapping across all workloads, reducing memory access latency.

# Class Discussion

#### Ghost Pages
They reduce unnecessary evictions, but they also affect the measured utilization and page-swap behavior (especially near the memory limit). 

#### Near-threshold behavior
When workloads are just above available RAM, small differences in overflow matter a lot, and page swap results can change noticeably. 

#### Front yard/backyard allocation 
The design relies on front yard-first placement and backyard fallback to find emptier buckets, plus eviction if there’s no free space. 

# Conclusion
Mosaic TLB shows promising results for improving on a core performance bottleneck, the TLB. The authors validate their design with thorough benchmarks that highlight the potential of this technology. However, this is still a long way away from being implemented on an actual product. Mosaic TLB would require changes to the operating system’s memory allocator. The paper does not touch on more complex memory access, such as NUMA or shared memory regions. These could cause complications to the virtual address encoding. Overall, the paper demonstrated a 

# Refrences
Gosakan, Krishnan, Han, Jaehyun, Kuszmaul, William, Mubarek, Ibrahim, Mukherjee, Nirjhar et al. 2023. "Mosaic pages: Big tlb reach with small pages". http://www.cs.yale.edu/homes/abhishek/ksriram-asplos23.pdf 
