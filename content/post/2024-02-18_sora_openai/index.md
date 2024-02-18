---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Summary #11 - Sora"
subtitle: "OpenAI announced a groundbreaking text-to-video diffusion model capable of generating high-definition videos upto 60 seconds long."
summary: ""
authors: ["Shreyansh Singh"]
tags: [diffusion, image generation, video generation, generative model]
categories: [Machine Learning]
date: 2024-02-18T19:40:18+05:30
lastmod: 2024-02-18T19:40:18+05:30
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

**Technical Paper**: [Sora - Creating video from text](https://openai.com/sora)  
**Blog**: [Video generation models as world simulators](https://openai.com/research/video-generation-models-as-world-simulators)  

These are just short notes / excerpts from the technical paper for quick lookup.

------

Sora is quite a breakthrough. It is able to understand and simulate the physical world, generating upto 60s long high-definition videos while maintaining the quality, scene continuation and following the user's prompt. 

Key papers Sora is built upon - 
* [Diffusion Transformer (DiT)](https://arxiv.org/abs/2212.09748)
* [Latent Diffusion Models](https://arxiv.org/abs/2112.10752)
* [DALL-E 3 Image Recaptioning](https://cdn.openai.com/papers/dall-e-3.pdf)

{{< figure src="/post/2024-02-18_sora_openai/images/diffusion.png" caption="" >}}

Sora (being a diffusion transformer) uses the idea from ViT of using patches. The videos are converted to patches by first first compressing videos into a lower-dimensional latent space (as in the Latent Diffusion Models paper) and then decomposing the representation into spacetime patches.

{{< figure src="/post/2024-02-18_sora_openai/images/patches1.png" caption="" >}}

The network takes raw video as input and outputs a latent representation that is compressed temporally and spatially. The video training and generation is done within this latent space. A decoder model is also trained that maps generated latents back to pixel space.

A sequence of spacetime patches is extracted from the compressed video. The patch-based representation enables Sora to train on videos and images of variable resolutions, durations and aspect ratios. During inference time, an appropriately-sized grid (for the random patches) can be selected to control the size of the generated videos.

Hence, while past approaches resized, cropped or trimmed videos to a standard size, Sora can sample widescreen 1920x1080p videos, vertical 1080x1920 videos and everything inbetween. Training on the native aspect ratios was in fact helpful for the model to improve composition and framing. This is similar to what Adept did in [Fuyu-8B](https://www.adept.ai/blog/fuyu-8b). 

{{< figure src="/post/2024-02-18_sora_openai/images/fuyu.png" caption="" >}}


### How does it adhere to text so well? 

OpenAI trained a very descriptive captioner model which they used to generate captions for all the videos in their training set (probably a finetuned GPT-4V ?). Training on highly descriptive video captions improves text fidelity as well as the overall quality of videos.

They also use query expansion - using GPT to turn short user prompts into longer detailed captions that are sent to the video model.

More details about the image captioner in the next blog post.

### What are some emergent capabilities that are exhibited with scale?

* 3D consistency - Sora videos can shift and rotate elements based on the camera motion.  
* Long-range coherence and object permanence  
* Interacting with the world - Though not perfect, Sora can sometimes remember the state of the world after an action was taken.  
* Simulating digital worlds - Sora can generate Minecraft videos, wherein it can simultaneously control the player with a basic policy while also rendering the world and its dynamics in high fidelity.

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