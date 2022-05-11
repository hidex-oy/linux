```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconfig
make -j6 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOADADDR=0x80000000 uImage dtbs modules deb-pkg
```
