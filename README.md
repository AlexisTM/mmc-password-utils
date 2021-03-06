# mmc-password-utils
User layer support for the kernel MMC Password Lock/Unlock feature
This contains the user space helper functions used by the MMC kernel driver password feature to enable the use of passwords to  protect MMC and SD devices. The MMC driver will use the Linux KEYS subsystem to get a password for password based operations on SD/MMC devices and requires that the "keyutils" package be installed on the system.

Most of these files are used to configure the KEYS user space helper layer to return a password to the kernel driver for any SD/MMC device based on its CID. The CID is a 32 digit hex string that is guaranteed to be unique for any device and can be determined by doing a cat of "/sys/bus/mmc/devices/mmc0\*/cid" for a device connected to the first MMC host controller (use mmc1* for the second MMC host controller, etc.). This project was created to facilitate testing of the kernel driver and uses a very basic clear text password file that contains just the CID/password pairs. Passwords must be added and deleted from the password file manually and the password for a device must be added before enabling password protection on the device.

The MMC driver will automatically try to unlock a password protected MMC/SD device during probe by using the KEYS subsystem and does not require user intervention. All other password operations, like setting and clearing the password, are done through sysfs. Commands can be echo'd to the sysfs file "/sys/bus/mmcdevices/mmc0*/lock" to trigger password operations. NOTE: The password/CID pair must be added to the password file before all operations except "erase". Following is a list of the possible commands:
setpw - Set the SD/MMC devices password. The next time the device is power cycled it will come up locked.
clrpw - Clear the SD/MMC devices password. This only works on unlocked password protected devices.
lock - Lock the SD/MMC device. The device must already have a password set.
unlock - Unlock the SD/MMC device. The device must already have a password set. 
erase - Erase the entire devices contents and then clear the password. Only works on a locked device.

There can be an issue when booting a system with a locked device already installed where the unlock operation during the device probe happens before the filesystem that contains the password file is available. In this case the kernel driver will initialize the device enough to create the sysfs entries and the /dev/mmcblk0 special device even though all read/writes to the block device will fail. Once the filesystem containing the password information is available, doing an echo to "/sys/bus/mmc/devices/mmc0*/unlock_retry" will cause the driver to retry the unlock operation and if successful to bring the device up fully, including all it's partitions. The best way to handle this issue is to run the included mmc-password-retry rc file during boot after sysfs and the required filesystem are available.

The supplied Makefile will install the config files and scripts.
