# Contributor: Jonathan Liu <net147@gmail.com>
pkgname=kernel26-vserver-grsec
_kernelname=${pkgname#kernel26}
_basekernel=2.6.32
pkgver=${_basekernel}.28
pkgrel=5
_patchname="patch-${pkgver}-1-ARCH"
_vserver=2.3.0.36.29.6
_grsec=2.2.1
_timestamp=20110215
pkgdesc="The Linux Kernel and modules with Linux-VServer and grsecurity patchset"
arch=('i686' 'x86_64')
license=('GPL2')
groups=('base')
url="http://linux-vserver.org/"
backup=("etc/mkinitcpio.d/${pkgname}.preset")
depends=('coreutils' 'linux-firmware' 'module-init-tools' 'mkinitcpio')
install=$pkgname.install
source=(ftp://ftp.kernel.org/pub/linux/kernel/v2.6/linux-$_basekernel.tar.bz2
        ftp://ftp.kernel.org/pub/linux/kernel/v2.6/longterm/v2.6.32/patch-$pkgver.bz2
        libata-alignment.patch
        xen-hypervisor.patch
        undef.patch
        http://people.linux-vserver.org/~harry/patch-${pkgver}-vs${_vserver}-grsec${_grsec}-${_timestamp}.diff
        # the main kernel config files
        config config.x86_64
        # standard config files for mkinitcpio ramdisk
        $pkgname.preset)
optdepends=('crda: to set the correct wireless channels of your country')
md5sums=('260551284ac224c3a43c4adac7df4879'
         'fc8c36b4638d8384a5d26a50413a1d11'
         '30851deee235a7486bd408cbe8eb2d71'
         '4b53693735fa0e0f7142985739780e10'
         'dc30610b3e531e7ffa6b4cd121950215'
         'a4975629aff5f6175c90434eb60ccf3a'
         '953f53206c55eb7d80b281d4b7b52953'
         '1c15c7d757168876a919ad2c4e779812'
         '232661e2e74d134e30d0f3a854403d5a')

build() {
  KARCH=x86

  cd ${srcdir}/linux-${_basekernel}
   if [ "$_basekernel" != "$pkgver" ]; then
     # add latest kernel stable patch
     patch -Np1 -i ${srcdir}/patch-$pkgver
   fi

  # Add libata ATAPI alignment patch
  patch -Np1 -i ${srcdir}/libata-alignment.patch
  # Add Hypervisor compiling error patch
  patch -Np1 -i ${srcdir}/xen-hypervisor.patch
  # Add Linux-VServer patches and grsecurity patchset
  patch -Np1 -i ${srcdir}/patch-${pkgver}-vs${_vserver}-grsec${_grsec}-${_timestamp}.diff
  # Add GCC compile patch
  patch -Np1 -i ${srcdir}/undef.patch
  # Strip extra version
  sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION = |g' Makefile

  if [ "$CARCH" = "x86_64" ]; then
    cat ../config.x86_64 >./.config
  else
    cat ../config >./.config
  fi
  # Strip local version
  echo "-grsec" >./localversion-grsec
  # Add Linux-VServer to local version
  echo "-vs" >./localversion-vserver
  # get kernel version  
  make prepare
  _kernver="$(make kernelrelease)"
  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  # ... or manually edit .config
  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################
  yes "" | make config
  # build!
  make bzImage modules
  mkdir -p ${pkgdir}/{lib/modules,boot}
  make INSTALL_MOD_PATH=${pkgdir} modules_install
  cp System.map ${pkgdir}/boot/System.map26${_kernelname}
  cp arch/$KARCH/boot/bzImage ${pkgdir}/boot/vmlinuz26${_kernelname}
  install -D -m644 Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Makefile
  install -D -m644 kernel/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/kernel/Makefile
  install -D -m644 .config \
    ${pkgdir}/usr/src/linux-${_kernver}/.config
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include

  for i in acpi asm-{generic,x86} config linux math-emu media net pcmcia scsi sound trace video; do
    cp -a include/$i ${pkgdir}/usr/src/linux-${_kernver}/include/
  done

  # copy arch includes for external modules
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/x86
  cp -a arch/x86/include ${pkgdir}/usr/src/linux-${_kernver}/arch/x86/

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers ${pkgdir}/usr/src/linux-${_kernver}
  cp -a scripts ${pkgdir}/usr/src/linux-${_kernver}
  # fix permissions on scripts dir
  chmod og-w -R ${pkgdir}/usr/src/linux-${_kernver}/scripts
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/.tmp_versions

  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel

  cp arch/$KARCH/Makefile ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  if [ "$CARCH" = "i686" ]; then
    cp arch/$KARCH/Makefile_32.cpu ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  fi
  cp arch/$KARCH/kernel/asm-offsets.s ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel/

  # add headers for lirc package
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video
  cp drivers/media/video/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/
  for i in bt8xx cpia2 cx25840 cx88 em28xx et61x251 pwc saa7134 sn9c102 usbvideo zc0301; do
   mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
   cp -a drivers/media/video/$i/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
  done
  # add docbook makefile
  install -D -m644 Documentation/DocBook/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Documentation/DocBook/Makefile
  # add dm headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  cp drivers/md/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  # add inotify.h
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/linux
  cp include/linux/inotify.h ${pkgdir}/usr/src/linux-${_kernver}/include/linux/
  # add wireless headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  cp net/mac80211/*.h ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core
  cp drivers/media/dvb/dvb-core/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core/
  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/11194
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/
  cp include/config/dvb/*.h ${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/
  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  cp drivers/media/dvb/frontends/lgdt330x.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  cp drivers/media/video/msp3400-driver.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  # add dvb headers
  # in reference to:
  # http://bugs.archlinux.org/task/20402
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-usb
  cp drivers/media/dvb/dvb-usb/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-usb/
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends
  cp drivers/media/dvb/frontends/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/common/tuners
  cp drivers/media/common/tuners/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/common/tuners/
  # add xfs and shmem for aufs building
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/mm
  cp fs/xfs/xfs_sb.h ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs/xfs_sb.h
  # add headers vor virtualbox
  # in reference to:
  # http://bugs.archlinux.org/task/14568
  cp -a include/drm $pkgdir/usr/src/linux-${_kernver}/include/
  # add headers for broadcom wl
  # in reference to:
  # http://bugs.archlinux.org/task/14568
  cp -a include/trace $pkgdir/usr/src/linux-${_kernver}/include/
  # add headers for crypto modules
  # in reference to:
  # http://bugs.archlinux.org/task/22081
  cp -a include/crypto $pkgdir/usr/src/linux-${_kernver}/include/
  # add vmlinux
  cp vmlinux ${pkgdir}/usr/src/linux-${_kernver}
  # copy in Kconfig files
  for i in `find . -name "Kconfig*"`; do 
    mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/`echo $i | sed 's|/Kconfig.*||'`
    cp $i ${pkgdir}/usr/src/linux-${_kernver}/$i
  done
  # copy local version
  cp localversion-* $pkgdir/usr/src/linux-${_kernver}

  cd ${pkgdir}/usr/src/linux-${_kernver}/include && ln -s asm-$KARCH asm
  # add header for aufs2-util
  cp -a ${srcdir}/linux-$_basekernel/include/asm-generic/bitsperlong.h ${pkgdir}/usr/src/linux-${_kernver}/include/asm/

  chown -R root.root ${pkgdir}/usr/src/linux-${_kernver}
  find ${pkgdir}/usr/src/linux-${_kernver} -type d -exec chmod 755 {} \;
  cd ${pkgdir}/lib/modules/${_kernver} && \
    (rm -f source build; ln -sf ../../../usr/src/linux-${_kernver} build)
  # install fallback mkinitcpio.conf file and preset file for kernel
  install -m644 -D ${srcdir}/${pkgname}.preset ${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset
  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/g" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i ${srcdir}/../${pkgname}.install
  sed \
    -e "s|source .*|source /etc/mkinitcpio.d/kernel26${_kernelname}.kver|g" \
    -e "s|default_image=.*|default_image=\"/boot/${pkgname}.img\"|g" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/${pkgname}-fallback.img\"|g" \
    -i ${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset

  echo -e "# DO NOT EDIT THIS FILE\nALL_kver='${_kernver}'" > ${pkgdir}/etc/mkinitcpio.d/${pkgname}.kver
  # remove unneeded architectures
  rm -rf ${pkgdir}/usr/src/linux-${_kernver}/arch/{alpha,arm,arm26,avr32,blackfin,cris,frv,h8300,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,parisc,powerpc,ppc,s390,sh,sh64,sparc,sparc64,um,v850,xtensa}
  # remove the firmware
  rm -rf ${pkgdir}/lib/firmware
}

# vim:set ts=2 sw=2 et:
