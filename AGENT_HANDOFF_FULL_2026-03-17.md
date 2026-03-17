# AGENT HANDOFF (FULL) - 2026-03-17

## 1) Purpose
This is a comprehensive transfer document for continuing work in a new agent with minimal context loss.
It captures architecture decisions, UI/firmware behavior decisions, operational conventions, unresolved items, and full commit ledgers.

## 2) Repositories and Current Heads
- Private repo: C:\Users\tarek\OneDrive\Documents\TalosHand
- Public repo: C:\Users\tarek\OneDrive\Documents\TalosHandTrackpadWeb
- Private branch: $privateBranch
- Public branch: $publicBranch
- Private HEAD: $privateHeadShort ($privateHead)
- Public HEAD: $publicHeadShort ($publicHead)

## 3) Canonical Cloud Endpoints
- Private repo remote: https://github.com/tarek-io/TalosHand
- Public repo remote: https://github.com/tarek-io/talos-control-center-web
- Public site: https://tarek-io.github.io/talos-control-center-web/

## 4) Stable Tags
### Private tags
trackpad-interface-baseline-v2
trackpad-interface-stable-six-gestures
trackpad-interface-stable-palm-reject
control-center-ui-stable-v1
control-center-ui-stable-v2
control-center-ui-stable-v3

### Public tags
control-center-ui-stable-v1
control-center-ui-stable-v2
control-center-ui-stable-v3

## 5) Collaboration and Process Decisions (Operational)
These process decisions were explicitly established and repeatedly followed:
- Work in two repos in sync: private source + public Pages mirror.
- Push private content commit and public content commit for each user-visible change.
- Because Pages can serve stale content, use cache-busting checks and, when stale, push an empty Trigger Pages redeploy commit.
- Verify live HTML/CSS/JS after each important visual/logic change before reporting done.
- Keep untracked 	mp/ in private repo untouched.
- Use branch prefix codex/ for new working branches.

## 6) Architecture Decisions (Control Center + Firmware)
### 6.1 Pipeline ownership
- Intermediate board owns: acquisition, processing, intent, command synthesis, RS-485 publication, telemetry stream.
- Master board owns: adaptive grasping execution, tactile interpretation, actuator/slave control.

### 6.2 Signal meaning separation
- drive != speed.
- drive = intent magnitude, speed = synthesized requested motion after threshold/mapping/caps/smoothing.

### 6.3 Current data model
- Dual-channel schema preserved (ch1/ch2, open/close) for compatibility.
- Telemetry families retained:
  - EMG_RAW
  - EMG_FILTERED
  - EMG_DRIVE
  - CONTROL_CMD
  - EVENT

### 6.4 Dashboard semantics
- EMG analysis lane is diagnostic; card-level values may use additional display smoothing.
- Adaptive grasping panel is simulation-ready scaffolding pending real tactile/master feedback.

## 7) Major Product/UI Decisions and Resulting State
### 7.1 Layout and hierarchy
- Final main hierarchy settled to 3 columns:
  - left: gesture input
  - center: EMG drive + EMG signal analysis
  - right: behavior monitor + adaptive grasping
- Behavior Monitor moved out of center hero position to right support column.
- EMG Signal Analysis renamed from generic "Signal Analysis".

### 7.2 EMG analysis philosophy
- Moved from "display-only" toward "instrument" with thresholds, event labels, and lane cleanup.
- Rows simplified over time to reduce redundancy and improve readability.
- Time-window behavior adjusted repeatedly (cycle-fit vs rolling) and settled with rolling-window expectations for continuity.

### 7.3 Closure command card
- Evolved into bidirectional display (open and close visible concurrently) with final decision state text.
- ON/OFF guide labels and lines added and repeatedly tuned for placement and readability.
- Multiple fixes for line visibility/thickness/perception, including pixel snapping and style unification attempts.
- Current style at HEAD restores brown ON/OFF color coding and neutral center line color separation.

### 7.4 Adaptive grasping panel
- Reworked multiple times (compact grid, long map, compact restore, merged card strategy).
- Current state: response content merged into primary adaptive card (not separate outer block), with fingertip section separate.
- Redundant summary text removed in inactive/monitoring and now also removed in active/contact states (no narrative sentence line).

