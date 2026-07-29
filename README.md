## Setup

### 1. Create the conda environment

```bash
conda env create -f environment.yml
conda activate causal_env
```

### 2. Register the environment as a Jupyter kernel

```bash
python -m ipykernel install --user --name causal_env --display-name "Python (causal_env)"
```

### 3. Open the notebook

Launch Jupyter (or VS Code) and select the **Python (causal_env)** kernel:

```bash
jupyter notebook
```

## Requirements

- Python 3.10
- [bnlearn](https://github.com/erdogant/bnlearn)
- [causal-learn](https://github.com/py-why/causal-learn)
- [Graphviz](https://graphviz.org/)

All dependencies are pinned in `environment.yml`.