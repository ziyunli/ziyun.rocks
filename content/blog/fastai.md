---
title: "Practical Deep Learning"
summary: "Practical Deep Learning for Coders"
tags: [Deep Learning, Notes, Book, Course]
draft: true
---

# 2. Deployment

Before you clean the data, you train the model. Why?

> To find out what things are difficult to recognize in your data, and to find the things that the model can help you find data problems.

Look at
- confusion matrix
- top losses `plot_top_losses`
  - images that the model is most confident about but wrong, or
  - images that the model is least confident about but right

Then use `ImageClassifierCleaner` to clean the data.

Terms:
- Data augmentation:
