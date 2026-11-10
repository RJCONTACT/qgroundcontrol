# Black Blade GCS — Decision Log

Running record of every major decision on Black Blade GCS (a redesign of QGroundControl).
Written to be turned into two portfolio case studies later:

- **Product UX case study** — the design problem, options, decisions, before/after, outcome.
- **Strategic Design Manager (SDM) case study** — product vision and positioning, design principles,
  design-system strategy, build-vs-adopt bets, scope trade-offs, and how design impact is measured.

## Rules for this log

1. One entry per decision that changed direction, scope, tooling or design. Skip trivial choices.
2. Record **who made the call** — Rishik, Claude (proposed, accepted by Rishik), or joint. Case studies use "I", so this must stay honest.
3. Never write a number that was not measured. If a metric is planned, put it under "To measure".
4. Anything inferred and not yet confirmed by Rishik is marked **[to confirm]**.
5. Link evidence (commit, PR, build run, Figma/Claude Design frame, screenshot path) in every entry.
6. Failures go in "What didn't work" — they are the most valuable part of a case study.

## Entry template

```md
### D-XXX · <Decision as an action, e.g. "Build on a QGC fork instead of from scratch">
- Date:
- Status: Proposed | Decided | Reversed | Superseded by D-YYY
- Decided by:
- Lens: UX | Strategic | Both

**Context** — what situation forced a decision (2–3 lines).
**Problem / evidence** — what was wrong or unknown, and how we know.
**Options considered**
| Option | Pros | Cons |
|---|---|---|
**Decision** — what was chosen.
**Rationale** — why this option won.
**Trade-offs accepted** — what we gave up.
**Risks** — what could go wrong, and the mitigation.
**Evidence / artefacts** — links, commits, screenshots.
**To measure** — how we will know it worked.
**Case-study angle** — the one-line story this decision tells.
```

---

## Project meta (for the case-study header)

| Field | Value |
|---|---|
| Project | Black Blade GCS — "the most user-friendly and cool-looking GCS" |
| Base | Fork of QGroundControl (`mavlink/qgroundcontrol` → `RJCONTACT/qgroundcontrol`) |
| Role | Rishik Jha — product/UX lead and owner; Claude as AI engineering partner |
| Team size | 1 human + AI |
| Started | 2026-11-10 |
| Status | Setup |
| Primary users | [to confirm] — drone operators / pilots |

---

## Decisions

### D-001 · Build Black Blade on a fork of QGroundControl
- Date: 2026-11-10
- Status: Decided
- Decided by: Rishik
- Lens: Both

**Context** — Black Blade needs flight-critical features (MAVLink comms, vehicle state, mission planning, video) before any UX work can be shown working.
**Decision** — Fork QGroundControl and redesign on top of it instead of building a GCS from scratch.
**Rationale** [to confirm] — QGC already handles PX4 and ArduPilot vehicles, simulation and mission planning, so the effort goes into UX rather than plumbing.
**Trade-offs accepted** [to confirm] — inherits QGC's architecture and QML UI stack; must stay mergeable with upstream to keep receiving fixes.
**Risks** — upstream drift; licensing obligations (QGC is dual-licensed Apache 2.0 / GPLv3) — see Open questions.
**Evidence** — https://github.com/RJCONTACT/qgroundcontrol
**Case-study angle** — Choosing to stand on a proven open-source core so design effort goes where operators feel it.

### D-002 · Keep `master` as a clean upstream mirror; all work on `blackblade/ui`
- Date: 2026-11-10
- Status: Decided
- Decided by: Claude proposed, Rishik accepted
- Lens: Strategic

**Context** — Need a way to change the UI while still pulling upstream QGC fixes.
**Decision** — `master` mirrors upstream untouched; all Black Blade work goes on `blackblade/ui`, tracked by internal PR #1 (fork-only, not sent upstream).
**Rationale** — Upstream syncs stay simple; every Black Blade change is reviewable as one diff against stock QGC.
**Trade-offs accepted** — Must rebase/merge `blackblade/ui` onto new upstream periodically.
**Evidence** — https://github.com/RJCONTACT/qgroundcontrol/pull/1
**Case-study angle** — Treating a personal project with production branching discipline.

### D-003 · Test with CI-built Windows installers plus a flight simulator
- Date: 2026-11-10
- Status: Decided
- Decided by: Rishik (chose from three options Claude laid out)
- Lens: Both

**Context** — QGC is a native Qt/QML desktop app; changes must be compiled before they can be seen.
**Options considered**
| Option | Pros | Cons |
|---|---|---|
| A. GitHub Actions builds a Windows installer per change | No local setup; real app; tests on own PC with simulator | ~40–60 min per build; slow for small visual tweaks |
| B. Local Qt build on Rishik's PC | Fastest iteration (1–2 min rebuilds) | ~10 GB Qt install, compiler setup, 30+ min first build |
| C. Claude builds in the cloud and sends screenshots | Zero setup | Static images only; small machine, slow first build |
**Decision** — Option A, with screenshots (C) for small changes in between.
**Trade-offs accepted** — Slower feedback loop per change.
**To measure** — Time from code change to installable build; number of design iterations tested per week.
**Evidence** — First baseline build: https://github.com/RJCONTACT/qgroundcontrol/actions/runs/42862053858
**Case-study angle** — Picking a feedback loop that fits a solo designer's setup, and knowing what it costs.

### D-004 · Design tool for concepts and the design system
- Date: 2026-11-10
- Status: Open
- Decided by: —
- Lens: UX

