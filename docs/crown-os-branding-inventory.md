# Crown OS — Master Branding and Configuration Inventory

**Document Reference:** `docs/crown-os-branding-inventory.md`  
**Base Distribution:** Linux Mint 22.3 Cinnamon 64-bit ("Zena", Ubuntu 24.04 "Noble")  
**Audit Status:** Phase 1 Read-Only Complete — Evidence Based  
**Inspection Date:** 2026-10-22  

---

## 1. Overview and Methodology

This document contains an itemized inventory of branding assets, configuration overrides, hooks, and metadata identified across both the **Outer ISO filesystem** (`iso-work/iso-tree`) and the **Compressed Root Filesystem** (`iso-work/squashfs-root`).

Each entry adheres to the 12 required audit fields:
1. **Exact Path**
2. **Layer** (Outer ISO vs. SquashFS Root)
3. **Current Purpose**
4. **Current Behavior / Value** (with direct inspection evidence)
5. **Classification** (Cosmetic, Functional, Mixed)
6. **Likely Effect of Changing It**
7. **Risk Level** (Low, Medium, High, Critical)
8. **Dependencies and Related Files**
9. **Recommended Future Treatment**
10. **Verification Method**
11. **Rollback Procedure**
12. **Phasing Suitability** (Phase B through Phase G)

Package provenance is recorded where tracked in the `dpkg` database.

---

## 2. Outer ISO Filesystem Inventory (`iso-tree`)

### ITEM-ISO-01: ISO Volume Label (`xorriso -V`)
- **Exact Path:** Embedded in ISO 9660 PVD via xorriso `-V` argument
- **Layer:** Outer ISO filesystem metadata
- **Current Purpose:** Identifies the optical / USB disk volume name to firmware, BIOS/UEFI, and host file managers.
- **Current Behavior / Value:** `Linux Mint 22.3 Cinnamon 64-bit` (Evidence: `iso-work/original-mkisofs.txt` and `.disk/mkisofs`).
- **Classification:** Cosmetic / Identification
- **Likely Effect:** Modifies how the booted USB/DVD appears in BIOS boot selector and file managers when mounted.
- **Risk Level:** Low
- **Dependencies:** `iso-work/original-mkisofs.txt`, build script xorriso command line.
- **Recommended Treatment:** Update to `Crown OS 1.0 Cinnamon 64-bit` during ISO synthesis in Phase F.
- **Verification Method:** Inspect built ISO header using `xorriso -indev <iso> -pvd_info`.
- **Rollback Procedure:** Revert `-V` parameter in build script.
- **Phasing Suitability:** Phase F (ISO Build Pipeline).

---

### ITEM-ISO-02: Disk Identity Metadata (`.disk/info`)
- **Exact Path:** `.disk/info`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Defines the human-readable media name and release string; parsed by `casper` initramfs scripts to label desktop shortcuts and installer prompts.
- **Current Behavior / Value:** `Linux Mint 22.3 "Zena" - Release amd64 20260108`
- **Classification:** Mixed (Cosmetic label, but critically consumed by `casper-bottom/25adduser` to substitute `RELEASE` in `ubiquity.desktop`).
- **Likely Effect:** Replaces the text on the live desktop installer icon (`Install RELEASE` -> `Install Crown OS`).
- **Risk Level:** Medium
- **Dependencies:** `iso-work/squashfs-root/usr/share/initramfs-tools/scripts/casper-bottom/25adduser`, `ubiquity.desktop`.
- **Recommended Treatment:** Set to `Crown OS 1.0 "Aura" - Release amd64 <build_date>`. Ensure tokens 1-2 parse cleanly into `Crown OS` via `cut -d' ' -f1-2`.
- **Verification Method:** Check `.disk/info` text; boot live ISO and verify desktop icon reads "Install Crown OS".
- **Rollback Procedure:** Restore original string.
- **Phasing Suitability:** Phase E (Installer & Release Identity) / Phase F (Build).

---

### ITEM-ISO-03: Disk Defines (`.disk/README.diskdefines`)
- **Exact Path:** `.disk/README.diskdefines`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Macro definitions for debian-cd/live-build media tooling.
- **Current Behavior / Value:** Contains `#define DISKNAME Linux Mint 22.3 "Zena" - Release amd64`.
- **Classification:** Cosmetic / Informational
- **Likely Effect:** Minimal runtime impact; updates metadata compatibility.
- **Risk Level:** Low
- **Dependencies:** None directly runtime critical.
- **Recommended Treatment:** Update `DISKNAME` to Crown OS equivalent.
- **Verification Method:** Inspect file content.
- **Rollback Procedure:** Restore original text.
- **Phasing Suitability:** Phase E.

---

### ITEM-ISO-04: Release Notes URL Marker (`.disk/release_notes_url`)
- **Exact Path:** `.disk/release_notes_url`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Upstream documentation link embedded on installation media.
- **Current Behavior / Value:** `https://www.linuxmint.com/rel_zena.php`
- **Classification:** Cosmetic / Informational
- **Likely Effect:** None at runtime unless parsed by external disk inspection utilities.
- **Risk Level:** Low
- **Dependencies:** None.
- **Recommended Treatment:** Point to local documentation or Crown OS release documentation URL.
- **Verification Method:** Inspect file content.
- **Rollback Procedure:** Restore original URL.
- **Phasing Suitability:** Phase E.

---

