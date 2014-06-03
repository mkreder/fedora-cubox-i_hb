Fedora 20 Cubox-i4Pro Kernel
==============

What works
--------------
- All 4 Cores and other basics
- Booting from MMC
- Serial Output
- HDMI Output
- eSata
- USB
- Sound (probably S/PDIF only without better X11 driver than fbdev)
- RTC
- Infrared
- Wireless
- Bluetooth

What does not work
--------------
- UHS for mmc0
- fbdev is working for X11, but a better driver would improve performance

Using a preconfigured image
--------------
Download the image

    wget https://googledrive.com/host/0B0vm64JM4bFZVkFWa0tpdzhHLVk -O Fedora-Minimal-armhfp-cubox-i4pro-20-1-sda.raw.xz

Write the image to your media

    xzcat Fedora-Minimal-armhfp-cubox-i4pro-20-1-sda.raw.xz > /dev/<location-of-your-fedora-20-arm-media>

Extend the root partition to fill your media if you wish

    fdisk /dev/<location-of-your-fedora-20-arm-media> <<EOF
    d
    3
    n
    p

    1251954

    w
    EOF
    e2fsck -f /dev/<location-of-your-fedora-20-arm-media>3
    resize2fs /dev/<location-of-your-fedora-20-arm-media>3

Boot, login (root/fedora), and get up to date

    yum -y update

Or you may modify the Fedora 20 Minimal image yourself
--------------
Download Fedora 20 Minimal, the u-boot images, and kernel

    wget http://mirror.nexcess.net/fedora/releases/20/Images/armhfp/Fedora-Minimal-armhfp-20-1-sda.raw.xz
    wget http://people.redhat.com/jmontleo/cubox-i4pro/u-boot-images/SPL
    wget http://people.redhat.com/jmontleo/cubox-i4pro/u-boot-images/u-boot.img
    wget http://people.redhat.com/jmontleo/cubox-i4pro/rpms/stable/armhfp/kernel-3.14.4-202.cubox_i4pro.fc20.armv7hl.rpm 

Write everything to the media, and perform some additional setup

    xzcat Fedora-Minimal-armhfp-20-1-sda.raw.xz > /dev/<location-of-your-fedora-20-arm-media>
    dd if=SPL of=/dev/<location-of-your-fedora-20-arm-media> bs=512 seek=2
    dd if=u-boot.img of=/dev/<location-of-your-fedora-20-arm-media> bs=1K seek=42
    partprobe /dev/<location-of-your-fedora-20-arm-media>
    mkdir /mnt/f20cuboxi4root
    mount /dev/<location-of-your-fedora-20-arm-media>3 /mnt/f20cuboxi4root
    mount /dev/<location-of-your-fedora-20-arm-media>1 /mnt/f20cuboxi4root/boot
    rm -f /mnt/f20cuboxi4root/var/lib/rpm/__*
    rm -f /mnt/f20cuboxi4root/boot/boot.*
    unlink /mnt/f20cuboxi4root/etc/systemd/system/multi-user.target.wants/initial-setup-text.service
    sed -i s@^root:\\*:@root:\\\$6\\\$VpqypThR\\\$QZF3tM8USR6bnIK.CQn3bnj0SU5VeStkKA56ZEtAoPCECe23RqPgWzafuoKGzdWzUz9z8ctjSEhHrVg63wzra0:@g /mnt/f20cuboxi4root/etc/shadow
    rpm -i --noscripts --ignorearch --root /mnt/f20cuboxi4root ./kernel-3.14.4-202.cubox_i4pro.fc20.armv7hl.rpm
    ln -sf uEnv.txt-3.14.4-202.cubox_i4pro.fc20.armv7hl /mnt/f20cuboxi4root/boot/uEnv.txt
    wget http://people.redhat.com/jmontleo/cubox-i4pro/cubox-i4pro.repo -O /mnt/f20cuboxi4root/etc/yum.repos.d/cubox-i4pro.repo
    umount /mnt/f20cuboxi4root/boot
    umount /mnt/f20cuboxi4root
    rmdir /mnt/f20cuboxi4root

    fdisk /dev/<location-of-your-fedora-20-arm-media> <<EOF
    d
    3
    n
    p

    1251954

    w
    EOF
    e2fsck -f /dev/<location-of-your-fedora-20-arm-media>3
    resize2fs /dev/<location-of-your-fedora-20-arm-media>3

Boot and login (root/fedora)

After reboot USB and other modules probably won't load. The easiest way to fix this is to run:

    rpm -ivh --force http://people.redhat.com/jmontleo/cubox-i4pro/rpms/stable/armhfp/kernel-3.14.4-202.cubox_i4pro.fc20.armv7hl.rpm
    
To get blueooth support set up:

    yum -y install cubox-i-brcm4329-bluetooth 

To get mainline kernels on top of stable, enable the mainline repo in /etc/yum.repos.d/cubox-i4pro.repo

I also recommend installing yum-plugin-priorities to ensure you only get kernels from these repos since the generic Fedora kernels will not boot. These repos have a slightly higher priority (98 vs. 99), so packages in them will be used before those from other repos regardless of verion-release-epoch.

    yum -y install yum-plugin-priorities

Finally, the wireless driver prints ugly messages to the console every so often. You can suppress all of these except the first set at ~1 second after boot with the following command and a reboot.

    echo "kernel.printk = 1 4 1 7" > /etc/sysctl.d/10-printk.conf

Building your own u-boot
--------------
    git clone https://github.com/rabeeh/u-boot-imx6.git
    cd u-boot-imx6
    make mx6_cubox-i_config
    make
    sudo dd if=SPL of=/dev/<location-of-your-fedora-20-arm-media> bs=512 seek=2
    sudo dd if=u-boot.img of=/dev/<location-of-your-fedora-20-arm-media> bs=1K seek=42

Building your own kernel
--------------
You can build using the SRPM:

    yum -y install yum-utils rpm-build
    yumdownloader --source kernel-3.14.4-202.cubox_i4pro.fc20
    yum-builddep -y kernel-3.14.4-202.cubox_i4pro.fc20.src.rpm
    rpm -ivh  kernel-3.14.4-202.cubox_i4pro.fc20.src.rpm
    rpmbuild -bb ~/rpmbuild/SPECS/kernel.spec

Or you can clone the repo:

    git clone https://github.com/jmontleon/fedora-20-cubox-i4pro.git
    cd fedora-20-cubox-i4pro
    make oldconfig #Or make menuconfig if you want to change stuff, and so on.
    make zImage
    make dtbs
    sudo cp arch/arm/boot/zImage /boot
    sudo cp arch/arm/boot/dts/imx6dl-hummingboard.dtb /boot
    sudo cp System.map /boot
    sudo su -
    cd /boot
    cat zImage imx6dl-hummingboard.dtb > zImage_dtb;  mkimage -A arm -O linux -T kernel -a 0x10008000 -e 0x10008000 -n vmlinuz-3.14.0 -C none -d zImage_dtb uImage
    exit
    make modules
    sudo make modules_install

Ensure you have a uEnv.txt if you haven't already:

    bootfile=/uImage
    mmcargs=setenv bootargs root=/dev/mmcblk0p3 rootfstype=ext4 rootwait console=ttymxc0,115200n8 console=tty1 video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32


