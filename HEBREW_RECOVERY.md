# Meetily Hebrew Recovery

This branch was created from the exact upstream Meetily v0.4.0 release commit:

`0281737d87d26352fb0adc78c8c0975f691b23d1`

It is a reconstruction of the Hebrew-specific changes observed in the supplied compiled macOS application (`Meetily.app`, version 0.4.0). The original Rust/TypeScript source for those custom changes is not embedded verbatim in the `.app`, so recovered code must distinguish binary-proven behavior from source-level reconstruction.

## Binary-proven Hebrew behavior

The supplied executable contains the following identifiers and messages:

- `set_hebrew_transcription_preference`
- `HEBREW_TRANSCRIPTION_PROVIDER`
- `HEBREW_TRANSCRIPTION_MODEL`
- `effective_whisper_language`
- `is_hebrew_language_preference`
- `default_hebrew_model_for`
- `normalize_hebrew_provider`
- `get_hebrew_transcription_choice`
- `resolve_hebrew_transcription_request`
- `config::whisper_model_url`
- `config::whisper_model_filename`
- `config::is_hebrew_language`
- model ID: `ivrit-ai-whisper-large-v3-turbo`
- model filename: `ggml-ivrit-ai-whisper-large-v3-turbo.bin`
- model description: `ivrit.ai Hebrew fine-tune of whisper-large-v3-turbo`
- model URL: `https://huggingface.co/ivrit-ai/whisper-large-v3-turbo-ggml/resolve/main/ggml-model.bin`
- Whisper language is explicitly set to `he` for Hebrew sessions.
- A Hebrew session can override the general transcript configuration and load the selected Hebrew Whisper model without replacing the user's general/default model choice.
- Recording validation refuses to silently fall back to a non-Hebrew Whisper model when the required Hebrew model is missing.
- The binary contains separate Hebrew provider/model preference state.
- The binary contains logic capable of resolving a Hebrew request to either a Hebrew Whisper choice or Parakeet; a Whisper command emits `Hebrew speech is set to Parakeet. Use the Parakeet engine, not Whisper.` when appropriate.

## Recovered in source

`frontend/src-tauri/src/config.rs` now restores:

- the ivrit.ai Whisper model catalog entry;
- its exact model ID and filename;
- its exact Hugging Face download URL;
- `whisper_model_url` and `whisper_model_filename` helpers corresponding to symbols present in the binary;
- an `is_hebrew_language` helper;
- named constants for the Hebrew model.

## Remaining reconstruction work

The binary proves that the customized build also modified the runtime routing layer. The most likely source locations, based on embedded Rust paths and existing v0.4.0 architecture, are:

- `frontend/src-tauri/src/lib.rs`
- `frontend/src-tauri/src/whisper_engine/commands.rs`
- `frontend/src-tauri/src/whisper_engine/whisper_engine.rs`
- `frontend/src-tauri/src/audio/transcription/engine.rs`
- `frontend/src-tauri/src/audio/recording_commands.rs`
- possibly the transcript/settings UI that invokes `set_language_preference` and `set_hebrew_transcription_preference`.

Those runtime changes should implement the binary-proven behavior above and use the centralized model helpers restored in `config.rs`.

## Accuracy note

This branch intentionally does **not** claim byte-for-byte recovery of the custom source. The upstream v0.4.0 source is exact; Hebrew-specific behavior is reconstructed from executable strings, Rust symbols, embedded source paths, and observed model metadata. This document should remain with the branch so future changes can distinguish verified behavior from inferred implementation details.