### 7.5 Logs and typography
- Latest entry in behavior and gesture logs bolded.
- Empty-state placeholders added for logs.
- Repeated typography/spacing passes to harmonize headers, button alignment, and panel rhythm.

## 8) EMG Simulation + Realism Decisions
### 8.1 Simulation honesty
- Acknowledged and corrected top-down "authored derived" behavior toward raw-up pipeline behavior in demo.
- Processing path mirrored from firmware logic (rectify, envelope, normalize, threshold/hysteresis, command synthesis).

### 8.2 Dataset-informed calibration
- Ninapro DB1 used for calibration insight (not direct end-to-end target labels for proprietary derived outputs).
- Key realization: low visible speed was mapping/synthesis design, not only plot scaling.

### 8.3 Speed mapping and smoothing
- Mapping adjusted to be more responsive above threshold.
- Several stabilization iterations added at firmware + web mirror level.
- Final smoothing still explicitly marked as tunable/uncertain by user decision.

## 9) Hardware-Readiness Reality Check (Current)
Current firmware is not yet wired for real ADC acquisition in this project setup:
- .ioc currently includes I2C1 + USART3 + GPIO for trackpad path.
- No active ADC init/use path in current main.c loop for EMG hardware sampling.
- emg_acquisition.c still synthetic generator backend.

Implication:
- For single-electrode close-only bring-up, required work includes CubeMX ADC configuration + acquisition backend replacement + close-only intent logic mode.

## 10) Single-Electrode Close-Only Plan (Agreed Direction)
- Add ADC pin/peripheral config in TalosControl.ioc.
- Replace synthetic acquisition with live ADC sample path.
- Keep compatibility schema by publishing open-side as zero in transition mode.
- Intent resolution for this mode: close-or-neutral only (open disabled).
- Keep existing RS-485 packet schema and telemetry families for compatibility.

## 11) Risk Notes
- Smoothing: improved significantly, but risk remains of over-damping responsiveness when moved to live hardware.
- Closure guide visual consistency: multiple attempts made; current style restored per user preference (colors and shape preserved), but this area remains visually sensitive.
- Adaptive panel: now cleaner and less redundant, but still simulated tactile/controller behavior until hardware feedback exists.

## 12) Exact Continuation Checklist for Next Agent
1. Confirm branch and heads:
   - private/public both on codex/single-emg-close-only.
2. Preserve current UI visual language and spacing rhythm; avoid broad restyles.
3. Start single-electrode firmware implementation behind a clear mode/flag:
   - acquisition backend
   - intent engine close-only mode
   - telemetry compatibility values
4. Add a small validation script/serial sanity checklist for first hardware signal.
5. Do not delete or mutate untracked 	mp/.
6. Continue push pattern with Pages verification + redeploy nudge when stale.

