# Windows llama-server 构建

手动编译官方 llama.cpp + LLGuidance + CUDA，不应用额外源码补丁。

## 运行与下载

1. 打开仓库顶部 Actions。
2. 选择 Build llama-server (Windows CUDA + LLGuidance)。
3. 点击 Run workflow，选择 main，再点击 Run workflow。
4. 成功后在摘要点击 Download the Windows ZIP，或在 Artifacts 下载。
5. llama-build-logs 包含日志和 ZIP 校验值。产物保留 14 天。

只在手动运行时构建，不自动发布 Release。构建机无需 NVIDIA GPU；运行模型的电脑需要兼容的显卡及驱动。完整解压，保留 EXE 和 DLL 在同一目录。

## 构建输入和验证边界

- llama.cpp：149f5641b931aaf33697d70f9abc90c3e5300247
- LLGuidance：d795912fedc7d393de740177ea9ea761e7905774，由上述官方源码固定
- CUDA 13.0.2；MSVC 14.44；Rust 1.91.1
- CMake 3.31.10；Ninja 1.13.0；Windows 2022 runner
- CUDA 13 不覆盖早于 Turing 的显卡。

工作流检查编译、依赖和 --version 基础启动。实际构建状态以 Actions 为准；真实模型推理、GPU 卸载和 LLGuidance 约束效果仍须在目标电脑验收。采用官方错误处理行为，不承诺所有内部 grammar 故障立即终止生成。

[构建配置](build-llama.yml)
