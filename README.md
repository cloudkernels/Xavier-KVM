## KVM for Nvidia AGX Xavier/Nano - Installation Guide

This guide provides instructions to enable KVM virtualization on the Jetson AGX Xavier/Nano.
The Jetson AGX Xavier incorporates an Nvidia Xavier t194 SoC with 8 Carmel processor cores.
The Carmel cores of the Xavier T194 provides full support for virtualization with ARMv8.1 VHE.

##### Requirements:
- An Nvidia Jetson AGX Xavier/Nano

##### On your Xavier/Nano:
```
# Grab build dependencies
sudo apt install build-essential libssl-dev bc

# Download the L4T source tarball
wget -c https://developer.nvidia.com/embedded/dlc/r32-3-1_Release_v1.0/Sources/T186/public_sources.tbz2

# Unpack the kernel tarball
tar -jxpf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar -jxpf kernel_src.tbz2

# Download patch and kernel config.
# If no branch is specified, then the latest L4T release is used.
git clone https://github.com/b-man/Xavier-KVM.git ~/Xavier-KVM

# Patch the device tree
cd hardware/
# for Xavier
patch -Np1 -i ~/Xavier-KVM/patches/hardware/0001-Enable-KVM-support-for-t194.patch
# for Nano
patch -Np1 -i ~/Xavier-KVM/patches/hardware/0001-Enable-KVM-support-for-t210.patch

# Patch the kernel
cd ../kernel/kernel-4.9
for i in ~/Xavier-KVM/patches/kernel/*; do patch -Np1 -i $i; done

# Copy the kernel config to the kernel source tree
cp ~/Xavier-KVM/config-4.9.140-kvm .config

# Build kernel components
make ARCH=arm64 Image -j8
make ARCH=arm64 dtbs -j8
make ARCH=arm64 modules -j8

# Install kernel modules and firmware
sudo make ARCH=arm64 modules_install -j8

# Install the kernel
# On AGX Xavier, cboot accepts unsigned kernels when Secure Boot is off, which is the default.
# for Xavier
sudo rm /boot/Image.sig
sudo rm /boot/tegra194-p2888-0001-p2822-0000.dtb
# for Nano
sudo rm /boot/Image
sudo rm /boot/tegra210-p3448-0000-p3449-0000-a02.dtb

sudo install -m 0644 arch/arm64/boot/Image /boot/
# for Xavier
sudo install -m 0644 arch/arm64/boot/dts/tegra194-p2888-0001-p2822-0000.dtb /boot/dtb/
# for Nano
sudo install -m 0644 arch/arm64/boot/dts/tegra210-p3448-0000-p3449-0000-a02.dtb /boot/dtb/
```

Reboot your Jetson AGX Xavier/Nano. After rebooting you can verify that virtualization is enabled
by running the following commands:

##### Command:
```
dmesg | grep -i 'CPU features:'
```
Expected output:
```
[    0.762599] CPU features: detected feature: Privileged Access Never
[    0.762615] CPU features: detected feature: LSE atomic instructions
[    0.762630] CPU features: detected feature: User Access Override
[    0.762646] CPU features: detected feature: Virtualization Host Extensions
[    0.762660] CPU features: detected feature: 32-bit EL0 Support
```
##### Command:
```
dmesg | grep -i kvm
```
Expected Output:
```
[    1.580251] kvm [1]: 16-bit VMID
[    1.580258] kvm [1]: VHE mode initialized successfully
[    1.580357] kvm [1]: vgic-v2@3884000
[    1.580972] kvm [1]: vgic interrupt IRQ1
[    1.581006] kvm [1]: virtual timer IRQ4
```
