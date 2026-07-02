import joblib
import os
import numpy as np

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
MODEL_PATH = os.path.join(BASE_DIR, "..", "model", "emotion_model.pkl")
VECTORIZER_PATH = os.path.join(BASE_DIR, "..", "model", "tfidf_vectorizer.pkl")

model = joblib.load(MODEL_PATH)
vectorizer = joblib.load(VECTORIZER_PATH)


def get_emotion_probabilities(text: str):
    vector = vectorizer.transform([text])
    probs = model.predict_proba(vector)[0]
    labels = model.classes_
    return dict(zip(labels, probs))


def analyze_emotion(text: str):
    emotion_probs = get_emotion_probabilities(text)

    sorted_emotions = sorted(
        emotion_probs.items(),
        key=lambda x: x[1],
        reverse=True
    )

    primary_emotion, primary_prob = sorted_emotions[0]
    secondary_emotion, secondary_prob = sorted_emotions[1]

    primary_intensity = int(primary_prob * 100)
    secondary_intensity = int(secondary_prob * 100)

    gap = primary_prob - secondary_prob

    if primary_prob >= 0.80 and gap >= 0.40:
        confidence = "High"
    elif primary_prob >= 0.55 and gap >= 0.20:
        confidence = "Medium"
    else:
        confidence = "Low"


    return {
        "primary_emotion": primary_emotion,
        "primary_intensity": primary_intensity,
        "secondary_emotion": secondary_emotion,
        "secondary_intensity": secondary_intensity,
        "confidence": confidence
    }