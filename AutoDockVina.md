# AutoDock Vina

https://autodock-vina.readthedocs.io/en/latest/introduction.html
https://github.com/ccsb-scripps/AutoDock-Vina/tree/develop

```
cd /home/structure/docking
```

Install using Conda

```
conda create -n autodockvina python=3
conda activate autodockvina
conda config --env --add channels conda-forge

conda install -c bioconda vina 
```

Check install
```
vina --help
```

Install AFDR suite
(software tools for automated docking and peripheral tasks)
```
conda install hcc::adfr-suite
```

Install meeko
(tools covering other docking aspects not handled by the ADFR software suite)
conda install -c conda-forge numpy openbabel scipy rdkit
```
pip install meeko
```

1. Prepare the receptor
```
prepare_receptor -h
prepare_receptor -r 1iep_receptorH.pdb -o 1iep_receptor.pdbqt
```

2. Prepare the ligand
```
mk_prepare_ligand.py --help
mk_prepare_ligand.py -i 1iep_ligand.sdf -o 1iep_ligand.pdbqt
```

3. Generate affinity maps for AutoDock FF
(define the grid space for the docking, typically, a 3D box around a the potential binding site of a receptor) - copy script from GitHub
```
emacs prepare_gpf.py
pythonsh ./prepare_gpf.py -l 1iep_ligand.pdbqt -r 1iep_receptor.pdbqt -y
less 1iep_receptor.gpf
```

generate the different map files that will be used for the molecular docking
```
autogrid4 -p 1iep_receptor.gpf -l 1iep.glg
ls
```

4. Run AutoDock Vina

4a. Using AutoDock4 forcefield
(requires affinity maps)
```
vina --ligand 1iep_ligand.pdbqt --maps 1iep_receptor --scoring ad4 --exhaustiveness 32 --out 1iep_ligand_ad4_out.pdbqt --cpu 10 > AutoDock4_forcefield.out
# use '> AutoDock4_forcefield.out' to write the results to a file
```

4b. Using Vina forcefield
(does not require affinity maps)
```
emacs 1iep_receptor_vina_box.txt

center_x = 15.190
center_y = 53.903
center_z = 16.917
size_x = 20.0
size_y = 20.0
size_z = 20.0

vina --receptor 1iep_receptor.pdbqt --ligand 1iep_ligand.pdbqt --config 1iep_receptor_vina_box.txt --exhaustiveness=32 --out 1iep_ligand_vina_out.pdbqt --cpu 10 > Vina_forcefield.out
```

5. Results

5a. Using AutoDock forcefield
```
less AutoDock4_forcefield.out
```

5b. Using Vina forcefield
```
less Vina_forcefield.out
```
