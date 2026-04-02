from flask import Flask, request, jsonify
from flask_cors import CORS
import anthropic
import os

app = Flask(__name__)
CORS(app)  # Επιτρέπει κλήσεις από το Google Sites

# Βάλε το API key σου εδώ ή σε environment variable
ANTHROPIC_API_KEY = os.environ.get("ANTHROPIC_API_KEY", "sk-ant-ΒΑΛΕ_ΤΟ_KEY_ΣΟΥ_ΕΔΩ")

SYSTEM_PROMPT = """
Είσαι ο AI βοηθός του Γιώργου Γκαρίπη, Συντονιστή Τεχνικών Μελετών & Κατασκευών με έδρα τη Θεσσαλονίκη.

ΠΛΗΡΟΦΟΡΙΕΣ ΓΙΑ ΤΟΝ ΓΙΩΡΓΟ:
- Ονοματεπώνυμο: Γιώργος Γκαρίπης
- Ειδικότητα: Συντονιστής Μελετών και Κατασκευών Δομικών Έργων
- Τοποθεσία: Θεσσαλονίκη, Ελλάδα
- Τηλέφωνο: +30 6948 069 131
- Email: giorgiogaripis@gmail.com
- LinkedIn: https://www.linkedin.com/in/george-gkaripis-9aba09134
- Instagram: https://www.instagram.com/george_gkaripis/

ΥΠΗΡΕΣΙΕΣ:
1. Αρχιτεκτονική Επίβλεψη
2. Μελέτη & Κατασκευή
3. Σχέδια Εφαρμογής
4. Διαχείριση Έργων / Συντονισμός
5. Τεχνική Συμβουλευτική & Ασφάλεια

ΤΡΟΠΟΣ ΕΠΙΚΟΙΝΩΝΙΑΣ:
- Μιλάς στα ελληνικά (εκτός αν ο χρήστης μιλήσει αγγλικά)
- Είσαι ευγενικός, επαγγελματικός και χρήσιμος
- Για επαγγελματικές ερωτήσεις σχετικές με τον Γιώργο, δίνεις πληροφορίες και προτείνεις επικοινωνία μαζί του
- Για γενικές ερωτήσεις (τεχνικές, κατασκευαστικές, project management κ.λπ.) απαντάς χρήσιμα και περιεκτικά
- Αν κάποιος θέλει να κλείσει συνεργασία, δίνεις τα στοιχεία επικοινωνίας του Γιώργου
"""

@app.route("/chat", methods=["POST"])
def chat():
    try:
        data = request.get_json()
        messages = data.get("messages", [])

        if not messages:
            return jsonify({"error": "Δεν υπάρχουν μηνύματα"}), 400

        client = anthropic.Anthropic(api_key=ANTHROPIC_API_KEY)

        response = client.messages.create(
            model="claude-haiku-4-5-20251001",
            max_tokens=1024,
            system=SYSTEM_PROMPT,
            messages=messages
        )

        reply = response.content[0].text
        return jsonify({"reply": reply})

    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route("/health", methods=["GET"])
def health():
    return jsonify({"status": "ok"})

if __name__ == "__main__":
    app.run(debug=False, host="0.0.0.0", port=5000)