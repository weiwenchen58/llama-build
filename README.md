# llama-build
Build unmodified official llama.cpp with LLGuidance and CUDA for Windows. Downloadable llama-server ZIP via manual GitHub Actions.

通过 GitHub Actions 手动构建官方 llama.cpp + LLGuidance + CUDA，生成 Windows 64 位 llama-server ZIP，不添加额外源码补丁。

- [打开构建页面](https://github.com/weiwenchen58/llama-build/actions/workflows/build-llama.yml)，点击 Run workflow → main → Run workflow。
- 成功后在运行摘要点击 Download the Windows ZIP，或在 Artifacts 下载。产物保留 14 天。
- [使用说明与验证范围](.github/workflows/README.md)
- [构建配置](.github/workflows/build-llama.yml)

构建在 GitHub 的 Windows 机器上执行，无需本机安装编译器或拥有 GPU。实际模型推理需要目标电脑具备兼容的 NVIDIA GPU 和驱动；真机验收单独进行。
