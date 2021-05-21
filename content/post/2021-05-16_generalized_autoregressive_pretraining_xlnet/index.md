---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Notes #5 - XLNet: Generalized Autoregressive Pretraining for Language Understanding"
subtitle: "XLNet tries to overcome the limitations of BERT by having a autoregressive component while also capturing the bidirectional context."
authors: ["Shreyansh Singh"]
tags: [paper reading, autoregressive, transformer, transformer-xl, nlp, language model, deep learning]
categories: [Machine Learning]
date: 2021-05-16T14:25:04+05:30
lastmod: 2021-05-16T14:25:04+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "XLNet: Generalized Autoregressive Pretraining for Language Understanding"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: [Paper Notes]
---

**Paper**: XLNet: Generalized Autoregressive Pretraining for Language Understanding  
**Link**: [https://arxiv.org/pdf/1906.08237.pdf](https://arxiv.org/pdf/1906.08237.pdf)  
**Authors**: Zhilin Yang, Zihang Dai, Yiming Yang, Jaime Carbonell, Ruslan Salakhutdinov, Quoc V. Le  
**Code**: [https://github.com/zihangdai/xlnet](https://github.com/zihangdai/xlnet)  

----------------

## What?
The paper proposes XLNet, a generalized autoregressive pretraining method that enables learning bidirectional contexts over all permutations of the factorization order and overcomes the limitations of BERT due to the autoregressive formulation of XLNet. XLNet incorporates Transformer-XL as the underlying model. It outperforms BERT in 20 NLP tasks like question answering, natural language inference, sentiment analysis and document ranking.

## Why?
The existing unsupervised representation learning approaches can be divided into two types - autoregressive language modeling and autoencoding approaches. The autoregressive methods like ELMo and GPT tried to estimate the probability distribution of a text corpus with an autoregressive model. They had a limitation in that they only captured the unidirectional context. BERT aimed to solve this problem by aiming to reconstruct the original data from the corrupted input. So BERT could capture the bidirectional context, but by converting this into a prediction problem, BERT assumed that the predicted tokens are independent of each other. However, that is not the case in natural language where long term dependency is prevalent. Moreover, the use of the \[MASK\] tokens also created a pretrain-finetune discrepancy as there are no \[MASK\] tokens available during finetuning.

XLNet tries to leverage the best of both worlds. The qualities of XLNet are - 
* XLNet computes the maximum likelihood of a sequence w.r.t. all possible permutations of the factorization order. So when calculating the expectation, each position learns to capture the context from all positions, hence capturing bidirectional context.
* XLNet does not rely on data corruption as in BERT and hence does not suffer from the pretrain-finetune discrepancy. 
* XLNet integrates the novelties from Transformer-XL like recurrence mechanism and relative encoding scheme (explained later as well). This improves the performance of tasks that utilise a longer text sequence. 

## How?
Autoregressive language modeling performs pretraining by maximizing the likelihood under the forward autoregressive factorization - 

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/arobjective.PNG" caption="" >}}

Here *x* is the given text sequence. h<sub>&Theta;</sub>(x<sub>1:t-1</sub>) is the context representation produced by the model and *e*(x) is the embedding of *x*.

Denoising autoencoding approach like BERT first constructs a corrupt version *x*(cap) by randomly masking a fraction (15%) of tokens of *x* to a special symbol \[MASK\]. The masked tokens are denoted by *x*(bar). So, the training objective in the case of BERT becomes - 

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/aeobjective.PNG" caption="" >}}

Here *m*<sub>t</sub> is 1 when *x*<sub>t</sub> is masked. Here H<sub>&Theta;</sub> is a Transformer that maps each token to a sequence of length *T* to hidden vectors \[H<sub>&Theta;</sub>(x)<sub>1</sub>, H<sub>&Theta;</sub>(x)<sub>2</sub>, ..., H<sub>&Theta;</sub>(x)<sub>T</sub>\].

In BERT, the conditional probability is taken when the input is masked, denoted using *m*<sub>t</sub>. Hence denoting the independence assumption among the targets.

### Objective: Permutation Language Modeling
Both autoregressive and autoencoding approaches have their benefits over each other. XLNet tries to bring both their advantages into the picture while avoiding their weaknesses.

XLNet proposes the use of permutation language modeling objective that looks like the general autoregressive language modeling approach but it allows the model to capture bidirectional context as well. Here, the training is performed for each valid autoregressive factorization order (permutations) of the sequence. The model parameters are shared across all the factorization orders, and hence the model learns to capture information from all positions on both sides. 

The proposed permutation language modeling approach is -

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/xlnetobjective.PNG" caption="" >}}

Here *Z*<sub>T</sub> is the set of all possible permutations of the length-*T* index sequence \[1, 2, 3..., T\]. *z*<sub>t</sub> and z<sub>\<t</sub> denote the t-th element and the first *t-1* elements of the permutation. So, basically the autoregressive formulation is applied for each factorization order z.

Since this is based on the autoregressive framework, the independence assumption of BERT is no longer present in this case and the pretrain-finetune discrepancy is also not present.

\* One must note that here the objective does not permute the sequence order. The sequence order remains as it is and the positional encodings correspond to the original sequence itself. Here, the attention mask in Transformers is used to achieve the permutation of the factorization order. This is done because permuting the sequence itself can cause problems as during finetuning, the natural order will always be preserved. The authors do not want to include any other pretrain-finetune discrepancy.

### Architecture: Two-Stream Self-Attention for Target-Aware Representations
Using the Transformer(-XL) directly with the permutation language modeling objective will not work. This is because, say we have two sequences, in which z<sub>\<t</sub> sequence is same but the z<sub>t</sub> token is different. And in the current formulation using transformers(-XL) the z<sub>\<t</sub> sequence determines z<sub>t</sub> but that would not be correct if we predict the same distribution for both the sequences while they have two different tokens as the target.

To solve this, a re-parameterization of the next-token distribution with the target-position (z<sub>t</sub>) is required.

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/rexlnetobjective.png" caption="" >}}

Here, g<sub>&Theta;</sub>(x<sub>z<sub>\<t</sub></sub>, <i>z</i><sub></sub>) is a new type of representation that takes in the z<sub>t</sub> as input as well.


#### Two-Stream Self-Attention

Now, there is a contradiction here. If we want to predict x<sub>z<sub>t</sub></sub>, g<sub>&Theta;</sub>(x<sub>z<sub>\<t</sub></sub>, <i>z</i><sub>t</sub>) should only use position z<sub>t</sub> and not x<sub>z<sub>t</sub></sub> itself. Also, to predict the future tokens x<sub>z<sub>j</sub></sub> with j > t, we need  x<sub>z<sub>t</sub></sub> to provide the full context information.

So, to resolve this, XLNet uses two hidden representations - 
* Content representation h<sub>&Theta;</sub>(x<sub>z<sub>\<=t</sub></sub>) abbreviated as h<sub>z<sub>t</sub></sub>, which is similar to the general Transformer hidden state. This encodes both the context and the token x<sub>z<sub>t</sub></sub>.
* Query representation g<sub>&Theta;</sub>(x<sub>z<sub>\<t</sub></sub>, <i>z</i><sub>t</sub>) , abbreviated as g<sub>z<sub>t</sub></sub>, which only has access to the contextual information x<sub>z<sub>\<t</sub></sub> and the position z<sub>t</sub> but not the contents x<sub>z<sub>t</sub></sub>.

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/arch.PNG" caption="" >}}

The above diagram shows the flow of the two streams. The two streams are updated with a set of shared parameters as follows - 

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/update.PNG" caption="" >}}

The update rule of the content representations is exactly the same as the standard self-attention. During finetuning, the query stream can be dropped and the content stream can be used as a normal Transformer(-XL). And in the end, the last-layer query representation g<sub>z<sub>t</sub></sub> is used to compute the likelihood.

------

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

-------

#### Partial Prediction
For a given factorization order ***z***,  cutting point *c* is chosen which splits the sequence into two subsequences. ***z***<sub>\<=c</sub> is the non-target subsequence and ***z***<sub>\>c</sub> is the target sequence. The objective to maximize the log-likelihood of the target subsequence conditioned on the non-target subsequence is written as - 

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/partial.PNG" caption="" >}}

A hyperparameter *K* is chosen to determine what fraction of the sequence length will be the target sequence. This is done so that sufficient sequence length is present for the model to learn the context.

Here again, XLNet differs from BERT. Let us consider an example \[New, York, is, a city\]. If both BERT and XLNet take two tokens [New, York] as the prediction task and so they have to maximize p(New York | is a city). Here BERT and XLNet get the following objectives - 

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/bertxlnet.PNG" caption="" >}}

XLNet considers the dependency in the target subsequence as well i.e., how "York" depends on "New" as well. XLNet always learns more dependency pairs given the same target and contains “denser” effective training signals.

### Ideas from Transformer-XL
Transformer-XL introduced the segment recurrence mechanism for caching and reuse of the previous segment knowledge. For a long sequence **s**, if we take two segments *z*(bar) and *z* which are permutations of the segment, then the obtained representations from the first segment h(bar)<sup>(m)</sup> for each layer *m* can be cached and reused for the next segment. This can be written as - 

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/recur.PNG" caption="" >}}

Also since the positional embeddings depend on the actual positions in the original sequence, the above attention update is independent of the previous segment once the hidden representations have been calculated. So the factorization order of the previous segment need not be known. Subsequently, the model learns to utilize the memory over all factorization orders of the last segment. The same is done for the query stream as well.

### Modeling Multiple Segments
Like BERT, XLNet randomly samples two segments (either from the same context or not) and treats the concatenation of two segments as one sequence to perform permutation language modeling.

XLNET introduces Relative Segment Encodings. Unlike BERT which had absolute segment embeddings that were added to the word embedding at each position, here, rather than giving the entire segment an encoding, relative encoding is used between positions to denote whether they belong to the same segment or not.
The segment encoding of the positions is used to compute the attention weight. So, when position *i* attends to *j*, the segment encoding s<sub><i>ij</i></sub> is used to compute an attention weight a<sub>ij</sub> = (q<sub>i</sub> + b)<sup>T</sup>s<sub>ij</sub> , where q<sub>i</sub> is the query vector as in a standard attention operation and *b* is a learnable head-specific bias vector.

Relative segment encodings help because - 
* Inductive bias of the relative encodings improves the generalization.
* Opens up the possibility of finetuning on tasks that have more than two input segments, which is not possible when using absolute segment encodings.

## Results
Two datasets were the same as the ones BERT used i.e., BooksCorpus and English Wikipedia. Furthermore, Giga5, ClueWeb 2012-B and CommonCrawl datasets were also used. SentencePiece tokenization was used.

XLNet had the same architecture hyperparameters as BERT-Base and XLNet-Large had the same hyperparameters as BERT-Large. this resulted in similar model size and hence a fair comparison. 

XLNet was trained on 512 TPU v3 chips for 500K steps with an Adam weight decay
optimizer, linear learning rate decay, and a batch size of 8192, which took about 5.5 days. And even after using so much compute and time, the model still underfitted on the data at the end of the training. 

Since the recurrence mechanism is introduced, XLNet uses a bidirectional data input pipeline where each of the forward and backward directions takes half of the batch size. The idea of span-based prediction, where first, a sample length *L* from \[1, ..., 5] is chosen and then a consecutive span of *L* tokens is randomly selected as prediction targets within a context of (*KL*) tokens.

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/comparisonwithbert.PNG" caption="" >}}

As seen above, trained on the same data with an almost identical training recipe,
XLNet outperforms BERT by a sizable margin on all the considered datasets.

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/comp1.PNG" caption="Performance on reading comprehension and document ranking tasks. Comparison with GPT, BERT, RoBERTa and a BERT ensemble" >}}

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/comp2.PNG" caption="Performance on question answering tasks - SQuADv1.1 and SQuADv2.0" >}}

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/comp3.PNG" caption="Performance on text classification task." >}}

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/comp4.PNG" caption="Performance on natural language understanding tasks - the GLUE benchmark." >}}

* For explicit reasoning tasks like SQuAD and RACE that involve longer context, the performance gain of XLNet is larger. The use of Transformer-XL could be the main reason behind this. 
* For classification tasks that already have abundant supervised examples such as MNLI (>390K), Yelp (>560K) and Amazon (>3M), XLNet still lead to substantial gains.

**Ablation study** was also performed to understand the importance and effect of introducing each component. The points of the study were - 

* The effectiveness of the permutation language modeling objective alone, especially compared to the denoising auto-encoding objective used by BERT. 
* The importance of using Transformer-XL as the backbone neural architecture. For this, a DAE + Transformer-XL model was used.
* The necessity of some implementation details including span-based prediction, the bidirectional input pipeline, and next-sentence prediction.

For a fair comparison, all models were based on a 12-layer architecture with the same model hyper-parameters as BERT-Base and were trained on only Wikipedia and the BooksCorpus. All results reported are the median of 5 runs.

{{< figure src="/post/2021-05-16_generalized_autoregressive_pretraining_xlnet/images/ablation.PNG" caption="Performance on natural language understanding tasks - the GLUE benchmark." >}}

From the table - 

* Transformer-XL and the permutation LM (the basis of XLNet) are big factors in the superior performance of XLNet over BERT.
* On removing the memory caching mechanism, the performance drops especially for RACE where long context understanding is needed.
* Span-based prediction and bidirectional input pipeline also help in the performance of XLNet.
* The next-sentence prediction objective does not lead to an improvement. Hence the next-sentence prediction objective is excluded from XLNet.

-------

**I have also released an annotated version of the paper. If you are interested, you can find it [here](https://github.com/shreyansh26/Annotated-ML-Papers/blob/main/XLNet.pdf).**

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
