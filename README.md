## Troubleshooting "NotAllowedError: Failed to open the device" for K.T.E.C. Ergodone

This document outlines steps to resolve the "NotAllowedError: Failed to open the device" and "Received invalid protocol version from device" errors when trying to configure your K.T.E.C. Ergodone keyboard (Vid: 0x1209, Pid: 0x2328) on Linux, likely when using a web-based configuration tool like VIA.

These errors typically indicate that the software trying to access your keyboard does not have the necessary permissions to communicate with the device.

### Cause

The primary cause is often that the user (and therefore the browser running the configuration tool) does not have direct read/write access to the raw HID device file (e.g., `/dev/hidrawX`) associated with your keyboard. Web-based tools like VIA, especially when run through browsers like Chrome or Chromium, need these permissions to detect and interact with the keyboard.

### Solutions

Here are a couple of methods to resolve this issue, ranging from a temporary fix to a more permanent one.

#### Method 1: Temporarily Adjusting Device Permissions (for immediate testing)

This method involves manually changing the permissions of the device file. This change is temporary and will likely revert upon reboot or if the device is unplugged and replugged.

1.  **Identify the hidraw device:**
    *   Open a terminal.
    *   You might need to list available `hidraw` devices. You can try looking at `dmesg` output after plugging in the keyboard or checking logs in your browser if it provides any (e.g., `chrome://device-log` for Chromium-based browsers).
    *   The device will be something like `/dev/hidraw0`, `/dev/hidraw1`, etc. You may need to identify the correct one corresponding to your K.T.E.C. Ergodone. The `lsusb` command can help identify connected USB devices, and you can then correlate this with `hidraw` devices.

2.  **Grant permissions:**
    Once you've identified the correct device (e.g., `/dev/hidrawX`, replace `X` with the actual number):
    ```bash
    sudo chmod a+rw /dev/hidrawX
    ```
    This command gives all users read and write permissions to that specific device file.

3.  **Attempt to connect with your configuration tool (e.g., VIA).**

4.  **Revert permissions (optional but recommended for security):**
    After you are done configuring your keyboard, you can revert the permissions. If the original permissions were, for example, `crw------- 1 root root`, you can restore them:
    ```bash
    sudo chmod 600 /dev/hidrawX
    ```
    Or simply reboot, as these permissions are often reset.

#### Method 2: Using a `udev` Rule (Recommended for a permanent fix)

A `udev` rule is a more robust and persistent way to manage device permissions. It automatically sets the correct permissions for your keyboard whenever it's connected.

1.  **Create a `udev` rule file:**
    You'll need to create a new file in `/etc/udev/rules.d/`. For example, `50-ktec-ergodone.rules`.
    ```bash
    sudo nano /etc/udev/rules.d/50-ktec-ergodone.rules
    ```
    (You can use any text editor you prefer instead of `nano`).

2.  **Add the rule content:**
    The content of the rule should specify your keyboard's Vendor ID (0x1209) and Product ID (0x2328). A common rule for QMK-based devices (which many custom keyboards use) looks like this, adapted for your device:
    ```udev
    KERNEL=="hidraw*", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="2328", MODE="0666", TAG+="uaccess"
    ```
    Or, as suggested in the forum for QMK devices, you might use a rule that grants access to the `plugdev` group or directly sets `uaccess` which is often preferred:
    ```udev
    SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="2328", TAG+="uaccess"
    ```
    Alternatively, a rule from the QMK firmware repository (`qmk_firmware/util/udev/50-qmk.rules`) might be suitable. Such a rule might look like:
    ```udev
    # K.T.E.C. Ergodone
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="2328", MODE="0660", GROUP="plugdev"
    KERNEL=="hidraw*", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="2328", MODE="0660", GROUP="plugdev"
    ```
    *   `ATTRS{idVendor}=="1209"`: Matches your keyboard's vendor ID.
    *   `ATTRS{idProduct}=="2328"`: Matches your keyboard's product ID.
    *   `MODE="0666"` or `MODE="0660"`: Sets the permissions (read/write for all users, or read/write for owner and group).
    *   `GROUP="plugdev"`: Assigns the device to the `plugdev` group. Your user needs to be a member of this group (`sudo usermod -aG plugdev yourusername`).
    *   `TAG+="uaccess"`: This is often used with `systemd` to grant access to the currently logged-in user at the console. This is generally a good option.

3.  **Reload `udev` rules:**
    After saving the file, tell `udev` to reload its rules:
    ```bash
    sudo udevadm control --reload-rules
    sudo udevadm trigger
    ```

4.  **Reconnect your keyboard:**
    Unplug and replug your K.T.E.C. Ergodone keyboard to allow the new rule to take effect.

5.  **Test again with your configuration software.**

### Additional Considerations

*   **Browser Compatibility:** Ensure you are using a Chromium-based browser (like Google Chrome, Chromium, Brave, etc.) for web-based configurators like VIA. These tools often do not work with Firefox.
*   **Correct Device IDs:** Double-check that the Vendor ID (`0x1209`) and Product ID (`0x2328`) in your error message and `udev` rule are correct for your specific device.

By following these steps, you should be able to grant the necessary permissions for your configuration software to connect to your K.T.E.C. Ergodone keyboard. The `udev` rule method is generally preferred for a long-term solution.
