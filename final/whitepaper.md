# Shot Outcome Classification and Expected Goals Modeling Using StatsBomb Event Data
**Author: Felipe Rotelli Prado Moreira**  
Computer Science, Cal Poly San Luis Obispo  
CSC 466 Knowledge Discovery From Data  

---

## Abstract

Expected Goals (xG) is one of the most important metrics in modern football analytics because it captures the true quality of a chance instead of just whether the shot went in. In this project, I build a full xG pipeline using only StatsBomb event data from the top five European leagues (Premier League, LaLiga, Bundesliga, Serie A, Ligue 1). My goal was to see how far event information can take us without extensive tracking data.

I engineered a large set of features (distance, angle, footedness, header context, pass origin, etc.) and compared several machine learning models. The HistGradientBoosting model ended up performing the best, reaching a ROC-AUC of 0.863 and producing well-calibrated xG predictions. After tuning the decision threshold to 0.23, it achieved an F1-score of 0.51 and significantly improved recall over the default threshold.  
This whitepaper walks through the full process, from building features to evaluating calibration and error patterns on the pitch.

---

# 1. Introduction

Football is a beautiful sport filled with low-probability events. Only around one in ten shots becomes a goal, which means scoring is rare and makes outcomes noisy and hard to predict. Because of this, metrics like goals, assists, or shooting percentage can be misleading if used on their own.

Expected Goals (xG) gives us a better understanding of *shot quality* by estimating the probability that a specific shot becomes a goal.

My goal was to build a high-quality xG model using event-only data and strong feature engineering.

**Can we build a reliable shot-outcome classifier and xG probability model using only StatsBomb event data, and how much can football-specific feature engineering improve performance?**

## What I Built

To answer this question, I created a complete end-to-end pipeline that:

1. Loads and cleans StatsBomb event data  
2. Engineers 21 features informed by football knowledge  
3. Trains Logistic Regression, Random Forest, and HistGradientBoosting models  
4. Tunes the probability threshold for classification  
5. Evaluates models with ROC-AUC, F1, calibration, and subgroup analyses  
6. Produces visualizations (xG surfaces, error maps, ROC curves, etc.)  

---

# 2. Data and Feature Engineering

## Dataset

I used the free StatsBomb Open Data.

The raw dataset includes:

- 75 StatsBomb event files  
- 61,173 shots  
- 6,747 goals  
- An overall goal rate of ~11%  
- Competitions: Premier League, La Liga, Serie A, Bundesliga, Ligue 1  

## Data Processing Steps

- Filtering only events where type_name == "Shot"  
- Flattening nested StatsBomb dictionaries  
- Merging pass-origin information when a key pass exists  
- Handling missing coordinates  
- Creating a clean player identifier  
- Removing rows missing any essential geometric fields  

## Feature Engineering

### Shot Geometry
The classic xG drivers:

- Distance to goal  
- Shot angle  
- Exact (x, y) location of the shot  

### Player Shooting Tendencies
Using each player’s shot history:

- Average xG per shot  
- Historical conversion rate  
- Goals minus xG (finishing over/under-performance)  

### Footedness and Headers
I inferred each player’s preferred foot and created:

- Strong foot / weak foot indicators  
- Weak-foot penalties  
- Header flag  
- Header angle, header distance  
- Header × cross features  

### Pass-Origin Features
For shots with a key pass:

- Pass start (x, y)  
- Incoming pass angle  
- High/low/ground pass height  
- Cross indicator  

### Pressure and Location Zones
- Under pressure  
- Inside the penalty box  
- Inside the six-yard box  

All of these combined give a much more realistic picture of the context behind the shot.

---

# 3. Modeling Methodology

## Models Compared

I trained and evaluated three models:

1. **Logistic Regression** (baseline)  
2. **Random Forest**  
3. **HistGradientBoosting (HGB)** (best performer)  

## Training Setup

- 80/20 stratified train-test split  
- Random seed = 42  
- 21 numeric engineered features  

## Evaluation Metrics

I used:

- ROC-AUC (ranking ability)  
- Precision, recall, F1  
- Calibration (calibration curve + Brier score)  
- Confusion matrices  
- Subgroup ROC curves (header vs non-header, strong vs weak foot)  

## Threshold Optimization

Since goals are rare, the default 0.50 threshold causes the model to miss too many real goals.

I evaluated thresholds from 0.01 to 0.99 and found:

- **Best F1 threshold ≈ 0.23**  
- Recall nearly doubled by lowering the threshold  
- F1 increased from 0.28 → **0.51**  

<img width="613" height="470" alt="image" src="https://github.com/user-attachments/assets/b5abbb61-bb42-4fe9-8388-c4e10760c7c2" />

