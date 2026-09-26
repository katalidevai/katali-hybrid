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

This package is approximately 3.3 GB and is currently about 1.9x slower in cold-start testing than the original pair, but it passed the initial factual, coding, reasoning, and instruction tests more reliably.

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
