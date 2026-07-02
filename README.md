const API_URL = "http://127.0.0.1:5000/chat";
let jitterInterval;

const capitalize = (str) => {
    if (!str) return "-";
    return str.toString().charAt(0).toUpperCase() + str.toString().slice(1).toLowerCase();
};

function analyzeEmotion() {
    const text = document.getElementById("userInput").value.trim();
    if (!text) return alert("System requires input text.");

    const btn = document.getElementById("analyzeBtn");
    const arrow = document.getElementById("arrow");
    const loading = document.getElementById("loading");
    const result = document.getElementById("result");

    btn.disabled = true;
    loading.classList.remove("hidden");
    result.classList.remove("hidden");

    // Live Jitter Simulation
    jitterInterval = setInterval(() => {
        const randomPos = Math.floor(Math.random() * 30) + 35; // Jitter around the center
        arrow.style.left = `${randomPos}%`;
    }, 120);

    fetch(API_URL, {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({ message: text, context: "Personal" })
    })
    .then(res => res.json())
    .then(data => {
        clearInterval(jitterInterval);
        loading.classList.add("hidden");
        btn.disabled = false;

        // Populate Data with % and Caps
        document.getElementById("primaryEmotion").innerText = `${capitalize(data.primary_emotion)} (${data.primary_intensity}%)`;
        document.getElementById("secondaryEmotion").innerText = `${capitalize(data.secondary_emotion)} (${data.secondary_intensity}%)`;
        document.getElementById("confidence").innerText = data.confidence; // Showing the 'Signal Accuracy'
        document.getElementById("botReply").innerText = data.chatbot_reply;

        // Settle on Intensity
        setTimeout(() => {
            arrow.style.left = `${data.primary_intensity}%`;
        }, 100);

        if (data.emotion_drift) {
            document.getElementById("driftSection").classList.remove("hidden");
            document.getElementById("driftStart").innerText = capitalize(data.emotion_drift.beginning);
            document.getElementById("driftMiddle").innerText = capitalize(data.emotion_drift.middle);
            document.getElementById("driftEnd").innerText = capitalize(data.emotion_drift.end);
        }
    })
    .catch(() => {
        clearInterval(jitterInterval);
        loading.classList.add("hidden");
        btn.disabled = false;
        alert("Neural link failed. Check local server.");
    });
}