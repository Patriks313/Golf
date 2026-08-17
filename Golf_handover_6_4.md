
/
Golf app

Beta
Golf app



Type / for skills
How can I help you today?




Pinned
Hide
Digitalizing a golf trainer with phone camera
Apr 20
Recents
Analyzing P1 and P7 frames with video-JSON drift
May 8
CAOS thread: Validating wristX motion detection parameters
May 8
0.4U implementing wristY
May 7
0.4T Reviewing brief v4.4 for validation
May 7
0.4S Merging project briefs 4.3 and 4.4 for iPad
May 7
0.4R Extracting Px frames from golf swing video
May 7
0.4Q Missing frames analysis in session brief
May 6
0.4P (DIED) Missing frames in JSON pose data
May 6
0.4O Consolidating documentation into single brief
May 3
O.4N Brief 3.11 consistency review and v0.4.2 conflicts..
May 3
0.4M Lost frames 2.0 next steps
May 3
0.4L second missing frames
May 3
0.4K update brief on double frames JSON vs video
May 3
0.4J Adjusting P7 algo
May 2
0.4I Document reading order for context building
May 1
0.4H Check slomo cam
May 1
0.4G Velocity mitigation documentation review
May 1
0.4F Multiple swing download from phone video
Apr 30
0.4D Separating recording from analysis workflow
Apr 30
0.4C Elbow method for segmentation analysis
Apr 30
Show more
Instructions
We're building a web-first AI golf coaching app that uses phone camera + MediaPipe Pose to analyze swings and give hands-free voice + audio-tone feedback. Target user is an improver, not a beginner. Read the project brief for full context and decisions made so far. When we make new decisions, remind me to update the brief. Background info: I'm not technical, so please spare me the deep technical discussion, I will defer those decisions to you. Testing is done at home with an upside down club i.e shadow swings. All filming, if not specified otherwise, are done with iphone14 front camera ("selfie cam") standing format. Some testing is done on driving range.

Memory
Only you
Purpose & context Patrik is the sole developer and product owner of Golf Coach App — a voice-driven, hands-free golf swing analysis web app. It runs as a single index.html file (vanilla JS, no build step) hosted on GitHub Pages at golf.greblats.com, using MediaPipe Pose for skeleton tracking and iPhone Safari + AirPods as the primary target environment. The app records swings via phone camera on a tripod (~20–30cm high, steep upward angle, 1–2m from golfer), detects four swing phase gates (P1/address, P4/top, P7/impact, P10/finish) mapped to the Mac O'Grady P-position model, and delivers per-swing audio coaching feedback. Core architecture has a deliberate plumbing vs. drills seam: capture pipeline, voice loop, swing segmenter, and session review are plumbing; self-contained analysis modules (Tempo, Head Stability) are drills. A third drill is gated on the segmenter reaching >95% success rate on a labeled dataset. The project uses a versioned brief as the canonical Claude-to-Claude handoff document. Brief is currently at v4.7 (May 7, 2026, post first v0.4.1.13 validation pass). Brief filenames follow normal version numbering (no digit reversal). Claude's role is to drive technical decisions and produce concrete plans; Patrik serves as visual validator and golf domain expert. --- Current state Shipped: v0.4.1.13 — implements the v4.5 hypothesis: P1 detection switched from wristY to wristX motion-onset; P7 switched to wristX-return-through-P1-x as primary detector, legacy median (shoulderRot, wristY, elbowY) as backup. P4/P10 remain on wristY. Expanded diagnostic fields on every swing record; smoothed wristX trace persisted alongside wristY. First validation pass (May 7, two sessions, 6 shadow swings): P1 frames: visually correct across all 6 swings ✓ P7 frames: consistently late by ~5–6 frames (~140–180ms), landing in early follow-through rather than at impact ✗ Root cause confirmed: The wristX-return detector is mathematically functioning as designed, but MediaPipe wrist landmark data is unreliable during the high-speed impact window. MediaPipe reports stale trail-side wrist positions during blur frames with visibility scores ~0.6 — not low enough to trigger quality rejection. The detector only sees the wristX crossing when MediaPipe re-acquires the wrists post-blur, which is already in follow-through. Visually correct frames cannot be identified mathematically because the underlying MediaPipe data for those frames is bad. The v4.5 P7 hypothesis is structurally suspect; a different anchor is needed. Critical open infrastructure issue: extractgates.py does not account for the v0.4.1.11+ JSON↔video time offset. Video starts ~1.7s before pose collection begins (MediaRecorder warmup). Naive use produces visually-identical pre-swing setup frames at all gates — a silent failure mode. Workaround: per-session calibration using offset = videoduration − JSONspan, then apply offset to all gate timestamps. Empirical offset on May 7 sessions: 1727ms. Brief v4.7 Validation Technique section documents this. Patch to extractgates.py is still open. --- On the horizon P7 detector redesign: wristX-return-through-P1-x is validated as the wrong approach given MediaPipe's impact-window data quality. A new P7 anchor is needed that doesn't rely on wrist landmark accuracy during blur frames. P1 eyeball sign-off: Never completed on the May 7 sessions — needs a formal validation pass. Larger dataset validation: May 1 outdoor 12-swing session not yet run through the v0.4.1.13 detector. extractgates.py offset patch: Still owed; until patched, all validation extraction requires manual per-session calibration. Brief structural reorganization: Active blockers section flagged as confusing to read; deferred to after validation data is available. --- Key learnings & principles MediaPipe wrist data is unreliable at impact. Visibility scores ~0.6 during blur frames are not low enough to trigger rejection, but the coordinates are garbage (30%+ frame-width jumps per frame, stale positions). Any detector relying on wrist landmarks at impact will fire late. The wristX-return hypothesis is structurally wrong for P7. wristX moves monotonically through the impact region; any "address-X closest to impact-X" math produces the same systematically-late frame the current detector already picks. Refining the math is not the solution. P1 and P7 are horizontal-motion gates being detected on the wrong axis in prior versions (vertical wristY). P4/P10 are correctly detected on wristY (vertical hinge motion dominates). This axis-mismatch framing from v4.5 remains valid for P1. Silent failures are the dangerous failure mode. The extractgates.py offset bug is a prime example: setup frames look like plausible address postures, so wrong extractions pass eyeball inspection. The v0.4.1.10 addressMatch P7 detector fired systematically early and also looked plausible. Visual frame validation is mandatory before accepting any algorithmic claim. No segmenter conclusion is trustworthy without eyeball sign-off on extracted video frames. JSON analysis alone consistently misleads. addressMatch quality is bounded by P1 correctness. Any P7 detector that uses the address frame as a reference is directly limited by how accurate P1 detection is. Validation sessions should be fresh conversations. Building and validating in the same thread produces chaotic, verbose output. Patrik explicitly prefers new threads for validation work. --- Approach & patterns Document-driven continuity: Versioned project brief is the canonical handoff. Each session ends with a brief update. Brief is read at the start of every new session before any work begins. Ship then validate: Code ships first; field validation (eyeball sign-off on extracted frames) follows in a subsequent session/thread. Patrik's validation role: Patrik provides visual ground truth on extracted frame contact sheets. Claude interprets and acts on verdicts without requesting implementation decisions from Patrik. Conciseness preference (explicitly stated): Future sessions must be significantly more concise. Use graphs and visuals rather than number tables. Keep outputs precise and direct. Avoid burying key findings in long explanations or large number dumps without visual context. One swing / one topic at a time when presenting validation results; avoid overwhelming feedback. Outputs sized for iPad: Contact sheets and file outputs should be readable on iPad screen real estate. No pre-committed directions in the brief when a design decision hasn't been made yet. State the finding and that next steps are open, not a list of options. --- Tools & resources Stack: Single index.html, vanilla JS, MediaPipe Pose, Web Speech API, JSZip, GitHub Pages Deployment: Direct edit in GitHub web UI (mark all → delete → paste → commit); live at golf.greblats.com Validation tooling: extract_gates.py + ffmpeg for frame extraction from session ZIP bundles (JSON + MP4); contact sheets via Python/PIL Brief format: .docx generated via pandoc (markdown → docx) with LibreOffice round-trip for strict Word compatibility; outputs to /mnt/user-data/outputs/ Session data format: ZIP bundles containing session.json (frames array at ~37fps, gate indices, diagnostic fields) + per-swing MP4 videos Known offset: v0.4.1.11+ sessions have a ~1.7s JSON↔video timestamp offset requiring per-session calibration before frame extraction

Last updated Jun 7

Context
24% of project capacity used
Search mode

Golf_app_brief_v4_6.docx
781 lines

docx



Index_04.1.12.html
5,231 lines

html



p7_window.py
183 lines

py



extract_gates.py
153 lines

py



FOR FUTURE REF_0_4_2_0_9.html
5,102 lines

html



brief_patch_v3_5_validation_section.md
147 lines

md



json_schema_readme_v2.md
366 lines

md



claude 0.4b conversation.txt
2,689 lines

txt



Conversation 0.4 for claude 0.4B.txt
5,226 lines

txt



segmenter_hardening_plan.md
158 lines

md



head_stability_algorithm_sketch.md
308 lines

md



v0_4_0_walkthrough.md
324 lines

md



conversation 0.3 for 0.4 knowledge.txt
3,655 lines

txt



v0_3_3_design.md
312 lines

md



v0_3_2_design.md
204 lines

md



v0_3_1_design.md
288 lines

md



Conversation 0.2 for learning 0.3 coversation.txt
1,784 lines

txt



set up game app git, web etc
74 lines

text



Golf app conversation.docx
467 lines

docx


10 swing positions.png

Scheduled
Set up recurring tasks for this project.

Golf_app_brief_v4_6.docx
**Golf Coach App — Brief v4.6**
 
**Golf Coach App — Project Brief**
 
**Version 4.6**
 
A voice-driven AI golf coaching app that uses a phone camera to analyze swings and gives hands-free audio feedback through earbuds.
 
This brief is the canonical handoff document. Any future session should read this first.
 
**Last updated:**** **May 7, 2026 (v0.4.1.13 shipped — P1 horizontal motion-onset and P7 wristX-return primary now in code, awaiting field validation)
 
**What changed in v4.6:**** **v0.4.1.13 shipped, implementing the v4.5 hypothesis. The segmenter now uses **horizontal-axis (wristX) detection for P1 and P7** while P4 and P10 remain on wristY. P1 walks |Δ wristX| with the same 5-frame, 0.005-units-per-frame threshold structure as the previous vertical detector. P7 picks the first frame in [P4..P10] where wristX crosses back through wristX(P1), with a 0.005-unit tolerance band; if no clean crossing is found, it falls back to the **legacy median of (wristY argmax, elbowY argmax, shoulderRot argmax)**. addressMatch (the v0.4.1.10 detector) is retained as a tertiary diagnostic candidate but no longer drives impactIdx. Every swing record now persists wristXReturn, legacyMedian, addressMatch, the three legacy candidates, a usedDetector flag ('wristXReturn' or 'legacyMedian'), spread, and lowConfidence. The smoothed wristX trace is persisted alongside wristY for post-hoc tooling. **Validation status: not yet field-tested.** Brief v4.5 explicitly required validation against extracted frames before adopting wristX-return as primary; that validation is the next thing on the path. The May 1 12-swing outdoor dataset, indoor shadow-swing data, and the May 6 outdoor session are all queued. Active blockers section updated below to reflect the shipped-pending-validation state. *Patrik flagged that the Active blockers / what next section was confusing on read; minor cleanup applied here, deeper restructure deferred to v4.7 when validation results are in.*
 
**What changed in v4.5:**** **The v4.4 finding is reframed as a single structural hypothesis rather than two independent detector swaps. **The segmenter has been using the wrong axis for the gates that happen during predominantly horizontal hand motion** — specifically P1 (early takeaway, hands sweeping back before wrist hinge starts) and P7 (impact, hands crossing back through the address-x line). P4 and P10 are vertical-extreme events and stay on wristY. With this framing, the P7 fix is no longer “switch to legacy median” — it’s “detect the horizontal-axis crossing event that defines impact.”
 
**Revised P7 hypothesis:**** **primary detector is **wristX return through P1-x** (the first frame in [P4..P10] where wristX crosses back through its P1 value), with legacy median (shoulderRot, wristY, elbowY) as the backup signal when the crossing is ambiguous. This is structurally consistent with the P1 wristX-onset fix — both gates are horizontal-motion events being detected on the wrong axis.
 
**Caveat:**** **wristX-return-through-P1-x has not been directly validated against extracted frames yet. The May 6 outdoor session (3 swings) showed legacy median landing within ±1 frame of true impact, but did not test wristX-crossing as the primary signal. The v4.5 reframing commits to the structurally cleaner detector ahead of the data, which makes the May 1 12-swing validation more important, not less. **No code change shipped.** Validate wristX-return-through-P1-x on (a) May 1 12-swing outdoor dataset, (b) indoor shadow-swing data, and (c) the May 6 outdoor session itself before adopting.
 
**What changed in v4.4:**** ***[Superseded by v4.5 reframing — kept here for context.]* A single-session investigation (session-2026-05-06T18-00-15, 3 outdoor swings) re-examined the segmenter on a per-axis basis and produced the structural finding that different gates are best detected on different axes. The original v4.4 P7 proposal — switch from addressMatch to legacy median of (shoulderRot, wristY, elbowY) — has been **demoted to backup signal** in v4.5 in favour of wristX-return-through-P1-x as primary. The P1 finding (wristX motion-onset) and the P4/P10 findings (stay on wristY) are unchanged. The lowConfidence threshold flag (>8 frame spread between P7 candidates may be too lenient) carries forward.
 
