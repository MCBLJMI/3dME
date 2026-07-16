# 3dME — 3-Dimensional Molecular Extractor & Cheminformatics Suite

A desktop application that automates the front end of a structure-based
drug-discovery pipeline: fetch 3D molecular structures from public chemical
databases, prepare them for docking, compute ADMET and drug-likeness
descriptors, screen libraries by similarity, visualise the physicochemical
profile of a compound set, and run and rank molecular docking — all from one
graphical interface, without writing code.

Developed by the **Mathematical and Computational Biology Lab (MCBL)**.

---

## Table of contents

- [Features](#features)
- [Screenshots](#screenshots)
- [Requirements](#requirements)
- [Installation](#installation)
  - [Option A — run the pre-built executable (recommended)](#option-a--run-the-pre-built-executable-recommended)
  - [Option B — run from source](#option-b--run-from-source)
- [External tools (Open Babel & AutoDock Vina)](#external-tools)
- [Building the executable yourself](#building-the-executable-yourself)
- [Usage](#usage)
- [Output files](#output-files)
- [Troubleshooting](#troubleshooting)
- [Known limitations](#known-limitations)
- [Documentation](#documentation)
- [License & citation](#license--citation)

---

## Features

3dME is organised into six panels:

| Panel | What it does |
|---|---|
| **3D Downloader** | Fetches 3D (or 2D) structures from eleven databases (IMPPAT, ZINC, BIMP, RCSB, PubChem, ChEMBL, ChEBI, EPA CompTox, DrugBank…) by identifier, name, or whole plant; converts each to MOL2 and PDBQT; computes properties. |
| **ADMET Predictor** | Computes MW, WLOGP, HBD, HBA, TPSA, rotatable bonds, MR and atom count from SMILES; evaluates Lipinski, Ghose and Veber rules and the PAINS / Brenk / NIH structural-alert catalogs. |
| **Batch Converter** | Bulk folder utilities: 2D→3D energy minimisation, conversion to PDBQT, canonical-SMILES extraction, and multi-SDF splitting. |
| **Screening** | Ranks a folder of structures by Tanimoto similarity (Morgan/ECFP4) to a reference molecule and copies out the hits. |
| **Analytics** | Eight interactive, publication-ready plots of an ADMET workbook (BOILED-Egg, Lipinski & Veber space, filter summary, bioavailability radar, descriptor distributions, correlation heatmap, chemical-space bubble). Exports PDF and 600-dpi TIFF. |
| **Molecular Docking** | Runs AutoDock Vina over a ligand folder against a receptor and compiles the best binding affinities into one ranked workbook. |

Every long-running job runs on a background thread, reports live progress,
can be interrupted with a **Stop** button, and reports which items failed on
completion.

---

## Screenshots

<!-- Add screenshots to the docs/ folder and reference them here, e.g.:
![3D Downloader](docs/screenshot_downloader.png)
![BOILED-Egg plot](docs/screenshot_boiledegg.png)
-->

_(Add screenshots here.)_

---

## Requirements

- **Windows 10/11 (64-bit)** — the primary supported platform.
  Linux and macOS can run from source but need one code change first
  (see [Known limitations](#known-limitations)).
- **For the pre-built .exe:** nothing except the two external tools below.
- **For running from source:** Python 3.10–3.12.
- **Always required (not bundled):** Open Babel and AutoDock Vina — see
  [External tools](#external-tools).

---

## Installation

### For end users — the all-in-one package (no setup)

1. Download `3dME-windows.zip` from the [Releases](../../releases) page.
2. Unzip it anywhere (e.g. `C:\3dME\`).
3. Double-click `3dME.exe`.

That's it. This package bundles **everything** — Python, all libraries, and the
chemistry tools (Open Babel and AutoDock Vina) in a `tools\` folder next to the
exe. Nothing to install, no PATH to edit. On first launch 3dME checks that the
tools are present and tells you if anything is missing.

> On Windows, SmartScreen may warn that the app is from an unknown publisher
> (it is unsigned). Choose **More info → Run anyway**.

### For developers — run from source

```bash
git clone https://github.com/<your-org>/3dME.git
cd 3dME

python -m venv venv
venv\Scripts\activate            # Windows
# source venv/bin/activate       # Linux / macOS

pip install -r requirements.txt
python 3dme.py
```

When run from source, 3dME looks for `obabel`/`vina` in a local `tools\` folder
first, then falls back to your system PATH — so a developer install of the
tools works too.

---

## External tools

3dME calls two command-line chemistry programs: **Open Babel** (`obabel`,
`obgen`, `obprop`) for format conversion, 3D generation and properties, and
**AutoDock Vina** (`vina`) for docking.

**End users of the packaged release do not need to do anything** — these are
bundled in the `tools\` folder.

**If you are assembling the release yourself**, place the tool binaries in
`tools\` before building. Full step-by-step instructions, including the Open
Babel DLL/data caveat and the licensing considerations, are in
[`tools/README_ASSEMBLE_TOOLS.md`](tools/README_ASSEMBLE_TOOLS.md).

Quick reference:

- **Open Babel** — install `OpenBabel-3.1.1-x64.exe` from the
  [releases](https://github.com/openbabel/openbabel/releases), then copy the
  **whole install folder** (exe + DLLs + `data`) into `tools\`.
- **AutoDock Vina** — download `vina_1.2.x_windows_x86_64.exe` from the
  [releases](https://github.com/ccsb-scripps/AutoDock-Vina/releases), rename to
  `vina.exe`, drop into `tools\`. (3dME auto-detects Vina 1.1.2 vs 1.2.x.)

> **Receptor prep is out of scope.** The Molecular Docking panel expects the
> receptor already in PDBQT format (water removed, polar hydrogens and charges
> added). Prepare it with AutoDockTools/MGLTools or Meeko. 3dME produces the
> **ligand** PDBQTs for you.

---

## Building the executable yourself

The repository includes a PyInstaller spec that handles RDKit, matplotlib and
customtkinter data files (the parts PyInstaller misses by default).

### Windows

```bat
build_windows.bat
```

This creates a virtual environment, installs dependencies, and produces
`dist\3dME\3dME.exe`. Zip the whole `dist\3dME\` folder to distribute it.

### Manually (any platform)

```bash
pip install -r requirements.txt
pyinstaller 3dme.spec --noconfirm
```

The build is a **--onedir** bundle (a folder, not a single file). This starts
faster than `--onefile` and avoids the antivirus false-positives that a
single-file RDKit bundle often triggers on first launch.

---

## Usage

A typical end-to-end workflow moves left to right across the panels:

1. **3D Downloader** — type an identifier (e.g. `2244`, `CHEMBL25`,
   `PLANT_Withania somnifera`) or load a `.txt` list. Click the **(i)** icon for
   the full list of accepted formats, or download a sample input file.
2. **Batch Converter** — run *Convert 2D to 3D* on anything downloaded flat
   (ChEMBL, ChEBI, CompTox), then *Extract Canonical SMILES* to produce a
   combined SMILES file.
3. **ADMET Predictor** — feed that SMILES file in and save the descriptor
   workbook.
4. **Analytics** — load the workbook and explore the eight plots; export the
   ones you need as PDF/TIFF.
5. **Screening** — narrow a structure folder down to molecules resembling a
   reference drug.
6. **Molecular Docking** — select a receptor PDBQT and a ligand folder, set the
   grid box, and run. The best affinities are compiled into
   `Vina_Binding_Affinities.xlsx`, ranked strongest-first.

---

## Output files

All outputs are written to the current working directory (where you launch the
app) and its sub-folders:

```
<Source> Compounds/
    SDF Files/  MOL2 Files/  PDBQT Files/
    Molecular Properties/<ID>_property.txt
    Molecular Properties.txt          (combined, per folder)
    Log Files/
Molecular Properties.xlsx             (one aggregated workbook per run)

<ligand folder>/Vina_Results/
    <ligand>_out.pdbqt, <ligand>_log.txt
    Vina_Binding_Affinities.xlsx      (ligand + affinity, ranked ascending)
```

On completion, each panel names its output folder and lists any items that
failed (also written to `failed_items.txt`).

---

## Troubleshooting

**"No convertible files found" / conversions all fail / docking does nothing.**
Almost always Open Babel or Vina is not on your PATH. Open a terminal and run
`obabel -V` and `vina --version`. If either is "not recognised", the tool is
not installed or not on PATH.

**Adding a tool to PATH on Windows.**
Settings → *Edit the system environment variables* → *Environment Variables* →
select **Path** → *Edit* → *New* → paste the folder containing `obabel.exe`
(typically `C:\Program Files\OpenBabel-3.1.1\`) or `vina.exe`. Restart 3dME
afterwards.

**Windows SmartScreen / antivirus warns on first launch.**
The executable is unsigned. Choose *More info → Run anyway*, or build it
yourself with the spec provided.

**Downloads fail for a specific database.**
Some endpoints (ZINC, DrugBank) are less reliable than others; the completion
dialog and `failed_items.txt` list exactly which IDs failed so you can retry.

---

## Known limitations

- **Linux/macOS need one code change before they will work.** The external
  tools are invoked with `subprocess` using `shell=True` **and** a list of
  arguments. On Windows this works; on POSIX it runs only the first token and
  drops the arguments, so conversions silently no-op. Fixing this (passing the
  command as a list **without** `shell=True`) is required for a Linux build and
  is on the roadmap. Open Babel and Vina themselves are fully cross-platform.
- A handful of database routes are best-effort: the DrugBank route resolves
  through PubChem and can 404, and the ZINC15 fallback endpoint is retired.
- The Molecular Docking panel does not prepare receptors; use AutoDockTools or
  Meeko for that.

---

## Documentation

A full technical document — design rationale, data sources, algorithms and a
per-panel walkthrough — is in
[`docs/3dME_Technical_Documentation.pdf`](docs/).

---

## License & citation

Released under the [MIT License](LICENSE).

3dME builds on open-source scientific software. If you use it in published
work, please cite the underlying tools:

- **RDKit** — G. Landrum et al., *RDKit: Open-source cheminformatics.*
- **Open Babel** — N. M. O'Boyle et al., *Open Babel: An open chemical
  toolbox*, J. Cheminform. 2011, 3:33.
- **AutoDock Vina** — J. Eberhardt, D. Santos-Martins, A. F. Tillack, S. Forli,
  *AutoDock Vina 1.2.0*, J. Chem. Inf. Model. 2021; and O. Trott, A. J. Olson,
  *AutoDock Vina*, J. Comput. Chem. 2010.
- **BOILED-Egg model** — A. Daina, O. Zoete, *ChemMedChem* 2016, 11, 1117.

---

_Maintained by the Mathematical and Computational Biology Lab (MCBL)._
