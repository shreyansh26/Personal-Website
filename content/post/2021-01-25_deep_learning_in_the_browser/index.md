---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Deep Learning in the Browser - Exploring TF.js, WebDNN and ONNX.js"
subtitle: "Just a quick tutorial to set up a small scale deployment for your ML or DL model"
authors: ["Shreyansh Singh"]
tags: [deep learning, machine learning, model-deployment, web]
categories: [Machine Learning]
date: 2021-01-25T12:53:13+05:30
lastmod: 2021-01-25T12:53:13+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Tensorflow.js, WebDNN, ONNX.js"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

After my [last post](/post/2020-11-30_fast_api_docker_ml_deploy) on deploying Machine Learning and Deep Learning models using FastAPI and Docker, I wanted to explore a bit more on deploying deep learning models. My last post discussed a server-side method for deploying the model. This post will discuss client side frameworks and techniques to deploy those models such that they work directly on the client side.

In this tutorial I will be giving an overview of three frameworks, [Tensorflow.js](https://www.tensorflow.org/js), [WebDNN](https://mil-tokyo.github.io/webdnn/) and [ONNX.js](https://microsoft.github.io/onnxjs-demo/#/). I will be a deploying a simple pretrained image classification model (ResNet or Mobilenet) on the three frameworks and also tell you the comparsion between them. In this tutorial, I haven't deployed custom models of my own but I will be explaining how you can do it and the difficulties you could encounter.

The goal of this blog post is to introduce the three frameworks and how you can use them for deploying your models as well. Personally, I had not heard of WebDNN and ONNX.js before diving into this project, so I believe it can help some others like me to get familiar with these frameworks.


## Tensorflow.js

I found Tensorflow.js to be the easiest to use. It already has a large collection of some [pretrained models](https://github.com/tensorflow/tfjs-models). With Tensorflow.js, we don't have a pretrained Resnet model because it is not exactly a lightweight model that can be deployed on a device with low compute power. So, I used [Mobilenet](https://arxiv.org/abs/1704.04861) (which is trained on the Imagenet dataset). Mobilenet was available in the Tensorflow.js pretrained models repository so I decided to use that directly.

Now, on to the fun part, actually using the model and making a webapp. For the webapp portion, I am using [Express](https://expressjs.com/), a web framework for Node.js. I have tried to keep the code structure and the webapp visually similar for all the three frameworks.

Loading the model is as simple as -

{{< gist shreyansh26 dfedd9a445841a8bb963af9526a9f21c >}}

Now after loading the model, we call the `imgSet` function which bascially loads the image from the path we specify and loads it onto a canvas. Details of this can be seen in the code which I will post at the end. 

Although the Mobilenet model in Tensoflow.js doesn't require a fixed size of the image, but for uniformity in all other frameworks (WebDNN, ONNX.js), I decided to resize the images to 224x224 size. The main code for running the model is shown below - 

{{< gist shreyansh26 3b1bc92aa52a13cabae2f426b36c2576 >}}

The final webapp looks something like this - 

{{< figure src="/post/2021-01-25_deep_learning_in_the_browser/images/tfapp.gif" caption="Image loading and prediction" >}}

The model works well. It knows it is some kind of a water related animal, and given the Imagenet classes it has been trained on, it gies the closest result possible. 

The first prediction takes time (196ms) because the model is loaded and run for the first time. After that, the predictions take very little time (~80ms) mainly because the model is cached and predictions can be served faster.

The average time taken by different backends (over 20 predictions) is also shown below - 

| Backend     | Time |
| ----------- | ----------- |
| cpu         | 2100ms      |
| wasm        | 82ms        |
| webgl       | 70ms        |


If one wants to convert their own models to a Tensorflow.js compatible version, it is very easy to convert the model as well as load it into your web application. One can refer to [tfjs-converter](https://github.com/tensorflow/tfjs/tree/master/tfjs-converter) and the documentation given [here](https://www.tensorflow.org/js/guide/conversion).

The code for this section is present [on my Github](https://github.com/shreyansh26/DeepLearning-in-the-Browser/tree/main/TF).

## WebDNN

[WebDNN](https://mil-tokyo.github.io/webdnn/) was developed by the Machine Intellignece Laboratory at the University of Tokyo. From its website, 

> WebDNN optimizes the trained DNN model to compress the model data and accelerate the execution, and executes it with novel JavaScript API such as WebAssembly and WebGPU to achieve zero-overhead execution. WebDNN supports 4 execution backend implementations: WebMetal, WebGL, WebAssembly, and fallback pure javascript implementation. By using these backends, WebDNN works all major browsers.

More details are available on the website, but the image below accurately depicts the steps involved in this procedure.

{{< figure src="/post/2021-01-25_deep_learning_in_the_browser/images/webdnn-arch.PNG" caption="WebDNN model conversion flow" >}}

WebDNN can be used to deploy tarined DNN models trained using popular DL frameworks like Tensorflow, Keras, PyTorch, Chainer, Kaffe. One disadvantage I found of using WebDNN is that the current model conversion module (as of writing the post) does not allow conversion using Tensorflow 2 and also does not support the latest versions of Keras (let alone `tensorflow.keras`). 

I used a pretrained ResNet50 model (trained on Imagnet dataset) for this. I am sharing the [following Colab notebook](https://colab.research.google.com/drive/1pFdbZc5_Dd78twKshl-MH8T_EuVrH0Nw?usp=sharing) which contains the code to convert the ResNet50 Keras model. 

On to the web app coding part! The first thing the webapp does is to load the model. 

{{< gist shreyansh26 a527987583919e53b237a7d1a312f3a8 >}}

Next, we write the code to run the model on the image input.

{{< gist shreyansh26 a7a9eb637f31e9dee0a2b39822ebc4b7 >}}

The final webapp looks something like this - 

{{< figure src="/post/2021-01-25_deep_learning_in_the_browser/images/webdnn.gif" caption="WebDNN predictions" >}}

The model does a very good job of identifying it is a bus. The top two predictions relate to it.

Again, the first run takes a long time (~242ms) but the subsequent runs take quite less (~63ms average). Now one must note that ResNet50 is a relatively heavier model as compared to Mobilenet, but WebDNN manages to load it much faster than or at par with Mobilenet as we saw in the case with Tensorflow.js. Also, in the COlab notebook, we can see that for the same image, the ResNet50 model around 645ms to run the model. We easily see a ~10x improvement on converting the model to WebDNN.

The average time taken by different backends (over 20 predictions) is also shown below - 

| Backend     | Time |
| ----------- | ----------- |
| cpu         | 10000ms      |
| webgl       | 60ms        |

WebDNN is quite optimised to run on futuristic hardware. The time it takes on a normal fallback vanilla-JS model version running on the CPU is around 10 seconds. But on WebGL, it takes much much less. I didn't have access to a WebMetal backend, which they claim is the fastest. I would like to know if anyone runs it on WebGPU (WebMetal) and the average time the model took to run on it.

The code for this section is present [on my Github](https://github.com/shreyansh26/DeepLearning-in-the-Browser/tree/main/WebDNN).


## ONNX

[Open Neural Network Exchange (ONNX)](https://onnx.ai/) is an open source format for AI models, both deep learning and traditional ML.

From their website -

> ONNX defines a common set of operators - the building blocks of machine learning and deep learning models - and a common file format to enable AI developers to use models with a variety of frameworks, tools, runtimes, and compilers.

[ONNX.js](https://github.com/microsoft/onnxjs) is an open source Javascript library by Microsoft for running ONNX models on browsers and on Node.js. Like Tensorflow.js and WebDNN, it also has support for WebGL and CPU. From theit Github 

> With ONNX.js, web developers can score pre-trained ONNX models directly on browsers with various benefits of reducing server-client communication and protecting user privacy, as well as offering install-free and cross-platform in-browser ML experience.

With ONNX.js, I used a pretrained [ResNet50 model](/post/2021-01-25_deep_learning_in_the_browser/files/resnet50_8.onnx). 

Loading the model is similar -

{{< gist shreyansh26 07658e9b0b4dc759fb4f081cd9ea7b78 >}}

The ONNX examples on their repository gives some nice code snippets to show basic image preprocessing. I have used it directly in my code.

{{< gist shreyansh26 1a2f6059395c60485e4d721c7afd761b >}}

After that, the following code snippet loads the preprocessed image to an input tensor and then runs the model on it and then prints the predictions.

{{< gist shreyansh26 7e4366058eac9a8c1f05a82b569ff91a >}}

A demo of the webapp using ONNX.js is shown below.

{{< figure src="/post/2021-01-25_deep_learning_in_the_browser/images/onnx1.gif" caption="ONNX.js predictions" >}}

{{< figure src="/post/2021-01-25_deep_learning_in_the_browser/images/onnx2.gif" caption="ONNX.js predictions" >}}

The Resnet model does an awesome job with the airline image and classifies it correctly. It also performs decently on the bus image giving the top prediction as *minibus*. However, the goal of this post is not to judge how well the model works, but the technique of deploying the models and receiving predictions from them.

I used the WebGL model for testing. It takes an average of 70ms to serve the predictions. The CPU version takes a VERY long time ~15000ms (15 seconds).

The average time taken by different backends (over 20 predictions) is also shown below. I had some trouble with the WASM version so I didn't include them in the results.

| Backend     | Time |
| ----------- | ----------- |
| cpu         | 15000ms      |
| webgl       | 71ms        |


The best part about ONNX is that it is an open standard and allows easy conversion of models made in different frameworks to a `.onnx` model. I would suggest going through [this tutorial](https://github.com/onnx/tutorials) for this.

The code for this section is present [on my Github](https://github.com/shreyansh26/DeepLearning-in-the-Browser/tree/main/ONNX).

## The End

That is all for now. I hope that this tutorial will help the reader get an idea of these frameworks for client-side model deployment and one can also use my code as a boilerplate for setting up webapps of your own for deploying ML models using these frameworks.


---

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