### ITEM-ISO-05: Mint ISO Diagnostic Marker (`.disk/mint_iso`)
- **Exact Path:** `.disk/mint_iso`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Upstream troubleshooting marker unique to Mint ISO generation scripts.
- **Current Behavior / Value:** Comments indicating `# This filename is unique to Mint ISO images.`
- **Classification:** Informational
- **Likely Effect:** None known in casper startup.
- **Risk Level:** Low
- **Dependencies:** Potential upstream diagnostic scripts.
- **Recommended Treatment:** Retain file or replace with `.disk/crown_iso` while preserving marker if check scripts require it.
- **Verification Method:** Grep across `squashfs-root` to confirm whether any script queries `.disk/mint_iso`.
- **Rollback Procedure:** Restore file.
- **Phasing Suitability:** Phase E.

---

### ITEM-ISO-06: UEFI GRUB Configuration (`boot/grub/grub.cfg`)
- **Exact Path:** `boot/grub/grub.cfg`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Primary boot menu definition for UEFI systems booted from ISO/USB.
- **Current Behavior / Value:**
  - `menuentry "Start Linux Mint 22.3 Cinnamon 64-bit" --class linuxmint`
  - `menuentry "Start Linux Mint 22.3 Cinnamon 64-bit (compatibility mode)"`
  - Passes kernel arguments: `boot=casper uuid=6e72f523-... username=mint hostname=mint quiet splash --`
- **Classification:** Mixed (Branding titles are cosmetic; boot options, UUID, username, and hostname are strictly functional).
- **Likely Effect:** Changes boot menu titles displayed on modern UEFI systems.
- **Risk Level:** High (Syntax errors or broken arguments cause failure to boot).
- **Dependencies:** `casper/vmlinuz`, `casper/initrd.lz`, `casper-uuid-generic`, `boot/grub/theme.cfg`.
- **Recommended Treatment:** Rebrand `menuentry` labels to `"Start Crown OS 1.0 Cinnamon 64-bit"`. In early phases, keep `username=mint hostname=mint` to avoid breaking casper hooks; plan controlled transition to `username=crown hostname=crown` with verified casper compatibility.
- **Verification Method:** UEFI boot in QEMU/hardware; verify menu rendering and boot progression.
- **Rollback Procedure:** Restore original `grub.cfg`.
- **Phasing Suitability:** Phase D (Startup & Boot Presentation).

---

### ITEM-ISO-07: UEFI GRUB Theme (`boot/grub/live-theme/theme.txt`)
- **Exact Path:** `boot/grub/live-theme/theme.txt`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Visual styling specification for GRUB gfxterm bootloader.
- **Current Behavior / Value:** References `desktop-image: "/isolinux/splash800x600.png"`, sets font `Unifont Regular 16`, text colors `#ffffff` and `#a8a8a8`.
- **Classification:** Cosmetic
- **Likely Effect:** Controls the background artwork, progress bar, and layout of UEFI GRUB menu.
- **Risk Level:** Medium (Missing background or syntax error falls back to text-mode GRUB).
- **Dependencies:** `isolinux/splash.png` or `boot/grub/splash.png`, `boot/grub/unicode.pf2`.
- **Recommended Treatment:** Provide a custom Crown OS 800x600 background asset aligned with the emerald/teal design system; refine font color tokens (`selected_item_color = #10B981` or `#FFFFFF`).
- **Verification Method:** Boot UEFI ISO in QEMU with 800x600 and 1024x768 display modes.
- **Rollback Procedure:** Restore original `theme.txt` and background image.
- **Phasing Suitability:** Phase D.

---

### ITEM-ISO-08: BIOS ISOLINUX Boot Menu (`isolinux/live.cfg`)
- **Exact Path:** `isolinux/live.cfg`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Bootloader configuration for legacy BIOS / CSM hardware.
- **Current Behavior / Value:**
  - `menu background splash.png`
  - `menu title Welcome to Linux Mint 22.3 64-bit`
  - `label live ... menu label Start Linux Mint`
  - Kernel append line: `boot=casper initrd=/casper/initrd.lz uuid=... username=mint hostname=mint quiet splash --`
- **Classification:** Mixed (Menu titles/artwork cosmetic; kernel arguments functional).
- **Likely Effect:** Updates presentation for BIOS boot environments.
- **Risk Level:** High (Command line corruption causes boot lockup).
- **Dependencies:** `isolinux/isolinux.bin`, `isolinux/vesamenu.c32`, `isolinux/splash.png`.
- **Recommended Treatment:** Update `menu title` and `menu label` to Crown OS; supply new 640x480 `splash.png`. Keep kernel parameters consistent with `grub.cfg`.
- **Verification Method:** Boot BIOS/Legacy ISO in QEMU (`-boot d`).
- **Rollback Procedure:** Restore original `live.cfg`.
- **Phasing Suitability:** Phase D.

---

### ITEM-ISO-09: ISOLINUX Splash Artwork (`isolinux/splash.png`)
- **Exact Path:** `isolinux/splash.png`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Background graphic for legacy BIOS syslinux/isolinux menu.
- **Current Behavior / Value:** PNG image, 640x480, 8-bit RGBA, displaying Linux Mint logo and green background.
- **Classification:** Cosmetic
- **Likely Effect:** Replaces background banner during legacy boot.
- **Risk Level:** Low
- **Dependencies:** `isolinux/live.cfg`, `isolinux/vesamenu.c32`.
- **Recommended Treatment:** Replace with Crown OS 640x480 8-bit indexed or RGBA PNG.
- **Verification Method:** Inspect image dimensions with `file`; test legacy boot menu in QEMU.
- **Rollback Procedure:** Revert to baseline `splash.png`.
- **Phasing Suitability:** Phase D.

