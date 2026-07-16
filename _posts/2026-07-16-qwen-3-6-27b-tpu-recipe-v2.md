---
layout: post
title: "Deploying a Code Harness with Qwen 3.6 27B on TPUs: 15X Cost Improvement over closed-source alternatives"
date: 2026-07-16 13:00:00
description: "Notes, setup recipes, and commentary on serving Qwen 3.6 27B on Cloud TPUs and integrating with an agentic harness."
tags: tpu qwen llm benchmark google-cloud
categories: ai-infra
---

# What problem are we solving?

Gartner recently published compelling research outlining that enterprises are spending somewhere between $200-$500 per developer per month on coding agents, and in some cases, exceeding $2,000 a month per dev. The types of tasks executed by developers on their agentic harnesses range from single-file autocompletion, to multi-file feature implementations, to remotely-monitored, autonomously-executed, long-running experiments around the clock. While developers are clearly significantly more productive with closely-monitored usage of coding agents, CFOs and CTOs are asking the hard question around Return-on-Investment (ROI). If you have to budget an extra 30-50% cost per dev in your product roadmap, it may overturn the business case behind the upcoming feature releases altogether. With this dilemma in mind, teams are looking to preserve their productivity and output quality while looking for off-ramps on cost.

With this context in mind, this blog focuses on enabling Qwen 3.6 27B, a highly performant SOTA 27B dense open-source model from Alibaba, optimized for coding tasks. Qwen 3.6 27B scores a 77% on SWE-Bench-Verified, which puts it within punching distance of Claude-Opus-4.5 (at 81% on SWE-Bench-Verified). The dense architecture makes it easy to deploy and maintain for smaller AI infrastructure teams, and there are widely available public optimizations to make this model excel at agentic code generation (more on this later!). Given the demand on latest generation GPUs and TPUs, this endeavor also focuses on enabling Qwen 3.6 27B on TPU v6e—highlighting the fact that older generation AI accelerators when combined with SOTA models can actually win in the performance-per-dollar frontier. Lastly, this blog goes one step further beyond an optimized serving recipe to showcase integration with an open-source coding harness (e.g. Pi)—lighting up the end-to-end path for implementation. Before we dive into the details, and to unbury the lede, this configuration can easily provide 10-15x cost savings compared to closed-source alternatives, enabling businesses to re-focus on revenue growth with product development vs. staying mired in cost anxiety.

---

# Context

For this recipe, we start by picking Qwen 3.6 27B (the FP8 rendition). As stated earlier, there are three primary reasons behind this choice:

* **(a) Dense Model Sizing & Deployment Flexibility:** This dense model at its sizing, particularly in FP8, is easy to deploy, maintain, and post-train—offering engineering teams flexibility and freedom to explore.
* **(b) High Agentic Capability:** This model punches above its weight class in terms of agentic ability and instruction following as it pertains to SWE tasks, as evidenced by where it ranks in SWE-Bench compared to other higher weight-class models.
* **(c) Ecosystem Adoption:** This model has had a lot of traction in public domains, leading to an abundance of serving and orchestration templates/hooks etc. which will make our effort more efficient.

In terms of the hardware and inference serving stack, we focus on an acceptable-latency, throughput-optimized recipe running on TPU v6e. This approach is (a) to get around the demand of latest generation TPUs (e.g. TPU v7x) and GPUs (e.g. Blackwell), and (b) to achieve the most efficient performance-per-dollar frontier given price relief on older generations with every latest AI accelerator generation being released. We choose a 4-chip, chip-to-chip interconnected topology of TPU v6e chips with a storage disk for persistent model weight storage. We orchestrate via GKE (giving opportunity for scale-out for larger dev teams) and leverage vLLM for an optimized inference engine.

---

# Inference Optimizations

It is important to spend some time highlighting the key optimizations for serving that we employed in this recipe. These are discussed below:

* **(a) FP8 Weights and FP8 KV Cache:** We compressed both the weights and the KV cache tokens to 8-bit precision in HBM, enabling maximum concurrency capacity (i.e. number of developers) and context headroom (i.e. code/instruction size) for our TPU chips. FP8 compression for this class of models leads to <1% degradation in accuracy (ranging from 0.2% to 0.8% in major benchmarks).
* **(b) Tensor Parallelism = 4 (Splitting Model Across 4 Chips):** We evenly shared the 27B dense model across all 4 chips of the TPU v6e-4 node over low-latency Inter-Chip Interconnect (ICI). This enables us to free up more High-Bandwidth Memory (HBM) for larger KV caches (enabling larger context lengths and more concurrency)—and the ICI connectivity ensures that we don't pay a large penalty with communication overhead across chips during `AllGather` and `ReduceScatter` collectives.
* **(c) Prefix Caching:** We share KV cache blocks across identical prompt prefixes (e.g. system instructions, code imports, common codebase context etc.)—allowing us to reduce Time-to-First-Token (TTFT) by >80% for multi-turn agentic coding. This has two implications: giving developers a snappy experience when interacting with their coding agents, and capitalizing on the fact that agentic workloads with large context and tool outputs typically carry large repeated contexts forward at each turn.
* **(d) Chunked Prefill:** We chunk large prompt prefills into smaller stages so that long input requests do not stall active token generation streams, allowing us to serve more developers with a reasonable inter-token latency (ITL).
* **(e) Language Model Only Mode:** We disable multimodal image/video feature extractors, freeing memory and compute pipelines to focus solely on text/code.

