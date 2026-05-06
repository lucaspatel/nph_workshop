# HMP-IBD Microbiome Tutorial

End-to-end tutorial walking through metagenomic + metatranscriptomic
analysis of HMP2 / IBDMDB shotgun data, ending with a Random Forest
classifier for IBD vs nonIBD.

The notebook (`tutorial.ipynb`) covers:

1. Fetching IBD-relevant reference genomes from NCBI
2. Streaming HMP2 paired-end shotgun samples and aligning with MIINT/DuckDB
3. Building feature tables (metagenomic + metatranscriptomic)
4. Comparing MTX vs MGX coverage and activity ratios
5. Diversity analysis on the full 130-participant dataset
   (alpha, weighted/unweighted UniFrac PCoA, PERMANOVA)
6. Random Forest classification with ROC curves and feature importance

---

## 1. Install Miniforge (a minimal conda distribution)

Miniforge ships `conda` pre-configured with `conda-forge` as the default
channel, which is what we want for scientific Python packages.

### macOS / Linux

```bash
# Download the installer for your platform (this auto-detects)
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"

# Run it (accept the license, accept the default install location)
bash Miniforge3-$(uname)-$(uname -m).sh

# Restart your shell, or:
source ~/.bashrc      # bash users
source ~/.zshrc       # zsh users (default on macOS)
```

### Windows

Download and run the installer from
<https://github.com/conda-forge/miniforge/releases/latest>
(`Miniforge3-Windows-x86_64.exe`), then open the **Miniforge Prompt** from
the Start menu.

### Verify

```bash
conda --version    # should print conda 24.x or similar
```

---

## 2. Create the tutorial conda environment

From the directory containing this README:

```bash
conda create -n miint-tutorial -c conda-forge -c bioconda -y \
    python=3.11 \
    duckdb python-duckdb \
    biom-format scikit-bio scikit-learn \
    pandas numpy scipy \
    matplotlib seaborn \
    jupyterlab ipykernel
```

Then activate it:

```bash
conda activate miint-tutorial
```

Register the env as a Jupyter kernel so the notebook can find it:

```bash
python -m ipykernel install --user \
    --name miint-tutorial \
    --display-name "Python (miint-tutorial)"
```

### Verify the install

```bash
python -c "import duckdb, biom, skbio, sklearn, seaborn; \
print('duckdb',  duckdb.__version__); \
print('skbio',   skbio.__version__); \
print('sklearn', sklearn.__version__)"
```

---

## 3. Launch JupyterLab

From this directory, with the env activated:

```bash
jupyter lab
```

This opens JupyterLab in your browser. Open `tutorial.ipynb`,
then in the top-right kernel picker select
**Python (miint-tutorial)**.

If the kernel doesn't appear in the picker, click `Kernel -> Change Kernel`
and pick it from the list. If it's still missing, re-run the
`python -m ipykernel install` step above with the env activated.

---

## 4. Required data files

The notebook expects these files in the working directory:

| File | Purpose |
|---|---|
| `hmp-metadata.txt` | sample mapping (diagnosis, host_subject_id, etc.) |
| `metaG-feature-table.biom` | metagenomic feature table |
| `metaT-feature-table.biom` | metatranscriptomic feature table |
| `metag_taxonomy.tsv` | metaG feature -> lineage (for species names) |
| `metaT_taxonomy.tsv` | metaT feature -> lineage |
| `tree.nwk` | phylogeny (Newick) for UniFrac and Faith's PD |

Earlier notebook cells (Parts 1 & 2) fetch reference genomes and HMP2
samples from NCBI / EBI on the fly, so the only files you need to
provide locally are those for Part 3.

---

## 5. Running the notebook

Run cells top to bottom. The first time you run Parts 1 and 2 will be
slow because they stream FASTQ data from EBI. Part 3 reads only the
local files above and is much faster.

If you want to skip the streaming steps and jump straight to the
analysis, you can run only Part 3 (the cells under "Part 3: Analyzing a
full dataset").

---

## Troubleshooting

- **`ModuleNotFoundError: skbio`** — the kernel selected in JupyterLab
  is not the `miint-tutorial` kernel. Switch via `Kernel -> Change Kernel`.
- **`conda: command not found`** — restart your terminal after
  installing Miniforge, or `source` the appropriate rc file.
