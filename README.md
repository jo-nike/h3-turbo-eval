# MiniMax H3 — Turbo LoRA comparisons

Side-by-side evaluation of [MiniMax H3](https://www.minimax.io/blog/minimax-h3) against two
independent Turbo distillations and a DiT-eval-skipping patch, rendered locally on an
RTX 5080 (16 GB) with vanilla ComfyUI nodes.

**View the comparisons: https://jo-nike.github.io/h3-turbo-eval/**

## Round 4 (current)

14 configurations × 10 stress-test scenes = 140 clips, 5.17 s each at 960×544 with native
audio, same seed (424242) and prompt in every cell. One page per scene puts all 14 side by
side; the A/B tool overlays any two with a draggable wipe and switches which one you hear.

Everything was rendered on a single ComfyUI commit —
[`344b4398`](https://github.com/comfyanonymous/ComfyUI/commit/344b43989e8c56b5bb4a66cf028c834192ab59dd)
(0.30.0, 2026-08-07), torch 2.12.1+cu130 — which is recorded in the footer of every page.
That matters more than usual here: see the round-3 note at the bottom.

| tag | LoRA / patch | strength | steps | sampler | sigma shift |
|---|---|---|---|---|---|
| base20 | none | — | 20 | res_multistep | 12 / 3 |
| emck8 | [larryvrh](https://huggingface.co/larryvrh/MiniMax-H3-Turbo-Lora) EMA ckpt500 ([drbaph conversion](https://huggingface.co/drbaph/MiniMax-H3-Turbo-Lora-ComfyUI)) | 1.0 | 8 | res_multistep | 12 / 6 |
| ck8 | larryvrh ckpt500 (non-EMA) | 1.0 | 8 | res_multistep | 12 / 6 |
| lx4-ref | [lightx2v](https://huggingface.co/lightx2v/Minimax-h3-Turbo) v0.1 ([Kijai conversion](https://huggingface.co/Kijai/MiniMax-H3_comfy)) | 1.0 | 4 | res_multistep | 12 / 3 |
| lx4-house | lightx2v v0.1 | 1.0 | 4 | res_multistep | 12 / 6 |
| lx4-ersde10 | lightx2v v0.1 | 1.0 | 4 | er_sde | 12 / 3 |
| lx4-ref75 | lightx2v v0.1 | 0.75 | 4 | res_multistep | 12 / 3 |
| lx4-75house | lightx2v v0.1 | 0.75 | 4 | res_multistep | 12 / 6 |
| lx4-kijai | lightx2v v0.1 | 0.75 | 4 | er_sde | 12 / 3 |
| lx8-ref | lightx2v v0.1 | 1.0 | 8 | res_multistep | 12 / 3 |
| lx8-house | lightx2v v0.1 | 1.0 | 8 | res_multistep | 12 / 6 |
| lx8-kijai | lightx2v v0.1 | 0.75 | 8 | er_sde | 12 / 3 |
| base20-spec | [Spectrum](https://github.com/xmarre/ComfyUI-Spectrum-MiniMax-H3) v0.1.9 (degree 1, warmup 1) | — | 20 | res_multistep | 12 / 3 |
| emck8-spec | larryvrh EMA ckpt500 **+** Spectrum v0.1.9 | 1.0 | 8 | res_multistep | 12 / 6 |

Scenes stress singing, overlapping dialogue, a mid-sentence language switch, an off-screen
sound source with nothing on camera, near-silence into a hard transient, four hard cuts,
mirror reflections, close-up hands, metronomic timing, and a dense soundscape with one voice
that must stay intelligible.

## A note on the "detail" metric

`detail` is the variance of a Laplacian — the standard blur/focus measure. It has a trap:
**it scales with image contrast**, so a brighter or punchier render scores higher for
identical structure. These configs differ in average luma by up to +26, which is more than
enough to invent a detail gap that does not exist.

The tables therefore report **contrast-normalised** detail and jitter (each frame normalised
to zero mean / unit standard deviation before measuring) as the primary figures, with raw
detail kept alongside for reference. The difference is not cosmetic. On the
`offscreen-source` scene, `lx4-ref` measures **3.81×** the baseline's raw detail but only
**1.12×** normalised — the rest was brightness. An earlier version of this page cited that
raw figure as evidence of grain; that was wrong.

Note also that neither figure distinguishes fine texture from noise. High detail next to
high jitter means grain, not sharpness.

## What the measurements show

Averaged over all ten scenes, each config against the baseline render of the same scene.
`njit ×` and `ndet ×` are the contrast-normalised ratios; `det ×` is raw, for comparison.

| config | time | Δluma | njit × | ndet × | det × | Δpeak | Δfloor |
|---|---|---|---|---|---|---|---|
| base20 | 166s | — | 1.0 | 1.00 | 1.00 | — | — |
| emck8 | 77s | +1.5 | 1.1 | 0.95 | 0.97 | +1.2 | −5.6 |
| ck8 | 77s | +1.5 | 1.3 | 1.50 | 1.63 | +2.6 | +0.7 |
| lx4-ref | 45s | +11.4 | 4.1 | 1.48 | 2.22 | +4.7 | −1.3 |
| lx4-house | 45s | +11.7 | 4.1 | 1.41 | 2.17 | +3.2 | −7.7 |
| lx4-ersde10 | 45s | +5.7 | 3.1 | 0.94 | 1.22 | +2.2 | −7.1 |
| lx4-ref75 | 45s | +8.2 | 2.1 | 1.42 | 1.95 | +3.8 | −4.4 |
| lx4-75house | 45s | +8.3 | 1.9 | 1.37 | 1.89 | +3.7 | −10.1 |
| lx4-kijai | 45s | +3.8 | 1.7 | 0.88 | 1.06 | +1.4 | −7.7 |
| lx8-ref | 77s | +8.2 | 3.9 | 1.04 | 1.51 | +2.6 | −1.1 |
| lx8-house | 76s | +8.1 | 4.1 | 1.02 | 1.50 | +2.4 | −5.1 |
| lx8-kijai | 76s | +10.0 | 2.2 | 0.94 | 1.26 | +0.8 | −4.7 |
| base20-spec | 107s | −4.7 | 1.0 | 0.90 | 0.75 | −0.7 | −2.9 |
| **emck8-spec** | **61s** | −4.5 | 1.3 | **1.03** | 0.80 | −1.4 | −3.0 |

- **larryvrh's EMA checkpoint at 8 steps tracks the base model closely** — 1.1× jitter,
  0.95× normalised detail, a 5.6 dB quieter noise floor, at 2.2× the speed.
- **Stacking Spectrum on it costs nothing measurable and saves 20%.** `emck8-spec` runs
  **61 s against the baseline's 166 s — 2.7×** — at 1.03× normalised detail. Its raw detail
  of 0.80 is entirely the contrast shift; there is no texture loss. This combination only
  became possible with Spectrum v0.1.9, whose new `warmup_steps 1` default leaves a
  forecastable window at 8 steps where the old `warmup_steps 5` left none.
- **Spectrum on the 20-step baseline does soften genuinely** — 0.90× normalised — but far
  less than its raw 0.75× implies.
- **EMA vs non-EMA is still the decisive axis for the older LoRA.** Non-EMA sits 6.3 dB
  above EMA on the noise floor with 1.50× normalised detail from the added grain.
- **lightx2v v0.1 needs strength 0.75 and er_sde.** At strength 1.0 it runs ~4× baseline
  jitter, and on locked-off shots — where the baseline barely moves — it reaches 8× and 19×
  normalised, i.e. visible crawl in scenes where nothing should move. At 0.75 with er_sde,
  the author's own recommendation, it lands at 1.7× jitter in 45 s, the fastest config here.
- **lightx2v is softer than the baseline, not grainier.** `lx4-kijai` measures 0.88×
  normalised detail — it genuinely loses fine texture. This matches ModelTC flagging image
  detail as v0.1's known limitation; it is a preview release.
- **8 steps is worse than 4 for lightx2v**, which is distilled for 4.

## Round 3 (archive)

The [earlier grid](https://jo-nike.github.io/h3-turbo-eval/round3.html) (10 scenes × 4
configs) was rendered before ComfyUI commit `bdcb886a`, which reworked how the MiniMax audio
stream is carried through the sampler. Re-rendering the identical graph and seed after that
commit produced the same scene with a **different take** (frame correlation 0.92–0.94), so
round-3 audio measurements are not comparable with round 4 and the two are kept separate.
