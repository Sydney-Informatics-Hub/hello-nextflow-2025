
# Developer Installation  

## `mamba` for local testing

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
# complete prompts with defaults

export PATH="$HOME/miniforge3/bin:$PATH"
source ~/.bashrc

# confirm installation versions mamba --version
```

```console
mamba 1.5.8
conda 24.3.0
```

Install packages:  

```bash
mamba create -n day2
mamba activate day2
mamba install -c bioconda nextflow # process tools via singularity containers
mamba install -c conda-forge mkdocs mkdocs-material
```

```bash
mamba list | grep -E "nextflow|mkdocs |mkdocs-material"
```

```console
mkdocs                    1.6.0              pyhd8ed1ab_0    conda-forge
mkdocs-material           9.5.31             pyhd8ed1ab_0    conda-forge
mkdocs-material-extensions 1.3.1              pyhd8ed1ab_0    conda-forge
nextflow                  24.04.4              hdfd78af_0    bioconda
```

**Singularity**

Requires installation of singularity. See: https://docs.sylabs.io/guides/latest/user-guide/quick_start.html#quick-installation-steps

## BioImage VM setup

First, SSH into test VM.

Load dependencies

```bash
$ module load nextflow singularity
$ mkdir -p $HOME/singularity_cache
$ export SINGULARITY_CACHEDIR=$HOME/singularity_cache
# NXF_SINGULARITY_CACHEDIR exported in nextflow.config
```

### Pulling containers  

`singularity pull docker://quay.io/biocontainers/salmon:1.10.1--h7e5ed60_0` pulls to `salmon_1.10.1--h7e5ed60_0.sif`. Pull so it matches the proces.container definitions and doesn't
rebuild the image.

quay.io/biocontainers/salmon:1.10.1--h7e5ed60_0
```bash
$ cd $NXF_SINGULARITY_CACHEDIR
$ singularity pull quay.io-biocontainers-salmon-1.10.1--h7e5ed60_0.img docker://quay.io/biocontainers/salmon:1.10.1--h7e5ed60_0 &&
$ singularity pull quay.io-biocontainers-fastqc-0.12.1--hdfd78af_0.img docker://quay.io/biocontainers/fastqc:0.12.1--hdfd78af_0 &&
$ singularity pull quay.io-biocontainers-multiqc-1.19--pyhdfd78af_0.img docker://quay.io/biocontainers/multiqc:1.19--pyhdfd78af_0
```

### Testing content

```bash
$ cd ~
$ git clone git@github.com:Sydney-Informatics-Hub/hello-nextflow-2025.git
$ cd hello-nextflow-2025
```

#### Part 1

Pull Nextflow's `hello` pipeline and run:

```bash
$ nextflow run nextflow-io/hello
```

Test `tree` and check outputs:
```
$ tree
```

#### Part 2

**Note:** `.main.nf` and `.nextflow.config` runs the final pipeline. 

```bash
$ cd part2  
$ mv .nextflow.config nextflow.config
```

Run completed pipeline (includes introspection reports etc., multi-sample and multiple cpus):  

```bash
$ nextflow run .main.nf --reads data/samplesheet_full.csv
```

## mkdocs  

To generate html docs during development (before deploying):

```bash
cd ~/hello-nextflow/
mkdocs serve
# open http://127.0.0.1:8000/ in browser
```

To render docs for website (manually):  

```bash
mkdocs gh-deploy
```

Open [https://sydney-informatics-hub.github.io/hello-nextflow/](https://sydney-informatics-hub.github.io/hello-nextflow/) in a browser.  

