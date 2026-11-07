# OpenPronounce — Technical Review

Upstream: https://github.com/Halleck45/OpenPronounce  
Package: https://pypi.org/project/openpronounce/  
License: MIT (verify the exact upstream LICENSE at the version pinned by a product).  
Role: local/self-hosted phoneme-level pronunciation assessment.

## 1. What it does

Input: learner audio, expected sentence and language code.

Output includes:
- Overall score from 0 to 100.
- Recognized transcript.
- Expected and heard phoneme sequences.
- Word-level errors with expected vs. heard sounds and confidence.
- Phoneme error rate (PER) and word error rate (WER).
- Acoustic distance based on aligned speech embeddings.
- Pitch (F0) and energy contours for prosody visualization.

This is a useful building block for an ELSA-inspired practice flow, but it is not equivalent to a fully human-calibrated commercial assessment product.

## 2. Processing pipeline

1. Audio is decoded and normalized/resampled to 16 kHz mono.
2. eSpeak-NG converts the target sentence into expected phonemes and word mapping.
3. A Wav2Vec2 phone-recognition model estimates the sounds actually spoken. This is intended to avoid a conventional language model silently correcting the learner's pronunciation into the expected sentence.
4. A word-level ASR model produces the transcript and WER signal.
5. Expected and heard phone sequences are aligned with edit-distance logic; substitutions and deletions receive confidence estimates.
6. A reference utterance is synthesized (gTTS by default; Piper or Kokoro can be configured) and encoded for acoustic comparison.
7. DTW compares speech embeddings; pYIN and RMS provide pitch and energy curves.
8. Component scores are combined into one heuristic score.

Pin a release/commit and record it in deployment metadata because weights, thresholds and language normalization can change.

## 3. API surface

| Endpoint | Input | Purpose |
|---|---|---|
| POST /pronunciation | file, expected_text, lang | Full pronunciation analysis |
| POST /speech2text | file, lang | Transcript |
| POST /phonemes | text, lang | Expected phonemes and word mapping |
| POST /tts | text, lang | Reference pronunciation audio |
| GET /languages | — | Language registry |
| GET /health | — | Health check |
| GET /docs | — | Swagger/OpenAPI documentation |

Recommended integration:
- Frontend records audio and sends it to the application backend.
- Backend validates file type/size, consent and rate limits.
- Backend calls OpenPronounce locally or through a private service boundary.
- Backend maps the raw response to an application-owned DTO.
- Frontend displays score, transcript, per-word feedback and prosody.
- Store only the minimum audio/result data needed; define retention and deletion.

Do not expose an unauthenticated scoring service directly to the public internet.

## 4. Runtime and deployment

Upstream installation notes specify Python 3.10+, ffmpeg, eSpeak-NG, PyTorch and the openpronounce package. First use downloads two Wav2Vec2 checkpoints (the upstream README estimates about 1.2 GB each). CPU is supported; CUDA can be selected when available. Piper/Kokoro extras can provide offline reference TTS.

Operational implications:
- Provision model downloads and disk cache explicitly.
- Cold starts can be slow; warm model instances and cache weights.
- CPU-only inference is possible, but benchmark realistic concurrency.
- Use a job queue for long recordings or batch workloads.
- Monitor memory, CPU/GPU utilization, latency, audio decode failures and model-load errors.
- Pin Python/package/model versions and record upstream commit hashes.

## 5. Accuracy and limitations

The upstream project states:
- English is calibrated against human ratings.
- French, Spanish, German, Italian, Portuguese and Dutch are listed as experimental.
- Other languages reuse English phone thresholds/score weights (with some language-specific acoustic baseline handling); do not assume equivalent validity.
- Strong accents, children, noisy recordings and short words can degrade recognition.
- The phone recognizer itself can make errors; false positives are expected.
- The score is a heuristic proxy, not a definitive grade or proof of proficiency.

The upstream benchmark README reports Spearman correlation around 0.65 with human total ratings in one 500-utterance evaluation for the documented scoring setup. This is useful evidence, not a universal guarantee. Reproduce the benchmark on the intended learner population, accent, microphone and environment.

Product recommendation: show uncertainty and actionable sound-level feedback. Avoid presenting a single number as objective truth. Let users replay both original and reference audio.

## 6. Suggested application data contract

The application should normalize the upstream response into its own stable schema, for example:
- assessmentId
- locale (such as en-US or en-GB)
- score
- transcript
- wordFeedback[] containing word, expectedIpa, heardIpa, confidence and status
- prosody.pitch[] and prosody.energy[]
- model.name and model.version

This is a proposed application contract, not the exact upstream response schema.

## 7. Validation checklist

- [ ] Pin package version and upstream commit.
- [ ] Confirm MIT license and retain copyright/license notice.
- [ ] Test native speech, learner accents, quiet/noisy recordings and varied microphones.
- [ ] Build a human-rated validation set for the target learners.
- [ ] Measure false-positive and false-negative rates per phoneme.
- [ ] Check US/UK pronunciation dictionaries and phoneme mapping separately.
- [ ] Test short words, connected speech, contractions and sentence boundaries.
- [ ] Set upload limits, authentication, retention and deletion controls.
- [ ] Benchmark latency, RAM/VRAM, CPU/GPU cost and concurrency.
- [ ] Label automated feedback as approximate, not an official proficiency certificate.

## Sources

- https://github.com/Halleck45/OpenPronounce
- https://github.com/Halleck45/OpenPronounce/blob/main/server.py
- https://github.com/Halleck45/OpenPronounce/blob/main/benchmarks/README.md
- https://pypi.org/project/openpronounce/