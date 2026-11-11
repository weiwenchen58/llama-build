# llama-build
Build llama.cpp with LLGuidance and CUDA for Windows. This branch adds runtime format descriptors and per-tool grammar constraints.

本分支为 llama.cpp + LLGuidance 增加运行时格式描述补丁，通过 GitHub Actions 构建 Windows 64 位 llama-server ZIP。格式配置同时驱动采样约束和输出解析；在描述协议可表达的范围内，后续添加格式只更新 JSON 数据和测试，不必修改引擎、重新编译或重启服务。

- [打开构建页面](https://github.com/weiwenchen58/llama-build/actions/workflows/build-llama.yml)，选择 `codex/runtime-tool-formats` 分支运行；补丁和格式数据的推送也会触发测试与构建。
- 成功后在运行摘要点击 Download the Windows ZIP，或在 Artifacts 下载。产物保留 14 天。
- [请求参数、格式协议及验证边界](patches/README.md)
- [运行时格式 JSON](formats/)，[数据驱动测试样例](formats/tests.json)
- [基础构建说明](.github/workflows/README.md)
- [构建配置](.github/workflows/build-llama.yml)

构建在 GitHub 的 Windows 机器上执行，无需本机安装编译器或拥有 GPU。实际模型推理需要目标电脑具备兼容的 NVIDIA GPU 和驱动；真机验收单独进行。
