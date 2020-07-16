# Sailfish OS port to Sony Xperia Tama devices

Please read this document fully before planning or starting to use this port.

This is a community port, meaning that there is no official support and extensions included in the paid version:

* There is no Android app support
* There is no MS exchange support
* There is no Jolla Store predictive text support. Use Presage-based keyboards instead (see below)

The port is based on official port for Xperia 10. As a result, it has similar tools available for flashing
and the device userdata is partitioned using LVM.

As it requires unlocking bootloader on Sony Xperia, you will loose DRM keys and associated functionality. See
AOSP9 threads at XDA for your device for details. In particular, camera functionality will be considerably worse
than on stock Android by Sony, but should be similar to AOSP9.

While the care has been taken during porting, please see [LICENSE](LICENSE) for legal details.

## Issues 

Port issues are all reported within this repository: https://github.com/sailfishos-sony-tama/main/issues

## Supported devices

The following devices are supported:

* Xperia XZ2 single sim variant (H8216)
* Xperia XZ2 dual sim variant (H8266)
* Xperia XZ2 Compact single sim variant (H8314)
* Xperia XZ2 Compact dual sim variant (H8324)
* Xperia XZ3 single sim variant (H8416)
* Xperia XZ3 dual sim variant (H9436)

## Over-the-Air updates (OTA)

OTA updates are supported. Currently supported OTA updates are to the following releases:
- 3.3.0.16
- 3.2.1.20

