# fastapi-llm-gateway
Lightweight AI inference bridge for running LLMs and Stable Diffusion on AWS CPU instances using llama.cpp, stable-diffusion.cpp, and FastAPI
---
**fastapi-llm-gateway** is a high-performance, lightweight AI inference bridge designed to run Large Language Models (LLMs) and Image Generation models on resource-constrained environments like AWS CPU instances. By leveraging llama.cpp and stable-diffusion.cpp, this system provides a unified FastAPI gateway for text and image generation.


## 🚀 Features
* **Brain Node:** Powered by Llama 3.2 1B for lightning-fast text generation.
* **Artist Node:** Powered by SD-Turbo with 384px CPU optimization.
* **The Bridge:** A FastAPI-based REST API with Bearer Token security.
* **Resource Efficient:** Optimized for AWS t3/t2 instances (vCPU/RAM focused).



# 🛠️ Step 1: AWS Server Configuration

## 1. Instance Setup
- **AMI**: Ubuntu 24.04 LTS (HVM)  
- **Instance Type**: `t3.medium` or `t3.large`  
  - **Recommended**: `m7i-flex.large`  
- **Storage**: 20GB+ (SSD gp3)

---

## 2. Network Security (Security Group)
You must configure the **Inbound Rules** to allow traffic to the API and the internal engines.

### Suggested Rules:
- **HTTP (Port 80)** → Allow from `0.0.0.0/0`  
- **HTTPS (Port 443)** → Allow from `0.0.0.0/0`  
- **Custom TCP (API Port)** → Allow from trusted IP ranges  
- **SSH (Port 22)** → Restrict to your admin IP only  

> ⚠️ Always follow the principle of least privilege when setting inbound rules.

You must configure the **Inbound Rules** to allow traffic to the API and the internal engines:

| Protocol     | Port Range | Source        | Description              |
|--------------|------------|---------------|--------------------------|
| **SSH**      | 22         | My IP         | Remote Access            |
| **Custom TCP** | 8000     | 0.0.0.0/0     | Cosmic Bridge (API)      |
| **Custom TCP** | 8080     | 127.0.0.1/32  | Internal Text Engine     |
