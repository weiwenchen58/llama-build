# Windows llama-server 构建

编译固定版本 llama.cpp + LLGuidance + CUDA。默认手动模式不应用额外源码补丁；`native_tool_grammar` 输入可启用原生工具正文 grammar 补丁。

## 运行与下载

1. 打开仓库顶部 Actions。
2. 选择 Build llama-server (Windows CUDA + LLGuidance)。
3. 点击 Run workflow，选择 main，再点击 Run workflow。
4. 成功后在摘要点击 Download the Windows ZIP，或在 Artifacts 下载。
5. llama-build-logs 包含日志和 ZIP 校验值。产物保留 14 天。

默认模式手动运行；`codex/native-tool-grammar` 分支的补丁和工作流推送自动执行补丁模式。不会自动发布 Release。构建机无需 NVIDIA GPU；运行模型的电脑需要兼容的显卡及驱动。完整解压，保留 EXE 和 DLL 在同一目录。

需要重新运行补丁分支时，可使用 GitHub CLI：

```sh
gh workflow run build-llama.yml --repo weiwenchen58/llama-build --ref codex/native-tool-grammar -f native_tool_grammar=true
```

补丁模式先编译 Linux CPU 服务器与针对性测试，测试通过才进入 Windows CUDA 打包。ZIP 名称包含 `tool-grammar`，`build-info.json` 记录补丁 SHA256 与验证范围。详细契约与限制见[补丁说明](../../patches/README.md)。

## 构建输入和验证边界

- llama.cpp：149f5641b931aaf33697d70f9abc90c3e5300247
- LLGuidance：d795912fedc7d393de740177ea9ea761e7905774，由上述官方源码固定
- CUDA 13.0.2；MSVC 14.44；Rust 1.91.1
- CMake 3.31.10；Ninja 1.13.0；Windows 2022 runner
- CUDA 13 不覆盖早于 Turing 的显卡。

工作流检查编译、依赖和 --version 基础启动。补丁模式额外执行真实 grammar 编译、token mask 和原生 parser 测试，使用合成词表，不下载或加载模型。实际构建状态以 Actions 为准；真实模型推理、GPU 卸载、质量和性能仍须在目标电脑验收。无补丁模式保留上游错误处理行为；补丁模式将 matcher 错误传播为请求失败。

[构建配置](build-llama.yml)
