# LTX-2.5 First Frame + Last Frame to Video (FLF2V)

A ComfyUI workflow for **LTX-2.5** that adds first-frame/last-frame keyframe conditioning to the stock image-to-video template. The official 2.5 I2V workflow only conditions on a starting image and lets the model improvise where the clip ends up; this version also locks in an ending image, so the generated motion travels from your chosen start frame to your chosen end frame instead of drifting freely.

Built on top of Comfy's own LTX-2.5 template nodes — no custom node packs required.

## What it does

- Takes a starting image **and** an ending image, plus a text prompt, and outputs a video that transitions between them.
- Uses `LTXVAddGuide` to inject both keyframes into the latent as fixed conditioning — first frame at `frame_idx: 0`, last frame at `frame_idx: -1` — updating the positive/negative conditioning as well as the latent itself, rather than just pasting the image into the latent with no attention hint.
- The two guides are chained (first → last) and reapplied on **both** the base pass and the refine/upscale pass, so the end-frame identity survives the upscale instead of getting diluted.
- Audio generation is untouched — this only changes video conditioning.

## Requirements

- ComfyUI with the LTX-2.5 nodes (`comfy-core`, no extra custom nodes)
- LTX-2.5 checkpoint/model files as used by the stock LTX-2.5 template

## Usage

1. Load the workflow into ComfyUI.
2. In **Load First Frame**, upload your starting image.
3. In **Load Last Frame**, upload your ending image.
4. Write your prompt describing the motion/transition between the two, and set duration/resolution as usual.
5. Queue the prompt.

## Notes / tuning

- `strength` on each `LTXVAddGuide` node controls how strongly that keyframe is enforced (`0.0` = no influence, `1.0` = full conditioning). This workflow ships with `0.7` on the base pass and `1.0` on the refine pass for both guides, matching the strengths the stock template used for the first frame alone — tune independently per frame if one end is overpowering the other.
- There's no built-in bypass toggle on `LTXVAddGuide`. To fall back to first-frame-only generation, either set the "last frame" guide nodes' `strength` to `0` or mute them (right-click → Bypass).
- `frame_idx: -1` targets the last frame of the latent regardless of duration, so you don't need to recalculate an index when you change clip length.
- Keep the guide order as first → last (not last → first) in each pass — conditioning is threaded through both guides sequentially, and reversing the order biases the chain toward whichever guide runs second.
- If the transition looks rushed or the end frame "snaps in" too early, lower the last-frame `strength` slightly before touching CFG or steps.

## Credit

Adapted from Comfy's stock LTX-2.5 image-to-video template, using `LTXVAddGuide` — the same keyframe-conditioning node used in Lightricks' official first-frame/last-frame example workflows — in place of the single-image `LTXVImgToVideoInplace` node.
