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

`gen.py` has two classes, `ImageGenerator` and `VideoGenerator`, that wrap the endpoint requests from module 1.

Put the endpoint URLs and tokens in `.env` (leave a token empty if the endpoint has no authentication):

```bash
IMAGE_ENDPOINT=https://...
IMAGE_TOKEN=...
VIDEO_ENDPOINT=https://...
VIDEO_TOKEN=...
```

In Jupyter, I'll send a request to the endpoint:

```python
%load_ext autoreload
%autoreload 2

import os
import dotenv
dotenv.load_dotenv()

from gen import ImageGenerator

image_endpoint = os.getenv('IMAGE_ENDPOINT')
image_token = os.getenv('IMAGE_TOKEN')
image_model = 'Efficient-Large-Model/Sana_1600M_1024px_diffusers'

sana_generator = ImageGenerator(image_endpoint, image_model, token=image_token)

seed_image = sana_generator.generate_image_tool('a desk lamp, a person working from home, dusk')
seed_image
```

`generate_image_tool` saves the image to `out/quickstart/` and returns the path.

Next, we set up a video endpoint - I'll use the [Wan2.2-I2V-A14B template](https://github.com/nebius/serverless-ai-cookbook/blob/main/templates/endpoint-wan22-i2v-a14b/README.md). It's image to video: the image we just generated becomes the first frame of the clip.

```python
from gen import VideoGenerator

video_endpoint = os.getenv('VIDEO_ENDPOINT')
video_token = os.getenv('VIDEO_TOKEN')
video_model = 'Wan-AI/Wan2.2-I2V-A14B-Diffusers'

wan_generator = VideoGenerator(video_endpoint, video_model, token=video_token)

video = wan_generator.generate_video_tool('advertisement', seed_image, 5.0, 16)
video
```

The clip keeps the aspect ratio of the input image. Generating it takes a few minutes.

<!-- nav:start -->
[← Previous: 05 - Measuring Startup, Load and Generation Costs](../01-consistent-media-generation-on-serverless-gpus/05-measuring-costs.md) · [Next: 2.2 The Agent Loop: Deep Dive →](02-agent.md)
<!-- nav:end -->
