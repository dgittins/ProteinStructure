## RoseTTAFold All-Atom

1. Clone the package
```
cd ~/bin/
git clone https://github.com/baker-laboratory/RoseTTAFold-All-Atom
cd RoseTTAFold-All-Atom
```

2. Create a conda environment for RoseTTAFold-All-Atom 
```
conda env create -f environment.yaml
conda activate RFAA

cd rf2aa/SE3Transformer/
pip3 install --no-cache-dir -r requirements.txt
python3 setup.py install
cd ../../
```

3. Download a licensed (academic) copy of SignalP - 6.0

- Go to the 'Downloads' tab of https://services.healthtech.dtu.dk/services/SignalP-6.0/
- Select 'Version 6.0h - fast'
- Provide credentials
- Click on the link to the software provided in the subsequent email
- Use 'wget' to download the 'signalp-6.0h.fast.tar.gz' file

```
wget https://services.healthtech.dtu.dk/download/<...>/signalp-6.0h.fast.tar.gz
```

4. Configure signalp6
```
# Register the signalP copy
signalp6-register signalp-6.0h.fast.tar.gz

# Rename the "distilled" model weights
mv ~/miniconda3/envs/RFAA/lib/python3.10/site-packages/signalp/model_weights/distilled_model_signalp6.pt ~/miniconda3/envs/RFAA/lib/python3.10/site-packages/signalp/model_weights/ensemble_model_signalp6.pt
```

5. Install input preparation dependencies
```
bash install_dependencies.sh
```

6. Download the model weights
```
wget http://files.ipd.uw.edu/pub/RF-All-Atom/weights/RFAA_paper_weights.pt
```

7. Download sequence databases for MSA and template generation - **databases already downloaded at '/shared/db/rosettafold/' --> skip to creating symbolic links**
```
# uniref30 [46G]
wget http://wwwuser.gwdg.de/~compbiol/uniclust/2020_06/UniRef30_2020_06_hhsuite.tar.gz
mkdir -p UniRef30_2020_06
tar xfz UniRef30_2020_06_hhsuite.tar.gz -C ./UniRef30_2020_06

# BFD [272G]
wget https://bfd.mmseqs.com/bfd_metaclust_clu_complete_id30_c90_final_seq.sorted_opt.tar.gz
mkdir -p bfd
tar xfz bfd_metaclust_clu_complete_id30_c90_final_seq.sorted_opt.tar.gz -C ./bfd

# structure templates (including *_a3m.ffdata, *_a3m.ffindex)
wget https://files.ipd.uw.edu/pub/RoseTTAFold/pdb100_2021Mar03.tar.gz
tar xfz pdb100_2021Mar03.tar.gz

# Create symbolic links to the databases in working directory
cd ~/bin/RoseTTAFold-All-Atom
ln -s /shared/db/rosettafold/latest/* .
```

### Use RoseTTAFold All-Atom for predicting a protein monomer

a. Create a test fasta file, or use the one in this repository
- Monomer is hydA.faa, an electron-bifurcating [FeFe] hydrogenase

```
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydA.faa
```

b. Append a chain letter to the protein monomer fasta file (hydA.faa)
```
mv hydA.faa hydA_A.faa 
```

c. Generate a configuration file for a protein monomer called 'hydA.yaml'
```
emacs hydA.yaml


defaults:
  - base

job_name: "hydA"
protein_inputs: 
  A:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydA_A.faa

```

d. Copy configuration file to configuration directory
```
cp hydA.yaml ~/bin/RoseTTAFold-All-Atom/rf2aa/config/inference/
```

e. Predict the monomer structure
```
python -m rf2aa.run_inference --config-name hydA.yaml
```

```
emacs rosetta.bat

#!/bin/bash

#SBATCH --mail-user=dgittins@berkeley.edu
#SBATCH --mail-type=END,FAIL,INVALID_DEPEND,REQUEUE,STAGE_OUT
#SBATCH --job-name=RFAA

source /shared/software/miniconda3/latest/etc/profile.d/conda.sh
conda activate RFAA

echo "Starting run at : `date`"

cd ~/bin/RoseTTAFold-All-Atom
python -m rf2aa.run_inference --config-name hydA.yaml

echo "Job finished with exit code $? at: `date`"
```

