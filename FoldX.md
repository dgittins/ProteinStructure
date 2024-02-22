# FoldX

Predict the thermal stability of a protein structure

FoldX paper - https://academic.oup.com/bioinformatics/article/35/20/4168/5381539

FoldX manual - https://foldxsuite.crg.eu/documentation#manual

1. Download the latest FoldX software from https://foldxsuite.crg.eu/

Requires registration for an academic license

Copy the downloaded version (foldx5MacC11) to a working directory

```
cd /Users/dgittins/foldx5MacC11

./foldx_20241231 --help
```

2. Transfer a .pdb structure to the working directory

3. Run a command to calculate the thermal stability of a structure

Stability of object the FoldX ‘Stability’ command calculates the free energy of unfolding (Δ G). This is the difference in free energy between the folded and the unfolded states. All energies resulting from the plugin are expressed in kcal/mol.

```
./foldx_20241231 --command=Stability --pdb=test_relaxed_rank_001_alphafold2_ptm_model_4_seed_000.pdb

for f in *.pdb; do
  ./foldx_20241231 --command=Stability --pdb=$f > $f.foldx.out
done
```

Output:
```
   ********************************************
   ***                                      ***
   ***             FoldX 5 (c)            ***
   ***                                      ***
   ***     code by the FoldX Consortium     ***
   ***                                      ***
   ***     Jesper Borg, Frederic Rousseau   ***
   ***    Joost Schymkowitz, Luis Serrano   ***
   ***    Peter Vanhee, Erik Verschueren    ***
   ***     Lies Baeten, Javier Delgado      ***
   ***       and Francois Stricher          ***
   *** and any other of the 9! permutations ***
   ***   based on an original concept by    ***
   ***   Raphael Guerois and Luis Serrano   ***
   ********************************************

 *** THIS IS AN IMPORTANT MESSAGE: 
 >> External parametrized molecules were detected...
 >> Final energetic values will depend of the correct parametrization of this molecules. 


Stability >>>

1 models read: test_relaxed_rank_001_alphafold2_ptm_model_4_seed_000.pdb

BackHbond       =               -434.89
SideHbond       =               -94.31
Energy_VdW      =               -711.77
Electro         =               -31.40
Energy_SolvP    =               998.80
Energy_SolvH    =               -941.50
Energy_vdwclash =               49.76
energy_torsion  =               5.74
backbone_vdwclash=              456.86
Entropy_sidec   =               361.90
Entropy_mainc   =               953.62
water bonds     =               0.00
helix dipole    =               -7.12
loop_entropy    =               0.00
cis_bond        =               2.25
disulfide       =               0.00
kn electrostatic=               0.00
partial covalent interactions = 0.00
Energy_Ionisation =             1.41
Entropy Complex =               0.00
-----------------------------------------------------------
Total          = 				  152.49

FINISHING STABILITY ANALYSIS OPTION

Your file run OK
End time of FoldX: Thu Feb 22 13:52:51 2024
Total time spend: 0.94 seconds.
```

4. View the results of multiple runs

Total Energy - This is the predicted overall stability of your protein.

In general, a lower total energy value implies that the protein structure is more stable. Conversely, a higher total energy suggests less stability.

```
for f in *.pdb.foldx.out; do
  echo $f
  grep "Total          =" $f
done
```
