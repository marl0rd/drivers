# ==============================================================================
# Out-of-tree kernel module Makefile (LDD3-style)
#
# Key idea (two-pass / two-mode Makefile):
#   1) You run:      make            (in your module directory)
#      - This is a normal user-space "make" run.
#      - In this first pass, KERNELRELEASE is EMPTY.
#      - We do NOT compile anything ourselves here.
#      - We delegate the build to the kernel build system (kbuild) using:
#            $(MAKE) -C $(KERNELDIR) M=$(PWD) modules
#
#   2) Then the kernel build system runs make AGAIN for us
#      - kbuild sets KERNELRELEASE (e.g., "6.x.y-...") and other variables.
#      - It re-invokes make with M=<our module dir>, effectively "coming back"
#        to our directory (no copying of sources).
#      - Now our Makefile is being *included/parsed* in "kbuild mode".
#      - In this second pass we ONLY declare "what to build" via obj-m.
#      - kbuild already knows "how to build" modules (flags, config, linking,
#        symbol/versioning, etc).
#
# ==============================================================================

# Optional extra include path used by LDD examples:
#   - LDDINC points to ../include relative to this module dir.
#   - EXTRA_CFLAGS adds -I... for compiling this module.
# Note: newer kernels often prefer ccflags-y, but EXTRA_CFLAGS is still common
# in older examples and many setups.
LDDINC := $(PWD)/../include
EXTRA_CFLAGS += -I$(LDDINC)

ifeq ($(KERNELRELEASE),)

  # KERNELDIR points to the *target* kernel build directory used to compile
  # this module.
  #
  # - By default, it uses the build directory of the *currently running kernel*
  #   (uname -r), which is usually correct for modules you plan to load right now.
  #   On most distros, /lib/modules/<version>/build is a symlink to the kernel
  #   headers and build infrastructure for that kernel.
  # - The target kernel does NOT have to be the running kernel.
  # - To build a module for a different kernel (e.g. a newer kernel you built
  #   but have not booted yet), override KERNELDIR to point at that kernel’s
  #   prepared build tree.
  KERNELDIR ?= /lib/modules/$(shell uname -r)/build

  # PWD is the absolute path to this module directory.
  # We pass it to kbuild via M=$(PWD) so kbuild knows where our external
  # module sources live and where to put the build outputs.
  PWD := $(shell pwd)

modules:
	# This is the "handoff" to kbuild:
	#   -C $(KERNELDIR) : run make inside the kernel build directory
	#   M=$(PWD)        : tell kbuild our external module directory
	#   modules         : ask kbuild to build external modules
	#
	# This command triggers the SECOND make invocation where KERNELRELEASE
	# is set and our Makefile enters the else branch below.
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

modules_install:
	# Install the built .ko into /lib/modules/... and run depmod (as per kbuild).
	# Typically requires root privileges:
	#   sudo make modules_install
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:
	# Clean up common kbuild outputs generated in THIS module directory.
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions *.mod modules.order *.symvers Module.symvers .module-common*

.PHONY: modules modules_install clean

else

  obj-m := hello.o 

endif