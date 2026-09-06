# Computational Drug Discovery of Acetylcholinesterase Inhibitors

## Repository Structure
This repository contains the necessary scripts, data, and visualizations to perform computational drug discovery targeting human acetylcholinesterase:
* **`Data/`**: Contains the raw and processed datasets collected from the ChEMBL database.
* **`Figures/`**: Contains data visualizations and model performance plots generated during the analysis.
* **`1_BioactivityDataCollection.ipynb`**: Script for querying the ChEMBL API and retrieving baseline bioactivity data.
* **`2_ExploratoryDataAnalysis.ipynb`**: Script for filtering data, converting metrics (IC50 to pIC50), and conducting initial statistical analyses.
* **`3_DescriptorCalculation.ipynb`**: Script for calculating Lipinski drug-likeness descriptors and molecular fingerprints (PubChem).
* **`4_ModelBuilding.ipynb`**: Script for training, evaluating, and exporting various regression models to predict biological activity.

## How to Use
1. **Clone the repository** to your local machine.
2. **Install the required dependencies** using your preferred package manager (e.g., `pip install pandas numpy seaborn scikit-learn xgboost rdkit padelpy`).
3. **Execute the Jupyter Notebooks** in sequential order (1 through 4) to reproduce the data collection, exploratory data analysis, feature engineering, and model training processes.

---

## Introduction
Acetylcholinesterase (AChE) is an enzyme that regulates neurotransmission by breaking down acetylcholine, a major neurotransmitter involved in nerve signaling between neurons and muscle cells. Acetylcholinesterase inhibitors are widely studied for their potential to enhance cognitive function by preventing the rapid degradation of acetylcholine in the brain. Inhibiting AChE can help maintain higher levels of acetylcholine, which is beneficial for treating neurodegenerative diseases such as Alzheimer's disease.

Computational drug discovery methods have revolutionized the early stages of drug development by significantly reducing the time and cost required to identify potential drug candidates. Traditional drug discovery can take over a decade and cost billions of dollars, whereas computational approaches enable rapid virtual screening of extensive chemical libraries, prioritizing compounds with high predicted bioactivity. This targeted approach allows researchers to focus experimental efforts on the most promising candidates while avoiding costly and time-consuming testing of ineffective compounds.

![Drug Discovery Phases](Figures/CDD_Fig0.png)

## Purpose
The purpose of this study was to identify potential acetylcholinesterase inhibitors using computational methods. Thousands of chemical compounds were collected and analyzed to assess their potential as orally available drugs. Additionally, molecular features of the compounds were used to train a machine learning model capable of predicting the inhibitory activity of new molecules, facilitating more efficient identification of promising drug candidates.

The primary data source used was the ChEMBL database, a comprehensive repository of bioactivity data. As of February 27, 2025 (ChEMBL version 35), it contains curated information on over 2.5 million compounds, compiled from more than 92,100 research publications and 1.7 million assays.

## Process

### 1. Data Retrieval
To access relevant bioactivity data, the ChEMBL web service package was installed, and necessary Python libraries were imported:
* **Data handling and visualization:** pandas, seaborn, numpy, scipy
* **Chemical informatics:** rdkit, padelpy
* **Machine learning:** sklearn, lazypredict, xgboost

A search was performed for the target protein, human acetylcholinesterase (AChE), using the keyword "acetylcholinesterase". The ChEMBL database entry for human AChE is CHEMBL220, which was assigned to a variable for future reference.

![Search Results](Figures/CDD_Fig1.png)

Using the ChEMBL API, bioactivity data for over 9,000 molecules was retrieved, specifically targeting their IC50 values. IC50 measures the concentration required to inhibit 50% of AChE activity, where lower values indicate stronger inhibitors. This retrieved data was stored in a pandas DataFrame.

![IC50 DataFrame](Figures/CDD_Fig2.png)

### 2. Data Preprocessing
* **Handling missing values:** Molecules with incomplete bioactivity data were removed, leaving 6,641 viable molecules.

**Bioactivity Classification**
Molecules were categorized based on their IC50 values into three classes:
* **Active:** IC50 ≤ 1,000 nM
* **Inactive:** IC50 ≥ 10,000 nM
* **Intermediate:** IC50 between 1,000 and 10,000 nM

![Bioactivity Sorting Script](Figures/CDD_Fig3.png)

Each molecule's ChEMBL ID, canonical SMILES, IC50 value, and bioactivity class were stored in a new dataset. Canonical SMILES (Simplified Molecular Input Line Entry System) provide a unique string representation of a molecule's chemical structure in a linear text format.

![Final Processed Dataset](Figures/CDD_Fig4.png)

### 3. Drug-Likeness Evaluation (Lipinski's Rule of Five)
Christopher Lipinski, a medicinal chemist at Pfizer, developed descriptors to assess whether a compound is likely to be an orally bioavailable drug. These descriptors evaluate a molecule's ADME properties (absorption, distribution, metabolism, excretion).

