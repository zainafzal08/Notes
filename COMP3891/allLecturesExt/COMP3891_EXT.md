
# Extended Lecture 2

try atom, visual studio, etc.

 a context switch can refer to a switch betwen threads or processes. invovles saving and restroing of state. 
 
 when doing it with processes you need to switch the threads and the extra info associated with process, i.e memory maps. (this is called a full context switch)
 
 os161 only has threads, no processes. at least atm lmao we have to put that in 
 
spl stands for set priority level, allows us to meddle with interrupts. in the old days setting the priority too high means no interrupt could meet it and this interrupts are disabled. 

splhigh() disabled interrupts and returns the old interrupt level so it can be set. 

splx, sets the interrupt level. 

this is better then enable and disable because if yoi nest functions that play with it, each one will restore it back to what it was. 

don't use this in the assignments. 

if a critical section takes hours to run the interrupt blocking method will just cause a system wide pause.

spinlock aquire is a function that sets up a spin lock, i.e poll the lock until it's open instead of sleeping. 

HANGMAN_WAIT is a way to detect deadlock via a timeout

spinlock_aquire and release is just interrupt enable and disable for uniprocessor system. 

assert!!! do it!

lock_do_i_hold is better then lock_who_holds because by the time you know who owns the lock that info has changed. 

wchan is wait channel not w-chan lmao

each lock has a wait channel. 

the zombie state is just a stack which you cant destory within a thread. 


# Extended Lecture 2 (THIS CAN BE TESTED)

## Interfaces
---

An operating system has a well defined interface to abstract out the hardware details. 

The hardware details are also well defined instruction sets. Like AMD and Intel both implement mostly the same ISA. it allowed programs compiled down to be run on both mostly. 

Software can run on both the OS and the HARDWARE. 

#### Instruction Set Architecture (ISA)

interface between software and hardware, divided between privileged and un privileged parts. Of course the un privileged is just a subset.

#### Application Binary Interface (ABI)

This is the system call interface + un-privileged ISA and creates the interface between PROGRAMS and hardware

#### Application Programming Interface (API)

Interface between high level languages and the libraries/hardware/os. 

consists of library calls + unprivileged ISA. 

Portable via re-compilation to other systems supporting API or dynamic linking (which allows different versions of the api to link up to systems and is managed by the os). 

#### Some interface goals

Support deploying software across all computing platforms and provide a platform to securely share hardware resources. i.e cloud computing. Amazon sells you the illusion to access to a machine. 

## Virtual Machines
---
> OS is a extended virtual machine

it multiplexes the machine between application to allow for time sharing, multitasking and batching. 

provided a higher level machine for east of use, portability of applications, efficiency via letting you use optimised hardware without dealing with the optimisations and security. 

#### Abstraction vs Virtualisation

The os lets users think that memory is managed as files but really below the interface it is a SSD. But we could play this differently. 

We could provide a interface where the user thinks they have access to a disk when really beneath it it's just a file in a larger operating system managing the actual hardware. 

it's a higher level abstraction. 

#### Process versus System Virtual Machine

a process VM is where the hardware is interfaces with a OS which then helps support a virtualising software on which an appliction can run. 

`hardware -> os -> VM -> Application`

this looks like the application is running on a VM. 

with system VM we have the VM beneath the OS

`hardware -> VM -> OS -> Applications`

in this case the application runs on a os and both of these interface with the VM. 

THE VM mimics the hardware in this case. 

#### JVM

this is a process VM, it is architecture independent but the JVM runs on top of the OS. the language is clean, robust and garbage collection was great. 

The generated bytecode is interpreted or just-in-time compilation.
Lower than native performance though. 

this pissed off windows and caused drama

it was great though it provided a interface independent on underlying os. 

the thing is though that it doesn't provide a platform to securely share hardware resources. it's 1 JVM and the security sucks. it also can't run legacy applications. 

even if you had multiple VM's then you have to trust the underlying OS to give you security. do you trust?

#### Just in time compilation
where normal code is compiled down into a IR to object code (an executable) which is then loaded as a memory image which can be executed by the process. 

in java it goes compiled into byte code. when it get's loaded it turns into a VM image which then converts into Host Instructions. How it does this last step is Just In Time Compilation (JIT) which is better then just a giant if statement (move is mv in this computer)