---

### ITEM-ISO-10: Integrity Manifest (`md5sum.txt`)
- **Exact Path:** `md5sum.txt`
- **Layer:** Outer ISO filesystem root
- **Current Purpose:** Pre-boot self-check manifest parsed by `casper-md5check` when media verification is selected.
- **Current Behavior / Value:** Contains 1,900+ MD5 hashes for all files in the ISO (including `casper/filesystem.squashfs`, `casper/vmlinuz`, etc.).
- **Classification:** Functional / Integrity
- **Likely Effect:** If not regenerated after any ISO change, media check fails and reports corrupted media.
- **Risk Level:** High (Failed integrity test triggers user alarm or aborts installation).
- **Dependencies:** Every file on the ISO.
- **Recommended Treatment:** Automatically recalculate `md5sum.txt` during the final step of the ISO build pipeline.
- **Verification Method:** Run `md5sum -c md5sum.txt` against outer ISO directory; run `casper-md5check` in live boot test.
- **Rollback Procedure:** Regenerate from active ISO tree.
- **Phasing Suitability:** Phase F.

---

### ITEM-ISO-11: Casper Filesystem Size Metadata (`casper/filesystem.size`)
- **Exact Path:** `casper/filesystem.size`
- **Layer:** Outer ISO filesystem
- **Current Purpose:** Records the exact uncompressed byte size of the SquashFS root filesystem; parsed by Ubiquity installer to verify adequate disk target space.
- **Current Behavior / Value:** `8354459648` (approx 7.78 GiB).
- **Classification:** Functional
- **Likely Effect:** If missing or underestimated, installer may attempt installation onto an undersized target disk and crash mid-install.
- **Risk Level:** High
- **Dependencies:** `iso-work/squashfs-root`, Ubiquity installer disk space check.
- **Recommended Treatment:** Automatically recalculate via `du -sx --block-size=1 <squashfs-root> | cut -f1` during the SquashFS build pipeline.
- **Verification Method:** Compare value against `du` measurement of rootfs before mksquashfs.
- **Rollback Procedure:** Re-measure and update.
- **Phasing Suitability:** Phase F.

---

## 3. Root Filesystem / SquashFS Inventory (`squashfs-root`)

### ITEM-FS-01: Standard OS Release Metadata (`/etc/os-release`)
- **Exact Path:** `/etc/os-release` (symlinked from `/usr/lib/os-release`)
- **Layer:** SquashFS Root
- **Current Purpose:** POSIX standard distribution identification queried by `systemd`, `neofetch`, modern desktop applications, and `update-grub`.
- **Current Behavior / Value:**
  - `NAME="Linux Mint"`
  - `VERSION="22.3 (Zena)"`
  - `ID=linuxmint`
  - `ID_LIKE="ubuntu debian"`
  - `PRETTY_NAME="Linux Mint 22.3"`
  - `VERSION_CODENAME=zena`
  - `UBUNTU_CODENAME=noble`
- **Classification:** Mixed (Presentation names vs. package-management compatibility identifiers).
- **Likely Effect:** Affects desktop display, system info apps, and `GRUB_DISTRIBUTOR`.
- **Risk Level:** High (Breaking `ID_LIKE="ubuntu debian"` or `UBUNTU_CODENAME=noble` breaks third-party `.deb` repo installation scripts and driver managers).
- **Dependencies:** `dpkg`, APT repositories, `base-files` package, Cinnamon System Info, `update-grub`.
- **Package Provenance:** `base-files` (Mint modified).
- **Recommended Treatment:**
  - Retain: `ID_LIKE="ubuntu debian"`, `VERSION_CODENAME=zena`, `UBUNTU_CODENAME=noble`.
  - Update: `NAME="Crown OS"`, `PRETTY_NAME="Crown OS 1.0 (Aura)"`, `VERSION="1.0"`, `VERSION_ID="1.0"`, `ID=crown-os`. Keep `ID_LIKE="linuxmint ubuntu debian"` for maximum compatibility.
- **Verification Method:** Run `source /etc/os-release && echo $PRETTY_NAME`; run `update-grub` test.
- **Rollback Procedure:** Restore baseline `/etc/os-release`.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-02: Legacy LSB Release Metadata (`/etc/lsb-release`)
- **Exact Path:** `/etc/lsb-release`
- **Layer:** SquashFS Root
- **Current Purpose:** Legacy LSB release identity parsed by `lsb_release` command and legacy software installers.
- **Current Behavior / Value:**
  - `DISTRIB_ID=LinuxMint`
  - `DISTRIB_RELEASE=22.3`
  - `DISTRIB_CODENAME=zena`
  - `DISTRIB_DESCRIPTION="Linux Mint 22.3 Zena"`
- **Classification:** Mixed
- **Likely Effect:** Modifies output of `lsb_release -a`.
- **Risk Level:** Medium (Some vendor scripts check `DISTRIB_ID=LinuxMint` or `Ubuntu`).
- **Dependencies:** `lsb-release` utility, external third-party installer scripts.
- **Package Provenance:** `base-files`.
- **Recommended Treatment:** Update `DISTRIB_DESCRIPTION="Crown OS 1.0 Aura"`, `DISTRIB_RELEASE=1.0`. Set `DISTRIB_ID=CrownOS` only after verifying APT source substitution scripts.
- **Verification Method:** Run `lsb_release -a`.
- **Rollback Procedure:** Restore original file.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-03: Linux Mint Specific Info Descriptor (`/etc/linuxmint/info`)
- **Exact Path:** `/etc/linuxmint/info`
- **Layer:** SquashFS Root
- **Current Purpose:** Proprietary Mint configuration queried by `mintreport` (System Info), `mintwelcome`, `mintupdate`, and `mintsystem`.
- **Current Behavior / Value:**
  - `RELEASE=22.3`
  - `CODENAME=zena`
  - `EDITION="Cinnamon"`
  - `DESCRIPTION="Linux Mint 22.3 Zena"`
  - `GRUB_TITLE=Linux Mint 22.3 Cinnamon`
