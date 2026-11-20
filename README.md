# GMI Collage Narrator

Narrator-driven story, history, and unique-facts videos in cut-out paper collage style — made entirely on GMI Cloud.

The narrator carries the story; the paper cut-outs carry the picture. The paper illustrations are public-domain / CC0 photos collected from the internet — real archival photos and scans, not AI-generated pictures. Motion graphics (Remotion) form the cheap, deterministic paper backbone; MiniMax H3 clips mix in only for moments that need real motion or native audio.

## The all-GMI rule

Every generative step runs on a GMI Cloud model. No other provider's AI anywhere in the pipeline.

| Step | GMI model | Price |
| --- | --- | --- |
| Script | MiniMax M3.1 Flash (`MiniMaxAI/MiniMax-M3.1-Flash`) | $0.15 / $0.60 per 1M tokens in/out |
| Narration | Speech 2.6 Turbo (`minimax-tts-speech-2.6-turbo`) | $0.06 / 1,000 chars |
| Key-moment clips | MiniMax H3 (`MiniMax-H3`) | $0.08/s 768P, $0.13/s 2K |
| Music bed | Music 3.0 (`minimax-music-3.0`) | $0.15 / song (up to 5 min) |
| Generated image, if ever needed | hy-image-v3.5-preview | $0.024 / image |

> Note on TTS versions: the series uses Speech 2.6 Turbo (tested, series voice `Indonesian_CalmWoman`) — but point new work at Speech 2.8 Turbo (`minimax-tts-speech-2.8-turbo`): it is the newer generation, same $0.06 / 1,000 chars, same payload fields, same voices. Generate the same short line on both and compare before switching a series over to 2.8.

Local tools (Remotion renderer, ffmpeg, Whisper transcription check) only assemble and verify — they do not invent words, voices, pictures, or music. GMI has no speech-to-text model, so transcription stays on local Whisper as QA tooling.

## Episode structure

Title card (3s, newspaper masthead + voiced title) → beats (one cutout scene per narration line) → dark closing card → end card (3s, white GMI logo on pure black).

## Proof: episode 1

"Fakta Unik Ditemukannya Keyboard" — 11 beats, 2:16, portrait 720x1280 + landscape 1280x720. Total model spend ~$0.52 (script ~$0.001, narration $0.12, one 5s H3 clip $0.40, title voice $0.002). Eleven versions (v1–v11) iterated from user feedback; current reference is portrait v11 / landscape v6.

## Use

Read [`SKILL.md`](SKILL.md) — it is the full playbook: the design system, the pipeline with exact commands, the quote-confirm-create-once-poll rule, the narration verification procedure, and the iteration/versioning workflow.

Setup: a GMI Cloud account with the chat API and the request queue API, Remotion 4 for the render backbone, and local ffmpeg + faster-whisper for the share copy and the transcription check. Paths in the skill are the author's working layout — point the render project, episode folder, GMI helpers, and Whisper venv at your own layout.

## License

MIT — see [LICENSE](LICENSE).
