# U-Boot: Orange Pi 4
######################
# Current Maintainer: endlesseden <https://github.com/EndlessEden/>
######################
#
# Original Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>
# Original Maintainer: Joseph Kogut <joseph.kogut@gmail.com>
# 
######################
# NOTE: Much of this is now adapted from Manjaro-Arm (https://gitlab.manjaro.org/manjaro-arm/packages/core/uboot-orangepi4-lts/-/blob/main/PKGBUILD)
######################
buildarch=8
opi4ver=2 # 1: OrangePI 4 | 2: OrangePI 4 LTS

if [ "$opi4ver" == "1" ]; then
	pkgname=uboot-orangepi4
	pkgdesc="U-Boot for Orange Pi 4"
elif [ "$opi4ver" == "2" ]; then
	pkgname=uboot-orangepi4-lts
	pkgdesc="U-Boot for Orange Pi 4 LTS"
fi
pkgver=2023.01
pkgrel=2
arch=('x86_64' 'aarch64')
url='http://www.denx.de/wiki/U-Boot/WebHome'
license=('GPL')
backup=('boot/extlinux/extlinux.conf')
makedepends=('bc' 'git' 'dtc' 'python-setuptools' 'swig')
options=(!distcc !strip !ccache)
install=${pkgname}.install
_tfver="2.8"
_tfaver="2.8"
source=("https://ftp.denx.de/pub/u-boot/u-boot-${pkgver/rc/-rc}.tar.bz2"
	"https://github.com/ARM-software/arm-trusted-firmware/archive/v${_tfaver}.tar.gz"
	"0001-mmc-sdhci-allow-disabling-sdma-in-spl.patch"    # From list: https://patchwork.ozlabs.org/project/uboot/patch/20220222013131.3114990-3-pgwipeout@gmail.com/
        "0002-add-arm64-dts-rockchip-orangepi-4-lts.patch"
	"0003-add-arm64-dts-rockchip-orangepi-4.patch")
sha256sums=('69423bad380f89a0916636e89e6dcbd2e4512d584308d922d1039d1e4331950f'
            '42256fa354f32b09972e72e0570a0f73698785927f93163b1d1308c485fcb4a6'
            '7014c3f1ada93536787a4ce30b484dfe651c339391bd46869c61933825a0edcc'
            '48a737548fab5e95471ce9ae58becbb1351058f9b5ce8cb12c212de77bedb511'
	    'f348d1e7757395e1e6d639e39566cae1b48574dbdfbf7f2915715a7ad5b556d4')


PHOST="$CARCH"	
case "$CARCH" in	
    aarch64) makedepends+=('gcc-arm-none-eabi' 'gcc')	
            ;;	
    x86_64) source+=('gcc-arm-10.3-2021.07-x86_64-arm-none-eabi.tar.xz::https://developer.arm.com/-/media/Files/downloads/gnu-a/10.3-2021.07/binrel/gcc-arm-10.3-2021.07-x86_64-arm-none-eabi.tar.xz?rev=325890dc39394ec49a112e5a661f6497&hash=2BF5AF893AEBA55524D3E0F6A010ACE8'	
             'gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu.tar.xz::https://developer.arm.com/-/media/Files/downloads/gnu-a/10.3-2021.07/binrel/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu.tar.xz?rev=1cb9c51b94f54940bdcccd791451cec3&hash=A56CA491FA630C98F7162BC1A302F869')	
            sha256sums+=('45225813f74e0c3f76af2715d30d1fbebb873c1abe7098f9c694e5567cc2279c'
            '1e33d53dea59c8de823bbdfe0798280bdcd138636c7060da9d77a97ded095a84')          	
            _pkgarch="aarch64"	
            CARCH="aarch64"	
            CHOST="aarch64-none-linux-gnu-"	
            export PATH="${srcdir}/aarch64/bin:${PATH}"	
            export ARCH='aarch64'	
            export CROSS_COMPILE='aarch64-none-linux-gnu-'	
            ;;	
esac	

