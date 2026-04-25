---
layout: page
title: CDIApp — Crop Disease Detection
description: A CNN-based crop-disease classifier paired with an LLM-powered remedy recommender. Shortlisted for Smart India Hackathon 2022 finals.
img: assets/img/1.jpg
importance: 1
category: shipped
---

**CDIApp** is a mobile-friendly tool that lets a farmer photograph a diseased leaf and receive (a) an identification of the disease and (b) a suggested remedy. It was built for **Smart India Hackathon 2022** and shortlisted for the national finals.

### What it does
- Classifies crop leaf images into disease categories with **~95% accuracy** on the evaluation split.
- Queries the **Google Gemini API** to generate region-appropriate remedy suggestions for the detected disease, grounded in the model's label and confidence.
- Mobile-first UX so the tool is usable in low-bandwidth rural environments.

### How it works
- **Model:** A convolutional network trained on public plant-disease datasets, fine-tuned with class-balanced sampling because disease frequencies are long-tailed in real fields.
- **Inference:** Exported to a lightweight format for on-device or edge inference.
- **Remedy generation:** Prompt template conditions Gemini on the predicted disease, crop type, and geographic region to produce actionable, locally-feasible suggestions rather than generic pesticide advice.

### What I learned
- Class imbalance dominates model behaviour for long-tailed agricultural datasets — resampling and focal loss mattered far more than architecture tweaks.
- Pairing a narrow, accurate classifier with a general-purpose LLM is a pragmatic pattern: the CNN handles the part that needs precision, the LLM handles the part that needs breadth.

[Repository on GitHub →](https://github.com/Dutta-SD)
