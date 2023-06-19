---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "KV Cache in Nanogpt"
summary: ""
authors: []
tags: []
categories: []
date: 2023-06-17T20:18:33+05:30

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

url_code: "https://github.com/shreyansh26/nanoGPT-kvcache"
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

A modification of nanoGPT to use KV-Cache during inference. Using a KV Cache helps speed up inference since we don't have to do attention re-computation over the entire generated sequence every time to generate the next tokens.