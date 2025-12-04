## Goal Classifier: Can We Predict If a Shot Will Be a Goal?

The goal: given information about a shot (location, angle, body part, xG, player history, etc.), predict whether that shot will be a goal (`Goal`) or not (`No Goal`).

- Design and compare different models.
- Evaluate performance quantitatively (accuracy, precision, recall, ROC–AUC, etc.).
- Do proper error analysis (false positives / false negatives).
- Experiment with probability thresholds and calibration.

---

### Data

- **Source:** StatsBomb open data — all **75 competitions** in my `events/` folder.
- **Unit of analysis:** one row per **shot event**.
- **Total shots:** `61,173`
- **Total goals:** `6,747`
- **Base goal rate:** ≈ **11.0%**

I defined the target as:

- `Goal` (class 1) if `shot_outcome_name == "Goal"`.
- `No Goal` (class 0) otherwise.

Roughly 1 goal for every 8–9 shots, which affects metrics like accuracy and recall.

---

### Features

For each shot, I engineered features from the event data plus player history.

Geometric / context features

- `shot_x`, `shot_y` – raw StatsBomb pitch coordinates.
- `distance_to_goal` – Euclidean distance from shot location to the center of the goal.
- `shot_angle` – angle between the two goal posts as seen from the shot location.
- `shot_statsbomb_xg` – StatsBomb xG value for the shot (when available).
- `under_pressure_num` – 1 if `under_pressure` is true, else 0.

**Player-level history (aggregated across all competitions)**

Computed per `player_id` across all shots:

- `player_shots_total` – career shots in the dataset.
- `player_goals_total` – career goals in the dataset.
- `player_conv_rate` – goals / shots.
- `player_xg_sum` – total career xG.
- `player_xg_mean` – average xG per shot.
- `player_goals_minus_xg` – how much the player over- or under-performs their xG.

These features encode whether a player tends to finish above or below expectation.

**Categorical features (one-hot encoded)**

- `shot_body_part` – e.g. Left Foot, Right Foot, Head.
- `shot_technique` – e.g. Normal, Volley, Half-Volley.
- `shot_type` – e.g. Open Play, Free Kick, Penalty.
- `play_pattern` – e.g. From Counter, From Second Phase, From Corner.

After preprocessing, the feature matrix `X` has:

- 11 numeric features,
- plus multiple one-hot columns for the categorical features.

The target `y` is a binary label: `Goal` vs `No Goal`.

I used an 80/20 train–test split with stratification on the target to preserve the class imbalance in both sets.

---

### Models

I trained three different models on the same feature set:

1. **Logistic Regression**
2. **Random Forest Classifier**
3. **HistGradientBoostingClassifier**

For each model I measured:

- Training and test accuracy* 
- Test precision, recall, F1 (for the `Goal` class)  
- ROC–AUC on train and test

---

### Results

Below are the key test metrics for each model (Goal = positive class):

| Model                     | acc_train | acc_test | prec_test | rec_test | f1_test | roc_auc_train | roc_auc_test |
|---------------------------|-----------|----------|-----------|----------|---------|---------------|--------------|
| Logistic Regression       | 0.9069    | 0.9076   | 0.6946    | **0.2884**   | 0.4075  | 0.8520        | 0.8530       |
| Random Forest             | 0.9169    | 0.9058   | 0.6931    | 0.2305   | 0.3456  | **0.9862**        | 0.8501       |
| **HistGradientBoosting**  | 0.9118    | **0.9084** | **0.7119** | 0.2839   | **0.4059** | 0.8946        | **0.8670**   |

**Winner:** HistGradientBoosting gives the best combination of ROC–AUC, F1, and precision at the default 0.5 threshold.

<img width="581" height="590" alt="image" src="https://github.com/user-attachments/assets/ffe0b5db-1db4-4d64-893a-20ad692d6923" />

<img width="567" height="455" alt="image" src="https://github.com/user-attachments/assets/abb5ffbd-308c-41f7-9389-9f0d04abf097" />

<img width="613" height="470" alt="image" src="https://github.com/user-attachments/assets/ff7cd281-68dd-4e66-b2aa-74d78a9b1dfa" />

**Best F1 threshold:** 0.27.

**Best F1 threshold:** 0.5154859380562478.

---

### Confusion Matrix (HistGradientBoosting)

Using the default decision threshold of **0.5**:

<img width="572" height="457" alt="image" src="https://github.com/user-attachments/assets/383d6235-00d8-4bb3-b4e3-02f0aef61b72" />

- **TN (No Goal predicted No Goal):** 10,731  
- **FP (No Goal predicted Goal):** 155  
- **FN (Goal predicted No Goal):** 966  
- **TP (Goal predicted Goal):** 383  

## Discussion

### Strengths and Weaknesses of the Baseline Models

**Strengths**

- **High overall predictive performance**  
  All models reached ~90% accuracy, showing that shot geometry + xG + player history are strong predictors.

- **HistGradientBoosting performed best overall**  
  It delivered the strongest combination of:
  - Test accuracy (0.9084)  
  - Precision (0.7119)  
  - F1 (0.4059)  
  - ROC–AUC (0.8670)  

  This suggests that non-linear interactions (e.g., shot angle × distance × xG) matter.

- **Good generalization**  
  Training vs. test AUC values are close (train AUC: 0.8946 → test AUC: 0.8670), indicating a little overfitting.

- **Player-level historical features add meaningful signal**  
  Features such as conversion rate, xG over/under-performance, and career shot totals improved the model’s probability calibration.

---

**Weaknesses**

- **Low recall for the “Goal” class**  
  Even the best model only captures ~28% of real goals at the default 0.5 threshold.  
  Goals are rare, and the model tends to be careful when classifying something as a goal.

- **Class imbalance (89% No Goal)**  
  If I were to always predict "no goal" I would have ~89% accuracy, which makes accuracy a little misleading.

- **Hard-to-model Goals**  
  In soccer there are a lot of goals that are statistically improbable. These type of goals are oddities be it long range goals, deflections, or incredibly tight angles if something is a goal or not can be extremely difficult to predict from data alone. There is also the case of goals which are givens and the stats favor a goal but players miss a wide open net or favorable position.

- **Sparse categorical classes**  
  Rare shot types like a bicycle kick or  backheel can end up producing noisy, which may lead to some more unreliable predictions.

---

### Possible Reasons for Errors or Bias

**False Negatives (actual goals predicted as no goal)**

- Low-xG or long-range shots that models almost always reject  
- Missing contextual variables like keeper positioning, etc
- Rare or unusual shot techniques with very few training samples  

**False Positives (predicted goal but missed)**

- High-xG but poorly executed shots  
- Great shot geometry but saved by an crazy save by the goalkeeper  
- Pressure or defensive variables not fully represented  

---

## Ideas for Final Report & Future Improvements

### 1. Add More Match Context Features
StatsBomb already provides additional fields that can meaningfully boost predictive power:

- First-time shot vs. controlled  
- Open-play vs. set-piece  
- Last action before shot (pass, dribble, rebound, etc.)  
- Expanded pressure indicators  

---

### 2. Decision Threshold and Add Basic Hyperparameter Search

- Use ROC/PR curves to choose optimal probability thresholds  
  - Best recall  
  - Best precision  
  - Best F1  
- Perform hyperparameter tuning for  
  - Logistic Regression  
  - Random Forest  
  - HistGradientBoosting  

