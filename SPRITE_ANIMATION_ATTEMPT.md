# Sprite animation attempt

The sprite animation experiment was retired on 2026-11-16 after the results
proved unsuccessful for the intended use.

We tried animating finished character sprites through a local ComfyUI workflow
using Wan 2.2 TI2V 5B, the UMT5 text encoder, and the Wan 2.2 VAE. The workflow
composited the source sprite onto a dark grey canvas, generated motion, then
used BiRefNet to remove the background and restore transparency.

The default configuration was 576 × 832, 49 frames at 24 fps, and 20 sampling
steps. A smaller 384 × 576, 25-frame configuration was suggested for previews.
Prompts covered idle breathing, blinking, talking, nodding, and waving, with
instructions to preserve the character, illustration style, and camera framing.

A Python helper supported individual images and directory batches, custom motion
prompts, and exports of animated WebP, PNG frames, sprite sheets, and metadata.
It offered forward playback or a forward/reverse loop without duplicated
endpoints, plus frame subsampling.

The approach did not deliver satisfactory sprite animation. Known limitations
included possible character-detail drift, ignored motion prompts, flickering
edges from per-frame background removal, and unnatural reversed gestures.
Forward/reverse playback joined the loop boundary but did not guarantee natural
motion. These limitations describe the approach; they are not a diagnosis of
every unsuccessful run.

The repository helper, API workflow, and README usage section were removed.