Here there is a jump table of all the functions available in the byte code, when the code start's getting run (interpreted) 

the code it tries to run is converted to native code function by function and then referenced by the jump table. then if the function is called again you just use the jump instead of recompiling it down. 

it's great it only converts what you run so if 700 functions are never run they are never compiled down to native. 

#### Is the os the right level of extended machine?

we have to consider trusting the OS as every OS has security bugs. 
We also have to consider if your os can handle legacy. 
resource management, performance isolation. 

what about activities requiring root privileges. 

#### VMM

Virtual machine monitors also known as a hypervisor. It provides scheduling and resource management but doesn't provide a API. It lets us run a os ontop. 

---IBM VM/370

CMS was a light weight single user OS and this IBM let you run multiple copies of CMS. 

a CMS would call a Trap which went through the VM before getting to the hardware. 

It is as if the CMS has the machine to itself. 

the VMM had to handle the CMS ""system calls"" and make sure the CMS thinks the system calls is going straight to hardware. it emulated the calls but really managed the hardware so one os didn't do shifty stuff in privilege. 

This has many advantages, it lets us use Legacy OS and applications, even lets us use legacy hardware. 

we can use server consolidation to have cost and power saved. each server now can do multiple services. 

have concurrent OSes. easy server migration, move the server onto new hardware, the VM handles all the code as long as it works with the new hardware. we can even handle backups. test and development. 

The VMM sorta has more security, multiple OS's are isolated. But in reality they also have bug lists. 

the performance is near bare hardware though. the idea is lots of math isn't that bad, is quick, but if you do i/o and have to take a extra trap to pass through the VM it hits harder. 

compute bound is fine but IO bound gets slower. 

#### Type 1 (native) Hypervisor

this is the system VM. it runs in the most privileged mode. 
guest OS runs in non-privileged mode. the Hypervisor may implement the virtual kernal and user mode for the guest os or the cpu may have 3 modes (hypervisor mode, guest os mode, application mode)

#### Type 2 (Hosted) Hypervisor

this is the process VM. the hypervisor runs as user-mode. 

if the application tries to do a privileged instruction it goes down to the OS which tags the VM as a fucking asshole, he TRUSTED THE VM AND THE VM JUST DID SOMETHING BAD

what usually happened is the hypervisor installs a driver which lets it stop windows run itself on the physical machine. it's a world switch. 

they are linked if the os is fucked or the vm is they all go down. 

the vm stil uses the os for some needs. 

# Extended Lecture 3 (THIS CAN BE TESTED)

This is a lazy lecture, the slide numbers are given and any additional comments added. 

`25`

Every sensitive instruction should cause a trap into the hypervisor.
 
`27`
trap the sensitive or privileged instruction and emulate it's activity. 

i.e  disabling the interrupts just means that the internal timer interrupts are turned off in the hypervisor. 

One way the VMM/Hypervisor can handle this by having a set of variables per os which represent stuff like coprocessor 0 status registers

Then the VMM can force the system to never jump to the general interrupt vector so it looks like to the machine there are 0 interrupts. 

`28-29` it's sorta privileged not entirely. The EFLAGS register is available from user and kernel. What it did was not raise an exception but silently ignore any change made to sensitive registers. Which meant that it was impossible to tell as no exception was raised. This was not virtualisable. 
VMWare actually did fix this with binary rewriting. Every time it loaded a kernel page it replaced every pop stack setting the IF FLAG in the register with a actual call to the hypervisor. amazing. 

## Scheduler Activation
---

`8`

It is incredibly quick to do user level threads, like so quick, order of magnitude quicker the kernel threads. oh god and making a new process for every thread my fucken god so slow. 

`9`

here the Scheduler acts as the bridge, it assigns one of its 3 internal threads onto 2 of the available kernel threads. 

n on m threading (used in windows fibres)

here we create the kernel threads first so we don't have to recreate that, it's a bit slower at start time but then literally nothing because after they are created the internal code the kernel thread runs is swapped out by the user Scheduler. 

There is a issue though because if both the kernel threads block the 3rd internal user thread is blocked as well. Moves the problem back a bit but it's still there. You have to tune the number of n to m to make it work 

`11`

