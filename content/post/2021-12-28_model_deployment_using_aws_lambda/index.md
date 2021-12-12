---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Deploying Machine Learning models using AWS Lambda and Github Actions - A Detailed Tutorial"
subtitle: "A step-wise tutorial to demonstrate the steps required to deploy a ML model using AWS Lambda, Github Actions, API Gateway and use Streamlit to access the model API through a UI. "
summary: ""
authors: ["Shreyansh Singh"]
tags: [model deployment, aws, github actions, api, streamlit]
categories: [Machine Learning]
date: 2021-12-12T23:15:03+05:30
lastmod: 2021-12-12T23:15:03+05:30
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
projects: [Model Deployment]
---

------

Quite a while back, I had written a [post](https://shreyansh26.github.io/post/2020-11-30_fast_api_docker_ml_deploy/) in which I described how to package your Machine Learning models using Docker and deploy them using Flask. 

This post, through a PoC, describes -

1. How to package your model using Docker (similar as last [post](https://shreyansh26.github.io/post/2020-11-30_fast_api_docker_ml_deploy/))
2. How to push the Docker container to Amazon ECR 
3. Add a Lambda Function for your model
4. Make a REST API using Amazon API Gateway to access your model
5. Automate the whole process using Github Actions, so that any updates to the model can take effect immediately.
6. Make a Streamlit app to make a UI to access the REST API (for the model deployed on AWS).

**All the code can be found in my [Github repository](https://github.com/shreyansh26/Iris_classification-AWS-Lambda-PoC).**

The repository also contains the code to train, save and test a simple ML model on the Iris Dataset. 

The Iris dataset is a small dataset which contains attributes of the flower - Sepal length, Sepal width, Petal length and Petal width.
The goal of the task is to classify based on these dimensions, the type of the Iris, which in the dataset is among three classes - Setosa, Versicolour and Virginica.


## Package Requirements
* An Amazon Web Services account (I intentionally use a simple ML model to deploy as it remains in the AWS Free tier constraints across all the services I mention above. Larger models will require more storage and hence could be chargeable.)
* Python 3.6+
* A simple 
`pip install -r requirements.txt` from the [https://github.com/shreyansh26/Iris_classification-AWS-Lambda-PoC/tree/master/iris_classification](iris_classification) directory will install the other Python packages required.


## Steps to follow
In this PoC, I will be training and deploying a simple ML model. If you follow this tutorial, deploying complex models should be fairly easy as well. (I had to scratch my head a lot though!)

### 1. Training and Deploying the model locally

1. Clone this repo
```
git clone https://github.com/shreyansh26/Iris_classification-AWS-Lambda-PoC
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

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/ini-streamlit.PNG" caption="" >}}

Right now, the `Predict AWS` button will give an error on clicking. It is required to set up an API of your own that the code will send the POST request to.

A `main.py` file contains the event handler which will be used by Lambda later.

### 2. Packaging the model
I have included a Dockerfile which is used to package the model. Later I will automate all this using Github Actions.

```
cd iris_classification
docker build --tag iris_classification:latest .
```

### 3. Push the Docker container to Amazon ECR
First, create a private repository. The free tier only allows for 500MB of storage in a month in a private repository.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/ecr1.PNG" caption="" >}}

Use the following set of commands to push the local Docker container to the created repository.

```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 863244415814.dkr.ecr.us-east-1.amazonaws.com

docker tag iris_classification:latest 863244415814.dkr.ecr.us-east-1.amazonaws.com/iris_classification:latest

docker push 863244415814.dkr.ecr.us-east-1.amazonaws.com/iris_classification:latest
```

You may have to run 

```
aws configure
```
and provide your AWS Access Key ID and your AWS Secret Access Key to run the above commands successfully. 

### 4. Create a Lambda function

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/lambda1.PNG" caption="" >}}

The container image URI can be selected from the AWS console itself.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/lambda2.PNG" caption="" >}}

### 5. Test the Lambda

We can now test that the Lambda is correctly handling the request as we want it to. AWS allows for that. When we click on the Lambda function, it allows a Test option as well.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/lambda3.PNG" caption="" >}}

The test works and gives the correct result!!

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/lambda4.PNG" caption="" >}}

### 6. Create an API from the Amazon API Gateway

Make sure to make a REST API.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/api1.PNG" caption="" >}}

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/api2.PNG" caption="" >}}

Add a `/classify` resource to the API and and add a `POST` method to the API.
Add a POST request to the API under a `/classify` resource.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/api3.PNG" caption="" >}}

Integrate the Lambda function with the API.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/api4.PNG" caption="" >}}

Now, if you head back to the Lambda functions page, you will see that a Trigger has been added to the function.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/api5.PNG" caption="" >}}

The endpoint is clearly visible in the screenshot.
It will be something like `https://{SOME_ID}.execute-api.us-east-1.amazonaws.com/test/classify`.


### 7. Test the REST API

We use a client like Postman to check the API.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/postman.PNG" caption="" >}}

#### AND IT WORKS!

Programmatically, we can also check that the API works.

```python
import requests

url = 'https://ti53furxkb.execute-api.us-east-1.amazonaws.com/test/classify'

myobj = {
    "data": [
        [6.5, 3.0, 5.8, 2.2],
        [6.1, 2.8, 4.7, 1.2]
    ]
}

x = requests.post(url, json = myobj)

print(x.text)
```

```
{"prediction": ["virginica", "versicolor"], "log_proba": [[-35.82910355985537, -1.5907654693356144, -0.22786665344763715], [-26.20011949521101, -0.0783441410298827, -2.585560434227453]]}
```

#### THIS WORKS TOO!

&nbsp;

## Streamlit app to test the model

After making the appropriate changes to the configuration, running

```
streamlit run app.py
```

allows you to get the predictions from the AWS hosted model as well.

{{< figure src="/post/2021-12-28_model_deployment_using_aws_lambda/images/fin-streamlit.PNG" caption="" >}}


## Time to automate the whole thing using Github Actions

We use Github Actions to automate this whole process i.e. pushing the container to ECR, updating the Lambda function. The API then points to updated Lambda function automatically.

First, we will need to add the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` to Github secrets (in the Github repo settings).

You can refer to the yml file in [.github/workflows](https://github.com/shreyansh26/Iris_classification-AWS-Lambda-PoC/tree/master/.github/workflows) to see how the automation works. The Github Action is triggered when a pull request is made to the `master` branch.

If required, you can also restrict any pushes to the master branch from Github ([link](https://stackoverflow.com/questions/46146491/prevent-pushing-to-master-on-github)).


### AND WE ARE DONE!

-----

That's all for now!
I hope the tutorial helps you deploy your own models to AWS Lambda easily. Make sure to read the pricing for each AWS product you use to avoid being charged unknowingly.

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