# LLM Router VWS Toolkit Plan

## Vision
Transform this blueprint into a toolkit where a Virtual Workstation (VWS) with an L40S GPU
hosts a lightweight local reasoning/routing model that triages queries to LLMs on a larger remote server.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                   VIRTUAL WORKSTATION (L40S - 48GB)             │
│                                                                 │
│   ┌─────────────────────┐      ┌─────────────────────────────┐  │
│   │   Router Backend    │ ───▶ │   Routing Model (choose 1)  │  │
│   │   (FastAPI, 8001)   │      │   • Qwen 1.7B (~4GB VRAM)   │  │
│   │   ~minimal VRAM     │      │   • CLIP server (~2GB VRAM) │  │
│   └──────────┬──────────┘      └─────────────────────────────┘  │
│              │                                                  │
│              │ Returns model name (e.g., "nvidia/nemotron-9b")  │
│              ▼                                                  │
│   ┌─────────────────────┐                                       │
│   │   Application       │  Makes API call to recommended model  │
│   │   (Demo UI / Custom)│                                       │
│   └──────────┬──────────┘                                       │
└──────────────┼──────────────────────────────────────────────────┘
               │
               │  OpenAI-compatible API request
               ▼
┌─────────────────────────────────────────────────────────────────┐
│                   REMOTE SERVER (NIMs / Cloud)                  │
│                                                                 │
│   ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│   │ Nemotron-9B     │  │ Nemotron-12B-VL │  │ GPT-5 / Other  │  │
│   │ (simple tasks)  │  │ (vision tasks)  │  │ (hard tasks)   │  │
│   └─────────────────┘  └─────────────────┘  └────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Component GPU Memory Requirements

| Component              | Actual VRAM  | Why                                           |
|------------------------|--------------|-----------------------------------------------|
| Qwen 1.7B (FP16)       | ~3.5-4GB     | 1.7B params × 2 bytes + KV cache overhead     |
| Qwen 1.7B (INT8/INT4)  | ~2-2.5GB     | Quantized inference reduces memory            |
| CLIP ViT-L/14          | ~1.5-2GB     | 400M param vision encoder                     |
| NN Router inference    | 0 (CPU)      | Small PyTorch model runs on CPU               |
| Router Backend         | ~minimal     | FastAPI service, no model weights             |

**Note**: README lists 16GB as requirement for T4 minimum - this is conservative/safe margin.
Actual usage is much lower. L40S (48GB) has massive headroom.

## Current Testing Phase (Single Machine)

For now, everything runs on this L40S machine:
1. Router Backend + Routing Model (VWS role)
2. Local NIMs simulating remote server (temporary)
3. Demo application for testing

## Future Production State

1. VWS (L40S): Router only - fast local triage decisions
2. Remote Server: NIMs (Nemotron-9B, Nemotron-12B-VL, etc.)
3. Cloud APIs: Azure OpenAI / NVIDIA Build API (optional)

## Next Steps

- [ ] Test intent-based router (Qwen 1.7B) on this machine
- [ ] Verify actual VRAM usage with nvidia-smi
- [ ] Configure model endpoints for local NIMs
- [ ] Test full routing flow end-to-end
- [ ] Document actual memory usage for toolkit
- [ ] Prepare configuration for remote NIM deployment