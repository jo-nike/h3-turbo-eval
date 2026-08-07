# MiniMax H3 — Turbo LoRA comparisons

Side-by-side evaluation of [MiniMax H3](https://www.minimax.io/blog/minimax-h3) against two
independent Turbo distillations, rendered locally on an RTX 5080 (16 GB) with vanilla ComfyUI
nodes.

**View the comparisons: https://jo-nike.github.io/h3-turbo-eval/**

## Round 4 (current)

12 configurations × 10 stress-test scenes = 120 clips, 5.17 s each at 960×544 with native
audio, same seed (424242) and prompt in every cell. One page per scene puts all 12 side by
side; the A/B tool overlays any two with a draggable wipe and switches which one you hear.

| tag | LoRA | strength | steps | sampler | sigma shift |
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

Scenes stress singing, overlapping dialogue, a mid-sentence language switch, an off-screen
sound source with nothing on camera, near-silence into a hard transient, four hard cuts,
mirror reflections, close-up hands, metronomic timing, and a dense soundscape with one voice
that must stay intelligible.

### What the measurements show

Averaged over all ten scenes, each config against the baseline render of the same scene:

| config | time | Δluma | jitter × | detail × | Δpeak | Δfloor |
|---|---|---|---|---|---|---|
| base20 | 166s | — | 1.0 | 1.00 | — | — |
| emck8 | 77s | +1.5 | 1.1 | 0.97 | +1.2 | −5.6 |
| ck8 | 77s | +1.5 | 1.4 | 1.63 | +2.6 | +0.7 |
| lx4-ref | 45s | +11.4 | 6.0 | 2.22 | +4.7 | −1.3 |
| lx4-house | 45s | +11.7 | 6.1 | 2.17 | +3.2 | −7.7 |
| lx4-ersde10 | 45s | +5.7 | 4.0 | 1.22 | +2.2 | −7.1 |
| lx4-ref75 | 45s | +8.2 | 2.6 | 1.95 | +3.8 | −4.4 |
| lx4-75house | 45s | +8.3 | 2.4 | 1.89 | +3.7 | −10.1 |
| lx4-kijai | 45s | +3.8 | 2.0 | 1.06 | +1.4 | −7.7 |
| lx8-ref | 77s | +8.2 | 5.5 | 1.51 | +2.6 | −1.1 |
| lx8-house | 76s | +8.1 | 5.8 | 1.50 | +2.4 | −5.1 |
| lx8-kijai | 76s | +10.0 | 2.9 | 1.26 | +0.8 | −4.7 |

- **larryvrh's EMA checkpoint at 8 steps tracks the base model closely** — within +1.5 luma,
  1.1× jitter and 0.97× detail, with a 5.6 dB quieter noise floor, at 2.2× the speed.
- **EMA vs non-EMA is still the decisive axis.** Non-EMA sits 6.3 dB above EMA on the noise
  floor with 1.63× detail from the added grain.
- **lightx2v v0.1 needs strength 0.75 and er_sde.** At strength 1.0 it runs 6× baseline jitter
  and 2.2× detail (grain, not sharpness) with a large brightness lift. At 0.75 with er_sde —
  the author's own recommendation — it lands at +3.8 luma and 1.06× detail in 45 s, the fastest
  usable config here. It is a preview release and the authors flag image detail as a known
  limitation.
- **8 steps is worse than 4 for lightx2v**, which is distilled for 4.
- `detail` is Laplacian variance, which noise inflates as readily as sharpness — high values
  next to high jitter mean grain.

## Round 3 (archive)

The [earlier grid](https://jo-nike.github.io/h3-turbo-eval/round3.html) (10 scenes × 4
configs) was rendered before ComfyUI commit `bdcb886a`, which reworked how the MiniMax audio
stream is carried through the sampler. Its audio measurements are not directly comparable
with round 4.
