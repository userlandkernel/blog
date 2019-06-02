# Unlocking XNU
This post describes my research on XNU and my strive to make most out of XNU with kernel taskport sendrights.  
There might not be a goal here, but there is a lot of findings that make development more fun.  


## What I use
- Alex digital cable for serial debugging  
- Jailbreak with host_special_port 4, on iOS 12.0.1 (unc0ver)  
- Xcode on a macOS High Sierra Virtual Machine  
- XNU Source code, updated live at https://github.com/UKERN-developers/darwin-xnu  
- iBoot / SecureROM source code, forked from a leak at https://github.com/UKERN-developers/iboot  
- Symbolicated kernels and kernel extensions, from https://github.com/userlandkernel/ios-unstripped-kexts  
- IDA Pro 7.1 with Diaphora plugin, for diffing and symbolicating the production kernel with kexts against the symbolicated ones.  
- Radare2 with my automated offset finding script for generating an importable header file with offsets.  

## Some things exlained before getting started
- XNU contains of three components: Mach (MUCK), BSD Kernel and IOKit.  
- IOKit has a tree of classes and registries.  
- Mach is like UDP, sending messages between processes.  
- BSD kernel provides core logic and network interfaces.  

## In general a kernel can
- Access devices and their RAM.  
- Map physical memory to virtual memory, via the pagemap and the processor's Memory Management Unit (MMU).  
- Control and create processes (tasks) and threads.  
- Control the processor state.  

## Where are devices and their physical memory at on iDevices
- In the DeviceTree, a simple serialized dictionary of devices with properties of under more the base addresses.  
- Can be mapped early through ml_io_map.  
- IOKit accesses this memory through IOMemoryDescriptors and IOMemoryMap.  
- There are helper functions for translating virtual and physical memory addresses ptovirtk, virtktop.  
- The MMU translation can be called directly as well, through the ARMv8 system instructions.  
- Most base addresses of devices can be found in the sourcecode of XNU or iBoot.  

## What can we do with physical memory
- Access mapped device registers.  
- Possibly map VROM and SRAM into the kernel so that we can dump it. (VROM is SecureROM / BootROM).  
- Control screen, interact with the NAND / NOR and Secure Enclave Processor (possibly).  

## Serial Output
- In the kernel there are several functions for logging to serial, which can be read out through a dcsd cable.  
- We can craft primitives for printing text to the serial console for debugging.  
- We can enable panic logging so that we can see when a panic occurs via serial.  
- Can we perhaps enable the full serial console?  

## Video Console
- XNU has a full VT100 on-display terminal integrated, even on production devices.  
- We can patch most of its flags so that it can be enabled.  
- There are functions for painting graphics: vc_(...) and gc_(...).  
- The video console seems to fallback to serial when the display is not available.  
- The console gets the screen properties from the device tree.

## Highlevel JTAG / Kernel debugging
- Let's assume getting serial output to work is not a problem, getting the serial video console to work neither is and we can enable the serial keyboard.  
- In production kernels ml_dbgwrap functions and some debugger functions are still there.  
- With ml_dbgwrap_halt_cpu_with_state we can halt the CPU for a desired number of nanoseconds and give a thread state it should use for restoring execution.  
- Can we disable pagefaulting and bind ourselves to one CPU core? Can we make a full debugger on top of the serial console?  
- ml_dbgwrap_halt_cpu_with_state checks whether the provided cpu core supports halt, on production devices halt seems unsupported.  
- Can we support halt by patching it to be enabled in the kernel?  
- Can we access syscfg partition and turn the device into a PVT (prototype) stage device and possibly unlock the possibility for debug kernels and boot.  

## Mac policies and CSR
- CSR is what Apple calls rootless and what most people know as SIP.  
- Many think SIP does not exist in XNU on iOS, but it does. Try creating /AppleInternal per example and it will fail.  
- Simply renaming an existing rootvnode to /AppleInternal did not work. It can be renamed but not directory shows up with that name.  
- Disabling vnode_enforce mac policy did nothing to allow this, but all mac policies (security hardenings) can be patched in-kernel.  
- Can we bypass CSR by simply patching CSR flags?  

## Sysctls
- Sysctls are configurations / properties for the kernel, some are for security and some are for general configuration.  
- Can we patch the boot-arguments in the sysctl?  
- Can we make older devices faster by simply setting more swap memory to be supported.  
- We cannot patch the kernel version string since it is the kernels static (Ro / KTRR) region.  

## ARMv8 System Instructions
- Plenty of them are there, but on AMCC hardware memory protection enforced devices this will be a problem.  
- Touch CPACR and KPP will come to hit you up on older devices, but we can still implement Luca Todesco's KPP Bypass and patch first-level pagetable entries.  
- I tried to set the translation table base register on my N71AP (iPhone 6S) and KPP was of no issue, it did work flawlessly.  
- On KTRR devices, can't we just hit the exception vector and override a callback functionpointer so that we can fake a CPU state during the backup and then recover execution.  
- Can't we just instead of executing an instruction map the SoC's CPU registers into the kernel and write to them via direct memory access.  
- Do we need to bypass KPP to kload, since we have 10 minutes time which is enough to flush the pages and bootstrap a secondary bootchain.  