## 13) Full Commit Ledger (Private) from control-center-ui-stable-v1 to current HEAD
`	ext
384efd0 Clamp gesture detector log to fixed compact height
2498222 Hide large monitor readouts and keep logs
567908b Tighten log panel vertical spacing
711bda5 Normalize log panel header spacing
666a696 Normalize panel spacing using shared stack gaps
e1f9c18 Tighten subsystem title spacing
3632c48 Increase behavior log viewport slightly
e7a5a01 Reduce gesture log viewport slightly
2fca613 Restore spacing below behavior monitor
da15dfb (tag: control-center-ui-stable-v2) Loosen EMG panel title spacing
479d115 Bold latest entries in activity logs
bfbdb33 Make demo drive ramps more human
c1fd0a6 Offset two-finger swipe contacts in demo
14f959e Normalize state title font sizes
9f99082 Add empty-state text for activity logs
8197bb9 Soften and smooth simulated drive envelope
0b8bda9 (codex/intermediate-emg-pipeline-v1) Add intermediate EMG pipeline and control timeline v1
e5feab3 Add session recording and replay controls
0700fdb Promote signal analysis into main dashboard
f6b054c Place signal analysis beside EMG drive
1ce175e Rebalance right-side control center layout
23f1ff8 Lighten and enlarge signal analysis view
ab0a197 Increase signal analysis label readability
0464733 Simplify signal analysis into smarter lanes
37aa849 Make demo control cycle reopen between grasps
8615717 Show EMG decision logic in signal analysis
d86260a Focus signal analysis on latest control cycle
1643fb5 Window signal analysis to a single control cycle
90f3101 Use fixed rolling window for signal analysis
18e9797 Simplify signal analysis lanes
e61d2c2 Make raw signal lane visible
eebb1d3 Make synthetic raw EMG waveform visible
b7ee7b6 Drive demo EMG through real pipeline
9ac8f50 Tune demo EMG profiles from Ninapro data
5a44378 Calibrate synthetic EMG amplitude to normalization
f172bb0 Slow demo EMG pacing
e8c12b1 Raise EMG activation threshold
f493fbf Add EMG release hysteresis
35c4f76 Refine signal analysis typography
c7f0033 Polish signal analysis labels
bae369e Split speed into its own signal lane
80809eb Refine speed lane readability
432f2fe Strengthen speed cap styling
1eb0406 Auto-scale drive and speed lanes
9fcffb4 Simplify signal lane labels
aaff467 Restyle signal lane labels
c592992 Clarify signal event labels
3f24d5b Refine signal analysis event labels
ae63ffc Latch repeated signal start markers
3340c18 Polish signal analysis lane styling
64e9e0d Make requested speed mapping more responsive
ff399b5 Debounce signal analysis stop markers
e46f13c Clarify trackpad mode switch events
0b6495c Redesign adaptive grasping panel
bbe1b63 Restructure dashboard into three columns
4bf901c Add speed scale to EMG drive panel
de7ef89 Move behavior monitor into right column
c182468 Tighten EMG drive panel height
d30ac7a Expand gesture detector log height
54ecbb1 Tighten gesture log and mode summaries
6faf9d1 Keep demo mode selection between gestures
64923f0 Tighten adaptive summary copy
726ead6 Rename signal analysis panel
7d84cdc Reflow adaptive response cards
7120788 Add white content cards across dashboard
0c59aa6 Increase behavior monitor height
fb83f9a Simplify EMG drive panel layout
6c50e23 Remove redundant speed readout
1fcb853 Show bidirectional EMG command signal
69c3846 Unify adaptive grasping into finger response map
27db314 Add closure command threshold markers
7676e8b Add dashboard pause control
0ac59d6 Refine adaptive grasping panel layout
eaf50e5 Simplify closure command threshold guides
ce1029e Restyle closure command threshold guides
2ea212f Restore compact adaptive grasping layout
4361f1e Add on and off closure guides
1b60fbb Strengthen closure guide visibility
cf359c6 Simplify pause button label
ba755cd Increase closure guide contrast
2d3d2ac Style disconnect button disabled state
b4b070c Normalize disabled button outlines
005209d Extend closure threshold lines
a3a63cf Match threshold lines to center guide
6eb3028 Use center-line threshold geometry
761929a Tighten closure command label spacing
c67ed4f Raise on labels and tighten axis
f24e578 Lift closure labels and axis
bd0cfa4 Fine tune closure label spacing
ffbdcfa Add breathing room to gesture header
4fbaeb7 Stabilize EMG drive and speed
f4b4630 Trim EMG analysis height slightly
b0a12fe Strengthen EMG drive smoothing
87b3b01 Slightly restore EMG analysis height
a57651c Align detector and behavior header buttons
29de3cc Increase EMG analysis height slightly
d535f4a Unify closure command guide thickness
386c6d0 Further reduce EMG drive jitter
5408da8 Snap closure guide lines to pixels
3e5aafd Snap closure center line to pixels
e4f9dea Unify closure guide line styling
70bd035 Increase EMG analysis height to 1060
46d2f00 Make closure guides render more uniformly
6b3471f Restore closure guide style and refine thickness
abb8f69 Merge adaptive response into summary card
7b91731 Restore closure threshold line colors
a37c7fc (tag: control-center-ui-stable-v3) Hide redundant adaptive waiting summaries
f6a908a (HEAD -> codex/single-emg-close-only, origin/main, origin/codex/single-emg-close-only, origin/HEAD, main) Remove adaptive summary descriptions
`

