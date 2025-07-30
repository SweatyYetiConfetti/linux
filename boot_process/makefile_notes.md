____________________________________________________________
____________________________________________________________

ccflags-y := -fno-function-sections -fno-data-sections

**ccflags-y** specifies the compilation flags 
- Only applies to the kbuild makefile in which they are assigned

**fno-function-sections is** a compilation flag 
- Places all functions from a source file into a single combined ELF section within the object file
- Overrides the default **ffunction-sections** which specifies an individual ELF section for each function in the source

**fno-data-sections** is a compilation flag
- Places all data variables into a single combined ELF section within the object file
- Overrides the default **fdata-selections** which specifies an individual ELF section for each data variable in the source

____________________________________________________________
____________________________________________________________

obj-y := main.o version.o mounts.o
ifneq ($(CONFIG_BLK_DEV_INITRD),y)
obj-y += noinitramfs.o
else
obj-$(CONFIG_BLK_DEV_INITRD) += initramfs.o
endif
obj-$(CONFIG_GENERIC_CALIBRATE_DELAY) += calibrate.o
obj-$(CONFIG_INITRAMFS_TEST) += initramfs_test.o
obj-y += init_task.o

**obj-[ym]** - specifies goal definitions
- Main part/heartbeat of any kbuild Makefile
- Define the files to be built, special compilation options and subdirs to be entered recursively

**obj-y** - specifies built-in object goals
- Specifies object files for vmlinux in the $(obj-y) lists which depend on the kernel configuration
- Kbuild compiles all the $(obj-y) files then calls "$(AR) rcSTP" to merge these files into one built-in.a file
- built-in.a is a thin archive wout a symbol table and linked into vmlinux by scripts/link-vmlinux.sh
- Ordering of files in $(obj-y) matters, duplicates allowed but ignored
- Link order is significant because functions will be called during boot in the order they appear

**obj-m** - specifies loadable module goals
- Specifies object files which are built as loadable kernel modules
- The loadable module may be built from one or several source files
- If built from several sources then kbuild needs to know which object files you want to build your module from 
- Set it by $(<module_name>-y)
-- example
    obj-m += x.o
    x-y := a.o b.o c.o
    x-$(CONFIG_Y) += optional.o
- Kbuild will compile the objects listed in $(x-y) and then run "$(LD) -r" on the list of these files to generate x.o
- If CONFIG_Y is not set to y then the optional.o object file will be ignored

**CONFIG_BLK_DEV_INITRD** - configuration variable
- Defined in init/Kconfig as BLK_DEV_INITRD
- Assigned value in init/.kunitconfig as CONFIG_BLK_DEV_INITRD=y
- Assigned value in architecture specified in arch/x86/configs/*
- initramfs.c & noinitramfs.c are defined in the init dir and (no)initramfs.o is their associated object file

**CONFIG_GENERIC_CALIBRATE_DELAY** - configuration variable
- Defined in architecture specific Kconfig file - arch/x86/Kconfig
- calibrate.c is defined in the init dir and calibrate.o is their associated object file

**CONFIG_INITRAMFS_TEST** - configuration variable
- Defined in init/Kconfig as INITRAMFS_TEST
- Assigned value in init/.kunitconfig as CONFIG_BLK_DEV_INITRD=y
- initramfs_test.c is defined in the init dir and initramfs_test.o is their associated object file

____________________________________________________________
____________________________________________________________

mounts-y := do_mounts.o
mounts-$(CONFIG_BLK_DEV_RAM) += do_mounts_rd.o
mounts-$(CONFIG_BLK_DEV_INITRD) += do_mounts_initrd.o

**mounts-y** - specifies mounting config
- This will call the mount process for making a file system on a various memory devices
- mount /dev/example /path/to/location
- do_mounts.o is for general filesystems
- do_mounts_rd.o & do_mounts_initrd.o are for the rd filesystem
- initrd filesystem used exclusively during the boot process
- initrd stands for initial ram disk 
- This is a temporary root filesystem mounted by the boot loader which contains essential drivers and modules for further booting


**CONFIG_BLK_DEV_RAM** - configuration variable
- Defined in architecture specific file - arch/x86/kernel/setup.c
- Mentioned in BLK_DEV_INITRD Kconfig definition => Both together means support for initrd & the 15 Kbytes needed

____________________________________________________________
____________________________________________________________

smp-flag-$(CONFIG_SMP) := SMP
preepmt-flag-$(CONFIG_PREEMPT_BUILD) := PREEMPT
preepmt-flag-$(CONFIG_PREEMPT_DYNAMIC) := PREEMPT_DYNAMIC
preepmt-flag-$(CONFIG_PREEMPT_RT) := PREEMPT_RT

**smp-flag-y** - specifies symmetric multiprocessing config for CPU

**preempt-flag-y** - specifies preemption config for CPU

**CONFIG_SMP** - configuration variable
- Architecture specific variable
- Assigned value in arch/x86/configs/.*_defconfig 

**CONFIG_PREEMPT_DYNAMIC** - configuration variable
- allows choice of preemption strategy to be deferred until boot time where any mode but PREEMPT_RT can be selected
- architecture specific in arch/x86/configs/

**CONFIG_PREEMPT_RT** - configuration variable
- RT stands for Real-Time
- Two modes
-- Preemptible Kernel (Basic RT) 
--- Mainly used for testing and debugging of substitution mechanisms 
-- Fully Preemptible Kernel (RT)
--- All kernel code is preemptible except for a few select critical sections



____________________________________________________________
____________________________________________________________

build-version = $(or $(KBUILD_BUILD_VERSION), $(build-version-auto))
build-timestamp = $(or $(KBUILD_BUILD_TIMESTAMP), $(build-timestamp-auto))

____________________________________________________________
____________________________________________________________

filechk_uts_version = \
	utsver=$$(echo '$(pound)'"$(build-version)" $(smp-flag-y) $(preempt-flag-y) "$(build-timestamp)" | cut -b -64); \
	echo '$(pound)'define UTS_VERSION \""$${utsver}"\"

____________________________________________________________
____________________________________________________________

$(obj)/utsversion-tmp.h: FORCE
	$(call filechk,uts_version)

clean-files += utsversion-tmp.h

____________________________________________________________
____________________________________________________________

$(obj)/version.o: $(obj)/utsversion-tmp.h
CFLAGS_version.o := -include $(obj)/utsversion-tmp.h

____________________________________________________________
____________________________________________________________

include/generated/utsversion.h: build-version-auto = $(shell $(srctree)/scripts/build-version)
include/generated/utsversion.h: build-timestamp-auto = $(shell LC_ALL=C date)
include/generated/utsversion.h: FORCE
	$(call filechk,uts_version)

____________________________________________________________
____________________________________________________________

$(obj)/version-timestamp.o: include/generated/utsversion.h
CFLAGS_version-timestamp.o := -include include/generated/utsversion.h

____________________________________________________________
____________________________________________________________

