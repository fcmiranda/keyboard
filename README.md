# Fixing "NotAllowedError: Failed to open device" for K.T.E.C. ErgoDone Keyboard

This guide outlines how to resolve the "NotAllowedError: failed to open the device. device: k.t.e.c. ergodone" error, which typically occurs when trying to access the ErgoDone keyboard with tools like QMK Toolbox, Vial, or web-based configurators that require raw HID access.

The information is primarily sourced from [this Arch Linux forum thread](https://bbs.archlinux.org/viewtopic.php?id=285709).

## Problem

When attempting to flash firmware or use a configurator with the K.T.E.C. ErgoDone keyboard, you encounter an error similar to:

NotAllowedError: Failed to open the device.

Device: K.T.E.C. ErgoDone

### 1. Temporarily Change Permissions

*  Run the following command, replacing `/dev/hidraw3` with the correct device file you identified:
    
    ```bash
    sudo chmod a+rw /dev/hidraw3
    ```
    This gives all users read and write permission to the device.

*  Close the browser

### 2. Use the VIA Website

Go to [https://usevia.app](https://usevia.app) in your browser (Chrome, Edge, or another Chromium-based browser that supports WebHID).
*   Click "Authorize device +".
*   Select your "K.T.E.C. ErgoDone" (or similar) from the list and click "Connect".


You should now be able to configure your keyboard.

### 3. Restore Original Permissions (Important!)

Once you are done configuring your keyboard on VIA, it's good practice to restore the original, more restrictive permissions. Default permissions are often `600` (read/write for root only).

```bash
sudo chmod 600 /dev/hidraw3
```
(Again, replace `/dev/hidraw3` with your actual device).
