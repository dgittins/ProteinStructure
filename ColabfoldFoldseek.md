# Structural prediction and searching

## Colabfold

All protein prediction software generates a multiple sequence alignment (MSA) using a reference database, e.g., RefSeq, from input sequences. 
Colabfold is a modification to AlphaFold2, which uses MMseqs2 for k-mer matching to the reference dataset before alignment. This is computationally more efficient.

Useful resources:

- Paper - https://www.nature.com/articles/s41592-022-01488-1
- GitHub - https://github.com/sokrypton/ColabFold
- Tutorial - https://www.youtube.com/watch?v=Rfw7thgGTwI


There are two options for running Colabfold:

1. Web browser using [Google Colaboratory ColabFold](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb)
2. Local [ColabFold](https://github.com/YoshitakaMo/localcolabfold)

The most efficient way to run AlphaFold is through ColabFold on a GPU.

### Predictions for individual genes or gene sets

1. Input in fasta format
```s
>hydrogenase_sequence_1
MATEIHIDPVTRIEGHLGVSVKVDGGKVIDAQLSGTMFRGFETILIGRDPRDAPVLVQRICGVCHQSHRIAGVRALEDAAKTRVTDGARKMRNLIEAVTTLFSHALWFYALGGPDYSDAVTKNGLKRLDPITGRGYMEAVANQRKLHEVLSILGAKVPHQMTYIHGGLTHEPSVEKIMQMKTRLLEVSDWVGKTEDLPAVLEDKLAGKPNDYSKGYVLNDVVDWAVYALSEKADTWGVGHQKYISYGVYDMADGSLFMPSGVWDGGKVIPMDEKKITEYVKYSWFTDDSGGVTRDSPVPKIAPDKAGAYSWAKTPRYGTMPVEGGPLARMVIAGLDPFDLRKNLGGGAEKASTFNRTIARMQEALVLRDKIVEWLGEIKPGEKTYTDFEVPNDALGVGLWEAPRGPVGHWTDIKGKKINRYQVIAGTTWNATPRDDSGIRGVIEEALIGVPVPDETKPVNVVRVVRSYDPCLACTVHVLTPGGKKYSVQVT*
>hydrogenase_sequence_2
MATEINIDPVTRLEGHLGVSVSVEDGIVVDAKCSGTMFRGFETILIGRDPRDAPVLSQRICGVCHQSHRITGVRALENLAGTRVPDGARVMRNLIEAVTTVYSHALWFYALGGPDYSDAVTKNGLTRLDPITGQGYREAVANQRKLHEVMSILGAKVPHQMTYIHGGMTHQPSVEKIMQVKTRMLEVSDWVGSTANLPAVLEDKLAGKPNDYSKGHALNDVVDWAVYALSEGADKWGGGHGKYISYGVYDMADGSLFEPSGVWDGSKVIPMDEKKITEHVKYSWYTDDSGGTPASAPTPKTKHDKAGAYSWSKTPRYGTMAVEGGPLARLVIAGIDPFDLRKNLGGGSGKASTFNRTLARMQETLLLRDKIVEWLDELVPGQKTYTDFEVPEKGIGVGLWEAPRGPVGHWTEIEGKKIKRYQIIAGTTWNATPRDDMGTRGVIEEALIGVPVPDVKNPVNVVRTVRSYDPCLACTVHVMTPAGQKYSIQVT*
```
2. Remove asterisks from the end of protein predictions
```s
sed -i 's/\*//g' proteins.faa
```
3. Run Colabfold
```s
sbatch --partition gpu --gpus 1 --wrap "/software/bin/colabfold_batch --use-gpu-relax --amber --templates --num-recycle 3 proteins.faa proteins.faa.colabfold"
```

3. Run Colabfold (batch script)
```s
#!/bin/bash                                                                                        
#SBATCH --job-name=colabfold
#SBATCH --partition gpu
#SBATCH --gpus 1

echo "Starting run at : `date`"

/software/bin/colabfold_batch --use-gpu-relax --amber --templates --num-recycle 3 proteins.faa proteins.faa.colabfold

echo "Job finished with exit code $? at: `date`"
```
By default, Colabfold will produce five predictions in PDB format (atomic x, y, z coordinates describing the position of each atom in a molecule) for each protein sequence.

**--amber** option uses AMBER (Assisted Model Building with Energy Refinement) to refine or "relax" the structures.

**--num-recycle** option controls the number of prediction recycles. Increasing recycles can improve the quality but slows down the prediction.

**--templates** option to use templates from pdb. Excluding templates can ensure that the predictions are not biased by existing structures, which may be useful if the aim is to discover completely novel protein folds.

The rank in the output predictions indicates the model's relative confidence score or quality, with **rank_001** being the highest confidence or quality score according to AlphaFold2's internal scoring system.

### Predictions of multimers

1. Input in fasta format (insert ':' between the protein sequences)
```s
>hydrogenase_complex
MFDLEKMFGKKFSRRDFIKYSSSLAVLLGLSEMYVPRIASALELVVSQKQPVIWLHGAECTGCTVSFANTHYPPVAELVLDTLSVQYHETLMAASGHQAEQALKDAVRQFAGKYIMVVEGAIPTKDDGIYCKIGGRTFIEIVNEVAKDAAYKVALGTCASFGGIPAAGPNPTGAKGLQDVIGGTVVNIPGCPAHPDWLVGTIVSVLMFGKVPDLDNYGRPKLFYGKLIHENCPRRGGYEQGQFIRSLGEELPDIEGCLGAKGCRGPVASADCPHRLWNSGTSFCIAAGAPCAGCVEPTFPQTPLYEAIPEVAKVAEAEETDKREQAIGTFGASLGGAVAGAAAAAGGIYFAMEKAKNGKNGNEEV*:
MAQKIVIDPITRIEGHLRIEVEVEGGKVVDARAAGTLWRGFEVILQGRDPRDAGVITQRICGVCPAEHAHASVQNLDAAFGAEVPDNGRIIRNLVAGSNFVASHILHFYHLAALDYLDIMAVAAYNGSNKDLLKVKEKIVGLVKAGDTSPLTPRYTPDQFCVSDPEIVTSAVHHYIQALDMRKKAQEMLAIFGGKMPHHATFVAGGVTVQPTPDRVGAFRSRLLELIDFINNIYLKDVLLFGTGPLKPIHDAKIGFGCGNFLAYGNFDLGKSGDYKNRFLKSGAIFGGDISKVNDLDPGKIAEYVRYSWYGESTTGKHPSEGETEPEPNKPDAYSWLKAPRYDGKPMEVGPLARMLVTQDKGFMNLVSSIGAWPSAVARHAARAYECKMVAEGMLKWLEELKPGEPVCDEKPVPTTGKGMGLVEAARGALGHFVEIENAKSKRYQCVVPTTWNCSPRDDGGVRGPVEEALIGAPVPDPNNPLNVVRIVRSFDPCIACAVHLIHPESNEILKFRVI*

>hydrogenase_homotrimer
MFDLEKMFGKKFSRRDFIKYSSSLAVLLGLSEMYVPRIASALELVVSQKQPVIWLHGAECTGCTVSFANTHYPPVAELVLDTLSVQYHETLMAASGHQAEQALKDAVRQFAGKYIMVVEGAIPTKDDGIYCKIGGRTFIEIVNEVAKDAAYKVALGTCASFGGIPAAGPNPTGAKGLQDVIGGTVVNIPGCPAHPDWLVGTIVSVLMFGKVPDLDNYGRPKLFYGKLIHENCPRRGGYEQGQFIRSLGEELPDIEGCLGAKGCRGPVASADCPHRLWNSGTSFCIAAGAPCAGCVEPTFPQTPLYEAIPEVAKVAEAEETDKREQAIGTFGASLGGAVAGAAAAAGGIYFAMEKAKNGKNGNEEV*:
MFDLEKMFGKKFSRRDFIKYSSSLAVLLGLSEMYVPRIASALELVVSQKQPVIWLHGAECTGCTVSFANTHYPPVAELVLDTLSVQYHETLMAASGHQAEQALKDAVRQFAGKYIMVVEGAIPTKDDGIYCKIGGRTFIEIVNEVAKDAAYKVALGTCASFGGIPAAGPNPTGAKGLQDVIGGTVVNIPGCPAHPDWLVGTIVSVLMFGKVPDLDNYGRPKLFYGKLIHENCPRRGGYEQGQFIRSLGEELPDIEGCLGAKGCRGPVASADCPHRLWNSGTSFCIAAGAPCAGCVEPTFPQTPLYEAIPEVAKVAEAEETDKREQAIGTFGASLGGAVAGAAAAAGGIYFAMEKAKNGKNGNEEV*:
MFDLEKMFGKKFSRRDFIKYSSSLAVLLGLSEMYVPRIASALELVVSQKQPVIWLHGAECTGCTVSFANTHYPPVAELVLDTLSVQYHETLMAASGHQAEQALKDAVRQFAGKYIMVVEGAIPTKDDGIYCKIGGRTFIEIVNEVAKDAAYKVALGTCASFGGIPAAGPNPTGAKGLQDVIGGTVVNIPGCPAHPDWLVGTIVSVLMFGKVPDLDNYGRPKLFYGKLIHENCPRRGGYEQGQFIRSLGEELPDIEGCLGAKGCRGPVASADCPHRLWNSGTSFCIAAGAPCAGCVEPTFPQTPLYEAIPEVAKVAEAEETDKREQAIGTFGASLGGAVAGAAAAAGGIYFAMEKAKNGKNGNEEV*
```

2. Remove asterisks from the end of protein predictions
```s
sed -i 's/\*//g' proteins.faa
```
3. Run Colabfold

```s
sbatch --partition gpu --gpus 1 --wrap "/software/bin/colabfold_batch --use-gpu-relax --amber --templates --num-recycle 3 --model-type alphafold2_multimer_v3 proteins.faa proteins.faa.colabfold"
```

3. Run Colabfold (batch script)
```s
#!/bin/bash                                                                                        
#SBATCH --job-name=colabfold
#SBATCH --partition gpu
#SBATCH --gpus 1

echo "Starting run at : `date`"

/software/bin/colabfold_batch --use-gpu-relax --amber --templates --num-recycle 3 --model-type alphafold2_multimer_v3 proteins.faa proteins.faa.colabfold

echo "Job finished with exit code $? at: `date`"
```

**--model-type** is autodetected based on the format of the input, but can also be specified for clarity/reproducibility.

<br/>

## Foldseek

Foldseek enables fast and sensitive comparisons of large protein structure sets, making it possible to quickly search databases for proteins that have similar structures.


Useful resources:

- Paper - https://www.nature.com/articles/s41587-023-01773-0
- GitHub - https://github.com/steineggerlab/foldseek?tab=readme-ov-file

There are two options for running Foldseek:

1. Web browser using [Foldseek Search Server](https://search.foldseek.com/search)
2. Local [Foldseek](https://github.com/steineggerlab/foldseek?tab=readme-ov-file)


### Search against a custom database of structures

1. Create links to high relative confidence score structures in PDB format (e.g., structures predicted using Colabfold)
```s
ln -s /home/user/colabfold/*_relaxed_rank_001_alphafold2_*.pdb /home/user/colabfold/foldseek
```

2. Use foldseek to create a database from the protein structures
```s
sbatch --wrap "foldseek createdb /home/user/colabfold/foldseek protein.DB --threads $SLURM_CPUS_ON_NODE"
```

3. Run foldseek easy-search against the local foldseek database
```s
sbatch --wrap "foldseek easy-search query_protein.pdb protein.DB aln foldseek_temp $SLURM_CPUS_ON_NODE"
```

### Search against a preformatted database

Foldseek formatted databases on Biotite: **/shared/db/foldseek/latest/db/**

1. Search against a Foldseek database
```s
sbatch --wrap "foldseek easy-search query_protein.pdb /db/alphafold_uniprot query_protein_alignment_uniprot query_protein_uniprot --format-output "query,target,fident,alnlen,mismatch,gapopen,qstart,qend,tstart,tend,evalue,bits,alntmscore,qtmscore,ttmscore,lddt,prob,theader" --threads $SLURM_CPUS_ON_NODE"
```

**--format-output** option controls the output variables.

Many other options for controlling the sensitivity of the search and the alignment type.
