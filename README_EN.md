# AMS 1.12.0 - Quest/Kiosk Invert Fix

This fix was developed after a factory-original Nintendo Switch was updated through Nintendo's official update method and stopped booting entirely afterwards. Since the console was already out of factory warranty, the only viable recovery path was installing a modchip. After the modchip was installed and a full NAND backup was restored, the console still failed to boot — neither in Stock (OFW) nor in emuMMC (Atmosphère). The root cause was traced to the console being a demo/kiosk unit whose missing RDIM (Retail Interactive Display Menu) data caused qlaunch to fail with error `2162-0004` right after the boot logos.

This repository provides a patched `atmosphere/package3` that resolves this issue safely and reversibly.

## Root Cause

Demo/kiosk (store display) Switch units have an extra fuse burnt (the "Quest" fuse). Exosphère reads this fuse and sets the Quest flag. When this flag is active, `qlaunch` (the HOME Menu) automatically attempts to launch title `0100069000078000` (the Retail Interactive Display Menu / RDIM).

If that demo application is missing (commonly after a factory reset/format, or as a side effect of certain updates on kiosk units), the launch fails and HOS shows `2162-0004` ("Could not start the software") immediately after the Nintendo logos, with `FocusedAppletHistory` empty.

This happens on both sysNAND and emuMMC, because the flag is determined by Exosphère at boot (based on the fuse interpretation).

## Solution

This repository provides a patched `atmosphere/package3` for **Atmosphère 1.12.0** with the Quest/Kiosk flag inverted (`quest-invert`). This makes kiosk/demo units behave like retail units, so `qlaunch` no longer tries to auto-launch the RDIM and boots directly into the HOME Menu.

The source-level change (from ZachyCatGames' fork):

```cpp
// libraries/libexosphere/source/fuse/fuse_api.cpp
RetailInteractiveDisplayState GetRetailInteractiveDisplayState() {
    return static_cast<RetailInteractiveDisplayState>(!util::BitPack32{GetCommonOdmWord(4)}.Get<OdmWord4::RetailInteractiveDisplayState>());
}
```

## What's Included

- `atmosphere/package3` – Patched Exosphère (quest-invert) for AMS 1.12.0
- `package3.sha256` – SHA256 checksum for integrity verification
- `LICENSE` – GPLv2 (Atmosphère license)
- `README.md` – Spanish version
- `README_EN.md` – English version

## How to Apply

1. **Back up** your original `atmosphere/package3` from your SD card (this is fully reversible).
2. Copy `atmosphere/package3` from this repo to `atmosphere/package3` on your SD card (replace it).
3. Ensure `atmosphere/contents/420000000000000B` exists (reactivate sys-patch if it was renamed to `420000000000000B_DISABLED`).
4. Boot via Hekate → `[Atmosphere (EMUMMC)]` (or inject `fusee.bin`). You should boot into the initial setup ("Welcome" / "Set up as new console") or the HOME Menu.
5. To **revert**, restore your original `package3` backup.

## Verified On

- Console: Nintendo Switch V2 (Mariko) with HWFLY/Picofly modchip
- Firmware: 23.0.1
- CFW: Atmosphère 1.12.0
- Bootloader: Hekate 6.5.4
- Result: Boots to "Set up as new console" (HOME Menu launches correctly)

## Credits

- [ZachyCatGames/Atmosphere](https://github.com/ZachyCatGames/Atmosphere) – [1.12.0-quest-invert](https://github.com/ZachyCatGames/Atmosphere/releases/tag/1.12.0-quest-invert) release
- [Atmosphere-NX/Atmosphere](https://github.com/Atmosphere-NX/Atmosphere) – Base Atmosphère

## License

This repository follows the GPLv2 license of Atmosphère. See [LICENSE](LICENSE).

## Verification

```text
SHA256: b26c297c77755041d87871d7614b13bd16ec8c946b524cc64107af5efcce3fde *atmosphere/package3
```
