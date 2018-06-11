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
The Pharmit searches for your query in large compound databases, and provides an "sdf" file that you can download to your computer. (The file would probably be a zipped sdf file. You do not need to unzip that file since the Schrödinger software can read also zipped sdf files).

## 2. Converting "sdf" files into "mae" files
- First, we need to log in HPCC and ask for 1 node with 28 processors in HPCC:

`qsub -I -l nodes=1:ppn=28 -l walltime=168:00:00 -l mem=256gb`

- Set environmental variables in your home folder. Please enter below commands:

`module load schrodinger`

`export schrodinger=/opt/software/modulefiles/binaries/schrodinger/suite2016-2.lua`


- Make your all shell scripts executable using the command: 

`chmod +x *.sh`

- Make necessary changes in "convertsdf.sh" file (like input and output file names), and run your shell script:

`nohup ./convertsdf.sh &` 

## 3. Quickprop

- To calculate ADME properties run "qikpropstep.sh" script (make necessary changes like input and output file names):

`nohup ./qikpropstep.sh &` 

## 4. Filtering

Here we have three criteria for filtering:
i)   Compounds with molecular weight greater than 225 and less than 550
ii)  The Log P  0<xx<4.5
iii) Polar surface area 40<xx<170 square angstroms

- Make necessary changes like input and output file names, and run your shell script:

`nohup ./filteringstep.sh &` 

## 5. Glide Docking 
### 5.1 Receptor Preparation:

- Download protein (receptor) PDB file from RCSB website. Remove unwanted molecules (ligands, ions etc.) or residues. If necessary, create the tertiary structure (for dimer, trimer etc., PISA website would be helpful). 

- Copy your protein structure to a folder in your folder on HPCC. Enter the command given below to convert ".pdb" file into ".mae" file (Maestro):

`$SCHRODINGER/utilities/prepwizard -WAIT -fix receptor.pdb receptor_prep.mae`  


### 5.2. Ligand Preparation:

- Download ligand structure data file (".sdf") from RCSB website using PDB ID of the receptor. Check how many molecules there are. Remove any unwanted copies. Use one of the commands below: 

`$SCHRODINGER/ligprep –WAIT -i 2 -r 1 -s 4 -t 4 -isd ligand.sdf -omae ligprep.mae`

`$SCHRODINGER/ligprep –WAIT –W e,-ph,7.0,-pht,2.0 –epik –i 1 –r 1 –nz –bff 14 –ac –isd ligand.sdf –omae ligprep.mae`


### 5.3. Grid Generation:

- We need to extract the ligand coordinates from the original pdb file. Check the three-letter name of the ligand in the pdb file and use this name to grep coordinates and calculate the average coordinates: Here we assume it is "LIG":

`grep "HETATM" protein.pdb| grep "LIG" | awk '{sumx+=$7; sumy+=$8; sumz+=$9; print sumx/NR, sumy/NR, sumz/NR}'` 

- Open "grid.in" file and enter correct GRID_CENTER x,y,z coordinates (also correct RECEP_FILE name).

`$SCHRODINGER/glide -WAIT grid.in`

### 5.4. Job Submission:

- Open "grid_htvs.in" file and make  necessart changes for the GRIDFILE and LIGANDFILE names, or PRECISION* method that you would like to use.

*PRECISION methods:
1- HTVS: High Throughput Virtual Screening that is a very fast evaluation method for very large libraries.
2- SP: Standard-Precision that is a quick way to evaluate the poses. SP is also used for high-throughput virtual screening of big libraries.
3- XP: Extra-Precision that is a refinement tool used only for very small libraries or a good subset of big library that has been pre-evaluated with HTVS or SP score, because it needs more computational time.

- Submit docking job:

`/opt/software/schrodinger/suite_2016-2/glide -WAIT -LOCAL -HOST localhost:25 glide_rgs_htvs.in`


### 6. Analysis of docking results

..........

EXTRAS:
Token: MSU has 25 tokens for Schrödinger Software. Some modules or packages require more than one token like Glide docking needs 5 for each submission.
How to check license usage amounts in HPCC: `lmstat -a -c 27008@lm-01.i`


