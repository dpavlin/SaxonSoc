
Clone and build

```
# Getting this repository
mkdir Ulx3sSmp 
cd Ulx3sSmp
git clone https://github.com/SpinalHDL/SaxonSoc.git -b dev-0.1 --recursive SaxonSoc

# Sourcing the build script
source SaxonSoc/bsp/radiona/ulx3s/smp/source.sh

# Clone opensbi, u-boot, linux, buildroot, openocd
saxon_clone

# Build the FPGA bitstream
saxon_standalone_compile bootloader CFLAGS_ARGS="-DSDRAM_TIMING=MT48LC16M16A2_6A_ps"
saxon_netlist
saxon_bitstream

# Build the firmware
saxon_opensbi
saxon_uboot
saxon_buildroot

# Build the programming tools
saxon_standalone_compile sdramInit CFLAGS_ARGS="-DSDRAM_TIMING=MT48LC16M16A2_6A_ps"
saxon_openocd
```

If you want once to update the repo, you can do a :

```sh
saxon_update
```

Customize SDRAM_SIZE, FPGA_SIZE and CPU_COUNT for blue ULX3S board with 85F and 64Mb SDRAM

```sh
saxon_standalone_compile bootloader CFLAGS_ARGS="-DSDRAM_TIMING=AS4C32M16SB_7TCN_ps"
SDRAM_SIZE=64 CPU_COUNT=4 saxon_netlist
FPGA_SIZE=85 saxon_bitstream
```

To activate additional CPUs you will also need to edit dts file to enable additional cores
under cpus and modify interrupt-controller before running saxon_buildroot

```sh
vi buildroot/board/spinal/saxon_ulx3s/dts
```

Flash SPI 

```sh
source SaxonSoc/bsp/radiona/ulx3s/smp/source.sh
cd $SAXON_ROOT
fujprog -j FLASH SaxonSoc/hardware/synthesis/radiona/ulx3s/smp/bin/toplevel.bit
fujprog -j FLASH -f 0x340000 opensbi/build/platform/spinal/saxon/radiona/ulx3s/firmware/fw_jump.bin
fujprog -j FLASH -f 0x380000 u-boot/u-boot.bin
```

Flash sdcard 
```sh
export SDCARD=???
export SDCARD_P1=???
export SDCARD_P2=???

source SaxonSoc/bsp/radiona/ulx3s/smp/source.sh
cd $SAXON_ROOT

(
echo o
echo n
echo p
echo 1
echo
echo +100M
echo y
echo n
echo p
echo 2
echo
echo +200M
echo y
echo t
echo 1
echo b
echo p
echo w
) | sudo fdisk $SDCARD

sudo mkfs.vfat $SDCARD_P1
echo Y | sudo  mke2fs $SDCARD_P2

mkdir -p sdcard
sudo mount $SDCARD_P1 sdcard
sudo cp buildroot/output/images/dtb sdcard/dtb
sudo cp buildroot/output/images/rootfs.cpio.uboot sdcard/rootfs.cpio.uboot
sudo cp buildroot/output/images/uImage sdcard/uImage
sudo umount sdcard
rm -r sdcard

mkdir -p sdcard
sudo mount $SDCARD_P2 sdcard
sudo tar xf buildroot/output/images/rootfs.tar -C sdcard
sudo umount sdcard
rm -r sdcard
```
