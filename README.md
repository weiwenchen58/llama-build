# llama-build

Build llama.cpp with LLGuidance and CUDA for Windows. This branch adds per-tool
Lark constraints by reusing native parsers, format metadata and output mappers.

请求提供普通 JSON Schema 工具及每工具 `function.input_grammar`，不再提供模型格式 JSON。
引擎在同一次连续生成中完成工具选择与正文约束，输出继续使用原生 `tool_calls`。
新增可由现有自动解析器和编码机制描述的格式，只增加模板与用例，不新增 C++ 格式分支。

- [接口、实现及限制](patches/README.md)
- [无模型测试数据](formats/tests.json)、[数据扩展测试模板](formats/runtime-new-format.jinja)
- [构建流程](.github/workflows/build-llama.yml)
- [运行与验证说明](.github/workflows/README.md)

`codex/runtime-tool-formats` 分支的补丁、测试数据及工作流变更会触发 CPU 测试和 Windows
CUDA 构建，也可以手动启动。产物保留 14 天，不自动发布 Release。编译、无模型验证、
真实模型验收和 AIOS 集成分别记录，不能互相替代。
