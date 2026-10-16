# Notera — interactive frontend prototype

A mobile-first, local-first study-notes prototype built with **HTML5, CSS3, and vanilla JavaScript**. No framework, backend, login, or cloud sync is used.

## Run it

### Recommended: serve locally

PWA service workers require a secure context (`localhost` counts). From the extracted `notera` folder, run one of:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000` in your browser. On Android, host the folder on a static HTTPS host such as GitHub Pages or Vercel to test installation and offline app-shell caching.

### Quick open

You can open `index.html` directly for basic UI testing, but browser storage/PWA behaviour can differ for `file://` URLs. Use a local server for a fair test.

## Files

- `index.html` — semantic app shell and five-tab navigation
- `styles.css` — design tokens, responsive mobile-first layout, light/dark themes
- `app.js` — IndexedDB persistence, rendering/navigation, content management, editor, search, history, import/export
- `manifest.webmanifest` — PWA metadata
- `sw.js` — caches the static app shell, not note content
- `icon.svg` — app icon

## Implemented

- Home greeting with editable name, continue-reading note, live chapter progress metrics, and subject cards
- Subjects → chapters → notes hierarchy; create, rename, and delete with confirmation
- Four chapter preparation statuses with immediate dashboard recalculation
- Separate read and edit modes; formatting toolbar for headings, bold, italic, underline, lists, highlights, callouts, tables, and image insertion/captions
- HTML paste sanitization and plain-text paste fallback
- Autosave to IndexedDB, save-state feedback, undo/redo commands where supported by the browser, and up to 20 locally retained note versions
- Local search across subjects, chapters, note titles, and note text; recent searches
- Original-note bookmarks, settings, persistent light/dark mode, JSON export/import with merge/replace confirmation
- Responsive bottom navigation, empty states, offline indicator, and installable PWA shell where the host supports it

## Known limitations

- The prototype uses the browser's native `contenteditable` and `document.execCommand` editing commands. Those commands are deprecated by browser standards, but remain available in many mobile browsers; formatting behaviour can vary. For production-grade editing, replace the editor adapter with a maintained editor such as Tiptap/ProseMirror or Lexical after a deliberate dependency decision.
- Images are stored as data URLs inside note HTML in IndexedDB, limited to 4 MB per selected image. This keeps the prototype dependency-free, but a separate image-blob object store would scale better for a production app.
- Table editing is practical but basic: insert a starter table and edit its cells directly; there is no dedicated row/column management UI yet.
- Version history is created when finishing an editing session after a content change, and is capped at 20 versions per note.
- The service worker caches only the static app shell. User content remains in browser IndexedDB and is not copied into the service-worker cache. Browser storage can still be cleared by the user or evicted by the browser.
- Browser-level testing on physical Android devices cannot be performed from this generation environment. Please use the checklist below on the target device.

## Test checklist

- [ ] Open all five navigation tabs without a full reload
- [ ] Create, rename, and delete a subject; confirm deletion prompt
- [ ] Create a chapter, change each preparation status, and verify Home metrics
- [ ] Create multiple notes in one chapter, rename and delete a note
- [ ] Format text, paste rich HTML and plain text, add an image and caption
- [ ] Reload the page and verify notes and image content persist
- [ ] Bookmark/unbookmark; rename a bookmarked note and verify the title updates
- [ ] Search subject/chapter/note titles and text; rerun and clear recent searches
- [ ] Switch theme and reload to verify persistence
- [ ] Edit a note, finish editing, open version history, restore an older version
- [ ] Export JSON; import it using both merge and replace modes
- [ ] Test empty states and offline app-shell behaviour on an HTTPS host

**Verification performed here:** JavaScript syntax checked with `node --check`; files packaged into a ZIP. Interactive browser/device tests have not been run in this environment, so the checklist is provided as unverified manual QA rather than a claim that every runtime behaviour has passed.
