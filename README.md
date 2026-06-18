# Rocknix ZRAM Swap Script for Handheld Devices

A lightweight autostart script to enable ZRAM swap on ROCKNIX-powered handheld devices. This helps improve memory management, reduce stuttering, and prevent out-of-memory (OOM) crashes in demanding games/emulators by allocating compressed RAM as swap space.

## Installation

### 1. Create the Directory and Script File
Connect to your ROCKNIX device via SSH (or use a file manager) and run the following commands to create the autostart directory and a sequentially numbered script file:

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

## Disabling or Removing the Script

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
