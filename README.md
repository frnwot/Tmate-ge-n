---



# OrangeFox Recovery Build Guide (Ubuntu + tmate + GitHub Actions)

This guide helps you build OrangeFox Recovery using a cloud-hosted Ubuntu environment powered by GitHub Actions and `tmate` SSH. You can use Termux or any terminal to connect and build.

---

## 1. GitHub Actions Setup (12-Hour Ubuntu Server with tmate)

Create this workflow in your repository at `.github/workflows/ubuntu-tmate.yml`:

```
name: Ubuntu tmate session

on:
  workflow_dispatch:

jobs:
  tmate-session:
    runs-on: ubuntu-latest
    timeout-minutes: 720

    steps:
      - name: Install tmate and ssh
        run: |
          sudo apt update
          sudo apt install -y tmate openssh-client

      - name: Generate SSH keys
        run: |
          ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ""

      - name: Start tmate session
        run: |
          tmate -S /tmp/tmate.sock new-session -d
          tmate -S /tmp/tmate.sock wait tmate-ready

      - name: Show tmate connection info
        run: |
          echo "SSH connection:"
          tmate -S /tmp/tmate.sock display -p '#{tmate_ssh}'
          echo "Web terminal (for browser):"
          tmate -S /tmp/tmate.sock display -p '#{tmate_web}'
          echo "Session will remain active for 12 hours."
          sleep 43200
````

---

## 2. Connect from Termux

After running the workflow, GitHub will show an SSH command like:

```bash
ssh abcdef@ny.tmate.io
```

Paste it in Termux to connect:

```bash
ssh abcdef@ny.tmate.io
```

---

## 3. Setup Build Environment (Ubuntu)

```bash
sudo apt update
sudo apt install -y git wget curl unzip bc bison build-essential ccache flex \
  g++-multilib gcc-multilib gnupg gperf imagemagick lib32ncurses5-dev \
  lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev \
  libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools \
  xsltproc zip zlib1g-dev python3 python-is-python3 openjdk-8-jdk repo

# Set Java 8 if needed
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

---

## 4. Initialize OrangeFox Source

```bash
mkdir OrangeFox && cd OrangeFox
repo init -u https://gitlab.com/OrangeFox/sync.git -b 12.1
repo sync -j$(nproc) --force-sync --no-tags --no-clone-bundle
```

---

## 5. Clone Device Tree, Vendor, Kernel

Replace the links below with your own device sources:

```bash
git clone https://github.com/yourname/device_xxx_yourdevice.git device/xxx/yourdevice
git clone https://github.com/yourname/vendor_xxx_yourdevice.git vendor/xxx/yourdevice
git clone https://github.com/yourname/kernel_xxx_yourdevice.git kernel/xxx/yourdevice
```

---

## 6. Export Build Environment Variables

```bash
export ALLOW_MISSING_DEPENDENCIES=true
export LC_ALL=C
export FOX_USE_TWRP_RECOVERY_IMAGE_BUILDER=1
export OF_MAINTAINER="YourName"
export FOX_VERSION="R12.1"
export FOX_BUILD_TYPE="Stable"

# Optional
export FOX_USE_NANO_EDITOR=1
export FOX_USE_BASH_SHELL=1
export FOX_REPLACE_BUSYBOX_PS=1
```

---

## 7. Build OrangeFox Recovery

```bash
source build/envsetup.sh
lunch omni_yourdevice-eng
mka recoveryimage
```

Replace `yourdevice` with your actual codename.

---

## 8. Output

The built recovery image will be located at:

```
out/target/product/<yourdevice>/recovery.img
```

---

## License

This project is open-source and free to use for educational and development purposes.

```

---


