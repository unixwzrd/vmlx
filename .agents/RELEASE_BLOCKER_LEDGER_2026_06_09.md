# vMLX / MLXStudio release blocker ledger - 2026-06-09

Scope: active Python engine and Electron/panel app in `/Users/eric/mlx/vllm-mlx-finite-launch-guard`.

## 2026-06-09 18:00 PDT checkpoint DMGs

Eric explicitly overrode the red prepackage gate for a checkpoint build. Current-source vMLX 1.5.56 Sequoia/Tahoe DMGs were built, Developer ID signed, notarized, stapled, blockmap-regenerated, and post-staple verified from `/Users/eric/mlx/vllm-mlx-finite-launch-guard`.

- Sequoia: `panel/release/vMLX-1.5.56-sequoia-arm64.dmg`
  - SHA-256: `014ef3a9d729bf6b63091e28c82cfe86a9921397aa3d27621cab5f0e0541652f`
- Tahoe: `panel/release/vMLX-1.5.56-tahoe-arm64.dmg`
  - SHA-256: `272f9c9551fa99332b66c0a686083d94d0b2bf7c5359d310d4983d322dd01686`

Commands used:

```sh
security unlock-keychain -p vmlx-release ~/Library/Keychains/vmlx-build.keychain-db
security set-keychain-settings ~/Library/Keychains/vmlx-build.keychain-db
security set-key-partition-list -S apple-tool:,apple:,codesign: -s -k vmlx-release ~/Library/Keychains/vmlx-build.keychain-db
VMLINUX_CHECKPOINT_RELEASE_OVERRIDE=1 \
  VMLX_PREPACKAGE_READY_MANIFEST_OUT=build/current-release-regression-manifest-checkpoint-dmg-override-20260609.json \
  panel/scripts/build-release-dmgs.sh all
VMLINUX_NOTARY_KEYCHAIN=$HOME/Library/Keychains/vmlx-build.keychain-db panel/scripts/notarize-release-dmgs.sh
panel/scripts/verify-release-dmgs.sh
```

Final verify reported `Notarization Ticket=stapled`, `source=Notarized Developer ID`, `TeamIdentifier=55KGF2S5AY`, valid `hdiutil` checksums, and Developer ID signatures for both DMGs.

Boundary: the override manifest remains `status=fail`, `prepackage_ready=false`, and `release_ready=false`. This is a checkpoint artifact only unless Eric explicitly publishes it as such. Do not claim full production clearance for N2/MiMo/Gemma/media/tool/UI/cache/runtime rows from notarization alone. No tag, appcast/latest.json mutation, GitHub release publish, public download update, or PyPI publish was performed.

Hard boundary: no fake fixes, no forced sampling/parser/cache behavior to hide runtime/model bugs, and no signing/notarization/tag/download release until the runtime, cache, parser, UI, and installed-app rows are green or Eric explicitly overrides.

Reporter credit: include GitHub `@Hornsan1` in next release notes/changelog/public acknowledgement for reported runtime/model/UI/API issues.

## Active blockers and proof requirements

1. Responses streaming tool arguments

- Need raw SSE local direct vs gateway vs tunnel comparison on the same model and same request.
- Do not fix by disabling reasoning or reducing reasoning effort.
- Current synthetic/local engine paths can emit non-empty `response.function_call_arguments.delta` and `response.function_call_arguments.done`; next proof must use the reported model/request and deployed tunnel path.
- Specific issue to trace: streaming code near line `13592` checks `if tc_args:`; reported failure means `tc_args` is empty. Trace why `_parse_tool_calls_with_parser` returns empty args when reasoning is on, and trace how the streaming loop accumulates tool-call deltas.
- Gateway/tunnel/port/wake/sleep/session routing must be checked separately from engine parsing. A Cloudflare/gateway model miss or stale sleeping backend is not the same as a local parser bug.
- No executable `{}` argument payload should be emitted for required tool calls when required args are actually missing; fail closed instead.
- Current split: local no-heavy guards cover empty XML required-arg fail-closed behavior and output-index ordering. Gemma4 E2B direct/gateway captures preserve `record_fact` args and valid indices, but the public tunnel does not advertise `gemma4-e2b-sse`. Qwen35 public tunnel preserves args and reasoning events, but reuses `output_index=0` for both message and function_call. Treat Qwen35 as deployed/tunnel freshness unless current-source direct/gateway raw SSE reproduces the duplicate index.
- Current same-model Qwen35 direct proof: `build/current-responses-raw-sse-parity-qwen35-direct-source-vs-tunnel-20260609.json` captures current source against `models/Qwen3.6-35B-A3B-MXFP8-CRACK-MTP`; direct source uses `message=[0]`, `function_call=[1]`, preserves `{"value": "blue-cat"}`, and keeps reasoning events on. The artifact still fails because gateway is missing and the tunnel still duplicates `output_index=0`.
- Next proof: panel-gateway raw SSE with reasoning enabled and the exact Qwen35 tunnel request. If gateway uses output index `1`, rebuild/redeploy the tunnel backend and recapture; if it duplicates `0`, reopen the source streaming finalization path.

2. MiniMax random Chinese / visible planning

- Not cleared.
- Do not clamp sampling, inject sentinels, or hide visible planning as a "fix."
- Need isolate cache-on vs cache-off, TurboQuant KV vs none, L2 vs no L2, parser/template boundary, request/session reuse, and model-owned `generation_config.json`.
- Random Chinese or random-language output is likely cache continuation, prompt-contract contamination, parser/tool memory injection, L2 disk stale restore, or logit/decode-loop corruption until proven otherwise.
- Read prior docs/artifacts for similar cross-model cache/language/planning leaks before touching defaults.

3. MiMo V2.5

- JANGTQ_2 speed/cache is partially good, but release proof is not complete.
- JANG_2L source fresh-process block-disk L2 restore is green in
  `build/current-mimo-v25-jang2l-restart-l2-restore-20260610-rerun/summary.json`;
  tool/JSON/loop, Responses semantics, media, UI parity, installed-app tool/media,
  and release proof remain open.
- 2026-06-10 update: JANG_2L current dev-app Responses/tools rerun is still red
  in `build/current-real-ui-live-model-mimo-v25-jang2l-responses-tools-rerun-20260610.json`.
  It proves Responses transport, deltas, `previous_response_id` tool follow-up,
  paged cache, and block L2, but fails release assertions because no
  `long_tool_loop` surface was recorded and visible/tool semantics drifted
  `REAL_UI_LIVE_TOOL_ONE` to `REAL_UI_LAND_TOOL_ONE`.
