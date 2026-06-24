---
title: "refactor: Convert Sideload Spanish fork to Sideload Turkish"
type: refactor
date: 2026-06-24
depth: lightweight
status: ready
---

# refactor: Convert Sideload Spanish fork to Sideload Turkish

## Summary

This repo (`sideloadturkish.com`) is a verbatim fork of the Sideload Spanish marketing
site and still carries Spanish branding, copy, demo translations, and external resource
slugs throughout. The one change: re-target everything from Spanish to Turkish. Four
tracked files carry Spanish content — `index.html`, `privacy.html`, `sync-success.html`,
`CNAME`. `styles.css` is language-neutral and stays untouched.

Two confirmed forks shape the work:
- **Drop grammatical gender.** Turkish nouns have no gender. The Spanish demo carries
  `data-gender` attributes and a masculine/feminine tooltip block — both are removed, not
  ported.
- **Rewrite external slugs to Turkish parallels.** Domain, Firefox add-on slug, Buttondown
  endpoint, and GitHub repo all move to their `…turkish` equivalents (may 404 until those
  resources are created — accepted).

---

## Problem Frame

A fork was taken from the Spanish site and the working directory renamed to
`sideloadturkish.com`, but no content was localized. Every user-visible string, the demo
widget, the canonical/domain config, and all outbound links still say Spanish. Goal:
produce a clean Turkish-branded static site with no residual Spanish references.

This is a mechanical localization refactor, not a behavior change — with one deliberate
behavioral simplification (gender removal).

---

## Requirements

- **R1** — All user-visible "Spanish" / "español" branding becomes Turkish / Türkçe.
- **R2** — Hero demo words show correct Turkish translations; CEFR tier metadata (A1–C1)
  is preserved (CEFR applies to Turkish).
- **R3** — The gender feature (`data-gender`, masculine/feminine tooltip line) is removed
  entirely.
- **R4** — Domain config (`CNAME`, `<link rel="canonical">`) points to
  `sideloadturkish.com`.
- **R5** — All external links retarget to Turkish slugs: Firefox add-on
  `sideload-turkish`, Buttondown `sideloadturkish`, GitHub `meowmeowco/sideloadturkish`.
- **R6** — No residual "spanish" / "español" / `data-es` token remains in any tracked file
  (`styles.css` excepted only because it has none).

---

## Key Technical Decisions

- **KTD1 — Rename `data-es` → `data-tr`.** The demo widget stores the target-language word
  in a `data-es` attribute and the tooltip JS reads `target.dataset.es`. Rename both the
  attribute and the dataset read to `data-tr` / `dataset.tr` so the markup is honest. Pure
  rename, no behavior change.
- **KTD2 — Remove the gender code path, not just the data.** Per the confirmed fork, delete
  the `data-gender` attribute on the demo word AND the `if (gender) { … }` block plus the
  `gender` variable read in the tooltip script. Leaving dead code would invite a future
  contributor to re-add gender, which is wrong for Turkish.
- **KTD3 — Turkish demo translations.** English original → Turkish: `Spanish`→`Türkçe`,
  `words`→`kelimeler`, `effort`→`çaba`, `web`→`web`. Tier values stay as-is. Türkçe carries
  Turkish-specific characters (ç, ü, ş, ğ, ı, ö) — emit as UTF-8 (the file is
  `<meta charset="UTF-8">`), not HTML entities, matching how the rest of the prose reads.
- **KTD4 — External slugs may 404 until provisioned.** Rewriting links to `…turkish`
  parallels is correct branding even though the add-on / Buttondown / GitHub targets may
  not exist yet. This is the user's accepted trade-off; no placeholder hedging in copy.

---

## Implementation Units

### U1. index.html — branding, head metadata, copy, external links

**Goal:** Retarget all non-demo Spanish strings, head metadata, and outbound links in the
homepage.

**Requirements:** R1, R4, R5

**Dependencies:** none

**Files:** `index.html`

**Approach:** Sweep the static markup (everything except the demo `<span class="sl-word">`
elements and the `<script>`, which U2 owns):
- `<title>` and `<meta name="description">` — "Spanish" → "Turkish", reword the description
  ("replaces English words with Turkish").
- `<link rel="canonical">` → `https://sideloadturkish.com`.
- Nav logo: `sideload<span>spanish</span>` → `sideload<span>turkish</span>`.
- Hero `<h1>`/`<p>` prose around the demo spans — "Learn Türkçe while you browse", "pick up
  Türkçe along the way" (translations of the demo words themselves are U2).
- Firefox add-on link `addons.mozilla.org/.../sideload-spanish/` → `…/sideload-turkish/`.
- Buttondown form `action` → `https://buttondown.com/api/emails/embed-subscribe/sideloadturkish`.
- Footer GitHub source link → `https://github.com/meowmeowco/sideloadturkish`.
- Any remaining "Spanish" in section copy (e.g. install/pricing/privacy section text) →
  "Turkish".

**Patterns to follow:** Existing copy tone — concise, no marketing fluff. Mirror the
current sentence structure; only swap the language noun.

**Test scenarios:** Test expectation: none — static content/attribute edits, no behavioral
change. Verified by U-wide grep in Verification.

**Verification:** `grep -niE 'spanish|español|espa&ntilde;ol' index.html` returns only demo
`data-original="Turkish"` style hits owned by U2 (ideally zero); canonical, form action,
add-on, and GitHub URLs all read `…turkish`.

