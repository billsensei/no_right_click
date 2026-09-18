# quizaccess_contentlock

A Moodle 5.1 `quizaccess` subplugin that adds a **client-side content-protection
deterrent** to the Quiz activity.

## This is still in Beta, please use in testing environments only!

## What this is

When enabled on a quiz, the live attempt page (`mod/quiz/attempt.php`) will,
for students only:

1. Disable the right-click context menu.
2. Block copying and cutting content (Ctrl/Cmd+C, Ctrl/Cmd+X, and the browser's
   copy/cut events, however they are triggered).
3. Block pasting into quiz answer fields (Ctrl/Cmd+V and the paste event).
4. Prevent selecting/highlighting question text (`user-select: none` plus a
   `selectstart` handler), while leaving answer fields (`input`, `textarea`,
   `contenteditable`) fully selectable and editable as normal.

A small on-screen notice is shown (via `core/toast`) when an action is
blocked, rather than failing silently.

## What this is **not**

**This is a deterrent, not a security guarantee.** It is ordinary
client-side JavaScript and CSS, and cannot stop a determined user. In
particular, it does **not** protect against:

- Opening browser developer tools and reading the DOM/network responses.
- Disabling JavaScript before loading the page.
- Taking a screenshot or photo of the screen.
- Using external OCR software on a screenshot.
- Any other client-side bypass — anything running in the student's own
  browser is, by definition, under the student's control.

Do not present this plugin to students, staff or institutional stakeholders
as an anti-cheating or content-security control. It exists purely to raise
the (very low) bar against casual, opportunistic copying, alongside your
existing academic integrity policies and, where real lockdown is required,
something like [Safe Exam Browser](https://safeexambrowser.org/) (the
`quizaccess_seb` plugin). This plugin is independent of `quizaccess_seb` and
does not touch or duplicate its browser lockdown/config-key mechanisms — the
two can be enabled together or separately.

## Settings

### Per quiz ("Extra restrictions on attempts")

- **Enable content protection** — turns the deterrent on for this quiz's live
  attempt page. Off by default (unless changed by the site default below).
- **Also apply during attempt review** — off by default, and only shown once
  content protection is enabled. By default the deterrent only applies while
  a student is *attempting* the quiz, not while reviewing a finished attempt,
  so that teachers who want students to be able to copy their own submitted
  answers during review are not silently blocked. Turn this on to explicitly
  extend the same restrictions to the review page as well.

### Site-wide (Site administration > Plugins > Quiz access rules > Content protection)

- **Enable content protection by default** — controls the default value of
  the per-quiz checkbox for *newly created* quizzes only. Existing quizzes
  are unaffected.

## Who is restricted

Only users **without** the `mod/quiz:preview` capability (i.e. students) are
restricted. Teachers, managers and anyone else who can preview the quiz can
always right-click, copy, cut, paste and select text normally, whether they
are attempting, previewing or reviewing the quiz.

## Scope of the JavaScript

`amd/src/contentlock.js` binds its `contextmenu`, `copy`, `cut`, `paste`,
`selectstart` and `keydown` listeners to `#region-main` only — never the
whole `document`. The `keydown` listener only intercepts the specific
Ctrl/Cmd+C, X, V and A combinations; `X`/`A` inside a student's own answer
field are left alone so normal editing (cut, select-all-to-replace) still
works. Every other key, and focus handling, is left completely untouched so
that keyboard navigation, screen readers and other assistive technology, the
built-in calculator, drag-and-drop questions and the "flag question" control
all continue to work exactly as normal.

**Known gap:** DOM events do not cross into an `<iframe>`'s own document, and
`core/modal` dialogues are appended to `document.body` as siblings of
`#region-main`, not descendants of it. This means the deterrent currently
does **not** reach:

- The default rich-text (Atto/TinyMCE) editor used for Essay question
  answers, which renders inside an iframe — copy/paste there is unrestricted.
- Content inside pop-up dialogues used by some question types (e.g. certain
  drag-and-drop or equation-editor questions).

This is a real, tested coverage gap, not just the general "a determined user
can always bypass client-side JS" caveat above — right-clicking or pasting
into an Essay answer box today takes no special effort at all. If this
matters for your use case, treat it as a reason to lean on
`quizaccess_seb`/institutional policy for anything you actually need to rely
on, the same as any other bypass of a client-side deterrent.

## Data stored

A single table, `quizaccess_contentlock`, stores one row per quiz that has
either setting turned on (`quizid`, `enabled`, `protectreview`,
`timecreated`, `timemodified`). No personal user data is stored; the privacy
provider implements `null_provider`.

## CI status

This plugin has been machine-verified against a real Moodle 5.1.7 test site
(PHP 8.4, MariaDB 11.8, headless Chrome via chromedriver). All of
`moodle-plugin-ci`'s checks pass: `phplint`, `phpcs`, `phpmd`, `validate`,
`savepoints`, `mustache`, `grunt` (including a real `grunt amd` build of
`amd/build/contentlock.min.js` and its `.min.js.map`), `phpunit` (13 tests),
and `behat` (6 scenarios, 84 steps, including simulated right-click/copy/
cut/paste/select-text events dispatched in real Chrome).

## Files

```
mod/quiz/accessrule/contentlock/
├── version.php
├── rule.php                                  quizaccess_contentlock rule class
├── settings.php                              site-wide default setting
├── styles.css
├── db/install.xml                            quizaccess_contentlock table
├── amd/src/contentlock.js                    deterrent logic (ES module)
├── amd/build/contentlock.min.js              grunt-built, plus contentlock.min.js.map
├── classes/privacy/provider.php              null_provider
├── lang/en/quizaccess_contentlock.php
├── tests/rule_test.php                       PHPUnit
└── tests/behat/
    ├── behat_quizaccess_contentlock.php      custom steps (simulate blocked actions)
    └── contentlock.feature
```
# no_right_click
