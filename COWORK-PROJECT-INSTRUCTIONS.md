# Cowork project instructions — Spec Finder

Paste the section below into the project's custom instructions. Everything after the line is the
prompt; this header is not part of it.

State as of 2026-08-07: 472 sections · 3,284 alias terms · 4,284 alias edges · 80 evaluation cases.
Recall 71%, library precision 35.3%, scan badge precision 86.4% — all measured on one project.

---

## Project: Spec Finder (shadiaalkhaledi-dot/Spec-Search)

A client-side app that maps construction drawing terminology to MasterFormat specification sections
and acceptable manufacturers. No server, no build step, no network calls. My job with you is to keep
improving its accuracy from finished projects.

### The two files

| File | Role | Changes |
|---|---|---|
| `index.html` | Application code. ~79 KB. **Contains no spec content.** | Rarely |
| `data.js` | The entire spec library. Data only. | Every curation pass |

**They deploy together, in the same commit.** One without the other breaks the site.

### The rule that makes everything else work

A result is badged **"required specification"** only if it came from `data.js -> aliases`. Title-text
matches are shown separately, unbadged, and can never be presented as required. The curated library
is therefore the single source of truth for what the app asserts. A missing term produces "no curated
match", which is correct behaviour and a prompt to curate — not a bug.

### Data model

```js
DB.sections    // 472 · {code, title, div, divName, mfr[], bod[]}
DB.aliases     // 3,284 · "drawing term" -> [section code]     <- the asset
DB.kwByCode    // reverse index, ALWAYS rebuilt from aliases, never edited directly
DB.projects    // registry of projects that have taught this library
DB.provenance  // edges / confirmations / retired / declined / review
```

`provenance.retired` and `provenance.declined` are decisions I already made, with reasons.
**Read them before proposing anything.** Never put a rejected mapping in front of me twice.

Edges absent from `provenance.edges` are **unrecorded**, not unverified — most were derived from
earlier projects and specs, the source just was not captured at the time. Do not describe them as
untrusted.

`provenance.confirmations` counts projects that independently corroborate an edge. That is the trust
signal. One project corroborates roughly 4% of the library, so it grows slowly and by design.

### Standing rules

1. **Never hand-edit `data.js`.** Write a script. Validate that every referenced code exists in
   `sections` and fail loudly on any that does not.
2. **Always rebuild `kwByCode` from `aliases`** so the two cannot drift. Re-sort keys.
3. **Run `node eval/eval.js data.js eval/evaluation-set.json` before any commit.** A pass that lowers
   the score made the app worse, whatever else it did.
4. **Record provenance** for every edge added, and a `retired` entry with a reason for every edge
   removed.
5. **Baseline before changing anything.** Report the current numbers first, so the effect is
   measurable rather than asserted.
6. **Extraction proposes, I decide.** Automated candidates run about one third usable. Show me the
   evidence — where the term appears, what was issued — and let me reject freely.
7. **Give numbers, not adjectives.** Recall, precision, counts before and after. If an earlier claim
   turns out wrong, say so plainly and give the corrected figure.
8. **`String.replace` with a string replacement interprets `$&` and `` $` ``.** Always use a function
   replacement when inserting generated content. This has already caused one silent data-loss bug
   here.

### What I'll usually ask for

**"Curation pass on <project>"** — I give drawings plus the issued spec list. Use the
`spec-library-curator` skill. Baseline, four buckets (confirmed / gap / wrong / orphan), proposed
diff, my review, scripted apply, evaluation, changelog entry.

**"Why is it returning X for Y?"** — First check whether the result was alias-backed or title-only
(`viaAlias`). Title-only results are unbadged by design and are not library defects. If alias-backed,
find the offending alias; fix the data before touching matching logic.

**"What's the state of the library?"** — `node tools/provenance.js data.js`.

**"Add sections"** — sections issued on a project but absent from `sections` block all curation for
their terms. Add them from the spec table of contents, then map.

### Open decisions waiting on me

- `acoustic insulation` → `09 81 29`. That section is *sprayed*; the drawings I checked show batt.
  Sitting in `declined` until I rule.
- 9 entries in `provenance.review` — terms where a sibling section was issued instead.
- 17 entries in `analysis/suspects.json` from the mis-targeting audit.

### Known weaknesses — do not let me forget these

- The evaluation set is **one building**. It will happily certify a library overfit to airport
  vocabulary. Every pass should add 10–20 cases from that project.
- The scan's confidence weights were tuned on **one project**. Eight parameters, one building —
  re-validate them against ground truth on the next package.
- Only **architectural** drawings have ever been tested. Structural, mechanical and electrical
  vocabulary is unmeasured.
- Feedback flags stay in the user's browser unless `FLAG_ENDPOINT` is set in `index.html`.
- Manufacturer extraction still lets through the occasional product code or company fragment.
