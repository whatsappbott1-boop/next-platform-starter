# Local AI + 3D Production Stack for a Faceless, Voice-Free Interactive TikTok LIVE (RTX 5060 Ti 16 GB / Ryzen 5 7500F / 32 GB RAM / Windows 11)

Research date: 2026-11-21. Method note: in this session many primary hosts were not reachable by the fetch tool (arxiv.org, huggingface.co, developer.nvidia.com, dev.epicgames.com, docs.warudo.app, resemble.ai, glukhov.org, ablt.dev, ssdnodes.com). Where a claim comes only from a search-engine summary of a page that could not be opened, that is marked "(search summary)". GitHub pages could be opened and were read directly. Estimates I made myself are kept in the "Inferences" sections and labelled as estimates.

---

## 1. Local LLMs (Ollama / LM Studio / llama.cpp): which models, how fast, how good in Turkish, what license, and what size fits next to UE5

### Takeaway
With UE5 rendering on the same 16 GB card, the realistic LLM tier is about 4B–9B at Q4: Qwen3.5-4B/9B, Gemma 4 E4B, or Ministral 3 8B. All three are Apache-2.0, so they are safe on a monetized stream. Gemma 3/4 12B and other 12–14B models fit only if the UE5 scene stays lean. Turkish-only fine-tunes such as Trendyol, Cosmos and Kumru have not been shown to beat general multilingual models. Use one only if your own Turkish tests favour it.

