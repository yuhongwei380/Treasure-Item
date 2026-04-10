TurboQuant+ (llama.cpp Fork) 使用指南TurboQuant+ 是一个针对 llama.cpp 的实验性增强分支，通过 TurboQuant (ICLR 2026) 技术实现 KV Cache 的高倍率压缩（3.8x - 6.4x），显著降低长文本推理时的显存压力。1. 编译安装首先克隆仓库并根据你的硬件环境进行编译：
Bash
# 1. 克隆仓库
git clone https://github.com/TheTom/turboquant_plus.git
cd turboquant_plus

# 2. 创建编译目录
mkdir build && cd build

# 3. 根据硬件执行编译命令
# Apple Silicon (Mac)
cmake .. -DGGML_METAL=ON
# NVIDIA GPU (CUDA)
cmake .. -DGGML_CUDA=ON

# 4. 开始编译
cmake --build . --config Release
## 2. 核心运行参数在使用 llama-server 或 llama-cli 时，需通过以下参数激活 TurboQuant 功能：参数说明可选值-ctkKey 缓存量化类型q8_0, turbo4, turbo3, turbo2-ctvValue 缓存量化类型q8_0, turbo4, turbo3, turbo2-fa 1Flash Attention必须开启 (1)

## 3. 推荐配置建议为了在“内存节省”与“模型智商”之间取得平衡，建议参考以下配置：

### A. 最佳平衡模式 (推荐)保持 Key 的高精度，对 Value 进行 4-bit 压缩。这是最稳妥的方案。
Bash./bin/llama-server -m model.gguf -ctk q8_0 -ctv turbo4 -fa 1

### B. 极致压缩模式 (长文本/低内存)适用于显存极度紧缺，或需要运行 128K 以上超长上下文的情况。
Bash./bin/llama-server -m model.gguf -ctk q8_0 -ctv turbo2 -fa 1

### C. 全量压缩 (Symmetric)仅推荐在模型参数量较大（如 70B 或 104B 以上）时使用，大模型对压缩的容忍度更高。
Bash./bin/llama-server -m model.gguf -ctk turbo4 -ctv turbo4 -fa 1

## 4. 进阶优化 (环境变量)该项目支持一些隐藏的“黑科技”开关，可以在启动命令前添加环境变量：Boundary V (边界层保护): 保护模型的第一层和最后两层不被过度压缩，大幅提升质量。
Bash# 开启方式 (设置等级为 7)
TURBO_LAYER_ADAPTIVE=7 ./bin/llama-server ...
Sparse V (稀疏解码): 跳过低权重位置的解码，提升约 20% 的推理速度。Bash# 默认通常已优化，但在 MoE 模型上效果显著

# 6. 常见问题排查模型胡言乱语？
检查是否开启了 -fa 1。尝试将 -ctk 改回 q8_0（Key 对质量的影响远大于 Value）。
Mac 显存报错？
如果运行 70B/104B 模型，可能需要调整系统 wired 内存限制：sudo sysctl iogpu.wired_limit_mb=117964 (根据你的 RAM 调整)。
CUDA 兼容性：目前该项目在 Apple Silicon 上最为稳定，CUDA 环境下建议优先测试 turbo4。
