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