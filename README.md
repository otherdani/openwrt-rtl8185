# openwrt-rtl8185
OpenWRT's source code with enable Realtek RTL8185L

This repository uses GitHub Actions to compile a custom OpenWrt x86_64 firmware image in the cloud with kernel support enabled for legacy Realtek RTL8180 / RTL8185 PCI wireless cards (CONFIG_RTL8180), along with the LuCI web interface.

## Why Build via GitHub Actions?
Zero Local Disk Space: Compiles entirely on GitHub's cloud servers without requiring 20+ GB of Linux build tools and source code on your Windows PC.
No WSL2 or Linux Setup Needed: Configure and trigger builds directly from your web browser.
Fast & Free: Public repositories get free build minutes, and GitHub's runners finish compiling in roughly 35–45 minutes.
Note: While the standard OpenWrt ImageBuilder only takes ~1 GB of space, it cannot compile missing kernel drivers like CONFIG_RTL8180. A full source build (automated here) is required.

## Repository Setup & Workflow
To set up or update the build workflow, ensure the file .github/workflows/build.yml contains the following configuration:
´´´
name: Build OpenWrt x86_64 with RTL8180/RTL8185

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout OpenWrt
        uses: actions/checkout@v4
        with:
          repository: openwrt/openwrt
          ref: openwrt-23.05

      - name: Install build dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y build-essential clang flex bison g++ gawk \
            gcc-multilib g++-multilib gettext git libncurses5-dev libssl-dev \
            python3-setuptools rsync swig unzip zlib1g-dev file wget

      - name: Update and install feeds
        run: |
          ./scripts/feeds update -a
          ./scripts/feeds install -a

      - name: Configure Target & RTL8180 Driver
        run: |
          cat << 'EOF' > .config
          CONFIG_TARGET_x86=y
          CONFIG_TARGET_x86_64=y
          CONFIG_TARGET_x86_64_DEVICE_generic=y
          CONFIG_PACKAGE_kmod-mac80211=y
          CONFIG_PACKAGE_kmod-cfg80211=y
          CONFIG_PACKAGE_kmod-eeprom-93cx6=y
          CONFIG_PACKAGE_wpad-basic-mbedtls=y
          CONFIG_PACKAGE_iwinfo=y
          CONFIG_PACKAGE_luci=y
          EOF
          echo "CONFIG_RTL8180=m" >> target/linux/x86/config-5.15
          make defconfig

      - name: Compile OpenWrt
        run: |
          make -j$(nproc) V=s

      - name: Upload Firmware
        uses: actions/upload-artifact@v4
        with:
          name: openwrt-x86-64-rtl8185
          path: bin/targets/x86/64/*.img.gz
´´´
## How to Run the Build

Trigger the Workflow:
Go to the Actions tab at the top of this GitHub repository.
Select Build OpenWrt x86_64 with RTL8180/RTL8185 from the left sidebar.
Click the Run workflow dropdown on the right and confirm by clicking Run workflow.

Download Your Firmware Image:

Wait approximately 35–45 minutes for the build to complete (indicated by a green checkmark).
Click on the completed workflow run.
Scroll down to the Artifacts section at the bottom and download openwrt-x86-64-rtl8185.
Extract the .zip archive to find your flashable .img.gz firmware file.

## Hardware Compatibility Note
Target Architecture: This workflow builds for 64-bit x86 (x86_64) systems.

32-bit CPUs: If your target PC with the PCI slot uses an older 32-bit-only processor (such as an early Intel Pentium 4 or Pentium M), the .config target lines in .github/workflows/build.yml will need to be adjusted to x86/generic before running the build.
