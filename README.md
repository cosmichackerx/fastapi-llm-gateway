# fastapi-llm-gateway
Lightweight AI inference bridge for running LLMs and Stable Diffusion on AWS CPU instances using llama.cpp, stable-diffusion.cpp, and FastAPI
---
**fastapi-llm-gateway** is a high-performance, lightweight AI inference bridge designed to run Large Language Models (LLMs) and Image Generation models on resource-constrained environments like AWS CPU instances. By leveraging llama.cpp and stable-diffusion.cpp, this system provides a unified FastAPI gateway for text and image generation.


## 🚀 Features
* **Brain Node:** Powered by Llama 3.2 1B for lightning-fast text generation.
* **Artist Node:** Powered by SD-Turbo with 384px CPU optimization.
* **The Bridge:** A FastAPI-based REST API with Bearer Token security.
* **Resource Efficient:** Optimized for AWS t3/t2 instances (vCPU/RAM focused).
