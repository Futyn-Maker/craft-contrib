# RFC-0014: Accessibility in every UI change

Related: storytold/craft-libs#437 (shared work), RFC-0005, RFC-0006,
RFC-0007.

## Summary

Make accessibility a requirement for every change to an app's interface, as
never-crash is for every change to its code. The craft apps draw their own
interface with egui. People who use a screen reader or braille display, a
magnifier, high contrast, large text, voice control or only a keyboard reach
it only through what each app provides: its AccessKit tree, its keyboard
focus and its colours.

This RFC sets the bar (WCAG 2.2 AA as applied to software), seven rules for
every UI change, a design step, a check of the accessibility tree next to
every screenshot check, a list of failures the apps have already produced,
two labels, builds for testers, and a priority rule. The appendix is the text
for each app's `AGENTS.md`.

Tests and agents can't replace a person using a screen reader. The aim is
that everything a test or an agent can check is checked, and that what only a
person can check is easy to find.

## Motivation

Accessibility already gets real attention from individual contributors.
AccessKit is on in every app. WordCraft describes its pages to screen readers
(storytold/wordcraft#488). Open PRs bring the ribbon
(storytold/wordcraft#2012) and formatting at the caret
(storytold/wordcraft#4860) to screen readers. EffectCraft is adding an
accessibility audit test (storytold/effectcraft#1335), and PdfCraft's Linux
accessibility is being fixed (storytold/pdfcraft#4612). About two dozen
screen-reader and keyboard issues have been filed. But nothing written asks
for any of it, so the result depends on who picks up which issue:

- **No rule.** No app's `AGENTS.md` asks for accessibility. Neither do the PR
  templates, the `CONTRIBUTING` files, the shared standards they point to, or
  RFC-0005, RFC-0006 and RFC-0007. Agents follow their instructions
  literally (RFC-0006), so what isn't written isn't done.
- **The check is visual.** `AGENTS.md` tells agents to take `ui.screenshot`
  and compare it with the reference app. Nothing tells them to read the
  accessibility tree, although PdfCraft's `ui.inspect` already returns it.
- **The automatic check covers names only.** egui_kittest fails a test when
  a visible input widget has no accessible name, which is why most controls
  are named. Nothing checks focus, announcements, editability or contrast.
  WordCraft's page passed its tree tests, yet a blind user found that NVDA
  doesn't echo typing: the page is a `Document` node, which AccessKit always
  reports as read-only (storytold/wordcraft#3174). KeyTips left focus in the
  page, so their letters typed into it (storytold/wordcraft#3171). No app
  uses a live region to announce anything, so Ctrl+B is silent; only
  craft-keymap's shortcut editor uses one.
- **The work can't be found.** No app repository has an accessibility label.
  storytold/wordcraft#2012 has been ready since 15 October without a
  screen-reader test, and a report from a JAWS and NVDA user
  (storytold/filmcraft#1074) has no reply yet.
- **The last step has no owner.** Agents are told not to drive the GUI with
  synthetic input on a shared machine, and PRs say so: "No real VoiceOver
  session was run (it needs a person's screen and keyboard)"
  (storytold/lightcraft#3216). No app's CI keeps a build of a PR, so a
  tester has to build it from source.
- **Progress doesn't show it.** WordCraft counts Accessibility as one area
  with weight 1 of 100, mostly Word's tools for making accessible documents.
  GridCraft's gap list says "no accessible grid/ribbon tree".

This is cheap to fix now. Later, every menu, dialog and drawn surface built
without real focus and semantics has to be reworked.

## Proposal

1. **The bar.** The interface meets WCAG 2.2 level AA as applied to software
   by W3C's WCAG2ICT, the approach EN 301 549 (chapter 11) takes for software
   that public bodies buy. The web builds meet WCAG 2.2 AA directly. This
   covers people who use screen readers or braille, magnifiers, high
   contrast, large text, voice control or only a keyboard. Assistive
   technology reaches the apps only through AccessKit (UI Automation on
   Windows, NSAccessibility on macOS, AT-SPI on Linux), so that is what we
   build and test.
2. **Seven rules for every UI change.** WCAG success criteria are in
   parentheses.
   - *Keyboard.* Every action works from the keyboard, in the order things
     appear, with no trap; freehand drawing is the exception WCAG allows. Esc
     closes a popup or dialog and puts focus back where it was. Alt or F10
     moves focus into the menu bar or ribbon, and F6 between regions, where
     the platform does so. Single-key shortcuts can be turned off or
     remapped. (2.1.1, 2.1.2, 2.1.4, 2.4.3)
   - *Real focus.* Keyboard navigation moves egui's focus (`request_focus`),
     never only a painted highlight, so screen readers and magnifiers follow
     it. Focus is visible in both themes and not hidden under other UI.
     (2.4.7, 2.4.11)
   - *Name, role, state.* Every control exposes its role, a translated name
     that starts with its visible label, and its state and value: checked,
     pressed, selected, expanded, disabled. Shortcuts and access keys are set
     on the node, in the current platform's form. (2.5.3, 4.1.2)
   - *Spoken feedback.* A change that happens away from focus, such as a
     toggle by shortcut, "Saved", a result count, an error or a finished
     export, is announced through a live region. Errors say in text what is
     wrong. Messages that close on their own can be read again. (2.2.1,
     3.3.1, 4.1.3)
   - *Drawn surfaces.* A page, canvas, grid, timeline or layer list publishes
     its content as AccessKit nodes: text with caret and selection, cells
     with their position, objects with their alt text, and the caret's place
     on screen for magnifiers. An editable surface is reported as editable.
     (1.1.1, 1.3.1, 4.1.2)
   - *Seeing.* Text has a contrast of at least 4.5:1, and controls and focus
     indicators at least 3:1, in light and dark themes. Colour is never the
     only signal. The app follows the system's high contrast, text size and
     reduced motion settings. (1.4.1, 1.4.3, 1.4.4, 1.4.11)
   - *Pointer.* Anything done by dragging can also be done without dragging,
     unless the drag is the point, as in freehand drawing. Targets are at
     least 24 by 24 points or have that much space around them. (2.5.7,
     2.5.8)
3. **Design before building.** Before writing a UI feature, write down in the
   issue or PR:
   - which standard pattern it is (menu bar, menu, tab list, toolbar, tree,
     grid, list, dialog, slider, canvas), and follow that pattern's keyboard
     model and roles from the WAI-ARIA Authoring Practices Guide and the
     platform's own guidelines;
   - how the reference app, and other apps of the same kind that
     screen-reader users rely on, expose it, observed from outside with the
     platform's inspector (Accessibility Insights for Windows, Accessibility
     Inspector on macOS, Accerciser on Linux), under the existing clean-room
     rules;
   - how a keyboard user, a screen-reader user and a magnifier user do the
     task.

   Record the observed behaviour in the app's parity docs, as WordCraft
   records typing in `docs/typing-parity.md`.
4. **Check the tree, not only the picture.**
   - A change that adds or changes a control, a dialog or a keyboard path
     adds or updates an egui_kittest test. The test drives it by keyboard and
     asserts on the accessibility tree: roles, names, states, where focus is
     after each step, and the announcement if there is one.
   - The built-in name check stays on. Turning it off
     (`with_accessibility_check(false)`) needs a comment saying why.
   - Whenever an agent takes `ui.screenshot`, it also reads the
     accessibility tree of the same window and checks it against point 2:
     through the control channel where it returns the tree, otherwise in a
     kittest test.
   - A test checks the contrast of new colours in both themes.
5. **Rule out the failures we already have.** Every UI PR checks for:
   - focus left in the document behind a menu, ribbon or KeyTips, so keys
     type into the page (storytold/wordcraft#3171);
   - an editable surface reported as read-only (storytold/wordcraft#3174);
   - a dialog that doesn't take focus, doesn't give it back, or leaves the
     controls behind it reachable (storytold/wordcraft#4269);
   - painted rows, tiles or handles with no node, or with the same name
     (storytold/soundcraft#651, storytold/craft-libs#410);
   - no visible focus (storytold/wordcraft#4495);
   - a state change nobody hears, such as Bold by shortcut;
   - another platform's text in a name, such as "Paste (⌘V)" on Windows
     (storytold/wordcraft#3164);
   - an action that only works by dragging (storytold/filmcraft#1074).

   Failures found later in review or by testers are added to this list.
6. **Say what was checked.** A UI PR's description has an Accessibility
   section. It says what the tests and the tree check cover, and what only a
   person with assistive technology can confirm. Tree tests alone never
   support "works with screen readers". When only a person can confirm the
   result, the section says so, and the triage bot or a triager adds
   `needs screen reader test`.
7. **Labels and builds for testers.**
   - `accessibility` in every repository, with the description craft-libs
     already uses: "Barrier affecting people with disabilities". The triage
     bot applies it as it applies `ui`.
   - `needs screen reader test` on a PR whose result only a person using a
     screen reader, braille display or magnifier can confirm. The PR may
     merge on headless evidence. The label stays until someone tests a build
     that contains the change and reports on the PR what they heard, with
     the screen reader, its version and the platform. If the test fails, the
     issue is reopened.
   - For PRs with that label, CI keeps a Windows build of the PR, and macOS
     and Linux builds where the gate makes them, through
     craft-common-actions.
   - People who test with assistive technology can get GitHub's Triage role
     to set and clear these labels.
8. **Priority** (adds to RFC-0007). A barrier that stops a keyboard or
   screen-reader user from a core task (open, type, edit, save, reach the
   menus or ribbon, use a dialog) is `priority: high`; if it can cost them
   their work, `priority: critical`. A barrier someone met while using
   assistive technology is a reproduced failure, not a code-reading finding.
9. **Measure it.** Each app's progress docs say whether its core tasks can be
   done with a screen reader, with only a keyboard, at 200% text size and in
   high contrast, as part of its readiness. Tools for making accessible
   documents, such as alt text and the accessibility checker, stay a feature
   area.
10. **Where it lives.** The appendix goes into every app's `AGENTS.md` (and
    `CLAUDE.md` where it is a copy), and into the shared section if RFC-0006
    is accepted. The same text goes into craftrules as
    `standards/accessibility.md`, next to never-crash. Shared code (a
    live-region helper, real focus in craft-menu, the tree in every control
    channel, test helpers, PR builds) is tracked in storytold/craft-libs#437.

## Alternatives

- **Leave it to each app.** This is the current state: real work, but
  uneven, and the same failures repeat from app to app.
- **An accessibility pass later.** Every surface built without focus and
  semantics would have to be reworked, and agents would keep building new
  ones meanwhile.
- **Automated checks only.** They catch missing names, not whether a task
  can be done. storytold/wordcraft#3174 passed them.
- **No merge without a screen-reader test.** Testers are few and review is
  already the bottleneck (RFC-0008). The label keeps the work findable
  without blocking the queue.

## Open questions

- Should the rules and the failure list live in `docs/accessibility.md`
  here, as RFC-0013 proposes for living documents, so the list can grow
  without a new RFC?
- Which platform gets PR builds first? Windows has the most screen-reader
  users (NVDA, JAWS, Narrator). Unsigned macOS builds need extra steps to
  open.
- Is 24 by 24 points right for dense toolbars, or should apps rely on
  WCAG's spacing exception there?

## Appendix: text for `AGENTS.md`

### Accessibility

People use this app with screen readers and braille displays, magnifiers,
high contrast, large text, voice control and the keyboard alone. They reach
it only through its AccessKit tree, its keyboard focus and its colours. A UI
change isn't done until it works for them. Standard: craft-contrib RFC-0014
(WCAG 2.2 AA as applied to software).

- **Design first.** Name the pattern (menu bar, menu, tab list, toolbar,
  tree, grid, list, dialog, slider, canvas) and follow its keyboard model
  and roles from the WAI-ARIA Authoring Practices Guide and the platform's
  guidelines. Observe from outside how the reference app exposes it, with
  Accessibility Insights, Accessibility Inspector or Accerciser.
- **Keyboard.** Every action works from the keyboard, in visual order, with
  no trap. Esc closes and returns focus. Alt/F10 and F6 behave as on the
  platform. Move egui's focus (`request_focus`), never only a highlight.
- **Semantics.** Every control has a role, a translated name that starts
  with its visible label, and its state and value. Shortcuts and access keys
  are set on the node, in the current platform's form. Drawn surfaces
  publish nodes for their content, with caret, selection and correct
  editability.
- **Feedback.** Announce changes that happen away from focus through a live
  region.
- **Seeing.** Contrast at least 4.5:1 for text and 3:1 for controls and
  focus, in both themes. Never colour alone. Follow the system's high
  contrast, text size and reduced motion settings.
- **Pointer.** Anything done by dragging can be done without dragging.
- **Check the tree, not only the screenshot.** Every new or changed
  control, dialog or keyboard path gets a kittest test that drives it by
  keyboard and asserts roles, names, states, focus and announcements. Keep
  `with_accessibility_check` on. When you take `ui.screenshot`, also read
  the accessibility tree and check it.
- **Rule out known failures:** focus left in the document behind a menu,
  ribbon or KeyTips; editable surfaces reported read-only; dialogs that
  don't take or return focus, or leave controls behind them reachable;
  painted rows with no node; identical names; another platform's shortcut
  text in a name; silent state changes; drag-only actions.
- **Be honest in the PR.** Say what the tests and the tree cover and what
  only a person can confirm. Never claim "works with screen readers" from
  tree tests; ask for `needs screen reader test` instead.
