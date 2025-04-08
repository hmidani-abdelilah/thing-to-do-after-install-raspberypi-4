# thing-to-do-after-install-raspberypi-4
### Step 1: 📦 APT Package Manager & System Update

```bash
sudo apt update          # Refresh the list of packages
sudo apt upgrade -y      # Upgrade all installed packages
sudo apt full-upgrade -y # full upgrade
```

### Step 2: 🧬 Update Raspberry Pi Firmware

**Why:** Keeps your Pi's firmware (bootloader, EEPROM) up-to-date for stability and new features.

**How:**

```bash
sudo rpi-update    # firmware updater 
sudo rpi-eeprom-update   # Check for available EEPROM updates
sudo reboot              # Reboot to apply firmware updates
```

### Step 3: 🧰 Install Base Packages

**Why:** These are essential tools for development and hardware control.

#### ⚙️ Install and Configure GPIO Tools

```
sudo apt install python3-gpiozero python3-rpi.gpio pigpio wiringpi i2c-tools
```

**How:**

```bash
sudo apt install -y git curl  build-essential python3 python3-pip  
```

* * *

# copy files from-to

- ssh-keygen
- ssh-copy-id
- sftp scp
- vscode
- compile gcc

* * *

# 🌐 Configure WiFi Auto-Connect

### Option 1: Using NetworkManager GUI

- if you need to show ip address

```
ifconfig
```

or

```
ip addr show 
```

- if you need check status of devices network :

```
sudo nmcli device status
```

- If the wifi radio has been disabled in NetworkManager, enable this feature:

```
nmcli wifi radio on
```

- If you need to find the network name of your local network you can run the following command in the Raspberry terminal:

```
sudo iwlist wlan0 scan
```

or

```
nmcli device wifi rescan 
```

- if you need to view list wifi via nmcli :

```
nmcli device wifi list
```

- if you need to connect wfi via nmcli :

```
nmcli device wifi connect <<SSID-WIFI>> password <<PASSWORD-WIFI>>

```

### 1\. Option 2 : nmcli

```bash
nmcli dev wifi list
nmcli dev wifi connect "SSID" password "your_password"
nmcli connection modify "SSID" connection.autoconnect yes
```

- if you need to show SSID and PASSWORD are connected:

```
nmcli device wifi show-password ifname wlan0
```

#### docs

- if you need to auto-connect wifi with nmcli "wlan0" is name of the interface :

```
nmcli device set wlan0 autoconnect yes
```

```
nmcli connection modify <<SSID-YOUR-WIFI>> connection.autoconnect yes
```

- if you have more than one network wifi use priority :

```
nmcli connection modify <<SSID-YOUR-NETWORK>> connection.autoconnect-priority  1
```

| The autoconnect priority in range -999 to 999. If the connection is set to autoconnect, connections with higher priority will be preferred. The higher number means higher priority. Defaults to 0.

- if you need to use auto autoconnect retries :

```
nmcli connection modify <<SSID-YOU-WIFI>> connection.autoconnect-retries -1
```

| The number of times a connection should be tried when autoactivating before giving up. Zero means forever, -1 means the global default (4 times if not overridden). Setting this to 1 means to try activation only once before blocking autoconnect. Note that after a timeout, NetworkManager will try to autoconnect again.

### 2\. Option 2 : wpa_supplicant

- First, you’ll want to create a wpa_supplicant.conf file with the following content :
    - nano is text editor

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

- content file :

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Country Code>