**What changed in v4.3:**** **Lost-frames hole **provisionally resolved**. Two new releases shipped (v0.4.1.11 and v0.4.1.12) and the first session under v0.4.1.11 produced zero gaps anywhere across 636 frame intervals in two 8-second recordings — answering hypothesis (a) “does priming MediaPipe before recording starts eliminate the hole?” with a clean yes, and simultaneously answering hypothesis (b) “is the hole periodic on a longer window?” with a clean no (the 8s window would have shown a second hole at ~2.3s if the cause were periodic). The hole was a one-time MediaRecorder startup cost; warming MediaRecorder up for 2s before flipping into pose-collection mode absorbs it. Status downgraded from “active investigation, Priority 1 blocker” to “provisionally resolved pending 2–3 more confirmation sessions across mixed conditions”. P1 (address) detection is now Priority 1 — the demotion in v4.1 is reversed. v0.4.1.12 layered on top: recording window reverted 8s → 5s (the 8s window blew through the 36MB session-export limit at 4+ swings), a clearer “GO” tone (rising chirp 600→1000Hz, ~250ms, ~2× louder than the old soft tick) replaces the silent transition at swing-window-open, and the button-red flip + status text + REC indicators all moved from warmup-start to swing-window-open so visual and audio cues land together at the moment the player can actually swing.
 
**What changed in v4.2:**** **Absorbed two pieces of evidence from the standalone “Lost video frames investigation 2.0” doc and the v0.4.1.10 handover addendum into the brief, so those two docs can be deleted without losing context. (1) Per-swing evidence table added to the missing-frames Known Limitations entry — the four swings of session-2026-05-02 with hole length, JSON index, t-range, and position relative to P7 — so future readers can see the data behind the “wall-clock-anchored at ~1.15s” claim without needing the supplementary doc. (2) The “no catch-up burst” reasoning added — explains *why* main-thread blocking is ruled out (a frozen JS event loop would cause MediaPipe results to queue and fire as a burst on resume; observed cadence resumes cleanly with no backlog). No factual changes to project state — pure consolidation.
 
**What changed in v4.1:**** **Four corrections from Patrik review of v4.0. (1) Active blocker priorities reordered — the per-recording missing-frames investigation is now Priority 1, ahead of P1 detection, because no segmenter conclusion is trustworthy until we know whether the data going into the segmenter is whole. P1 fix moves to Priority 2. (2) Priority 2 (was: outdoor P7 validation) rewritten with concrete next step instead of vague “ground-truth-validate.” (3) Capture frame rate corrected from “60fps” to **30fps** in the platform table — the code requests frameRate: { ideal: 30 } in getUserMedia. The “60fps” figure in v4.0 was the theoretical web ceiling, not what we actually run. (4) Environmental requirements rewritten to match Patrik’s actual setup — tripod 20–30cm high at the end of a range ball tray, 1–2m from golfer, steep upward camera angle — replacing the inherited “waist-level / 2–2.5m / perpendicular” description that was wrong. Implications of the steep upward angle on shoulder-width-based displacement measurements flagged as a calibration gap.
 
**What changed in v4.0:**** **Structural rewrite of the brief itself. No factual changes to the project except (a) the head-stability spoken-phrase description was corrected to match the shipped code (numbers are included: “vertical down 2, horizontal forward 4”), and (b) v0.4.2 (record-first / 60fps / Savitzky-Golay) is now formally halted, not “shelved pending evidence.” The brief was reorganized to put current-state content up front and move version history into a Decision archive, because v3.x had grown reverse-chronological and was hard to read fresh. Versioning convention updated to allow minor bumps (v4.0) for structural rewrites of the brief itself, in addition to product-strategy pivots.
 
# What this app is
 
A practice companion that answers specific, golfer-initiated questions about their swing in real time. The user sets up their phone on a tripod, picks a drill (“tempo”, “head stability”), says “ready” when they are at address, swings, and hears a short spoken verdict through earbuds telling them the result. They can then optionally call out the shot shape (“pull”, “straight”) and the app logs everything.
 
**The differentiator:**** **most golf apps are swing recorders. This is a swing interrogator — the user brings a hypothesis, the app tests it. Over time the app builds a personalized understanding of the user’s patterns by correlating swing mechanics with self-reported ball outcomes.
 
**Target user:**** **an improver, not a beginner. Someone who has swung a club before, understands basic golf terminology (pull, hook, slice), and wants to work on specific aspects of their swing. Typically a range-practice user rather than on-course real-time feedback. Complete beginners need human instruction first; tour-level players need launch monitors.
 
# Current state — at a glance
 
**Shipped version:**** **v0.4.1.13 (May 7, 2026), running on iPhone Safari + AirPods. P1 and P7 segmenter changes shipped; field validation pending. Previous shipped: v0.4.1.12 (May 6, 2026 evening).
 
**Architecture:**** **web app, single index.html, vanilla JS, MediaPipe Pose via CDN. No build step. No backend. No accounts. Hosted on GitHub Pages at golf.greblats.com.
 
**Drills shipped:**** **Tempo (face-on or down-the-line), Head stability (face-on only).
 
**Capture pipeline:**** **live MediaPipe pose detection on the camera stream during a 5-second recording window, plus parallel MediaRecorder video capture for ground-truth review. v0.4.1.11+ runs MediaRecorder for 2 seconds before pose collection begins (warmup phase) to absorb a one-time encoder-startup cost that previously caused a ~350ms hole in the JSON pose data — see Known Limitations.
 
**Active blockers:**** **the v4.5 P1+P7 axis fix is now SHIPPED in v0.4.1.13. **Field validation against extracted video frames is the next gate** before we can call the segmenter ready for drill #3. P7 detector now uses wristX return through P1-x as primary, with legacy median as backup. P1 detector now uses wristX motion-onset. Both detailed in Active blockers below. **No drill #3 ships until segmenter success rate is ****>****95%** on a labeled JSON+video dataset.
 
**Last field validation:**** **session-2026-05-06T18-00-15 (3 outdoor swings, v0.4.1.12, May 6 evening) — produced the segmenter signal-axis findings now driving the next workstream. Earlier under v0.4.1.11: session-2026-05-06T15-41-15 (2 indoor shadow swings) — first session under MediaRecorder warmup, zero gaps in either recording across 636 frame intervals. Prior session: session-2026-05-02T08-46-19 (4 outdoor swings, v0.4.1.10), with open eyeball-validation tasks on sw3/sw4 P7.
 
# Active blockers and what’s next
 
**Unifying frame for Priorities 1 and 2 (v4.6):**** **the v4.5 hypothesis — P1 and P7 are both horizontal-motion gates that were being detected on the wrong axis — is now implemented in v0.4.1.13. P4 (top of backswing) and P10 (finish) remain vertical-extreme events on wristY. The current blocker shape has flipped: from "decide and code the fix" to "validate the shipped fix against extracted frames." Both priorities below are validation tasks, not coding tasks.
 
**Priority 1: validate v0.4.1.13 P7 detector against extracted video frames.**** **The detector now picks the first frame in [P4..P10] where wristX crosses back through wristX(P1) (with a 0.005-unit tolerance band to ignore jitter near the address line). If no clean crossing is found, it falls back to the legacy median of (wristY argmax, elbowY argmax, shoulderRot argmax). The swing record carries a usedDetector flag indicating which path fired. **Validation needed against extracted frames** on (a) the May 1 12-swing outdoor dataset, (b) indoor shadow-swing data, (c) re-running the May 6 outdoor session. Pre-shipping the May 6 outdoor session showed legacy median within ±1 frame of true impact on 3/3 swings; that was the backup signal validation, not the new primary. Use extract_gates.py to extract frames at the JSON's claimed impactIdx, eyeball for true impact. Three outcomes shape next steps: (i) wristX-return correct on most swings → primary detector validated, move on; (ii) wristX-return systematically off → re-examine the crossing definition (tolerance band? sign convention? handedness assumption?); (iii) wristX-return frequently doesn't fire and the legacy-median fallback is doing all the work → reconsider whether the structural framing is right or whether the P1 reference x-value is the issue.
 
**Priority 2: validate v0.4.1.13 P1 detector against extracted frames.**** **The detector now walks |Δ wristX| with the same 5-frame, 0.005-units-per-frame threshold structure as the previous vertical detector. Direction-agnostic via abs() — should work for right-handed face-on (the testing setup) but the handedness × camera-angle assumption is not yet stress-tested. **Validation needed**: same datasets as Priority 1. The May 6 outdoor pre-shipping data (3 swings) showed first-sustained-horizontal-motion shifting P1 4–5 frames earlier (122–134 ms) and eyeball-validating spot-on; that's encouraging but a 3-swing sample. Look for: P1 firing at the moment the clubhead actually starts moving back, not 5 frames before (over-eager) or 5 frames after (still vertical-blind).
 
**Priority 3: confirm the missing-frames hole stays gone across more sessions.**** **v0.4.1.11's MediaRecorder warmup eliminated the hole in the first session under it (May 6 indoor, 2 swings, 636 frame intervals all 23–28ms). One session is encouraging but not proof — the prior pattern was 20-for-20 across four sessions. Need 2–3 more confirmation sessions ideally covering: indoor shadow (more swings), outdoor range, and a multi-swing session at the 5-swing-per-session limit. The May 6 outdoor session above is one such confirmation candidate (analysis still queued). If the hole stays absent, the Known Limitations entry promotes from "provisionally resolved" to "resolved" and the v0.4.1.11 warmup becomes a permanent feature of the capture pipeline. If the hole reappears, hypothesis (c) — code-level instrumentation distinguishing MediaPipe-internal pause vs our pipeline dropping outputs — becomes the next experiment. **This priority piggybacks on Priorities 1 and 2** — every validation session also counts as a missing-frames-hole confirmation session.
 
**Priority 4: tighten the lowConfidence threshold for P7 candidates.**** **Currently the segmenter flags low confidence when the spread between P7 candidates exceeds 8 frames. The May 6 outdoor session showed addressMatch firing materially early while the spread was 7, 4, 4 — never tripping the flag. Threshold may be too lenient for outdoor data. The candidate set used to compute the spread metric has changed in v0.4.1.13 — it now includes wristXReturn alongside the legacy three and addressMatch — so the right threshold may be different again. Revisit once Priorities 1 and 2 produce validation data showing how spread correlates with actual P7 errors.
 
**Priority 5: verify v0.4.1.10****'****s outdoor P7 fix on sw3/sw4 of session-2026-05-02.**** **Largely overtaken by Priorities 1 and 2 — addressMatch is now a deprecated detector retained as a diagnostic only. Keep on the list because the sw3/sw4 video is still useful validation data, but the question shape has changed: it now reads as "does v0.4.1.13 land P7 correctly on these swings?" — which is just an instance of Priority 1.
 
**Halted: v0.4.2**** **(record-first / 60fps / Savitzky-Golay). See “v0.4.2 status” below for the full picture; in short: shelved on iPhone after April 30 WebKit failures, formally halted May 3. No work planned.
 
**Gating rule (adopted April 27, 2026):**** **the swing segmenter (P1/P4/P7/P10 detection) is plumbing every drill depends on. If it fails 22% of the time, every drill inherits that 22% failure rate regardless of how good the drill algorithm is. **Drill #3 does not ship until segmenter success rate is ****>****95% on realistic swings, validated against a labeled JSON+video dataset.**
 
# Guiding principles
 
**The golfer drives.**** **They ask a question, the app answers it. The app does not lecture.
 
**Hands-free for the whole session.**** **Not just per-swing. From walking to the hitting position until walking back, the phone is not touched. Every new feature must preserve this.
 
**Preserve practice rhythm.**** **Feedback must not break the flow between swings. Short speech over long speech. Silent confirmation when voice isn’t needed.
 
**Data over judgment.**** **Frame feedback as information, never as criticism. The verdict describes what the app measured — “on”, “below”, “above” — never “right” or “wrong”.
 
**Personalization through data.**** **Generic golf advice is cheap. The app’s value grows as it learns the user’s specific patterns.
 
**Ship narrow, ship often.**** **Each version is a surgical addition. Add complexity only when data says you need it.
 
**Design for phone + AirPods first.**** **This is where the real user lives. Desktop is for development only. Speech recognition quality is materially better with AirPods’ beamforming mics than with PC built-in mics — optimize for that environment.
 
**Build the shared infrastructure before the next feature that needs it.**** **Audio feedback and session review are not features of the tempo drill — they are the feedback layer for every drill. Adding a second drill before that layer exists would ship it half-pregnant. Apply the same logic to future decisions: before adding drill N+1, ask whether N+1 would need anything that doesn’t exist yet — if so, build that first.
 
**Segmenter reliability gates drill expansion**** **(see Active blockers).
 
# Platform
 
Web app first, native later. Iterates fast across any device, nails the analysis pipeline, validates the product before native commitment. Entire pipeline runs client-side in the browser.
 
**Primary target environment:**** **phone + AirPods. Desktop browsers work for development, but the product is designed and tuned for the phone experience.
 
## Web vs native trade-offs
 
