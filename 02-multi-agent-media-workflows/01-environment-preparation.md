---
prev_url: ../01-consistent-media-generation-on-serverless-gpus/05-measuring-costs.md
next_url: 02-agent.md
---

# 2.1 Environment preparation

We want to create a video clip. In this clip, we'll have a desk lamp, a person working from home, dusk.

First, we'll create an image. For that we'll use a Sana endpoint. I'll use [the sana template](https://github.com/nebius/serverless-ai-cookbook/blob/main/templates/endpoint-sana/README.md) that we used in module 1.

Now we'll set up a project for that:

```bash
uv init
uv add python-dotenv requests jupyter pillow
wget https://raw.githubusercontent.com/alexeygrigorev/production-ai-agents-for-media-generation/main/02-multi-agent-media-workflows/code-agent/gen.py
```

The last command downloads `gen.py`. It's the code from module 1 for sending requests to the endpoints, wrapped into two classes: `ImageGenerator` and `VideoGenerator`.

We keep the endpoint URLs in `.env`:

```bash
IMAGE_ENDPOINT=https://...
VIDEO_ENDPOINT=https://...
VIDEO_TOKEN=...
```

Now start Jupyter:

```bash
uv run jupyter notebook
```

First, I want the notebook to pick up changes in `gen.py` without restarting the kernel:

```python
%load_ext autoreload
%autoreload 2
```

Load the `.env` file:

```python
import os
import dotenv
dotenv.load_dotenv()
```

In Jupyter, I'll send a request to the endpoint:

```python
image_endpoint = os.getenv('IMAGE_ENDPOINT')
image_model = 'Efficient-Large-Model/Sana_1600M_1024px_diffusers'

from gen import ImageGenerator

sana_generator = ImageGenerator(image_endpoint, image_model)

seed_image = sana_generator.generate_image_tool('a desk lamp, a person working from home, dusk')
seed_image
```

It saves the image and returns the path:

```
'out\\quickstart\\qwen_20261126-181336_ftm.png'
```

If your Sana endpoint has authentication on, pass the token too: `ImageGenerator(image_endpoint, image_model, token=...)`.

Next, we set up a video endpoint - I'll use the [Wan2.2-I2V-A14B template](https://github.com/nebius/serverless-ai-cookbook/blob/main/templates/endpoint-wan22-i2v-a14b/README.md). This model turns an image into a video: the image we just generated becomes the first frame.

```python
video_endpoint = os.getenv('VIDEO_ENDPOINT')
video_token = os.getenv('VIDEO_TOKEN')
video_model = 'Wan-AI/Wan2.2-I2V-A14B-Diffusers'

from gen import VideoGenerator

wan_generator = VideoGenerator(video_endpoint, video_model, token=video_token)

video = wan_generator.generate_video_tool('advertisement', seed_image, 5.0, 16)
video
```

This is a 5 second video with 16 frames per second. It takes a few minutes, and we get the path back:

```
'out\\quickstart\\wan_20261126-181422_4pu.mp4'
```

The video has the same aspect ratio as the image.

Now we have two tools: one for images and one for videos. In the next lesson, we'll give them to an agent.

<!-- nav:start -->
[← Previous: 05 - Measuring Startup, Load and Generation Costs](../01-consistent-media-generation-on-serverless-gpus/05-measuring-costs.md) · [Next: 2.2 The Agent Loop: Deep Dive →](02-agent.md)
<!-- nav:end -->
