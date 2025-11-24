# Spotify Hit Prediction

Predicting Billboard Hot 100 hits from audio features and artist metadata. This project validates published benchmarks (85-89% audio-only, 90%+ with artist data) through a three-stage modeling progression.

## Key Results

| Model | Accuracy |
|-------|----------|
| Baseline (1960-2019) | 78.9% |
| Audio-Only (2000-2019) | 85.4% |
| Augmented (+ artist data) | 90.8% |
| Temporal Validation (train 00s, test 10s) | 89.3% |

The core finding is that 47% of prediction power comes from artist popularity and follower count, not the audio itself. Temporal segmentation (+6.5%) contributed more than artist augmentation (+5.5%).

## Repository Structure
```
spotify-hit-prediction/
├── data/
│   ├── dataset-of-*.csv          # Spotify audio features by decade
│   └── artist_cache.pkl          # Cached Spotify API artist metadata
├── eda/
│   ├── eda.ipynb                 # Exploratory data analysis
│   ├── eda_summary.md            # EDA findings
│   └── figures/                  # EDA visualizations
├── models/
│   ├── modeling.ipynb            # Model training and evaluation
│   ├── modeling_summary.md       # Modeling report
│   └── figures/                  # Model performance charts
├── presentation/
│   ├── presentation_figures.ipynb
│   ├── billboard-hit-prediction-slides.pdf
│   └── figures/                  # Presentation visuals
├── FINAL_REPORT.md               # 3-page technical whitepaper
├── requirements.txt
└── README.md
```

## Dataset

The Spotify Hit Predictor Dataset contains 41,106 tracks from 1960-2019 with 15 audio features and binary labels indicating Billboard Hot 100 appearance. Classes are perfectly balanced with 20,553 hits and 20,553 flops.

Source: [theoverman/Kaggle](https://www.kaggle.com/datasets/theoverman/the-spotify-hit-predictor-dataset)

## Methodology

**Progression 1: Baseline**
Random Forest on full 1960-2019 dataset with default parameters.

**Progression 2: Audio-Only**
Temporal segmentation restricts training to 2000-2019. Musical characteristics defining hits changed dramatically across six decades. Acousticness dropped from 0.62 in the 1960s to 0.22 in the 2000s. Training on all eras dilutes the model with outdated patterns.

**Progression 3: Augmented**
Artist metadata fetched via Spotify API are artist_popularity (0-100) and artist_followers. These two features capture 47% of the model's decision weight, confirming that who made the song matters more than what it sounds like.

## Setup
```bash
pip install -r requirements.txt
```

For artist augmentation, create `credentials.py`:
```python
SPOTIFY_CLIENT_ID = 'your_client_id'
SPOTIFY_CLIENT_SECRET = 'your_client_secret'
```

## Reports

**EDA Summary:** `eda/eda_summary.md`  
**Modeling Report:** `models/modeling_summary.md`  
**Final Report:** `FINAL_REPORT.md`

## Limitations

The 9% error rate reflects factors the model cannot capture are TikTok viral moments (84% of 2024 chart hits), TV/film sync placements, and social contagion effects that create 20-30% random variance in hit success. The augmented model also creates a paradox for talent discovery. It needs artist popularity to work, so it cannot identify breakthrough potential in unsigned artists.

## Author

Kevin Rutledge