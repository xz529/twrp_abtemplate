## Universal TWRP Installer for A/B Devices (with Vendor Boot Support)
This script is a universal TWRP installer backend designed for devices with A/B partition layouts. It automatically detects the target device architecture and switches between on-the-fly image patching or a direct clean flash of a pre-compiled .img file.

## ⚡ Key Features
* Smart Partition Auto-Detection: The script checks partition existence in a strict priority chain: vendor_boot ➡️ recovery ➡️ boot.
* Intelligent Kernel Bypassing: The hexpatch kernel and cmdline cleanup routines are automatically skipped when vendor_boot is targeted. Since vendor_boot doesn't contain a kernel, this prevents magiskboot compilation errors.
* Bootloop Prevention: The script restricts on-the-fly ramdisk modification (via .cpio) on newer vendor_boot devices. Starting with Android 12+, vendor_boot utilizes multiple ramdisks (vendor_ramdisk and vendor_boot_recovery_ramdisk). Repacking them dynamically often breaks the structure and leads to bootloops. Thus, a clean pre-built image flash is enforced for these devices.
* Dual-Slot Installation: The installer targets both slots (_a and _b) simultaneously, ensuring successful boot under any active slot.

## 📂 ZIP Archive Structure
For the installer to function correctly, your flashable ZIP must maintain the following file tree:
```text
Your_Installer.zip
├── META-INF/
│   └── com/
│       └── google/
│           └── android/
│               ├── update-binary      <-- (The installer script)
│               └── updater-script     <-- (Empty file or text placeholder)
├── magiskboot-arm                     <-- (32-bit Magisk binary)
├── magiskboot-arm64                   <-- (64-bit Magisk binary)
├── [TWRP Files]                       <-- (Device-dependent, see below) `
```


## 🛠️ How to Pack Your Archive
Depending on your smartphone's partition architecture, place the appropriate files into the root of the ZIP:
Case 1: Modern Devices (Android 11-13+) with vendor_boot partition
What to include: A pre-compiled, ready-to-flash TWRP image.
Filename match: Must match twrp*.img, recovery.img, or vendor_boot*.img (e.g., vendor_boot.img).
How it works: The script detects the vendor_boot partition, skips the unpack/repack steps entirely, and securely flashes the image into both slots. Do NOT use .cpio files here.

Case 2: Devices with a dedicated recovery partition (No vendor_boot)
What to include: Either a ready image (recovery.img) OR a TWRP ramdisk (ramdisk-twrp.cpio).
How it works: If a .cpio is supplied, the script dumps the stock recovery, unpacks it via magiskboot, replaces the stock ramdisk with TWRP, repacks it, and flashes it to recovery_a and recovery_b. The main boot partition remains untouched.

Case 3: Legacy A/B Devices (No dedicated recovery, TWRP inside boot)
What to include: A TWRP ramdisk (ramdisk-twrp.cpio or ramdisk-recovery.cpio).
How it works: The script dumps the stock boot image, unpacks it, hex-patches the kernel strings, swaps the stock ramdisk for the TWRP ramdisk, repacks a new boot.img, and flashes it to both slots.

## ⚠️ Packing Note:
When creating the final .zip file, select the files themselves (META-INF, binaries, image), not the parent folder containing them. The archive must open directly into the root file structure. Use "Normal" or "Store" (No compression) zip modes.