the idea is having the user and kernel schedule communicate. 
this is self tuning. 

`12`

a normal system is a downcall, the user calls to the os, transitions, runs code then returns. 

a upcall is when the OS wants to give the control to the application it actually just returns exeuction at a well known SET entry point and before going there gives the application some info on where it was before i,e registers stack pointer etc. 

the application then figures out what it wanted to do, the entry point is usually the sceduler.
The application can restore or switch. 

`15`

basically instead of assuming the thread can't do anything until the system call finished the os goes "hey user, i'm gonna be a while, you can do some other stuff while you wait"

when the os is done it goes "hey bro the guy who was previously blocked is fine, you should prob deal with the fact that your hypothetical thread that was running before finished"

note that the kernel thread is what gets blocked it just lets the user level handle that. 

`16`

virtual CPU = kernel thread

preempted is "user yo, i'm fucking with you and taking a kernel thread, deal with it"

`17-27`
the system PROVIDES for another kernel thread to the process if one blocks. 












 
# Extension Lecture 5, Log Structured File Systems

## Original Motivating Observations

Memory Size is growing at a rapid rate, so we have a growing proportion of file systen reads will be satisfied by file system buffer cache. 

Writes will increasingly dominate over reads and thus must be optimised. We can avoid accessing the disk in terms od reads thanks to caching but not writes. 

I-node does poorly for small files. 1 very small file (which is the typical use case for coders and system designers, we work with smaller files) wuld take 5 writes to set up the whole block.

and each of these writes are at a different point in the group meaning head moves. 

Synchronous writes (write-through caching) of metadata and directories makes it worse as each operation will wait for the disk write to complete. Basically usually we have a 30 second buffer where any write to the disk is cahced for 30 seconds then flushed into the actual disk. But the OS doesn't do this, it it does a small meta data write and waits for the disk to finish before going to the next one. It can't risk having a 30 second window where something goes wrong. 

Furthermore every thing you do usually results in multiple head moves and look ups and meta data updates. 

so what we want to do is imporve our performance for small writes and see if we can improve this slow and annoying meta data process. 

We also needed to deal with the awful boot times and file system checking (this method, log stuff, came before journaling)

## Basic Idea

we just fill up the disk with all the seperate sections of a block in no real order, we just write sequencially. this basically looks like a log as we write files in order of time. hence the name. 

we place the data next to the i node next to the directory entry. Because we writes these sequencially the write is super fast, no seeking. 

## Locating i nodes

But now how the `fuck` do we find out i nodes. there is no i node table anymore. 

What we do is keep a inode map. it maps inode numbers to locations. 

Keep a map of inode locations

- Inode map is also “logged” as in written sequencially. we don't update the mpa, we copy it, add it to the front of the log and change it so it is consistent. 
- Assumption is I-node map is heavily cached and rarely results in extra disk accesses. Remember reading isn't as big of a issue as writing. 
- To find block in the I-node map, use two fixed location on the disk contains address of the inode map blocks. 
    - Two copies of the inode map addresses so we can recover if error during updating map.
    - Basically both should point to the most recent and if either one is corrupted we just grab the most recent pointer (via timestamps) that isn't corrupted. At best we just update the second pointer to match the first.
    - At worst we roll back to the old version so we lose some data but roll back to a consistent state close to what we had at the time of crash. 

<img src="src/EXT_05/D_1.png" width="400px">

the image says 2 disks, it means 2 disk blocks. 

## Cleaner

File system cleaner runs in the background and recovers blocks that are no longer in use by consulting current inode map. 

Compacts remaining blocks on disk to form contiguous segments for improved write performance. 

Basically when we delete a block we just remove the inode map and then the cleaner takes care of the rest, sorta like java's garbage collector. 

How the cleaner does this is via 2 methods, threaded log (put the free blocks into a linked list so we can write semi sequencially.) and copy and compact which is stock standard just grab all the used blocks and chuck them next to each other. 

threaded log is faster, less overhead but results in more fragmentation whereas the copy and compact has more compression but way more overhead. 


<img src="src/EXT_05/D_2.png" width="400px">

But wait, if we move blocks around doesn't that make the inode map inconsistent. 

How it works is we copy all the data into a new area where it can be written sequencially and then tag the previous copy as free. 

