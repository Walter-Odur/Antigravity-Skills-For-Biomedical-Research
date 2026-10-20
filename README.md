# Antigravity Skills for Biomedical Research

Four agentic skills for structure-based drug discovery, built for
[Google Antigravity](https://antigravity.google).

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## Overview

These are [Antigravity agent skills](https://github.com/google-deepmind/science-skills)
for computational drug discovery. You prompt the agent in natural language and it
selects the right skill, runs the pipeline, and returns structured results.

**Example prompts:**

- *"Prepare PDB 1ITF for docking"*
- *"Find known actives against DprE1 from M. tuberculosis"*
- *"Screen these compounds for drug-likeness"*
- *"Dock the top 10 hits against the receptor"*
- *"Design new molecules around the best scaffold using REINVENT"*
- *"Set up a 100 ns MD simulation at 310 K with CHARMM36"*
- *"Run the full pipeline for PDB 6HEZ"*

## Skills

| Skill | Description |
|---|---|
| **Target Preparation** | Fetch structures from RCSB PDB or AlphaFold DB, predict from sequence with ESMFold, clean receptors, convert to PDBQT, assess model confidence, define the docking search box |
| **Compound Screening** | Query ChEMBL, COCONUT and ZINC for compounds; dock with AutoDock Vina; two-tier ADMET screening (Lipinski, Veber, Egan, Ghose, Muegge); PAINS, Brenk and aggregator alerts |
| **Compound Synthesis** | De novo design, scaffold hopping, R-group replacement, linker design, transfer learning and multi-objective reinforcement learning with REINVENT 4 |
| **MD Simulation** | GROMACS workflows and MDP files with force-field-aware non-bonded settings; MDP validation; SLURM scripts for MD, REINVENT, Vina, ColabFold and AlphaFold2 |

The **Drug Discovery Wizard** orchestrates all four across a seven-stage pipeline:
target preparation, compound sourcing, relaxed ADMET triage, docking, strict ADMET
filtering, generative chemistry, and MD setup.

## Installation

Install via [skills.sh](https://skills.sh):

```bash
npx skills add Walter-Odur/Antigravity-Skills-For-Biomedical-Research
```

Or clone directly into your Antigravity workspace:

```bash
git clone https://github.com/Walter-Odur/Antigravity-Skills-For-Biomedical-Research.git
```

## Skill structure

```
.agents/skills/
  plugin.json                        bundle manifest
  SKILL_LICENSES.md                  third-party data source terms
  target_preparation/
    SKILL.md                         skill instructions
    scripts/                         PEP 723 launcher script
    references/                      citation.bib, questions.md, best_practices.md
  compound_screening/
    SKILL.md
    scripts/
    references/                      citation.bib, questions.md, admet_thresholds.md
  compound_synthesis/
    SKILL.md
    scripts/
    examples/                        REINVENT 4 TOML configs
    references/                      citation.bib, questions.md, scoring_components.md
  md_simulation/
    SKILL.md
    scripts/
    references/                      citation.bib, questions.md, mdp_validation.md
  drug_discovery_wizard/
    SKILL.md
    scripts/
    references/                      citation.bib, questions.md, admet_thresholds.md
```

## Prerequisites

The skills use Python scripts with [PEP 723](https://peps.python.org/pep-0723/)
inline metadata. Dependencies are resolved automatically via `uv run`.

**Required:** Python 3.10+, `uv`

**Optional (installed as needed):**

| Tool | Used by |
|---|---|
| RDKit, BioPython | All skills |
| Meeko, SciPy, gemmi | Target Preparation (PDBQT conversion) |
| AutoDock Vina | Compound Screening (docking) |
| REINVENT 4 | Compound Synthesis (generative chemistry) |
| GROMACS | MD Simulation (running workflows) |

## References

Each skill ships a `citation.bib` with verified DOIs. Key references:

- Lipinski CA et al. *Adv Drug Deliv Rev* 1997;23:3-25
- Baell JB, Holloway GA. *J Med Chem* 2010;53:2719-2740 (PAINS)
- Trott O, Olson AJ. *J Comput Chem* 2010;31:455-461 (AutoDock Vina)
- Jumper J et al. *Nature* 2021;596:583-589 (AlphaFold)
- Lin Z et al. *Science* 2023;379:1123-1130 (ESMFold)
- Loeffler HH et al. *J Cheminform* 2024;16:20 (REINVENT 4)
- Zdrazil B et al. *Nucleic Acids Res* 2024;52:D1180-D1192 (ChEMBL)

## License

[MIT](LICENSE). Third-party tools and data sources carry their own terms; see
[SKILL_LICENSES.md](.agents/skills/SKILL_LICENSES.md).

## Author

**Walter Odur**

Built during the [Google DeepMind Antigravity Workshop](https://github.com/google-deepmind/science-skills), 2026.
