---
prev_url: 01-environment-preparation.md
next_url: 03-gpu-tools.md
---

# 2.2 The Agent Loop: Deep Dive


Now let's create an agent. 

for that install [PydanticAI](https://pydantic.dev/docs/ai/):

uv init pydantic-ai

I'll use Nebius Token Factory for the agent, but you can use any other LLM provider supported by PydanticAI. 

Create the token:

...

example:


https://github.com/nebius/token-factory-cookbook/blob/main/agents/README.md


Prompt:


You help user create a video advertisement based on the idea they give you. 

First generate an image and then use the image to generate a video. The 
video should be 5 seconds with 16 fps unless user asks for a different 
configuration.



<!-- nav:start -->
[← Previous: 2.1 Environment preparation](01-environment-preparation.md) · [Next: 2.3 GPU Workloads as Agentic Tools →](03-gpu-tools.md)
<!-- nav:end -->