- **Classification:** Mixed (Branding text, but required by Mint desktop tools).
- **Likely Effect:** Updates OS name in Mint System Reports, Welcome Screen, and Update Manager.
- **Risk Level:** Medium (Removing the file crashes `mintreport` and `mintwelcome`).
- **Dependencies:** `mint-info-cinnamon` package, `usr/lib/linuxmint/mintreport/app.py`, `usr/bin/mintwelcome-launcher`.
- **Package Provenance:** `mint-info-cinnamon`.
- **Recommended Treatment:** Retain file structure and key names. Update values: `RELEASE=1.0`, `CODENAME=aura`, `DESCRIPTION="Crown OS 1.0 Cinnamon"`, `GRUB_TITLE=Crown OS 1.0 Cinnamon`.
- **Verification Method:** Launch `mintreport` and inspect System Information tab.
- **Rollback Procedure:** Revert `/etc/linuxmint/info`.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-04: Central Artwork GSettings Schema Override (`mint-artwork.gschema.override`)
- **Exact Path:** `/usr/share/glib-2.0/schemas/mint-artwork.gschema.override`
- **Layer:** SquashFS Root
- **Current Purpose:** Defines the default desktop theme, icon theme, cursor, wallpaper, font sizes, Cinnamon applet icons, and Slick Greeter appearance.
- **Current Behavior / Value:**
  - `[x.dm.slick-greeter] theme-name='Mint-Y-Aqua'`, `icon-theme-name='Mint-Y-Sand'`, `background='/usr/share/backgrounds/linuxmint/default_background.jpg'`, `cursor-theme-name='Bibata-Modern-Classic'`
  - `[org.cinnamon.desktop.interface] gtk-theme='Mint-Y-Aqua'`, `icon-theme='Mint-Y-Sand'`, `font-name='Ubuntu 10'`
  - `[org.cinnamon.theme] name='Mint-Y-Dark-Aqua'`
  - `[org.cinnamon] app-menu-icon-name="linuxmint-logo-ring-symbolic"`, `system-icon="linuxmint-logo-ring-symbolic"`
  - `[org.cinnamon.desktop.background] picture-uri='file:///usr/share/backgrounds/linuxmint/default_background.jpg'`
- **Classification:** Cosmetic & Desktop Configuration Defaults
- **Likely Effect:** Determines out-of-the-box appearance for all new users, live session, and login greeter.
- **Risk Level:** Medium (Syntax error corrupts `gschemas.compiled`, resetting GNOME/Cinnamon to unstyled fallback).
- **Dependencies:** `glib-compile-schemas`, `cinnamon`, `slick-greeter`, `mint-artwork`.
- **Package Provenance:** `mint-artwork`.
- **Recommended Treatment:** Do NOT overwrite `mint-artwork.gschema.override` directly. Introduce `/usr/share/glib-2.0/schemas/99_crown-os.gschema.override` so that Crown OS settings take precedence naturally based on GLib override alphabetical evaluation order. Compile with `glib-compile-schemas`.
- **Verification Method:** Inspect with `gsettings get <schema> <key>` in test chroot/session.
- **Rollback Procedure:** Delete `99_crown-os.gschema.override` and recompile schemas.
- **Phasing Suitability:** Phase B (Colors/Themes) & Phase C (Desktop & Login).

---

### ITEM-FS-05: Cinnamon Favorite Applications Override (`10_cinnamon.gschema.override`)
- **Exact Path:** `/usr/share/glib-2.0/schemas/10_cinnamon.gschema.override`
- **Layer:** SquashFS Root
- **Current Purpose:** Defines default pinned panel launcher items.
- **Current Behavior / Value:** `favorite-apps=[ 'org.gnome.Calculator.desktop', 'org.gnome.Calendar.desktop', 'org.x.editor.desktop', 'mintinstall.desktop', 'cinnamon-settings.desktop' ]`
- **Classification:** Usability / Configuration
- **Likely Effect:** Changes default pinned icons on the Cinnamon panel.
- **Risk Level:** Low
- **Dependencies:** Cinnamon panel launcher applet, installed `.desktop` files.
- **Package Provenance:** `cinnamon`.
- **Recommended Treatment:** Override in `99_crown-os.gschema.override` with curated Crown OS favorites (e.g. File Manager, Web Browser, Terminal, Software Manager, Settings).
- **Verification Method:** Boot live session; inspect panel pinned launchers.
- **Rollback Procedure:** Remove override key.
- **Phasing Suitability:** Phase C.

---

### ITEM-FS-06: Cinnamon Menu Applet Settings Override (`menu@cinnamon.org/settings-override.json`)
- **Exact Path:** `/usr/share/cinnamon/applets/menu@cinnamon.org/settings-override.json`
- **Layer:** SquashFS Root
- **Current Purpose:** Hardcoded default icon and category configuration for the Cinnamon Main Application Menu.
- **Current Behavior / Value:**
  - `"menu-icon": { "default_category": "Linux Mint", "default": "linuxmint-logo-ring-symbolic", ... }`