## Recovery

File system is check-pointed regularly which saves – A pointer to the current head of the log

- The current Inode Map blocks
    - On recovery, simply restart from previous checkpoint.
- Can scan forward in log and recover any updates written after previous checkpoint
- Write updates to log (no update in place), so previous checkpoint always consistent

this means we don't have to have a file system checker! we just use a snapshot. 

## Performance

<img src="src/EXT_05/D_3.png" width="400px">

wow it's amazing huh? well....

Margo Seltzer and Keith A. Smith and Hari Balakrishnan and Jacqueline Chang and Sara Mcmains and Venkata Padmanabhan all basically set up a log file system in BSD 4.4 and did a direct comparison with the normal FFS and this. 

they most importantly did a critical examination of cleaning overhead. furthermore FFS just has clustering come out with was a good optimisation so it was a good time to have another look at it. 

note with this graph we have BSD-FFS with a clusterfactor of 1 and 8 (m1 m8)
and the r value just the fact that they stored data on the disk with gaps between. What happened was if you sent 2 requests chances are it's sequencial so the disk really should just grab both and return them. but if you weren't fast enough with your requests you might grab one block and by the time the second request gets there the disk has flown past and the head now needs to wait for the disk to do another turn become getting the next data block which was stored right after the first.

the empty spots inbetween give you time to send 2 requests and the disk grab both quickily without waiting for the disk to turn.

<img src="src/EXT_05/D_4.png" width="400px">

so hey deleting is fast but who the fuck does this many small file deletions etc. how does it behave for large files, because those still exist

new benchmarks:

1. Create the file by sequentially writing 8 KB units.
2. Read the file sequentially in 8 KB units.
3. Write 100 KB of data randomly in 8 KB units.
4. Read 100 KB of data randomly in 8 KB units.
5. Re-read the file sequentially in 8 KB units

<img src="src/EXT_05/D_5.png" width="400px">

the clustering effect makes it just as good as the log structured file system. 

random writes is basically just rsequential writes to the head of the log. 

note with the random read that BSD does this thing where if you ask for 2 blocks that are next to each other it assumes you are doing sequential reads and turns on prefetching where it grabs the next set of blocks before you ask for them expecting that you will. 
if the 3rd block you request isn't sequencital it realises it was wrong in assuming you were going to be reading sequentially and turns that off. 
This is usually a good optimisation but sometimes you get fucked in the worst case like in this test. We reads 2 4K blocks next to each other then jump to another area. 

that's 2 sequential reads then a random read meaning that we are CONSTANTLY turining on sequenital pre fetching, doing the over head with that then turning it off and basically just wasting all the prefetched data. 

<img src="src/EXT_05/D_6.png" width="400px">

you can see here the cross over point is when the files are so large that meta data optimisations like those in the log FS don't mean much. 

<img src="src/EXT_05/D_7.png" width="400px">

here note that the random drop happens because BDS switches from the single indriect to multi indirect when the files start getting big so it has to switch between data blocks. 

<img src="src/EXT_05/D_8.png" width="400px">

yeah about the same, smaller files work bettwe with log. 

<img src="src/EXT_05/D_9.png" width="400px">

log wins because it cheats with the delete, doesn't actually delete it just removes from the map. 

But now what happens when the disk fills up!!

<img src="src/EXT_05/D_10.png" width="400px">

the cleaner fucks shit up hard. 

## Is LFS a clear winner?

When LFS cleaner overhead is ignored, and FFS runs on a new, unfragmented file system, each file system has regions of performance dominance.

- LFS is an order of magnitude faster on small file creates and deletes.
- The systems are comparable on creates of large files (one-half megabyte or more).
- The systems are comparable on reads of files less than 64 kilobytes.
- LFS read performance is superior between 64 kilobytes and four megabytes, after which FFS is comparable.
- LFS write performance is superior for files of 256 kilobytes or less.
- FFS write performance is superior for files larger than 256 kilobytes.

Cleaning overhead can degrade LFS performance by more than 34% in a transaction processing environment. Fragmentation can degrade FFS performance, over a two to three year period, by at most 15% in most environments but by as much as 30% in file systems such as a news partition.

BUT when meta-data operations are the bottle neck, LFS wins and it still used in ZFS and BTRFS (Better FS) which use this idea of snapshots.

