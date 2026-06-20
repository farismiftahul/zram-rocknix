# Rocknix ZRAM Swap Script & Manager for Handheld Devices

A collection of tools to configure and manage ZRAM swap memory on ROCKNIX-powered handheld gaming devices. This helps improve memory management, reduce stuttering, and prevent out-of-memory (OOM) crashes in demanding games/emulators by allocating compressed RAM as swap space.

You can set up ZRAM in two ways:
1. **[Method 1: Gamepad TUI ZRAM Manager (Recommended)](#method-1-gamepad-tui-zram-manager-recommended)** - A full terminal user interface (TUI) that can be run from the EmulationStation Ports menu and navigated entirely using your device's joystick/d-pad.
2. **[Method 2: Manual Autostart Script](#method-2-manual-autostart-script)** - A quick, lightweight bash script written directly to ROCKNIX's autostart directory.

---

## Method 1: Gamepad TUI ZRAM Manager (Recommended)

The ZRAM Manager is a Portmaster-compatible utility that provides a terminal menu system. It allows you to:
* **Check live statistics** (free/used RAM, active swap space, ZRAM compression ratio, memory allocated).
* **Enable ZRAM** using either optimized quick settings (50% RAM, `lz4` compression) or custom configurations.
* **Select compression algorithms** supported by your kernel (e.g., `lz4`, `zstd`, `lzo`).
* **Toggle autostart** on or off dynamically.
* **Operate it using the gamepad** (D-pad/Joystick for navigation, `A` to select, `B` to cancel).

### Installation Instructions

1. **Download the TUI Manager files** from this repository:
   * [ZRam_Manager.sh](file:///d:/Projects/zram-rocknix/ZRam_Manager.sh)
   * [zram-manager.gptk](file:///d:/Projects/zram-rocknix/zram-manager.gptk)
2. **Transfer these files** to your handheld device's ports folder at `/storage/roms/ports/` (using SFTP, Samba network share, or an SSH terminal).
3. **Make the script executable** via SSH:
   ```bash
   chmod +x /storage/roms/ports/ZRam_Manager.sh
   ```
4. **Launch the TUI**:
   * Restart EmulationStation or refresh your game list.
   * Go to the **Ports** menu on your handheld and launch **ZRam_Manager**.
   * Use your D-pad/Left Joystick to navigate, `A` to enter/select, and `B` to exit.

---

## Method 2: Manual Autostart Script

If you don't need a terminal user interface and want to manually enable ZRAM, follow the steps below.

### Connecting via SSH
Before running the commands below, you need to connect to your ROCKNIX handheld device from another device (such as a PC) over your local network:
1. **Find your Handheld's IP Address:** Look up your device's IP address (e.g., `192.168.x.x`) under the network settings of your handheld.
2. **Open your SSH Client:** Use a client like **PuTTY** (on Windows) or the terminal (on macOS/Linux).
3. **Login Credentials:**
   * **Host Name/IP:** The IP address of your handheld (e.g., `192.168.x.x`)
   * **Port:** `22`
   * **User:** `root`
   * **Password:** `rocknix` *(This is the default password. You can check or change it under **Settings** -> **Security** on your handheld).*

---

### 1. Create the Directory and Script File
Once connected via SSH, run the following commands to create the autostart directory and a sequentially numbered script file:

```bash
mkdir -p /storage/.config/autostart
nano /storage/.config/autostart/010-zram-swap
```

### 2. Paste the Script Content
Inside the nano editor, paste the following script logic. Keeping the delay helper (`sleep 5`) is highly recommended to prevent the script from running before the kernel modules are fully available:

```bash
#!/bin/bash

# Give the core subsystems a moment to stabilize
sleep 5

# Enable ZRAM
modprobe zram num_devices=1
TOTAL_RAM=$(awk '/MemTotal/ {print $2}' /proc/meminfo)
ZRAM_SIZE=$((TOTAL_RAM * 1024 / 2))
echo lz4 > /sys/block/zram0/comp_algorithm
echo $ZRAM_SIZE > /sys/block/zram0/disksize
mkswap /dev/zram0
swapon -p 100 /dev/zram0
sysctl -w vm.swappiness=100
```
*(If you want to use the standard swap file fallback instead of ZRAM, you can just paste the swap file code block here instead).*

Press `Ctrl+O` and then `Enter` to save, and `Ctrl+X` to exit nano.

> [!TIP]
> **Quick Installation Alternative:**
> Instead of manually copy-pasting, you can download the script directly to your device via SSH using this command:
> ```bash
> wget -O /storage/.config/autostart/010-zram-swap https://raw.githubusercontent.com/farismiftahul/zram-rocknix/main/010-zram-swap
> ```

### 3. Make it Executable
Make sure to set the execution permissions on the newly created script file:

```bash
chmod +x /storage/.config/autostart/010-zram-swap
```

Now, when you restart your ROCKNIX device, it will scan the directory, pick up `010-zram-swap`, and initialize your memory configurations before launching the EmulationStation frontend!

---

## Disabling or Removing the Manual Script

To disable the script in ROCKNIX, you have two clean options depending on whether you want to turn it off temporarily or remove it completely.

### Option 1: Disable it temporarily (Recommended)
The easiest way to stop the script from running without deleting your hard work is to remove its executable permission. If a script in the autostart folder isn't marked as executable, ROCKNIX will simply skip right over it during boot.

Run this command in your terminal:

```bash
chmod -x /storage/.config/autostart/010-zram-swap
```

If you ever want to turn it back on in the future, just swap the minus sign for a plus sign:
```bash
chmod +x /storage/.config/autostart/010-zram-swap
```

### Option 2: Remove it completely
If you decide you no longer need the configuration at all, you can safely delete the file from the autostart directory:

```bash
rm /storage/.config/autostart/010-zram-swap
```

Once you've done either option, just restart your device, and your system memory behavior will return back to its default ROCKNIX stock settings!