- **Classification:** Cosmetic
- **Likely Effect:** Changes the icon displayed on the bottom-left panel menu button.
- **Risk Level:** Medium (Malformed JSON breaks menu applet loading).
- **Dependencies:** Cinnamon desktop session, `menu@cinnamon.org/applet.js`.
- **Package Provenance:** `cinnamon`.
- **Recommended Treatment:** Update `default` and `default_icon` to `crown-os-logo-symbolic`; provide Crown OS category entry in icon selector.
- **Verification Method:** Inspect Cinnamon bottom panel menu button icon.
- **Rollback Procedure:** Restore original `settings-override.json`.
- **Phasing Suitability:** Phase C.

---

### ITEM-FS-07: Mintsystem Dynamic Logo Adjustment Overwrites (`15-mint-logos.overwrite`)
- **Exact Path:** `/usr/share/linuxmint/adjustments/15-mint-logos.overwrite`
- **Layer:** SquashFS Root
- **Current Purpose:** Automatic boot-time service (`mintsystem.service` executing `mint-adjust.py`) that forcefully overwrites distributor logos in icon themes with Mint artwork.
- **Current Behavior / Value:** Copies `/usr/share/mint-artwork/logos/*` into:
  - `/usr/share/icons/*/16x16/places/distributor-logo.png`
  - `/usr/share/icons/*/22x22/places/distributor-logo.png`
  - `/usr/share/icons/*/24x24/places/distributor-logo.png`
  - `/usr/share/icons/*/32x32/places/distributor-logo.png`
  - `/usr/share/icons/*/48x48/places/distributor-logo.png`
  - `/usr/share/icons/*/scalable/places/distributor-logo.svg`
  - Along with `start-here.png` and `gnome-main-menu.png`.
- **Classification:** Functional / Branding Enforcement Mechanism
- **Likely Effect:** Replaces distributor logos across all installed icon themes on every boot.
- **Risk Level:** High (If overlooked, any custom distributor logo placed in an icon theme will be clobbered on boot by `mintsystem`!).
- **Dependencies:** `mintsystem` package, `/usr/lib/linuxmint/mintsystem/mint-adjust.py`, `mintsystem.service`.
- **Package Provenance:** `mint-artwork`.
- **Recommended Treatment:**
  - Option 1 (Recommended): Add higher-numbered adjustment `/usr/share/linuxmint/adjustments/90-crown-os-logos.overwrite` pointing to `/usr/share/crown-os/branding/logos/` so Crown OS logos are applied cleanly using Mint's own supported adjustment architecture.
  - Option 2: Add a `.preserve` entry in `/usr/share/linuxmint/adjustments/` to protect Crown OS icon files.
- **Verification Method:** Boot system, inspect `/var/log/mintsystem.log`, check destination files under `/usr/share/icons/`.
- **Rollback Procedure:** Remove custom adjustment file.
- **Phasing Suitability:** Phase B.

---

### ITEM-FS-08: Default Wallpaper Symlink & Assets (`/usr/share/backgrounds/linuxmint/`)
- **Exact Path:**
  - `/usr/share/backgrounds/linuxmint/default_background.jpg` (symlink to `sele_ring.jpg`)
  - Target: `/usr/share/backgrounds/linuxmint/sele_ring.jpg`
- **Layer:** SquashFS Root
- **Current Purpose:** Default desktop and login screen wallpaper.
- **Current Behavior / Value:** Points to Mint ring wallpaper.
- **Classification:** Cosmetic
- **Likely Effect:** Defines default desktop backdrop for live environment, new user accounts, and Slick Greeter.
- **Risk Level:** Low
- **Dependencies:** `mint-artwork.gschema.override` (`picture-uri` and `background`), Slick Greeter.
- **Package Provenance:** `mint-backgrounds-wallpapers` / `mint-artwork`.
- **Recommended Treatment:** Place Crown OS high-resolution wallpapers in `/usr/share/backgrounds/crown-os/` (e.g. `crown-emerald-light.jpg` and `crown-emerald-dark.jpg`). Update the GSettings override `picture-uri` to point to the new asset.
- **Verification Method:** Verify resolution, color rendering, and contrast with desktop icons.
- **Rollback Procedure:** Point GSettings back to `/usr/share/backgrounds/linuxmint/default_background.jpg`.
- **Phasing Suitability:** Phase B.

---

### ITEM-FS-09: Plymouth Boot Splash Theme (`mint-logo.plymouth`)
- **Exact Path:**
  - `/usr/share/plymouth/themes/mint-logo/mint-logo.plymouth`
  - Active Link: `/etc/alternatives/default.plymouth -> /usr/share/plymouth/themes/mint-logo/mint-logo.plymouth`
- **Layer:** SquashFS Root (and bundled inside `/boot/initrd.img-*` and outer `/casper/initrd.lz`)
- **Current Purpose:** Displays graphical boot splash and spinner during system startup and shutdown.
- **Current Behavior / Value:** Uses `two-step` module, displays 36-frame Mint logo pulse animation (`animation-0001.png` to `0036.png`), throbber animation (`throbber-0001.png` to `0030.png`), and `watermark.png`.
- **Classification:** Mixed (Cosmetic presentation, but initramfs regeneration is functional/high risk).
- **Likely Effect:** Transforms the visual boot splash from Linux Mint to Crown OS.
- **Risk Level:** High (Errors during initramfs update can produce an unbootable kernel image).
- **Dependencies:** `plymouth`, `initramfs-tools`, `/casper/initrd.lz`.
- **Package Provenance:** `plymouth-theme-mint-logo` / `mint-artwork`.
- **Recommended Treatment:**
  - Create `/usr/share/plymouth/themes/crown-logo/` modeled after `two-step`.
  - Provide Crown OS emerald spinner / crown logo assets.
  - Set active alternative using `update-alternatives --set default.plymouth /usr/share/plymouth/themes/crown-logo/crown-logo.plymouth`.
  - Regenerate initramfs in chroot (`update-initramfs -u`) and synchronize to outer ISO `/casper/initrd.lz`.
