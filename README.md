# DKMS bnx2x with 2.5G patches

Based on the Debian 5.10.0-9 kernel package (Linux kernel 5.10.70), and the work from JAMESMTL: \
https://www.dslreports.com/forum/r32440802-
https://raw.githubusercontent.com/JAMESMTL/snippets/master/bnx2x/patches/bnx2x_warpcore+8727_2_5g_sgmii.patch

## Tested on
Debian 12 and Debian 13 with several 6.1.* and 6.12.* kernels.

## Other versions

For other versions please check
* https://github.com/severnt/bnx2x-2_5g-dkms/branches
* https://github.com/darkbasic/bnx2x-2_5g-dkms
* https://github.com/jaanusjot/bnx2x-2_5g-dkms

## Additional Changes
Added a module option that can be set to disable SFP TX fault detection:

`disable_sfp_tx_fault_detection`

To use, add a file to /etc/modprobe.d with the contents:

`options bnx2x disable_sfp_tx_fault_detection=1`

## Why DKMS
This driver doesn't seem to change too often. Instead of grabbing the full source and rebuilding, this will automatically rebuild the driver every time the kernel is updated.

## How to use DKMS
```sh
sudo apt-get update
sudo apt-get install dkms
cd /usr/src
sudo git clone -b debian13-6.12 https://github.com/jaanusjot/bnx2x-2_5g-dkms.git bnx2x-99.1.713.36-1
sudo dkms add bnx2x/99.1.713.36-1
sudo dkms install bnx2x/99.1.713.36-1 -k $(uname -r) --force
sudo update-initramfs -u
```

It used to rebuild every time you update your kernel, but because DKMS on Debian 12 changed how it does version checks, \
you'll have to re-install this module each time you update your kernel.

```sh
sudo dkms install bnx2x/99.1.713.36-1 -k $(uname -r) --force
```

If you just installed a new kernel then you need to manually speficy its version for dkms.\
For example, if you installed 6.12.43+deb13-amd64 then you need to run:
```shell
sudo dkms install bnx2x/99.1.713.36-0 -k 6.12.43+deb13-amd64 --force
sudo update-initramfs -u -k 6.12.43+deb13-amd64
```

Also, you need to make sure that bnx2x module is listed in /etc/initramfs-tools/modules. \
Without it update-initramfs might not include updated kernel module in initramfs.

```ini
root@server:~# cat  /etc/initramfs-tools/modules

# List of modules that you want to include in your initramfs.
# They will be loaded at boot time in the order below.
#
# Syntax:  module_name [args ...]
#
# You must run update-initramfs(8) to effect this change.
#
# Examples:
#
# raid1
# sd_mod

bnx2x
```

Regenerate initramfs with update-initramfs -u every time you install a new kernel \
You can run update for all kernels with:
```sh
sudo vupdate-initramfs -u -k all
```

If you do not want to wait for the network interface to come online during boot for some reason, \
then you can disable that long wait by modifying a related .network configuration file in \
/etc/systemd/network/ and add:
```ini
[Link]
RequiredForOnline=no
```
There is also another way to edit systemd service systemd-networkd-wait-online and specify exactly what interface to wait for. \
For example, to wait for lan0 interface only add the following to the service override:
```shell
sudo systemctl edit systemd-networkd-wait-online.service
```
```ini
[Service]
ExecStart=
ExecStart=/usr/lib/systemd/systemd-networkd-wait-online --interface=lan0
```

To check for result you can use
```shell
sudo systemctl cat systemd-networkd-wait-online.service
```
To revert the change
```shell
sudo systemctl revert systemd-networkd-wait-online.service
```