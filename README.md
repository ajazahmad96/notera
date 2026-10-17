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
- `styles.css` — design tokens, sky + frosted-glass design system, responsive mobile-first layout, light/dark themes
- `app.js` — IndexedDB persistence, rendering/navigation, content management, editor, search, history, import/export
- `manifest.webmanifest` — PWA metadata
- `sw.js` — caches the static app shell, not note content
- `icon.svg` — app icon

## Design system (v0.2 redesign; see v0.3 notes below)

Soft sky + frosted glass, white bottom sheets, black pills, thin-line icons, light-weight type.

- **Sky:** a fixed pale blue-teal gradient with soft cloud washes (`body::before`). Dark mode uses a deep slate-teal night sky.
- **Glass:** translucent white cards (`--glass`, `--glass-strong`) with a 1px white border, backdrop blur, and a soft blue-grey shadow. Radii: 26px cards, 30px sheets/paper/modals, pills are fully round.
- **Sheet:** Home puts progress and subjects on a white bottom sheet with a grabber; modals are bottom sheets too.
- **Buttons:** primary = black pill (inverts to white in dark mode); secondary = white glass pill; round 40px icon buttons.
- **Type:** system stack (`Inter` → `SF Pro` → `Google Sans` → `Roboto` → system). Large headings are weight 400, big numbers weight 300 with a dimmed unit, small cool-grey labels. No web fonts are loaded, so it stays fully offline.
- **Icons:** 1.5px round-cap line icons; the active tab switches to a filled black glyph.
- **Progress:** hatched tick bars (the “bar chart” look) instead of solid fills.
- **Status colours:** chapter status shows as small coloured text on the right of each chapter row.

All colours, radii and shadows are CSS variables at the top of `styles.css` (`:root` and `body.dark`).

## v0.3 refinement

- **Home:** Continue Reading, the Offline Ready badge and the dashboard Search/Library/Plus shortcuts are gone. The logo and name appear on Home only.
- **Progress rings:** large *Overall progress* (weighted: Pending 0 · Notes Completed 33 · Learning & Revision 67 · Exam Ready 100, averaged over all chapters) with *Notes completed* and *Learning & revision* beside it (exact-status counts ÷ total chapters). Empty data shows 0%, never NaN. 100% is only shown when every chapter is Exam Ready.
- **Subject cards** (Home and Library share one component): own card, line icon (picked from the subject name), thin progress bar and percentage. Each subject stores an `accent` (one of 8 palette colours, assigned to the least-used one) so colours are stable.
- **Chapters:** name + chapter number, sorted by number; duplicate names/numbers are blocked with inline messages; rename keeps (and can edit) the number.
- **Compact actions:** one round + per list; Rename/Delete live in the ⋯ menus; status is a small labelled selector; the open note has one ⋯ menu (Edit Note, View History).
- **Read mode:** no card, white page, no duplicated title/metadata.
- **Flat empty states** on Subject, Chapter, Search and Bookmarks.
- **Safeguards:** duplicate subject/chapter names (case/space-insensitive), duplicate note titles warn ("Create anyway"), empty names rejected, Enter submits forms, rapid double-taps ignored.

### Data compatibility
Storage keys and structures are unchanged. On first load a small migration adds `accent` to subjects and `serial` to chapters (existing alphabetical order is kept as 1, 2, 3…). Imports are migrated too. Nothing is deleted or overwritten. `sw.js` cache bumped to `notera-shell-v3`.

## Implemented

- Home greeting with editable name, syllabus progress rings, and subject cards
- Subjects → chapters → notes hierarchy; create, rename, and delete with confirmation
- Four chapter preparation statuses with immediate dashboard recalculation
- Separate read and edit modes; formatting toolbar for headings, bold, italic, underline, lists, highlights, callouts, tables, and image insertion/captions
- HTML paste sanitization and plain-text paste fallback
- Autosave to IndexedDB, save-state feedback, undo/redo commands where supported by the browser, and up to 20 locally retained note versions
- Local search across subjects, chapters, note titles, and note text; recent searches
- Original-note bookmarks, settings, persistent light/dark mode, JSON export/import with merge/replace confirmation
- Responsive bottom navigation, empty states, and installable PWA shell where the host supports it

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

**Verification performed here:** `node --check` on `app.js`, plus an automated end-to-end run in headless Chromium with an Android-sized viewport (390×844, touch). It drove the real UI through 40 checks: creating subjects/chapters/notes, rich-text editing (heading, bold, list, definition, highlight, table, image), autosave, version history and restore, bookmarks, status changes and Home metrics, search and recents, dark mode persistence across reload, JSON export, import (replace), rename and delete. All passed, with no JavaScript errors. Layouts were also checked for horizontal overflow at 360, 390, 412 and 768 px wide, including very long subject, chapter and note names. Not tested: a physical Android device, installed-PWA behaviour, and the real Android system font (screenshots used a fallback font).

## v0.4 — cumulative preparation checkboxes
- Each chapter has four stage checkboxes (Pending Notes, Notes Completed, Learning & Revision, Exam Ready) with dependency rules. Stored as `chapter.stages={notes,learning,exam}`; `status` is kept in sync (highest stage) for backward compatibility.
- Progress uses the highest completed stage only: 0 / 25 / 60 / 100, averaged and rounded to the nearest whole number. Subject bars use the same rule per subject.
- Notes Completed and Learning & Revision rings are cumulative counts ÷ total chapters.
- Existing chapters are migrated from `status` on load (and on import). `sw.js` cache bumped to `notera-shell-v4`.

## v0.5 — UI fixes
- Home: the white bottom "sheet" (rounded top, shadow, grabber) is now transparent, so the greeting, rings and subjects sit on one continuous sky screen.
- Chapter: the four checkboxes live in a compact "Status" control (shows the current stage). Tap to open a popover; close with Done, outside tap, Esc or the control itself. Progress rules and storage are unchanged. `sw.js` cache bumped to `notera-shell-v5`.