- **Verification Method:** Test boot in QEMU without `quiet` to verify transition into Plymouth and out to X11.
- **Rollback Procedure:** Reset alternative to `mint-logo.plymouth` and re-run `update-initramfs -u`.
- **Phasing Suitability:** Phase D.

---

### ITEM-FS-10: Slick Greeter Display Manager Configuration (`slick-greeter.gschema.xml`)
- **Exact Path:**
  - Configuration: `/usr/share/glib-2.0/schemas/mint-artwork.gschema.override` (`[x.dm.slick-greeter]`)
  - Optional File: `/etc/lightdm/slick-greeter.conf`
- **Layer:** SquashFS Root
- **Current Purpose:** Configures login screen appearance (background, logo, badge, fonts, themes).
- **Current Behavior / Value:** Theme `Mint-Y-Aqua`, background `/usr/share/backgrounds/linuxmint/default_background.jpg`, font `Ubuntu 11`.
- **Classification:** Cosmetic / Greeter Presentation
- **Likely Effect:** Customizes the LightDM login screen.
- **Risk Level:** Medium (Misconfiguration can cause greeter crash, dropping to console prompt).
- **Dependencies:** `lightdm`, `slick-greeter`, `liblightdm-gobject-1-0`.
- **Package Provenance:** `slick-greeter`.
- **Recommended Treatment:** Configure via `/etc/lightdm/slick-greeter.conf` or `99_crown-os.gschema.override`:
  - `background=/usr/share/backgrounds/crown-os/crown-greeter.jpg`
  - `logo=/usr/share/crown-os/branding/crown-logo-text.svg`
  - `theme-name=Mint-Y-Teal` or Crown OS theme
- **Verification Method:** Test `lightdm --test-mode --debug` in nested X server or reboot installed system.
- **Rollback Procedure:** Remove custom `slick-greeter.conf` and revert GSettings keys.
- **Phasing Suitability:** Phase C.

---

### ITEM-FS-11: Ubiquity Installer Slideshow Assets (`ubiquity-slideshow/slides/`)
- **Exact Path:** `/usr/share/ubiquity-slideshow/slides/`
  - Key files: `welcome.html`, `customize.html`, `software.html`, `help.html`, `directory.jsonp`, `link/base.css`, `screenshots/`
- **Layer:** SquashFS Root
- **Current Purpose:** Web-based HTML/CSS carousel rendered by WebKitGTK inside Ubiquity while packages copy.
- **Current Behavior / Value:** Contains "Welcome to Linux Mint", Mint community links, and screenshots of Mint desktop.
- **Classification:** Cosmetic / Informational
- **Likely Effect:** Shows Crown OS feature highlights, documentation links, and artwork during system installation.
- **Risk Level:** Low to Medium (Broken HTML/JS may stop the carousel from rotating, but does not abort installation).
- **Dependencies:** `ubiquity-frontend-gtk`, `ubiquity-slideshow-mint`, WebKitGTK.
- **Package Provenance:** `ubiquity-slideshow-mint`.
- **Recommended Treatment:**
  - Replace slide HTML text with Crown OS introduction and user guides.
  - Provide Crown OS desktop screenshots in `screenshots/`.
  - Update `base.css` styling to match the Emerald/Teal design palette.
- **Verification Method:** Launch Ubiquity in live session test; verify slide progression.
- **Rollback Procedure:** Reinstall or restore original `slides/` directory.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-12: Live Session Desktop Installer Launcher (`ubiquity.desktop`)
- **Exact Path:**
  - Base: `/usr/share/applications/ubiquity.desktop`
  - Generation Hook: `/usr/share/initramfs-tools/scripts/casper-bottom/25adduser`
- **Layer:** SquashFS Root
- **Current Purpose:** Creates the desktop launcher that launches Ubiquity installer on the live desktop.
- **Current Behavior / Value:** Contains marker `Name=Install RELEASE`. On boot, `casper-bottom/25adduser` reads `.disk/info` and performs `sed -i "s/RELEASE/$RELEASE/"`, placing the resulting launcher on `/home/mint/Desktop/ubiquity.desktop` and setting `metadata::trusted true`.
- **Classification:** Mixed (Functional hook with cosmetic label).
- **Likely Effect:** Modifies the live session installer launcher name and icon.
- **Risk Level:** Medium (Modifying the file incorrectly can break desktop trust metadata).
- **Dependencies:** `casper`, `.disk/info`, `gio`, `dbus-run-session`.
- **Package Provenance:** `ubiquity`.
- **Recommended Treatment:** Keep the `# Do not translate the word "RELEASE"` structure in `/usr/share/applications/ubiquity.desktop`. Control the resulting title cleanly by setting `.disk/info` to `Crown OS 1.0 "Aura" ...`. If custom icon is desired, update `Icon=ubiquity` or supply Crown OS installer icon.
- **Verification Method:** Boot live ISO; verify presence, trust badge, and click-to-launch behavior of installer shortcut.
- **Rollback Procedure:** Restore original `ubiquity.desktop`.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-13: System Information Utility (`mintreport` / `app.py`)
- **Exact Path:**
  - `/usr/lib/linuxmint/mintreport/app.py`
  - `/usr/share/linuxmint/mintreport/mintreport.ui`
