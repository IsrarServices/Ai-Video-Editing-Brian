# The Ultimate Claude Code Video Editing Skill — Research & Planning Report

**Status: PHASE 1 — Research & Planning only. No skill files have been built yet. Awaiting approval.**

This document is the persisted copy of the plan presented in chat. It exists purely as a
working record inside the project — it does not change, depend on, or affect the Next.js
application scaffold in this repository. No `SKILL.md`, scripts, or rendering code have
been created yet, per the explicit instruction to get plan approval first.

---

## A. Reference Audit

### Reference A — Rem Prakash / Vibha Ramprakash's "45 Video Effects" skill
Repo: `github.com/Vibha-Ramprakash/claude-video-effects` — **fully accessible**, inspected directly (README.md, SKILL.md, file tree, requirements.txt).

What it actually is:
- A single Claude Code skill (`SKILL.md`, ~10.6 KB) plus one large Python script (`scripts/fx.py`, ~147 KB) that implements 45 named "effects" as OpenCV/MoviePy/Pillow-style frame-compositing routines, driven by a face tracker.
- **Face/person detection stack**: OpenCV **YuNet** (ONNX, MIT) for face bounding boxes + Google **MediaPipe Selfie Segmenter** (Apache-2.0, `.tflite`) for person cutouts. Both models are vendored in `models/`.
- **Fonts**: Archivo Black, Instrument Serif, Space Mono, Inter, Anton, Montserrat (SIL OFL) and Permanent Marker (Apache-2.0) — all vendored with license files in `fonts/`.
- **Dependencies**: `numpy`, `opencv-python`, `pillow`, `mediapipe` (requirements.txt) + `ffmpeg` binary + Python 3.12 venv (`scripts/setup.sh`).
- **Workflow baked into SKILL.md**: (1) run setup once, (2) get video + requested effect name, (3) resolve timing either from user's words (there's a `scripts/words.py` helper implying a transcription/word-matching utility) or ask once, (4) **write a JSON plan** (`<video>_effects.json`) describing effects+start/end+params, (5) **render check-frame PNGs at the midpoint of each effect** and visually inspect before committing, (6) render final MP4 preserving original audio, (7) tell the user where the file is and remind them to review before posting.
- **Core design principle** (its single most valuable idea): *"Every effect follows your face. Anything that would cover your face goes behind your head instead."* — i.e., every visual effect is layout-aware and subject-aware, not blindly composited on top of the frame.
- The 45 effects are essentially **parametrized overlay templates** for one specific genre: a single vertical talking-head creator sitting roughly centered, filmed for Instagram Reels / TikTok. Examples: `orbit` (reels circle the head), `spotlight` (dark room, rim light), `glass-widgets` (frosted-glass UI floating around the head), `liquid-glass-graph` (KPI card + animated chart), `match-cut` (cut on a word to a locked graphic), `countdown-flip`, `comment-box`, `tap` (Follow→Following), `the-bill` (printable receipt graphic), etc.
- **License**: MIT for the code; model/font licenses documented separately and are permissive (MIT / Apache-2.0 / SIL OFL). **This means we can study the *techniques* (face tracking + segmentation + behind-the-head compositing + check-frame-before-render loop) freely, and could even vendor the actual face/segmentation models if we choose, with attribution.** We will not copy `fx.py` wholesale — it's a single 147 KB monolith hard-coded to one visual style (influencer/creator aesthetic) and one effect catalogue; several effects (Follow-button mockups, Instagram comment UI, "reel" cards) are platform/brand-specific gimmicks that do not generalize to tutorials, B2B, documentaries, etc.
- **What's valuable to borrow conceptually**:
  1. Face-aware / subject-aware placement ("never cover the face; put things behind the head instead") — a *general* layout rule, not just an effect.
  2. A JSON "edit plan" as an intermediate artifact between a creative decision and a render.
  3. **Render a few representative still frames first, inspect them, THEN render the full video.** This is the single best engineering idea in the repo, and we adopt it directly as a mandatory QC gate.
  4. Resolving "on the word X" timing via transcript search rather than the user supplying timestamps.
  5. Ordering rule: camera moves (push-ins, locks) before decorative overlays, because overlay geometry depends on where the subject ends up.
- **What's explicitly out of scope / not reusable**: it has **no sound design, no captions** (README says so explicitly — "Add those in CapCut"), no B-roll ingestion, no narrative/storyboard reasoning, no multi-shot editing (it only ever touches a single continuous clip), no color-grading pipeline beyond LUT-like presets, and the effect catalogue is heavily skewed to one platform's visual meme language (Instagram UI chrome), which would look dated/out of place in an explainer, documentary, or B2B deliverable.
- **Limitation acknowledged by the author**: "These are starter versions… My full edit style is coming later." The repo is intentionally a teaser/starter kit, not a complete professional pipeline.

### Reference B — "Sweet Motion" / feedmedia Notion guide
URL: `feedmedia.notion.site/How-to-edit-videos-with-Claude...` — **partially accessible**. I fetched the live public page in both rendered-text and raw-HTML form; both returned the same four sections, which appear to be the entirety of what's currently published (the page itself states *"My full edit style is coming later"*, i.e., this is a teaser identical in spirit to Reference A, from the same creator/network). I could not find additional hidden sections, further scrolling, or a paywalled continuation — what's below is everything the public URL serves today.

Content found:
1. **Setup prompt**: tells the user to literally paste *"Install Remotion, FFmpeg, Whisper (with word-level timestamps) and yt-dlp. Check each one runs."* and to use **Apple Vision person segmentation on macOS** or `rembg` on Windows/Linux for cutouts. This independently **confirms our technology shortlist** (Remotion + FFmpeg + Whisper) before I'd even finished researching it.
2. **"Storyboard first"** — the single most important workflow instruction in the whole project brief: *"write a storyboard as a table: one row per beat with the time, my exact words, what's on screen, and the tool. Don't render anything until I approve it."* This is a direct, explicit mandate for a **plan-before-render** gate, which we adopt and expand into Stage 3 of our workflow (with more columns, as the user's brief already specifies).
3. **"Text behind your head"** — person-segmentation cutout + title text sandwiched between background and the speaker, with exact typographic parameters (font, size, color, two-line layout, percentage-based vertical anchors, per-line staggered fade/rise easing). This is a good concrete pattern for our "kinetic typography" reference doc: titles behind a foreground subject read as more cinematic/premium than titles simply overlaid on top.
4. **"Captions"** — Whisper word-level timestamps → Remotion captions, max 3 words visible at a time, centered at 61.5% of frame height, with **only the currently-spoken word highlighted on a colored rounded box** (explicitly correcting the "naive" version where every word gets a box — the Notion doc itself notes "Make it better: don't put a colored box behind every caption. Highlight only the word being spoken."). This self-correction is itself a useful lesson: **a documented first instinct ("box every word") is called out as the worse version**, reinforcing our principle of restraint over maximal styling.
5. **"Tool labels"** — when a tool name is spoken, show its logo, then slide the wordmark out from behind the logo. A clean pattern for "named-entity → auto-generated lower-third" in software/tutorial content.

Access limitation: I could not access any further pages of this guide beyond these four recipes (no "every effect" content beyond these appears to be public yet). I'm treating these four patterns as **confirmed, usable technique references**, not as the complete "every effect" promised in the page title — the rest is apparently unpublished.

### Reference C — User's own videos / additional file
**Not accessible.** The user stated they could not upload the video examples or the additional document, and explicitly said to proceed without them ("they were mostly reference ke liye the, not as important"). No style benchmark from personal footage is available. **Mitigation**: the editing-profile system (Section 8 of the brief) will ship with multiple distinct, clearly-labelled visual/editorial profiles (premium personal-brand reel, technical explainer, B2B minimalist, documentary, etc.) precisely so that once real footage/examples are available, we select or tune a profile instead of inventing a single house style blind. If the user later shares footage or a reference doc, we will treat it as a style calibration step rather than re-architecting the skill.

### Reference D — HyperFrames (I added this myself; user listed it under "open-source existing skills")
Repo: `github.com/heygen-com/hyperframes` — **fully accessible**, inspected README in depth. This turned out to be materially important and is **not** a marginal reference:
- It is an actively maintained (npm-versioned, Node ≥22), **Apache-2.0**, HeyGen-backed open-source renderer that turns **plain HTML + CSS + data attributes** into deterministic MP4 via headless Chrome + FFmpeg — explicitly positioned as **"Built for agents"** and ships its own first-party **Claude Code / Codex / Cursor skill plugin** (`claude plugin install hyperframes@hyperframes`, or `npx skills add heygen-com/hyperframes`).
- It already includes a **490-block registry/catalog** (transitions, kinetic type, captions, charts, maps, 3D, camera moves) installable à la shadcn (`npx hyperframes add data-chart`), a **media skill** (`/media-use`) for sourcing music/SFX/voiceover/cutouts/color grades, and a **storyboard/slideshow skill**. It is animation-engine agnostic (GSAP, CSS, Lottie, Three.js, Anime.js, native WAAPI).
- Crucially: **no per-render fees, no headcount licensing restriction** (unlike Remotion, see Section B research below), **no build step** (an `index.html` plays as-is, which is dramatically friendlier to an LLM agent that has to both author and debug the composition), and **deterministic, seekable rendering** (same input → same frames, good for automated QC/diffing).
- This changes our architecture recommendation materially (see Proposed Architecture).

---

## B. Independent Research Findings

| Area | Finding | Source | Why it matters here |
|---|---|---|---|
| **Claude Code / Agent Skills spec** | A Skill = a directory with `SKILL.md` (YAML frontmatter: `name`, `description`, optional `license`, `compatibility`, `allowed-tools`, `metadata`) + optional `scripts/`, `references/`, `assets/`. Three-level progressive disclosure: (1) name+description always loaded (~100 tokens), (2) SKILL.md body loaded when triggered (<5k tokens recommended), (3) bundled files loaded only when explicitly read, and scripts only contribute their *output* to context, not their source. Now a **cross-vendor open standard** (agentskills.io, published Dec 18 2025, adopted by 44+ tools). | code.claude.com/docs/en/skills, agentskills.io/specification, anthropic.com engineering post, platform.claude.com best-practices | This is the backbone of our architecture: a lean root `SKILL.md` plus modular `references/*.md` that are only pulled into context on demand, and `scripts/*.py` whose *outputs* (JSON plans, QC reports, rendered stills) are what actually enters the transcript — not their code. Also: descriptions should be "a little pushy" per Anthropic's own skill-creator skill, because models under-trigger skills by default. |
| **Rendering engine choice** | Two credible, current, HTML/JS-based "render video with code" engines: **Remotion** (React, huge ecosystem, mature Lambda renderer) vs **HyperFrames** (plain HTML, Apache-2.0, agent-native, 2025-era, built specifically to be operated by coding agents). Remotion's license (v5.0 terms, checked live) requires a **paid Company License once total org headcount ≥ 4**, at $25/seat/mo (Creators) or $0.01/render with $100/mo minimum (Automators) — **free only for individuals and orgs of ≤3 people**, even though rendered output can be delivered commercially without restriction for those orgs. | remotion.dev/docs/license/{pricing,faq}, github.com/remotion-dev/remotion LICENSE.md, hyperframes README "HyperFrames vs Remotion" table | For a *skill* meant to be installed broadly (including by teams > 3 people, agencies, and anyone building an automated pipeline — exactly the "Automators" case Remotion's license targets), defaulting to Remotion as the only renderer would quietly create a commercial-licensing trap for larger users. HyperFrames has no such ceiling (Apache-2.0) and is explicitly designed to be driven by an agent rather than a human React developer, so it is our **primary/default renderer recommendation**, with Remotion documented and supported as an **opt-in alternative** for users who are already Remotion shops or need its mature cloud/Lambda rendering at scale and qualify for its free tier.
| **Animation engine** | **GSAP became 100% free, including every previously-paid Club plugin (SplitText, MorphSVG, ScrollTrigger, DrawSVG, etc.), for commercial use, after Webflow's acquisition of GreenSock** (announced 30 Apr 2025, confirmed current in 2026 sources). License is GreenSock's own "no-charge" license — not OSI/MIT — commercial use permitted, but you can't resell GSAP itself or build a competing animation library on it. | gsap.com/pricing, github.com/greensock/gsap, webflow.com/blog/gsap-becomes-free | GSAP (plus native CSS/WAAPI) is our default animation layer inside HyperFrames compositions: it is now unambiguously free, extremely capable for kinetic typography/SplitText/easing, and is one of HyperFrames' native adapters, so no extra integration work is required. |
| **FFmpeg** | Remains the universal substrate for decoding/encoding, audio mixing/ducking, LUT/color filters, concatenation, format/codec verification (`ffprobe`), and is required by both HyperFrames and Remotion renderers as well as the Vibha effects skill. | ffmpeg.org, all three reference repos | FFmpeg + `ffprobe` is our mandatory "ground truth" layer for ingest analysis *and* final output verification (duration, resolution, fps, stream presence) — this is how we satisfy the brief's "never claim a render succeeded without verifying the output" requirement. |
| **Speech-to-text / word timestamps** | `faster-whisper` ≥1.0 gives native word-level timestamps via cross-attention + DTW (fast, CPU/GPU, no extra model), but accuracy is "good, not perfect," especially on `base`/`small` models. **WhisperX** layers a `wav2vec2` forced-alignment pass + `pyannote` diarization on top of faster-whisper for materially tighter word boundaries and *speaker labels* — valuable for interview/multi-speaker footage. Apple's on-device Speech framework (via "Apple Vision"/Speech APIs, macOS-only) is what the Sweet Motion guide actually recommends for person segmentation (not transcription) on Mac. | faster-whisper docs, whisperX GitHub, vexascribe.com/whisperx comparison, the Notion guide itself | Default to `faster-whisper` (`small`/`medium`/`large-v3` depending on available compute) for transcription+word timestamps everywhere; use WhisperX's alignment pass opt-in when precise karaoke-style word highlighting or multi-speaker diarization is required; never hand-type or hallucinate timestamps. |
| **Scene / shot detection** | `PySceneDetect` (BSD-3-Clause, actively released, v0.7.2 Oct 2026) provides `ContentDetector` (HSV-based fast cuts), `AdaptiveDetector` (robust to camera motion/handheld footage), `ThresholdDetector` (fades), `HistogramDetector`, `HashDetector`, plus one-line FFmpeg-backed splitting (`split_video_ffmpeg`). | scenedetect.com, PyPI, GitHub | This is how we turn raw multi-shot footage (interviews, b-roll dumps, screen recordings) into a structured shot list *before* storyboard planning — replacing guesswork about "where are the cut points" with measured data. |
| **Person/subject segmentation & face tracking** | MediaPipe Selfie Segmenter (Apache-2.0) + OpenCV YuNet (MIT) — both small, fast, CPU-friendly, already vendored and proven inside the Vibha skill — are a safe, license-clean, offline choice for "cut the presenter out of the background" and "track the face for crop/zoom/overlay placement," vs. Apple Vision (macOS-only, not cross-platform) or `rembg` (U^2-Net based, heavier, GPU-friendlier, cross-platform fallback for non-Mac/no-mediapipe environments). | MediaPipe docs, OpenCV Zoo, rembg GitHub, Sweet Motion guide | We standardize on **MediaPipe + YuNet as the default**, `rembg` as the pure-Python cross-platform fallback when MediaPipe isn't installable, and Apple Vision as an optional macOS-native fast path — mirroring exactly what both references independently converged on. |
| **Audio analysis / music sync / beat detection** | `librosa` (onset/beat/tempo detection, BSD-licensed) is the standard, maintained Python library for finding beats to cut on, detecting silence/loudness for dead-air trimming, and computing RMS for auto-ducking dialogue under music. FFmpeg's `silencedetect`/`loudnorm` filters complement this for silence trimming and loudness normalization (EBU R128). | librosa docs, ffmpeg filters docs | Used in the "remove dead air," "cut on the beat," and "duck music under dialogue" decision rules. |
| **Stock media & SFX sourcing** | For B-roll/SFX that the user hasn't supplied, licensed sources (Pexels/Pixabay-style stock APIs already available to this environment via `fetch_media`, or user-approved external APIs) must always be *contextually matched* and clearly logged as external assets (never silently substituted for something unrelated), honoring the brief's explicit warning against random stock insertion. | — | Encoded as a hard rule in the B-roll reference doc: every non-user-supplied asset must be logged with its source, search query, and why it was chosen, in the storyboard/QC report. |

