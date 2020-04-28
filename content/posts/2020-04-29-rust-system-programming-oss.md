---
title: "Rustで書かれた低レイヤOSS"
slug: "rust-system-programming-oss"
subtitle:    ""
description: ""
date:        "2020-04-29"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["Rust"]
categories:  ["Tech" ]
draft:       false
---

Rustで書かれたOS、VMM、Bootloaderについて調べた。

# OS

- [Tock](https://github.com/tock/tock)
  - ARM Cortex-M、、STM32、RISC-Vで動く組み込みOS
  - 論文も出てる
    - [Multiprogramming a 64kB Computer Safely and Efficiently](https://sing.stanford.edu/site/publications/levy17-tock.pdf)
- [Redox](https://www.redox-os.org/jp/)
  - x86-64で動くUNIX系のマイクロカーネルOS
  - エディタ、GUI、ビルドツールもRustで書かれている
- [intermezzOS](https://intermezzos.github.io/)
  - 教育用OS

# VMM

- [crosvm](https://chromium.googlesource.com/chromiumos/platform/crosvm/)
  - Chromium OSに搭載されているLinux sandbox VM
- [Firecracker](https://github.com/firecracker-microvm/firecracker)
  - KVMを使用してmicroVMを起動できる
  - crosvmをベースに開発した

# Bootloader

- [oreboot](https://github.com/oreboot/oreboot)
  - corebootのRust移植
