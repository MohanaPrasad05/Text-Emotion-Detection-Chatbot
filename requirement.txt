from flask import Flask, request, jsonify
from flask_cors import CORS
from model_loader import analyze_emotion, get_emotion_probabilities

app = Flask(__name__)
CORS(app)

RESPONSE_MAP = {
    "fear": "It sounds like this situation is causing anxiety. Please remember that you're not alone. You can get through this.",
    "sadness": "This seems emotionally heavy. Take things one step at a time. I'm sure you can get through this",
    "anger": "I sense frustration. Taking a pause might help. Take a deep breeath and try to relax. Time will heal everything",
    "joy": "That sounds positive. I'm glad to hear this. May you have more moments like this.",
    "neutral": "The text does not indicate a dominant emotional state. Further context may be needed to understand the emotional undertone."
}


def emotion_drift(text):
    words = text.split()
    if len(words) < 30:
        return None

    n = len(words)
    start = " ".join(words[:n//3])
    middle = " ".join(words[n//3:2*n//3])
    end = " ".join(words[2*n//3:])

    def dominant_emotion(segment):
        probs = get_emotion_probabilities(segment)
        return max(probs, key=probs.get)

    return {
        "beginning": dominant_emotion(start),
        "middle": dominant_emotion(middle),
        "end": dominant_emotion(end)
    }


@app.route("/chat", methods=["POST"])
def chat():
    data = request.get_json(silent=True) or {}

    text = data.get("message", "").strip()
    context = data.get("context", "Personal")

    if not text:
        return jsonify({"error": "Empty input"}), 400

    analysis = analyze_emotion(text)
    primary_emotion = analysis["primary_emotion"]

    reply = RESPONSE_MAP.get(primary_emotion, "I'm here to listen.")

    drift = emotion_drift(text)

    response = {
        **analysis,
        "chatbot_reply": reply,
        "emotion_drift": drift
    }

    return jsonify(response)


if __name__ == "__main__":
      app.run(debug=False)