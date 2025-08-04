+++
title = 'How to estimate the materialized model size'
date = 2025-08-03T21:11:54-07:00
tags = ['LLM fundamental']
draft = false
math = true
+++

When we heard about Large Language Models, we always hear about the parameter size of the model.
E.g. GPT-3.5 has 175 billion parameters, Deepseek R-1 has 671 billion parameters and GPT-4 is rumored
to even have 1.8 trillion parameters.

## What exactly does that mean?

A fundamental knowledge is, for most part, the model is composed of the model's parameters (often called
weights and biases). Each parameter is a numerical value that the model learned during training and the
precision of these numbers dictates how much space they occupy.

Common data types used in LLM training and inference include:

- FP32 (32-bit floating point): each parameter takes 4 bytes
- FP16 (16-bit floating point) or BF16 (BFloat16): each parameter takes 2 bytes
- INT8 (8-bit integer): each parameter takes 1 byte
- INT4 (4-bit integer): each parameter takes 0.5 bytes

NOTE: Detail explanation of these data types refer to this [guide](https://moocaholic.medium.com/fp64-fp32-fp16-bfloat16-tf32-and-other-members-of-the-zoo-a1ca7897d407)

With above knowledge, it is very easy to estimate the model size using this formula:

$$
  \text{model\\_size} = \text{number\\_of\\_parameters} \times \text{bytes\\_per\\_parameter}
$$

## How do we calculate the estimated size?

Take an example of Deepseek R1 from [Ollama](https://ollama.com/library/deepseek-r1:671b)

![Deepseek R1](https://github.com/user-attachments/assets/7686a864-df8b-47d6-bd66-a91a60f059bc)

We know that:

1. There is 671 billion parameters
1. It is using Q4_K_M quantization, that means 0.5 bytes per parameter

Therefore, we can calculate the space usage using

$$
671,000,000,000 \text{ parameters} \times 0.5 \text{ bytes/parameter} = 335,500,000,000 \text{ bytes}
$$

And the final result is around `312.4GiB`, this is closer to the 404 GiB size. The additional size in the actual file can be attributed to the overhead of the Q4_K_M quantization (the scaling factors and other metadata that add to the size) and the model's architecture itself, which includes more than just the parameters.

Overall, using above method should give us a rough estimated scale of the model size, and help us
design the system using LLM.