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

## 🎨 Step 4: Image Engine Setup (stable-diffusion.cpp)

**1. Build the Engine**
```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

**2. Download SD-Turbo**
```bash
mkdir models && cd models
# Download the SD-Turbo Safetensors from HuggingFace
curl -L -o sd_turbo.safetensors https://huggingface.co/stabilityai/sd-turbo/resolve/main/sd_turbo.safetensors
```

# 🌉 Step 5: The Cosmic Bridge (FastAPI)

**1. Environment Setup**
```bash
cd ~
python3 -m venv cosmic_env
source cosmic_env/bin/activate
pip install fastapi uvicorn requests pydantic
```

**2. Create main.py**

The **main.py** acts as the traffic controller, routing requests to the local engines running on ports 8080 and subprocesses.

```bash
cd ~
/nano main.py
```

---
```python
import os, json, subprocess, requests, socket, time
from fastapi import FastAPI, Header, HTTPException
from fastapi.responses import StreamingResponse, FileResponse
from pydantic import BaseModel
from typing import Optional

app = FastAPI(title="Cosmic Army X - Titan Lite Bridge")

# --- CONFIGURATION ---
SECRET_KEY = os.environ.get("MOCK_CLIENT_SECRET", "mock_api_key_12345")
TEXT_URL = "http://127.0.0.1:8080/v1/chat/completions"
SD_PATH = "/home/mockuser/stable-diffusion.cpp/build/bin/sd-cli"
SD_MODEL = "/home/mockuser/stable-diffusion.cpp/models/sd_turbo.safetensors"

class RequestData(BaseModel):
    inputs: Optional[str] = None
    prompt: Optional[str] = None

def verify_auth(auth: str):
    if not auth or auth != f"Bearer {SECRET_KEY}":
        raise HTTPException(status_code=401, detail="Unauthorized")

def is_port_open(port: int):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.settimeout(1.0)
        return s.connect_ex(('127.0.0.1', port)) == 0

# --- 1. SYSTEM STATUS (Health Check) ---
@app.get("/api/status")
async def get_status(authorization: str = Header(None)):
    verify_auth(authorization)
    llama_online = is_port_open(8080)
    return {
        "status": "Titan-Lite Online",
        "text_engine_8080": "ONLINE" if llama_online else "OFFLINE",
        "image_engine": "READY (384px Boost)",
        "timestamp": int(time.time())
    }

# --- 2. TEXT GENERATION ---
@app.post("/api/generate")
async def generate(data: RequestData, authorization: str = Header(None)):
    verify_auth(authorization)
    if not data.inputs:
        raise HTTPException(status_code=400, detail="Missing inputs")
    
    payload = {
        "messages": [{"role": "user", "content": data.inputs}],
        "stream": True, "temperature": 0.7, "max_tokens": 512
    }
    
    def stream_logic():
        try:
            with requests.post(TEXT_URL, json=payload, stream=True, timeout=30) as r:
                r.raise_for_status()
                for line in r.iter_lines():
                    if line: yield f"{line.decode('utf-8')}\n\n"
        except Exception as e:
            yield f"data: {{\"error\": \"Text Engine Error: {str(e)}\"}}\n\n"
            
    return StreamingResponse(stream_logic(), media_type="text/event-stream")

# --- 3. IMAGE GENERATION ---
@app.post("/api/image")
async def generate_image(data: RequestData, authorization: str = Header(None)):
    verify_auth(authorization)
    if not data.prompt:
        raise HTTPException(status_code=400, detail="Missing prompt")
        
    output_file = "/home/mockuser/stable-diffusion.cpp/api_output.png"
    # Optimized for CPU Speed: 3 steps + 384px canvas
    cmd = [
        SD_PATH, "-m", SD_MODEL, "-p", data.prompt, 
        "--steps", "3", "-o", output_file, 
        "--width", "384", "--height", "384", "-t", "2"
    ]
    try:
        subprocess.run(cmd, check=True)
        return FileResponse(output_file, media_type="image/png")
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## Saving & Exiting Nano

After editing your main.py :

1. Press **Ctrl + O** to save changes.
2. Hit **Enter** to confirm the filename.
3. Press **Ctrl + X** to exit Nano.

---

```bash
# Launch with: 
# uvicorn main:app --host 0.0.0.0 --port 8000
```

# ⚡ Step 6: Launch Protocol
**Before we launch, remember these three "Golden Commands":**
- Detach (Hide the screen): Press **CTRL + B**, then let go, then immediately press **D**.

- List Sessions:
```bash
tmux ls
```

- Re-attach (Open the screen): 
```bash
tmux attach -t session_name
```

**🚀 Part 1: Launch the Brain (Text Engine)**
This session handles the Llama 3.2 1B model.
- Create the session:
  ```bash
  tmux new -s text_engine
  ```
  
- Run the Launch Command:
  ```bash
  cd ~/llama.cpp
  ./build/bin/llama-server -m models/llama-3.2-1b-instruct-q4_k_m.gguf --port 8080 --host 0.0.0.0 -c 2048 -t 2 --flash-attn on
  ```

> Wait for the green light: Once you see main: server is listening on **http://0.0.0.0:8080**, hide the session.
> Detach: Press **CTRL + B**, then *D*.
