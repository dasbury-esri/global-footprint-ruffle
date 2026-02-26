# Ruffle triage report (2026-02-26)

## Scope

- macOS 26.3
- Local HTTP server: `python3 -m http.server 8000`
- URLs tested in external browser:
  - `http://127.0.0.1:8000/app.html?renderer=canvas`
  - `http://127.0.0.1:8000/app.html?renderer=webgl`
  - `http://127.0.0.1:8000/app.html?renderer=wgpu-webgl`
- Ruffle build: `0.2.0-nightly.2026.2.25`

## Current file state

- `app.html`
  - SWFObject gating relaxed (`swfVersionStr = "0.0.0"`, `xiSwfUrlStr = ""`).
  - Ruffle config uses URL param `renderer` with default `canvas`.
  - Console logs include renderer and `embedSWF` result.
- `index.html` and `index-pre-ga-removal-3-2022.html`
  - iframe points to `app.html?intro=true`.

## What works

- Ruffle initializes and loads `Main.swf` over HTTP.
- `embedSWF` success and `hasRef` are true.
- Renderer can be switched via `?renderer=` in URL.

## What fails

- `canvas` and `webgl` renderers:
  - `Error dispatching event "enterFrame": RustError("Render backend does not support BitmapData.draw")`
- `wgpu-webgl` renderer:
  - `Error #1009: Cannot access a property or method of a null object reference` (TextLayout stack).
- AudioContext warnings persist due to autoplay policy (not a blocker).

## Compatibility matrix (macOS, Chrome)

| Renderer | Boot | Loads SWF | Blocking error |
| --- | --- | --- | --- |
| canvas | yes | yes | BitmapData.draw unsupported |
| webgl | yes | yes | BitmapData.draw unsupported |
| wgpu-webgl | yes | yes | TextLayout Error #1009 |

## Conclusion

- Primary blockers are Ruffle feature gaps, not host/browser issues.
- `canvas` is the most stable path but still blocked by `BitmapData.draw`.

## Suggested next steps

1. Test in another browser (Safari and Firefox) using the same renderer URLs to confirm parity.
2. Pin a different Ruffle build (new nightly or stable) and re-run the matrix.
3. Track Ruffle issues for `BitmapData.draw` and TextLayout stability; retest when updates land.
4. Optional: pass `renderer` through `index.html` iframe for easier testing from the wrapper page.
