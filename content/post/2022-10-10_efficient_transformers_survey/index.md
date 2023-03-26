---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Summary #7 - Efficient Transformers: A Survey"
subtitle: "A survey paper of improvements over the original Transformer architecture in terms of memory-efficiency."
summary: ""
authors: ["Shreyansh Singh"]
tags: [transformers, efficient-ml, deep learning, paper-summaries]
categories: [Machine Learning]
date: 2022-10-10T14:57:33+05:30
lastmod: 2022-10-10T14:57:33+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: [Paper Notes]
---


**Paper**: Efficient Transformers: A Survey  
**Link**: [https://arxiv.org/abs/2009.06732](https://arxiv.org/abs/2009.06732)  
**Authors**: Yi Tay, Mostafa Dehghani, Dara Bahri, Donald Metzler

--------------------

I wanted to summarize this paper for a long time now because of the immense amount of information in this paper. Thanks to the [Cohere For AI](https://cohere.for.ai/) community for having a session on this paper which made me revisit this.

# What?

This is a survey paper on the various memory-efficiency based improvements on the original Transformers architecture by Vaswani et al. But wait, for those unaware, how is the Transformers architecture inefficient?

* The attention operation has a quadratic complexity over the sequence length L, also sometimes represented using N (since each token attends to other set of tokens in the sequence)
* The Attention operation of Q*K<sup>T</sup> uses N<sup>2</sup> time and memory. Here (in no-batching case) Q, K, V (query, key and value matrices) have dimensions <i>N x d </i> where d is the dimension of query, key and value vectors.

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/attention.PNG" caption="Attention calculation" >}}

# Glossary

* [Low-rank Methods](#low-rank-methods)
  * [Linformer](#linformer---httpsarxivorgabs200604768httpsarxivorgabs200604768)
  * [Perfomer](#performer---httpsarxivorgabs200914794httpsarxivorgabs200914794)
* [Learnable Patterns based methods](#learnable-patterns)
  * [Clustered Attention](#clustered-attention---httpsarxivorgabs200704825httpsarxivorgabs200704825--httpsclustered-transformersgithubiobloghttpsclustered-transformersgithubioblog)
  * [Reformer](#reformer---httpsarxivorgabs200104451httpsarxivorgabs200104451)
* [Memory-based methods](#memory-based)
  * [Big Bird](#big-bird-httpsarxivorgabs200714062httpsarxivorgabs200714062--httpshuggingfacecoblogbig-birdhttpshuggingfacecoblogbig-bird)
* [Complexity summary of various models](#complexity-summary-of-various-models)

# Memory-Efficient Transformers

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/arch-vaswani.png" caption="" >}}

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/summary.png" caption="" >}}

## Low-Rank methods

### Linformer - [https://arxiv.org/abs/2006.04768](https://arxiv.org/abs/2006.04768)

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/linformer.png" caption="Left and bottom-right show architecture and example of our proposed multihead linear self-attention. Top right shows inference time vs. sequence length for various Linformer models." >}}

In Linformer, the original Key and Value matrices are projected from <i>(N x d)</i> to a reduced <i>(k x d)</i>.

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/linformer-dets.png" caption="" >}}

The above operations only require `O(n*k)` time and space complexity. Thus, if we can choose a very small projected dimension k, such that k < < N, then we can significantly reduce the memory and space consumption.

### Performer - [https://arxiv.org/abs/2009.14794](https://arxiv.org/abs/2009.14794)

The goal in the Performer paper was to reduce the complexity of attention calculation (Q * K<sup>T</sup>) * V of O(L<sup>2</sup> * d) to O (L * d<sup>2</sup>) by transforming the order of operations and using a kernel operation to approximate the softmax operation so that the order of operations can be changed.

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/performer.png" caption="An overview from the paper" >}}

From, a [great blog on the Performer paper](https://chiaracampagnola.io/2020/10/29/from-transformers-to-performers/) - 

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/performer-dets.png" caption="Change of operation order" >}}

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/performer-dets2.png" caption="" >}}

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/performer-dets3.png" caption="" >}}

## Learnable Patterns

### Clustered Attention - [https://arxiv.org/abs/2007.04825](https://arxiv.org/abs/2007.04825) + [https://clustered-transformers.github.io/blog/](https://clustered-transformers.github.io/blog/)

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/clusteredatt.png" caption="" >}}

* First cluster the queries into  non-overlapping clusters.
* Attention weights A<sup>c</sup> are computed using the centroids instead of computing them for every query
* Use clustered attention weights A<sup>c</sup> to compute new Values V<sup>c</sup>
* Use the same attention weights and new values for queries that belong to same cluster.
* Computational complexity becomes <code>O(N * C * max (D<sub>k</sub> * D<sub>v</sub>))</code>

They also propose an Improved Clustered Attention in their blog. The complexity comaprisons are here - 

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/clusteredatt-dets.png" caption="" >}}


### Reformer - [https://arxiv.org/abs/2001.04451](https://arxiv.org/abs/2001.04451)

Uses the concept of Locality sensitive hashing (LSH) attention, where the goal is to not store the entire Q * K<sup>T</sup> matrix but only the softmax(Q * K<sup>T</sup>), which is dominated by the largest elements in a typically sparse matrix. For each query q we only need to pay attention to the keys k that are closest to q. For example, if K is of length 64K, for each q we could only consider a small subset of the 32 or 64 closest keys. So the attention mechanism finds the nearest neighbor keys of a query but in an inefficient manner. 

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/reformer-lsh.png" caption="" >}}

* Calcluate LSH hashes of Queries and Keys (Q and K)
* Make chunks and compute attention only for vectors in the same bucket

The paper also introduces the concept of Reversible residual networks (RevNets). In the residual connections in Transformers, one needs to store the activations in each layer in memory in order to calculate gradients during backpropagation. RevNets are composed of a series of reversible blocks. In RevNet, each layer’s activations can be reconstructed exactly from the subsequent layer’s activations, which enables us to perform backpropagation without storing the activations in memory.

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/reformer-revnet.png" caption="" >}}

Reformer applies the RevNet idea to the Transformer by combining the attention and feed-forward layers inside the RevNet block. Now F becomes an attention layer and G becomes the feed-forward layer:

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/reformer-revnet2.png" caption="" >}}

The reversible residual layers allows storing activations only once during the training process instead of N times.

The memory complexity of Reformer is <code>O(N log N)</code>.

## Memory-based

### Big Bird [https://arxiv.org/abs/2007.14062](https://arxiv.org/abs/2007.14062) + [https://huggingface.co/blog/big-bird](https://huggingface.co/blog/big-bird)

BigBird relies on block sparse attention and can handle sequences up to a length of 4096 at a much lower computational cost compared to BERT. It has achieved SOTA on various tasks involving very long sequences such as long documents summarization, question-answering with long contexts.

BigBird proposes three ways of allowing long-term attention dependencies while staying computationally efficient - 
* **Global attention** - Introduce some tokens which will attend to every token and which are attended by every token. The authors call this the 'internal transformer construction (ITC)' in which a subset of indices is selected as global tokens. This can be interpreted as a model-memory-based approach.
* **Sliding attention** - Tokens close to each other, attend together. In BigBird, each query attends to w/2 tokens to the left and w/2 tokens to the right. This corresponds to a fixed pattern (FP) approach.
* **Random attention** - Select some tokens randomly which will transfer information by transferring to other tokens which in turn can transfer to other tokens. This may reduce the cost of information travel from one token to other. Each query attends to r random keys. This pattern is fixed

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/bigbird-graph.gif" caption="" >}}

BigBird block sparse attention is a combination of sliding, global & random connections (total 10 connections) as shown in gif above. While a graph of normal attention (bottom) will have all 15 connections (note: total 6 nodes are present). One can simply think of normal attention as all the tokens attending globally.

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/bigbird-full.png" caption="" >}}

The attention calculation in BigBird is slightly complex and I would refer to the [Huggingface blog](https://huggingface.co/blog/big-bird#bigbird-block-sparse-attention) for it - 

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/bigbird-attention-gif.gif" caption="" >}}

<i>blue -> global blocks, red -> random blocks, orange -> sliding blocks</i>

The memory complexity of the self-attention is linear, i.e., <code>O(n)</code>. The BigBird model does not introduce new parameters beyond the Transformer model.

## Complexity Summary of various models

{{< figure src="/post/2022-10-10_efficient_transformers_survey/images/complexity-summary.png" caption="" >}}


-------

There are many more papers discussed in the survey. I will add their summaries here as I go through them.

&nbsp;

<script type="text/javascript" src="//downloads.mailchimp.com/js/signup-forms/popup/unique-methods/embed.js" data-dojo-config="usePlainJson: true, isDebug: false"></script>

<!-- <button style="background-color: #70ab17; color: #1770AB" id="openpopup">Subscribe to my posts!</button> -->
<div class="button_cont" align="center"><button id="openpopup" class="example_a">Subscribe to my posts!</button></div>

<style>
    .example_a {
        color: #fff !important;
        text-transform: uppercase;
        text-decoration: none;
        background: #3f51b5;
        padding: 20px;
        border-radius: 5px;
        cursor: pointer;
        display: inline-block;
        border: none;
        transition: all 0.4s ease 0s;
    }

    .example_a:hover {
        background: #434343;
        letter-spacing: 1px;
        -webkit-box-shadow: 0px 5px 40px -10px rgba(0,0,0,0.57);
        -moz-box-shadow: 0px 5px 40px -10px rgba(0,0,0,0.57);
        box-shadow: 5px 40px -10px rgba(0,0,0,0.57);
        transition: all 0.4s ease 0s;
    }
</style>


<script type="text/javascript">

function showMailingPopUp() {
    window.dojoRequire(["mojo/signup-forms/Loader"], function(L) { L.start({"baseUrl":"mc.us4.list-manage.com","uuid":"0b10ac14f50d7f4e7d11cf26a","lid":"667a1bb3da","uniqueMethods":true}) })

    document.cookie = "MCPopupClosed=;path=/;expires=Thu, 01 Jan 1970 00:00:00 UTC";
}

document.getElementById("openpopup").onclick = function() {showMailingPopUp()};

</script>

&nbsp;  

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).