---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Flash Attention in Pytorch"
summary: ""
authors: [Shreyansh Singh]
tags: [attention, transformer, efficient-ml, deep learning]
categories: [Machine Learning]
date: 2023-06-19T20:10:00+05:30

# Optional external URL for project (replaces project detail page).
external_link: ""

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: "https://github.com/shreyansh26/FlashAttention-PyTorch"
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

A simplified implementation of FlashAttention in PyTorch. I have implemented the forward pass and backward pass algorithms from the paper, and also shown that it is equivalent to the normal attention formulation in Transformers. I also include some code for benchmarking.

Note that this is for educational purposes only as I haven't implemented any of the CUDA and SRAM memory tricks as described in the paper.