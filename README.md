# ediKing
### A client-side HTML/JS editor
Short link: [tfi.la/e](https://tfi.la/e)

For teaching p5.js: [tfi.la/p5](https://tfi.la/p5)  (p5.js template + disable LLM)

The panEdiKon gallery for monitoring students remotely: https://io.eyalgruss.com/ediking/panedikon (see [below](#pan))

### Keys and gestures
#### Global
- `Ctrl`+`Shift`+`F` - Show / hide editor (full screen)
- `Ctrl`+`Enter` - Run / stop generation (when LLM is available)
- `Ctrl`+`Shift`+`Enter` (or `Ctrl`+`Click` on the button) - Looping / no-loop generation (when LLM is available)
- `Ctrl`+`S` - Save HTML file (currently rendered version)
- `Alt`+`0`/`-`/`+` - Reset / decrease / increase editor font size

#### Editor text areas
- `Tab`, `Shift`+`Tab` - Indent / unindent at caret or multi-line for selection
- `Enter`, `Shift`+`Enter` - New line with current or coding-language-based extra indentation (`Shift` skips to line end beforehand)
- `Home`, `Shift`+`Home` - Move caret to indentation; if already there, move to logical line start (`Shift` for selection) 
- `End`, `Shift`+`End` - For wrapped lines, second press moves caret to logical line end (`Shift` for selection)
- `Ctrl`+`C` with no selection - Duplicate line
- `Ctrl`+`X` with no selection - Delete line
- `Ctrl`+`/` - Multi-line `//` comment / uncomment
- Drag and drop file(s) - Open and add HTML tags for CSS and JS, `.prompt` / `.poml` files loaded to prompt (as raw text)

### URL API (synced with GUI)
- `#...` - Code


- `?disp` - Hide editor
- `?disp=2` - Hide editor, no show/hide button, global keyboard shortcuts work
- `?disp=3` - Hide editor, no show/hide button, no keyboard shortcut


- [`?temp`](https://tfi.la/e?temp) - Empty
- [`?temp=html`](https://tfi.la/e?temp=html) - HTML template
- [`?temp=js`](https://tfi.la/e?temp=js) - JS template
- [`?temp=p5`](https://tfi.la/e?temp=p5) - [p5.js](https://p5js.org/) template
- [`?temp=q5`](https://tfi.la/e?temp=q5) - [q5.js](https://q5js.org/) template


- `?prompt=...` - Prompt (when LLM is available)
- `?loop` - Looping generation (when LLM is available)
- [`?noai`](https://tfi.la/e?noai) - Disable LLM (aka student mode)


- `?session=...` - Broadcast session number
- `?user=...` - Broadcast username
- `?group=...` - Broadcast group name
- `?key=...` - Broadcast key code


- `?pause` - Delay loading code into iframe until next edit (to allow fixing bad code)

### <a id="pan"></a>Broadcasting to The panEdiKon 👁️
- Uses gunDB for live many-to-one sharing
- All broadcasted content and meta-data are public while enabled
- Open the panEdiKon: https://io.eyalgruss.com/ediking/panedikon,
and manually add these params, replacing the `...` with your own details:
  - `?session=...` - Broadcast session number (optional)
  - `?group=...` - Broadcast group name
  - `?secret=...` - Broadcast secret (don't share)
- Use a short string as a secret for obfuscation.
You will get a (different) key to give your students.
Note that this just adds very basic obfuscation, and is not secure at all
- Click the username/timestamp to enter/exit full screen
- You can edit the code, but this is __not__ reflected back, and will be overwritten on remote update
- Note that while there are some protective measures in place, a rogue broadcast could hang the whole gallery.
Limit this risk by using unique session, group and secret/key

### iframe convenience features
- Prevent iframe from stealing focus on load
- Have iframe listen to the global keyboard shortcuts
- Add `target="_blank"` to `a` links of different origin, to allow opening from iframe
- Grab title, description and favicon from iframe
- Prevent top level navigation when file dropped on iframe
- Update code saved in the URL only after iframe has loaded, to make it easier to recover from infinite loops 
- Helper functions for output in iframe: `parent.clear()`, `parent.log()`, `parent.safeLog()` / `parent.safelog()` (infinite loop protection; but will also stop p5.js `draw()` unless you call `top.clear()`). If not embedded, instead of `parent` you can also use `top`

### Experimental Writer / Rewriter Chromium API integration
- Models and requirements:
  - Gemini Nano: Chrome desktop 138+, 22 GB disk space, and 4 GB GPU (https://developer.chrome.com/docs/ai/writer-api)
  - Phi-4-mini: Edge desktop 139+, 20 GB disk space, and 5.5 GB GPU (https://learn.microsoft.com/sr-latn-rs/microsoft-edge/web-platform/writing-assistance-apis)
- API explainer: https://github.com/webmachinelearning/writing-assistance-apis
- LLM patterns used here:
  1. __Write from scratch__: no code -> `write(fullcode_prompt + prompt)`
  2. __Full rewrite__: code + no/full selection -> `rewrite(code, {context: fullcode_prompt + prompt})`
  3. __Complete missing code__: code + selected* whitespace -> `code_before SAFE+ write(truncate_to_quota(prompt + 'CURRENT CODE:' + code_before + '<!--/* COMPLETE MISSING CODE HERE AND OUTPUT ONLY THIS PART */-->' + code_after)) SAFE+ code_after` (does not change surrounding code, model sees it and may duplicate it)
  4. __Partial rewrite__: code + selected* code -> `code_before SAFE+ rewrite(selected_code, {context: prompt}) SAFE+ code_after` (does not change surrounding code, but the model does not see it to prevent confusion)
  5. __Finish HTML__: for all the above, if not in/after loop mode, and the final code has no closing `</html>` tag, and there is either no selection or no code after the selection, then iterate using the __complete missing code__ pattern with `code_before = code`
  - __sharedContext__ = `'Never use external files or remote media! Address any comments marked by FIXME, and remove them.'` (always applied)
  - __fullcode_prompt__ = `'Output only a complete single-file HTML code (including CSS/JS inside). Popular JS libraries can be used from CDN.'`
  - __SAFE+__ = trailing single-line `//` JS comments are closed with newline before concatenation
  - __*__ Selection in code editor is not visible while editing the prompt or other elements are focused
- Known API issues:
  - Broken model download - https://issues.chromium.org/issues/427520275
  - Not enough disk space - https://issues.chromium.org/issues/427535092
  - Page reload may crash Chrome - https://issues.chromium.org/issues/428688095
  - Wrong model availability on Chrome launch - https://issues.chromium.org/issues/429183223
  - Invalid state error when loading model - https://issues.chromium.org/issues/441547039
  - Aborting during model download may crash Chrome
  - After aborting the next generation is delayed
- Future opportunities:
  - Tab completions
  - Inline protected code regions (e.g. for tests)
  - Automatic error capturing and prompting for explanations and automatic fixes
  - Prompt API and better model settings
  - Multimodal input of the rendered iframe screenshot
  - Parse `.prompt` / `.poml` files for system prompt and params
  - Allow LLM to generate its next prompt (can currently be emulated via generated code comments)

### Other things I may add
- Compress the code and prompt saved in the URL
- Local storage for saner local persistence, and default/custom templates
- Upload media files as data blobs
- Drag and drop HTML content and links to grab code
- Direct editing of the iframe (aka designMode) reflected in the code editor
- Use gunDB or Nostr for persistence, sharing and collaborative editing
- Syntax highlighting ([WIP](https://codepen.io/eyaler/pen/NPGOaJb)) and other code editor niceties
- Individual play/pause and remove/nullify/block controls in panEdiKon
- Browse next/prev in fullscreen in panEdiKon

### Fun
- Recursion (code = ediKing): https://tfi.la/e#recur (LLM is disabled to prevent browser crash)
- Alien translation airlock: https://tfi.la/e?loop&prompt=Communicate+with+the+aliens+using+text%2C+shapes+and+colors.+Change+the+prevoius+message+and+make+it+even+more+psychedelic%21#%3C!DOCTYPE%20html%3E%0A%3Chtml%20lang%3D%22en%22%3E%0A%3Cbody%3E%0A%3Ch1%3EMessage%20to%20the%20aliens...%3C%2Fh1%3E%0A%3C%2Fbody%3E%0A%3C%2Fhtml%3E (when LLM is available)