In order to test our serving performance, we leverage two benchmarks—one for medium context tasks (such as inline completion, single file assistance, class/method completions), and one for long context tasks (for multi-file feature implementations, extended call stack debugging, and long conversation histories). With the medium context benchmark (6.4k input tokens), we evaluate fast TTFT and ITL (perceived by developers as token streaming speeds); while the long context benchmark evaluates KV cache performance under heavy prefill pressure. Below are the results across these two types of workloads with a concurrency set to 120 streams/developers.

### Performance at 120 Concurrency (TPU v6e-4, TP=4)

| Concurrency Level | Context Size | Mean TTFT | Mean TPOT | Request Throughput | Output Token Throughput | Total Throughput / Chip |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **Concurrency = 120** | **6.4K Medium Context** | 503.85 ms | 30.17 ms | 6.07 req/s | 1,499.77 tok/s | 374.94 tok/s/chip |
| **Concurrency = 120** | **26.8K Long Context** | 827.27 ms | 39.47 ms | 4.36 req/s | 1,116.49 tok/s | 279.12 tok/s/chip |

### Key Operational Takeaways

1. **Sub-Second TTFT at Scale:** Both short and long context workloads maintain sub-second Time-To-First-Token (**503ms–827ms**), significantly enriching developer responsiveness.
2. **Blazing Output Throughput (1,100–1,500 tok/s):** Generating ~1,500 tokens per second equates to roughly **1,125 words per second**—vastly outpacing human reading speed.
3. **High Request Processing (4–6 req/s):** Processing 4 to 6 requests per second per node easily exceeds the typical human pacing requirement (where developers review code and formulate follow-ups every 15–20 seconds). This allows high developer density per TPU node without queuing lag.

---

# Orchestration, Harness Integration, and User Experience Optimizations

Now that we have an optimized serving recipe, it is important to ensure that serving enables a high-quality user experience for developers while also fixing some of the quirks of a smaller model. For example: Qwen 3 models are fine-tuned to emit tool calls using native XML tags, whereas a lot of standard tool-calling APIs (like OpenAI) format function calls as JSON strings. 

We integrate a custom Jinja Chat Template (courtesy of [Froggeric Qwen Fixed Templates](https://huggingface.co/froggeric/Qwen-Fixed-Chat-Templates)) that dictates how prompt conversations, system instructions, and tool definitions are formatted before entering the model. This also enables us to isolate reasoning blocks (which a lot of harnesses render in the TUI differently) and automated failure warning injections (for 2+ consecutive tool call failures).

We also configure a server-side token parser to intercept and convert XML tool tags on the fly to standard OpenAI JSON `tool_calls` responses. This enables this recipe to be utilized by a variety of open-source harnesses that expect OpenAI-style JSON API responses.

Lastly, we enable dynamic tool choice, which the client can drive to either force `tool_choice` or leave it to the model as automatic.

From an end-to-end recipe perspective, the proof is in the pudding in how well this integrates with an open-source harness. Therefore, we tested this recipe leveraging the open-source [Pi harness](https://pi.dev) with a variety of coding tasks to ensure that user experience is indeed seamless and intuitive, and capabilities such as tool calling, reasoning blocks, MCP integration, skills injection, context compaction etc. work effortlessly. The setup guide for Pi is included in the recipe repository.

---

# Pricing and Final Observations 

Last, but not least, and to wrap up with the promise we made at the beginning of this blog, this setup enables you to run **24/7 coding agents** across short, medium, and long context coding tasks across a **team of 120 developers**. 

Since both the **Qwen 3.6 27B** model and the **Pi harness** are fully licensed and available open-source for commercial and enterprise use, your primary operational cost stems from the TPU commitment. With a **3-year commitment on a 4-chip TPU v6e configuration**, you are looking at approximately **~$30 per month per developer**—which is easily **10-15x cheaper** than API-based closed-source coding agent competitors, particularly when considering token-based billing. 

Having a dedicated TPU cluster also enables you to run remote coding agents across your SWE domain **around the clock**, helping triage production challenges, resolve backlogged bugs, and engage in experimental spikes—ultimately accelerating your product and revenue roadmap.
