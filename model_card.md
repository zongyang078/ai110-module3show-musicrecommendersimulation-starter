# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

Give your model a short, descriptive name.  
Example: **VibeFinder 1.0**  

**SoundCompass 1.0**

---

## 2. Intended Use  

Describe what your recommender is designed to do and who it is for. 

Prompts:  

- What kind of recommendations does it generate  
- What assumptions does it make about the user  
- Is this for real users or classroom exploration  

SoundCompass suggests the top 5 songs from a small catalog based on a user's preferred genre, mood, energy level, danceability, and valence. It supports five scoring strategies (balanced, genre-first, energy-first, mood-first, discovery) and an optional diversity penalty to prevent artist/genre repetition. It assumes the user can articulate their preferences as explicit values. It is designed for classroom exploration to understand how content-based recommendation systems work, not for real users or production deployment.

---

## 3. How the Model Works  

Explain your scoring approach in simple language.  

Prompts:  

- What features of each song are used (genre, energy, mood, etc.)  
- What user preferences are considered  
- How does the model turn those into a score  
- What changes did you make from the starter logic  

Avoid code here. Pretend you are explaining the idea to a friend who does not program.

The system looks at each song in the catalog and compares it to the user's taste profile. It awards points for matching genre (the biggest bonus), matching mood, and having similar energy, danceability, and valence levels. Songs with higher total scores rank higher in the recommendation list.

Think of it like judging a cooking competition: genre match is like getting the main ingredient right (+2 points), mood match is like nailing the seasoning (+1 point), and energy/danceability/valence are like presentation and texture (up to 0.5–1.0 points each based on how close they are to the target).

Five scoring modes let you shift these weights: genre-first amplifies genre to +3.0, energy-first amplifies energy to 3x, mood-first amplifies mood to +3.0, and discovery mode adds bonus points for song popularity, release decade, and mood tags like "euphoric" or "driving."

An optional diversity penalty deducts points when the same artist appears multiple times (-1.0 per repeat) or when a genre dominates the top results (-0.5 after 2 appearances), so the recommendations feel more varied.

The system then sorts all songs by their total score and returns the top 5 with an explanation of why each song scored the way it did.

Changes from the starter logic: implemented all scoring from scratch, added danceability and valence as scoring features, built 5 scoring mode presets with a strategy pattern, added greedy diversity re-ranking, expanded the dataset with 3 new columns (popularity, release_decade, mood_tags), and formatted output using tabulate.

---

## 4. Data  

Describe the dataset the model uses.  

Prompts:  

- How many songs are in the catalog  
- What genres or moods are represented  
- Did you add or remove data  
- Are there parts of musical taste missing in the dataset  

The catalog contains 18 songs in data/songs.csv. The original starter dataset had 10 songs; 8 additional songs were added to increase genre and mood diversity. Each song now has 12 attributes: id, title, artist, genre, mood, energy, tempo_bpm, valence, danceability, acousticness, popularity (0–100), release_decade, and mood_tags (comma-separated descriptors like "euphoric,driving").

Genres represented include pop, lofi, rock, ambient, jazz, synthwave, indie pop, electronic, folk, hip-hop, and classical. Moods include happy, chill, intense, relaxed, moody, focused, energetic, and melancholy.

The dataset is small and hand-curated. Genre distribution is uneven: lofi has 3 songs while classical and folk have 1 each. Missing entirely: R&B, country, metal, reggae, and non-English music.

---

## 5. Strengths  

Where does your system seem to work well  

Prompts:  

- User types for which it gives reasonable results  
- Any patterns you think your scoring captures correctly  
- Cases where the recommendations matched your intuition  

When a user's preferred genre and mood both exist in the catalog, the top recommendations feel accurate and intuitive. Every profile tested (pop fan, lofi listener, rock fan, jazz lover) returned a top-1 result that matched both genre and mood.

The explanation system makes every recommendation transparent — users can see exactly why each song was chosen and how much each feature contributed.

Multiple scoring modes let users switch strategies depending on context. Energy-first mode works well for workout playlists, while mood-first suits emotional/atmosphere listening.

The diversity penalty successfully prevents a single artist from appearing multiple times in the top 5, which makes results feel more like a real playlist.

Discovery mode with mood tags and popularity enables cross-genre exploration that the basic modes cannot achieve.

---

## 6. Limitations and Bias 

Where the system struggles or behaves unfairly. 

Prompts:  

- Features it does not consider  
- Genres or moods that are underrepresented  
- Cases where the system overfits to one preference  
- Ways the scoring might unintentionally favor some users  