| **Dimension** | **Web (current)** | **Native iOS** |
| --- | --- | --- |
| Frame rate (currently shipped) | 30fps (getUserMedia requests frameRate: { ideal: 30 }) | 30–240fps via rear-camera slow-mo |
| Frame rate (theoretical ceiling) | ~60fps on iOS Safari front camera | 240fps |
| Velocity-at-impact problem | Mostly unsolved | Mostly solved |
| Build/deploy speed | Edit file → live in 1 minute | App Store review, Xcode, Apple dev account |
| Cross-device iteration | Any phone or desktop browser | iOS only initially |
| User install friction | Open URL | App Store download |
| Cost | $0 | $99/yr Apple dev + review overhead |
 
**Why this matters:**** **the gap between what we ask for (30fps) and what’s available on iOS Safari (~60fps) is a deliberate choice — 30fps keeps MediaPipe’s per-frame cost manageable on the live pose pump and matches the MediaRecorder encode rate, which keeps the JSON↔video frame relationship as close to 1:1 as the live-pump architecture permits. Bumping to 60fps was scoped inside v0.4.2 as one of three coordinated changes; v0.4.2 is now halted.
 
**Why even 60fps is a ceiling:**** **iPhone’s rear camera shoots 1080p at 240fps in slow-mo mode, but the slow-mo pipeline is only exposed to native apps via AVFoundation. The browser’s getUserMedia API tops out around 60fps on iOS Safari. Apple has never exposed slow-mo to the web, and there is no current sign of that changing. Same on Android: getUserMedia typically caps at 30 or 60fps regardless of camera hardware.
 
## Re-examination log
 
**May 1, 2026:**** **the rear-camera 240fps argument was raised while debugging P7 detection — slow-mo would mostly eliminate the velocity-at-impact problem and frame-drops. **Decision: stay on web for now.**** **The web-side pattern-matching approach (addressMatch, see Segmenter section) is structurally better-grounded than “find the extreme of a signal” approaches and is cheap to test. If pattern-matching fails to reach the >95% phase-index correctness gate on a labeled dataset, the slow-mo argument becomes one of the stronger reasons to commit to native — revisit then. **Note (May 7, 2026):**** **pattern-matching (addressMatch) appears to be losing ground in light of the May 6 outdoor session — the v4.4 finding suggests the better path on web is a per-axis, per-gate detector strategy rather than pattern-matching. Native-vs-web is unchanged for now, but if the per-axis detector also fails to reach the >95% gate, the velocity-at-impact-via-slow-mo argument resurfaces.
 
## Primary use cases (priority order)
 
Driving range practice — single swings with feedback between each
 
Indoor practice / simulator setups / shadow swings
 
Eventually: on-course check-ins during a round
 
# Architecture: Plumbing vs Drills
 
This is the most important architectural line in the codebase. Read this section before touching any code.
 
## 1. Principle
 
The codebase has two kinds of code, separated by a deliberate seam:
 
**Plumbing** is everything not specific to any one drill. **Drills** are small modules that own only what’s specific to one coaching question.
 
Two corollaries that govern how the seam evolves:
 
**Plumbing serves drills; drills do not serve plumbing.**** **When a drill needs a capability the plumbing doesn’t provide, the plumbing changes. Drills contorting themselves to fit plumbing limitations is the design failure to avoid.
 
**The plumbing API will be unstable through v0.4.x — that’s the design.**** **Each new drill will likely force adjustments to the seam as new measurement shapes (positional, angular, time-position, multi-axis) surface assumptions baked in by earlier drills. With one consumer (us) and one file, this is free; in a published library it would be catastrophic. Plumbing stabilises empirically, probably around drill #4.
 
## 2. Inventory — what’s on each side
 
| **Plumbing owns** | **Drills own** |
| --- | --- |
| Capture pipeline (camera, MediaPipe, MediaRecorder) | The analysis function (analyze(ctx)) |
| Live skeleton overlay | Thresholds and verdict-class boundaries |
| Hands-free voice loop (“ready” detection, TTS) | Headline format and panel content |
| Swing segmentation (addressIdx, topIdx, impactIdx, finishIdx) | Spoken verdict copy and TTS strings |
| Result panel rendering | Drill-specific CSV columns |
| Session review screen | Distribution-bucket definitions |
| Drill picker UI | supportedAngles declaration |
| Camera-angle handling (session config, CSV column) | What the drill chooses to compute and report |
| CSV and JSON export |  |
| Voice command handling (SHAPE_WORDS, “ready”) |  |
| Distribution-chart drawing primitives |  |
| Analysis dispatch (DRILLS[sessionConfig.drill]) |  |
| Per-swing video capture (MediaRecorder) |  |
| spokenPhraseOverride mechanism (added v0.4.1.2) |  |
 
## 3. Drill API contract
 
Every drill is registered as DRILLS. and conforms to this shape:
 
{ id: string — stable identifier, used in session config and CSV displayName: string — picker tile label description: string — short subtitle on the picker tile supportedAngles: array — e.g. [‘faceOn’] or [‘faceOn’, ‘downTheLine’] verdictCommentMap: object — verdict-class → plain-language comment spokenForClass: object — verdict-class → TTS string (used unless override) analyze: function(ctx) → { value, verdict, verdictClass, diag, warnings, extra, spokenPhraseOverride } csvColumns: array — additional CSV column names (added to universal core) reviewSummary: function — drill’s contribution to review-screen aggregate stats tableColumns: array — additional per-swing-table column definitions distributionBuckets: object — bucket schema for the distribution chart }
 
**analyze(ctx) receives**** **a shared context: frames (per-frame landmarks), smoothed (smoothed wristY trace), the four phase indices (addressIdx, topIdx, impactIdx, finishIdx), tempo timing helpers (tAddress, beepTime, fps, totalMs, clippedDurationMs), peak-detection diagnostics (rawPeaks, mergedPeaks, warnings), and p7Candidates (the four P7 detector candidates from v0.4.1.6+v0.4.1.9).
 
**verdictClass vocabulary:**** **plumbing knows about good, below, above, off, error. A drill uses any subset that fits its question. Tempo uses good / below / above / error. Head stability uses good / off / error. Other classes can be added when a future drill needs them.
 
**spokenPhraseOverride**** **(added v0.4.1.2): when an analyze() return includes this field, plumbing speaks it instead of looking up spokenForClass[verdictClass]. Lets drills compute speech from analysis output dynamically (head stability uses it for “vertical down 2, horizontal forward 4”). Existing drills are unaffected — the field defaults to null.
 
## 4. Architectural rules that follow from the seam
 
**All capture is plumbing; drills are pure consumers.**** **One capture pipeline, all data, always. Every session records the full skeleton (33 keypoints), shape, voice transcripts, frame timing — regardless of which drill is “focused.” Drills only differ in what they compute from the data and what they report. Concretely: a tempo session contains all the data needed for retrospective head-stability analysis, and vice versa. Cross-drill analysis comes for free.
 
**Camera angle is plumbing; drills declare which angles they support.**** **Camera angle (face-on, down-the-line, etc.) is a session-level parameter alongside drill choice. Capture, segmentation, and UI plumbing handle angle generically. Each drill declares a supportedAngles list. Some drills work from any angle (tempo: [‘faceOn’, ‘downTheLine’]), some from one only (head stability: [‘faceOn’]; future hip lead: [‘downTheLine’]). v0.4.0 wired cameraAngle through session config, drill registry, and CSV even though only face-on is supported in v0.4.x — retrofitting later would require touching every drill module and the seam itself.
 
**Swing segmentation is plumbing; drills are pure consumers of the four phase indices.**** **A drill that invented its own “where did the swing start / end” logic would produce a slightly different window than tempo for the same frames, and over six drills that divergence becomes a correctness problem. Canonical segmentation lives in plumbing.
 
**The canonical swing window is address → finish, follow-through included.**** **Not address → impact. Drills that only care about the downswing (tempo) still get the full window; drills that care about post-impact behaviour (head stability past the ball, posture-at-finish) read the portion they want. Sub-window analysis is supported via drill config (full / backswing / downswing / followthrough / custom slices).
 
**The four phase indices are the contract.**** **Every swing that analyzes successfully has all four (addressIdx, topIdx, impactIdx, finishIdx). A swing with any index missing is marked as unanalyzable by the shared segmenter; drills do not attempt to fill in missing indices or work with partial windows.
 
**Mapping to standard golf vocabulary**** **(Mac O’Grady 10-position model, used by HackMotion, Trackman, et al.): addressIdx = P1, topIdx = P4, impactIdx = P7, finishIdx = P10. Internal code uses our names for clarity; the brief uses both interchangeably. The remaining six positions (P2, P3, P5, P6, P8, P9) are not detected by the v1 segmenter. If a future drill needs one (e.g. transition / P5, club parallel in downswing / P6), that position becomes a plumbing addition driven by drill need.
 
**What is NOT detectable from skeleton-only:**** **anything club-orientation-specific — face angle, shaft plane, lag angle. P-positions defined by club geometry can be approximated from wrist/lead-arm geometry but not measured precisely. Drill specs that require club-orientation data are out of scope until club detection arrives (deferred).
 
# Analysis pipeline
 
All client-side. As shipped in v0.4.1.13:
 
**Video capture**** **via getUserMedia — measured ~37fps on iPhone Safari (May 2 outdoor field data). Live pose pump runs at the device-paced rate; MediaRecorder encodes the saved video at 30fps. The fps mismatch causes JSON↔video duplicate-frame entries — see Known Limitations.
 
**Pose detection**** **via MediaPipe Pose @0.5.1675469404 (pinned version — the unpinned @latest specifier broke pose loading during v0.2 development and cost a day of debugging).
 
**Per-swing video**** **via MediaRecorder, capturing the same camera stream the pose pipeline uses, for ground-truth review (added v0.4.1.3).
 
**Metrics computation**** **— per-drill math on the keypoint sequence.
 
**Drill evaluation**** **— rule-based pass/fail/uncertain check.
 
**Audio output**** **via speechSynthesis (TTS) for user-facing events: echoing “ready”, speaking the verdict, echoing the recognized shape word.
 
## Sound design
 
TTS is the primary feedback channel. Field testing showed the screen is unreadable outdoors and tone vocabularies don’t scale to multiple drills (each new drill would need its own tone or collide with an existing one). Speech is self-documenting and scales trivially.
 
| **Event** | **Positive** | **Off / Amber** | **Error** |
| --- | --- | --- | --- |
| “ready” heard | TTS: “ready” | n/a | silence |
| Shot analysed (tempo) | TTS: “on” | TTS: “below” / “above” | TTS: “no swing” |
| Shot analysed (head stability) | TTS: “vertical same, horizontal same” | TTS: “vertical down 2, horizontal forward 4” (numbers included; drill-specific phrasing) | TTS: “no swing” |
| Shape heard | TTS: the word (“pull”, “slice”, …) | n/a | silence |
 
**Implementation notes for TTS:**** **uses window.speechSynthesis (Web Speech API). Works on iOS Safari + AirPods, unlocked at Start session along with AudioContext. speechSynthesis.cancel() is called before each utterance to prevent queue buildup on iOS Safari (known issue). Platform-default voice, rate ~1.1×, no pitch modification. Utterances kept to short phrases to minimize latency (typical iOS TTS first-word latency is ~100–300ms).
 
**Legacy tones:**** **the v0.2.x tick (440 Hz) and beep (880 Hz swing-start cue) survive in the code but are no longer called. v0.3.3.3 had simplified the cue to a single 440 Hz tick at the end of POST_READY_DELAY_MS; v0.4.1.12 replaced that tick with a clearer **GO tone**** **(rising chirp 600→1000 Hz over 180ms inside a ~250ms envelope, peak gain ~2× the old tick) fired at the moment the swing window actually opens — i.e. at the end of the 2s MediaRecorder warmup, simultaneously with the button flipping red. Shape-chime (660→990 Hz sweep) was removed in v0.3.0. **Design caution preserved:**** **if any future tone is added, keep it to a single-oscillator envelope (a frequency ramp on one oscillator is fine; the GO tone uses this pattern). A two-oscillator chime was tried in v0.2.3 and failed silently on desktop Edge.
 
## Feedback layers
 
**Real-time per-swing:**** **TTS verdict through earbuds + golfer-facing result panel (big headline, plain-language comment when relevant, shape line when called). Dev content (chart, diagnostics, transcript log) lives behind a “Show debug” link.
 
**Session review**** **(v0.3.3): “End session” routes to a review screen with aggregate stats, distribution chart, averaged swing-shape chart, per-swing table, and CSV download. Data is in-memory only — evaporates on Discard. Persistence (IndexedDB) is deferred (v0.3.4, not yet shipped).
 
# Swing segmentation
 
**Status:**** **v0.2.2 baseline; HARDENING IN PROGRESS as of v0.4.x. v0.4.1.6 added multi-signal P7 consensus; v0.4.1.9 introduced addressMatch as the primary P7 detector; v0.4.1.10 narrowed the addressMatch landmark set to body-only. **May 7, 2026 finding (v4.5):**** **addressMatch is now suspected of over-correcting and firing early on outdoor data. The v4.5 hypothesis reframes P7 as a horizontal-motion event and proposes **wristX return through P1-x as the new primary detector**, with legacy median (shoulderRot, wristY, elbowY) as backup. This is structurally consistent with the P1 wristX-onset fix — see Active blockers Priority 1 and 2, and the per-axis subsection below. Pending validation.
 