f. Copy output (hydA.pdb) to local computer and view in chimeraX


### Use RoseTTAFold All-Atom for predicting a protein polymer

a. Create test fasta files, or use the ones in this repository
- Polymer is hydABC, an electron-bifurcating [FeFe] hydrogenase

```
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydA.faa
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydB.faa
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydC.faa
```

b. Append a chain letter to the protein monomer fasta file
```
mv hydA.faa hydA_A.faa
mv hydB.faa hydB_B.faa
mv hydC.faa hydC_C.faa
```

c. Generate a configuration file for a protein polymer called 'hydA_polymer.yaml'
```
emacs hydA_polymer.yaml


defaults:
  - base

job_name: "hydA_polymer"
protein_inputs: 
  A:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydA_A.faa
  B:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydB_B.faa
  C:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydC_C.faa
```

d. Move configuration file to configuration directory
```
mv hydA_polymer.yaml ~/bin/RoseTTAFold-All-Atom/rf2aa/config/inference/
```

e. Predict the polymer structure
```
python -m rf2aa.run_inference --config-name hydA_polymer.yaml
```

```
emacs rosetta_polymer.bat

#!/bin/bash

#SBATCH --mail-user=dgittins@berkeley.edu
#SBATCH --mail-type=END,FAIL,INVALID_DEPEND,REQUEUE,STAGE_OUT
#SBATCH --job-name=RFAA

source /shared/software/miniconda3/latest/etc/profile.d/conda.sh
conda activate RFAA

echo "Starting run at : `date`"

cd ~/bin/RoseTTAFold-All-Atom
python -m rf2aa.run_inference --config-name hydA_polymer.yaml

echo "Job finished with exit code $? at: `date`"
```

### Use RoseTTAFold All-Atom for predicting a protein polymer + a small molecule

a. Create test fasta files, or use the ones in this repository
- Polymer is hydABC, an electron-bifurcating [FeFe] hydrogenase

```
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydA.faa
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydB.faa
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/hydC.faa
```

b. Append a chain letter to the protein monomer fasta file
```
mv hydA.faa hydA_A.faa
mv hydB.faa hydB_B.faa
mv hydC.faa hydC_C.faa
```

c. Copy a small molecule of interest to the working directory, or use one of the ones in this repository
- Molecule is a flavin mononucleotide from https://www.rcsb.org/structure/7p5h (and the accompanying manuscript https://elifesciences.org/articles/79361)

```
wget https://github.com/dgittins/ProteinStructure/raw/main/RoseTTAFoldAll-Atom/FMN_ideal.sdf
```

d. Generate a configuration file for a protein polymer + small molecule called 'hydA_polymer_molecule.yaml'
```
emacs hydA_polymer_molecule.yaml


defaults:
  - base

job_name: "hydA_polymer_molecule"
protein_inputs: 
  A:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydA_A.faa
  B:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydB_B.faa
  C:
    fasta_file: ~/bin/RoseTTAFold-All-Atom/hydC_C.faa

sm_inputs:
  D:
    input: ~/bin/RoseTTAFold-All-Atom/FMN_ideal.sdf
    input_type: "sdf"
```

e. Move configuration file to configuration directory
```
mv hydA_polymer_molecule.yaml ~/bin/RoseTTAFold-All-Atom/rf2aa/config/inference/
```

f. Predict the polymer structure
```
python -m rf2aa.run_inference --config-name hydA_polymer_molecule.yaml
```

```
emacs rosetta_polymer_molecule.bat

#!/bin/bash

#SBATCH --mail-user=dgittins@berkeley.edu
#SBATCH --mail-type=END,FAIL,INVALID_DEPEND,REQUEUE,STAGE_OUT
#SBATCH --job-name=RFAA

source /shared/software/miniconda3/latest/etc/profile.d/conda.sh
conda activate RFAA

echo "Starting run at : `date`"

cd ~/bin/RoseTTAFold-All-Atom
python -m rf2aa.run_inference --config-name hydA_polymer_molecule.yaml

echo "Job finished with exit code $? at: `date`"
```


