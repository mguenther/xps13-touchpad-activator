# DELL XPS13 (2015) Touchpad Activator

The DELL XPS13 (2015) still has some issues when using Ubuntu 14.04 LTS. Most of the issues can be resolved by installing a Linux kernel of the 4.x-line. However, switching the enabled status of the touchpad is still finicky. Since I use a small mouse most of the time when working with the machine, I'd like to deactivate / activate the touchpad in a convenient way.

For that matter, I wrote a small BASH script that allows me to toggle the enabled status of the touchpad. The BASH script uses `xinput` to read the current status of the touchpad and sets it to enabled / disabled as appropriate.

I got most of the information for fixing the issues with Ubuntu 14.04 LTS and the DELL XPS13 (2015) from [this blog post](http://forthescience.org/blog/tag/xps13.html) which goes into great detail on how to configure the system properly.

## Prerequisites

The DELL XPS13 (2015) touchpad can operate via PS2 or I2C. In my case, both operation modes showed an entry via `xinput`. This seems to cause another problem: If you use the Ubuntu System Tools to manually deactivate the touchpad, it will switch right back to the enabled state once you disabled it. To fix this behavior, you have to blacklist the PS2 operation mode, which can be done as follows.

Use `xinput` to get some information on the touchpanel.

	$ xinput

This should yield something like this:

	Virtual core pointer                    id=2   [master pointer  (3)]
	  ↳ Virtual core XTEST pointer          id=4    [slave  pointer  (2)]
	  ↳ DLL0665:01 06CB:76AD UNKNOWN        id=14   [slave  pointer  (2)]


If there is no mouse connected to the system, these two entries should be the only ones showing under "Virtual core pointer". If you notice something like "PS2 touchpad", then the PS2 operation is still active. To blacklistn the PS2 operation mode, add the line `blacklist psmouse` to `/etc/modprobe.d/blacklist.conf`.

	$ echo -e "\nblacklist psmouse" | sudo tee -a /etc/modprobe.d/blacklist.conf
	$ sudo update-initramfs -u

Reboot the system and check the status using `xinput` again. It should look like the example above.

## Using the script

The script reads the device ID of the virtual core pointer associated with the touchpad in I2C mode. To extract the necessary information from xinput, the script needs to identify the correct device. On my machine, the device name is "DLL0665:01 06CB:76AD UNKNOWN". The script specifically looks for this device name within the output if `xinput`, so if this is different on your machine for whatever reason, you need to adjust the variable `$TOUCHPAD_DEVICE_PATTERN` to your needs.

The script does not preserve the last enabled status between system reboots.

### Toggling the enabled status

Calling the script without additional parameters toggles the enabled status.

	$ touchpad-activator

### Disabling the touchpad

Calling the script with the parameter `off` disables the touchpad.

	$ touchpad-activator off

### Enabling the touchpad

Calling the script with the parameter `on` enables the touchpad.

	$ touchpad-activator on

### Showing the current status

Calling the script with the parameter `status` shows the associated device ID and the current enabled status.

	$ touchpad-activator status

This yields

	Touchpad Device ID: 14
	Touchpad Status   : Off

## Logging

The script creates the directory `/home/$USER/.touchpad-activator` and logs its activity and errors to the file `/home/$USER/.touchpad-activator/touchpad-activator.log`.

## License

This script is released under the MIT license.
