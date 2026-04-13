# Tulpar OS: Master Architecture & Build Guide (v1.0)

Bu rehber, Tulpar OS'un sanal makine (VirtualBox/VMware) üzerinde, LFS (Linux From Scratch) prensipleriyle sıfırdan inşası için gereken tüm adımları içerir.

---

## 🏗️ Adım 1: Laboratuvar Kurulumu (Host System)

1. **VirtualBox İndir:** [VirtualBox Download](https://www.virtualbox.org/wiki/Downloads)
2. **Ubuntu 24.04 LTS ISO İndir:** [Ubuntu Release](https://releases.ubuntu.com/24.04/)

### Sanal Makine Ayarları:
- **RAM:** En az 4 GB (4096 MB)
- **CPU:** En az 2 Çekirdek
- **Disk:** En az 30 GB (VDI)
- **ISO:** İndirilen Ubuntu dosyasını seçin ve sistemi kurun.

---

## 🛠️ Adım 2: Bağımlılıkların Kurulumu (Ubuntu Terminal)

Ubuntu kurulduktan sonra Terminal'i (Ctrl+Alt+T) açın ve aşağıdaki ağır sanayi araçlarını kurun:

```bash
sudo apt update && sudo apt install -y build-essential bison flex bc libssl-dev libelf-dev libncurses-dev kmod cpio xz-utils wget git python3 python3-pip python3-pyqt6 dosfstools mtools xorriso
```

---

## 🚀 Adım 3: Tulpar OS İskeletini (RootFS) İnşa Etme

Aşağıdaki script bloğu; dizin yapısını kurar, BusyBox'ı (temel komutlar) statik olarak derler ve başlangıç (init) sistemini hazırlar:

```bash
# 1. Klasörleri Hazırla
export LFS=$HOME/tulpar_os
mkdir -pv $LFS/rootfs
export ROOTFS=$LFS/rootfs
mkdir -pv $ROOTFS/{bin,etc,lib,lib64,sbin,usr,var,dev,proc,sys,run,tmp,home,root}

# 2. BusyBox (Temel Komutlar) Kurulumu
cd /tmp
rm -rf busybox-1.36.1
wget -q https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar -xf busybox-1.36.1.tar.bz2 && cd busybox-1.36.1
make defconfig
# Statik derleme ayarı (bağımlılığı sıfıra indirmek için)
sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config
make -j$(nproc)
make CONFIG_PREFIX=$ROOTFS install
ln -sf busybox $ROOTFS/bin/sh

# 3. Tulpar Başlangıç (Init) Scripti
cat << 'EOF' > $ROOTFS/sbin/init
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
echo "-----------------------------------------"
echo "    WELCOME TO TULPAR OS (Hybrid Ed.)    "
echo "-----------------------------------------"
exec /bin/sh
EOF
chmod +x $ROOTFS/sbin/init
```

---

## 🧠 Adım 4: Tulpar OS Beyni (Linux Kernel) Derleme

Çekirdek derleme işlemi donanımla konuşmayı sağlar. Bu adım en uzun süren adımdır:

```bash
cd $LFS
wget -q https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.tar.xz
tar -xf linux-6.6.tar.xz && cd linux-6.6
make defconfig
# Motoru çalıştır (İşlemci gücüne göre 10-30 dk sürer)
make -j$(nproc)

# Derlenen Kernel'i (Motoru) Tulpar'ın içine kopyala
mkdir -p $LFS/boot
cp -v arch/x86/boot/bzImage $LFS/boot/vmlinuz-tulpar
```

---

## 🎨 Adım 5: Tulpar Shell (Siberpunk GUI) Hazırlığı

Python tabanlı özgün arayüzümüzü sistem içine dahil etmek için (Sanal makinede Python yüklüyken):

```bash
# Arayüzü test etmek için (Windows veya Sanal Makine):
python3 src/gui/tulpar_shell.py
```

---

## 📀 Adım 6: ISO Oluşturma (Final)

Tulpar OS'u boot edilebilir bir ISO yapmak için gereken `xorriso` komutları:

```bash
# ISO Paketleme adımları (Kernel ve RootFS birleşimi) yakında eklenecek...
```