they also sorta exist in journaling file systems. 

Hybrid of
- I-node based file system
- Log structured file system (journal)
- 
Two variations
- log only meta-data to journal (default)
- log-all to journal

Need to write-twice (i.e. copy from journal to i- node based files)

Example
- ext3
- Main advantage is guaranteed meta-data consistency

















# Extended Lecture 6, Anticipatory scheduling
a disk scheduling framework to overcome deceptive idleness in synchronous I/O

Proceedings of the 18th ACM symposium on Operating systems principles, 2001

## Disk Scheduler
---

we want to reorder available disk requests for seek optimisation or proportional resource allocation (i.e some users should get better access to the disk then others)

The issue with synchronous I/O is that often a program will not request the next piece of sequential data until it recieves back the first one and processes it. which means you can read some data, go and handle some other reuqest and the nhave to come back to because in you going off to handle another one the first program submitted the next peice it needed which was right next to the first. 

#### Deceptive idleness

Process A is about to issue next request but Scheduler hastily assumes that process A has no further requests!

#### Proportional scheduler

These schedule service is a ratio, it serves process B twice for ever process A request it processes. 

This works if there are already 10 for both process A and B. but if B has 1 and is about to call another one the scheduler just shifts back to process A again. 

#### Prefetch

Overlaps computation with I/O, it avoids deceptive idleness. 
we can do this in the kernel OR in the application. 

Application driven is done via async io. 

<img src="src/EXT_06/D_1.png" style="400px">

Here we can issue multiple reads in the application and because i don't wait for one to finish before requesting the next one i get the desired effect. I can do this via blocking where i wait for all the previous reads to complete. You can also poll where we test if our calls have finished. Usually you'd aim to do some useful computing between checking for them to be finished. 

<img src="src/EXT_06/D_2.png" style="400px">

You can also use signals and signal handlers to get a interrupt when it's done

This is hard to program, applications need to know their future i.e don't need every piece of data to know where to look next, think linked lists. 

existing apps now need to be re written and also may be less efficient than mmap (memory mapped files and paging). ALSO a lot of kernels at the time of this paper didn't have async lmao

#### Memory Mapped Files and Paging

look at the video lmao hard to explain

#### Kernel Driven

