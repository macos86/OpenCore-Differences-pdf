# Differences pdf

**Failsafe**: 0

**Description**: Maximum number of bytes to search for. Can be set to 0 to look through the whole ACPI table.
  
* **6. Mask**

   - **Type**: plist data
   - **Failsafe**: Empty data
   - **Description**: Data bitwise mask used during find comparison. Allows fuzzy search by ignoring not masked (set to zero) bits. Can be set to empty data to be ignored. Must equal to Replace in size otherwise.
   
* **7. OemTableId**

   - **Type**: plist data, 8 bytes
   
   - **Failsafe**: All zero
   
   - **Description**: Match table OEM ID to be equal to this value unless all zero.

* **8. Replace**
   
   - **Type**: plist data
   
   - **Failsafe**: Empty data
   
   - **Description**: Replacement data of one or more bytes.

* **9. ReplaceMask**
   
   - **Type**: plist data
   
   - **Failsafe**: Empty data
   
   - **Description:** Data bitwise mask used during replacement. Allows fuzzy replacement by updating masked (set to non-zero) bits. Can be set to empty data to be ignored. Must equal to Replace in size otherwise.

* **10. Skip**

   - **Type** plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Number of found occurrences to be skipped before replacement is done.

* **11. TableLength**
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Match table size to be equal to this value unless 0.

* **12. TableSignature**
   
   - **Type**: plist data, 4 bytes
   
   - **Failsafe**: All zero
   
   - **Description**: Match table signature to be equal to this value unless all zero.
   
In the majority of the cases ACPI patches are not useful and harmful:
   
   - Avoid renaming devices with ACPI patches. This may fail or perform improper renaming of unrelated devices (e.g. EC and EC0), be unnecessary, or even fail to rename devices in select tables. For ACPI consistency it is much
   safer to rename devices at I/O Registry level, as done by WhateverGreen.
   
   - Avoid patching _OSI to support a higher level of feature sets unless absolutely required. Commonly this enables a number of hacks on APTIO firmwares, which result in the need to add more patches. Modern firmwares generally do not need it at all, and those that do are fine with much smaller patches.
   - Try to avoid hacky changes like renaming _PRW or _DSM whenever possible. Several cases, where patching actually does make sense, include:
   
   - Refreshing HPET (or another device) method header to avoid compatibility checks by _OSI on legacy hardware. _STA method with `if ((OSFL () == Zero)) { If (HPTE) ... Return (Zero)` content may be forced to alwaysreturn 0xF by replacing `A0 10 93 4F 53 46 4C 00` with `A4 0A 0F A3 A3 A3 A3 A3.
   
   - To provide custom method implementation with in an SSDT, for instance, to report functional key presses on a laptop, the original method can be replaced with a dummy name by patching _Q11 with XQ11.

Tianocore AcpiAml.h source file may help understanding ACPI opcodes.

* **5. DiscardHibernateMap**
   
   - Type: plist boolean
   
   - Failsafe: false
   
   - Description: Reuse original hibernate memory map.

This option forces XNU kernel to ignore newly supplied memory map and assume that it did not change after waking from hibernation. This behaviour is required to work by Windows, which mandates to preserve runtime memory size and location after S4 wake.

**Note**: This may be used to workaround buggy memory maps on older hardware, and is now considered rare legacy. Examples of such hardware are Ivy Bridge laptops with Insyde firmware, like Acer V3-571G. Do not use this unless you fully understand the consequences.

* **6. EnableSafeModeSlide**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false

   - Description: Patch bootloader to have KASLR enabled in safe mode.

This option is relevant to the users that have issues booting to safe mode (e.g. by holding shift or using -x boot argument). By default safe mode forces 0 slide as if the system was launched with slide=0 boot argument. This quirk tries to patch boot.efi to lift that limitation and let some other value (from 1 to 255) be used. This quirk requires ProvideCustomSlide to be enabled.

**Note**: The necessity of this quirk is determined by safe mode availability. If booting to safe mode fails, this option can be tried to be enabled.

* **7. EnableWriteUnprotector**

   - Type: plist boolean
   
   - Failsafe: false
   
   - Description: Permit write access to UEFI runtime services code.

This option bypasses RˆX permissions in code pages of UEFI runtime services by removing write protection (WP) bit from CR0 register during their execution. This quirk requires `OC_FIRMWARE_RUNTIME` protocol implemented in FwRuntimeServices.efi.

**Note**: The necessity of this quirk is determined by early boot crashes of the firmware.

* **8. ForceExitBootServices**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Descriptio**n: Retry ExitBootServices with new memory map on failure.

Try to ensure that ExitBootServices call succeeds even with outdated MemoryMap key argument by obtaining current memory map and retrying ExitBootServices call.

**Note**: The necessity of this quirk is determined by early boot crashes of the firmware. Do not use this unless you fully understand the consequences.

* **9. ProtectCsmRegion**
   
   - Type: plist boolean
   
   - Failsafe: false
   
   - Description: Protect CSM region areas from relocation.

Ensure that CSM memory regions are marked as ACPI NVS to prevent boot.efi boot.efi or XNU from relocating 
   or using them.

**Note**: The necessity of this quirk is determined by artifacts and sleep wake issues. As AvoidRuntimeDefrag resolves a similar problem, no known firmwares should need this quirk. Do not use this unless you fully understand the consequences.

* **10. ProtectSecureBoot**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Protect UEFI Secure Boot variables from being written.

Reports security violation during attempts to write to db, dbx, PK, and KEK variables from the operating system. Note: This quirk mainly attempts to avoid issues with NVRAM implementations with problematic defragmentation,
   such as select Insyde or MacPro5,1. 11. 
   
* **ProtectUefiServices**
   
   - **Type**: plist boolean 
   
   - **Failsafe**: false 
   
   - **Description**: Protect UEFI services from being overridden by the firmware. 
   
Some modern firmwares including both hardware and virtual machines, like VMware, may update pointers to UEFI services during driver loading and related actions. Consequentially this directly breaks other quirks that
   affect memory management, like DevirtualiseMmio, ProtectCsmRegion, or ShrinkMemoryMap, and may also 
   break other quirks depending on the effects of these.
   
**Note**: On VMware the need for this quirk may be diagnosed by “Your Mac OS guest might run unreliably with 
   more than one virtual core.” message.


* **12. ProvideCustomSlide**
   
   - Type: plist boolean
   
   - Failsafe: false
   
   - Description: Provide custom KASLR slide on low memory.

This option performs memory map analysis of your firmware and checks whether all slides (from 1 to 255) can be used. As boot.efi generates this value randomly with rdrand or pseudo randomly rdtsc, there is a chance of boot failure when it chooses a conflicting slide. In case potential conflicts exist, this option forces macOS to use a pseudo random value among the available ones. This also ensures that slide= argument is never passed to the operating system for security reasons.

**Note**: The necessity of this quirk is determined by OCABC: Only N/256 slide values are usable! message in the debug log. If the message is present, this option is to be enabled.

* **13. SetupVirtualMap**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Setup virtual memory at SetVirtualAddresses.

Select firmwares access memory by virtual addresses after SetVirtualAddresses call, which results in early boot crashes. This quirk workarounds the problem by performing early boot identity mapping of assigned virtual addresses to physical memory.

**Note**: The necessity of this quirk is determined by early boot failures. Currently new firmwares with memory protection support (like OVMF) do not support this quirk due to acidanthera/bugtracker#719.

* **14. ShrinkMemoryMap**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Attempt to join similar memory map entries.

Select firmwares have very large memory maps, which do not fit Apple kernel, permitting up to 64 slots for runtime memory. This quirk attempts to unify contiguous slots of similar types to prevent boot failures.

**Note**: The necessity of this quirk is determined by early boot failures. It is rare to need this quirk on Haswell or newer. Do not use unless you fully understand the consequences.

**15. SignalAppleOS**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Report macOS being loaded through OS Info for any OS.

This quirk is useful on Mac firmwares, which behave differently in different OS. For example, it is supposed to enable Intel GPU in Windows and Linux in some dual-GPU MacBook models.

# 6 DeviceProperties

* ** 6.1 Introduction**
   Device configuration is provided to macOS with a dedicated buffer, called EfiDevicePropertyDatabaseEfiDevicePathPropertyDat 
   This buffer is a serialised map of DevicePaths to a map of property names and their values.
   Property data can be debugged with gfxutil. To obtain current property data use the following command in macOS:

```   
   ioreg -lw0 -p IODeviceTree -n efi -r -x | grep device-properties | sed 's/.*<//;s/>.*//' > /tmp/device-properties.hex &&
   gfxutil /tmp/device-properties.hex /tmp/device-properties.plist && cat /tmp/device-properties.plist
