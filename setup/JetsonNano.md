# Setting Up Jetson Nano with Ubuntu 20.04: A Streamlined Guide
---

The NVIDIA Jetson Nano is a powerful edge AI platform, but out of the box, it comes with unnecessary software that can slow it down. This guide will walk you through setting up Jetson Nano with Ubuntu 20.04, optimizing resources, and installing essential tools like JupyterLab.

## Step 1: Flash Ubuntu 20.04 Image

Download the pre-built Ubuntu 20.04 image from QEngineering:
[QEngineering Jetson Nano Ubuntu 20.04](https://github.com/Qengineering/Jetson-Nano-Ubuntu-20-image)

Write the image to your SD card using tools like `balenaEtcher` or `dd`. The system will require a reboot after the first boot.

## Step 2: Free Up Resources

### Remove Unnecessary Software
By default, the image includes Chromium and LibreOffice, which consume resources. Remove them with:
```sh
sudo apt-get remove --purge chromium-browser chromium-browser-l10n libreoffice*
```

### Disable Desktop Environment
Jetson Nano is often used in headless mode. Removing the desktop environment frees up memory and processing power:
```sh
sudo systemctl disable gdm3
sudo systemctl set-default multi-user.target

sudo apt remove --purge ubuntu-desktop gdm3
sudo apt autoremove
```

## Step 3: Resize SD Card Partition
If your SD card is larger than 32GB, you need to resize the partition:

1. Open fdisk:
   ```sh
   sudo fdisk /dev/mmcblk0
   ```
2. Follow these steps:
   ```
   - p (note the start sector of /dev/mmcblk0p1)
   - d (delete the partition)
   - 1 (the partition number to delete)
   - n (new partition)
   - First sector: <use the noted number from step 1>
   - Last sector: hit enter (use the full available space)
   - p (verify changes)
   - w (save and exit)
   ```
3. Reboot and resize:
   ```sh
   sudo reboot
   sudo resize2fs /dev/mmcblk0p1
   df -h  # Verify new size
   ```

## Step 4: Update and Upgrade System
Run the following commands to update your system:
```sh
sudo apt update
sudo apt upgrade
sudo apt clean
sudo apt autoremove
```

## Step 5: Install JupyterLab
JupyterLab is a great tool for development on Jetson Nano. Install it with:
```sh
pip3 install typing-extensions==3.7.4
pip3 install jupyterlab
```

## Step 6: Run JupyterLab
Log in with the default credentials (`jetson/jetson`) and start JupyterLab:
```sh
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token=
```
Now, you can access JupyterLab from your browser and start coding!

## Optional step to install DWA-171 drivers (if you have a DWA-171 dongle WiFi)
Linux 20.04 Tegra does not come with the drivers for DLink DWA-171 WIFI dongle.
Execute these steps to setup driver.

```
git clone https://github.com/CarlosDev314159/d-link-dwa-171-wifi-adapter-automatic-driver-installer.git
cd d-link-dwa-171-wifi-adapter-automatic-driver-installer
pip3 install -r requirements.txt
python3 main.py
```

Sometimes your module will not be detected, and it'll still show as a drive rather than a Wifi Module.

To fix that run `sudo usb_modeswitch -KW -v 0bda -p 1a2b`.  [source](https://forums.linuxmint.com/viewtopic.php?t=330933) 

---
Here's your hot off the pan, lightweight, optimized Jetson Nano, ready for AI and robotics projects. Bon apetite!
