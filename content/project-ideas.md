+++
title = "Project Ideas"
[extra]
hide = false
+++

# Project Ideas and Inspiration

I will first give some general approaches you can use to come up with ideas, then list a few specific ideas. 
While there are many ways to come up with promising research projects, there are two methods that tend to be
pretty reliable, though if you're not careful they can result in work that some might describe as "incremental." 

## Approach 1: Measure, Then Build
This approach, [made explicit by Remzi Arpaci-Dusseau][measure], involves taking some existing system, framework, or method
and building on it. For computer systems work, this usually starts with measurement. A common series of steps:
1. Try to get the code or method for the system/paper working
2. Try to reproduce the results in the authors' paper
3. Try to question the assumptions of the original paper. Maybe they missed some benchmarks. Maybe they cherry-picked benchmarks a bit, presenting the results of the system in a more positive light than it deserved, or maybe the types of workloads that we care about have changed, so we want to provide more representative inputs to the system. Papers from top researchers will tend to be honest about what the limitations of the present work are, and sometimes you'll get lucky and the paper has a "weaknesses" or "limitations" section that can give you pretty obvious starting points for how to improve it. 
4. Work to improve the system based on your findings in (3). This may sound easy, but there's often a good reason for limitations in the paper. It may be that the authors just didn't have enough time, or maybe the challenge they identified is a fundamental one that would require an entire PhD to solve. If this is your first time conducting research, probably best not to try to tackle the latter. If you're lucky again, the paper authors have already identified this challenge as a Hard one, so you can proceed with caution. 
5. Many times (4) will lead you on side paths that have their own merit. You may find these more interesting. Other times, "fixing" the system to solve whatever problem you've identified is sufficiently cumbersome such that starting with a clean slate would be better. This is good, because it means you're learning about designing systems with constraints. 

Of course, even getting to step 1 assumes that you've found some paper/system/tool to start with. That's of course why we're reading a bunch of papers in this course! You can start with any of the papers you find interesting or exciting. Or, maybe you have a paper or system you're already excited about. Start with that!

## Approach 2: Adaptation
Sometimes very good solutions have been found to certain systems challenges, but
the assumptions those solutions are based on have changed. So this approach basically
involves taking some established system, and looking to see if it still works or applies
to modern constraints or scenarios. Examples:
* Modern out-of-order cores are great, but do they work well for emerging AI workloads? Turns out not near as well
as a GPU, or an accelerator. 
* The OS page cache is a great way to minimize unnecessary disk I/O. But what happens when you run a data-intensive
database application on it? This great mechanism, and the OS itself, can get in the way. 
* Integer benchmarks make our CPU look really good, but what if customers are mostly interested in half-precision 
floating point math? 

This approach, a bit like the first, involves taking a close look at the assumptions of whatever system/paper
you're considering. These assumptions will often be explicitly stated in good papers. Sometimes the inverse
of this approach can be useful too: namely, applying an old idea to a new problem (especially in an unexpected way).

-----------

**Note on research novelty**: If you're not interested in publishing your research, you
can ignore this note. If you _are_, you must realize that _novelty_ is
important in getting your research published. Basically, this means that you're
really doing something no one else has done before. Say the system you're
building on is from 2014, and you've found some issues with it. You could spend
a quarter or an entire year solving those problems, submit your paper to a research conference,
and get reviews back saying "the paper lacks novelty because it turns out John
Doe published a paper that already solved those problems in 2020." What would
have helped mitigate this problem was doing a literature search. So really,
what you need to do is identify a problem, and first look for papers that may
have solved that problem. This is an acquired skill, and if you think you've
gotten to this point, come talk to me. There is _a lot_ of research out there,
so it's entirely possible that you miss a paper that solved your problem even
after you had done an exhaustive search. This happens to the best of us, but it can
be avoided if you know where and how to look. Even more infuriating is when
someone is working on the same problem in parallel and happens to publish their paper
a month before yours. Despite these risks, even if you get "scooped" it isn't like you've wasted your
time. Working on challenging problems helps you grow, and it can be fun! Especially
if you don't have any particular time constraints, it can still be exhilarating to 
work on something that you know someone else has already solved. Sometimes you'll come up
with a new and original way to solve the problem that's better than the original! In cases like this,
it's even helpful to ignore the original solution entirely then compare your approach after the fact. 
I like to call this the "working without the back of the book" approach. 

[measure]: https://www.usenix.org/conference/atc19/presentation/keynote

## Some Project Ideas
You can of course build on any paper we look at in class (or other papers!). Here are some rough ideas. These don't necessarily follow the two approaches above. If you'd like to know more about one of these, feel free to ask me!

- Take an existing published system (e.g., from the papers we cover) and validate their experimental results/findings. 
- Work with my PhD student on energy-efficient, hybrid architectures. Namely, trying to understand good power control techniques for memory-intensive applications.
- See if you can apply the idea of [time travel](https://www.usenix.org/conference/2005-usenix-annual-technical-conference/debugging-operating-systems-time-traveling) or [versioning](https://en.wikipedia.org/wiki/Versioning_file_system) to small objects. Maybe also explore how to expose this feature in the programming language. 
- Investigate the use of [page deduplication](https://en.wikipedia.org/wiki/Kernel_same-page_merging) in disaggregated memory systems. 
- See if [OS kernels](https://www.usenix.org/conference/osdi21/presentation/bhardwaj) that use [node replication](https://cs.brown.edu/~irina/papers/asplos2017-final.pdf) work well when applied to a disaggregated system.
- Develop and explore system software support for [Buffets](https://dl.acm.org/doi/10.1145/3297858.3304025), a new communication primitive for AI hardware. 
- Develop a CXL emulator (on a NUMA system) and implement a user-space daemon to migrate pages between the local and "far" node
- Compare memory compression (e.g., Linux zswap) to memory tiering (i.e. allocating pages on a remote CXL node)
- Develop a custom tiered memory allocator for the Linux kernel
- Run memory performance experiments on the Cerebras waferscale engine. For example,
you might approximate a memory hierarchy using the core-local memories. 
- Add a processing-in-memory (PIM) ISA extension to a RISC-V core that triggers in-memory computation (ala [Tesseract](https://users.ece.cmu.edu/~omutlu/pub/tesseract-pim-architecture-for-graph-processing_isca15.pdf)
- Apply disaggregated memory (CXL, far memory, etc.) to Genome mapping algorithms (or other scientific workloads) 
- Implement a CXL controller on an FPGA (doing experiments on CXL access delay)
- Design a hardware-managed memory-tiering device (simulated or on FPGA). This would track page hotness and automatically migrate data between memory tiers, like [HeteroMem](https://arxiv.org/abs/2502.19233)
- Take a modern AI workload and understand how it is bottlenecked by the hardware (through experimentation, profiling, and performance counters) 
- Model the cost/performance of a set of workloads to provision a virtual datacenter, i.e. estimating the total cost of ownership (TCO) for CXL expanders vs local memory
- Develop a discrete event simulator for analyzing the projected benefits of near-data processing (NDP) in the context of databases
- Investigate how a state-of-the-art consensus protocol, e.g. Raft or HotStuff, is affected by far memory (CXL) latencies. 
- Explore using GPU memory (or accelerator memory for some other device) as a far/disaggregated memory tier 
- Apply [paged attention](https://arxiv.org/pdf/2309.06180) to newer open-source AI models. 
- Explore hardware support for paged attention or [vAttention](vAttention: Dynamic Memory Management for Serving LLMs without PagedAttention)

