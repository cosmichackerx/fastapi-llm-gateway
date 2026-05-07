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

**1. Instance Setup**

- **AMI**: Ubuntu 24.04 LTS (HVM)  
- **Instance Type**: `t3.medium` or `t3.large`  
  - **Recommended**: `m7i-flex.large`  
- **Storage**: 20GB+ (SSD gp3)

---

**2. Network Security (Security Group)**

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

> After configuration, launch an instance and wait until it is marked as **Running**.

# 🔑 Step 2: Connection & Local Setup

**1. Connect to Your Server**
From your local **Windows/Linux terminal**:

```bash
# Navigate to the folder containing your .pem key
ssh -i "your-cosmic-key.pem" ubuntu@your-aws-public-ip
```

**2. Initial Server Preparation**
Run the following commands to update and install essential packages:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential cmake git python3-pip python3-venv -y
```

# 🧠 Step 3: Text Engine Setup (llama.cpp)

**1. Build the Engine**

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

**2. Download the Optimized Model**

```bash
cd models
curl -L -o llama-3.2-1b-instruct-q4_k_m.gguf https://huggingface.co/unsloth/Llama-3.2-1B-Instruct-GGUF/resolve/main/Llama-3.2-1B-Instruct-Q4_K_M.gguf
```
