# `/goal` driver prompt

A ready-to-run [`/goal`](https://code.claude.com/docs/en/goal) objective that
drives this port from the current state to **playable single-player on this
host**, following `PLAN.md`. `/goal` keeps the objective active (a Stop hook
blocks stopping) until the Definition of Done holds, so the agent works through
the phases iteratively across sessions.

**How to use:** open Claude Code in the super-repo root and run `/goal`, pasting
the prompt below (Russian canonical; English mirror follows). Make sure the game
dump is present (git-ignored) and you have rights to install Clang.

---

## Промпт (canonical, RU)

```text
Цель: довести «South Park: Let's Go Tower Defense Play!» (Xbox 360 / XBLA,
Title ID 58410931) до играбельного состояния на ЭТОЙ Windows 11 машине методом
статической рекомпиляции. Не останавливайся, пока цель не достигнута.

КОНТЕКСТ И СТРУКТУРА
- Ты в супер-репозитории. Дамп игры — в папке "South Park - Let's Go Tower
  Defense Play!/" (git-ignored; НИКОГДА не коммить игровой код/ассеты).
- Следуй PLAN.md (фазы 0→6). Анализ, решения и риски — в knowledge-base/.
- Основной тулчейн: third_party/rexglue-sdk (codegen + рантайм, бэкенд D3D12,
  своя трансляция шейдеров Xenos, аудио XMA, ввод SDL, VFS).
- Референс/фоллбэк: third_party/XenonRecomp и third_party/XenosRecomp.

ОБЪЁМ (SCOPE)
- v1 = ОФЛАЙН одиночная игра (+ локальный кооп, если достаётся дёшево):
  запуск → меню → матч → победа/поражение → сохранение → продолжение,
  со звуком и управлением.
- Вне объёма: онлайн-кооп, лидерборды, синхронизация ачивок, avatar awards —
  заглушай xam/сессии в «офлайн» аккуратно.

КАК РАБОТАТЬ
- Итеративно по фазам PLAN.md. Каждый логический шаг — ОТДЕЛЬНЫЙ коммит,
  автор Claude Code:
  git -c user.name="Claude Code" -c user.email="noreply@anthropic.com" commit ...
  Коммить в тот под-репозиторий, который меняешь (south-park-recomp/,
  knowledge-base/), и бампай gitlink супер-репо после продвижения сабмодуля.
- Всё локально. Под-репозитории south-park-recomp и knowledge-base имеют
  локальные bare-remote под ~/xbla-remotes — пушь туда.
- Веди knowledge-base в актуальном состоянии: новые kernel/xam-импорты,
  mid-asm хуки, jump tables, особенности движка Doublesix, заметки по рендеру,
  лог загрузки. Это отдельные коммиты.
- Основной путь — rexglue:
    rexglue init --app_name south_park_td --app_root south-park-recomp
    (правишь south_park_td_config.toml: file_path -> private/default.xex,
     switch tables, save/restore регистры, longjmp/setjmp, functions,
     invalid_instructions)
    cmake --build --preset win-amd64-debug --target south_park_td_codegen
    cmake --build --preset win-amd64-debug
  Используй --force, чтобы проходить мимо неразрешённых вызовов и сходиться
  итеративно. Для jump tables сверяйся с XenonAnalyse.
- Если ранняя версия rexglue блокирует этап — переключи ЭТОТ этап на путь
  XenonRecomp + кастомный рантайм (рецепт Unleashed Recompiled) и продолжай.
- ПРОВЕРЯЙ ЗАПУСКОМ: собирай и реально запускай билд, смотри на кадр/поведение,
  сохраняй скриншоты и логи в knowledge-base. Не объявляй успех без проверки.

КОГДА СПРАШИВАТЬ ПОЛЬЗОВАТЕЛЯ (иначе действуй автономно и не останавливайся)
- Нужна установка с правами администратора или GUI-действие, которое нельзя
  сделать из CLI (например, поставить Clang 20+ / Vulkan SDK / PowerShell 7).
- Нужен внешний/платный ресурс, либо требуется решение по геймдизайну/объёму.
- Реальный юридический/этический стоп. В остальных случаях — не спрашивай.

ПЕРВЫЕ ШАГИ ПРЯМО СЕЙЧАС
1) Фаза 0: проверь наличие Clang 20+. Если нет и нельзя поставить из CLI —
   попроси пользователя установить LLVM/Clang 20+ (и pwsh 7). Собери и установи
   rexglue-sdk (cmake --preset win-amd64 ; --target install).
2) Фаза 1: извлеки private/default.xex из главного STFS-пакета
   (58410931/000D0000/...), извлеки ассет-дерево, классифицируй два пакета
   00000002 (нет ли там Title Update *.xexp), сделай XEX-recon и запиши
   результаты в knowledge-base/20-dump-analysis.md.
Дальше двигайся по PLAN.md.

DEFINITION OF DONE (условие завершения цели)
Игра собрана Clang-ом, запускается на этой машине и ПРОХОДИМА в одиночном
режиме от запуска до экрана победы/поражения: рендер корректен, звук работает
(XMA→SDL), геймпад управляет, сохранение/продолжение работает. Финальная
инструкция запуска зафиксирована в south-park-recomp/docs/RUN.md, краткий
итоговый отчёт — в knowledge-base. Только тогда цель считается достигнутой.
```

---

## Prompt (mirror, EN)

```text
Goal: bring "South Park: Let's Go Tower Defense Play!" (Xbox 360 / XBLA,
Title ID 58410931) to a playable state on THIS Windows 11 host via static
recompilation. Do not stop until the goal is met.

CONTEXT & LAYOUT
- You are in the super-repo. The dump is in "South Park - Let's Go Tower Defense
  Play!/" (git-ignored; NEVER commit game code/assets).
- Follow PLAN.md (phases 0->6). Analysis, decisions and risks: knowledge-base/.
- Primary toolchain: third_party/rexglue-sdk (codegen + runtime, D3D12 backend,
  own Xenos shader translation, XMA audio, SDL input, VFS).
- Reference/fallback: third_party/XenonRecomp and third_party/XenosRecomp.

SCOPE
- v1 = OFFLINE single-player (+ local co-op if cheap): boot -> menu -> match ->
  win/lose -> save -> continue, with audio and input.
- Out of scope: online co-op, leaderboards, achievement sync, avatar awards —
  stub xam/session APIs gracefully to "offline".

HOW TO WORK
- Iterate through PLAN.md phases. Every logical step is a SEPARATE commit,
  authored Claude Code:
  git -c user.name="Claude Code" -c user.email="noreply@anthropic.com" commit ...
  Commit into whichever sub-repo you change (south-park-recomp/, knowledge-base/)
  and bump the super-repo gitlink after a submodule advances. Keep it all local
  (bare remotes under ~/xbla-remotes; push there).
- Keep knowledge-base current (new kernel/xam imports, mid-asm hooks, jump
  tables, engine quirks, render notes, boot log) — as separate commits.
- Primary path is rexglue: `rexglue init` -> edit south_park_td_config.toml
  (file_path -> private/default.xex; switch tables; save/restore regs;
  longjmp/setjmp; functions; invalid_instructions) -> build the *_codegen
  target -> build the app. Use --force to converge past unresolved calls; cross-
  check jump tables with XenonAnalyse. If rexglue's early-dev state blocks a
  stage, switch THAT stage to the XenonRecomp + custom-runtime recipe.
- VERIFY BY RUNNING: actually launch the build, observe the frame/behavior, save
  screenshots and logs to knowledge-base. Never claim success unverified.

WHEN TO ASK THE USER (otherwise act autonomously, don't stop)
- Admin-rights install or a GUI action you can't do from the CLI (e.g. install
  Clang 20+ / Vulkan SDK / PowerShell 7).
- External/paid resource, or a game-design/scope decision. Real legal/ethical
  stop. Otherwise do not ask.

FIRST STEPS NOW
1) Phase 0: check for Clang 20+. If missing and not CLI-installable, ask the
   user to install LLVM/Clang 20+ (and pwsh 7). Build & install rexglue-sdk.
2) Phase 1: extract private/default.xex from the main STFS package
   (58410931/000D0000/...), extract the asset tree, classify the two 00000002
   packages (any Title Update *.xexp?), do XEX recon and record results in
   knowledge-base/20-dump-analysis.md. Then proceed through PLAN.md.

DEFINITION OF DONE
The game compiles with Clang, launches on this machine, and is PLAYABLE in
single-player from boot to a win/lose screen: rendering correct, audio working
(XMA->SDL), gamepad input working, save/continue working. The final run
instructions are recorded in south-park-recomp/docs/RUN.md and a short final
report in knowledge-base. Only then is the goal complete.
```

---

### Notes on realism

- This prompt expects **iteration**, not a one-shot. Phases 3–5 are where the
  time goes; the agent should keep looping, committing, and recording.
- It explicitly **bounds scope** (no online) so the goal is *achievable* — the
  one open-ended risk (netcode) cannot stall the Definition of Done.
- It names the **one human-in-the-loop** dependency up front (installing Clang),
  so the agent asks early instead of stalling.
- It keeps the **fallback** (XenonRecomp + custom runtime) explicit, so a rough
  edge in early-development rexglue cannot dead-end the whole effort.
