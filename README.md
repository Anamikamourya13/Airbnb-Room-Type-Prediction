# NYC Airbnb Room Type Predictor

A machine learning web app that predicts the **room type** of an Airbnb listing in New York City (Entire home/apt, Private room, or Shared room) from the listing's details. The model is served through a **FastAPI** backend, and a custom HTML/CSS/JavaScript frontend is served from the same app, so the whole project deploys as a single service on Render.

**Live demo:** https://airbnb-room-type-prediction-1.onrender.com/

> The app is hosted on Render's free tier. If it has been idle for a while, the first request can take 30 to 60 seconds while the service wakes up.

---

## Features

- Predicts the room type of an NYC Airbnb listing along with class probabilities
- Input validation with Pydantic (latitude/longitude ranges, positive price, and so on)
- Interactive frontend with example listings to try without typing
- Single-service deployment: FastAPI serves both the API and the frontend
- Interactive API docs at `/docs` (Swagger UI)

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Model | scikit-learn pipeline, saved with joblib (`Model_Pipeline.pkl`) |
| Backend | Python 3.12, FastAPI, Uvicorn, Pydantic |
| Data handling | pandas, NumPy |
| Frontend | HTML, CSS, vanilla JavaScript |
| Deployment | Render (Web Service) |

---

## Project Structure

```
Airbnb-Room-Type-Prediction/
├── main.py               # FastAPI app: API routes + serves the frontend
├── Model_Pipeline.pkl    # Pre-trained scikit-learn pipeline
├── index.html            # Frontend page
├── script.js             # Frontend logic (calls the API)
├── style.css             # Frontend styling
├── requirements.txt      # Python dependencies
├── .python-version       # Python version used by Render (3.12.7)
└── README.md
```

---

## Model Inputs

The model takes the following 10 features:

| Feature | Type | Description |
| --- | --- | --- |
| `latitude` | float | Latitude coordinate (-90 to 90) |
| `longitude` | float | Longitude coordinate (-180 to 180) |
| `price` | float | Price per night, must be positive |
| `minimum_nights` | int | Minimum nights required for booking (1 to 365) |
| `number_of_reviews` | int | Total number of reviews |
| `reviews_per_month` | float | Average reviews per month |
| `calculated_host_listings_count` | int | Number of listings by this host |
| `availability_365` | int | Days available out of 365 (0 to 365) |
| `neighbourhood_group` | str | Borough (e.g. Manhattan, Brooklyn) |
| `neighbourhood` | str | Specific neighbourhood name |

**Output:** the predicted room type (`Entire home/apt`, `Private room`, or `Shared room`) and the probability for each class.

---

## API Endpoints

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/` | Serves the web frontend |
| `GET` | `/health` | Health check |
| `POST` | `/predict` | Returns the predicted room type and probabilities |
| `GET` | `/docs` | Interactive Swagger documentation |

### Example request

```bash
curl -X POST "https://airbnb-room-type-prediction-1.onrender.com/predict" \
  -H "Content-Type: application/json" \
  -d '{
    "latitude": 40.7484,
    "longitude": -73.9857,
    "price": 120,
    "minimum_nights": 2,
    "number_of_reviews": 84,
    "reviews_per_month": 2.3,
    "calculated_host_listings_count": 1,
    "availability_365": 210,
    "neighbourhood_group": "Manhattan",
    "neighbourhood": "Midtown"
  }'
```

### Example response

```json
{
  "Predicted_room_type": "Entire home/apt",
  "Probability": [0.82, 0.16, 0.02]
}
```

The values above are illustrative. Probabilities follow the class order of the trained model.

---

## Run Locally

**1. Clone the repository**

```bash
git clone https://github.com/Anamikamourya13/Airbnb-Room-Type-Prediction.git
cd Airbnb-Room-Type-Prediction
```

**2. Create a virtual environment (Python 3.12 recommended)**

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

**3. Install dependencies**

```bash
pip install -r requirements.txt
```

**4. Start the server**

```bash
uvicorn main:app --reload
```

**5. Open the app**

- Website: http://127.0.0.1:8000/
- API docs: http://127.0.0.1:8000/docs

When running locally, keep `API_BASE_URL = ""` in `script.js` so the frontend calls the same server.

---

## Deploy on Render

The frontend and backend run in one Render Web Service.

1. Push the project to GitHub.
2. On [Render](https://render.com), click **New +** and choose **Web Service**, then select this repository.
3. Use these settings:
   - **Runtime:** Python 3
   - **Branch:** `main`
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - **Instance Type:** Free
4. Make sure the repository contains a `.python-version` file with `3.12.7`. Render does not read `runtime.txt`, and without this file it may default to a newer Python version (such as 3.14) where `pydantic-core` fails to build.
5. Click **Create Web Service**. Every `git push` to `main` triggers an automatic redeploy.

### Troubleshooting

| Problem | Fix |
| --- | --- |
| Build fails with `pydantic-core` / `metadata-generation-failed` | Add `.python-version` with `3.12.7` (or set `PYTHON_VERSION=3.12.7` in Render's Environment tab) |
| Site shows only `"Hello Guyss"` | Two routes are defined on `/`. Rename the greeting route to `/health` |
| Page loads without styling or does nothing | Confirm `/style.css` and `/script.js` routes exist in `main.py` |
| `joblib.load` errors or version warnings | Pin the same `scikit-learn`, `numpy`, and `pandas` versions in `requirements.txt` that were used to train the model |
| First request is very slow | Normal on the free tier after idle time |

---

## How It Works

1. The user enters listing details in the web form (or picks an example listing).
2. `script.js` sends the data as JSON to `POST /predict`.
3. FastAPI validates the input with Pydantic and converts it to a pandas DataFrame.
4. The pre-trained pipeline in `Model_Pipeline.pkl` runs the preprocessing and prediction.
5. The predicted room type and class probabilities are returned and displayed in the UI.

---

## Future Improvements

- Add model performance metrics and feature importance to the UI
- Add dropdowns for neighbourhoods populated from the training data
- Add automated tests for the API
- Containerize with Docker

---

## Author

**Anamika Maurya**
GitHub: [@Anamikamourya13](https://github.com/Anamikamourya13)

---

## License

This project is for educational purposes. Add a license of your choice (for example MIT) if you plan to share or reuse it.
