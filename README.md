# Installing NVIDIA Drivers on Kali Linux 2022

## 1. Checking the Graphics Card

Check which graphics card is being used

```bash
lspci | grep -E "VGA|3D"
```

## 2. Disable the NOEVEAU Proprietary Driver

To disable **NOEVEAU**, it needs to be blacklisted, update the configuration, and reboot the machine

```bash
sudo echo -e "blacklist nouveau\noptions nouveau modeset=0\nalias nouveau off" | sudo tee --append /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
sudo reboot
```

## 3. Checking the NOEVEAU Driver

After rebooting, check if the **NOEVEAU** driver is working

```bash
lsmod | grep -i nouveau
```

## 4. Installing NVIDIA Drivers

After disabling **NOEVEAU**, you can install the **NVIDIA** drivers
Most likely, the drivers installed will not be the latest, but they will be stable

```bash
sudo apt-get install nvidia-driver nvidia-xconfig nvidia-prime
```

## 5. Checking the Graphics Card Bus

Check which bus the discrete graphics card is on, in order to record these values in the configuration (next step)

```bash
nvidia-xconfig --query-gpu-info | grep 'BusID'
```

Remember the BusID values, for example `PCI:3:0:0`

## 6. Creating 'Xconfig'

**Xconfig** is needed for the system to understand how to work with the graphics card; the following example makes the system work with the discrete graphics card.

- The configuration should be stored at the following path: `/etc/X11/xorg.conf `
- Replace `PCI:x:x:x` with your own value taken from the previous step.

```bash
Section "ServerLayout"
    Identifier "layout"
    Screen 0 "nvidia"
    Inactive "intel"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "PCI:x:x:x"
EndSection

Section "Screen"
    Identifier "nvidia"
    Device "nvidia"
    Option "AllowEmptyInitialConfiguration"
EndSection

Section "Device"
    Identifier "intel"
    Driver "modesetting"
EndSection

Section "Screen"
    Identifier "intel"
    Device "intel"
EndSection
```

If you are not satisfied with this configuration, you can customize it following this [instruction](http://us.download.nvidia.com/XFree86/Linux-x86/375.39/README/randr14.html "instruction").

Alternatively, you can try generating the correct `/etc/X11/xorg.conf` using the `nvidia-xconfig` command.

```bash
nvidia-xconfig --enable-all-gpus
nvidia-xconfig --cool-bits=4
```

## 7. Creating DisplayManager Script

The script should fit your desktop environment. All scripts for desktop environments can be found [here](https://wiki.archlinux.org/title/NVIDIA_Optimus#Display_Managers "here").

Check which DisplayManager you have

```bash
cat /etc/X11/default-display-manager
```

### 7.1 Script for LightDM

```bash
sudo nano /etc/lightdm/display_setup.sh

# Script:
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

### 7.2 Script for SDDM (Default for KDE)

```bash
sudo nano /usr/share/sddm/scripts/Xsetup

# Script:
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

### 7.3 Script for GDM

```bash
sudo nano /usr/share/gdm/greeter/autostart/optimus.desktop
sudo nano /etc/xdg/autostart/optimus.desktop

# Script:
[Desktop Entry]
Type=Application
Name=Optimus
Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
NoDisplay=true
X-GNOME-Autostart-Phase=DisplayServer
```

## 8. Reboot the System

After creating the scripts and configurations, reboot the system

```bash
sudo reboot -h now
```

## 9. Checking NVIDIA Driver Usage

If your system is using **NVIDIA** drivers, the output of the following command will be `direct rendering: Yes`.

```bash
glxinfo | grep -i "direct rendering"
```

## 10. Installing CUDA Toolkit

Now you can install **CUDA**

```bash
sudo apt-get install ocl-icd-libopencl1 nvidia-cuda-toolkit
```

## 11. Updating GRUB

If you have multiple operating systems on one machine, you need to update GRUB.

### 11.1 Checking PRIME NVIDIA Usage

All values should be "1" (Example: `PRIME Synchronization: 1`)

```bash
xrandr --verbose | grep PRIME
```

### 11.2 Updating GRUB

The **GRUB** configuration is located at: `/etc/default/grub`

You need to change the following field to the code below:

```bash
....
GRUB_CMDLINE_LINUX_DEFAULT="quiet nvidia-drm.modeset=1"
...
```

After saving, update **GRUB**

```bash
sudo update-grub
sudo reboot -h now
```

After rebooting, check if **PRIME NVIDIA** is being used

```bash
xrandr --verbose|grep PRIME
```

The output should be the following:

```bash
PRIME Synchronization: 1
PRIME Synchronization: 1
```

# Potential Issues

## 1. Black Screen

If you see a black screen after rebooting, you need to:

1. Switch to a pseudo-graphical terminal (ALT+CTRL+F2 or ALT+CTRL+F3 and so on).
2. Remove the previously created configurations; they were likely created incorrectly and are causing the display not to work.

```bash
sudo apt-get remove --purge nvidia*
sudo rm -rf /etc/X11/xorg.conf

# For LightDM
sudo rm -rf /etc/lightdm/lightdm.conf

# For SDDM
sudo rm -rf /usr/share/sddm/scripts/Xsetup

# For GDM
sudo rm -rf /usr/share/gdm/greeter/autostart/optimus.desktop
sudo rm -rf /etc/xdg/autostart/optimus.desktop
```

3. Reboot the system

```bash
sudo reboot -h now
```

## 2. NVIDIA Driver Installation Failure

1. Check if **NOUVEAU** is actually in use

```bash
lsmod | grep -i nouveau
```

2. If it is in use, disable it by proceeding to [step 2](#2-Disable-the-NOEVEAU-Proprietary-Driver "step 2")


```bash
sudo echo -e "blacklist nouveau\noptions nouveau modeset=0\nalias nouveau off" > /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
```

3. Reboot the system

```bash
sudo reboot -h now
```
