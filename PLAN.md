# PLAN — *South Park: Let's Go Tower Defense Play!* — Xbox 360 → PC, end to end

A complete roadmap from the current repository state to **the game running,
single-player, on this Windows 11 host**. The companion `GOAL_PROMPT.md` is a
ready-to-run `/goal` prompt that executes this plan iteratively.

Background and justification for the approach live in `knowledge-base/`
(feasibility verdict: **feasible, medium difficulty**). Primary toolchain:
**rexglue-sdk**; XenonRecomp / XenosRecomp kept as reference/fallback.

This title is a deliberate **onboarding pilot**, so the project has a **second,
co-equal goal**: grow a **large, reusable, publishable** Xbox 360 recompilation
knowledge base (`knowledge-base/general/` + `templates/`). Every phase must leave
that KB richer — promote anything transferable out of the title's case study into
`general/` so the next port starts ahead. Treat the KB as a first-class
deliverable, not a byproduct.

> **Legal:** personal preservation / interoperability only. Bring your own dump.
> No game code or assets are committed; everything derived is git-ignored.

---

## Conventions (apply to every phase)

- **Commit discipline.** Every logical step is its own commit, author **Claude
  Code** (`-c user.name="Claude Code" -c user.email="noreply@anthropic.com"`),
  in whichever repo it touches. Bump the super-repo gitlink after a submodule
  advances.
- **Three working repos:** `south-park-recomp/` (the port), `knowledge-base/`
  (findings — keep current), super-repo (coordination, plan, gitlinks).
- **Local only.** Submodule remotes for the two new repos are local bare repos
  under `~/xbla-remotes`. (To publish later: re-point URLs to GitHub.)
- **Verify, don't assume.** Each phase has an acceptance check; record real
  output (build logs, screenshots) rather than declaring success.
- **Record as you learn (and generalize).** New imports, hooks, jump tables,
  engine quirks → the title's case study (`knowledge-base/titles/<id>/`); then
  **promote transferable findings into `knowledge-base/general/`** so they help
  future ports. The KB is a co-equal deliverable.
- **English only.** All content in every repository here is written in English
  (commit messages, docs, code comments). Chat with the maintainer may be in
  another language, but the repos are English.

---

## Phase 0 — Prerequisites & toolchain build

**Goal:** a working build environment and built recompiler/runtime.

1. Install **Clang 20+** (LLVM standalone *or* VS2022 "C++ Clang tools";
   `clang-cl` on Windows). This is the one hard gap on the host today (R1).
2. Install **PowerShell 7 (`pwsh`)** for PSReX; confirm CMake ≥ 3.25 (have
   4.3.1) and Ninja (have 1.13.2). Vulkan SDK optional (D3D12 is primary).
3. `git submodule update --init --recursive` so the toolchain submodules pull
   their own dependencies.
4. Build & install **rexglue-sdk**:
   `cmake --preset win-amd64 && cmake --build --preset win-amd64 --target install`.
   Optionally run `Invoke-ReXSetup` and build XenonRecomp/XenosRecomp (reference).

**Acceptance:** `clang --version` ≥ 20; `rexglue --help` runs; rexglue builds
clean. **Commit:** record exact tool versions in `knowledge-base/40-environment.md`.

---

## Phase 1 — Extraction & XEX recon

**Goal:** the actual executable to recompile, plus concrete recon.

1. **Extract `default.xex`** from the main LIVE/STFS package
   (`58410931/000D0000/…`) into `south-park-recomp/private/` (git-ignored).
   Prefer a reproducible extractor in `south-park-recomp/tools/` (STFS: data
   base `0xC000`, 4 KiB blocks, hash stride `0xAA`); a third-party extractor
   (wxPirs/Velocity/QuickBMS) is an acceptable bootstrap.
2. **Extract the full asset tree** from the package (everything the loader reads
   besides the XEX) into a git-ignored content root for the VFS to mount later.
3. **Classify the two `00000002` marketplace packages** — is either a Title
   Update (`*.xexp`)? If so, keep it for the recompiler's patch input (R9).
4. **XEX recon** on `default.xex`: XEX2 confirmed; record base address, entry
   point, section/segment table, `.pdata` presence, compression/encryption
   flags, and the **resolved import ordinal list** (the real shim backlog).

**Acceptance:** `private/default.xex` exists; first 4 bytes are `XEX2`; recon
written to `knowledge-base/20-dump-analysis.md` ("XEX recon results"). **Commit**
the extractor/tooling in `south-park-recomp`.

---

## Phase 2 — Project scaffold & first codegen

**Goal:** generated C++ that **compiles** into an (not-yet-working) executable.

1. `rexglue init --app_name south_park_td --app_root <south-park-recomp>` →
   scaffold (CMakeLists, presets, `src/main.cpp`, `src/south_park_td_app.h`,
   `south_park_td_config.toml`, `generated/rexglue.cmake`).
2. **Configure** `south_park_td_config.toml`:
   - `file_path` → `private/default.xex` (+ patch path if a TU exists);
   - register **save/restore** addresses, `longjmp`/`setjmp`, explicit
     `functions`, `invalid_instructions` (EH data), and the **switch tables**.
