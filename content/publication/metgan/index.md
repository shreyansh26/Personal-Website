---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "MeTGAN: Memory efficient Tabular GAN for high cardinality categorical datasets"
authors: [Shreyansh Singh, Kanishka Kayathwal, Hardik Wadhwa, and Gaurav Dhama]
date: 2021-09-26T19:52:05+05:30
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: 2021-09-26T19:52:05+05:30

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["1"]

# Publication name and optional abbreviated publication name.
publication: "28th International Conference on Neural Information Processing (ICONIP), 2021"
publication_short: "ICONIP 2021"

abstract: "Generative Adversarial Networks (GANs) have seen their use
for generating synthetic data expand, from unstructured data like images
to structured tabular data. One of the recently proposed models in the
field of tabular data generation, CTGAN, demonstrated state-of-the-art
performance on this task even in the presence of a high class imbalance
in categorical columns or multiple modes in continuous columns. Many
of the recently proposed methods have also derived ideas from CTGAN.
However, training CTGAN requires a high memory footprint while dealing
with high cardinality categorical columns in the dataset. In this paper, we
propose MeTGAN, a memory-efficient version of CTGAN, which reduces
memory usage by roughly 80%, with a minimal effect on performance.
MeTGAN uses sparse linear layers to overcome the memory bottlenecks
of CTGAN. We compare the performance of MeTGAN with the other
models on publicly available datasets. Quality of data generation, memory
requirements, and the privacy guarantees of the models are the metrics
considered in this study. The goal of this paper is also to draw the
attention of the research community on the issue of the computational
footprint of tabular data generation methods to enable them on larger
datasets especially ones with high cardinality categorical variables."

# Summary. An optional shortened abstract.
summary: ""

tags: []
categories: []
featured: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_pdf:
url_code:
url_dataset:
url_poster:
url_project:
url_slides:
url_source:
url_video:

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""
---
