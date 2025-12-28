# My Arch Linux + Omarchy Installation Journey 🐧

This repository documents my **personal journey and notes** on installing **Arch Linux with Omarchy**, mainly for learning and future reference.

> **Notes** 💡 
> - Windows is installed on a **separate SSD**
> - Arch + Omarchy is installed on an **HDD**
> - I use **Ventoy** for everything here

---

## Tools Used 🧰 

- **Ventoy** – boot multiple ISOs
- **GParted Live** – partitioning
- **Arch Linux ISO**
- **archinstall**
- **Limine bootloader**
- **Omarchy**

---

## Disk Preparation (GParted) 🧱 

1. Boot into **GParted Live**
2. Create a **new 200GB partition** on my HDD  
   (this is where Arch + Omarchy will live)

---

## Boot Arch Linux & Connect to Wi-Fi 🌐 

After booting into the Arch ISO:

```bash
iwctl
````

List devices:

```bash
device list
```

Scan for networks:

```bash
station "<device name>" get-networks
```

Connect to Wi-Fi:

```bash
station "<device name>" connect "<wifi name>"
```

Exit iwctl:

```bash
exit
```

Test internet:

```bash
ping -c 3 google.com
```

---

## Disk Partitioning (cfdisk) 💽 

List disks:

```bash
lsblk
```

Identify the correct HDD (based on size), then:

```bash
cfdisk /dev/<diskname>
```

### Partition layout

#### EFI Partition

* Select `[ New ]`
* Size: `4G`

  > Omarchy EFI requires **3GB+**
* Change type to:

  ```
  EFI System
  ```

#### Root Partition

* Use remaining space (~196G)
* Type:

  ```
  Linux filesystem
  ```

Write changes:

```text
[ Write ] → yes
```

Exit `cfdisk`.

Verify:

```bash
lsblk
```

---

## 🧪 Formatting Partitions

### EFI (4GB)

```bash
mkfs.fat -F32 /dev/<efi-partition>
```

### Root (196GB)

```bash
mkfs.btrfs -f /dev/<root-partition>
```

Verify again:

```bash
lsblk
```

---

## Mounting Partitions 📂 

Mount root:

```bash
mount /dev/<root-partition> /mnt
```

Create boot directory:

```bash
mkdir /mnt/boot
```

Mount EFI:

```bash
mount /dev/<efi-partition> /mnt/boot
```

Verify:

```bash
lsblk
```

Expected:

* `/mnt` → 196G
* `/mnt/boot` → 4G

---

## Installing Arch Linux (archinstall) 🚀 

Start installer:

```bash
archinstall
```

### My selections

* **Mirrors & repositories**

  * Country: `Malaysia`

* **Disk configuration**

  * `Pre-mounted configuration`
  * Root mount directory: `/mnt`

* **Encryption**

  * Optional (Omarchy recommends it, but your choice)

* **Swap**

  * Enabled

* **Bootloader**

  * `Limine` (**required for Omarchy**)

* **Authentication**

  * Set root password
  * Create user
  * Ensure user has `sudo`

* **Applications → Audio**

  * `Pipewire`

* **Network configuration**

  * `Copy ISO network configuration`

* **Timezone**

  * Select correct timezone

Everything else left at default.

Start installation → **Yes**

Wait for the magic. ☕ 

---

## Post-Install (chroot) 🔧 

After install completes, choose:

```
chroot into installation for post-installation configuration
```

Update pacman:

```bash
pacman -Sy
```

Install reflector:

```bash
pacman -S reflector
```

Backup mirrorlist:

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

Generate fast mirrors:

```bash
reflector --verbose --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

Full system update:

```bash
pacman -Syyu
```

Install Omarchy dependencies:

```bash
pacman -S vim nano fastfetch htop gcc make cmake git curl perl wget terminus-font
```

Exit chroot:

```bash
exit
```

Unmount and reboot:

```bash
umount -IR /mnt
reboot now
```

---

## First Boot into Arch 🧑‍💻 

In Limine menu:

```
Arch Linux (Linux)
```

Login as **your user** (not root).

Increase terminal font if needed:

```bash
setfont ter-132n
```

---

## Installing Omarchy 🎨 

Run:

```bash
curl -fsSL https://omarchy.org/install | bash
```

Wait for magic ✨

When done:

```
Reboot Now
```

---

## Adding Windows to Limine 🪟 

Login, then become root:

```bash
sudo -i
```

Scan EFI entries:

```bash
limine-scan
```

Select your **Windows Boot Manager**
(confirm based on disk/UUID).

---

## Limine Configuration Cleanup 🧠 

Navigate:

```bash
cd /boot/EFI/Linux
ls
```

You should see:

```
omarchy_linux.efi
```

Copy it:

```bash
cp omarchy_linux.efi ../limine
```

Rename original:

```bash
mv omarchy_linux.efi omarchy_linux.efi.bak
```

Check Limine folder:

```bash
cd /boot/EFI/limine
```

If `limine.conf` exists:

```bash
mv limine.conf limine.conf.bak
```

---

## Editing limine.conf 📝 

Go to `/boot`:

```bash
cd /boot
nvim limine.conf
```

### Changes made

* Increase timeout:

```ini
timeout: 30
```

* Update paths:

```ini
image_path: /EFI/limine/...
```

(Replace any `/EFI/Linux` paths with `/EFI/limine`)

Save and exit.

---

## Final Reboot 🔁 

```bash
reboot
```

If everything works…

🎉 **Voila! Enjoy Omarchy 😄**

---

## 📌 Final Notes

* Windows runs on **SSD**
* Omarchy runs on **HDD**
* Limine handles dual-boot
* This repo exists purely for **learning & reference**