---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Deploying Machine Learning models using GCP's Google AI Platform - A Detailed Tutorial"
subtitle: "A step-wise tutorial to demonstrate the steps required to deploy a ML model using GCP, specifically the Google AI Platform and use Streamlit to access the model through a UI."
authors: ["Shreyansh Singh"]
tags: [model-deployment, gcp, streamlit, postman]
categories: [Machine Learning]
date: 2022-03-06T00:37:30+05:30
lastmod: 2022-03-06T00:37:30+05:30
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

----

In my [last post](https://shreyansh26.github.io/post/2022-01-23_model_deployment_using_aws_lambda/) I had written about deploying models on AWS. So, I though it would only be fitting to write one for GCP, for all the GCP lovers out there.

GCP has a service called the AI Platform which, as the name suggest, is responsible for training and hosting ML/AI models.

Just like the last post, this post, through a PoC, describes -

1. How to add a trained model to a Google Cloud bucket
2. Host the saved model on the AI Platform
3. Create a Service Account to use the model hosted on AI Platform externally
3. Make a Streamlit app to make a UI to access the hosted model


**All the code can be found in my [Github repository](https://github.com/shreyansh26/Iris_classification-GCP-AI-Platform-PoC).**

The repository also contains the code to train, save and test a simple ML model on the Iris Dataset. 

The Iris dataset is a small dataset which contains attributes of the flower - Sepal length, Sepal width, Petal length and Petal width.
The goal of the task is to classify based on these dimensions, the type of the Iris, which in the dataset is among three classes - Setosa, Versicolour and Virginica.

## Package Requirements
* A Google Cloud account and a Google Cloud Project (using GCP will cause money if you don't have any of the free $300 credits you get when you first sign up)
* Python 3.6+
* A simple 
`pip install -r requirements.txt` from the [iris_classification](https://github.com/shreyansh26/Iris_classification-GCP-AI-Platform-PoC/tree/master/iris_classification) directory will install the other Python packages required.

## Steps to follow
In this PoC, I will be training and deploying a simple ML model. If you follow this tutorial, deploying complex models should be fairly easy as well.

### 1. Training and Deploying the model locally

1. Clone this repo
```
git clone https://github.com/shreyansh26/Iris_classification-GCP-AI-Platform-PoC
```

2. Create a virtual environment - I use [Miniconda](https://docs.conda.io/en/latest/miniconda.html), but you can use any method (virtualenv, venv)
```
conda create -n iris_project python=3.8
conda activate iris_project
```

3. Install the required dependencies
```
pip install -r requirements.txt
```

4. Train the model
```
cd iris_classification/src
python train.py
```

3. Verify the model trained correctly using pytest
```
pytest
```

4. Activate Streamlit and run `app.py`
```
streamlit run app.py
```

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/ini-streamlit.PNG" caption="" >}}

Right now, the `Predict GCP` button will give an error on clicking. It requires a json configuration file which we will obtain when we deploy our model. To get the `Predict AWS` button working for your model, refer to a separate [tutorial](https://shreyansh26.github.io/post/2021-12-28_model_deployment_using_aws_lambda/) I made on that.


### 2. Storing the model in a GCP Bucket
The saved `model.pkl` has to be stored in a Google Storage Bucket. First, create a bucket.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/gcp-bucket.PNG" caption="" >}}

The rest of the inputs can be kept as default. 

And then upload the `model.pkl` to the bucket.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/bucket-upload.PNG" caption="" >}}

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

### 3. Hosting the model on AI Platform
Using the AI Platform, we need to create a model

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/aiplatform-models.PNG" caption="" >}}

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/aiplatfrom-create.PNG" caption="" >}}

Next, create a version of the model.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/version.PNG" caption="" >}}

Choose the bucket location which has the `model.pkl` file.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/version2.PNG" caption="" >}}

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/version3.PNG" caption="" >}}

The model will take some time to be hosted.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/version4.PNG" caption="" >}}

### 4. Creating a Service Account

Finally, head to `IAM -> Service Accounts` and add a Service Account which basically allows to use the model hosted on AI Platform externally.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/service.PNG" caption="" >}}

Next, select `AI Platform Developer` as the role and click `Done`.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/service2.PNG" caption="" >}}

Now, in the `Service Accounts` console, we see that there are no keys. Yet.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/service3.PNG" caption="" >}}

We go to `Manage Keys`

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/service4.PNG" caption="" >}}

Creating the key downloads a JSON file which basically has the key our code will be using.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/service5.PNG" caption="" >}}


The following configurations should be updated in the `app.py` file.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/code.PNG" caption="" >}}

## Testing the hosted model

After making the appropriate changes to the configuration, running

```
streamlit run app.py
```

allows you to get the predictions from the GCP hosted model as well.

{{< figure src="/post/2022-03-06_model_deployment_using_gcp_google_ai_platform/images/fin-streamlit.PNG" caption="" >}}


### AND WE ARE DONE!

Hope this gives you a good idea on how to deploy ML models on GCP. Obviously, there can be extensions which can be done. 

* Github Actions could be used to automate the whole deployment process. 
* Google App Engine could be used to deploy and host the Streamlit app.

----

That's all for now!
I hope this tutorial helps you deploy your own models to Google Cloud Platform easily. Make sure to read the pricing for each GCP product (if you are not using the initial free credits) you use to avoid being charged unknowingly.

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