---
title: "DualViT"
date: 2024-01-01
summary: "Official implementation of DualViT (ICPR 2024) — a hierarchical vision transformer that learns broad and fine class embeddings simultaneously via dual encoders and tensor product fusion."
tags: ["deep-learning", "vision-transformer", "python", "research"]
weight: 3
---

I co-authored DualViT during my time at IIT Kharagpur's VIP Lab. The core idea is that a single encoder has to collapse the full class hierarchy into one representation — coarse structure and fine-grained detail at the same time. DualViT splits this into two transformer encoders trained alternately: one learning broad class embeddings, one learning fine-grained ones. The two representations are fused via tensor products, encoding hierarchical knowledge explicitly rather than hoping the model learns it implicitly.

We evaluated it on CIFAR-10, CIFAR-100, and ImageNet-1k, achieving competitive accuracy with fewer training epochs than comparable approaches. The paper was presented at ICPR 2024.

This repo is the official PyTorch implementation.

**Paper:** [Springer](https://link.springer.com/chapter/10.1007/978-3-031-78456-9_14) · **Code:** [GitHub](https://github.com/Dutta-SD/DualViT)
