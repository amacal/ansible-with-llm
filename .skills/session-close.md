Write the notes file, the `.history/` entry, and the `.index/` session directory for the session that just ended. Follow every CLAUDE.md rule exactly. No scripts — every write here is a direct `Write`/`Edit`; every lookup is `Read`/`Grep`/`Glob` against `.index/`/`.history/`, per `.index/schema.yml` and `.history/schema.yml`.

## Steps

1. **Identify the exercise and target.** Find the exercise directory (`{target-dir}/{dir}/`, a short name — not the slug) created/primarily worked on this session, and which target environment it belongs to. Review session (no exercise directory) → identify the concept(s) consolidated instead, and the target(s) they concern.

2. **Write the notes file** at `{target-dir}/{dir}.md` (sibling of the exercise directory, same basename) — **implementation sessions only**. Review session → skip this step entirely; review sessions get no companion `.md` notes file, per `.history/schema.yml`'s `file: — for review sessions` (the exercise's own earlier notes files are the reference; `.history`'s `completed`/`explored` fields and `.index`'s `summary.txt` carry the session record instead). Otherwise, follow CLAUDE.md's "Notes writing style" exactly:
   - Section order: `# Title`, `## Overview`, zero+ bespoke theory/mechanism sections, `## Observable behavior`, `## Internal mechanism`, `## Design rationale`, `## Edge cases` (if applicable), `## Worked example`.
   - No `## Depends on`/`## Unlocks` — relationship structure lives solely in `.index/`.
   - Full prose paragraphs only, everywhere, including the worked example — no bullets.
   - All math on its own `$$...$$` line, never inline (rare in this domain; apply only when one actually appears).
   - Every formula: a sentence before (why) and after (what it means).
   - Patient-engineer-to-peer voice — slow, explicit, nothing obvious.
   - No one-sentence paragraphs — attach orphans to a neighbor.
   - Concept defined in an earlier session's notes → cite that file by name, state only the reused fact, don't re-derive.
   - Target 600–1400 words. Shorter (thin retarget) or longer (multi-concept synthesis) → say so explicitly in Overview or Design rationale.
   - Worked example: non-trivial (exercises the interesting case), verifiable mentally in under a minute, the actual task list/variables/managed-node state traced fully in prose.
   - Every recap figure and run outcome stated as observed (`ok=`/`changed=` counts, a handler firing, a failure) is taken from a run that actually happened this session, paired with the playbook version and node state that run really had — never assembled from two different runs, and never borrowed from a similar-looking run of another design iteration or another node state (fresh vs converged) when the session went through several. When the notes name a scenario, the figure quoted for it comes from the run of exactly that scenario under the design being described. Anything only reasoned through in dialogue is written as reasoning ("a later run would…"), never as an observation.

3. **Confirm idempotency before writing anything else (implementation sessions only).** The exercise's playbook must have been run twice this session with the second run reporting zero `changed` tasks (per CLAUDE.md's "Idempotency discipline") — if this didn't happen, the session isn't closeable as `completed`; stop here and report that back instead of writing a close.

4. **Write the `.history/` entry.** Determine `date` as today. Derive `<slug>` from the exact Concept Title via `.index/schema.yml`'s `slug_algorithm`. `Write` a single new file at `.history/<YYYY-MM>/<date>-<slug>.yml` (create the month directory if it doesn't exist yet — nothing else to set up, no header/chaining fields) with exactly the shape in `.history/schema.yml`'s `file_format`:

   ```yaml
   date: "YYYY-MM-DD"
   title: "Exact Concept Title"
   session:
     file: docker-local/{dir}/   # or the correct target-dir, or — for a review session
     status: completed
     attempted: [...]
     explored: [...]
     tried: [...]
     corrections: [...]
     bugs_found: [...]             # always — for review sessions
     completed: [...]
     not_completed: [...]
     open_questions: [...]
     notes: [...]
   ```

   Fill every field per CLAUDE.md's field semantics, `—` for empty (never omit a field, never an empty list, never mix `—` into a populated list; each list field is either the literal `—` or a non-empty list). Observable session events only — no module/variable-precedence descriptions or idempotency-correctness arguments (→ notes file, cited by fact if reused), no `Depends on`/`Unlocks` field, no personal info/personality/psychological interpretation/subjective judgment. A diagnosed wrong prediction (per CLAUDE.md's "Prediction and confirmation") goes in `corrections`, stated factually before/after. This `Write` is the entire operation — never touch any other file under `.history/`.

5. **Check word count** (`wc -w`) on the notes file — implementation sessions only (skipped along with step 2 for review sessions). Outside 600–1400 without a stated justification in the file → revise.

