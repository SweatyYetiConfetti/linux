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

- set_task_stack_end_magic(&init_task)


