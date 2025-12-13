# RobustLight: Diffusion-Based Robust Traffic Signal Control

This repository provides the official implementation of **RobustLight**, a diffusion-based framework for **robust traffic signal control under corrupted or missing perception data**.  
The core idea is to leverage a **diffusion model for traffic state recovery**, enabling downstream reinforcement learning (RL) controllers to operate reliably under noise, masking, and adversarial disturbances.

The framework supports **training**, **inference (state recovery)**, and **evaluation** under multiple noise settings, consistent with the experimental setup described in the paper.

---

## 📌 Key Features

- Diffusion-based traffic state recovery  
- Robustness to multiple noise types and masking
- Seamless integration with RL traffic signal controllers
- Support for **training-only**, **inference-only**, or **end-to-end evaluation**
- Configurable experimental parameters aligned with the paper

---

## 🧠 Overall Pipeline

1. **Noise Injection**  
   Raw traffic states are corrupted by configurable noise or masking.

2. **Diffusion Model**
   - Trained to reconstruct clean traffic states from corrupted inputs.
   - Can be **trained**, **frozen**, or **used only for inference**.

3. **Recovered States**
   - Output of the diffusion model.
   - Used as inputs to downstream traffic signal control algorithms.

4. **Traffic Signal Control**
   - RL-based controller operates on recovered states.
   - Performance is evaluated under different noise conditions.

---

## ⚙️ Main Configuration Parameters

### 1️⃣ Diffusion Training & Inference Control

| Parameter | Type | Description |
|--------|------|------------|
| `is_test` | bool | Controls whether the diffusion model is trained. <br>• `False`: Train the diffusion model <br>• `True`: Skip training and load a pretrained model |
| `inference_epoch` | int | Number of diffusion training epochs. <br>**Set to 50 in the paper.** |
| `is_inference` | bool | Controls whether diffusion-based inference (state recovery) is performed. <br>• `True`: Enable state recovery <br>• `False`: Use noisy states directly |

---

### 2️⃣ Noise Configuration

#### Noise Type (`NOISE_TYPE`)

| Value | Noise Type | Description |
|------|-----------|------------|
| `0` | Gaussian | Additive Gaussian noise |
| `1` | Uniform | Uniform random noise |
| `2` | Q-Min | Minimum Q-value–based perturbation |
| `3` | Action-Diff | Action difference–based noise |

#### Noise Level (`noise_level`)

| Value | Meaning | Description |
|------|--------|------------|
| `0` | Noise | Additive noise corruption |
| `1` | Mask | Feature masking (missing data) |

> **Note:**  
> `noise_level = 1` simulates sensor failure or missing perception data, while  
> `noise_level = 0` simulates noisy but available sensor measurements.

---

## 🚀 Typical Usage Scenarios

### 🔹 1. Train Diffusion Model Only
```python
is_test = False
is_inference = False
inference_epoch = 50

