﻿![enter image description here](https://i.imgur.com/MIALh25.png)
**Molecular Dynamics Simulation and Analysis of Interaction Between the ZO-1 PDZ1 Domain and Claudin-5**

Roshan Pathak (1,*)


**Overview**

This is a step-by-step document to journal the process of setting up and running this specific MD simulation. It follows the common structure of most MD simulation experiments: (1) structure and simulation preparation, (2) energy minimization and analysis, (3) heating, (4) equilibration, (5) MD production run.

**Structure Preparation**

This simulation was limited mostly by the dearth of structures available for ZO-1 and human Claudin-5. ZO-1 has many largely disordered regions, although its binding domains (PDZs 1-3, SH3, GUK, U5, and U6) seem to be conserved. The original plan was to utilize a structure including the F-actin filament binding C-terminus of ZO-1 as well as one of these domains. However, such a structure is not available to my knowledge. A chimeric structure could be made in the future, but its structural accuracy would be questionable. As a result, this simulation uses a crystal structure of the ZO-1 PDZ1 binding domain (RCSB: 2H3M) with a resolution of 2.90 Å.

For claudin-5, there are no publicly available structures, so an AlphaFold prediction of the structure was made for the purposes of MD. However, the pLDDT value for much of the intracellular tail (where ZO-1 most likely binds) is <70, limiting the accuracy of the structure in its initial configuration.

**Docking and Solvation**

In order to get a reasonable initial configuration of the two molecules relative to each other, protein-protein docking was used. The HADDOCK 2.40 webserver was used to determine an initial docking configuration. The C-terminal tail of claudin-5 and the entirety of the PDZ1 structure were used as active residues, and a pose (Cluster 3, pose 1) was chosen based on the HADDOCK Score (9.8), the Van der Waals energy, and visual inspection.

After determining this pose, the PDB file was imported to VMD. In VMD, a bond was found between a Claudin-5 residue and a ZO-1 residue, likely due to HADDOCK merging the two initial structures into one PDB file. This bond was removed and the PDZ1 domain was pulled slightly apart from the Claudin-5 tail to prevent bad contacts in the MD simulation.

To solvate this structure, CHARMM-GUI was used to make a TIP3 waterbox, using the structure dimensions to make the waterbox and solvating the box with 0.15M NaCl.

This solvated structure was imported back into VMD and a PSF structure file was generated for later use. ![Image of the system rendered in ChimeraX](https://i.imgur.com/i5blCQI.png)

**Preparation of Configuration Files**

The contents of all configuration files are in this document. However, a few things were determined before running any of the configuration files. First, the periodic boundary conditions of the simulation had to be determined. Periodic boundary conditions are the dimensions chosen for approximating a large system (like the solvated waterbox in this simulation) using a subset or unit cell. The unit cell was determined using a `minmax.txt` script (https://www.ks.uiuc.edu/Research/namd/mailinglist/namd-l.2012-2013/1496.html):

    proc get\_cell {{molid top}} {
    set all [atomselect $molid all]
    set minmax [measure minmax $all]
    set vec [vecsub [lindex $minmax 1] [lindex $minmax 0]]
    puts "cellBasisVector1 [lindex $vec 0] 0 0"
    puts "cellBasisVector2 0 [lindex $vec 1] 0"
    puts "cellBasisVector3 0 0 [lindex $vec 2]"
    set center [measure center $all]
    puts "cellOrigin $center"
    $all delete
    }

The dimensions from this script were inputted into the configuration files for NAMD.

For all subsequent steps, the configuration files were altered versions of the configuration files from the lab of Dr. Tamal Banerjee (IIT Guwhati).

Additionally, CHARMM36 force fields and parameters were downloaded and are loaded in all subsequent configuration files. CHARMM topology files were used to generate the PSF file of the solvated and ionized complex.

![Image of the system rendered in VMD, colored beads are Na and Cl ions, water molecules are not visible.](https://i.imgur.com/c1EelUA.png)

**Minimization**

Prior to beginning simulation, the energy minimum of a system must be reached. In order to do this, overlapping atoms (or bad contacts) must be eliminated and a stable conformation of atoms must be reached. For this minimization run, the system was minimized 10,000 times and was then allowed to run for 20,000 timesteps. The timestep used in the configuration file was `timestep = 2.0`, or 2.0 femtoseconds. Thus, the system was minimized and run for 40 picoseconds (40 ps). A constant pressure (1 atm) and temperature (0K) were maintained using LangevinPiston and LangevinDamping respectively. These systems control temperature and pressure by introducing friction to the system and controlling kinetic energy.

To see if energy minimization occurred ,the NAMDPlot program was used to graph the variable TOTAL (total energy) against TS (timesteps). From here, it seems that the system did reach an energy minimum and stabilized at a value of approx. -1143945kcal/mol:
![Energy minimization.](https://i.imgur.com/Msm8uCL.png)

**Heating**

For the heating run, the coordinates from the minimization run were loaded as starting positions. From here, a random velocity was generated for each atom such that they followed a Maxwell distribution and achieved an overall temperature of 0K. The temperature of the system was increased from 0K to 300K in increments of 0.01K. At each increment, the velocities of all atoms were reassigned:

    From heating.namd:
    
    seed 1010 # Random number seed used to generate initial Maxwell distribution ofvelocities
    numsteps 30000 # Number of integration steps
    reassignFreq 1 # Number of steps between reassignment of velocities (e)
    temperature 0 #initial velocity distribution is performed to achieve this initial K temp
    reassignIncr 0.01 # Increment used to adjust temperature during temperature reassignment
    reassignHold $temperature # The value of temperature to be kept after heating is completed

**Equilibration**

Much like minimization, equilibration runs aim to bring the system to the most stable state. Most unequilibrated runs have atoms move and structures expand and contract in extreme ways, which should not be interpreted as viable simulation results.

Additionally, the temperature and pressure were maintained using the same tools as in the heating step (canonical and grand canonical ensembles).

For this equilibration run, the final heat coordinates and extended system files were used as the starting point:

    #Equilibration
    seed 1010 # Random number seed used to generate initial Maxwell distribution of velocities
    numsteps 1000000 # Number of integration steps
    The number of integration steps was also calculated using a timestep of 2.0fs, so a total of 2ns of simulation time will be recorded.

_Currently running equilibration, production run is next._
```