- 2026-06-10 update: MiMo JANG_2L installed-app text/cache is now green in
  `build/current-real-ui-installed-app-mimo-v25-jang2l-text-cache-proof-20260610.json`.
  The local rebuilt `/Applications/vMLX.app` loaded the 105 GiB row, produced
  exact visible text turns, used native `mixed_swa_kv_v1` /
  `mimo_v2_asymmetric_swa`, hit paged cache, and wrote block L2. This does not
  clear MiMo installed-app tools, media, JANGTQ_2 exactness, speed, public
  tunnel SSE, or Developer ID DMG readiness.
- 2026-06-10 update: MiMo JANG_2L installed-app tools are now explicitly red in
  `build/current-real-ui-installed-app-mimo-v25-jang2l-tools-proof-20260610.json`.
  The installed app reached the real built-in tool surface and executed one
  `run_command` on the second turn, but failed `long_tool_loop`: the first
  marker mutated to `REAL_UI_LAND_TOOL_ONE`, the first probe file was missing,
  and visible output degraded into repetitive tool-planning prose. Cache/L2 was
  positive, so do not chase prefix/L2/TurboQuant as the primary blocker.
- 2026-06-10 update: MiMo JANG_2L installed-app image/media is now explicitly
  red in `build/current-real-ui-installed-app-mimo-v25-jang2l-image-proof-20260610.json`.
  The installed app attached an image and server `MEDIA_DIAG` saw `image_url`,
  but the runtime returned the honest guard `400 - unsupported media modality
  image because the loaded runtime is text-only; supported modalities: text`.
  Forced MLLM is intentionally overridden because media weights are preserved
  but unwired. Cache/L2 was positive, so do not claim media support or chase
  cache as the blocker.
- 2026-06-10 update: MiMo JANG_2L current dev-build video/media is now
  explicitly red in `build/current-real-ui-dev-app-mimo-v25-jang2l-video-proof-20260610.json`.
  The Electron dev app loaded the real 105 GiB bundle, completed two visible
  text turns, sent one video attachment, and server `MEDIA_DIAG` saw
  `video_url`; `/v1/chat/completions` returned `400`: `received unsupported
  media modality video because the loaded runtime is text-only. Supported
  modalities: text.` Runtime/cache stayed live before the guard with active
  memory `105016.1 MB`, peak `106152.4 MB`, native `mixed_swa_kv_v1` /
  `mimo_v2_asymmetric_swa`, `l2_block_tokens_on_disk=110`, and block-disk
  writes `3`. Do not claim MiMo JANG_2L video support from preserved media
  metadata.
- 2026-06-10 update: MiMo JANG_2L current dev-build audio/media is now
  explicitly red in `build/current-real-ui-dev-app-mimo-v25-jang2l-audio-proof-20260610.json`.
  The Electron dev app loaded the real 105 GiB bundle, completed two visible
  text turns, sent one audio attachment, and server `MEDIA_DIAG` saw
  `input_audio`; `/v1/chat/completions` returned `400`: `received unsupported
  media modality audio because the loaded runtime is text-only. Supported
  modalities: text.` Runtime/cache stayed live before the guard with active
  memory `105016.1 MB`, peak `106151.0 MB`, native `mixed_swa_kv_v1` /
  `mimo_v2_asymmetric_swa`, `l2_block_tokens_on_disk=110`, and block-disk
  writes `3`. Do not claim MiMo JANG_2L audio support from preserved media
  metadata.
- 2026-06-10 update: MiMo JANGTQ_2 installed-app short text/cache is green in
  `build/current-real-ui-installed-app-mimo-v25-jangtq2-text-cache-proof-20260610.json`.
  The rebuilt app loaded the 79 GiB bundle, produced exact
  `MIMO_JANGTQ2_TEXT_ONE` / `MIMO_JANGTQ2_TEXT_TWO`, hit paged cache with
  `cache_hit_tokens=42`, and wrote block L2. This does not clear the broader
  JANGTQ_2 artifact exactness/tool/media/source-vs-quant blockers.
- 2026-06-10 update: MiMo JANGTQ_2 installed-app Chat Completions built-in tool
  loop is green in `build/current-real-ui-installed-app-mimo-v25-jangtq2-tools-proof-20260610.json`.
  The rebuilt app loaded the 79 GiB bundle, executed `run_command`, created both
  expected probe files exactly (`REAL_UI_LIVE_TOOL_ONE` and
  `REAL_UI_LIVE_TOOL_TWO`), completed visible assistant turns, and recorded
  paged mixed-SWA cache/L2 evidence. This does not clear broader JANGTQ_2
  exactness/source-vs-quant, Responses tools, media, or MiMo JANG_2L tool rows.
- 2026-06-10 update: MiMo JANGTQ_2 installed-app Responses built-in tool loop
  is green in `build/current-real-ui-installed-app-mimo-v25-jangtq2-responses-tools-proof-20260610.json`.
  The rebuilt app used `/v1/responses`, executed `run_command`, sent
  `function_call_output` follow-ups with `previous_response_id`, recorded
  Responses delta/cache-detail surfaces, created both expected probe files
  exactly, and kept paged mixed-SWA cache/L2 evidence. This does not clear
  broader JANGTQ_2 literal/JSON/source-vs-quant exactness, media, or MiMo
  JANG_2L tool rows.
- 2026-06-10 update: MiMo JANGTQ_2 current dev-build Responses built-in tool
  loop is green in `build/current-real-ui-dev-app-mimo-v25-jangtq2-responses-tools-cache-20260610.json`.
  The `npm run dev` Electron app loaded the real 79 GiB bundle, used
  `/v1/responses`, executed `run_command`, sent `function_call_output`
  follow-ups with `previous_response_id`, recorded Responses delta/cache-detail
  surfaces, created both expected probe files exactly, and kept paged
  mixed-SWA cache/L2 evidence (`cache_hit_tokens=4548`,
  `l2_block_tokens_on_disk=4225`, block-disk hits `36`, writes `68`). This
  does not clear broader JANGTQ_2 literal/JSON/source-vs-quant exactness or
  media support.
- 2026-06-10 update: MiMo JANGTQ_2 current dev-build exact-output probe is
  also red in `build/current-real-ui-dev-app-mimo-v25-jangtq2-exact-output-proof-20260610.json`.
  The dev app loaded the real bundle, kept parser/reasoning leak checks clean,
  hit paged mixed-SWA cache with `cache_hit_tokens=41`, and wrote block L2;
  however it returned `ACKCB-742` for expected `ACK-CB-742` and only `{"` for
  expected `{"status":"ok","value":"blue-cat"}`. This matches installed app and
  confirms the blocker is not dev-vs-installed UI drift.