- **Layer:** SquashFS Root
- **Current Purpose:** Provides the "System Information" dialog accessed from Cinnamon System Settings.
- **Current Behavior / Value:**
  - Line 364 of `app.py` hardcodes `distribution = "Linux Mint"` regardless of `/etc/os-release`!
  - `mintreport.ui` line 76 references icon `<property name="icon-name">linuxmint-logo-filled-ring</property>`.
- **Classification:** Functional / Hardcoded Presentation
- **Likely Effect:** If only `/etc/linuxmint/info` is edited, System Info will still display "Linux Mint 1.0 - Cinnamon 64-bit".
- **Risk Level:** Medium (Requires code patch to `app.py` or wrapper to correctly report Crown OS).
- **Dependencies:** `mintreport` package, Python 3, GTK 3, `/etc/linuxmint/info`.
- **Package Provenance:** `mintreport`.
- **Recommended Treatment:** Apply a minimal, clean patch to `usr/lib/linuxmint/mintreport/app.py` line 364 to query `/etc/os-release` `NAME` if present, falling back to `config.get('DISTRIBUTION', 'Crown OS')`. Update `mintreport.ui` icon name to `distributor-logo` or `crown-os-logo`.
- **Verification Method:** Launch `mintreport` -> System Information tab; verify reported distribution name and logo.
- **Rollback Procedure:** Revert patched `app.py`.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-14: Welcome Screen Application (`mintwelcome`)
- **Exact Path:**
  - `/usr/bin/mintwelcome-launcher`
  - `/usr/share/linuxmint/mintwelcome/mintwelcome.ui`
  - `/etc/xdg/autostart/mintwelcome.desktop`
- **Layer:** SquashFS Root
- **Current Purpose:** First-boot onboarding dialog shown to newly installed users.
- **Current Behavior / Value:** Checked by `mintwelcome-launcher` (suppressed in live session, launched on first installed boot). Displays Mint logo (`linuxmint-logo-ring-symbolic`), Mint forum links, and documentation links.
- **Classification:** Cosmetic & Usability
- **Likely Effect:** Provides onboarding instructions tailored to Crown OS users.
- **Risk Level:** Low to Medium
- **Dependencies:** `mintwelcome` package, `xapp.os`, Python 3.
- **Package Provenance:** `mintwelcome`.
- **Recommended Treatment:**
  - Update `mintwelcome.ui` to display Crown OS branding and updated welcome copy.
  - Point documentation links to Crown OS resources.
- **Verification Method:** Launch `mintwelcome` from terminal; verify layout and links.
- **Rollback Procedure:** Restore original `mintwelcome.ui`.
- **Phasing Suitability:** Phase E.

---

### ITEM-FS-15: Scalable Distributor and System Icons (`/usr/share/icons/hicolor/scalable/apps/`)
- **Exact Path:** `/usr/share/icons/hicolor/scalable/apps/linuxmint-logo-*.svg`
- **Layer:** SquashFS Root
- **Current Purpose:** Source SVG icons for system menus, welcome dialogs, panels, and reports.
- **Current Behavior / Value:** 21 SVG files supplying variants of the Mint logo (`linuxmint-logo-ring-symbolic`, `linuxmint-logo-filled-ring`, etc.).
- **Classification:** Cosmetic
- **Likely Effect:** Powers vector rendering of distributor logos across the environment.
- **Risk Level:** Low
- **Dependencies:** `mint-artwork`, `gtk-update-icon-cache`.
- **Package Provenance:** `mint-artwork`.
- **Recommended Treatment:** Supply corresponding Crown OS SVGs in `/usr/share/icons/hicolor/scalable/apps/` (or dedicated Crown OS theme directory), and update icon cache with `gtk-update-icon-cache -f -t /usr/share/icons/hicolor`.
- **Verification Method:** Check rendering in Cinnamon Menu, System Info, and file manager.
- **Rollback Procedure:** Re-extract original SVGs.
- **Phasing Suitability:** Phase B.

---

### ITEM-FS-16: Official Package Repositories Configuration (`/etc/apt/sources.list.d/official-package-repositories.list`)
- **Exact Path:** `/etc/apt/sources.list.d/official-package-repositories.list`
- **Layer:** SquashFS Root
- **Current Purpose:** Primary APT repository definitions for security fixes, core updates, and Mint software.
- **Current Behavior / Value:**
  - `deb http://packages.linuxmint.com zena main upstream import backport`
  - `deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse`
  - `deb http://security.ubuntu.com/ubuntu/ noble-security ...`
- **Classification:** Functional / Core System Maintenance (PROTECTED)
- **Likely Effect:** Breaking or removing these lines prevents security updates, kernel updates, and software installations.
- **Risk Level:** Critical
- **Dependencies:** `apt`, `dpkg`, Ubuntu archive keys, Mint keyring.
- **Package Provenance:** `mintsystem`.
- **Recommended Treatment:** LEAVE UNCHANGED during early customization phases. Preserving upstream repositories guarantees full package compatibility, security patches, and hardware support.
- **Verification Method:** Run `apt-get update --simulate`.
- **Rollback Procedure:** Restore official repo list.
- **Phasing Suitability:** Phase G / Future maintenance (DO NOT ALTER IN PHASES A–F).

