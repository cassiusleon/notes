---
title: Hardware
project: "[[Home AI Assistant]]"
chapter: 3
date: 2026-10-02
tags:
  - home-ai
  - hardware
aliases:
  - Beelink SER9 MAX
---

# 03 · Hardware

## The pick: Beelink SER9 MAX
- Ryzen 7 H255 (8C/16T), Radeon 780M iGPU, 10GbE, 2× M.2, removable SO-DIMM RAM
- **32 GB RAM** (decided; 64 GB was the earlier plan)
- Earlier candidate: SER8 (8845HS), same chip class with 1G LAN
- Shortlist: GMKtec K12, Beelink SER8, Minisforum UM890 Pro, GEEKOM A8 Max, SER9 MAX

## RAM size vs speed
> [!important] Size decides *what fits*. Bandwidth decides *how fast*.
> **tok/s ≈ memory bandwidth ÷ model size**
> All shortlisted boxes: DDR5-5600 dual channel = **89.6 GB/s**. 8B Q4 (~5 GB) → ~18 tok/s

| On 32 GB | GB |
|---|---|
| 14B model (Q4) | ~9 |
| Context for long emails/PDFs | 2–4 |
| Ubuntu + Home Assistant + vector DB | 4–6 |
| **Spare** | **13–17** |

- 32 GB fits a 14B dense model or a 26–30B MoE (~16–18 GB)
- 64 GB only adds room for a second model or longer documents, **not speed**
- The GPU (780M) decides how fast long inputs are *read*: a 20-page PDF takes tens of seconds

## Must-have specs
- [ ] RAM as **2×16 GB** (one stick = single channel = half speed)
- [ ] **SO-DIMM**, not soldered (⚠️ the SER9 **Pro** is soldered)
- [ ] Radeon 780M or better (avoid Intel N100/N150)
- [ ] 2× M.2: OS drive + data drive (keeps data portable)
- [ ] Wired Ethernet with a Linux-friendly NIC (Wi-Fi off)
- [ ] BIOS "Restore on AC power loss" + small UPS (power cuts)
- [ ] Spare USB 2.0 port for a Zigbee/Thread dongle
- [ ] HDMI for the lyrics screen

## Operating system
- Ships with **Windows 11 Pro**. Plan: **Ubuntu Server, headless**
- Install: Ventoy/Rufus USB → **F7** boot menu (**Del** for BIOS)
- Use a 2026 distro: the 10GbE (Realtek RTL8127) needs kernel 6.16+, or Realtek's driver
- Barebones (no Windows) units are sold in some regions

## Gotchas
- No NPU despite the "AI PC" marketing; models run on the CPU/iGPU
- Linux lets the iGPU use ~½ of RAM by default (~16 GB). 14B fits; an MoE needs a one-line boot setting
- "Up to 256 GB" is theoretical; practical max is ~128 GB
- Buying on amazon.com from India: check import duty, and that warranty means shipping it back to Beelink

## If more speed is needed later
- An external GPU over **OCuLink** (GMKtec K12 and Minisforum UM890 Pro have the port)
- Strix Halo (Ryzen AI Max+ 395) boxes: 256 GB/s, ~3× faster

## Test bench now
- Dell Precision 3571 (i7-12800H, 32 GB DDR5, RTX A1000 4 GB). See [[04 - Local LLM Test Run]]
- Chrome background apps were the main RAM hog. If 14B won't load, lower the context window

---
← [[02 - Architecture]] | [[Home AI Assistant]] | [[04 - Local LLM Test Run]] →
