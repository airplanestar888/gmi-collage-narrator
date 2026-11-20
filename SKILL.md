# GMI Collage Narrator

- Use this skill whenever the user wants a narrator-driven story, history, or unique-facts video in cut-out paper collage style, made on GMI Cloud with MiniMax models. The narrator carries the story; the paper cut-outs carry the picture. The paper illustrations are collected from public-domain and CC0 images on the internet — real archival photos and scans, not AI-generated pictures. (Episode 1 v1 used generated illustrations; the user decided 2026-11-20 that generated images are cheating for GMI ambassador work, so the skill collects instead.)

## The all-GMI rule

- Every generative step in this format runs on a GMI Cloud model — no other provider's AI anywhere in the pipeline. Script: `MiniMaxAI/MiniMax-M3.1-Flash`. Voice: `minimax-tts-speech-2.6-turbo`. Key-moment clips: `MiniMax-H3` (draft on H3-Lite / H3-Highspeed). Music bed: `minimax-music-3.0`. If an episode ever needs a generated image, it must be a GMI image model (`hy-image-v3.5-preview`, $0.024/image) — never another provider's image generator. The format itself does not generate images at all: collect public-domain/CC0 photos first, fall back to a typographic paper card (see "Collecting assets").

- What is NOT a GMI step, and stays local on purpose: the renderer (Remotion), the transcoder/cutter (ffmpeg), the transcription check (local faster-whisper — GMI has no speech-to-text model, so Whisper is QA tooling, not a creative step), the image locator (image-search CLI), and the official GMI brand logo (Brandfetch, see "The GMI logo end card"). These tools move bytes around; they do not invent words, voices, pictures, or music. That is the line: GMI creates, local tools assemble and verify.

- Newer models appear in the catalog over time (Speech 2.8 Turbo/HD arrived after episode 1). Do not switch the series voice or any pinned model silently — quote the new option to the user first. The pinned stack is the tested one; a newer model is a proposal, not an upgrade.

## The format

- Cut-out animation (animasi potongan kertas), also called collage animation: flat paper pieces — newspaper clippings, old photos, headlines — arranged, moved a little, and photographed frame by frame. Today it is rendered digitally with the same jerky, pop-in look. The reference is Terry Gilliam's Monty Python animations.

- The motion is deliberately not smooth. Scenes change per beat, images pop in, collide, and get replaced. That abruptness is the style, not a flaw.

- The narrator explains the story, history, or unique facts over the picture. Narration first, visuals second. A beat that the narration cannot carry is a beat to rewrite, not a beat to throw more video at.

- Each video is a list of beats. One beat is one short narration line plus one visual note (`[Visual: ...]`) describing the paper collage that shows while the line is read.



## Episode structure

- Every episode has the same four parts, in order. The frame math lives in `src/beats.ts`: `TOTAL_FRAMES = TITLE_FRAMES + sum(beat frames) + ENDCARD_FRAMES`.

