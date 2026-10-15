# Carbon Tracing AI Review Skill

A Codex skill for reviewing flagged reactions in step-2 Excel workbooks from
[Atom-Mapped Carbon Tracing](https://github.com/LGJee/atom-mapped-carbon-tracing).
It uses the separate
[Enzymatic Atom Mapping skill](https://github.com/LGJee/enzymatic-atom-mapping-skill)
for evidence-based biochemical analysis, then validates and saves accepted
mapping replacements. It does not replace that scientific skill or the carbon
tracing project.

## Install and use

Install this repository as a Codex skill, together with its scientific dependency.
For a standard installation, their entry files are:

```text
~/.codex/skills/carbon-tracing-ai-review-skill/SKILL.md
~/.codex/skills/enzymatic-atom-mapping/SKILL.md
```

With Git available, these commands install into those locations when the target
directories do not already exist:

```bash
git clone https://github.com/LGJee/carbon-tracing-ai-review-skill.git \
  ~/.codex/skills/carbon-tracing-ai-review-skill
git clone https://github.com/LGJee/enzymatic-atom-mapping-skill.git \
  ~/.codex/skills/enzymatic-atom-mapping
```

Give Codex one or more `<model>_2_atom_mapping.xlsx` files and invoke:

```text
$carbon-tracing-ai-review-skill
Review these step-2 workbooks. My carbon-tracing project is at <project path>.
Save the reviewed copies and detailed report to its configured results directory.
```

Use a compatible local or published tracing project. The helper checks its actual
interfaces; a repository name alone does not establish compatibility. Review
uses the tracing Python environment with openpyxl, RDKit and the scientific
validator. Integrated validation also uses the project's existing dependencies.
No mapper models or mapper predictions are required by this review helper.
Structural review can proceed without a compatible project, with integration
explicitly reported as unchecked.

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
python3 step4_carbon_flow_ai.py
python3 step4_carbon_flow_ai.py MyCulture --rounds 1
```

Step 4 reads the reviewed `Final_Mapping` using the project's existing propagation
implementation. Model names, input hashes and ordered carbon sources must match
the analysis plan. A missing step-4 entry does not prevent workbook delivery.

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
copies, report counts, archives and the existing step-4 path. These tests do not
establish biochemical accuracy. Bookkeeping validation and AI provenance do not
certify experimental correctness. Carbon propagation describes possible origin
label reachability, not isotope abundance or quantitative carbon allocation.