**Net technology recommendation**: FFmpeg + ffprobe (ingest & verify) → PySceneDetect (shot boundaries) → faster-whisper/WhisperX (transcript + word timestamps, optional diarization) → MediaPipe/YuNet (+ rembg fallback) for subject segmentation/tracking → **HyperFrames** (HTML/CSS + GSAP/CSS/Lottie animation, Apache-2.0, agent-native) as the default compositor/renderer, with **Remotion** supported as an opt-in alternative for users already in a React/Remotion shop who qualify for or hold its license → librosa for audio/beat analysis → FFmpeg again for final mux, loudness normalization, and encode. All are current, maintained, and either fully open (MIT/BSD/Apache) or explicitly free for our use case (GSAP), and none require a paid API key by default (optional paid APIs — better ASR, stock libraries, upscaling — are treated as opt-in enhancements behind environment-variable detection, never hard requirements).

---

## C. Proposed Architecture

```
video-editor/                         (the Claude Code Skill directory)
├── SKILL.md                          # lean entry point (<5k tokens): triggers, mental model,
│                                      #   the 6-stage workflow, pointers into references/, hard rules
├── LICENSE                           # MIT for our original code
├── NOTICE.md                         # attributions: MediaPipe (Apache-2.0), YuNet (MIT),
│                                      #   fonts (OFL/Apache), GSAP (GreenSock no-charge), etc.
├── references/                       # progressive-disclosure knowledge base (loaded on demand)
│   ├── 00-editorial-decision-engine.md   # THE core doc — how to think like an editor (Section D below)
│   ├── 01-storyboard-workflow.md         # the 6-stage pipeline, storyboard schema, approval gate
│   ├── 02-camera-and-pacing.md           # punch-ins, pans, speed, freeze, pacing/rhythm rules
│   ├── 03-broll-and-compositions.md      # b-roll sourcing, PIP/split-screen, screen recordings
│   ├── 04-motion-graphics-catalog.md     # curated, *generalized* effect families (not 45 brand gimmicks)
│   ├── 05-typography-and-captions.md     # caption schema, kinetic type patterns, safe areas
│   ├── 06-audio-and-sound-design.md      # music, SFX, ducking, silence, loudness targets
│   ├── 07-color-and-finishing.md         # grading, shot matching, encode/export settings
│   ├── 08-editing-profiles.md            # the adaptive style profiles (Section 8 of brief)
│   └── 09-quality-control-checklist.md   # the objective QC rubric, pass/fail + diagnostic scoring
├── assets/
│   ├── storyboard-template.csv / .md     # reusable beat-by-beat table template
│   ├── project-analysis-template.md      # Stage 1/2 report template
│   ├── qc-report-template.md             # Stage 6 report template
│   └── design-tokens.example.json        # starter type scale / color / spacing tokens an edit can inherit
├── scripts/                          # thin, composable, individually-testable utilities
│   ├── setup_check.py                 # detect ffmpeg/python/node/hyperframes/whisper/mediapipe; report, don't assume
│   ├── probe_media.py                 # ffprobe wrapper → JSON (duration, res, fps, streams, loudness)
│   ├── transcribe.py                  # faster-whisper → word-level JSON transcript (+ optional WhisperX align/diarize)
│   ├── detect_scenes.py               # PySceneDetect wrapper → shot list JSON
│   ├── segment_subject.py             # MediaPipe/rembg wrapper → per-frame/track cutout + face bbox track
│   ├── analyze_audio.py               # librosa wrapper → beats, silences, RMS envelope JSON
│   ├── build_storyboard.py            # merges transcript+scenes+audio analysis into a draft storyboard skeleton
│   ├── render_hyperframes.sh          # thin CLI wrapper: preview stills → full render → verify
│   └── verify_output.py               # ffprobe-based post-render QC (duration match, streams present, no zero-byte, caption/text overflow heuristic)
└── examples/
    ├── talking-head-reel/              # worked example: storyboard.md + composition + qc-report.md
    ├── tutorial-explainer/             # worked example with UI callouts + animated diagram
    └── before-after-promo/             # worked example with split-screen + typography-led promo
```

