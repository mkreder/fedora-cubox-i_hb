Fedora on Cubox-i and Hummingboard
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
- 2D accelertion. 3D acceleration is more or less working.

Using a preconfigured image
--------------
Download the image

Fedora 22:

    wget https://googledrive.com/host/0B0vm64JM4bFZbVdvMU9tZzQ2cWc -O Fedora-Minimal-armhfp-cubox-i_hb-22-3-sda.raw.xz
    
Fedora 21: 

    wget https://googledrive.com/host/0B0vm64JM4bFZN3RIOUZoT3Y5RnM -O Fedora-Minimal-armhfp-cubox-i_hb-21-5-sda.raw.xz
    
Fedora 20: 

    wget https://googledrive.com/host/0B0vm64JM4bFZcjNjakNIZG9CMnM -O Fedora-Minimal-armhfp-cubox-i_hb-20-1-sda.raw.xz

md5sums:

    696e6534839a37a090e730b87eebbbfd  Fedora-Minimal-armhfp-cubox-i_hb-22-3-sda.raw.xz
    c41ed4b7aec0c8e7de7999c9244e9098  Fedora-Minimal-armhfp-cubox-i_hb-21-5-sda.raw.xz
    62a5866aa1d6d72b21901dd7f962f482  Fedora-Minimal-armhfp-cubox-i_hb-20-1-sda.raw.xz

Write the image to your media

    xzcat <fedora-arm-image> > /dev/<location-of-your-fedora-arm-media>

Extend the root partition to fill your media if you wish

Fedora 22:

    fdisk /dev/<location-of-your-fedora-arm-media> <<EOF
    d
    3
    n
    p

    1087488

    w
    EOF
    partprobe /dev/<location-of-your-fedora-arm-media>
    e2fsck -f /dev/<location-of-your-fedora-arm-media>3
    resize2fs /dev/<location-of-your-fedora-arm-media>3
 
Fedora 21:

    fdisk /dev/<location-of-your-fedora-arm-media> <<EOF
    d
    3
    n
    p

    1251328

    w
    EOF
    partprobe /dev/<location-of-your-fedora-arm-media>
    e2fsck -f /dev/<location-of-your-fedora-arm-media>3
    resize2fs /dev/<location-of-your-fedora-arm-media>3


Fedora 20:

    fdisk /dev/<location-of-your-fedora-arm-media> <<EOF
    d
    3
    n
    p

    1251954

    w
    EOF
    partprobe /dev/<location-of-your-fedora-arm-media>
    e2fsck -f /dev/<location-of-your-fedora-arm-media>3
    resize2fs /dev/<location-of-your-fedora-arm-media>3

Boot, login (root/fedora), and get up to date

    yum -y update