OTA updates are supported via command line, as described below.
- Backup of Sailfish OS user files to a sdcard or to another device is strongly encouraged before updating Sailfish OS.
- Updates are tested by making updates between consecutive versions of Sailfish. Before skipping versions,
  check at [TJC](https://together.jolla.com) whether it is recommended. In addition, read the
  [release notes](https://github.com/sailfishos-sony-tama/main/releases) for the versions that you plan to skip.
- Enable ability to change to root in Settings/Developer tools:
  - Enable 'Developer tools'
  - Set the password
  - Allow 'Remote connection' if you wish to update via ssh
- Open shell with normal nemo user preferrably via ssh
- Updating Sailfish OS via commandline:
```bash
# Replace with the release you are updating to
ssu release 3.3.0.16

ssu lr
# Check the output that you have repos adaptation-community and adaptation-community-common

# You may have many of OpenRepos enabled. It's recommended to disable them, even
# though version --dup will do its best-effort to isolate repositories:
ssu lr | grep openrepos

devel-su zypper clean -a
devel-su zypper ref -f

version --dup
# if above fails, try again
# version --dup
sync
```
- Reboot

## Current state

Port is based on AOSP9 / Linux kernel 4.9.

Software stack state:

* Jolla Store access
* Kernel OTA update works
* OTA update for full OS not tested

Working hardware:

* Display
* Touch, multitouch
* LED
* Audio
* Bluetooth
* GPS
* WLAN (connect and hotspot)
* Camera (as in AOSP9)
* GSM (SMS, voice, data)
* Keys (Vol +/-, camera, power)
* Power management
* USB Charging, Network, MTP
* Wireless Charging
* Fingerprint
* Sensors: light, proximity, gyroscope, acceloremeter
* Sensors: magnetometer, compass, step counter, pickup
* Vibrator
* SD card
* NFC

## Flashing

Before flashing, please check the current [issues](https://github.com/sailfishos-sony-tama/main/issues) and 
pay attention to the [critical ones](https://github.com/sailfishos-sony-tama/main/issues?q=is%3Aopen+is%3Aissue+label%3Acritical). The critical issues may damage your hardware, so please
be aware of them.

Flashing guide is at [flashing.md](flashing.md).

## Tips

### Predictive text support

For predictive text support, install Presage-based predictive keyboards. These keyboards are available 
at OpenRepos, under [sailfish_keyboard applications](https://openrepos.net/user/12231/programs). After enabling OpenRepos,
you will need to enable that repository and install the keyboard layout(s) on your device. All the dependencies will be pulled 
during installation. For example, for English, install 
[English US Keyboard layout](https://openrepos.net/content/sailfishkeyboard/english-us-keyboard-layout-presage-based-text-prediction). 

### Fingerprint support

Fingerprint is supported through community effort
[sailfish-fpd-community](https://github.com/piggz/sailfish-fpd-community). For
installation, run in terminal

```
devel-su zypper ref
devel-su zypper up
devel-su zypper in sailfish-fpd-community-test sailfish-fpd-community sailfish-devicelock-fpd
```

The last command will ask for your help on how to proceed:
```
Problem: sailfish-devicelock-fpd-1.0.8-1.5.6.jolla.armv7hl conflicts with jolla-devicelock-plugin-encsfa provided by jolla-devicelock-plugin-encsfa-0.2.37-1.9.6.jolla.armv7hl
 Solution 1: Following actions will be done:
  deinstallation of jolla-devicelock-plugin-encsfa-0.2.37-1.9.6.jolla.armv7hl
  deinstallation of jolla-devicelock-daemon-encsfa-0.2.37-1.9.6.jolla.armv7hl
 Solution 2: do not install sailfish-devicelock-fpd-1.0.8-1.5.6.jolla.armv7hl

Choose from above solutions by number or cancel [1/2/c] (c):
```

Choose to uninstall `jolla-devicelock-plugin-encsfa` as it is replaced
by `sailfish-devicelock-fpd`.

On XZ3, please also install
```
devel-su zypper in droid-fake-crypt
```
This is needed on XZ3 only, do not install on XZ2 or XZ2c.

After installation of the fingerprint packages, reboot.

Fingerprints can be used for authentication if you enable in Settings,
under "Device lock", use of security code.

When adding fingerprints, it is recommended to use separate
application installed above: "Fingerprints". If you add fingerprints
under Sailfish Settings, it may lead to device reboot
([issue](https://github.com/piggz/sailfish-fpd-community/issues/10)). In
the dedicated application, such issue was not encountered.


### H.265 decoding

H.265 hardware accelerated decoding should work. To enable it when
coming from older versions, run

```
rm ~/.cache/gstreamer-1.0/registry.*
```

### Tracker and SD Cards

To allow tracker to index files on SD Card, run

```
gsettings set org.freedesktop.Tracker.Miner.Files index-recursive-directories "['&DESKTOP', '&DOCUMENTS', '&DOWNLOAD', '&MUSIC', '&PICTURES', '&VIDEOS',  '/run/media/nemo']"
```

### Step counter

Step counter is enabled and requires user space programs. Currently,
the best available interaction is through
[stpcntrd](https://github.com/kimmoli/stpcntrd) which registers DBus
session interface. You can use "Visual D-Bus" app to navigate to
`com.kimmoli.stpcntrd` and through that app call the methods. To
install the daemon, use
```
pkcon install stpcntrd
```
The daemon is included in Xperia Tama repositories.

### Pickup gesture

Pickup gesture switches on the screen when you pickup the phone and
allows you to avoid pressing a power button. It may require relatively
faster movement, try with the different movement patterns if it does
not work. Note that sometimes the gesture is not registered, try to
switch on / off the screen to re-arm the sensor. In addition, there is
some delay between pickup and switching on the screen, but that seems
to be originating mostly from the sensor or sensor-sensorfwd
interaction.

Pickup gesture requires development branch of
[sensorfwd](https://git.sailfishos.org/rinigus/sensorfw/tree/pickup)
and [pickupd](https://github.com/sailfishos-sony-tama/pickupd). The
both are available in Xperia Tama repositories.

To enable pickup gesture, you need to just install `pickupd`:
```
pkcon install pickupd
```
To disable, uninstall the daemon. 

### Pressure

Pressure sensor is configured. To my knowledge, the only application
interfacing it is Messwerk from
https://build.merproject.org/project/show/home:mal:apps .

### Launcher icons appear to small

A workaround around is to install [launcher-combined-patch](https://coderus.openrepos.net/pm2/project/launcher-combined) with [patchmanager](https://openrepos.net/content/coderus/patchmanager-30) and increase the icon size.

First install either patchmanager by hand or via [Storeman](https://openrepos.net/content/osetr/storeman).

Then install the patch by opening the settings app -> patchmanger -> pull-menu -> web-catalogue.

After this go a page back, select launcher-combined and enable change icon size. 

## Development

Port is developed under https://github.com/sailfishos-sony-tama with the local and OBS builds supported. For setting
up local build, see [HADK](hadk-sony-xz2.md).
