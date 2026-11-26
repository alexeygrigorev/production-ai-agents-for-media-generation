---
prev_url: 01-environment-preparation.md
next_url: 03-gpu-tools.md
---

# 2.2 The Agent Loop: Deep Dive

Now let's create an agent.

For that, install [PydanticAI](https://pydantic.dev/docs/ai/):

```bash
uv add pydantic-ai
```

I'll use Nebius Token Factory for the agent, but you can use any other LLM provider supported by PydanticAI.

Create an API key in [Token Factory](https://docs.tokenfactory.nebius.com/quickstart) and add it to `.env`:

```bash
NEBIUS_API_KEY=...
```

Token Factory is OpenAI-compatible, so we use the OpenAI model class from PydanticAI and point it to Token Factory. See more examples in the [Token Factory cookbook](https://github.com/nebius/token-factory-cookbook/blob/main/agents/README.md).

We continue in the same notebook as in the previous lesson, so `sana_generator` and `wan_generator` are already there.

```python
from pydantic_ai.models.openai import OpenAIChatModel
from pydantic_ai.providers.openai import OpenAIProvider

model = OpenAIChatModel(
    model_name='zai-org/GLM-5.3-Flash',
    provider=OpenAIProvider(
        base_url='https://api.tokenfactory.nebius.com/v1/',
        api_key=os.environ['NEBIUS_API_KEY']
    )
)
```

Now the agent. We give it the prompt and the two tools from the previous lesson:

```python
from pydantic_ai import Agent

media_instructions = """
You help user create a video advertisement based on the idea they give you. 

First generate an image and then use the image to generate a video. The 
video should be 5 seconds with 16 fps unless user asks for a different 
configuration.
""".strip()

media_agent = Agent(
    model=model,
    tools=[
        sana_generator.generate_image_tool,
        wan_generator.generate_video_tool
    ],
    instructions=media_instructions
)
```

We don't need to describe the tools ourselves. PydanticAI reads the function signatures and docstrings and sends them to the LLM. That's why `generate_image_tool` and `generate_video_tool` have docstrings.

Let's run it:

```python
result = await media_agent.run('a desk lamp, a person working from home, dusk')
print(result.output)
```

```
Your ad is ready! Here's what I created:

**The scene:** A warm, cinematic dusk shot — a sleek desk lamp casting amber light over a person working from home on their laptop, with deep blue twilight glowing through the window behind them.

**The video (5s @ 16fps):**
- A slow cinematic push-in toward the glowing lamp
- The person's hands typing gently on the laptop
- Subtle light flicker from the lamp with dust particles drifting through the beam
- Dusk deepening outside the window for that calm, focused evening-work mood

**Files:**
- 🖼️ Image: `out\quickstart\qwen_20261126-184233_b1k.png`
- 🎬 Video: `out\quickstart\wan_20261126-184315_s4z.mp4`

Want a different vibe (brighter, more minimalist, a specific lamp style) or a different camera movement? I can regenerate either the image or the video.
```

It worked, but we don't see what happened inside. Which tools did the agent call, and with which arguments?

For that, we can pass a callback that prints every tool call:

```python
from pydantic_ai.messages import FunctionToolCallEvent

class NamedCallback:

    def __init__(self, agent):
        self.agent_name = agent.name

    async def print_function_calls(self, ctx, event):
        # Detect nested streams
        if hasattr(event, "__aiter__"):
            async for sub in event:
                await self.print_function_calls(ctx, sub)
            return

        if isinstance(event, FunctionToolCallEvent):
            tool_name = event.part.tool_name
            args = event.part.args
            print(f"TOOL CALL ({self.agent_name}): {tool_name}({args})")

    async def __call__(self, ctx, event):
        return await self.print_function_calls(ctx, event)
```

Let's run the agent again with it:

```python
callback = NamedCallback(media_agent)

result = await media_agent.run(
    'a desk lamp, a person working from home, dusk',
    event_stream_handler=callback
)
```

```
TOOL CALL (media_agent): generate_image_tool({"prompt": "Cinematic advertisement still: a modern minimalist desk lamp glowing warm amber as the hero of the scene, on a tidy wooden desk where a person works from home at dusk. Soft blue-purple twilight light through a large window behind them, laptop open, coffee mug steaming, notebook and plant on desk. Shallow depth of field, warm-cool color contrast, moody product photography, photorealistic, 35mm lens, cozy atmosphere"})
TOOL CALL (media_agent): generate_video_tool({"duration": 5, "fps": 16, "image_path": "out\\quickstart\\qwen_20261126-184847_ezg.png", "prompt": "Slow cinematic camera push-in toward the glowing desk lamp. The person types gently on the laptop, steam rises softly from the coffee mug, warm lamp light subtly pulses, dusk light outside the window slowly deepens. Calm, cozy, premium advertisement mood"})
```

Now we see the loop: the agent first calls `generate_image_tool`, gets the path to the image, and then passes this path to `generate_video_tool`.

Note that it didn't use our short idea as is. It wrote a much more detailed prompt for each model. And it set `duration` and `fps` from the instructions.

<!-- nav:start -->
[← Previous: 2.1 Environment preparation](01-environment-preparation.md) · [Next: 2.3 GPU Workloads as Agentic Tools →](03-gpu-tools.md)
<!-- nav:end -->