## 14) Full Commit Ledger (Public) from control-center-ui-stable-v1 to current HEAD
`	ext
721da8d Clamp gesture detector log to fixed compact height
e08ebe2 Hide large monitor readouts and keep logs
19f8625 Tighten log panel vertical spacing
6aaf172 Trigger Pages redeploy for tighter log spacing
1b8d006 Normalize log panel header spacing
3f126cf Normalize panel spacing using shared stack gaps
188b63c Tighten subsystem title spacing
97a56e8 Trigger Pages redeploy for subsystem spacing
9fdd733 Increase behavior log viewport slightly
26f6dac Trigger Pages redeploy for behavior viewport
90a8c25 Reduce gesture log viewport slightly
b61199e Trigger Pages redeploy for gesture viewport
323136e Restore spacing below behavior monitor
79c697b (tag: control-center-ui-stable-v2) Loosen EMG panel title spacing
29a9193 Bold latest entries in activity logs
68da0d0 Trigger Pages redeploy for latest log emphasis
6a30100 Make demo drive ramps more human
c48aa85 Trigger Pages redeploy for humanized demo ramps
0cd5e58 Offset two-finger swipe contacts in demo
072e660 Trigger Pages redeploy for swipe contact offset
8a0da48 Normalize state title font sizes
0c23d8e Add empty-state text for activity logs
355e1ea Trigger Pages redeploy for empty-state placeholders
5843210 Soften and smooth simulated drive envelope
77ec735 Trigger Pages redeploy for smoother demo envelope
116e9d4 Add control timeline and layered EMG telemetry
da92acd Trigger Pages redeploy for control timeline
42bd5cc Add session recording and replay controls
b0471a1 Trigger Pages redeploy for session recorder
d5eb92a Elevate signal analysis layout
e02d4ad Trigger Pages redeploy for signal analysis layout
50b7308 Align signal analysis beside EMG drive
01d05c9 Rebalance control center analysis layout
20a6513 Lighten and enlarge signal analysis view
df60ee2 Increase signal analysis label readability
4bca78e Simplify signal analysis into smarter lanes
5a3038e Make demo control cycle reopen between grasps
2983c6d Show EMG decision logic in signal analysis
0b8fe73 Focus signal analysis on latest control cycle
2ece870 Window signal analysis to a single control cycle
18f85d1 Use fixed rolling window for signal analysis
ed7edd8 Simplify signal analysis lanes
078f942 Make raw signal lane visible
bb9df71 Make synthetic raw EMG waveform visible
441d616 Drive demo EMG through real pipeline
1e63622 Tune demo EMG profiles from Ninapro data
6cf56d9 Calibrate synthetic EMG amplitude to normalization
b5db6e7 Slow demo EMG pacing
e884d36 Raise EMG activation threshold
7ab822a Add EMG release hysteresis
6a48aa6 Refine signal analysis typography
6b85095 Polish signal analysis labels
4ef4f01 Split speed into its own signal lane
0e2466d Refine speed lane readability
20309f1 Strengthen speed cap styling
1fca0b6 Auto-scale drive and speed lanes
47df5fd Simplify signal lane labels
dea677b Restyle signal lane labels
89f1da0 Clarify signal event labels
92cf6c3 Refine signal analysis event labels
d473c5a Latch repeated signal start markers
71bb6c8 Polish signal analysis lane styling
5169c9a Make requested speed mapping more responsive
d55d3f2 Debounce signal analysis stop markers
e7e0f44 Clarify trackpad mode switch events
48f7553 Redesign adaptive grasping panel
f2109c5 Restructure dashboard into three columns
c198d30 Add speed scale to EMG drive panel
7f3e658 Trigger Pages redeploy
a5870bf Move behavior monitor into right column
601123d Tighten EMG drive panel height
af9ddf9 Expand gesture detector log height
c38d081 Tighten gesture log and mode summaries
6110db4 Keep demo mode selection between gestures
775fced Tighten adaptive summary copy
397d17f Rename signal analysis panel
98f8698 Reflow adaptive response cards
37aab13 Add white content cards across dashboard
be6ce70 Increase behavior monitor height
e0f329e Simplify EMG drive panel layout
0a2c73c Remove redundant speed readout
a278263 Show bidirectional EMG command signal
9551693 Unify adaptive grasping into finger response map
2c8d058 Trigger Pages redeploy
ad78d5c Add closure command threshold markers
f319df4 Trigger Pages redeploy
625c970 Add dashboard pause control
3d94b1e Trigger Pages redeploy
85df72d Refine adaptive grasping panel layout
28e19de Trigger Pages redeploy
8814ff7 Simplify closure command threshold guides
115bdfd Trigger Pages redeploy
898542b Restyle closure command threshold guides
68147d6 Trigger Pages redeploy
220482b Restore compact adaptive grasping layout
251ef2d Trigger Pages redeploy
2b95142 Add on and off closure guides
c0faa48 Trigger Pages redeploy
8aec58b Strengthen closure guide visibility
7bb05f9 Trigger Pages redeploy
aae5591 Simplify pause button label
d060f72 Trigger Pages redeploy
1ec1bd2 Increase closure guide contrast
e506ca1 Trigger Pages redeploy
e66980b Style disconnect button disabled state
16f16ee Trigger Pages redeploy
8e96ba2 Normalize disabled button outlines
3d5d783 Trigger Pages redeploy
2dd4701 Extend closure threshold lines
67f4fd9 Trigger Pages redeploy
0225894 Match threshold lines to center guide
58381b2 Use center-line threshold geometry
ccfd651 Trigger Pages redeploy
56621f2 Tighten closure command label spacing
c21b109 Trigger Pages redeploy
a07d378 Raise on labels and tighten axis
4fd5db0 Trigger Pages redeploy
7a2f037 Lift closure labels and axis
7566a3f Trigger Pages redeploy
4ed951d Fine tune closure label spacing
92895e8 Trigger Pages redeploy
7f05170 Add breathing room to gesture header
8da432e Trigger Pages redeploy
9079af9 Stabilize EMG drive and speed
9f4ccac Trigger Pages redeploy
6e535a9 Trim EMG analysis height slightly
68a8b54 Trigger Pages redeploy
5dc9958 Strengthen EMG drive smoothing
204cfa8 Trigger Pages redeploy
da53263 Slightly restore EMG analysis height
eba1679 Trigger Pages redeploy
c34764e Align detector and behavior header buttons
f2656c1 Trigger Pages redeploy
ddf8bd8 Increase EMG analysis height slightly
e67e284 Trigger Pages redeploy
716f8e5 Unify closure command guide thickness
890e0e6 Trigger Pages redeploy
9278caf Further reduce EMG drive jitter
f6f1ac9 Trigger Pages redeploy
64c343b Snap closure guide lines to pixels
fc2cdf3 Trigger Pages redeploy
ff91c9e Snap closure center line to pixels
e5330fb Trigger Pages redeploy
fe37bbf Unify closure guide line styling
a1b65c2 Trigger Pages redeploy
6b44e04 Increase EMG analysis height to 1060
4ec78d6 Trigger Pages redeploy
3bbdb5e Make closure guides render more uniformly
4309b74 Trigger Pages redeploy
aa08826 Restore closure guide style and refine thickness
1aa9bcb Trigger Pages redeploy
5048aa7 Merge adaptive response into summary card
fb781d2 Trigger Pages redeploy
9263926 Restore closure threshold line colors
994a5da Trigger Pages redeploy
ea4565a Hide redundant adaptive waiting summaries
148e604 (tag: control-center-ui-stable-v3) Trigger Pages redeploy
244deb6 Remove adaptive summary descriptions
9e62e04 (HEAD -> codex/single-emg-close-only, origin/main, origin/codex/single-emg-close-only, main) Trigger Pages redeploy
`

## 15) Handoff Notes for Another Agent Prompt
Suggested bootstrap prompt for next agent:
- "Use AGENT_HANDOFF_FULL_2026-03-17.md as source of truth. Continue on branch codex/single-emg-close-only. Implement single-electrode close-only EMG ingestion (ADC-backed) while preserving current telemetry schema and dashboard compatibility. Keep UI design language unchanged unless explicitly requested."
