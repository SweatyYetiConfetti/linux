____________________________________________________________
____________________________________________________________

start_kernel is the first architecture agnostic call into the linux source code during the boot process. 

For this reason, it is the vital function call for initializing and configuring a linux system. 

Lets work through it line by line to gain a deeper understanding of the linux kernels internal operations.

The header file for start_kernel is found in the include/linux/start_kernel.h and is a small header file with a single function specificied start_kernel and the header files it includes 

start_kernel.h includes

``` extern asmlinkage void __init __noreturn start_kernel(void);``` 

Breaking down this function signature word by word gives
- extern => function assumed to be found elsewhere with resolution deferred to the linker

- asmlinkage => a macro which denotes a function callable from assembly language

- void => return type

- __init => a macro denoting the function is specifically used for booting process

- __noreturn => a macro denoting no return value

- start_kernel(void) => function name and parameters

____________________________________________________________
____________________________________________________________

``` #include <linux/linkage.h>``` 
- primarily used for managing function calling conventions in the context of assembly language and arch specific code
- it does this by defining 59 macros

``` #include <linux/init.h>``` 
- primarily used for defining macros and attributes related to booting and exiting processes of the kernel and its modules 
- important macros are the __init, __initdata and __exit macros 
- also defines the module_init and module_exit macros

Now into init/main.c we can find the start_kernel function source code
``` void start_kernel(void);``` 

- First defines two variables used later in the function
-- char *command_line;
-- char *after_dashes;
--- two string literals that will be used to construct a command line instruction

____________________________________________________________
____________________________________________________________

``` set_task_stack_end_magic(&init_task);``` 
- #include <linux/sched/task_stack.h> => kernel/fork.c
- inits a task stack with magic number sentinel
- provides end of stack notice 
- provides stack overflow detection

____________________________________________________________
____________________________________________________________

``` smp_setup_processor_id();``` 
- #include <linux/smp.h>
- hook provided in init/main.c
- makes use of __init and __weak macros
- this provides a basic implementation when an architecture specific implementation is not specified (utilized by linker)
- set up symmetric multi-processing
- not utilized in the x86 architecture
 
____________________________________________________________
____________________________________________________________

``` debug_objects_early_init();``` 
- #include <linux/debugobjects.h> => lib/debugobjects.c
- static inline hook specified in include/linux/debugobjects.h
- initializing the hash bucks and link the static object pool objects into the poll list
- essentially creating the underlying datastructures for the object tracker
- the kernel level debug objects object tracker are used to track the lifetime of kernel objects and validate operations on them

____________________________________________________________
____________________________________________________________

``` init_vmlinux_build_id();``` 
- #include <linux/buildid.h> => lib/buildid.c
- static inline hook specified in include/linux/buildid.h 
- compute and stash the running kernels build ID
- typically stored in the ELF file under section .note.gnu.build-id

____________________________________________________________
____________________________________________________________

``` cgroup_init_early();``` 
- #include <linux/cgroup.h> => kernel/cgroup/cgroup.c
- static inline hook specified in include/linux/cgroup.h
- initializes cgroups and any early subsystems that request early init
- the css are the cgroup subsystems

____________________________________________________________
____________________________________________________________

``` local_irq_disable();``` 
- #include <linux/sched.h> => #include <linux/irqflags.h>
- defined as macros in the include/linux/irqflags.h where it calls raw_local_irq_disable();
- raw_local_irq_disable defined as a macro in include/linux/irqflags.h which refers to arch_local_irq_disable();
- arch_local_irq_disable defined as static and always inline function in arch/x86/include/asm/irqflags and calls native_irq_disable() in same file
- native_irq_disable calls asm volatile embedding asm instruction "cli" to turn off interrupts and not to be altered by the compiler 
- disables cpu interrupts for part of the boot process
- disabling interrupts in for kernel space only
- following this function call, early_boot_irqs_disabled is set to true for future accounting

____________________________________________________________
____________________________________________________________

``` boot_cpu_init();``` 
- function signature defined in include/linux/cpu.h
- no hook defined in include/linux/cpu.h
- defined in kernel/cpu.c
- overview: activate the first processor
- obtain symmetric multiprocessor id
- set cpu online, active, present and possible
- set_cpu_online, set_cpu_active, set_cpu_present, set_cpu_possible(cpu, bool)
- set_cpu_online is found in kernel/cpu.c
- set_cpu_active, set_cpu_present, set_cpu_possible defined as macro in include/linux/cpumask.h
- all three of these macros make use of the assign_cpu macro defined in the same file
- if CONFIG_SMP => __boot_cpu_id = symmetric multiprocessor id

____________________________________________________________
____________________________________________________________

