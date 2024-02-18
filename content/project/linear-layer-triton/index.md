---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Optimized NN Inference using custom Triton kernels"
summary: ""
authors: [Shreyansh Singh]
tags: [triton, mlsys, optimization]
categories: [Machine Learning Systems]
date: 2024-02-18T19:32:59+05:30

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

url_code: "https://github.com/shreyansh26/MLSys-Experiments/tree/main/linear_layer_triton"
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

Implemented a high-performance linear layer (both forward and backward pass) with (optional) activation layer fusion using OpenAI’s Triton.  
* The use of the custom Triton-based linear layer demonstrated up to 1.6x speedup in training FlanT5-Base on the Samsum dataset and up to 3.5x speedup in inference.  
* Automated the patching of PyTorch's nn.LinearLayer and associated activation layers to the new custom layers for inference using torch.fx for pattern matching and CUDA Graphs for reducing overheads.