- 2026-06-10 update: the current raw real-UI harness now has optional exact
  assistant-content assertions (`VMLINUX_REAL_UI_EXPECT_ASSISTANT_1/2`), and
  the refreshed MiMo JANGTQ_2 dev-build exact-output proof is red directly in
  the raw artifact:
  `build/current-real-ui-dev-app-mimo-v25-jangtq2-exact-output-harness-assert-proof-20260610.json`.
  It loaded the real bundle, completed two UI turns, and failed on exact visible
  mismatches: `ACK-CB-742` -> `ACKCB-742`, and
  `{"status":"ok","value":"blue-cat"}` -> `{"status":"ok","value":"blue"}`.
  Cache and parser surfaces were healthy (`cache_hit_tokens=40`,
  `l2_block_tokens_on_disk=114`, no raw parser/reasoning leak), so do not
  paper this over with parser repair, JSON repair, sampling clamps, or cache
  changes. Treat it as artifact/logit/codebook/decode exactness until a
  stronger runtime-kernel proof says otherwise.
- 2026-06-10 update: MiMo JANGTQ_2 current dev-build image/media is explicitly
  red in `build/current-real-ui-dev-app-mimo-v25-jangtq2-image-proof-20260610.json`.
  The `npm run dev` Electron app loaded the real 79 GiB bundle, completed two
  visible text turns, sent one image attachment, and server `MEDIA_DIAG` saw
  `image_url`; `/v1/chat/completions` returned `400`: `received unsupported
  media modality image because the loaded runtime is text-only. Supported
  modalities: text.` Runtime/cache stayed live with `cache_detail=paged`,
  `cache_hit_tokens=39`, `l2_block_tokens_on_disk=132`, and block-disk writes
  `3`. Do not claim MiMo JANGTQ_2 image/VL support.
- 2026-06-10 update: MiMo JANGTQ_2 current dev-build video/media is explicitly
  red in `build/current-real-ui-dev-app-mimo-v25-jangtq2-video-proof-20260610.json`.
  The same Electron dev-app path loaded the real bundle, completed two visible
  text turns, sent one video attachment, and server `MEDIA_DIAG` saw
  `video_url`; `/v1/chat/completions` returned `400`: `received unsupported
  media modality video because the loaded runtime is text-only. Supported
  modalities: text.` Runtime/cache stayed live before the guard with native
  `mixed_swa_kv_v1` / `mimo_v2_asymmetric_swa`, `ram_tokens_cached=132`,
  `l2_block_tokens_on_disk=132`, `l2_tokens_on_disk=132`, and block-disk writes
  `3`. Do not claim MiMo JANGTQ_2 video support from preserved media metadata.
- 2026-06-10 update: MiMo JANGTQ_2 current dev-build audio/media is explicitly
  red in `build/current-real-ui-dev-app-mimo-v25-jangtq2-audio-proof-20260610.json`.
  The same Electron dev-app path loaded the real bundle, completed two visible
  text turns, sent one audio attachment, and server `MEDIA_DIAG` saw
  `input_audio`; `/v1/chat/completions` returned `400`: `received unsupported
  media modality audio because the loaded runtime is text-only. Supported
  modalities: text.` Runtime/cache stayed live before the guard with native
  `mixed_swa_kv_v1` / `mimo_v2_asymmetric_swa`, `ram_tokens_cached=132`,
  `l2_block_tokens_on_disk=132`, `l2_tokens_on_disk=132`, and block-disk writes
  `3`. Do not claim MiMo JANGTQ_2 audio support from preserved media metadata.
- 2026-06-10 update: MiMo JANGTQ_2 installed-app exact-output probe is still
  red in `build/current-real-ui-installed-app-mimo-v25-jangtq2-exact-output-proof-20260610.json`.
  The same rebuilt app loaded the real 79 GiB bundle, kept parser/reasoning
  leak checks clean, hit paged mixed-SWA cache with `cache_hit_tokens=41`, and
  wrote block L2; however it returned `ACKCB-742` for expected `ACK-CB-742`
  and only `{"` for expected `{"status":"ok","value":"blue-cat"}`. Treat this
  as artifact/logit/quant/decode exactness, not parser repair or cache.
- 2026-06-10 update: MiMo JANGTQ_2 installed-app image/media is now explicitly
  red in `build/current-real-ui-installed-app-mimo-v25-jangtq2-image-proof-20260610.json`.
  The app attached an image and server `MEDIA_DIAG` saw `image_url`, but the
  runtime returned the honest text-only guard. Vision tensors/config metadata
  are present, but this runtime is not wired for media.
- Do not run source-vs-quant comparisons if RAM-blocked unless Eric explicitly allows.
- Exactness failures must not be papered over by parser repair, sampling clamps, cache disabling, or JSON repair.
- Confirm runtime dynamically reads artifact config for bit size, grouped experts, stacked vs legacy layout, JANG/JANGTQ/MXFP metadata, and model-owned generation defaults.

4. N2 / Qwen-family JANG and JANGTQ

- Need live tool, reasoning, parser, cache, Responses, streaming args, and UI proof.
- Especially cover `gdn_sink`, MTP, hybrid SSM/cache, paged cache, TurboQuant KV encode/decode, block-disk L2, and no parser/reasoning leaks.
- JANG_1L should fit with careful RAM handling; treat current blocker as careful live-proof scheduling and memory discipline, not permanent infeasibility.
- Do not launch N2/JANG_1L below the preflight headroom gate after Metal OOM evidence. If preflight says `do_not_launch`, clear RAM or schedule later instead of forcing it.
- 2026-06-10 update: refreshed no-load preflight
  `build/current-n2-pro-jang1l-local-memory-preflight-20260610-after-installed-app-proofs.json`
  still says `decision=do_not_launch`: payload `110.57 GiB`, required
  available `118.57 GiB`, observed available `112.77 GiB`, gap `5.8 GiB`.
  Eric explicitly overrode the launch-safe gate, so
  `build/current-n2-jang1l-live-chat-cache-override-20260610.json` launched
  anyway with smaller batches/cache knobs. It still failed before health with
  Metal OOM after `Wired limit set to 115 GB (model 119 GB)`. JANG_1L still
  needs a real lower-peak runtime strategy before release support can be
  claimed.
- 2026-06-10 launch-safe refresh:
  `build/current-n2-pro-jang1l-local-memory-preflight-launch-safe-20260610.json`
  and `build/current-n2-jang1l-chat-cache-launch-safe-20260610.json` confirm
  the safe gate still skips before launch: available `114.22-114.23 GiB`,
  required `118.57 GiB`, gap about `4.35 GiB`. Requested tool, Responses,
  Responses stream, and L2 restart probes were recorded but not launched.
- 2026-06-10 high-free launch attempt:
  `build/current-n2-pro-jang1l-local-memory-preflight-ultrafree-20260610.json`
  still says strict `decision=do_not_launch` with observed available
  `114.09 GiB`, required `118.57 GiB`, gap `4.48 GiB`. Per Eric's instruction
  to launch one-at-a-time anyway,
  `build/current-n2-jang1l-live-chat-cache-ultrafree-20260610.json` ran with
  lowered JANG_1L headroom (`3 GiB`) plus low-peak knobs and still failed at
  `phase=server_startup`: qwen3_5_moe/JANG_1L detection, qwen parser, qwen3
  reasoning parser, hybrid cache, attention-only TurboQuant KV, native SSM
  companion state, mmap JANG loader, `123` shards, bfloat16 for `512` experts,
  `Wired limit set to 115 GB (model 119 GB)`, then Metal OOM before health.
