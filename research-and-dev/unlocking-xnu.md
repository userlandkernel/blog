# Unleasing the power of XNU
This post describes my research on the XNU kernel and is a log for my strive to get most out of the possibilities a kernel taskport sendrights provides.  
Although I do not have a specific goal in this research I aim to find resources and techniques for making development, research and debugging easier.  
One of my goals is to attempt the creation of a good kernel debugger and access to physical memory.  

## What I use
- Alex digital cable for serial debugging  
- Nanokdp (AppleInternal), but any serial terminal will work.  
- Jailbreak with host_special_port 4, on iOS 12.0.1 (unc0ver)  
- Xcode on a macOS High Sierra Virtual Machine  
- XNU Source code, updated live at https://github.com/UKERN-developers/darwin-xnu  
- iBoot / SecureROM source code, forked from a leak at https://github.com/UKERN-developers/iboot  
- Symbolicated kernels and kernel extensions, from https://github.com/userlandkernel/ios-unstripped-kexts  
- IDA Pro 7.1 with Diaphora plugin, for diffing and symbolicating the production kernel with kexts against the symbolicated ones.  
- Radare2 with my automated offset finding script for generating an importable header file with offsets.  

## Some things explained before getting started
- XNU consits of of three components: Mach (MUCK), BSD Kernel and IOKit.  
- IOKit has a tree of classes and registries.  
- Mach is like UDP, sending messages between processes.  
- BSD kernel provides core logic and network interfaces.  

## In general a kernel can
- Access devices and their RAM.  
- Map physical memory to virtual memory, via the pagemap and the processor's Memory Management Unit (MMU).  
- Control and create processes (tasks) and threads.  
- Control the processor state.  

## Where are devices mapped / originated in the kernel
- The DeviceTree is a (serialized) dictionary of devices with properties including base addresses of hardware and registers.  
- XNU uses the devicetree to map hardware into the kernel so that it can be accessed.  
- In early stages, XNU uses `ml_io_map` to map hardware (physical) memory into the kernel.  
- Later XNU will use IOMemoryDescriptor and IOMemoryMap from IOKit to translate, map and access memory.  \
- For translation of physical and virtual addresses, XNU has helper functions: ptovirtk and virtktop.
- Instead of using these, one can translate addresses directly via the MMU too, by jumping to the matching ARMv8 system instructions.  
- Most base addresses of devices can be found in the sourcecode of XNU or iBoot, such as: GPIO, VROM, SRAM and UART.  

## What can we do with physical memory
- Access mapped device registers.  
- Possibly map VROM and SRAM into the kernel for dumping purposes. (VROM is SecureROM / BootROM).  
- Control screen, interact with the NAND / NOR and Secure Enclave Processor (possibly).  

## Serial Output
- In the kernel there are several functions for serial logging of which the log can be read through a dcsd cable.  
- It is possible to craft primitives for printing text to the serial console, useful when debugging.  
- There are patchable flags for enabling serial panic logging.  
- It may be perhaps possible to enable the full serial console.  (WIP)

## Video Console
- XNU has a full VT100 on-display terminal integrated which even exits on production devices.  
- There exist patchable flags to enable this console. (WIP).
- There are functions for painting graphics: ```C vc_(...)``` and ```C gc_(...)```.  
- The video console seems to fall back to serial when the display is not available.  
- The console gets the screen properties from the device tree.  

## Highlevel JTAG / Kernel debugging
- If the serial console works and the serial keyboard is enabled, it perhaps may be possible to create a debug console.  
- XNU in production leaves out some debugging features, but many of them still exist and can be enabled or patched in a chain.  
- ml_dbgwrap functions are amazing for debugging, but do require some patches and chaining to work in most cases.  
- ml_dbgwrap_halt_cpu_with_state can halt the CPU for any nanoseconds and restore the execution to a given threadstate (struct of registers) when the timer runs out.  
- It may be possible to perhaps disable pagefaulting.  
- It may be possible to perhaps bind execution to a specific CPU core and make a debugger on top of the serial console.  
- ```C ml_dbgwrap_halt_cpu_with_state()``` checks if halt is supported on the CPU core, which is not the case on production devices. We can patch the ```C cpu_datap()``` structure to enable halt.  
- Can we access syscfg partition and turn the device into a PVT (prototype) stage device and possibly unlock the possibility for debug kernels and boot.  

## Mac policies and CSR
- CSR is what Apple calls rootless and what is generally more known as SIP (System Integrity Protection).  
- SIP does exist in XNU on iOS, but many have not noticed. For example one can see that creating /AppleInternal is prohibited by CSR and will fail.  
- A bypass renaming another vnode to /AppleInternal is not enough, I tried this but the directory does not change on the actual filesystem eventhough the vnode gets created.  
- Any mac policy on the system can be patched and thus enabled or disabled, but patching vnode_enforce still does not allow the creation of ```/AppleInternal```.  
- CSR operates on flags, which may mean that it is possible to disable it by patching that flags.  

## Sysctls
- Sysctls are configuration properties for the kernel with domains such as hardware, memory, kernel and security.  
- mac policies are also sysctls.  
- The boot-arguments in the NVRAM are not the same as the boot-arguments in the sysctl, is it possible to patch this?  
- Is it possible to enlarge swap memory size with a sysctl patch for improving older device performance.  
- It is impossible to patch the kernel versionstring, as it resides in the KTRR (static Read-Only) region of the kernel.

## ARMv8 System Instructions
- Many system instructions exist in the kernel.  
- AMCC on memory protection enforced devices will not allow the execution of system instructions, a special segment with enforced mappings makes sure of that.  
- Accessing CPACR triggers the kernel patch protection (KPP / Apple WatchTower) on older devices, a bypass can be implemented as presented by Luca Todesco and the firt-level pagetable entries can then be patched without any problem.  
- On my iPhone 6S (N71AP) KPP did not prevent the setting of the TTBR (translation table baseregister), which means pointing to fake pagetables may be possible for the time cycle that KPP is not there to check.  
- KTRR devices with memory protection are hard to gain full page access on, but is it maybe possible to forge a CPU state during exception handling (through a callback functionpointer for example) setting the MMU to be turned off?  
- Is it perhaps possible to map the CPU's system registers to kernel memory and directly access them through physical memory write operations, enabling the creation of fake kernel pages or MMU to be turned off.  
- Is it mandatory to bypass KPP for kload purposes, as there is a rough delay of 10 minutes before KPP kicks in. Which is enough to flush the pagetables, craft fake pages and bootstrap a secondary bootstage.  

