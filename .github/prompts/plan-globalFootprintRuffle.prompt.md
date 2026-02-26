## Plan: Restore Local Ruffle Playback

Goal is near-full parity on a local static server, using a compatibility-safe Ruffle setup because your environment is “both / not sure.” The main issue today is that the wrapper still points to the archived remote app, while local Flash boot depends on SWFObject version gating that can fail unless Ruffle hooks correctly. This plan first restores deterministic local boot, then narrows runtime incompatibilities (Flex/OSMF/services) with concrete checkpoints so we can separate “embed problem” from “SWF feature/support problem.”

**Steps**
1. Establish local baseline entry by treating [app.html](app.html#L17-L56) as the primary startup path (not the wrapper iframe in [index.html](index.html#L117)).
2. Normalize host/runtime assumptions for a simple HTTP server: ensure SWF/SWZ/history assets resolve from project root and verify IIS-only behavior in [web.config](web.config#L1-L13) is not being implicitly relied on.
3. Resolve SWFObject gating path in [swfobject.js](swfobject.js#L647-L704) and embed config in [app.html](app.html#L39-L56) so `Main.swf` is inserted reliably under current Ruffle injection mode.
4. Re-point wrapper for local troubleshooting by updating iframe targets in [index.html](index.html#L117) and, if needed, [index-pre-ga-removal-3-2022.html](index-pre-ga-removal-3-2022.html#L125) to local [app.html](app.html), keeping external analytics/share scripts non-blocking.
5. Validate BrowserHistory/ExternalInterface expectations (`browserURLChange`) across [app.html](app.html#L32-L33) and [history/history.js](history/history.js#L95-L105), [history/history.js](history/history.js#L633-L649).
6. Capture runtime dependency gaps from `Main.swf` execution (network calls, missing APIs, OSMF/Flex behaviors) and classify each as: host config issue, URL remap issue, or Ruffle compatibility limitation.
7. Apply parity hardening only where needed: local URL substitution strategy, optional CORS/MIME tweaks for chosen server, and fallback behavior for unsupported Flash features.

**Verification**
- Start local server and open local [app.html](app.html), then local [index.html](index.html).
- In DevTools, confirm successful fetches for `Main.swf`, `.swz`, and `history/history.js`; no 404/blocked MIME/CORS on required assets.
- Confirm DOM contains embedded SWF container (not fallback-only state from SWFObject gate).
- Exercise navigation/state changes to verify history bridge callbacks.
- Record remaining console/network errors and map to compatibility vs configuration.

**Decisions**
- Chose near-full parity over “boot only,” so plan includes service/runtime triage beyond initial embed.
- Chose simple local server over IIS first; only add IIS-specific behavior if logs show MIME/CORS gaps.
- Chose compatibility-safe Ruffle path because browser setup is uncertain, avoiding assumptions about extension-only injection.
