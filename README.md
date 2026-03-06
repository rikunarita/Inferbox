# Inferbox

<p align="center">
  <b>Browser‑native AI inference.</b><br>
  Run LLMs locally in your browser — no server, no API keys, complete privacy.
</p>

<!-- ⭘⭘の画像を入れる (Hero / banner image) -->

---

## TL;DR

Inferbox is a production‑grade, privacy‑first, single‑file web application that runs large language models **entirely in the browser**. Load ONNX or GGUF models from Hugging Face, run inference with WebGPU/WASM, attach files for context, and keep full conversation history — all without a backend.

**Why this repo will attract stars:** privacy-first, extremely low friction (single file), demonstrates modern browser AI techniques (WebGPU, wasm LLMs), and is immediately usable for demos and research.

---

## Demo

* Open `index.html` on a static host or local server and load a model from the settings panel.
* Example models: ONNX-based transformer repos or GGUF files for wllama/llama.cpp WASM.

**Demo GIF / short video placeholder:**

<!-- ⭘⭘の画像を入れる (Demo GIF showing load → chat → streaming) -->

---

## Features (concise)

* **Local client-side inference** (no server, no API keys)
* **ONNX & GGUF support** (Transformers.js + wllama/llama.cpp WASM)
* **WebGPU-first** with WASM/CPU fallback
* **Streaming token output** and partial responses
* **IndexedDB caching** for models & chat history (Dexie.js)
* **File attachments**: text, code, PDF; optional image/audio when mmproj is set
* **Markdown + syntax highlighting** with DOMPurify + Highlight.js
* **i18n** (EN / JA / ZH / ES / FR)
* **Single-file deploy** (just `index.html`)

---

## Table of Contents

1. [Quickstart](#quickstart)
2. [How it works](#how-it-works)
3. [Model compatibility & recommended models](#model-compatibility--recommended-models)
4. [Deployment & hosting recommendations](#deployment--hosting-recommendations)
5. [Performance tuning & tips](#performance-tuning--tips)
6. [Security & privacy](#security--privacy)
7. [Benchmarks](#benchmarks)
8. [Troubleshooting & FAQ](#troubleshooting--faq)
9. [Contributing](#contributing)
10. [Roadmap](#roadmap)
11. [License & credits](#license--credits)

---

## Quickstart

**Get a local copy running (30s):**

```bash
# Serve the repo (or the single file) locally
python -m http.server 8000
# or
npx serve .
```

Open `http://localhost:8000` (or the deployed URL), click **Settings → Load Model**, and paste a Hugging Face repo ID or a direct GGUF URL.

**Notes:**

* For ONNX, use a repo with `transformers.js` compatible exports.
* For GGUF, use a direct URL or HF repo/filename and respect browser memory limits (see Model Requirements).

---

## How it works (technical summary)

1. **Model acquisition** — The UI downloads model artifacts from Hugging Face or a direct URL. Downloads are performed with range requests where possible and cached to IndexedDB.
2. **Runtime selection** — Detects WebGPU; falls back to WASM/CPU. WebGL not used for compute in the default flow.
3. **Inference runtime** —

   * ONNX → loaded with `@huggingface/transformers` (Transformers.js). Streaming via `TextStreamer`.
   * GGUF → loaded with `@wllama/wllama` (WASM port of llama.cpp). Streaming via onNewToken callbacks.
4. **UI** — Streaming tokens are appended to a message row for instant feedback. Messages and attachments are saved to IndexedDB using Dexie.
5. **Multimodal** — Optional mmproj file (CLIP projection) enables image/audio attachments which are handled as context markers for models that support vision.

---

## Model compatibility & recommended models

### Supported formats

* **ONNX** (Transformers.js compatible): best for WebGPU acceleration and smaller quantized graphs (q4/q8).
* **GGUF** (llama.cpp / wllama): WASM-based, broad compatibility with GGUF artifacts (Q4/K_M variants recommended for browser).

### Recommended starter models

* `onnx-community/Qwen2.5-0.5B-Instruct` (example ONNX small instruct model)
* `unsloth/Qwen3-0.6B-GGUF` (example GGUF small)

> For high-quality chat on consumer devices, prefer models in the 100M–2B parameter range quantized to Q4 or Q8 for reasonable latency.

---

## Deployment & hosting recommendations

**Static hosting (simple, recommended):** GitHub Pages, Cloudflare Pages, Netlify, Vercel, or Hugo/Docs sites. Upload the single `index.html` file.

**COOP/COEP for SharedArrayBuffer (optional):**
To enable multi-threaded WASM for wllama use, serve with the following headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

This is optional; the app will run in single-threaded mode without those headers.

**CSP guidance:** In production add strict `Content-Security-Policy` headers to restrict `script-src` and `connect-src` to known CDNs and Hugging Face.

---

## Performance tuning & tips

* **Prefer WebGPU:** Much faster for ONNX on supported GPUs. Use `navigator.gpu` detection to confirm availability.
* **Use q4/q8 quantized models:** Dramatically reduces memory and improves throughput in the browser.
* **Split very large GGUF files:** Browsers have limits on ArrayBuffer size; split models if > 2GB.
* **Limit `max_new_tokens`:** For interactive latency, keep generations small (e.g., 128–512) and fetch followups if needed.
* **Use IndexedDB caching:** Models will reuse downloads across sessions; ensure cache storage quota is available.

---

## Security & privacy

* **Data never leaves the user’s device** (except model weights fetched from Hugging Face or direct URLs).
* **No telemetry** and no external backends by default.
* **Sanitize rendered output** using DOMPurify to avoid XSS when rendering markdown/HTML returned by models.

If you integrate this into a hosted product, consider privacy disclosures around downloaded model provenance.

---

## Troubleshooting & FAQ

**Q: The model fails to load with an "ArrayBuffer" error.**
A: Large single-file downloads can exceed browser limits. Use multi-part GGUF or smaller quantized models.

**Q: WebGPU is not detected.**
A: Ensure your browser supports WebGPU (Chrome/Edge Canary or recent builds). Fallback to WASM is automatic.

**Q: How do I enable image uploads?**
A: Provide an mmproj URL in settings that points to a CLIP projection model compatible with your multimodal LLM.

**Q: Why is streaming slow/stops?**
A: Check console for memory pressure. Reduce `max_new_tokens` and use quantized models.

---

## Contributing

Contributions, issues and feature requests are welcome! Please follow these guidelines:

1. Open an issue describing your change first.
2. Fork the repo and create a branch for your feature/bugfix.
3. Keep commits focused and tests (where relevant) included.

**Suggested contributions:**

* Add example repos for ONNX/GGUF models that are verified to work
* Add benchmark scripts & CI performance tests
* Add additional language translations

---

## Roadmap

Planned improvements to increase adoption:

* Demo hosting with sample models & pre‑downloaded caches
* Native WebGPU GGUF support via WASM compute
* Official model compatibility matrix and verified model list
* GitHub Actions to run CI performance smoke tests

---

## License & credits

MIT License 
Key libraries & credits:

* Dexie.js, Marked, DOMPurify, Highlight.js, PDF.js, Lucide, @huggingface/transformers, @wllama/wllama

---

## Contact / Maintainer

Project created & maintained by the repository owner.
