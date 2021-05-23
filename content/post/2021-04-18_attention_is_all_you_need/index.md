---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Summary #1 - Attention Is All You Need"
subtitle: "The first of the paper summary series. This is where I briefly summarise the important papers that I read for my job or just for fun :P"
authors: ["Shreyansh Singh"]
tags: [paper reading, transformers, nlp, deep learning]
categories: [Machine Learning]
date: 2021-04-18T16:57:49+05:30
lastmod: 2021-04-18T16:57:49+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Attention Is All You Need"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: [Paper Notes]
---

**Paper**: Attention Is All You Need  
**Link**: [https://bit.ly/3aklLFY](https://bit.ly/3aklLFY)  
**Authors**: Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, Illia Polosukhin  
**Code**: [https://github.com/tensorflow/tensor2tensor](https://github.com/tensorflow/tensor2tensor)

------

## What?
Proposes Transformers, a new simple architecture for sequence transduction that uses only an attention mechanism and does not use any kind of recurrence or convolution. This model achieves SOTA (at the time) on the WMT 2014 English-to-French translation task with a score of 41.0 BLEU. Also beats the existing best results on the WMT 2014 English-to-German translation task with a score of 28.4 BLEU. The training cost is also much less than the best models chosen in the paper (at the time).

## Why?
Existing recurrent models like RNNs, LSTMs or GRUs work sequentially. They align the positions to steps in computation time. They generate a sequence of hidden states as a function of the previous hidden state and the input for the current position. But sequential computation has constraints. They are not easily parallelizable which is required when the sequence lengths become large. The Transformer model eschews recurrence and allows for more parallelization and requires less training time to achieve SOTA in the machine translation task.

## How?

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/arch.PNG" caption="Detailed Transformer Architecture" >}}

The model is auto-regressive, it consumes the previously generated symbols as additional input when generating the next.

### Encoder
The figure above shows just one layer of the encoder on the left. There are `N=6` such layers. Each layer has two sub-layers - a multi-head self-attention layer and a position-wise fully connected feed-forward network. [Residual connections](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf) and [layer normalization](https://arxiv.org/abs/1607.06450) is used for each sub-layer.

### Decoder
This also has `N=6` stacked layers. The architecture diagram shows one layer of the decoder on the right. Each layer has three sub-layers. Two of them are the same as the encoder. The third layer performs multi-head attention over the output of the encoder stack. This is modified to prevent positions from attending to subsequent positions. Additionally, the output embeddings are also offset by one position. These features ensure that the predictions for a position depend only on the known outputs for positions before it.

### Attention
The paper uses a modified dot product attention, and it is called "Scaled Dot Product Attention". Given queries and keys of dimension d<sub>k</sub> and values of dimension d<sub>v</sub>, the attention matrix is calculated as shown below.

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/attention.PNG" caption="Attention Matrix Calculation" >}}

Since, for large values of d<sub>k</sub> the dot product grows large in magnitude, it pushes the softmax function into regions where it has extremely small gradients. The scaling of 1/sqrt(d<sub>k</sub>) is done to avoid the problem of vanishing gradients. 


Multi-Head attention allows computing this attention in parallel. This helps to focus on different positions. Secondly, it also helps to attend to information from different subspaces due to the more number of attention heads. 

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/multihead-attention.PNG" caption="Multihead Attention Calculation" >}}

The paper uses `h=8` parallel attention layers or heads. The reduced dimension of each head compensates for the more number of heads and hence the computational cost remains the same as with single-head attention with full dimensionality.

Applications of multi-head attention in the paper are given below - 

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/application-attention.PNG" caption="Application of multi-head attention in the model" >}}

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/multihead-attention-fig.PNG" caption="Pictorial representaion of Multi-head attention" >}}

### Position-wise Feed-Forward Networks
The FFN sub-layer shown in the encoder and decoder architecture is a 2-hidden layer FC FNN with a ReLU activation in between. 

### Positional Encodings
Positional encodings are injected (added) to the input embeddings at the bottom of the encoder and decoder stack to add some information about the relative order of the tokens in the sequence. The positional encodings have the same dimension as the input embeddings so that they can be added.
For position *pos* and dimension *i* the paper uses the following positional embeddings - 

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/positional.PNG" caption="Positional Encoding calculation" >}}

This choice allows the model to easily learn by the relative positions. The learned positional embeddings also perform about the same as the sinusoidal version. The sinusoidal version may allow the model to extrapolate to sequence lengths longer than the ones encountered in training.


## Results

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/experiments.PNG" caption="Experimental results when varying parameters" >}}

* Form (A), it can be seen that single-head attention is slightly worse than the best setting. The quality also drops off with too many heads.
* (B) shows that reducing the attention key size <i>d<sub>k</sub></i> hurts model quality. 
* In (C) and (D), it is visible that bigger models are better and dropout helps in avoiding overfitting.
* (E) shows that sinusoidal positional encoding when replaced with learned positional embeddings also does not lead to a loss in quality

For the base models, the authors used a single model obtained by averaging the last 5 checkpoints, which were written at 10-minute intervals. The big models were averaged over the last 20 checkpoints. Beam search with a beam size of 4 and length penalty α = 0.6. The maximum output length during inference is set to input length +50, but if it is possible, the model terminates early.

The performance comparison with the other models is shown below -

{{< figure src="/post/2021-04-18_attention_is_all_you_need/images/results.PNG" caption="Model performance" >}}


-----

**I have also released an annotated version of the paper. If you are interested, you can find it [here](https://github.com/shreyansh26/Annotated-ML-Papers/blob/main/Attention%20Is%20All%20You%20Need.pdf).**

This is all for now!

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