The Rule of Five predicts whether a compound will be well-absorbed in the human body by ensuring values are generally multiples of 5:
* Molecular weight (MW) < 500 Daltons
* LogP (lipophilicity) < 5
* ≤ 5 hydrogen bond donors
* ≤ 10 hydrogen bond acceptors

These guidelines suggest that a compliant drug can cross cell membranes while remaining soluble in the gut. Violating these rules reduces the likelihood of oral bioavailability. Lipinski descriptors were calculated for each molecule in the dataset to filter out poor candidates.

![Lipinski Descriptors DataFrame](Figures/CDD_Fig5.png)

### 4. IC50 to pIC50 Conversion
The IC50 (half-maximal inhibitory concentration) values of molecules were converted into pIC50 values using a logarithmic transformation:

pIC50 = -log10(IC50 in M)

This conversion logarithmically scales the wide-ranging IC50 data into a normalized distribution, benefiting statistical analysis and machine learning optimization. 
* **Interpretation:** Higher pIC50 values indicate more potent inhibitors, while lower pIC50 values denote weaker inhibitors. 

To prevent negative pIC50 outputs, baseline IC50 values were capped at 10^9 nM (1 M) prior to conversion. Molecules with IC50 values above 1 M are practically ineffective, so capping them ensures the machine learning model is not skewed by meaningless negative pIC50 values.

![pIC50 Statistics](Figures/CDD_Fig6.png)

### 5. Removal of the Intermediate Class
The dataset was refined by completely removing molecules classified as "intermediate". Eliminating this biologically ambiguous class creates a clearer binary sorting problem (active vs. inactive), enabling the machine learning model to better distinguish the definitive features of effective inhibitors.

![Frequency Plot](Figures/CDD_Fig7.png)
![pIC50 Box Plot](Figures/CDD_Fig8.png)

### 6. Screening and Selection of Drug-Like Acetylcholinesterase Inhibitors

**Statistical Analysis of Lipinski Descriptors (Mann-Whitney U Test)**
A Mann-Whitney U test was performed to determine if the calculated Lipinski descriptors differed significantly between the active and inactive molecules. The probability results (p-values) were well below the standard alpha threshold of 0.05 across all four descriptors, confirming they are statistically significant features for predicting bioactivity.

![Mann-Whitney LogP](Figures/CDD_Fig9.png)
![Mann-Whitney MW](Figures/CDD_Fig10.png)
![Mann-Whitney NumHDonors](Figures/CDD_Fig11.png)
![Mann-Whitney NumHAcceptors](Figures/CDD_Fig12.png)

**Screening for the Best Drug-Like Inhibitors**
A scatter plot comparing molecular weight against lipophilicity (LogP) was generated to screen the dataset. Because these two descriptors exert a stronger influence on bioavailability than hydrogen bonding limits, they act as an effective filter. Active molecules falling within "Lipinski's Region" (MW < 500 Daltons and LogP < 5) represent highly promising drug candidates.

![Scatter Plot](Figures/CDD_Fig13.png)

### 7. Molecular Fingerprint Generation and Feature Selection
While Lipinski descriptors evaluate overall oral bioavailability, fingerprint descriptors encode granular structural features at the atomic level. They describe the presence or absence of specific substructures, allowing machine learning models to identify patterns directly from chemical compositions. 

**Calculation and Selection**
The PubChem fingerprint method (accessed via the `padelpy` Python wrapper) generated binary vectors of 881 features for each molecule. To optimize model performance, low-variance features (variance < 0.16) were removed using `sklearn`, reducing the descriptor set down to 146 highly informative features.

![Pubchem Fingerprint Table](Figures/CDD_Fig14.png)

### 8. Machine Learning Model Training
The optimized dataset was split 80/20 into training and testing sets. A total of 42 regression models were trained on the fingerprint features to predict pIC50 outputs. After comparing performance metrics, the `XGBRegressor` was chosen as the final model due to its superior accuracy and exceptional processing speed.

![Model R-Squared Comparison](Figures/CDD_Fig15.png)
![Model RMSE Comparison](Figures/CDD_Fig16.png)
![Model Time Comparison](Figures/CDD_Fig17.png)
![Top 5 Models Table](Figures/CDD_Fig18.png)

### 9. Model Export
The trained `XGBRegressor` model was saved as a pickle file (`.pkl`) for rapid deployment and future prediction of bioactivity for novel chemical compounds. 

![XGBRegressor Scatter Plot](Figures/CDD_Fig19.png)

## Conclusion
This study demonstrates the power of bioinformatics and computational drug discovery in accelerating the identification of potential acetylcholinesterase inhibitors. By leveraging large-scale bioactivity databases like ChEMBL, cheminformatics techniques, and machine learning models, 6,641 molecules were rapidly screened, reducing the time and cost associated with traditional drug discovery.

This highlights how bioinformatics is transforming pharmacology by uniting biological data with artificial intelligence. Computational tools are increasingly becoming the backbone of early-stage drug development, granting researchers the ability to navigate vast chemical spaces and design novel therapeutics with remarkable speed and precision.

*Code adapted from Chanin Nantasenamat, Ph.D.*