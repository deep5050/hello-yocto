## Building an out-of-tree kernel module from scratch

### Steps
1. `cd ~/poky` : Replace with your poky path
2. create a new layer. let say `meta-hello-yocto`: `bitbake-layers create-layer ./meta-hello-yocto`
3. initialize the build environment variables : `source oe-init-build-env`
4. `cd build/` if not already there.
5. add the newly created layer to the build-system: `bitbake-layers add-layer "../meta-hello-yocto"`. This should automatically update the `bblayers.conf` file under `build/conf`
6. `cd ~/poky/meta-hello-yocto`
7. remove the receipes-exaple directory :`rm -rf recipes-example/`
8. create a dir . let say `recepies-kernel`: `mkdir recipes-kernel`
9. `cd recipes-kernel/`
10. make a directory. let say hello-mod: `mkdir hello-mod`
11. `cd hello-mod`
12. create a file name, say `kernel-module-hello_0.1.bb` with the following content:
```
SUMMARY = "Example of how to build an external Linux kernel module"
LICENSE = "GPLv2"
LIC_FILES_CHKSUM = "file://COPYING;md5=12f884d2ae1ff87c09e5b7ccc2c4ca7e"

inherit module

SRC_URI = "file://Makefile \
           file://hello.c \
           file://COPYING \
          "

S = "${WORKDIR}"

# The inherit of module.bbclass will automatically name module packages with
# "kernel-module-" prefix as required by the oe-core build environment.

RPROVIDES_${PN} += "kernel-module-hello"
```
13. `mkdir files`
14. `cd files`
15. create a file named `hello.c` with the following content:
```c
/******************************************************************************
 *
 *   Copyright (C) 2011  Intel Corporation. All rights reserved.
 *
 *   This program is free software;  you can redistribute it and/or modify
 *   it under the terms of the GNU General Public License as published by
 *   the Free Software Foundation; version 2 of the License.
 *
 *   This program is distributed in the hope that it will be useful,
 *   but WITHOUT ANY WARRANTY;  without even the implied warranty of
 *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See
 *   the GNU General Public License for more details.
 *
 *   You should have received a copy of the GNU General Public License
 *   along with this program;  if not, write to the Free Software
 *   Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307 USA
 *
 *****************************************************************************/

#include <linux/module.h>

int init_module(void)
{
	printk("Hello World!\n");
	return 0;
}

void cleanup_module(void)
{
	printk("Goodbye Cruel World!\n");
}

MODULE_LICENSE("GPL");
```

16. create a file named `COPYING` (optional)

17. create a `Makefile`
```makefile
obj-m := hello.o

SRC := $(shell pwd)

all:
	$(MAKE) -C $(KERNEL_SRC) M=$(SRC)

modules_install:
	$(MAKE) -C $(KERNEL_SRC) M=$(SRC) modules_install

clean:
	rm -f *.o *~ core .depend .*.cmd *.ko *.mod.c
	rm -f Module.markers Module.symvers modules.order
	rm -rf .tmp_versions Modules.symvers

```
18. tree should be like this 
```
.
├── conf
│   └── layer.conf
├── COPYING.MIT
├── README
└── recipes-kernel
    └── hello-mod
        ├── files
        │   ├── COPYING
        │   ├── hello.c
        │   └── Makefile
        └── kernel-module-hello_0.1.bb

4 directories, 7 files

```

19. recheck the layers `bitbake-layers show-layers`

20. update the `poky/build/conf/local.conf` file by appending `MACHINE_ESSENTIAL_EXTRA_DEPENDS +="kernel-module-hello"`
21. compile the the code: `cd ~/poky/build && bitbake kernel-module-hello`
22. `bitbake package-index`
23. re-build the image ( in this example `core-image-sato`) :` bitbake core-image-sato`
24. run the image in QEMU: `runqemu qemux86-64` : (in this example I'm using `qemux86-64` , add `nographic` to the command if you are remotely logged into a machine )
25. `modprobe hello`
26. :zap:



