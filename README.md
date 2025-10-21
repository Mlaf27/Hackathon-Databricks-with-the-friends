# **Spotify Feature-Based Model**

Our model classifies songs based on their audio features (such as energy, acousticness, danceability, and more) to group and recommend music in a smart, data-driven way.

---

## **Without User Information**

When no user data is available, recommendations rely on general trends:

- **Artist-based requests** (e.g., “Katy Perry songs”) return the most popular songs by that artist.  
- **Mood-based requests** (e.g., “Chill playlist”) use a default threshold for “chill” songs (low energy, high acousticness, etc.) and return the top globally popular tracks in that mood category.

---

## **With User Information**

When user preferences are known, recommendations become personalized:

- **Artist-based requests** (e.g., “Katy Perry songs”) return songs by that artist that best match the user’s listening style, mixing about 30% familiar and 70% new tracks.  
- **Mood-based requests** (e.g., “Chill playlist”) adapt the “chill” thresholds to the user’s own listening patterns, finding songs across Spotify that match their personalized chill profile, again with a mix of known and new discoveries.

---

## **1. Core Model (Common to Both Modes)**

**Goal:** Classify and group songs using only audio features, not user data.  

**Pipeline:**

- **Feature Inputs:**  
  energy, danceability, valence, acousticness, instrumentalness, tempo, loudness, speechiness, etc.  

- **Clustering / Embedding:**  
  Use K-Means or PCA + UMAP to project songs into a low-dimensional “mood space.”  

- **Labeling:**  
  Label clusters by dominant characteristics such as “Chill,” “Party,” “Focus,” or “Sad.”  

- **Fame / Popularity Metric:**  
  Rank within clusters by Spotify popularity score.

---

## **2. Without User Information (Cold Start Mode)**

**Logic:**  
When a user asks for a category or artist, recommendations rely on global patterns.

**Example Behaviors:**
- *“Katy Perry songs”* → returns the most popular songs by her.  
- *“Chill playlist”* → uses predefined “Chill cluster” thresholds (low energy, high acousticness, mid valence) and outputs globally top songs in that cluster.

---

## **3. With User Information (Personalized Mode)**

**Logic:**  
The personalization layer activates once user playlists or liked tracks are known.

**Steps:**

- **Build user profile vector:**  
  Calculate the average of the user’s song features (mean energy, mean valence, etc.).  

- **Find nearest neighbors among global songs:**  
  Use cosine similarity or K-Nearest Neighbors (KNN) within the feature space.  

- **Apply contextual filters:**  
  - If the query mentions an artist, return songs by that artist closest to the user’s taste vector (e.g., Katy Perry songs similar to the user’s energy/acousticness preferences).  
  - If the query mentions a mood (“Chill”), adapt the chill thresholds based on the user’s average chill song features.  

- **Diversity rule:**  
  Mix approximately 30% well-known (high popularity) songs and 70% new or less-known songs that are highly similar to the user’s taste.

---

## **4. Training Environment**

The model was trained and tested using **Databricks Playground**, leveraging PySpark for scalable data processing and MLlib for clustering and feature transformation.  

**Steps:**
1. Imported the Spotify dataset with song-level audio features.  
2. Cleaned and standardized the data using Spark ML’s `StandardScaler`.  
3. Applied dimensionality reduction with PCA and UMAP to create a “mood embedding space.”  
4. Performed K-Means clustering to group songs by mood type.  
5. Stored model outputs in Databricks tables for querying and recommendation generation.  
6. Implemented prompt-based interactions to generate user-facing responses.

---

## **5. Output Generation and Behavior**

When the model generates a playlist or recommendation, it follows a structured response format:

- **Song Name**  
- **Artist Name**  
- **Explanation** of why the song is a good fit based on the input description.  
- **Match Percentage** indicating how closely the song aligns with the user’s input or preference profile.

**Rules:**
- When a playlist is requested, provide a curated list with names, artists, and explanations.  
- When asked for songs similar to another track, include similarity reasoning and match percentage.  
- When only an activity is given (e.g., “while I cook”), the model explains why that playlist fits the described activity.

---

## **6. Example Prompts**

You can interact with the model using natural language requests like:

- `Create a playlist of 15 songs to listen to while I drive and watch the sunset.`  
- `Give me a playlist with 10 songs while I clean my room.`  
- `Give me 10 songs that are similar to Katy Perry songs.`  

Each output includes explanations and similarity percentages to make recommendations transparent and user-friendly.

---

## **7. Future Improvements**

- Expand the feature set with lyrical sentiment and genre embeddings.  
- Integrate reinforcement learning to adapt recommendations over time.  
- Deploy as an interactive web app using Databricks + Streamlit for live demos.
