---
title: "FFLUNet"
date: 2025-01-01
summary: "Official implementation of FFLUNet (Computers in Biology and Medicine, 2025) — a lightweight U-Net for 3D brain tumor segmentation with multi-view feature fusion, 1.45M parameters."
tags: ["deep-learning", "medical-imaging", "python", "research"]
weight: 4
---

I co-authored FFLUNet while working on medical image segmentation at IIT Kharagpur's VIP Lab. The problem with standard U-Net variants is that they scale accuracy by scaling parameters — but in clinical settings you often need something that runs fast on modest hardware. FFLUNet gets competitive segmentation performance at 1.45M parameters by fusing multi-view features progressively across encoder-decoder paths rather than adding width.

We evaluated it on three benchmark datasets: BraTS 2020, BraTS Africa Glioma, and ACDC (cardiac MRI). It's implemented as a custom trainer on top of nnU-Net, so you get the full nnU-Net preprocessing and experiment pipeline with the FFLUNet backbone swapped in.

Published in *Computers in Biology and Medicine*, 2025. This repo is the official implementation.

**Paper:** [DOI](https://doi.org/10.1016/j.compbiomed.2025.110460) · **Code:** [GitHub](https://github.com/Dutta-SD/FFLUNet)
