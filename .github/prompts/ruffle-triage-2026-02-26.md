# Ruffle triage report (2026-02-26)

## Scope

- macOS 26.3
- Local HTTP server: `python3 -m http.server 8000`
- URLs tested in external browser:
  - `http://127.0.0.1:8000/app.html?renderer=canvas`
  - `http://127.0.0.1:8000/app.html?renderer=webgl`
  - `http://127.0.0.1:8000/app.html?renderer=wgpu-webgl`
  - `http://127.0.0.1:8000/housingprices/main.html?renderer=canvas`
  - `http://127.0.0.1:8000/housingprices/extension-test.html`
  - `http://127.0.0.1:8000/extension-test.html`
- Ruffle build: `0.2.0-nightly.2026.2.25`

## Current file state

- `app.html`
  - SWFObject gating relaxed (`swfVersionStr = "0.0.0"`, `xiSwfUrlStr = ""`).
  - Ruffle config uses URL param `renderer` with default `canvas`.
  - Console logs include renderer and `embedSWF` result.
- `index.html` and `index-pre-ga-removal-3-2022.html`
  - iframe points to `app.html?intro=true`.
- `housingprices/main.html` and `housingprices/default.html`
  - SWFObject gating relaxed (`swfVersionStr = "0.0.0"`, `xiSwfUrlStr = ""`).
  - Ruffle config uses URL param `renderer` with default `canvas`.
  - Console logs include renderer and `embedSWF` result.
- `housingprices/extension-test.html` and `extension-test.html`
  - Extension-first embed with web Ruffle fallback.

## What works

- Ruffle initializes and loads `Main.swf` over HTTP.
- `embedSWF` success and `hasRef` are true.
- Renderer can be switched via `?renderer=` in URL.

## What fails

- `canvas` and `webgl` renderers:
  - `Error dispatching event "enterFrame": RustError("Render backend does not support BitmapData.draw")`
- `wgpu-webgl` renderer:
  - `Error #1009: Cannot access a property or method of a null object reference` (TextLayout stack).
- Housing prices local (`housingprices/main.html?renderer=canvas`):
  - `Error dispatching event "enterFrame": RustError("Render backend does not support BitmapData.draw")`
- Global footprint extension test (`extension-test.html`):
  - `Error #1009: Cannot access a property or method of a null object reference` (TextLayout stack).
- Housing prices extension test (`housingprices/extension-test.html`) when navigating in-app:
  - `Uncaught RangeError: Maximum call stack size exceeded` followed by
    `panicked ... cannot recursively acquire mutex` and `Unable to lock Ruffle core`.
- Snack extension test (`snack/extension-test.html`) on load:
  - `Uncaught RangeError: Maximum call stack size exceeded` followed by
    `Unable to lock Ruffle core`.
- AudioContext warnings persist due to autoplay policy (not a blocker).

## Compatibility matrix (macOS, Chrome)

| Renderer | Boot | Loads SWF | Blocking error |
| --- | --- | --- | --- |
| canvas | yes | yes | BitmapData.draw unsupported |
| webgl | yes | yes | BitmapData.draw unsupported |
| wgpu-webgl | yes | yes | TextLayout Error #1009 |

## Housing prices (local)

- Local SWF matches hosted SWF (SHA-256 identical).
- `housingprices/main.html?renderer=canvas` hits `BitmapData.draw` unsupported.
- `housingprices/extension-test.html` boots via web Ruffle fallback and renders full-screen.
- Navigating within `housingprices/extension-test.html` can crash Ruffle with a mutex panic.

## Snack (local)

- `snack/extension-test.html` crashes in `wgpu-webgl` with a call stack overflow in URL parsing,
  then `Unable to lock Ruffle core`.

## Global footprint (extension test)

- `extension-test.html` boots via web Ruffle fallback and hits TextLayout `Error #1009` under `wgpu-webgl`.

## Conclusion

- Primary blockers are Ruffle feature gaps, not host/browser issues.
- `canvas` is the most stable path but still blocked by `BitmapData.draw`.
- Housing prices uses a similar embedding path but still hits `BitmapData.draw` locally.
- Global footprint still fails with TextLayout under the extension-style embed.

## Suggested next steps

1. Test in another browser (Safari and Firefox) using the same renderer URLs to confirm parity.
2. Pin a different Ruffle build (new nightly or stable) and re-run the matrix.
3. Track Ruffle issues for `BitmapData.draw` and TextLayout stability; retest when updates land.
4. Optional: pass `renderer` through `index.html` iframe for easier testing from the wrapper page.