```

* **6.2 Properties**
      
   1.Add
   
   - **Type**: plist dict
   
   - **Description**: Sets device properties from a map (plist dict) of deivce paths to a map (plist dict) of variable names and their values in plist metadata format. Device paths must be provided in canonic string format (e.g. `PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0))`. Properties will only be set if not present and not blocked.

**Note**: Currently properties may only be (formerly) added by the original driver, so unless a separate driver was installed, there is no reason to block the variables.
   
   2.Block
   
   - **Type**: plist dict
   
   - **Description**: Removes device properties from a map (plist dict) of deivce paths to an array (plist array) of variable names in plist string format.
   
* **6.3 Common Properties**
   Some known properties include:
  
   - **device-id**
   User-specified device identifier used for I/O Kit matching. Has 4 byte data type.
   
   - **vendor-id**
   User-specified vendor identifier used for I/O Kit matching. Has 4 byte data type.
   
   - **AAPL,ig-platform-id**
   Intel GPU framebuffer identifier used for framebuffer selection on Ivy Bridge and newer. Has 4 byte data type.
   
   - **AAPL,snb-platform-id**
   Intel GPU framebuffer identifier used for framebuffer selection on Sandy Bridge. Has 4 byte data type.
   
   - **layout-id**
   Audio layout used for AppleHDA layout selection. Has 4 byte data type.
   17

   - Entry is macOS recovery.
   
   - Entry is macOS Time Machine.
   
   - Entry is explicitly marked as Auxiliary. 
   
   - Entry is system (e.g. Clean NVRAM).
   To see all entries picker menu needs to be reloaded in extended mode by pressing Spacebar key. Hiding auxiliary
   entries may increase boot performance for multidisk systems.

* **3. HideSelf**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Hides own boot entry from boot picker. This may potentially hide other entries, for instance, when another UEFI OS is installed on the same volume and driver boot is used.

* **4. PickerAttributes**
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Sets specific attributes for picker.
Builtin picker supports colour arguments as a sum of foreground and background colors according to UEFI specification. The value of black background and black foreground (0) is reserved. List of colour names:
``` 
   • 0x00 — EFI_BLACK
   • 0x01 — EFI_BLUE
   • 0x02 — EFI_GREEN
   • 0x03 — EFI_CYAN
   • 0x04 — EFI_RED
   • 0x05 — EFI_MAGENTA
   • 0x06 — EFI_BROWN
   • 0x07 — EFI_LIGHTGRAY
   • 0x08 — EFI_DARKGRAY
   • 0x09 — EFI_LIGHTBLUE
   • 0x0A — EFI_LIGHTGREEN 
   • 0x0B — EFI_LIGHTCYAN
   • 0x0C — EFI_LIGHTRED
   • 0x0D — EFI_LIGHTMAGENTA 
   • 0x0E — EFI_YELLOW
   • 0x0F — EFI_WHITE
   • 0x00 — EFI_BACKGROUND_BLACK
   • 0x10 — EFI_BACKGROUND_BLUE
   • 0x20 — EFI_BACKGROUND_GREEN
   • 0x30 — EFI_BACKGROUND_CYAN
   • 0x40 — EFI_BACKGROUND_RED
   • 0x50 — EFI_BACKGROUND_MAGENTA 
   • 0x60 — EFI_BACKGROUND_BROWN
   • 0x70 — EFI_BACKGROUND_LIGHTGRAY
