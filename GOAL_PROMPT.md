# `/goal` driver prompt

A ready-to-run [`/goal`](https://code.claude.com/docs/en/goal) objective that
drives this port to **playable single-player on this host** while **growing the
reusable knowledge base**, following `PLAN.md`. `/goal` keeps the objective
active (a Stop hook blocks stopping) until the Definition of Done holds.

**How to use:** open Claude Code in the super-repo root and run `/goal`, pasting
the prompt below. The prompt is kept **under the `/goal` 4000-character limit**.
Ensure the dump is present (git-ignored). All repo content is English-only.

---

## Prompt (paste into `/goal`)

```text
Goal: make "South Park: Let's Go Tower Defense Play!" (Xbox 360/XBLA, Title ID
58410931) PLAYABLE on THIS Windows 11 host via static recompilation, AND grow a
reusable, publishable Xbox 360 recompilation knowledge base. Both required;
don't stop until both hold.

CONTEXT
- You are in the super-repo. Dump: "South Park - Let's Go Tower Defense Play!/"
  (git-ignored; NEVER commit game code/assets).
- Follow PLAN.md (phases 0-6). Knowledge: knowledge-base/ (general/=reusable,
  titles/south-park-lgtdp/=this port, templates/).
- Primary toolchain: third_party/rexglue-sdk (codegen + runtime, D3D12, own
  shader translation, XMA audio, SDL, VFS). Fallback: third_party/XenonRecomp +
  XenosRecomp.
- Already installed: Clang 22.1.6, pwsh 7.6.2 (scoop, on user PATH), CMake,
  Ninja, Python, Git.

SCOPE
- v1 = OFFLINE single-player (+ local co-op if cheap): boot -> menu -> match ->
  win/lose -> save -> continue, with audio + input.
- Out of scope: online co-op, leaderboards, achievements, avatar awards -> stub
  xam/sessions to "offline".

HOW TO WORK
- Iterate PLAN.md. Every logical step = a SEPARATE commit, author Claude Code:
  git -c user.name="Claude Code" -c user.email="noreply@anthropic.com" commit ...
  Commit into the sub-repo you change; bump the super-repo gitlink after a
  submodule advances. Stay local (bare remotes under ~/xbla-remotes).
- ALL repo content is English-only.
- KB is a co-equal deliverable: log findings in titles/south-park-lgtdp/
  (imports backlog, boot log, render notes) AND promote transferable lessons
  into general/ + templates/ (keep general/ clean, cited, publishable).
- rexglue path: `rexglue init` -> edit *_config.toml (file_path ->
  private/default.xex; switch tables; save/restore regs; longjmp/setjmp;
  functions; invalid_instructions) -> build *_codegen target -> build app. Use
  --force to converge; cross-check jump tables with XenonAnalyse. If rexglue
  blocks a stage, switch THAT stage to XenonRecomp + custom runtime.
- Use the KB: general/90 (playbook), 95 (pitfalls), 50 (CPU), 60 (GPU), 70/75
  (runtime), 80 (hooks).
- VERIFY BY RUNNING: launch the build, observe, save screenshots/logs to the KB.
  Never claim success unverified.

ASK THE MAINTAINER ONLY IF: an admin/GUI action you can't do via CLI; an
external/paid resource or a scope/game-design decision; a real legal stop.
Otherwise act autonomously and don't stop.

FIRST STEPS
1) Finish Phase 0: git submodule update --init --recursive; build+install
   rexglue-sdk (cmake --preset win-amd64 ; --target install).
2) Phase 1: extract private/default.xex + asset tree from the main STFS
   (58410931/000D0000/...); classify the two 00000002 packages (Title Update
   *.xexp?); do XEX recon -> titles/south-park-lgtdp/10-dump-analysis.md
   (promote general insight to general/20,25). Then continue PLAN.md.

DONE WHEN (both):
A) Compiles with Clang, launches, playable single-player from boot to win/lose:
   rendering correct, audio (XMA->SDL), gamepad, save/continue. Record run steps
   in south-park-recomp/docs/RUN.md.
B) KB reflects the journey: titles/ logs filled + transferable lessons promoted
   to general/ (publishable) + a short final report (what worked, real effort,
   top reusable lessons).
```

---

### Notes on realism

- Expects **iteration**, not one-shot; phases 3–5 are where the time goes.
- **Bounds scope** (no online) so goal A is achievable; keeps the **fallback**
  (XenonRecomp + custom runtime) explicit.
- Makes the **knowledge base co-equal** (goal B) — the durable payoff that makes
  the next title cheaper.
- The full, untruncated rationale lives in `PLAN.md`; this prompt is the compact
  driver that fits the `/goal` limit.
