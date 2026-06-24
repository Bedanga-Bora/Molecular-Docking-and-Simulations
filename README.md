# Molecular-Docking-and-Simulations
This is made for proper understanding and the files to understand the basis of MD Simulations on AChE.
# How to utilize it in step by step
## No 1: This can be seen that this whole setup contains only a couple files. Trying to Dock is the first thing to utilize.
# Molecular Docking and Simulations: AChE Inhibitors

This repository provides an automated workflow for screening chemical libraries, identifying high-affinity lead compounds, and evaluating their structural stability against Acetylcholinesterase (AChE) using Molecular Dynamics (MD) simulations.

---

## Repository Architecture & "Stuffs" Explained

| File / Folder | Description |
| :--- | :--- |
| **`4EY7_clean.pdb`** | The processed, clean crystal structure of target protein Acetylcholinesterase (AChE), optimized for docking. |
| **`Molecules.tar.xz`** | A high-ratio compressed archive containing the input ligand library/structures for screening. |
| **`Virtual_Screening_BBB.ipynb`** | Jupyter notebook executing the virtual screening workflow, factoring in Blood-Brain Barrier (BBB) permeation filters. |
| **`ligand_92.acpype/`** | Topology parameters and coordinate files generated via ACPYPE to make the top ligand compatible with GROMACS. |
| **`top_ache_hits.zip`** | Packed archive containing configuration scripts, plotted stability datasets (`.xvg`), and PyMOL analytical environments. |
| **`Docked/`** | Target directory where successful docking poses and structural output configurations are preserved. |

### Why `.tar.xz`?
Molecular database libraries contain thousands of lines of coordinate geometries (such as `.sdf` or `.pdbqt`), which can quickly hit GitHub's individual file hosting thresholds. The `.tar.xz` format wraps these files into a single tape archive (`.tar`) and uses **LZMA2 compression** (`.xz`) to compress the dataset tightly without losing a single line of structural data.

---

## Prerequisites & Setup

Ensure you have a Linux environment (Ubuntu/WSL2 preferred) with Conda, GROMACS, and PyMOL installed.

# 1. Extract the Core Data
Before executing any scripts, unpack the molecule libraries and project archives:

**in your terminal or Bash, Utilize this**

# Extract the compressed ligand library
tar -xf Molecules.tar.xz

# Unzip the screening results and simulation tracking files
unzip top_ache_hits.zip -d top_ache_hits/

# Phase 1: Virtual Screening & Selection

    Launch the Virtual_Screening_BBB.ipynb notebook in Google Colab or your local Jupyter instance.

    Run the filtering sequences to pass the ligand library through ADME/BBB criteria.

    Execute the high-throughput molecular docking scripts against 4EY7_clean.pdb.

    Output files will gather inside the Docked/ folder, identifying the top candidate hits (e.g., ligand_92).

# Phase 2: Topology Generation

To simulate the ligand in GROMACS, generate its force field topology compatible with AMBER/CHARMM using ACPYPE:

acpype -i ./Docked/ligand_92.sdf -c bcc -n 0

Phase 3: Molecular Dynamics Simulation (GROMACS)

Inside your working directory, run the command architecture to build the system topology, add water solvents, neutralize ions, and execute production steps:
Bash

# 1. Assemble the complex topology (.tpr)
gmx grompp -f nvt_production.mdp -c reference.gro -p topol.top -o nvt_production.tpr

# 2. Execute the production simulation engine (e.g., 100 ps trajectory run)
gmx mdrun -v -deffnm nvt_production -notunepme -nt 8 -pin on

Phase 4: Structural Stability Analysis

Process your trajectory coordinate files (nvt_fit.xtc) relative to your initial topology setup to derive tracking metrics:
Bash

# Calculate Protein Backbone RMSD
gmx rms -s nvt_production.tpr -f nvt_fit.xtc -o rmsd.xvg -tu ps

# Calculate Target Ligand RMSD
gmx rms -s nvt_production.tpr -f nvt_fit.xtc -o rmsd_ligand.xvg -tu ps

# Monitor minimum atomic distance and contact matrices
gmx mindist -s nvt_production.tpr -f nvt_fit.xtc -on num_contacts.xvg -d 0.35

# Visualizing Results

The tracked trajectories and structural alignment curves can be instantly examined using the included PyMOL script templates:
Bash

pymol reference.gro nvt_fit.xtc visualize_hit.pml

# if it fails to run, force Qt to Use the X11 Backend
export QT_QPA_PLATFORM=xcb
pymol reference.gro nvt_fit.xtc
