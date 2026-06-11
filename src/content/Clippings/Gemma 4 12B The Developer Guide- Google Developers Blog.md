---
title: "Gemma 4 12B: The Developer Guide- Google Developers Blog"
source: "https://developers.googleblog.com/gemma-4-12b-the-developer-guide/"
author:
  - "[[André Susano Pinto]]"
  - "[[Andreas Steiner]]"
  - "[[Karolis Misiunas]]"
  - "[[Karsten Roth]]"
  - "[[Michael Tschannen]]"
  - "[[Omar Sanseviero]]"
published: 2026-06-03
created: 2026-06-05
description: "Meet Gemma 4 12B: the first medium-sized, encoder-free multimodal model capable of natively ingesting audio and video. Ideal for local AI development with 16GB VRAM, Hugging Face integrations, and drop-in local API servers."
tags:
  - "clippings"
---
## Gemma 4 12B: The Developer Guide

[André Susano Pinto](https://developers.googleblog.com/search/?author=Andr%C3%A9+Susano+Pinto) Research Engineer

[Andreas Steiner](https://developers.googleblog.com/search/?author=Andreas+Steiner) Research Engineer

[Karolis Misiunas](https://developers.googleblog.com/search/?author=Karolis+Misiunas) Research Engineer

[Karsten Roth](https://developers.googleblog.com/search/?author=Karsten+Roth) Research Scientist

[Michael Tschannen](https://developers.googleblog.com/search/?author=Michael+Tschannen) Research Scientist

[Omar Sanseviero](https://developers.googleblog.com/search/?author=Omar+Sanseviero) Member of the Technical Staff

![placeholder](https://storage.googleapis.com/gweb-developer-goog-blog-assets/images/hello-gemma.original.png)

Following the announcement in our [launch blog](https://blog.google/innovation-and-ai/technology/developers-tools/introducing-gemma-4-12B/), we are releasing **Gemma 4 12B**, a dense multimodal model with a **unified, encoder-free architecture**.

Gemma 4 12B introduces several milestones for local AI:

1. **A multimodal encoder-free architecture:** Bypassing heavy multi-stage vision and audio encoders entirely, multimodal data is fed straight into the LLM backbone, reducing multimodal latency.
2. **Our first medium-sized model with audio input:** In the Gemma family, audio inputs were restricted to small, lightweight edge architectures (e.g. E4B). Gemma 4 12B is the first medium-sized model capable of natively ingesting audio.
3. **Developer-friendly size**: Small enough to run locally on dedicated GPU laptops with 16GB VRAM or unified memory. To maximize local inference speeds, we are additionally releasing a dedicated [multi-token prediction (MTP)](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/) model.
4. **New MacOS desktop experience**: For the first time, we are releasing downloadable macOS desktop applications, letting developers experience fully local spoken and visual interaction directly on consumer-grade devices.

## The Architecture

Traditional multimodal models rely on frozen, separate vision encoders (e.g., Gemma 4 uses a 150M parameter vision model for edge sizes and 550M for medium-sized models) and audio encoders (300M parameters for Gemma 4 E2B and E4B). Processing multimodal inputs with multiple separate encoders before feeding them to the LLM leads to increased latency and fragmented memory footprints.

Gemma 4 12B solves these issues by utilizing a single decoder-only transformer containing the same advanced decoder structure as the Gemma 4 31B Dense model.

![overview](https://storage.googleapis.com/gweb-developer-goog-blog-assets/images/overview.original.png)

- **Vision embedder (35M parameters):** Replaces the 27 vision transformer layers of the other medium-sized Gemma 4 models. Raw 48x48 pixel patches are projected to the LLM hidden dimension with a single matmul. A factorized coordinate lookup (X and Y matrices) attaches spatial location information directly to the input.
- **Audio wave projection:** Eliminates the separate audio encoder (skipping the 12 conformer layers used in Gemma 4 E2B and E4B). Raw 16 kHz audio signals are sliced into 40ms frames (640 floats each) and projected linearly to the LLM input space.
- **Unified fine-tuning advantage:** Because vision, audio, and text inputs share the exact same weights, **you no longer have to co-tune separate frozen encoders**. Downstream adapter (e.g. LoRA) or full tuning naturally update the entire multimodal token loop in a single pass (via Hugging Face or Unsloth).

For a more in-depth overview of how this encoder-free architecture works, check out [A Visual Guide to Gemma 4 12B](https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-gemma-4-12b).

## Capabilities

Gemma 4 12B achieves outstanding performance, with capabilities such as automatic speech recognition, agentic reasoning, diarization, video understanding, coding, and more.

See below examples for a demonstration of the model's agentic and multimodal capabilities:

## Example 1: Gemma 4 12B creates a local image processing app that uses Gemma 4 12B

<video controls=""><source src="https://storage.googleapis.com/gweb-developer-goog-blog-assets/original_videos/Gemma_4_-_12B_-_Create_Object_Detection_App_-_001_-_Web.mp4" type="video/mp4"><p>Sorry, your browser doesn't support playback for this video</p></video>

Thanks to its agentic and multimodal understanding capabilities, Gemma 4 12B can be easily used with existing agent harnesses like OpenCode. In this example, we served it locally using llama.cpp using the [gemma-skills](https://github.com/google-gemma/gemma-skills) to code a Gradio app that helped the user process images. This app was powered by the same Gemma 4 12B model that built it!

## Example 2: Processing 5 minutes of Video at 1 FPS with audio

We used Gemma 4 12B to analyse a part from the [Google IO Keynote moment](https://www.youtube.com/watch?v=wYSncx9zLIU&start=932&end=1245) from May 19, specifically the 5 minutes between 00:15:32 and 00:20:45. To do that, we extracted all the frames of that segment (at 1 FPS), plus the prompt and the audio from the video:

![](https://www.youtube.com/watch?v=wYSncx9zLIU)

[Link to Youtube Video](https://www.youtube.com/watch?v=wYSncx9zLIU) (visible only when JS is disabled)

Prompt:

1. 313 frames (at 1FPS, images resized to [visual token budget](https://ai.google.dev/gemma/docs/capabilities/vision/image?_gl=1*jm0coc*_up*MQ..*_ga*MTI2MzkwODM0NS4xNzgwNDI5Mzg5*_ga_P1DBVKWT6V*czE3ODA0MjkzODkkbzEkZzAkdDE3ODA0MjkzODkkajYwJGwwJGgxOTEyOTEwNzc5#variable_resolution_token_budget) 70)
2. `"what happens when the man takes a selfie?"`
3. the Video's audio + the prompt below

> When the man takes a "selfie" or is shown holding a smartphone in front of his face in these demonstration videos, it is a clever way to visually represent how the AI models can take existing media—like a personal photo or a video clip—and "reimagine" it. In these specific clips, the model is taking a selfie and using it as a base to generate various scenarios (like a person in a space station, or walking through a forest).  
>   
> Essentially, the man is not actually taking a selfie; rather, he is \*\*acting out a visual metaphor for the AI's capability to take one specific input (a "selfie") and generate a whole world of new content based on it.\*\* This is part of the "Swap" and "Build worlds" demonstrations of the Gemini Omni model, showing its ability to perform complex, multi-modal reasoning and creative generation.

## On-Device & Desktop Serving: Powered by LiteRT-LM

In tandem with the Gemma 4 12B launch, we are officially introducing powerful on-device developer integrations powered by LiteRT-LM, bringing zero-latency local AI execution natively to standard desktop environments:

**1.Native MacOS Apps**: The mobile [**Google AI Edge Gallery**](https://developers.google.com/edge/gallery) is officially expanding to desktop platforms, running Gemma 4 12B offline, natively on Apple Silicon GPUs. It comes with a secure sandboxed Python execution loop to write, execute, and plot scientific charts inside the chat bubble. In parallel, the [**Google AI Edge Eloquent**](https://ai.google.dev/edge/eloquent) app on Mac launches support for Gemma 12B to power Voice Edit conversational inputs.

<video controls=""><source src="https://storage.googleapis.com/gweb-developer-goog-blog-assets/original_videos/eloquent_recording_deviceFrame.mp4" type="video/mp4"><p>Sorry, your browser doesn't support playback for this video</p></video>

**2\. Drop-in Local API Servers (litert-lm serve):** Run Gemma 4 12B as a local, OpenAI-compatible API server using the new litert-lm serve [**CLI command**](https://ai.google.dev/edge/litert-lm/cli)**.** Seamlessly connect standard integrations (e.g., Continue, Aider, OpenClaw, Hermes or OpenCode), leveraging stateless prefix caching in memory to match context history and instantly bypass prefill latency.

```shell
litert-lm import --from-huggingface-repo=litert-community/gemma-4-12B-it-litert-lm  gemma-4-12B-it.litertlm gemma4-12b

# Start the OpenAI-compatible server
litert-lm serve
```

Find a deep dive about it on the Google AI Edge Gallery [blog](https://developers.googleblog.com/bringing-gemma-4-12b-to-your-laptop-unlocking-local-agentic-workflows-with-google-ai-edge).

## Getting Started Today

Ready to build local multimodal agents with the first encoder-free architecture of the Gemma family? Here is how you can jump in today

- **Try it yourself**: Experiment with a couple of clicks in [LM Studio](https://lmstudio.ai/models/gemma-4), [Ollama](https://ollama.com/library/gemma4), [Google AI Edge Gallery App](https://developers.google.com/edge/gallery), the [Google AI Edge Eloquent](https://ai.google.dev/edge/eloquent) app and the [LiteRT-LM CLI](https://ai.google.dev/edge/litert-lm/cli)
- **Download the weights**: Download the pre-trained and instruction-tuned checkpoints directly from [Hugging Face](https://huggingface.co/collections/google/gemma-4) and [Kaggle](https://www.kaggle.com/models/google/gemma-4).
- **Integrate & learn:** Review the [developer documentation](https://ai.google.dev/gemma/docs/core) and the [quick start notebook](https://ai.google.dev/gemma/docs/capabilities/text/basic).
- **Use your favorite development tools**: Implement local inference pipelines with [Hugging Face Transformers](https://huggingface.co/google/gemma-4-12B-it), [llama.cpp](https://huggingface.co/collections/ggml-org/gemma-4), [MLX](https://huggingface.co/collections/mlx-community/gemma-4), [SGLang](https://docs.sglang.io/cookbook/autoregressive/Google/Gemma4), and [vLLM](https://docs.vllm.ai/projects/recipes/en/latest/Google/Gemma4.html), or fine-tune with efficiency using [Unsloth](https://unsloth.ai/docs/models/gemma-4).
- **Unlock Agentic Development with Gemma Skills:** To support agents to build with the latest Gemma advancements, we are releasing our official [Skills Repository](https://github.com/google-gemma/gemma-skills). This is a library of skills designed specifically to enable agents to build with Gemma models.
- **Deploy your way:** Spin up endpoints in production using Google Cloud. Deploy your way through [Gemini Enterprise Agent Platform Model Garden](https://console.cloud.google.com/agent-platform/publishers/google/model-garden/gemma4;publisherModelVersion=gemma-4-12b-it), [Cloud Run](https://codelabs.developers.google.com/codelabs/cloud-run/cloud-run-gpu-rtx-pro-6000-gemma4-vllm) and [GKE](https://docs.cloud.google.com/kubernetes-engine/docs/tutorials/serve-gemma-gpu-vllm).