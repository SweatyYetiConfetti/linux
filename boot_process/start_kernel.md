start_kernel is the first architecture agnostic call into the linux source code during the boot process. 

For this reason, it is the vital function call for initializing and configuring a linux system. 

Lets work through it line by line to gain a deeper understanding of the linux kernels internal operations.

The header file for start_kernel is found in the include/linux/start_kernel.h and is a small header file with a single function specificied start_kernel and the header files it includes 

start_kernel.h includes

> extern asmlinkage void __init __noreturn start_kernel(void);

Breaking down this function signature word by word gives
- extern => function assumed to be found elsewhere with resolution deferred to the linker

- asmlinkage => a macro which denotes a function callable from assembly language

- void => return type

- __init => a macro denoting the function is specifically used for booting process

- __noreturn => a macro denoting no return value

- start_kernel(void) => function name and parameters

> #include <linux/linkage.h>
- primarily used for managing function calling conventions in the context of assembly language and arch specific code
- it does this by defining 59 macros

> #include <linux/init.h>
- primarily used for defining macros and attributes related to booting and exiting processes of the kernel and its modules 
- important macros are the __init, __initdata and __exit macros 
- also defines the module_init and module_exit macros

Now into init/main.c we can find the start_kernel function source code
> void start_kernel(void) 

- First defines two variables used later in the function
-- char *command_line;
-- char *after_dashes;
--- two string literals that will be used to construct a command line instruction

> set_task_stack_end_magic(&init_task);
- #include <linux/sched/task_stack.h> => kernel/fork.c
- inits a task stack with magic number sentinel
- provides end of stack notice 
- provides stack overflow detection

> smp_setup_processor_id();
- #include <linux/smp.h>
- hook provided in init/main.c
- makes use of __init and __weak macros
- this provides a basic implementation when an architecture specific implementation is not specified (utilized by linker)
- set up symmetric multi-processing
- not utilized in the x86 architecture
 
> debug_objects_early_init();
- #include <linux/debugobjects.h> => lib/debugobjects.c
- static inline hook specified in include/linux/debugobjects.h
- initializing the hash bucks and link the static object pool objects into the poll list
- essentially creating the underlying datastructures for the object tracker
- the kernel level debug objects object tracker are used to track the lifetime of kernel objects and validate operations on them

> init_vmlinux_build_id();
- #include <linux/buildid.h> => lib/buildid.c
- static inline hook specified in include/linux/buildid.h 
- compute and stash the running kernels build ID
- typically stored in the ELF file under section .note.gnu.build-id

> cgroup_init_early();
- #include <linux/cgroup.h> => kernel/cgroup/cgroup.c
- static inline hook specified in include/linux/cgroup.h
- initializes cgroups and any early subsystems that request early init
- the css are the cgroup subsystems

> local_irq_disable();
- #include <linux/sched.h> => #include <linux/irqflags.h>
- defined as macros in the include/linux/irqflags.h where it calls raw_local_irq_disable();
- raw_local_irq_disable defined as a macro in include/linux/irqflags.h which refers to arch_local_irq_disable();
- arch_local_irq_disable defined as static and always inline function in arch/x86/include/asm/irqflags and calls native_irq_disable() in same file
- native_irq_disable calls asm volatile embedding asm instruction "cli" to turn off interrupts and not to be altered by the compiler 
- disables cpu interrupts for part of the boot process
- disabling interrupts in for kernel space only
- following this function call, early_boot_irqs_disabled is set to true for future accounting

> boot_cpu_init();
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

> page_address_init();
- imported into start_kernel via #include <linux/memblock.h> 
- function signature and macro defined in include/linux/mm.h
- source found in mm/highmem.c
- highmem.c defines a static page_address_slot struct array as page_address_htable
- iterates over the page_address_htable array and initializes the page_address_htables list head and spinlock

> setup_arch(&command_line);
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





- #include <linux/jump_label.h> => kernel/jump_label.c
- 