```

**Note**: This option may not work well with System text renderer. Setting a background different from black could help testing proper GOP functioning.

* **5. PickerAudioAssist**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Enable screen reader by default in boot picker.

For macOS bootloader screen reader preference is set in preferences.efires archive in isVOEnabled.int32 file and is controlled by the operating system. For OpenCore screen reader support this option is an independent equivalent. Toggling screen reader support in both OpenCore boot picker and macOS bootloader FileVault 2 login window can also be done with Command + F5 key combination.

**Note**: screen reader requires working audio support, see UEFI Audio Properties section for more details.

   - Default — this is the default option, and it lets OpenCore built-in boot picker to loads the default boot option as specified in Startup Disk preference pane.
   
   - ShowPicker — this option forces picker to show. Normally it can be achieved by holding OPT key during boot. Setting ShowPicker to true will make ShowPicker the default option.
   
   - ResetNvram — this option performs select UEFI variable erase and is normally achieved by holding CMD+OPT+P+R key combination during boot. Another way to erase UEFI variables is to choose Reset NVRAM in the picker. This option requires AllowNvramReset to be set to true.
   
   - BootApple — this options performs booting to the first found Apple operating system unless the default chosen operating system is already made by Apple. Hold X key to choose this option.
   - BootAppleRecovery — this option performs booting to Apple operating system recovery. Either the one related to the default chosen operating system, or first found in case default chosen operating system is not made by Apple or has no recovery. Hold CMD+R key combination to choose this option.

**Note 1**: Activated KeySupport, AppleUsbKbDxe, or similar driver is required for key handling to work. On many firmwares it is not possible to get all the keys function.

**Note 2**: In addition to OPT OpenCore supports Escape key to display picker when ShowPicker is disabled. This key exists for Apple picker mode and for firmwares with PS/2 keyboards that fail to report held OPT key and require continual presses of Escape key to enter the boot menu.

**Note 3**: On Macs with problematic GOP it may be difficult to access Apple BootPicker. To workaround this problem even without loading OpenCore BootKicker utility can be blessed.

* **8.4 Debug Properties**

1. AppleDebug
   
   - **Type**: plist boolean 
   
   - **Failsafe**: false 
   
   - **Description**: Enable boot.efi debug log saving to OpenCore log. 

**Note**: This option only applies to 10.15.4 and newer. 

* **2. DisableWatchDog**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Select firmwares may not succeed in quickly booting the operating system, especially in debug mode, which results in watch dog timer aborting the process. This option turns off watch dog timer.

* **3. DisplayDelay**
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Delay in microseconds performed after every printed line visible onscreen (i.e. console).

* **4. DisplayLevel**
   
   - **Type**: plist integer, 64 bit
   
   - **Failsafe**: 0
   
   - **Description**: EDK II debug level bitmask (sum) showed onscreen. Unless Target enables console (onscreen) printing, onscreen debug output will not be visible. The following levels are supported (discover more in DebugLib.h):
```   
   • 0x00000002 (bit 1) — DEBUG_WARN in DEBUG, NOOPT, RELEASE.
   • 0x00000040 (bit 6) — DEBUG_INFO in DEBUG, NOOPT.
   • 0x00400000 (bit 22) — DEBUG_VERBOSE in custom builds.
   • 0x80000000 (bit 31) — DEBUG_ERROR in DEBUG, NOOPT, RELEASE.
