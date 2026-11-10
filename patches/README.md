# Native tool input grammar

This optional patch set adds `function.input_grammar = {"syntax":"lark","definition":"..."}` to native chat completion tools. Ordinary tools keep JSON Schema. A grammar tool must declare exactly one required string property `input`, with `additionalProperties: false`. The response remains ordinary `tool_calls`; there is no second completion or custom outer protocol.

Pinned inputs:

- llama.cpp `149f5641b931aaf33697d70f9abc90c3e5300247`
- LLGuidance `d795912fedc7d393de740177ea9ea761e7905774`

The C++ patch binds a body to the existing native PEG parser, reuses the native GBNF/Schema generator and GBNF parser, and emits LLGuidance rules from parsed grammar elements. Ordinary requests retain the original path. The Rust patch maps text terminals and ignored text to JSON string encoding while preserving recursive productions. Encoding mode is passed outside the user Lark source. Existing response parsing and escaping remain unchanged.

Initial supported paths are the generic JSON tool handler and MiniCPM5's raw XML parameter handler. Other handlers reject this extension rather than ignoring it. Raw XML v1 requires that every reachable body terminal and ignore expression exclude `<`; otherwise compilation fails without narrowing the submitted grammar. This intentionally rejects grammars that could close the XML envelope. JSON tools do not have that restriction. Special tokens, nested grammars, generation attributes and parametric rules are not supported in a tool body. This is LLGuidance's Lark dialect, not every Python Lark feature.

Auto mode uses the native word trigger boundaries; ordinary text remains available until a tool marker. Required and filtered/selected tools reuse the original native branch construction. Multi-call behavior follows `parallel_tool_calls`. Native streaming stays unchanged: clients must not execute partial deltas or calls from length/cancellation/error outcomes.

`/props.native_tool_input_grammar` advertises the extension in LLGuidance builds; individual requests still validate their actual handler. Matcher failures fail the request instead of dropping the constraint.

## Build and verification

Use the existing workflow with `native_tool_grammar: true`. The default manual build remains unpatched. The development branch also runs this mode on patch/workflow pushes. CPU tests run first; only a successful test job permits the Windows CUDA package job. No model or external tokenizer is downloaded or loaded. The test uses a synthetic byte vocabulary and actual LLGuidance token masks, together with the native templates/parser.

Artifacts record upstream commits, patch hashes, workflow commit, toolchain and the Actions run. Tests, compilation, package smoke checks and actual model inference are separate acceptance levels. Real-model/GPU inference, quality and performance remain untested. See the exact Actions run for current test/build status; source preparation alone is not a passing result.

The LLGuidance patch is applied by CMake's ExternalProject to the source actually used by Cargo. Both patches must be used together.
