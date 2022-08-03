---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "ConvNeXt - Adversarial images generation"
summary: "I implemented [Stanislav Fort's project](https://twitter.com/stanislavfort/status/1481263565998805002?s=20) in Pytorch. The Github repo has a notebook which looks at generating adversarial images to 'fool' the ConvNeXt model's image classification capabilities. ConvNeXt came out earlier this year (2022) from Meta AI. The FGSM (Fast Gradient Sign Method) is a great algorithm to attack models in a white-box fashion with the goal of misclassification. Noise is added to the input image (not randomly) but in a manner such that the direction is the same as the gradient of the cost function with respect to the data."
authors: [Shreyansh Singh]
tags: [adversarial machine learning, machine learning, transformer, computer vision, cnn]
categories: [Machine Learning]
date: 2022-03-06T20:40:05+05:30

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

url_code: "https://github.com/shreyansh26/ConvNeXt-Adversarial-Examples"
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

I implemented [Stanislav Fort's project](https://twitter.com/stanislavfort/status/1481263565998805002?s=20) in Pytorch. The Github repo has a notebook which looks at generating adversarial images to "fool" the ConvNeXt model's image classification capabilities. ConvNeXt came out earlier this year (2022) from Meta AI.

The FGSM (Fast Gradient Sign Method) is a great algorithm to attack models in a white-box fashion with the goal of misclassification. Noise is added to the input image (not randomly) but in a manner such that the direction is the same as the gradient of the cost function with respect to the data.