# Windows llama-server 构建

构建固定版本 llama.cpp + LLGuidance + CUDA。`codex/runtime-tool-formats` 默认应用原生 parser 工具 grammar 补丁；手动关闭补丁选项时才构建未修改的上游源码。

## 运行与下载

1. 打开仓库顶部 Actions。
2. 选择 Build llama-server (Windows CUDA + LLGuidance)。
3. 点击 Run workflow，选择 `codex/runtime-tool-formats`，确认补丁选项，再启动。
4. 成功后在摘要点击 Download the Windows ZIP，或在 Artifacts 下载。
5. llama-build-logs 包含日志和 ZIP 校验值。产物保留 14 天。

该分支的补丁、测试数据及工作流推送也会触发构建，不自动发布 Release。构建机无需 NVIDIA GPU；运行模型的电脑需要兼容的显卡及驱动。完整解压，保留 EXE 和 DLL 在同一目录。

## 构建输入和验证边界

- llama.cpp：149f5641b931aaf33697d70f9abc90c3e5300247
- LLGuidance：d795912fedc7d393de740177ea9ea761e7905774，由上述官方源码固定
- CUDA 13.0.2；MSVC 14.44；Rust 1.91.1
- CMake 3.31.10；Ninja 1.13.0；Windows 2022 runner
- CUDA 13 不覆盖早于 Turing 的显卡。

补丁模式先执行五组 CPU 无模型测试，再检查 Windows 编译、依赖和 --version 启动；build-info 与校验文件记录源码及补丁身份。实际状态以 Actions 为准；真实模型、GPU 和 AIOS 验收须在目标电脑执行。补丁模式的 grammar 编译/mask 错误明确失败，不移除约束重试；未修改模式保留上游原始错误处理行为。

[构建配置](build-llama.yml)
