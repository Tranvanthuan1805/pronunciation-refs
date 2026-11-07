# Pronunciation AI — Reference Library

Curated technical notes for future pronunciation-learning and EdTech products. This repository records upstream projects, capabilities, integration notes, licensing checks and limitations.

## Technology map

| Component | Role | Upstream | License / verification |
|---|---|---|---|
| OpenPronounce | Phoneme-level pronunciation assessment, IPA differences, transcript and prosody | https://github.com/Halleck45/OpenPronounce | MIT; English calibrated, other listed languages experimental |
| Kokoro | Text-to-speech reference voice for shadowing | https://github.com/hexgrad/kokoro and https://huggingface.co/hexgrad/Kokoro-82M | Official model card declares Apache-2.0 |
| Kokoro Vietnamese / Storyvert | Candidate Vietnamese voice pack | Community Google Drive link was truncated | Unverified; do not redistribute or use commercially yet |

## Documents

- docs/openpronounce.md — architecture, API, deployment, accuracy and validation.
- docs/kokoro.md — TTS usage, US/UK voices, operations and licensing.
- docs/vietnamese-voice.md — verification record for the community Vietnamese asset.

## Key findings

### OpenPronounce
- Accepts a recording and expected sentence.
- Returns a 0–100 heuristic score, transcript, expected/heard phonemes, word-level errors and confidence, phoneme/word error rates, acoustic distance and prosody traces.
- Exposes Python functions and a FastAPI service: pronunciation assessment, speech-to-text, phoneme conversion, TTS, language registry and health check.
- Uses Wav2Vec2, eSpeak-NG, sequence alignment/edit distance, DTW acoustic comparison, pitch and RMS energy.
- Upstream warns that English is calibrated against human ratings; other supported languages are experimental and feedback can be wrong.

### Kokoro
- Open-weight 82M-parameter TTS model.
- Official model card declares Apache-2.0.
- Official catalogue includes American and British English voices.
- Check the upstream model card and voice catalogue at the pinned revision before integrating.

## Repository principles

1. Upstream is the source of truth. Link to original code and model cards; do not copy upstream source or weights unless necessary and license-reviewed.
2. Separate verified facts from hypotheses and untested community claims.
3. Preserve attribution and original license files.
4. Do not commit large model binaries by default. Use authorized upstream hosting or approved artifact storage.
5. Re-check versions, language support, license and API behavior before production.

Companion application blueprint: https://github.com/Tranvanthuan1805/pronunciation-app

Initial research was based on upstream OpenPronounce documentation, the official Kokoro GitHub repository and the hexgrad/Kokoro-82M Hugging Face model card.