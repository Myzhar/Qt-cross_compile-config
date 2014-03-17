Qt 5.2.1 on PandaBoard ES
=========================

[Original source: http://qt-project.org/wiki/TIPandaBoard]

These instructions only cover running Qt in a single-window, fullscreen fashion without X11.

Acquire an image
----------------
There are many ways to build or get an Ubuntu disk image – you can install Ubuntu server, for example. 
Or, you might download a pre-built image from Linaro (or write one with linaro-media-create). 
Here we will use Ubuntu Core and walk you through the process. For more details and other fun PandaBoard stuff, 
please visit omapedia [omappedia.com], from which much of this information was borrowed.

Get qemu-utils:
apt-get install qemu-utils
And create an image
qemu-image create disk.img 1G
1G means 1 gigabyte. Feel free to choose a different size!
Let’s create a few partitions. We need a small partition (32mb in the example) for the bootloader, and the rest of the card can be ext4:
# Partition the image
printf ",32,C,*\n,,L\n\n\n" | sfdisk -uM -D disk.img
# Format boot partition
sudo losetup /dev/loop0 disk.img -o 32256 --sizelimit 41094144
sudo mkfs.vfat -F 32 -n "bootfs" /dev/loop0
sudo losetup -d /dev/loop0
# Format root partition
sudo losetup /dev/loop0 disk.img -o 41126400
sudo mkfs.ext4 -L "rootfs" /dev/loop0
sudo losetup -d /dev/loop0
And now it’s time to put some files on our image. Let’s start with the Ubuntu core rootfs, available from here [cdimage.ubuntu.com].
# Mount the rootfs
mkdir rootfs
sudo mount -o loop,offset=41126400 disk.img rootfs
# Download & unpack Ubuntu Core
wget http://cdimage.ubuntu.com/ubuntu-core/releases/12.04.2/release/ubuntu-core-12.04.2-core-armhf.tar.gz
sudo tar --numeric-owner -xf ubuntu-core-12.04.2-core-armhf.tar.gz -C rootfs/
Now’s a good time to tweak the file system. Let’s start by installing the packages that we need in order to compile Qt.
# Get qemu-user-static so that we can chroot in
sudo apt-get install qemu-user-static
sudo cp /usr/bin/qemu-arm-static rootfs/usr/bin/
# Chroot in
sudo chroot rootfs
# Add the TI PPA to apt sources
printf "deb http://ppa.launchpad.net/tiomap-dev/release/ubuntu precise main\ndeb-src http://ppa.launchpad.net/tiomap-dev/release/ubuntu precise main" > /etc/apt/sources.list.d/ti.list
apt-key adv --recv-keys --keyserver keyserver.ubuntu.com B2E908737DB60AD5
# Also enable universe
sed -i 's/# \(.*\) universe$/\1 universe/g' /etc/apt/sources.list
# Better mount a few things before we start installing...
mount /dev
mount /dev/pts
mount /proc
# And update & upgrade
apt-get update
apt-get upgrade -y
# Install packages which we require for Qt
apt-get install libegl1-sgx-omap4 libgles2-sgx-omap4 libegl1-sgx-omap4-dev libgles2-sgx-omap4-dev libdrm-dev libwayland-dev libgbm-dev libffi-dev
# ...and anything else we might need, such as an SSH server
apt-get install netbase isc-dhcp-client openssh-server -y
# All done, clean up and get out
service udev stop
umount /proc
umount /dev/pts
umount /dev
exit
sudo rm rootfs/usr/bin/qemu-arm-static
Here are a few other things which may be handy for development:
# Enable serial console
cat > ttyO2.conf <<EOF
start on stopped rc or RUNLEVEL=[2345]
stop on runlevel [!2345]
 
respawn
exec /sbin/getty -L -8 115200 ttyO2
EOF
sudo mv ttyO2.conf rootfs/etc/init/
sudo chmod +x rootfs/etc/init/ttyO2.conf
 
# Allow root to login without a password
sudo sed -i 's/root:\*/root:/' rootfs/etc/shadow
When we are done with the rootfs, unmount it:
sudo umount rootfs
Now it’s time to deal with the boot partition. There are other bootloaders out there, but U-boot is popular and easy to get working.
# Create a boot.script for U-boot
cat > boot.script <<EOF
setenv bootargs console=tty0 console=ttyO2,115200n8 root=/dev/mmcblk0p2 rw earlyprintk vram=48M omapfb.vram=0:24M,1:24M consoleblank=0
fatload mmc 0:1 0x80000000 uImage
bootm 0x80000000
EOF
# Compile the script for U-boot
mkimage -A arm -T script -C none -n "Boot Image" -d boot.script boot.scr
# Get the kernel and bootloader files from Ubuntu
wget http://ports.ubuntu.com/ubuntu-ports/dists/precise/main/installer-armhf/current/images/omap4/netboot/MLO
wget http://ports.ubuntu.com/ubuntu-ports/dists/precise/main/installer-armhf/current/images/omap4/netboot/u-boot.bin
wget http://ports.ubuntu.com/ubuntu-ports/dists/precise/main/installer-armhf/current/images/omap4/netboot/uImage
# Mount the boot partition
mkdir bootfs
sudo mount -o loop,offset=32256 disk.img bootfs
# And copy everything over
sudo cp MLO u-boot.bin uImage boot.scr bootfs/
# All done, unmount
sudo umount bootfs
Ok, now you’ve got an image. As long as it’s been properly unmounted and you should be able to dd it to an SD card.
