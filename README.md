# MiniMax H3 Turbo LoRA — evaluation grid

Side-by-side comparison of the [MiniMax H3](https://www.minimax.io/blog/minimax-h3) base model
against the [Turbo LoRA](https://huggingface.co/larryvrh/MiniMax-H3-Turbo-Lora) by larryvrh
([ComfyUI conversions](https://huggingface.co/drbaph/MiniMax-H3-Turbo-Lora-ComfyUI) by drbaph).

10 scenarios x 4 configs, 5.17-second 960x544 clips with native audio, all generated locally
on an RTX 5080 (16 GB) via vanilla ComfyUI nodes (`LoraLoaderModelOnly` + `MiniMaxH3SigmaShift`),
same seed and prompt per row.

| Config | LoRA | Steps | Audio shift |
|---|---|---|---|
| baseline | none | 20 | default (3) |
| ckpt500 | non-EMA ckpt500, strength 1.0 | 8 | 6 |
| ema-ckpt500 | EMA ckpt500, strength 1.0 | 8 | 6 |
| ema-ckpt500 | EMA ckpt500, strength 1.0 | 10 | 6 |

View the grid: https://jo-nike.github.io/h3-turbo-eval/
