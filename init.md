# AutoDL + ComfyUI + Comfy CLI + MiniMax H3

> AutoDL 上部署 ComfyUI、配置 Comfy CLI、迁移模型到数据盘，以及使用 aria2 下载大模型。

---

## 1. 环境与目录

### 常用目录

```text
/root/comfy/ComfyUI
    ComfyUI 程序

/root/autodl-tmp
    数据盘，存放模型、数据集、输出等

/autodl-pub
    公共数据盘
```

检查环境：

```bash
nvidia-smi
df -hT
```

建议：

```text
程序 → /root/comfy/ComfyUI
模型 → /root/autodl-tmp/ComfyUI/models
```

不要把几十 GB 的模型放在系统盘。

---

## 2. Comfy CLI 安装

### 安装 / 更新

推荐使用 pip：

```bash
pip install -U comfy-cli
```

检查：

```bash
comfy --version
```

如果提示找不到：

```text
comfy: command not found
```

可以检查：

```bash
which python
which pip
which comfy
```

也可以：

```bash
python -m pip show comfy-cli
```

### 更新 Comfy CLI

```bash
pip install -U comfy-cli
```

再次确认：

```bash
comfy --version
```

---

## 3. Comfy CLI 初始化

### 查看帮助

```bash
comfy --help
```

查看版本：

```bash
comfy --version
```

### 查看当前 ComfyUI

```bash
comfy env
```

如果需要指定 ComfyUI 安装目录，可以进入：

```bash
cd /root/comfy/ComfyUI
```

然后使用：

```bash
comfy launch
```

### 查看 launch 参数

```bash
comfy launch --help
```

---

## 4. ComfyUI 启动

### GPU 启动

```bash
cd /root/comfy/ComfyUI

comfy launch -- \
  --listen 0.0.0.0 \
  --port 6006
```

### 指定 GPU

例如使用 GPU 0：

```bash
CUDA_VISIBLE_DEVICES=0 comfy launch -- \
  --listen 0.0.0.0 \
  --port 6006
```

### CPU 模式

```bash
comfy launch -- \
  --cpu \
  --listen 0.0.0.0 \
  --port 6006
```

### AutoDL 网络加速

需要时：

```bash
source /etc/network_turbo
```

检查：

```bash
env | grep -i proxy
```

---

## 5. Comfy CLI 模型管理

查看模型相关命令：

```bash
comfy model --help
```

查看下载帮助：

```bash
comfy model download --help
```

### Comfy CLI 下载模型

例如：

```bash
comfy model download \
  --url "https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors" \
  --relative-path models/diffusion_models \
  --filename minimax_h3_ref2va_pruned_int8_convrot.safetensors
```

### 后台下载

如果 CLI 支持：

```bash
comfy model download --help | grep background
```

出现：

```text
--background
```

即可：

```bash
comfy model download \
  --url "https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors" \
  --relative-path models/diffusion_models \
  --filename minimax_h3_ref2va_pruned_int8_convrot.safetensors \
  --background
```

命令会返回：

```text
Downloading in the background: <DOWNLOAD_ID>
Track it with: comfy model download-status <DOWNLOAD_ID>
```

查看：

```bash
comfy model download-status <DOWNLOAD_ID>
```

> `--background` 主要解决“终端关闭后下载是否继续”的问题。
>
> 对 10GB+ 的超大文件，推荐使用下面的 `aria2c` 做断点续传。

---

## 6. 模型目录迁移到数据盘

创建：

```bash
mkdir -p /root/autodl-tmp/ComfyUI/models
```

复制已有模型：

```bash
cp -a /root/comfy/ComfyUI/models/. \
  /root/autodl-tmp/ComfyUI/models/
```

确认：

```bash
du -sh /root/autodl-tmp/ComfyUI/models
```

备份原目录：

```bash
mv /root/comfy/ComfyUI/models \
  /root/comfy/ComfyUI/models.bak
```

建立软链接：

```bash
ln -s /root/autodl-tmp/ComfyUI/models \
  /root/comfy/ComfyUI/models
```

确认：

```bash
ls -ld /root/comfy/ComfyUI/models
```

应该显示：

```text
/root/comfy/ComfyUI/models
-> /root/autodl-tmp/ComfyUI/models
```

测试：

```bash
touch /root/comfy/ComfyUI/models/test.txt
ls /root/autodl-tmp/ComfyUI/models/test.txt
rm /root/comfy/ComfyUI/models/test.txt
```

确认正常后：

```bash
rm -rf /root/comfy/ComfyUI/models.bak
```

---

