# 🤖 Automated Social Data Pipeline with Emotion AI (n8n + BERT)

This project builds an automated workflow using [n8n](https://n8n.io) to process social media data, integrate a fine-tuned BERT model for emotion prediction, and store the results into a SQL Server database. The entire process — from data ingestion to analysis and storage — is fully automated.

---

## 🧠 Integrated Emotion Prediction AI Model

The project uses a fine-tuned deep learning model (BERT) to predict emotions from social text content. It can classify emotions such as:

- `happy`, `sad`, `angry`, `surprised`, and more.

After training, the model is deployed as a **local REST API** using **FastAPI**, which is then connected to the n8n workflow via an `HTTP Request` node.

Integrating the AI model enables:
- Real-time, automated emotion prediction
- Easy model replacement or updates without impacting the data pipeline
- Clear separation between ETL pipeline and the emotion inference engine

---

## 🧭 Workflow Overview (n8n)

### 1. 🔘 Workflow Trigger
- **Node:** `When clicking 'Execute workflow'`
- The workflow starts when the user manually clicks "Execute" in n8n.

### 2. 📥 Load Data from Google Sheets
- **Node:** `Google Sheets - Read Sheet`
- Reads 12,000 rows of social media data from a connected Google Sheet via OAuth2.
- 🔗 [View raw input data in Google Sheets](https://docs.google.com/spreadsheets/d/11A1bMHU0LZTiXiksdAuYQiA0eLVqR4pUzlVyDnYQ3K0/edit?usp=sharing)

> Make sure the authenticated Google account in n8n has access to this sheet.


### 3. 🔄 Batch Processing (200 rows per batch)

| Node | Description |
|------|-------------|
| `Loop Over Items` | Splits the input into batches of 200 rows. Sends each batch via `Loop`; triggers `Done` after all batches are processed. |
| `Wait` | Pauses for 5 seconds between batches to avoid API rate limiting. |
| `HTTP Request` | Sends each batch to a local API (`http://host.docker.internal`) using a POST request.This API returns emotion predictions for the text content. |
| `Merge` | Merges the original batch with the API's prediction results. |
| `Replace Me` | Final node to handle post-processing or saving results. |

**Why batching?** Prevents API overload, makes logging/retrying easier, improves error handling.

---

## ⚙️ Text Cleaning & Enrichment Pipeline

| Component | Function |
|----------|----------|
| `clean_text` | Removes emojis, special characters, HTML tags, links, extra spaces, etc. |
| `Extract Date & Time Features` | Extracts timestamps and time-based features. |
| `Extract City & Country` | Infers geographic locations from the content. |
| `Map Full Language Name` | Converts language codes to readable names. |
| `Calculate Sentiment & Engagement` | Combines emotion prediction with interaction metrics. |
| `Identify Brand & Product` | Detects mentioned brands and products. |
| `Hashtags & Mentions` | Extracts hashtags (#) and mentions (@). |

---

## 🗄️ Storing Data in SQL Server

The final enriched data is stored in SQL Server using `Execute Query` nodes.

| Table | Description |
|-------|-------------|
| `Users` | Information about the user or author of the post. |
| `Brand` | Extracted brand names mentioned in content. |
| `Campaigns` | Identified campaigns or marketing initiatives. |
| `Post` | Cleaned and enriched text content. |
| `Post_Analytics` | Emotion scores and engagement metrics. |

---

## 🚀 Emotion Prediction API (FastAPI)

### Structure:
- Endpoint: `POST /predict`
- Input: `{ "text": "..." }`
- Output: `{ "label": "happy" }`

### How to Run:
```bash
# Install dependencies
pip install fastapi uvicorn torch transformers joblib

# Run the API
uvicorn emotion_api:app --reload
