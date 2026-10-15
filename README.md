# AES Side-Channel Analysis with Correlation Power Analysis

A Python-based AES side-channel analysis project using Correlation Power Analysis (CPA) to recover candidate AES round-key bytes from measured power traces. The project also investigates how the number of traces affects key-recovery success.

## Overview

This project implements a power side-channel analysis workflow that compares hypothetical data-dependent leakage against measured power traces. It evaluates all 256 possible key-byte guesses and selects the candidate with the strongest absolute Pearson correlation.

The repository contains five Jupyter notebooks covering dataset inspection, preprocessing, AES leakage modeling, CPA key recovery, and minimum-trace analysis.

## Project Structure

```text
aesSca/
├── .gitignore
├── README.md
└── notebooks/
    ├── 01_datasetInspection.ipynb
    ├── 02_dataPreprocess.ipynb
    ├── 03_AESLeakageModel.ipynb
    ├── 04_CPA_Attack.ipynb
    ├── 05_minimumTraces.ipynb
    └── figures/
        └── cpa/
            ├── byte_00.png
            ├── byte_01.png
            ├── ...
            └── byte_15.png
```

The raw dataset and generated result files are excluded from version control.

## Notebook Workflow

### 1. Dataset Inspection

File: `01_datasetInspection.ipynb`

Inspects the raw CSV dataset and examines its structure, ciphertext formatting, trace columns, data types, missing values, and basic statistical properties.

### 2. Data Preprocessing

File: `02_dataPreprocess.ipynb`

Loads and validates the dataset, converts hexadecimal ciphertexts into 16-byte arrays, checks trace values, and examines trace statistics and potential alignment issues.

The preprocessing stage saves the processed ciphertext and trace arrays in a compressed NumPy artifact for subsequent analysis.

### 3. AES Leakage Model

File: `03_AESLeakageModel.ipynb`

Constructs leakage hypotheses for all 256 possible key-byte guesses across the 16 byte positions.

The model uses the AES inverse S-box, Hamming weight, and a Hamming-distance leakage assumption to estimate the relationship between hypothetical intermediate values and measured power consumption.

The generated hypothesis matrices are saved for the CPA attack.

### 4. Correlation Power Analysis

File: `04_CPA_Attack.ipynb`

Implements CPA to identify candidate key bytes by correlating predicted leakage with measured power traces.

For each byte position, the notebook:

- Evaluates all 256 possible key-byte guesses.
- Computes Pearson correlation scores across trace sample positions.
- Identifies the strongest absolute correlation for each guess.
- Selects the candidate with the highest correlation score.
- Records the candidate byte, peak sample position, and correlation statistics.
- Generates correlation plots for individual byte positions.

The selected bytes are assembled into a candidate 16-byte round-10 key. Results are saved as a CSV file and a hexadecimal text file.

### 5. Minimum-Trace Analysis

File: `05_minimumTraces.ipynb`

Investigates how many power traces are required to recover the candidate key under the configured experimental conditions.

The notebook includes:

- Incremental CPA calculations at selected trace-count checkpoints.
- A coarse scan followed by a finer search around the observed recovery threshold.
- One-trace-resolution analysis within a configured interval.
- Randomized trace-order experiments.
- Estimates of full-key recovery rates across multiple trials.
- Wilson 95% confidence intervals for observed recovery proportions.
- Comparisons of recovery results at adjacent trace counts.

A key byte is considered successfully recovered when the expected candidate is the unique highest-scoring guess.

The minimum-trace result is empirical and depends on the dataset, trace ordering, leakage model, sample window, and success criterion.

## Methodology

### Correlation Power Analysis

CPA is a side-channel technique that compares hypothetical leakage values with measurements collected during cryptographic operations.

For each key-byte position, the analysis evaluates 256 possible guesses and computes the Pearson correlation between the predicted leakage and the measured traces.

The candidate with the strongest absolute correlation is selected for that byte. Repeating this process across all 16 positions produces a candidate round-10 key.

### Leakage Model

The project uses an inverse AES S-box-based intermediate and a Hamming-distance model to estimate data-dependent leakage.

The model incorporates a byte mapping derived from the AES `ShiftRows` permutation to relate ciphertext bytes to the relevant final-round intermediate values.

This model depends on assumptions about the target implementation. Its accuracy must be evaluated against the actual device and trace-acquisition setup.

### Minimum-Trace Evaluation

The minimum-trace experiment evaluates key recovery at increasing trace counts and checks whether the candidate bytes match the expected key under the notebook's configured success criterion.

Additional randomized trials estimate how sensitive recovery is to trace selection and ordering.

The observed threshold should be reported alongside the dataset, experimental settings, success criterion, and number of trials. It is not a universal guarantee for other devices or datasets.

## Requirements

- Python 3.10 or newer
- NumPy
- pandas
- Matplotlib
- Jupyter Notebook or JupyterLab

