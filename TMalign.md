# TMalign

Install TMalign using C++ within a Conda environment. Useful for users on shared Linux servers without administrative privileges.

1. Create a conda environment
```s
conda create -n tmalign python=3.8
conda activate tmalign
```

2. Install C++ compiler
```s
conda install -c conda-forge gxx_linux-64
```

3. Download TMalign source code
```s
cd /home/dgittins/bin/tmalign
wget https://seq2fun.dcmb.med.umich.edu//TM-align/TMalign.cpp
```

4. Compile TMalign
```s
g++ -O3 -ffast-math -lm -o TMalign TMalign.cpp
```

5. Verify installation
```s
./TMalign
```
