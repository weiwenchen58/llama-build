# Runtime native tool formats

This branch replaces per-model C++ body bindings with the request parameter
`native_tool_format`. One versioned JSON descriptor builds both the output parser
and a joint LLGuidance sampling grammar. Native Jinja prompt rendering remains.
Tool Lark grammars and ordinary JSON Schema leaves are composed before sampling.
The mask is applied before token selection; there is no second completion or
output-repair retry. Requests without the extension retain the upstream path.

Apply both patches to these pinned sources:

- llama.cpp `149f5641b931aaf33697d70f9abc90c3e5300247` (b12098)
- LLGuidance `d795912fedc7d393de740177ea9ea761e7905774`

CMake applies the Rust patch to the actual Cargo source. The Actions workflow
runs CPU tests before building the Windows CUDA package. Packages include format
data, patches, source/toolchain identifiers and checksums. No model is downloaded
and no GitHub Release is published. See the Actions run for actual results.

## Request contract

Read a descriptor from `formats/` and send the **JSON object** in the chat
completion request. The engine receives data, not a filesystem path or a name in
a compiled model registry.

```python
request["native_tool_format"] = json.loads(
    Path("formats/qwen-coder.json").read_text(encoding="utf-8")
)
request["tools"][0]["function"]["input_grammar"] = {
    "syntax": "lark",
    "definition": 'start: expr\nexpr: /[0-9]+/ | "(" expr "+" expr ")"',
}
```

A grammar tool declares exactly one required string property `input` and
`additionalProperties: false`. Ordinary tools retain JSON Schema. Responses use
normal `tool_calls` and streaming argument deltas. The profiles support `auto`,
`required`, `none`, named function selection and `parallel_tool_calls`. Named
selection filters tools before rendering and grammar construction.

`/props.native_tool_input_grammar` advertises extension version 2, parameter name,
descriptor version 1 and body encodings. The old handler-specific version 1 is no
longer advertised. Grammar tools without a descriptor fail explicitly.

## Descriptor version 1

A string is literal wire text. An array is a sequence. Objects have an `op` key.
`root` describes generated output after the native generation prefill, which the
engine automatically includes in both grammar and parser. `definitions` holds
reusable fragments. `ref` is bounded expansion, not recursive PEG execution;
recursive body languages belong in Lark.

| Operation | Fields and behavior |
| --- | --- |
| `ref` | `name`: expand a definition |
| `choice` | `items`: ordered parser alternatives / grammar alternatives |
| `repeat` | `item`, `min` (0), `max` (-1 means unlimited) |
| `if` | `condition`, `then`, `else`: choose at request preparation |
| `space` | Optional space, tab and newline |
| `until` | `stops`: consume until a delimiter, leave it for the next literal |
| `content`, `reasoning` | `item`: capture assistant text or reasoning |
| `tools` | `item`: expand one alternative per allowed tool |
| `name` | Match and capture the current tool name |
| `args` | JSON arguments, bound to JSON Schema or encoded Lark input |
| `input` | `stop`: raw decoded Lark input ending before this exact delimiter |
| `parameters` | `item`, optional wire `separator`: expand closed-object fields |
| `parameter_name` | Match a property name and emit its JSON key |
| `parameter_value` | `encoding`: `json` or `raw`; raw needs `stop`; optional `scalar_only` |
| `emit` | `value`: emit a fixed JSON fragment without consuming wire bytes |

Conditions are `tools`, `required`, `parallel`, `thinking`, `thinking_prefilled`,
`input_grammar` (inside tools), and `string` (inside parameters). Metadata includes
`preserved_tokens`, `additional_stops`, `thinking_start`, `thinking_end`.
Names use ASCII letters/digits/underscore/dot/dash. Descriptor size is limited to
1 MiB, expansion to 64 levels and 10,000 nodes.

`parameters` supports closed objects with properties and required fields, in
schema declaration order. Optional fields remain optional. Other object-level
constraints are rejected in this representation; use `args` for general JSON
Schema. Raw string schemas reuse LLGuidance pattern/enum/length compilation;
unsupported raw types are rejected. JSON Schema and Lark support is bounded by
the pinned LLGuidance implementation and these additional restrictions, not all
JSON Schema drafts or all Python Lark features.

Descriptors are protocol definitions and require tests. Ordered PEG choices must
be unambiguous. Free text delimiters must exclude the control markers used by
following alternatives. Incorrect descriptors are not repaired automatically or
inferred from Jinja templates.

## Encodings and errors

`args` encodes the recursive body as JSON string content. Terminal DFA transitions
are escaped while recursive productions are retained; this is not a regex-only
approximation. Quotes, backslashes, control characters and Unicode are covered.

`input` embeds raw UTF-8. A CFG × delimiter-DFA reachability proof checks complete
derivations, including recursion, ignores and delimiters split across terminals.
It rejects a body language that can contain or overlap the closing delimiter.
Ordinary `<div>` content is allowed; the old blanket `<` prohibition is removed.
The proof has a compilation budget and accepts delimiters of 1–63 bytes. It does
not silently narrow a submitted grammar. Compiler, matcher and mask errors fail
the request instead of disabling the constraint.

Native protocols must represent the body losslessly. MiniCPM CDATA supports
`</param>` inside the body, but cannot contain `]]>`. A different data-defined
encoding may solve a particular case; an ambiguous native protocol cannot encode
every arbitrary string. Special tokens, nested grammars, generation attributes
and parametric rules are not supported inside tool body grammars.

Continuation/prefill editing, separate response schemas, top-level custom grammar
and pure-content override cannot be combined with this extension. Incomplete or
cancelled streams are not repaired into executable calls; clients must wait for a
successfully completed response.

## Data updates and tests

Profiles include Qwen JSON, Qwen Coder parameters, MiniCPM CDATA, Gemma 4 scalar
parameters and an unfamiliar example format. The Gemma profile explicitly rejects
nested object/array codecs; it does not silently use JSON quote rules for them.
The unfamiliar format is an interface test, not a claim about model training.

Add a descriptor and entries in `formats/tests.json`, then pass the descriptor on
the next request. **Formats expressible by version 1 need no patch edit, engine
rebuild or server restart.** A genuinely new encoding/operation outside this
contract needs a versioned engine capability. Adding ordinary format data does
not require a C++ model registration.

The no-model test reads external JSON at runtime and uses actual LLGuidance masks
with byte and compound synthetic tokens. It checks valid/invalid output, exact
argument parsing, streaming-prefix consistency, call counts and delimiter proofs,
including the `A` versus required `AB` premature-delimiter counterexample. Existing
PEG, chat-PEG and JSON-schema tests also run. Compilation and these tests do not
establish real-model quality, GPU inference, performance or AIOS integration.
