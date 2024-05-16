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

- Monomer is hydA.faa, an electron-bifurcating [FeFe] hydrogenase

a. Append a chain letter to the protein monomer fasta file (hydA.faa)
```
mv hydA.faa hydA_A.faa 
```

b. Generate a configuration file for a protein monomer
```
emacs hydA.yaml

defaults:
  - base

job_name: "hydA"
protein_inputs: 
  A:
    fasta_file: /groups/banfield/projects/environmental/borehole/hydrogenase/rosettafold/hydA_A.fasta
```

c. Copy configuration file to configuration directory
```
cp hydA.yaml ~/bin/RoseTTAFold-All-Atom/rf2aa/config/inference/
```

d. Predict the monomer structure
```
python -m rf2aa.run_inference --config-name hydA.yaml
```
