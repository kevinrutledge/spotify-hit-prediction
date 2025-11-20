# Modeling Report: Spotify Hit Predictor

## Model Design & Implementation

The modeling approach follows three stages to isolate improvement sources. Baseline models on the full 1960-2019 dataset establish the floor. Temporal segmentation on 2000-2019 data tests era-specific training. Artist metadata from Spotify API completes the picture.

Three baseline algorithms ran with default parameters: Logistic Regression (71.71%), Decision Tree (70.75%), and Random Forest (78.91%). Random Forest won, confirming ensemble methods capture non-linear patterns better than simpler approaches.

![Baseline Model Comparison](figures/baseline_comparison.png)

Restricting training to 2000-2019 songs pushed Random Forest to 85.37%, a 6.46 point gain. Musical characteristics defining hits changed across decades. Acousticness dropped from 0.62 in the 60s to 0.22 in the 2000s. Loudness increased 4.5dB. Training on all eras dilutes the model with outdated patterns.

Hyperparameter tuning tested 20 configurations but achieved only 85.21% accuracy, slightly worse than default. Temporal segmentation drove the entire improvement. XGBoost hit 85.04%, confirming the 85% result stems from data and features, not algorithm choice.

## Feature Engineering & Augmentation

The dataset contains 15 audio features from Spotify's API: danceability, energy, key, loudness, mode, speechiness, acousticness, instrumentalness, liveness, valence, tempo, duration_ms, time_signature, plus chorus_hit and sections.

Audio-only feature importance showed instrumentalness dominating at 23.7%, followed by danceability (11.2%) and loudness (10.4%). Hits are vocally-driven. Instrumentalness drops 89% from flops (0.28) to hits (0.03). Danceability increases 12%.

![Feature Importance: Audio-Only](figures/feature_importance_audio.png)

Artist augmentation fetched metadata for 12,270 songs: artist_popularity (0-100) and artist_followers (total count). Mean popularity hit 53.4 with median followers of 729,000. Range spanned from zero to 165 million.

Training on 17 features (15 audio + 2 artist) jumped accuracy to 90.83%, a 5.46 point improvement. Artist_followers claimed 29.1% importance, artist_popularity took 18.3%. Together: 47.4% of decision weight. Instrumentalness dropped from 23.7% to 12.6%.

![Feature Importance: Augmented](figures/feature_importance_augmented.png)

Who made the song matters more than what it sounds like. Results match literature benchmarks: 85-89% audio-only, 90-92% with artist context.

## Model Evaluation

| Model                         | Accuracy | Precision | Recall | F1    |
| ----------------------------- | -------- | --------- | ------ | ----- |
| Baseline: Logistic Regression | 71.7%    | 68.8%     | 79.6%  | 73.8% |
| Baseline: Decision Tree       | 70.7%    | 70.8%     | 70.6%  | 70.7% |
| Baseline: Random Forest       | 78.9%    | 76.5%     | 83.4%  | 79.8% |
| Audio-Only: Random Forest     | 85.4%    | 83.1%     | 88.8%  | 85.9% |
| Audio-Only: XGBoost           | 85.0%    | 83.1%     | 87.9%  | 85.5% |
| Augmented: Random Forest      | 90.8%    | 90.4%     | 91.4%  | 90.9% |

The augmented model minimizes misclassifications to 119 false positives and 106 false negatives out of 2,454 test samples.

![Confusion Matrices](figures/confusion_matrices.png)

ROC curves show the progression. Baseline AUC: 0.71-0.87. Audio-only: 0.92. Augmented: 0.97.

![ROC Curves](figures/roc_curves.png)

Temporal segmentation contributed 6.5 points. Artist augmentation added 5.5 points.

![Accuracy Progression](figures/accuracy_progression.png)

## Strengths & Weaknesses

The 85.4% audio-only result validates research claiming 85-89% represents the acoustic feature ceiling. Temporal segmentation drove more improvement than any algorithmic choice. Random Forest with default parameters works. Hyperparameter tuning and XGBoost added no value.

The augmented model defeats its purpose for A&R (Artists & Repertoire - talent scouting) applications. Labels want to discover unknown artists before they hit. A model dependent on artist_followers can't identify breakthrough potential in unsigned acts. Songs from artists with 238-7,694 followers that still charted represent exactly the opportunities labels seek. The model missed these.

The 9.2% error rate appears irreducible. False positives reveal high-quality tracks from established artists that failed despite favorable metrics. David Guetta's "When Love Takes Over" (88 popularity, 27M followers, 67% danceability) didn't chart. Marketing timing and promotional budget determine success in these cases.

False negatives expose the opposite. "Better Off Alone" by Alice Deejay charted despite zero popularity score, 1,240 followers, and 77% instrumentalness. Viral moments override predictions.

The binary hit definition introduces noise. Any Billboard Hot 100 appearance counts equally whether the song peaked at #1 for 20 weeks or #100 for one week.

## Ideas for Final Report

Temporal validation represents the highest priority. Current evaluation uses random 80/20 splits, leaking future information into training. The correct approach trains on 2000-2015 and tests on 2016-2019. Expected accuracy will drop 3-5 points but provides realistic performance.

Feature engineering remains unexplored. Energy-loudness correlation at 0.77 suggests creating composite features. Interaction terms between danceability and valence might capture combinations that predict hits.

Cost-sensitive learning addresses A&R needs directly. Missing a hit costs more than investing in a flop. Asymmetric loss functions would catch more potential hits at the expense of false positives. Labels accept this trade.

The final report implements temporal validation first to test whether 90.8% holds under realistic conditions. Cost-sensitive learning comes second to optimize for talent discovery rather than balanced accuracy.
