---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "PPML Series #1 - An introduction to Federated Learning"
subtitle: "A short general introduction to Federated Learning (FL) for folks interested in privacy-preserving machine learning (PPML)."
authors: ["Shreyansh Singh"]
tags: [federated-learning, ppml, privacy]
categories: [Machine Learning]
date: 2021-12-11T16:17:16+05:30
lastmod: 2021-12-11T16:17:16+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Source: https://bit.ly/3ykHlEP"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: [Federated Learning]
---

-------

## Motivation

Privacy-preserving Machine Learning had always been exciting for me. Since my B.Tech. thesis involving PPML (SMPC + Computer Vision), I didn't get a chance to work on it after that. So, after about 2 years, I have started to read about it again, and sharing it with the community.

Federated Learning is a domain that I had somewhat eluded during my thesis. I had some idea about the topic but didn't get into it much. So, I decided to start with FL this time. There is a ton of literature out there and is a field of active interest right now.

## Introduction
Modern mobile devices have abundance of data, majorly textual data, image data. Applying machine learning to these can definitely help improve user experience. For example - your mobile keyboard uses language models can improve speech recognition and text entry, your photos apps (Google Photos, say) has image models that can automatically select good photos.

However, we have two problems here. Firstly, if we consider millions of devices, then this data is large in quantity. Secondly, this dataset may have personal pictures, textual information written by the device owners,among other things.  Hence, this data is privacy sensitive in most cases. This creates problems to store this data in a database. It can be both infeasible as well as cause privacy violations. 

*Federated Learning* is a decentralised machine learning approach which allows to leave the training data on the individual devices and learns a shared model by aggregating locally computed gradient updates. Federated Learning can significantly reduce privacy and security risks by limiting the attack surface to only the device, rather than the device and the cloud. Obviously, some level of trust on the server coordinating the training is still required.

## Why FL?

Federated Learning usually helps in three contexts - 

- Training on real-world data from mobile devices provides advantage over training on proxy data stored in data centres. The distributions from which these examples are drawn are likely to differ substantially from easily available proxy datasets: the use of language in chat and text messages is generally much different than standard language corpora, e.g., chat messages are not like Wikipedia articles. Images taken through the camera are also not like Flickr images.
- The  data is privacy sensitive or large in size (compared to the size of the model), so it is preferable not to log it to the data centre purely for the purpose of model training.
- Sometimes, labels for tasks can naturally be obtained from user interaction. Entered text is self-labeled for learning a language model, and photo labels can be defined by natural user interaction with their photo app (which photos are deleted, shared, or viewed).

Applications of FL could include image classification, predicting which images will be viewed multiple times in the future, language modelling, next word/phrase prediction.

### How does FL provide privacy (upto a certain extent)?

Handling even anonymized data can lead to privacy concerns. What better way than to use the data itself to train the models but at the same time, not risk its privacy. In contrast, the information transmitted for federated learning is the minimal update necessary to improve a particular model. They will generally contain much less information about the raw data. Further, the source of the updates is not needed by the aggregation algorithm, so updates can be transmitted without identifying meta-data over a mix network such as Tor or trusted third-parties.

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


## Federated Optimization

The optimization problem implicit in federated learning as federated optimization, drawing a connection (and contrast) to distributed optimization.

### How is FL different from any other distributed optimization problem?

In federated optimization, there are a few key properties that differentiate from a typical distributed optimization problem.

- **Non-IID** - Training data will vary from user to user and will not have properties that are similar to the population.
- **Unbalanced** - Some users use a device more and generate more data, some less.
- **Massively distributed** - Number of users are much more than the average number of examples per client.
- **Limited communication** - Devices are frequently offline and are on slow or expensive connections.

### How to perform Federated Optimization?

In this blog post, I won't go into the mathematical details regarding the optimization techniques used in Federated Learning. However, we will be discussing the high-level overview of how training is performed. The steps are as follows - 

- There is a fixed set of *K* clients, each with a fixed local dataset.
- At the beginning of each round, a random fraction *C* of clients is selected, and the server sends the current global algorithm state to each of these clients (e.g., the current model parameters). 
- Only a fraction of clients is selected for efficiency, as experiments show diminishing returns for adding more clients beyond a certain point. 
- Each selected client then performs local computation based on the global state and its local dataset, and sends an update to the server. 
- The server then applies these updates to its global state, and the process repeats.


## Challenges

In general, for ML tasks, and in data centres, the costs of compute is what is the most important - GPUs are used to lower the computation cost.
In Federated Learning, the communication costs somewhat dominate. 

But what do communication costs mean here?

- Upload bandwidths in mobiles (globally) is limited to 1 MB/s or less. 
- Clients volunteer only if the charged, plugged-in and on free/unmetered WiFi connections.
- Each client participates in only a small number of rounds per day. 

In Federated Learning, computation is not much of an issue because the dataset size on each device will be relatively much less and modern smartphones now have processors fast enough to do those computations locally. So, the goal becomes to use additional computation in order to decrease the number of rounds of communication needed to train a model.

So, the goal becomes to use additional computation in order to lower the number of rounds of communication needed to train the model.

There are two approaches which can be adopted for this -
- *Increased parallelism* -  More clients are used which work independently between each communication round.
- *Increased computation on each client* - Rather than performing a simple computation like gradient calculation, each client performs a more complex calculation between each communication round.

In practice, major speedups are obtained when computation on each client is improved, once a minimum level of parallelism over clients is achieved.

-------

This is all for now. Thanks for reading!

In my next post, I'll share a mathematical explanation as to how optimization (learning) is done in a Federated Learning setting. I will also explain some experimental results that have been published.

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
