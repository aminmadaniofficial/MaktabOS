# MaktabOS (سیستم‌عامل مکتب)

**An ultra-lightweight, immutable, and tamper-proof Linux operating system engineered to resurrect decommissioned and low-spec PCs across Iranian schools.**

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Target Architectures](https://img.shields.io/badge/Arch-i386%20%7C%20amd64-informational.svg)]()
[![Base Distribution](https://img.shields.io/badge/Base-Debian%2012%20(Bookworm)-D70A53.svg)](https://www.debian.org/)
[![Target Idle RAM](https://img.shields.io/badge/Idle%20RAM-%3C%20100%20MB-success.svg)]()
[![Storage Layer](https://img.shields.io/badge/RootFS-SquashFS%20%2B%20OverlayFS-blueviolet.svg)]()

---

## Table of Contents

1. [Executive Summary & Problem Statement](#1-executive-summary--problem-statement)
2. [Field Constraints in Target Environments](#2-field-constraints-in-target-environments)
3. [Comparative Analysis: Why Prior Art Fails](#3-comparative-analysis-why-prior-art-fails)
4. [System Architecture & Core Specifications](#4-system-architecture--core-specifications)
5. [Key Subsystems](#5-key-subsystems)
   - [5.1. Immutable Storage & RAM Budgeting](#51-immutable-storage--ram-budgeting)
   - [5.2. Curriculum Compatibility Layer (`school-exe-runner`)](#52-curriculum-compatibility-layer-school-exe-runner)
   - [5.3. Zero-Footprint Modular Offline Store](#53-zero-footprint-modular-offline-store)
   - [5.4. Kiosk Mode, Removable Media & Panic Recovery](#54-kiosk-mode-removable-media--panic-recovery)
   - [5.5. Degraded Video & Display Automation](#55-degraded-video--display-automation)
   - [5.6. ZeroConf Unmanaged LAN Networking](#56-zeroconf-unmanaged-lan-networking)
6. [Repository Structure](#6-repository-structure)
7. [Development Roadmap](#7-development-roadmap)
8. [بخش فارسی (Persian Technical Overview)](#بخش-فارسی-persian-technical-overview)
9. [License](#license)

---

## 1. Executive Summary & Problem Statement

Public schools and vocational academies (*Honarestan*) in developing regions and underfunded school districts across Iran house tens of thousands of decommissioned desktop computers. These machines typically feature:
* **Memory:** 512 MB to 2 GB DDR2/DDR3 RAM.
* **CPU:** Single-core to early dual-core 32-bit and 64-bit processors (Intel Pentium 4, Celeron D, Pentium Dual-Core, Core 2 Duo, AMD Athlon 64).
* **Storage:** Legacy IDE/SATA HDDs (40 GB to 160 GB) with significant bad-sector wear.
* **Displays:** 4:3 CRT monitors or early LCD panels driven over worn VGA cabling.

When running legacy Windows installations (Windows XP or Windows 7), these machines suffer from persistent malware infections transmitted via student USB drives, accidental or malicious desktop misconfiguration during recess, and severe filesystem corruption caused by improper hard shutdowns (unplugging wall power).

**MaktabOS** is an open-source, reproducible, immutable Linux operating system engineered specifically to resolve these constraints. It combines an immutable read-only root filesystem with memory-backed runtime overlays, a sub-100MB idle RAM desktop environment conforming to national textbook guidelines, and zero-maintenance automated hardware recovery.

---

## 2. Field Constraints in Target Environments

Any operating system deployed in this context must satisfy five non-negotiable operational requirements:

1. **National Curriculum Conformance:**
   The national curriculum for *Work and Technology* (Grades 7–9) and vocational IT coursework mandates instruction based on the standard Windows desktop paradigm (e.g., *This PC*, Drive *C:* and *D:*, standard wizard installers, and Python's standard `idle3` IDE). Presenting an abstract Unix filesystem (`/usr`, `/etc`) or unconventional desktop metaphors causes immediate rejection by educators.

2. **Unregulated Power Terminations:**
   Instructors and custodial staff routinely shut down computer labs by pulling main circuit breakers or toggling power strips. The OS must be 100% immune to filesystem corruption resulting from sudden loss of power.

3. **Recess Tampering & Breakage:**
   Between scheduled classes, unattended students routinely alter system resolutions, rotate screen axes, delete desktop shortcuts, modify wallpaper, and plug in infected removable drives. The classroom monitor or student lab lead must be able to restore the system to baseline state within seconds without administrative intervention.

4. **Zero-Infrastructure Local Networks:**
   School computer labs generally connect 15 to 25 workstations to an unmanaged network switch without access to an upstream router, gateway, DHCP server, or Internet connectivity.

5. **Degraded Video Cabling:**
   Classroom computers connected to ceiling-mounted projectors commonly use aged, stretched VGA cables with fractured Display Data Channel (DDC/EDID) pins, preventing GPUs from detecting display timings and causing frequent signal blackout.

---

## 3. Comparative Analysis: Why Prior Art Fails

| Distribution / Solution | Target Profile | Failure Mode in Iranian Schools |
| :--- | :--- | :--- |
| **Sugar OS** | Elementary 1-to-1 learning (OLPC) | Non-standard interface; does not teach transferable desktop skills; breaks textbook compatibility; abandoned ecosystem. |
| **Endless OS** | Offline-first family/student OS | Heavy GNOME base; Flatpak distribution model requires massive memory overhead and modern OpenGL; download payloads (>16 GB) unmanageable offline. |
| **Debian Edu / Skolelinux** | School network distribution | Architected for centralized LDAP/Kerberos and thin-client servers; demands dedicated network administrators unavailable in schools. |
| **ChromeOS Flex** | Refurbished hardware | Drops 32-bit hardware support entirely; strictly dependent on high-speed Internet connectivity and Google cloud services. |
| **Ubermix** | Netbook/student distribution | Memory footprint too high for ≤1 GB systems; lacks localization, RTL typography, and national textbook tooling. |
| **Legacy Windows (XP / 7)** | Commercial legacy desktop | Highly vulnerable to USB autorun malware; NTFS journal corruption on power loss; proprietary licensing barriers. |
| **MaktabOS** | **Sub-2GB legacy educational hardware** | **Designed from the ground up for zero-connectivity, immutable operations, and textbook-exact UI parity.** |

---

## 4. System Architecture & Core Specifications

```text
+-----------------------------------------------------------------------------------+
|                                 USER SESSION                                      |
|  - Openbox Window Manager (Memory budget: < 15 MB)                                |
|  - Tint2 Taskbar (Windows 10/11 layout, Shamsi/Jalali calendar, FA/EN layout)    |
|  - Rofi App Launcher (Windows Start menu theme, sub-millisecond search)           |
|  - Picom Compositor (XRender software fallback backend, rounded corners)          |
|  - PCManFM File Manager (Exposing This PC, Drive C, Drive D, Trash)               |
+-----------------------------------------------------------------------------------+
|                              EXECUTION DISPATCHER                                 |
|  - school-exe-runner: Native binary mapper + 32-bit WINE fallback with Tahoma     |
|  - fake-installer: Simulated Step-by-Step wizard for curriculum training          |
|  - offline-store: On-demand loop-mounted SquashFS application bundles             |
+-----------------------------------------------------------------------------------+
|                            IMMUTABLE STORAGE STACK                                |
|  - OverlayFS Union Mount                                                          |
|    ├── Upper Layer (Read-Write): tmpfs (RAM-backed, size-constrained)             |
|    └── Lower Layer (Read-Only): SquashFS rootfs (xz / zstd-22 compressed)        |
|  - zram: Compressed block device backing swap (LZ4/ZSTD, 100-150% RAM capacity)   |
|  - Persistent Partition: Ext4 formatted "/home/student/Drive_D" on local disk     |
+-----------------------------------------------------------------------------------+
|                        BASE OPERATING SYSTEM & KERNEL                             |
|  - Debian 12 (Bookworm) minbase bootstrap                                         |
|  - Linux Kernel (LTS, i386 and amd64 architectures)                               |
|  - Systemd-free or stripped-down SysVinit/OpenRC option for sub-5-second boot     |
+-----------------------------------------------------------------------------------+
```

### Technical Specification Matrix

| Parameter | Specification | Engineering Rationale |
| :--- | :--- | :--- |
| **Base Distribution** | Debian 12 (*Bookworm*) minimal | Unmatched repository stability, long-term multi-arch support, `i386` multiarch compatibility. |
| **Target Architectures** | `i386` (x86 32-bit), `amd64` (x86_64) | Full hardware coverage for legacy NetBurst/Core2 and modern 64-bit platforms. |
| **Target Idle RAM** | **65 MB – 95 MB** | Leaves >75% of a 512 MB physical memory budget free for application workloads. |
| **Filesystem Architecture** | `SquashFS` + `OverlayFS` + `tmpfs` | Absolute immunity to dirty power cuts; zero persistent writes to root volume. |
| **Memory Compression** | `zram` with `lz4` compression | Expands usable memory by 2x–2.5x without disk thrashing on legacy HDDs. |
| **Display Server & WM** | X.Org Server + `Openbox` | Minimal memory footprint, high stability on legacy SiS, VIA, and Intel GMA graphics chips. |
| **Taskbar & Panel** | `Tint2` | Native support for system trays, keyboard layout indicators, and Shamsi (Solar Hijri) clock. |
| **Application Launcher** | `Rofi` (custom Windows 10/11 theme) | Instantaneous cached index, zero desktop clutter. |
| **Typography** | **Vazirmatn** (FreeType hintfull / subpixel) | Optimum legibility for Persian text across low-DPI CRT and aging LCD monitors. |

---

## 5. Key Subsystems

### 5.1. Immutable Storage & RAM Budgeting

To ensure that sudden power termination cannot damage the operating system:
1. The base root filesystem resides in a compressed, read-only `SquashFS` container.
2. At boot time, `initramfs` initializes an `OverlayFS` instance where:
   - **`lowerdir`:** The immutable SquashFS container.
   - **`upperdir`:** A memory-backed `tmpfs` instance mounted with an enforced quota (`size=150M`).
3. **RAM Preservation on 512MB Systems:**
   - Any runtime writes by students or background services are diverted into memory.
   - A `zram` swap device is initialized with equal capacity to physical RAM.
   - Student assignments and projects are routed to a distinct, locally mounted Ext4 partition mounted at `/home/student/Drive_D`. All other modifications are purged automatically upon reboot.

### 5.2. Curriculum Compatibility Layer (`school-exe-runner`)

National textbooks assume a Windows operating system and instruct students to interact with `.exe` files and Python IDLE. MaktabOS addresses this via an intelligent execution interceptor registered with `binfmt_misc` and desktop MIME handlers:

* **Native Application Redirection:** If a student attempts to run an installer for software that exists natively in MaktabOS (such as the official Python 3 Windows installer or Scratch installer), the runner intercepts the call, displays a localized dialog informing the student that the software is pre-installed, and launches the native package (e.g., native `idle3` or native Scratch).
* **Curriculum Simulation (Fake Wizard Installer):** For practical textbook exams requiring students to demonstrate installation knowledge (e.g., clicking *Next > Next > I Agree > Finish*), MaktabOS provides an interactive simulation sandbox that replicates standard installer wizard steps without altering system files.
* **Educational CD & Media Compatibility:** Educational multimedia discs distributed with school textbooks (frequently packaged using AutoPlay Media Studio or Macromedia Director) are dispatched to a pre-configured 32-bit WINE prefix pre-populated with standard Windows core fonts (Tahoma, Arial) and necessary runtime libraries (`mfc42`, `msvcr`).

### 5.3. Zero-Footprint Modular Offline Store

Legacy school computers often feature 40 GB or 80 GB hard drives with limited free space. Heavy educational software (GCompris, GeoGebra, Tux Paint, LibreOffice) cannot all be unpacked simultaneously without exhausting disk capacity.

MaktabOS implements an on-demand loop-mount architecture:
* Application packages are compiled into standalone, highly compressed `.sqfs` modules located in `/opt/offline-store/packages/`.
* A lightweight Python/GTK interface lists available tools with Persian metadata.
* Clicking "Enable" executes an instantaneous loop-mount:
  ```bash
  mount -o loop,ro /opt/offline-store/packages/geogebra.sqfs /opt/apps/geogebra
  ```
* Desktop integration files (`.desktop`, icons) are symlinked into the desktop environment within 200 ms without requiring file extraction or persistent storage allocation.

### 5.4. Kiosk Mode, Removable Media & Panic Recovery

To counteract recess tampering:
* **Removable Media Isolation:** All student USB flash drives are mounted automatically via `udev` rules using the `nosuid,nodev,noexec` mount options. Students can read/write data, PDFs, and run Python source scripts via the native Python interpreter (`python3 /media/usb/...`), but no compiled binary or shell script can execute directly from the drive.
* **Rapid Panic Button:** In the event of desktop disorder before a teacher enters the room, the designated student monitor presses `Ctrl + Alt + Backspace` or clicks the taskbar Panic icon. The system purges the `tmpfs` user overlay, re-links `/etc/skel`, and restarts the Openbox session in under 2 seconds.
* **Teacher Master Key:** Inserting a teacher-authorized USB drive containing an authenticated cryptographic token (`.masterkey`) triggers a `udev` event that unlocks administrative controls, displays configuration tools, and enables persistent writes to system modules.

### 5.5. Degraded Video & Display Automation

School video projectors typically connect over aged, extended VGA cables where the DDC/EDID wiring is severed. This prevents graphics drivers from querying valid video timings, resulting in out-of-range errors or blank screens.

* MaktabOS ships with pre-calculated, conservative display modelines (`1024x768@60Hz` and `1280x720@60Hz`).
* A background display daemon listens for VGA hotplug interrupts and falls back to enforced safe timings via `xrandr` when EDID is absent.
* **Instructional Overlays:** The `Super + P` key combination cycles through mirrored and external display profiles. Screen annotation tool `Gromit-MPX` is pre-mapped to `F9`, enabling teachers to draw and annotate directly over any running application using a standard mouse or digital pen.

### 5.6. ZeroConf Unmanaged LAN Networking

Computer labs in Iranian schools frequently operate without dedicated DHCP routers:
* MaktabOS enables `avahi-daemon` with automatic IPv4 Link-Local address allocation (RFC 3927: `169.254.0.0/16`).
* Workstations auto-assign deterministic hostnames (`maktab-01.local`, `maktab-02.local`) based on MAC addresses.
* A minimal local management service allows the teacher's workstation to discover all peers, project the teacher's screen, lock student inputs during lectures, and broadcast assignment files directly to the persistent `Drive_D` directory of all student machines simultaneously.

---

## 6. Repository Structure

```text
MaktabOS/
├── build/                   # OS image bootstrap and ISO generation
│   ├── configs/             # Debootstrap package manifests
│   ├── scripts/             # Chroot configuration and image mastering
│   └── build-iso.sh         # Top-level ISO compiler script
├── overlay/                 # File hierarchy overlaid onto rootfs
│   ├── etc/                 # System configuration (Openbox, Tint2, Rofi, Picom, udev)
│   ├── usr/                 # Core scripts and custom binaries
│   └── skel/                # Default student user profile template
├── modules/                 # Standalone custom engineering components
│   ├── school-exe-runner/   # Intelligent .exe dispatcher & WINE sandbox
│   ├── fake-installer/      # Curriculum installer simulation tool
│   ├── offline-store/       # Modular SquashFS package manager GUI
│   ├── kiosk-security/      # Panic scripts, udev rules, and USB sandboxing
│   └── projector-auto/      # VGA EDID fallback and display daemon
├── packages/                # Build recipes for custom modular .sqfs bundles
├── docs/                    # Architectural documents and design specifications
├── LICENSE                  # GNU General Public License v3.0
└── README.md                # Project documentation and specifications
```

---

## 7. Development Roadmap

- [ ] **Milestone 1: Repository Foundation & Architectural Baseline**
  - [x] Publish complete architectural specifications and field constraints under GPLv3.
  - [ ] Initialize standard directory structure and build environment scaffolding.
- [ ] **Milestone 2: RootFS Generation & Live-Boot Layer (MVP)**
  - [ ] Implement reproducible `debootstrap` scripts for Debian 12 minimal (`i386` and `amd64`).
  - [ ] Configure `initramfs` hooks for SquashFS root mounting, OverlayFS union, and `zram` activation.
  - [ ] Implement persistent partition detection for `Drive_D`.
- [ ] **Milestone 3: Desktop Environment & UX Conformance**
  - [ ] Implement customized Openbox, Tint2, and Rofi configurations replicating the Windows paradigm.
  - [ ] Integrate Vazirmatn typography with tuned subpixel rendering for legacy CRT/LCD screens.
  - [ ] Implement Shamsi/Jalali desktop panel calendar support.
- [ ] **Milestone 4: Core Educational & Security Modules**
  - [ ] Build and validate `school-exe-runner` and WINE 32-bit execution harness.
  - [ ] Develop the GTK/Python modular offline store for on-demand `.sqfs` loop-mounting.
  - [ ] Package and test udev rules for USB `noexec` isolation, panic reset, and teacher master keys.
- [ ] **Milestone 5: Classroom Automation, Printing & ISO Release**
  - [ ] Package offline GDI/host-based printer drivers (`foo2zjs`, `hplip`) and firmware files.
  - [ ] Implement Avahi Link-Local discovery and zero-configuration lab tooling.
  - [ ] Generate, test, and release verifiable live installation ISO images.

---

## بخش فارسی (Persian Technical Overview)

### سیستم‌عامل مکتب (MaktabOS) چیست؟

مکتب‌او‌اس یک سیستم‌عامل لینوکسی مستقل، فوق‌سبک و ایستا (Immutable) است که به طور اختصاصی برای **احیای کامپیوترهای قدیمی و اسقاطی مدارس ایران (با حافظه رم ۵۱۲ مگابایت تا ۲ گیگابایت)** مهندسی شده است.

### صورت‌مسئله در مدارس کشور
* **ویروسی شدن مداوم:** ویندوزهای قدیمی (XP و ۷) در مدارس با اتصال اولین فلش مموری دانش‌آموزی آلوده شده و از کار می‌افتند.
* **خرابی ناشی از قطع برق:** خاموش کردن ناگهانی سیستم‌ها با قطع کلید فیوز یا کشیدن دوشاخه، فایل‌سیستم NTFS را خراب و هارد دیسک‌های فرسوده را غیرقابل استفاده می‌کند.
* **تطابق با کتب درسی:** کتب کار و فناوری و رایانه بر مبنای مفاهیم محیط ویندوز (رایانه من، درایو C و D، پایتون IDLE و نصب گام‌به‌گام برنامه‌ها) تدریس می‌شوند. سیستم‌عامل‌های موجود لینوکسی به دلیل تفاوت در رابط کاربری توسط معلمان پذیرفته نمی‌شوند.
* **نبود شبکه استاندارد:** کارگاه‌های مدارس اغلب فاقد اینترنت، روتر یا سرور DHCP بوده و رایانه‌ها تنها با یک سوئیچ معمولی به هم متصل هستند.

### مشخصات و نوآوری‌های فنی مکتب‌او‌اس
1. **فایل‌سیستم پایدار و ضد ضربه الکتریکی:** قرارگیری ریشه سیستم‌عامل در یک کانتینر فشرده و فقط‌خواندنی (SquashFS) و سوار شدن لایه تغییرات روی حافظه رم (OverlayFS + tmpfs). با قطع ناگهانی برق یا کشیدن کابل، هیچ آسیبی به فایل‌های سیستم‌عامل وارد نمی‌شود.
2. **مصرف حافظه رم زیر ۱۰۰ مگابایت:** استفاده از ترکیب Openbox، پنل سفارشی Tint2 (شامل ساعت و تقویم شمسی)، لانچر Rofi و فونت استاندارد وزیرمتن با ظاهری مشابه ویندوز برای انطباق کامل با سرفصل‌های آموزشی.
3. **اجراکننده هوشمند فایلهای ویندوزی (`school-exe-runner`):** شناسایی و تفکیک فایل‌های `.exe`؛ در صورت وجود نسخه بومی (نظیر پایتون و اسکرچ)، نسخه بهینه لینوکسی باز می‌شود و در صورت نیاز به اجرای لوح‌های فشرده آموزشی مدارس، برنامه به صورت خودکار داخل WINE 32-bit بهینه‌شده با فونت فارسی اجرا می‌گردد.
4. **فروشگاه آفلاین ماژولار بدون اشغال هارد دیسک:** نرم‌افزارهای سنگین آموزشی (مانند جئوژبرا، گکومپریس، اسکرچ) به صورت فایل‌های فوق‌فشرده `.sqfs` ذخیره شده و با کلیک کاربر، ظرف چند میلی‌ثانیه با دستور `mount -o loop` بدون نیاز به نصب یا مصرف حافظه دیسک فعال می‌شوند.
5. **امنیت درگاه‌های USB و دکمه نجات فوری:** اتصال فلش دانش‌آموزان به طور پیش‌فرض بدون قابلیت اجرای مستقیم باینری (`noexec`) ماونت می‌شود تا انتقال بدافزار ناممکن گردد. در صورت دستکاری تنظیمات توسط دانش‌آموزان در زنگ تفریح، مسئول کلاس با کلید میانبر `Ctrl+Alt+Backspace` تمام تغییرات را در ۲ ثانیه ریست می‌کند.
6. **مدیریت کابل‌های فرسوده پروژکتور کلاس:** اسکریپت خودکار تشخیص سیگنال با تزریق رزولوشن پایدار و امن (`1024x768`) بدون وابستگی به اطلاعات EDID کابل‌های فرسوده VGA، به همراه ابزار تدریس و خط‌کشی روی صفحه (`Gromit-MPX` با کلید `F9`).

---

## License

MaktabOS is free software licensed under the **GNU General Public License Version 3.0 (GPLv3)**. See the [LICENSE](LICENSE) file for complete licensing terms and conditions.
