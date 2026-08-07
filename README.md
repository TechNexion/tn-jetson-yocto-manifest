![video_orin-Banner-TEK(1) >](https://github.com/TechNexion-Vision/TEV-Jetson_Jetpack_script/assets/83322668/f699fae3-22a0-4eb0-9334-023286b953ca)

# TechNexion TEK-ORIN Yocto 6.0 Wrynose BSP Manifests

NOTE: *This BSP is a TechNexion release providing supprot nVidia Jetson Orin Nano series processors*

## System Requirements
Building an image from source with Yocto requires a host with the following:

- OS: native x86-64 Ubuntu 24.04 host
- CPU: 4 Core
- RAM: 8/16GB RAM (more is better)
- Storage: 400 GB
- SWAP space: 16GB
    - If less memory is used, then some additional swap space may be needed. Inadequate memory may result in slow builds and/or random build errors.
- Network
    - The host must be connected to a network and have access to the Internet so that all source code and tools may be downloaded.

## Set up build environment on host PC
```bash
sudo apt update
sudo apt install bash bmap-tools cpp device-tree-compiler gdisk libxml2-utils \
  python3 tar udisks2 usbutils zstd
```

## Prepare the desktop host
On an Ubuntu 24.04 desktop host, disable automatic mounting of removable media before flashing. The Orin flashing flow exposes storage over USB, and desktop automount can interfere with it:
```bash
gsettings set org.gnome.desktop.media-handling automount false
gsettings set org.gnome.desktop.media-handling automount-open false
```
NOTE: Avoid flashing from a virtual machine, container, WSL environment, or through an external USB hub. *Connect device to USB slot on host PC directly.*


## 🛠️ Prerequisites

Before getting started, ensure the Google `repo` tool is installed in your development environment and your SSH key is authorized on the internal Git server.

### 1. Install the Repo Tool
```bash
mkdir -p ~/.bin
PATH=~/.bin:$PATH
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+rx ~/.bin/repo
```

### 2. Configure Git Identity
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@technexion.com"
```

## 🚀 Download the BSP source
Follow these steps in your workspace directory (e.g., ~/work/tegra-bsp) to initialize and fetch the full source tree

- Create and Enter Workspace Directory, `tegra-demo-distro` as example.
```bash
mkdir -p ~/work/tegra-demo-distro
cd ~/work/tegra-demo-distro
```

- Initialize the repositories
```bash
repo init -u https://github.com/TechNexion/tn-jetson-yocto-manifest.git -b tn_l4t-r39.2.ga_kernel-6.8 -m jetson-r39.2_k6.8.xml
```

- Start to fetch source code
```bash
repo sync -j$(nproc)
```

## 🔧 Building the System
Table 1. Build configurations for supported hardware

|Parameter| Available options| Description|
|---|---|---|
|**MACHINE**|tn-tek7000-orin-nx| Nvidia Jetson Orin NX 16GB module in TEK-ORIN2 carrier|
|           |tn-tek7000-orin-nano| Nvidia Jetson Orin NANO 8GB module in TEK-ORIN2 carrier|

`$BUILDDIR` is specify the build directory, default is "build".


Initialize the Yocto build environment and launch BitBake.

```bash
cd tegra-demo-distro
. ./tn-setup-env -m $MACHINE $BUILDDIR
```
### For TEK7000-ORIN-NX
```bash
cd tegra-demo-distro
. ./tn-setup-env -m tn-tek7000-orin-nx build-tek7000-orin-nx
bitbake demo-image-full
```
### For TEK7000-ORIN-NANO
```bash
cd tegra-demo-distro
. ./tn-setup-env -m tn-tek7000-orin-nano build-tek7000-orin-nano
bitbake demo-image-full
```

## ⚡ Image Deployment

When the build completes, the generated release image tarball is `$BUILDDIR/tmp/deploy/images/$MACHINE/demo-image-full-${MACHINE}.rootfs.tegraflash-tar.zst`


### Step 1 - Unpack the tegraflash tarball

Unpack the `*.tegraflash-tar.zst` file that was created by your image build into an empty directory. 

For example with TEK7000-ORIN-NX:

```bash
mkdir -p $HOME/tegra-flashing
cd $HOME/tegra-flashing
tar xf $HOME/tegra-demo-distro/$BUILDDIR/tmp/deploy/images/tn-tek7000-orin-nx/demo-image-full-tn-tek7000-orin-nx.rootfs.tegraflash-tar.zst
```

### Step 2 - Put the target in recovery mode
Next, make sure your target device is in recovery mode, with the USB OTG port connected to your host machine. You can use the command
```bash
lsusb -d 0955:
```
to check this.

### Step 3 - Run the script for flashing
#### Flash all images
```bash
sudo ./initrd-flash
```
#### Only flash qspi. Also can update uefi firmware.
```bash
sudo ./initrd-flash --qspi-only
```
#### Only flash external storage on target device
```bash
sudo ./initrd-flash --external-only
```

<br/>

## More information
Please visit [TechNexion Wiki](https://developer.technexion.com/docs/embedded-software/linux/nvidia-jetpack/) for TEK-ORIN device.