- 2026-06-10 after-MiMo refresh:
  `build/current-n2-pro-jang1l-local-memory-preflight-after-mimo-exact-20260610.json`
  still reports `decision=do_not_launch` with `available_gib=113.29`,
  required `118.57`, gap `5.28`. The paired launch-safe gate
  `build/current-n2-jang1l-chat-cache-after-mimo-exact-20260610.json` skipped
  before loading weights, preserved requested tool/Responses/Responses-stream/L2
  probes, and left the cache directory empty. This is current scheduling
  evidence, not live runtime clearance.
- 2026-06-10 deferred-startup-eval update:
  `vmlx_engine/utils/tokenizer.py` now applies a narrow qwen3_5_moe affine
  `JANG_1L` policy that calls `load_jang_model(..., skip_eval=True)`, avoiding
  eager all-parameter eval at startup. `build/current-n2-jang1l-deferred-eval-startup-proof-20260610.json`
  records the live boundary: with `114.04 GiB` available and 3 GiB lower-peak
  headroom, JANG_1L reached `/health`, loaded `123` shards, applied `482`
  quant-shape repairs, initialized hybrid SSM/cache, attention TurboQuant KV,
  paged cache, block L2, and SSM companion L2, and completed one bounded Chat
  Completions request with HTTP `200`. Full support remains red: subsequent
  cache/tool/Responses probes hit working-set guard at `102%` of the `107.5GB`
  cap, and a guard-104/wired-limit experiment reproduced first-request Metal
  OOM. Do not clear JANG_1L cache/tool/Responses/L2/UI/media/release support
  from this partial fix.
- 2026-06-10 forced after-Gemma-video update:
  `build/current-n2-jang1l-live-chat-cache-forced-after-gemma-video-20260610.json`
  was launched with `--jang1l-required-extra-headroom-gib 0` per Eric's
  direction. It proves the 128GB Mac can load the real JANG_1L artifact and
  serve one bounded Chat request: `/health` reached, first Chat Completions
  request returned HTTP `200`, qwen3_5_moe/JANG_1L detection was correct,
  native `hybrid_ssm_v1`, live attention TurboQuant KV, SSM companion state,
  async rederive, paged cache, block L2, and SSM companion L2 initialized. The
  row remains red because the cache-warm and cache-hit requests returned
  HTTP `503` at `102%` of the `107.5GB` Metal working-set cap after the first
  request, with available memory down to `6.41 GiB`. This clears load + one
  bounded Chat request only, not cache/tool/Responses/L2/UI/media/release.
- 2026-06-10 current dev-app bounded Chat update:
  `build/current-real-ui-dev-app-n2-jang1l-bounded-chat-proof-20260610.json`
  proves the same boundary through the real Electron dev app. The row loaded
  `/Users/eric/.mlxstudio/models/JANGQ-AI/Nex-N2-Pro-JANG_1L`, reached
  `/health`, completed one Chat Completions request with HTTP `200`, selected
  qwen3_5_moe/JANG_1L, qwen tool parser, qwen3 reasoning parser, native
  `hybrid_ssm_v1`, live attention TurboQuant KV, SSM companion state, async
  rederive, paged cache, block L2, and SSM companion L2. It remains red because
  the first visible assistant content was whitespace/empty and the second UI
  turn returned HTTP `503` at `102%` of the `107.5GB` Metal working-set cap.
  Do not claim N2 JANG_1L dev-app usable support yet.
- 2026-06-10 current dev-app one-turn visible-output update:
  `build/current-real-ui-dev-app-n2-jang1l-one-turn-visible-proof-20260610.json`
  adds a bounded one-turn diagnosis using the new proof-harness switch
  `VMLINUX_REAL_UI_SECOND_TURN=0`. This run loaded the same real JANG_1L row
  and completed one Chat request with no renderer send error or Metal 503, but
  it still failed release assertions because the assistant streamed `16`
  whitespace tokens and persisted empty visible content. Runtime/cache evidence
  remained live: qwen3_5_moe/JANG_1L, qwen parser, qwen3 reasoning parser,
  native `hybrid_ssm_v1`, attention-only live TurboQuant KV, SSM companion
  state, async rederive, paged cache, block L2, SSM companion L2,
  `l2_block_tokens_on_disk=19`, `l2_ssm_tokens_on_disk=19`, and
  `l2_tokens_on_disk=38`. This separates first-turn whitespace decode/output
  from the second-turn working-set guard; it does not clear cache reuse, tools,
  Responses, L2 restart, media, installed-app parity, or release support.
- 2026-06-10 gate-refresh update:
  `build/current-objective-proof-after-mimo-n2-dev-app-proof-refresh-20260610.json`
  is now the canonical objective digest. It includes the forced current-source
  JANG_1L proof plus the real Electron dev-app bounded and one-turn proofs.
  `build/current-full-release-objective-checklist-after-mimo-n2-dev-app-proof-refresh-20260610.json`
  remains `status=open`, `failed_count=73`, and
  `build/current-release-regression-manifest-after-mimo-n2-dev-app-proof-refresh-20260610.json`
  remains `prepackage_ready=false`, `release_ready=false`. This is proof
  pointer repair only; it does not clear N2 release support.
- N2 JANGTQ_2 proof does not clear N2 JANG_1L.
- Keep architecture names explicit in every proof: base Qwen/Qwen35 MXFP8-MTP direct-source proof does not clear Nex/N2 Pro 397B JANG_1L, and N2 JANG_1L does not clear regular Qwen MTP/JANGTQ rows. Record `format`, `weight_format`, `artifact_profile`, MTP depth, `gdn_sink`, hybrid SSM/native-cache schema, TurboQuant KV state, and media weight backing from loaded health/config rather than inferred family names.

5. DSV4

- Memory-unit harness fix is landed by the other agent.
- Still needs live default-cache tool-loop proof when memory gate allows.
- Must use native SWA/CSA/HCA cache behavior, not a generic fake KV substitute.
- Do not claim DSV4 release clearance from load, health, narrow cache proof, or memory-label cleanup alone.

6. Gemma 4 QAT/native MXFP4/MXFP8/JANG

