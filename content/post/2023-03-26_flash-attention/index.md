---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Summary #8 - FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"
subtitle: "Understanding FlashAttention which is the most efficient exact attention implementation out there, which optimizes for both memory requirements and wall-clock time."
summary: ""
authors: ["Shreyansh Singh"]
tags: [transformers, attention, deep learning, paper-summaries]
categories: [Machine Learning]
date: 2023-03-26T15:47:48+05:30
lastmod: 2023-03-26T15:47:48+05:30
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

**Paper**: FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness  
**Link**: [https://arxiv.org/abs/2205.14135](https://arxiv.org/abs/2205.14135)  
**Authors**: Tri Dao, Daniel Y. Fu, Stefano Ermon, Atri Rudra, Christopher Ré  
**Code**: [https://github.com/HazyResearch/flash-attention](https://github.com/HazyResearch/flash-attention)

**I have also released an annotated version of the paper. If you are interested, you can find it [here](https://github.com/shreyansh26/Annotated-ML-Papers/blob/main/General-DL/FlashAttention%20-%20Fast%20and%20Memory-Efficient%20Exact%20Attention.pdf).**

--------------------

**\[Update\] - I implemented a simplified version of FlashAttention (without the CUDA and SRAM memory optimizations) in PyTorch. [Check it out on Github.](https://github.com/shreyansh26/FlashAttention-PyTorch)**

I finished reading the FlashAttention paper recently and thought that it would be good to have a technical write-up of the paper, so that it can help me understand the concept well. I decided to make it public and hopefully it can help anyone reading this.

## Overview

Attention as we know, in its standard implementation is an $O(N^2)$ operation, where N is the sequence length. There are many approximate attention methods out there like Reformer, Smyrf, Reformer, Performer and others ([you can find more details on a few of these in my previous blog](https://shreyansh26.github.io/post/2022-10-10_efficient_transformers_survey/)) which aim to reduce the compute requirements to linear or near-linear in sequence length, but many of them do not display wall-clock speedup against standard attention. They focus on FLOP reduction (which doesn't always correlate with wall-clock speed) and tend to ignore overheads from memory access (IO).FlashAttention aims to incorporate IO-awareness i.e. dividing operations between faster and slower levels of GPU memory to make the whole computation faster. The algorithm uses tiling to reduce the number of memory reads/writes between GPU high bandwidth memory (HBM) and GPU on-chip SRAM. FlashAttention can also be extended to block-spare attention and this results in the fastest approximate (or not) attention algorithm out there. 

All this helps to improve the training time of Transformer models - a 15% end-to-end wall-clock speedup on BERT-large (seq. length 512) compared to the MLPerf 1.1 training speed record, 3× speedup on GPT-2 (seq. length 1K). This memory-efficient approach also helps to incorporate a longer context (up to 16k/64k tokens) which also results in better models (0.7 better perplexity on GPT-2).

I'll describe more details in the future sections.

## Background - Hardware Performance

Since FlashAttention computes exact attention, and the major crux of their work is the efficient hardware usage, it is important to know a bit about GPU memory and the performance characteristics of various kinds of operations on it. 

{{< figure src="/post/2023-03-26_flash-attention/images/gpu_mem.png" caption="A100 GPU Memory Hierarchy. Source - [https://arxiv.org/abs/2205.14135](https://arxiv.org/abs/2205.14135)" >}}

### GPU Memory Hierarchy

For a A100 GPU with 40GB of High Memory Bandwidth (HBM), a rough diagram of the memory hierarchy is shown above. The SRAM memory us spread across 108 streaming multiprocessors (SMs), 192KB for each. As one can see, the on-chip SRAM is much faster the HBM but is much smaller than size. In terms of compute, the theoretical peak throughput for BFLOAT16 using Tensor Core is 312 TFLOPS. With time, compute has gotten much faster relative to memory speed, hence processes (operations) are increasingly bottlenecked by memory (HBM) access. Thus, the goal of the FlashAttention paper was to use the SRAM as well as efficiently as possible to speed up the computation.

### Execution Model

he typical way in which GPUs operate are that they use a large number of threads to perform an operation, which is called a kernel. The input is loaded from the HBM to the registers and SRAM, and written back to the HBM after computation. 

### Performance Characteristics

There is a term called **arithmetic intensity** which is given by the number of arithmetic operations per byte of memory access. It helps to understand the bottleneck of an operation. An operation can be characterized as compute-bound (also called math-bound) or memory-bound. 

* **Compute-bound** - When the bottleneck is the compute i.e., the time taken by the operation is determined by how many arithmetic operations there are since the time taken due to HBM accesses is relative lower. E.g. of such operations are matrix multiplication with large inner dimension, and convolution with large number of channels.

* **Memory-bound** - When the bottleneck is the memory i.e., the time taken by the operation is determined by the number of memory accesses there are since the time spent in computation is relative lower. E.g. of such processes are most other operation like elementwise operations - activation, dropout and reduction operations - sum, softmax, batch normalization, layer normalization.

To understand this better, let's analyze it mathematically. Let $N_{op}$ be the number of arithmetic/floating point operations, $N_{byte}$ be the number of memory accesses, ${BW}_{compute}$ and ${BW}_{memory}$ be the compute and memory bandwidth respectively, the time taken for compute operations and memory accesses can be determined as - 

$$t_{compute} = \frac{N_{op}}{{BW}_{compute}}$$
$$t_{memory} = \frac{N_{byte}}{{BW}_{memory}}$$

The operation is compute-bound if $t_{compute}$ is greater than $t_{memory}$ and vice-versa for memory bound. Which mathematically becomes -

**For compute-bound**
$$\frac{N_{op}}{N_{byte}} \gt \frac{{BW}_{compute}}{{BW}_{memory}}$$

**For memory-bound**
$$\frac{N_{op}}{N_{byte}} \lt \frac{{BW}_{compute}}{{BW}_{memory}}$$

As mentioned above as well, matrix multiplication for large inner dimensions is compute bound but below that it is memory bound. If using FP32 and plugging in numbers for A100 40GB, then for $N \lt 74$, the $N \times N$ multiplication is memory bound, but compute bound when $N$ is greater than that. A great and detailed resource to understand this theory is this [blog post by Lei Mao](https://leimao.github.io/blog/Math-Bound-VS-Memory-Bound-Operations/).

### Kernel Fusion

Kernel Fusion is often down by compilers to fuse together multiple elementwise operations. It is used to accelerate memory-bound operations. The basic ideas is that instead of loading the input from the HBM, performing the operation and writing back to the HBM and repeating that for each operation applied to the same input, the operation can be fused so that all of the operations are performed at once when the input is loaded from the HBM.

However, one must note that when performing model training, the effectiveness of kernel fusion is reduced as the intermediate values still have to be written to the HBM to save for the backward pass.

## Background - Standard Attention

For anyone familiar with transformers, this equation is well-known - 


$$Attention(Q, K, V) = softmax(\frac{QK^\mathsf{T}}{\sqrt{d_k}})V$$

Here, the sequences $Q, K, V  \in \mathbb{R}^{N \times d}$ where $N$ is the sequence length and $d$ is the head dimension. The attention output, above, can be denoted by $O \in \mathbb{R}^{N \times d}$. The equation can be broken down as - 

$$\mathbf{S} = \mathbf{QK^\mathsf{T}} \in \mathbb{R}^{N \times N},\quad \mathbf{P} = softmax(\mathbf{S}) \in \mathbb{R}^{N \times N},\quad \mathbf{O} = \mathbf{PV} \in \mathbb{R}^{N \times d}$$

{{< figure src="/post/2023-03-26_flash-attention/images/att2.png" caption="Scaled Dot Product Attention" >}}

In standard attention implementations, the $\mathbf{S}$ and $\mathbf{P}$ matrices are materialized in the HBM, which takes $O(N^2)$ memory. Also, most operations are memory-bound/elementwise operations, e.g. softmax applied on $\mathbf{P}$, masking applied to $\mathbf{S}$, dropout applied to $\mathbf{P}$. This leads to slow wall-clock time.

{{< figure src="/post/2023-03-26_flash-attention/images/standard-att-algo.png" caption="Standard Attention Implementation" >}}


## FlashAttention - Algorithm details

As one may understand, the materialization of the $N \times N$ attention matrix on the HBM and its repeated reading and writing is a major bottleneck. To solve this, two main things need to be done - 

1. Computing the softmax reduction without access to the whole input
2. Not storing the large intermediate attention matrix for the backward pass

Two established techniques, namely **tiling** and **recomputation** are used to solve this. 

1. Tiling - The attention computation is restructured to split the input into blocks and performing the softmax operation incrementally by making several passes over the input blocks. 
2. Recomputation - The softmax normalization factor from the forward pass is stored to quickly recompute attention on-chip in the backward pass, which is faster than the standard attention approach of reading the intermediate matrix from HBM.

This does lead to increased FLOPs due to recomputation, however FlashAttention runs both faster (up to 7.6x on GPT-2) and uses less memory — linear in sequence length, due to the massively reduced amount of HBM access.

{{< figure src="/post/2023-03-26_flash-attention/images/gpt2-att-speedup.png" caption="Speedup over the PyTorch implementation of attention on GPT-2" >}}

### Understanding the algorithm

{{< figure src="/post/2023-03-26_flash-attention/images/flash-attention-schematic.png" caption="FlashAttention Forward Pass Algorithm" >}}

The main idea behind the algorithm is to split the inputs $\mathbf{Q, K, V}$ into blocks, loading them from slow HBM to fast SRAM and then computing the attention output w.r.t those blocks. The output of each block is scaled by the right normalization factor before adding them up, which gives the correct result.

$$\mathbf{S} = \mathbf{\tau QK^\mathsf{T}} \in \mathbb{R}^{N \times N},\quad \mathbf{S}^\mathrm{masked} = \mathrm{MASK}(S) \in \mathbb{R}^{N \times N},\quad \mathbf{P} = softmax(\mathbf{S^\mathrm{masked}}) \in \mathbb{R}^{N \times N},$$

$$\mathbf{P}^\mathrm{dropped} = \mathrm{dropout}(\mathbf{P}, p_\mathrm{drop}), \quad \mathbf{O} = \mathbf{P^\mathrm{dropped}V} \in \mathbb{R}^{N \times d},$$

where $\tau \in \mathbb{R}$ is some softmax scaling factor (typically $\frac{1}{\sqrt{d}}$), $\mathrm{MASK}$ is some masking function that sets some entries of
the input to $-\infty$ and keep other entries the same, and $\mathrm{dropout}(x, p)$ applies dropout to 𝑥 elementwise (i.e., output $\frac{x}{1-p}$ with probability $1 − p$ and output $0$ with probability $p$ for each element $x$)

{{< figure src="/post/2023-03-26_flash-attention/images/flash-attention-forward-algo.png" caption="FlashAttention Forward Pass Algorithm" >}}

#### Tiling

The key part in understanding the block-wise computation of attention in the algorithm above is the block-wise computation of the softmax. The paper explains it well though. The softmax of a vector $x \in \mathbb{R}^B$ can be computed as -

{{< figure src="/post/2023-03-26_flash-attention/images/softmax-1.png" caption="" >}}

And for vectors $x^\mathrm{(1)}, x^\mathrm{(2)} \in \mathbb{R}^B$, the softmax of the concatenated $x = [x^\mathrm{(1)}, x^\mathrm{(2)}] \in \mathbb{R}^{2B}$ is given by -

{{< figure src="/post/2023-03-26_flash-attention/images/softmax-2.png" caption="" >}}

Let's understand this better. In the above equations, $m(x)$ holds the maximum between $m(x^\mathrm{(1)})$ and $m(x^\mathrm{(2)})$. Now, $m(x^\mathrm{(1)})$ is the maximum element of $x^\mathrm{(1)}$ and $m(x^\mathrm{(2)})$ is the maximum element of $x^\mathrm{(2)}$ which means that $m(x)$ is basically the maximum of the whole concatenated vector. The beauty is that this was done blockwise.

So, if statistics $(m(x), l(x))$ are tracked then softmax can be computed one block at a time. In line 12 of the algorithm, $\tilde{m_{ij}}$ has the maximum element of each row of $S_{ij}^\mathrm{masked}$, and next in line 13, $m_i^\mathrm{new}$ holds the row-wise maximum of the $m_i$ till now and the new one i.e., $\tilde{m_{ij}}$. Hence $m_i$ is updated every column from the outer loop and eventually stores the row-wise max of the matrix $\mathbf{S}$. The same logic goes for $l_i$ and the matrix $\mathbf{P}$. The results are combined to get the output attention matrix in line 15.

#### Recomputation
The backward pass of FlashAttention requires the $\mathbf{S}$ and $\mathbf{P}$ matrices to compute the gradients w.r.t $\mathbf{Q}$,$\mathbf{K}$,$\mathbf{V}$. However, they are $N \times N$ matrices and as it can be seen in the algorithm above, they aren't stored explicitly. The trick is to use the output $\mathbf{O}$ and the softmax normalization statistics $(m, l)$, we can recompute the attention matrix $\mathbf{S}$ and $\mathbf{P}$ easily in the backward pass from blocks of $\mathbf{Q}$,$\mathbf{K}$,$\mathbf{V}$ in SRAM. even with more FLOPs, the recomputation step speeds up the backward pass due to reduced HBM accesses. The backward pass is very interesting too but slightly more complicated hence I'll probably cover it in a separate post. One can cover the Appendix B of the paper to learn more.

Kernel Fusion is also used to implement the algorithm in one CUDA kernel, loading input from HBM, performing all the computation steps (matrix multiply, softmax, optionally masking and dropout, matrix multiply), then writing the result back to HBM. This avoids repeatedly reading and writing of inputs and outputs from and to HBM.

**Important Information** - *The FlashAttention algorithm computed $\mathbf{O} = softmax(QK^\mathsf{T})V$ with $O(N^2d)$ FLOPs and requires $O(N)$ additional memory beyond inputs and output (for the $(l, m)$ statistics).*

The proof for the FLOPs calculation is given in Appendix C of the paper, which should be checked out by the curious reader.

**Important Information** - *Let $N$ be the sequence length, $d$ be the head dimension, and $M$ be the size of SRAM with $d \leq M \leq Nd$. Standard attention requires $\Theta(Nd + N^2)$ HBM accesses while FlashAttention requires $\Theta(N^2d^2M^{-1})$ HBM accesses.*

For typical values of $d$ (64-128) and $M$ (around 100KB), $d^2$ is many times smaller than $M$, and thus FlashAttention requires many times fewer HBM accesses than standard implementation. This leads to both faster execution and a lower memory footprint. 

The authors also go on to show that the number of HBM accesses by FlashAttention is a lower-bound. There can be no implementation which can asymptotically improve on the number of HBM accesses for all values of $M$ when doing exact attention calculation.

As the block size increases, the number of HBM accesses decreases as there are less passes over the input, and the runtime also decreases. However, beyond 256, the runtime starts getting bottlenecked by factors like arithmetic operations. And there is also a limit on how large we can choose the block size to be, as we want it to be able to fit in the SRAM.

{{< figure src="/post/2023-03-26_flash-attention/images/res-1.png" caption="**Left** - Comparison of standard attention and FlashAttention for GPT-2 medium on A100. Despite the higher FLOPs (due to the recomputation step in backward pass), the lesser number of HBM access leads to a much faster runtime. **Right** - The effect of block size on the forward runtime and HBM accesses." >}}

### Block-Sparse FlashAttention

As mentioned in the overview, FlashAttention can be used to make a approximate attention algorithm as well. The authors call it Block-Sparse FlashAttention and it is the fastest approximate attention algorithm. The memory complexity is smaller than FlashAttention by a factor proportional to the sparsity.

For inputs $\mathbf{Q, K, V} \in \mathbb{R}^{N \times d}$ and a mask $\tilde{\mathbf{M}} \in \{ 0,1 \}^{N \times N}$, we want to calculate -

$$\mathbf{S} = \mathbf{QK^\mathsf{T}} \in \mathbb{R}^{N \times N},\quad \mathbf{P} = softmax(\mathbf{S} \odot \mathbb{1}_{\tilde{\mathbf{\mathrm{M}}}}) \in \mathbb{R}^{N \times N},\quad \mathbf{O} = \mathbf{PV} \in \mathbb{R}^{N \times d}$$

Given a pre-defined block sparsity mask $\mathbf{M} \in \{ 0,1 \}^{N/B_r \times N/B_c}$, Algorithm 2 above can be adapted to only compute the nonzero blocks of the attention matrix. We can just skip the zero blocks. The Algorithm shown below describes the forward pass of Block-sparse FlashAttention.

{{< figure src="/post/2023-03-26_flash-attention/images/blocksparse-flash-attention-forward-algo.png" caption="Blcok-Sparse FlashAttention Forward Pass Algorithm" >}}

**Important Information** - *Let $N$ be the sequence length, $d$ be the head dimension, and $M$ be the size of SRAM with $d \leq M \leq Nd$. Block-sparse FlashAttention requires $\Theta(Nd + N^2d^2M^{-1}s)$ HBM accesses where $s$ is the fraction of nonzero blocks in the block-sparsity mask.*

For large sequence lengths, $s$ is set to $N^{-1/2}$ or $N^{-1} \log N$ resulting in $\Theta(N \sqrt{N})$ or $\Theta(N \log N)$ IO complexity. As the sparsity increases, the runtime of block-sparse FlashAttention improves proportionally.

{{< figure src="/post/2023-03-26_flash-attention/images/res-2.png" caption="" >}}

## Experiments

There are tons of results in the paper. But the TL;DR is that FlashAttention beats all other exact attention algorithms in both training speed and quality of the models/down stream models especially when pushed to the limits of sequence length. I'll add the plots and graphs for their various results here. Additional results are present in the paper.

### Training Speed

#### BERT
{{< figure src="/post/2023-03-26_flash-attention/images/res-3.png" caption="Training time of BERT-large. starting from the same initialization provided by the MLPerf benchmark, to reach the target accuracy of 72.0% on masked language modeling. Averaged over 10 runs on 8×A100 GPUs." >}}

#### GPT-2
{{< figure src="/post/2023-03-26_flash-attention/images/res-4.png" caption="GPT-2 small and medium using FlashAttention achieve up to 3× speed up compared to Huggingface implementation and up to 1.7× compared to Megatron-LM. Training time reported on 8×A100s GPUs." >}}

#### Long-range Arena
{{< figure src="/post/2023-03-26_flash-attention/images/res-5.png" caption="The performance of standard attention, FlashAttention, block-sparse FlashAttention, and approximate attention baselines on the Long-Range-Arena benchmarks. Each task has a different sequence length varying between 1024 and 4096." >}}

Block-sparse FlashAttention is faster than all of the approximate attention methods that were tested.

### Model Quality

#### Language Modeling with Long Context

{{< figure src="/post/2023-03-26_flash-attention/images/res-6.png" caption="GPT-2 small with FlashAttention, with 4× larger context length compared to Megatron-LM, is still 30% faster while achieving 0.7 better perplexity. Training time on 8×A100 GPUs is reported." >}}

#### Long Document Classification

Since FlashAttention allows training on longer sequences, it improves performance on such datasets. MIMIC-III contains intensive care unit patient discharge summaries, each annotated with multiple labels. ECtHR contains legal cases from the European Court of Human Rights, each of which is mapped to articles of the Convention of Human Rights
that were allegedly violated. Both of these datasets contain very long text documents. The average number of tokens in MIMIC-III is 2395 tokens and the longest document contains 14562 tokens.

{{< figure src="/post/2023-03-26_flash-attention/images/res-7.png" caption="Sequence length 16K outperforms length 512 by 4.3 points on MIMIC, and that length 8K outperforms length 512 by 8.5 points on ECtHR. The discrepancies may be due to subtle distribution shifts: MIMIC-III contains specialized medical text and thus may be more susceptible to a distribution shift in the document length, whereas ECtHR contains general language." >}}

#### Path-X and Path-256
These are challenging tasks from the long range arena benchmark where the task is to classify whether two points in a black and white 128×128 (or 256×256) image have a path connecting them, and the images are fed to the transformer one pixel at a time. No transformer model in the past has been able to model these tasks effectively. They have either ran out of memory or achieved random performance. FlashAttention yields the first Transformer that can achieve better-than-random performance on the challenging Path-X task (sequence length 16K), and block-sparse FlashAttention yields the first sequence model that can achieve better-than-random performance on Path-256 (sequence length 64K).


{{< figure src="/post/2023-03-26_flash-attention/images/res-8.png" caption="First Transformer model that can achieve non-random performance on Path-X and Path-256. Path-256 requires longer sequences but has relatively shorter paths than Path-X, so it is easier to obtain a higher accuracy." >}}

### Benchmarking Attention

{{< figure src="/post/2023-03-26_flash-attention/images/res-9.png" caption="**Left** - runtime of forward pass + backward pass. **Right** - attention memory usage" >}}

#### Runtime
FlashAttention beats all exact attention baselines and is about 3× faster than the PyTorch implementation. The runtimes of many approximate/sparse attention mechanisms grow linearly with sequence length, but FlashAttention still runs faster than approximate and sparse attention for short sequences due to fewer memory accesses. The approximate attention runtimes begin to cross over with FlashAttention at sequences between 512 and 1024. On the other hand, block-sparse FlashAttention is faster than all implementations of exact, sparse, and approximate attention that are available, across all sequence lengths.

#### Memory Footprint
FlashAttention and block-sparse FlashAttention have the same memory footprint, which grows linearly with sequence length. FlashAttention is up to 20× more memory efficient than exact attention baselines, and is more memory-efficient than the approximate attention baselines.  All other algorithms except for Linformer run out of memory on an A100 GPU before 64K, and FlashAttention is still 2× more efficient than Linformer. 

-------

A great paper overall, tremendous impact and personally, I had loads to learn from it!

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

<script data-name="BMC-Widget" data-cfasync="false" src="https://cdnjs.buymeacoffee.com/1.0.0/widget.prod.min.js" data-id="shreyanshsingh" data-description="Support me on Buy me a coffee!" data-message="" data-color="#FF5F5F" data-position="Right" data-x_margin="18" data-y_margin="18"></script>

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).