# Android Q10 for imx8mm

## Prerequisites

### repo

Make sure that the necessary packages have been installed.
<pre>sudo apt-get install xz-utils make flex lib32z1 zip curl</pre>
Install <b>repo</b> tool:
<pre>mkdir ~/Android/{bin,}
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/Android/bin/repo
chmod a+x ~/Android/bin/repo
export PATH=~/Android/bin:${PATH}</pre>

### GCC
Download the tool chain for the A-profile architecture from [arm Developer GNU-A Downloads page](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads) and deploy it in a directory on your discretion (hereafter /opt)

Recommended tool chain is <b>gcc-arm-8.3-2019.03-x86_64-arm-eabi</b>
<pre>export AARCH64_GCC_CROSS_COMPILE=/opt/gcc-arm-8.3-2019.03-x86_64-aarch64-elf/bin/aarch64-elf-
export AARCH32_GCC_CROSS_COMPILE=/opt/gcc-arm-8.3-2019.03-x86_64-arm-eabi/bin/arm-eabi-</pre>

## Getting Android source code

### NXP sources

* Download the NXP Android sources from this location: [imx-android-10.0.0_2.5.0.tar.gz](https://www.nxp.com/webapp/sps/download/license.jsp?colCode=Q10.0.0_2.5.0_ANDROID_SOURCE&appType=file1&DOWNLOAD_ID=null)

* Untar the image:
  <pre>tar -C ~/Android -xf imx-android-10.0.0_2.5.0.tar.gz</pre>

* Download and setup source:
  <pre>source ~/Android/imx-android-10.0.0_2.5.0/imx_android_setup.sh
  export A=$(pwd)</pre>
* Follow the instructions in the <b><i>~/imx-android-10.0.0_2.5.0/README</i></b> and make sure that the build environment is ready.

For more details see [the NXP Android User Guide](https://www.nxp.com/docs/en/user-guide/ANDROID_USERS_GUIDE.pdf)

## CompuLab sources

* Clone the repository

  ```bash
  git clone https://github.com/compulab-yokneam/imx8mm-android-q10.git
  export C=$(pwd)/imx8mm-android-q10
  ```

## Apply the patches
* U-boot

  ```bash
  git -C ${A}/vendor/nxp-opensource/uboot-imx checkout -b compulab-imx8-android android-10.0.0_2.5.0
  git -C ${A}/vendor/nxp-opensource/uboot-imx am ${C}/vendor/nxp-opensource/uboot-imx/*.patch
  ```

* ATF

  ```bash
  git -C ${A}/vendor/nxp-opensource/arm-trusted-firmware/ checkout -b compulab-imx8-android android-10.0.0_2.5.0
  git -C ${A}/vendor/nxp-opensource/arm-trusted-firmware am ${C}/vendor/nxp-opensource/arm-trusted-firmware/*.patch
  ```
* Kernel

  ```bash
  git -C ${A}/vendor/nxp-opensource/kernel_imx checkout -b compulab-imx8-android android-10.0.0_2.5.0
  git -C ${A}/vendor/nxp-opensource/kernel_imx am ${C}/vendor/nxp-opensource/kernel_imx/*.patch

* Board

  ```bash
  git -C ${A}/device/fsl checkout -b compulab-imx8-android android-10.0.0_2.5.0
  git -C ${A}/device/fsl am ${C}/device/fsl/*.patch
  ```

* Peripherals

  ```bash
  git -C ${A}/hardware/broadcom/libbt checkout -b compulab-imx8-android android-10.0.0_2.5.0
  git -C ${A}/hardware/broadcom/libbt am ${C}/hardware/broadcom/libbt/*.patch
  ```

Set a desired machine configuration:

```bash
export MACHINE=ucm_imx8m_mini SOC=imx8mm
```

## Build Android

* Go to Android build directory and issue:

  ```bash
  cd ${A}
  source build/envsetup.sh
  lunch ${MACHINE}-eng
  ./imx-make.sh -j16 2>&1 | tee build-log.txt
  ```

## Create a bootable SD-card

* Use a uSD card reader and a uSD сard. Any commercially available micro-SD card of <b>16GB+</b> may be used for the installation.

* Plug the USB SD card reader into the host PC. Insert the micro-SD сard into the сard reader. From now we assume the device name of the MMC/SD card on your Linux PC is <b>/dev/sdX</b>.

* Prepare the uSD card:

  ```bash
  sudo dd if=/dev/zero bs=1M count=16 of=/dev/sdX
  sudo partprobe /dev/sdx
  ```

* In the Android build directory issue the following commands:

  ```bash
  export MACHINE=ucm_imx8m_mini SOC=imx8mm
  cd out/target/product/${MACHINE}
  sudo ./fsl-sdcard-partition.sh -f ${SOC} /dev/sdX
  ```

## What's next?
Boot Android using the uSD card as described here [Boot from the SD-card](https://mediawiki.compulab.com/w/index.php?title=UCM-iMX8M-Mini:_Android:_Running_from_SD_card)

## Known issues:
* Wi-Fi interface is disabled in GUI