- Source startup and several source smokes exist, but full MXFP4/MXFP8/JANG_4M/QAT media/cache/UI/installed-app/tunnel matrix remains open.
- Need E2B, E4B, 12B, 26B, and 31B downloaded/present from JANGQ HF repos for full multi-turn, tool-call, parser, cache-reuse, coherency, and media proof.
- Incoherent multilingual/token-soup output from older app versions is likely real runtime/load/decode corruption until proven otherwise. Known class: Gemma4 MoE/native-MXFP sidecar hydration or shared-KV/load compatibility can corrupt generation.
- `ModuleNotFoundError: No module named 'mlx_vlm.models.gemma4_unified'` is a release blocker until the packaged/bundled runtime has the current compatibility alias/shim and installed-app parity is green.
- VLM/image prefill guard recovery and post-error text recovery still need proof. A failed image turn must not poison later text turns in the same chat/session.
- Audio/video claims must be weight-backed and runtime-proven. Token metadata alone is not native audio proof.

7. Step 3.7

- Metadata/runtime route matrix is improved.
- Do not make a fake `has_vision=false` release claim unless the release row is explicitly scoped text-only.
- Tool dialect loops, raw XML-like leaks, tool-result continuation, multi-turn synthesis, and thinking-template mismatch remain model/runtime parser issues to prove per path.
- If Step3p7 VLM runtime is advertised, prove live media or fail closed with honest capability reporting.

8. Structured JSON/XML

- Repair/validation is application and benchmark hygiene, not a substitute for runtime coherence.
- Guided/schema decoding should only be claimed if real runtime support exists.
- JSON/XML repair should report raw parse success, repair-needed status, normalized object, schema validation result, and retry/fail-closed behavior.

9. UI/CLI/API parity

- Parser, reasoning, cache type, prefix cache, paged cache, L2 disk cache, TurboQuant KV, MTP, max output tokens, max context, generation defaults, and media settings must match CLI, API, panel settings, installed-app launch, and persisted session settings.
- Responses endpoints must cover content-delta streaming, function-call argument streaming, final object consistency, previous response/session reuse, cancel/restart, and gateway/tunnel parity.
- Do not clear panel/UI rows from CLI source proof alone.

10. Release/package/sign/notarize

- Developer ID signing and notary profile access are currently usable after the documented keychain unlock/partition-list sequence; `build/current-signed-checkpoint-dmg-readiness-20260609.json` records fresh signing `pass` and notarization `pass`.
- Current release packaging is still blocked on rebuilding current-source DMGs, notarizing/stapling/verifying those current artifacts, and clearing or explicitly scoping the remaining runtime/model/UI/cache rows.
- No signing, notarization, tag, public download update, or release announcement until runtime/model/UI/cache blockers are green or Eric explicitly overrides.
- If a release is forced with known open rows, the release notes must list exact open rows and not imply full clearance.
- 2026-06-10 update: `/Applications/vMLX.app` was rebuilt and locally installed
  with `panel/scripts/build-and-install.sh` after a pre-rebuild audit found only
  `vmlx_engine/utils/jang_loader.py` stale. The refreshed installed-app runtime
  parity audit is
  `build/current-installed-app-runtime-parity-audit-after-local-install-20260610.json`
  with `status=pass`, `missing_or_stale=[]`, bundled engine hash parity true,
  and packaged source hash parity true. This clears the local installed-app
  runtime/source parity blocker only; it is not a Developer ID notarized DMG
  release and does not clear model-specific installed-app chat proofs or the
  remaining runtime/model/UI/cache rows.
- 2026-06-10 N2 installed-app update:
  `build/current-real-ui-installed-app-n2-jangtq2-responses-tools-cache-20260610.json`
  is `status=pass` for the local rebuilt `/Applications/vMLX.app` plus
  `/Users/eric/.mlxstudio/models/JANGQ-AI/Nex-N2-Pro-JANGTQ2`. It proves
  installed-app UI, `/v1/responses`, two built-in `run_command` calls,
  tool-result continuation, visible content deltas, server cache controls,
  no raw parser/reasoning leak, `hybrid_ssm_v1`, attention-only TurboQuant KV,
  native SSM companion state, block L2, and SSM disk hit. It does not clear
  public tunnel SSE, N2 audio, N2 JANG_1L, stricter prompt quality, or
  Developer ID DMG package/sign/notarize/release readiness.
- 2026-06-10 N2 dev-app exactness update:
  `build/current-real-ui-dev-app-n2-jangtq2-exact-output-proof-20260610.json`
  is green for the current Electron dev build and the same local JANGTQ2 row.
  The app returned exact text `N2-ACK-742` and exact JSON
  `{"status":"ok","value":"n2-blue"}`, with no parser/reasoning leak, while
  recording `hybrid_ssm_v1`, attention-only TurboQuant KV,
  `cache_detail=paged+ssm`, `cache_hit_tokens=21`,
  `l2_block_tokens_on_disk=59`, `l2_ssm_tokens_on_disk=80`, block-disk hits
  `3`, writes `2`, and SSM stores `2`. This does not clear the stricter custom
  long-delta prompt, public tunnel SSE parity, N2 audio, N2 JANG_1L, or release
  readiness.
- 2026-06-10 N2 installed-app media update:
  `build/current-real-ui-installed-app-n2-jangtq2-image-proof-20260610.json`
  and `build/current-real-ui-installed-app-n2-jangtq2-video-proof-20260610.json`
  are both `status=pass` for the same local rebuilt `/Applications/vMLX.app`
  and N2 JANGTQ2 row. They prove installed-app image/video attachment
  persistence, server `MEDIA_DIAG` detection, image `num_images_processed=1`,
  video base64 MP4 decode and `4` extracted frames, visible answers `Red` and
  `solid red screen`, no raw parser/reasoning leak, `hybrid_ssm_v1`,
  attention-only TurboQuant KV, native SSM companion state, block L2, and SSM
  companion disk stores. They do not clear N2 audio, N2 JANG_1L, public tunnel
  SSE, or Developer ID DMG package/sign/notarize/release readiness.
- 2026-06-10 N2 installed-app audio update:
  `build/current-real-ui-installed-app-n2-jangtq2-audio-proof-20260610.json`
  is `status=fail` for the same local rebuilt `/Applications/vMLX.app` and N2
  JANGTQ2 row. It proves installed-app audio attachment plumbing reached the
  server boundary: the app completed two text turns first, forced multimodal
  for one attached audio file, server `MEDIA_DIAG` saw `input_audio`, and the
  API returned the honest guard `400 - unsupported media modality audio;
  supported modalities: text, vision, video`. Runtime/cache remained green
  before the gate with `hybrid_ssm_v1`, attention-only TurboQuant KV,
  `cache_detail=paged+ssm`, block L2, and SSM companion disk stores. This does
  not clear N2 audio support, N2 JANG_1L, public tunnel SSE, or Developer ID DMG
  package/sign/notarize/release readiness.