**Context** — QGC's UI is QML, not HTML, so any design tool is a reference that gets rebuilt by hand in QML.
**Options considered**
| Option | Pros | Cons |
|---|---|---|
| Claude Design (+ Design System artifact) | Fast concept exploration; tokens map onto QGC's `QGCPalette` theme | Newer; less precise for detailed manual work |
| Figma | Rishik's fastest tool; precise component work; portfolio-ready assets | Every concept drawn by hand |
| Paper | HTML-native canvas | Its HTML advantage does not carry to QML; not connected |
**Recommendation (Claude)** — Claude Design for concepts and the token system; Figma for screens that need hand refinement.
**Decision** — Pending Rishik.

### D-005 · Simulator: PX4 SITL in WSL2 on Rishik's Windows PC
- Date: 2026-11-10
- Status: Decided
- Decided by: Claude proposed, Rishik accepted
- Lens: Both

**Context** — Need a simulated drone to exercise the GCS without hardware.
**Options considered**
| Option | Pros | Cons |
|---|---|---|
| PX4 SITL + Gazebo in WSL2 | QGC's primary autopilot; 3D sim; official Windows path | WSL2 install + restart; WSL IP changes on restart unless mirrored networking is on |
| ArduPilot SITL | Strong Windows tooling | Different flight-mode model from PX4 |
**Decision** — PX4 SITL with Gazebo (`gz_x500` quadcopter), following D-006.
**Risks** — Windows QGC may not auto-connect to WSL2. Rishik is on Windows 11, so WSL mirrored networking
(`networkingMode=mirrored` in `.wslconfig`) is the mitigation; a manual UDP link on port 18570 is the fallback.
**Evidence** — Set up on 2026-11-10 by Claude Code on Rishik's PC from one prompt (~22 min, unattended after launch):
WSL 2.6.3.0, Ubuntu 22.04.5, PX4 main `8d5be2b9cb` (v1.18.0-beta1), Gazebo Harmonic 8.15.0. Smoke test passed:
MAVLink streaming to UDP 14550. Hyper-V firewall inbound set to Allow (was NotConfigured).
Launcher: `Start PX4 Sim.cmd` on the Desktop; details in `px4-setup-status.txt`.

### D-006 · Design for PX4 first; keep ArduPilot compatible but out of design scope for v1
- Date: 2026-11-10
- Status: Decided
- Decided by: Claude proposed, Rishik accepted
- Lens: Strategic

**Context** — QGC supports both autopilots, but flight-mode names and behaviour, parameters and Setup screens differ.
Designing for both roughly doubles the edge cases in Fly View and Setup.
**Options considered**
| Option | Pros | Cons |
|---|---|---|
| PX4 primary | QGC's reference platform; best simulator support in QGC; professional-operator positioning | Excludes ArduPilot-specific modes from v1 design |
| ArduPilot primary | Large community; users mostly on Mission Planner, so stronger differentiation story | Weaker fit with QGC's reference workflows |
| Both equally | Widest reach | Not realistic for a solo designer |
**Decision** — PX4 is the design and test target for v1. ArduPilot must not break, but gets no dedicated design work.
**Rationale** — Fits D-001 (QGC base) and D-005 (PX4 simulator); a focused v1 scope for one designer.
**Trade-offs accepted** — ArduPilot operators get no tailored experience in v1.
**Risks** — Target users may turn out to fly ArduPilot; mitigation is to revisit after the first Fly View redesign.
**Case-study angle** — Narrowing scope deliberately so one designer can ship a coherent v1.

---

## What didn't work

### W-001 · Forked repo would not register the Windows build workflow
- Date: 2026-11-10
- **What happened** — After enabling Actions on the fork, GitHub registered only 24 of 35 workflow files; Windows was missing, so a build could not be started.
- **Tried** — Waiting and re-checking; pushing a no-op commit to `blackblade/ui`. Neither registered it.
- **Fix** — Opening internal PR #1 fired a `pull_request` event, which registered all workflows; the build was then started manually.
- **Lesson** — Fork CI is not plug-and-play; budget setup time for build infrastructure before design work starts.

### W-002 · First Windows build compiled but could not package an installer
- Date: 2026-11-10
- **What happened** — The baseline build compiled, then failed at "Create Installer": the app version resolved to
  0.0.0 because the fork was created without upstream's tags, and QGC refuses to package a 0.0.0 build.
- **Tried** — Pushing upstream's `v5.2.0-dev` tag to the fork; blocked from Claude's workspace.
- **Fix** — Added a fork-only step to the Windows workflow on `blackblade/ui` that fetches upstream tags before
  building (commit `968a16374`).
- **Lesson** — "Copy master branch only" when forking drops tags the build depends on; a one-line CI change fixed it.

---

## Open questions

- Primary operator profile and mission type for Black Blade.
- First screen to redesign — Fly View recommended (where operators spend most of their time).
- Distribution and licensing: QGC is dual-licensed Apache 2.0 / GPLv3; confirm which applies if Black Blade is ever shared or sold.

---

## Case-study mapping

| Case-study section | Pull from |
|---|---|
| Title (action + result), summary | Decisions marked Decided + outcomes |
| Meta row | Project meta |
| My contribution | "Decided by" fields where Rishik made the call |
| Proof near top | "To measure" items once measured; shipped builds |
| Context / problem with evidence | D-001 context; usability findings (to be added) |
| Options → decision (1–3 deep) | Strongest 1–3 decisions from this log |
| What didn't work | W-entries |
| Final design, before/after | Stock QGC vs Black Blade screenshots, linked from entries |
| Outcome / reflection | Final review entry (to be added) |

**Strategic Design Manager lens**: D-001 (build vs. adopt), D-006 (autopilot positioning), D-004 (design
tooling and design-system strategy), D-003 (feedback-loop trade-off), W-001 (infrastructure risk), plus design
principles, vision and impact measures as they are set.
