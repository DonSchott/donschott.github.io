---
layout: post
title: ComfyUI on the DGX Spark
date: 2026-09-25
description: "The ComfyUI playbook brings AI image and video creation to the DGX Spark."
tags: comfyui image-generation local-ai
categories: ai
thumbnail: assets/img/2026-09-25-comfyui-dgx-spark/flamingo-bicycle-photo.jpg
giscus_comments: false
related_posts: false
---

I created a couple of AI-generated pictures before, since it is a fun way to spend time. I fooled around with DALL·E 2 (OpenAI) and Google Gemini when they came out, but I am not a content creator, so my personal interest was limited.

The ComfyUI playbook brings AI image and video creation to the DGX Spark. So I tried it out to see how far local models can go, how long it takes and if it can compete with models that run in the cloud. It takes less than an hour to follow the [playbook](https://build.nvidia.com/spark/comfyui/overview) to set it up and produce the first images. The playbook offers a setup script that installs PyTorch, ComfyUI and about 20GB of models. There are many models and templates to choose from. I picked the "Z-Image-Turbo: Text to Image" template and hit Run.

I did not run into problems and the setup was smooth. I was delighted by how mature the UI felt. The UI uses nodes to visualize the data flow and shows each step as a box (load model → prompt → sample → decode → save). I think that is a nice and intuitive way and more beginner-friendly than e.g. a bunch of YAML config files.

{% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/comfyui-graph-baboons.png" class="img-fluid rounded z-depth-1" zoomable=true alt="ComfyUI: load model → prompt → sample → decode → save" caption="“a rockband of baboons”" %}

I use the DGX Spark as a remote machine, so I forwarded the port to my laptop: `ssh -L 8188:localhost:8188 USER@SPARK`. The remote use worked well, Spark doing the heavy lifting (computing) and my laptop accessing the control panel in the browser at `http://localhost:8188`. This was my setup: the model was Z-Image-Turbo, 6B parameters, from Alibaba Tongyi Lab. Its text encoder was Qwen3-4B and its VAE was a Flux-compatible autoencoder.

It took about 7 s to create a picture at 1024×1024 resolution -- not instantly, but fast enough to stay in the flow changing the input text and seeing how the generated image changes. The popular blogger Simon Willison famously uses the [pelican riding a bicycle](https://simonwillison.net/tags/pelican-riding-a-bicycle/) prompt to benchmark LLMs; that is why I played around with prompts involving animals such as octopus, flamingo or pelican doing things like playing in bands, riding bicycles or rollerblades. It was interesting to see that even typos ("octupus" for octopus, "harf" for harp) were interpreted correctly.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/octopus-harp-drums.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="draw a picture of an octupus playing harf and drums" caption="“draw a picture of an octupus playing harf and drums”" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/octopus-band.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="draw a picture of band of octupus playing harmonica" caption="“draw a picture of band of octupus playing harmonica”" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/octopus-rollerblades.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="draw a picture of octupus on rollerblades" caption="“draw a picture of octupus on rollerblades”" %}
    </div>
</div>

The bicycle stress test was passed successfully! The bikes look good and the flamingo pedals on one leg, which is very flamingo-like to do.
When prompting to draw, the results came out as flat illustrations. When instead of prompting "draw a picture of…" I just wrote "picture of…", the result was photorealistic. So tiny changes in the prompt can have a big effect.

<div class="row mt-3">
    <div class="col-6">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/flamingo-bicycle-drawn.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="draw a picture of a flamingo on a bicycle" caption="“draw a picture of a flamingo on a bicycle”" %}
    </div>
    <div class="col-6">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/flamingo-bicycle-photo.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="picture of a flamingo on a bicycle" caption="“picture of a flamingo on a bicycle”" %}
    </div>
</div>
<div class="row">
    <div class="col-6">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/octopus-bicycle-drawn.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="draw a picture of octupus on a bicycle" caption="“draw a picture of octupus on a bicycle”" %}
    </div>
    <div class="col-6">
        {% include figure.liquid path="assets/img/2026-09-25-comfyui-dgx-spark/octopus-bicycle-photo.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="an octopus riding a bicycle" caption="“an octopus riding a bicycle”" %}
    </div>
</div>

I also tried out the video creation. After all, a video is just a sequence of images, right? What I found is that in contrast to images, the creation of videos takes some serious time. Fifteen seconds of a low-res video took around 20 min on the Spark. The machine had to work really hard all the time at 100% GPU usage, heating up to 89 °C. I used a model that took images as input to output videos. The degree of photorealism of the output was astonishing, but the action was rather nonsensical. It seems that creating professional AI-generated films is orders of magnitude harder than creating images.

To sum up my experience, I am impressed by how polished ComfyUI is. DGX Spark and ComfyUI are a great combo to create AI-generated images in a few seconds that can be as useful as services from frontier labs and cloud providers. Just using open models and templates already gave nice results. For video creation more experience and time seem to be necessary. Videos are a different beast and need orders of magnitude more compute. I assume good old batch process workflows are needed, starting the job in the evening, going to sleep and hoping the job is finished next morning...far from instantaneous.