- 2026-06-10 Gemma installed-app update:
  `build/current-real-ui-installed-app-gemma4-12b-mxfp4-responses-tools-cache-20260610.json`
  is `status=pass` for the local rebuilt `/Applications/vMLX.app` plus
  `/Users/eric/models/JANGQ-AI/gemma-4-12B-it-qat-MXFP4`. It proves
  installed-app UI, `/v1/responses`, two built-in `run_command` calls,
  tool-result continuation, visible content deltas, server cache controls,
  no raw parser/reasoning leak, MXFP4 affine matmul with Metal NA active,
  native `mixed_swa_kv_v1`, paged mixed-SWA cache, and block L2. It does not
  clear Gemma installed-app media, Gemma audio, public tunnel SSE, or Developer
  ID DMG package/sign/notarize/release readiness.
- 2026-06-10 Gemma dev-app exactness update:
  `build/current-real-ui-dev-app-gemma4-12b-mxfp4-exact-output-proof-20260610.json`
  is green for the current Electron dev build and the Gemma 12B QAT MXFP4 row.
  The app returned exact text `GEMMA-ACK-742` and exact JSON
  `{"status":"ok","value":"gemma-blue"}`, with no parser/reasoning leak, while
  recording `weight_format=mxfp4`, Metal NA active, `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=22`,
  `l2_block_tokens_on_disk=61`, and block-disk writes `2`. This does not clear
  Gemma audio, larger Gemma QAT rows, public tunnel SSE parity, or release
  readiness.
- 2026-06-10 Gemma installed-app image update:
  `build/current-real-ui-installed-app-gemma4-12b-mxfp4-image-proof-20260610.json`
  is `status=pass` for the same local rebuilt `/Applications/vMLX.app` and
  Gemma 12B QAT MXFP4 row. It proves installed-app image attachment persistence,
  server `MEDIA_DIAG` detection, Gemma media fallback with `1 image(s)`,
  visible answer `Red`, no raw parser/reasoning leak, MXFP4 affine matmul with
  Metal NA active, native `mixed_swa_kv_v1`, paged mixed-SWA cache, and block
  L2 write. It does not clear installed-app video, Gemma audio, public tunnel
  SSE, or Developer ID DMG package/sign/notarize/release readiness.
- 2026-06-10 Gemma installed-app video update:
  `build/current-real-ui-installed-app-gemma4-12b-mxfp4-video-proof-20260610.json`
  is `status=pass` for the same local rebuilt `/Applications/vMLX.app` and
  Gemma 12B QAT MXFP4 row. It proves installed-app video attachment persistence,
  server `MEDIA_DIAG` detection, base64 MP4 decode, `25` frames at `25.0 fps`,
  `4` extracted frames, Gemma media fallback, visible answer describing a solid
  red background, no raw parser/reasoning leak, MXFP4 affine matmul with Metal
  NA active, native `mixed_swa_kv_v1`, paged mixed-SWA cache, and block L2
  write. It does not clear Gemma audio, public tunnel SSE, or Developer ID DMG
  package/sign/notarize/release readiness.
- 2026-06-10 Gemma installed-app audio update:
  `build/current-real-ui-installed-app-gemma4-12b-mxfp4-audio-proof-20260610.json`
  is `status=fail` for the same local rebuilt `/Applications/vMLX.app` and
  Gemma 12B QAT MXFP4 row. It proves installed-app audio attachment plumbing
  reached the server boundary: the app completed two text turns first, forced
  multimodal for one attached audio file, server `MEDIA_DIAG` saw
  `input_audio`, and the API returned the honest guard `400 - unsupported media
  modality audio; supported modalities: text, vision, video`. Runtime/cache
  remained green before the gate with MXFP4 affine matmul, Metal NA active,
  native `mixed_swa_kv_v1`, generic TurboQuant KV correctly disabled,
  `cache_detail=paged+mixed_swa`, and block L2 writes. This does not clear
  Gemma audio support, public tunnel SSE, or Developer ID DMG
  package/sign/notarize/release readiness.
- 2026-06-10 Gemma JANG4M installed-app update:
  `build/current-real-ui-installed-app-gemma4-12b-jang4m-responses-tools-cache-20260610.json`
  is `status=pass` for the local rebuilt `/Applications/vMLX.app` and
  `/Users/eric/models/JANGQ-AI/gemma-4-12B-it-JANG_4M`. It proves installed-app
  `/v1/responses`, built-in `run_command` tool calls, `previous_response_id`
  tool-result continuations, visible assistant turns, streaming deltas,
  JANG affine Metal NA, native mixed-SWA cache, and block L2. It does not clear
  JANG4M installed-app video/audio, larger Gemma QAT rows, public tunnel SSE, or
  package/sign/notarize/release readiness.
- 2026-06-10 Gemma JANG4M installed-app image update:
  `build/current-real-ui-installed-app-gemma4-12b-jang4m-image-proof-20260610.json`
  is `status=pass` for the local rebuilt `/Applications/vMLX.app` and
  `/Users/eric/models/JANGQ-AI/gemma-4-12B-it-JANG_4M`. It proves installed-app
  image attachment persistence, Gemma media fallback with `1 image(s)`, visible
  answer `Red`, no raw parser/reasoning leak, JANG affine Metal NA, native
  mixed-SWA cache, and block L2 writes. It does not clear JANG4M installed-app
  video/audio, public tunnel SSE, or package/sign/notarize/release readiness.
- 2026-06-10 Gemma JANG4M installed-app video update:
  `build/current-real-ui-installed-app-gemma4-12b-jang4m-video-proof-20260610.json`
  is `status=pass` for the local rebuilt `/Applications/vMLX.app` and
  `/Users/eric/models/JANGQ-AI/gemma-4-12B-it-JANG_4M` with explicit
  `max_prompt_tokens=12000`. It proves installed-app video attachment
  persistence, server `MEDIA_DIAG` video detection, base64 MP4 decode, 4-frame
  extraction from the 25 fps fixture, visible solid-red-screen answer, no raw
  parser/reasoning leak, JANG affine Metal NA, native mixed-SWA cache, and block
  L2 writes. It does not clear JANG4M installed-app audio, default-4k video,
  public tunnel SSE, or package/sign/notarize/release readiness.
- 2026-06-10 Gemma JANG4M installed-app audio update:
  `build/current-real-ui-installed-app-gemma4-12b-jang4m-audio-proof-20260610.json`
  is `status=fail` for the local rebuilt `/Applications/vMLX.app` and
  `/Users/eric/models/JANGQ-AI/gemma-4-12B-it-JANG_4M`. It proves audio
  attachment plumbing reached runtime: `persistedAudioAttachment=true`, server
  `MEDIA_DIAG` saw `input_audio`, and the server decoded the base64 WAV. The
  audio turn still ended with empty visible assistant content and no
  `audio_where_supported` surface, so JANG4M installed-app audio remains red.
  This does not invalidate installed-app text/tools/image/video green rows, and
  it does not clear public tunnel SSE or package/sign/notarize/release readiness.
