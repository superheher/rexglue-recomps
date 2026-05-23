# South Park: Let's Go Tower Defense Play! — Static Recompilation (XBLA → PC)

Super-repository coordinating a static recompilation ("recomp") of the Xbox 360
Live Arcade title **South Park: Let's Go Tower Defense Play!** (Title ID
`58410931`) into a native PC executable.

This title was chosen deliberately as a **small onboarding pilot** to validate
the workflow and measure effort. The project therefore has **two goals**:

1. **A playable native build** of the game on this host (offline single-player).
2. **A large, reusable, publishable knowledge base** for Xbox 360 static
   recompilation in general — so future ports (and other people / agent
   sessions) start far ahead. The pilot is the first case study feeding it.

In the spirit of the N64 / PS2 / Xbox 360 (e.g. *Unleashed Recompiled*) recomp
efforts. **No copyrighted game data is distributed here.** Bring your own legally
dumped copy; the dump and anything extracted from it are git-ignored.

## Repository layout

```
.
├── third_party/
│   ├── XenonRecomp/      Xbox 360 PowerPC (Xenon) → C++ recompiler   [submodule, reference/fallback]
│   ├── XenosRecomp/      Xenos shader microcode → HLSL/SPIR-V        [submodule, reference/fallback]
│   └── rexglue-sdk/      Integrated codegen + runtime SDK (PRIMARY)  [submodule]
├── south-park-recomp/    The actual port: config, tools, host, patches  [submodule, new repo]
├── knowledge-base/       Reusable + publishable 360-recomp KB:           [submodule, new repo]
│                           general/ (transferable) · titles/ · templates/
├── PLAN.md               Full end-to-end plan (setup → playable on this host)
├── GOAL_PROMPT.md        Ready-to-run /goal prompt that drives the port to completion
└── "South Park - Let's Go Tower Defense Play!/"   Your game dump (GIT-IGNORED)
```

## Toolchain

| Component | Role |
|---|---|
| **rexglue-sdk** (primary) | Integrated SDK: PPC→C++ codegen **and** runtime — D3D12/Vulkan renderer with own shader translation, XMA audio, SDL input, VFS, kernel/XAM, scaffolder, PSReX lifecycle. |
| **XenonRecomp** (reference/fallback) | PPC→C++ recompiler; the canonical spec for per-title config knobs; `XenonAnalyse` for jump tables. |
| **XenosRecomp** (reference/fallback) | Xenos shader → HLSL/DXIL/SPIR-V; shader-cache fallback. |
| **south-park-recomp** | Title-specific config, extraction tooling, host shims, hooks, the final app. |

See `knowledge-base/general/30-toolchains.md` for the decision rationale.

## Conventions

- **All repository content is English-only** (across every repo here).
- **Each logical step is its own commit**, authored **Claude Code**; the
  super-repo gitlink is bumped after a submodule advances.
- **Work is local**: the two new repos use local bare remotes under
  `~/xbla-remotes` (re-point to GitHub to publish).

## Status

Base deployed; toolchain studied; feasibility assessed (**feasible, medium
difficulty**); knowledge base populated. Next: execute `PLAN.md` via
`GOAL_PROMPT.md` (Phase 0 = install Clang 20+; Phase 1 = extract `default.xex`).

## Legal

For personal preservation and interoperability research only. Bring your own
dump. Do not redistribute game code or assets.
