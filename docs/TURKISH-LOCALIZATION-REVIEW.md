# Turkish Localization Review — Action Needed

**Status:** Needs a native/fluent Turkish review before this site should be considered final.

This site was **ported from the Sideload Spanish marketing site** by mechanical
find-and-replace (Spanish → Turkish). The structure, copy, and demo widget all came from the
Spanish original. That means correctness was driven by porting rules, **not** by a Turkish
speaker — so anything language-specific needs a human pass. This doc is the checklist for that
pass.

GitHub Issues are disabled on this repo, so this lives as a tracked doc. Delete it (or check
the boxes and commit) once reviewed.

## What the port already handled

These were deliberate, language-aware decisions during the port — listed so you know they were
*considered*, not missed:

- **Grammatical gender removed.** Turkish has no noun gender, so the Spanish demo's
  `data-gender` attributes and the masculine/feminine tooltip code path were deleted, not
  ported. (Spanish original showed "♂ masculine / ♀ feminine" in the hover tooltip.)
- **Domain / external slugs** rewritten to Turkish parallels: `sideloadturkish.com`, Firefox
  add-on `sideload-turkish`, Buttondown `sideloadturkish`, GitHub `sideloadlanguage/sideloadturkish`.
- `data-es` attribute renamed to `data-tr`.

## What a Turkish reviewer must verify

### 1. Demo word translations (`index.html`, hero `<span class="sl-word">`)
The hero shows 5 inline English→Turkish word swaps. Confirm each is natural and correct in
context (the surrounding sentence is English; only the swapped word is Turkish):

- [ ] `Turkish` → **Türkçe**
- [ ] `words` → **kelimeler**
- [ ] `effort` → **çaba**
- [ ] `web` → **web** (kept as-is — confirm this reads naturally vs. "internet"/"ağ")
- [ ] Türkçe-specific characters (ç, ü, ş, ğ, ı, ö, İ) render correctly in all target browsers
      (file is UTF-8; chars are emitted raw, not as HTML entities).

### 2. Product claims about the extension (`index.html` — Features / Pricing)
These describe the **actual Turkish extension's** data and must match reality, not the Spanish
extension's:

- [ ] "the 500 most common words" — is there a Turkish 500-word tier-1 list?
- [ ] "5,000+ word vocabulary" — does the Turkish extension actually ship this?
- [ ] "Five tiers from A1 to C1" — are the Turkish tiers mapped to CEFR levels?
- [ ] "Struggling detection" / "Seen a word 10 times" — behavior parity in the Turkish build.

If the Turkish extension isn't built yet or has different numbers, update or soften these
claims so the marketing site doesn't overstate.

### 3. Turkish-language structural caveat (product-level — worth a conscious decision)
The core premise — "quietly replaces English **words** with Turkish on every page" — was
designed for Spanish, where word-for-word substitution mostly works. **Turkish is
agglutinative**: a single dictionary word takes case/possessive/plural suffixes and obeys
vowel harmony depending on its grammatical role. A naïve 1:1 word swap can produce
unnatural, suffix-less, or wrong-form Turkish.

- [ ] Decide whether the extension's replacement strategy handles Turkish morphology, and
      whether the marketing copy should set expectations accordingly. This is an extension
      concern, but the site's promise should not exceed what the extension delivers.

### 4. İ / ı casing (only if any text is uppercased/lowercased dynamically)
Turkish has dotted/dotless i (İ/i, I/ı). The marketing site is mostly static English, so this
is likely N/A — but if any Turkish string is transformed with CSS `text-transform` or JS
`toUpperCase()`, verify it doesn't mangle Turkish i's.

- [ ] Confirm no Turkish text is case-transformed incorrectly (grep for `text-transform`,
      `toUpperCase`, `toLocaleUpperCase`).

### 5. External resources actually exist
Links were rewritten to Turkish slugs that **may not be provisioned yet** (accepted at port
time):

- [ ] Firefox add-on `sideload-turkish` published?
- [ ] Buttondown list `sideloadturkish` exists (waitlist form posts to it)?
- [ ] GitHub `sideloadlanguage/sideloadturkish` (product repo, linked as "Source") exists?
- [ ] `sideloadturkish.com` DNS + GitHub Pages cert live (DNS is set; HTTPS cert
      auto-provisioning at time of writing).

## Files in scope
- `index.html` — branding, hero demo, feature/pricing claims, tooltip script
- `privacy.html` — branding + GitHub issues link
- `sync-success.html` — branding
- `CNAME` — `sideloadturkish.com`
- `styles.css` — untouched (language-neutral); no review needed

## Reference
Port plan: `docs/plans/2026-06-24-001-refactor-spanish-to-turkish-fork-plan.md`
