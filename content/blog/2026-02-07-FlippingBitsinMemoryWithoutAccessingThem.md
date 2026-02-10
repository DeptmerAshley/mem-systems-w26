+++
title = "Flipping Bits in Memory Without Accessing Them"
[extra]
bio = """ """
[[extra.authors]]
name = "William Davis (Leader Presentor)"
[[extra.authors]]
name = "Carlos Alvarado-Lopez (Presentor)"
[[extra.authors]]
name = "James Tappert (Scribe)"
[[extra.authors]]
name = "Paul Suvrojyoti (Blogger)"
[[extra.authors]]
name = "Kabir Vidyarthi(Blogger)"
+++



# Introduction

First let us start with the physical structure of a DRAM. At the lowest level Dram is a two dimensional Grid of Cells and each Cell stores either 0 or 1. The cells are mode of two components i)Capacitor - the one that holds the electrical charge ii) Access Transistor - these act as switch, that locks/releases the charge in the capacitors.



##### The Grid:

* Wordline - A horizontal line connecting all the cells in a row
* Bitline  - A vertical line connecting all the cells in a column



##### The Access:

Computer cannot access single cell directly, it has to get a entire row if a data first and memory controller controls this process. To read a cell the memory controller issues an activate command and this triggers(high voltage) the voltage of a specific wordline, which turns on all the access transistors of that row, connecting each capacitors to it corresponding bitlines.



## But before we go further we need to understand what happens when Dram reads first

* Precharge - The memory keeps the bitline to Vdd/2 (neutral position). Bitlines are seen as bigger capacitors than the cells.
* Activate - When the activate commands connects the cell capacitors to the bitlines, the cell shares charge with the bitline. As a result the bitline moves a tiny bit up/down from vdd/2 (as the cell capacitor charge is much smaller compared to the bitline charge).
* Sense Amplify - This tiny movement is noticed and amplified by this sense amplifier and it drives it hard towards the noticed direction too. That is tiny up becomes full 1 for bitline and tiny down becomes full 0 for bitline.
* Restore - While it is amplifying the btiline it also starts restoring the cell capacitors to its original value or threshold value.

Once the read is done, the row values are in the row buffer (sense amp) and cell values can be accessed quickly.

* Precharge again - Once the row is closed, bitline precharging starts again that is bring to it is Vdd/2 level again for next row access.



##### Dram Refresh:

Since the cells are capacitors, it tends to leak and lose charge overtime, so Dram needs to refreshed (restoring its charge to is orginal or threshold value)



## Row Hammer Mechanism and Error:

Now that we know how a Dram access happens, let us see what rowhammer error is. Memory chips are getting smaller and smaller and more cells are getting crammed up into smaller spaces, and because of this tight packaging interference is happening (electromagnetic coupling). And also since the cells are getting smaller too for more density, they hold less charge therefore losing or leaking charge and going below threshold is easier.



The Authors found out that if they access a row repeatedly which will mean toggling the voltage of the wordline on and off rapidly, it creates a electrical noise  and this noise causes the cell in the neighboring rows to leak charge much faster than normal. They will end up leaking and losing charge and going below the threshold before the DRAM gets refreshed, which means permanent loss of data or flips (1 to 0).



The wrote a simple program to prove this happens on real Intel and AMD processors. It showed a simple user level program can induce flips by:

* generating loads that actually go to DRAM (not cache) and
* choosing addresses that force the memory controller to open/close rows repeatedly

The key trick is evicting cache lines between accesses, so each read becomes a DRAM access. For making this happen they used an instruction called clflush (Cache Line Flush). They showed that alternately touching two addresses mapped to different rows in the same bank forces the controller into a pattern like open > row X > close > open row Y > close > repeat.



They tested 129 DDR3 modules from 3 major manufacturers and 110 were found with this bit flipping error disturbance. All modules from 2012 to 2013 showed this problem but the older ones did not, which shows as technology got smaller this became worse.



##### Key Findings:

