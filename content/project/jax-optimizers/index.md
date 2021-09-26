---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "ML Optimizers in JAX"
summary: "Implementations of some popular optimizers from scratch for a simple model i.e., Linear Regression on a dataset of 5 features. The goal of this project was to understand how these optimizers work under the hood and try to do a toy implementation myself. I also use a bit of JAX magic to perform the differentiation of the loss function w.r.t to the weights and the bias without explicitly writing their derivatives as a separate function. This can help to generalize this notebook for other types of loss functions as well."
authors: [Shreyansh Singh]
tags: [jax, optimizers, machine learning, linear regression]
categories: [Machine Learning]
date: 2021-09-26T19:19:28+05:30

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

url_code: "https://github.com/shreyansh26/ML-Optimizers-JAX"
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

Implementations of some popular optimizers from scratch for a simple model i.e., Linear Regression on a dataset of 5 features. The goal of this project was to understand how these optimizers work under the hood and try to do a toy implementation myself. I also use a bit of JAX magic to perform the differentiation of the loss function w.r.t to the weights and the bias without explicitly writing their derivatives as a separate function. This can help to generalize this notebook for other types of loss functions as well.

The optimizers I have implemented are -

* Batch Gradient Descent
* Batch Gradient Descent + Momentum
* Nesterov Accelerated Momentum
* Adagrad
* RMSprop
* Adam
* Adamax
* Nadam
* Adabelief