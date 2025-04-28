# entiment-analysis-feedback
A Flask app for analyzing customer feedback sentiments.
# Sentiment Analysis Project - Customer Feedback (Flask App)

from flask import Flask, request, jsonify, render_template
import json
from datetime import datetime
import random

# Initialize Flask app
app = Flask(__name__)

# Dummy sentiment analyzer to replace unavailable 'transformers' module
def dummy_sentiment_analyzer(text):
    sentiments = [
        {'label': 'POSITIVE', 'score': round(random.uniform(0.7, 1.0), 2)},
        {'label': 'NEGATIVE', 'score': round(random.uniform(0.7, 1.0), 2)}
    ]
    return [random.choice(sentiments)]

# Store feedback data
feedback_data = []

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/analyze', methods=['POST'])
def analyze_sentiment():
    data = request.json
    if not data or 'feedback' not in data:
        return jsonify({'error': 'No feedback provided'}), 400

    feedback = data['feedback'].strip().lower()
    if not feedback:
        return jsonify({'error': 'Feedback cannot be empty'}), 400
    if len(feedback) > 500:
        return jsonify({'error': 'Feedback is too long (max 500 characters)'}), 400

    try:
        result = dummy_sentiment_analyzer(feedback)[0]
    except Exception as e:
        return jsonify({'error': f'Sentiment analysis failed: {str(e)}'}), 500

    # Confidence label based on score
    confidence_label = (
        'Very confident' if result['score'] > 0.9 
        else 'Confident' if result['score'] > 0.7 
        else 'Less confident'
    )

    feedback_entry = {
        'timestamp': datetime.now().isoformat(),
        'feedback': feedback,
        'sentiment': result['label'],
        'score': result['score'],
        'confidence_label': confidence_label
    }
    feedback_data.append(feedback_entry)

    return jsonify(feedback_entry)

@app.route('/feedback')
def get_feedback():
    return jsonify(list(reversed(feedback_data[-10:])))

if __name__ == '__main__':
    app.run(debug=False, port=8050)
