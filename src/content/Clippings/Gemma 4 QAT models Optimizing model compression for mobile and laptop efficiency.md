---
title: "Gemma 4 QAT models: Optimizing model compression for mobile and laptop efficiency"
source: "https://blog.google/innovation-and-ai/technology/developers-tools/quantization-aware-training-gemma-4/"
author:
  - "[[Olivier Lacombe]]"
published: 2026-06-05
created: 2026-06-06
description: "We’re releasing Gemma 4 quantization-aware training checkpoints, reducing memory requirements and improving on-device performance."
tags:
  - "clippings"
---
Since releasing [Gemma 4](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/) two months ago, we've been continuously working to expand its capabilities. First, we introduced [Multi-Token Prediction](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/) (MTP) to accelerate inference, and just a couple of days ago, we released [a 12B model](https://blog.google/innovation-and-ai/technology/developers-tools/introducing-gemma-4-12b/) to bridge the gap between our E4B and 26B MOE models.

Today, we are releasing new checkpoints optimized with Quantization-Aware Training (QAT) to make Gemma 4 even more efficient, so you can run models locally on everyday edge devices and consumer GPUs.

By simulating quantization during training, QAT minimizes quality loss when the model is compressed. This release includes QAT checkpoints for the popular Q4\_0 quantization format as well as a novel quantization format specialized for mobile use cases. Using this mobile format, we’ve reduced the memory footprint of Gemma 4 E2B to 1GB. Together, these dramatically reduce memory requirements while preserving the capabilities and quality you expect from Gemma 4.

## Keeping model quality while making them smaller

Quantization is a key technology to run models on consumer hardware by reducing their memory footprint while also accelerating decode speed. However, standard Post-Training Quantization (PTQ) often leads to performance degradation. Instead of simply quantizing the model after training, QAT integrates the quantization process directly into training. While PTQ is already effective at preserving quality, our QAT results yield even higher overall quality compared to standard PTQ baselines.

We applied this QAT recipe to the popular Q4\_0 format to maximize performance for all the models. For the edge models (E2B and E4B), we rethought how we approach quantization with a special mobile-specialized quantization schema.

## Saving on VRAM and Storage

Below are the approximate memory requirements indicating how much VRAM is required to load the models:

![Approximate memory requirements indicating how much VRAM is required to load the models.](https://storage.googleapis.com/gweb-uniblog-publish-prod/images/2096x1182-D.width-1000.format-webp.webp)

Approximate memory requirements indicating how much VRAM is required to load the models.

## Optimizing for mobile devices under the hood

Standard compression formats are often hard for mobile processors to run efficiently. To ensure Gemma 4 performs smoothly on mobile, we engineered a custom mobile-quantization schema designed for edge hardware:

- **Static activations:** Normally, models waste processing power calculating how to scale data on the fly. We pre-calculate these settings during training, which reduces workload on mobile chips and makes responses faster.
- **Channel-wise quantization:** We structured the compressed data to fit the design of mobile accelerators. This allows the phone to run calculations natively without needing slow workarounds.
- **Targeted 2-bit quantization:** We heavily compressed (to 2-bit) the specific parts of the model that generate tokens, while keeping the core reasoning layers at higher precision. This saves storage without making the model less smart.
- **Embedding and KV cache optimization**: We focused compression on the model’s vocabulary list and its short-term memory. This drastically reduces the active memory footprint, letting you have long chats without running out of space.

Because our audio and vision encoders are not needed in many use cases, you can optimize your memory footprint even further by deploying only the modalities you need. For example, the Gemma 4 E2B text-only model (without Per-Layer Embeddings) requires less than 1 GB of memory.

## Get started today

To make those models easily usable with your preferred workflow, we’ve partnered with popular developer tools across the ecosystem to seamlessly support the Gemma 4 QAT checkpoints starting today:

- **Download the weights:** Access the [Q4\_0](https://huggingface.co/collections/google/gemma-4-qat-q4-0) and [mobile](https://huggingface.co/collections/google/gemma-4-qat-mobile) model weights right now on Hugging Face. We've tailored the formats to fit your workflow: GGUF formats are ready for use with llama.cpp, and compressed tensors are provided for vLLM. For everything else, we share unquantized checkpoints that can be converted and quantized into formats supporting Q4\_0.
- **Integrate & learn:** Explore our [documentation](https://ai.google.dev/gemma/docs/core#qat) to learn how to best deploy the QAT checkpoints.
- **Try on your desktop:** Easily download, manage, and run Gemma 4 QAT models locally on your desktop using user-friendly interfaces like [llama.cpp](https://huggingface.co/collections/google/gemma-4-qat-q4-0), [Ollama](https://ollama.com/library/gemma4) and [LM Studio](https://lmstudio.ai/models/gemma-4).
- **Deploy on-device:** Use Google's lightweight [LiteRT-LM](https://huggingface.co/collections/litert-community/gemma-family) runtime for optimized edge deployment or run the models directly on the web with [Transformers.js](https://huggingface.co/collections/onnx-community/gemma-4-onnx)
- **Use your favorite development tools:** Serve larger models efficiently with [SGLang](https://docs.sglang.io/cookbook/autoregressive/Google/Gemma4) and [vLLM](https://huggingface.co/collections/google/gemma-4-qat-q4-0), optimize for Apple Silicon with [MLX](https://huggingface.co/collections/mlx-community/gemma-4-qat). Use the MTP QAT checkpoints to preserve the speedup of [MTP](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/) while quantizing the models. Fine-tune weights directly using Hugging Face Transformers and [Unsloth](https://unsloth.ai/docs/models/gemma-4/qat).

We can't wait to see what you build with Gemma 4 running locally!