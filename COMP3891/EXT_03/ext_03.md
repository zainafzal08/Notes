
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
