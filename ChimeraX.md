# ChimeraX

UCSF ChimeraX is the next‐generation interactive visualization program.

Useful resources:

- Paper - https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7737788/
- Quick start guide - https://www.rbvi.ucsf.edu/chimerax/docs/quickstart/

### General commands

Open a model using PDB id
```
open <pdb id>
```

Select model 1 by ID
```s
select #1
```

Get information about model 1
```s
info #1
```

Get a list of chains in complex #1
```s
info chains #1
```

Select chain A of model 2
```s
select #2/A
```

Remove a selection
```s
~select #2/A
```

Get the sequence of chain A in model 1
```s
sequence chain #1/A
```

Color structure using hexcode (color <structure ID> <hexcode>)
```s
color #1 #2d5280ff
```

Superimpose (align) two sturctures:

'Tools' > 'Structure Analysis' > ' Matchmaker'
```
mm #2 to #1
```

Save image:

'File' > 'Save...' > 'Files of type: PNG image (*.png)' > "structure.png"

### Color by pLDDT 

Color structures by AlphaFold2’s pLDDT (predicted local distance difference test). Note: the pLDDT values are the b-factor column in PDB.

https://alphafold.ebi.ac.uk/faq - AlphaFold produces a per-residue estimate of its confidence on a scale from 0 - 100. This confidence measure is called pLDDT and corresponds to the model’s predicted score on the lDDT-Cα metric. It is stored in the B-factor fields of the mmCIF and PDB files available for download (although unlike a B-factor, higher pLDDT is better).

```s
color bfactor palette alphafold
```

### Evaluate protein-protein binding - view the PAE plot

Useful to assess if complexes are a complex. Copy the corresponding .json file for the structure prediction (created by Colabfold) from the server to the local direcory, then:

'Tools' > 'Structure Prediction' > 'AlphaFold Error Plot' (select .json file corresponding to the .pdb file)

Select portions of the PAE plot to view the part of the structure it petains to.

View contacts at 8 angstroms (3 angstroms is the default) - blue lines are high confidence, yellow lines low confidence.
```
alphafold contacts #2 distance 8
```


### Color by sequence conservation

Color a structure by the conservation in a multiple sequence alignment. Tutorial -
https://www.rbvi.ucsf.edu/chimerax/data/conservation-coloring/conservation-coloring.html

Open a .pdb file

Open an MSA file: (rename MSA file to have one of the following extensions - .afasta, .afa, .fasta, .fa)

Color by sequence conservation attributes (specify structure by ID)
```s
color byattr seq_conservation #1
```

Change the color and show sequence conservation - a few color themes
```s
color byattr seq_conservation #1 palette bluered range -2,3 novalue #2d5280ff key true

color byattr seq_conservation #1 range -2,3 novalue silver key true

color byattr seq_conservation #1 palette cyanmaroon range -2,3 novalue #2d5280ff key true
```

A 2D label to the key:
```s
2dlab text "Conservation (AL2CO entropy measure)"
```

