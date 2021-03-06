---
date: '2020-01-07T16:07:30.187Z'
docname: setup/install_switch_image
images: {}
path: /setup-install-switch-image
title: Install BISDN Linux
nav_order: 4
---

# Install BISDN Linux

Installing BISDN Linux on a whitebox switch can be done via the ONIE installer. This section shows how to connect to the switch and guides through the installation process.

## Connect to the switch console

Connect the CONSOLE port of the switch to a computer of your choice. We recommend using the tool kermit to connect to the switch console. If a Linux machine is used, install ckermit, edit a file named .kermrc and put in your configuration with the used line port.

Example of .kermrc:

```
set line /dev/ttyUSB0
set speed 115200
set carrier-watch off
set flow-control xon/xoff
connect
```

## Setup ONIE 

Check the current ONIE version of your switch by executing the following command in e.g. the ONIE-rescue shell:

```
onie-sysinfo -v
```

If your switch has the supported ONIE preinstalled you can skip this part and [install BISDN Linux right away](install_switch_image.md#install-bisdn-linux-via-onie) (see next section). In the other case you can either install a complete ONIE or upgrade an existing one.

### Supported ONIE versions

Only the following ONIE versions are tested and supported. Installation on other version may not work as expected.

| Device                 | Bootloader | ONIE version    |
|------------------------|------------|-----------------|
| Delta AG5648           | GRUB       |[V1.00](https://github.com/DeltaProducts/ag5648/tree/master/onie_image/) |
| Delta AG7648           | GRUB       |[2017.08.01-V1.12](https://github.com/DeltaProducts/AG7648/tree/master/onie_image/) (Build date 20181109) |
| Edgecore AS4610-30T/P  | U-Boot     |[2016.05.00.04](https://support.edge-core.com/hc/en-us/articles/360035081033-AS4610-ONIE-v2016-05-00-04)<sup>1</sup> |
| Edgecore AS4610-54T/P  | U-Boot     |[2016.05.00.04](https://support.edge-core.com/hc/en-us/articles/360033232494-AS4610-ONIE-v2016-05-00-04)<sup>1</sup> |

<sup>1</sup> Edgecore support account required

### Install ONIE

Prepare a bootable USB device and copy the proper ONIE image to it. One way is to download the .iso file given by the links above. Copy the file to the USB device like in the example below.


This example copies the .iso of the ONIE installer for AG7648 to the USB device on sdb:
```
sudo dd if=20181109-onie-recovery-x86_64-delta_ag7648-r0.iso of=/dev/sdb bs=10M
sync
```

Attach the USB device to your switch and reboot it. Enter the ONIE boot menu then press `c' to get into the grub CLI. Enter the following commands to boot from a USB device.

```
set root=(hd1)
chainloader +1
boot
```

Then select `ONIE: Embed ONIE` and the switch is going to install ONIE from the USB device.

### Update ONIE

Reboot the switch.

On switches using GRUB bootloader:

Enter the ONIE boot menu then select `ONIE: Rescue` to get into the ONIE CLI.

On switches using U-Boot bootloader:

Interrupt the U-Boot boot countdown by pressing any key and enter

```
run onie_rescue
```

to get into the ONIE CLI.

Download the .bin file given by the links above and put it onto an http server that is reachable by the switch. Start the update via the CLI command `onie-self-update` as shown in the example below.

```
onie-self-update -v http://local-http-server/onie-updater
```

**Note**: The ONIE CLI command can only process http URLs`
{: .label .label-yellow }

More information about the ONIE CLI command can be found [here](https://opencomputeproject.github.io/onie/cli/index.html#onie-self-update).

## Install BISDN Linux via ONIE

The recommended switch image installation is done via ONIE, a tool that allows installation of Network Operating Systems on bare metal servers. This will prevent issues due to the bootloader difference between x86 and ARM platforms, where GRUB and coreboot as used, respectively.

On switches using GRUB bootloader:

Select `ONIE: Install OS` in the ONIE menu to install a switch image. To remove the image select `ONIE: Uninstall OS`.

On switches using U-Boot bootloader:

Interrupt the U-Boot boot countdown by pressing any key and enter

```
run onie_install
```

to install a switch image. To remove the image, enter
```
run onie_uninstall
```

**Note**: It is recommended to uninstall any existing OS before installing BISDN Linux.
{: .label .label-yellow }

### Get the image via the CLI

On switches using GRUB bootloader:

Enter the ONIE boot menu then select `ONIE: Rescue` to get into the ONIE CLI.

On switches using U-Boot bootloader:

Interrupt the U-Boot boot countdown by pressing any key and enter

```
run onie_rescue
```

to get into the ONIE CLI.

Install the image via a CLI command as in the example below. All images are hosted in our [image repo](http://repo.bisdn.de/) while released images can be directly installed from [here](http://repo.bisdn.de/pub/onie/).

This example installs BISDN Linux v2.0.0 for the AG7648 platform:
```
onie-nos-install http://repo.bisdn.de.s3-eu-central-1.amazonaws.com/pub/onie/agema-ag7648/onie-bisdn-agema-ag7648-v2.0.0.bin
```

**Note**: The ONIE CLI command can only process http URLs.`
{: .label .label-yellow }

More information about the ONIE CLI command can be found [here](https://opencomputeproject.github.io/onie/cli/index.html#onie-nos-install).

### Get the image via DHCP option 60

Connect the management port to a DHCP server of your choice. The DHCP server uses “Vendor Class Identifier – Option 60” to tell the switch the URL of the image.

Example of dnsmasq configuration:

```
dhcp-vendorclass=set:ag7648,"onie_vendor:x86_64-ag7648-r0"
dhcp-option=tag:ag7648,114,"http://example_webserver.com/onie/onie-bisdn-agema-ag7648.bin"
```

In the example “example_webserver.com” is the server that must host the BISDN Linux image, the location of the actual file is then managed by the webserver (out of scope here). Any switch of type ag7648 will be given the link and is then able to fetch the listed image.

You should see a similar log on the system:

```
ONIE: Using DHCPv4 addr: eth0: 172.16.253.110 / 255.255.255.0
ONIE: Starting ONIE Service Discovery
Info: Fetching http://example_webserver.com/onie/onie-bisdn-agema-ag7648.bin ...
ONIE: Executing installer: http://example_webserver.com/onie/onie-bisdn-agema-ag7648.bin
Verifying image checksum ... OK.
Preparing image archive ... OK.
Demo Installer: platform: x86_64-agema_ag7648-r0
```

After successful installation the switch will reboot itself. Once it has finished booting you should see a similar message:

```
BISDN Linux 2.0.0 agema-ag7648 ttyUSB0

agema-ag7648 login:
```

Log into the switch with the credentials mentioned in the section [System Configuration](setup_standalone.md) . You should then see the console of BISDN Linux. See the OS information via `cat /etc/os-release `.

## Uninstall BISDN Linux

The script `onie-bisdn-uninstall` enables you to uninstall a running BISDN Linux. The corresponding man pages and usage help can be displayed like this:

```
man onie-bisdn-uninstall
onie-bisdn-uninstall -h
```

## Upgrade a running system

The script `onie-bisdn-upgrade` enables you to upgrade a running BISDN Linux to a newer image. The corresponding man pages and usage help can be displayed like this:

```
man onie-bisdn-upgrade
onie-bisdn-upgrade -h
```

This shows an example usage:

```
onie-bisdn-upgrade http://example_webserver.com/onie/onie-bisdn-agema-ag7648.bin
```

**Note**: All data is deleted during the upgrade process except for configuration files in /etc/systemd/network/ that apply on interfaces named `enp*`.
{: .label .label-yellow }

## Boot into ONIE-rescue

The script `onie-bisdn-rescue` enables you to boot into ONIE-rescue mode from the BISDN Linux shell. The corresponding man pages and usage help can be displayed like this:

```
man onie-bisdn-rescue
onie-bisdn-rescue -h
```

The ONIE-rescue shell provides troubleshooting. More information can be found here: [ONIE-rescue mode](https://opencomputeproject.github.io/onie/design-spec/nos_interface.html#rescue-and-recovery). 
