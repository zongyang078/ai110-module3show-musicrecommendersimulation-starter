# 🎵 Music Recommender Simulation

## Project Summary

This project simulates a content-based music recommendation system. Given a catalog of 18 songs with attributes like genre, mood, energy, danceability, valence, popularity, release decade, and mood tags, the system scores each song against a user's taste profile and returns a ranked list of top recommendations with explanations. It supports multiple scoring strategies, diversity-aware re-ranking, and formatted table output.

---

## How The System Works

### Song Features
Each song uses: genre, mood, energy (0.0–1.0), tempo_bpm, valence (0.0–1.0), danceability (0.0–1.0), acousticness (0.0–1.0), popularity (0–100), release_decade, and mood_tags (comma-separated descriptors like "euphoric,driving").

### User Profile
A user profile stores preferred genre, mood, target energy, and optionally danceability, valence, preferred decade, and preferred mood tags.

### Scoring Algorithm ("Algorithm Recipe")
The default "balanced" mode scores each song as follows:

- **Genre match:** +2.0 points if the song's genre matches the user's preference
- **Mood match:** +1.0 point if the song's mood matches
- **Energy similarity:** up to 1.0 points (calculated as `1.0 - abs(song_energy - target_energy)`)
- **Danceability similarity:** up to 0.5 points based on closeness
- **Valence similarity:** up to 0.5 points based on closeness

Songs are sorted by total score descending, and the top-k are returned with explanations.

### Multiple Scoring Modes (Challenge 2)
Five scoring strategies are available, each with different weight distributions:

| Mode | Genre | Mood | Energy | Dance | Valence | Use Case |
|------|-------|------|--------|-------|---------|----------|
| balanced | 2.0 | 1.0 | 1.0 | 0.5 | 0.5 | Default, well-rounded |
| genre-first | 3.0 | 0.5 | 0.5 | 0.3 | 0.3 | When genre matters most |
| energy-first | 0.5 | 0.5 | 3.0 | 1.0 | 0.5 | Workout/activity playlists |
| mood-first | 0.5 | 3.0 | 0.5 | 0.5 | 1.0 | Emotional/atmosphere playlists |
| discovery | 0.5 | 1.0 | 1.0 | 0.5 | 0.5 | Uses popularity, decade, mood tags |

### Diversity Penalty (Challenge 3)
When enabled, the system penalizes repeated artists (-1.0 per previous appearance) and overrepresented genres (-0.5 when a genre appears 3+ times). This prevents a single artist from dominating the top results.

### Advanced Features (Challenge 1)
The dataset includes three additional columns: popularity (0–100), release_decade, and mood_tags. The "discovery" scoring mode uses these to reward popular songs, decade preferences, and matching mood descriptors.

### Potential Biases
- Genre match contributes the most points in balanced mode, so cross-genre discoveries are rare
- The dataset is small (18 songs) with uneven genre distribution
- All users are scored with the same weights per mode — real listeners weight features differently

---

## Getting Started

### Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Run the app

```bash
python -m src.main
```

### Run tests

```bash
pytest
```

---

## Experiments

### Experiment 1: Four User Profiles (Balanced Mode)

| Profile | #1 Recommendation | Score | Key Match |
|---------|-------------------|-------|-----------|
| High-Energy Pop Fan | Sunrise City | 4.95 | genre + mood + energy |
| Chill Lofi Listener | Library Rain | 4.99 | genre + mood + perfect energy |
| Deep Intense Rock | Storm Runner | 4.96 | genre + mood + energy |
| Mellow Jazz Lover | Sunday Morning Tea | 4.90 | genre + mood + energy |

### Experiment 2: Scoring Mode Comparison (Pop Fan Profile)

| Mode | #1 Song | #2 Song | Key Observation |
|------|---------|---------|-----------------|
| balanced | Sunrise City (4.95) | Gym Hero (3.86) | Genre dominates |
| genre-first | Sunrise City (4.57) | Gym Hero (4.03) | Pop songs even more dominant |
| energy-first | Sunrise City (5.38) | Downtown Bounce (4.90) | Hip-hop song rises due to perfect energy |
| mood-first | Sunrise City (5.44) | Downtown Bounce (4.95) | Happy songs cluster at top regardless of genre |

### Experiment 3: Genre Weight Sensitivity
When genre weight was halved (2.0 → 1.0), Gym Hero dropped from #2 to #4 in the Pop Fan profile, overtaken by mood-matching songs from other genres.

### Experiment 4: Diversity Penalty
With diversity penalty enabled, Max Pulse's Downtown Bounce was penalized -1.0 for artist repetition (Max Pulse already appeared via Gym Hero), dropping from #3 to #4.

---

## Limitations and Risks

- **Small catalog:** 18 songs — real systems have millions
- **No audio analysis:** Only metadata, not actual sound features
- **Genre dominance:** In balanced mode, genre match outweighs all numerical features combined
- **No collaborative filtering:** Doesn't learn from other users' behavior
- **Static preferences:** User profiles don't evolve over time
- **No diversity by default:** Must explicitly enable diversity penalty

---

## Reflection

[**Model Card**](model_card.md)

Building this recommender showed how even simple weighted scoring can produce surprisingly reasonable results — every profile's top pick felt right. But the scoring mode comparison revealed that weight choices are really value judgments: energy-first mode promotes a hip-hop song for a pop fan because the energy is a perfect match, while genre-first mode would never surface it.

The diversity penalty experiment was eye-opening — without it, Max Pulse appears twice in the top 5 for the Pop Fan. In a real product, that would feel repetitive. The penalty is a simple fix, but it raises a deeper question: should the system optimize for relevance or variety?
