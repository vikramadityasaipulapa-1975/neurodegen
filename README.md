<img width="1396" height="808" alt="04-two-tier-screening-vision" src="https://github.com/user-attachments/assets/930e0283-d6d5-4fbd-967c-c87e9f65207a" /># Genetic Risk Stratification for Neurodegenerative Disorders

Early identification of Alzheimer's and Parkinson's disease using genomic and RNA-based signatures together with clinical data. The project is a joint effort between Solaiera (STEOAIS Technologies Private Limited) and VNRVJIET, prepared in the context of a BIRAC Bio-AI proposal under India's BioE3 policy.

Both diseases have a long silent phase, often fifteen to twenty years, during which damage accumulates before symptoms appear. By the time a standard cognitive or motor examination picks something up, a great deal of neural loss has already happened. This project looks for signals that show up earlier, in genetic variants, molecular biomarkers and routine clinical measurements, so that people at higher risk can be identified while intervention is still useful.

## How the work is organised

The project runs two parallel tracks, one for Alzheimer's and one for Parkinson's. Each track has a clinical model, built from cognitive, motor, non-motor and demographic measurements, and a lab and genetic model, built from genes, APOE status and biomarkers. The idea is to first estimate risk from inexpensive clinical data and then decide whether genomic analysis is worth doing, and finally to combine both views into one assessment.

![System architecture]
<img width="1396" height="845" alt="01-system-architecture" src="https://github.com/user-attachments/assets/8ecaf9af-9a13-4774-a739-908fc5a24790" />

## Data

Alzheimer's data comes from the Alzheimer's Disease Neuroimaging Initiative (ADNI). The curated working set has 3,252 patient visits and 134 attributes. Parkinson's data comes from the Parkinson's Progression Markers Initiative (PPMI), using the curated public data cut released on 11 May 2026, which has 19,450 longitudinal observation records and 212 features, split into a clinical frame of 167 attributes and a laboratory and imaging frame of 45.

The raw data is not included in this repository. ADNI and PPMI are access-controlled and their data use agreements do not allow redistribution, so anyone reproducing the work needs to apply for access directly through the respective portals and place the files locally. The datasets folder should stay out of version control.

## Data preparation

Relevant tables were extracted from the large and heterogeneous releases, and the feature list was shaped by a literature review and by consultation with neurosurgeons about which measurements matter clinically. The workflow then moves through target definition, cleaning, feature engineering, scaling, class imbalance handling and label encoding, a train and test split, and five-fold cross-validation. The clinical dataset uses an 80 and 20 split and the laboratory dataset uses 85 and 15, stratified where needed.

![Data preparation workflow]

<img width="1147" height="1300" alt="02-data-preparation-workflow" src="https://github.com/user-attachments/assets/4b32cc97-4fad-45a7-9516-208848464d33" />


One note on the Parkinson's data from the working notes: rather than filtering to a single event ID, the earliest available visit per feature is used for each patient, missing genetics are encoded with a has_wgs flag and zero-filled variants instead of being dropped, and the prodromal group, patients who have not yet converted to Parkinson's, is deliberately preserved because it is the most valuable label for early prediction.

## Models and targets

For Alzheimer's, the clinical model uses XGBoost, LightGBM and Random Forest with ADGROUP as a binary target, and the lab and genetic model uses XGBoost with AD_STATUS as a binary target. For Parkinson's, the clinical model uses the same family of tree-based models with PRIMDIAG as a binary target, recoded so that all diagnoses other than Parkinson's map to zero, and the lab and genetic model uses logistic regression on targeted carrier flags with a binary progression event as the target.

The Alzheimer's lab and genetic model follows a strict protocol designed to avoid leakage. Only subjects who are cognitively normal or have mild cognitive impairment at baseline are kept, features that essentially encode the diagnosis, such as MMSE, CDR and amyloid PET, are excluded, hyperparameters are tuned with Optuna on out-of-fold PR-AUC, the decision threshold is chosen using repeated cross-validation on training data only, and the held-out test set is touched exactly once at the end. SHAP is used to explain the final model.

![Alzheimer's training pipeline]



## Longer-term vision

The feasibility blueprint describes a two-tier public health pathway. Primary health centres would run a cheap first screen using clinical-note text and a short voice recording, and only people flagged as high risk would be referred for a blood-based DNA and RNA panel at a central lab. Specialist models for DNA, RNA, voice and clinical notes would be trained separately and then combined through a lightweight late-fusion layer. This is a roadmap concept and is not part of the current prototype.

![Proposed two-tier screening pathway]
<img width="1396" height="808" alt="04-two-tier-screening-vision" src="https://github.com/user-attachments/assets/e58a1612-6ca2-44f2-8617-f07e9b7cbe60" />

## Status and limitations

This is an early prototype built with limited domain-expert support, at technology readiness level 2. Performance numbers are not reported in this README. The plan is to work with clinicians on feature engineering, validation and a knowledge base, and to extend toward RNA signatures, Indian cohorts and explainable clinical reporting.

## Team

Aman Mouzan, Vikramaditya Sai Pulapa and Sumukh Adiraju, with Dr. Mangatayaru (HoD, IT, VNRVJIET) and Dr. Rajesh (VNRVJIET) on programme supervision, Dr. Suma Kandukuri (Suma Neurocare) as clinical subject matter expert, and Murthy Parupudi as AI enterprise architect.

## Regenerating the figures

The diagrams in the images folder are produced by make_diagrams.py using matplotlib. Run it from the repository root to rebuild them.
