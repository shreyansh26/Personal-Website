---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Notes #3 - Improving Language Understanding by Generative Pre-Training"
subtitle: "The first paper in the GPT set of models. This is OpenAI's GPT-1."
authors: ["Shreyansh Singh"]
tags: [paper reading, word representations, nlp, language model, gpt, transformer, deep learning]
categories: [Machine Learning]
date: 2021-05-02T13:42:14+05:30
lastmod: 2021-05-02T13:42:14+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Improving Language Understanding by Generative Pre-Training"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: [Paper Notes]
---

**Paper**: Improving Language Understanding by Generative Pre-Training  
**Link**: [https://bit.ly/3xITvGP](https://s3-us-west-2.amazonaws.com/openai-assets/research-covers/language-unsupervised/language_understanding_paper.pdf)  
**Blog**: [https://openai.com/blog/language-unsupervised/](https://openai.com/blog/language-unsupervised/)    
**Authors**: Alec Radford, Karthik Narasimhan, Tim Salimans, Ilya Sutskever  
**Code**: [https://bit.ly/3gUFrUX](https://github.com/openai/finetune-transformer-lm)

---------

## What?
The paper proposes a smi-supervised technique which shows better performance on a wide variety of tasks like textual entailment, question answering, semantic similarity text classification by using a single task-agnostic model. The model can overcome the constraints of small amount of annotated data for these specific tasks by performing a unsupervised generative-pretraining of a language model on a large diverse text corpus followed by supervised discriminative fine-tuning on each specific task. The pretraining model remains the same for all the tasks. Only a small, task-aware input adaptation is required when performing the fine-tuning. The model significantly improved the state-of-the-art (at the time) in 9 of the 12 tasks studied.

## Why?
Most deep learning models require a substantial amount of data, which makes them difficult to train for tasks in which there is a dearth of good quallity annotated data. Historically, pre-trained word embeddings have been used for such cases but the word-level information in itself is sometimes not enough for many of the complex tasks.

## How?
The goal of the model is to learn a universal representation that transfers with little adaptation to a wide range of tasks. The paper assumes access to a large corpus of unlabeled text and several datasets with manually annotated training examples (the target tasks). The unlabeled corpus and the annotated datasets need not be in the same domain.

A two-stage training procedure is used. First, a language modeling (LM) objective is used on the unlabeled data to learn the initial parameters of the model. Next, these parameters are adapted to a target task using the corresponding supervised objective. 

A Tansformer (specifically a Transfomer decoder) is used as the underlying architecture. Transformers work better than LSTMs (shown in the results as well) because they can capture long-term dependencies well which results in robust transfer performance across diverse tasks.  Furthermore, during the transfer, as mentioned above, task-specific input adaptations are used which process the structured text input as a single contiguous sequence of tokens. This is something very interesting, and will be shown in the subsequent sections.

### Unsupervised pre-training
A standard forward LM objective is used to maximise the likelihood -

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/unsupervised-lm.PNG" caption="" >}}

Here, , *U* is the corpus of tokens {*u*<sub>1</sub>,... *u*<sub>n</sub>}, *k* is the context window size and the conditional probability *P*  is modeled using a network with parameters &Theta;. SGD is used to learn the parameters. The model uses a multi-layer Transformer decoder. The multi-head self-attention is appled over the input context tokens. This is followed by a position-wise feedforward layers to produce a oouput probability distribution over the target tokens.

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/probcalc.PNG" caption="" >}}

Here *U* is (u<sub>-k</sub>,..., u<sub>-1</sub>) which is the context vector of tokens, *n* is the number of layers, *W*<sub>e</sub> is the token embedding matrix and *W*<sub>p</sub> is the position embedding matrix.


### Supervised fine-tuning
After the training of the model with optimization *L*<sub>1</sub>, the parameters are now adapted to the supervised target task. The labeled dataset is denoted by *C*, where each instance is a sequence of input tokens, *x*<sup>1</sup>,...,*x*<sup>m</sup>, along with a label *y*. The inputs are passed through the pre-trained model to obtain the final transformer block's activation *h*<sub>l</sub><sup>m</sup>, which is then fed into an added linear output layer with parameters *W*<sub>y</sub> to predict *y*.

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/fintune1.PNG" caption="" >}}

The objective to be maximized is as follows

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/fintune2.PNG" caption="" >}}

Using a LM objective as an auxilliary objective to the finetuning helped to improve the generalization of the supervised model and make it converge faster.

The overall objective can be written as -

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/objective-fin.PNG" caption="" >}}

### Task-specific input transformations

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/input-transform.PNG" caption="" >}}

Since the pretrained model is trained on contiguous sequence of texts, to handle the inputs of the various tasks, certain input transformations is needed as shown above. These transformations help to avoid making extensive changes to the architecture across tasks.

* **Textual Entailment** - The premise (*p*) and the hypothesis (*h*) sequences are concatenated with a delimiter token in between.
* **Similarity** - Since there is no inherent ordering of the two sequences being compared, the input sequence is modified to contain both possible sentence orderings (with a delimiter in between). Each of these concatentaed sequences is processed independently to produce two sequence representartions *h*<sub>l</sub><sup>m</sup> which are then element-wise added before feeding to the linear output layer.
* **Question Answering** - This one is interesting. For a given context document *z*, question *q* and a set of possible answers {*a*<sub>k</sub>}. The document and question is concatenated with each of the possible answers, with a delimiter token in between [*z*; *q*;$;*a*<sub>k</sub>]. Each of these sequences are processed independently by the model and then normalized by a softmax layer to produce an output distribution over possible answers.

The model specifications for the experiment setup are shown below -

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/setup.PNG" caption="Experimental Setup" >}}

## Results

The datasets that were used are listed below - 

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/datasets.PNG" caption="" >}}

* **Natural Language Inference** - This task is challenging due to the presence of a wide variety of phenomena like lexical entailment, coreference, and lexical and syntactic ambiguity. The model performs better than the state-of-the-art in 4 (MNLI, QNLI, SNLI, SciTail) out of 5 datasets. 

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/nli.PNG" caption="" >}}


* **Question Answering and Commonsense Reasoning** - The RACE dataset (passages with associated questions from middle and high school exams) and Story Cloze dataset (selecting correct ending to multi-sentence stories from two options) were used. The model outperformed on the basline on both these datasets.

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/qa.PNG" caption="" >}}

* **Semantic Similarity** - The challenges in this task are recognizing reprhasing, negation, and handling ambiguity. The model performs better on 2 (QQP and STS-B) of the 3 datasets.

* **Classification** - The model performs better on both Corpus of Linguistic Accepttability (CoLA) dataset and is at-par with the state-of-the-art results on the Stanford Sentiment Teebank dataset.

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/classification.PNG" caption="" >}}

Key points from the analysis section -   
* More the number of layers that are transferred from the pretrained model to the supervised target task, the better is the performance on the target tasks. 
* To understand whether the unsupervised pre-training is effective or not, zero shot testing was also perfomed i.e., using the pre-trained model directly without any finetuning. The model performance is stable with and staedily increases over training suggesting that the generative pre-training supports the learning of a wide variety of task relevant functionality. LSTMs exhibit higher variance in its zero-shot performance.
The testing and input transformations for using the pretrained model directly are explained below -
{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/zeroshot.PNG" caption="" >}}

{{< figure src="/post/2021-05-02_language_understanding_generative_pretraining/images/trend.PNG" caption="" >}}

* From the ablation studies, the authors show that the auxiliary LM objective helps on the NLI tasks and QQP (Quora Question Pairs data).
* Overall, larger datasets benefit from the auxiliary objective more than the samller datasets.
* In general, the Transformer architecture performs better than a 2048 unit single layer LSTM model (if the Transformer in the pretraining model is replaced by a LSTM) on all datasets except the MRPC (Microsoft Paraphrase Corpus for semantic similarity) dataset.
* On comparing this model with the same transformer architecture trained in a supervised manner, it is observed that the model with pre-training performs better. This consistent for all the tasks mentioned in the paper, suggesting that pre-training helps to capture important linguistic information which is not captured when training with a supervised approach alone.

-------

**I have also released an annotated version of the paper. If you are interested, you can find it [here](https://github.com/shreyansh26/Annotated-ML-Papers/blob/main/GPT1.pdf).**

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

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).