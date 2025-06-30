# ediKing
### A client-side HTML editor
Short link: [tfi.la/e](https://tfi.la/e)

### Shortcuts
- `Ctrl+Shift+F` - Show/hide editor (full screen)
- `Ctrl+Enter` - Run/stop generation (when LLM is available)
- `Ctrl+Shift+Enter` (or `Ctrl+Click` on the button) - Looping/no-loop generation (when LLM is available)
- `Ctrl+S` - Save HTML file (currently rendered version)
- `Tab`, `Shift+Tab`, `Enter` - Indentation at cursor location (not line-level but preserves undo stack)
- Drag and drop file(s) over editor - Open and add HTML tags for CSS and JS, `.prompt` files loaded to prompt (as raw text)

### URL API (auto-synced)
- `#...` - Code
- `?disp=1` - Hide editor
- `?disp=2` - Hide editor, no show/hide button (keyboard shortcut still works)
- `?disp=3` - Hide editor, no show/hide button, no keyboard shortcut
- `?prompt=...` - Prompt (when LLM is available)
- `?loop=1` - Looping generation (when LLM is available)
- `?noai=1` - Disable LLM (aka student mode)

### iframe convenience features
- Prevent iframe from stealing focus on load
- Have iframe listen to keyboard shortcut to show editor
- Add `target="_blank"` to `a` links of different origin, to allow opening from iframe
- Grab title and favicon from iframe

### Experimental Writer / Rewriter Chrome API integration
- Requires Chrome Desktop 138+, 22 GB disk space, and 4 GB GPU (see: https://developer.chrome.com/docs/ai/writer-api)
- LLM patterns:
  1. __No code__ -> `write(fullcode_prompt + prompt)`
  2. __Code + no/full selection__ -> `rewrite(code, {context: fullcode_prompt + prompt})`
  3. __Code + selected* whitespace__ -> `code_before SAFE+ write(truncate_to_quota(prompt + 'CURRENT CODE:' + code_before + '<!--COMPLETE MISSING CODE HERE AND OUTPUT ONLY THIS PART-->' + code_after)) SAFE+ code_after`
  4. __Code + selected* code__ -> `code_before SAFE+ rewrite(selected_code, {context: prompt}) SAFE+ code_after`
  5. For all the above - if the final code has no closing `</html>` tag, and there is either no selection or no code after the selection, then iterate using pattern iii with `code_before = code`
  - __sharedContext__ = `'Address any comment marked by FIXME, and remove them'` (always applied)
  - __fullcode_prompt__ = `'Output only a complete single-file HTML code (including CSS/JS inside). JS libraries can be used from CDN. NO external files!'`
  - __SAFE+__ = Trailing single-line `//` JS comments are closed with newline before concatenation
  - __*__ Selection in code editor is not visible while editing the prompt or other elements are focused
- Known API issues:
  - Broken model download - https://issues.chromium.org/issues/427520275
  - Not enough disk space - https://issues.chromium.org/issues/427535092
  - After aborting the next generation is delayed
- Future opportunities:
  - Code completion
  - Inline protected code regions (e.g. tests)
  - Automatic error capturing
  - Multimodal input of the rendered iframe screenshot
  - Parse `.prompt` files for system prompt and params
  - Allow LLM to generate its next prompt (can currently be emulated via generated code comments)

### Fun
- Recursion: [tfi.la/e#recur](https://tfi.la/e/#recur)
- Alien translation airlock: https://tfi.la/e?loop=1&prompt=communicate+with+the+aliens+using+text%2C+emoji+and+colors#%3C!DOCTYPE%20html%3E%0A%3Chtml%20lang%3D%22en%22%3E%0A%3Cbody%3E%0A%3Ch1%3EAlien%20message...%3C%2Fh1%3E%0A%3C%2Fbody%3E%0A%3C%2Fhtml%3E (when LLM is available)