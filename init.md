
# AutoDL + ComfyUI + MiniMax H3 命令记录

## 1. 环境

```text
ComfyUI: /root/comfy/ComfyUI
ComfyUI: 0.37.0
comfy-cli: 1.20.0
Python: 3.12.3
PyTorch: 2.12.1+cu130
````

---

## 2. AutoDL 网络加速

```bash
source /etc/network_turbo
```

```bash
env | grep -i proxy
```

---

## 3. ComfyUI 启动

无 GPU：

```bash
comfy launch -- --cpu --listen 0.0.0.0 --port 6006
```

GPU：

```bash
comfy launch -- --listen 0.0.0.0 --port 6006
```

---

## 4. 存储检查

查看磁盘：

```bash
lsblk
```

查看文件系统：

```bash
lsblk -f
```

查看全部容量：

```bash
df -hT
```

查看系统盘：

```bash
df -h /root/comfy/ComfyUI
```

查看本地数据盘：

```bash
df -h /root/autodl-tmp
```

查看挂载：

```bash
mount | grep -E 'autodl|nfs|sda'
```

---

## 5. 当前存储

```text
/root/comfy/ComfyUI
系统盘
约 30GB

/root/autodl-tmp
本地数据盘
50GB
可写

/autodl-pub
公共数据盘
只读

/root/autodl-fs
当前不存在

/dev/sda2
约 446GB
当前未挂载
```

测试 `/autodl-pub`：

```bash
touch /autodl-pub/test_write_$$ && echo "可写" && rm /autodl-pub/test_write_$$
```

---

## 6. MiniMax H3 模型

### Diffusion

```text
minimax_h3_ref2va_pruned_int8_convrot.safetensors
约 20GB
```

### Text Encoder

```text
qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
约 15.7GB
```

### Video VAE

```text
minimax_h3_video_vae_fp16.safetensors
约 5.21GB
```

### Audio VAE

```text
minimax_h3_audio_vae_fp32.safetensors
约 605MB
```

### Turbo LoRA

```text
minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors
约 1.96GB
```

总量：

```text
约 43.5GB+
```

---

## 7. Hugging Face 下载

曾尝试：

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

暂时不使用 `hf + Xet`。

---

## 8. hf-mirror 下载

### Diffusion

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors \
  --relative-path models/diffusion_models \
  --filename minimax_h3_ref2va_pruned_int8_convrot.safetensors
```

### Text Encoder

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors \
  --relative-path models/text_encoders \
  --filename qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
```

### Video VAE

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors \
  --relative-path models/vae \
  --filename minimax_h3_video_vae_fp16.safetensors
```

### Audio VAE

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors \
  --relative-path models/vae \
  --filename minimax_h3_audio_vae_fp32.safetensors
```

### Turbo LoRA

```bash
comfy model download \
  --url https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/loras/minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors \
  --relative-path models/loras \
  --filename minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors
```

---

## 9. 下载失败检查

查看模型目录：

```bash
du -sh /root/comfy/ComfyUI/models/diffusion_models/*
```

删除失败的 `.part`：

```bash
rm -f /root/comfy/ComfyUI/models/diffusion_models/*.part
```

---

## 10. ComfyUI models → 数据盘

创建目录：

```bash
mkdir -p /root/autodl-tmp/ComfyUI/models
```

查看原目录：

```bash
ls -ld /root/comfy/ComfyUI/models
```

迁移：

```bash
cp -a /root/comfy/ComfyUI/models/. \
/root/autodl-tmp/ComfyUI/models/
```

检查：

```bash
du -sh /root/autodl-tmp/ComfyUI/models
```

备份：

```bash
mv /root/comfy/ComfyUI/models \
/root/comfy/ComfyUI/models.bak
```

建立软链接：

```bash
ln -s /root/autodl-tmp/ComfyUI/models \
/root/comfy/ComfyUI/models
```

检查：

```bash
ls -ld /root/comfy/ComfyUI/models
```

正确结果：

```text
/root/comfy/ComfyUI/models
-> /root/autodl-tmp/ComfyUI/models
```

---

## 11. 测试软链接

```bash
touch /root/comfy/ComfyUI/models/test.txt
```

```bash
ls -l /root/autodl-tmp/ComfyUI/models/test.txt
```

```bash
rm /root/comfy/ComfyUI/models/test.txt
```

---

## 12. 删除 models.bak

确认：

```bash
ls -ld /root/comfy/ComfyUI/models
```

检查备份：

```bash
du -sh /root/comfy/ComfyUI/models.bak
```

确认不需要后：

```bash
rm -rf /root/comfy/ComfyUI/models.bak
```

---

## 13. MiniMax H3 工作流

工作流：

```text
video_minimax_h3_multiframe_reference.json
```

下载：

```bash
cd /root/comfy/ComfyUI
```

```bash
wget -O video_minimax_h3_multiframe_reference.json \
https://raw.githubusercontent.com/Comfy-Org/workflow_templates/main/templates/video_minimax_h3_multiframe_reference.json
```

官方文档：

```text
https://docs.comfy.org/tutorials/video/minimax/minimax-h3-multiframe
```

---

## 14. GPU 实例检查

```bash
nvidia-smi
```

```bash
df -h
```

```bash
lsblk -f
```

```bash
ls -ld /root/comfy/ComfyUI/models
```

```bash
ls -lah /root/comfy/ComfyUI/models
```

---

## 15. GPU 实例启动 ComfyUI

```bash
comfy launch -- --listen 0.0.0.0 --port 6006
```

---

## 16. 重要结论

```text
系统盘：
/root/comfy/ComfyUI
约 30GB

本地数据盘：
/root/autodl-tmp
50GB

公共盘：
/autodl-pub
只读

文件存储：
/root/autodl-fs
当前不存在

sda2：
约 446GB
当前未挂载
```

```text
MiniMax H3 总模型约 43.5GB+
```

```text
不要把完整 H3 放系统盘
不要把个人模型放 /autodl-pub
50GB 数据盘放完整 H3 余量很小
```

---

## 17. 常用命令速查

```bash
df -h
```

```bash
df -hT
```

```bash
lsblk
```

```bash
lsblk -f
```

```bash
mount | grep -E 'autodl|nfs|sda'
```

```bash
du -sh /root/comfy/ComfyUI/models
```

```bash
du -sh /root/autodl-tmp/ComfyUI/models
```

```bash
ls -ld /root/comfy/ComfyUI/models
```

```bash
nvidia-smi
```

```
```