## 7. 推荐：aria2 下载超大模型

安装：

```bash
apt update
apt install -y aria2
```

检查：

```bash
aria2c --version
```

进入模型目录：

```bash
cd /root/autodl-tmp/ComfyUI/models/diffusion_models
```

### 多线程 + 断点续传

```bash
aria2c \
  -c \
  -x 8 \
  -s 8 \
  --file-allocation=none \
  -o minimax_h3_ref2va_pruned_int8_convrot.safetensors \
  "https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors"
```

参数：

```text
-c
断点续传

-x 8
最多 8 个连接

-s 8
8 个分片

--file-allocation=none
不提前分配完整文件
```

下载中断：

```text
Ctrl+C
```

重新执行同一个命令：

```bash
aria2c -c ...
```

即可继续。

**不要删除 `.aria2` / `.part` 文件。**

---

## 8. aria2 后台下载

确认前台下载速度正常后，可以后台运行：

```bash
aria2c \
  -c \
  -x 8 \
  -s 8 \
  --file-allocation=none \
  --daemon=true \
  -o minimax_h3_ref2va_pruned_int8_convrot.safetensors \
  "https://hf-mirror.com/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors"
```

查看进程：

```bash
ps aux | grep aria2c
```

查看文件：

```bash
ls -lh /root/autodl-tmp/ComfyUI/models/diffusion_models/
```

实时查看：

```bash
watch -n 2 \
'ls -lh /root/autodl-tmp/ComfyUI/models/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors'
```

关闭网页终端 / SSH 后，后台 aria2 下载仍可继续。

---

## 9. MiniMax H3 模型与下载问题

主要模型：

```text
diffusion_models/
└── minimax_h3_ref2va_pruned_int8_convrot.safetensors
    ~20GB

text_encoders/
└── qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
    ~15.7GB

vae/
├── minimax_h3_video_vae_fp16.safetensors
│   ~5.21GB
└── minimax_h3_audio_vae_fp32.safetensors
    ~605MB

loras/
└── minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors
    ~1.96GB
```

总计约：

```text
43.5GB+
```

因此 50GB 数据盘需要注意空间。

检查：

```bash
df -h /root/autodl-tmp
du -sh /root/autodl-tmp/ComfyUI/models
```

### HuggingFace 镜像

优先使用：

```text
https://hf-mirror.com
```

如果 `hf download` 出现：

```text
401 Unauthorized
CAS Client Error
cas-server.xethub.hf.co
```

可以改用镜像 URL + `aria2c`。

### 完全重新下载

确定不要之前的文件时：

```bash
rm -f \
/root/autodl-tmp/ComfyUI/models/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors*
```

然后重新运行 aria2。

---

## 10. 日常维护速查

### GPU

```bash
nvidia-smi
```

### 磁盘

```bash
df -h
```

### 模型占用

```bash
du -sh /root/autodl-tmp/ComfyUI/models
```

### 模型目录

```bash
ls -lah /root/autodl-tmp/ComfyUI/models
```

### ComfyUI 软链接

```bash
ls -ld /root/comfy/ComfyUI/models
```

### Comfy CLI

```bash
comfy --version
comfy --help
comfy model --help
```

### Comfy 后台下载

```bash
comfy model download-status <DOWNLOAD_ID>
```

### aria2

```bash
ps aux | grep aria2c
```

### ComfyUI 启动

```bash
cd /root/comfy/ComfyUI

comfy launch -- \
  --listen 0.0.0.0 \
  --port 6006
```

### 不用 GPU

直接关闭 AutoDL 实例。

数据盘中的：

```text
/root/autodl-tmp
```

数据会保留。

下次开机后：

```bash
cd /root/comfy/ComfyUI
```

检查模型即可。

---

# 推荐结构

```text
/root/
├── comfy/
│   └── ComfyUI/
│       ├── models -> /root/autodl-tmp/ComfyUI/models
│       ├── custom_nodes/
│       └── ...
│
└── autodl-tmp/
    └── ComfyUI/
        └── models/
            ├── diffusion_models/
            ├── text_encoders/
            ├── vae/
            ├── loras/
            └── ...
```

## 核心原则

```text
ComfyUI 程序
→ /root/comfy/ComfyUI

模型 / 大文件
→ /root/autodl-tmp/ComfyUI/models

Comfy CLI
→ 安装、启动、简单模型管理

10GB+ 大模型
→ aria2c

需要断点续传
→ aria2c -c

需要脱离终端运行
→ aria2c --daemon=true

不用 GPU
→ 关闭 AutoDL 实例
```
