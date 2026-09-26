# KATALI Hybrid

KATALI Hybrid is an experimental small-model integration runtime. It combines two compact GGUF language models into one KATALI Hybrid (`.khyb`) package and uses both models during one request:

- Qwen3 0.6B Q4_K_M creates a compact draft.
- Llama 3.2 1B Instruct Q4_K_M reviews and produces the final answer.

The purpose is to test whether two small models can provide better practical answers while remaining lightweight, local, and easy to benchmark. This project is focused on integration, speed, memory use, and answer quality. It is not a model router: both models participate in the same request.

## What is `.khyb`?

`.khyb` is a KATALI container format. It stores the original GGUF model payloads together in one file and allows the runtime to map them directly without extracting or re-quantizing them.

The embedded models remain Q4_K_M GGUF models. The `.khyb` file is not a standalone GGUF file and must be opened with `katali-hybrid.exe`.

## Included files

- `katali-hybrid.exe` - Hybrid inference runtime.
- `katali-hybrid-api.exe` - Loopback HTTP API wrapper.
- `katali-hybrid-gui.exe` - Compact Windows chat interface build.
- `pack-khyb.exe` - Creates a `.khyb` package from two GGUF files.
- `katali_cuda.dll` - KATALI CUDA runtime dependency.
- `cudart64_13.dll` - CUDA runtime dependency.

No C source code, headers, build files, or model weights are included in this binary release.

## Download the combined model

The complete Qwen3 + Llama model package is hosted separately on Hugging Face:

