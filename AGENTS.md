# AGENTS.md — DELL-5488 Hackintosh (OpenCore)

## Repo purpose
OpenCore EFI configuration for Dell Inspiron 5488 (i5-8265u) running macOS.
Contains ACPI SSDTs, kexts, drivers, and config.plist.

## Key directories
- `EFI/OC/config.plist` — main OpenCore configuration
- `EFI/OC/ACPI/`, `EFI/OC/Drivers/`, `EFI/OC/Kexts/` — respectively ACPI tables, boot drivers, and kernel extensions
- `EFI/OC/Tools/` — OpenCore tools (OpenShell, ResetSystem, CleanNvram)

## Editing config.plist
- Use **PlistEdit Pro** or `plistbuddy` / Python `plistlib` for edits
- Most kexts are configured via `config.plist Kernel -> Add` entries (BundlePath, ExecutablePath, PlistPath)
- Kext ordering matters: Lilu loads first, then VirtualSMC, then family kexts (AppleALC, WhateverGreen, etc.)
- `Kernel -> Quirks -> DisableIoMapper = true` and `DisableIoMapperMapping = false` are set
- `Boot -> Quirks -> ProvideCustomSlide = true`, `RebuildAppleMemoryMap = true`, `EnableSafeModeSlide = true`

## ACPI SSDTs (ACPI/ directory)
- `SSDT-EC-USBX-LAPTOP.aml` — must be in Add
- `SSDT-PNLF.aml` — backlight brightness control
- `SSDT-AWAC.aml` — AWAC/EC power management patch (_WAK→ZWAK, _PTS→ZPTS)
- `SSDT-OCWork-dell.aml` — Dell-specific quirks
- `SSDT-BKeyBRT6-Dell.aml` — Fn+Brightness control for Dell
- `SSDT-FnInsert_BTNV-dell.aml` — Fn+Insert sleep
- `SSDT-LIDpatch.aml` — lid open-to-wake
- `SSDT-GPRW.aml` — GPRW→XPRW replacement
- `SSDT-SBUS.aml` — SBUS to system
- `SSDT-I2C.aml` — trackpad interrupts mode
- `SSDT-EXT4-WakeScreen.aml` — wake screen without keys
- `SSDT-PLUG-_SB.PR00.aml` — native PM
- `SSDT-MCHC.aml` — _MCHC patch
- `SSDT-DMAC.aml`, `SSDT-MEM2.aml`, `SSDT-PPMC.aml`, `SSDT-PTSWAK.aml`

## Kexts (Kexts/ directory)
- **Lilu.kext** — patch engine (required by all other third-party kexts)
- **VirtualSMC.kext** — SMC emulator
- **AppleALC.kext** — audio (Realtek ALC236, layout-id 68; see ComboJack below)
- **WhateverGreen.kext** — graphics (Intel UHD 620)
- **SMCBatteryManager.kext** — battery reporting
- **SMCDellSensors.kext** — Dell sensor monitoring
- **SMCProcessor.kext** — CPU monitoring
- **NoTouchID.kext** — disables Touch ID
- **RealtekRTL8100.kext** — wired Ethernet
- **VoodooI2C/VoodooI2CHID** — I2C devices
- **VoodooPS2Controller** — keyboard/trackpad
- **USBPorts.kext** — custom USB mapping

## Audio / combo jack (headset mic)
- Codec: Realtek **ALC236** (`0x10ec0236`, rev `0x100002`, Dell subsystem `0x1028089c`)
- Uses **layout-id 68** (`ALC236 for Dell, use with ComboJack`), set under `DeviceProperties -> PciRoot(0x0)/Pci(0x1f,0x3)`:
  - `layout-id` = `44000000` (68 LE)
  - `alc-verbs` = `01000000` (enables AppleALC verb interface, equivalent to boot-arg `alcverbs=1`)
- Layout 54 (the old DELL-5488 layout) does NOT map the combo-jack headset mic (pin 0x19 disabled) — keep 68.
- **ComboJack** (in `ComboJack/`): user-space daemon + LaunchDaemon that pops up a Headphone/Headset chooser when a headset is plugged in. It sends HDA verbs (pin 0x19) through AppleALC's `alcverbs` interface — no extra kext needed.
  - Install on the target machine: `sudo ./ComboJack/install.command` (writes to `/usr/local/bin/ComboJack`, `/usr/local/share/ComboJack/`, `/Library/LaunchDaemons/com.ComboJack.plist`)
  - Uninstall: `sudo ./ComboJack/uninstall.command`
  - Requires reboot; must be reinstalled after macOS reinstall.

## Common edit patterns
- To add a new SSDT: place `.aml` in `EFI/OC/ACPI/`, add entry to `config.plist ACPI -> Add` with `Path` and `Comment`
- To enable/disable a kext: toggle `Enabled` boolean in `config.plist Kernel -> Add`
- To change framebuffer settings: modify `config.plist DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0)` entries
- To change boot timeout: modify `Misc -> Boot -> Timeout` in config.plist

## Build / verification
- No formal build system — OpenCore is loaded by the firmware
- Validate config.plist with `plistbuddy` or online plist validators
- Ensure SSDTs compile with `iasl` if modified: `iasl -tc SSDT-Filename.dsl`
- OpenCanopy.efi provides the boot picker; can be replaced with OpenCore.efi alone if desired

## Workflow notes
- This is a static EFI package — changes require a reboot to take effect
- Always keep backups of the original `EFI/OC/` before editing
- If booting fails, use `OpenShell.efi` or `CleanNvram.efi` from the Tools menu
- `ResetSystem.efi` can be used to reset firmware state