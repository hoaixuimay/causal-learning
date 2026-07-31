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

### 4. Run notebook examples
#### 4.1. PC on ASIA
File 01_pc_asia.ipynb
#### 4.2. PC and SHD on ALARM
File 02_pc_alarm.ipynb


## Requirements

- Python 3.10
- [bnlearn](https://github.com/erdogant/bnlearn)
- [causal-learn](https://github.com/py-why/causal-learn)
- [Graphviz](https://graphviz.org/)

All dependencies are pinned in `environment.yml`.