---

### ITEM-FS-17: APT Pinning Rules (`/etc/apt/preferences.d/official-package-repositories.pref`)
- **Exact Path:** `/etc/apt/preferences.d/official-package-repositories.pref`
- **Layer:** SquashFS Root
- **Current Purpose:** Blocks `snapd` installation and prioritizes Mint packages over Ubuntu packages (Pin-Priority: 700).
- **Current Behavior / Value:**
  - `Package: snapd / Pin: release a=* / Pin-Priority: -10`
  - `Package: * / Pin: origin "build.linuxmint.com" / Pin-Priority: 700`
- **Classification:** Functional / Package Management Policy (PROTECTED)
- **Likely Effect:** Modifying priorities can pull conflicting Ubuntu packages or accidentally install unwanted snap daemons.
- **Risk Level:** Critical
- **Dependencies:** `apt`, `dpkg`.
- **Package Provenance:** `mintsystem`.
- **Recommended Treatment:** PRESERVE UNCHANGED.
- **Verification Method:** Run `apt-cache policy`.
- **Rollback Procedure:** Restore original preference file.
- **Phasing Suitability:** Protected / Unchanged.

---

## 4. Summary Matrix of Audited Items

| ID | Exact Path | Layer | Class | Risk | Target Phase | Rollback Complexity |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **ITEM-ISO-01** | PVD Volume Label (`-V`) | Outer ISO | Cosmetic | Low | Phase F | Trivial (xorriso arg) |
| **ITEM-ISO-02** | `.disk/info` | Outer ISO | Mixed | Medium | Phase E | Trivial (text file) |
| **ITEM-ISO-03** | `.disk/README.diskdefines` | Outer ISO | Cosmetic | Low | Phase E | Trivial (text file) |
| **ITEM-ISO-04** | `.disk/release_notes_url` | Outer ISO | Cosmetic | Low | Phase E | Trivial (text file) |
| **ITEM-ISO-05** | `.disk/mint_iso` | Outer ISO | Info | Low | Phase E | Trivial (file restore) |
| **ITEM-ISO-06** | `boot/grub/grub.cfg` | Outer ISO | Mixed | High | Phase D | Low (file revert) |
| **ITEM-ISO-07** | `boot/grub/live-theme/theme.txt` | Outer ISO | Cosmetic | Medium | Phase D | Low (file revert) |
| **ITEM-ISO-08** | `isolinux/live.cfg` | Outer ISO | Mixed | High | Phase D | Low (file revert) |
| **ITEM-ISO-09** | `isolinux/splash.png` | Outer ISO | Cosmetic | Low | Phase D | Low (file revert) |
| **ITEM-ISO-10** | `md5sum.txt` | Outer ISO | Functional | High | Phase F | Medium (recalculate) |
| **ITEM-ISO-11** | `casper/filesystem.size` | Outer ISO | Functional | High | Phase F | Medium (re-measure) |
| **ITEM-FS-01** | `/etc/os-release` | SquashFS | Mixed | High | Phase E | Low (file revert) |
| **ITEM-FS-02** | `/etc/lsb-release` | SquashFS | Mixed | Medium | Phase E | Low (file revert) |
| **ITEM-FS-03** | `/etc/linuxmint/info` | SquashFS | Mixed | Medium | Phase E | Low (file revert) |
| **ITEM-FS-04** | `mint-artwork.gschema.override` | SquashFS | Config | Medium | Phase B/C | Low (add 99_crown.override) |
| **ITEM-FS-05** | `10_cinnamon.gschema.override` | SquashFS | Config | Low | Phase C | Low (override in 99_) |
| **ITEM-FS-06** | `menu@cinnamon.org/settings-override.json` | SquashFS | Cosmetic | Medium | Phase C | Low (file revert) |
| **ITEM-FS-07** | `15-mint-logos.overwrite` | SquashFS | Functional | High | Phase B | Low (add 90-crown.overwrite) |
| **ITEM-FS-08** | `/usr/share/backgrounds/linuxmint/` | SquashFS | Cosmetic | Low | Phase B | Low (re-point override) |
| **ITEM-FS-09** | `mint-logo.plymouth` | SquashFS | Mixed | High | Phase D | Medium (update-alternatives & initramfs) |
| **ITEM-FS-10** | `slick-greeter` settings | SquashFS | Cosmetic | Medium | Phase C | Low (file revert) |
| **ITEM-FS-11** | `ubiquity-slideshow/slides/` | SquashFS | Cosmetic | Low | Phase E | Low (file revert) |
| **ITEM-FS-12** | `ubiquity.desktop` & casper hook | SquashFS | Mixed | Medium | Phase E | Low (file revert) |
| **ITEM-FS-13** | `mintreport/app.py` & `.ui` | SquashFS | Mixed | Medium | Phase E | Low (revert patch) |
| **ITEM-FS-14** | `mintwelcome` & `.ui` | SquashFS | Cosmetic | Low | Phase E | Low (file revert) |
| **ITEM-FS-15** | `scalable/apps/linuxmint-logo-*.svg` | SquashFS | Cosmetic | Low | Phase B | Low (re-extract) |
| **ITEM-FS-16** | `sources.list.d/official-*.list` | SquashFS | Functional | Critical | Protected | N/A (Do not alter) |
| **ITEM-FS-17** | `preferences.d/official-*.pref` | SquashFS | Functional | Critical | Protected | N/A (Do not alter) |
