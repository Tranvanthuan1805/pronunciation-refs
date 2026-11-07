# Kokoro TTS — Technical Review

Inference library: https://github.com/hexgrad/kokoro  
Model: https://huggingface.co/hexgrad/Kokoro-82M  
Voice catalogue: https://huggingface.co/hexgrad/Kokoro-82M/blob/main/VOICES.md  
Model card license: Apache-2.0.

## 1. What Kokoro is

Kokoro is an open-weight text-to-speech model with 82 million parameters. The official model card describes it as a lightweight model intended to provide high-quality speech with comparatively modest compute requirements. The inference package exposes a Python pipeline and uses Misaki for grapheme-to-phoneme processing.

Kokoro is a TTS system: text goes in and synthesized audio comes out. It does not evaluate learner speech. Pair it with an assessment engine such as OpenPronounce for a complete learning loop.

## 2. Language and voice selection

The official catalogue includes American English and British English voices.

Examples in upstream usage:
- American English uses language code a; an example voice is af_heart.
- British English uses language code b; select a matching British voice from the official VOICES.md catalogue.

Voice IDs and language codes must be a compatible pair. Do not guess voice IDs or mix a British voice with an American pipeline code. The upstream catalogue includes quality/training-duration notes; audition voices with the target audience before choosing defaults.

The catalogue evolves. Check the model card and VOICES.md at the exact pinned revision before documenting a definitive inventory.

## 3. Basic integration pattern

The official Python README demonstrates this general pattern:

    from kokoro import KPipeline
    import soundfile as sf

    pipeline = KPipeline(lang_code="a")  # American English
    generator = pipeline(
        "Hello, welcome to your pronunciation practice.",
        voice="af_heart",
        speed=1,
    )

    for index, (graphemes, phonemes, audio) in enumerate(generator):
        sf.write(f"reference-{index}.wav", audio, 24000)

This is an illustrative upstream-style snippet. Pin a tested package version and follow the official installation instructions, including eSpeak-NG where required.

## 4. Where it fits in the app

- Play a reference sentence before recording.
- Replay a target word or phrase.
- Generate audio for minimal-pair and phoneme lessons.
- Provide a reference track for shadowing.
- Pre-generate frequently used lesson audio and cache it.
- Offer a US/UK voice selector while keeping the target accent explicit.

Suggested flow: choose accent → play reference → show text and optional IPA → record learner → assess with OpenPronounce. Do not treat Kokoro output as ground-truth human speech.

## 5. Performance and operations

- Audio generation returns chunks; write or stream chunks appropriately.
- Official example writes audio at 24 kHz.
- Cache by normalized text + language code + voice ID + model version + speed.
- Avoid generating the same lesson audio on every page load.
- Pre-warm the pipeline and benchmark cold-start vs. warm inference.
- Test names, numbers, abbreviations, punctuation, contractions and long paragraphs.
- Include replay and speed controls; do not alter the canonical stored reference when the user changes playback speed.

## 6. Licensing and redistribution

The official hexgrad/Kokoro-82M model card declares Apache-2.0. The inference package metadata also identifies Apache Software License. Preserve upstream license and notices when redistributing code, weights or voice assets.

Important distinction:
- Official Kokoro model and official voice packs have upstream provenance.
- Community-created or fine-tuned voice packs are separate assets. Check their license, speaker consent, training-data provenance and commercial permissions independently.
- Apache-2.0 on the base model does not automatically license every third-party voice pack.

Do not place large model weights or voice binaries in this documentation repository by default. Link to authorized sources and record exact revision, file hash and license when approved.

## 7. Vietnamese voice pack — separate verification

The community post mentions “Kokoro Vietnamese (Storyvert)” and a Google Drive link, but the provided URL is truncated (https://drive.google.com/...). The file, model format, source author and terms cannot be inspected.

The official Kokoro catalogue reviewed for this research does not establish that this is an official Vietnamese voice. Treat it as an unverified community asset until the exact share URL, file, source author and license are supplied.

## Sources

- https://github.com/hexgrad/kokoro
- https://github.com/hexgrad/kokoro/blob/main/README.md
- https://huggingface.co/hexgrad/Kokoro-82M
- https://huggingface.co/hexgrad/Kokoro-82M/blob/main/VOICES.md
- https://github.com/hexgrad/misaki