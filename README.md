# NLP Emotion Detection

A small Flask web application that analyzes English text with the Watson NLP EmotionPredict service. The application displays scores for anger, disgust, fear, joy, and sadness, and identifies the highest-scoring emotion.

## Features

- Browser UI for submitting text for analysis.
- Flask route for rendering the application page.
- HTTP API route that sends text to the hosted Watson NLP service.
- Dominant-emotion calculation based on the largest returned score.

## Technology

- Python 3
- Flask
- Requests
- Bootstrap 4.3.1, loaded from the page's CDN link
- Watson NLP EmotionPredict service at `sn-watson-emotion.labs.skills.network`

The repository does not contain a database, local machine-learning model, frontend build system, or application configuration file. The Watson service URL and model ID are constants in `EmotionDetection/emotion_detection.py`; no environment variables are currently read.

## Architecture

```text
Browser
	|
	| GET /emotionDetector?textToAnalyze=...
	v
server.py (Flask)
	|
	| emotion_detector(text)
	v
EmotionDetection/emotion_detection.py
	|
	| POST JSON + grpc-metadata-mm-model-id header
	v
Watson NLP EmotionPredict service
```

The root route serves `templates/index.html`. That page loads `static/mywebscript.js`, which makes the API request and places the response in the page. The Python client reads the Watson response, extracts the five emotion scores, and computes `dominant_emotion` with Python's `max` function.

## Project Structure

```text
.
├── server.py                         # Flask application and HTTP routes
├── EmotionDetection/
│   ├── __init__.py
│   └── emotion_detection.py          # Watson NLP client and response parsing
├── templates/
│   └── index.html                    # Browser interface
├── static/
│   └── mywebscript.js                # Browser API request
├── test_emotion_detection.py         # Unit test source
├── .gitignore
└── LICENSE
```

## Installation

Clone the repository and enter its directory:

```bash
git clone https://github.com/ibm-developer-skills-network/oaqjp-final-project-emb-ai.git
cd oaqjp-final-project-emb-ai
```

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the two Python packages imported by the application:

```bash
python -m pip install Flask requests
```

There is currently no `requirements.txt`, `pyproject.toml`, or other dependency lock file in the repository.

## Configuration and Prerequisites

The application has no configurable environment variables. It requires:

- Python 3 and the installed `Flask` and `requests` packages.
- Outbound HTTPS access to `https://sn-watson-emotion.labs.skills.network`.
- An English text value to analyze. The code selects the model with the header `grpc-metadata-mm-model-id: emotion_aggregated-workflow_lang_en_stock`.

The external service is not included in this repository. Its availability and response determine whether analysis succeeds.

## Run Locally

Start the application from the repository root:

```bash
python server.py
```

`server.py` binds Flask to `0.0.0.0` on port `5000`. Open `http://localhost:5000/` in a browser. The built-in Flask server is intended for local development, not production hosting.

## HTTP API

### `GET /`

Returns the HTML user interface from `templates/index.html`.

### `GET /emotionDetector`

Analyzes a query-string parameter named `textToAnalyze`.

Example:

```bash
curl --get 'http://localhost:5000/emotionDetector' \
	--data-urlencode 'textToAnalyze=I am glad this happened'
```

Successful responses are plain text in this format:

```text
For the given statement, the system response is 'anger': 0.01, 'disgust': 0.01, 'fear': 0.01, 'joy': 0.95 and 'sadness': 0.02. The dominant emotion is joy
```

The numeric values in a real response come from Watson and vary by input. The service client sends this JSON body upstream:

```json
{"raw_document": {"text": "I am glad this happened"}}
```

If the local `emotion_detector` function returns `None`, Flask responds with `Invalid text! Please try again.`. With the current implementation, the Watson 400 branch instead returns a response object whose emotion values and dominant emotion are `None`, which the route formats as plain text. Other upstream status codes are not handled explicitly.

## Testing

The intended test command is:

```bash
python -m unittest test_emotion_detection.py
```

The test calls the live Watson NLP endpoint with five sample sentences and checks that the dominant emotions are `joy`, `anger`, `disgust`, `sadness`, and `fear`. Therefore, tests require network access to the external service and are not fully isolated.

At the time of writing, `test_emotion_detection.py` contains `import unittest from EmotionDetection.emotion_detection import emotion_detector`, which is invalid Python syntax. The test suite must be corrected to separate those imports before it can run; this README-only update does not modify the test file.

## Build and Deployment

There is no build step: the application is a server-rendered Flask app with a static JavaScript file and an externally hosted Bootstrap stylesheet. There is also no Dockerfile, Procfile, CI workflow, WSGI server configuration, or platform-specific deployment configuration in the repository.

For a deployment, provide Python 3, install `Flask` and `requests`, preserve the repository layout, and ensure the runtime can make outbound HTTPS requests to the Watson endpoint. The WSGI application object is `app` in `server.py`. The checked-in entry point runs the Flask development server on port 5000; a production deployment should use the hosting platform's supported production WSGI process and its configured port rather than relying on `python server.py`.

## License

This project is distributed under the Apache License 2.0. See [LICENSE](LICENSE).