- 2026-06-10 Gemma JANG4M dev-app exactness update:
  `build/current-real-ui-dev-app-gemma4-12b-jang4m-exact-output-proof-20260610.json`
  is green for the current Electron dev build and
  `/Users/eric/models/JANGQ-AI/gemma-4-12B-it-JANG_4M`. The app returned exact
  text `JANG4M-ACK-742` and exact JSON
  `{"status":"ok","value":"jang4m-blue"}`, with no parser/reasoning leak, while
  auto-detecting Gemma4 tool/reasoning parsers and recording
  `weight_format=jang_affine`, Metal NA active, `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=24`,
  `l2_block_tokens_on_disk=66`, and block-disk writes `2`. This does not clear
  Gemma audio, larger Gemma QAT rows, public tunnel SSE parity, package/sign/
  notarize/tag/upload, or release readiness.
- 2026-06-10 Gemma 26B JANG4M dev-app exactness update:
  `build/current-real-ui-dev-app-gemma4-26b-jang4m-exact-output-proof-20260610.json`
  is green for the current Electron dev build and
  `/Users/eric/models/JANGQ-AI/gemma-4-26B-A4B-it-qat-JANG_4M`. The app
  returned exact text `GEMMA26-JANG4M-ACK-742` and exact JSON
  `{"status":"ok","value":"gemma26-jang4m-blue"}`, with no parser/reasoning
  leak and no persisted tools/reasoning, while using the Gemma4 parser family.
  Runtime evidence recorded `weight_format=jang_affine`, `profile=JANG_4M`,
  Metal NA eligibility, native `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=29`,
  `l2_block_tokens_on_disk=81`, `l2_tokens_on_disk=81`, and block-disk writes
  `2`. This does not clear 26B tools, Responses, media, installed-app parity,
  Gemma 31B, public tunnel SSE parity, package/sign/notarize/tag/upload, or
  release readiness.
- 2026-06-10 Gemma 31B JANG4M dev-app exactness update:
  `build/current-real-ui-dev-app-gemma4-31b-jang4m-exact-output-proof-20260610.json`
  is green for the current Electron dev build and
  `/Users/eric/models/JANGQ-AI/gemma-4-31B-it-qat-JANG_4M`. The app returned
  exact text `GEMMA31-JANG4M-ACK-742` and exact JSON
  `{"status":"ok","value":"gemma31-jang4m-blue"}`, with no parser/reasoning
  leak and no persisted tools/reasoning, while using the Gemma4 parser family.
  Runtime evidence recorded `weight_format=jang_affine`, `profile=JANG_4M`,
  Metal NA eligibility, native `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=29`,
  `l2_block_tokens_on_disk=81`, `l2_tokens_on_disk=81`, and block-disk writes
  `2`. This does not clear 31B tools, Responses, media, installed-app parity,
  public tunnel SSE parity, package/sign/notarize/tag/upload, or release
  readiness.
- 2026-06-10 Gemma 31B JANG4M dev-app Responses/tools update:
  `build/current-real-ui-dev-app-gemma4-31b-jang4m-responses-tools-cache-20260610.json`
  is green for the current Electron dev build and the same 31B JANG4M row. The
  app used `/v1/responses`, executed built-in `run_command` on both turns, sent
  tool-result continuations with `previous_response_id` and
  `function_call_output`, completed visible assistant turns, and wrote exact
  probe files `REAL_UI_LIVE_TOOL_ONE` and `REAL_UI_LIVE_TOOL_TWO`. Runtime
  evidence recorded active memory `28090.3 MB`, peak `34587.3 MB`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=320`,
  `l2_block_tokens_on_disk=1960`, `l2_tokens_on_disk=1960`, block-disk writes
  `58`, and block-disk evictions `26` inside the 2 GB L2 cap. This does not
  clear 31B media, installed-app parity, public tunnel SSE parity, 26B
  tools/media, package/sign/notarize/tag/upload, or release readiness.
- 2026-06-10 Gemma 26B JANG4M dev-app Responses/tools update:
  `build/current-real-ui-dev-app-gemma4-26b-jang4m-responses-tools-cache-20260610.json`
  is green for the current Electron dev build and the 26B A4B JANG4M row. The
  app used `/v1/responses`, executed built-in `run_command` on both turns, sent
  tool-result continuations with `previous_response_id` and
  `function_call_output`, completed visible assistant turns, and wrote exact
  probe files `REAL_UI_LIVE_TOOL_ONE` and `REAL_UI_LIVE_TOOL_TWO`. Runtime
  evidence recorded active memory `17782 MB`, peak `19943.9 MB`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=3538`,
  `l2_block_tokens_on_disk=3559`, `l2_tokens_on_disk=3559`, block-disk hits
  `30`, and block-disk writes `58`. This does not clear 26B media,
  installed-app parity, public tunnel SSE parity, package/sign/notarize/tag/
  upload, or release readiness.
- 2026-06-10 Gemma 26B JANG4M dev-app image/VL update:
  `build/current-real-ui-dev-app-gemma4-26b-jang4m-image-proof-20260610.json`
  is green for the current Electron dev build and the 26B A4B JANG4M row. The
  app persisted one image attachment, server `MEDIA_DIAG` saw `image_url`, the
  Gemma media fallback ran with `1 image(s)`, and the assistant answered `Red`
  with `imageSemanticVerified=true`. Runtime evidence recorded active memory
  `17780.6 MB`, peak `18557.2 MB`, native `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=20`,
  `l2_block_tokens_on_disk=64`, `l2_tokens_on_disk=64`, block-disk writes `2`,
  and media-prefix cache storage for `367` prompt tokens. This does not clear
  26B video/audio, installed-app parity, public tunnel SSE parity, package/
  sign/notarize/tag/upload, or release readiness.
- 2026-06-10 Gemma 26B JANG4M dev-app video/VL update:
  `build/current-real-ui-dev-app-gemma4-26b-jang4m-video-proof-20260610.json`
  is green for the current Electron dev build and the 26B A4B JANG4M row with
  explicit `max_prompt_tokens=12000`. The app persisted one `video_url`
  attachment, server `MEDIA_DIAG` saw `video_url`, the server decoded the
  base64 MP4, extracted `4` frames, routed those frames through the Gemma media
  fallback, and the assistant answered `The video is a solid, static red
  square. REAL_UI_LIVE.` with `videoSemanticVerified=true`. Runtime evidence
  recorded active memory `17779.1 MB`, peak `18557.2 MB`, native
  `mixed_swa_kv_v1`, `cache_detail=paged+mixed_swa`, `cache_hit_tokens=20`,
  `l2_block_tokens_on_disk=64`, `l2_tokens_on_disk=64`, block-disk writes `2`,
  and video-turn media-prefix cache storage for `357` prompt tokens. This does
  not clear default-4k video behavior, 26B audio, installed-app parity, public
  tunnel SSE parity, package/sign/notarize/tag/upload, or release readiness.