3. **Jump tables:** run `XenonAnalyse default.xex tables.toml` as a cross-check;
   reconcile with rexglue's scanners; hand-author entries the analyzer misses
   (R3). Use `--force` to push past unresolved calls and converge iteratively.
4. **Codegen:** `cmake --build --preset win-amd64-debug --target
   south_park_td_codegen`; fix analysis errors until codegen is clean.
5. **Compile** the generated sources + runtime.

**Acceptance:** the app **links** to an executable (running may still crash).
**Commit** config + scaffold in `south-park-recomp`; note jump-table/boundary
findings in the knowledge base.

---

## Phase 3 — Boot bring-up (first frame)

**Goal:** the executable **boots** and reaches a first rendered frame / title.

1. Run it; triage the first crash. Implement **missing kernel/XAM imports** as
   they are hit (most already exist in the rexglue runtime; fill the gaps).
2. Stand up early runtime state: memory/heap, TLS, thread creation, timing,
   and the **VFS mount** of the extracted asset tree so the title's file opens
   resolve.
3. Handle recomp structural issues: register save/restore correctness,
   `longjmp`/`setjmp`, EH-data skips, any remaining bad function boundaries.
4. Bring the **GPU command processor** to life enough to present a frame.

**Acceptance:** a window presents the title's first frame / menu (even if
imperfect). **Commit** each shim/hook batch; log the boot sequence in the KB.

---

## Phase 4 — Rendering correctness

**Goal:** menus and gameplay **render correctly**.

1. **Shaders:** drive rexglue's runtime Xenos→DXIL/SPIR-V path; fix mismatches
   per shader (vertex fetch, vertex declarations, semantics, `R11G11B10`,
   alpha-test, any instancing) (R5). XenosRecomp is the fallback reference.
2. **Render state:** blending, depth/stencil, viewport/scissor, render-target
   formats, EDRAM resolve, gamma/sRGB.
3. **Textures/samplers:** formats, swizzles, mip/filtering, cube maps.

**Acceptance:** title, menus, and an in-game match look correct (compare to
reference footage). **Commit** per-fix; capture before/after screenshots in KB.

---

## Phase 5 — Audio, input, save, timing

**Goal:** the full play loop works with sound, controls, and persistence.

1. **Audio:** route XMA decode (rexglue FFmpeg/XMA) → SDL; music, SFX, voice;
   correct mixing/sample rate; any cutscene audio/video.
2. **Input:** map XInput → SDL game controller (+ keyboard); menus and gameplay.
3. **Save/continue:** persist via the VFS to a host save location; verify
   load-after-restart.
4. **Timing:** stable frame pacing; fix any timing-dependent gameplay/AI.

**Acceptance:** start app → menu → play a full match → win/lose → save →
relaunch → continue, all with audio and controls. **Commit** per subsystem.

---

## Phase 6 — Polish, performance, packaging

**Goal:** a pleasant, runnable build on this host.

1. Enable codegen **optimizations** now that it is stable (`skip_lr`,
   `*_as_local`, etc.); re-verify after each.
2. Performance pass (CPU hot functions, GPU state churn, shader cache warm-up).
3. CVars/config, controller remap, window/resolution options.
4. Package a runnable Release build; write `south-park-recomp/docs/RUN.md`.

**Out of scope for v1:** online co-op / leaderboards / achievement sync /
avatar awards — stub `xam`/session APIs to graceful "offline" (R8). Local co-op
included only if it falls out cheaply.

**Acceptance = Definition of Done:** the game **boots and is playable
single-player from start to a win/lose** on this Windows 11 machine, with audio,
controller input, and working save.

---

## Cross-cutting workstreams

- **Knowledge base** stays current: `40-environment.md` (versions),
  `50-imports-backlog.md` (kernel/XAM shims: done/todo), `60-render-notes.md`,
  `70-boot-log.md`. Each is a commit as it grows.
- **Fallback path:** if rexglue's early-development state blocks a milestone,
  switch the affected stage to the proven **XenonRecomp + custom runtime**
  recipe (the *Unleashed Recompiled* approach) for that piece.
- **Risk owners:** track R1–R9 from `knowledge-base/00-feasibility-analysis.md`;
  close or re-rate each as the port progresses.
- **Verification:** prefer running the app and observing over assuming; keep
  screenshots/logs in the KB.

## Milestone summary

| Phase | Done when | Rough effort |
|---|---|---|
| 0 Prereqs | rexglue builds; Clang 20+ present | hours |
| 1 Extract & recon | `default.xex` out; recon recorded | 0.5–1 day |
| 2 Codegen | app links | days |
| 3 Boot | first frame / menu | 1–3 weeks |
| 4 Render | looks correct | 2–6 weeks |
| 5 A/V/save | full play loop | 2–4 weeks |
| 6 Polish | packaged, playable | 1–2 weeks |

**Total to playable single-player: ~2–4 months part-time** (faster under a tight
agent loop; slower if the custom engine throws curveballs). The decisive early
signal is the end of Phase 3 — once it boots and presents a frame, the rest is
iteration, not research.
