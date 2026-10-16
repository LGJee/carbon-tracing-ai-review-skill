# Carbon Tracing AI Review Skill

A Codex skill that reviews flagged atom mappings from
[Atom-Mapped Carbon Tracing](https://github.com/LGJee/atom-mapped-carbon-tracing)
and saves reviewed Excel copies with a detailed report.

## Install with AI (recommended)

Copy this one-line instruction into Codex:

```text
$skill-installer Install https://github.com/LGJee/carbon-tracing-ai-review-skill and its required enzymatic-atom-mapping skill.
```

The required dependency is
[Enzymatic Atom Mapping](https://github.com/LGJee/enzymatic-atom-mapping-skill).
Codex can handle the download and installation paths. Keep existing installations
unless an update is needed. If the skills do not appear, restart Codex.
See the [official skill installation guidance](https://learn.chatgpt.com/docs/build-skills#install-curated-skills-for-local-use).

<details>
<summary>Manual installation (optional)</summary>

Clone both repositories into the user skill directory supported by your Codex
installation. Current versions use `~/.agents/skills`:

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/LGJee/carbon-tracing-ai-review-skill.git \
  ~/.agents/skills/carbon-tracing-ai-review-skill
git clone https://github.com/LGJee/enzymatic-atom-mapping-skill.git \
  ~/.agents/skills/enzymatic-atom-mapping
```

Use the configured skill directory instead if your installation differs. Do not
overwrite existing directories without checking their contents.

</details>

## Use

Attach one or more `<model>_2_atom_mapping.xlsx` files and send:

```text
$carbon-tracing-ai-review-skill
Review these files and save the reviewed copies and report to my project's results directory.
```

Provide your project path if Codex cannot locate it. Installing the skills does
not install the carbon-tracing project or its Python environment.

<details>
<summary>Runtime requirements</summary>

Use a compatible local or published tracing project. The helper checks its actual
interfaces; a repository name alone does not establish compatibility. Review
uses the tracing Python environment with openpyxl, RDKit and the scientific
validator. Integrated validation also uses the project's existing dependencies.
No mapper models or mapper predictions are required by this review helper.
Structural review can proceed without a compatible project, with integration
explicitly reported as unchecked.

</details>

## Review and output

The skill reads flagged reactions from `Mapping_Selection` and `Final_Mapping`,
including failed reactions absent from the final sheet. It analyzes the supplied
unmapped reaction structures and existing tool candidates. It does not invent
missing structures, add omitted atoms or cofactors, or apply unsupported
reaction-specific corrections.

Original workbooks are preserved. Reviewed copies use the name
`<model>_2_atom_mapping_AIGenerated.xlsx`; existing completed copies are not
silently overwritten. **Only `Final_Mapping` changes.** Raw tool predictions,
`Mapping_Selection`, scores, input tables and hidden provenance remain intact.

| Final_Mapping field | Meaning |
| --- | --- |
| `MappedReactionSMILES` | Complete validated mapping, replaced only when a proposal is accepted |
| `SelectedMapper` | `AI-assisted` for accepted replacements |
| `ReviewRequired` | Original review flag; preserved by default |
| `AIReviewed` | Boolean TRUE after reaction-level scientific analysis, including retained or unresolved decisions; inserted immediately right of `ReviewRequired` |
| `AIGenerated` | Boolean TRUE only for accepted AI-assisted replacements |

Automatic input screening alone does not set `AIReviewed`. A reaction can be
reviewed without receiving a new mapping. Missing final rows are added only
when a complete mapping is accepted, not solely to hold a review flag.
Review-only copies can therefore have no TRUE `AIGenerated` values.

The configured results directory receives timestamped reports:

```text
AI_mapping_review_<timestamp>.md
AI_mapping_review_<timestamp>.json
```

The Markdown report includes per-workbook totals and per-reaction decisions,
evidence, limitations and unresolved reasons. The JSON retains full validation
outcomes. Reports include unsuccessful and zero-replacement runs. Counts are
workbook records; a shared reaction in multiple models is counted separately.
AI-reviewed counts overlap the decision categories and are not added to them.

Immutable review records are stored separately under
`<project>/data/ai_mapping_reviews/`. Exact-input accepted candidates can be
reused after reviewing applicability and rerunning validation. Cache presence is
not proof of correctness. User workbooks, reports and cache records are not
included in this repository.

## Optional carbon propagation

Review and delivery do not automatically run carbon propagation. When explicitly
requested, a compatible tracing checkout can consume the reviewed copies with:

```bash
python3 step3_carbon_flow.py
python3 step3_carbon_flow.py MyCulture --rounds 1
```

Step 3 reads available ordinary and AI-reviewed mappings independently and writes
separate `_3_carbon_flow_Enumeration.xlsx` and
`_3_carbon_flow_Enumeration_AIGenerated.xlsx` outputs. Use `--plan` for a custom
analysis. Model names, input hashes and ordered carbon sources must match the
analysis plan. No method selector or separate step4 is needed. A missing
propagation entry does not prevent workbook delivery.

Newer project interfaces also copy saved result Summaries into
`carbon_flow_summary.xlsx`, one sheet per workbook. That collection may include
older saved results; use `carbon_flow_run.json` to identify current-run outputs.
Inspect the installed project interface; this skill update does not publish or
upgrade the main computation project.

## Helper interface and checks

[SKILL.md](SKILL.md) defines the agent workflow.
[Workbook handoff](references/workbook-handoff.md) documents helper commands,
proposal/review JSON, archives and output rules. The helper applies supplied
review decisions; running it alone does not perform literature research or AI
prediction.

Run the isolated regression suite with your compatible tracing interpreter:

```bash
"$TRACING_PYTHON" scripts/test_review_workbooks.py \
  --project "$PROJECT_DIR" --mapping-skill "$MAPPING_SKILL_DIR" -v
```

The publication check on 2026-10-15 passed 19 synthetic tests covering flags,
input hashes, rejected proposals, original-sheet preservation, review-only
copies, report counts, archives and the then-current propagation handoff.
On 2026-10-17 the same 19-test suite passed against the maintained step-3
Enumeration interface, including a reviewed mapping consumed without a mapper. These tests do not
establish biochemical accuracy. Bookkeeping validation and AI provenance do not
certify experimental correctness. Carbon propagation describes possible origin
label reachability, not isotope abundance or quantitative carbon allocation.