```

* **5. Target**
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: A bitmask (sum) of enabled logging targets. By default all the logging output is hidden, so this option is required to be set when debugging is necessary.

The following logging targets are supported:

   third-party scripts may require ExposeSensitiveData set to 0x3 to provide boot-path variable with OpenCore EFI partition UUID.

**WARNING**: This feature is very dangerous as it passes unprotected data to your firmware variable services. Use it only when no hardware NVRAM implementation is provided by the firmware or it is incompatible.

* **4. LegacyOverwrite**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Permits overwriting firmware variables from nvram.plist.

**Note**: Only variables accessible from the operating system will be overwritten.

* **5. LegacySchema**
   
   - **Type**: plist dict
   
   - **Description**: Allows setting select NVRAM variables from a map (plist dict) of GUIDs to an array (plist array) of variable names in plist string format.
You can use * value to accept all variables for select GUID.

**WARNING**: Choose variables very carefully, as nvram.plist is not vaulted. For instance, do not put boot-args
   or csr-active-config, as this can bypass SIP.

* **6. WriteFlash**
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Enables writing to flash memory for all added variables.

**Note**: This value is recommended to be enabled on most firmwares, but is left configurable for firmwares that may have issues with NVRAM variable storage garbage collection or alike.

To read NVRAM variable value from macOS one could use nvram by concatenating variable GUID and name separated by : symbol. For example, nvram 7C436110-AB2A-4BBB-A880-FE41995C9F82:boot-args.
   A continuously updated variable list can be found in a corresponding document: NVRAM Variables.

* **9.3 Mandatory Variables**

**Warning**: These variables may be added by PlatformNVRAM or Generic subsections of PlatformInfo section. Using PlatformInfo is the recommend way of setting these variables.

The following variables are mandatory for macOS functioning:
    
   - 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:FirmwareFeatures
   32-bit FirmwareFeatures. Present on all Macs to avoid extra parsing of SMBIOS tables
   
   - 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:FirmwareFeaturesMask
   32-bit FirmwareFeaturesMask. Present on all Macs to avoid extra parsing of SMBIOS tables.
   
   - 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:MLB
   BoardSerialNumber. Present on newer Macs (2013+ at least) to avoid extra parsing of SMBIOS tables, especially
   in boot. efi. boot.efi. 
   
   - 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM
   Primary network adapter MAC address or replacement value. Present on newer Macs (2013+ at least) to
   avoid accessing special memory region, especially in boot. efi. boot.efi.

* **9.4 Recommended Variables**

The following variables are recommended for faster startup or other improvements:
   
**7C436110-AB2A-4BBB-A880-FE41995C9F82:csr-active-config**
   
   - 32-bit System Integrity Protection bitmask. Declared in XNU source code in csr.h.
   
**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ExtendedFirmwareFeatures**

   - Combined FirmwareFeatures and ExtendedFirmwareFeatures. Present on newer Macs to avoid extra parsing of SMBIOS tables
   

**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ExtendedFirmwareFeaturesMask**
   
   - Combined FirmwareFeaturesMask and ExtendedFirmwareFeaturesMask. Present on newer Macs to avoid extra parsing of SMBIOS tables.
   
**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:HW_BID**
   
   - Hardware BoardProduct (e.g. Mac-35C1E88140C3E6CF). Not present on real Macs, but used to avoid extra
   parsing of SMBIOS tables, especially in boot. efi. boot.efi. 
   
**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:HW_MLB**
   
   - Hardware BoardSerialNumber. Override for MLB. Present on newer Macs (2013+ at least).
   
**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:HW_ROM**
   
   - Hardware ROM. Override for ROM. Present on newer Macs (2013+ at least).
   
**7C436110-AB2A-4BBB-A880-FE41995C9F82:prev-lang:kbd**
   
   - ASCII string defining default keyboard layout. Format is lang-COUNTRY:keyboard, e.g. ru-RU:252 for Russian locale and ABC keyboard. Also accepts short forms: ru:252 or ru:0 (U.S. keyboard, compatible with 10.9). Full decoded keyboard list from AppleKeyboardLayouts-L.dat can be found here. Using non-latin keyboard on 10.14 will not enable ABC keyboard, unlike previous and subsequent macOS versions, and is thus not recommended in case you need 10.14.
  
**7C436110-AB2A-4BBB-A880-FE41995C9F82:security-mode**
   
   - ASCII string defining FireWire security mode. Legacy, can be found in IOFireWireFamily source code in IOFireWireController.cpp. It is recommended not to set this variable, which may speedup system startup. Setting to full is equivalent to not setting the variable and none disables FireWire security.

**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:UIScale**
 
   - One-byte data defining boot.efi boot.efi user interface scaling. Should be 01 for normal screens and 02
   for HiDPI screens.

**4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:DefaultBackgroundColor**
   
   - Four-byte RGBA data defining boot.efi boot.efi user interface background colour. Standard colours include `BF
   BF BF 00` (Light Gray) and `00 00 00 00` (Syrah Black). Other colours may be set at user’s preference.

* **9.5 Other Variables**

The following variables may be useful for certain configurations or troubleshooting:

**7C436110-AB2A-4BBB-A880-FE41995C9F82:boot-args**
   Kernel arguments, used to pass configuration to Apple kernel and drivers. There are many arguments, which may be found by looking for the use of PE_parse_boot_argn function in the kernel or driver code. Some of the known boot arguments include:
```
   – acpi_layer=0xFFFFFFFF
   – acpi_level=0xFFFF5F (implies ACPI_ALL_COMPONENTS)
   – batman=VALUE (AppleSmartBatteryManager debug mask)
   – batman-nosmc=1 (disable AppleSmartBatteryManager SMC interface)
   – cpus=VALUE (maximum number of CPUs used)
   – debug=VALUE (debug mask)
   – io=VALUE (IOKit debug mask)
   – keepsyms=1 (show panic log debug symbols)
   – kextlog=VALUE (kernel extension loading debug mask)
   – nv_disable=1 (disables NVIDIA GPU acceleration)
   – nvda_drv=1 (legacy way to enable NVIDIA web driver, removed in 10.12)
   – npci=0x2000 (legacy, disables kIOPCIConfiguratorPFM64)
   – lapic_dont_panic=1
   – slide=VALUE (manually set KASLR slide)
   – smcdebug=VALUE (AppleSMC debug mask)
   – -amd_no_dgpu_accel (alternative to WhateverGreen’s -radvesa for new GPUs) – -nehalem_error_disable
   – -no_compat_check (disable model checking)
   – -s (single mode)
   – -v (verbose mode)
   – -x (safe mode)
