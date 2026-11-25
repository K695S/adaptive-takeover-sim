# Adaptive Takeover Request Simulator

README prose word count: ~1,200 (code and commands not counted)

A browser prototype of an **adaptive takeover request** for an SAE Level 3 car on a motorway. It builds on my Assignment 1 literature review: instead of one fixed warning for every driver, the car estimates how ready the driver is from what they are doing and adapts the warning's timing and cues.

## Project status

| Part | Status |
|---|---|
| Request planner (lead time, cues, screen pause, escalation timing), fixed vs adaptive mode | Implemented |
| Rule-based readiness estimate (eyes on road / reading / film → High / Medium / Low) | Implemented |
| Escalation when no hands-on response within 2 s (4 s non-urgent); minimal-risk stop when time runs out (move to hard shoulder, brake) | Implemented (stop is simulated) |
| Personal adjustment: +1 s per slow takeover in the last two, max +2 s | Implemented |
| Manual driving model, takeover log (takeover time, lane deviation in metres, outcome), CSV export | Implemented |
| Driver activity sensing, sensor-error case | Simulated with buttons and a checkbox |
| Seat vibration, in-car screen, hazards (roadworks / exit) | Simulated on screen |
| Spoken prompt ("Please take over…", "Take over now") | Implemented (browser speech; falls back to tones) |
| "You have control" / "Automation resumed" confirmations | Implemented |
| Camera-based activity classifier, real hazard sensing, user study | Planned / not started |

The log shows what happened in a test run. It is **not** evidence that adaptation improves takeovers: the tester plays the driver, and lane deviation is an approximation (pixels converted assuming a 3.6 m lane).

## Technology stack

- **HTML, CSS, JavaScript** in one file (`index.html`), no libraries. Chosen so it runs in any browser and can be hosted on GitHub Pages.
- **Canvas 2D** for the road view; **Web Audio API** for chime and alarm tones; **Web Speech API** for the spoken prompt (no audio files).
- **Rule-based planner** rather than a trained model: there is no driver data to train on yet, and fixed rules make every decision traceable to a reference. A learned readiness model is undecided/planned.
- `docs/`: storyboards, system diagram (`diagram.svg`/`.png`), takeover log image.

## Running or inspecting

- Open `index.html` in Chrome, Edge or Firefox (double-click it), or use the GitHub Pages link: https://k695s.github.io/adaptive-takeover-sim/ (after GitHub Pages is switched on)
- Pick **Fixed** or **Adaptive**, pick a driver activity, optionally tick **Sensor error**, then press **Urgent: roadworks** or **Non-urgent: exit**.
- Take over with **Space** or an arrow key; steer with **← →**. Roadworks close the middle lane; for the exit, reach the left lane.
- The **Planner reasoning** panel explains each decision; the **Takeover log** records each episode and can be downloaded as CSV.
- Timings are constants at the top of the script (`FIXED`, `ADAPTIVE`, `ADAPTIVE_ESCALATE_AFTER`, `NONURGENT_FACTOR`).

## Development record

1. **Design from A1.** Turned the A1 findings into a rules table: lead time from Gold et al. (2013), multimodal cues from Politis et al. (2015), Zhang et al. (2019) and Wickens (2002), pausing the secondary task from Wintersberger et al. (2018).
2. **Core loop.** Built the road view, the fixed baseline (7 s, visual + chime) and the adaptive planner. Checked that each activity produced the lead time and cues in the table.
3. **Sensor error case.** Added a toggle where the sensor wrongly reports "eyes on road". In A1 I noted the inference "could be wrong"; without this the prototype only showed the system working when it is right. This led to adding escalation after 2 s without a hands-on response.
4. **Non-urgent escalation.** First version escalated non-urgent requests after 2 s too, which produced a loud alarm for an exit 16 s away. That contradicts the cry-wolf point (Breznitz, 1984), so non-urgent escalation time is now doubled. Checked: the non-urgent run no longer escalates when the driver responds within 4 s.
5. **Steering tuning.** In the first test runs the car hit the road edge and every lane deviation read 25 px, so the metric was measuring the clamp, not the driver. Reduced steering acceleration; values now vary (about 0.6–1.0 m).
6. **Personal adjustment.** Added the takeover history and +1 s rule to address the repeated-takeover gap (Hergeth et al., 2017), capped at +2 s so one bad run cannot change the system much.
7. **Evidence.** Scripted scenario runs (headless Chrome) to capture storyboard frames.
8. **Review round (v2).** Checked v1 against the marking rubric and fixed what it exposed:
   - *Spoken prompt:* v1 only played a tone pattern labelled "voice". Politis et al. (2015) tested language-based warnings, so the cue now speaks ("Please take over. Roadworks ahead in 8 seconds"; "Take over now" on escalation). Moved from planned to implemented.
   - *Control-transfer feedback:* v1 gave no confirmation when control changed hands, so the driver could not be sure who was driving (mode confusion, Week 2 feedback). Added "You have control" and "Automation resumed" messages with a short tone.
   - *Minimal-risk stop:* in v1 the car kept moving at full speed and drove into the cones when nobody responded. The hazard now sits 3 s beyond the request deadline, and the car moves to a drawn hard shoulder and brakes to a stop. The banner now says "TAKE OVER within X s", which matches what the countdown means.
   - *Units:* lane deviation is now reported in approximate metres instead of pixels.
   - *Images:* storyboards cropped to the road and the relevant panels with DRIVER/SYSTEM captions; the log redrawn as a readable table; the system diagram redrawn on a grid with straight, right-angled arrows because the auto-laid-out version was hard to follow.

9. **Spoken prompt says what to do (v3).** The prompt only said that a takeover was needed. It now also says which lane is closed and which way to move ("Middle lane closed in 8 seconds. Move left or right."; "Exit on the left…"), so the driver knows the action, not just the alarm (Politis et al., 2015; Endsley, 1995). Checked by running the urgent and non-urgent scenarios with sound on.

The repository holds the final version (v3), uploaded in one commit; the stages above are the development record. Submitted version: release **v3** (https://github.com/K695S/adaptive-takeover-sim/releases/tag/v3).

## AI assistance

- **Tool:** Claude (Anthropic).
- **Developing the system:** I set the design from my A1 sources (adaptive vs fixed requests, sensor error, escalation, logging). Claude wrote the code for each version and proposed fixes found during testing (steps 4, 5 and 8). I reviewed each change against my rationale table and A1 evidence before keeping it.
- **My own design decision (v3):** After reviewing the prototype with Claude, I decided the spoken prompt should say which lane is closed and which way to move, because Politis et al. (2015) found language-based warnings help drivers understand the situation. I tested it in the urgent film scenario.
- **Diagrams and storyboards:** Claude drew the system diagram and captured and laid out the storyboard screenshots. I checked that each implemented/simulated/planned label matches the code and each caption matches its screenshot.
- **Report and README:** The report is my own writing, based on my A1 and my design decisions. Claude gave me a first draft, and then proofread my final text for grammar and accuracy. Claude also drafted this README, which I reviewed. I checked every reference against my A1 reading list.

## Additional files

- `docs/storyboard1.png`, `storyboard2.png`, `storyboard3.png`: interaction storyboards used in the report.
- `docs/diagram.png` (source `docs/diagram.svg`): system diagram.
- `docs/log.png`: takeover log from the storyboard runs.
- No ZIP file; everything is in this repository.
