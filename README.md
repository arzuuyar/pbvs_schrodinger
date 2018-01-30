# Schrödinger Tutorial for Pharmacophore-Based Virtual Screening
There are 6 steps in this workflow:

1. Getting "sdf" file(s)
2. Converting "sdf" files into "mae" files
3. Quickprop
4. Filtering
5. Docking
6. Analysis of docking results

## 1. Getting "sdf" file(s)
- We will do pharmacophore-based virtual screening, so we need to find ligands having similar pharmacophore features. To do that, we will use Pharmit online tool (http://pharmit.csb.pitt.edu). 
The Pharmit searches for your query in large compound databases, and provides an "sdf" file that you can download to your own computer.
The file would probably a zipped sdf file. You do not need to unzip that file since the Schrödinger software can read also zipped sdf files.

## 2. Converting "sdf" files into "mae" files
- First, we need to log in HPCC and set environmental variables in home folder. Please enter below commands:

module load schrodinger
export schrodinger=/opt/software/modulefiles/binaries/schrodinger/suite2016-2.lua

- You can check if the Schrödinger environment is set correctly:

$SCHRODINGER

This command would give you "  " meaning that it is correctly set.

- Ask for 1 node with 28 processors in HPCC:

qsub -I -l nodes=1:ppn=28 -l walltime=168:00:00 -l mem=256gb

- Make a shell script having two lines given below and name the file as "sdfconvert.sh".

#!/bin/bash                                                                                                                                                                     
/opt/software/schrodinger/suite_2016-2/ligprep -LOCAL -HOST localhost:28 -i 2 -r 1 -s 4 -t 4 -isd file.sdf -omae outputfile.mae

- Make the script executable using the command: 

chmod +x convertsdf.sh

- Run your shell script:

nohup ./convertsdf.sh & 

## 3. Quickprop

- Make a script having two lines given below and name the file as "qikpropstep.sh"

#!/bin/bash                                                                                                                                                                     
/opt/software/schrodinger/suite_2016-2/qikprop -LOCAL -nosa -nosim  outputfile.mae

- Make the script executable using the command: 

chmod +x qikpropstep.sh

- Run your shell script:

nohup ./qikpropstep.sh & 

## 4. Filtering

Here we have three criteria for filtering:
i)   Compounds with molecular weight less than 225 and greater than 550
ii)  The Log P  0<xx<4.5
iii) Polar surface area 40<xx<170 square angstroms

#!/bin/bash                                                                                                                                                                     
/opt/software/schrodinger/suite_2016-2/utilities/ligfilter -LOCAL  -e "r_qp_mol_MW >= 225" -e "r_qp_mol_MW <= 550" -e "r_qp_QPlogPo/w > 0" -e "r_qp_QPlogPo/w < 4.5" -e "r_qp_FISA > 40" -e "r_qp_FISA < 170" -o filter_outputfile-out.mae outputfile-out.mae

## 5. Glide Docking 
### 5.1 Receptor Preparation:

- Download protein (receptor) PDB file from RCSB website. Remove unwanted molecules (ligands, ions etc.) or residues. If necessary, create the tertiary structure (for dimer, trimer etc., PISA website would be helpful). 

- Copy your protein structure to a folder in your folder on HPCC. Enter the command given below to convert ".pdb" file into ".mae" file (Maestro):

$SCHRODINGER/utilities/prepwizard -WAIT -fix receptor.pdb rec_prep.mae  


### 5.2. Ligand Preparation:

- Download ligand structure data file (".sdf") from RCSB website using PDB ID of the receptor. Check how many molecules there are. Remove any unwanted copies. Use one of the commands below: 

$SCHRODINGER/ligprep –WAIT -i 2 -r 1 -s 4 -t 4 -isd ligand.sdf -omae ligprep.mae

$SCHRODINGER/ligprep –WAIT –W e,-ph,7.0,-pht,2.0 –epik –i 1 –r 1 –nz –bff 14 –ac –isd ligand.sdf –omae ligprep.mae


### 5.3. Grid Generation:

- We need to extract the ligand coordinates from the original pdb file. Check the three-letter name of the ligand in the pdb file and use this name to grep coordinates and calculate the average coordinates: Here we assume it is "LIG":

grep HETATM protein.pdb| grep “LIG” | awk '{sumx+=$7; sumy+=$8; sumz+=$9; print sumx/NR, sumy/NR, sumz/NR}' 

- Open a new file and name it as "grid.in". Copy the lines below into "grid.in" file. Rename your receptor.mae file and paste correct GRID_CENTER x,y,z coordinates 

USECOMPMAE YES
INNERBOX 10, 10, 10
ACTXRANGE 30.000000
ACTYRANGE 30.000000
ACTZRANGE 30.000000
GRID_CENTER 9.940000, -8.310000, -11.660000
OUTERBOX 30.000000, 30.000000, 30.000000
ENTRYTITLE 3TZR
GRIDFILE grid.zip
RECEP_FILE 3TZR_prepared.mae


### 5.4. Job Submission:

- Make a file that has these lines below (check file GRIDFILE and LIGANDFILE names, and enter which PRECISION* method you would like to use):

WRITEREPT YES
USECOMPMAE YES
POSTDOCK_NPOSE 10
MAXREF 800
RINGCONFCUT 2.500000
GRIDFILE grid.zip
LIGANDFILE  ligprep.mae
PRECISION HTVS


*PRECISION methods:
1- HTVS: High Throughput Virtual Screening that is a very fast evaluation method for very large libraries.
2- SP: Standard-Precision that is a quick way to evaluate the poses. SP is also used for high-throughput virtual screening of big libraries.
3- XP: Extra-Precision that is a refinement tool used only for very small libraries or a good subset of big library that has been pre-evaluated with HTVS or SP score, because it needs more computational time.


$SCHRODINGER/glide -WAIT —LOCAL NJOBS 25 glide_rgs_htvs.in

### 6. Analysis of docking results



EXTRAS:
Token: MSU has 25 tokens for Schrödinger Software. Some modules or packages require more than one token like Glide docking needs 5 for each submission.
How to check license usage amounts in HPCC: lmstat -a -c 27008@lm-01.i


