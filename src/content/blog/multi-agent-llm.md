---
title: "Fine-Tuning LLMs for Multi-Agent Collaboration"
date: 2026-02-24
description: "Can collaboration be trained into a language model? We built a benchmark of 7 coordination tasks and fine-tuned LLaMA-3 8B to match a model 9x its size."
---

Can you fine-tune a language model to be a better collaborator? That was the central question behind our [multi-agent LLM project](/projects/multi-agent-llm) at Northeastern — and the answer turns out to be yes, definitively.

We built a grid-world environment with 7 tasks requiring multi-agent coordination, then fine-tuned LLaMA-3.1 8B with LoRA on successful collaboration trajectories. The fine-tuned 8B model matched or exceeded the performance of the base 70B model across most tasks.

## The setup

Multiple LLM-powered agents operate in a shared grid environment. They take turns, can move in four directions, pick up and drop items. Crucially, agents don't know each other's positions — they can only communicate through an inbox messaging system, sending one message per turn.

We designed 7 tasks of increasing complexity:

1. **Single-agent navigation** — baseline, navigate to a goal
2. **Corner multi-agent navigation** — multiple agents must reach assigned corners
3. **Alphabetical line task** — agents must arrange themselves in name order
4. **Random points navigation** — coordinate to cover dynamically assigned targets
5. **Single-agent pick up item** — navigate to an item, grab it, deliver it
6. **Multi-agent pick up items** — coordinate item collection across agents
7. **Multi-agent pick up with permissions** — only authorized agents can grab specific items

## Agent architecture

Each agent outputs a structured JSON response with fields for reflection, rationale, action, message, and memory. The reflection mechanism lets agents evaluate their own past actions before deciding what to do next. The message field enables inter-agent communication — agents discuss task assignments, share goals, resolve conflicts, and confirm task completion.

The communication protocol maps onto the standard LLM role structure: system role provides environment state and rules, user role provides current observations and inbox messages, assistant role is the agent's response.

## Training

We generated training data by running simulations and recording all agent interactions. Trajectories scoring below 50/100 were pruned, leaving 5.3M tokens for training and 1.6M for validation. We fine-tuned LLaMA-3.1 8B using LoRA (rank 8, alpha 16, dropout 0.2) for 6 epochs with a learning rate of 5e-5. Training loss dropped from 0.46 to 0.35.

## Results

The results tell a clear story about what fine-tuning buys you:

- **Single-agent navigation**: Base 8B scored ~60, 70B scored ~90, fine-tuned 8B scored **100**
- **Corner navigation**: Base 8B scored ~30, 70B scored ~65, fine-tuned 8B scored **~70**
- **Alphabetical ordering**: Base 8B scored ~10, 70B scored ~60, fine-tuned 8B scored **~70**
- **Multi-agent pick up with permissions**: Base 8B scored ~10, 70B scored ~90, fine-tuned 8B scored **~85**

The pattern is consistent: the fine-tuned 8B model rivals or beats the 70B model across tasks, despite being 9x smaller. The largest gains come on the hardest coordination tasks — exactly where the base 8B model struggled most.

## Takeaway

Collaboration isn't just an emergent property of scale. It's a trainable dimension. A small model with targeted fine-tuning on collaborative trajectories can match a model nearly an order of magnitude larger. The implication for building multi-agent systems: you don't need massive models if you train specifically for coordination.

[Read the full report (PDF)](/documents/multi-agent-llm-report.pdf) | [GitHub](https://github.com/tamteaa/CS5100B-Project)