Genre weight dominance: In balanced mode, genre match (+2.0) is worth more than all other numerical features combined (~2.0 max). This means a mediocre genre match always beats a perfect mood+energy match in a different genre.

No understanding of music itself: The system only knows metadata labels — it can't tell that a "chill electronic" track might appeal to a lofi listener.

Dataset imbalance: Some genres have 3 songs while others have 1. Users who prefer underrepresented genres get fewer relevant options.

One-size-fits-all weights per mode: Every user in "balanced" mode gets the same formula, but real people weight features differently — some care mostly about energy for workouts, others about mood for studying.

Mood tag coverage is uneven: Not all songs have rich mood tags, which limits discovery mode's effectiveness for certain songs.

If used in a real product, the genre bias would create filter bubbles — pop listeners would never discover jazz, and vice versa, unless they manually switch to discovery mode.

---

## 7. Evaluation  

How you checked whether the recommender behaved as expected. 

Prompts:  

- Which user profiles you tested  
- What you looked for in the recommendations  
- What surprised you  
- Any simple tests or comparisons you ran  

No need for numeric metrics unless you created some.

Four distinct user profiles were tested: High-Energy Pop Fan, Chill Lofi Listener, Deep Intense Rock, and Mellow Jazz Lover. In all cases, the top-ranked song matched both the preferred genre and mood, which aligned with expectations.

A sensitivity experiment was run by halving the genre weight from 2.0 to 1.0. This caused noticeable ranking changes — for the Pop Fan profile, Gym Hero dropped from #2 to #4 as mood-matching songs from other genres rose in rank. This confirmed that the system's behavior is heavily dependent on weight choices.

Scoring mode comparison: Running the same Pop Fan profile across all four modes showed meaningful differences. In energy-first mode, Downtown Bounce (hip-hop) rose to #2 due to perfect energy match, displacing Gym Hero (pop). In mood-first mode, happy songs from other genres rose to the top regardless of genre.

Diversity penalty test: Without the penalty, Max Pulse appeared twice in the top 5 (Gym Hero #2, Downtown Bounce #3). With the penalty enabled, Downtown Bounce received a -1.0 artist repeat penalty and dropped to #4, making room for more variety.

Two pytest unit tests verify that recommend() returns sorted results with pop/happy first, and that explain_recommendation() returns a non-empty string. Both pass.

An adversarial edge case was also considered: a user with conflicting preferences (e.g., high energy + sad mood) would likely get poor recommendations because few songs in the catalog combine those attributes.

---

## 8. Future Work  

Ideas for how you would improve the model next.  

Prompts:  

- Additional features or preferences  
- Better ways to explain recommendations  
- Improving diversity among the top results  
- Handling more complex user tastes  

Learned weights: Instead of hand-tuning, use listening history data to learn optimal weights per user so the system adapts to individual taste over time.

Collaborative filtering: Incorporate signals from similar users' behavior, not just the current user's declared preferences, to enable "people like you also liked" recommendations.

Richer audio features: Add BPM range matching, key compatibility, and lyric sentiment analysis to go beyond metadata labels.

Dynamic diversity: Automatically adjust the diversity penalty strength based on catalog size and genre distribution, rather than using fixed penalty values.

Group recommendations: Support multiple user profiles at once to generate "group vibe" playlists for shared listening sessions.

---

## 9. Personal Reflection  

A few sentences about your experience.  

Prompts:  

- What you learned about recommender systems  
- Something unexpected or interesting you discovered  
- How this changed the way you think about music recommendation apps  

The most surprising thing was how much a single weight change reshuffles everything. Halving the genre weight from 2.0 to 1.0 completely changed the Pop Fan's top 5 — it made me realize that the "algorithm recipe" is really a set of value judgments about what matters most, not an objective truth.

The scoring mode comparison was particularly revealing. Energy-first mode promotes a hip-hop song for a pop fan because the energy is perfect, while genre-first would never surface it. Both are "correct" — they just answer different questions about what a good recommendation means.

Building this changed how I think about Spotify's recommendations. What feels like "the algorithm knows me" is really just a much more sophisticated version of the same score-and-rank approach, with weights learned from billions of interactions instead of hand-picked. The core idea — turn preferences into numbers, compare, sort — is the same.

Where human judgment still matters: deciding what features to include, how to handle edge cases (should a sad-but-energetic user get sad songs or energetic songs?), and whether the system is fair to all types of listeners. The model can optimize a score, but it can't decide what "good recommendations" means for everyone.
