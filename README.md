# South Park: Let's Go Tower Defense Play! — Static Recompilation (XBLA → PC)

Super-repository coordinating a static recompilation ("recomp") of the Xbox 360
Live Arcade title **South Park: Let's Go Tower Defense Play!** (Title ID
`58410931`) into a native PC executable, using the hedge-dev recompiler toolchain.

This is a research / preservation project in the same spirit as the N64, PS2 and
Xbox 360 (e.g. *Unleashed Recompiled*) static-recompilation efforts. **No
copyrighted game data is distributed here.** You must supply your own legally
dumped copy of the game; the dump and anything extracted from it are git-ignored.

## Repository layout

```
.
├── third_party/
│   ├── XenonRecomp/      Xbox 360 PowerPC (Xenon) → C++ static recompiler        [submodule]
│   ├── XenosRecomp/      Xbox 360 Xenos shader microcode → HLSL/SPIR-V recompiler [submodule]
│   └── rexglue-sdk/      Glue/runtime SDK for recomp host integration             [submodule]
├── south-park-recomp/    The actual port: runtime, host, build, game patches      [submodule, new repo]
├── knowledge-base/       Research notes, feasibility analysis, RE findings, logs   [submodule, new repo]
├── PLAN.md               Full end-to-end plan (setup → playable on this host)
├── GOAL_PROMPT.md        A ready-to-run /goal prompt that drives the port to completion
└── "South Park - Let's Go Tower Defense Play!/"   Your game dump (GIT-IGNORED)
```

## Toolchain components

| Component        | Role                                                                 |
|------------------|----------------------------------------------------------------------|
| **XenonRecomp**  | Translates the title's PowerPC machine code (incl. VMX128) to C++.    |
| **XenosRecomp**  | Translates Xenos GPU shader microcode to portable HLSL/SPIR-V.        |
| **rexglue-sdk**  | Host-side glue: kernel/XAM imports, graphics/audio/input abstraction. |
| **south-park-recomp** | Title-specific runtime, patches, asset loading and the final app.|

## Status

Bootstrapping. See `PLAN.md` for the roadmap and `knowledge-base/` for analysis.

## Legal

For personal preservation and interoperability research only. Bring your own
dump. Do not redistribute game code or assets.
