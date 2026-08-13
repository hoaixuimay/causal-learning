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
**Notebook:** `notebook/01_pc_asia.ipynb`

**Objective:** A notebook that generates data from ASIA and runs PC on it.

#### 5.2. PC and SHD on ALARM
**Notebook:** `notebook/02_pc_alarm.ipynb`

**Objective:** Compute the structural Hamming distance (SHD) between the learned graph and the true graph

**Data Generation:**
Generate 5,000 synthetic observational samples from the ALARM Bayesian Network:

```python
df_alarm = bn.sampling(alarm_model, n=5000, methodtype='bayes')
```
**PC Algorithm:**
Run the PC algorithm on the generated data using the Chi-square independence test:
```python
cg = pc(data,
 indep_test='chisq', 
 node_names=list(df_alarm.columns)
 )
```
**SHD Calculation:** 
Calculate the Structural Hamming Distance (SHD) between the learned graph and the ground-truth ALARM graph.

**Result:**
```log
SHD:  8
```
**Notes:** 
The data are randomly sampled from the ALARM Bayesian Network. Therefore, the learned graph and SHD may vary slightly between different runs.

#### 5.3. Experiments on LLM for causal predictions
**Notebook:** `notebook/03_llm_pairs.ipynb`

**Objective:**
- Use LLM to generate causal direction for 10 pairs from the Tübingen cause-effect pairs dataset with this template:
```
You are an expert in Causal Inference. Your task is to determine the causal direction between two variables based on their context and metadata.

Dataset Context: {context}
Variable X: {var_x_desc}
Variable Y: {var_y_desc}

Based on physical laws, common sense, and scientific facts, select the most plausible causal direction:
- X -> Y (X causes Y)
- Y -> X (Y causes X)
- Independent (No direct causal relationship)

Strict Output Format (DO NOT use any markdown, do not use double asterisks ** anywhere):
Direction: [Your choice: X -> Y, Y -> X, or Independent]
Reason: [Provide a brief explanation in 1-2 sentences]
```
- Change variable name to neutral labels (A and B) to make experiment for the pairs by replace above template with {var_x_desc} by A and {var_y_desc} by B
```
You are an expert in Causal Inference. Your task is to determine the causal direction between two variables based on their context and metadata.

Dataset Context: {context}
Variable X: A
Variable Y: B

Based on physical laws, common sense, and scientific facts, select the most plausible causal direction:
- X -> Y (X causes Y)
- Y -> X (Y causes X)
- Independent (No direct causal relationship)

Strict Output Format (DO NOT use any markdown, do not use double asterisks ** anywhere):
Direction: [Your choice: X -> Y, Y -> X, or Independent]
Reason: [Provide a brief explanation in 1-2 sentences]
```
- Paraphrase the prompt to make experiment for the pairs with this template:
```
Your role is to perform causal reasoning. Using the provided context and metadata, decide which variable is the most likely cause of the other.

Dataset Context: {context}
Variable X: {var_x_desc}
Variable Y: {var_y_desc}

Based on the information provided, identify the causal direction between the two variables:
- X -> Y (X causes Y)
- Y -> X (Y causes X)
- Independent (No direct causal relationship)

Strict Output Format (DO NOT use any markdown, do not use double asterisks ** anywhere):
Direction: [Your choice: X -> Y, Y -> X, or Independent]
Reason: [Provide a brief explanation in 1-2 sentences]
```

#### **Result**:

|   Pair | Var_X     | Var_Y                                                | Ground_Truth   | Original   | Neutral(A/B)    | Paraphrased   | Flip?(Original ~ Neutral)   | Flip?(Original ~ Paraphrased)   |
|-------:|:----------|:-----------------------------------------------------|:---------------|:-----------|:----------------|:--------------|:----------------------------|:--------------------------------|
|   0001 | altitude  | temperature (average over 1961-1990)                 | X -> Y         | X -> Y     | X -> Y          | Y -> X        | False                       | True                            |
|   0002 | altitude  | precipitation (yearly value averaged over 1961-1990) | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |
|   0003 | longitude | temperature (averaged over 1961-1990)                | X -> Y         | Y -> X     | X -> Y          | Y -> X        | True                        | False                           |
|   0004 | altitude  | sunshine (yearly value averaged over 1961-1990)      | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |
|   0005 | Rings     | Length                                               | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |
|   0006 | Rings     | Shell weight                                         | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |
|   0007 | Rings     | Diameter                                             | X -> Y         | Y -> X     | X -> Y          | Y -> X        | True                        | False                           |
|   0008 | Rings     | Height                                               | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |
|   0009 | Rings     | Whole weight                                         | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |
|   0010 | Rings     | Shucked weight                                       | X -> Y         | X -> Y     | X -> Y          | X -> Y        | False                       | False                           |

#### **Analysis on the result:**
- **Original LLM**: There are two failure cases when comparing the LLM predictions with the ground truth:
    - **Pair 0003:** Ground truth: **Longitude → Temperature**. The LLM predicted the opposite direction. Its reasoning was: "The temperature of an area is determined by its geographical location, and longitude is a key component of that location. Therefore, it is more plausible to assume that the temperature (Y) causes or is influenced by the longitude (X), rather than the other way around."
        - **Analyse reason:** Its reasoning recognizes that geographical location influences temperature, but the final conclusion incorrectly assigns the causal direction as **Temperature → Longitude**.
    - **Pair 0007:** Ground truth: **Rings -> Diameters**. The LLM predicted the opposite direction. Its reasoning was: "In the context of abalone data, it is well-established that the diameter (Y) of an abalone shell increases as more rings (X) form on its surface. This relationship is consistent with physical laws governing growth patterns in shells."
        - **Analyse reason:** The LLM correctly identifies the relationship between the number of rings and shell diameter, but incorrectly infers the causal direction as **Diameters → Rings**.
    - **Summary**: Both cases suggest that the LLM has relevant domain knowledge and can identify plausible relationships between variables, but it may struggle to correctly translate this knowledge into the causal direction.
- **Neutral (A/B):**
    - A **flip** occurs in **Pairs 0003 and 0007** compared with the original LLM predictions.
    - **Analyse the reasoning in** `output/table_reason.md`. We can see that the LLM tends to associate Variables A and B with attributes that it considers likely to be correct. However, this assumption is not always correct in real-world situations. For example, in Pair 0003, the LLM **assumes** that temperature (Variable A) and precipitation (Variable B) have a particular causal relationship based on **its prior knowledge**.

- **Paraphrased**:
    - A **flip** occurs in **Pair 0001** compared with the original LLM prediction.
    - **Analyse the reasoning in** `output/table_reason.md`. For this pair, the ground truth is **Altitude → Temperature**. The paraphrased LLM reasoning states that **altitude (X) affects temperature (Y)**, which is consistent with the ground truth. However, the final prediction is **Y → X (Temperature → Altitude)**, which contradicts both the reasoning and the ground truth.
    - **Summary:** This indicates an inconsistency between the LLM's reasoning and its final causal direction.

## Requirements

- Python 3.10
- jupyter
- notebook
- [bnlearn](https://github.com/erdogant/bnlearn)
- [causal-learn](https://github.com/py-why/causal-learn)
- [Graphviz](https://graphviz.org/)
- ollama-python

All dependencies are pinned in `environment.yml`.