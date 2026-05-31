# uaamd

**Universal UAA-Aware MD Preparation Pipeline**

Turn any protein structure into a validated, simulation-ready GROMACS input — in one command.

```bash
uaamd prep --input-pdb protein.pdb
```

```
--- UAAMD Report ---
Starting pipeline for: 1AKI.pdb
Stripped crystal waters and alternate conformations.
Successfully parsed structure.
All residues found in force field database.
pdb2gmx passed
genion passed
grompp passed
MD-ready: yes ✓
```

---

## The Problem

Setting up a protein for MD simulation with GROMACS is painful:

- Crystal waters and alternate conformations break `pdb2gmx`
- Unnatural amino acids (UAAs) have inconsistent atom names across tools
- Force field compatibility issues cause cryptic errors
- Every step — cleaning, solvating, neutralizing, validating — is manual

`uaamd` automates all of it. Input a PDB. Get MD-ready files.

---

## Features

- **One-command pipeline** — `pdb2gmx → editconf → solvate → genion → grompp`
- **Auto-cleaning** — strips crystal waters and alternate conformations before GROMACS sees the file
- **Force field management** — install CHARMM force fields with `uaamd ff update charmm27`
- **UAA-aware reporting** — flags non-standard residues with specific warnings
- **Per-protein output folders** — clean separation of outputs for batch runs
- **Validation report** — explicit `MD-ready: yes/no` with full GROMACS log on failure
- **Sequence input** — build 3D structures from amino acid sequences with phi/psi angles
- **SMILES input** — generate 3D coordinates from SMILES strings via RDKit

---

## Validated Proteins

| PDB  | Description                        | Residues | UAA | MD-Ready |
|------|------------------------------------|----------|-----|----------|
| 1AKI | Hen egg-white lysozyme             | 129      | —   | ✓        |
| 1UBQ | Human ubiquitin (alt conformations)| 76       | —   | ✓        |
| 1PGB | Protein G B1 domain                | 56       | —   | ✓        |
| 1MK4 | Selenomethionine-containing protein| 348      | MSE | ✓        |
| 1L2Y | Trp-cage miniprotein               | 20       | —   | ✓        |
| 1CRN | Crambin (disulfide bonds)          | 46       | —   | ✓        |
| 2LZM | T4 lysozyme mutant                 | 164      | —   | ✓        |

---

## Installation

**Requirements:** Python ≥ 3.9, GROMACS installed

```bash
git clone https://github.com/lxzy8/uaamd.git
cd uaamd
pip install -e .
```

Install GROMACS force field:
```bash
uaamd ff update charmm27
```

---

## Usage

### Prepare a PDB for MD simulation
```bash
uaamd prep --input-pdb protein.pdb
```

Output files land in `uaamd_out/protein/`:
```
protein/
├── fixed.pdb          # cleaned input
├── complex.gro        # GROMACS structure
├── topol.top          # topology
├── complex_solv.gro   # solvated system
├── complex_ions.gro   # neutralized system
├── min.tpr            # MD-ready — pass directly to gmx mdrun
└── uaamd_report.txt   # full pipeline log
```

### Build from sequence + angles
```bash
uaamd prep --input-seq sequence_angles.txt
```

### Build from SMILES
```bash
uaamd prep --input-smiles "CC(N)C(=O)O"
```

### Update force field
```bash
uaamd ff update charmm27
```

### Custom output directory
```bash
uaamd prep --input-pdb protein.pdb --work-dir my_outputs/
```

---

## Pipeline Architecture

```
Input (PDB / Sequence+Angles / SMILES)
    │
    ▼
Text-level PDB cleaning
(strip HOH, alternate conformations)
    │
    ▼
BioPython parsing + FF residue check
(report unknown/UAA residues)
    │
    ▼
gmx pdb2gmx     ← CHARMM force field
    │
    ▼
gmx editconf    ← periodic box
    │
    ▼
gmx solvate     ← TIP3P water
    │
    ▼
gmx genion      ← neutralize charges
    │
    ▼
gmx grompp      ← final validation
    │
    ▼
MD-ready: yes/no + report
```

---

## Why uaamd?

Most structure preparation tools convert formats. `uaamd` **proves** the structure is simulation-ready by actually running GROMACS. If `gmx grompp` passes, your system is ready to simulate — no guessing.

This is especially important for **unnatural amino acids**, where atom naming inconsistencies between databases, tools, and force fields are the primary source of preparation failures.

---

## Roadmap

- [ ] CHARMM36 compatibility fix
- [ ] SMILES → MD-ready pipeline via CGenFF
- [ ] Batch processing (`uaamd prep --input-dir pdbs/`)
- [ ] Energy minimization step before validation
- [ ] PyPI package release

---

## Citation

If you use `uaamd` in your research, please cite this repository:

```
Khushal (2026). uaamd: Universal UAA-Aware MD Preparation Pipeline.
https://github.com/lxzy8/uaamd
```

---

## License

MIT