6. **Write the `.index/sessions/<slug>/` directory (10 files) — never a full-tree read, no script.** Treat every already-completed session's existing files as ground truth; do not re-read every old notes/exercise/`.history/` file to re-derive them from scratch. Classify honestly against CLAUDE.md's "Fast index (`.index/`)" tests, using targeted lookups only:
   - `grep -l 'title: "<candidate title>"' .index/sessions/*/meta.yml` to confirm a candidate prerequisite/related session actually exists and get its directory.
   - `Read` that session's own `.yml`/`.txt` files (not the whole tree) for the specific reused fact.

   Then `Write` all 10 files in `.index/sessions/<slug>/`: `meta.yml` (`title`/`kind`/`target`/`file`/`date`), `summary.txt` (plain text, one sentence), and `prerequisites.yml`/`uses_concepts.yml`/`derived_from.yml`/`related_to.yml`/`unlocks.yml`/`future_targets.yml`/`concepts.yml`/`capabilities.yml` (each a YAML list of exact titles/tags, or the literal scalar `—`, per `.index/schema.yml`'s `empty_field_convention`). If this exercise's pattern also exists against another target/OS-family already covered, add that session's exact title to `related_to.yml` (the cross-target comparison pair — see CLAUDE.md's "Cross-target comparison"). No `reuses_code` file — a genuine continuation belongs in `derived_from.yml` instead, per `.index/schema.yml`.

7. **Future targets.** For each title in this session's own `future_targets.yml` that isn't already a `.index/future-targets/<slug>.yml` file, `Write` one (`title`/`status: not-completed`/`mentioned_by`/`evidence`, per `.index/schema.yml`). If this session's own title matches an *existing* `.index/future-targets/<slug>.yml`, `rm` that file now — a title cannot be both a pending target and a completed session.

8. **Branches.** If this session extends an existing branch, `Edit` `.index/branches/<slug>.yml`: append the title to `sessions`, and replace `frontier` with whatever the new frontier genuinely is (usually just this session, but say so explicitly if a sibling session in the same branch is still also frontier). If this session opens a brand-new branch, `Write` a new `.index/branches/<slug>.yml` (`title`/`sessions`/`frontier`/`explicit_future_targets`).

9. **Everything else is curated, not mechanical — touch only what genuinely changed.** `.index/teaching/calibration.yml` gets one `trials` entry per dimension whose variant this session visibly exercised, with its observable signals (the user's own words quoted, counts, prediction outcomes — never a judgment of the person), and a `current_default` change only when those signals or an explicit user request warrant it, per CLAUDE.md's "Teaching calibration". `.index/selection-context/active-branches.yml`, `candidate-signals.yml`, and `reusable-recent-capabilities.yml` are hand-curated judgment (not a formula — don't try to make one). `.index/open-gaps/<category>/<slug>.yml` and `.index/edges/<slug>.yml` are curated prose, written rarely. Edit only the specific file this session's evidence actually changes. Never touch `.index/schema.yml` itself as part of a normal close (a session doesn't change the schema). Nothing needs to be written for `recent_sessions` or `explicit_unfinished_targets` — those are computed on demand, never stored (see `.index/schema.yml`'s `computed_not_stored`).

10. **Retrospective revision of an *older* session's fields is still allowed** (that's `.index/`'s whole point of difference from `.history/`), but is a deliberate, targeted `Edit` to that one file — only when this new session's evidence specifically implicates a prior classification (state the reason in your report). Never a routine side effect, never a from-scratch re-derivation of an older session just to double-check it, never mirrored into that older session's `.history/` entry.

11. **Validate before finishing — no script, run these directly with Grep/Glob** (the exact checks are in `.index/schema.yml`'s `validation` section):
    - `ls .index/sessions/<slug>/ | wc -l` is exactly 10.
    - Every title in every `prerequisites.yml`/`uses_concepts.yml`/`derived_from.yml`/`related_to.yml` you just wrote resolves: `grep -l 'title: "<ref>"' .index/sessions/*/meta.yml` finds exactly one file.
    - Every title in `unlocks.yml`/`future_targets.yml` resolves to either a session or a `.index/future-targets/*.yml` title.
    - No title exists as both a `.index/sessions/*/meta.yml` title and a `.index/future-targets/*.yml` title.
    - Every tag in `concepts.yml`/`capabilities.yml` matches `^[a-z0-9]+(-[a-z0-9]+)*$`.
    - `meta.yml`'s `target` field is one of `docker-local | vagrant-local | lxd-local | hetzner-cloud | aws-cloud`.

12. **Report back** every file you wrote/edited/removed, verbatim, with full paths (the new `.history/` entry path, the `.index/sessions/<slug>/` directory and its 10 files, any future-target/branch files touched, any retrospective edit and its stated reason) — the calling conversation passes this into the verify agent's prompt so it can check the actual files directly instead of re-deriving "what changed" by globbing the whole tree.
