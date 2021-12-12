---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "PPML Series #2 - Federated Optimization Algorithms - FedSGD and FedAvg"
subtitle: "A mathematical deep dive on a Federated Optimization algorithm - FedAvg and comparing it with a standard approach - FedSGD."
authors: ["Shreyansh Singh"]
tags: [federated learning, ppml]
categories: [Machine Learning]
date: 2021-12-12T00:16:23+05:30
lastmod: 2021-12-12T00:16:23+05:30
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
projects: []
---
In my last post, I covered a high-level overview of Federated Learning, its applications, advantages & challenges.

We also went through a high-level overview of how Federated Optimization algorithms work. But from a mathematical sense, how is Federated Learning training actually performed? That's what we will be looking at in this post.

------

There was a [paper](https://arxiv.org/abs/1602.05629), Communication-Efficient Learning of Deep Networks from Decentralized Data by Google (3637 citations!!!), in which the authors had proposed a federated optimization algorithm called FedAvg and compared it with a naive baseline, FedSGD.

## FedSGD 

Stochastic Gradient Descent (SGD) had shown great results in deep learning. So, as a baseline, the researchers decided to base the Federated Learning training algorithm on SGD as well. SGD can be applied naively to the federated optimization problem, where a single batch gradient calculation (say on a randomly selected client) is done per round of communication.

The paper showed that this approach is computationally efficient, but requires very large numbers of rounds of training to produce good models.

Before we get into the maths, I'll define a few terms - 

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/Latex-FL.PNG" caption="" >}}

The baseline algorithm, was called FedSGD, short for Federated SGD. 

For FedSGD, the parameter *C* (explained above) which controls the global batch size is set to 1. This corresponds to a full-batch (non-stochastic) gradient descent. For the current global model <i>w<sup>t</sup></i>, the average gradient on its global model is calculated for each client *k*.

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/FedSGD-updates.PNG" caption="" >}}

The central server then aggregates these gradients and applies the update.

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/FedSGD-updates2.PNG" caption="" >}}


## FedAvg

We saw FedSGD. Now let's make a small change to the update step above. 

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/FedAvg-updates.PNG" caption="" >}}

What this does is that now each client locally takes one step of gradient descent
on the current model using its local data, and the server then takes a weighted average of the resulting models.

This way we can add more computation to each client by iterating the local update multiple times before doing the averaging step. This small modification results in the FederatedAveraging (FedAvg) algorithm.

But why make this change? The answer is in my last post - 

> In practice, major speedups are obtained when computation on each client is improved, once a minimum level of parallelism over clients is achieved. 

The amount of computation is controlled by three parameters -

**C** - Fraction of clients participating in that round
**E** - No. of training passes each client makes over its local dataset each round
**B** - Local minibatch size used for client updates

The pseudocode for the FedAvg algorithm is shown below. 

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/FedAvg-algo.PNG" caption="" >}}


B = ꝏ (used in experiments) implies full local dataset is treated as the minibatch. So, setting B = ꝏ and E = 1 makes this the FedSGD algorithm.

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

## Results

Okay, now let's look at some experimental results, although I would also suggest looking up the results from the original paper as well.
One experiment showed the number of rounds required to attain a target accuracy, in two tasks - MNIST and a character modelling task.

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/res1.PNG" caption="" >}}

Here, IID and non-IID here refer to the datasets that were artificially generated by the authors to represent two kinds of distributions - IID, in which there is in fact an IID distribution among the clients. And non-IID in which the data is not IID among the clients. For example, for the MNIST dataset the authors studied two ways of partitioning the MNIST data over clients: IID, where the data is shuffled, and then partitioned into 100 clients each receiving 600 examples, and Non-IID, where we first sort the data by digit label, divide it into 200 shards of size 300, and assign each of 100 clients 2 shards. For the language modeling task, the dataset was built from *The Complete Works of William Shakespeare*. From the paper - 

> We construct a client dataset for each speaking role in each play with at least two lines. This produced a dataset with 1146 clients. For each client, we split the data into a set of training lines (the first 80% of lines for the role), and test lines (the last 20%, rounded up to at least one line). The resulting dataset has 3,564,579 characters in the training set, and 870,014 characters in the test set. This data is substantially unbalanced, with many roles having only a few lines, and a few with a large number of lines. Further, observe the test set is not a random sample of lines, but is temporally separated by the chronology of each play. Using an identical train/test split, we also form a balanced and IID version of the dataset, also with 1146 clients.

For the MNIST dataset, a CNN with two 5x5 convolution layers (the first with 32 channels, the second with 64, each followed with 2x2 max pooling), a fully connected layer with 512 units and ReLu activation, and a final softmax output layer was used. And for the language modeling task, a stacked character-level LSTM language model, which after reading each character in a line, predicts the next character. The model takes a series of characters as input and embeds each of these into a learned 8 dimensional space. The embedded characters are then processed through 2 LSTM layers, each with 256 nodes. Finally the output of the second LSTM layer is sent to a softmax output layer with one node per character. The full model has 866,578 parameters, and we trained using an unroll length of 80 characters.

From the results in the paper, it could be seen that in both the IID and non-IID settings, keeping a small mini-batch size and higher number of training passes on each client per round resulted in the model converging faster. For all model classes, FedAvg converges to a higher level of test accuracy than the baseline FedSGD models. For the CNN, the B = ꝏ; E = 1 FedSGD model reaches 99.22% accuracy in 1200 rounds, while the B = 10 ;E = 20 FedAvg model reaches an accuracy of 99.44% in 300 rounds.

{{< figure src="/post/2021-12-12_federated_optimization_fedavg/images/res2.PNG" caption="" >}}

The authors also hypothesise that in addition to lowering communication costs, model averaging produces a regularization benefit similar to that achieved by dropout. 

All in all, the experiments demonstrated that the FedAvg algorithm was robust to unbalanced and non-IID distributions, and also reduced the number of rounds of communication required for training, by orders of magnitude.

----

I wrote a Twitter thread on this topic as well - do give it a like/follow me if you liked the article. 

{{< tweet user="shreyansh_26" id="1463454860460785670" >}}

That's the end for now!

This post finishes my summary on the basics of Federated Learning and is also a concise version of the very famous paper "Communication-Efficient Learning of Deep Networks from Decentralized Data" by Google.

**I have also released an annotated version of the paper. If you are interested, you can find it [here](https://github.com/shreyansh26/Annotated-ML-Papers/blob/main/PPML/Federated%20Learning/Communication-Efficient%20Learning%20of%20Deep%20Networks%20from%20Decentralized%20Data.pdf).**

If the post helps you or you have any questions, do let me know! 

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
