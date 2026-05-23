# `/goal` driver prompt

A ready-to-run [`/goal`](https://code.claude.com/docs/en/goal) objective that
drives this port from the current state to **playable single-player on this
host** while **growing the reusable knowledge base**, following `PLAN.md`.
`/goal` keeps the objective active (a Stop hook blocks stopping) until the
Definition of Done holds, so the agent works through the phases iteratively
across sessions.

**How to use:** open Claude Code in the super-repo root and run `/goal`, pasting
the prompt below. Ensure the game dump is present (git-ignored) and that you have
rights to install Clang. (All repository content is English-only; this prompt is
in English by design.)

---

## Prompt

```text
Goal: bring "South Park: Let's Go Tower Defense Play!" (Xbox 360 / XBLA,
Title ID 58410931) to a playable state on THIS Windows 11 host via static
recompilation, AND continuously grow a large, reusable, publishable Xbox 360
recompilation knowledge base. Both are required. Do not stop until both hold.

CONTEXT & LAYOUT
- You are in the super-repo. The dump is in "South Park - Let's Go Tower Defense
  Play!/" (git-ignored; NEVER commit game code/assets).
- Follow PLAN.md (phases 0->6). Reusable knowledge + this title's case study live
  in knowledge-base/ (general/ = transferable; titles/south-park-lgtdp/ = live
  findings; templates/ = starting points).
- Primary toolchain: third_party/rexglue-sdk (codegen + runtime, D3D12 backend,
  own Xenos shader translation, XMA audio, SDL input, VFS).
- Reference/fallback: third_party/XenonRecomp and third_party/XenosRecomp.

DUAL GOAL (co-equal)
1) A playable native build: OFFLINE single-player (+ local co-op if cheap):
   boot -> menu -> match -> win/lose -> save -> continue, with audio and input.
2) A knowledge base that compounds: every phase must leave knowledge-base/
   richer. Record findings in titles/south-park-lgtdp/ (imports backlog, boot
   log, render notes) AND PROMOTE anything transferable into general/ and update
   templates/. Keep general/ clean, cited, and publishable (CC BY 4.0).

SCOPE
- v1 excludes online co-op, leaderboards, achievement sync, avatar awards — stub
  xam/session APIs gracefully to "offline".

HOW TO WORK
- Iterate through PLAN.md phases. Every logical step is a SEPARATE commit,
  authored Claude Code:
  git -c user.name="Claude Code" -c user.email="noreply@anthropic.com" commit ...
  Commit into whichever sub-repo you change (south-park-recomp/, knowledge-base/)
  and bump the super-repo gitlink after a submodule advances. Keep it all local
  (bare remotes under ~/xbla-remotes; push there).
- ALL repository content is English-only (docs, code, comments, commit messages).
- Primary path is rexglue: `rexglue init` -> edit south_park_td_config.toml
  (file_path -> private/default.xex; switch tables; save/restore regs;
  longjmp/setjmp; functions; invalid_instructions) -> build the *_codegen
  target -> build the app. Use --force to converge past unresolved calls; cross-
  check jump tables with XenonAnalyse. If rexglue's early-dev state blocks a
  stage, switch THAT stage to the XenonRecomp + custom-runtime recipe.
- Use the KB as you go: follow general/90-new-title-onboarding-playbook.md and
  general/95-pitfalls-and-patterns.md; consult general/50 (CPU), 60 (GPU),
  70/75 (runtime), 80 (hooks). When you learn something general, write it back.
- VERIFY BY RUNNING: actually launch the build, observe the frame/behavior, save
  screenshots and logs into knowledge-base. Never claim success unverified.

WHEN TO ASK THE MAINTAINER (otherwise act autonomously, don't stop)
- Admin-rights install or a GUI action you can't do from the CLI (e.g. install
  Clang 20+ / Vulkan SDK / PowerShell 7).
- External/paid resource, or a game-design/scope decision. Real legal/ethical
  stop. Otherwise do not ask.

FIRST STEPS NOW
1) Phase 0: check for Clang 20+. If missing and not CLI-installable, ask the
   maintainer to install LLVM/Clang 20+ (and pwsh 7). Build & install rexglue-sdk;
   record exact tool versions in knowledge-base/titles/south-park-lgtdp/.
2) Phase 1: extract private/default.xex from the main STFS package
   (58410931/000D0000/...), extract the asset tree, classify the two 00000002
   packages (any Title Update *.xexp?), do XEX recon and record results in
   knowledge-base/titles/south-park-lgtdp/10-dump-analysis.md (and promote any
   general extraction/recon insight into general/20 and general/25).
Then proceed through PLAN.md.

DEFINITION OF DONE (both required)
A) The game compiles with Clang, launches on this machine, and is PLAYABLE in
   single-player from boot to a win/lose screen: rendering correct, audio working
   (XMA->SDL), gamepad input working, save/continue working. Final run
   instructions recorded in south-park-recomp/docs/RUN.md.
B) The knowledge base reflects the journey: titles/south-park-lgtdp/ logs are
   filled (imports backlog, boot log, render notes) and all transferable lessons
   are promoted into general/ (kept publishable). A short final report summarizes
   what worked, the real effort, and the most reusable lessons.
Only when BOTH A and B hold is the goal complete.
```

---

### Notes on realism

- This prompt expects **iteration**, not a one-shot. Phases 3–5 are where the
  time goes; the agent should keep looping, committing, and recording.
- It **bounds scope** (no online) so goal A is *achievable* — the one open-ended
  risk (netcode) cannot stall it.
- It makes the **knowledge base co-equal** (goal B), matching the project's
  onboarding-pilot intent: the KB should be the durable payoff that makes the
  *next* title cheaper.
- It names the **one human-in-the-loop** dependency up front (installing Clang),
  and keeps the **fallback** (XenonRecomp + custom runtime) explicit.