prepare() {
  cd ${srcdir}/u-boot-${pkgver/rc/-rc}

  patch -N -p1 -i "${srcdir}/0001-mmc-sdhci-allow-disabling-sdma-in-spl.patch"  	# RK3399 suspend/resume
  patch -N -p1 -i "${srcdir}/0002-add-arm64-dts-rockchip-orangepi-4-lts.patch"		# OrangePi 4 LTS support
  patch -N -p1 -i "${srcdir}/0003-add-arm64-dts-rockchip-orangepi-4.patch"              # OrangePi 4 support

  if [ -e ${srcdir}/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu.tar.xz ]; then	
    if [ -d ${srcdir}/aarch64 ]; then	
        rm -r ${srcdir}/aarch64 	
    fi	
    echo "Preparing Cross-Compilers"	
    cd ${srcdir}/	
    mkdir aarch64	
    cd aarch64	
    cp -r ../gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/* ./	
    cp -r ../gcc-arm-10.3-2021.07-x86_64-arm-none-eabi/* ./	
  fi	
}

build() {
  # Avoid build warnings by editing a .config option in place instead of
  # appending an option to .config, if an option is already present

  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS
  
  if [ "$PHOST" = "x86_64" ]; then	
    CPPFLAGS=""	
    CFLAGS=""	
    CXXFLAGS=""	
    LDFLAGS=""	
    export PATH="${srcdir}/aarch64/bin:${PATH}"	
  fi

  update_config() {
    if ! grep -q "^$1=$2$" .config; then
      if grep -q "^# $1 is not set$" .config; then
        sed -i -e "s/^# $1 is not set$/$1=$2/g" .config
      elif grep -q "^$1=" .config; then
        sed -i -e "s/^$1=.*/$1=$2/g" .config
      else
        echo "$1=$2" >> .config
      fi
    fi
  }

  if [ $opi4ver == "2" ]; then
	  echo -e "\nBuilding TF-A for OrangePi 4 LTS...\n"
	  cd ${srcdir}/arm-trusted-firmware-${_tfaver}
	  make PLAT=rk3399
	  cp build/rk3399/release/bl31/bl31.elf ../u-boot-${pkgver/rc/-rc}

	  cd ${srcdir}/u-boot-${pkgver/rc/-rc}

	  echo -e "\nBuilding U-Boot for OrangePi 4/4 LTS...\n"
	  make orangepi_4_lts_rk3399_defconfig


	  update_config 'CONFIG_IDENT_STRING' '"Arch Linux ARM"' 
	  update_config 'CONFIG_OF_LIBFDT_OVERLAY' 'y'
	  update_config 'CONFIG_SPL_MMC_SDHCI_SDMA' 'n'
	  update_config 'CONFIG_MMC_SDHCI_SDMA' 'y'
	  update_config 'CONFIG_MMC_SPEED_MODE_SET' 'y'
	  update_config 'CONFIG_MMC_IO_VOLTAGE' 'y'
	  update_config 'CONFIG_MMC_UHS_SUPPORT' 'y'
	  update_config 'CONFIG_MMC_HS400_ES_SUPPORT' 'y'
	  update_config 'CONFIG_MMC_HS400_SUPPORT' 'y'
	  update_config 'CONFIG_SYS_LOAD_ADDR' '0x800800'
	  update_config 'CONFIG_TEXT_BASE' '0x00200000'
	  update_config 'CONFIG_SPL_HAS_BSS_LINKER_SECTION' 'y'
	  update_config 'CONFIG_SPL_BSS_START_ADDR' '0x400000'
	  update_config 'CONFIG_SPL_BSS_MAX_SIZE' '0x2000'
	  update_config 'CONFIG_HAS_CUSTOM_SYS_INIT_SP_ADDR' 'y'
	  update_config 'CONFIG_CUSTOM_SYS_INIT_SP_ADDR' '0x300000'

	  make EXTRAVERSION=-${pkgrel}
  fi
  if [ $opi4ver == "1" ]; then
          echo -e "\nBuilding TF-A for OrangePi 4...\n"
          cd ${srcdir}/arm-trusted-firmware-${_tfaver}
          make PLAT=rk3399
          cp build/rk3399/release/bl31/bl31.elf ../u-boot-${pkgver/rc/-rc}

          cd ${srcdir}/u-boot-${pkgver/rc/-rc}

          echo -e "\nBuilding U-Boot for OrangePi 4...\n"
          make orangepi_4_rk3399_defconfig


          update_config 'CONFIG_IDENT_STRING' '"Arch Linux ARM"'
          update_config 'CONFIG_OF_LIBFDT_OVERLAY' 'y'
          update_config 'CONFIG_SPL_MMC_SDHCI_SDMA' 'n'
          update_config 'CONFIG_MMC_SDHCI_SDMA' 'y'
          update_config 'CONFIG_MMC_SPEED_MODE_SET' 'y'
          update_config 'CONFIG_MMC_IO_VOLTAGE' 'y'
          update_config 'CONFIG_MMC_UHS_SUPPORT' 'y'
          update_config 'CONFIG_MMC_HS400_ES_SUPPORT' 'y'
          update_config 'CONFIG_MMC_HS400_SUPPORT' 'y'
          update_config 'CONFIG_SYS_LOAD_ADDR' '0x800800'
          update_config 'CONFIG_TEXT_BASE' '0x00200000'
          update_config 'CONFIG_SPL_HAS_BSS_LINKER_SECTION' 'y'
          update_config 'CONFIG_SPL_BSS_START_ADDR' '0x400000'
          update_config 'CONFIG_SPL_BSS_MAX_SIZE' '0x2000'
          update_config 'CONFIG_HAS_CUSTOM_SYS_INIT_SP_ADDR' 'y'
          update_config 'CONFIG_CUSTOM_SYS_INIT_SP_ADDR' '0x300000'

          make EXTRAVERSION=-${pkgrel}
  fi
}

package() {
  cd ${srcdir}/u-boot-${pkgver/rc/-rc}

  mkdir -p "${pkgdir}/boot/extlinux"

  install -D -m 0644 idbloader.img u-boot.itb -t "${pkgdir}/boot"
}