Or you may modify the Fedora 20 or 21 Minimal image yourself
--------------
You can use the script below:

    #Edit lines below
    export fedoraimx6image=Fedora-Minimal-armhfp-22-3-sda.raw.xz #or Fedora-Minimal-armhfp-21-5-sda.raw.xz or Fedora-Minimal-armhfp-20-1-sda.raw.xz
    export fedoraimx6release=22 #or 21 or 20
    export kernelversion=4.0.4-300.14.1.cuboxi_hb #use ver. at: http://repo.maltegrosse.de/fedora/$releasever/stable/armv7hl/
    export cuboxihbenvver=1-3.1 #use ver at: http://repo.maltegrosse.de/fedora/$releasever/common/noarch/
    export locationofyourfedoraarmmedia=sdzzzzzzz #BE CAREFUL. This script will overwrite the location you choose.
    export mediamountpoint=/mnt/fedoraimx6root
    #Edit lines above

    wget http://people.redhat.com/jmontleo/cubox-i_hb/u-boot-images/SPL
    wget http://people.redhat.com/jmontleo/cubox-i_hb/u-boot-images/u-boot.img
    wget http://mirror.nexcess.net/fedora/releases/${fedoraimx6release}/Images/armhfp/${fedoraimx6image}
    wget http://repo.maltegrosse.de/fedora/${fedoraimx6release}/common/noarch/cubox-i_hb-uenv-${cuboxihbenvver}.fc${fedoraimx6release}.noarch.rpm
    wget http://repo.maltegrosse.de/fedora/${fedoraimx6release}/stable/armv7hl/kernel-${kernelversion}.fc${fedoraimx6release}.armv7hl.rpm

    xzcat ${fedoraimx6image} > /dev/${locationofyourfedoraarmmedia}
    dd if=SPL of=/dev/${locationofyourfedoraarmmedia} bs=512 seek=2
    dd if=u-boot.img of=/dev/${locationofyourfedoraarmmedia} bs=1K seek=42

    partprobe /dev/${locationofyourfedoraarmmedia}
    mkdir ${mediamountpoint}
    mount /dev/${locationofyourfedoraarmmedia}3 ${mediamountpoint}
    mount /dev/${locationofyourfedoraarmmedia}1 ${mediamountpoint}/boot

    rm -f ${mediamountpoint}/var/lib/rpm/__*
    rm -f ${mediamountpoint}/boot/boot.*
    unlink ${mediamountpoint}/etc/systemd/system/multi-user.target.wants/initial-setup-text.service
    sed -i s@^root:\\*:@root:\\\$6\\\$VpqypThR\\\$QZF3tM8USR6bnIK.CQn3bnj0SU5VeStkKA56ZEtAoPCECe23RqPgWzafuoKGzdWzUz9z8ctjSEhHrVg63wzra0:@g ${mediamountpoint}/etc/shadow
    rpm -i --noscripts --ignorearch --root ${mediamountpoint} ./kernel-${kernelversion}.fc${fedoraimx6release}.armv7hl.rpm ./cubox-i_hb-uenv-${cuboxihbenvver}.fc${fedoraimx6release}.noarch.rpm

    depmod -ab ${mediamountpoint}/ ${kernelversion}.fc${fedoraimx6release}.armv7hl
    cp ${mediamountpoint}/boot/dtb-${kernelversion}.fc${fedoraimx6release}.armv7hl/imx6dl-cubox-i.dtb ${mediamountpoint}/boot/imx6dl-cubox-i.dtb
    cp ${mediamountpoint}/boot/dtb-${kernelversion}.fc${fedoraimx6release}.armv7hl/imx6dl-hummingboard.dtb ${mediamountpoint}/boot/imx6dl-hummingboard.dtb
    cp ${mediamountpoint}/boot/dtb-${kernelversion}.fc${fedoraimx6release}.armv7hl/imx6q-hummingboard.dtb ${mediamountpoint}/boot/imx6q-hummingboard.dtb
    cp ${mediamountpoint}/boot/dtb-${kernelversion}.fc${fedoraimx6release}.armv7hl/imx6q-cubox-i.dtb ${mediamountpoint}/boot/imx6q-cubox-i.dtb
    ln -sf vmlinuz-${kernelversion}.fc${fedoraimx6release}.armv7hl ${mediamountpoint}/boot/zImage
    wget http://people.redhat.com/jmontleo/cubox-i_hb/cubox-i_hb.repo -O ${mediamountpoint}/etc/yum.repos.d/cubox-i_hb.repo

    umount ${mediamountpoint}/boot
    umount ${mediamountpoint}
    rmdir ${mediamountpoint}

    if [ $fedoraimx6release == 22 ]; then
    fdisk /dev/${locationofyourfedoraarmmedia} <<EOF
    d
    3
    n
    p

    1087488

    w
    EOF
    fi

    if [ $fedoraimx6release == 21 ]; then
    fdisk /dev/${locationofyourfedoraarmmedia} <<EOF
    d
    3
    n
    p

    1251328

    w
    EOF
    fi

    if [ $fedoraimx6release == 20 ]; then
    fdisk /dev/${locationofyourfedoraarmmedia} <<EOF
    d
    3
    n
    p

    1251954

    w
    EOF
    fi

    e2fsck -f /dev/${locationofyourfedoraarmmedia}3
    resize2fs /dev/${locationofyourfedoraarmmedia}3

Boot and login (root/fedora)

To get blueooth support set up:

    yum -y install cubox-i_hb-brcm43xx-bluetooth 
    systemctl enable brcm43xx.timer
    systemctl start brcm43xx

To get mainline kernels on top of stable, enable the mainline repo in /etc/yum.repos.d/cubox-i_hb.repo