**Why this shape:**
- **One lean `SKILL.md`** so the *always-loaded* metadata (~100 tokens) and *on-trigger* body (<5k tokens) stay cheap; all deep reference material (the actual decision engine, the 45-effect-style catalog successor, QC rubric) is pulled in only when Claude actually needs it (matches the official three-level progressive-disclosure model, and avoids the brief's warning against "one giant SKILL.md").
- **Scripts produce JSON/Markdown artifacts, not prose** — their *stdout* is what enters Claude's context, keeping token usage proportional to what's actually needed (per Claude Code skills best practice: "scripts run through bash, only output enters context").
- **HyperFrames is the default render target** because it is literally designed to be authored by a coding agent (plain HTML, no build step, deterministic, Apache-2.0, first-party Claude Code plugin), which removes an entire class of "agent writes JSX, bundler fails, agent can't see why" failure modes that plague React-based pipelines. Remotion remains fully documented as an equally valid alternate renderer (same editing *decisions*, different *implementation* layer) for teams that prefer/require it and qualify under its license.
- **Analysis scripts are decoupled from the renderer.** Transcription, scene detection, subject segmentation and audio analysis all emit renderer-agnostic JSON. This means the "thinking" (storyboard, decision engine) never has to change if we later add a third renderer, and it lets us swap faster-whisper↔WhisperX or MediaPipe↔rembg per-environment without touching the creative layer.
- **Dependency detection is explicit and honest** (`setup_check.py`): every optional capability (GPU Whisper, MediaPipe, HyperFrames vs Remotion, 4K) is probed, reported, and gracefully degraded — the skill must never silently pretend a capability exists.

---

## D. Intelligent Editing Decision Framework

The core deliverable here is `references/00-editorial-decision-engine.md`: not a lookup table of "trigger word → effect," but a **layered reasoning process** Claude runs per beat. The levels, in order:

1. **What is this beat *for*?** (hook / claim / proof / explanation / transition / emotional beat / payoff / CTA). The narrative function constrains what's even appropriate — e.g. a payoff beat earns a visual "exhale" (clean shot, let it land), while an explanation beat earns clarifying graphics.
2. **What does the footage already give us?** (framing quality, is the subject static or moving, is there a natural cutaway, is dialogue confident/energetic or tentative, is there dead air, is a concrete noun/number/comparison spoken that could be visualized).
3. **What problem — if any — needs solving here?** Only intervene when there's an actual gap: unclear info → diagram; flat delivery on an important line → punch-in; a referenced object/tool/place the audience can't see → B-roll/screenshot; repetition/filler → cut; monotonous unbroken pacing for >~8s → *some* visual variation, but not necessarily an "effect."
4. **What's the smallest effective intervention?** Default to the least visually loud solution that solves the problem: a silent jump-cut beats a transition; a static clean frame beats an unnecessary pan; one well-timed caption emphasis beats a wall of animated text. Escalate intensity only when the content's energy and platform justify it (see editing profiles).
5. **Does this repeat a treatment we just used?** Track "treatment memory" across the storyboard — intentionally vary punch-ins/B-roll/graphic styles so five beats in a row don't use the same trick, which is how automated edits start to feel like a template rather than an edit.
6. **Would removing this effect make the video worse, the same, or better?** This is the mandatory "devil's advocate" pass per beat before finalizing the storyboard — if an effect doesn't clearly win against "no effect," cut it.

**Concrete decision examples (illustrative, not exhaustive — this is the pattern, not the whole rulebook):**

- *Speaker says a specific statistic ("we grew 240% in six weeks")* → Signal: concrete number + emphasis in delivery (slight pause/stress before or after). Decision: short counter/graph motion graphic timed to land exactly as the number is spoken, OR (if the overall profile is minimalist/B2B) just a clean text-on-screen restating the number with no chart, no counter. The decision between these two is driven by the *editing profile* (Section 8), not a hard rule.
- *Speaker references a tool/app/product by name for an instructional video* → Signal: named entity + likely has UI/screenshot available or sourceable. Decision: tool-label lower-third (logo + name, Sweet Motion pattern) for a passing mention; a full screen-recording cutaway if the speaker spends >3s explaining *how* to use it.
- *3+ seconds of silence or a filler repetition ("um, so, like I said, basically...")* → Signal: dead air / redundant phrasing detected from transcript + audio silence analysis. Decision: cut it. Default action is removal, not decoration — padding dead air with an effect is explicitly the wrong move.
- *An important, emotionally weighted statement delivered in a single steady shot* → Signal: audio emphasis (volume/pitch change) or a transcript cue like a personal story's turning point. Decision: a **slow, restrained punch-in** (not a fast whip-zoom) timed to complete exactly as the key phrase lands, usually with no added text — letting the moment breathe is itself the "effect."
- *A process/workflow is described step by step* → Signal: sequential language ("first... then... finally") or an enumerable list. Decision: an animated step list/checklist/numbered diagram synced to each "then," so the structure is visible, not just heard.
- *A comparison is made ("before, X took 3 hours; now it takes 3 minutes")* → Signal: explicit contrast language. Decision: before/after split-screen or swipe transition, or a two-column stat card — never both "swipe" and "split-screen" and "counter" stacked on the same beat (treatment-memory rule).
- *A long, well-lit, well-composed single shot where the speaker is already engaging* → Signal: no actual visual/narrative gap exists. Decision: **do nothing** — preserve the clean shot. This is one of the brief's explicit tests ("a clip where adding more effects would make the result worse") and the decision engine must be comfortable outputting "no treatment" as a first-class answer.
- *Footage has poor lighting / limited B-roll available* → Signal: technical constraint detected during Stage 2 analysis. Decision: lean harder into typography/motion graphics and color grading to add production value instead of forcing in irrelevant stock B-roll; document the constraint and the chosen workaround explicitly in the storyboard rationale column.

---

## E. End-to-End Workflow

1. **Inspect the Project** — enumerate source files (`probe_media.py` via ffprobe) for every clip: duration, resolution, fps, aspect ratio, audio stream presence/channels; scan for existing project assets, fonts, brand files, prior edits; run `setup_check.py` to see what's actually installed (HyperFrames? Remotion? GPU? MediaPipe? faster-whisper model size feasible?); capture the user's stated goal, platform, audience, and tone in a short brief. Never overwrite originals — all work happens in a `project/` working copy with content-addressed or timestamped output filenames.
2. **Analyze the Footage** — `transcribe.py` (word-level transcript + confidence), `detect_scenes.py` (shot boundaries for multi-shot sources), `segment_subject.py` (face/person tracks where a presenter is on camera), `analyze_audio.py` (silences, loudness, beats). Every output is saved as JSON next to the source and is explicitly labeled "observed" — nothing here is invented.
3. **Create the Beat-by-Beat Storyboard** — `build_storyboard.py` merges the above into a draft table (the schema from the brief: time range, spoken line/beat, narrative purpose, visual treatment, editing technique, motion graphics, typography/captions, sound design, implementation, rationale), then Claude *reasons over* that draft using the decision engine to fill in/adjust treatment, purpose, and rationale columns — this is a Claude reasoning step, not a script. Estimated/unconfirmed timings are explicitly flagged. **The storyboard is presented to the user as a markdown table before any rendering begins** (direct adoption of the Sweet Motion "approve before render" gate), *unless* the user's instruction already grants full autonomy (e.g., "use your best judgment, just send me the result") — see Section F for how we resolve that tension by default.
4. **Develop the Visual Direction** — Claude selects (or blends) one or more editing profiles (Section 8), picks a type scale/color/motion "voice," and records it as a short design-tokens block so every beat pulls from the same palette/fonts/easing rather than improvising per-shot.
5. **Implement the Edit** — translate storyboard rows into a HyperFrames (default) or Remotion composition: cut list, overlay components from the motion-graphics reference + registry, captions from the transcript, audio timeline (music/SFX/ducking).
6. **Render, Inspect, Improve** — render a handful of representative still frames at key timestamps first (direct adoption of the Vibha "check frames before full render" step) and inspect them for text-over-face, overflow, unreadable contrast, off-safe-area placement; only then render the full video; run `verify_output.py` (ffprobe-based) to confirm duration/resolution/streams match expectations and the file is a valid, playable MP4; run the QC checklist (Section 11 of the brief) against the rendered result; iterate on specific flagged beats rather than re-doing the whole edit; deliver the file plus a short QC report summarizing what was checked, what passed, and any known limitations.

---

## F. Creative Autonomy

The skill is deliberately written as **expertise, not a cage**:
- `references/00-editorial-decision-engine.md` encodes *reasoning patterns and trade-offs*, phrased as "signals to look for" and "questions to ask yourself," not as `if transcript contains X then apply effect Y` pseudocode.
- `references/04-motion-graphics-catalog.md` documents effect *families* (kinetic type, data visualization, callouts/annotations, compositing/PIP, camera-simulated movement, transitions) with their underlying technique (GSAP timeline pattern, HyperFrames registry block, or hand-built composition) rather than a fixed list of 45 brand-specific templates — Claude is explicitly told it may invent a new treatment, combine two reference patterns, reach for a different registry block, or hand-roll a bespoke GSAP timeline when the footage calls for something the catalog doesn't cover, and is told to prefer that over forcing the footage into an ill-fitting preset.
- The storyboard review gate defaults to **"decide and proceed"**: Claude fills in every creative decision itself using the engine and only surfaces the storyboard for a quick skim/approval when the user's framing implies they want to co-direct (e.g., they reference a specific creative wish) — but if the user has clearly granted full autonomy ("use your best judgment"), Claude proceeds straight through analysis → storyboard → render → QC and delivers the finished video plus a short rationale summary, rather than stalling on a plan the user didn't ask to review. Ambiguity is resolved with a sensible default and a one-line note in the QC report, not a blocking question — the brief's "ask only when genuinely necessary" rule is implemented as an explicit decision point in the workflow doc.
- The "devil's advocate" pass (Section D, point 6) and the treatment-memory rule exist specifically so creative freedom doesn't drift into either (a) rigid template-following or (b) undisciplined maximalism — both failure modes the brief explicitly warns against.

---

## G. Implementation Roadmap

**Phase 0 (this document) — Research & planning. Awaiting approval.**

**Phase 1 — Core skill skeleton & analysis scripts (essential).**
`SKILL.md`, `setup_check.py`, `probe_media.py`, `transcribe.py`, `detect_scenes.py`, `verify_output.py`, storyboard template, the editorial decision engine doc, the QC checklist doc. *Risk*: environment may lack ffmpeg/Python/whisper preinstalled — mitigated by `setup_check.py` reporting precisely what's missing and how to install it, never silently failing.

**Phase 2 — Subject tracking & HyperFrames rendering path (essential).**
`segment_subject.py` (MediaPipe + YuNet, rembg fallback), HyperFrames composition patterns for captions/kinetic-type/B-roll/split-screen, `render_hyperframes.sh`, the check-frames-before-full-render loop. *Risk*: HyperFrames CLI/plugin availability inside a given sandbox is unverified until we actually install and run it — must be confirmed empirically in Phase 2, with Remotion documented as the fallback path if HyperFrames cannot be installed in a given environment.

**Phase 3 — Audio/music/SFX + full motion-graphics catalog + editing profiles (essential for "ultimate" scope).**
`analyze_audio.py`, sound-design reference doc, the adaptive editing-profiles doc, the expanded motion-graphics catalog (data viz, callouts, compositing, transitions).

**Phase 4 — Worked examples + QC automation + validation report (essential before calling it done).**
Three fully worked examples end-to-end (talking-head reel, tutorial/explainer, promo), `verify_output.py` hardened, a written validation report distinguishing what was actually executed/rendered vs. documented-only, and a limitations/future-work report.

**Phase 5 (optional/stretch, only if time and value justify it).**
WhisperX diarization for multi-speaker interviews; Remotion-parity example (same storyboard rendered via both engines) to prove the renderer-agnostic design; a small eval harness that runs the seven brief-specified test scenarios and records pass/fail notes.

**Testing requirements**: every script must be run against at least one real (synthetic/stock, since the user's own footage is unavailable) sample clip during Phase 1–4, with actual command output captured — not just "the code looks right." The final deliverable includes a validation report stating exactly which scripts were executed, against what input, and what the verified output was (duration/resolution/stream checks from `ffprobe`, not visual inspection claims we can't back up in a non-interactive sandbox where we cannot literally watch a video).

**Essential vs. optional**: Phases 1–4 are the "ultimate skill" MVP the brief requires. Phase 5 items are valuable but not required for the skill to be genuinely useful and complete.

---

## H. Evaluation Criteria

The finished skill will be judged against:
1. **Structural correctness** — valid `SKILL.md` frontmatter per the Agent Skills spec; correct progressive disclosure; no single file bloats context unnecessarily.
2. **Actual executability** — `setup_check.py`, `probe_media.py`, `transcribe.py`, `detect_scenes.py`, `verify_output.py` must run successfully against at least one real sample video/audio file during our own build, with captured output, not just "should work" code.
3. **Decision quality on the seven brief-specified scenarios** (Section 13) — judged qualitatively: does the storyboard produced for each scenario make defensible, non-generic editorial choices, including correctly choosing *not* to add effects where that's the right call?
4. **Honesty of claims** — the validation report must clearly separate "tested and verified," "implemented but not executed end-to-end (e.g., needs GPU/Mac-only dependency unavailable in this sandbox)," and "documented pattern, not yet automated."
5. **Improvement over a bare prompt** — the comparison isn't "does it look cool," it's "does a user who only says 'use your best judgement' get a materially better, more deliberate, more justified edit than they would from asking an unassisted Claude to 'edit my video,' because of the storyboard discipline, decision engine, and QC gate this skill adds."
6. **Non-destructive, verifiable technical reliability** — originals untouched, output file existence/duration/stream integrity verified via `ffprobe`, no silent fallback that pretends to be the requested effect.

---

## I. Questions and Missing Inputs

1. **User footage/reference doc** — confirmed unavailable; proceeding with synthetic/stock sample clips for our own testing and multiple adaptive profiles instead of one house style. If you later get me even one short sample clip, I will use it to calibrate the default profile more specifically to your taste.
2. **Primary install/runtime environment** — should I assume the skill will mainly run inside **Claude Code CLI on the user's own machine** (so it can install `ffmpeg`/Python packages/HyperFrames itself, as both reference skills do), rather than inside this sandbox's Next.js/Postgres web app? (My working assumption, given your brief, is "yes, standalone Claude Code skill for local use," and I will deliver it as a self-contained skill directory inside this project for you to copy into `~/.claude/skills/` or `.claude/skills/`, without altering the existing Next.js scaffold.) Please confirm or correct.
3. **Renderer preference** — I'm defaulting to **HyperFrames as the primary renderer** (Apache-2.0, agent-native, no licensing ceiling) with Remotion fully documented as an equal alternative, instead of defaulting to Remotion as your brief's phrasing might imply. Flag now if you specifically want Remotion-first (e.g., because you already have a Remotion project/team), and I'll flip the default and keep HyperFrames as the alternative.
4. **Paid API appetite** — do you want optional hooks for paid upgrades (e.g., a hosted ASR API more accurate than local Whisper, a premium stock-footage API, an upscaler) behind environment-variable detection, or should the skill stay 100% local/open-source tools only, by default with no paid dependencies at all?
5. **Primary target platform(s)** for the default editing profile when none is specified — vertical short-form (Reels/TikTok/Shorts) is my assumed default given the brief's closing example ("premium, highly engaging reel for my personal brand"); confirm this should be the out-of-the-box default profile, with the others (explainer, B2B, documentary, etc.) available as alternates the user or Claude can switch into.

---

## J. Approval Gate

**Proposed approach, in one paragraph:** Build a standalone, installable Claude Code Skill (`video-editor/`) that encodes an editorial decision *engine* (not a rigid effect list) across modular reference docs loaded only on demand, backed by a renderer-agnostic analysis layer (FFmpeg/ffprobe, PySceneDetect, faster-whisper/WhisperX, MediaPipe+YuNet/rembg, librosa) feeding a mandatory storyboard-first workflow adapted from the Sweet Motion guide, implemented by default through **HyperFrames** (Apache-2.0, agent-native HTML/CSS/GSAP renderer) with **Remotion** fully supported as an alternative, finished with a check-frames-before-full-render discipline and an `ffprobe`-verified QC gate adapted from the Vibha effects skill's best engineering idea — and ship it with real worked examples and an honest validation report, not just documentation.

I have **not** built any skill files yet. Please confirm:
- (1) this architecture (HyperFrames-default, Remotion-alternative; standalone skill delivered inside this project for you to install elsewhere),
- (2) the default target profile (premium short-form personal-brand reel) when no profile is specified, and
- (3) your answers (or "use your best judgment") on the five questions in Section I —

and I will proceed straight to full implementation, build real sample test footage, execute and verify the scripts, and deliver the complete skill with a validation report.
