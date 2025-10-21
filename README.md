# **Spotify Feature-Based Recommendation Model**

This project builds an intelligent song recommendation system using the **Spotify Tracks dataset**.  
The model classifies, clusters, and recommends songs based on their **audio features** (e.g., energy, acousticness, valence, tempo, etc.), and can operate in two modes:
1. **Without user information (unsupervised)**
2. **With user preferences (personalized / semi-supervised)**

---

## **1. Overview**

The system learns from the intrinsic properties of songs to create “mood clusters” such as *Chill*, *Party*, *Focus*, and *Sad*.  
When user data (playlists or listening history) is available, it personalizes these clusters by finding the closest matches to the user’s musical taste.

---

## **2. Without User Information — Unsupervised Learning**

When there’s no user dataset involved, the model operates in **unsupervised mode** using clustering algorithms.

### **Goal**
Discover natural groupings (moods) in the Spotify dataset using only song-level audio features.

### **Process**
1. **Feature Extraction**  
   Extract numerical features such as  
   `energy`, `danceability`, `valence`, `acousticness`, `instrumentalness`, `tempo`, `speechiness`, and `loudness`.

2. **Preprocessing**  
   - Normalize features using `StandardScaler`.  
   - Optionally apply **PCA** or **UMAP** for dimensionality reduction and visualization.

3. **Unsupervised Clustering**  
   Use algorithms such as **K-Means** or **Gaussian Mixture Models (GMM)** to discover mood-based clusters.  
   Each cluster represents a musical “theme” or “emotion.”

4. **Cluster Labeling**  
   Assign human-readable mood names to clusters based on dominant feature averages:  
   - High energy, high danceability → *Party*  
   - Low energy, high acousticness → *Chill*  
   - Mid valence, moderate tempo → *Focus*  

5. **Popularity Ranking**  
   Rank songs within each cluster using Spotify’s popularity metric.  
   Example: “Top 10 Chill Songs” or “Top 10 Party Tracks.”

---

## **3. With User Information — Semi-Supervised Personalization**

When user listening data or playlists are available, the system personalizes its recommendations.  
At this stage, it transitions from pure unsupervised clustering to **semi-supervised, similarity-based learning**.

### **Goal**
Use the user’s listening history to tailor recommendations that reflect their individual preferences.

### **Process**
1. **User Profile Vector**  
   - Compute the mean of all audio features from songs in the user’s playlist.  
   - This creates a single vector representing the user’s “musical fingerprint.”

2. **Similarity Search**  
   - Compare this user vector against all songs in the database using **Cosine Similarity** or **K-Nearest Neighbors (KNN)**.  
   - Identify the most similar songs (the “nearest neighbors”).

3. **Context-Aware Recommendation**
   - **Artist-based queries:**  
     Recommend songs by the specified artist that are closest to the user’s preferences  
     (e.g., “Katy Perry songs” → songs by her that align with the user’s energy/acousticness levels).  
   - **Mood-based queries:**  
     Adapt the thresholds for that mood based on the user’s previous songs labeled with that mood (e.g., redefine “Chill” based on what the user considers chill).

4. **Diversity Rule**  
   - To maintain freshness and discovery, output is mixed:  
     **30% familiar (popular)** songs and **70% new but similar** songs.

---

## **4. Supervised Learning (Optional Future Layer)**

If explicit feedback (likes, skips, ratings) becomes available, the system can evolve into a **fully supervised model** that learns user preferences directly.

Example:  
Train a classifier (Logistic Regression, XGBoost, etc.) using `(song_features → liked/disliked)` pairs to predict preference scores.

---

## **5. Training Environment**

The model was trained in **Databricks Playground** using **PySpark** for distributed data processing and **Spark MLlib** for clustering and feature engineering.

### **Training Steps**
1. Load and clean the Spotify dataset.  
2. Standardize all numerical features.  
3. Apply **PCA** to reduce noise and dimensionality.  
4. Train the unsupervised clustering model (e.g., K-Means).  
5. Store cluster assignments and centroids in Databricks tables.  
6. Build the similarity-based personalization layer for user data.  

---

## **6. Query-Based Learning in Databricks Genie**

To make the system interactive and continuously improving, the model is connected to **Databricks Genie**, allowing users to ask natural-language questions or playlist requests.  
Each query not only generates a recommendation but also **feeds back information** that helps the model refine its thresholds and improve personalization over time.

### **Example Queries**
- `Can you give me a playlist for running?`  
- `Give me a playlist with jazz music that is chill.`  
- `Give me a playlist with Katy Perry.`  
- `I want classical music as well.`  

### **How Query Training Works**
1. **User Query Understanding**  
   - Genie parses each user question and identifies **intent** (e.g., activity-based, artist-based, mood-based).  
   - It maps keywords (like “running,” “chill,” “Katy Perry”) to audio feature expectations.  
     - “Running” → high energy, high tempo  
     - “Chill” → low energy, high acousticness  
     - “Jazz” → mid acousticness, low speechiness  

2. **Dynamic Threshold Adjustment**  
   - When a user’s playlist or behavior (e.g., skipping or replaying songs) is known, Genie updates the **feature thresholds** that define moods for that specific user.  
   - Over time, the “chill” or “energetic” definitions become personalized — each query slightly **re-trains the feature averages** based on user activity.

3. **Continuous Learning Loop**  
   - Every new query provides context on what the user values (genre, tempo, mood).  
   - The model adjusts cluster boundaries and similarity thresholds, leading to more relevant and accurate playlists the next time.

4. **Model Feedback Integration**  
   - Query results and user engagement metrics (e.g., liked, skipped, saved) are stored in Databricks tables.  
   - This data serves as a **retraining signal**, improving recommendation logic on subsequent runs.

---

## **7. Output Behavior**

The model outputs structured, human-readable playlists and explanations.

### **Output Format**
Each result includes:
- **Song Name**  
- **Artist Name**  
- **Explanation**: why the song fits the user’s description or query  
- **Match Percentage**: how closely the song aligns with the user’s taste or activity context

### **Rules**
- For playlist requests: generate a list with 10–15 songs, including explanations and match percentages.  
- For similarity requests: include reason for similarity and percentage match.  
- For activity-only prompts: describe why each song fits that activity.

---

## **8. Example Prompts**

Users can interact naturally using questions or prompts such as:

- `Create a playlist of 15 songs to listen to while I drive and watch the sunset.`  
- `Give me a playlist with 10 songs while I clean my room.`  
- `Give me 10 songs that are similar to Katy Perry songs.`  

Each response provides contextual explanations and similarity metrics to ensure transparency and personalization.

---

## **9. Summary of Learning Modes**

| Scenario | Learning Type | Description |
|-----------|----------------|-------------|
| No User Data | **Unsupervised Clustering** | Groups songs into mood clusters based on audio features |
| With User Data (Playlists Only) | **Semi-Supervised / Similarity-Based** | Matches user’s profile vector to nearest songs in feature space |
| With User Queries (Genie Feedback) | **Adaptive / Incremental Learning** | Updates thresholds dynamically based on user prompts and preferences |
| With User Feedback (Likes/Skips) | **Supervised Learning** | Predicts song preference directly from labeled examples |

---

## **10. Future Improvements**

- Add **lyrical sentiment analysis** and **genre embeddings** for richer representations.  
- Implement **reinforcement learning** for continuous preference updates.  
- Build a **web-based demo** using Streamlit + Databricks for live recommendations.  
- Integrate Spotify API for real-time playback and playlist creation.
