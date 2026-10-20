# Antigravity Skills for Biomedical Research

Four agentic skills for structure-based drug discovery, built for
[Google Antigravity](https://antigravity.google).

## Overview

These are [Antigravity agent skills](https://github.com/google-deepmind/science-skills)
for computational drug discovery. Prompt the agent in natural language and it
selects the right skill, runs the pipeline, and returns structured results.

**Example prompts:**

- *"Prepare PDB 1ITF for docking"*
- *"Find known actives against DprE1 from M. tuberculosis"*
- *"Screen these compounds for drug-likeness"*
- *"Dock the top 10 hits against the receptor"*
- *"Design new molecules around the best scaffold using REINVENT"*
- *"Set up a 100 ns MD simulation at 310 K with CHARMM36"*

## Skills

| Skill | Description |
|---|---|
| [**Target Preparation**](target_preparation/) | Fetch structures from RCSB PDB or AlphaFold DB, predict from sequence with ESMFold, clean receptors, convert to PDBQT, assess model confidence, define the docking search box |
| [**Compound Screening**](compound_screening/) | Query ChEMBL, COCONUT and ZINC for compounds; dock with AutoDock Vina; two-tier ADMET screening (Lipinski, Veber, Egan, Ghose, Muegge); PAINS, Brenk and aggregator alerts |
| [**Compound Synthesis**](compound_synthesis/) | De novo design, scaffold hopping, R-group replacement, linker design, transfer learning and multi-objective reinforcement learning with REINVENT 4 |
| [**MD Simulation**](md_simulation/) | GROMACS workflows and MDP files with force-field-aware non-bonded settings; MDP validation; SLURM scripts for MD, REINVENT, Vina, ColabFold and AlphaFold2 |

## Installation

```bash
npx skills add Walter-Odur/Antigravity-Skills-For-Biomedical-Research
```

## Skill structure

Each skill folder contains:

- `SKILL.md` - Instructions for the agent
- `scripts/` - PEP 723 launcher scripts
- `references/` - Citations, question index, and reference material

## Prerequisites

Python 3.10+ and `uv`. Dependencies are resolved automatically via `uv run`.

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

## Author

**Walter Odur**

Built during the [Google DeepMind Antigravity Workshop](https://github.com/google-deepmind/science-skills), 2026.
