# SketchByte — Qwen3 TTS Voice System

SketchByte turns a written narration into a natural-sounding voiceover track with a **cloned voice you supply once**, using Qwen3 models on Google Colab.

Two models do the work:

1. **Qwen3-4B-Instruct-2507** rewrites a raw script into a TTS-ready script (numbers as words, punctuation tuned for natural pacing, no conversational framing).
2. **Qwen3-TTS-12Hz-1.7B-Base** clones your permanent reference voice and speaks the formatted script, chunk by chunk.

The result is a single 24 kHz WAV (`SketchByte_Voiceover.wav`) with live per-chunk audio previews inside the notebook.

---

## How it works

The pipeline is split into **4 notebooks that share one Colab runtime**. Run them **in order, in the same runtime session**:

| Step | Notebook | What it does |
| ---- | -------- | ------------- |
| 1 | `installer.ipynb` | Installs packages, verifies a GPU (T4), defines the config (models, reference voice, output). |
| 2 | `loading resource.ipynb` | Mounts Google Drive, locates and validates the reference voice, loads the Qwen3-4B formatter. |
| 3 | `script formatting.ipynb` | Enter your script — Qwen3-4B formats it — saves the TTS-ready text. |
| 4 | `voiceover.ipynb` | Unloads the formatter, loads Qwen3-TTS, clones the voice, writes the final WAV. |

State is passed between notebooks through **kernel globals** plus **files written to `/content/`**:

- `/content/SketchByte_TTS_Ready_Script.txt` — handoff from step 3 to step 4 (read as a fallback if the runtime was restarted).
- `/content/SketchByte_Voiceover.wav` — the final narrated track (auto-downloaded).

> **Why the model swap in step 4?** On a Colab T4 (16 GB) the Qwen3-4B formatter (~8 GB bf16) and the Qwen3-TTS model cannot live in VRAM at the same time. Step 4 deliberately unloads the formatter before loading the TTS model. If you want to re-run step 3 after step 4, re-run step 2 first.

---

## Setup

1. **Runtime**: open the notebooks in [Google Colab](https://colab.research.google.com/) → Runtime → Change runtime type → **T4 GPU**.

2. **Reference voice** — in your Google Drive, create this layout, matching `REFERENCE_AUDIO` in `installer.ipynb` (default `fishengagement.mp3`):

   ```
   My Drive/SketchByte/Voice/fishengagement.mp3
   ```

   Use a clean, isolated clip of the voice you want cloned (3+ seconds recommended). Step 2 **auto-converts** the file to a canonical 24 kHz mono WAV (`<name>_ref_24k.wav`) if it isn't already a WAV, so MP3/OGG references are handled safely.

3. **Transcript** — set `REFERENCE_TEXT` in `installer.ipynb` to exactly what is said in the reference audio. Without it, voice cloning degrades to speaker-embedding-only.

4. **Run** the notebooks in order: 1 → 2 → 3 → 4.

---

## Usage

- Enter the narration you want to voice in the **form field at the top of `script formatting.ipynb`** (`SCRIPT_INPUT_FORM`).
- Tune chunk size / sampling in `voiceover.ipynb` (`split_script(..., max_words=...)`, `temperature`, `top_k`).
- Check the **live audio players** under the progress bar in step 4 — each chunk plays as it finishes.

---

## Models

| Model | Role | License |
| ----- | ---- | ------- |
| [Qwen/Qwen3-4B-Instruct-2507](https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507) | Script formatter | Apache-2.0 |
| [Qwen/Qwen3-TTS-12Hz-1.7B-Base](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-Base) | Voice cloning TTS | Apache-2.0 |

Reference implementations: [Qwen3](https://github.com/QwenLM/Qwen3) · [Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS).

---

## Running outside Colab

The notebooks depend on Colab services (`google.colab` drive/files/output) and a CUDA GPU. For a local environment, install the same set of packages from `requirements.txt`; you'll need to swap the Drive-based reference path and the Colab widget helpers yourself.

## License

[MIT](LICENSE)