### Cited Findings
**Raw throughput on this exact GPU**
- llama.cpp official CUDA scoreboard, RTX 5060 Ti 16 GB, Llama-2 7B Q4_0 (posted 2 Aug 2025): prompt processing pp512 = 3,737 t/s (4,196 t/s with flash-attention); generation tg128 = 90.9 t/s (93.5 t/s with FA). On the same build an RTX 4060 Ti 8 GB generated 64 t/s and an RTX 5070 12 GB generated 128 t/s — [llama.cpp discussion #15013](https://github.com/ggml-org/llama.cpp/discussions/15013)
- Third-party numbers for Llama 3.1 8B Q4 on the 5060 Ti differ: one guide reports 51–60 t/s and another 75.1 t/s, so setup matters — [InsiderLLM 5060 Ti guide](https://insiderllm.com/guides/rtx-5060-ti-local-ai-benchmarks/) (search summary); [glukhov.org 16 GB benchmarks](https://glukhov.org/llm-performance/benchmarks/best-llm-on-16gb-vram-gpu/) (search summary)
- Qwen3-14B Q4_K_M: about 35.4 t/s decode with a 33K-context ceiling on the 5060 Ti 16 GB. This is a calculator estimate, not a measured run — [willitrunai](https://willitrunai.com/es/can-run/qwen-3-14b-on-rtx-5060-ti-16gb) (search summary)
- Gemma 3 12B Q4_K_M: about 40 t/s (estimate). The VRAM estimate is about 11.1 GB at 8K context and 20.4 GB at 32K, which would not fit. Other calculators give 7.2–9.6 GB for the weights alone — [willitrunai](https://willitrunai.com/can-run/gemma-3-12b-on-rtx-5060-ti-16gb); [llmconfigurator](https://llmconfigurator.com/en/can-i-run/gemma-3-12b-on-rtx-5060-ti-16) (search summaries)
- Qwen 3.5 35B MoE reportedly ran at about 40 t/s with 100K context on a 5060 Ti. This most likely relies on offloading experts to system RAM — [ai-radar.it](https://ai-radar.it/article/qwen-3-5-35b-moe-40-token-s-su-rtx-5060-ti-con-contesto-100k) (search summary). A claim of "Qwen3.6-27B at 40 tok/s on a single 5060 Ti" — [aiweekly](https://aiweekly.co/alerts/qwen36-27b-hits-40-toks-on-single-rtx-5060-ti-16gb) (search summary, unverified; such a model would leave no room for UE5 in any case)
- An arXiv paper (2601.09527, "Private LLM Inference on Consumer Blackwell GPUs") benchmarks Qwen3-8B, Gemma3-12B, Gemma3-27B and GPT-OSS-20B on the 5060 Ti across 79 configurations. Its tables could not be retrieved — [arXiv 2601.09527](https://arxiv.org/html/2601.09527v1)

**2026 model families (sizes, VRAM, license)**
- **Gemma 4** (released 2 Apr 2026): dense E2B (2.3B effective), E4B (4.5B effective), 12B "Unified" (11.95B) and 31B, plus a 26B-A4B MoE (3.8B active). Context is 128K (E2B/E4B) or 256K. Audio input works on E2B/E4B/12B. 35+ languages are supported out of the box and 140+ appear in pre-training. **License: Apache 2.0**, which replaces the older restrictive Gemma terms — [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4); [Google blog](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/); [The Register](https://www.theregister.com/2026/04/02/googles_gemma_4_open_weights/)
- Gemma 4 Ollama download sizes: e2b about 7.2 GB, e4b about 9.6 GB, 12b about 7.6 GB, 26b about 18 GB, 31b about 20 GB. Unsloth's 4-bit VRAM estimates: E4B 5.5–6 GB, 12B 7–8 GB, 26B-A4B 16–18 GB. Google's own table gives 4-bit weights of 4.5 GB (E4B) and 6.7 GB (12B), with context on top — [Ollama gemma4](https://ollama.com/library/gemma4:e4b); [Unsloth Gemma 4 docs](https://unsloth.ai/docs/models/gemma-4) (search summaries)
- **Qwen3.5 small models** (released 2 Mar 2026): 0.8B, 2B, 4B and 9B dense, base and instruct, **Apache 2.0**, available on Ollama (`qwen3.5:9b`). At 4-bit the 9B needs about 6 GB and the 4B about 3 GB. Artificial Analysis rates the 9B and 4B as the most intelligent models in their size classes — [Artificial Analysis](https://artificialanalysis.ai/articles/qwen3-5-small-models); [VentureBeat](https://venturebeat.com/technology/alibabas-small-open-source-qwen3-5-9b-beats-openais-gpt-oss-120b-and-can-run); [HF Qwen3.5-9B](https://huggingface.co/Qwen/Qwen3.5-9B) (search summaries)
- **Ministral 3** (2 Dec 2025): 3B, 8B and 14B, each as base, instruct and reasoning. All are **Apache 2.0**, multimodal and multilingual, with a 256K context — [Mistral](https://mistral.ai/news/mistral-3); [The Register](https://www.theregister.com/2025/12/02/mistral_3/) (search summaries)

**Turkish quality evidence**
- **Cetvel** benchmark (EACL 2026): 23 Turkish tasks, including grammar correction, machine translation and QA on Turkish history and idioms. It evaluated 33 open models up to 70B. Llama 3 variants were strongest within their size classes, and "Turkish-centric instruction-tuned models generally underperform relative to multilingual or general-purpose models (e.g. Llama 3 and Mistral)" — [ACL Anthology 2026.eacl-long.46](https://aclanthology.org/2026.eacl-long.46); [arXiv 2508.16431](https://arxiv.org/abs/2508.16431v1) (search summary; Gemma 3, Qwen3 and newer models apparently not covered)
- **TurkBench** (SIGTURK 2026): 8,151 samples across 21 subtasks, with an online HF leaderboard (huggingface.co/turkbench). No scores were retrieved — [ACL Anthology 2026.sigturk-1.12](https://aclanthology.org/2026.sigturk-1.12)
- The YTU Cosmos Turkish-Gemma-9b-v0.1 model card is self-reported, not independent. Turkish win rates: gemma-3-12b-it 61.61%, Qwen3-30B-A3B 62.39%, Cosmos Turkish-Llama-8b-DPO 36.89%. Turkish benchmark averages: gemma-3-12b-it 62.94, gemma-3-27b-it 67.36, Llama-3.1-70B-Instruct 63.92 — [HF ytu-ce-cosmos/Turkish-Gemma-9b-v0.1](https://huggingface.co/ytu-ce-cosmos/Turkish-Gemma-9b-v0.1) (search summary)
- **Trendyol-LLM-8B-T1**: built on Qwen3-8B, listed as **Apache-2.0** — [Featherless model readme](https://featherless.ai/models/Trendyol/Trendyol-LLM-8B-T1/readme) (search summary)
- **Kumru-2B (VNGRS, Oct 2025)**: trained from scratch for Turkish on a 500 GB corpus (300B tokens), with 1M SFT examples. It runs in about 4.8 GB. A larger 7.4B Kumru exists. Some observers questioned how novel it is. A community Ollama port exists — [HF vngrs-ai/Kumru-2B](https://huggingface.co/vngrs-ai/Kumru-2B); [VNGRS Medium](https://medium.com/vngrs/kumru-llm-34d1628cfd93); [Tamindir](https://www.tamindir.com/haber/kumru-llm-tanitildi_100596/); [Erkan Saka commentary](https://erkansaka.net/2025/10/14/turna-ai-kumru-2b-launch-reactions-turkey-llm/); [Ollama alibayram/kumru](https://ollama.com/alibayram/kumru) (search summaries)
- YTU Cosmos also publishes Turkish-Llama-8b and Turkish-Gemma-9b. A community Ollama port of Turkish-Gemma-9b exists — [Ollama alibayram/turkish-gemma-9b-v0.1](https://ollama.com/alibayram/turkish-gemma-9b-v0.1); [turkish-nlp-resources](https://github.com/agmmnn/turkish-nlp-resources) (search summary)

### Inferences
- **Recommended LLM tier while UE5 runs:** Qwen3.5-4B Q4_K_M (about 3 GB weights plus about 0.5–1 GB KV cache at 4–8K context) or Gemma 4 E4B (about 5.5–6 GB). These are the default picks for chat, quiz and story generation. Qwen3.5-9B, Ministral 3 8B or Trendyol-8B-T1 (about 6–7 GB with context) fit only with a light UE5 scene (≤6 GB). 12–14B models (Gemma 4 12B, Gemma 3 12B, Ministral 14B) are realistic only if the game uses ≤4–5 GB or the LLM runs on the CPU.
- The llama.cpp result of about 91 t/s for a 7B model on an idle GPU suggests a 4B model will exceed 100 t/s alone. Even if UE5 contention halves that, it stays well above the roughly 15–25 t/s needed to show chat replies in real time. Quiz and story items are short. Generation speed is not the bottleneck; VRAM headroom and GPU contention (frame drops while the LLM decodes) are.
- **Turkish choice:** Gemma-family models score well in the available Turkish evidence, and Gemma 4 covers 140+ languages in pre-training. Gemma 4 E4B is therefore the best first candidate for Turkish chat, with Qwen3.5-4B/9B as the alternative. Trendyol-8B-T1 (Apache, Qwen3 base) is the best Turkish fine-tune to A/B test. Cetvel suggests Turkish-only fine-tunes rarely beat general models.
- **Licenses for a monetized stream:** Qwen3.5, Gemma 4, Ministral 3 and Trendyol-8B-T1 are Apache-2.0, the safest option. From prior knowledge (not re-verified this session): Llama 3.x and 4 use the Llama Community License, which allows commercial use under 700M MAU with "Built with Llama" attribution. Gemma 3 and Gemma 2-based fine-tunes (Cosmos Turkish-Gemma-9b) fall under the older Gemma Terms of Use with a prohibited-use policy. Commercial use is allowed, but these terms are less clean than Apache.
- **CPU fallback (estimate, not benchmarked):** the Ryzen 5 7500F with dual-channel DDR5 should give roughly 8–15 t/s for a 4B Q4 model and roughly 5–8 t/s for 8B Q4. That is fine for pre-generating quiz banks, and marginal for live chat.

### Gaps
- Measured (not calculator) tokens/s for Qwen3.5-4B/9B, Gemma 4 E4B/12B and Ministral 3 on the 5060 Ti could not be found. The arXiv 2601.09527 tables were unreachable.
- No independent Turkish leaderboard results for Gemma 4, Qwen3.5 or Ministral 3 were retrieved. The TurkBench HF leaderboard and Cetvel GitHub were not reachable.
- Licenses for Kumru-2B and the Cosmos models were not verified from primary model cards.
- No benchmark exists for LLM throughput while a UE5 game runs on the same GPU. This needs a local test.

---

## 2. Free TTS with commercial-friendly licenses and Turkish support

### Takeaway
No permissively licensed, maintained, high-quality Turkish TTS exists as of Nov 2026. The best commercial-safe Turkish option is **Chatterbox Multilingual V3** (MIT, 500M parameters, includes Turkish). Two caveats: it was "retired" in Aug 2026 with no further maintenance, and every output carries an inaudible PerTh watermark. XTTS-v2 has the best-known Turkish quality but is **non-commercial (CPML)**. No Piper Turkish voice is on Piper's commercial-safe list. For English, **Kokoro-82M** (Apache-2.0, CPU-capable) is the safest choice, with **Qwen3-TTS** (Apache-2.0) for expressive or cloned English voices.

### Cited Findings
**Chatterbox (Resemble AI)**
- Chatterbox-Multilingual V3: 500M parameters, 23+ languages, **Turkish (tr) included**, MIT license. Variants: Turbo (350M, English only, paralinguistic tags such as [laugh], 1-step decoder, lower VRAM), Nano (110M, English only, about 3× realtime on 8 CPU cores) and a legacy V2 multilingual checkpoint. "As of August 2026, Chatterbox is retired research and is no longer maintained"; the code stays public as an archive under MIT. Resemble says it no longer sells TTS. Every output carries an imperceptible **PerTh watermark** that survives MP3 compression — [GitHub resemble-ai/chatterbox](https://github.com/resemble-ai/chatterbox)
- The HF card lists license MIT and "tr" among the language codes. NVIDIA's NIM packaging falls under the NVIDIA Open Model Agreement, but the base model is MIT — [HF README](https://huggingface.co/ResembleAI/chatterbox/blob/main/README.md); [NVIDIA build model card](https://build.nvidia.com/resembleai/chatterbox-multilingual-tts/modelcard) (search summaries)

**Piper**
- The current engine repo is OHF-Voice/piper1-gpl, **GPL-3.0**. The Open Home Foundation is "looking for maintainers" — [GitHub piper1-gpl](https://github.com/OHF-Voice/piper1-gpl)
- Turkish voices: `tr_TR-dfki-medium`, `tr_TR-fahrettin-medium`, `tr_TR-fettah-medium`, all medium quality — [Piper VOICES.md](https://github.com/rhasspy/piper/blob/master/VOICES.md)
- Piper "makes no license grant of its own over voice models". Each voice's rights come from its training data. Lessac-derived voices are limited to "research and exploration". Piper's COMMERCIAL_USE list (69 voices whose whole dataset chain permits commercial use) contains **no Turkish voice**. It does contain 14 English voices, including en_US-ljspeech-high, en_US-libritts-high, en_GB-cori-high, en_US-norman-medium and en_GB-alba-medium — [LICENSING.md](https://github.com/OHF-Voice/piper1-gpl/blob/main/docs/LICENSING.md); [COMMERCIAL_USE.md](https://github.com/OHF-Voice/piper1-gpl/blob/main/docs/COMMERCIAL_USE.md)
- Third-party listings say tr_TR-dfki was trained on MaryTTS DFKI Turkish data and fine-tuned from the en_US lessac voice, and label it CC BY-NC-SA 4.0 (non-commercial) — [tts.ai Piper Turkish](https://tts.ai/voices/piper/?lang=tr); [SSD Nodes TR TTS comparison](https://www.ssdnodes.com/learn/lang/tr/open-source-turkish-tts-models-compared) (search summaries; not primary)

**Kokoro-82M**
- Apache-2.0 weights. Languages: American/British English, Spanish, French, Hindi, Italian, Japanese, Brazilian Portuguese, Mandarin. **No Turkish** — [GitHub hexgrad/kokoro](https://github.com/hexgrad/kokoro). It depends on eSpeak-ng (GPLv3) — [HF Kokoro README](https://huggingface.co/hexgrad/Kokoro-82M/blob/refs%2Fpr%2F44/README.md) (search summary)
- A community Turkish fine-tune, duxx/kikiri-turkish-base (about 53 h of Turkish audio), exists. Its quality and data license were not verified — [HF kikiri-turkish-base](https://huggingface.co/duxx/kikiri-turkish-base) (search summary)

**Non-commercial or problematic options**
- **Coqui XTTS-v2**: Coqui Public Model License 1.0.0, non-commercial for both the model and its outputs. Coqui shut down in early 2024, so no commercial license can be bought, and emails to licensing@coqui.ai went unanswered. The library code is MPL-2.0 — [HF XTTS-v2 discussion #65](https://huggingface.co/coqui/XTTS-v2/discussions/65); [GitHub coqui-ai/TTS #4145](https://github.com/coqui-ai/TTS/discussions/4145); [LocalAIMaster](https://localaimaster.com/blog/xtts-coqui-commercial-license) (search summaries). A developer guide calls XTTS-v2 "arguably the most capable open-source multilingual TTS for Turkish" — [ablt.dev Turkish TTS guide](https://www.ablt.dev/blog/turkish-tts/) (search summary)
- **F5-TTS**: code MIT; official checkpoints relicensed to **CC-BY-NC-4.0** in Oct 2024 because the Emilia training data is in-the-wild. A commercial-friendly retrain, OpenF5 (CC-BY), is a work in progress with low quality — [HF SWivid/F5-TTS README](https://huggingface.co/SWivid/F5-TTS/blob/refs%2Fpr%2F11/README.md); [HF OpenF5](https://huggingface.co/mrfakename/OpenF5-Intermediate) (search summaries)
- **Fish Speech / OpenAudio S1-mini**: CC-BY-NC-SA-4.0, and the gated form requires "non-commercial use ONLY". S2-Pro: research license, commercial use needs a separate deal. Turkish not confirmed — [HF openaudio-s1-mini](https://huggingface.co/fishaudio/openaudio-s1-mini/blob/main/README.md); [HF s2-pro LICENSE](https://huggingface.co/fishaudio/s2-pro/blob/main/LICENSE.md) (search summaries)
- **Qwen3-TTS** (Jan 2026): Apache-2.0; 0.6B and 1.7B (Base, CustomVoice, VoiceDesign). Ten languages: zh, en, ja, ko, de, fr, ru, pt, es, it. **No Turkish.** Streaming with first packet as low as 97 ms — [GitHub QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS); [Alibaba Cloud blog](https://www.alibabacloud.com/blog/qwen3-tts-family-is-now-open-sourced-voice-design-clone-and-generation_602826)
- **Microsoft VibeVoice**: Microsoft removed the TTS code in Sept 2025, citing responsible AI. VibeVoice-Realtime-0.5B remains MIT with streaming text input and about 300 ms first chunk (secondary source) — [GitHub microsoft/VibeVoice](https://github.com/microsoft/VibeVoice); [HF VibeVoice-Realtime-0.5B](https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B); [Apidog](https://apidog.com/blog/microsoft-vibevoice) (search summaries; Turkish not confirmed)
- **Edge TTS (edge-tts Python)**: GPL-3 code that calls Microsoft's proprietary online service. A Microsoft Q&A answer warns that commercial use without an Azure subscription "could be a violation of our terms of service" and points to Azure AI Speech. Custom SSML is blocked. There is no SLA — [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/2088770/are-opensource-edge-tts-free-for-commercial-use); [Microsoft Q&A 2](https://learn.microsoft.com/en-us/answers/questions/2392491/unofficial-edge-tts-api); [PyPI edge-tts](https://pypi.python.org/pypi/edge-tts); [wikidocs blog](https://wikidocs.net/blog/@jaehong/9554/) (search summaries)
- Other Turkish-specific models: Orkhon-TTS (F5-TTS-based, Apache-2.0, alpha, voice cloning); EMA Lightning (8.6M params, about 34 MB, Apache-2.0); Trendyol-TTS (tagged MIT but the repo has no LICENSE file); Coqui Glow-TTS on Turkish Common Voice (MPL-2.0, robotic) — [Turkish TTS survey](https://danielquillanroxas.github.io/turkish-tts-survey); [SSD Nodes TR comparison](https://www.ssdnodes.com/learn/lang/tr/open-source-turkish-tts-models-compared); [ablt.dev](https://www.ablt.dev/blog/turkish-tts/) (search summaries)
- Several 2026 license-checked roundups also list XTTS-v2, Fish S2 Pro, Voxtral TTS and F5-TTS weights as non-commercial — [Cekura](https://www.cekura.ai/blogs/open-source-tts); [Yaps](https://www.yaps.ai/blog/best-local-tts-models) (search summaries)

### Inferences
- **Best Turkish (commercial-safe): Chatterbox Multilingual V3.** It is MIT-licensed and supports Turkish. Pin the exact checkpoint and keep a local copy, because the project is archived. Run it on the GPU. My estimate for a 500M model in fp16 plus vocoder is about 2–3 GB VRAM. It is not streaming-native, so synthesize sentence by sentence. The PerTh watermark is inaudible and not a legal problem, but viewers or platforms could in principle detect it as AI audio. Fallbacks: Orkhon-TTS or EMA Lightning (Apache, alpha quality; test them) or the paid Azure Speech cloud API for neural Turkish voices. Edge TTS is free but carries real ToS risk on a monetized stream.
- **Best English: Kokoro-82M** on CPU (Apache, tiny, 0 GB VRAM) or Piper's commercial-list English voices (CPU, real-time). Use Qwen3-TTS 0.6B (Apache, streaming) for more expressive or cloned English voices on the GPU, about 1.5–3 GB (estimate).
- **Avoid on a monetized stream:** XTTS-v2, F5-TTS official weights, Fish/OpenAudio, Piper Turkish voices (none commercially cleared; dfki is reportedly lessac-derived and NC), and any voice cloned from a person without consent.
- **"Voice-free" stream:** if the concept is literally voice-free (text, music and SFX only), TTS can be dropped, freeing about 2–3 GB VRAM. If a character voice is wanted, put Turkish TTS on the GPU and English TTS on the CPU to balance load.
- **Pre-generation:** fixed lines (quiz intros, gift thank-yous by tier, story narration) can be pre-rendered offline into a WAV bank. Live TTS is then only needed for dynamic chat replies, which cuts GPU contention.

### Gaps
- No objective Turkish MOS or CER comparison of Chatterbox ML, kikiri, Orkhon and EMA Lightning was found.
- Chatterbox ML VRAM and RTF on a 5060 Ti were not measured.
- The fahrettin and fettah Piper voice licences could not be read: the MODEL_CARDs are on HF, which was unreachable. They are absent from the commercial list, so treat them as uncleared.
- Kyutai TTS, Orpheus, Dia, MeloTTS, CosyVoice and IndexTTS were not researched this session. From prior knowledge none of them lists Turkish, but this was not verified.

---

## 3. Lip-sync / facial animation without a camera (3D or 2D avatar)

### Takeaway
For UE5, NVIDIA Audio2Face-3D was open-sourced in 2025. The SDK and UE5 plugin are MIT, the weights fall under the NVIDIA Open Model License (commercial use allowed), and it is the strongest free real-time audio-to-face option. Epic's own MetaHuman audio-driven animation is free (UE EULA from 5.6) but officially offline/editor-centric. For 2D, VTube Studio requires a paid version for monetized streams because of Live2D licensing. The zero-budget path is VRoid Studio avatar + Mixamo animations + UE5 (or Warudo) with Audio2Face or simple viseme/amplitude lip-sync.

### Cited Findings
- **Audio2Face-3D components and licenses:** SDK (C++, real-time inference, GPU with CPU fallback) **MIT**; UE5 plugin v2.5 (Blueprint, UE 5.5/5.6) **MIT**; Maya plugin v2.0 **MIT**; Training Framework v1.0 **Apache**; NIM microservice under NVIDIA Software License + AI Product terms. Audio2Face-3D models (regression v2.3, diffusion v3.0: Mark, Claire, James) under "NVIDIA Open Model". Audio2Emotion v2.2/v3.0 under a custom license that allows use only with Audio2Face. Claire sample dataset: evaluation only — [GitHub NVIDIA/Audio2Face-3D](https://github.com/NVIDIA/Audio2Face-3D)
- The HF model card says the model is "ready for commercial/non-commercial use" under the NVIDIA Open Model License — [HF Audio2Face-3D-v2.3-Mark](https://huggingface.co/nvidia/Audio2Face-3D-v2.3-Mark/blob/main/README.md) (search summary). NVIDIA's ACE page lists UE 5.4, 5.5, 5.6 and 5.7 plugin downloads — [NVIDIA ACE](https://developer.nvidia.com/ace-for-games) (search summary). Open-source announcement Sept 2025 — [NVIDIA blog](https://developer.nvidia.com/blog/nvidia-open-sources-audio2face-animation-model); [Gigazine](https://gigazine.net/gsc_news/en/20250926-nvidia-audio2face-open-sources) (search summaries)
- **MetaHuman licensing:** from UE 5.6, MetaHuman is covered by the standard UE EULA. It is free under $1M USD revenue and may be used "with any engine or creative software", including Unity and Godot. The old cloud MetaHuman Creator EULA applies only to UE ≤5.5 — [metahuman.com/license](https://www.metahuman.com/license); [CG Channel, Jun 2025](https://www.cgchannel.com/2025/06/you-can-now-sell-metahumans-or-use-them-in-unity-or-godot/); [MetaHuman 5.6 news](https://metahuman.com/news/metahuman-leaves-early-access-with-a-feature-packed-new-release) (search summaries)
- **MetaHuman audio-driven animation:** Epic's docs describe the audio solver as an offline process, distinct from the real-time MetaHuman Audio Live Link Source. A UE 5.7 developer reports the live Audio Live Link path is designed for physical audio devices, not programmatic PCM from TTS. An Epic reply said "at the moment we don't directly support audio driven animation at runtime" — [Epic docs](https://dev.epicgames.com/documentation/en-us/metahuman/audio-driven-animation); [UE forum xADA thread](https://forums.unrealengine.com/t/metahuman-xada-for-custom-audio-streams-guidance-needed/2675424); [UE forum MH Animator runtime](https://forums.unrealengine.com/t/mh-animator-runtime-lipsync-from-audio/2611561) (search summaries)
- A paid community plugin, "Runtime MetaHuman Lip Sync" (UE 5.0–5.7), supports real-time lip-sync from mic, TTS or any streaming audio in any language — [UE forum](https://forums.unrealengine.com/t/runtime-metahuman-lip-sync-real-time-offline-lip-sync-with-audio-tts-cc4-genesis-arkit-and-more/2383289) (search summary)
- **VTube Studio:** "If you use VTube Studio commercially (YouTube superchats, monetized Twitch, etc.)" you must buy a paid version or DLC; "This is required due to Live2D licensing." On Steam a watermark shows only while webcam tracking is active — [VTube Studio FAQ (GitHub wiki)](https://github.com/DenchiSoft/VTubeStudio/wiki/FAQ); [Steam EULA](https://store.steampowered.com//eula/1325860_eula_0)
- **Warudo:** free for non-streaming use. Free for streaming "if and only if" certain conditions hold, which were not retrievable. Steam says "100% free for personal use" and the developer says it will remain free for indie VTubers. Supports VRM and humanoid rigs — [Warudo docs](https://docs.warudo.app/docs/); [Steam Warudo](https://store.steampowered.com/app/2079120) (search summaries)
- **VRoid Studio:** any individual or company may sell or use commercially the meshes, textures and presets VRoid Studio provides, unless a special clause says otherwise. Third-party assets follow their own licenses — [VRoid Studio Guidelines](https://vroid.com/en/studio/guidelines) (search summary)
- **Mixamo:** per Adobe community and FAQ answers (informal), characters and animations may be used free in commercial projects, but must not be resold or redistributed as raw files — [Adobe Mixamo FAQ thread](https://community.adobe.com/t5/mixamo-discussions/mixamo-faq-licensing-royalties-ownership-eula-and-tos/td-p/13234775) (search summary)

### Inferences
- **Most value for zero budget (3D/UE5):** a VRoid-made stylized avatar or a MetaHuman (free under $1M), Mixamo idle/gesture loops, and the Audio2Face-3D UE plugin driven by the TTS WAV/PCM. The project is faceless, so a stylized mascot or creature whose mouth only needs basic visemes may be enough. A cheaper option is to drive jaw and mouth blendshapes from TTS audio amplitude, or from phoneme timings if the TTS exposes them, at zero VRAM.
- Audio2Face-3D is trained mainly on English speech. Lip shapes for Turkish should still look plausible, because the model is audio-driven and does not need text. Test this; it is not verified.
- **2D route:** Live2D + VTube Studio costs money for monetized use, which rules it out at strictly zero budget. A VRM avatar in Warudo (free tier conditions to be confirmed) or directly in UE5 avoids Live2D fees.
- On this concept (gift-driven game, no presenter) an avatar may not be necessary at all. Characters inside the game world (zoo animals, a narrator mascot) can use canned Mixamo or keyframed animations plus simple mouth flaps.

### Gaps
- Audio2Face-3D VRAM and latency on a 5060 Ti alongside a UE5 game: no data found.
- Warudo's exact free-streaming conditions, the Live2D Cubism SDK/editor license for small creators, and the OVR LipSync and VSeeFace licenses were not verified this session.
- Adobe's binding Mixamo supplementary terms were not read; only forum answers were.

---

## 4. UE5 on the RTX 5060 Ti 16 GB (1080×1920 vertical), DLSS 4, and engine alternatives (Unity 6, Godot 4)

### Takeaway
The 5060 Ti handles a stylized, medium UE5 scene comfortably at 1080×1920/30 fps if you avoid heavy Lumen hardware RT and use baked or simple lighting plus DLSS Super Resolution. The VRAM figures below are estimates; no measured UE5 VRAM data at this resolution was found. Licensing: UE is free with a 5% royalty only on interactive "Products" above $1M gross. Linear content is royalty-free, and a livestream of an interactive game is a grey area that is irrelevant below $1M. Unity 6 Personal is free under $200K, the Runtime Fee is cancelled, and the splash screen is optional. Godot is the lightest-weight, fully free option for a simple 2D/2.5D gift game.

### Cited Findings
- Tom's Hardware tested UE5 games in rasterization mode and called Lumen software RT "still quite taxing" and the full-RT hit "quite severe" — [Tom's Hardware 5060 Ti 16GB review](https://www.tomshardware.com/pc-components/gpus/nvidia-geforce-rtx-5060-ti-16gb-review/5) (search summary)
- The 5060 Ti 16 GB is about 15–17% faster than the 4060 Ti at 1080p. In Star Wars Outlaws at 1080p Ultra with DLSS Performance it ran 91 fps (25 ms latency), and 156 fps with 2× frame generation (30 ms). Cyberpunk 2077 RT Overdrive reached 186 fps with 4× MFG — [Tom's Hardware](https://www.tomshardware.com/pc-components/gpus/nvidia-geforce-rtx-5060-ti-16gb-review); [Wccftech Gigabyte review](https://wccftech.com/review/gigabyte-geforce-rtx-5060-ti-gaming-oc-16-gb-graphics-card-review-top-tier-design-for-a-mainstream-gpu/5/amp/); [Engadget](https://www.engadget.com/gaming/pc/nvidia-rtx-5060-ti-16gb-review-a-solid-semi-budget-gpu-for-429-but-good-luck-scoring-that-price-130058124.html) (search summaries)
- A reviewer comment (opinion, not a measurement) says resolution changes VRAM by only a few hundred MB, because about 90% of VRAM goes to textures — [TechSpot forum](https://www.techspot.com/community/topics/nvidia-geforce-rtx-5060-ti-16gb-review-not-great-not-terrible.291808) (search summary)
- The NVIDIA DLSS 4 plugin for UE 5.2–5.6 (July 2025) includes Multi Frame Generation (RTX 50 only), Frame Generation, Ray Reconstruction, Super Resolution, DLAA and Reflex — [Wccftech](https://wccftech.com/nvidia-officially-releases-dlss-4-plugin-for-unreal-engine-5-6/); [TweakTown](https://www.tweaktown.com/news/106246/nvidia-unveils-dlss-4-plugin-for-unreal-engine-5-6-next-gen-ai-upscaling-game-developers/index.html) (search summaries; UE 5.7 plugin status not confirmed)
- **UE licensing:** standard EULA royalty is 5% of worldwide gross revenue on products beyond the threshold; "no fees or royalties" for linear content (films, videos, stills). Linear-content companies over $1M annual revenue need Unreal Subscription seats. Whether a monetized livestream of an interactive UE game counts as a royalty-bearing "Product" is not addressed explicitly — [Unreal Engine FAQ](https://www.unrealengine.com/en-US/faq); [UE EULA change log](https://www.unrealengine.com/eula-change-log/unreal) (search summaries)
- **Unity:** Runtime Fee cancelled Sept 2024. The Personal plan ceiling rose to $200,000 in revenue plus funding. The splash screen is optional in Unity 6 — [Unity blog](https://unity.com/blog/unity-is-canceling-the-runtime-fee); [Unity pricing updates](https://unity.com/pricing-updates) (search summaries)

### Inferences
- **UE5 VRAM budget (estimate):** a stylized/low-poly scene at 1080×1920 with baked lighting or Lumen software at low settings, no Virtual Shadow Maps or a low VSM page pool, DLSS Quality/Balanced, and a capped texture streaming pool should use about **3.5–6 GB**. A medium scene with Lumen + Nanite + VSM at high settings will likely use **6–9 GB**. In UE, set `r.Streaming.PoolSize` (e.g. 1500–2500 MB) and cap at `t.MaxFPS 30–60` to protect headroom for the LLM and TTS.
- **Frame generation:** not useful here. TikTok ingest is 30 fps, so render 30–60 fps natively with DLSS SR. Frame generation adds VRAM and latency and only multiplies displayed frames. Note that OBS game capture of generated frames is also not guaranteed to be smooth (inference).
- **Engine choice for a gift-driven interactive game:**
  - **UE5:** best visuals, MetaHuman and Audio2Face integration, Blueprint. Heaviest on VRAM and disk, with long shader compiles. Choose it if 3D spectacle (a virtual zoo, physics chaos) is the hook.
  - **Unity 6:** lighter, huge asset ecosystem, C#, easy to hook TikTok-gift WebSocket and REST libraries. Free under $200K.
  - **Godot 4:** fully free (MIT, from prior knowledge, not re-verified), tiny footprint (often under 1 GB VRAM for 2D/2.5D), and leaves maximum VRAM for a bigger LLM or TTS. Best for 2D, pixel or stylized-3D gift games where logic and comedy matter more than graphics.
  - For a 6–12 h unattended stream, Godot or Unity are typically more stable and less resource-hungry. UE5 is worth it only if its visuals are central to the content.

### Gaps
- No measured UE5 VRAM data for any scene type on the 5060 Ti at 1080×1920 was found.
- The UE 5.7 DLSS 4 plugin status and UE 5.7-specific Blackwell notes were not found.
- Epic's exact EULA wording for monetized livestreams of interactive UE content was not available. Contact Epic if revenue approaches $1M.

---

## 5. Image, video and 3D asset generation (ComfyUI, Flux, Qwen-Image, Wan, Hunyuan3D, TRELLIS)

### Takeaway
Use these offline (between streams) to build assets. They need most or all of the 16 GB, and compute bursts would stall UE5 and NVENC. For commercial licenses, prefer Qwen-Image (Apache-2.0), Wan 2.2 (Apache-2.0) and TRELLIS.2 (MIT, but it needs 24 GB on Linux). FLUX.1 [dev] and Kontext [dev] are non-commercial for the model, with an ambiguous status for outputs. Hunyuan3D 2.1 is usable in Turkey: the license excludes only the EU, UK and South Korea. It needs about 10 GB for shape and 21 GB for texture.

### Cited Findings
- **FLUX.1 Kontext [dev]**: FLUX.1 Non-Commercial License. The HF card says outputs may be used commercially, but a community discussion flags that the license text conflicts. Commercial deployment of the model needs a paid BFL license. FP8 runs in about 12 GB and NF4 in about 7 GB — [BFL blog](https://bfl.ai/blog/flux-1-kontext-dev); [HF discussion](https://huggingface.co/black-forest-labs/FLUX.1-Kontext-dev/discussions/6); [DEV Community guide](https://dev.to/jovan_chan_9500711396d4e6/flux1-kontext-dev-for-local-ai-in-2026-image-editing-on-consumer-gpus-without-the-api-bills-3nd3) (search summaries)
- **Qwen-Image**: Qwen-Image, 2512, the Edit releases through 2511 and Layered are Apache-2.0. Qwen-Image-2.0/Pro are API-only — [InVideo license matrix, Aug 2026](https://invideo.io/blog/open-source-image-models-licenses/) (search summary; third-party)
- **Wan 2.2 TI2V-5B**: Apache-2.0 (secondary). The official README says 720p (1280×704, 24 fps) needs ≥24 GB, e.g. an RTX 4090. The community reports about 8 GB at reduced resolution. Alibaba reports a 5 s 720p clip in under 9 minutes on a 4090 — [HF Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B); [The Decoder](https://the-decoder.com/alibaba-launches-wan2-2-its-improved-open-source-video-generation-model/); [MindStudio](https://www.mindstudio.ai/blog/what-is-wan-2-2-video-open-source) (search summaries)
- **Hunyuan3D 2.1** (13 Jun 2025): 10 GB VRAM for shape, 21 GB for texture, 29 GB for both. There is a `--low_vram_mode` flag. One user generated meshes with 6 GB but recommends ≥20 GB for texturing. Hunyuan3D 2.0's full pipeline needs only 12 GB in ComfyUI — [Hunyuan3D-2.1 README](https://raw.githubusercontent.com/Tencent-Hunyuan/Hunyuan3D-2.1/main/README.md); [TheLocalLab](https://www.patreon.com/TheLocalLab/posts/free-open-source-133710331); [ComfyUI docs](https://docs.comfy.org/tutorials/3d/hunyuan3D-2) (search summaries)
- The Hunyuan3D 2.1 Community License territory is worldwide **excluding the EU, UK and South Korea**. A secondary source says products with >1M MAU need a separate Tencent license — [Hunyuan3D-2.1 LICENSE mirror](https://gitserver.onethingai.com/ai-models/Hunyuan3D-2.1/raw/branch/main/LICENSE); [EachLabs](https://www.eachlabs.ai/blog/hunyuan3d-things-to-know) (search summaries)
- **TRELLIS.2-4B** (Microsoft): MIT for model and code; some dependencies (nvdiffrast, nvdiffrec) have their own licenses. Needs ≥24 GB VRAM and is tested only on Linux (A100/H100). About 17 s at 1024³ on an H100. A community fork aimed at easier local use exists — [HF TRELLIS.2-4B README](https://huggingface.co/microsoft/TRELLIS.2-4B/blob/main/README.md); [GitHub TRELLIS.2-stableprojectorz](https://github.com/IgorAherne/TRELLIS.2-stableprojectorz) (search summaries)

### Inferences
- **Offline asset pipeline on this PC:** generate concept art and textures with Qwen-Image (Apache) or SDXL-class models in ComfyUI. Make 3D creatures and props with Hunyuan3D 2.0 or 2.1 shape in low-VRAM mode, then texture with 2.0 or in Blender. Rig with Mixamo and import into UE5/Unity/Godot. Short Wan 2.2 5B clips at reduced resolution can serve as intro or transition videos.
- Do not run diffusion or video generation during a live stream on the same 16 GB GPU. Even a 7–12 GB image model plus UE5 will overflow VRAM and cause stutter or encoder overload. Dynamic in-stream visuals should come from pre-generated libraries (e.g. 200 creature variants made offline), selected by gifts or chat.
- Hunyuan3D 2.1 is legally usable by a creator in Turkey, since Turkey is not in the excluded territories. If the business ever serves or incorporates in the EU or UK, re-check.
- FLUX [dev] variants are best avoided for a monetized brand because the commercial status of outputs is ambiguous. Prefer Apache-licensed models.

### Gaps
- No measured Qwen-Image VRAM on 16 GB was found; it is commonly run as FP8 or GGUF, not verified here.
- SDXL licence details (CreativeML OpenRAIL++-M) and FLUX.2 licence terms were not researched this session.
- TRELLIS.2 on Windows and 16 GB (community fork) is unverified.

---

## 6. Blackwell (RTX 50xx, sm_120) compatibility gotchas, 2025–2026

### Takeaway
By late 2026 most problems are solved if you use PyTorch ≥2.7 built with CUDA 12.8 or 13.0. The remaining traps are add-ons that silently reinstall an older torch (xformers), flash-attention / SageAttention wheels, and Triton on Windows. llama.cpp, Ollama and LM Studio ship their own CUDA builds and are the least troublesome.

### Cited Findings
- PyTorch supports all Blackwell architectures since 2.7.0 built with CUDA 12.8. Earlier "sm_120 not supported" errors come from older or mismatched installs — [PyTorch forum](https://discuss.pytorch.org/t/rtx-5050-sm-120-support-timeline-when-in-stable-pytorch/224288); [NVIDIA forum](https://forums.developer.nvidia.com/t/rtx-5090-not-working-with-pytorch-and-stable-diffusion-sm-120-unsupported/338015) (search summaries)
- **xformers:** installing it can force-downgrade torch to a build without sm_120 kernels, with crashes only mid-inference. Other sources say newer xFormers added Blackwell CUTLASS FMHA. The sources conflict — [PyTorch forum: xformers with RTX50](https://discuss.pytorch.org/t/use-xformers-with-rtx50-series/224056) (search summary)
- **flash-attention:** a kernel-launch error was reported on sm_120 (CUDA 12.8, torch 2.8). The README listed only Ampere, Ada and Hopper at that time. Community Windows wheels exist, e.g. torch 2.11 + CUDA 13.0 — [Dao-AILab/flash-attention #1987](https://github.com/Dao-AILab/flash-attention/issues/1987) (search summary)
- **SageAttention:** `--use-sage-attention` reportedly causes black output for Qwen and Wan workflows in ComfyUI. Prebuilt sm_120 wheels exist for torch 2.11 + cu128 — [ComfyUI Blackwell thread #6643](https://github.com/Comfy-Org/ComfyUI/discussions/6643) (search summary)
- **Triton:** "sm_120 is not defined for option 'gpu-name'" errors appeared with nightly torch and Triton. On Windows, triton-windows + torch.compile is the recommended route instead of xformers — [PyTorch forum](https://discuss.pytorch.org/t/rtx-5070-ti-blackwell-pytorch-nightly-triton-still-getting-sm-120-is-not-defined-for-option-gpu-name-error/220460); [ComfyUI-Win-Blackwell](https://github.com/hiroki-abe-58/ComfyUI-Win-Blackwell) (search summaries)
- vLLM works on RTX 5090 with torch 2.9.0 cu128 — [vLLM forum](https://discuss.vllm.ai/t/vllm-on-rtx5090-working-gpu-setup-with-torch-2-9-0-cu128/1492/6) (search summary)
- The llama.cpp CUDA benchmark ran on the 5060 Ti with flash-attention enabled without issue (Aug 2025) — [llama.cpp #15013](https://github.com/ggml-org/llama.cpp/discussions/15013)

### Inferences
- Practical checklist: use the ComfyUI portable build for CUDA 12.8/13 (or a Blackwell-specific installer). Verify that `torch.cuda.get_arch_list()` includes `sm_120`. Don't `pip install xformers` blindly. Pin each Python TTS tool (Chatterbox, Qwen3-TTS) in its own venv with a cu128 torch wheel. Prefer Ollama, LM Studio or llama.cpp for the LLM.
- Python TTS repos that pin old torch versions are the most likely breakage point. Chatterbox originally pinned older torch versions in its requirements (prior knowledge, not re-verified), so install torch cu128 first and then install the package without its dependencies.

### Gaps
- No Blackwell issue reports specific to Chatterbox, Qwen3-TTS or Audio2Face-3D SDK were found.
- The Ollama and LM Studio release that first supported sm_120 was not looked up.

---

## 7. OBS on this machine: NVENC load, vertical TikTok settings, plugins

### Takeaway
The 5060 Ti has **one** 9th-generation NVENC engine, a dedicated ASIC, so encoding barely touches the shaders. A GPU saturated by UE5 plus the LLM can still starve OBS compositing and cause "encoding overloaded" or dropped frames. Use 1080×1920, 30 fps, NVENC H.264 CBR at about 4,000–6,000 kbps with a 2 s keyframe interval. Use obs-websocket (built in) for automation and Advanced Scene Switcher for watchdog-style rules.

### Cited Findings
- NVIDIA's support matrix lists the RTX 5060 Ti as Blackwell with 9th-gen NVENC, **1 NVENC unit** (the 5070 Ti has 2). Blackwell supports 4:2:2 encode in H.264 and HEVC. AV1 is listed as 4:2:0 only — [NVIDIA Video Encode/Decode Support Matrix](https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new); [NVENC Application Note 13.1](https://docs.nvidia.com/video-technologies/video-codec-sdk/13.1/nvenc-application-note/index.html) (search summaries)
- A Dec 2025 driver reportedly raised the GeForce concurrent NVENC session limit to 12 — [vMix blog](https://blog.vmix.com/?p=4331) (search summary)
- TikTok OBS settings (community guidance, not official TikTok specs): canvas and output both 1080×1920, or TikTok may add black bars. NVENC H.264, CBR, keyframe 2 s; "0/Auto" can cause audio desync. Bitrate 4,000–6,000 kbps for 1080×1920 (other guides say 2,500–4,000 or 6,000). 30 fps — [streamer.guide cheat-sheet (Jun 2026)](https://streamer.guide/guides/bitrate-encoder-settings-cheat-sheet); [Evetech TikTok OBS 2026](https://evezone.evetech.co.za/game-desk/best-obs-settings-for-tiktok-live-streaming-in-sa-2026); [MakeUseOf](https://www.makeuseof.com/how-to-make-tiktoks-shorts-reels-in-obs-studio/); [obs-versions TikTok guide](https://obs-versions.com/blog/obs-tiktok-live-streaming-guide) (search summaries)
- obs-websocket has been bundled with OBS Studio since 28.0.0 (5.x server, default port 4455) — [GitHub obsproject/obs-websocket](https://github.com/obsproject/obs-websocket)
- Aitum Vertical (free, Win/Mac/Linux, needs OBS 29–30+) adds a separate vertical canvas with linked scenes. Some users report crashes on close, a streaming bug after 1.4.4, and in-app promo banners — [OBS forum: Aitum Vertical](https://obsproject.com/forum/resources/aitum-vertical.1715/) (search summary)

### Inferences
- **Recommended OBS profile:** Base = Output = 1080×1920; 30 fps; NVENC H.264 (TikTok RTMP ingest generally expects H.264; HEVC and AV1 are not assumed). Rate control CBR, 4,500–6,000 kbps, keyframe 2 s, preset P5–P6 "Quality", tuning High Quality, multipass Quarter-res, profile High, B-frames 2, Look-ahead off. Look-ahead and psycho-visual tuning use CUDA cores and some VRAM, so turn them off when the GPU is busy.
- **Aitum Vertical is unnecessary:** this stream is vertical-only, so set the main canvas to 1080×1920 and skip the extra canvas and its instability.
- **Capturing the game:** use Game Capture, or Spout2 / NDI from UE5/Unity/Godot into OBS. Run the game windowed at exactly 1080×1920 and cap its FPS at 30–60. Set OBS to "High" process priority. On Windows 11, enabling Hardware-Accelerated GPU Scheduling usually helps NVENC under load (common guidance, not verified this session).
- **Useful plugins and tools:** obs-websocket (built in) to let a Python/Node "director" switch scenes and show gift and LLM overlays. Advanced Scene Switcher for rules such as "if the game window disappears, switch to the BRB scene" and for timed restarts. Move (transition animations). Browser Source for TikTok gift and chat overlays (e.g. TikFinity / TikTok-Live-Connector-based overlays; another researcher's scope). Background Removal is not needed.
- **VRAM:** NVENC plus OBS compositing at 1080×1920 probably uses about 0.3–0.8 GB (estimate).

### Gaps
- No measured data on NVENC or OBS overhead on the 5060 Ti under combined UE5 and LLM load was found.
- Official TikTok LIVE ingest specs (codec, maximum bitrate) were not found in these results.
- Advanced Scene Switcher and Move plugin details were not verified this session.

---

## 8. Concurrent VRAM budget, offloading strategy, 24/7 reliability and electricity cost (Turkey)

### Takeaway
All four pieces (UE5 + LLM + TTS + NVENC) fit in 16 GB only with discipline: a lean game (≤5–6 GB), a ~4B Q4 LLM (≈3.5–5 GB), one GPU TTS (≈2–3 GB) or CPU TTS, and about 1–1.5 GB headroom. Pre-generate as much content offline as possible. Running 12 h/day costs roughly 400–500 TL/month in electricity at 2026 Turkish household tariffs (my estimate). 24/7 operation risks pushing the household over the 4,000 kWh/yr subsidized-tariff threshold.

### Cited Findings
- Model VRAM anchors: Qwen3.5-4B about 3 GB and 9B about 6 GB at 4-bit — [Artificial Analysis](https://artificialanalysis.ai/articles/qwen3-5-small-models). Gemma 4 E4B 5.5–6 GB and 12B 7–8 GB at 4-bit — [Unsloth](https://unsloth.ai/docs/models/gemma-4). Gemma 3 12B Q4 about 11.1 GB at 8K context — [willitrunai](https://willitrunai.com/can-run/gemma-3-12b-on-rtx-5060-ti-16gb) (search summaries)
- Chatterbox Nano (English) runs at about 3× realtime on 8 CPU cores, so CPU TTS is viable — [GitHub resemble-ai/chatterbox](https://github.com/resemble-ai/chatterbox)
- **Turkey electricity:** the residential price was TRY 2.80/kWh (about USD 0.063) up to Mar 2026, per EPDK/TEDAŞ data — [GlobalPetrolPrices](https://www.globalpetrolprices.com/Turkey/electricity_prices/). Residential electricity rose 25% effective 4 Apr 2026 — [GMK Center](https://gmk.center/en/news/electricity-and-gas-rates-have-increased-in-turkey-since-april-4/); [The Star](https://www.thestar.com.my/news/world/2026/04/04/trkiye-announces-double-digit-price-hikes-for-electricity-natural-gas). A post-revision example: 100 kWh/month costs a residential customer 323.8 TL — [Qatar Tribune](https://www.qatar-tribune.com/article/262041/business/turkiye-revises-household-power-tariffs); [Daily Sabah](https://www.dailysabah.com/business/energy/how-much-will-electricity-gas-price-hike-impact-turkiyes-inflation) (search summaries; exact attribution of the 323.8 TL figure between these two articles is uncertain)
- The subsidized household tariff threshold is 4,000 kWh/yr in 2026 and falls to 3,000 kWh/yr in 2027. Households above it pay the higher tariff — [Daily Sabah](https://www.dailysabah.com/business/economy/turkiye-lowers-electricity-subsidy-limit-affecting-65m-households) (search summary)

### Inferences
**Sample concurrent VRAM budgets (all estimates, to validate with nvidia-smi / HWiNFO):**

| Component | Config A "Full local, Turkish voice" | Config B "Lean, recommended" | Config C "Bigger brain" |
|---|---|---|---|
| Windows desktop + DWM + browser sources | 0.8 GB | 0.8 GB | 0.8 GB |
| OBS compositing + NVENC (1080×1920@30) | 0.5 GB | 0.5 GB | 0.5 GB |
| Game engine | UE5 stylized, baked/low Lumen: 5.5 GB | Godot/Unity or lean UE5: 3–4 GB | Godot 2D: 1–1.5 GB |
| LLM (llama.cpp/Ollama, Q4_K_M, 4–8K ctx) | Qwen3.5-4B / Gemma 4 E4B: 3.5–5.5 GB | Qwen3.5-4B: 3.5–4 GB | Gemma 4 12B / Qwen3.5-9B: 7–9 GB |
| TTS | Chatterbox Multilingual (TR) on GPU: 2–3 GB | Kokoro/Piper EN on CPU (0 GB) or Chatterbox TR on GPU 2–3 GB | Chatterbox TR on GPU: 2–3 GB |
| Lip-sync | Audio2Face-3D: ~1–1.5 GB (unknown) | Amplitude/viseme (0 GB) | none |
| **Total** | **~14–16.8 GB (too tight)** | **~8–11 GB (safe)** | **~12–15 GB** |

- **What to cut first if VRAM runs out:** (1) Audio2Face, replaced with amplitude lip-flap; (2) move English TTS to the CPU; (3) drop the LLM to 4B or 2B, or run it on the CPU for pre-generation only; (4) lower UE5 texture pool, shadows and Lumen. On Windows, VRAM overflow does not crash immediately. The NVIDIA driver spills to system RAM ("sysmem fallback") and everything slows sharply, so keep about 1 GB free. In NVIDIA Control Panel, set "CUDA – Sysmem Fallback Policy: Prefer No Sysmem Fallback" for llama.cpp so it fails fast instead of crawling (from prior knowledge, not re-verified).
- **Compute contention matters as much as VRAM.** LLM decoding bursts and TTS synthesis compete with UE5 for SMs and can cause frame-time spikes. Mitigations: cap game FPS at 30–60; queue LLM and TTS jobs one at a time; use short outputs (max_tokens 60–120); pre-generate quiz banks, story branches, gift reactions and TTS clips offline (for example, overnight while not streaming). That keeps the live LLM for occasional chat replies only. The CPU (6 cores, 32 GB RAM) can host a second small LLM, e.g. Qwen3.5-2B or 4B on CPU, for moderation or classification of chat messages.
- **RAM budget (32 GB):** Windows 3–4 GB, UE5 editor-less packaged game 3–6 GB, OBS 1 GB, Ollama CPU-side buffers 1–2 GB, Python TTS 2–3 GB, browser overlays 1 GB, leaving about 12–18 GB free. A CPU LLM (4B Q4 ≈ 3 GB) fits easily. Use a packaged build, not the UE editor, during streams.
- **24/7 reliability:**
  - Run Ollama or llama-server, the TTS server and the "director" bot as Windows services (NSSM or Task Scheduler "restart on failure") with HTTP health checks every 30–60 s.
  - Use a watchdog script (Python) that pings each service, checks OBS via obs-websocket (`GetStreamStatus`, dropped frames, `outputCongestion`) and restarts the game executable if its window or heartbeat disappears. Use an Advanced Scene Switcher BRB fallback scene.
  - Schedule a planned restart between streams to clear VRAM fragmentation and memory leaks (long-running UE5 and Python processes often leak).
  - Pause Windows Update during stream hours, use a high-performance power plan, and disable sleep and USB suspend.
  - Thermals: log GPU and CPU temperatures with HWiNFO. A mild undervolt or 90% power limit on the 5060 Ti typically costs little performance and lowers heat over 12 h (common practice, not benchmarked here). Clean dust filters, and use a UPS if power is unstable.
  - Keep a second stream key or TikTok LIVE Studio fallback profile ready, and an auto-reconnect setting in OBS (Settings → Advanced → Automatically Reconnect).
- **Electricity estimate:** system draw under this mixed load is roughly 250–350 W at the wall (RTX 5060 Ti TGP about 180 W from prior knowledge, plus a 65 W CPU, plus the rest).
  - 12 h/day uses about 3–4.2 kWh/day, about 90–126 kWh/month.
  - At about TL 3.2–3.5/kWh after Apr 2026 (derived from TRY 2.80 + 25%, and from the 323.8 TL per 100 kWh example), that is **≈ TL 300–450/month**.
  - 24/7 uses about 180–250 kWh/month, **≈ TL 600–900/month**.
  - 24/7 alone adds about 2,200–3,000 kWh/yr. Together with normal household use, that can exceed the 4,000 kWh/yr subsidy threshold (3,000 in 2027) and move the whole household onto the higher, unsubsidized tariff. Check your own bill and tariff tier.

### Gaps
- Measured wall power for this exact PC under combined UE5, LLM and NVENC load is not available; it needs a plug-in power meter.
- The exact post-April-2026 per-kWh residential tariff (low and high tiers, with taxes) was not found on an official EPDK page. Estimates are derived from the pre-hike price plus 25% and a news example.
- Audio2Face-3D and Chatterbox VRAM figures are unverified estimates, so the Config A total is uncertain.