- **Title card (3 s, 90 frames).** A newspaper-masthead screen before beat 1: dateline, masthead in blackletter, double rule, headline. The narrator reads the title over it (a short Speech 2.6 Turbo job in the same series voice — episode 1's title voice was 33 characters, $0.002). Narration and subtitles start after the card (`<Sequence from={TITLE_FRAMES}>`).

- **Beats.** One cutout scene per beat, in script order. Beat lengths come from the true narration timing (Whisper word timestamps on the repaired narration), never from character estimates.

- **Closing card.** The last beat is a dark card (`#141414`, Georgia serif, paper-colored text) carrying the episode's closing line. No image, no subtitles — the card already carries those words.

- **End card (3 s, 90 frames).** The official white GMI logo, centered on pure black, fading in and out. Silent — the narration ends exactly when the end card starts. See "The GMI logo end card".

- Episode 1 totals: 90 title + 3,890 beat + 90 end card = 4,070 frames, 135.68 s at 30 fps.

## The stack

- Write the script with `MiniMaxAI/MiniMax-M3.1-Flash` on the chat API. It costs $0.15 per 1M input tokens and $0.60 per 1M output tokens on GMI, with a 1M token context.

- Narrate with `minimax-tts-speech-2.6-turbo` on the request queue API. It is low latency multilingual text to speech with emotion control and 40 plus languages, including Indonesian. It costs $0.06 per 1,000 characters on GMI, so a 1,000 character narration is $0.06. The series voice is `Indonesian_CalmWoman` with `language_boost: Indonesian` — reuse it unless the user asks for a different voice. (Speech 2.8 Turbo/HD exists in the catalog now; it is not the series voice until the user approves a switch.)

- Generate key-moment clips with `MiniMax-H3` on the request queue API. It makes 4 to 15 second clips at 768P or 2K with native stereo audio, including dialogue, ambience, and music in the same pass. It costs $0.08 per second at 768P and $0.13 per second at 2K, so a 5 second 768P clip is $0.40.

- Collect the paper illustrations from public-domain and CC0 images on the internet (see "Collecting assets"). Real archival photos and scans fit the vintage paper look better than generated pictures, and they cost nothing. Generated images do not count as GMI work — and if generation is ever needed, it must be a GMI image model (`hy-image-v3.5-preview`), never another provider.

- Draft cheap with `MiniMax-H3-Lite` at $0.04 per second or `MiniMax-H3-Highspeed` at $0.06 per second, then generate the keepers on `MiniMax-H3`.

- Render the paper backbone with Remotion (see "Renderer setup"). It is deterministic and nearly free once built. Narration, motion, H3 clips, and the music bed all assemble inside the render, in beat order. ffmpeg is only for the small share copy afterwards.

- Score the episode with `minimax-music-3.0` on the request queue API. It makes one full song, up to 5 minutes, from lyrics plus a style prompt, for a flat $0.15 per song. For a narration bed, generate an instrumental track (see "Music bed") and mix it low under the voice.

- The chat API lists only language models. Video and speech run on the request queue API on a different host, so the connector must cover both hosts before any video or speech call works.

## Design system

- Keep every episode on the same look, so the series is recognizable. These tokens come from episode 1's render code (`~/workspace/collage-render/src/BeatScene.tsx`, `src/TitleCard.tsx`, `src/EndCard.tsx`); change them deliberately, not by accident.

- Composition: 1280x720 at 30 fps (landscape), 720x1280 at 30 fps (portrait). Total 4,070 frames / 135.68 s: a 90-frame title card + 3,890 beat frames + a 90-frame end card.

- Paper background `#efe6d0` with a procedural grain overlay (SVG feTurbulence, opacity 0.08, multiply — `PaperNoise` in `src/cutout.tsx`) so the paper reads as paper, not a flat color. No texture asset needed. Paper card `#f7f1de` with a 2px `#d8cbb0` border and a soft shadow. Beat title text white, Georgia serif, bold, 38px landscape (34px portrait), center-aligned, in a matte dark brown strip (`#4e342e` bg, `#6d4c41` border) **above the image** — the beat title is a headline over the cutout, not a caption under it. (v8 briefly inverted the subtitle strip instead — wrong strip; v9 corrects it.)

- Each beat image is a pasted cutout, not a framed photo: a torn edge (seeded jagged `clip-path: polygon`, a different tear per beat, `src/cutout.tsx`), a slightly larger off-white paper backing peeking out behind it, and a drop shadow that follows the torn silhouette. Rotated slightly (plus or minus 1.5 degrees, alternating per beat), popping in with a spring (damping 14, stiffness 110). The caption slides up and fades in over the first 23 frames.

- Motion is slow Ken Burns only: zoom in from 1.02 to 1.14, zoom out from 1.14 to 1.02, or a pan at scale 1.14 shifting 2.5% left or right across the beat. No fast moves; the abruptness comes from the beat cuts, not from the motion inside a beat.

- **Title card** (`src/TitleCard.tsx`, 90 frames): dateline (Georgia, letterSpacing 4, 17px landscape / 14px portrait), masthead in UnifrakturMaguntia blackletter (120px landscape / 88px portrait — the user asked for a distinctive newspaper-header character), double rule (4px + 1px), headline in Georgia bold (52px / 36px, letterSpacing 3), thin rule at 60% width under it. Animations: dateline fade 0–16, masthead fade+scale 2–28, rules scaleX 14–34, headline fade+rise 20–38, whole card fades out 78–90. Font file `public/fonts/UnifrakturMaguntia-Regular.woff2` (OFL), registered via inline `<style>` @font-face with staticFile. Title voice plays inside the card (`<Audio src={staticFile("title-voice.mp3")} />`).

- The last beat is a dark closing card: `#141414` background, Georgia serif 40px (36px portrait), text in paper color `#efe6d0`, fading and scaling from 0.96 to 1 over 70 frames.

- **End card** (`src/EndCard.tsx`, 90 frames): pure black `#000000` background, official white GMI logo centered (see "The GMI logo end card"), fade in over 15 frames, hold, fade out over the last 15 frames. Silent.

- An H3 beat keeps the same cutout treatment and caption. The clip sits in the middle of the beat with the still image on both sides (side length = (beat frames − 150) / 2), and the clip audio plays at volume 0.5.

## Renderer setup

- The render project lives at `~/workspace/collage-render/` (Remotion 4 + React 18): `src/beats.ts` holds the beat list with frame counts plus `TITLE_FRAMES` and `ENDCARD_FRAMES`, `src/BeatScene.tsx` the paper look, `src/TitleCard.tsx` the title card, `src/EndCard.tsx` the end card, `src/Episode.tsx` the beat order and the audio layers, `src/Root.tsx` the compositions. Assets go in `public/` (images, narration mp3, H3 clips, music bed, logo SVG, fonts).

- The stock Chromium at `/opt/meta-chromium/chrome` is policy-locked and cannot render. Download Remotion's own headless shell first:

```text
npx remotion browser ensure
```

- In Remotion 4 the config function is `setBrowserExecutable` (not `setChromiumExecutable`); the CLI also accepts `--browser-executable`. A `tsconfig.json` is required for the render to type-check.

- Render with:

```text
npx remotion render src/index.ts KeyboardEp01 <output.mp4>
```

- Do NOT prepend `npx tsc --noEmit` — typescript is not installed in the project; the npx tsc step fails and kills the chain.

- A ~136 second episode takes about 5–6 minutes to render on 1.5 shared CPUs. If local CPU is the bottleneck, move the render to the VPS (`/home/it/muse`, 8 cores) — the user offered it as overflow workspace.

- Episode folders live at `~/workspace/collage-ep<NN>/`: assets, H3 clips, narration, master mp4, and the share mp4. Write render logs into the episode folder (`render-vN.log`), not /tmp — /tmp was wiped by a runtime restart mid-render once.

## Portrait version and captions

- Every episode also gets a portrait cut for phones (Shorts/Reels/TikTok): 720x1280 (9:16) at 30 fps, same beats, same frame counts, same narration as the landscape version. Only the layout changes, plus narration subtitles.

- Portrait layout (720x1280, same paper tokens): the content is one centered stack, not spread top to bottom — the beat title strip on top (bottom-anchored zone top 212, height 136 — the strip's bottom sits 28px above the card however many lines it wraps to, Georgia bold 34px, center-aligned, dark brown bg + white text), the cutout card under it (top 376, left/right 32, height 445 — about 3:2, so the 16:9 sources crop slightly at the sides), the narration subtitle strip right below the card (top 861, height 160, centered, Georgia 30px, light paper). The bottom ~270px of the frame stays empty on purpose: Shorts/Reels/TikTok put their progress bar, buttons, and title there, and any text in that zone gets covered. The closing card is the same dark card with 36px text and 48px padding.

- Subtitles come from Whisper word timestamps of the repaired narration (`narasi-fixed-words.json`), grouped into cues of at most 84 characters (about 2 lines at 30px), split on silence gaps over 0.7 seconds or a 5 second maximum. The text is hand-corrected against the script, because Whisper mishears names and numbers — see `~/workspace/collage-ep01/build-captions.py` for the grouping rules and the correction map. The cues live in `src/captions.ts` as `{ start, end, text }` in seconds.

- In the render, the active cue shows as a light paper strip (same `#f7f1de` card, `#d8cbb0` border, `#1a1a1a` text, Georgia 30px, line-height 1.45), fading in and out over 6 frames at each cue boundary. The beat title strip above the image is the dark brown one (`#4e342e` bg, white text) — v8 inverted the subtitle strip by mistake and v9 swapped it back. Subtitles stop at the closing beat, because the closing card already carries those words.

- The portrait composition is a second composition in the same Remotion project: `KeyboardEp01Portrait` in `src/Root.tsx`, component `src/PortraitEpisode.tsx`, scenes in `src/PortraitBeatScene.tsx`. The H3 beat keeps the same middle-of-beat clip placement as landscape. Render with `npx remotion render src/index.ts KeyboardEp01Portrait <output.mp4>` and make a share copy the same way (`-crf 28`).

- Verify a portrait render with the same checklist plus: subtitle text is readable at phone size (no overflow past the strip), subtitles match the spoken words at the spot-checks, and no subtitle shows on the closing card.

## Collecting assets

- Find images with the image-search CLI:

```text
/opt/hatch/bin/image-search "<query>" --max-results 5
```

- The CLI returns locators only — it does not download. `media_url` is the preferred full-image URL, `thumbnail_cdn_url` is the fallback, and `page_url` is the source page. Keep the `page_url` for provenance.

- Only public domain or CC0 images (CC-BY is acceptable with attribution recorded). Check the license on the source page before using an image — the search results do not filter by license. Prefer sources whose license is explicit: Wikimedia Commons, Smithsonian Open Access, Library of Congress, and Openverse (which lets you filter by license). Never use an image with an unclear license, a non-commercial restriction, or a watermark.

- Download with the preflight pattern from the image-search skill: HEAD first with a browser user agent and a cross-origin referer, then GET with bounded timeouts, and accept only a final 2xx whose content type and file magic are an image. Reject hotlink-blocked, expiring, redirecting, 403/404, non-image, or watermarked URLs.

- Make the mixed sources look like one series: crop every image to 16:9 at 2048x1152 and apply the sepia treatment (verified 2026-11-20):

```text
ffmpeg -y -i <source> -vf "scale=2048:1152:force_original_aspect_ratio=increase,crop=2048:1152,colorchannelmixer=.393:.769:.189:0:.349:.686:.168:0:.272:.534:.131" <beatN-slug.jpg>
```

- Record the source: save each image as `beat<N>-<slug>.jpg` with its `page_url` and license noted in the episode folder (for example in an `assets/SOURCES.md`), so the episode's provenance is auditable.

- If no suitable licensed image exists for a beat, use a typographic paper card for that beat instead of generating an image. The format already has a dark closing card; a headline-style card in the same paper style fits the same language. Generation is not the fallback — collection is the format, and a paper card is the fallback. (If generation is ever unavoidable, it must be a GMI image model — see "The all-GMI rule".)

## The GMI logo end card

- The end card carries the official white GMI wordmark: big "GMI" letters plus "General Machine Intelligence" stacked on the right. Source: Brandfetch, `brandfetch.com/gmicloud.ai` — the page lists four versions (white logo, symbol, white symbol, icon) as SVG / transparent PNG / JPEG. The brand ID is `idSLfuG3JC`; the CDN pattern is `https://cdn.brandfetch.io/idSLfuG3JC/theme/<theme>/<asset>.svg`.

- Brandfetch's theme naming is backwards from what you would guess: the **light** theme files carry white fills (`fill="white"`), the dark theme files carry black fills. The white logo is the light-theme one. Download with a browser user agent and a `Referer: https://brandfetch.com/gmicloud.ai` header, or the CDN returns empty files. The file used in episode 1 is the light-theme wordmark SVG (560x123, `logo-light-1.svg`), saved as `public/gmi-logo-white.svg`. The emblem-only variant is the light-theme symbol; keep both files in the episode folder for later (`~/workspace/collage-ep01/gmi-logo/`).

- The logo is an SVG with intrinsic dimensions, so Remotion's `<Img>` renders it crisp at any size. Size it by frame width: **39% of the frame width** (portrait 720 → 281px). Episode 1 v10 used 78% and the user said it was too big; v11 halved it to 39% and the user approved. Do not grow it back without asking.

- The card is pure black `#000000` (the user corrected this explicitly — not the `#141414` of the closing card), logo centered both ways, fade in 0–15 frames, hold, fade out over the last 15 frames. Silent. `src/EndCard.tsx`, `ENDCARD_FRAMES = 90` in `src/beats.ts`.

## Pipeline

1. Take the topic from the user, then research the facts with web search first and keep the source list. Never let the model invent facts. A narration that states a wrong fact is worse than no video.

2. Draft the script with `MiniMaxAI/MiniMax-M3.1-Flash` through the chat helper, feeding the researched facts in the prompt. Demand a hook opening, short beats, each beat with a `[Visual: ...]` note, and a total narration of 900 to 1,100 characters for 60 to 75 seconds, or the length the user asked for.

```text
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_chat.py --model MiniMaxAI/MiniMax-M3.1-Flash --messages-file <messages.json> --temperature 0.7 --max-tokens 2500
```

3. Clean the draft by hand. Check every fact against the research, fix garbled fragments, and trim to the target length. Never generate voice from an uncleaned draft — voice costs money and bakes the errors in.

4. Quote the voice. Count spoken characters only, stripping `[Visual: ...]` lines and headings. Price is characters divided by 1,000 times $0.06. State the total and get explicit human approval before submitting.

```text
python3 -c "
import re
txt = open('<script.md>').read()
spoken = re.sub(r'\[Visual:.*?\]\n?', '', txt)
spoken = re.sub(r'^#.*$', '', spoken, flags=re.M)
spoken = '\n'.join(l for l in spoken.splitlines() if l.strip())
print('spoken chars:', len(spoken))
print('voice price USD:', round(len(spoken)/1000*0.06, 4))
"
```

5. Generate the voice. Read the live speech schema first, then submit once with the queue helper and keep the returned request id. Pick the voice by filtering the schema's voice list by the narration language prefix (for example `Indonesian_*`), preferring narrator-type voices (Calm, Narrator, Storyteller), and set `language_boost` to the narration language. The series voice is `Indonesian_CalmWoman` with `language_boost: Indonesian` — reuse it unless the user asks for a different voice.

```text
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py model minimax-tts-speech-2.6-turbo
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py submit --model minimax-tts-speech-2.6-turbo --payload-file voice.json
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py wait <request-id> --timeout-min 10
```

6. Transcribe-verify the narration with local Whisper before building any visuals (see "Verify the narration before rendering"). Fix any inserted garbage by cutting at silence boundaries — no re-generation when the garbage is pure insertion. Re-time the beats from Whisper word timestamps on the repaired audio.

7. Collect the collage assets from the internet — one image per beat plus a closing card (see "Collecting assets"). Search with the image-search CLI, prefer public-domain and CC0 sources (Wikimedia Commons, Smithsonian Open Access, Library of Congress, Openverse), and check the license on the source page before using any image. Never use an image whose license is unclear or non-commercial-only. Real archival photos and scans are preferred over generic stock — for historical topics they are the point, not a compromise. Download each approved image, crop it to 16:9 at 2048x1152, and apply the sepia treatment so the mixed sources look like one series. Save as `beat<N>-<slug>.jpg` in the episode folder and record the source page URL next to each file. If no suitable licensed image exists for a beat, use a typographic paper card for that beat (a headline-style card like the closing card) instead of generating an image. Copy the approved images into the render project's `public/` folder.

8. Decide the visuals per beat. Motion graphics for most beats — that is the paper backbone. H3 only for beats that need real motion or native audio. Quote each H3 clip (model, resolution, seconds, total) and get approval before submitting it. Read the live H3 schema before every submit. Make the H3 clip blend with the paper: prompt it sepia or vintage, keep the ratio 16:9 to match the composition, and frame it inside the same paper card as the stills, like a photograph pasted on the page — never a full-screen realistic shot that breaks the collage look. In the render, the clip sits in the middle of its beat with the still image on both sides.

```text
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py model MiniMax-H3
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py submit --model MiniMax-H3 --payload-file clip1.json
```

9. Generate the music bed, unless the user says no music. See "Music bed": quote the $0.15, get approval, submit once with `minimax-music-3.0`, and mix it low under the narration in the render.

10. Add the title card and the end card. The title card gets its own short voice job (same series voice, title text only — quote it, it is a few cents). The end card uses the official white GMI logo SVG from Brandfetch (see "The GMI logo end card") — no model call, no approval needed beyond the format itself.

11. Render the episode with Remotion: title card, narration audio plus motion segments plus H3 clips plus the music bed plus closing card plus end card assemble in the render itself, in order. Export the final file, then make a smaller share copy with ffmpeg (`-crf 28`, about a third of the master size). Run the pre-delivery checklist before handing anything over. Report the path, the duration, and the total spend.

12. Record the episode: save the script, the spoken text, the request ids, and the total spend in the project folder, so the next episode reuses the same recipe. Version every render (see "Iteration & versioning").

## Quote, confirm, create once, poll

- Quote the exact price before every job and get explicit human approval to spend it. State the model, the resolution or character count, the seconds, and the total, such as Speech 2.6 Turbo for 1,934 characters is $0.12, H3 at 768P for 5 seconds is $0.40, or Music 3.0 for one song is $0.15. A vague request to generate is not approval.

- Submit exactly once with the queue helper and keep the returned request id. Reuse that id only to poll the same job.

- Poll with the queue helper using the same request id. Poll with a growing interval and a deadline that fits the job length. Voice jobs take seconds to a couple of minutes; video jobs take minutes.

- Never submit again to check on a job. A second submit is a second charge. Treat `failed` and `cancelled` as terminal and stop there.

- Read the result URL from the outcome on success: video jobs return `outcome.video_url` plus `outcome.thumbnail_image_url`, speech jobs return `outcome.audio_url`, music jobs return `outcome.audio_url` too. Copy the result files into the project storage and report the file path and the URL.

- A retry is a new job. Quote it again and get approval again.

## M3.1 Flash writing lessons

- Use `max_tokens` 2500 or higher for writing tasks. A call with 1200 returned an empty reply, because the reasoning budget ate the limit; 2500 worked.

- Drafts can contain garbled fragments: nonsense words, stray non-Indonesian tokens, broken sentences. Always hand-clean before generating voice.

- Ground the model with researched facts in the prompt and forbid new facts. The model invents plausible-sounding details when the prompt leaves gaps.

## Verify the narration before rendering

- Transcribe the generated narration with local Whisper before building any visuals. Speech 2.6 Turbo can degenerate mid-generation into a repeated fragment loop: in episode 1 it inserted about 9 seconds of looped garbage at 76.8 seconds ("of falling off" repeated), while all the script words were still present before and after it. Whisper returns zero segments for the looped stretch while the neighbors transcribe fine, which is how you spot it.

- If the narration has inserted garbage, cut it out at silence boundaries with ffmpeg (`silencedetect` to find the pauses, `atrim` plus `concat` to join). No re-generation is needed when the garbage is pure insertion. Then transcribe the repaired audio again to confirm it reads clean end to end.

- Never time beats from character counts. Measure the true beat boundaries with Whisper word timestamps on the repaired narration and set the frame counts from those times. Character estimates drifted up to about 10 seconds off the visuals by beat 7 in episode 1. The first render's timings were estimates; the fixed render's timings are measured.

- Verification tooling: a faster-whisper venv already exists at `~/workspace/whisper-env` with the `small` model cached (~464 MB). Decode with ffmpeg first and pass a numpy array — the library's built-in file loader crashes on this install (`av.open` fails on a `metadata_errors` argument).

```text
ffmpeg -i <narration.mp3> -ar 16000 -ac 1 -f s16le <narration.raw>
~/workspace/whisper-env/bin/python -c "
import numpy as np
from faster_whisper import WhisperModel
audio = np.fromfile('<narration.raw>', dtype=np.int16).astype(np.float32) / 32768.0
model = WhisperModel('small', device='cpu', compute_type='int8')
segments, info = model.transcribe(audio, language='id', word_timestamps=True)
for s in segments:
    print(f'[{s.start:.1f}-{s.end:.1f}] {s.text}')
"
```

- Use `word_timestamps=True` to get the true beat boundaries: the start of each beat is the timestamp of its first word. Segment boundaries alone are not enough — two beats can share one segment.

- This step is local QA, not a GMI step: GMI has no speech-to-text model (checked 2026-11-21 — the queue catalog's speech models are TTS and voice-clone only), so transcription stays on local Whisper. It verifies GMI output; it does not create any of it.

## Music bed

- Generate the bed with `minimax-music-3.0` on the request queue API. It makes one full song, up to 5 minutes, from lyrics plus a style prompt, for a flat **$0.15 per song**. One song covers a whole episode; trim or loop it to the narration length in the mix.

- Read the live schema before every submit. The payload fields are `lyrics` (required, 1–3,500 characters, with structure tags like `[Intro]`, `[Verse]`, `[Chorus]`, `[Bridge]`, `[Outro]`, `[Interlude]`, `[Hook]`, `[Inst]`, `[Solo]`), `prompt` (optional style, 0–2,000 characters), `sample_rate` (default 44100), `bitrate` (default 256000), and `format` (default `mp3`). The result comes back as `outcome.audio_url` with `duration_ms`. The job is synchronous and usually returns in 30–60 seconds.

```text
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py model minimax-music-3.0
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py submit --model minimax-music-3.0 --payload-file music.json
python3 ~/workspace/custom-connectors/custom.gmi/bin/gmi_queue.py wait <request-id> --timeout-min 10
```

- The model writes songs from lyrics, so a narration bed needs an instrumental-only recipe: put `[Inst]` sections in the lyrics with parenthetical instrument directions (for example `[Inst]\n(gentle piano, soft strings, slow)`), and a style prompt that says instrumental with no vocals (for example "gentle piano and soft strings, calm, warm, instrumental, no vocals"). Quote the $0.15 and get approval before submitting, like any other job.

- Mix the bed low under the narration so the voice stays clear: about 0.15–0.25 volume on the music audio layer in the render, narration at full volume. Fade the bed out in the last 2 seconds so the episode ends clean.

- Status: the schema and price were verified live 2026-11-20, but no song has been generated yet — the instrumental-bed recipe above is untested. Run one $0.15 test song first and listen to it before using music in a real episode.

## Iteration & versioning

- Episode 1 took eleven versions (v1–v11) across one day, and every version taught something. Version every render: `keyboard-ep01-<cut>-v<N>.mp4` plus a `-share.mp4` copy, with the render log in the episode folder (`render-vN.log`). Never overwrite the current reference — a new version is a new file.

- The user corrects by looking, not by spec. Expect feedback on the frame itself ("the brown belongs on the title strip", "the margin still touches the photo", "the logo is too big") and fix exactly what was pointed at — do not redesign the rest. If a fix touches the design system, update the tokens in this skill in the same round, so the next episode starts from the corrected look.

- Render portrait only once the user says so. Episode 1's standing instruction (2026-11-20): landscape stays at v6, do not re-render it. Code changes still go into both cuts so the design stays consistent; only the render is portrait-only. A new instruction from the user overrides this.

- Two layout bugs to never repeat: (1) a 2-line beat title eats the gap to the photo — the title zone is bottom-anchored (top 212, height 136, strip bottom 28px above the card) so the gap is constant however many lines the title wraps to. (2) The subtitle strip and the title strip are different elements — when the user says "invert the caption", check which strip they mean before changing colors; v8 inverted the wrong one.

- Verify frames before delivering, every version. Extract stills with ffmpeg and look at them: the title-card frame, the beat-1 hook, the H3 beat (clip actually playing, not a frozen frame), the longest-title beat (2-line title gap), the end card (logo size and centering), and the closing card. The user's screenshot of a frame is the bug report — match the timestamp and fix what it shows.

- The share copy is part of the delivery, not an afterthought: master plus share, with sizes and duration, in the same message. The master is the archive; the share is what gets watched.

- Cleanup: when the user says to delete the old artifacts, move the superseded version files to the recoverable trash (30 days), never `rm`. Keep the source clip (`beat6-h3.mp4`), the current landscape reference (if any), and the current portrait. Say what was trashed and what was kept.

## Pre-delivery checklist

- Run this before handing any video to the user. The episode 1 glitch was caught by the user, not by the process — this list exists so that never happens again.

- Audio duration equals video duration (within 0.1 seconds).
- Transcribe the risky regions of the final mix (any stretch Whisper returned zero segments for in the narration check) and confirm they read clean.
- Spot-check frames: the title card, the beat 1 hook, the H3 beat (clip actually playing, not a frozen frame), the longest-title beat (2-line title gap to the photo), the end card (logo size, centering, black background), and the closing card.
- Confirm the caption text is right on every beat and the beat order matches the script.
- Confirm the end card is silent and the narration ends exactly when it starts.
- Make the share copy (`ffmpeg -i <master.mp4> -c:v libx264 -crf 28 -preset medium -c:a aac -b:a 96k <share.mp4>`) and deliver the master plus the share copy with their sizes and duration.

## Pricing quick table

- Script with M3.1 Flash: about $0.001 per draft, negligible.
- Voice with Speech 2.6 Turbo: $0.06 per 1,000 characters. A 1,934 character narration is about $0.12 and runs about 2 minutes. Title voice (33 chars) is $0.002.
- H3 at 768P: $0.08 per second. H3 at 2K: $0.13 per second. H3-Lite: $0.04 per second. H3-Highspeed: $0.06 per second.
- Music bed with Music 3.0: $0.15 per song flat (up to 5 minutes). One song per episode is enough.
- Generated image with hy-image-v3.5-preview: $0.024 per image (only if generation is ever needed — the format collects instead).
- Paper backbone with a motion renderer: render cost only, no per-second model charge.
- Episode 1 total model spend: about $0.52 (script ~$0.001, narration $0.12, H3 $0.40, title voice $0.002).

## Paths & porting

- The paths in this skill are the author's working layout: render project `~/workspace/collage-render/`, episode folders `~/workspace/collage-ep<NN>/`, GMI helpers in `~/workspace/custom-connectors/custom.gmi/bin/`, Whisper venv `~/workspace/whisper-env`. To use this skill elsewhere, point those four at your own layout — the file names inside (`src/beats.ts`, `public/`, `gmi_queue.py`) stay the same.

- The skill depends on the `custom.gmi` connector (chat API + request queue API, both hosts) and its helpers `gmi_chat.py` / `gmi_queue.py`. Without them, the model calls in "Pipeline" have no transport — set up the connector first (see "Available connectors").

## Worked example

- "Fakta Unik Ditemukannya Keyboard" (episode 1, 2026-11-20–21): 11 beats, 1,934 spoken characters, voice quoted at $0.12, about 2 minutes. Script at `~/workspace/narasi-keyboard-script.md`, spoken text at `~/workspace/narasi-keyboard-spoken.txt`. The voice job is done: request `6421cb21-71a6-47c9-b17b-8096fe709e92`, audio at `~/workspace/narasi-keyboard-voice.mp3` (2.2 MB, 138.9 seconds). Series voice, approved by the user: `Indonesian_CalmWoman` with `language_boost: Indonesian` — the user describes it as anime-girl style and says it fits the format. Reuse it for the next episode unless the user asks for a different voice. The first render had a TTS loop glitch at 76.8 seconds and beat timings estimated from character counts; the reference version is the FIXED one: narration `~/workspace/collage-ep01/narasi-keyboard-voice-fixed.mp3` (129.67 seconds, garbage cut at silence boundaries, beats re-timed from Whisper word timestamps), video `~/workspace/collage-ep01/keyboard-ep01-fixed.mp4` (160 MB master) and `keyboard-ep01-fixed-share.mp4` (34 MB share copy). Total model spend for the episode: about $0.52. Portrait cut with subtitles: `~/workspace/collage-ep01/keyboard-ep01-portrait.mp4` (74 MB master) and `keyboard-ep01-portrait-share.mp4` (17 MB share copy), 720x1280, same 129.67 seconds, no extra model spend (subtitles come from the existing Whisper transcription). Rebuilt 2026-11-20 as v2: all 10 assets replaced with collected public-domain/CC0 photos (sources in `~/workspace/collage-ep01/assets-collected/SOURCES.md`; beat 7 is a Remington No. 6, not a No. 2 — no free-licensed No. 2 photo exists; beat 9 is CC-BY with attribution recorded), portrait layout fixed (bigger 3:2 card, subtitle right under the card, beat caption at bottom 200px so the bottom 200px stays clear of platform UI). v2 files `keyboard-ep01-v2.mp4` (89 MB master) + `keyboard-ep01-v2-share.mp4` (25 MB), portrait `keyboard-ep01-portrait-v2.mp4` (41 MB master) + `keyboard-ep01-portrait-v2-share.mp4` (13 MB). Rebuilt 2026-11-20 as v3 per the user's direction: images are pasted cutouts with torn edges (`src/cutout.tsx`, seeded clip-path, paper backing, drop shadow following the torn silhouette) instead of white-framed photos, procedural paper grain on the background, captions bigger (38px landscape, 34px portrait), portrait content as one centered stack (card top 300, subtitle 769, caption 953). v3 files `keyboard-ep01-v3.mp4` (64 MB) + `keyboard-ep01-v3-share.mp4` (17 MB), portrait `keyboard-ep01-portrait-v3.mp4` (33 MB) + `keyboard-ep01-portrait-v3-share.mp4` (11 MB). Portrait re-rendered once more as v4 (same look, caption container gets 24px side margins so the longest caption stays a label instead of a full-bleed strip): `keyboard-ep01-portrait-v4.mp4` (33 MB) + `keyboard-ep01-portrait-v4-share.mp4` (11 MB). Current reference: landscape v3 + portrait v4. Rebuilt 2026-11-20 as v5: the beat title moved above the cutout (headline over the photo, not a caption under it) — landscape title top 24, image 112–688; portrait title top 280, card 376–821, subtitle 849; all captions center-aligned; beat 6 caption shortened to 'Sengaja dilambatin'. v5 files `keyboard-ep01-v5.mp4` (67 MB) + `keyboard-ep01-v5-share.mp4` (18 MB), portrait `keyboard-ep01-portrait-v5.mp4` (31 MB) + `keyboard-ep01-portrait-v5-share.mp4` (11 MB). Current reference: v5 both. Rebuilt 2026-11-20 as v6: a 3 s newspaper-masthead title card opens the video (TITLE_FRAMES = 90, total 3,980 frames / 132.67 s) — dateline 'EDISI KHUSUS • 20 NOVEMBER 2026', masthead 'Fakta Unik' in UnifrakturMaguntia blackletter (public/fonts/, @font-face via staticFile), double rule, headline 'DITEMUKANNYA KEYBOARD'. Beats, narration, and subtitles all shift +90 frames (wrap Audio/Subtitles in a Sequence from TITLE_FRAMES). v6 files `keyboard-ep01-v6.mp4` (67 MB) + `keyboard-ep01-v6-share.mp4` (18 MB), portrait `keyboard-ep01-portrait-v6.mp4` (31 MB) + `keyboard-ep01-portrait-v6-share.mp4` (11 MB). Current reference: v6 both. Rebuilt 2026-11-20 as v7 (portrait only — landscape stays v6 per the user's instruction): portrait gaps around the asset widened to 28px (card top 376–821, subtitle top 849), and the title card now has voice — the narrator reads 'Fakta unik: ditemukannya keyboard.' (Speech 2.6 Turbo, Indonesian_CalmWoman, 33 chars, $0.002, request af257b71-d472-409d-b852-b674ea9157ca, 2.76 s, Whisper-verified clean, mixed in via an <Audio> in TitleCard). v7 files `keyboard-ep01-portrait-v7.mp4` (32 MB) + `keyboard-ep01-portrait-v7-share.mp4` (11 MB). Current reference: landscape v6 + portrait v7. Rebuilt 2026-11-20 as v8 (portrait only): subtitle strip inverted — matte dark brown `#4e342e` bg, `#6d4c41` border, white text (beat title strip stays light paper). v8 files `keyboard-ep01-portrait-v8.mp4` (31 MB) + `keyboard-ep01-portrait-v8-share.mp4` (11 MB). Current reference: landscape v6 + portrait v8. Rebuilt 2026-11-20 as v9 (portrait only): brown inversion moved to the beat title strip (v8 had inverted the subtitle strip by mistake) — title strip `#4e342e` bg + white text, subtitle strip back to light paper; title zone bottom-anchored (top 212, height 136) so 2-line titles keep a 28px gap above the card; subtitle gap widened to 40px (top 861). v9 files `keyboard-ep01-portrait-v9.mp4` (32 MB) + `keyboard-ep01-portrait-v9-share.mp4` (11 MB). Current reference: landscape v6 + portrait v9. Added 2026-11-21 as v10 (portrait only): 3 s end card after the closing card — official white GMI logo (Brandfetch white wordmark, `public/gmi-logo-white.svg`, "GMI" + "General Machine Intelligence"), centered on pure black `#000000`, fade in/out; `ENDCARD_FRAMES = 90` in beats.ts, `src/EndCard.tsx`, total 4,070 frames / 135.68 s. v10 files `keyboard-ep01-portrait-v10.mp4` (32 MB) + `keyboard-ep01-portrait-v10-share.mp4` (11 MB). Current reference: landscape v6 + portrait v10. Rebuilt 2026-11-21 as v11 (portrait only): end-card logo halved (39% of frame width, user said v10's logo was too big) — `src/EndCard.tsx` width multiplier 0.78 → 0.39. v11 files `keyboard-ep01-portrait-v11.mp4` (32 MB) + `keyboard-ep01-portrait-v11-share.mp4` (11 MB). Current reference: landscape v6 + portrait v11. Cleanup 2026-11-21 (user: 'artefak yg lain hapus'): 28 superseded mp4s (landscape v1–v5, portrait v1–v9) + v10 master/share moved to recoverable trash (30 days); kept: `beat6-h3.mp4` (source clip), landscape v6 (only landscape cut), current portrait.

## Verification status

- Verified live 2026-11-20: the chat API lists 70 models including `MiniMaxAI/MiniMax-M3.1-Flash`, and the queue API lists 384 models including `MiniMax-H3` and `minimax-tts-speech-2.6-turbo`. The live H3 schema confirms the payload fields and the price of $0.08 per second at 768P and $0.13 per second at 2K. The live speech schema confirms the payload fields and the price of $0.06 per 1,000 characters. A full narration script was drafted live with M3.1 Flash and hand-cleaned. Speech submit and poll verified live 2026-11-20: request `6421cb21-71a6-47c9-b17b-8096fe709e92`, voice `Indonesian_CalmWoman` with `language_boost: Indonesian`, 1,934 characters, $0.12, dispatched to success in about 15 seconds, result 2.2 MB mp3, 138.9 seconds. Speech results come back as `outcome.audio_url`, not `media_urls`. H3 submit and poll verified live 2026-11-20: request `4ed53084-3107-4a06-86b3-18814e5c5c6d`, 5 seconds at 768P, $0.40, dispatched to success in about 2 minutes, result 1344x768 h264 with aac audio, 5.17 seconds. H3 results come back as `outcome.video_url` plus `outcome.thumbnail_image_url`.

- Full pipeline verified end to end 2026-11-20–21 with episode 1, "Fakta Unik Ditemukannya Keyboard": 11 beats, Remotion 4 render of 4,070 frames at 1280x720 and 720x1280 (local render took about 5–6 minutes on 1.5 shared CPUs; the stock Chromium at `/opt/meta-chromium/chrome` cannot be used because it is policy-locked — use `npx remotion browser ensure` to download Remotion's own headless shell), title card with voice, narration plus one H3 clip plus closing card plus GMI logo end card assembled in the render, final video 135.68 seconds after the TTS glitch fix, beat re-timing, and the end card. Total model spend for the episode: about $0.52 (script ~$0.001, voice $0.12, H3 $0.40, title voice $0.002). Current reference: landscape v6 + portrait v11.

- Asset collection from the internet verified 2026-11-20 (v2+): 10/10 beats collected as public-domain/CC0 photos (sources + licenses in `~/workspace/collage-ep01/assets-collected/SOURCES.md`), treated to 2048x1152 sepia, no generation. Beat 7 is a Remington No. 6, not a No. 2 — no free-licensed No. 2 photo exists; beat 9 is CC-BY with attribution recorded.

- Portrait version with subtitles verified 2026-11-20: episode 1 portrait render, 720x1280, 4,070 frames, 135.68 seconds, 30 subtitle cues from Whisper word timestamps, spot-checked at 2s (hook + first subtitle), 60s (H3 clip + subtitle), 77s (beat 7 + subtitle), 126s (closing card, no subtitle), 134s (end card, white GMI logo centered on black).

- All-GMI audit 2026-11-21: every generative step runs on a GMI model (M3.1 Flash, Speech 2.6 Turbo, MiniMax-H3, Music 3.0; hy-image-v3.5-preview confirmed in the queue catalog for any future image need). The queue catalog was re-checked 2026-11-21 (385 models): no speech-to-text model exists on GMI, so transcription verification stays on local Whisper as QA tooling. No non-GMI AI is used anywhere in the pipeline.

- Untested: music generation with `minimax-music-3.0` — the schema and the $0.15 flat price were verified live 2026-11-20, but no song has been generated yet, so the instrumental-bed recipe is unproven. Run one $0.15 test song first and listen to it before using music in a real episode.

## Available connectors

- [GMI Cloud](~/workspace/custom-connectors/custom.gmi/CONNECTOR.md) for the chat API and the request queue API, including the helpers this skill calls.

- Related skill: `gmi-minimax-video`, same MiniMax stack, clip-first videos. This skill is the narrator-first, collage-format sibling.

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Alternatives for better results

- Keep the paper backbone doing most of the work and spend H3 only on the one or two beats that motion alone cannot carry. H3 is charged per second; motion is nearly free once built.

- Render motion at the final resolution from the start, so the H3 clips and the motion segments match without a re-render.

- Keep episodes short while the format is unproven. A 60 to 75 second episode costs about $0.06 of voice and tests the whole pipeline; a 2 minute one costs about $0.12.
