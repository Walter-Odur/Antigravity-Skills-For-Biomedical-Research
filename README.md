# Antigravity Skills for Biomedical Research

Four agentic skills for structure-based drug discovery, built for
[Google Antigravity](https://github.com/google-deepmind/science-skills).
Prompt the agent in natural language and it selects the right skill, runs the
pipeline, and returns structured results.

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

---

## Skills

```
                        ┌──────────────────────────┐
                        │   drug-discovery-wizard   │
                        │      (orchestrator)       │
                        └────────────┬─────────────┘
                                     │
             ┌───────────┬───────────┼───────────┐
             ▼           ▼           ▼           ▼
     ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌──────────┐
     │  target  │ │ compound   │ │ compound │ │    md    │
     │   prep   │→│ screening  │→│ synthesis│→│   sim    │
     └──────────┘ └────────────┘ └──────────┘ └──────────┘
       PDB/AF2     ChEMBL/ZINC    REINVENT 4    GROMACS
       ESMFold     COCONUT/Vina                 MDP gen
                   ADMET 2-tier
```

| Skill | Description |
|---|---|
| **Target Preparation** | Fetch structures from RCSB PDB or AlphaFold DB, predict from sequence with ESMFold, clean receptors, convert to PDBQT, assess model confidence (pLDDT), define the docking search box |
| **Compound Screening** | Query ChEMBL, COCONUT and ZINC; dock with AutoDock Vina; two-tier ADMET screening with Lipinski, Veber, Egan, Ghose and Muegge rules; PAINS, Brenk and aggregator alerts |
| **Compound Synthesis** | De novo design, scaffold hopping, R-group replacement, linker design, transfer learning and multi-objective reinforcement learning with REINVENT 4 |
| **MD Simulation** | GROMACS workflows and MDP files with force-field-aware non-bonded settings; MDP validation; SLURM scripts for MD, REINVENT, Vina batch screening, ColabFold and AlphaFold2 |

The **Drug Discovery Wizard** orchestrates all four across a seven-stage
pipeline. It delegates to the same code the individual skills use.

---

## Usage

Prompt Antigravity with what you need:

- *"Prepare PDB 1ITF for docking"*
- *"Find known actives against DprE1 from M. tuberculosis"*
- *"Screen these compounds for drug-likeness"*
- *"Dock the top 10 hits against the receptor"*
- *"Design new molecules around the best scaffold using REINVENT"*
- *"Set up a 100 ns MD simulation at 310 K with CHARMM36"*
- *"Run the full pipeline for PDB 6HEZ"*

The agent reads each skill's `SKILL.md`, selects the appropriate one, and
handles the rest.

---

## Setup

Requires Python 3.10+.

```bash
git clone https://github.com/Walter-Odur/Antigravity-Skills-For-Biomedical-Research.git
cd Antigravity-Skills-For-Biomedical-Research

pip install -e ".[all]"
```

Verify:

```bash
pytest
```

No network access or external binaries required for the test suite.

### Optional dependencies

The core runs on `rdkit` and `biopython` alone. Everything else degrades
gracefully with a clear message.

| Tool | Required for | Install |
|---|---|---|
| Meeko, SciPy, gemmi | PDBQT conversion | `pip install meeko scipy gemmi` |
| AutoDock Vina | Docking | `conda install -c conda-forge vina` |
| GROMACS | Running MD workflows | `conda install -c conda-forge gromacs` |
| REINVENT 4 | Generative chemistry | [github.com/MolecularAI/REINVENT4](https://github.com/MolecularAI/REINVENT4) |

---

## What each skill does

### Target Preparation

Takes a PDB ID, UniProt accession, amino-acid sequence or local file and
produces a docking-ready receptor with a defined binding site. Gene names are
refused because they are ambiguous across organisms.

- Fetches from RCSB PDB, AlphaFold DB (current model version), or local files
- Cleans receptors: removes water, cryoprotectants, buffer ions; keeps catalytic metals
- Converts to PDBQT via Meeko (preferred) or Open Babel
- Detects the B-factor scale from provenance, not magnitude, and issues
  PROCEED / REFINE / STOP verdicts for predicted models
- Defines the binding site from a co-crystallised ligand, pocket residues, or
  whole-structure blind docking
- Ranks heterogens by druggability, not size: cofactors (FAD, heme, NADH) are
  flagged, not selected
- Warns when chain removal breaks an interface site
- Predicts structures with ESMFold (up to 400 residues) or generates SLURM
  scripts for ColabFold / AlphaFold2

### Compound Screening

- Queries ChEMBL for targets and bioactivities, COCONUT for natural products,
  ZINC for purchasable compounds
- Docks with AutoDock Vina using automatic box placement
- Two-tier ADMET battery: relaxed (lead-like) and strict (drug-like)
- Structural alerts: PAINS, Brenk, aggregator filters (flagged, not gating)
- Molecular descriptors: MW, LogP, TPSA, HBD, HBA, rotatable bonds, QED,
  synthetic accessibility, Fsp3, aromatic rings

### Compound Synthesis

- De novo design, scaffold hopping, R-group replacement, linker design
- Transfer learning on known actives
- Multi-objective reinforcement learning (docking score, QED, synthetic
  accessibility, custom components)
- Scaffold-diversity diagnostics via Murcko decomposition and Tanimoto
  clustering

### MD Simulation

Produces files only. Production MD belongs in a batch job.

- Force-field-aware MDP generation (CHARMM36 and AMBER have incompatible
  non-bonded settings; the skill enforces the correct ones)
- Four-stage workflow: energy minimisation, NVT, NPT, production
- MDP validation against a declared force field
- SLURM scripts for GROMACS, REINVENT, Vina batch screening, ColabFold,
  AlphaFold2

---

## Diagnostic figures

Subcommands that produce reports can also generate figures (`--plot`):

| Subcommand | Figure |
|---|---|
| `prepare-receptor` | Per-residue trace, cleaning counts, ligand ranking, docking box over structure |
| `assess-structure` | Per-residue pLDDT with confidence bands and low-confidence regions marked |
| `admet-filter` | Outcome funnel, failure breakdown, descriptor spread against tier bounds |
| `dock` | Ranked affinities with 1 kcal/mol band, pose spread |
| `wizard run` | Stage-by-stage progress |

Requires matplotlib: `pip install "agskills[plots]"`

---

## Repository layout

```
src/agskills/          shared implementation
  chem/                descriptors, rules, alerts, ADMET tiers
  targets/             fetch, clean, quality, PDBQT, binding sites
  libraries/           ChEMBL, COCONUT, ZINC clients
  docking/             AutoDock Vina
  md/                  MDP generation/validation, GROMACS workflows
  reinvent/            REINVENT 4 configs, scoring, analysis
  cli/                 one entry point per skill
.agents/skills/        skill bundle (what Antigravity reads)
  <skill>/SKILL.md     instructions for the agent
  <skill>/scripts/     PEP 723 launcher scripts
  <skill>/references/  citation.bib, questions.md
tests/                 21 test modules, real-data fixtures
documentation/         per-skill documentation (DOCX)
verification/          hand-written verification protocols
```

---

## Testing

```bash
pytest                       # hermetic, no network
pytest -m network            # live API calls
pytest -m requires_vina      # real docking
pytest --cov=agskills        # coverage
```

Tests use real data: PDB structures, AlphaFold models, reference properties for
marketed drugs, REINVENT output, and authentic Vina results.

### Question indexes

106 questions across the four skills mapping common questions to the commands
that answer them:

| Skill | Questions |
|---|---:|
| [Target Preparation](.agents/skills/target_preparation/references/questions.md) | 23 |
| [Compound Screening](.agents/skills/compound_screening/references/questions.md) | 25 |
| [Compound Synthesis](.agents/skills/compound_synthesis/references/questions.md) | 21 |
| [MD Simulation](.agents/skills/md_simulation/references/questions.md) | 20 |

### Verification protocols

Hand-written protocols under `verification/` with measured expected values:

| Skill | Cases |
|---|---:|
| [Target Preparation](verification/target_preparation/TEST.md) | 14 |
| [Compound Screening](verification/compound_screening/TEST.md) | 13 |
| [Compound Synthesis](verification/compound_synthesis/TEST.md) | 12 |
| [MD Simulation](verification/md_simulation/TEST.md) | 11 |

---

## Configuration

| Variable | Purpose |
|---|---|
| `AGSKILLS_USER_AGENT` | User-Agent for public API requests |
| `AGSKILLS_CACHE_DIR` | Rate-limiter state directory |
| `REINVENT_DIR` | REINVENT 4 checkout path |
| `REINVENT_PRIOR_BASE` | Prior model directory |

---

## Scientific conventions

- **Molecular weight** is average mass (as in Lipinski and PubChem), not
  monoisotopic.
- **Published rules report their authors' thresholds.** A tier changes which
  rules must pass, never redefines a violation.
- **Structural alerts are flagged, not gating.** PAINS and Brenk hits are
  reported but do not reject compounds.
- **Non-bonded settings belong to the force field.** CHARMM36 and AMBER
  settings are not interchangeable.
- **Docking scores are triage.** Differences under ~1 kcal/mol are not
  meaningful.
- **pLDDT is not a B-factor.** Higher is better (0-100) for predicted models;
  lower is better (unbounded) for crystal structures. The skills detect which
  scale applies.

---

## Citations

The methods and thresholds used come from:

| Method | Reference |
|---|---|
| Lipinski's Rule of Five | Lipinski CA et al. *Adv Drug Deliv Rev* 1997;23:3-25 |
| Veber rules | Veber DF et al. *J Med Chem* 2002;45:2615-2623 |
| Egan filter | Egan WJ et al. *J Med Chem* 2000;43:3867-3877 |
| Ghose filter | Ghose AK et al. *J Comb Chem* 1999;1:55-68 |
| Muegge filter | Muegge I et al. *J Med Chem* 2001;44:1841-1846 |
| Lead-likeness | Hughes JD et al. *Bioorg Med Chem Lett* 2008;18:4872-4875 |
| PAINS | Baell JB, Holloway GA. *J Med Chem* 2010;53:2719-2740 |
| Structural alerts | Brenk R et al. *ChemMedChem* 2008;3:435-444 |
| Synthetic accessibility | Ertl P, Schuffenhauer A. *J Cheminform* 2009;1:8 |
| QED | Bickerton GR et al. *Nat Chem* 2012;4:90-98 |
| AutoDock Vina | Trott O, Olson AJ. *J Comput Chem* 2010;31:455-461 |
| Vina 1.2 | Eberhardt J et al. *J Chem Inf Model* 2021;61:3891-3898 |
| CHARMM36 | Huang J, MacKerell AD. *J Comput Chem* 2013;34:2135-2145 |
| ESMFold | Lin Z et al. *Science* 2023;379:1123-1130 |
| AlphaFold | Jumper J et al. *Nature* 2021;596:583-589 |
| AlphaFold DB | Varadi M et al. *Nucleic Acids Res* 2024;52:D368-D375 |
| ChEMBL | Zdrazil B et al. *Nucleic Acids Res* 2024;52:D1180-D1192 |
| COCONUT | Sorokina M et al. *J Cheminform* 2021;13:2 |
| REINVENT 4 | Loeffler HH et al. *J Cheminform* 2024;16:20 |

Individual thresholds carry their citation in the JSON output. Each skill ships
`references/citation.bib`. The bundle carries **171 citation entries**, all with
DOIs verified through Crossref.

---

## License

[MIT](LICENSE). Third-party tools and data sources carry their own terms; see
[SKILL_LICENSES.md](.agents/skills/SKILL_LICENSES.md).

---

## Author

**Walter Odur**

Built during the [Google DeepMind Antigravity Workshop](https://github.com/google-deepmind/science-skills), 2026.