```   
There are multiple external places summarising macOS argument lists: example 1, example 2.

**7C436110-AB2A-4BBB-A880-FE41995C9F82:bootercfg**
   
   - ∗ 2 — AppleLoggingStdErrSet/AppleLoggingStdErrPrint (StdErr or serial?)
   
   - ∗ 4 — AppleLoggingFileSet/AppleLoggingFilePrint (BOOTER.LOG/BOOTER.OLD file on EFI partition) 

`debug=VALUE` — deprecated starting from 10.15.
   
   - ∗ 1 — enables print something to BOOTER.LOG (stripped code implies there may be a crash) 
   
   - ∗ 2 — enables perf logging to /efi/debug-log in the device three
   
   - ∗ 4 — enables timestamp printing for styled printf calls
   
   
`level=VALUE` — deprecated starting from 10.15. Verbosity level of DEBUG output. Everything but 0x80000000 is stripped from the binary, and this is the default value.
   
**Note**: To quickly see verbose output from boot.efi on versions modern macOS versions enable AppleDebug 
   option. This will save the log to general OpenCore log. For versions before 10.15.4 set bootercfg to log=1. This  will print verbose output onscreen.
   
**7C436110-AB2A-4BBB-A880-FE41995C9F82:bootercfg-once**
   
   - Booter arguments override removed after first launch. Otherwise equivalent to bootercfg.

**7C436110-AB2A-4BBB-A880-FE41995C9F82:efiboot-perf-record** 
  
   - Enable performance log saving in boot.efi. Performance log is saved to physical memory and is pointed 
   by efiboot-perf-record-data and efiboot-perf-record-size variables. Starting from 10.15.4 it can also be  saved to OpenCore log by AppleDebug option. 
   
**7C436110-AB2A-4BBB-A880-FE41995C9F82:fmm-computer-name** 
   
   - Current saved host name. ASCII string.
   
**7C436110-AB2A-4BBB-A880-FE41995C9F82:nvda_drv**
   
   - NVIDIA Web Driver control variable. Takes ASCII digit 1 or 0 to enable or disable installed driver.

**7C436110-AB2A-4BBB-A880-FE41995C9F82:StartupMute**
   
   - Mute startup chime sound in firmware audio support. 8-bit integer. The value of 0x00 means unmuted. Missing variable or any other value means muted. This variable only affects Gibraltar machines (T2).

**7C436110-AB2A-4BBB-A880-FE41995C9F82:SystemAudioVolume**
   
   - System audio volume level for firmware audio support. 8-bit integer. The bit of 0x80 means muted. Lower bits are used to encode volume range specific to installed audio codec. The value is capped by MaximumBootBeepVolume AppleHDA layout value to avoid too loud audio playback in the firmware.

* **10 PlatformInfo**

Platform information is comprised of several identification fields generated or filled manually to be compatible with macOS services. The base part of the configuration may be obtained from MacInfoPkg package, which itself generates a set of interfaces based on a database in YAML format. These fields are written to three select destinations:
   
   - SMBIOS 
   
   - Data Hub 
   
   - NVRAM

Most of the fields specify the overrides in SMBIOS, and their field names conform to EDK2 SmBios.h header file. However, several important fields reside in Data Hub and NVRAM. Some of the values can be found in more than one field and/or destination, so there are two ways to control their update process: manual, where one specifies all the values (the default), and semi-automatic, where (Automatic) only select values are specified, and later used for system configuration.

To inspect SMBIOS contents dmidecode utility can be used. Version with macOS specific enhancements can be downloaded from Acidanthera/dmidecode.

* **10.1 Properties**
   
- 1. Automatic
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Generate PlatformInfo based on Generic section instead of using values from DataHub, NVRAM, and SMBIOS sections.
   
   Enabling this option is useful when Generic section is flexible enough. When enabled SMBIOS, DataHub, and PlatformNVRAM data is unused.

- 2. UpdateDataHub
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Update Data Hub fields. These fields are read from Generic or DataHub sections depending on Automatic value.
- 3. UpdateNVRAM
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Update NVRAM fields related to platform information.
   
   These fields are read from Generic or PlatformNVRAM sections depending on Automatic value. All the other fields are to be specified with NVRAM section.
   If UpdateNVRAM is set to false the aforementioned variables can be updated with NVRAM section. If UpdateNVRAM is set to true the behaviour is undefined when any of the fields are present in NVRAM section.

- 4. UpdateSMBIOS
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Update SMBIOS fields. These fields are read from Generic or SMBIOS sections depending on Automatic value.

- 5. UpdateSMBIOSMode
   
   - **Type**: plist string
   
   - **Failsafe**: Create
   
   - **Description**: Update SMBIOS fields approach:

   - TryOverwrite — Overwrite if new size is <= than the page-aligned original and there are no issues with legacy region unlock. Create otherwise. Has issues with some firmwares.
   
   - Create — Replace the tables with newly allocated EfiReservedMemoryType at AllocateMaxAddress without any fallbacks.

- 7. Quirks
   
   - **Type**: plist dict
   
   - **Failsafe**: None
   
   - **Description**: Apply individual firmware quirks described in Quirks Properties section below.

* **11.3 Audio Properties**

- 1. AudioCodec
   
   - **Type**: plist integer
   
   - **Failsafe**: empty stringX0
   
   - **Description**: Codec address on the specified audio controller for audio support.

Normally this contains first audio codec address on the builtin analog audio controller (HDEF). Audio codec addresses, e.g. 2, can be found in the debug log (marked in bold):

```
   OCAU: 1/3 PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x1)/VenMsg(<redacted>,00000000) (4 outputs) 
   OCAU: 2/3 PciRoot(0x0)/Pci(0x3,0x0)/VenMsg(<redacted>,00000000) (1 outputs)
   OCAU: 3/3 PciRoot(0x0)/Pci(0x1B,0x0)/VenMsg(<redacted>,02000000) (7 outputs)
