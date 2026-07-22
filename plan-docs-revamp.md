# DimOS Docs Revamp Plan (hackathon edition)

Target repo: `swstica/mintlify-docs` (this repo). Push straight to main, no review cycle.
Code source of truth for verifying claims: `~/dimos2` (dimos repo).
Deadline: hackathon starts tomorrow. Priorities are ordered so we can stop anywhere after P0 and still have shippable docs.

Audience for this revamp: a hackathon participant who has never seen dimos. Every page must serve "install, run, understand, build" - not internal benchmarking or contributor workflow.

## 1. Target sidebar (full site, one screen)

```
Getting started
  Introduction              <- rewrite: what DimOS is, honest capability map
  Quickstart                <- rewrite: ONE happy path
  Requirements
  Installation: Ubuntu / macOS / Nix

Build with DimOS
  How DimOS fits together   <- NEW: modules/streams/topics/blueprints mental model
  Modules
  Streams                   <- merged data_streams + sensor_streams
    Overview, ReactiveX, Storage & replay, Temporal alignment,
    Quality filter, Advanced patterns
  Blueprints
  Configuration
  CLI reference

Agents                      <- promoted to top-level, skills-first story
  What an agent is          <- rewrite of capabilities/agents
  Skills & MCP              <- NEW: @skill, tool schema, mcp list-tools / call
  Tutorial: drive the Go2 with language   <- NEW (kitchen/nav demo)
  Tutorial: add your own skill            <- NEW (the hackathon money page)

Capabilities
  Memory
    Three kinds of memory   <- NEW landing: SpatialMemory / memory2 / TemporalMemory
    Record & replay (memory2)   <- rework of current index.md
    Offline analysis (optional) <- current plot.md, honestly labeled
    memory2 reference           <- promoted from dimos/memory2/*.md
  Navigation (keep: index, deep_dive, relocalization)
  Manipulation (restructured - see P1 item below)
    Overview                <- rewritten: what dimos offers (planning, teleop,
                               teach mode, imitation learning), not a feature braindump
    Collect data & train    <- NEW: teach mode / VR / keyboard -> dataprep -> LeRobot -> train -> deploy
    VR teleoperation        <- promoted from dimos/teleop/quest/README.md
    Adding a custom arm     <- keep (arm-agnostic, stays in capability)
  Perception              <- fill the stub with one honest page
  Teleoperation (keep)

Platforms
  Unitree Go2 (index, setup, simulation)
  Unitree G1
  Arms
    Galaxea A1Z             <- NEW: setup, teach mode, record, dataprep config
    A-750                   <- moved from Capabilities/Manipulation
    OpenArm                 <- moved from Capabilities/Manipulation

Going deeper
  Transports (slimmed: concepts move to "How DimOS fits together")
  Transforms / TF
  Visualization
  Native modules
  Tool streams
  Camera calibration
  Python API
  LCM messages

Contribute (unchanged, bottom of nav)
  development/* as-is
```

Rules encoded in this IA:
- One learning path top to bottom: install -> mental model -> core primitives -> agents -> capabilities.
- Core vs niche separated: camera calibration and native modules never sit next to Modules again.
- Agents is a first-class product track, not a one-page capability.
- Everything under "Going deeper" is optional; nothing in the top three groups links into it as a prerequisite.

## 2. Kill list (delete files + nav entries)

| Target | Reason |
|---|---|
| `usage/sensor_streams/` (6 pages + assets) | Byte-identical fork of `data_streams/`. Keep one tree, rename group to "Streams". Add redirects. |
| `coding-agents/` (8 pages) | Already orphaned (redirect-only, not in nav). Contributor tooling docs, wrong audience for this repo. |
| `capabilities/memory/algo_comparison.mdx` | Internal microbench ("new algo strictly better"). Not product docs. |
| `capabilities/memory/demo_rerun.py` | Stray script in docs tree. |
| `usage/index.mdx` (dead TOC lobby) | Replaced by the new "How DimOS fits together" page. |

## 3. Merge / move list

| From | To | Notes |
|---|---|---|
| `usage/data_streams/*` | `usage/streams/*` | Rename dir, retitle "Streams", fix self-links. |
| Concepts half of `usage/transports/index.mdx` (541 lines) | New `usage/how-dimos-fits-together.mdx` | This page already explains module/stream/topic/transport better than anything else on the site. Mine it, leave transports a slim "choosing and configuring transports" page. |
| `dimos/memory2/{intro,architecture,streaming,embeddings}.md` (dimos repo) | `capabilities/memory/reference/*` | Best memory docs currently invisible on the site. Copy over, light edit. Skip `notes.md`. |
| `capabilities/memory/plot.mdx` | `capabilities/memory/offline-analysis.mdx` | Keep but retitle and add one honest intro line: this is offline notebook analysis of recorded data, not live robot memory. |

## 4. New pages to write (the real work)

### P0 - must exist before hackathon
1. **Quickstart rewrite** (`quickstart.mdx`)
   - One happy path: install (pick ONE blessed method, link the others) -> `dimos run unitree-go2 --replay` -> see Rerun -> `dimos run unitree-go2-agentic` -> `dimos agent-send "..."`.
   - Fix the lie: `unitree-go2-memory` is a sensor recorder, never call it "temporal memory replay".
   - Fix all 7 raw `/docs/*.md` and `/AGENTS.md` links.
