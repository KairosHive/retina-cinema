<p align="center">
    <img src=https://github.com/user-attachments/assets/7907accf-39e7-464e-b647-d7435a873bda width=250>
</p>


# ReTiNA-Cinema: TouchDesigner Patches for Adaptive Neuro-Augmented Cinema

This repository contains the **TouchDesigner project** and **configuration files** used for the [ReTiNA-Cinema](https://github.com/KairosCollective/retina-cinema) system described in our paper:

> **Introducing ReTiNA-Cinema: Real-Time Neuro-Augmented Cinema via Generative AI**
> *(Paper link coming soon)*

**ReTiNA-Cinema** transforms movie visuals in real-time, using viewers' EEG brain signals to modulate AI-driven cinematic effects, supporting both single-user and multi-user (hyperscanning) modes.

---

## Instructions
1. Install [**TouchDesigner**](https://derivative.ca/download) and [**goofi-pipe**](https://github.com/dav0dea/goofi-pipe?tab=readme-ov-file#installation) by following the installation instructions at the respective links.
2. Download the repository files by cloning or downloading the ZIP.
3. Unlock and download the StreamDiffusion Node on dotsimulate's Patreon: [StreamDiffusion Node](https://www.patreon.com/posts/122151912?collection=565003).
4. Open the `retina-cinema.toe` file in TouchDesigner and drag the StreamDiffusion Node into the project.
5. Connect the StreamDiffusion node's first input to the MoviePlayer's TOP output, and the StramDiffusion output to the input of the SDOutput node as shown in the screenshot:
<p align="center">
    <img src=https://github.com/user-attachments/assets/781d77a0-fd64-4de7-aca9-9e0bbdbfbeea width="900">
</p>

6. Configure the parameters of the StreamDiffusion node as shown in the following screenshot:
<p align="center">
    <img src=https://github.com/user-attachments/assets/c40a39b0-c33a-42da-941e-876d8a13c90c width="500">
</p>

> [!NOTE]
> If you want to use your own custom prompts instead of context-aware and dynamically generated ones, enter your custom prompts in the four respective `Prompt` fields.

For improved temporal consistency of the generated video, also enable the V2V mode and set the Feature Injection Strength to 1.9.
<p align="center">
    <img src=https://github.com/user-attachments/assets/308f1e22-108c-48ad-b833-501df9269dfa width="500">
</p>

7. Start [goofi v3](https://github.com/dav0dea/goofi-pipe) and load the patch:
    ```
    goofi --load retina-cinema.gfi
    ```
    The patch bundles its custom nodes (`MovieFrame`, `ImgToText`, `InterBrain`) in its workspace, so no `--extra-nodes` flag is needed. The nodes import `Pillow` and `litellm`; install them into goofi's node interpreters once (see `nodes/requirements*.txt`), or start goofi with `--extra-nodes nodes` the first time so it offers to install them.
    One patch covers both modes (replacing the old `single-user.gfi` and `multi-user.gfi`). It sends `feat1` (EEG stream A, LZ complexity), `feat2` (stream B) and `blend` (inter-brain wPLI coupling) plus `style1`–`style4` as OSC to `127.0.0.1:8000` under `/goofi`. For a single user, only stream A is needed.
8. Dynamic prompt generation uses the `describe` node (`ImgToText`), which calls a vision LLM through [litellm](https://github.com/BerriAI/litellm). It defaults to `gpt-6-luna`; set `OPENAI_API_KEY` in the environment goofi starts from (or put the key in a file called `openai.key` in the working directory). Any other litellm model name works too. Prompt cues arrive over OSC on port 8001 (`/prompt_cue`), and the newest `MovieFrame.0.png` is read by the `frame` node (set its `file/path`).
9. By default both streams replay the same sample EEG recording. To use live EEG, set the `index` of `sourceA` / `sourceB` to 1 (LSL) and pick the LSL stream names in `lslA` / `lslB`. If your cap has channels that should be dropped (e.g. `Iz,T9,T10`), enter them in the `drop` field of `channelsA` / `channelsB`.
10. Start the StreamDiffusion model (in TouchDesigner).
11. Enjoy ReTiNA-Cinema!

---

## Included Files

* **`retina-cinema.toe`** The main TouchDesigner project file that implements the adaptive cinema system.
* **`retina-cinema.gfi`** goofi v3 patch: feature extraction from one or two EEG streams and dynamic prompt generation.
* **`nodes/`** Source of the custom goofi v3 nodes: `MovieFrame` (emits each newly saved movie frame), `ImgToText` (vision LLM via litellm) and `InterBrain` (mean of the between-participant block of the wPLI matrix). Copies of these are bundled inside `retina-cinema.gfi` under `workspace/nodes_signal/`.
* **`tools/build_patch.py`** Regenerates `retina-cinema.gfi` through the goofi CLI and bundles the custom nodes into it.

---

## Requirements

* [**TouchDesigner**](https://derivative.ca/download) for real-time video processing
* [**goofi-pipe**](https://github.com/dav0dea/goofi-pipe?tab=readme-ov-file#installation) for EEG feature extraction
* [**StreamDiffusion Node**](https://www.patreon.com/posts/122151912?collection=565003) for real-time AI visuals (by [dotsimulate](https://dotsimulate.com/))

---

## License

This project is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/).
