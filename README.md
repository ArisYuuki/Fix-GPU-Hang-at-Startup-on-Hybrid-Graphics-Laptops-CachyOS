# FIX-CACHYOS-HYBRID-GPU-BOOT
![Arch Linux](https://img.shields.io/badge/Arch-Linux-blue?logo=arch-linux)
![CachyOS](https://img.shields.io/badge/CachyOS-Optimized-brightgreen)
![NVIDIA](https://img.shields.io/badge/GPU-NVIDIA-76B900?logo=nvidia)
![Intel](https://img.shields.io/badge/GPU-Intel-blue)

Fix random boot hangs on CachyOS systems using Intel + NVIDIA hybrid graphics.

This repository provides a simple fix for a common issue on hybrid laptops where the system may hang during boot due to improper GPU initialization order. By forcing the Intel iGPU to load first in initramfs, you ensure consistent and reliable boot behavior.

# Hybrid GPU Boot Fix

## The Problem

On Intel + NVIDIA hybrid laptops, CachyOS may randomly fail to boot properly.

Common issues include:

- Black screen after LUKS decryption  
- System hang before SDDM / login manager  
- Boot works inconsistently  
- Requires hard reset  

## The Cause

CachyOS does not always initialize the correct GPU first during early boot.

On hybrid systems:

- Intel iGPU (`i915`) should initialize first  
- NVIDIA may sometimes load first instead  

This creates a race condition that breaks display initialization.

## The Fix

Force the Intel iGPU to load first by modifying the mkinitcpio MODULES order.

---

## Edit mkinitcpio Configuration

```bash
sudo nano /etc/mkinitcpio.conf
