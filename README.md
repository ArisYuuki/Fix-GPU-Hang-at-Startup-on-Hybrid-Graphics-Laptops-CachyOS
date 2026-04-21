# FIX-CACHYOS-HYBRID-GPU-BOOT
![Arch Linux](https://img.shields.io/badge/Arch-Linux-blue?logo=arch-linux)
![CachyOS](https://img.shields.io/badge/CachyOS-Optimized-brightgreen)
![NVIDIA](https://img.shields.io/badge/GPU-NVIDIA-76B900?logo=nvidia)
![Intel](https://img.shields.io/badge/GPU-Intel-blue)

Fix random boot hangs on CachyOS systems using Intel + NVIDIA hybrid graphics.

This repository provides a simple and reliable fix for hybrid laptops where the system may hang during boot due to improper GPU initialization order. By forcing the Intel iGPU to load first in initramfs, you ensure consistent and stable boot behavior.

# Hybrid GPU Boot Fix

## The Problem

On Intel + NVIDIA hybrid laptops, CachyOS may randomly fail to boot properly.

Common issues include:

- Black screen after LUKS decryption
- System hang before SDDM / login manager
- Boot works inconsistently
- Requires hard reset

---

## The Cause

CachyOS does not always initialize the correct GPU during early boot.

On hybrid systems:

- Intel iGPU (`i915`) should initialize first
- NVIDIA may load first instead

This creates a race condition that breaks display initialization and causes boot failure.

---

## The Fix

Force the Intel iGPU to load first by modifying the mkinitcpio `MODULES` order.

---

## Edit mkinitcpio Configuration

```bash
sudo nano /etc/mkinitcpio.conf
```
---

## Find the following line in your config

```bash
MODULES=()
```
---

## Replace it with

```bash
MODULES=(i915 nvidia nvidia_modeset nvidia_uvm nvidia_drm crc32c)
```
---

## Important

i915 must be first

This ensures Intel initializes before NVIDIA

Prevents GPU initialization conflicts during boot

---

## Rebuild Initramfs

```bash
sudo mkinitcpio -P
```
---
## Reboot

```bash
sudo reboot
```
---

## Result

After applying this fix:

No more boot hangs

No black screen after LUKS

SDDM loads consistently

Reliable boot every time

## Why This Works

Hybrid laptops rely on the Intel iGPU for display output.

Correct initialization order:

```bash
i915 → NVIDIA modules
```
---

## This ensures:

Intel initializes the display first

NVIDIA loads afterward without conflict

Boot completes successfully

## Compatibility
Intel + NVIDIA hybrid laptops (Optimus)

CachyOS (all kernels)

nvidia and nvidia-dkms

Wayland and X11

Works with nvidia_drm.modeset=1

## Troubleshooting

If issues persist, check logs:

```bash
journalctl -b -1 | grep -Ei "nvidia|drm|gpu"

dmesg | grep -Ei "nvidia|drm|gpu"
```
---

## If All Else Fails (Specifiic to Nvidia Hybrid Laptops)

If none of the above has helped and you are still experiencing issues with booting to SDDM/PLM or see issues related to login, try switching from SDDM to LightDM. 

## Install LightDM
```bash
sudo pacman -S lightdm lightdm-gtk-greeter
```
---

## Stop and Disable SDDM
```bash
sudo systemctl stop sddm
sudo systemctl disable sddm
```
---

## Remove SDDM from Display Manager Priority
```bash
sudo rm -f /etc/systemd/system/display-manager.service
```
---

## Enable and Start LightDM
```bash
sudo systemctl reset-failed lightdm
sudo systemctl enable lightdm
sudo systemctl start lightdm
```
---

## Check the Status of LightDM and Display Manager
Make sure that LightDM is working and is hooked to Display Manager BEFORE restarting as it will cause your laptop to not boot to GUI
```bash
systemctl status lightdm --no-pager
systemctl status display-manager --no-pager
```
---

## Reboot
```bash
sudo reboot
```
---

## Why LightDM?
LightDM is known to have superior compatability when running an Nvidia Hybrid Laptop as Nvidia+KDE+Wayland can have conflict with eachother.
LightDM by default runs in x11 making it a very safe choice for Nvidia graphics in general over SDDM or PLM which use Wayland.
