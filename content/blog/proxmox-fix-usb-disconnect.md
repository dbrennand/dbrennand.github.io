---
title: "Fixing Proxmox USB Disconnects"
date: 2024-02-15T20:33:49Z
draft: false
tags: ["Proxmox", "USB", "Disconnects"]
showToc: true
---

Towards the end of last year, I introduced [Proxmox](https://www.proxmox.com/en/) into my Homelab. I'm using it to host various services on VMs and LXC containers. In my setup, I have two external SSDs providing storage, these are connected to my Proxmox node via USB 3.0. However, recently I was experiencing issues with the SSDs suddenly disconnecting from the Proxmox node!

This was very frustrating because the VM filesystems that were on these SSDs were becoming corrupted! :scream:
This issue was happening on Proxmox `8.1.3` and kernel version `6.5.11-7-pve`.

## Troubleshooting

On Proxmox, you can check the system logs via the GUI by navigating to the node, and under `System` is `syslog`. It's also possible via the `journalctl` command on the Proxmox node itself.

Upon inspecting the Proxmox node logs, I couldn't see any indication as to why the disconnects were happening. I could see the disconnect event occurring but prior to that, everything was normal. Here's an example of the kernel logs:

```
Feb 09 22:11:04 proxmox01 kernel: usb 2-1: USB disconnect, device number 17
Feb 09 22:11:04 proxmox01 kernel: sd 4:0:0:0: [sdb] Synchronizing SCSI cache
Feb 09 22:11:04 proxmox01 kernel: sd 4:0:0:0: [sdb] Synchronize Cache(10) failed: Result: hostbyte=DID_ERROR driverbyte=DRIVER_OK
```

I checked the SMART status of the SSDs and they were healthy, I tried different USB ports on the Proxmox node, swapping the USB 3.0 to SATA cables, nothing seemed to help :confused:

## The Fix

After some research, I found that the issue may have been related to USB power saving settings. After a certain period of time, the OS places USB devices in a low-power state to save power, which was causing the USB drives to disconnect! :exploding_head:

To resolve the problem, with some help from our good friend ChatGPT, I permanently disabled the USB power saving settings by adding `usbcore.autosuspend=-1` to the GRUB kernel boot options in `/etc/default/grub`:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet usbcore.autosuspend=-1"
```

Then, applying the changes and rebooting the Proxmox node:

```bash
update-grub
reboot
```

After making these changes, I haven't experienced any further disconnects! :tada: 🥳
