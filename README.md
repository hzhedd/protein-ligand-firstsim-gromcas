# protein-ligand-firstsim-gromcas

## Target System Overview

-*Protein / Receptor*: T4 lysozyme (L99A/M102Q mutant)
-*Ligand*: 2-ethoxyphenol (JZ4)
-*PDB ID*: 3HU8
-*Structure Source*: [RCSB PDB - 3HU8](https://www.rcsb.org/structure/3HU8)

## Prerequisites & Force Field

Before preparing the structures, the required force field files and conversion script were downloaded into the working directory:
**Force Field:** CHARMM36 all-atom force field (`charmm36-jul2022.ff`) — [MacKerell Lab Force Fields](https://mackerell.umaryland.edu/charmm_ff.shtml#gromacs)
**Conversion Script:** `cgenff_charmm2gmx_py3_nx2.py` — [MacKerell Lab CGenFF Script](https://mackerell.umaryland.edu/charmm_ff.shtml#gromacs)

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
   Chosen Option: `CHARMM36 all-atom force field (july 2022)`
   *Provides accurate CHARMM force field parameters for the protein backbone and side chains.*
2. **Water Model Selection:**
   Chosen Option: `TIP3P`
   *Sets the explicit solvent water model to TIP3P, compatible with the CHARMM36 parameter set.*
3. **Termini Interactive Assignment (`-ter`):**
   Chosen Option: `MET1`
   *Explicitly set the N-terminus residue starting at `MET1` to determine the protonation state at the start of the protein chain.*
   ### Output Files
* `3hu8_processed.gro`: Processed protein structure file.
* `topol.top`: System topology file.
* `posre.itp`: Position restraint file for the protein backbone.