**Recording window:**** **RECORDING_DURATION_MS = 5000 (5 seconds), opened after the TTS echo of “ready” (POST_READY_DELAY_MS = 1500ms settle) and a 2s MediaRecorder warmup phase (MEDIARECORDER_WARMUP_MS = 2000, added v0.4.1.11). Total delay from user saying “ready” to swing window opening: ~5 seconds (~1.5s TTS + 1.5s settle + 2s warmup). During warmup, MediaRecorder is running and burning through its encoder-startup cost, but pose collection has not yet begun — the JSON timestamp t=0 is set at the post-warmup moment, so all per-swing analysis ranges remain comparable to pre-v0.4.1.11 sessions. The captured video file therefore contains ~2s of pre-swing warmup footage at the head; eyeball-checks need to add this offset when locating swing events. The 5s recording window length is unchanged from v0.3.0 and chosen so the golfer’s settling time at address is contained inside the window.
 
**The swing produces a characteristic W-shape in the wrist-height trace,**** **which is what the segmenter looks for: Address (hands low, stable) → Top of backswing (minimum wristY, hands high) → Impact (local maximum wristY between Top and Finish) → Finish (second minimum wristY, hands high again). **Caveat (May 7, 2026):**** **the W-shape mental model is structurally incomplete. Hand motion at each gate is dominated by a different axis — horizontal at P1 (takeaway sweep) and during the impact crossing; vertical at P4 (top of backswing, full hinge) and P10 (finish, hands over lead shoulder). The current vertical-only segmenter is correct for P4 and P10 but blind at P1, and addressMatch — designed to compensate for the W-shape’s ambiguity at impact — appears to over-correct on outdoor data. The proposed v4.5 fix uses gate-specific axes; see “Per-axis, per-gate detection (v4.5 hypothesis)” below.
 
## Algorithm
 
Track wrist height per frame using average of both wrists when visibility ≥ 0.7. Below that, fall back to the more-visible wrist. Below 0.5, drop the frame entirely and linear-interpolate.
 
Smooth with a 5-frame moving average.
 
Find Address as motion onset — the last frame before sustained upward hand motion begins (5 confirming frames, 0.005 normalized units per frame). [v4.5 note: structurally blind to early takeaway because wrists move horizontally first; replace with |Δ wristX| onset, see Active blockers Priority 2.]
 
Define analysis window as Address → Address + MAX_SWING_MS (4000ms — no real swing takes longer).
 
Find prominent local minima (hands-high peaks) in the window, with prominence threshold max(0.04, 0.15 × window_range).
 
**Plateau-tolerant peak detection:**** **a minimum is valid if it’s at the bottom of a flat plateau (the Finish often plateaus — hands held high at end of swing).
 
**Edge-aware prominence:**** **if a peak is at the end of the window (right side cut off by recording end), use only the left-side basin wall for prominence rather than rejecting the peak. Key fix for swings where the Finish plateaued to end of recording.
 
Merge peaks within 300ms — occlusion artifacts, not separate swing events. Keep the one with higher hands.
 
Top = first merged peak; Finish = second merged peak.
 
**P7 (impact) detection**** **— see “P7 detector” below.
 
**Tempo ratio**** **(tempo-drill-specific output) = (Top − Address) / (Impact − Top). Tour average ~3:1; acceptable 2.5:1 to 3.5:1.
 
## Per-axis, per-gate detection (v4.5 hypothesis — pending validation)
 
Single-session investigation (session-2026-05-06T18-00-15, 3 outdoor swings) found that the current vertical-axis-only detection logic is the right axis for P4 and P10 but the wrong axis for P1, and that addressMatch — the current P7 detector — over-corrects on outdoor data and should be replaced. The v4.5 framing unifies these into a single hypothesis: **the segmenter has been using the wrong axis for the gates that happen during predominantly horizontal hand motion** (P1 takeaway, P7 impact). P4 and P10 are vertical-extreme events and stay on wristY.
 
| **Gate** | **Best signal** | **Why** |
| --- | --- | --- |
| P1 | wristX motion-onset | Wrists move horizontally first in takeaway; vertical motion lags by ~5 frames |
| P4 | wristY argmin (current) | Wrists fully hinged at top = vertical extreme. Hands reach horizontal extreme ~10 frames before true P4 |
| P7 | wristX return through P1-x (or legacy median as backup) | Sharp horizontal crossing at impact; addressMatch over-corrects for late-bias and now fires early |
| P10 | wristY argmin (current) | Wrists over lead shoulder = vertical extreme. Hands reach horizontal extreme during early follow-through, not at finish |
 
**Mental model:**** **during a swing, hands move predominantly horizontally at the start and end (sweeping/rotational motion), and predominantly vertically at the top and finish (hinge motion). The right detection signal for each gate is whichever axis dominates at that gate.
 
**Evidence summary (this session, 3 swings):**
 
P1 shifts (wristX motion-onset vs current wristY motion-onset):
 
| **Swing** | **Current P1 (wristY)** | **New P1 (wristX)** | **Shift** |
| --- | --- | --- | --- |
| 1 | idx 52 | idx 47 | −5 frames (−122 ms) |
| 2 | idx 29 | idx 25 | −4 frames (−134 ms) |
| 3 | idx 39 | idx 35 | −4 frames (−122 ms) |
 
Eyeball validation: NEW P1 frames look like settled address (club down, no motion blur). CURRENT P1 frames show takeaway already started. NEW P1 verdict: spot on, all 3 swings.
 
P7 shifts (legacy median vs current addressMatch):
 
| **Swing** | **Current P7 (addressMatch)** | **Legacy median** | **Shift** |
| --- | --- | --- | --- |
| 1 | idx 85 | idx 91 | +6 frames (+157 ms) |
| 2 | idx 61 | idx 64 | +3 frames (+94 ms) |
| 3 | idx 73 | idx 76 | +3 frames (+99 ms) |
 
Eyeball validation: legacy median lands closer to true impact than addressMatch on all 3 swings, though Patrik judged “1 frame too late” — likely because pros’ hands lead address-x at impact (forward shaft lean), so the closest-to-address-x frame is naturally 1 frame before true impact at 30fps. Accepting that 1-frame imprecision keeps the algorithm simple. **Legacy median verdict (v4.5):**** **better than current, within ±1 frame of truth — kept as the backup signal in the v4.5 hypothesis.
 
**v4.5 primary detector (wristX return through P1-x) — unmeasured on this session.**** **The May 6 evidence above tested legacy median directly; it did not directly test wristX-return-through-P1-x as the primary signal. The v4.5 framing argues wristX-return is structurally the right detector (consistent with the P1 fix — both are horizontal-motion gates), but the figures above are evidence for the *backup* signal, not the primary. Re-running this session under wristX-return-primary is part of the validation set listed below.
 
**P4/P10 horizontal-axis check (rejected):**** **tried wristX argmin/argmax for P4 and P10. Found massive shifts (P4: 9–11 frames earlier; P10: 3–6 frames earlier) and visually wrong. Reason: hands reach horizontal extreme well before vertical extreme in the backswing/finish. Hand vertical:horizontal travel ratio P1→P4 = 1.4–1.8× across the 3 swings. Vertical (current logic) is correct for P4 and P10. No change there.
 
**Caveats before adopting:**** **n=3, single session, single golfer, single camera setup (May 6 outdoor, face-on selfie cam, home patio). Needs validation against (a) indoor shadow-swing data (different lighting, smaller swing arc), (b) the May 1 12-swing outdoor dataset (the original P7 calibration set), and (c) at least one driving range session (real impact dynamics). Also: the wristX-crossing P7 has the same single-frame imprecision at 30fps that wristY had — at impact, hands cross address-x in ~33 ms covering 10–15% of frame width per frame; sub-frame precision isn’t possible without 60fps capture (which v0.4.2 was halted for). And: P1’s new logic depends on the lead hand’s horizontal motion direction, which depends on golfer handedness and camera angle — current logic is written assuming a face-on selfie cam with a right-handed golfer (hands sweep right-to-left in the image during takeaway). Generalizing to down-the-line camera or left-handed golfers needs explicit handling.
 
## P7 detector (current shipped code — v0.4.1.10, under review per v4.5)
 
P7 is the index in [topIdx+1 .. finishIdx-1] whose pose most closely matches the address pose, scored on a body-only landmark set:
 
ADDRESS_MATCH_LMS = { NOSE, LEFT_SHOULDER, RIGHT_SHOULDER, LEFT_KNEE, RIGHT_KNEE }
 
Wrists and elbows were dropped in v0.4.1.10 because MediaPipe wrist landmarks go to garbage during impact-blur frames despite reporting high visibility (0.85+) — wrist x-coordinates jumping 30%+ of frame width per frame, physically impossible. addressMatch was faithfully matching against those garbage values, picking the first POST-blur frame where the recorded wrist position “agreed with” P1’s wrist position by coincidence. Re-scoring with body-only landmarks hits true impact within ±1 frame on 4/4 indoor and shifts the May 1 outdoor 12-swing set ~2 frames earlier on 10/12 swings.
 
**Update (May 7, 2026, v4.5):**** **on the 3-swing May 6 outdoor session, body-only addressMatch fires 3–6 frames early on all 3 swings and lands further from true impact than the legacy median. The body-only narrowing fixed one bias and introduced another. **v4.5 hypothesis:**** **switch the primary P7 signal to **wristX return through P1-x** (the first frame in [P4..P10] where wristX crosses back through wristX(P1)), with legacy median (shoulderRot, wristY, elbowY) as the backup signal when the crossing is ambiguous. This reframes P7 as a horizontal-motion event, structurally consistent with the P1 wristX-onset fix. **Bigger code change than the v4.4 single-line legacy-median proposal** but still small relative to P1 (search window already bounded by P4..P10). **Validation status:**** **wristX-return-primary has not been directly tested on extracted frames yet — May 6 figures above measured legacy median, not wristX-return. Validate on (a) May 1 12-swing outdoor dataset, (b) indoor shadow-swing data, (c) re-run on May 6 outdoor session, before adopting.
 
**Three legacy P7 signals retained as diagnostics**** **(from v0.4.1.6 multi-signal consensus): wristY argmax, elbowY argmax, |Δshoulder.x| argmax (body-rotation proxy). All three stored in JSON alongside the addressMatch primary, used for post-hoc analysis when addressMatch disagrees materially with the legacy consensus. Per the v4.5 hypothesis, if the switch to wristX-return-primary ships, these become the **backup candidate set** invoked when wristX-crossing is ambiguous (no clean crossing detected, or multiple crossings within a small window).
 
**addressMatch quality is fundamentally bounded by P1 quality.**** **If P1 lands on a frame where the club is already in the air, the “match” is to a wrong reference, and P7 inherits the error. P1 detection is a priority blocker (see Active blockers Priority 2). **Note (v4.5):**** **the proposed wristX-return-through-P1-x detector also depends on P1 — wristX(P1) is the reference value the crossing is measured against. So P1 quality continues to matter for P7 even after addressMatch is retired. The legacy-median backup signal is the only P7 candidate that does not depend on P1; it’s the safe fallback when P1 itself is suspect.
 
## Incomplete-swing rejection
 
Fewer than 2 merged peaks → “Incomplete swing — swing all the way through to finish.” No segmentation produced; drills do not run. As of v0.4.x range testing: completion rate on real swings is 100%; the open problem is phase-index correctness, not completion.
 
## Held-finish swings
 
Can produce false “Incomplete swing” verdicts when the segmenter cannot find a second minimum because the user holds the finish position past the end of the recording window. Edge-aware prominence (step 7 above) handles the common case.
 
# Drill library
 
Each drill is a specific, testable question with binary or ternary output, computed from pose keypoints. Each drill declares its preferred camera angle.
 
