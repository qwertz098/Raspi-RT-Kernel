# RT-Kernel
How to compile 64-bit RT-kernel for Raspberry Pi 4/5 for Debian bookworm (i.e., /boot/firmware/)

## Prepare the environment
```bash
sudo apt install git bc bison flex libssl-dev make
sudo apt install libncurses5-dev
sudo apt install raspberrypi-kernel-headers

mkdir ~/kernel
```
## Clone the git, in this case kernel 6.10, from from https://github.com/raspberrypi/linux/tree/rpi-6.10.y
```bash
cd ~
git clone --depth 1 --branch rpi-6.10.y https://github.com/raspberrypi/linux
```
## Get the latest RT-patch from https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/, in this case RT11 for kernel 6.10-rc6, from https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/6.10/ respectively
```bash
cd ~/kernel
wget -c https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/6.10/patch-6.10-rc6-rt11.patch.xz
xz -d patch-6.10-rc6-rt11.patch.xz
```
## Go back into the cloned linux
```bash
cd ~/linux
```
## Undo prior patch, if necessary, in this case the one for 6.10-rc6-rt10 – please perform this only, if you've applied a patch before and the respective kernel patch is still available in ~/kernel!
```bash
#patch -R -p1 < ~/kernel/patch-6.10-rc6-rt10.patch
```
## Update if necessary while scrapping all your local stuff
```bash
git stash
git pull --rebase
#git stash clear
```
P.S.: If resetting and updating your local (git-) environment with the last two steps does not work for any reason, you can always run `sudo rm -rd ~/linux` to start from scratch @ https://github.com/by/RT-Kernel?tab=readme-ov-file#clone-the-git-in-this-case-kernel-610-from-from-httpsgithubcomraspberrypilinuxtreerpi-610y
## Or simply pull
```bash
#git pull
```
## Patch the kernel
```bash
patch -p1 < ~/kernel/patch-6.10-rc6-rt11.patch
```
## Make for Raspberry Pi 4/5
```bash
make bcm2711_defconfig #RPI4
#make bcm2712_defconfig #RPI5
```
## Start menuconfig
```bash
make menuconfig
```
## Configuring the Kernel (from: https://lemariva.com/blog/2021/08/raspberry-pi-rt-preempt-tutorial-for-kernel-4-14-y)
### OPTION 1: Edit ```.config```
```
nano .config
```
and search for the options using CTRL+W and set
```
HIGH_RES_TIMERS=y
CONFIG_PREEMPT_RT_FULL=y
CONFIG_HZ_1000=y
CONFIG_HZ=1000
```
Do not insert spaces!

### OPTION 2: Graphical Settings 
- CONFIG_PREEMPT_RT_FULL: Kernel Features → Preemption Model (Fully Preemptible Kernel (RT)) → Fully Preemptible Kernel (RT)
- Set CONFIG_HZ to 1000Hz: Kernel Features → Timer frequency = 1000 Hz
- Enable HIGH_RES_TIMERS: General setup → Timers subsystem → High Resolution Timer Support
After changing these options, don't forget to save the .config file (option Save) and then exit.


## Build the kernel using all 4 cores (and try gcc optimization level -O3, if you like)
```bash
make prepare
make -j4 Image.gz modules dtbs
make CFLAGS='-O3 -march=native' -j4 Image.gz modules dtbs
sudo make modules_install
```
## Create the required directories once
```bash
sudo mkdir /boot/firmware/NTP
sudo mkdir /boot/firmware/NTP/overlays-NTP
```
## Add this to /boot/firmware/config.txt in order to preserve the standard kernel
```bash
os_prefix=NTP/
overlay_prefix=overlays-NTP/
kernel=/kernel_2712-NTP.img
```
## Copy the file ino the right directories
```bash
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/NTP/; sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/NTP/overlays-NTP/; sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/NTP/overlays-NTP/; sudo cp arch/arm64/boot/Image.gz /boot/firmware/kernel_2712-NTP.img
```
## Reboot to activate the kernel
```bash
sudo reboot now
```