network={
     ssid="<SSID-1>"
     psk="<PASSWORD>"
     key_mgmt=WPA-PSK
     scan_ssid=1
}
network={
     ssid="<SSID-2>"
     psk="<PASSWORD>"
     key_mgmt=WPA-PSK
     # this config is 0 or 1 for hidden wifi 1 for hidden 
     scan_ssid=1
}
```

- Remember to replace <country code="">, <ssid>, and <password>with your own country code, WiFi SSID and password.</password></ssid></country>
    
- country code : https://www.iban.com/country-codes
    
- Morocco is : MA
    
- type 'ctrl + x' bottom to save the file and type 'enter' bottom
    
- restart the service
    

```
sudo systemctl restart wpa_supplicant
```

- check status to see your config is correct or not :

```
sudo systemctl status wpa_supplicant
```

* * *

### Step 4: 🧠 Install & Configure ZRAM and Resize Swap

1.  Swap resize :

- to check user

```
free -mh
```

- turn off the active swap space

```
sudo dphys-swapfile swapoff
```

- Edit the swap configuration file

```
sudo nano /etc/dphys-swapfile
```

- Look for the line CONF_SWAPSIZE=200 (or a similar value).This line defines the current swap size in megabytes.
- Increase the swap size
- Change the value to your desired size. For instance, setting it to 2048 will give you 2GB of swap space

```
CONF_SWAPSIZE=2048
```

- Apply the new swap size . After updating the configuration, recreate the swap file to match the new size by running:

```
sudo dphys-swapfile setup
```

- Turn the swap back on. Now, re-enable swap with:

```
sudo dphys-swapfile swapon
```

- and reboot

```
sudo reboot now
```

* * *

## Install zram-tools

```
sudo apt install zram-tools
```

## Configuration

|You can edit `/etc/default/zramswap` to change the compression algorithm or swap allocation. Before you do that it is essential to note that  
|the ZRAM device size refers to the **uncompressed data** size, actual memory utilization will be ~2-3x smaller than the zram device size due to compression.

`sudo nano /etc/default/zramswap`

Here is an example:

```
# Compression algorithm selection
# speed: lz4 > zstd > lzo
# compression: zstd > lzo > lz4
# This is not inclusive of all that is available in latest kernels
# See /sys/block/zram0/comp_algorithm (when zram module is loaded) to see
# what is currently set and available for your kernel[1]
# [1]  https://github.com/torvalds/linux/blob/master/Documentation/blockdev/zram.txt#L86
#ALGO=lz4

# Specifies the amount of RAM that should be used for zram
# based on a percentage the total amount of available memory
# This takes precedence and overrides SIZE below
#PERCENT=50

# Specifies a static amount of RAM that should be used for
# the ZRAM devices, this is in MiB
# Use 256 for a Raspberry Pi Zero 2 with 512MB of RAM
#SIZE=256
# Use 1024 for a Raspberry Pi 4 or Raspberry Pi 5 with 4GB of RAM
#SIZE=1024

# Specifies the priority for the swap devices, see swapon(2)
# for more details. Higher number = higher priority
# This should be higher than hdd/ssd swaps.
PRIORITY=100
```

Then restart zramswap with `systemctl restart zramswap`.  
Kernel Parameters to make better use of ZRAM  
The Linux kernel can be tuned and tweaked to make better use of ZRAM.

- Edit /etc/sysctl.conf with sudo vi /etc/sysctl.conf and add these lines and the end:

```
vm.vfs_cache_pressure=200
vm.swappiness=80
vm.dirty_background_ratio=1
vm.dirty_ratio=50
vm.watermark_boost_factor = 0
vm.watermark_scale_factor = 125
vm.page-cluster = 0
```

| Parameter | Effect |
| --- | --- |
| vfs_cache_pressure = 500 | Aggressively drops cache (less memory used, but slower access) |
| swappiness = 100 | Uses swap a lot — suitable only if using ZRAM |
| dirty_background_ratio = 1 | Starts background disk writes early |
| dirty_ratio = 50 | High limit for blocking writes, allows bursty dirtying |
|     |     |
| watermark_boost_factor = 0 | No emergency boost in memory pressure |
|     |     |
| watermark_scale_factor = 125 | Slightly conservative free memory policy |
|     |     |
| page-cluster = 0 | Swaps pages one at a time (better with ZRAM) |

- Then reboot your system, or enable the new parameters with:

```
sudo sysctl --system
```

or

```
sudo sysctl -p
```

### Step 4:📊 System Monitoring Tools

```
sudo apt install btop lm-sensors
```

- btop: Interactive system monitor
    
- lm-sensors: Hardware monitoring (temperature, voltage, fan speed)
    
    - To detect sensors:

```
sudo sensors-detect
sensors
```

- For network traffic:

```bash
sudo apt install ntopng
```

- Access via browser: http://localhost:3000

### Step 5: 🚀 Overclocking the Raspberry Pi 4

#### Purpose:

To boost performance for heavier tasks in robotics/embedded projects.

## Warning:

**Ensure you have proper cooling (heatsink/fan).**

#### Steps:

Edit the boot config:

```bash
sudo nano /boot/firmware/config.txt
```

Add:

```ini
over_voltage=6
arm_freq=2000
gpu_freq=750
```

- Reboot to apply.
    - example file :

```
# For more options and information see http://rpf.io/configtxt Some settings may impact device functionality. See link above for
# details

