# MiniMax H3 — Turbo LoRA comparisons

Side-by-side evaluation of [MiniMax H3](https://www.minimax.io/blog/minimax-h3) against two
independent Turbo distillations, a DiT-eval-skipping patch and two attention backends, rendered locally on an
RTX 5080 (16 GB) with vanilla ComfyUI nodes.

**View the comparisons: https://jo-nike.github.io/h3-turbo-eval/**

## Round 4 (current)

19 configurations × 10 stress-test scenes = 190 clips, 5.17 s each at 960×544 with native
audio, same seed (424242) and prompt in every cell. One page per scene puts all 19 side by
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
| emck8-sage | larryvrh EMA ckpt500 **+** SageAttention 2.2.0 | 1.0 | 8 | res_multistep | 12 / 6 |
| emck8-sage-spec | larryvrh EMA ckpt500 **+** Sage **+** Spectrum | 1.0 | 8 | res_multistep | 12 / 6 |
| base20-sage | SageAttention 2.2.0 (KJNodes patch, backend `auto`) | — | 20 | res_multistep | 12 / 3 |
| base20-sol | [Sol-Attn](https://github.com/Saganaki22/ComfyUI-sol-attn) H3 zero-copy patch (tau 1.0) | — | 20 | res_multistep | 12 / 3 |
| emck850 | larryvrh **EMA ckpt850** (his recommended checkpoint) | 1.0 | 8 | res_multistep | 12 / 6 |

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
| emck8-spec | 61s | −4.5 | 1.3 | 1.03 | 0.80 | −1.4 | −3.0 |
| **emck8-sage** | **45s** | +1.5 | 1.1 | 0.94 | 0.97 | +0.9 | −5.1 |
| emck8-sage-spec | 41s | −4.1 | 1.4 | 0.96 | 0.79 | −0.8 | −3.8 |
| base20-sage | 98s | +1.2 | 1.0 | 0.98 | 1.00 | +0.2 | −0.5 |
| base20-sol | 99s | −3.1 | 1.4 | 1.09 | 1.00 | +0.1 | +0.1 |
| **emck850** | **77s** | +2.5 | 1.2 | 1.32 | 1.51 | +0.9 | −1.9 |

- **larryvrh's EMA checkpoint at 8 steps tracks the base model closely** — 1.1× jitter,
  0.95× normalised detail, a 5.6 dB quieter noise floor, at 2.2× the speed.
- **SageAttention is close to free, and it is where the speed actually is.** `emck8-sage`
  matches plain `emck8` on every axis (Δluma +1.5 vs +1.5, jitter 1.1 vs 1.1, detail 0.94 vs
  0.95) while running **45 s against the baseline's 166 s — 3.7×**. On a single scene measured
  against its own stock render it holds 0.955 frame correlation, i.e. the *same take*, not a
  similar one. SageAttention 2.2.0 was already installed and simply not switched on; the
  KJNodes `PathchSageAttentionKJ` patch enables it per-model without a launch flag.
- **Sage and Spectrum accelerate differently.** Sage preserves the take (0.955 / 0.974
  correlation); Spectrum changes it (0.789, with or without Sage — the drift is Spectrum's).
  Stacked, `emck8-sage-spec` is the fastest usable config at **41 s (4.1×)**, but it is a
  different take, so it belongs to drafts rather than finals.
- **Sol-Attn is not competitive here at its defaults.** `base20-sol` is level with Sage on
  speed (99 s vs 98 s) but runs 1.4× jitter against Sage's 1.0 and *raises* normalised detail
  to 1.09 — adding high-frequency content rather than resolving more. On a single scene it
  scored 0.723 correlation against stock, closer to changing the seed than to accelerating the
  same render. It is tunable (`tau`, `dense_blocks`, `dense_percent`), so this is a verdict on
  the defaults, not on the method.
- **ckpt850 is measurably different from ckpt500, and these metrics cannot say whether it is
  better.** `emck850` shows **1.32× normalised detail** against ckpt500's 0.95, with a floor
  3.7 dB less quiet. That profile sits between ckpt500 and the visibly grainy non-EMA `ck8`
  (1.50×). It is either meaningfully sharper or meaningfully grainier, and Laplacian variance
  cannot distinguish those — this one needs eyes.
- **EMA vs non-EMA is still the decisive axis for the older LoRA.** Non-EMA sits 6.3 dB
  above EMA on the noise floor with 1.50× normalised detail from the added grain.
- **lightx2v v0.1 needs strength 0.75 and er_sde.** At strength 1.0 it runs ~4× baseline
  jitter, and on locked-off shots — where the baseline barely moves — it reaches 8× and 19×
  normalised, i.e. visible crawl in scenes where nothing should move. At 0.75 with er_sde,
  the author's own recommendation, it lands at 1.7× jitter in 45 s.
- **lightx2v is softer than the baseline, not grainier.** `lx4-kijai` measures 0.88×
  normalised detail — it genuinely loses fine texture. This matches ModelTC flagging image
  detail as v0.1's known limitation; it is a preview release. At the same 45 s, `emck8-sage`
  is the better config on every measure.
- **8 steps is worse than 4 for lightx2v**, which is distilled for 4. The reverse is true of
  larryvrh's LoRA: 4 steps is underbaked there regardless of strength (1.0, 0.75), video sigma
  shift (12, 5, 3, 2) or checkpoint (ckpt500, ckpt850), because `res_multistep` needs step
  history that a 4-step schedule cannot provide. `er_sde` at 4 steps produces usable but
  degraded output. With Sage putting 8 steps at 45 s, 4 steps would save 1 s — there is no
  reason to chase it.
## Round 3 (archive)

The [earlier grid](https://jo-nike.github.io/h3-turbo-eval/round3.html) (10 scenes × 4
configs) was rendered before ComfyUI commit `bdcb886a`, which reworked how the MiniMax audio
stream is carried through the sampler. Re-rendering the identical graph and seed after that
commit produced the same scene with a **different take** (frame correlation 0.92–0.94), so
round-3 audio measurements are not comparable with round 4 and the two are kept separate.