``` page_address_init();``` 
- imported into start_kernel via #include <linux/memblock.h> 
- function signature and macro defined in include/linux/mm.h
- source found in mm/highmem.c
- highmem.c defines a static page_address_slot struct array as page_address_htable
- iterates over the page_address_htable array and initializes the page_address_htables list head and spinlock

____________________________________________________________
____________________________________________________________

``` setup_arch(&command_line);``` 
- imported into start kernel via #include <linux/init.h>
- function signature defined in include/linux/init.h
- cpu arch specific source can be found in arch/x86/kernel/setup.c
- functionality depends on whether (U)EFI loader was successful
- make use of (U)EFI data structs and interfaces if successful
- efi init code enabled via global efi_enabled
- if 32 bit
-- memcpy new_cpu_data into boot_cpu_data
-- clone initial page table into swapper page dir
-- load swapper page dir into cr3 
-- cr3 is the x86 register that holds the current processes page directory base register (PDBR) aka root page table
-- flush translation lookaside buffer (tlb)
- if 64 bit
-- set boot_cpu_data.x86_phys_bits
- append boot loader command line to builtin command line
- copy boot_command_line into command_line
- point cmdline_p at command_line
- check for olpc ofw (one laptop per child - open firmware) - bootloader
- initialize the Interrupt Descriptor Table (IDT) with early traps
- initialize static const struct cpu_dev cpu_devs[index] with the cpudev
- loop over cpu_devs and point them at c_ident[index]
- make use of struct cpuinfo_x86 boot_cpu_data in early_identify_cpu to perform minimum cpu detection early
- fields needed in cpuinfo_x86 are vendor, cpuid_level, family, model, mask, cache_alignment
- only called during boot of CPU
- jump_label_init() for jump label, which is a mechanism for optimizing conditional branch conditions
- particularly useful for kernel tracing, hence its early initialization in the boot process especially in debugging and tracing
- initialize static call support that will allow code patching to hard-code function pointers into direct branch instructions
- setup temporary memory mappings to facilitate further booting until full ioremap is established
- map the one laptop per child open firmware page global directory memory region into the kernels address space - allowing for kernel interaction with OPLC OFW
- translate the fields of struct boot_param into global variables
- reserve some memory before memory is added to memblock so memblock allocations wont overwrite it when invoking e820
- everything still needed from the boot loader or firmware or kernel _text in elf file should be early reserved or marked not RAM
- all other memory is free and fair game in accordance with e820 table (reports memory usage to kernel)
- call e820__memory_setup to pick up the firmware/bootloader E820 map
- get boot_params struct, loop over the data and assign data to the memory in accordance with the type defined in a while loop with a switch statement
- copy BIOS Enhanced Disk Drive data from the boot_params into a safe place
- assign start_code, end_code, end_data and brk for the struct mm_struct init_mm
- configure the non-executable bit used to mark memory pages as non-executable - a security feature used to prevent all arbitrary pages from being executable
- called before parse_early_param to determine if HW supports the NX bit
- copy boot_command_line into tmp_cmdline and parse the early options
- check if (U)EFI boot is enabled
-- if (U)EFI is enabled
--- reserve memory ranges for use of (U)EFI 
- report the state of NX bit
- setup Advanced Programmable Interrupt Controller (APIC)
- after parse_early_params call e820__finish_early_params since we already have an e820 table filled in via the parameter callback functions, but needs to be sorted and printed via this function call
- if (U)EFI enable then efi_init()
-- setup UEFI runtime services and environment
- find, reserve memory region and conserve the iSCSI Boot Format Table
- initialize the desktop management interface via x86_init.resources.dmi_setup
- detect VMware (requires DMI) 
- initialize hypervisor platform by copying detected hypervisor vendor init and runtime, setting x86_hyper_type and init_platform()
- initialize the Time Stamp Counter (TSC) frequency and begin counting 
- probe BIOS read only memory
- use __pa_symbol macro to reach the physical address of code_resource, rodata_resource, data_resource and bss_resource
- insert the above physical addresses into iomem_resource and must be done before trim_bios_range
- call trim_bios_range and work out special case of 4Kb of memory that is BIOS owned, not by the kernel, general ignored detail in the e820 table
- if 32 bit system
-- if ppro_with_ram_bug()
--- then more explicit memory management to mitigate the bug
- if 64 bit system
-- run early_gart_iommu_check
--- Graphics Address Remapping Table (GART)
--- check for overlapping GART regions from a kernel that doesn't shutdown the GART properly via kexec/kdump
--- update the e820 map to mark new region to reserve
- get the highest page frame number available for the maximum architecture page frame number available on the system (partially used pages are not usable) 
- initialize the Memory Type Range Registers (MTRR) and Page Attribute Table (PAT) which are HW mechanisms used for caching in different memory regions 
- PAT is newer version of MTRR
- confirm that cache config is uniform across the system
- if mtrr_trim_uncached_memory(max_pfn) 
-- reobtain the max_pfn via e820__end_of_ram_pfn()
- perform Kernel Address Space Layout Randomization (KASLR) 
-- KASLR is a security feature that randomizes the memory address of kernel code and data at each boot to obfuscate memory addresses so reboots cannot be used during unauthorized memory access - remove predictable addressing
- if 32 bit system
-- find the lowest page frame number range
-if 64 bit system
-- check_x2apic - see if system is compatible with x2apic which is a more advanced version of apic for systems with a large number of cpus/threads
- reassign page frame number to max_low_pfn based on current max_pfn
- find multi processing configuration table (mptable) and reserve the memory region 
- allocate memory space for page table structs via early_alloc_pgt_buf
- must conclude brk before memblock setup, since there could be overlap 
- brk segment marks the end of the data section, specifically unitialized data segment (BSS) 
- cleanup the high memory address space before setting up memblock
- if CONFIG_MEMORY_HOTPLUG
-- Memory used by kernel cannot be hot-removed - no action or memblock_set_bottom_up = true
- since only first megabyte is mapped for certain, use it set a current limit up to ISA_END_ADDRESS
- allow for memblock to be resized for e820 table entries and loop over e820 table entries and reserve and add memblock for the table entry
- then trim the partial pages and get report on e820 table entry mapped memory region
- run memory encryption architecture setup after report on memory regions, must happen after memblock setup because it needs the physical memory size
- initialize RNG for confidential computing (CoCo/CC) 
- loop over the efi memory descriptions and mark the mirror, sum the size and output mirror size and total size
- remap the efi memory descriptions table, validate it, mark it reserved and unmap it
- ESRT is EFI System Resource Table (efi_esrt_init) and is a data structure that supports firmware updates and ops
- init EFI Machine Owner Keys by init on the MOK var table
- EFI MOKvar table must be initialized before boot services in order to guarantee that it can mark the table as reserved
- now the system can reserve memory regions for the boot services
- allocate 4k memblock for the e820 multi-process configuration
- if configured to check bios corruption, do so
- if 32 bit config then print initial memory mapped address
- find free memory for the real mode trampoline, on (U)EFI there is not enough free memory under 1M and make another attempt to reclaim the memory at efi_free_boot_services
- regardless, reserve entire first 1< of RAM because BIOS is known for corrupting low memory
- after init_mem_mapping is done with the early IDT page fault handling, you enable Flex Return and Event Delivery or install the real page fault handler
- update mmu cr4 features - mmu cr4 is a x86 register that enables memory management and virtualization extensions
- set the physical address of the current allocation limit
- if CONFIG_PROVIDE_OHCI1394_DMA_INIT then start the dma controllers
- test whether the EFI bits are enabled or false if the CONFIG_EFI macro is not set
- ensure that the init ram disk image containing drivers and tools for mounting real filesystem are properly loaded into memory
- consider CONFIG_ACPI_TABLE_OVERRIDE_VIA_BUILTIN_INITRD to customize the acpi table from the initrd
- checksum all the acpi tables, enumerate lapics (local acpi) and io-apics
- detect and initialize the handling of virtual symmetric multi-processing via vsmp_init
- if no io_delay_override specified then run desktop management interface check on system on dmi table
- check early platform quirks to detect apple computer systems 
- complete the initialization of acpi table and process the multiple apic description table (MADT) and HW ACPI mode initialization
- parse the SMP config data before the initmem_init 
- retrieve system hardware configs stored in the flattened device tree
- initialize memory structures: NUMA, APCI Static Resource Affinity Table (SRAT) and registering memblock to establish node specific structures (NODE_DATA)
- reserve areas for contiguous memory handling, reserving memory from early allocator, calling arch specific code once the early allocator has been activated and all other subsystems have already allocated/reserved memory
- if large pages (GB) features is set then reserve memory for huge table lookaside buffer and allocate the memory
- reserve memory for a kernel crash dump allowing a separate kernel to inspect prev crashed kernel data
- platform specific paging initialization call to setup the kernel pagetables and prepare accessors functions, callback must call paging_init(), called once after the direct mapping for physical memory is available
- init Kernal Address Sanitizer, a dynamic memory safety error detector in the kernel by setting up the shadow memory which tracks the allocated memory addresses and other data structures used to validate memory accesses and error reporting
- sync the back kernel address range by creating, setting up and updating the inital page tables used by the kernel during boot
- probe and signal the use of trusted boot framework
- setup the virtual system calls page, a memory mapped into every processes address space





____________________________________________________________
____________________________________________________________

- #include <linux/jump_label.h> => kernel/jump_label.c
- 