- 2026-06-10 Gemma 26B JANG4M dev-app audio update:
  `build/current-real-ui-dev-app-gemma4-26b-jang4m-audio-proof-20260610.json`
  is red by honest unsupported-modality guard. The app forced multimodal for
  one audio file and server `MEDIA_DIAG` saw `input_audio`, but
  `/v1/chat/completions` returned HTTP `400`: `unsupported media modality
  audio. Supported modalities: text, vision, video.` Runtime/cache stayed
  healthy before the guarded audio turn: active memory `17648.7 MB`, peak
  `17843.1 MB`, native `mixed_swa_kv_v1`, `cache_detail=paged+mixed_swa`,
  `cache_hit_tokens=20`, `l2_block_tokens_on_disk=64`, `l2_tokens_on_disk=64`,
  and block-disk writes `2`. Do not claim 26B audio support.
- 2026-06-10 Gemma 31B JANG4M dev-app image/VL update:
  `build/current-real-ui-dev-app-gemma4-31b-jang4m-image-proof-20260610.json`
  is green for the current Electron dev build and the 31B JANG4M row. The app
  persisted one image attachment, server `MEDIA_DIAG` saw `image_url`, the
  Gemma media fallback ran with `1 image(s)`, and the assistant answered `Red`
  with `imageSemanticVerified=true`. Runtime evidence recorded active memory
  `25850.6 MB`, peak `26233.1 MB`, native `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=20`,
  `l2_block_tokens_on_disk=62`, `l2_tokens_on_disk=62`, block-disk writes `2`,
  and media-prefix cache storage for `365` prompt tokens. This does not clear
  31B video/audio, installed-app parity, public tunnel SSE parity, package/
  sign/notarize/tag/upload, or release readiness.
- 2026-06-10 Gemma 31B JANG4M dev-app video/VL update:
  `build/current-real-ui-dev-app-gemma4-31b-jang4m-video-proof-20260610.json`
  is green for the current Electron dev build and the 31B JANG4M row with
  explicit `max_prompt_tokens=12000`. The app persisted one `video_url`
  attachment, server `MEDIA_DIAG` saw `video_url`, the server decoded the
  base64 MP4, extracted `4` frames, routed those frames through the Gemma media
  fallback, and the assistant answered `The provided image is a solid red
  square.` with `videoSemanticVerified=true`. Runtime evidence recorded active
  memory `25842.8 MB`, peak `26233.1 MB`, native `mixed_swa_kv_v1`,
  `cache_detail=paged+mixed_swa`, `cache_hit_tokens=20`,
  `l2_block_tokens_on_disk=62`, `l2_tokens_on_disk=62`, block-disk writes `2`,
  and video-turn media-prefix cache storage for `355` prompt tokens. This does
  not clear default-4k video behavior, 31B audio, installed-app parity, public
  tunnel SSE parity, package/sign/notarize/tag/upload, or release readiness.
- 2026-06-10 Gemma 31B JANG4M dev-app audio update:
  `build/current-real-ui-dev-app-gemma4-31b-jang4m-audio-proof-20260610.json`
  is red by honest unsupported-modality guard. The app forced multimodal for
  one audio file and server `MEDIA_DIAG` saw `input_audio`, but
  `/v1/chat/completions` returned HTTP `400`: `unsupported media modality
  audio. Supported modalities: text, vision, video.` Runtime/cache stayed
  healthy before the guarded audio turn: active memory `25324.6 MB`, peak
  `25728.6 MB`, native `mixed_swa_kv_v1`, `cache_detail=paged+mixed_swa`,
  `cache_hit_tokens=20`, `l2_block_tokens_on_disk=62`, `l2_tokens_on_disk=62`,
  and block-disk writes `2`. Do not claim 31B audio support.
- Proper release mechanics are documented in `/Users/eric/wiki/infra/apple-notarization.md`; do not invent an alternate path. The canonical keychain is `~/Library/Keychains/vmlx-build.keychain-db`, the Developer ID identity is `Developer ID Application: ShieldStack LLC (55KGF2S5AY)`, and notarization uses the `vmlx-notary` keychain profile.
- If signing returns `errSecInternalComponent`, fix key access with the documented sequence and retry once after the partition-list grant settles:

```sh
security unlock-keychain -p vmlx-release ~/Library/Keychains/vmlx-build.keychain-db
security set-keychain-settings ~/Library/Keychains/vmlx-build.keychain-db
security set-key-partition-list -S apple-tool:,apple:,codesign: -s -k vmlx-release ~/Library/Keychains/vmlx-build.keychain-db
```

- Current Python/Electron DMG flow is the repo script path, not a manual ad-hoc app signing path:

```sh
VMLINUX_PREPACKAGE_READY_MANIFEST_OUT=build/current-release-regression-manifest-pre-dmg-release-build-<scope>.json panel/scripts/build-release-dmgs.sh all
VMLINUX_NOTARY_KEYCHAIN=$HOME/Library/Keychains/vmlx-build.keychain-db panel/scripts/notarize-release-dmgs.sh
panel/scripts/verify-release-dmgs.sh
```

- `panel/scripts/build-release-dmgs.sh` first runs `tests/cross_matrix/run_release_regression_manifest.py --require-prepackage-ready`; if that ledger fails, the script must stop before DMG build. Do not bypass this stop except under an explicit checkpoint-release override that records the open rows in the release notes.
- `panel/scripts/notarize-release-dmgs.sh` submits the final signed Sequoia and Tahoe DMG containers, staples both, regenerates blockmaps, runs `spctl`, and prints final SHA-256 values. `panel/scripts/verify-release-dmgs.sh` is the post-staple verification pass.

## Other-agent integration requirements

- Before release, include all pushed source fixes from the other agent and verify they are present in the active worktree, `origin/main`, bundled Python, and installed app.
- Track issue/fix coverage by architecture: dense, MoE/routed, hybrid SSM, SWA/CSA/HCA, MLLM/VLM, audio/video, JANG, JANGTQ/MXTQ, MXFP4, MXFP8, QAT, native MTP.
- A fix for one family does not clear another family unless the same path is live-proven.
- Do not duplicate old work from `/Users/eric/vmlx`, ADLab, Max2 transport notes, or deprecated Swift lanes.

## Immediate focus order

1. Keep status/docs current so a second agent can pick up without guessing.
2. Fix and prove Responses streaming tool arguments with reasoning enabled on the real reported model/request and direct/gateway/tunnel surfaces.
3. Continue Gemma QAT/native MXFP4/MXFP8/JANG proof after downloads are available, including multi-turn, tools, parser leaks, cache reuse, media, and installed app.
4. Continue MiMo and N2 live runtime proof under memory gates, not source-vs-quant heavy comparisons.
5. Rebuild current-source DMGs only after the chosen checkpoint scope is explicit; then notarize/staple/verify with the documented `vmlx-build.keychain-db`/`vmlx-notary` path.
