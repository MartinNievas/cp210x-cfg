# cp210x-cfg
CLI utility for programming CP210x USB&lt;-&gt;UART bridges

Intended as a scriptable alternative to the official Silicon Labs CP21xx Customization Utility.

Tested with CP2102 and CP2105 chips.

Supported fields that can be programmed:
  - Vendor ID
  - Product ID
  - Product Name string
  - Serial string
  - Buffer flush bitmap
  - SCI/ECI gpio/modem mode

# CAUTION
Programming a CP210x config field is a write-once operation. It's perfectly possible to write bad values.
The consequences are yours, and yours alone.

Suffice to say that I've got a couple of modules which have a VID/PID of 0000:0000. Don't do that.
If you happen to screw up the VID/PID anyway, in Linux you can sort-of work around it by registering the
new VID/PID combo with the cp210x driver like so:

```
echo 0000 0000 | sudo tee /sys/bus/usb-serial/drivers/cp210x/new_id
```
Consider yourself forewarned, and forearmed.

## Examples
#### Showing the built-in help message (may differ from below)
```
$ ./cp210x-cfg -h
Syntax:
cp210x-cfg [-h ] |
           [-m vid:pid] [-d bus:dev]
           [ -l | [-V vid] [-P pid] [-F flush] [-M mode] [-N name] [-S serial]]

  -h            This help
  -m vid:pid    Find and use first device with vid:pid
  -d bus:dev    Find and use device at bus:dev
  -l            List all CP210x devices connected
  -V vid        Program the given Vendor ID
  -P pid        Program the given Product ID
  -F flush      Program the given buffer flush bitmap (CP2105 only)
  -M mode       Program the given SCI/ECI mode (CP2105 only)
  -N name       Program the given product name string
  -S serial     Program the given serial string

Unless the -d option is used, the first found CP210x device is used.
If no programming options are used, the current values are printed.
```

#### Switching a CP2105 into modem mode
```
$ sudo ./cp210x-cfg -d 4.113 -M 0
ID 10c4:ea70 @ bus 004, dev 113: CP2105 Dual USB to UART Bridge Controller
Model: CP2105
Vendor ID: 10c4
Product ID: ea70
Name: CP2105 Dual USB to UART Bridge Controller
Serial: 007079CC
Flush buffers: 33
SCI/ECI mode: 0000

```
### Config twins cp2102 usb-uart adapters
1. Get usb device bus and dev:
```bash
$ lsusb
Bus 001 Device 005: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 004: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
```
2. Config devices:
```
# first device
$ sudo ./cp210x-cfg -d 001.004 -S 1111
ID 10c4:ea70 @ bus 001, dev 004: Silicon Labs CP210x UART Bridge
Model: CP2105
Vendor ID: 10c4
Product ID: ea70
Name: Silicon Labs CP210x UART Bridge
Serial: 1111

# second device
$ sudo ./cp210x-cfg -d 001.005 -S 2222
ID 10c4:ea70 @ bus 001, dev 005: Silicon Labs CP210x UART Bridge
Model: CP2105
Vendor ID: 10c4
Product ID: ea70
Name: Silicon Labs CP210x UART Bridge
Serial: 2222
```
If it fail try again, be sure to check again device's bus and dev (`lsusb`)


3. Optional: create specific udev rules at `/etc/udev/rules.d/htb.rules`:
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="1111", MODE:="0777", SYMLINK+="htb_driver"
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="2222", MODE:="0777", SYMLINK+="rplidar"
```
Check:
```bash
$ ll /dev/rplidar
lrwxrwxrwx 1 user user 7 Apr  2 00:20 /dev/rplidar -> ttyUSB1
$ ll /dev/htb_driver
lrwxrwxrwx 1 user user 7 Apr  2 00:20 /dev/htb_driver -> ttyUSB0
```
