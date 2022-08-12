## Set the version (just to make this example less pseudo code)
```bash
export KERNELVERSION=4.19.94-ti-r73-hidex.10
````

**Note:** The build command seems to append a `+` at the end of the name when using the
`EXTRAVERSION` field in the `Makefile` (see below). That's why all the commands have that
plus at the end of the path, after the kernel version.

## Kernel configuration (if needed)

```bash
cd <path/to/this/repo/dir>

# First copy the pre-configured config file to place:
cp -pn config-${KERNELVERSION} .config

# If you need to change the kernel config:
make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

## Kernel build instructions

**Note:** The "extra version" suffix is set in the `Makefile`.
```Makefile
EXTRAVERSION = -ti-r73-hidex.10
```

Compile the kernel, modules an Device Tree blob files:

```bash
cd <path/to/this/repo/dir>

# First copy the pre-configured config file provided in the repo
# into place (if not done above already):
cp -pn config-${KERNELVERSION} .config

# Compile the kernel code and modules
make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

# This command should be redundant to run separately
#make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules

# Install the kernel modules (optionally to a temporary location at first).
# Here the path can be a temporary directory, from which you will then
# copy the contents to the final location at /lib/modules/<version>/
# on the SD card (or rather to '/' since it has the full path from root inside the directory).
make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_STRIP=1\
    INSTALL_MOD_PATH=/path/to/bbb_modules_dir modules_install

# Build the final kernel image and the Device Tree file for the board.
# (Building the dtb here seems to do nothing, the above commands already built all of them.)

# This is also redundant, since the zImage file is actually being used
# instead of the uImage, and the above make command builds the zImage already.
# make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOADADDR=0x80000000 uImage am335x-boneblack-uboot-univ.dtb

# Or alternatively this to build all the Device Tree files:
# make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOADADDR=0x80000000 uImage dtbs
```

## Kernel and modules and Device Tree file installation

**Note:** The following assumes `/media/sd` to be the mounted SD card!

```bash
# Install the kernel and the modules and the Device Tree file to the SD card.
cp -p arch/arm/boot/zImage /media/sd/boot/vmlinuz-${KERNELVERSION}+
cp -a /path/to/bbb_modules_dir/lib/modules/${KERNELVERSION}+ /media/sd/lib/modules/

# Copy over the Device Tree file
mkdir -p /media/sd/boot/dtbs/${KERNELVERSION}+
cp -p arch/arm/boot/dts/am335x-boneblack-uboot-univ.dtb /media/sd/boot/dtbs/${KERNELVERSION}+/

# Or copy all of them:
# cp -p arch/arm/boot/dts/*.dtb /media/sd/boot/dtbs/5.10.120-hidex.5+/
```

Finally edit the file /media/sd/boot/uEnv.txt and change the uname_r value to point to this kernel name/file.
```
uname_r=${KERNELVERSION}+
```
