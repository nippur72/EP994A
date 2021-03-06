# EP994A
My TI-99/4A clone implemented with a TMS99105 CPU and FPGA (master branch).
Another version of the clone (the latest development in soft-cpu-tms9902 branch) includes my own
TMS9900 CPU core written in VHDL.

See the file LICENSE for license terms. At least for now (without contributors from others)
the source code is made available under the LGPL license terms.
You need to retain copyright notices in the source code.

Latest changes
--------------
Commit 2019-01-31:
Merged the soft-cpu-tms9902 branch to soft-cpu branch. I have not used enough git to be great at it, but this first major merge seems to have gone well. There are two generics in top level object:
- **cfg_spi_memloader** enables SPI (microcontroller) boot instead of boot from PC.
- **cfg_hw_keyboard** enables the use of an genuine TI-99/4A keyboard with the FPGA. The keyboard needs to be wired to the 20-pin GPIO connector.
- I haven't tested these flags; they were features incorporated to the **soft-cpu** branch earlier, while my main development branch has been the **soft-cpu-tms9902** branch, which includes the TMS9902 compatible UART. Now the soft-cpu branch includes all features.
- Before the merge I have been working on optimizing the CPU core, removing unneeded states which didn't do much and adding some shortcuts in the instruction decode states, avoiding some more unnecessary states.
- The net effect of the optimizations so far is that my stupid Basic benchmark now runs at 39x faster than the original TI (when the cache is enabled). This is still slow, but probably the world's fastest TI-99/4A implementation still.


Commit 2019-01-19:
**cache**
- Added a system level cache, outside of the CPU core (soft-cpu-tms9902 branch). 
- The cache size is is 1K byte for code and data. Occupies a 512x36bit RAM block. The bottom 16 bits of each of the 512 entries is the 16-bit data word, followed by tag bits. The tag is 10 bits. 
- The topmost bit, bit 35, indicates if a cache entry is valid. When set, the entry is valid. On reset the cache cycles through all 512 locations and zeroes them out, thus invalidating all entries of the cache.
- The update policy is write-through. Thus all memory writes go to main memory. The cache will also allocate entries on writes, if the memory region being written to is cacheable.
- During reads the read is simultaneously started for both main memory and the cache. Cache read takes one cycle. Another cycle is currently required for tag comparison. Thus read cycles which hit cache take two clocks. Any main memory cycles will be aborted if a cache hit is detected.
- Using my simple stupid BASIC test program the cache improves performance by 22% in the current very simplistic setup. At 100MHz the system reaches over 30x the performance of the original TMS9900.

```
10 for i=0 to 1000
20 print i;" ";
30 next i
```
**video**
- A small modification to the TMS9918 core to better center horizontally the video output.

Commit 2018-09-22:
- After a long while worked on the project. This was pretty much trying to remember where I was in the project.

- I synthesized again the master branch and also worked on the soft-cpu-tms9902 branch. The master branch is the branch which supports the TMS99105 CPU on the daughterboard / shield that I designed two years ago. Still works.

- There was some actual progress on the aforementioned **soft-cpu-tms9902** branch. I clarified the naming and processing of reset signals. Thanks to this now two bugs are fixed:
	- Sound works (again?) now that the audio DAC is not constantly being reset.
	
	- The serloader component (handling communication from the host PC over USB serial port to the memory of the TMS9900 via DMA) was being reset by mistake while the CPU was placed to reset. Now if the host PC put the CPU to reset, that reset would also the serloader, effectively preventing any furhter communication with the system. This of course sucks big time, as the main use case for putting the CPU to reset in the first place is to load software to the memory of the TMS9900 system.
	
	- I know that my FPGA CPU core has bugs, and I found a repeatable one: running BASIC and doing a simple multiplication with PRINT 1*-1 yields always 1 (plus one) with my FPGA CPU, while an actual TI-99/4A or my TMS99105 FPGA system (i.e. the master branch using real CPU silicon) yields -1 as they should. So this bug can be observed with high level software... There we go. It is a miracle BASIC runs in the first place. The bug is probably related to CPU flag handling. I also have been reported by **pnr** that my divider implementation does not work properly in all cases, so need to check that too.
	
Good to be back with the project.


Commit 2018-01-03:
- So I have been very lazy at updating this README file. There has been a ton of changes.
  Note that there are two branches, master branch contains the **TMS99105** version and soft-cpu contains the **FPGA CPU** version. 


Commit 2016-11-13:
	
- Added firmware/diskdsr.asm which is a Device Service Routine for disk I/O support. It currently
	registers DSK1 and DSK2. It support LOAD and SAVE opcodes. Support means that it will
	pass the PAB to the PC host to read by copying it to system RAM at address 0x8020.
	There is a command buffer at address 0x800A..0x8013 which is used for communication between
	the TMS99105 system and the host PC.
	
- Refactored memloader code:
	- Added disk io support. Now if memloader is started with the command "-k" it 
		will not only poll keyboard but also poll memory location updated by the DSR when
		disk I/O requests happen.
		
	- Memloader now parses command line arguments better. Output is less verbose.
		
- FPGA code now supports SAMS memory extension, currently configured to 256Kbytes.
	This required a bunch of other changes, as the scratchpad area needs to be unpaged.
	This is done by remapping the scratchpad above the 256K area used by SAMS.

Hackaday
--------
Project is documented to an extent at Hackaday and AtariAge TI-99/4A forums.

https://hackaday.io/project/15430-rc201699-ti-994a-clone-using-tms99105-cpu

AtariAge
--------
The AtariAge forum thread talks about my other FPGA project as well, but contains information about 
http://atariage.com/forums/topic/255855-ti-994a-with-a-pipistrello-fpga-board/page-8

About the directories
---------------------
**firmware** test software I used to debug the hardware. Written in assembler. Also some loading scripts.
- 2016-11-13 now here is also the diskdsr.asm assembly module, which implements a starting point for disk access. Currently it relies on support by the PC program "memloader".

**fpga** the VHDL code implementing the TI-99/4A (except the CPU).

**memloader** a program for Windows (compiled with Cygwin) to transfer data from PC to the FPGA. This program is used for a few purposes:
- load software from PC to the memory of the EP994A
- reset the EP994A
- pass keypresses from host PC to the EP994A
- 2016-11-13: poll certain memory locations to enable disk access, i.e. saving and loading 
- 2016-11-13: Now there are project files for Visual Studio 2015 community edition. This is just a great IDE and speeds up programming.

**schematics** the schematics of the protoboard (incl. CPU, clock, a buffer chip) connected to the FPGA board. Note: the schematics are in a need of an update, the current version lacks to wires:
- CPU reset from FPGA to buffer to CPU
- VDP interrupt signal from FPGA to buffer to CPU
