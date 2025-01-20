# IgDesign
Read the [IgDesign paper](https://www.biorxiv.org/content/10.1101/2023.12.08.570889v2).


## Installation
We recommend [micromamba](https://mamba.readthedocs.io) as the package manager.

### Nvidia CUDA
```sh
$ micromamba env create -f environment.yml
$ micromamba activate igdesign
$ pip install -e .
```

### AMD ROCm 
This is a multi-step process so we've provided a script. We also assume you're using micromamba as the package manager. If not, please edit the script accordingly.
```sh
$ chmod +x install_amd_env.sh
$ ./install_amd_env.sh
$ micromamba activate igdesign
```


## Download model weights
In order to download our pretrained model weights please use the provided shell script:
```sh
$ chmod +x download_ckpts.sh
$ ./download_ckpts.sh
```
This script will download the LMDesign and fine-tuned ProteinMPNN weights associated
with the split which excluded the [ACVR2B](https://www.ncbi.nlm.nih.gov/gene/93) target.

Version mismatches in the checkpoints from Lightning can be fixed with the following snippet permanently:

```bash
python -m pytorch_lightning.utilities.upgrade_checkpoint ckpts/igdesign_acvr2b_holdout.ckpt
python -m pytorch_lightning.utilities.upgrade_checkpoint ckpts/igmpnn_acvr2b_holdout.ckpt
```

## Inference
To inverse fold sequences, run `predict.py` with the environment activated to use the default config with PDB:1N8Z (Trastuzumab-HER2):

```sh
$ (igdesign) python predict.py
```

To use a different config specify the `--config_name` parameter, for example using PDB:5NGV (Bimagrumab-ACVR2B)
```sh
$ (igdesign) python predict.py --config_name 5ngv.yaml
```

The output will be a CSV containing sequences and (if specified) cross-entropy losses for all of the specified CDRs. For example, the column `"hcdr3"` corresponds to the generated HCDR3s and the column `"ce_loss_independent_hcdr3"` corresponds to the cross-entropy scores.

## Configuration
Inference details are specified using YAML configs, examples of which can be found in `./configs`. Each config uses the following parameters:
- `structure_path`: Path to a PDB file with the desired structure
- `lmdesign_checkpoint`: Path to IgDesign checkpoint
- `pmpnn_checkpoint`: Path to IgMPNN checkpoint
- `save_path`: Path to save generated sequences and scores
- `region_order`: List of CDRs to be designed and the order they will be designed in
- `lmdesign_num_decoding_orders`: Number of decoding orders used during IgDesign inference
- `lmdesign_num_pmpnn_seqs`: Number of IgMPNN sequences used during IgDesign inference
- `lmdesign_num_lm_seqs`: Number of language model samples used during IgDesign inference
- `lmdesign_pmpnn_logit_temperature`: Sampling temperature for IgMPNN
- `lmdesign_output_logit_temperature`: Sampling temperature for IgDesign
- `independent_loss`: If true, computes cross-entropy loss using IgDesign
- `condition_on_light_chain`: If true, conditions on light chain sequence
- `condition_on_antigen`: If true, conditions on antigen sequence
- `epitope_idxs_or_all`: Set to `"all"` to use full antigen. Set to a list of indices to use a subset of the antigen
- `antigen_chain_id`: Chain ID in PDB file for antigen
- `heavy_chain_id`: Chain ID in PDB file for heavy chain
- `light_chain_id`: Chain ID in PDB file for light chain
- `regions`: Each key of this should be a CDR in `region_order` mapping to `positions` (list of indices corresponding to the CDR) and `chain` (either `"heavy"` or `"light"`)

## Surface Plasmon Resonance (SPR) Data
SPR data generated from validating IgDesign is included in `"./data"` with representative sensorgrams in `"./data/Sensorgrams"`. Each CSV corresponds to one antibody-antigen system. The columns for each dataset are:
- `Target`: The target screened against
- `Reference Ab`: The reference antibody used
- `Method`: Method for sequence generation. Either `"Positive Control"`, `"SAbDab"`, or `"Inverse Folding"`
- `HCDRs Designed`: Except for positive control, either `"HCDR3"` or `"HCDR123"`
- `HCDR1`: HCDR1 sequence
- `HCDR2`: HCDR2 sequence
- `HCDR3`: HCDR3 sequence
- `KD (nM)`: Binding affinity measured in SPR (average of replicates)
- `Binding Replicates`: Number of observed binding replicates
- `Replicates`: Number of observed replicates
- `Ratio Binding Replicates`: Ratio of replicates that show binding
- `Binder`: True for binders, False for non-binders. Except for positive controls, we call a sequence a binder if it binds in all replicates, otherwise we call it a non-binder.
- scRMSD columns: Columns for scRMSD are organized by region (one of `"HCDR1"`, `"HCDR2"`, `"HCDR3"`, `"HCDR123"`, or `"Fv"`) followed by `"scRMSD"` and followed by the model or strategy in parentheses (`"ABB2"` for ABodyBuilder2, `"ABB3"` for ABodyBuilder3, `"ABB3-LM"` for ABodyBuilder3-LM, `"ESMFold"` for ESMFold, `"Mean/Min/Max Ensemble"` for mean/min/max ensembles).

`Note`: At this time, we have released datasets for 7 out of 8 antibody-antigen systems used to evaluate IgDesign. The dataset for CD40-Ravagalimab has sequences blinded as X's and is included in `"./data/Blinded"`.

## Hardware compatibility
We've tested this model on Nvidia A100 and H100 and AMD Mi300x gpus. We expect the model to perform equally well on the Nvidia H200 and the upcoming AMD Mi325x. If you encounter any bugs, feel free to open a ticket.

## Citations
If you find our code, data, or results useful, we ask that you cite our work: 
```
@article{Shanehsazzadeh2023igdesign,
  title = {IgDesign: In vitro validated antibody design against multiple therapeutic antigens using inverse folding},
  url = {http://dx.doi.org/10.1101/2023.12.08.570889},
  DOI = {10.1101/2023.12.08.570889},
  publisher = {Cold Spring Harbor Laboratory},
  author = {Shanehsazzadeh,  Amir and Alverio,  Julian and Kasun,  George and Levine,  Simon and Calman,  Ido and Khan,  Jibran A and Chung,  Chelsea and Diaz,  Nicolas and Luton,  Breanna K and Tarter,  Ysis and McCloskey,  Cailen and Bateman,  Katherine B and Carter,  Hayley and Chapman,  Dalton and Consbruck,  Rebecca and Jaeger,  Alec and Kohnert,  Christa and Kopec-Belliveau,  Gaelin and Sutton,  John M and Guo,  Zheyuan and Canales,  Gustavo and Ejan,  Kai and Marsh,  Emily and Ruelos,  Alyssa and Ripley,  Rylee and Stoddard,  Brooke and Caguiat,  Rodante and Chapman,  Kyra and Saunders,  Matthew and Sharp,  Jared and Ganini da Silva,  Douglas and Feltner,  Audree and Ripley,  Jake and Bryant,  Megan E and Castillo,  Danni and Meier,  Joshua and Stegmann,  Christian M and Moran,  Katherine and Lemke,  Christine and Abdulhaqq,  Shaheed and Klug,  Lillian R and Bachas,  Sharrol},
  year = {2023},
  month = dec 
}
```
