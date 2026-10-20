# Antigravity Skills for Biomedical Research

Four agentic skills for structure-based drug discovery, built for
[Google Antigravity](https://antigravity.google).

## Overview

These are [Antigravity agent skills](https://github.com/google-deepmind/science-skills)
for computational drug discovery. Prompt the agent in natural language and it
selects the right skill, runs the pipeline, and returns structured results.
All dependencies are installed automatically when a skill runs for the first
time.

**Example prompts:**

- *"Prepare PDB 1ITF for docking"*
- *"Find known actives against DprE1 from M. tuberculosis"*
- *"Screen these compounds for drug-likeness"*
- *"Dock the top 10 hits against the receptor"*
- *"Design new molecules around the best scaffold using REINVENT"*
- *"Set up a 100 ns MD simulation at 310 K with CHARMM36"*

## Getting started

1. Install [Google Antigravity](https://antigravity.google)
2. Clone this repo and move the `.agents/` folder into your workspace:

```bash
git clone https://github.com/Walter-Odur/Antigravity-Skills-For-Biomedical-Research.git
cp -r Antigravity-Skills-For-Biomedical-Research/.agents /path/to/your/workspace/
```

3. Open your workspace in Antigravity and start prompting.

Antigravity discovers skills from the `.agents/skills/` directory at the root of
your workspace.

## Skills

| Skill | Description |
|---|---|
| [**Target Preparation**](.agents/skills/target_preparation/) | Fetch, clean and prepare protein structures for molecular docking |
| [**Compound Screening**](.agents/skills/compound_screening/) | Search compound libraries, dock molecules, and screen for drug-likeness |
| [**Compound Synthesis**](.agents/skills/compound_synthesis/) | Generate and optimise novel molecules with REINVENT 4 |
| [**MD Simulation**](.agents/skills/md_simulation/) | Set up molecular dynamics simulations and generate cluster submission scripts |

---

### Target Preparation

Takes a protein target from a database identifier to a docking-ready receptor
with a defined binding site.

**Accepts:** PDB ID, UniProt accession, amino-acid sequence (FASTA), or a local
PDB file. Gene names are refused because they are ambiguous across organisms.

**Capabilities:**

- Fetches structures from RCSB PDB by ID, AlphaFold DB by UniProt accession
  (resolves the current model version), or reads local files
- Cleans receptors by removing water, cryoprotectants and buffer ions while
  keeping catalytic metals. Every removal is recorded with its reason
- Preserves provenance headers (`HEADER`, `TITLE`, `EXPDTA`, `CRYST1`,
  `REMARK 2 RESOLUTION`) so the output file stays self-describing
- Converts to PDBQT format via Meeko (preferred) or Open Babel, and warns
  about approximated side chains inside the docking box
- Detects the B-factor scale from provenance, not magnitude. This matters
  because a well-refined crystal has B-factors of 20-50 and so does a poor
  predicted model's pLDDT. Reading the header avoids rejecting good crystals
  or approving bad predictions. The ESMFold API writes pLDDT between 0 and 1;
  the skill detects this and rescales correctly
- Issues PROCEED / REFINE / STOP verdicts for predicted models based on pLDDT
  confidence bands
- Defines the binding site from a co-crystallised ligand (gold standard),
  pocket residues, or whole-structure blind docking
- Ranks heterogens by druggability, not size. Cofactors like FAD, heme and
  NADH are flagged rather than selected as the docking target
- Warns when chain removal would break an interface that the binding site
  ligand sits in (the obligate-oligomer trap)
- Predicts structures with ESMFold for sequences up to 400 residues, or
  generates SLURM scripts for ColabFold and AlphaFold2 for longer sequences

---

### Compound Screening

Searches compound libraries, docks molecules, and screens them on ADMET
properties using a two-tier system.

**Capabilities:**

- Queries ChEMBL for targets and bioactivities, filtered by pChEMBL threshold,
  assay type and organism
- Searches COCONUT for natural products by substructure, molecular formula or
  text
- Generates verified ZINC bulk download URLs for purchasable compound subsets
- Docks with AutoDock Vina using automatic box placement from the binding site
  definition. Supports re-docking validation
- Two-tier ADMET screening battery:
  - **Relaxed tier** (lead-like): broader rules for early-stage filtering
  - **Strict tier** (drug-like): tighter rules for candidate selection
  - Published rules (Lipinski, Veber, Egan, Ghose, Muegge) always report their
    authors' original thresholds. A tier changes which rules must pass, never
    redefines what a violation is
- Structural alerts: PAINS, Brenk and aggregator filters are reported but do
  not gate compounds. Flagging is distinct from rejecting
- Computes molecular descriptors: MW, LogP, TPSA, HBD, HBA, rotatable bonds,
  QED, synthetic accessibility, Fsp3, aromatic ring count

---

### Compound Synthesis

Generates and optimises novel molecules using REINVENT 4 for generative
chemistry.

**Capabilities:**

- **De novo design**: unrestricted generation from a prior model
- **Scaffold hopping**: generation constrained to a core with varied
  decorations
- **R-group replacement**: systematic substitution at defined positions
- **Linker design**: connecting two fragments with generated linkers
- **Transfer learning**: fine-tuning the prior on a set of known active
  compounds
- **Multi-objective reinforcement learning**: simultaneous optimisation of
  docking score, QED, synthetic accessibility and custom scoring components
- **Scaffold-diversity diagnostics**: Murcko decomposition and Tanimoto
  clustering on REINVENT output to detect mode collapse

---

### MD Simulation

Generates GROMACS molecular dynamics workflows and validated MDP parameter
files. This skill produces files; it does not run simulations. Production MD
is GPU work that belongs in a batch job.

**Capabilities:**

- Force-field-aware MDP generation. CHARMM36 requires a force-switched
  Lennard-Jones potential and no dispersion correction; AMBER requires a plain
  cutoff with dispersion correction. They are not interchangeable and the skill
  enforces the correct settings for each
- Four-stage workflow: energy minimisation, NVT equilibration, NPT
  equilibration, and production MD, each with correct parameters
- MDP validation that cross-checks an existing MDP file against a declared
  force field and reports incompatibilities
- SLURM script generation for GROMACS MD, REINVENT, AutoDock Vina batch
  screening, ColabFold and AlphaFold2

---

## Verification

Each skill has a hand-written [verification protocol](verification/) with
measured expected values:

| Skill | Test cases |
|---|---:|
| [Target Preparation](verification/target_preparation/TEST.md) | 14 |
| [Compound Screening](verification/compound_screening/TEST.md) | 13 |
| [Compound Synthesis](verification/compound_synthesis/TEST.md) | 12 |
| [MD Simulation](verification/md_simulation/TEST.md) | 11 |

## Skill structure

Each skill folder contains:

- `SKILL.md` - Instructions the agent reads to execute the skill
- `scripts/` - PEP 723 scripts with inline dependency metadata (auto-installed via `uv run`)
- `references/` - `citation.bib`, question index, and reference material

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
