# Jetson 上 PyTorch 安装避坑指南（ARM + CUDA）

&gt; 一句话总结：Jetson 上只认 NVIDIA 自家 PyTorch，其他都是“假 CUDA”。

---

## 问题现象
在 NVIDIA Jetson 设备（ARM64 架构）上运行基于 `funasr` 的语音识别脚本时，出现  
`OSError: libtorch_cuda.so: cannot open shared object file: No such file or directory`  
即使 `nvcc --version` 显示已装 CUDA 12.6，PyTorch 仍无法加载 GPU 动态库。

---

## 根本原因
1. Jetson = ARM64 + NVIDIA GPU，**必须**用 NVIDIA 针对 JetPack 预编译的 PyTorch。  
2. 官网/清华源等通用 `cu121` 包：  
   - 仅 aarch64 架构，**不含 Jetson 专用 CUDA 扩展**（`libtorch_cuda.so` 等）  
   - 实为 CPU-only 或 ABI 不兼容版本  
3. 混装不同来源的 `torch` / `torchaudio` 会导致依赖断裂。

---

## 正确安装方式

### ✅ 方法一：Jetson 官方 PyPI 源（在线，推荐）
```bash
# JetPack 6 + CUDA 12.6 专用源
pip install torch==2.8.0 torchaudio==2.8.0 torchvision==0.23.0 \
            --index-url https://pypi.jetson-ai-lab.io/jp6/cu126
```

### ✅ 方法二：离线安装 NVIDIA 官方 wheel
从 [NVIDIA 嵌入式下载页](https://developer.nvidia.com/embedded/downloads) 获取对应 `.whl`：  
```bash
pip install \
    torch-2.5.0a0+872d972e41.nv24.08.17622132-cp310-cp310-linux_aarch64.whl \
    torchaudio-2.5.1a0-cp310-cp310-linux_aarch64.whl \
    -i https://pypi.tuna.tsinghua.edu.cn/simple
```
&gt; ⚠️ 所有组件必须来自**同一 JetPack 版本**，禁止混用！

---

## 常用检查命令
| 用途                        | 命令                                                         |
| --------------------------- | ------------------------------------------------------------ |
| CUDA 编译器版本             | `nvcc --version`                                             |
| GPU 驱动/状态               | `nvidia-smi`                                                 |
| 实时 CPU/GPU/内存/温度/功耗 | `jtop`（需 `pip install jetson-stats`）                      |
| PyTorch 是否识别 GPU        | `python -c "import torch; print(torch.cuda.is_available())"` |
| 系统架构                    | `uname -m`（应输出 `aarch64`）                               |

---

## 验证是否成功
```python
import torch, torchaudio

print("PyTorch version:", torch.__version__)
print("CUDA available :", torch.cuda.is_available())
print("torchaudio     :", torchaudio.__version__, "loaded OK")
```
✅ **成功标志**  
- `torch.__version__` 含 `nv` 或明确版本标识  
- `torch.cuda.is_available()` 返回 `True`  
- 无任何 `OSError` 报错

---

## 关键注意事项
❌ **不要使用 PyTorch 官方 cu121 源**  
```bash
# 错误示例（会导致 libtorch_cuda.so 缺失）
pip install torch --index-url https://download.pytorch.org/whl/cu121
```
❌ **不要混装** 不同来源的 `torch` / `torchaudio` / `torchvision`  
✅ **所有包必须匹配同一 JetPack 版本**（如 JP6 / cu126）