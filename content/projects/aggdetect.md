---
title: "AggDetectApp"
date: 2021-12-01
summary: "A web app that detects aggression and misogyny in social media text using BERT augmentation, XGBoost, TF-IDF, and LIME for explainability. Paper accepted at ICON 2021."
tags: ["nlp", "bert", "python", "research"]
weight: 5
---

Social media platforms generate far more content than anyone can manually review for aggression and hate speech. I built AggDetectApp to make a fast, practical classifier for this problem — the goal was high throughput with a small model footprint, not squeezing out the last point of F1.

The pipeline uses BERT-augmented features combined with TF-IDF vectorisation and XGBoost. On the ICON 2021 shared task data it achieves F1 0.735 on aggression detection and 0.852 on misogyny detection. The explainability piece mattered to me: every prediction comes with LIME attribution scores showing which words drove the result, so the model's reasoning is auditable rather than opaque.

The app is live on Hugging Face Spaces. The accompanying paper was accepted at ICON 2021.

**Live:** [HuggingFace Spaces](https://huggingface.co/spaces/sdutta28/AggDetectApp) · **Paper:** [ACL Anthology](https://aclanthology.org/2021.icon-main.60) · **Code:** [GitHub](https://github.com/Dutta-SD/AggDetectApp)