---

# 4. Results

## Model Performance

### Model Comparison (Test Set)

| Model | ROC-AUC | F1 (thr=0.23) | Recall | Precision |
|-------|---------|---------------|--------|-----------|
| Logistic Regression | 0.849 | 0.485 | 0.455 | 0.519 |
| Random Forest | 0.848 | 0.491 | 0.516 | 0.469 |
| HistGradientBoosting | **0.863** | **0.510** | **0.529** | **0.492** |

<img width="581" height="590" alt="image" src="https://github.com/user-attachments/assets/7a9869ee-9a85-4872-a49f-ca3b2019dcd8" />

## Calibration

The HistGradientBoosting model was very well-calibrated:

- Brier score ≈ 0.09  
- Predicted probabilities matched real scoring frequencies closely  

<img width="567" height="455" alt="image" src="https://github.com/user-attachments/assets/5e9fb24a-2abc-4caf-8e9f-be5077796875" />

## Confusion Matrix at Threshold 0.23

- True Negatives: 9,941  
- False Positives: 945  
- False Negatives: 574  
- True Positives: 775  

<img width="582" height="474" alt="image" src="https://github.com/user-attachments/assets/81b23f4e-681e-4273-8fc2-e8a640913355" />

## Feature Importance

### Table 2 — Top Features (Average Rank Across Models)

| Rank | Feature |
|------|---------|
| 1 | Player Conversion Rate |
| 2 | StatsBomb xG |
| 3 | Shot Angle |
| 4 | Player xG Mean |
| 5 | Distance to Goal |
| 6 | Header Distance Angle |
| 7 | Weak Foot Angle |
| 8 | Header Angle |
| 9 | Header Distance |
| 10 | Strong Foot Angle |

<img width="989" height="1190" alt="image" src="https://github.com/user-attachments/assets/86a0334c-1171-4b2e-b64c-b76aba44dc06" />

## Subgroup Evaluation

I checked how the model behaves across different shot types.

<p align="center">
  <img src="https://github.com/user-attachments/assets/77d4719a-3564-4721-9d65-a550bbbcaebc" width="45%" />
  <img src="https://github.com/user-attachments/assets/a6f0b7d2-20db-4fe6-b8ad-8fe3b54f30b9" width="45%" />
</p>

Findings:

- Strong-foot shots behave much more predictably  
- Headers form their own statistical pattern  
- Non-header footed shots follow classic geometry rules  

## Spatial and Error Analysis

I plotted:

- Error across the pitch (proba - y)  
- Heatmaps of false negatives and false positives  

Key issues:

- Sharp-angle finishes are often underestimated  
- Deep looping headers are difficult to classify  
- Rebound shots produce inconsistent danger signals  

<img width="674" height="489" alt="image" src="https://github.com/user-attachments/assets/a5505b43-4add-4159-bb4f-d0dc9c6ee864" />

---

# 5. Discussion

## What Worked Well

- HistGradientBoosting was clearly the strongest model  
- ROC-AUC of 0.863 is competitive with public xG implementations  
- The model produced excellent probability calibration  
- Threshold tuning dramatically improved classification metrics  
- Football-specific feature engineering was essential  

## Limitations

- No goalkeeper positioning  
- No defender proximity due to lack of tracking data  
- Some StatsBomb files lack full pass metadata  
- Ball velocity and trajectory are not available  
- Tracking-only features (body orientation, pressure intensity) are missing  

---

# 6. Conclusion

The HistGradientBoosting model achieved:

- High discriminative ability (ROC-AUC 0.863)  
- Strong calibration  
- Good recall at an optimized threshold  
- Logical, football-consistent feature importance  

Overall, this project provides a solid blueprint for building xG systems and shows what is possible without tracking data.

---

# 7. Future Work

There are several directions I would explore next:

1. Incorporating tracking data for goalkeeper and defender positioning  
2. Adding expected threat (xT) to model pre-shot danger buildup  
3. Deep learning shot-shape embeddings  
4. Modeling goalkeeper shot-stopping ability (post-shot xG)  
5. Adding team style features (pressing, buildup structure, tempo)  

---

# 8. References

- StatsBomb Open Data — https://github.com/statsbomb  

---

# Appendix: Reproducibility

This project is designed to run entirely in a **Jupyter Notebook** environment (Google Colab or JupyterLab).

### Notebook Included
- shot_classifier.ipynb — contains:
  - Data loading  
  - Feature engineering  
  - Model training  
  - Evaluation  
  - Visualizations  
  - Threshold analysis  

### How to Run

1. Open the notebook in Colab or Jupyter.  
2. Upload/mount StatsBomb event files in a directory like /content/events/.  
3. Run all cells from top to bottom.
