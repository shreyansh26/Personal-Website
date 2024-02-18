---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Summary #12 - Image Recaptioning in DALL-E 3"
subtitle: "The image recaptioning technique used in DALL-E 3 was extended to videos in Sora."
summary: ""
authors: ["Shreyansh Singh"]
tags: [image captioning, generative model]
categories: [Machine Learning]
date: 2024-02-18T19:56:44+05:30
lastmod: 2024-02-18T19:56:44+05:30
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

**Technical Paper**: [Improving Image Generation with Better Captions](https://cdn.openai.com/papers/dall-e-3.pdf)  

OpenAI's Sora is built upon the image captioning model which was described in quite some detail in the DALL-E 3 technical report.

-----

In general, in text-image datasets, the captions omit background details or common sense relationships, e.g. sink in a kitchen or stop signs along the road. They also omit the position and count of objects in the picture, color and size of the objects and any text present in the image.

OpenAI trained a captioner model to solve this.

An image captioner is similar to a language model that predicts the next token conditioned on the image and the past generated tokens. Since images are composed of many thousands of pixel values, conditioning on all of this information is very complex and inefficient. The solution DALL-E 3 used was to use CLIP's compressed representational space to condition upon. OpenAI jointly pre-trained a captioner with a CLIP and a language modeling objective using this formulation on the text and image pairs.

{{< figure src="/post/2024-02-18_dalle3_image_recaptioner/images/lm.png" caption="" >}}

However, this still doesn't solve the reluctance of the captioning model to provide details, as mentioned above. The fine-tuning of this captioner to be more descriptive and helpful for the text-to-image task is done in two phases.

First, they built a small dataset of captions that describe only the main subject of the image. The captioner is fine-tuned on this dataset. Now the model is more biased towards describing the main subject of the image.

Next, they created a dataset of long, highly-descriptive captions describing the contents of each image in the fine-tuning dataset. In addition to the main subject, these captions describe the surroundings, background, text found in the image, styles, coloration, etc. as well. The captioner is then fine-tuned on this dataset to generate descriptive synthetic captions.

{{< figure src="/post/2024-02-18_dalle3_image_recaptioner/images/synthetic_captions.png" caption="Examples of alt-text accompanying selected images scraped from the internet, short synthetic captions (SSC), and descriptive synthetic captions (DSC)." >}}

**There is a still a problem though...**

Text-to-Image diffusion models have a tendency to overfit to the text distribution. Using synthetic captions can lead to such issues since the captioner model can have many modal behaviours that won't be apparent but can bias the text-to-image model when trained on the synthetic captions.  
Examples of where this might occur is in letter casing, where punctuation appears in the caption (e.g. does it always end with a period?), how long the captions are, or stylistic tendencies such as starting all captions with the words "a" or "an".

OpenAI overcame this issue by regularizing the text inputs to a distribution that is similar to the humans. The ground-truth captions provide this already since they were drawn from a distribution of human-written text. The regularization can be introduced during training the model by randomly selecting either the ground truth or synthetic caption with a fixed percent chance. DALLE-3 used 95% synthetic captions and 5% ground truth captions.

{{< figure src="/post/2024-02-18_dalle3_image_recaptioner/images/perf.png" caption="CLIP scores for text-to-image models trained on different caption types. Left is evaluation results with ground truth captions on our evaluation dataset. Right uses the descriptive synthetic captions from the same dataset" >}}

{{< figure src="/post/2024-02-18_dalle3_image_recaptioner/images/synthetic_ratio.png" caption="CLIP scores for text-to-image models trained on various blending ratios of descriptive synthetic captions and ground-truth captions. Evaluation performed using ground truth captions." >}}

The above figure shows that using very high percentage of synthetic captions maximizes the performance of the models. But increasing the synthetic caption ratio implies biasing the model to the distribution of long, highly-descriptive captions emitted by the captioning model. OpenAI used GPT-4 to upsamples any caption into a highly descriptive one.

Here are some examples -

{{< figure src="/post/2024-02-18_dalle3_image_recaptioner/images/gpt_upsample.png" caption="Effect of using 'upsampled' drawbench captions to create samples with DALL-E 3. Original drawbench captions on top, upsampled captions on bottom. Images are best of 4 for each caption." >}}

Below is the prompt OpenAI used to "upsample" the captions.

{{< figure src="/post/2024-02-18_dalle3_image_recaptioner/images/upsample_prompt.png" caption="" >}}

------

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