less capable of knowing the future, Access patterns difficult to predict even with locality (ready every 10th block in the file is hard to pick up), cost of misprediction can be high (reading in or removing something from the cache that isn't needed or is needed) 

medium files are also too small to trigger sequential access detection. 

## Anticipatory scheduling
---

key idea: sometimes we wait for process whose request was last serviced. 

keeps disk idle for short intervals but with informed decisions that improves throughout and achieves desired proportions (2:1).

When should we or shouldn't we delay disk requests, how long? HOW?

#### Cost-benefit analysis

Balance expected benefits of waiting against cost of keeping disk idle. 

<img src="src/EXT_06/D_3.png" style="400px">

think-time is computation time. graph 1 tells us at what time 95% of the requests would have come through. 

expected positioning time is the average time the process takes to switch between requests. 

<img src="src/EXT_06/D_4.png" style="400px">

here cost is the median think time. 

<img src="src/EXT_06/D_5.png" style="400px">

#### Experiment - Microbenchmark

it wwas pree good

<img src="src/EXT_06/D_6.png" style="400px">

Alternate is read, stop, read, stop. I.e it prevents the pre fetcher from kicking in. ... so what happens if we turn the prefetcher off for all of them??

<img src="src/EXT_06/D_7.png" style="400px">

but here we arn't measuring CPU use of the new code. the Andrew benchmark is a benchmark. 

<img src="src/EXT_06/D_8.png" style="400px">

Scan is a command that doesn't take any advantage of prefetching hence the improvement. 

GCC takes longer because GCC can not be improved at all, it's big reads and big writes, it's mostly computer bounded not I/O bounded. The change is due to the over head of keeping track of these think times etc. 

Apache web server is also a benchmark 

<img src="src/EXT_06/D_9.png" style="400px">

As is Database

<img src="src/EXT_06/D_10.png" style="400px">

The database gets betters with more being added as switching between databases sucks and waiting a bit longer to service the requests is a good idea. 

GnuLD is a basically a random access generator because it links files in separate locations. with on instance it's about the same with two though you have two instances and each get's a chance to maximise locality access now. 

An Intelligent adversary is a user who knows the algorithm and tries it's best to fuck you up, i.e it sets up the median think time to make it wait then don't do anything it is about to cut you off. 

But it's not that bad here, the system handles it well. 








# Extended lecture 7, Assignment 2 tips

Open is statically linked into the user mode, it triggers a syscall through a trap frame into the os with the information about what is happening. 




# Virtual Linear Array Page Table

It's like a 2 level page table but all in virtual table. Means we can do it in 1 lookup as we can index in the array. 

It's in the virtual address space so even if the array is 500 GB but if frames are empty then physical frames arn't allocated there. 

The root Page table is kept in physical memory.

the context register simply loads the Page table entry by indexing off the page table entry start point. 

ccasionally will get a double fault. User faults to os saying memory doesn't exist, Kernel tries to look up the page and faults because that page entry doesn't exist which then goes down to the physical 4k level one table, it assigns a page table entry which then maps to physical memory. 

C0 Context register contains the Base of the PTE and the VPN. 

C0_Context = PTEBase + 4*PAgeNUmber (32 bit page table entries)

whenever we fault this is loaded up already with what to look up. 

Tapeworm lets u take a stream of tlb misses and have a virtual tlb simulated. 

OSF1/Mach 3.0 have 3 levels of table but have the 2 deeoer ones in virtual memory. 

Ultrix and OSF/1 used virtual memory for various data structures. Mach 3.0 was exerminting pushing file system and nerworking and scheduling not in the kernel, this was called a micro kernel. 

Itanium Page Table. Takes a bet. 
# Extended Lecture 10, Assignment 3 Intro

## Pointer Recap
---

lets assume we have 4-bit addresses, i.e we have address range from 0-15

remember that if you write a int at address 4 it actually writes into address 4 5 6 and 7 because ints are 4 bytes long. 

(a lot of this is just how c handles pointers, nothing special)

## Intro
---

Has two parts
1. Frame Table (An allocator for physical memory)
    - Test Heavily if this fucks up, everything else fucks up in placed that makes it hard to trace the problem back 
2. Page table and region support
    - virtual memory for applications

There are some tests in the wiki that are ok. 

you can edit the os161 max ram to test your program with running out of memory. 

There is a macro defined for converting physical and virtual addresses into each other which adds or subtracts the needed offset. PADDR_TO_KVADDR converts a physical address into a kernel virtual address. 

ram_getfirstfree() gives you the bump allocator, you should only use it once and from that point onwards take over allocation. 

You can assume that only 1 page at a time, i.e they don't ask for >4k. if it's more then 1 page, return NULL. 

free frame table should be a liked list. 

we can place the frame table at the top of free ram and then decrease free frame size 

YOu can also place it at the bottom and use the bump allocator, this is a little encouraged this year. 

you can have frametable be t0 begin with and we can detect this as a signal to "use the bump allocator cause the actual stuff does'nt exist yet"

load_elf is good to read. you can use -objdump to get location info of memory. 

bin true is a good starting test program, thereis also huge which allocates HEAPS. 

# Extended Lecture 11, More Ass 3 Tips

address space functions are called for each for the sections of code, we have to keep track of what is assigned so we our vm_fault knows what's going on. 

There are sections for code, data, the user stack etc. 

walk through EFL loader is a good thing to read...

as_preapre_load() should make READONLY regions READWRITE for loading purposed and when as_complete_load() turn it back all. 

We remove a page from the page tables in as_destory. 

we don't care about freeing memory for the os shutting down. 
we only really want to free memroy that we can reuse usefully. 

the slides go through all of the address space functions .

rip the code for as_activate from dumb_vm

you can have as_deactivate call activate. 

there is a good flowchart of how VM Fault works. 

don't use kprintf() in the vm_fault(), kprintf() blocks the current process while printing, this calls a context switch which empties your table that you just filled lmao. 

use trace161 !!

note as_copy is where we do shared paged, you don't copy it for read only segements and also all the time, just copy it when one tries to use it. 

 there does need to be a reference count in the frame table then because of this. 

 ADVANCED COMPONENTS HAS TESTS!!! 

 

