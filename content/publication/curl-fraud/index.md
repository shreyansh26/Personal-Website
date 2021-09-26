---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "CuRL: Coupled Representation Learning of Cards and Merchants to Detect Transaction Frauds"
authors: [Shreyansh Singh, Maitrey Gramopadhye, Kushagra Agarwal, Nitish Srivasatava, Alok Singh, Siddhartha Asthana and Ankur
Arora]
date: 2021-09-26T19:34:07+05:30
doi: "10.1007/978-3-030-86383-8_2"

# Schedule page publish date (NOT publication's date).
publishDate: 2021-09-26T19:34:07+05:30

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["1"]

# Publication name and optional abbreviated publication name.
publication: "30th International Conference on Artificial Neural Networks (ICANN), 2021"
publication_short: "ICANN 2021"

abstract: "Payment networks like Mastercard or Visa process billions of transactions every year. A significant number of these transactions are fraudulent that cause huge losses to financial institutions. Conventional fraud detection methods fail to capture higher-order interactions between payment entities i.e., cards and merchants, which could be crucial to detect out-of-pattern, possibly fraudulent transactions. Several works have focused on capturing these interactions by representing the transaction data either as a bipartite graph or homogeneous graph projections of the payment entities. In a homogeneous graph, higher-order cross-interactions between the entities are lost and hence the representations learned are sub-optimal. In a bipartite graph, the sequences generated through random walk are stochastic, computationally expensive to generate, and sometimes drift away to include uncorrelated nodes. Moreover, scaling graph-learning algorithms and using them for real-time fraud scoring is an open challenge.

In this paper, we propose CuRL and tCuRL, coupled representation learning methods that can effectively capture the higher-order interactions in a bipartite graph of payment entities. Instead of relying on random walks, proposed methods generate coupled session-based interaction pairs of entities which are then fed as input to the skip-gram model to learn entity representations. The model learns the representations for both entities simultaneously and in the same embedding space, which helps to capture their cross-interactions effectively. Furthermore, considering the session constrained neighborhood structure of an entity makes the pair generation process efficient. This paper demonstrates that the proposed methods run faster than many state-of-the-art representation learning algorithms and produce embeddings that outperform other relevant baselines on fraud classification task."

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

url_pdf: https://link.springer.com/chapter/10.1007/978-3-030-86383-8_2
url_code:
url_dataset:
url_poster:
url_project:
url_slides: https://docs.google.com/presentation/d/1ne27Zgrr-nzruX5Rvg79IS6J-4eXNVie/edit?usp=sharing&ouid=118016587896212855658&rtpof=true&sd=true
url_source:
url_video: https://drive.google.com/file/d/1lr5ph6hkKZdWFgDcCeKGAzZdxt8chX7m/view?usp=sharing

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
