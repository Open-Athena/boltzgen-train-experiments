# Log

## Node setup (specific to our installation)
```
cp -r ~/tim-us-south-2/boltzgen ~
rm ~/boltzgen/training_data
cp -r ~/tim-us-south-2/training_data ~/boltzgen/training_data
wandb login

curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
source ~/miniconda3/etc/profile.d/conda.sh
conda activate base
pip install -e ~/boltzgen
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