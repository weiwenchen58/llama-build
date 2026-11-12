# Native parser tool grammars

The extension reuses llama.cpp's native Jinja rendering, selected handler,
PEG parser, format metadata and output mapper. The request carries ordinary
JSON Schema tools and per-tool `function.input_grammar`; it does not carry a
second description of the model format. `native_tool_format` is rejected.

Pinned sources:

- llama.cpp `149f5641b931aaf33697d70f9abc90c3e5300247`
- LLGuidance `d795912fedc7d393de740177ea9ea761e7905774`

Apply both patches. CMake applies the Rust patch to the actual Cargo source.
CPU tests run before the Windows CUDA package build. Compilation, no-model
validation, real-model inference and AIOS integration are separate results.
Windows first links the shared CPU `llama-common` target before compiling CUDA,
so missing DLL exports fail early. Referenced grammar/Unicode helpers are exported;
the LLGuidance target carries its Windows system-library dependencies.

## Request

A grammar tool has exactly one required string property `input` with
`additionalProperties: false`. Its `function.input_grammar` is an object with
`syntax: "lark"` and a nonempty `definition`, for example:

```json
{"syntax":"lark","definition":"start: expr\nexpr: /[0-9]+/ | \"(\" expr \"+\" expr \")\""}
```

Ordinary tools retain `parameters` JSON Schema. Responses and streaming deltas
use native `tool_calls`. Tool choice may be auto, required, none, or a named
function. There is one continuous generation, no second completion, output
repair or retry without constraints.

`/props.native_tool_input_grammar` reports version 3, syntax `lark`, and
`native_parser: true`. This is an engine capability, not a promise that every
upstream handler or arbitrary schema has a supported codec. Missing semantic
bindings, unsupported constraints, invalid grammar and lossy encodings fail.
Requests without an enabled grammar tool retain the upstream path.

## Native format as the source

The sampling graph is derived from native PEG nodes. Tool/argument semantics
identify the bindings. Context-specific graph copies keep shared string rules
from mixing grammars between tools. AC and explicit grammar overrides are
handled during composition; output retains the original parser and mapper.
Native prefix-control metadata retains thinking, content, fallback and call-entry
states. Prefill is consumed by the same DFA; native GBNF continuations are derived
after the exact call-entry marker. Nested triggers cannot skip their outer section.
Native literal nodes
supply the control tokens needed by the tokenizer.

Gemma's original parser registers generic value-codec metadata referring to its
existing string, object, array, key and delimiter nodes. There is no second
model-specific table of wire strings. Generic tagged arguments receive
transparent semantic metadata at their existing construction point.

Formats understood by the upstream automatic parser can be added as Jinja
files and test cases. The same executable reads these data at runtime; no new
C++ format registration is needed. `formats/` now contains test fixtures only,
including a synthetic new template, rather than request-format descriptors.
No model-format JSON is required in AIOS. Function parameter roots are objects;
empty parameters use the existing upstream normalization. Native tagged keys that
the original mapper would trim are rejected, rather than silently renamed. A new encoding primitive outside the
supported semantic/codec operations requires an explicit capability extension.

## Supported constraints and limits

- JSON arguments use the pinned LLGuidance JSON Schema compiler. Lark body
  recursion is retained and encoded as JSON string content when needed.
- Tagged objects use explicit properties and required fields. Unsupported
  object-level constraints or value codecs fail instead of being dropped.
- Native value codecs support strings, explicit-property objects, homogeneous
  arrays and JSON scalar values. Unsupported schema constructs are rejected.
- Raw/CDATA encodings must represent the entire submitted body language. Each
  retained encoding is checked for complete delimiter and overlap collisions,
  including recursion and ignored text. Reserved native prefixes are checked.
  An unsafe alternative is removed only if another can represent the whole
  language; otherwise the request fails.
- A language whose strings require different encodings, with no single encoding
  covering the whole language, may be conservatively refused. This is not a
  general inverse-codec implementation.
- Native formats cannot encode arbitrary strings losslessly. Full arbitrary-source
  apply_patch may conflict with raw/CDATA delimiters. Do not narrow its grammar
  or claim full support from an encodable sample.
- Tool-body special tokens, nested grammars, generation attributes and parametric
  rules are unsupported. Custom top-level grammar, response schema, pure-content
  override and continuation editing cannot be combined with this extension.
- Clients must wait for a successful final tool-call response. Partial,
  truncated, failed or cancelled output is not an executable call.

## Verification

The existing test runner uses real LLGuidance masks with byte and compound
synthetic tokens. It checks mixed ordinary/nested schemas, independent body
languages, repeated calls, decoded arguments, streaming prefixes, invalid output,
raw delimiter proof and native thinking/prefill. Existing PEG, chat-PEG,
auto-parser and JSON-schema suites also run without a model.

Freeze the test binary, add a new Jinja template and a case in `formats/tests.json`,
then rerun it to verify data-only extension. Keep the binary hash, template and
observed output. Real tokenizer/GPU, performance and AIOS acceptance must be
recorded separately.