I also recommend installing yum-plugin-priorities on Fedora 20 and 21 to ensure you only get kernels from these repos since the generic Fedora kernels will possibly not boot and will likely be missing drivers. These repos have a slightly higher priority (98 vs. 99), so packages in them will be used before those from other repos regardless of verion-release-epoch. dnf recognizes priorities with no additional packages.

    yum -y install yum-plugin-priorities

The wireless driver prints ugly messages to the console every so often. You can suppress all of these except the first set at ~1 second after boot with the following command and a reboot.

    echo "kernel.printk = 1 4 1 7" > /etc/sysctl.d/10-printk.conf

For anyone using an rtl8192cu based wireless card on a Cubox-i or Hummingboard that does not have built in wireless there is a dkms 8192cu package that should improve stability
    yum -y install 8192cu; reboot


Get up to date

    yum -y update


X.org Config
--------------

To get the armada driver working requires some configuration:

    yum -y install xorg-x11-drv-armada

    cat >> /etc/X11/xorg.conf.d/10-device.conf << EOF
    Section "Device"
    Identifier      "Videocard0"
    VendorName      "Freescale"
    BoardName       "IMX6 SOC"
    Driver          "armada"
    Option          "AccelModule"           "etnadrm_gpu"
    Option          "UseGPU"                "TRUE"
    Option          "XvAccel"               "TRUE"
    Option          "XvPreferOverlay"       "FALSE"
    EndSection
    EOF
    cat >> /etc/X11/xorg.conf.d/10-monitor.conf << EOF
    Section "Monitor"
    Identifier "Builtin Default Monitor"
    EndSection
    EOF
    cat >> /etc/X11/xorg.conf.d/10-screen.conf << EOF
    Section "Screen"
    Identifier "Builtin Default fbdev Screen 0"
    Device "Builtin Default fbdev Device 0"
    Monitor "Builtin Default Monitor"
    EndSection
    EOF
    cat >> /etc/X11/xorg.conf.d/10-server.conf << EOF
    Section "ServerLayout"
    Identifier "Builtin Default Layout"
    Screen "Builtin Default fbdev Screen 0"
    EndSection
    EOF
    
If you want XFCE

    yum groupinstall "Xfce Desktop"
    cat >> /etc/sysconfig/desktop << EOF
    PREFERRED=startxfce4
    EOF

or LXDE
   
    yum groupinstall "LXDE Desktop"
    cat >> /etc/sysconfig/desktop << EOF
    PREFERRED=lxsession
    EOF   

Building your own u-boot
--------------
    git clone https://github.com/rabeeh/u-boot-imx6.git
    cd u-boot-imx6
    make mx6_cubox-i_config
    make
    sudo dd if=SPL of=/dev/<location-of-your-fedora-arm-media> bs=512 seek=2
    sudo dd if=u-boot.img of=/dev/<location-of-your-fedora-arm-media> bs=1K seek=42

Building your own kernel
--------------
    You can build using the SRPM from http://repo.maltegrosse.de/fedora/ for the appropriate release mainline or stable repo src directory.

    yum -y install yum-utils rpm-build
    yum-builddep -y ./<download-kernel-src-rpm>
    rpm -ivh  ./<downloaded-kernel-src-rpm>
    rpmbuild -bb ~/rpmbuild/SPECS/kernel.spec

Or you can clone the repo:

    git clone https://github.com/jmontleon/fedora-cubox-i_hb.git
    cd fedora-cubox-i_hb
    git checkout 3.18.1
    make oldconfig #Or make menuconfig if you want to change stuff, and so on.
    make zImage
    make dtbs
    sudo cp arch/arm/boot/zImage /boot
    sudo cp arch/arm/boot/dts/*{cubox-i,hummingboard}*.dtb /boot
    sudo cp System.map /boot
    make modules
    sudo make modules_install

Ensure you have a /boot/uEnv.txt if you haven't already:

    bootfile=zImage
    mmcargs=setenv bootargs root=/dev/mmcblk0p3 rootfstype=ext4 rootwait console=ttymxc0,115200n8 console=tty1 video=1920x1080M@60 rd.dm=0 rd.luks=0 rd.lvm=0 raid=noautodetect zswap.enabled=1 coherent_pool=1M quiet
