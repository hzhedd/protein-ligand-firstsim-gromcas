[![GROMACS](https://img.shields.io/badge/GROMACS-2022%2B-blue?style=flat&logo=gromacs)](https://www.gromacs.org/)
[![ForceField](https://img.shields.io/badge/Force%20Field-CHARMM36-green)](https://mackerell.umaryland.edu/charmm_ff.shtml)
[![OS](https://img.shields.io/badge/OS-Linux-orange?style=flat&logo=linux)](https://www.linux.org/)
[![Status](https://img.shields.io/badge/Status-Work%20in%20Progress-yellow)](#)

# protein-ligand-firstsim-gromacs

## Target System Overview

* **Protein / Receptor:** T4 lysozyme (L99A/M102Q mutant)
* **Ligand:** 2-ethoxyphenol (2EP)
* **PDB ID:** 3HU8
* **Structure Source:** [RCSB PDB - 3HU8](https://www.rcsb.org/structure/3HU8)

  ## Repository Structure

| File / Folder | Description |
| :--- | :--- |
| `3HU8.pdb` | Raw crystal structure of T4 lysozyme mutant with ligand |
| `2EP.pdb` / `2EP.mol2` | Extracted and protonated ligand structure files |
| `charmm36-jul2022.ff/` | CHARMM36 force field directory |
| `topol.top` | System topology file |

## Prerequisites & Force Field

Before preparing the structures, the required force field files and conversion script were downloaded into the working directory:
* **Force Field:** CHARMM36 all-atom force field (`charmm36-jul2022.ff`) — [MacKerell Lab Force Fields](https://mackerell.umaryland.edu/charmm_ff.shtml#gromacs)
* **Conversion Script:** `cgenff_charmm2gmx_py3_nx2.py` — [MacKerell Lab CGenFF Script](https://mackerell.umaryland.edu/charmm_ff.shtml#gromacs)

 ## Step 1: Structure Editing & Component Separation

The raw PDB file (`3HU8.pdb`) was manually processed in a text editor to separate the complex into clean individual components:
* **Ligand Extraction (`2EP.pdb`):**
  Extracted atoms `1343` to `1352` (Residue `261`, containing the `2-ethoxyphenol` heavy atoms `CAA` through `OAB`) and saved them as the independent ligand file `2EP.pdb`.
* **Non-Essential Heteroatoms & Water Removal:**
  Deleted crystallographic water molecules (`HOH` starting from residue `168`), phosphate ions (`PO4`), and trailing `CONECT` records directly from `3HU8.pdb` to prepare a clean receptor backbone.
  
 ## Step 2: Protein Topology Generation

GROMACS was used to process the clean protein structure, assign force field parameters, and generate the system topology:
```bash
gmx pdb2gmx -f 3hu8.pdb -o 3hu8_processed.gro -ter
```
   ### Interactive Prompt Choices (Post-Command Selection)
Upon executing the `gmx pdb2gmx` command, the following choices were selected in the interactive terminal prompts:
1. **Force Field Selection:**
   Chosen Option: `CHARMM36 all-atom force field (July 2022)`
   *Provides accurate CHARMM force field parameters for the protein backbone and side chains.*
2. **Water Model Selection:**
   Chosen Option: `TIP3P`
   *Sets the explicit solvent water model to TIP3P, compatible with the CHARMM36 parameter set.*
3. **Termini Interactive Assignment (`-ter`):**
   Chosen Option: `MET1`
   *Explicitly set the N-terminus residue starting at `MET1` to determine the protonation state at the start of the protein chain.*
   ### Output Files
 - `3hu8_processed.gro`: Processed protein structure file.
 - `topol.top`: System topology file.
 - `posre.itp`: Position restraint file for the protein backbone.

  ## Step 3: Ligand Preparation & Topology Generation

### 3.1 Open Babel Installation & Initial MOL2 Conversion
Tool Setup: Download and install Open Babel via the official release page (https://openbabel.org/wiki/Category:Installation) or install directly via terminal (`sudo apt install openbabel`).
Convert the `2EP.pdb` file to `2ep.mol2` (thru UI)
* input: `2EP.pdb` (extracted raw ligand coordinates)
* draft output: `2ep.mol2` (initial protonated structure requiring manual inspection)

### 3.2 Manual Structure & MOL2 Formatting Correction
Because Open Babel output contained formatting discrepancies that prevent proper parsing on the CGenFF server, `2EP.mol2` was manually corrected in a text editor:

- **Residue ID Normalization:** Replaced mismatched residue IDs (`167`) assigned to the newly added hydrogens with a uniform ID (`1`) across all 20 atoms.
- **Residue Name Standardization:** Replaced generic numerical values (`261167`) in the substructure column with the proper 3-letter ligand identifier (`2EP`).
- **Output:** Saved the clean, CGenFF-ready file - `2EP_fix.mol2`.
  
### 3.3 Bond Sorting with Perl Script
To fix atom bond ordering issues caused by Open Babel and ensure CGenFF web server compatibility, process the file using Lemkul's bond sorting script (sort_mol2_bonds.pl):
```bash
perl sort_mol2_bonds.pl 2EP.mol2 2ep_fix.mol2
```
* Input: 2EP.mol2 (initial MOL2 output from Open Babel)
* Script: sort_mol2_bonds.pl (Perl utility script that reorders bond indices sequentially)
* Output: 2ep_fix.mol2 (bond-sorted MOL2 file ready for CGenFF topology upload)
  
### 3.4 CGenFF Parameter & GROMACS Topology Generation
Uploaded the corrected `2EP.mol2` file to the official [CGenFF Web Server](https://cgenff.umaryland.edu/) to obtain the CHARMM force field stream file (`2ep_fix.str`). 

### 3.5 Environment & Dependency Configuration
Before executing the conversion script in modern Linux/WSL environments (Python 3.12+), configure compatible package dependencies to prevent ImportError runtime crashes:
```bash
pip install numpy --break-system-packages
pip install "networkx>=2.8,<3.0" --break-system-packages
```
Dependency Note: Python 3.12 removed "gcd" from "fractions", breaking legacy networkx==2.3. Installing networkx 2.8+ resolves this compatibility issue without breaking the CGenFF script parser.

### 3.6 GROMACS Topology Conversion
Execute the conversion script to parse the stream file into GROMACS-compatible topology format:
```bash
python3 cgenff_charmm2gmx_py3_nx2.py 2EP 2ep_fix.mol2 2EP.str charmm36-jul2022.ff
```
* Input Files:
- `2EP`: The 3-letter target residue identifier
- `2ep_fix.mol2:` Clean, bond-sorted ligand MOL2 structure file
- `2ep_fix.str:` Parameter stream file downloaded from CGenFF server
- `charmm36-jul2022.ff:` Active CHARMM36 force field directory containing forcefield.doc
* Generated Output Files:
- `2ep.itp:` Ligand molecule topology containing [ moleculetype ], [ atoms ], [ bonds ], [ pairs ], [ angles ], and [ dihedrals ]
- `2ep.prm:` Additional force field parameters not natively included in CHARMM36 (may be empty if all parameters are already present in standard force field tables)  
- `2ep.top`: Standalone ligand topology file.
- `2ep_ini.pdb`: Re-ordered ligand coordinate reference file matched to the generated topology.


  ## Step 4: Complex Structure Assembly

### 4.1 Coordinate Conversion & Merging
Converted the re-ordered ligand coordinate file (`2ep_ini.pdb`) generated by CGenFF into GROMACS structure format (`2ep.gro`):

```bash
gmx editconf -f 2ep_ini.pdb -o 2ep.gro
```
### 4.2 Coordinate Merging (`complex.gro`)
Assembled the protein-ligand complex by combining the ligand coordinates (`2ep.gro`) into the processed protein structure file (`protein.gro`):

1. **Insertion:** Appended the 20 ligand atom coordinate lines directly after the protein atoms and immediately before the final box vectors line in `complex.gro`.
2. **Atom Count Update:** Incremented the total system atom count on line 2 from `2600` to `2620` to account for the added ligand.

- **Input Files:** `protein.gro` (2600 atoms) + `2ep.gro` (20 atoms)
- **Output File:** `complex.gro` (2620 total atoms)
  
 ### 4.3 System Topology Update (`topol.top`)
Updated `topol.top` to integrate the ligand's force field parameters and moleculetype definitions by inserting the following directives:

1. **Ligand Force Field Parameters:** Inserted directly after the force field import:
   ```text
   ; Include forcefield parameters
   #include "./charmm36-jul2022.ff/forcefield.itp"

   ;include ligand parameters
   #include 2ep.prm
   ```
2. **Ligand Topology Definition:** Inserted directly after the position restraint block:
   ```text
   #endif

   ;include ligand topology
   #include 2ep.itp
   ```
3. **Molecule Registry:** Appended under the [ molecules ] directive:
   ```text
   [ molecules ]
   ; Compound        #mols
   Protein_chain_A     1
   2EP                 1
   ```
   ## Step 5: Solvation and Charge Neutralization

### 5.1 Simulation Box Definition
Defined a rhombic dodecahedron simulation box around the complex structure:

```bash
gmx editconf -f complex.gro -o newbox.gro -bt dodecahedron -d 1.0
```
- **Input:** `complex.gro` (2620 atoms)
- **Output:** `newbox.gro`
- **Box Type:** Rhombic dodecahedron (-bt dodecahedron)
- **Solute-Box Distance:** Minimum 1.0 nm (-d 1.0)

### 5.2 System Solvation
Solvated the simulation box using the 3-point water model and updated `topol.top`:

```bash
gmx solvate -cp newbox.gro -cs spc216.gro -p topol.top -o solv.gro
```
- **Input Configuration:** `newbox.gro`
- **Water Model:** spc216.gro (TIP3P configuration)
- **Output File:** `solv.gro ` 