```   
   As an alternative this value can be obtained from IOHDACodecDevice class in I/O Registry containing it in IOHDACodecAddress field.

- 2. AudioDevice
   
   - **Type**: plist string
   
   - **Failsafe**: 0empty string 
   
   - **Description**: Device path of the specified audio controller for audio support.
   Normally this contains builtin analog audio controller (HDEF) device path, e.g. PciRoot(0x0)/Pci(0x1b,0x0).
   The list of recognised audio controllers can be found in the debug log (marked in bold):

```   
   OCAU: 1/3 PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x1)/VenMsg(<redacted>,00000000) (4 outputs) 
   OCAU: 2/3 PciRoot(0x0)/Pci(0x3,0x0)/VenMsg(<redacted>,00000000) (1 outputs)
   OCAU: 3/3 PciRoot(0x0)/Pci(0x1B,0x0)/VenMsg(<redacted>,02000000) (7 outputs)
```   
As an alternative gfxutil -f HDEF command can be used in macOS. Specifying empty device path will result in the first available audio controller to be used.

- 3. AudioOut
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Index of the output port of the specified codec starting from 0.
   
   Normally this contains the index of the green out of the builtin analog audio controller (HDEF). The number of output nodes (N) in the debug log (marked in bold):
 ``` 
   OCAU: 1/3 PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x1)/VenMsg(<redacted>,00000000) (4 outputs) 
   OCAU: 2/3 PciRoot(0x0)/Pci(0x3,0x0)/VenMsg(<redacted>,00000000) (1 outputs)
   OCAU: 3/3 PciRoot(0x0)/Pci(0x1B,0x0)/VenMsg(<redacted>,02000000) (7 outputs)
