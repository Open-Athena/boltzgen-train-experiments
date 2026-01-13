# Log

## Node setup (specific to our installation)
```
mv ~/.ssh ~/.ssh.bk
cp -r ~/tim-us-south-2/.ssh ~

curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
# install with defaults and restart terminal

git clone git@github.com:Open-Athena/boltzgen-train-experiments.git
cp -r ~/tim-us-south-2/training_data ~/boltzgen-train-experiments/training_data
# or:
ln -s ~/tim-us-south-2/training_data ~/boltzgen-train-experiments/training_data

pip install -e ~/boltzgen-train-experiments
pip install wandb
wandb login
```

## Runs

### vanilla-pretrained-1
Training using existing codebase, no distillation data, initialized from pretrained boltz-2

```
time python src/boltzgen/resources/main.py \
    src/boltzgen/resources/config/train/boltzgen.no_distillation.yaml \
    name=vanilla-pretrained-1 \
    wandb.entity=open-athena \
    wandb.project=boltzgen-train-experiments \
    wandb.group=vanilla \
    trainer.devices=8 \
    pretrained=training_data/boltz2_fold.ckpt \
    slurm=false \
    data.samples_per_epoch=20000
```
Runs:
* https://wandb.ai/open-athena/boltzgen-train-experiments/runs/ozldg4u7

### layer-drop-1
Pairformer layer pruning experiment. Based on activation analysis showing layers 5-50 have the smallest delta norms (least contribution to representation changes), we drop these 46 layers and fine-tune the remaining 18 layers.

**Motivation:** Analysis in `inference-experiments/pairformer_explore1.ipynb` showed:
- Layers 7-14 have smallest delta relative norms (~0.048-0.055) for random sequences
- Layers 26-33 have smallest delta relative norms (~0.039-0.044) for realistic nanobody design
- Layer 0 contributes most (delta norm ~1.9-2.2), followed by layer 63

**Parameter reduction:**
- Original: 496.50M total, 196.53M pairformer (64 layers)
- After drop: 355.24M total, 55.28M pairformer (18 layers)
- Reduction: 141.26M parameters (28.5% of total model)

**Layers kept:** 0-4, 51-63 (18 layers)
**Layers dropped:** 5-50 (46 layers)

```
time python src/boltzgen/resources/main.py \
    src/boltzgen/resources/config/train/boltzgen.no_distillation.layer_drop1.yaml \
    name=layer-drop-1 \
    wandb.entity=open-athena \
    wandb.project=boltzgen-train-experiments \
    wandb.group=layer-drop \
    trainer.devices=8 \
    pretrained=training_data/boltzgen1_adherence.ckpt \
    slurm=false \
    data.samples_per_epoch=20000
```
Runs:
* https://wandb.ai/open-athena/boltzgen-train-experiments/runs/jwiomtwb