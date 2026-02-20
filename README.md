# GxVAEs
A PyTorch implementation of “GxVAEs: Two Joint VAEs Generate Hit Molecules from Gene Expression Profiles“.
The paper has been accepted by [AAAI 2024](https://ojs.aaai.org/index.php/AAAI/article/view/29248). 

![Overview of GxVAEs](https://github.com/naruto7283/GxVAEs/blob/main/overview.png)

## Objectives of GxVAEs
This implementation was developed by Chen Li (li.chen.z2@a.mail.nagoya-u.ac.jp) and Yoshihiro Yamanishi (yamanishi@i.nagoya-u.ac.jp), affiliated with the Department of Complex Systems Science at the Graduate School of Informatics, Nagoya University, Japan, at the time of release.

GxVAEs aim to
- generate hit-like molecules from gene expression profiles.
- generate therapeutic molecules from patients’ disease profiles.

## Environment Installation
Execute the following command:
```
$ conda env create -f env/gxvaes_env.yml
$ conda activate py36tf
```

### Important: Git LFS dataset files
This repository stores large dataset files (for example `datasets/LINCS/mcf7.csv`) with Git LFS.
If Git LFS is not installed, you may get a pointer file that starts with:
`version https://git-lfs.github.com/spec/v1`

Install and fetch LFS files before training:
```bash
# Ubuntu/Debian
sudo apt-get update && sudo apt-get install -y git-lfs

git lfs install
git lfs pull
git lfs checkout
```

Quick check:
```bash
head -n 1 datasets/LINCS/mcf7.csv
```
If the first line starts with `version https://git-lfs.github.com/spec/v1`, the real dataset content has not been downloaded yet.

## Windows (PowerShell) Quick Start
From the `GxVAEs` directory in PowerShell:
```powershell
# 1) Create environment (first time only)
conda env create -f .\env\gxvaes_env.yml

# 2) Activate
conda activate py36tf

# 3) (Optional) Reproducibility
# Add --use_seed to commands below if you want deterministic behavior.

# 4) Train GeneVAE
python .\main.py --train_gene_vae

# 5) Test GeneVAE
python .\main.py --test_gene_vae

# 6) Train SmilesVAE
python .\main.py --train_smiles_vae --teacher_forcing_rate 0.5 --temperature 1.0

# 7) Generate molecules for one target protein
python .\main.py --generation --protein_name AKT1 --candidate_num 50 --temperature 1.0

# 8) Evaluate with Tanimoto similarity
python .\main.py --calculate_tanimoto --protein_name AKT1
```

If `conda activate` is not recognized in PowerShell, run once:
```powershell
conda init powershell
```
Then restart PowerShell.

## File Description

- **The datasets Folder**
    - `LINCS/mcf7.csv`: training gene expression profiles for the MCF7 cell line.
    - `test_protein/*.csv`: test gene profiles for proteins (AKT1, AKT2, AURKB, CTSK, EGFR, HDAC1, MTOR, PIK3CA, SMAD3, TP53).
    - `ligands/source_*.csv`: source ligands used for Tanimoto comparison.
    - `tools/`: helper mapping files (`symbol2hsa.json`, `source_genes.csv`, `cell_list.txt`).
- **The results Folder**
    - `results/saved_gene_vae_<cell>.pkl` is created by `--train_gene_vae` (for default cell: `results/saved_gene_vae_mcf7.pkl`).
    - `results/gene_vae_train_results.csv` is created during GeneVAE training.
    - `results/saved_smiles_vae.pkl` and `results/smiles_vae_train_results.csv` are created by `--train_smiles_vae`.
    - `results/generation/res-<protein>.csv` is created by `--generation` (example: `results/generation/res-AKT1.csv`).
- **main.py:** Define the main function for training, generation, and evaluation.
- **GeneVAE.py**: Defines the GeneVAE model for extracting gene expression profile features.
- **train_gene_vae.py**: Code for training the GeneVAE model.
- **SmilesVAE.py**: Defines the SmilesVAE model to generate SMILES strings with extracted gene features.
- **train_smiles_vae.py**: Code for training the SmilesVAE model.
- **utils.py**: Defines other functions used in GxVAEs.

## Experimental Reproduction

  - **STEP 1**: Pretrain GeneVAE:
  ``` 
  $ python main.py --train_gene_vae
  ```
  - **STEP 2**: Test the trained GeneVAE:
  ```
  $ python main.py --test_gene_vae
  ```
  Requires: `results/saved_gene_vae_mcf7.pkl` (generated in STEP 1).
  - **STEP 3**: Train SmilesVAE:
  ```  
  $ python main.py --train_smiles_vae --teacher_forcing_rate 0.5 --temperature 1.0
  ```
  - **STEP 4**: Test the trained SmilesVAE:
  ```
  $ python main.py --test_smiles_vae
  ```
  Requires: `results/saved_smiles_vae.pkl` (generated in STEP 3).
  - **STEP 5**: Generate molecules for the 10 ligands using GxVAEs
  ```
  $ python main.py --generation --protein_name AKT1 --candidate_num 50 --temperature 1.0
  ```	
  Output: `results/generation/res-AKT1.csv`
  - **STEP 6**: Calculate Tanimoto similarity between a source ligand and generated SMILES strings: 
  ```
  $ python main.py --calculate_tanimoto --protein_name ***
  ```
&nbsp;&nbsp;&nbsp;&nbsp;Note that '***' indicates a protein name, such as 'AKT1'.

## Important CLI Notes
- `--gene_hidden_sizes` accepts one or more integers, e.g.:
  ```
  $ python main.py --train_gene_vae --gene_hidden_sizes 512 256 128
  ```
- RNN direction flags:
  - bidirectional (default): `--bidirectional`
  - unidirectional: `--no_bidirectional`

## Contact
If you have any questions, please feel free to contact Chen Li at li.chen.z2@a.mail.nagoya-u.ac.jp.
  
## Citation
  ```
  C. Li and Y. Yamanishi (2024). GxVAEs： Two Joint VAEs Generate Hit Molecules from Gene Expression Profiles.
  ```
  
  BibTeX format:
  ```
  @inproceedings{li2024gxvaes,
  title={GxVAEs： Two Joint VAEs Generate Hit Molecules from Gene Expression Profiles},
  author={Li, Chen and Yamanishi, Yoshihiro},
  booktitle={Proceedings of the 38th AAAI Conference on Artificial Intelligence (AAAI 2024)},
  year={2024}
}
  ```