```   
   The quickest way to find the right port is to bruteforce the values from 0 to N - 1.

- 4. AudioSupport
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Activate audio support by connecting to a backend driver.
   
   Enabling this setting routes audio playback from builtin protocols to a dedicated audio port (AudioOut) of the specified codec (AudioCodec) located on the audio controller (AudioDevice).

- 5. MinimumVolume
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Minimal heard volume level from 0 to 100.

   Screen reader will use this volume level, when the calculated volume level is less than MinimumVolume. Boot chime sound will not play if the calculated volume level is less than MinimumVolume.

- 6. PlayChime
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Play chime sound at startup.
   
   Enabling this setting plays boot chime through builtin audio support. Volume level is determined by MinimumVolume and VolumeAmplifier settings and SystemAudioVolume NVRAM variable.
**Note**: this setting is separate from StartupMute NVRAM variable to avoid conflicts when the firmware is able to play boot chime.
   
- 7. VolumeAmplifier
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Multiplication coefficient for system volume to raw volume linear translation from 0 to 1000.
   
   Volume level range read from SystemAudioVolume varies depending on the codec. To transform read value in [0, 127] range into raw volume range [0, 100] the read value is scaled to VolumeAmplifier percents:

   ```
                        SystemAudioV olume ∗ VolumeAmplifier
   RawVolume = MIN (------------------------------------,100)
                       100
   ```
   
**Note**: the transformation used in macOS is not linear, but it is very close and this nuance is thus ignored.

* **11.4 Input Properties**
   
- 1. KeyFiltering 
   
   - **Type**: plist boolean 
   
   - **Failsafe**: false 
   
   - **Description**: Enable keyboard input sanity checking. 
   Apparently some boards like GA Z77P-D3 may return uninitialised data in EFI_INPUT_KEY with all input protocols. This option discards keys that are neither ASCII, nor are defined in the UEFI specification (see tables 107 and 108 in version 2.8).
   

- 2. KeyForgetThreshold
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Remove key unless it was submitted during this timeout in milliseconds.
   
   AppleKeyMapAggregator protocol is supposed to contain a fixed length buffer of currently pressed keys. However, the majority of the drivers only report key presses as interrupts and pressing and holding the key on the keyboard results in subsequent submissions of this key with some defined time interval. As a result we use a timeout to remove once pressed keys from the buffer once the timeout expires and no new submission of this key happened.
   
   This option allows to set this timeout based on your platform. The recommended value that works on the majority of the platforms is 5 milliseconds. For reference, holding one key on VMware will repeat it roughly every 2 milliseconds and the same value for APTIO V is 3-4 milliseconds. Thus it is possible to set a slightly lower value on faster platforms and slightly higher value on slower platforms for more responsive input.

- 3. KeyMergeThreshold
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Assume simultaneous combination for keys submitted within this timeout in milliseconds.
   
   Similarly to KeyForgetThreshold, this option works around the sequential nature of key submission. To be able to recognise simultaneously pressed keys in the situation when all keys arrive sequentially, we are required to set a timeout within which we assume the keys were pressed together.
   
   Holding multiple keys results in reports every 2 and 1 milliseconds for VMware and APTIO V respectively. Pressing keys one after the other results in delays of at least 6 and 10 milliseconds for the same platforms. The recommended value for this option is 2 milliseconds, but it may be decreased for faster platforms and increased for slower.

- 4. KeySupport
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Enable internal keyboard input translation to AppleKeyMapAggregator protocol.
   
   This option activates the internal keyboard interceptor driver, based on AppleGenericInput aka (AptioInputFix), to fill AppleKeyMapAggregator database for input functioning. In case a separate driver is used, such as AppleUsbKbDxe, this option should never be enabled.

- 5. KeySupportMode
   
   - **Type**: plist string
   
   - **Failsafe**: empty string
   
   - **Description**: Set internal keyboard input translation to AppleKeyMapAggregator protocol mode.
      - Auto — Performs automatic choice as available with the following preference: AMI, V2, V1. 
      - V1 — Uses UEFI standard legacy input protocol `EFI_SIMPLE_TEXT_INPUT_PROTOCOL`.
      - V2 — Uses UEFI standard modern input protocol `EFI_SIMPLE_TEXT_INPUT_EX_PROTOCOL`. 
      - AMI — Uses APTIO input protocol `AMI_EFIKEYCODE_PROTOCOL`.

**Note**: Currently V1, V2, and AMI unlike Auto only do filtering of the particular specified protocol. This may change in the future versions.

- 6. KeySwap
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Swap Command and Option keys during submission.
   This option may be useful for keyboard layouts with Option key situated to the right of Command key.

- 7. PointerSupport
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Enable internal pointer driver.
  
   This option implements standard UEFI pointer protocol (EFI_SIMPLE_POINTER_PROTOCOL) through select OEM protocols. The option may be useful on Z87 ASUS boards, where EFI_SIMPLE_POINTER_PROTOCOL is broken.

- 8. PointerSupportMode
   
   - **Type**: plist string
   
   - **Failsafe**: empty string
   
   - **Description**: Set OEM protocol used for internal pointer driver.
   
   Currently the only supported variant is ASUS, using specialised protocol available on select Z87 and Z97 ASUS boards. More details can be found in LongSoft/UefiTool#116.

- 9. TimerResolution
   
   - **Type**: plist integer
   
   - **Failsafe**: 0
   
   - **Description**: Set architecture timer resolution.
   
   This option allows to update firmware architecture timer period with the specified value in 100 nanosecond units. Setting a lower value generally improves performance and responsiveness of the interface and input handling.
   
   The recommended value is 50000 (5 milliseconds) or slightly higher. Select ASUS Z87 boards use 60000 for the interface. Apple boards use 100000. You may leave it as 0 in case there are issues.

* **11.5 Output Properties**
   
- 1. TextRenderer
   
   - **Type**: plist string
   
   - **Failsafe**: BuiltinGraphics
   
   - **Description**: Chooses renderer for text going through standard console output.
   50

   Currently two renderers are supported: Builtin and System. System renderer uses firmware services for text rendering. Builtin bypassing firmware services and performs text rendering on its own. Different renderers support a different set of options. It is recommended to use Builtin renderer, as it supports HiDPI mode and uses full screen resolution.
   
   UEFI firmwares generally support ConsoleControl with two rendering modes: Graphics and Text. Some firmwares do not support ConsoleControl and rendering modes. OpenCore and macOS expect text to only be shown in Graphics mode and graphics to be drawn in any mode. Since this is not required by UEFI specification, exact behaviour varies.
   
   Valid values are combinations of text renderer and rendering mode:
   
   - BuiltinGraphics — Switch to Graphics mode and use Builtin renderer with custom ConsoleControl. 
   
   - SystemGraphics — Switch to Graphics mode and use System renderer with custom ConsoleControl. 
   
   - SystemText — Switch to Text mode and use System renderer with custom ConsoleControl.
   
   - SystemGeneric — Use System renderer with system ConsoleControl assuming it behaves correctly.
  
   The use of BuiltinGraphics is generally straightforward. For most platforms it is necessary to enable ProvideConsoleGop, set Resolution to Max.
   
   The use of System protocols is more complicated. In general the preferred setting is SystemGraphics or SystemText. Enabling ProvideConsoleGop, setting Resolution to Max, enabling ReplaceTabWithSpace is useful on almost all platforms. SanitiseClearScreen, IgnoreTextInGraphics, and ClearScreenOnModeSwitch are more specific, and their use depends on the firmware.
   
**Note**: Some Macs, namely MacPro5,1, may have broken console output with newer GPUs, and thus only BuiltinGraphics may work for them.

- 2. ConsoleMode
   
   - **Type**: plist string
   
   - **Failsafe**: Empty string
   
   - **Description**: Sets console output mode as specified with the WxH (e.g. 80x24) formatted string.
   
   Set to empty string not to change console mode. Set to Max to try to use largest available console mode. Currently Builtin text renderer supports only one console mode, so this option is ignored.
**Note**: This field is best to be left empty on most firmwares.

- 3. Resolution
   
   - **Type**: plist string
   
   - **Failsafe**: Empty string
   
   - **Description**: Sets console output screen resolution.
   
   • Set to WxH@Bpp (e.g. 1920x1080@32) or WxH (e.g. 1920x1080) formatted string to request custom resolution from GOP if available.
   
   • Set to empty string not to change screen resolution.
   
   • Set to Max to try to use largest available screen resolution.

On HiDPI screens `APPLE_VENDOR_VARIABLE_GUID` UIScale NVRAM variable may need to be set to 02 to enable HiDPI scaling in Builtin text renderer, FileVault 2 UEFI password interface, and boot screen logo. Refer to Recommended Variables section for more details.
   Note: This will fail when console handle has no GOP protocol. When the firmware does not provide it, it can be added with ProvideConsoleGop set to true.

- 4. ClearScreenOnModeSwitch
   
   - **Type**: plist boolean
   
   * **Failsafe**: false
   
   - **Description**: Some firmwares clear only part of screen when switching from graphics to text mode, leaving a fragment of previously drawn image visible. This option fills the entire graphics screen with black color before switching to text mode.

**Note**: This option only applies to System renderer.

- 5. DirectGopCacheMode 
   
   - **Type**: plist string 
   
   - **Failsafe**: Empty string 
   
   - **Description**: Cache mode for builtin graphics output protocol framebuffer. 
   
   Tuning cache mode may provide better rendering performance on some firmwares. Providing empty string WriteThrough. 
   
**Note**: This option is not supported on most hardware (see acidanthera/bugtracker#755 for more details). 

- 6. DirectGopRendering
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Use builtin graphics output protocol renderer for console.
   
   On some firmwares this may provide better performance or even fix rendering issues, like on MacPro5,1. However, it is recommended not to use this option unless there is an obvious benefit as it may even result in slower scrolling.

- 7. IgnoreTextInGraphics
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Select firmwares output text onscreen in both graphics and text mode. This is normally unexpected, because random text may appear over graphical images and cause UI corruption. Setting this option to true will discard all text output when console control is in mode different from Text.

**Note**: This option only applies to System renderer.

- 8. ReplaceTabWithSpace
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Some firmwares do not print tab characters or even everything that follows them, causing difficulties or inability to use the UEFI Shell builtin text editor to edit property lists and other documents. This option makes the console output spaces instead of tabs.

**Note**: This option only applies to System renderer.

- 9. ProvideConsoleGop
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Ensure GOP (Graphics Output Protocol) on console handle.
   macOS bootloader requires GOP to be present on console handle, yet the exact location of GOP is not covered by the UEFI specification. This option will ensure GOP is installed on console handle if it is present.
   Note: This option will also replace broken GOP protocol on console handle, which may be the case on MacPro5,1 with newer GPUs.

- 10. ReconnectOnResChange
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reconnect console controllers after changing screen resolution.
   
   On some firmwares when screen resolution is changed via GOP, it is required to reconnect the controllers, which produce the console protocols (simple text out). Otherwise they will not produce text based on the new resolution.

**Note**: On several boards this logic may result in black screen when launching OpenCore from Shell and thus it is optional. In versions prior to 0.5.2 this option was mandatory and not configurable. Please do not use this unless required.

- 11. SanitiseClearScreen
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Some firmwares reset screen resolution to a failsafe value (like 1024x768) on the attempts to clear screen contents when large display (e.g. 2K or 4K) is used. This option attempts to apply a workaround.

**Note**: This option only applies to System renderer. On all known affected systems ConsoleMode had to be set to empty string for this to work.

* **11.6 Protocols Properties**
   
  - 1. AppleAudio
      
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple audio protocols with builtin versions.
   
   Apple audio protocols allow macOS bootloader and OpenCore to play sounds and signals for screen reading or audible error reporting. Supported protocols are beep generation and VoiceOver. VoiceOver protocol is specific to Gibraltar machines (T2) and is not supported before macOS High Sierra (10.13). Instead older macOS versions use AppleHDA protocol, which is currently not implemented.
   
   Only one set of audio protocols can be available at a time, so in order to get audio playback in OpenCore user interface on Mac system implementing some of these protocols this setting should be enabled.
  
**Note**: Backend audio driver needs to be configured in UEFI Audio section for these protocols to be able to stream audio.
   
  - 2. AppleBootPolicy
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple Boot Policy protocol with a builtin version. This may be used to ensure APFS compatibility on VMs or legacy Macs.

**Note**: Some Macs, namely MacPro5,1, do have APFS compatibility, but their Apple Boot Policy protocol contains recovery detection issues, thus using this option is advised on them as wellX.X

  - 3. AppleDebugLog 
   
   - **Type**: plist boolean 
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple Debug Log protocol with a builtin version.
   
  - 4. AppleEvent
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple Event protocol with a builtin version. This may be used to ensure File Vault 2 compatibility on VMs or legacy Macs.
  - 5. AppleImageConversion
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple Image Conversion protocol with a builtin version.
   
  - 6. AppleKeyMap
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple Key Map protocols with builtin versions.
   
  - 7. AppleSmcIo
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple SMC I/O protocol with a builtin version.
   
   This protocol replaces legacy VirtualSmc UEFI driver, and is compatible with any SMC kernel extension. However, in case FakeSMC kernel extension is used, manual NVRAM key variable addition may be needed.
   
  - 8. AppleUserInterfaceTheme
   
   - **Type**: plist boolean
   
   - **Failsafe**: false
   
   - **Description**: Reinstalls Apple User Interface Theme protocol with a builtin version.
