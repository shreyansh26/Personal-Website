---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Paper Summary #10 - Gemini 1.5 Pro"
subtitle: "Google DeepMind announced a multimodal LLM with support of up to 10M context length."
summary: ""
authors: ["Shreyansh Singh"]
tags: [llm, long context, multimodal]
categories: [Machine Learning]
date: 2024-02-18T19:01:35+05:30
lastmod: 2024-02-18T19:01:35+05:30
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

**Technical Paper**: [Gemini 1.5: Unlocking multimodal understanding across millions of tokens of context](https://storage.googleapis.com/deepmind-media/gemini/gemini_v1_5_report.pdf)  
**Blog**: [Our next-generation model: Gemini 1.5](https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024)  

These are just short notes / excerpts from the technical paper for quick lookup. Actual implementational details are anyways missing in the technical paper.

--------------------

Gemini 1.5 Pro is a sparse MoE Transformer-based model (seems like a trend these days, after GPT-4's rumors and Mixtral).

It supports upto 10M context multimodal context length which is about a day of 22 hours of audio recordings, more than ten times the entirety of the 1440 page book "War and Peace", the entire Flax codebase (41,070 lines of code), or three hours of video at 1 frame-per-second.

Near-perfect retrieval across complete context length. In all modalities, i.e., text, video and audio, it achieves 100% accurate needle-in-a-haystack recall for 530k tokens, 99.7% accurate recall for 1M tokens and 99.2% for 10M contexts. 

{{< figure src="/post/2024-02-18_gemini_pro_google/images/niah1.png" caption="" >}}

{{< figure src="/post/2024-02-18_gemini_pro_google/images/text_niah.png" caption="Text Haystack. This figure compares Gemini 1.5 Pro with GPT-4 Turbo for the text needle-in-a-haystack task. Green cells indicate the model successfully retrieved the secret number, gray cells indicate API errors, and red cells indicate that the model response did not contain the secret number. The top row shows results for Gemini 1.5 Pro, from 1k to 1M tokens (top left), and from 1M to 10M tokens (top right). The bottom row shows results on GPT-4 Turbo up to the maximum supported context length of 128k tokens. The results are color-coded to indicate: green for successful retrievals and red for unsuccessful ones." >}}

{{< figure src="/post/2024-02-18_gemini_pro_google/images/video_niah.png" caption="Video Haystack. This figure compares Gemini 1.5 Pro with GPT-4V for the video needle-in-a-haystack task, where the models are given video clips of different lengths up to three hours of video and are asked to retrieve a secret word embedded as text at different points within the clip. All video clips are sampled at one frame-per-second (1 fps). The first pair of 10 × 50 haystack plots on the left compare Gemini 1.5 Pro with GPT-4V on the first hour of the documentary. The x-axis represents the video duration which ranges from 1.2 minutes to 1 hour, and the y-axis represents the depth, namely the relative offset of the needle (e.g., the top left cell represents providing the model with the first 1.2 minutes and randomly sampling a frame in the first ten percent of that trimmed video where the needle is inserted). A green cell indicates that the model successfully retrieved the needle, whereas a gray cell indicates an API error. Whereas the GPT-4V API supports video lengths only up to around the first 3 minutes of the full hour, Gemini 1.5 Pro successfully retrieves the secret word inserted at all depth percentages up to an hour of video, as shown by the all-green plot. Finally, the 10 × 10 grid on the right shows Gemini 1.5 Pro’s perfect retrieval capabilities across three full hours of video, constructed by concatenating two copies of the documentary back-to-back." >}}

{{< figure src="/post/2024-02-18_gemini_pro_google/images/audio_niah.png" caption="Audio Haystack. This figure presents the audio version of the needle-in-a-haystack experiment comparing Gemini 1.5 Pro and a combination of Whisper and GPT-4 Turbo. In this setting, the needle is a short segment of audio that is inserted within a very large audio segment (of up to 22 hours) containing concatenated audio clips. The task is to retrieve the 'secret keyword' which is revealed in the needle. Red indicates that the model did not identify the keyword, whereas green indicates that the model identified the keyword correctly" >}}

However, with about 100 needles (with the model requiring to retrieve them all), the performance drops to 70% recall up to 128K tokens, and >60% recall up to 1M tokens. GPT-4 is 50% recall at 128k tokens (its maximum context).

{{< figure src="/post/2024-02-18_gemini_pro_google/images/multiple_niah.png" caption="" >}}

Gemini 1.5 Pro beats RAG-based systems for all modalities. Although this may not be cost efficient right now unless there are multiple queries and the long context can be prefix-cached.

Excellent use of context for learning new skills. It was able to learn to translate from/to English to/from Kalamang, an extremely low-resource language (spoken by less than 200 people in the world), with just a single set of linguistic documentation, a dictionary, and ≈ 400 parallel sentences. That's it!

{{< figure src="/post/2024-02-18_gemini_pro_google/images/kalamang.png" caption="Given a reference grammar book and a bilingual wordlist (dictionary), Gemini 1.5 Pro is able to translate from English to Kalamang with similar quality to a human who learned from the same materials." >}}

{{< figure src="/post/2024-02-18_gemini_pro_google/images/kalamang_results.png" caption="" >}}

Gemini 1.5 Pro is exceeds Gemini 1.0 Ultra on text. However the latter is still better on vision and audio tasks, but the compute-efficiency and speed of shipping does suggest some ideas of distillation coming into the picture.

{{< figure src="/post/2024-02-18_gemini_pro_google/images/results_comparison.png" caption="" >}}

Figures 4 and 5 in the Technical report are super cool. In Figure 4, Gemini 1.5 is able to extract the exact scene with the page number from a textbook given a hand-drawn sketch as the prompt. 

{{< figure src="/post/2024-02-18_gemini_pro_google/images/fig4_paper.png" caption="" >}}

In Figure 5 they show that it is able to do the same in videos as well. The model returns a detailed description and timestamp of the scene.

{{< figure src="/post/2024-02-18_gemini_pro_google/images/fig5_paper.png" caption="" >}}

The cumulative average negative log-likelihood (NLL) as a function of token position in long documents follows a power-law quite accurately up to 1M tokens for long-documents and about 2M tokens for code but deviates at the 10M scale.

{{< figure src="/post/2024-02-18_gemini_pro_google/images/pplx.png" caption="" >}}

Though it [seems like the inference speeds are quite slow](https://youtu.be/wa0MT8OwHuk) for long contexts at the moment, but the capabilities are absolutely insane!

------

An exciting future ahead!

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