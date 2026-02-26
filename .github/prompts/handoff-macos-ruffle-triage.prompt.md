## Handoff Prompt: Continue Ruffle Triage on macOS

You are continuing troubleshooting for an old Flex/Flash app in this repo, now moved from a VM (no reliable WebGL) to a local macOS 26.3 machine.

### Objective
Determine whether remaining failures are environment-related (GPU/WebGL/browser) or true Ruffle compatibility issues, and push the app to the furthest working state possible.

### Current Progress (Already Done)
1. Local hosting baseline is validated over HTTP.
   - `app.html`, `Main.swf`, `history/history.js`, and Flex `.swz` assets returned `200`.
2. SWFObject gating was relaxed for Ruffle boot.
   - In `app.html`: `swfVersionStr = "0.0.0"`, `xiSwfUrlStr = ""`.
3. Wrapper pages were repointed to local app.
   - `index.html` iframe uses `app.html?intro=true`.
   - `index-pre-ga-removal-3-2022.html` iframe uses `app.html?intro=true`.
4. Additional bootstrap hardening in `app.html`:
   - Doctype updated to `<!DOCTYPE html>`.
   - Added `window.RufflePlayer.config` with:
     - `preferredRenderer: "canvas"`
     - `autoplay: "on"`
   - Added `embedSWF` callback console log:
     - `[global-footprint] embedSWF { success, hasRef, id }`

### Latest Observed Signals
- Positive:
  - `[global-footprint] embedSWF` showed `success: true`, `hasRef: true`.
  - Ruffle initialized and loaded `Main.swf`.
- Important warnings/errors seen in prior environment:
  - WebGL context creation failures (VM likely lacked support).
  - Ruffle runtime error: `Render backend does not support BitmapData.draw`.
  - AVM2/TextLayout runtime fault (`Error #1009` null object in text layout flow).
- Noise to ignore for app triage:
  - Most VS Code Extension Host / Copilot / embeddings / deprecation logs.
  - VS Code webview sandbox plugin errors (`Failed to load ... as a plugin, frame is sandboxed`) when using embedded preview.

### Critical Test Constraints for This Next Pass
1. Test in an external browser tab/window (Safari/Chrome/Firefox), not VS Code webview/simple browser/live preview.
2. Serve over HTTP from repo root (not `file://`).
3. Capture only app-relevant console/network output for `app.html` and `Main.swf` execution.

### One-Command Local Server (macOS)
From the repo root, run:

`python3 -m http.server 8000`

Then open:

- `http://127.0.0.1:8000/app.html`
- `http://127.0.0.1:8000/index.html`

### Next Bot Tasks (In Order)
1. Re-validate current file state in:
   - `app.html`
   - `index.html`
   - `index-pre-ga-removal-3-2022.html`
2. Start a local server and test in at least two browsers on macOS.
3. Confirm whether these still occur in external browsers:
   - `BitmapData.draw` unsupported error
   - TextLayout `Error #1009`
4. If errors differ by browser/GPU path, document a compatibility matrix:
   - Browser/version
   - Ruffle build/version
   - Renderer selected (canvas/webgl/wgpu)
   - Boot status + key runtime breakpoints
5. If still blocked, try one targeted iteration at a time:
   - Pin Ruffle to a different recent nightly or stable build.
   - Adjust Ruffle config minimally and retest.
6. Produce a short conclusion:
   - “Host/setup issue” vs “Ruffle feature gap” vs “mixed”.
   - Most reliable run recipe for this repo.

### Deliverable Format
Return a concise report with:
- What works now
- What fails now
- Repro steps
- Best-known configuration on macOS
- Remaining blockers and whether they are fixable in-repo