* Minimum activations to see an error - The paper mentioned as few as 139k row activations before a refresh were enough, which is scary small.
* Locality - The errors (bit flips) are usually concentrated in two rows (called the Victims) near the hammered row (called the aggressor). They also mentioned the density of vulnerable cells to be upto 1 in 1.7k cells.
* Faster Refresh -  Refreshing Dram fast enough can make the errors disappear as we already know why.



This findings were alarming as memory is assigned to programs as pages. So one row might belong to the web browser(aggressor) and the row next to it might belong to the OS(victim) and by hammering the memory, a malicious website could flip bits in the OS memory and take control of the system.



## Solution:

* ECC fix - This is not a complete fix as Rowhammer can produce multiple bit flips in the same ECC word and ECC cannot repair multiple errors. It doesn't stop the error, rather tries to flag/repair it (some of it).
* Frequent Refresh - This does solve the problem but it very power hungry. And also refreshing the whole memory is too slow.



* The proposed solution -  PARA (Probabilistic Adjacent Row Activation)

 	i) Every time the controller closes a row R it flips a biased coin with probability p (refresh)

 	ii) If the coin hits, the controller refreshes on adjacent row (it could be R-1 or R+1, best is to do it 			alternatively)

 	iii) If it misses, do nothing.



Why PARA works - They chose p to be small but large enough so that the chance of no neighbour refresh during heavy hammering becomes almost 0. Let us say a program tries to hammer a row 139k times before the next default refresh, statistically the coin flip will hit at least once and an unofficial will happen at least once which prevents the error.

This is cheap and very effective with very low failures.



### Results:

* Nth - They first assume an attacker hammers an aggressor row just enough times with one refresh interval and they call this count Nth. They took Nth range between 139k-284k activations for being realistic.
* They took p=0.001 (smaller is better as less overhead), therefore for tow neighbour it will be  p/2 = 0.0005
* According to their table if N = 50k, 100k, 200k, then PARA fails are given by  P(fail in one refresh interval) = (1−p/2)^Nth and the results were (1.4 x 10^-11), (1.9 x 10^-22), (3.6 x 10^-44) respectively.



So we can see that the faliure probabilities are very very tiny. They simulated 29 worklaods on a modeled system(4 GHz , dual cahnnel DDR 1600) and also assumed a row can have upti 10 neighbours as a result they increased the p five times to 0.005. The average throughput degradations were 0.197% and worst case was  0.745%. This shows how reliable PARA is and it also casues a very small performance impact.





## Class discussion

* Would a combination of charged and uncharged capacitors result in more errors

  * Errors are more likely to be caused when we have uncharged capacitors in the proximity of charged capacitors rather than by the overall ratios

* Could we catagorize data stochastically by amount of zeroes to prevent this form of error

  * overhead would be extremely costly, though it has potential to work

* Vulnerabilities exist with rowhammer and arbitrary code injection into user machines



# Conclusion

This work shows that Rowhammer is not just a theoretical error. It constitutes a real hardware-level reliability and security vulnerability emerging from an increase in DRAM density. By explaining the mechanism through which DRAM is accessed, the paper makes clear how repeated row activations can induce charge loss and cause bit fliips in neighboring rows. Among proposed mitigations for these issues, PARA stands out for being practical and having low-overhead, both reducing the error rate and making it less deterministic, and thus less exploitable. A key takeaway to these findings is that memory can no longer be treated as a prefectly reliable abstraction, future systems must take into account both hardware and software defenses to ensure both correctness and memory safety.



# References

\[1] Yoongu Kim, Ross Daly, Jeremie Kim, Chris Fallin, Ji Hye Lee, Donghyuk Lee, Chris Wilkerson, Konrad Lai, and Onur Mutlu. 2014. "Flipping bits in memory without accessing them: an experimental study of DRAM disturbance errors". SIGARCH Comput. Archit. News 42, 3 (June 2014), 361–372. https://doi.org/10.1145/2678373.2665726

## AI Disclosure

* Generative AI was used to aid in research and formatting for this blog post
* Generative AI can be useful tools for tasks such as summarizing or drafting, however, they may give inaccurate information confidently and should always have generated information validated
