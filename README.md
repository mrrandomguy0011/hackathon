# 🚙 SegFormer: Advanced Off-Road Semantic Segmentation
👋 **Welcome to our 2026 Hackathon Project!**

Ever tried to teach a computer the difference between a dry bush and a rock covered in dirt? It’s tough. This repository contains our journey to build an incredibly fast, highly accurate semantic segmentation pipeline for extreme off-road environments. 

Built under the wire during a time-crunched hackathon, we evolved this project from a slow, "blobby" prototype into a lightning-fast, edge-detecting powerhouse optimized for dual-GPU training. 

---

## 🏕️ Our Journey: From Blobs to Boundaries

Building this wasn't a straight line. We went through three major phases of trial, error, and optimization:

### Phase 1: The "Blob" Baseline (DINOv2)
* **The Setup:** We started with a frozen DINOv2 backbone and a custom ConvNeXt head.
* **The Result:** It successfully learned the macro layout (Sky goes up, Dirt goes down), but DINOv2's `14x14` patch size meant our masks looked like melted watercolor paintings. We couldn't detect fine details like rocks or logs.

### Phase 2: The High-Res Pivot (SegFormer-B3)
* **The Setup:** We pivoted to a full SegFormer architecture to recover pixel-perfect boundaries at a 512x512 resolution.
* **The Result:** The math was beautiful, but the hardware hated it. Pushing high-res transformer attention maps across 10 classes instantly blew past our 16GB GPU VRAM and 30GB System RAM limits. *Kernel died.* 💀

### Phase 3: "Lightning Mode" ⚡ (The Winning Formula)
* **The Setup:** We scaled down to **SegFormer-B0** and reduced the image size to `384x384`. We implemented Automatic Mixed Precision (AMP), OneCycleLR scheduling, and aggressively patched RAM leaks in our DataLoaders.
* **The Result:** A highly aggressive, perfectly balanced pipeline that trains a complete, high-accuracy segmentation model in **under 30 minutes** on dual NVIDIA T4 GPUs. 

---

## 🛠️ The Tech Stack & Clever Hacks

To make this work in a hackathon setting, we had to engineer a few creative workarounds:
* **The 16-Bit Mask Rescue:** Standard OpenCV libraries silently crush 16-bit integer labels (like our `10000` value for Sky) into 8-bit garbage, destroying the dataset. We built a custom DataLoader using `cv2.IMREAD_UNCHANGED` and hardcoded tensor mapping to save our labels.
* **Taming the RAM Monster:** Multi-GPU `DataParallel` is great, but spinning up background workers to feed those GPUs causes catastrophic System RAM leaks. We perfectly balanced CPU loading and GPU batching to maintain 100% compute utilization without crashing the server.

---

## 📊 How'd We Do? (Metrics)

*Note: Off-road datasets are notoriously unbalanced (lots of sky, very few rocks). Our metrics reflect a heavy focus on navigational accuracy.*

| What We're Looking At | Mean IoU | How it actually looks |
| :--- | :--- | :--- |
| **Sky ☁️** | 0.982 | Near-perfect boundary detection. |
| **Trees 🌲** | 0.712 | Sharp edge recovery on foliage and branches. |
| **Landscape 🏜️** | 0.684 | Highly consistent spatial mapping for navigation. |
| **Dry Grass 🌾** | 0.610 | Strong performance, with minor confusion near dirt edges. |
| **Rocks 🪨** | 0.114 | *Hackathon limitation:* We need more training data to boost this! |
| **Overall mIoU** | **~0.540** | **Highly competitive for a 30-minute training sprint!** |

---

---

## 🚀 Get Started (Try it yourself!)

Want to run our Lightning Mode pipeline? It's super easy to set up.

**1. Install the good stuff:**
 - bash
pip install transformers albumentations segmentation-models-pytorch
2. Train the model (Grab a coffee, it takes <30 mins):

Bash
python train_lightning.py
3. See the magic (Run inference):

Bash
python inference.py --model_path ./outputs/best_model.pth --data_dir ./data/val
Built with lots of caffeine, two Kaggle GPUs, and a dream during the 2026 Hackathon. ✌️


***

This version adds a lot of personality, uses formatting to make it highly readable, and explains your complex engineering fixes in a way that anyone can understand and appreciate. 

Are you ready to zip this all up, or do you need me to help you draft the short blurb to submit on the actual hackathon website?
