# AutoDL + ComfyUI + MiniMax H3

> AutoDL 环境下部署、存储迁移、MiniMax H3 模型下载与 ComfyUI 启动记录。

---

## 📑 目录

* [1. 环境信息](#1-环境信息)
* [2. AutoDL 网络加速](#2-autodl-网络加速)
* [3. ComfyUI 启动](#3-comfyui-启动)
* [4. 存储检查](#4-存储检查)
* [5. 当前存储结构](#5-当前存储结构)
* [6. MiniMax H3 模型](#6-minimax-h3-模型)
* [7. Hugging Face 下载](#7-hugging-face-下载)
* [8. hf-mirror 下载](#8-hf-mirror-下载)
* [9. 下载失败检查](#9-下载失败检查)
* [10. 将 ComfyUI Models 迁移到数据盘](#10-将-comfyui-models-迁移到数据盘)
* [11. 测试软链接](#11-测试软链接)
* [12. 删除旧的 Models 备份](#12-删除旧的-models-备份)
* [13. MiniMax H3 工作流](#13-minimax-h3-工作流)
* [14. GPU 实例检查](#14-gpu-实例检查)
* [15. GPU 实例启动 ComfyUI](#15-gpu-实例启动-comfyui)
* [16. 重要结论](#16-重要结论)
* [17. 常用命令速查](#17-常用命令速查)

---

## 1. 环境信息

| 项目         | 信息                    |
| ---------- | --------------------- |
| ComfyUI 路径 | `/root/comfy/ComfyUI` |
| ComfyUI    | `0.37.0`              |
| comfy-cli  | `1.20.0`              |
| Python     | `3.12.3`              |
| PyTorch    | `2.12.1+cu130`        |

---

## 2. AutoDL 网络加速

启用 AutoDL 网络加速：

```bash
source /etc/network_turbo
```

检查代理环境：

```bash
env | grep -i proxy
```

---

## 3. ComfyUI 启动

### CPU 模式

无 GPU 时：

```bash
comfy launch -- --cpu --listen 0.0.0.0 --port 6006
```

### GPU 模式

有 GPU 时：

```bash
comfy launch -- --listen 0.0.0.0 --port 6006
```

> 默认端口：`6006`

---

## 4. 存储检查

### 查看磁盘

```bash
lsblk
```

### 查看文件系统

```bash
lsblk -f
```

### 查看所有磁盘容量

```bash
df -hT
```

### 查看系统盘

```bash
df -h /root/comfy/ComfyUI
```

### 查看本地数据盘

```bash
df -h /root/autodl-tmp
```

### 查看挂载情况

```bash
mount | grep -E 'autodl|nfs|sda'
```

---

## 5. 当前存储结构

| 路径                    | 类型    |    容量/状态 | 说明          |
| --------------------- | ----- | -------: | ----------- |
| `/root/comfy/ComfyUI` | 系统盘   |  ≈ 30 GB | ComfyUI 主目录 |
| `/root/autodl-tmp`    | 本地数据盘 |    50 GB | 可写          |
| `/autodl-pub`         | 公共数据盘 |       只读 | 不建议存个人模型    |
| `/root/autodl-fs`     | 文件存储  |      不存在 | 当前未使用       |
| `/dev/sda2`           | 本地磁盘  | ≈ 446 GB | 当前未挂载       |

### 测试 `/autodl-pub` 是否可写

```bash
touch /autodl-pub/test_write_$$ \
  && echo "可写" \
  && rm /autodl-pub/test_write_$$
```

> 如果测试失败，说明 `/autodl-pub` 不具备写权限。

---

## 6. MiniMax H3 模型

MiniMax H3 当前使用的模型文件如下：

| 类型           | 文件                                                           |             大小 |
| ------------ | ------------------------------------------------------------ | -------------: |
| Diffusion    | `minimax_h3_ref2va_pruned_int8_convrot.safetensors`          |        ≈ 20 GB |
| Text Encoder | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`               |      ≈ 15.7 GB |
| Video VAE    | `minimax_h3_video_vae_fp16.safetensors`                      |      ≈ 5.21 GB |
| Audio VAE    | `minimax_h3_audio_vae_fp32.safetensors`                      |       ≈ 605 MB |
| Turbo LoRA   | `minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors` |      ≈ 1.96 GB |
| **总计**       |                                                              | **≈ 43.5 GB+** |

### 模型目录

```text
/root/comfy/ComfyUI/models/
├── diffusion_models/
├── text_encoders/
├── vae/
└── loras/
```

---

## 7. Hugging Face 下载

曾尝试直接使用 `hf`：

```bash
hf download Comfy-Org/MiniMax-H3 \
  diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors \
  --local-dir /root/comfy/ComfyUI/models
```

遇到：

```text
401 Unauthorized
CAS Client Error
cas-server.xethub.hf.co
```

因此暂时不使用：

```text
hf + Xet
```

---

## 8. hf-mirror 下载

### 8.1 Diffusion

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors \
  --relative-path models/diffusion_models \
  --filename minimax_h3_ref2va_pruned_int8_convrot.safetensors
```

### 8.2 Text Encoder

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors \
  --relative-path models/text_encoders \
  --filename qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
```

### 8.3 Video VAE

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors \
  --relative-path models/vae \
  --filename minimax_h3_video_vae_fp16.safetensors
```

### 8.4 Audio VAE

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors \
  --relative-path models/vae \
  --filename minimax_h3_audio_vae_fp32.safetensors
```

### 8.5 Turbo LoRA

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/loras/minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors \
  --relative-path models/loras \
  --filename minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors
```

---

## 9. 下载失败检查

### 查看模型目录大小

```bash
du -sh /root/comfy/ComfyUI/models/diffusion_models/*
```

### 删除失败的 `.part` 文件

```bash
rm -f /root/comfy/ComfyUI/models/diffusion_models/*.part
```

> 如果下载中断后重新下载，建议先检查并清理残留的 `.part` 文件。

---

## 10. 将 ComfyUI Models 迁移到数据盘

由于 MiniMax H3 模型总大小约 **43.5 GB+**，不建议将完整模型放在系统盘。

### 10.1 创建数据盘目录

```bash
mkdir -p /root/autodl-tmp/ComfyUI/models
```

### 10.2 检查原 Models 目录

```bash
ls -ld /root/comfy/ComfyUI/models
```

### 10.3 将模型复制到数据盘

```bash
cp -a /root/comfy/ComfyUI/models/. \
  /root/autodl-tmp/ComfyUI/models/
```

### 10.4 检查复制结果

```bash
du -sh /root/autodl-tmp/ComfyUI/models
```

### 10.5 备份原目录

确认复制完成后：

```bash
mv /root/comfy/ComfyUI/models \
  /root/comfy/ComfyUI/models.bak
```

### 10.6 创建软链接

```bash
ln -s /root/autodl-tmp/ComfyUI/models \
  /root/comfy/ComfyUI/models
```

### 10.7 检查软链接

```bash
ls -ld /root/comfy/ComfyUI/models
```

正常情况下应显示：

```text
/root/comfy/ComfyUI/models
-> /root/autodl-tmp/ComfyUI/models
```

---

## 11. 测试软链接

创建测试文件：

```bash
touch /root/comfy/ComfyUI/models/test.txt
```

检查数据盘：

```bash
ls -l /root/autodl-tmp/ComfyUI/models/test.txt
```

删除测试文件：

```bash
rm /root/comfy/ComfyUI/models/test.txt
```

如果能够正常创建、查看和删除，说明软链接工作正常。

---

## 12. 删除旧的 Models 备份

### 确认当前 Models

```bash
ls -ld /root/comfy/ComfyUI/models
```

### 检查备份大小

```bash
du -sh /root/comfy/ComfyUI/models.bak
```

确认数据盘中的模型完整、ComfyUI 可以正常读取后，再删除备份：

```bash
rm -rf /root/comfy/ComfyUI/models.bak
```

> ⚠️ `rm -rf` 不可恢复。
> 建议确认软链接、模型文件和 ComfyUI 均正常后再执行。

---

## 13. MiniMax H3 工作流

### 工作流文件

```text
video_minimax_h3_multiframe_reference.json
```

### 下载工作流

进入 ComfyUI 目录：

```bash
cd /root/comfy/ComfyUI
```

下载：

```bash
wget -O video_minimax_h3_multiframe_reference.json \
  https://raw.githubusercontent.com/Comfy-Org/workflow_templates/main/templates/video_minimax_h3_multiframe_reference.json
```

### 官方文档

[MiniMax H3 Multiframe 官方文档](https://docs.comfy.org/tutorials/video/minimax/minimax-h3-multiframe?utm_source=chatgpt.com)

---

## 14. GPU 实例检查

启动 ComfyUI 前，建议依次检查：

### GPU

```bash
nvidia-smi
```

### 磁盘空间

```bash
df -h
```

### 磁盘与文件系统

```bash
lsblk -f
```

### Models 软链接

```bash
ls -ld /root/comfy/ComfyUI/models
```

### Models 文件

```bash
ls -lah /root/comfy/ComfyUI/models
```

---

## 15. GPU 实例启动 ComfyUI

确认 GPU、磁盘和 Models 均正常后：

```bash
comfy launch -- --listen 0.0.0.0 --port 6006
```

---

## 16. 重要结论

### 存储规划

| 存储位置                  | 状态           | 建议          |
| --------------------- | ------------ | ----------- |
| `/root/comfy/ComfyUI` | 系统盘，≈ 30 GB  | 放程序，不放完整 H3 |
| `/root/autodl-tmp`    | 数据盘，50 GB    | 可用于存放模型     |
| `/autodl-pub`         | 公共盘，只读       | 不存个人模型      |
| `/root/autodl-fs`     | 当前不存在        | 暂不使用        |
| `/dev/sda2`           | ≈ 446 GB，未挂载 | 当前未使用       |

### MiniMax H3

> **MiniMax H3 模型总大小约 43.5 GB+。**

因此：

* ❌ 不建议把完整 H3 放在系统盘
* ❌ 不要把个人模型放到 `/autodl-pub`
* ⚠️ 50 GB 数据盘放完整 H3 后剩余空间很小
* ✅ 优先使用数据盘保存 Models
* ✅ 使用软链接让 ComfyUI 继续访问原路径

推荐结构：

```text
/root/comfy/ComfyUI/models
        │
        │ symbolic link
        ▼
/root/autodl-tmp/ComfyUI/models
        ├── diffusion_models/
        ├── text_encoders/
        ├── vae/
        └── loras/
```

---

## 17. 常用命令速查

### 💾 磁盘

```bash
df -h
df -hT
lsblk
lsblk -f
```

### 📂 挂载

```bash
mount | grep -E 'autodl|nfs|sda'
```

### 🤖 Models

```bash
du -sh /root/comfy/ComfyUI/models
du -sh /root/autodl-tmp/ComfyUI/models
ls -ld /root/comfy/ComfyUI/models
ls -lah /root/comfy/ComfyUI/models
```

### 🎮 GPU

```bash
nvidia-smi
```

### 🚀 启动 ComfyUI

```bash
comfy launch -- --listen 0.0.0.0 --port 6006
```

---

## 附：一键检查环境

如果只是想快速确认当前实例状态，可以执行：

```bash
echo "===== GPU ====="
nvidia-smi

echo
echo "===== DISK ====="
df -h

echo
echo "===== BLOCK DEVICE ====="
lsblk -f

echo
echo "===== COMFYUI MODELS ====="
ls -ld /root/comfy/ComfyUI/models

echo
echo "===== MODEL SIZE ====="
du -sh /root/comfy/ComfyUI/models 2>/dev/null
```
