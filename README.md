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

### Things I may to add
- Traversable edit history (especially for looping generation)
- Direct editing of the iframe (aka designMode) reflected in the code editor
- Use GUN.js or Nostr for persistence and collaborative editing
- Syntax highlighting and other code editor niceties

### Experimental Writer / Rewriter Chrome API integration
- Requires Chrome desktop 138+, 22 GB disk space, and 4 GB GPU (see: https://developer.chrome.com/docs/ai/writer-api)
- API explainer: https://github.com/webmachinelearning/writing-assistance-apis
- LLM patterns used here:
  1. __Write from scratch__: no code -> `write(fullcode_prompt + prompt)`
  2. __Full rewrite__: code + no/full selection -> `rewrite(code, {context: fullcode_prompt + prompt})`
  3. __Complete missing code__: code + selected* whitespace -> `code_before SAFE+ write(truncate_to_quota(prompt + 'CURRENT CODE:' + code_before + '<!--COMPLETE MISSING CODE HERE AND OUTPUT ONLY THIS PART-->' + code_after)) SAFE+ code_after` (does not change surrounding code, model sees it and may duplicate it)
  4. __Partial rewrite__: code + selected* code -> `code_before SAFE+ rewrite(selected_code, {context: prompt}) SAFE+ code_after` (does not change surrounding code, but the model does not see it to prevent confusion)
  5. __Finish HTML__: for all the above, if the final code has no closing `</html>` tag, and there is either no selection or no code after the selection, then iterate using the __complete missing code__ pattern with `code_before = code`
  - __sharedContext__ = `'Never use external files or remote media! Address any comments marked by FIXME, and remove them.'` (always applied)
  - __fullcode_prompt__ = `'Output only a complete single-file HTML code (including CSS/JS inside). Popular JS libraries can be used from CDN.'`
  - __SAFE+__ = trailing single-line `//` JS comments are closed with newline before concatenation
  - __*__ Selection in code editor is not visible while editing the prompt or other elements are focused
- Known API issues:
  - Broken model download - https://issues.chromium.org/issues/427520275
  - Not enough disk space - https://issues.chromium.org/issues/427535092
  - Page reload may crash Chrome - https://issues.chromium.org/issues/428688095
  - Wrong model availability on Chrome launch - https://issues.chromium.org/issues/429183223
  - After aborting the next generation is delayed
- Future opportunities:
  - Tab completions
  - Inline protected code regions (e.g. tests)
  - Automatic error capturing
  - Prompt API
  - Multimodal input of the rendered iframe screenshot
  - Parse `.prompt` files for system prompt and params
  - Allow LLM to generate its next prompt (can currently be emulated via generated code comments)

### Fun
- Recursion: https://tfi.la/e#recur (LLM is disabled to prevent browser crash)
- Alien translation airlock: https://tfi.la/e?loop=1&prompt=Communicate+with+the+aliens+using+text%2C+shapes+and+colors.+Change+the+prevoius+message+and+make+it+even+more+psychedelic%21#%3C!DOCTYPE%20html%3E%0A%3Chtml%20lang%3D%22en%22%3E%0A%3Cbody%3E%0A%3Ch1%3EMessage%20to%20the%20aliens...%3C%2Fh1%3E%0A%3C%2Fbody%3E%0A%3C%2Fhtml%3E (when LLM is available)