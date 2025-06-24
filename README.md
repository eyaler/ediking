# ediKing
A client-side HTML editor

### Shortcuts
- Ctrl-Shift-F - Toggle code editor (full screen)
- Ctrl-Enter - Run/stop prompt (when LLM is available)
- Ctrl-S - Download HTML
- Tab, Shift-Tab, Enter - Indentation at cursor location (not line-level but preserves undo stack)
- Drag and drop (multiple) files - upload and add HTML tags for CSS and JS

### URL API
- Code is saved in #hash
- ?disp=1 - hide editor
- ?disp=2 - hide editor, no code toggle button (keyboard shortcut still works)
- ?disp=3 - hide editor, no code toggle button, no keyboard shortcut

### Convenience features
- Prevents iframe from stealing focus
- Adds `target="_blank"` to `a` links of different origin, to allow opening from iframe
- Grab title and favicon from code

### Experimental Writer / Rewriter Chrome API support
- Requires Chrome Desktop 138+, 22 GB disk space, and 4 GB GPU (see: https://developer.chrome.com/docs/ai/writer-api)
- Uses Write API when there is a prompt and no code, otherwise uses Rewrite API
- My implementation is very basic and with the small Gemini Nano model - results are not good
- Known model download issues: https://issues.chromium.org/issues/427520275,
https://issues.chromium.org/issues/427535092
