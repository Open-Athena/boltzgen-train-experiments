# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BoltzGen is a protein design system using diffusion models for computational binder design. It generates proteins/peptides that bind to target molecules (proteins, peptides, small molecules). The codebase is built on PyTorch Lightning with Hydra configuration.

## Commands

### Installation
```bash
pip install -e .           # Development install
pip install -e .[dev]      # With dev dependencies (wandb, ruff, pytest)
```

### Linting
```bash
ruff check src/
```

### Running Designs
```bash
boltzgen run <design_spec.yaml> --output <output_dir> --protocol <protocol> --num_designs <N>
```

Protocols: `protein-anything`, `peptide-anything`, `protein-small_molecule`, `nanobody-anything`, `antibody-anything`

### Training Models
```bash
# Small model (development, 8 GPUs)
python src/boltzgen/resources/main.py src/boltzgen/resources/config/train/boltzgen_small.yaml name=experiment_name

# With W&B tracking
python src/boltzgen/resources/main.py src/boltzgen/resources/config/train/boltzgen_small.yaml \
    name=experiment_name \
    wandb.entity=your-entity \
    wandb.project=your-project
```

Training configs in `src/boltzgen/resources/config/train/`:
- `boltzgen_small.yaml` - Small model for development
- `boltzgen.yaml` - Full model
- `boltzgen.no_distillation.yaml` - Without distillation data
- `inverse_folding.yaml` - Inverse folding model only

## Architecture

### Entry Points
- **CLI**: `src/boltzgen/cli/boltzgen.py` - Commands: `run`, `configure`, `execute`, `merge`, `download`, `check`
- **Training**: `src/boltzgen/resources/main.py` - Hydra-based task execution

### Core Components

**Task System** (`src/boltzgen/task/`):
- Abstract `Task` base class with `run()` method
- `train/` - Training task (PyTorch Lightning)
- `predict/` - Inference/design generation
- `analyze/` - Structure analysis
- `filter/` - Design filtering and ranking

**Model** (`src/boltzgen/model/`):
- `models/boltz.py` - Main Boltz LightningModule (diffusion-based design)
- `layers/` - Building blocks (pairformer, miniformer)
- `modules/` - Specialized components (diffusion, affinity, inverse_fold)
- `loss/` - Loss functions (distogram, confidence, diffusion)

**Data Pipeline** (`src/boltzgen/data/`):
- `data.py` - Main data loading system
- `parse/schema.py` - YAML design specification parsing
- `feature/` - Feature extraction
- `crop/` - Structure cropping
- `tokenize/` - Molecular tokenization
- `write/` - mmCIF output

**Config** (`src/boltzgen/resources/config/`):
- Training configs in `train/`
- Pipeline configs: `design.yaml`, `fold.yaml`, `inverse_fold.yaml`, `analysis.yaml`, `filtering.yaml`

### Design Pipeline Flow
1. **Design** - Diffusion model generates backbone structures
2. **Inverse Folding** - Predicts sequences for backbones
3. **Folding** - Re-folds complexes with Boltz-2
4. **Design Folding** - Re-folds binders alone (protein protocols only)
5. **Affinity** - Predicts binding affinity (small molecule protocols)
6. **Analysis** - Computes quality metrics
7. **Filtering** - Ranks and selects final designs

## Key Conventions

- All residue indices in design YAML files are **1-indexed** using `label_asym_id` (not `auth_asym_id`)
- File paths in design YAMLs are relative to the YAML file location
- Training data expected at `./training_data/` by default (targets, msa, mols directories)
- Model weights cached in `~/.cache/` (configurable via `$HF_HOME`)
- Hydra configuration: override any parameter via CLI with `key=value` syntax
