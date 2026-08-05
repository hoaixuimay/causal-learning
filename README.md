## Setup & Run

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

### 4. LLM - Download Ollama and run model llama3.1:8b
- Ollama https://ollama.com/download
- Model llama3.1 https://ollama.com/library/llama3.1 

Run Ollama: 
```bash
ollama run llama3.1:8b
```

### 5. Run notebook examples
#### 5.1. PC on ASIA
- A notebook that generates data from ASIA and runs PC on it.

`File notebook/01_pc_asia.ipynb`
#### 5.2. PC and SHD on ALARM
- Compute the structural Hamming distance (SHD) between the learned graph and the true graph

`
File notebook/02_pc_alarm.ipynb
`
#### 5.3. Experiments on LLM for causal predictions
- Use LLM to generate causal direction for 10 pairs from the Tübingen cause-effect pairs dataset.
- Change variable name to neutral labels (A and B) to make experiment for the pairs
- Paraphrase the prompt to make experiment for the pairs

`
File notebook/03_llm_pairs.ipynb
`


## Requirements

- Python 3.10
- [bnlearn](https://github.com/erdogant/bnlearn)
- [causal-learn](https://github.com/py-why/causal-learn)
- [Graphviz](https://graphviz.org/)

All dependencies are pinned in `environment.yml`.