[Download KATALI Hybrid Qwen3 0.6B + Llama 3.2 1B](https://huggingface.co/katalidevai/katali-hybrid-qwen06b-llama1b-q4)

Direct downloads:

- [Download the combined `.khyb` model](https://huggingface.co/katalidevai/katali-hybrid-qwen06b-llama1b-q4/resolve/main/katali-hybrid-qwen06b-llama1b-q4.khyb)
- [Download the self-contained Windows GUI](https://huggingface.co/katalidevai/katali-hybrid-qwen06b-llama1b-q4/resolve/main/katali-hybrid-gui.exe)

Users do not need to download the two original GGUF files separately.

## Larger experimental package

For improved factual and reasoning quality, a larger experimental pair is also available:

```text
Qwen3 1.7B Q4_K_M + Llama 3.2 3B Instruct Q4_K_M
```

- [Download the Qwen3 1.7B + Llama 3B `.khyb` package](https://huggingface.co/katalidevai/katali-hybrid-qwen17b-llama3b-q4/resolve/main/katali-hybrid-qwen17b-llama3b-q4.khyb)
- [Open the larger package repository](https://huggingface.co/katalidevai/katali-hybrid-qwen17b-llama3b-q4)

This package is approximately 3.3 GB and is currently about 1.9x slower in cold-start testing than the original pair. It passed the initial factual, coding, reasoning, and instruction tests more reliably, but arithmetic remains an open limitation in this experimental runtime.

## Requirements

- Windows 10 or newer.
- The two compatible GGUF models, or an existing `.khyb` package.
- For CUDA execution, a compatible NVIDIA driver and GPU. CPU execution may be available depending on the build.

## Quick start

Run an existing hybrid package:

```powershell
.\katali-hybrid.exe `
  C:\models\katali-hybrid-qwen06b-llama1b-q4.khyb `
  "What is the capital of France? Answer with only the city name." `
  --max 32
```

Expected output:

```text
Paris
```

The runtime prints token diagnostics such as draft and final token counts to the console.

## Create a hybrid package

Use the packer with a Qwen GGUF file followed by a Llama GGUF file:

```powershell
.\pack-khyb.exe `
  C:\models\qwen3-0.6b-q4_k_m.gguf `
  C:\models\Llama-3.2-1B-Instruct-Q4_K_M.gguf `
  C:\models\katali-hybrid-qwen06b-llama1b-q4.khyb
```

The input GGUF files are not modified. Keep them if you also want to run the models individually with other GGUF-compatible tools.

## Interactive mode

Start the persistent stdin chat server:

```powershell
.\katali-hybrid.exe C:\models\katali-hybrid-qwen06b-llama1b-q4.khyb --server --max 96
```

Enter one prompt per line. The process stays loaded between requests, which avoids paying the model startup cost for every prompt.

## GUI mode

Place these files in one folder:

```text
katali-hybrid-gui.exe
katali-hybrid.exe
katali-hybrid-qwen06b-llama1b-q4.khyb
katali_cuda.dll
cudart64_13.dll
```

Start `katali-hybrid-gui.exe`, select the `.khyb` file, and send a message. The compact GitHub build requires the .NET 9 Desktop Runtime. A full self-contained GUI build is available from the Hugging Face repository.

## Current status

## Three-coder experiment

The current experimental coding package adds a three-stage sequential pipeline:

```text
Qwen2.5-Coder 1.5B Q4_K_M -> CodeGemma 2B Q4_K_M -> Granite 3B Code Q4_K_M
```

Qwen drafts code, CodeGemma reviews it, and Granite performs the final review.
This is an integration pipeline, not a router or a mathematical weight merge.
The tested package is hosted on Hugging Face:

[Download the KATALI Code Hybrid package](https://huggingface.co/katalidevai/katali-code-hybrid-qwen15b-codegemma2b-granite3b-q4)

Run it with the standalone runtime:

```powershell
.\katali-hybrid.exe `
  C:\models\katali-code-hybrid-qwen15b-codegemma2b-granite3b-q4.khyb `
  "Write a safe C function that checks integer overflow. Return only code." `
  --max 64
```

The GUI can select the `.khyb` file from the normal models directory. Use
short controlled prompts for the first tests because the final Granite model
has a 2K context window.

The current prototype demonstrates:

- One-file hybrid packaging.
- Direct memory mapping of both embedded GGUF payloads.
- Qwen draft plus Llama final generation.
- Persistent interactive server mode.
- Compatibility with the KATALI Lab GUI through the standalone executable.
- Repeatable tests for factual, instruction, coding, reasoning, and knowledge prompts.

The current version is experimental. Small models can still make arithmetic and multi-step reasoning mistakes, and the hybrid process does not yet preserve full conversational history internally between separate stdin requests.

## Future direction

Planned research includes:

1. Measure speed, memory use, and answer quality against each model running alone.
2. Improve the draft-to-final handoff so the final model receives useful structured context with less overhead.
3. Add configurable decoding and prompt policies.
4. Add an HTTP or local API for applications and GUI clients.
5. Add longer-lived conversation state and session management.
6. Explore better small-model pairings and optional layer or head optimizations.
7. Add automated benchmark reports for latency, tokens per second, memory, and accuracy.
8. Evaluate whether specialized small models can complement one another without turning the system into a router.

The long-term goal is a practical, measurable, and lightweight hybrid model runtime: one local package, two complementary small models, and a clear speed-versus-quality benchmark.

---

**Developed by: Joan Apita**

## Complete HTTP API tutorial

`katali-hybrid-api.exe` is a small local HTTP server for applications that
want to call the KATALI Hybrid runtime. It loads one `.khyb` package once and
keeps all models resident for subsequent requests.

### 1. Install the files

Download the binary release from this GitHub repository and place these files
in one folder:

```text
katali-hybrid-api.exe
katali-hybrid.exe
katali_cuda.dll          (only if included by your build)
cudart64_13.dll          (only if included by your build)
```

Download the combined coding model from Hugging Face:

[KATALI Code Hybrid model](https://huggingface.co/katalidevai/katali-code-hybrid-qwen15b-codegemma2b-granite3b-q4)

Save the `.khyb` file, for example:

```text
C:\models\katali-code-hybrid-qwen15b-codegemma2b-granite3b-q4.khyb
```

The original GGUF files are not needed when using the combined package.

### 2. Start the API

Open PowerShell in the folder containing the executables:

```powershell
.\katali-hybrid-api.exe `
  C:\models\katali-code-hybrid-qwen15b-codegemma2b-granite3b-q4.khyb `
  --port 8080 `
  --max 96
```

Options:

```text
--port N     HTTP port; default is 8080
--max N      Default maximum generated tokens; default is 96
```

Keep this PowerShell window open while using the API. The server listens only
on `127.0.0.1`, so it is not exposed to other computers by default.

### 3. Check that the API is running

In a second PowerShell window:

```powershell
Invoke-RestMethod http://127.0.0.1:8080/health
```

Expected response:

```json
{"status":"ok","runtime":"katali-hybrid"}
```

### 4. Generate with the simple endpoint

`POST /generate` accepts either `prompt` or `content`:

```powershell
$body = @{
  prompt = "Write a Rust Hello World program. Return only code."
  max_tokens = 64
} | ConvertTo-Json

Invoke-RestMethod `
  http://127.0.0.1:8080/generate `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

### 5. Use the OpenAI-compatible endpoint

`POST /v1/chat/completions` accepts the familiar `messages` format:

```powershell
$body = @{
  model = "katali-code-hybrid"
  messages = @(
    @{ role = "user"; content = "Write a C function that checks integer overflow. Return only code." }
  )
  max_tokens = 96
} | ConvertTo-Json -Depth 5

$result = Invoke-RestMethod `
  http://127.0.0.1:8080/v1/chat/completions `
  -Method Post `
  -ContentType "application/json" `
  -Body $body

$result.choices[0].message.content
```

The server also accepts a direct `prompt` field on this endpoint, which is
useful for small scripts.

### 6. Call it with curl

PowerShell:

```powershell
curl.exe http://127.0.0.1:8080/v1/chat/completions `
  -H "Content-Type: application/json" `
  -d '{"model":"katali-code-hybrid","messages":[{"role":"user","content":"Write a Rust function that adds two i32 values."}],"max_tokens":64}'
```

### 7. Call it from Python

The API uses ordinary HTTP and does not require an SDK:

```python
import requests

response = requests.post(
    "http://127.0.0.1:8080/v1/chat/completions",
    json={
        "model": "katali-code-hybrid",
        "messages": [
            {"role": "user", "content": "Write a safe Rust hello-world program."}
        ],
        "max_tokens": 96,
    },
    timeout=300,
)
response.raise_for_status()
print(response.json()["choices"][0]["message"]["content"])
```

### 8. Response format

Successful generation returns an OpenAI-style response:

```json
{
  "id": "katali-hybrid",
  "object": "chat.completion",
  "model": "C:\\models\\model.khyb",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "generated text"},
      "finish_reason": "stop"
    }
  ]
}
```

### 9. Troubleshooting

- `connection refused`: start `katali-hybrid-api.exe` and check the port.
- `missing prompt`: include `prompt`, `content`, or a message with `content`.
- `hybrid runtime failed`: verify the `.khyb` path and keep the runtime DLLs
  beside the executables.
- Slow first request: the resident API loads all embedded models before the
  first generation. Later requests reuse them.
- Requests are currently processed one at a time. Run separate API processes
  on different ports if you need isolated parallel experiments.

### 10. Stop the API

Press `Ctrl+C` in the server PowerShell window.

The API is an experimental local code-generation service. It does not provide
authentication, HTTPS, remote access, streaming, or multi-user scheduling.