2. **Agents track** (3 pages)
   - "What an agent is": agent = LLM loop calling @skill methods over MCP; the robot stack stays modules + blueprints. Include the real control loop (human_input -> McpClient -> McpServer tools/call -> RPC -> str|image back). Remove the false "subscribes to camera, LiDAR, odometry" claim; perception is pull-based through skills.
   - "Skills & MCP": @skill docstring = tool schema, skill containers, prove-it-without-an-LLM via `dimos mcp list-tools` / `mcp call`.
   - "Add your own skill" tutorial: the 9-step path from the critique, verified against `unitree_go2_agentic.py` in the dimos repo. This is the page hackathon teams will actually live in.
3. **Memory landing** (`capabilities/memory/index.mdx`)
   - "Three kinds of memory" table: SpatialMemory (live place map, powers navigate_with_text), memory2 (episode store, record/replay/offline search), TemporalMemory (experimental).
   - Blueprint disambiguation table: `unitree-go2-memory` = recorder, `unitree-go2-temporal-memory` = experimental, `unitree-go2-agentic` = live SpatialMemory path.
   - Honest limits paragraph: no full object permanence.
4. **New docs.json** implementing the sidebar above + redirects for every killed/moved path.
5. **Galaxea A1Z platform page** (`platforms/arms/a1z/index.mdx`)
   - Setup (host setup, CAN, safety protocol), teach mode (zero-gravity drag teaching),
     replay, camera recording, where recordings land.
   - Source material: branch feat/galaxea-a1z-adapter (`dimos/robot/manipulators/a1z`,
     `a1z_scripts/HANDOFF.md`, teach/replay commits). This is the hackathon hardware, so it is P0.

### P1 - strongly want
5. **How DimOS fits together** - mined from transports/index, one diagram, module -> stream -> blueprint -> run.
6. **Streams overview rewrite** (`usage/streams/index.mdx`) - currently a 43-line stub; make it explain what a stream is and route to subpages.
7. **Introduction rewrite** - drop marketing cards that overclaim (spatial memory card sells object permanence); cards must match what `dimos list` ships.
8. **Tutorial: drive the Go2 with language** - kitchen demo: agentic blueprint + tag_location + navigate_with_text.
9. **Navigation restructure** - current pages explain how nav works, not how to use it. New order:
   - Overview (reframed index): what nav does + the three goal surfaces
   - "Using navigation" (NEW task guide): run a nav blueprint, then send goals via
     (a) click in the Rerun viewer (`clicked_point` from the websocket server),
     (b) `NavigationInterfaceSpec` from Python: `set_goal` / `get_state` / `is_goal_reached` / `cancel_goal`,
     (c) language skills via the agent (`navigate_with_text`, `tag_location` - link to Agents tutorial, do not duplicate)
     plus when to premap vs live-map
   - Premap & relocalization (keep as-is)
   - "How navigation works" (deep_dive renamed, moved last, explicitly optional)
   Principle applies sitewide: one task page per capability from the user's point of view; deep dives demoted below it.
10. **Manipulation restructure** (per manip-lead critique)
   - Move A-750 + OpenArm pages to Platforms > Arms; add Galaxea A1Z beside them.
   - Rewrite index as a product story: what dimos manipulation offers today -
     classical planning (Drake/RoboPlan), three teleop options honestly ranked
     (teach mode > VR Quest > keyboard; keyboard demoted, never the lead), and the
     imitation-learning loop. Not a feature braindump.
   - NEW "Collect data & train a policy" guide, promoted from dimos/learning/README.md
     (already on main, invisible on Mintlify) + branch A1Z teach-mode flow:
     collect (teach mode or Quest) -> session db -> `dimos dataprep` -> LeRobot dataset.
     Training/deploy section: LeRobot train commands (ACT baseline), marked
     "in progress" where the dimos-side deploy story is not yet landed.
   - Promote dimos/teleop/quest/README.md to a VR teleoperation page.
   - One-line tip in the collection guide: keep the operator out of camera frame
     during teach-mode demos (already handled by our camera placement).

### P2 - if time remains
10. Perception page: replace 5-line stub with one honest page of what perception ships today.
11. Sitewide link pass (no github blob links where a site page exists; extensionless Mintlify links everywhere).
12. Tighten platform pages and installation pages (they are fine, just trim).

## 5. Content honesty rules (apply to every touched page)

- Never present memory2 notebook analysis as "robot memory".
- Never claim continuous agent perception subscriptions; skills are pull-based.
- Blueprint names quoted exactly as `dimos list` prints them.
- Every command shown must be runnable as written from a fresh install.
- No page may require reading a "Going deeper" page to complete its task.

## 6. Execution order

1. Mechanical pass (delete + move + rename + redirects + new docs.json skeleton). Site builds again with old content in new shape.
2. P0 writes (quickstart, agents x3, memory landing).
3. P1 writes.
4. Link + honesty pass over every surviving page.
5. `mintlify dev` local check, then push to main.

Verification claims against code happens during writing, against `~/dimos2`:
- `dimos/robot/unitree/go2/blueprints/agentic/unitree_go2_agentic.py`
- `dimos/agents/mcp/{mcp_client,mcp_server}.py`
- `dimos/agents/system_prompt.py`
- `dimos/perception/spatial_perception.py`
- `dimos/memory2/`
