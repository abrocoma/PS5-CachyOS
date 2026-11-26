# CachyOS on PS5

**Run a full Linux desktop and Steam on a jailbroken PS5.** Flash a ready-made image to a USB SSD, boot it, play.

![Firmware 13.60](https://img.shields.io/badge/PS5_firmware-13.60-0070d1)
![Kernel 7.3.5](https://img.shields.io/badge/kernel-7.3.5-333)
![KDE Plasma 6.8](https://img.shields.io/badge/KDE_Plasma-6.8-1d99f3)
![Wayland](https://img.shields.io/badge/Wayland-yes-2ea44f)

![KDE "About this System" on the PS5: kernel 7.3.5-pc8fe3, KDE Plasma 6.8.3, Qt 6.12.1 on Wayland, 16 × AMD Custom Zen 2 (Oberon), 14.1 GiB RAM](images/about-this-system-pc8fe3.png)

**Tested on** a PS5 launch model Digital Edition (CFI-1016B), firmware 13.60, [ps5-linux-loader](https://github.com/ps5-linux/ps5-linux-loader) (build `badfe48`, attached to the [latest release](https://github.com/abrocoma/PS5-CachyOS/releases/latest)), a 1 TB USB-C SSD, a 1440p 144 Hz monitor and a 4K 60 Hz TV. The image is built on the official [ps5-linux CachyOS image](https://github.com/ps5-linux/ps5-linux-image), see [Credits](#credits).

> [!IMPORTANT]
> **Rebooting Linux takes you back to PS5 OS.** To get into Linux again, run the exploit and the loader again.

## Contents

1. [Flash the image](#1-flash-the-image)
2. [Boot into Linux](#2-boot-into-linux)
3. [First boot and getting online](#3-first-boot-and-getting-online)
4. [Gaming Mode and the desktop](#4-gaming-mode-and-the-desktop)
5. [Updating](#5-updating)
6. [Display, FreeSync and HDR](#6-display-freesync-and-hdr)
7. [Extras: boost, fan and video](#7-extras-boost-fan-and-video)
8. [Troubleshooting](#8-troubleshooting)

[Advanced](docs/advanced.md): the official image, building your own kernel, what's in this repo.

---

## 1. Flash the image

Linux runs entirely from an external SSD. Nothing is installed on the PS5's internal storage.

**You need:** a jailbroken PS5 with the Linux loader ([ps5-linux-loader](https://github.com/ps5-linux/ps5-linux-loader): its v2.5 release only supports firmware up to 7.61, so for 10.xx-13.60 use `ps5-linux-loader-badfe48.elf` from the [latest release](https://github.com/abrocoma/PS5-CachyOS/releases/latest)) and a USB 3 SSD.

> [!TIP]
> **No PC? Flash it from the PS5 itself** with [Linux Flasher](https://github.com/abrocoma/PS5-Linux-Flasher/releases/latest): a payload that downloads this image, writes it to the SSD and checks it. Then skip to [step 2](#2-boot-into-linux). With a PC, follow the steps below.

> [!CAUTION]
> Flashing **erases the whole SSD**.

**Download** `ps5-cachyos-7.3.5-pc8fe3.img.xz` (2 GB) from the [latest release](https://github.com/abrocoma/PS5-CachyOS/releases/latest). It's the official image with every fix from this guide installed: no black screens at 1080p/1440p, up to 144 Hz, FreeSync, desktop HDR, SteamOS-style Gaming Mode, Wi-Fi, DualSense reconnect, quiet fan curve and a one-click updater.

```sh
wget https://github.com/abrocoma/PS5-CachyOS/releases/download/image-2026-11-26/ps5-cachyos-7.3.5-pc8fe3.img.xz
wget https://github.com/abrocoma/PS5-CachyOS/releases/download/image-2026-11-26/ps5-cachyos-7.3.5-pc8fe3.img.xz.sha256
sha256sum -c ps5-cachyos-7.3.5-pc8fe3.img.xz.sha256     # must say OK
```

**Flash it:**
- **Windows / macOS:** extract the `.img.xz` to get the `.img` (Windows: [7-Zip](https://www.7-zip.org/); macOS: double-click it), then flash it with [balenaEtcher](https://etcher.balena.io/).
- **Linux:** find the SSD with `lsblk` and write to the **whole disk** (`/dev/sdX`, not `/dev/sdX1`):
  ```sh
  xzcat ps5-cachyos-7.3.5-pc8fe3.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
  ```

## 2. Boot into Linux

**Plug the SSD into** the **front USB-C port** or a **rear USB 3.0 port**. Not the front USB-A port: it's only USB 2.0.

> [!WARNING]
> **Don't run any other payloads before the Linux loader** (homebrew, FTP, cheats). The loader crashes if anything else is loaded. Start from a fresh boot every time.

1. **Turn on** (or restart) the PS5.
2. **Run your exploit** (Y2JB, Relapse, etc.).
3. **Load only the ps5-linux-loader payload.**
4. The PS5 **goes into rest mode** by itself. Wait until the power light **stops pulsing and stays solid orange**.
5. **Press the power button.** It boots into Linux.

Crashed or frozen? Restart the PS5 and start again from step 1. Stuck at "waiting for device"? See [Drive not found](#drive-not-found-at-boot).

## 3. First boot and getting online

On first boot the system partition **grows to fill the SSD**, a **setup screen** asks for timezone, hostname and password, then **Gaming Mode** starts.

The default login is **`steam`** / **`steam`**, hostname `ps5`.

> [!WARNING]
> **Change the password** (*Customize* in the setup screen, or `passwd` later). SSH with password login is on.

**Ethernet** works straight away. **Wi-Fi:** Steam → **Settings → Internet** in Gaming Mode, or the network icon on the desktop. It reconnects on its own at every boot.

> [!TIP]
> **No picture or need a terminal?** Plug in a USB keyboard, press <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F2</kbd> and log in as `steam`. <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F1</kbd> goes back. Wi-Fi from there: `nmtui`.

## 4. Gaming Mode and the desktop

The PS5 boots into **Gaming Mode**: Steam's console interface, as on SteamOS. It remembers the mode you used last: after **Switch to Desktop** it boots to the desktop, until you go back with **Return to Gaming Mode**.

![Gaming Mode on the PS5: the Steam library in the SteamOS-style interface](images/gaming-mode.jpg)

- **To the desktop:** Steam's power menu → **Switch to Desktop**. **Back:** the **Return to Gaming Mode** icon on the desktop.
- **Quick menu** (performance overlay, frame limit, FreeSync): <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Tab</kbd>, also in a game. See [FreeSync in Gaming Mode](#freesync-in-gaming-mode).
- **Display and network settings** are in Steam's settings, as on a Steam Deck.
- **Steam's Restart** is a real reboot, so it takes you back to PS5 OS.
- **Pick the mode for the next boot by hand:** `echo plasma > ~/.config/ps5-next-session` (`gamescope` for Gaming Mode).

![Steam's power menu with Switch to Desktop](images/switch-to-desktop.jpg)

## 5. Updating

Click **Update PS5 CachyOS** on the desktop. Steam's own "up to date" message doesn't cover the system.

On an older install without that icon, run this once on the desktop as your normal user (**without** `sudo`):

```
curl -fsSL https://raw.githubusercontent.com/abrocoma/PS5-CachyOS/main/scripts/update.sh | sh
```

![The PS5 CachyOS Updater: console info at the top, one line per step, all done in 1 min 21 s](images/updater-pc8fe.png)

It updates, in order: the **PS5 kernel**, **fixes**, the **Gaming Mode and Steam extras**, **FreeSync and 1080p 144 Hz** (asks), the **quiet fan curve** (asks, default no), the **update icon** with a daily check for new kernels, and finally **all system packages and Flatpak apps** (asks). Restart afterwards if the kernel changed.

**Undo:** the same line with `| sh -s -- --rollback` at the end. Package updates can't be undone.

<details>
<summary>Single steps and what they change</summary>

- **Kernel only:** `curl -fsSL https://raw.githubusercontent.com/abrocoma/PS5-CachyOS/main/scripts/update-kernel.sh | sudo sh`. Replaces `/boot/efi/bzImage` + `initrd.img` (old ones kept as `.bak`) and the modules, incl. the Wi-Fi driver. FreeSync and 1080p 144 Hz also need `hdmi.vrr=1 hdmi.extended_tmds=1` in `cmdline.txt`. Booting from USB with root on an M.2 drive works too.
- **Gaming Mode and Steam extras only:** `curl -fsSL https://raw.githubusercontent.com/abrocoma/PS5-CachyOS/main/scripts/update-extras.sh | sudo sh`. Stops Steam from "updating" on every switch between Gaming Mode and the desktop, adds the performance overlay, makes shader pre-caching use all 16 threads, lets you pick the refresh rate in Steam's own display settings, turns on HDR in Gaming Mode (PS5 Fat), and adds the **12-bit Colour** icon. Changed files are kept as `.bak-extras`; a launcher you edited yourself is left alone (`--force` replaces it).
- **Kernel headers** (to build extra drivers with DKMS, e.g. `xone` for the Xbox Wireless Adapter): `curl -fsSL https://raw.githubusercontent.com/abrocoma/PS5-CachyOS/main/scripts/update-kernel.sh | sudo sh -s -- --headers`. Puts the build files at `/usr/lib/modules/<kernel>/build` and builds your DKMS drivers. Later kernel updates install new headers by themselves.
- **Undo** each one: the same line with `| sudo sh -s -- --rollback` at the end.
</details>

## 6. Display, FreeSync and HDR

| Mode | Status |
|---|---|
| 1080p @ 60 / 120 / 144 Hz | ✅ (144 Hz needs a monitor that offers it) |
| 1440p @ 60 / 120 / 144 Hz | ✅ (PS5 Slim: 120 / 144 Hz with kernel 7.3.5-pc8fe2 or newer) |
| 2160p @ 60 Hz | ✅ |
| 75 Hz and other monitor modes (e.g. 1080p @ 75 Hz) | ✅ with kernel 7.3.5-pc8fe or newer |
| 2160p @ 120 Hz | Needs an HDMI 2.1 TV (untested) |

**FreeSync / VRR** works at **60 and 120 Hz**, not at 144 Hz (the PS5's HDMI chip doesn't do variable refresh there). First turn on **Adaptive-Sync / FreeSync** in the monitor's own menu. For fast games that already run above 144 fps (CS2, for example), fixed 144 Hz usually feels better.

- **Desktop:** pick 60 or 120 Hz, then **System Settings → Display → Adaptive sync:** *Automatic*.

### Refresh rate in Gaming Mode

Steam **Settings → Display** lists your screen's resolutions and refresh rates (for example 1440p at 60 / 120 / 144 Hz). The change is live, and Steam remembers it per screen. Games read the refresh rate when they start, so a game that's already running shows the new value in its own settings after a restart.

### FreeSync in Gaming Mode

1. In Gaming Mode: Steam **Settings → Display** → pick **120 Hz** (or 60 Hz).
2. Quick menu (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Tab</kbd>) → **Performance** (lightning icon) → **Enable VRR**.
3. To check it: set a **Frame Limit** below 120 (turn off **Disable Frame Limit** first). The monitor's info menu then shows **AMD FreeSync** and a refresh rate that follows the game.

The same menu has the **Performance Overlay Level** (fps, CPU/GPU load, temperatures). The frame limit wins over a higher in-game limit such as `fps_max` in CS2.

<img src="images/gaming-mode-performance-vrr.jpg" alt="Steam quick menu in Gaming Mode, Performance tab, with Enable VRR on and Display state: VRR Enabled marked" width="300">

### HDR and colour

**Desktop HDR:** **System Settings → Display → Enable HDR**. HDR at 4K needs 12-bit colour: click the **12-bit Colour** icon. 

**HDR in Gaming Mode** (PS5 Fat and Pro / FLAVA3): turn on HDR in the monitor's menu if it has a setting for it, then quick menu → **Performance** → **Enable HDR**. Steam's SDR brightness slider works too. The PS5 Slim keeps sRGB output without HDR, because HDR signalling gives a black screen on its HDMI chip.

**HDMI chip:** check yours with `sudo dmesg | grep "HDMI IC"`. **FLAVA3** (launch model) does HDR and 12-bit colour. **VERDE** (Slim) is limited to 8-bit colour without HDR.

![KDE Display settings on the PS5: 2560x1440 at 144 Hz, Adaptive sync (FreeSync) available, Enable HDR ticked, 12 bits per colour](images/display-vrr-pb33e.jpg)

## 7. Extras: boost, fan and video

**Update PS5 CachyOS** (the desktop icon, [section 5](#5-updating)) offers the extras at the end that aren't installed yet, one key each: CPU **boost**, **Cloudflare DNS**, the **Xbox Wireless Adapter** driver and the **PS5 Hardware Control** app.

On the official ps5-linux image, use `install.sh` instead ([docs/advanced.md](docs/advanced.md#using-the-official-image-instead)).

**Fan:** the image runs a quieter fan curve and shows "SoC Temp" in the panel. Back to the kernel's default curve: `curl -fsSL https://raw.githubusercontent.com/abrocoma/PS5-CachyOS/main/scripts/quiet-fan.sh | sh -s -- --remove`. Don't run ps5-linux-tools' `ps5fan` service with it: it pins the fan loud.

<details>
<summary>Changing the fan curve</summary>

8 points in the `ps5_fan` hwmon folder: `pwm1_auto_point1..8_temp` (millidegrees) and `pwm1_auto_point1..8_pwm` (0-255, the last must be 255). The firmware still forces full speed at 90 °C. The image's curve holds CS2 at about 67 °C:

| °C | 55 | 60 | 65 | 70 | 75 | 80 | 85 | 88 |
|---|---|---|---|---|---|---|---|---|
| PWM | 5 | 16 | 31 | 48 | 67 | 120 | 190 | 255 |

Values reset at every boot. Only lower the curve if your console's cooling is in good shape.
</details>

**PS5 Hardware Control:** [NotHavocc](https://github.com/NotHavocc)'s app ([ps5-hwcon-gui](https://github.com/NotHavocc/ps5-hwcon-gui)) shows temperatures, clocks and fan with history charts, and sets the fan, GPU clock, LED colour and more. Open it from the app menu; it asks for your password. Changing the fan curve in it replaces the image's quiet curve until the next restart. Its CPU page works with the CPU frequency driver in kernel 7.3.5-pc8fe3 or newer (see below). For Gaming Mode there's the Decky version, [ps5-hwcon-plugin](https://github.com/NotHavocc/ps5-hwcon-plugin).

**CPU frequency:** from kernel 7.3.5-pc8fe3 (Update PS5 CachyOS installs it) the CPU speeds up and slows down with load like on a PC: `/sys/devices/system/cpu/cpufreq` with the usual governors (default `schedutil`) and a `boost` switch. Boost on (the default with CPU boost installed): every core at 3.5 GHz, for gaming. Boost off: 800 MHz-3.2 GHz with the load, cooler and quieter on the desktop: `echo 0 | sudo tee /sys/devices/system/cpu/cpufreq/boost`. Driver: [`patches/0009`](patches/0009-ps5-cpufreq.patch), PS5 and Slim (not the Pro yet).

**Video:** Firefox and Chrome use hardware video decode. The APU has **no AV1 decode**, so use the enhanced-h264ify extension for YouTube. KDE can't show GPU load; use `nvtop`.

## 8. Troubleshooting

### Black screen

The GPU sends DisplayPort to an internal DP→HDMI bridge, which loses sync if the link speed drops. The image locks the link at boot, so this mostly happens on first boot, after a resolution change or after replugging the cable. Try in order:

1. **Switch TTYs** with a USB keyboard: <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F2</kbd>, wait 3 seconds, <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F1</kbd>. Repeat once or twice.
2. **Switch the TV input** away and back.
3. **Re-lock the link** from tty2 (log in blind) or over SSH (`ssh steam@ps5`):
   ```sh
   echo "4 0x1e" | sudo tee /sys/kernel/debug/dri/0/DP-1/link_settings
   echo 1 | sudo tee /sys/kernel/debug/dri/0/DP-1/trigger_hotplug
   ```
   Still black? Run the second line again a few seconds later. With this repo on the PS5, `sudo ./scripts/fix-black-screen.sh` does it all.

A few `CR` errors in dmesg at boot are normal.

### Drive not found at boot

**"Waiting for device", then an emergency shell:**
1. **Push the cable all the way in**, rear USB or front USB-C. A half-seated plug can drop to USB 2.0.
2. **Unplug the drive and plug it back in while the boot waits.** Some drives get stuck at the handover to Linux.
3. At the emergency shell: replug, wait 10 seconds, `mount /dev/sda1 /new_root`, then `exit`.

**Slow drive?** `lsusb -t` should show `5000M` or `10000M` for the SSD. `480M` means USB 2.0: reseat the plug while Linux is off.

<details>
<summary>USB boot settings in <code>cmdline.txt</code></summary>

`cmdline.txt` on the `boot` partition contains `rootwait rootdelay=60` (wait for the disk), `usbcore.autosuspend=-1` (no USB sleep) and `usbc_dp.enable=0` (without it, a USB-C SSD on the front port drops off after boot).

- **USB-C SSD with resets or freezes?** Add `usb-storage.quirks=xxxx:yyyy:u` with your drive's ID from `lsusb`.
- **Renaming the root label** (useful with more than one ps5-linux drive): change `root=LABEL=` in `cmdline.txt` **and** the `/` line in `/etc/fstab`.
</details>

### Games lag or stutter at high settings

The GPU gets only **512 MB of VRAM** by default, too little for high texture settings, so games keep swapping textures and stutter. Raise it in **PS5 Hardware Control → Memory → VRAM** to **4-6 GB**, then turn the PS5 fully off and boot Linux through the loader again (a warm restart doesn't change it). VRAM comes out of the 16 GB RAM: with 6 GB, Linux keeps about 8 GB. If swap fills up in long sessions, use 4 GB.

### Wi-Fi and Bluetooth

- **Wi-Fi gone after a warm restart (kexec):** turn the PS5 **fully off** (not rest mode) and boot Linux again.
- **Wi-Fi `unavailable`:** `nmcli radio wifi on`.
- **PS5 Pro Bluetooth:** works with kernel 7.3.5-pc8fe or newer (the updater installs it, plus the Pro Bluetooth services). On older kernels the internal adapter disappears at boot ("wmt command timed out"); a USB Bluetooth dongle works meanwhile. After using the controller in PS5 OS, it sometimes connects on its second try (about 30 s).
- **DualSense:** reconnects on its own. Speaker and headphone jack only work over USB, and don't plug in USB while it's connected over Bluetooth.
- **Pairings from PS5 OS:** with a loader that exports them ([ps5-linux-loader](https://github.com/ps5-linux/ps5-linux-loader) newer than v2.5), controllers paired in PS5 OS work in Linux without pairing again ([ps5-bt-sync](https://github.com/ps5-linux/ps5-linux-patches/tree/main/bluetooth), installed by the updater). A device paired in both uses the PS5 OS pairing: if a keyboard or headset stops connecting in Linux, remove it in PS5 OS (Settings → Accessories) and pair it again in Linux. Without such a loader, using a controller in PS5 OS breaks its Linux pairing, so pair it again.

### Websites slow to start loading

Some routers' DNS stalls on certain lookups, so websites and Steam take 5-10 seconds to start. Fix: **Update PS5 CachyOS** → **d** (Cloudflare DNS) at the end, or in **System Settings → Wi-Fi & Internet** set IPv4 and IPv6 to **Automatic (Only addresses)** with DNS `1.1.1.1, 1.0.0.1`.

---

## Credits

- **[ps5-linux/ps5-linux-loader](https://github.com/ps5-linux/ps5-linux-loader)**: the Linux loader (firmware 3.00-13.60, PS5 Phat, Slim and Pro)
- **[ps5-linux](https://github.com/ps5-linux)**: [image](https://github.com/ps5-linux/ps5-linux-image) this guide's image is built on, [kernel patches](https://github.com/ps5-linux/ps5-linux-patches), [Wi-Fi driver](https://github.com/ps5-linux/ps5-linux-mwifiex), [tools](https://github.com/ps5-linux/ps5-linux-tools)
- **[QuartConz/ps5-pro-viola-bluetooth](https://github.com/QuartConz/ps5-pro-viola-bluetooth)** and **[WizGery/ps5-mediatek-fixes](https://github.com/WizGery/ps5-mediatek-fixes)**: PS5 Pro Bluetooth driver work
- **[MastaG](https://github.com/MastaG)**: PS5 Pro Bluetooth port and testing ([#7](https://github.com/abrocoma/PS5-CachyOS/pull/7)), faster DualSense reconnects
- **[NotHavocc](https://github.com/NotHavocc)**: [PS5 Hardware Control](https://github.com/NotHavocc/ps5-hwcon-gui) app and [Decky plugin](https://github.com/NotHavocc/ps5-hwcon-plugin)
- **[MassZero0](https://github.com/MassZero0/ps5-linux-cpuclock)**: found the SMU message for CPU P-states that `ps5-cpufreq` uses
- **Slip-Up**: pairing export in the loader ([ps5-linux-loader #97](https://github.com/ps5-linux/ps5-linux-loader/pull/97))
- **kadhtobi** and everyone who tested and reported issues, especially on the PS5 Pro and Slim

This repo only collects the fixes I needed on top.

## License

The scripts, files and docs in this repo are under the [GNU GPL-3.0](LICENSE): you may use, change and share them, and changed versions must stay under the GPL-3.0 with the source available. The kernel patches in `patches/` keep the license of the code they change (the Linux kernel's GPL-2.0), and the software in the image keeps its own licenses.

If this guide helped you, you can [buy me a coffee on Ko-fi](https://ko-fi.com/abrocoma). Thanks!