| **Drill** | **Question** | **Measurement** | **Supported angles** | **Status** |
| --- | --- | --- | --- | --- |
| Tempo ratio | Is your backswing-to-downswing ratio in range? | (Top − Address) / (Impact − Top); 2.5:1 to 3.5:1 = good | faceOn, downTheLine | Shipped |
| Head stability | Did your head move at impact? | P7 head displacement vs P1, both axes; either > 4″ → off | faceOn | Shipped |
| Hip lead | Do hips lead the downswing? | Hip angle ahead of shoulder angle through impact | downTheLine | Planned (drill #3) |
| Completed backswing | Reach full top before reversing? | Continuous shoulder rotation peaks at top | faceOn or downTheLine | Planned |
| Weight shift | Weight finishes on lead foot? | Lateral hip displacement at finish | faceOn | Planned |
| Posture maintenance | Spine angle consistent address to impact? | Spine vector angle change | downTheLine preferred | Planned |
 
**New drills add by defining:**
 
Clear question
 
Pose-based computation that runs on the shared swing window (address → finish)
 
Which sub-window the computation cares about (full / backswing / downswing / follow-through / custom)
 
Pass/fail threshold(s)
 
supportedAngles list
 
Drills do not reimplement swing segmentation — they consume the shared phase indices. Drills do not reach into capture either — all skeleton + shape + voice data is plumbing-owned and available to every drill.
 
## Shape system (shot outcome reporting)
 
Six golfer-native words. Per-swing voice-callable in the waiting state when the shape toggle is on:
 
**straight**** **— went where aimed
 
**pull**** **— left of target, straight or mild left curve
 
**push**** **— right of target, straight or mild right curve
 
**hook**** **— hard left curve (from any start)
 
**slice**** **— hard right curve (from any start)
 
**bad**** **— thin, fat, top — contact issue, not a shape issue
 
Voice-friendly, low cognitive load, captures both direction and ball behavior. Dartboard-style distance scoring was considered in original design and rejected: adds per-swing cognitive load, and captures outcome-vs-intent rather than ball-behavior (which is what correlates with swing mechanics).
 
**Opt-out by design.**** **Shape toggle default is OFF. The user turns it on when they want shape data. Shadow swings, quiet-practice sessions, or moods where the golfer just wants mechanics data all work fine with shape off.
 
## Voice-matching strategy
 
The recognizer is the Web Speech API (iOS Safari: WebKit speech recognition). Continuous, auto-restarts on onend (Safari silently stops after silence), uses interimResults: true so matches fire fast.
 
**“****ready****”**** ****matches strictly:**** **ready, reddy, redi. Narrow by design — loosening invites false triggers from “already”, “red”, “reddit”. Kept strict because it works well in practice.
 
**Shape words match with expanded mishear tolerance**** **discovered during testing: pole, poll, paul, pool → pull; bat, bud, bed, bet → bad; strait, strayed, straighten → straight; hoop, hock → hook; splice, slight → slice; posh → push.
 
A transcript debug panel inside the result panel shows the last 6 recognized phrases (interim/final, with match info) — useful for diagnosing recognition issues without guessing.
 
# Interaction model
 
## Two user-facing toggles (start screen)
 
| **Toggle** | **Default** | **Behaviour** |
| --- | --- | --- |
| Voice trigger (“say ready”) | ON | If on, “ready” starts the next swing. If off, tap Start does. |
| Shape reporting | OFF | If on, the waiting state listens for shape words. If off, only “ready” is matched. |
 
Shape toggle is OFF by default because shadow swings have no ball to call, and range practice next to other people may feel awkward.
 
## Core principle for voice
 
**“****Ready****”**** ****is said at address, not before.**** **The golfer tees up, settles into stance, says “ready” when about to swing. The app echoes the word back (TTS), pauses briefly (~1.5s), then opens a silent recording window. Swing happens at the golfer’s own pace within that window.
 
**The recording window is a permission envelope, not a measurement anchor**** **— measurement is still derived from body motion in the captured frames, so the exact moment of swing within the window does not matter.
 
This replaces the v0.2.x tick-tick-beep countdown pattern. Field testing showed that at driving range pace (60+ balls per session) the countdown was slower than the golfer’s natural rhythm.
 
## Session loop (as of v0.4.1.12)
 
Page loads into the start screen — camera preview at top, drill picker (Tempo / Head stability) + shape/voice toggles + Start session button below.
 
User picks drill, picks settings, taps Start session (any start-screen tap unlocks TTS).
 
In-session mode begins. User is at address, teed up. Says “ready”.
 
TTS echoes “ready” back through earbuds — confirms the app heard them.
 
~1.5s settle pause (silent), then 2s MediaRecorder warmup (silent, button stays amber). At the end of warmup: GO tone fires, button flips red, REC indicator and border activate, and the 5-second recording window opens. Per-swing video recording started 2s earlier (at warmup begin); the captured video therefore includes ~2s of pre-swing warmup footage.
 
User swings any time within the window.
 
Recording auto-stops, video chunks finalize, analysis runs.
 
TTS speaks the drill-specific verdict. Tempo: “on / below / above / no swing”. Head stability: “vertical down 2, horizontal forward 4” (drill-supplied via spokenPhraseOverride).
 
Golfer-facing result panel appears with drill-specific headline. Tempo: big centered ratio (or “N/A” on error). Head stability: two rows showing Top and Impact gate displacements as plain words.
 
Optional shape callout. Result panel shows Next Swing / End session.
 
# Environmental requirements
 
The actual current testing setup, as of May 2026:
 
**Distance from phone:**** **1–2m from the golfer (measured shorter than earlier brief versions claimed; reflects real range and home practice).
 
**Phone height:**** **20–30cm above the ground — phone sits in a tripod placed at the end of the range ball tray (or equivalent low surface for indoor / shadow-swing testing). Significantly lower than waist level.
 
**Camera angle:**** **steep upward view of the golfer. The phone-low / golfer-standing geometry means the camera looks up at the body rather than straight across. This is the working setup, not an aspiration.
 
**Camera selection:**** **front-facing camera (selfie cam) on iPhone 14, portrait orientation.
 
**Connectivity:**** **HTTPS required for camera/mic permissions; everything else runs on-device.
 
**Audio:**** **AirPods strongly recommended. Built-in phone mic also works; desktop built-in mics work but speech recognition quality is noticeably worse.
 
**Implication of the steep upward angle (added v4.1, not yet investigated):**** **the body-displacement measurements (head stability, future weight shift) convert pixel distances to inches using a fixed shoulder-width assumption (ASSUMED_SHOULDER_INCHES = 16). That conversion implicitly assumes a roughly perpendicular camera-to-body view, where the shoulder line is parallel to the camera’s image plane. With a steep upward angle, vertical pixel distances are foreshortened and horizontal pixel distances have a perspective component that depends on how far up/down the body the measurement happens. **Net effect:**** **displacement inches reported for the head (high on the body) and the hips (mid-body) are read off different effective pixel-to-inch ratios — and neither matches the shoulder-line ratio the calibration assumes. Magnitude unknown. Filed as a calibration gap; investigate alongside the per-user shoulder-width calibration upgrade. Until then, head-stability inches should be read as relative (“more drift than my baseline”) rather than absolute (“4 inches forward”).
 
# Hosting and deployment
 
Follows the existing Shadow Council pattern (see set_up_game_app_git__web_etc):
 
**Code host:**** **GitHub (public repo Patriks313/Golf, default branch main)
 
**Web host:**** **GitHub Pages — free, serves directly from repo
 
**Domain:**** **golf.greblats.com (subdomain of existing Namecheap domain)
 
**HTTPS:**** **enforced (required for camera + mic access)
 
**Architecture:**** **single index.html at repo root containing HTML + CSS + vanilla JavaScript inline. No build step, no framework, no node_modules. External dependencies from CDN via
 
tags — currently MediaPipe Pose (Apache 2.0) and JSZip (for video bundling).
 
**No Firebase**** **— no cross-device sync, no multiplayer. Reassess if cross-device history becomes necessary.
 
**Deployment:**** **edit index.html on GitHub (or locally and push), commit, live within ~1 minute.
 
**Release tagging:**** **v0.1.6 was tagged as a release to enable rollback. Going forward, tag each stable version.
 
# Validation technique: frame-extraction against JSON gates
 
**Principle.**** **JSON-only checks (frame counts, ratios, verdicts) let bad gate placement hide behind plausible-looking numbers. A swing can report tempo ratio 1.87 and “Below” verdict while P1 is mid-takeaway, P7 is in mid-upswing, and P10 is the actual top of the backswing — and nothing in the JSON makes that obvious. The only reliable check on gate quality is to extract the actual video frame at each gate’s timestamp and look at it.
 
This was discovered April 30, 2026 after spending most of a day chasing data-quality bugs that turned out to be gate-placement bugs hiding in plausible JSON. The April 30 afternoon iPhone session reconfirmed the technique’s value: 6/6 swings showed a P7 bias that no JSON metric flagged. The May 7 v4.4/v4.5 finding — addressMatch over-correction — was discovered the same way: JSON metrics looked fine, eyeball check on extracted frames revealed the early-firing pattern.
 
**When to use it.**** **Any time gate indices are claimed correct. Especially after segmenter changes. Especially when a verdict surprises the user. Especially when a ratio looks “fine but off.” Generalises to all drills with gate indices.
 
**Inputs.**
 
Session JSON (session.json inside the v0.4.1.7+ ZIP bundle, or the JSON-only download)
 
Per-swing video files (swing-NN.mp4 inside the bundle)
 
ffmpeg installed in the analysis environment
 
**Recipe.**** **A working Python implementation lives in the project file extract_gates.py. The pattern: for each swing, look up addressIdx, topIdx, impactIdx, finishIdx, find the frame timestamp at each, run ffmpeg to extract that frame as PNG, then build a contact sheet for visual inspection. Drill-agnostic — only the gate-field map needs updating per drill.
 
**What to look for in tempo contact sheets:**
 
**P1 (address):**** **club at ball, hands at address position, body still. Common failure: hands already in takeaway. Cross-check: P1 wristY should be at or within 1% of max wristY in swing. (v4.4 update: also cross-check that wristX is not yet moving — if wristX has been ramping for 4+ frames, P1 is late.)
 
**P4 (top):**** **club furthest from ball, hands highest. wristY at min. Common failure: P4 falls 5–15 frames before actual minimum.
 
**P7 (impact):**** **club at ball, body uncoiling, blur on club at this point in fast swings. wristY back near address-level. Pre-v0.4.1.9 common failure: lands several frames after impact (now mostly fixed by addressMatch). v4.5 update: addressMatch on outdoor data now lands several frames before impact — switch to wristX-return-through-P1-x as primary (legacy median as backup) proposed.
 
**P10 (finish):**** **club over leading shoulder or beyond, balanced hold. wristY at or near min again. Common failure: P10 fires mid-follow-through, body still moving.
 
**Cross-check the JSON wristY values against the W-shape pattern.**** **P1 wristY ≈ max in swing; P4 wristY ≈ min; P7 wristY ≈ max again (or within 5–10% of P1); P10 wristY ≈ min again, or at least below P7. If the math doesn’t fit this pattern, the gates are wrong even if the ratio looks plausible. **v4.4 caveat:**** **on the May 6 outdoor session, P7 wristY was ~0.10 normalized units below P1 wristY (i.e. hands genuinely higher in the image at impact than at address — both wrists agreed, not a tracking glitch) because the algorithmic search window (P4..P10) cuts off before the wrists fully descend post-impact. The W-shape is the canonical pattern but real swings can deviate; cross-check against extracted frames if numbers look off.
 
# Known limitations
 
## Active / not yet mitigated
 
**P1 (address) detection fires too early on real swings — diagnosed as structural.**** **Catches mid-takeaway, not still address. Affects ~5/6 swings observed across multiple sessions. Corrupts everything downstream — P7 addressMatch quality is bounded by P1 quality, and all displacement-from-address drills inherit the error. **Diagnosis (May 7, 2026, v4.4):**** **wrists barely move vertically during early takeaway because the arms first sweep horizontally and the clubhead arcs back before wrist hinge starts. The current vertical-motion-onset detector is therefore structurally blind to this phase and consistently flags P1 ~5 frames after true address. Proposed fix: replace |Δ wristY| onset with |Δ wristX| onset (same threshold structure: 5 confirming frames, ~0.005 normalized units per frame). On the 3-swing May 6 outdoor session, the wristX-onset detector eyeball-validates spot-on on 3/3. Sequence after P7 fix ships and stabilizes — P1 rewrite is a bigger code change and depends on golfer handedness × camera angle. **Priority blocker for drill #3.**
 
**P7 (impact) detector — addressMatch over-corrects on outdoor data.**** **v0.4.1.10’s body-only addressMatch was a fix for the pre-v0.4.1.9 late-bias, and hits true impact within ±1 frame on indoor data. On the 3-swing May 6 outdoor session, however, addressMatch fires 3–6 frames early on all 3 swings, while the legacy median of (shoulderRot, wristY, elbowY) candidates lands within ±1 frame of true impact. **v4.5 proposed fix:**** **switch primary P7 to **wristX return through P1-x** (the first frame in [P4..P10] where wristX crosses back through wristX(P1)), with legacy median as the backup signal. This reframes P7 as a horizontal-motion event, structurally consistent with the P1 wristX-onset fix. wristX-return-primary itself is unmeasured on this session — May 6 figures measured legacy median. Validate against (a) May 1 12-swing outdoor dataset, (b) indoor shadow-swing data, (c) re-run May 6 outdoor, before adopting.
 
**lowConfidence threshold for P7 candidates may be too lenient.**** **Currently flags low confidence when spread between P7 candidates exceeds 8 frames. May 6 outdoor session showed addressMatch firing materially early while spread was 7, 4, 4 — never tripped. Revisit once P7 switches to legacy median (the candidate set that defines the spread metric will change).
 
**MediaPipe wrist visibility is unreliable under motion blur.**** **Wrists report visibility 0.85+ during impact-blur frames while the actual position data is meaningless (x-coords jumping 30%+ of frame width per frame, physically impossible). Confirmed May 1, 2026 indoor session. Defensive code: use body-only landmarks for impact-time detection (addressMatch v0.4.1.10). Note: the v4.4 finding implies that even body-only addressMatch over-corrects — the wrist-blur problem is real, but the fix introduced a different bias.
 
**Per-recording ~350ms hole in the live pose pipeline — PROVISIONALLY RESOLVED in v0.4.1.11.**** **v0.4.1.11 (May 6, 2026) added a 2s MediaRecorder warmup phase before pose collection begins: MediaRecorder.start() fires, the encoder burns through its startup cost in throwaway pre-swing footage, and only after MEDIARECORDER_WARMUP_MS does the pose pipeline begin emitting frames into the JSON window. First session under v0.4.1.11 (session-2026-05-06T15-41-15, 2 indoor shadow swings, 8s recording window for the test) showed **zero gaps anywhere across 636 frame intervals — every Δ between adjacent timestamps fell in the 23–28ms band**. This simultaneously answered hypothesis (a) (priming MediaPipe before recording starts eliminates the hole) and hypothesis (b) (the 8s window would have shown a periodic recurrence at ~2.3s if the hole were clock-driven; it did not). Status remains “provisionally resolved” pending 2–3 more confirmation sessions across mixed conditions — the prior pattern of 20 holes in 20 swings was strong, so a single clean session is encouraging but not yet proof. The historical analysis below is preserved for context.
 
**Per-recording ~350ms hole — historical analysis (pre-v0.4.1.11):**** **Every recorded swing across four sessions to date (20 swings, indoor + outdoor) had exactly one ~350ms gap (343–364ms range) in the JSON frame timestamps. Originally logged April 29 as “frame drops at impact”; May 3 investigation re-framed it: the hole is anchored to wall-clock recording start at ~1.15s, not to impact. Where it lands relative to the swing depends on swing tempo and pre-shot routine, not swing mechanics. Pattern across 12 May 2–3 swings: swings 2, 3, 4 of every session have their hole start 867–1159ms after recording start (n=9, 292ms spread); every “swing 1” has its hole start 300–500ms later (1468–1677ms after recording start, n=3) — a warmup artefact whose mechanism is not yet identified.
 
**Per-swing evidence (session-2026-05-02T08-46-19, the canonical four-swing dataset):**
 
| **Swing** | **JSON entries** | **Hole length** | **Hole at JSON idx (t-range)** | **Position relative to P7 (impact)** |
| --- | --- | --- | --- | --- |
| sw1 | 186 | 355 ms | 66 → 67 (1709 → 2064 ms) | Hole midpoint = P7 − 178 ms (just before impact) |
| sw2 | 187 | 364 ms | 45 → 46 (1162 → 1526 ms) | Hole midpoint = P7 + 308 ms (early follow-through) |
| sw3 | 186 | 343 ms | 45 → 46 (1182 → 1525 ms) | Hole midpoint = P7 + 324 ms (early follow-through) |
| sw4 | 181 | 343 ms | 45 → 46 (1186 → 1529 ms) | Hole midpoint = P7 + 276 ms (early follow-through) |
 
Sw2/3/4 hole-start times cluster within a 24ms spread despite three independent swings at slightly different paces — strong signal that something fires on a clock, not on a swing event. Sw1 is the warmup outlier (~500ms later) and is also the only swing where the hole lands inside the downswing window rather than after impact — so it’s the canonical case for why the hole matters even though it’s atypical in timing.
 
**What’s ruled out:**** **camera dropping frames (video has all frames inside the hole, visually clean); MediaPipe failing on motion blur (pose confidence is healthy on both sides — sw1 leftWrist 0.946 before / 0.951 after); main-thread blocking (no catch-up burst when pipeline resumes — if the JS event loop had been frozen, MediaPipe results would have queued during the freeze and fired as a burst of small-Δt entries on resume; the observed first entry after the hole is ~25–32ms from the next, exactly like before, so MediaPipe was not producing-and-queuing during the hole, it was not producing at all); lighting / auto-exposure (May 3 indoor and outdoor produced identical hole positions). **Best current explanation, not yet confirmed:**** **MediaPipe itself stops producing pose results for ~350ms once per recording, mechanism unknown — the v0.4.1.11 MediaRecorder-warmup fix is consistent with this being a one-time encoder-startup cost on the live capture stream. **Why it matters:**** **when the hole lands inside the [P4..P7] downswing window, the segmenter has only sparse data to choose P7 from (sw1 above is the canonical case — only three JSON samples between top and impact, chosen P7 ~150ms off from true impact). v0.4.2 was one possible escape but is now halted (see “v0.4.2 status”).
 
**JSON frames[] ≠ recorded video frames**** **(capture pipeline mismatch). The live pose pump runs at ~37fps (device-paced) but MediaRecorder encodes the saved video at 30fps. When the pose loop ticks faster than the camera produces new frames, MediaPipe re-processes the same camera frame multiple times and writes distinct JSON entries (distinct timestamps, distinct landmark detections) all pointing at one underlying video frame. Measured: ~186 JSON entries vs ~150 video frames per swing across 4 swings, ratio 1.24×, ~24–32% of video frames have 2 JSON entries pointing at them.
 
**Three implications:**** **(1) P7 ties between adjacent JSON indices may be floating-point noise on identical landmark data, not a real golf signal; (2) per-side tempo timings (frames → ms) are inflated by ~24% in JSON-land, though the tempo *ratio* survives (numerator and denominator inflate equally); (3) contact-sheet validation has known fuzz — adjacent identical extracted images at adjacent JSON indices are expected, read as ties. **Status:**** **acknowledged, not fixed. A cheap band-aid (dedupe identical landmark rows before scoring) is available if needed; not applied. The clean fix lived inside the v0.4.2.0 record-first work, which is now halted.
 
**Shoulder L/R label-swap during fast rotation.**** **Less severe than wrist version. Shoulders are individually well-tracked (visibility 1.00 throughout swing) but MediaPipe re-assigns left/right labels mid-rotation when the chest faces the target. Detected in ~half the April 28 tempo-session swings (faster swings) and rarely in head-stability swings (slower). Means body-rotation-derived signals need defensive code that doesn’t trust the labels. v0.4.1.6’s shoulder-rotation P7 signal uses Math.abs(rShoulder.x − lShoulder.x) for exactly this reason.
 
**Held-finish swings can produce**** ****“****Incomplete swing****”**** ****verdicts.**** **When the user holds the finish position past the end of the recording window and the segmenter can’t find a second minimum. Edge-aware prominence handles the common case; held-finish remains the residual.
 
**Camera-motion assumption.**** **All algorithms assume the phone is on a tripod (or otherwise stationary). Hand-held phone use will degrade segmentation and any displacement measurement.
 
## Tunables / calibration gaps
 
**Per-user shoulder-width calibration for head stability.**** **Currently hardcoded ASSUMED_SHOULDER_INCHES = 16. Tape-measure-based calibration is ~10–15% per-user variance, which propagates directly into reported displacement inches. Upgrade once we have multiple testers.
 
**Per-user verdict baselines.**** **Once anyone has 50+ swings, median of their “normal” swings becomes their personal “in range” anchor. Universal thresholds become absolute fallback. Activates after segmenter is hardened.
 
**Vertical**** ****“****same****”**** ****threshold (1″) for head stability.**** **Most swings register P7 vertical < 1″, so vertical reports “same” most of the time. Ground-truth data showed vertical genuinely is small (typical P7 vertical: 0.0–0.8″ for normal swings). Threshold isn’t masking signal but may still warrant tuning.
 
# Open design questions / not decided yet
 
## Per-axis, per-gate segmenter — shipped, pending validation
 
v4.5 hypothesis (May 7, 2026, **shipped in v0.4.1.13 on May 7 evening**): the segmenter has been using the wrong axis for the gates that happen during predominantly horizontal hand motion. P1 = wristX motion-onset; P7 = wristX return through P1-x as primary (legacy median of shoulderRot/wristY/elbowY as backup); P4 = wristY argmin (unchanged); P10 = wristY argmin (unchanged). The 3-swing May 6 outdoor session eyeball-validated wristX-onset for P1 spot-on on 3/3, and validated legacy median (the v4.5 backup) within ±1 frame for P7 on 3/3. **wristX-return-primary itself is unmeasured on that session — that****'****s what the post-v0.4.1.13 validation is for.**** **Datasets queued: May 1 12-swing outdoor, indoor shadow-swing, and a re-run of the May 6 session under v0.4.1.13. See Active blockers Priorities 1 and 2.
 
## v0.4.2 status — HALTED
 
**Decision (May 3, 2026):**** **v0.4.2 (record-first / 60fps / Savitzky-Golay) is halted. No work planned.
 
**History:**** **v0.4.2.0 was built on April 29, 2026 — an architectural split where MediaPipe runs offline over the saved MediaRecorder blob after recording ends, instead of live during capture. Worked correctly on PC (Edge / Chromium). On April 30, nine WebKit-specific patches (v0.4.2.0.1 through v0.4.2.0.9) all failed to produce usable data on iPhone Safari. Final symptom: pose.send() is called successfully, no exception thrown, but onResults never fires. Root cause never fully identified. April 30 EOD decision: roll back to v0.4.1.6 on iPhone, keep v0.4.2.0.9 on PC as architectural validation only. May 3 decision: halt entirely, do not revisit.
 
**What v0.4.2 would have addressed if it had worked:**** **the JSON↔video duplicate-frame mismatch (offline pose runs once per video frame by construction); the per-recording ~350ms hole (offline pose doesn’t depend on live timing); the 30fps live-pose ceiling (60fps capture is unblocked once pose is offline). The hole is now provisionally resolved by a different route (MediaRecorder warmup); the other two remain known limitations.
 
**Why halted rather than deferred:**** **the iPhone WebKit failure has no diagnosed root cause and no cheap path forward. The architectural improvement promised by v0.4.2 does not in itself fix the gate-placement bugs, which exist at the algorithm layer too and must be fixed there regardless. Continuing to chase iPhone WebKit quirks without controlled tests is the tarpit lesson from April 30.
 
**If revisiting ever becomes attractive,**** **the trigger would be: clear evidence that a current limitation is blocking shipping a needed drill AND that no algorithm-layer fix can address it. The architecture is documented in the Decision archive (“v0.4.2.0 — Record-first / analyze-after”) for that case.
 
## Lost-frames hole — provisionally resolved, pending more sessions
 
v0.4.1.11 (May 6, 2026) added a 2s MediaRecorder warmup phase before pose collection begins. First session under it showed zero gaps in two 8-second recordings (636 frame intervals, all 23–28ms). This answered hypotheses (a) and (b) from v4.1: priming MediaRecorder eliminates the hole, and the hole was a one-time encoder-startup cost rather than a periodic clock-driven artefact. See Known Limitations entry. Status: provisionally resolved pending 2–3 more confirmation sessions across mixed conditions (indoor shadow with more swings, outdoor range, multi-swing-at-the-5-swing-limit). If the hole stays absent, this section moves to the Decision archive and the warmup becomes a permanent feature of the capture pipeline. If it reappears, hypothesis (c) — code-level instrumentation in the live onResults handler — becomes the next experiment.
 
## Head stability layout simplification (deferred)
 
User feedback April 27: the v0.4.1.2 two-row “Top + Impact” panel is cramped. Probable simplification: single Impact line, big text, drop Top from panel (still in CSV/JSON). Spoken phrase already drops the Top gate per April 27 spec — only Impact is spoken (with numbers: “vertical down 2, horizontal forward 4”). Layout change deferred until segmenter is hardened.
 
## POST_READY_DELAY_MS tuning (1500 → 2000ms)
 
Known polish item. Field testing surfaced an iOS Safari TTS quirk — Safari sometimes plays a faint system sound when an utterance ends, making the cue feel like “ready · faint · BEEP” rather than “ready · BEEP.” Bumping POST_READY_DELAY_MS to 2000ms is the polish fix when next touched. Not blocking.
 
## Camera-angle picker UI
 
Deferred until first down-the-line drill (probably hip lead, drill #3). Currently both shipped drills work face-on, so the angle UI would be vestigial. Plumbing for cameraAngle is already in place.
 
## IndexedDB schema (deferred v0.3.4)
 
When persistence comes back: one store keyed by swing timestamp? Two-store (sessions + swings, joined by sessionId)? Eviction policy if Safari clears storage? Session boundaries: new session = new sessionId, or flat timeline with groupings derived from timestamps?
 
## Multi-session review UX (deferred v0.3.5)
 
How does the review screen change when there are multiple saved sessions? Toggle per session ID? Time-range picker? “Last 10 sessions” default view? Depends on v0.3.4.
 
## Wake word / push-to-talk vs continuous listening
 
Currently continuous. No issues in phone+AirPods testing. Revisit if false triggers appear in extended field use.
 
## CSV schema across drills
 
Settled in v0.4.0: wide CSV with universal core columns (swing_number, timestamp, drill, camera_angle, verdict, verdict_class, shape, four phase indices) plus per-drill metric columns. Empty cells where drill doesn’t apply. Confirmed working in production.
 
## Other open questions
 
Camera setup guidance — lighting, framing, distance. Need onboarding UX that catches bad setup before analysis suffers.
 
Session structure — open-ended practice vs guided drill blocks with rep counts.
 
First-run onboarding — calibration swings? tutorial? how much hand-holding?
 
LLM choice and prompt design for session-end coach (v0.6+).
 
Monetization — free, freemium, subscription.
 
Data privacy posture and required terms / disclaimers.
 
Native port timing — when does web stop being enough?
 
# Roadmap
 
| **Version** | **Focus** | **Status** |
| --- | --- | --- |
| v0.1.6 | Tempo detection, tap-to-start, on-screen chart | Shipped, tagged |
| v0.2.0 → v0.2.4 | Voice trigger, hands-free loop, shape reporting, audio chime, debug tooling | Shipped, field-tested |
| v0.3.0 | TTS feedback (verdict + ready echo + shape echo), countdown removed | Shipped, field-tested |
| v0.3.1 | Start screen with drill picker + shape/voice toggles; End-session button | Shipped, field-tested |
| v0.3.2 | Golfer-facing result panel | Shipped, field-tested |
| v0.3.3 | Session review screen + CSV export | Shipped, field-tested |
| v0.3.4 | IndexedDB persistence | Deferred — revisit when user feedback calls for it |
| v0.3.5 | Multi-session review UX | Deferred — depends on v0.3.4 |
| v0.4.0 | Multi-drill infrastructure (drill registry, per-drill dispatch) | Shipped |
| v0.4.1 | Head stability as second drill, drill picker | Shipped |
| v0.4.1.2 → v0.4.1.10 | P7 detection iteration (multi-signal → addressMatch → body-only) | Shipped |
| v0.4.1.11 → v0.4.1.12 | MediaRecorder warmup (lost-frames hole fix) + GO tone + button-red on swing-window-open | Shipped |
| v0.4.1.13 | P1 horizontal motion-onset + P7 wristX-return primary (legacy median backup) — both axis fixes in single release | Shipped, awaiting field-validation per v4.5 hypothesis |
| v0.4.2 | Capture pipeline rework — record-first / 60fps / Savitzky-Golay | HALTED (May 3, 2026) |
| v0.5+ | Remaining drills (hip lead, completed backswing, weight shift, posture) | Planned, gated on segmenter |
| v0.6+ | LLM-generated session summaries (the retrospective coach) | Planned |
| Post-v1 | Native port; user accounts; persistence (if still wanted) | Planned |
 
## Forward-compatibility placeholders already in the data model
 
Every swing record includes a userId field, always set to “local” in v0.x. Reserves a slot so that if accounts are introduced later, existing records can be migrated without a schema change. Costs effectively nothing today; buys a clean migration path later.
 
## Per-user baselines (planned post-segmenter)
 
After ~50 swings, grade metrics against the user’s own distribution rather than universal thresholds. Tempo that’s “bad” for a tour pro may be “great” for a senior amateur. Free intelligence sitting in collected data.
 
## The retrospective coach (v0.6+)
 
At session end, an LLM receives the full set of swing records and generates a summary. Example: “Your hip lead was great — 18 of 25. But I noticed your head drifted more than usual when you rushed the backswing. Want to work on that tomorrow?” Major planned differentiator. The app spots patterns the user didn’t ask about. Numbers are queryable in ways that video isn’t.
 
## Reference-only material
 
GolfDB and SwingNet are CC BY-NC 4.0 (non-commercial). Useful as reference for the 8-phase swing taxonomy and accuracy benchmarks, but not to be shipped or used to train bundled models.
 
## Club detection — skipped
 
Clubs are hard to detect (thin, motion-blurred at impact). v1 infers club position from hand keypoints, sufficient for every drill in the starter library. Revisit post-v1 if a drill genuinely requires the club.
 
## Model strategy
 
MediaPipe + rule-based logic gets us a capable v1 with zero training cost. Collect user data through normal usage. Once MediaPipe’s specific failure modes in our use case are clear, consider training targeted custom models.
 
**Future model swap, parked:**** **MoveNet Thunder via TensorFlow.js is the realistic next lever if MediaPipe noise becomes blocking. Apache 2.0 (no licensing landmines), web-deployable, runs at WebGL/WebGPU. 17 keypoints which is a strict subset of MediaPipe’s 33 — every drill in the current and planned library uses keypoints in that subset, so drill code wouldn’t change. **YOLOv8 / YOLO11 Pose are off the table**** **— Ultralytics ships them under AGPL-3.0, which would force opening the entire app’s source under AGPL. Commercial license is per-product fee. Not a free swap. Flagged so a future Claude doesn’t go down that path.
 
# Recent changes (newest first)
 
## Brief bumps (v3.7+)
 
**v4.6 — May 7, 2026 (evening):**** **v0.4.1.13 shipped, implementing the v4.5 hypothesis. P1 detector now uses |Δ wristX| motion-onset (5 confirming frames, 0.005 units per frame, abs-direction-agnostic). P7 detector now uses wristX-return-through-P1-x as primary, with legacy median (wristY/elbowY/shoulderRot argmax) as backup when the crossing is ambiguous. P4 and P10 unchanged on wristY. addressMatch retained as a tertiary diagnostic. Swing record persistence expanded: wristXReturn, legacyMedian, addressMatch, three legacy candidates, usedDetector flag, spread, lowConfidence; full smoothed wristX trace persisted alongside wristY. Result-screen diag panel surfaces all candidates and the detector-used flag for live testing. Active blockers Priorities 1 and 2 flipped from "decide and code" to "validate against extracted frames" — the gating shifted from architecture decision to field data. Section copy cleaned up per Patrik's note that it was confusing on read; deeper restructure deferred to v4.7 once validation results are in.
 
**v4.5 — May 7, 2026:**** **P7 detector hypothesis revised. The v4.4 finding is reframed as a single structural hypothesis: the segmenter has been using the wrong axis for the gates that happen during predominantly horizontal hand motion (P1 takeaway, P7 impact). Under this framing, the v4.4 P7 proposal of “switch addressMatch → legacy median” is replaced by “switch addressMatch → **wristX return through P1-x as primary**, with legacy median as backup.” Structurally consistent with the P1 wristX-onset fix — both are horizontal-motion gates being detected on the wrong axis. wristX-return-primary is not directly measured on the May 6 session (the figures there measured legacy median, the v4.5 backup signal); validation set expanded to (a) May 1 12-swing outdoor, (b) indoor shadow-swing, (c) re-run May 6 outdoor under the new primary. Active blockers Priority 1 reframed accordingly: no longer a single-line code change, but still smaller than the P1 rewrite. Priority 2 (P1 wristX-onset) substantively unchanged but sequenced under the unified-framing lens. Lead-in paragraph added to Active blockers establishing P1 and P7 as two implementations of one structural fix.
 
**v4.4 — May 7, 2026:**** **Segmenter signal-axis findings absorbed from the standalone Brief_patch_4.4 doc. Single-session investigation (session-2026-05-06T18-00-15, 3 outdoor swings) produced a structural finding: different gates are best detected on different axes because hand motion at each gate is dominated by a different axis. New per-gate signal map: P1 = wristX motion-onset (current vertical-only logic blind to early takeaway, fires ~5 frames late); P4 = wristY argmin (current, correct); P7 = legacy median of (shoulderRot, wristY, elbowY) (drop addressMatch, which now over-corrects and fires 3–6 frames early on outdoor data); P10 = wristY argmin (current, correct). *[The P7 portion of this finding was superseded in v4.5 — the new primary is wristX-return-through-P1-x, with legacy median as backup. P1, P4, P10 portions unchanged.]* Eyeball validation 3/3 spot-on for P1, 3/3 within ±1 frame for P7. Active blockers reshuffled: P7 switch was Priority 1 (single-line change in v4.4, restructured as wristX-return-primary in v4.5); P1 wristX-onset rewrite is Priority 2; missing-frames hole confirmation drops to Priority 3; lowConfidence threshold tightening Priority 4; sw3/sw4 P7 eyeball-validation drops to Priority 5. Findings only — no code change shipped. Validate before adopting.
 
**v4.3 — May 6, 2026 (evening):**** **Lost-frames hole provisionally resolved. v0.4.1.11 added a 2s MediaRecorder warmup before pose collection begins; first session under it (session-2026-05-06T15-41-15, 2 indoor shadow swings, 8s window for the test) showed zero gaps in 636 frame intervals. This answered hypothesis (a) (priming MediaRecorder eliminates the hole — yes) and hypothesis (b) (the 8s window would have shown periodic recurrence at ~2.3s if the hole were clock-driven — no, it didn’t recur), pinning the cause to a one-time MediaRecorder encoder-startup cost. v0.4.1.12 layered on: recording window reverted 8s→5s (the 8s test-window blew the 36MB session-export limit at >2 swings), GO tone added at the swing-window-open moment (rising chirp 600→1000Hz, ~250ms, ~2× louder than the v0.3.3.3 tick), button-red flip + REC indicators + status text moved from warmup-start to swing-window-open so visual+audio cues land together at the moment the player can actually swing. Active blocker priorities re-shuffled: P1 detection promoted Priority 3→Priority 1 (Patrik’s flagged next priority); v0.4.1.10 outdoor P7 validation stays Priority 2; missing-frames-hole confirmation across 2–3 more sessions is the new Priority 3.
 
**v4.2 — May 3, 2026 (late evening, cleanup pass):**** **Consolidation pass to retire two supplementary docs (“Lost video frames investigation 2.0” and “v0.4.1.10 handover addendum may3”). Two pieces of evidence absorbed into the missing-frames Known Limitations entry: (a) per-swing evidence table for session-2026-05-02 (sw1–sw4: hole length, JSON index, t-range, position relative to P7), so the “wall-clock-anchored at ~1.15s” claim is supported in-doc with the four-swing data; (b) the no-catch-up-burst reasoning that rules out main-thread blocking — explicit explanation that a frozen event loop would queue MediaPipe results and fire a burst on resume, observed cadence resumes cleanly with no backlog, so MediaPipe was not producing-and-queuing during the hole. No factual changes to project state. The brief is now fully self-contained — supplementary docs can be deleted.
 
**v4.1 — May 3, 2026 (evening, post-review):**** **Four corrections from Patrik review of v4.0. (1) Active blocker priorities reordered: missing-frames hole investigation is now Priority 1, P7 outdoor validation is Priority 2, P1 detection demoted to Priority 3 — no segmenter conclusion is trustworthy until we know whether the data going in is whole. (2) Old “Priority 2: ground-truth-validate” replaced with concrete next step (extract video frames at sw3/sw4’s impactIdx, eyeball them). (3) Capture frame rate corrected: code requests 30fps via getUserMedia, not 60fps. v4.0’s “60fps cap” was the theoretical iOS Safari ceiling, not what we run. Platform table now shows currently-shipped vs theoretical-ceiling rows separately. (4) Environmental requirements rewritten to match actual setup: tripod 20–30cm high at the end of range ball tray, 1–2m from golfer, steep upward angle. New calibration-gap note added: shoulder-width-based inches calibration assumes perpendicular view, breaks at steep upward angles, magnitude unknown — head-stability inches should be read as relative until investigated. ZIP bundle download confirmed shipped: Download bundle (videos+JSON, N) button is hidden when no swing has a captured video; check that the session contains at least one fully-completed swing with video.
 
**v4.0 — May 3, 2026 (evening):**** **Structural rewrite. No factual changes except (a) corrected head-stability spoken-phrase description to match shipped code (numbers included), and (b) v0.4.2 formally halted. Brief reorganized to lead with current state and put version history in a Decision archive. Plumbing/Drills architecture promoted to a dedicated front-and-center section with inventory table and drill API contract. Versioning convention updated to allow minor bumps for structural rewrites of the brief itself.
 
**v3.11 — May 3, 2026 (evening):**** **Lost-frames hole re-investigated with two new sessions (May 3 indoor + outdoor, 8 swings). Re-filed from “structural Known Limitation, unfixable without v0.4.2” to “active investigation under Open design questions.” New diagnosis: every recording has exactly one ~350ms hole anchored to wall-clock recording start at ~1.15s, not to impact. Lighting hypothesis ruled out by indoor/outdoor comparison. Swing 1 of every session is a separate warmup outlier. JSON frames[] ≠ recorded video frames finding still stands. P1 detection remains priority blocker.
 
**v3.10 — May 3, 2026 (morning):**** **JSON frames[] and recorded video frames are not 1:1 — discovered while eyeballing the v0.4.1.10 first field-validation session. Live pose pump runs at ~37fps (device-paced) but MediaRecorder encodes at 30fps; ~24% duplicate JSON entries pointing at same video frame. Closes the v3.4 “live-pump fps ~38” thread — that finding was the cause; duplicate JSON rows are the effect. New Known Limitation entry filed.
 
**v3.9 — May 1, 2026 (evening):**** **v0.4.1.10 ships body-only landmark set for addressMatch (drops wrists and elbow; adds shoulders; keeps nose and knees). Triggered by May 1 indoor shadow-swing session where addressMatch fired P7 +2 frames late on every swing. Diagnosis: MediaPipe wrist landmarks go to garbage during impact-blur frames despite reporting high visibility. Body-only re-scoring hits true impact within ±1 frame on 4/4 indoor and shifts May 1 outdoor 12-swing set ~2 frames earlier on 10/12 swings. Outdoor truth not eyeball-validated post-fix; first-session-after-deploy is the field-validation gate. New Known Limitation filed: MediaPipe wrist visibility unreliable under motion blur. **SUPERSEDED in part by v4.4/v4.5:**** **the body-only narrowing fixed the wrist-blur late-bias but introduced an early-bias on outdoor data; v4.5 proposes switching the primary P7 signal to wristX-return-through-P1-x with legacy median as backup.
 
**v3.8 — May 1, 2026 (afternoon):**** **addressMatch ships as primary P7 detector in v0.4.1.9, replacing v0.4.1.6 median-of-three. Stage 1 ran on 6 indoor shadow swings; Stage 2 on a fresh 12-swing outdoor session. addressMatch beat median-of-three: better on 8/12, tied on 4/12, worse on 0/12 outdoor; mean absolute error halved (2.0 → 0.92 frames); systematic late-bias eliminated. Three legacy signals retained as diagnostics. P1 detection emerges as next priority. SUPERSEDED by v3.9 — the “addressMatch is done” framing was overturned by the wrist-landmark-quality issue surfaced May 1 evening. Further superseded by v4.4/v4.5 — the median-of-three signals are now the backup-not-primary, but the v4.5 hypothesis pivots to wristX-return-through-P1-x as the primary P7 signal.
 
**v3.7 — May 1, 2026 (morning):**** **Platform section gained explicit web-vs-native trade-off matrix and a re-examination log. P7 problem reframed: velocity-at-impact is not the primary cause of the index bias; the dominant cause is structural (argmax of smoothed asymmetric signal). Pattern-matching against the swing’s own address pose proposed as next direction.
 
## Release entries (v0.4.1.6+)
 
**v0.4.1.12 (May 6, 2026, evening):**** **Recording window reverted 8000→5000ms after v0.4.1.11’s 8s test-window proved both that the warmup eliminates the missing-frames hole (zero gaps in two 8s recordings) and that the 8s window blew through the 36MB session-export ZIP limit at >2 swings. Replaced the legacy 440Hz tick with a clearer GO tone (rising chirp 600→1000Hz over 180ms, ~250ms total envelope, peak gain 0.25 — distinct from anything else in the app). Moved the audio cue from warmup-start to swing-window-open: silence through the warmup, then GO tone at the instant pose collection begins. Moved the button-red flip + “Stop” label + REC indicator + REC border + “Recording…” status text from warmup-start to swing-window-open as well — v0.4.1.11 had left these UI flips at warmup-start (carryover from pre-warmup behaviour), so the button was red for 2s while the player couldn’t yet swing. Now visual+audio cues are simultaneous and unambiguous: amber “…” through warmup, then GO tone + red “Stop” the moment the swing window opens. JSON version bumped 0.4.1.11→0.4.1.12 in all three locations (sessionConfig, swing record, session record). Status text updated from stale “on the beep” to “on the tone”.
 
**v0.4.1.2 (April 27, 2026, evening):**** **Head-stability redesign — gate-based reporting, impact-driven verdict. v0.4.1 ship was algorithmically right but display-wrong. Verdict driven by impact (P7) only, not full-swing peak. Either-axis impact displacement above IMPACT_OFF_INCHES (4″) → off. Result panel: two rows, “Top V: down 2 H: forward 2 / Impact V: down 2 H: forward 4”. Direction vocabulary: vertical up/down/same, horizontal forward/back/same. spokenPhraseOverride mechanism added — plumbing’s first concession to drills that need richer speech.
 
**v0.4.1 (April 27, 2026, afternoon):**** **Directional verdict vocabulary + head stability drill + drill picker. Tempo verdict-class vocabulary expanded to good/below/above/error. Head stability drill added as DRILLS.head_stability, the first real test of the v0.4.0 plumbing/drill seam — required ZERO plumbing changes other than the picker UI. Drill picker: two real selectable tiles on start screen.
 
**v0.4.0 (April 27, 2026, morning):**** **Plumbing/drill separation — architectural release, no new user-facing functionality. Tempo logic lifted from plumbing into a DRILLS.tempo registry entry. Drill registry shape locked. Camera angle wired through as first-class sessionConfig.cameraAngle. Field-validated by running tempo through the new structure (5/5 clean smoke test). Deprecated and removed: COUNTDOWN_SECONDS constant, legacy playBeep / playShapeChime helpers, startCountdown, per-swing raw save.
 
**v0.3.3.3 (April 27, 2026):**** **Single-beep cue. v0.3.3.2’s three-beep cue was confirmed effective but unnecessarily ceremonial. Simplified to a single beep at end of POST_READY_DELAY_MS. April 27 13:00 CEST session: 10/10 swings segmented cleanly.
 
**v0.3.3.2 (April 27, 2026):**** **Auditory ready-to-swing cueing — three beeps. A field session at floor-mounted phone setup exposed that the visual grey→red “Recording” transition was unreliable as a “time-to-swing” signal. Reinstated tick/beep tone in three-beep pattern. Re-established a long-deprecated principle: auditory cueing is primary for time-of-action signals; visual is supplementary. SUPERSEDED by v0.3.3.3.
 
**v0.3.3.1 (April 2026):**** **Diagnostic infrastructure for data-first drill development. Session-wide raw JSON dump added to review screen. sessionSwings retains per-swing frames[] array. Pure dev tooling.
 
**v0.3.3 (April 2026):**** **Session review screen. End session routes through a review screen showing aggregate stats, distribution chart, averaged swing-shape chart, per-swing table. CSV download. In-memory only (data evaporates on Discard).
 
**v0.3.2 (April 2026):**** **Golfer-facing result panel. Big centered tempo ratio as headline, plain-language comment, shape line. Dev content (chart, diagnostics, transcript log, save raw data) moved behind a “Show debug” text link. First field-test-driven size tuning: comment and shape text bumped to clamp(24px, 5vw, 36px) for readability at ~1.5m.
 
**v0.3.1 (April 2026):**** **Start screen + drill picker + End-session flow. Landing page with drill tile, shape and voice toggles moved out of in-session controls. Audio/TTS unlocks from any start-screen tap. Swing record gains drill field.
 
**v0.3.0 (April 2026):**** **TTS replaces the tone/visual feedback layer. “ready” echoed via speech; tempo verdict spoken; recognized shape words echoed back. Countdown tick/beep removed from user flow. Shape-chime removed. First field-test-driven principle change: the original “no TTS in v0.3” rule was relaxed because field testing showed visual confirmation is unreadable outdoors.
 
**v0.2.4 (April 2026):**** **Expanded shape-word matcher with Safari mishear tolerance; persistent Last-Heard display; transcript debug log; chime simplified to single-oscillator for desktop Edge compatibility.
 
**v0.2.3 (April 2026):**** **Audio chime confirmation when a shape word is recognized. SUPERSEDED — chime removed in v0.3.0 (TTS replaces it).
 
**v0.2.2 (April 2026):**** **Peak detector accepts plateau minima; edge-aware prominence; recording window bumped 3s → 4s. Fixed “Incomplete swing” false negatives.
 
**v0.2.1 (April 2026):**** **Hands-free session loop (auto-stop recording + waiting state + shape reporting toggle).
 
**v0.2.0 (April 2026):**** **Voice trigger (“ready” replaces tap Start). Initial v0.2 ship; had manual-stop gap requiring v0.2.1 fix.
 
**v0.1.6 (April 2026):**** **First working version, tagged release. Tempo detection validated on iPhone Safari and Edge.
 
**v0 (pre-code):**** **Design discussions in Golf_app_conversation.docx and original golf_coach_app_brief_v1.md.
 
## What was tried and shelved
 
**v0.4.2.0 (April 29 → April 30, 2026):**** **Record-first / analyze-after architectural split. Capture and analysis become separate passes: live pose pump becomes overlay-only, MediaRecorder writes to a video blob during recording, then runOfflinePoseOverBlob(blob) plays the saved blob through a hidden  element and runs MediaPipe over each frame to produce the same frames[] array the live pump used to emit. Worked correctly on PC (Edge / Chromium) throughout. **iPhone result:**** **nine WebKit-specific patches (v0.4.2.0.1 through v0.4.2.0.9) all failed — symptoms ranged from zero frames to frozen UI to ~38% unique-timestamp output. Final symptom: pose.send() called successfully, no exception, but onResults never fires. Root cause never fully identified. **Decision (April 30 EOD):**** **rolled back to v0.4.1.6 on iPhone, kept v0.4.2.0.9 working on PC as architectural validation. **Final decision (May 3, 2026, v4.0):**** **halted entirely. Lesson for future Claudes: iOS Safari quirk patches without controlled tests behind them are a tarpit. The architectural improvement promised by v0.4.2.0 does not in itself fix the gate-placement bugs found at the algorithm layer; those exist in v0.4.1.6 too and must be fixed at that layer regardless.
 
**iOS multi-download closure bug (v0.4.1.3 era) — FIXED in v0.4.1.7.**** **Originally parked because hardening workflow had moved to desktop Edge. Surfaced again when iPhone-only validation work needed it; root-caused as a closure-on-stale-mediaRecorder bug + Safari multi-download throttle, both fixed.
 
**Mirror question (front-facing selfie cam vs MediaPipe coordinates) — NOT a bug.**** **Selfie cam is mirrored on iOS Safari, but MediaPipe processes the displayed pixels, so the keypoint x-coords match what’s on screen. Algorithm sign convention +x = toward target is correct for the right-handed face-on setup. Apparent contradiction (deliberate-forward attempts reading “back”) was resolved when ground-truth video showed the user’s HEAD genuinely doesn’t move forward at impact even when the body lunges forward. Documented here so we don’t waste a half-day re-investigating.
 
**Vertical**** ****“****same****”**** ****threshold (1″) initially suspected too high.**** **Ground-truth data showed vertical genuinely is small (typical P7 vertical: 0.0–0.8″ for normal swings). The threshold isn’t masking signal; the measurement is matching reality.
 
## Why v0.3.4/5 deferred (architectural decision)
 
After v0.3.3 shipped, the observation: the remaining v0.3 work (persistence, multi-session review) deepens the single-drill pipeline rather than broadening what the app does. Adding more drills grows the product in a way that creates usage, which creates user feedback, which drives what to build next.
 
**Principle applied:**** **persistence is a retention feature, and retention matters only after the product does something worth retaining for. With one drill, there is no reason for a user to return — so no reason yet for their data to persist between visits. Multiple drills give them a reason to come back; persistence then becomes a legitimate user need.
 
The CSV export (v0.3.3) is the lifeboat in the meantime. Neither deferred version is cancelled. When real users ask where their history went, that is the signal that v0.3.4 is the next thing.
 
# Brief versioning convention
 
Bump the brief version on every meaningful update. This document had been stale at v3 across 12+ code releases — that’s a documentation bug to actively avoid going forward.
 
**Convention:**
 
**Patch bumps (v3.1, v3.2, …, v4.1, v4.2, …)**** **for content updates within an active development phase: new release entries, resolved investigations, known limitations, hypothesis updates. Most updates fall here.
 
**Minor bumps (v4.0, v5.0, …)**** **for one of two reasons:
 
**Strategic direction or scope change**** **— new product pillar, deprecation of a major architectural decision, pivot away from a previously-locked principle.
 
**Structural rewrite of the brief itself**** **— the document’s organization changes meaningfully (e.g. v4.0’s restructure to lead with current state and move version history into a Decision archive). Useful when the brief has accumulated chronological cruft and become hard to read fresh.
 
Update the “Last updated” date and write a one-line “What changed” summary at the top capturing what changed. The body of the brief is the truth; the summary line is the index.
 
**Future-Claude (or future-Patrik):**** **if you finish a working session and the brief content is now stale relative to what was discussed, bump the brief. Don’t wait for a milestone. Patrik explicitly asked for this.
 
# Hand-off note for the next Claude
 
You are inheriting an app at v0.4.1.13 with two shipped drills (tempo, head stability) and a freshly-shipped segmenter signal-axis fix that needs field validation. The single most important thing to know is the architectural rule in the Architecture: Plumbing vs Drills section above — read it first.
 
**The current phase is validation, not coding.**** **v0.4.1.13 implements the v4.5 hypothesis: P1 and P7 are horizontal-motion gates and now use wristX. P4 and P10 are vertical-extreme events and remain on wristY. The change is in code; it has not been validated against extracted video frames. That validation is the next gate.
 
**Where to focus (in order):**
 
**Validate v0.4.1.13 P7 detection against extracted frames.**** **Datasets: (a) May 1 12-swing outdoor session, (b) indoor shadow-swing data, (c) re-run May 6 outdoor (the session that produced the v4.5 hypothesis). Use extract_gates.py on the JSON's claimed impactIdx, eyeball for true impact. Each swing record now carries a usedDetector flag — note whether it's 'wristXReturn' (primary fired) or 'legacyMedian' (fallback fired); patterns there are themselves a finding. Three outcomes shape next steps: primary mostly correct → validated, move on; primary systematically off → re-examine crossing definition; primary frequently doesn't fire and fallback does the work → reconsider whether P1 reference x-value is the right anchor.
 
**Validate v0.4.1.13 P1 detection against extracted frames.**** **Same datasets. Look for P1 firing the moment the clubhead actually starts moving back. The May 6 pre-shipping data (3 swings) showed a 4–5 frame earlier shift and 3/3 eyeball-correct, but the handedness × camera-angle assumption is not yet stress-tested.
 
**Confirm the missing-frames hole stays gone.**** **Piggybacks on the validation sessions. If the hole stays absent across 2–3 more sessions, promote the Known Limitations entry from "provisionally resolved" to "resolved" and v0.4.1.11's MediaRecorder warmup becomes permanent.
 
**No drill #3 ships until segmenter success rate is ****>****95%**** **on a labeled JSON+video dataset.
 
**What NOT to chase:**
 
**v0.4.2 (record-first / 60fps / Savitzky-Golay) is HALTED.**** **Don't spin it back up without clear evidence a current limitation is blocking shipping AND that no algorithm-layer fix can address it. The April 30 lesson on iOS Safari quirk patches without controlled tests is real.
 
**Don****'****t reach for a model swap.**** **All general-purpose monocular 2D pose models struggle on golf swings (Yuan & Ndongmo 2025: MediaPipe OKS 0.636 vs YOLO 0.604 — neither sufficient without fine-tuning or temporal modeling). The foundation problem is "30fps live monocular pose," not "MediaPipe is wrong." Switching the model wins ~5–10% in one dimension and loses ~5–10% in another.
 
**Don****'****t reinvent the segmentation in a drill.**** **Consume the four phase indices from plumbing.
 
**Don****'****t tune v0.4.1.13 thresholds before validation data is in.**** **The 0.005-unit tolerance band for the wristX-return crossing, the 8-frame lowConfidence threshold, the abs() handedness assumption — all of these are first-principle defaults that may need adjustment, but the right time to adjust is after seeing how they perform on real frames, not before.
 
**The data-first principle:**** **before making a segmenter change, ground-truth-validate against extracted video frames. JSON-only checks let bad gate placement hide behind plausible-looking numbers. The extract_gates.py script in the project is the tool. Both the v3.x P7 late-bias finding and the v4.4/v4.5 P7 early-bias finding were caught by this technique and missed by JSON metrics.
 
**The brief:**** **if you make a meaningful change in conversation that the brief should reflect, bump it (patch bump for most things, minor for restructures or strategic shifts) and update "Last updated" + the "What changed" summary. Don't wait for a milestone.
 
Welcome.