---

### U2. index.html — interactive demo (Turkish translations, gender removal, data-tr)

**Goal:** Localize the hero demo widget to Turkish, rename the data attribute, and remove
the gender feature end to end.

**Requirements:** R2, R3, R6

**Dependencies:** U1 (same file; sequence after branding sweep to avoid edit overlap)

**Files:** `index.html`

**Approach:**
- For each `<span class="sl-word">`: set `data-original` to the English word (unchanged),
  rename `data-es="…"` → `data-tr="…"` with the Turkish translation per KTD3, and set the
  visible inner text to the Turkish word.
- Remove the `data-gender="m"` attribute from the second "Spanish/Türkçe" demo span.
- In the `<script>` tooltip code: delete the `const gender = target.dataset.gender;` read,
  delete the entire `if (gender) { … }` block that appends the masculine/feminine line, and
  change `const es = target.dataset.es;` → `const tr = target.dataset.tr;` plus the
  `transEl.textContent = '→ ' + es;` reference → `tr`.
- Keep `TIER_LABELS`, positioning logic, and "Click to mark as known" untouched.

**Execution note:** This unit changes runtime behavior (gender line removed). Verify the
tooltip renders correctly in a browser, not just by reading the diff.

**Technical design (directional):** tooltip object after edit shows three lines —
`original` → `→ tr` → tier label → action line; no gender line between translation and
tier.

**Patterns to follow:** The tooltip JS already builds each line as a `div` appended to
`tip`; removing the gender block means deleting one such builder, leaving the others' order
intact.

**Test scenarios (manual, no test runner in repo):**
- Happy path: hover a demo word → tooltip shows the English original, `→ <turkish word>`,
  the correct CEFR tier label, and "Click to mark as known". (no gender line)
- Edge: hover the formerly-gendered word (2nd Türkçe span) → no masculine/feminine line
  appears; layout has no empty gap.
- Regression: tooltip still positions above the word and clamps to viewport edges as before.
- Token check: no `data-es`, `dataset.es`, `data-gender`, or `dataset.gender` remains in
  the file.

**Verification:** Open `index.html` in Firefox + Chrome, hover each of the 5 demo words,
confirm tooltips match the scenarios above; `grep -nE 'data-es|dataset\.es|gender' index.html`
returns nothing.

---

### U3. privacy.html — Turkish branding and links

**Goal:** Retarget all Spanish references in the privacy policy.

**Requirements:** R1, R5

**Dependencies:** none

**Files:** `privacy.html`

**Approach:** Replace "Sideload Spanish" → "Sideload Turkish" in `<title>`, body copy
("Sideload Spanish does not collect…", "Sideload Spanish sync service", back-link "← Back
to Sideload Turkish"); update the GitHub issues link
`github.com/meowmeowco/sideloadspanish/issues` → `…/sideloadturkish/issues`. Leave the
"Last updated" date as-is unless content materially changes (it does not).

**Patterns to follow:** Keep the existing section headings and legal phrasing; swap only the
product name and repo slug.

**Test scenarios:** Test expectation: none — static text/link edits.

**Verification:** `grep -niE 'spanish' privacy.html` returns nothing; issues link reads
`…turkish/issues`.

---

### U4. sync-success.html + CNAME — branding and domain

**Goal:** Localize the sync-success page and point the domain config at the Turkish host.

**Requirements:** R1, R4

**Dependencies:** none

**Files:** `sync-success.html`, `CNAME`

**Approach:**
- `sync-success.html`: `<title>` "Sync Active — Sideload Turkish"; back-link text "Back to
  Sideload Turkish".
- `CNAME`: replace the single line `sideloadspanish.com` → `sideloadturkish.com`.

**Patterns to follow:** `CNAME` is a one-line file with no trailing content; preserve format
(single domain, newline-terminated).

**Test scenarios:** Test expectation: none — static text + single-line config.

**Verification:** `cat CNAME` shows `sideloadturkish.com`; `grep -ni spanish
sync-success.html` returns nothing.

---

## Scope Boundaries

In scope: localization of the four Spanish-bearing tracked files; gender-feature removal;
external-slug rewrite to Turkish parallels.

Out of scope (non-goals):
- The browser extension itself — this repo is the marketing site only. The actual Turkish
  vocabulary/tier data lives in the extension, not here.
- `styles.css` — language-neutral, no Spanish tokens, untouched.
- Creating the external resources (Turkish add-on, Buttondown list, GitHub repo) — the plan
  rewrites links to point at them; provisioning is separate.

### Deferred to Follow-Up Work
- Provision the `sideload-turkish` Firefox add-on, `sideloadturkish` Buttondown list, and
  `meowmeowco/sideloadturkish` GitHub repo so the rewritten links resolve.
- Confirm DNS / GitHub Pages custom-domain setup for `sideloadturkish.com` matches the new
  `CNAME`.

---

## Open Questions

- None blocking. The Turkish translations in KTD3 are standard; a native review pass on the
  demo words is a nice-to-have, not a blocker.

---

## Verification (whole-plan)

Final sweep across the repo: `grep -rniE 'spanish|español|espa&ntilde;|data-es|dataset\.es|data-gender'`
over all tracked files returns nothing. Site loads; all five demo tooltips render the
Turkish translation with no gender line; canonical, form, add-on, GitHub, and CNAME all read
`…turkish`.
