# llama-build
Build pinned llama.cpp with LLGuidance and CUDA for Windows. Optional native tool grammar patches; downloadable llama-server ZIP via GitHub Actions.

通过 GitHub Actions 构建固定版本 llama.cpp + LLGuidance + CUDA，生成 Windows 64 位 llama-server ZIP。默认手动构建不添加源码补丁；可选启用每工具正文 grammar 补丁。

- [打开构建页面](https://github.com/weiwenchen58/llama-build/actions/workflows/build-llama.yml)，点击 Run workflow → main → Run workflow。
- 成功后在运行摘要点击 Download the Windows ZIP，或在 Artifacts 下载。产物保留 14 天。
- [使用说明与验证范围](.github/workflows/README.md)
- [构建配置](.github/workflows/build-llama.yml)
- [原生工具 grammar 契约、支持范围和测试方式](patches/README.md)

`codex/native-tool-grammar` 分支的补丁或工作流推送会自动启用补丁模式：先编译服务器并执行不加载模型的 CPU 测试，通过后再编译 Windows CUDA 产物。具体通过状态和产物以对应 Actions 运行记录为准，不以分支存在代表已完成验证。

构建在 GitHub 的 Windows 机器上执行，无需本机安装编译器或拥有 GPU。实际模型推理需要目标电脑具备兼容的 NVIDIA GPU 和驱动；真机验收单独进行。
