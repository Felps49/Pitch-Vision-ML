# EDA Summary
---

## 1. What is your dataset and why did you choose it?

For this project I used the StatsBomb Open Data dataset, a publicly available soccer event dataset containing detailed logs for every possible action in a match. It includes passes, pressures, carries, duels, and especially shots.

I chose this dataset because:

- It is well-suited for classification problems, especially predicting whether a shot becomes a goal which is what I am thinking of doing with the dataset and for the project.
- It contains rich spatial data like pass locations, shots, crosses and anything you would ever want, which supports strong visual.
- It is well-documented, widely used in real soccer analytics work, and high quality.

---

## 2. What did you learn from your EDA?

###
After loading the Events file for one match:

- Approximately 4,000 events were recorded.
- 83 columns were present initially.
- Only 26–40 of these rows were shot events, depending on the match.
- Many columns are not relevant for analisis (goalkeeper techniques, foul-specific fields, etc.).

<img width="989" height="390" alt="image" src="https://github.com/user-attachments/assets/1c263b46-63ac-48b3-acec-36594e3ae676" />

<img width="790" height="390" alt="image" src="https://github.com/user-attachments/assets/1c3629fd-dcc3-45d2-9626-4cf628e13802" />

Shot-related columns were complete and included:

- `shot_statsbomb_xg`
- `shot_outcome`
- `shot_body_part`
- `shot_type`
- `location`
- `player`
- `team`
- `minute`

These columns form the foundation for a shot classifier.

---

### Missingness

A missingness review revealed:

- Many columns with more than 99% missing values, mostly action-specific fields.
- These can be removed without losing valuable information, but it is still cool to see all the data that is available with this dataset and how through and niche it can be.

<img width="288" height="366" alt="image" src="https://github.com/user-attachments/assets/a437bf20-6c58-45d4-a6e0-4fcccb470d1b" />

Shot-related fields had no missing values, confirming that the core data for this project is complete and clean.

---

### Visualization

Using the x/y event coordinates we are able to see and plot where different events happened in the game:

#### Shot Scatter Plots
- Shots cluster heavily in the right half-space near the top of the penalty box.
- Very few shots occur from long distances.

#### Shot Direction Vectors (Start → End Location)
- Revealed which shots were blocked, saved, or went wide.
- Showed clear directional tendencies for each team.

#### Team-by-Team Shot Maps
- Away team shots on the left pitch; home team shots on the right.
- One team preferred attacking down the right side.
- The other took more central shots.

<img width="1589" height="642" alt="image" src="https://github.com/user-attachments/assets/a97fdc32-d98e-4061-97c6-159651d3abaa" />

#### Pass Origin Heatmaps
- Highly dense activity in midfield zones.
- Wide areas used heavily for crossing.

These visualizations confirm that the dataset contains strong signals and is well-suited for classification problems involving location-based features.

<img width="752" height="559" alt="image" src="https://github.com/user-attachments/assets/250b3161-b6ba-4bf5-b933-a6159bf811b4" />

<img width="794" height="578" alt="image" src="https://github.com/user-attachments/assets/3f3fe343-08e6-47d6-a45f-fe117e3f5cac" />

---

### Goal Detection and Challenges

StatsBomb stores shot outcomes inside dictionaries such as:

{"id": 97, "name": "Goal"}

A helper function was needed to extract the `"name"` field.

Even after parsing correctly, some matches showed **0 goals detected**, even when the actual scoreline was not 0–0. This occurs because:

- Some goals may be recorded under different event types such as penalties.
- Own goals are logged separately from regular shots.
- Some events lack an explicit `"Goal"` label in the `shot_outcome` field.
- Certain competitions or matches have incomplete tagging.

This inconsistency is important and must be addressed before training a classification model.

---

## 3. What issues or open questions remain?

### Goal Labeling Difficulties

Some matches do not display any `"Goal"` outcomes in the `shot_outcome` field even though goals were scored. This means that relying only on that field may lead to incorrect labels.

To fix this:

- Identify goals using other event types (e.g., `Penalty`, `Own Goal Against`).
- Cross-reference the `shot_key_pass_id` and `related_events`.
- Validate detected goals using match metadata.

---

### Class Imbalance

Soccer naturally produces very imbalanced data:

- Only a small percentage of all events are shots.
- Only a small percentage of shots become goals.

This imbalance can cause a classifier to:

- Predict “no goal” for nearly all shots.
- Just perform poorly as a result:

---

### Missing shot_end_location

While most shots include an end location, some don't.

Possible solutions:
- Dropping rows without an end location.
- Estimate the end points.
- Building a model which doesnt take this into account.

---

### Ideas for Future

Things I can add after doing the EDA:

- shot angle calculate using shot coordinates.
- Adding distance to goal as a predictor.
- Add pressure, play pattern, or possession sequence.
- Classifying shot types (open play, set piece, counterattack).
- Using xG (shot_statsbomb_xg) as an input or comparison label.

Each of these can be good features to add when classifying shots.

---

## Conclusion

The StatsBomb dataset is a very through dataset and gives me a lot of freedom to explore and a good foundation to build a classifier.
There are still going to be some challenges but despite these challenges I believe there is a lot of meaningful information to use and a lot of fun to be had.

---