# uncomment if you get no picture on HDMI for a default "safe" mode
#hdmi_safe=1

# uncomment the following to adjust overscan. Use positive numbers if console
# goes off screen, and negative if there is too much border
#overscan_left=16
#overscan_right=16
#overscan_top=16
#overscan_bottom=16

# uncomment to force a console size. By default it will be display's size minus
# overscan.
#framebuffer_width=1280
#framebuffer_height=720

# uncomment if hdmi display is not detected and composite is being output
#hdmi_force_hotplug=1

# uncomment to force a specific HDMI mode (this will force VGA)
#hdmi_group=1
#hdmi_mode=16

# uncomment to force a HDMI mode rather than DVI. This can make audio work in
# DMT (computer monitor) modes
#hdmi_drive=2

# uncomment to increase signal to HDMI, if you have interference, blanking, or
# no display
#config_hdmi_boost=4

# uncomment for composite PAL
#sdtv_mode=2

#uncomment to overclock the arm. 700 MHz is the default.
#arm_freq=800

# Uncomment some or all of these to enable the optional hardware interfaces
dtparam=i2c_arm=on
#dtparam=i2s=on
dtparam=spi=on

# Uncomment this to enable infrared communication.
#dtoverlay=gpio-ir,gpio_pin=17
#dtoverlay=gpio-ir-tx,gpio_pin=18

# Additional overlays and parameters are documented /boot/overlays/README

# Enable audio (loads snd_bcm2835)
dtparam=audio=on

# Automatically load overlays for detected cameras
camera_auto_detect=1

# Automatically load overlays for detected DSI displays
display_auto_detect=1

# Enable DRM VC4 V3D driver
#dtoverlay=vc4-kms-v3d
#max_framebuffers=2

# Run in 64-bit mode
arm_64bit=1

# Disable compensation for displays with overscan
disable_overscan=1

[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
otg_mode=1

[all]

[pi4]
# Run as fast as firmware / board allows
arm_boost=1

[all]
dtoverlay=w1-gpio
enable_uart=1
#enable_tvout=1
gpu_mem=256
hdmi_enable_4kp60=1
#sdtv_mode=2
#sdtv_aspect=3
#enable_tvout=1
```

### test

```bash
sudo apt install stress
```

```bash
stress --cpu 4 --io 4 --vm 2 --vm-bytes 128M
```

### Step 6 :🧵 Apply Kernel Low Latency Patch

Purpose:  
Reduce latency for real-time applications (robotics, motor control, etc.)

## Options:

#### Option 1: Use Precompiled RT Kernel

Search and install a low-latency kernel package (if available)

```bash
sudo apt install linux-image-rt-arm64
```

#### Option 2: Compile Your Own Kernel with RT Patch

- Clone kernel source
    
- Apply PREEMPT_RT patch
    
- Compile and install manually
    
- https://jaybrockway.us/computer-science/raspberry-pi-kernel-real-time-build/
    

### Other docs:

https://github.com/raspberrypi/documentation/blob/develop/documentation/asciidoc/computers/raspberry-pi/boot-eeprom.adoc  
https://github.com/raspberrypi/documentation/blob/develop/documentation/asciidoc/computers/raspberry-pi/boot-eeprom-diagnostics.adoc  
https://www.raspberrypi.com/documentation/computers  
https://www.raspberrypi.com/documentation/computers/config_txt.html#what-is-config-txt  
https://github.com/raspberrypi/documentation/tree/develop