Install the dependencies using:

```bash
python -m pip install numpy pandas matplotlib jupyter
```

## Setup

Clone the repository:

```bash
git clone https://github.com/k-o-c-o/aesSca.git
cd aesSca
```

Create and activate a virtual environment.

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install numpy pandas matplotlib jupyter
```

macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install numpy pandas matplotlib jupyter
```

## Dataset

The notebooks expect a dataset named `Round10_traces.csv`.

Place the file in the following location:

```text
aesSca/
└── data/
    └── Round10_traces.csv
```

The expected dataset format is:

- The first column contains a 32-character hexadecimal ciphertext for each trace.
- The remaining columns contain measured power-trace samples.
- Each ciphertext represents 16 bytes.
- The CPA notebooks are configured around 200 samples per trace.

Check the dataset path and notebook working directory before execution. If the dataset uses a different structure or number of samples, adjust the relevant configuration and validation code.

The raw dataset is not included in the repository. Ensure that you have permission to use and analyze the data.

## Running the Notebooks

Start Jupyter from the repository root:

```bash
jupyter lab
```

Execute the notebooks in the following order:

1. `01_datasetInspection.ipynb`
2. `02_dataPreprocess.ipynb`
3. `03_AESLeakageModel.ipynb`
4. `04_CPA_Attack.ipynb`
5. `05_minimumTraces.ipynb`

The preprocessing and leakage-model notebooks must be executed before the CPA attack. The minimum-trace analysis depends on the generated preprocessing and hypothesis artifacts.

Check the configured dataset and results paths if your Jupyter environment uses a different working directory.

## Generated Outputs

The workflow generates intermediate and final artifacts under `notebooks/results/`.

Expected outputs include:

```text
notebooks/results/
├── preprocessing/
│   └── preprocessed_data.npz
├── leakage_model/
│   └── hd_hypotheses.npz
└── cpa/
    ├── fake_key_results.csv
    └── fake_round10_key.txt
```

The artifacts serve the following purposes:

| Artifact                | Purpose                                                  |
| ----------------------- | -------------------------------------------------------- |
| `preprocessed_data.npz` | Stores processed ciphertexts and power traces.           |
| `hd_hypotheses.npz`     | Stores leakage hypotheses for the 16 byte positions.     |
| `fake_key_results.csv`  | Records candidate key bytes and correlation statistics.  |
| `fake_round10_key.txt`  | Stores the candidate round-10 key in hexadecimal format. |

The CPA notebook also generates correlation plots under `notebooks/figures/cpa/`, with individual images corresponding to the 16 byte positions.

The minimum-trace notebook produces recovery tables, plots, and trial statistics as part of its analysis.

Generated result files are excluded from version control through `.gitignore`.

Note: The minimum-trace notebook may use a fallback 16-byte key if the candidate key file is missing. Verify that the key being evaluated is the intended reference before interpreting the recovery results.

## Interpretation and Limitations

- **Side-channel dependence:** CPA relies on information leaked through physical measurements, such as power consumption. It does not recover AES keys from ciphertexts alone.
- **Candidate key recovery:** A high correlation score does not automatically establish that the recovered key is correct. Validate the result against the appropriate reference or an independent verification method.
- **Implementation-specific leakage:** The assumed inverse-S-box and Hamming-distance model may not accurately represent every AES implementation.
- **Trace quality:** Noise, trace alignment, sample selection, and acquisition conditions can significantly affect the results.
- **Minimum-trace estimates:** The observed threshold depends on the dataset and experimental configuration. Results from one trace ordering should not be treated as a general guarantee.
- **Computational cost:** The shuffled-trace analysis can be expensive because it evaluates multiple trace counts, trials, byte positions, and key guesses. Reduce the number of trials or tested checkpoints during development, then document the final experimental settings.

## Troubleshooting

### Dataset not found

If the notebook raises `FileNotFoundError` for `Round10_traces.csv`, verify that the file is located at `data/Round10_traces.csv` relative to the repository root and that the configured path matches the notebook's working directory.

### Preprocessing artifacts missing

If `preprocessed_data.npz` or `hd_hypotheses.npz` cannot be found, run the preprocessing and leakage-model notebooks in order before starting the CPA attack.

### Unexpected array dimensions

Check that the dataset contains one ciphertext column followed by the trace-sample columns. Verify that each ciphertext is exactly 32 hexadecimal characters and that the saved arrays are compatible with the expected trace and hypothesis dimensions.

### Minimum-trace analysis is slow

The full analysis can require substantial computation. Reduce the number of shuffled trials or narrow the trace-count range during development. Record any configuration changes when comparing experimental results.

## Responsible Use

Use this project only with datasets and cryptographic devices that you are authorized to analyze. Side-channel measurements may contain sensitive information, so handle the dataset and any recovered key material responsibly.

## License

No license is specified in this README. Check the repository's licensing terms before redistributing or reusing the project.
