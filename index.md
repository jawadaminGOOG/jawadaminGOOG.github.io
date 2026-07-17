---
layout: default
title: Home
---

# Jawad Amin

<p class="lede">Customer Engineer at Google | AI Infrastructure</p>

Welcome to my personal site for AI hardware benchmarking, TPU recipes, and engineering notes.

---

## Blogs & Notes

{% for post in site.posts %}
* **[{{ post.date | date: "%B %d, %Y" }}]** — [{{ post.title }}]({{ post.url | relative_url }})
  <br><sub>{{ post.description }}</sub>
{% else %}
*No posts published yet. Stay tuned!*
{% endfor %}

---

## Public Repositories

* **[qwen3.6-27B-tpu-pi-agent](https://github.com/jawadaminGOOG/qwen3.6-27B-tpu-pi-agent)**  
  Benchmarking infrastructure and setup recipes for serving Qwen 3.6 27B on Cloud TPUs.
* **[tpu-recipes](https://github.com/jawadaminGOOG/tpu-recipes)**  
  Collection of Google Cloud TPU optimization recipes and benchmarks.

[View all public Google repositories on GitHub ↗](https://github.com/jawadaminGOOG?tab=repositories)

---

## Past Work (Microsoft AI Accelerators)

* **[WorkbenchIQ](https://github.com/JawadAminMSFT/WorkbenchIQ)**  
  Microsoft accelerator providing a modern workbench for underwriters and claims processors, combining Azure AI Content Understanding and Azure AI Foundry.
* **[agentic-diligence-ibanking](https://github.com/JawadAminMSFT/agentic-diligence-ibanking)**  
  AI-first agentic M&A due diligence platform powered by the GitHub Copilot SDK.
* **[agentic-rcsa](https://github.com/JawadAminMSFT/agentic-rcsa)**  
  Intelligent Risk and Control Self Assessment Accelerator with multi-agent architecture leveraging the OpenAI Agents SDK.
* **[sage-retirement-planning](https://github.com/JawadAminMSFT/sage-retirement-planning)**  
  Personalized retirement scenario planning, financial projections, and AI-powered recommendations.
* **[mike-ai-agent](https://github.com/JawadAminMSFT/mike-ai-agent)**  
  Azure OpenAI Speech-to-Action Agent Implementation.
* **[credit-card-agentic](https://github.com/JawadAminMSFT/credit-card-agentic)**  
  AI-powered credit card application system built with Microsoft Agent Framework for autonomous multi-agent approvals.
* **[document_indexer_gpt](https://github.com/JawadAminMSFT/document_indexer_gpt)**  
  PDF data extraction application using Azure OpenAI vision models.

[View all past Microsoft repositories on GitHub ↗](https://github.com/JawadAminMSFT?tab=repositories)
