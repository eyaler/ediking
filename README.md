# ediKing
### A client-side HTML editor
Short link: [tfi.la/e](https://tfi.la/e)

### Shortcuts
- Ctrl-Shift-F - Show/hide editor (full screen)
- Ctrl-Enter - Run/stop generation (when LLM is available)
- Ctrl-S - Save as HTML file
- Tab, Shift-Tab, Enter - Indentation at cursor location (not line-level but preserves undo stack)
- Drag and drop file(s) - Open and add HTML tags for CSS and JS

### URL API
- Code is auto-saved in #hash
- ?disp=1 - hide editor
- ?disp=2 - hide editor, no show/hide button (keyboard shortcut still works)
- ?disp=3 - hide editor, no show/hide button, no keyboard shortcut

### iframe convenience features
- Prevents iframe from stealing focus
- Adds `target="_blank"` to `a` links of different origin, to allow opening from iframe
- Grab title and favicon from iframe

### Experimental Writer / Rewriter Chrome API integration
- Requires Chrome Desktop 138+, 22 GB disk space, and 4 GB GPU (see: https://developer.chrome.com/docs/ai/writer-api)
- LLM patterns:
  - sharedContext = 'Make sure to address comments marked: FIXME, and edit to comments to show: FIXED'
  - No code -> write(fullcode_prompt* + prompt)
  - Code + no selection -> rewrite(code, {context: fullcode_prompt* + prompt})
  - Code + selected** whitespace -> code_before + write(code_before + '[COMPLETE MISSING CODE HERE - ONLY OUTPUT THIS PART]'*** + code_after + prompt) + code_after
  - Code + selected** code -> code_before + rewrite(selected_code, {context: prompt****}) + code_after
  - \* fullcode_prompt = 'Output only a complete single-file HTML code (including CSS/JS inside). JS libraries can be used from CDN. NO external files!'
  - \** Selection is not visible when editing the prompt
  - \*** Sometimes model duplicates the code outside anyway
  - \**** Context does not include surrounding code as that seems to confuse it
- My implementation is very basic and with the small Gemini Nano model - results are not good
- Known model download issues:
  - https://issues.chromium.org/issues/427520275
  - https://issues.chromium.org/issues/427535092
- Future opportunities:
  - Code completion
  - Inline protected code regions (e.g. tests)
  - Automatic error capturing
  - Multimodal input of the rendered iframe screenshot
  - Autopilot mode - mutate the code repeatedly with self-prompting

### Fun
- Recursion: [tfi.la/e/#recur](https://tfi.la/e/#recur)