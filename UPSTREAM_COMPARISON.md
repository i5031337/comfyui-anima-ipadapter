# Fork versus parent architecture

This checkout is the `i5031337/comfyui-anima-ipadapter` fork of
[`Wenaka2004/comfyui-anima-ipadapter`](https://github.com/Wenaka2004/comfyui-anima-ipadapter).
The comparison is between the two `main` branches as checked on 2026-11-16:
parent `5f53e2b`, fork `ff92354`. The fork is two commits ahead; the parent has
no additional commits. The substantive change is
[`f605b15`](https://github.com/i5031337/comfyui-anima-ipadapter/commit/f605b154ef6a6a0fda1cd1e4ba07c67a8c99bdef);
[`ff92354`](https://github.com/i5031337/comfyui-anima-ipadapter/commit/ff92354a07bb1d8fb0465fe9faed609467126b6c)
only corrects the README clone URL. The combined diff changes 17 files, with
1,994 insertions and 1,286 deletions.

## What changed

| Area | Parent | Fork |
| --- | --- | --- |
| Reference image input | A separate `AnimaSiglipeEncodeImage` node produces `SIGLIP_FEATURES` for Apply. | Apply accepts a ComfyUI `IMAGE` and runs SigLIP2 internally. |
| Adapter model | `IPAdapterSigLIP` contains a timestep-aware `TimeResampler`, per-block `IPCrossAttn` modules, and learned per-block scales. | Checkpoint-derived key/value projections and `adaln_ip` gates are installed on DiT blocks. Optional checkpoint components include shared projections, a SigLIP compressor, IP self-attention, and cross-attention LoRA. |
| Injection | Wraps each block's `cross_attn.forward` and adds the adapter output there. | Captures cross-attention input and patches the block forward to add image attention after the original block result. |
| Controls | `weight`, `start_at`, and `end_at` inputs. | `strength`, reference-image size, SigLIP layer, image CFG controls, gray null image, and LoRA toggle. No exposed `start_at`/`end_at`. |
| ComfyUI nodes | Loader, Apply, SigLIP image encoder, and legacy embedding loader. | Loader, Apply, and visualization node. The node data type changes from `ANIMA_IPADAPTER` to `ANIMA_IP_ADAPTER`. |
| Other files | Includes adapter training scripts. | Removes those scripts; adds model download, expression sprite generation, an API workflow, and a list of 28 expressions. |

Both versions use **SigLIP2 in their current code**. The parent's README still
describes Qwen3-VL, so its README is not a reliable guide to that version's
current image-encoding path.

## Capability implications

The fork is a different checkpoint format and workflow, not just a refactor.
The parent loader expects keys such as `resampler.time_proj.weight` and
`ip_cross_attns.*`; the fork looks for `blocks.*.ip_k_proj`, `blocks.*.ip_v_proj`,
`adaln_ip`, or shared projections. Do not assume a parent checkpoint works in
the fork. The fork loader does not explicitly reject the parent's key layout;
its Apply path can initialize projection weights when expected keys are absent,
so a checkpoint that appears to load may still produce invalid conditioning.

Direct `IMAGE` input makes setup simpler, while independent image CFG, LoRA,
SigLIP layer selection, and null-token options offer more conditioning control.
The separate encoder and precomputed-feature connection are gone. Existing
parent workflows need rewiring, even where node names look similar.

The parent's resampler is designed to depend on the sampling timestep, but the
current Apply path creates tokens once at timestep `0.5`. An
`IPAdapterHookManager` defines an update method, yet Apply does not attach or
call it. Thus per-step token updates should not be treated as an established
parent inference capability without testing. The fork also encodes its image
tokens once per Apply call.

The fork's README claims GGUF compatibility and describes Anima Turbo use.
The source includes dtype handling and block patching intended to support that
path, but the diff alone does not establish that it works with every GGUF
loader or improves VRAM use. The sprite script and visualization node expand
the tooling around inference; they do not demonstrate better adapter quality.
The visualization calculates token and projected-key statistics, rather than
recording actual attention weights from a generation.

## Checks for the other machine

1. Use a checkpoint trained for each architecture. Record its source and
   state-dict key prefixes before comparing results.
2. Load an existing parent workflow in the fork and identify broken node types
   or connections. Rebuild it with direct `IMAGE` input, then check each new
   control individually.
3. Compare likeness, prompt and expression flexibility, generation time, and
   peak VRAM with the same base model, reference, prompts, seed, and sampler.
   Report checkpoint differences alongside the results.
4. Test the claimed GGUF path with the actual `UnetLoaderGGUFAdvanced` setup.
   Test `ip_cfg_separate` and `use_lora` on and off with a compatible checkpoint.
5. For sprite generation, verify the API workflow's required models and nodes.
   The README also links to `workflows/Anima_Turbo_IPAdapter.json`, which is not
   present in this checkout; the included workflow is
   `workflows/Expression_Sprites_API.json`.

## Code references

- Parent: [`nodes.py`](https://github.com/Wenaka2004/comfyui-anima-ipadapter/blob/5f53e2b10802d5715967806cbe88d761b018a121/nodes.py), [`ip_adapter_model.py`](https://github.com/Wenaka2004/comfyui-anima-ipadapter/blob/5f53e2b10802d5715967806cbe88d761b018a121/ip_adapter_model.py)
- Fork: [`loader.py`](loader.py), [`apply.py`](apply.py), [`patcher.py`](patcher.py), [`visualize.py`](visualize.py), [`generate_expression_sprites.py`](generate_expression_sprites.py)
