# Employee Attendance and Analytics API

This project is a REST API for managing employee details and daily attendance. It supports employee creation and listing, punch-in and punch-out, attendance regularization, and attendance reports.

## Built with

- Python and FastAPI
- MongoDB
- Pydantic for request validation

## Run locally

You need Python 3.11 or later and a running MongoDB instance.

1. Install the packages:

   ```bash
   pip install -r requirements.txt
   ```

2. Set the MongoDB connection details in your environment:

   - `MONGO_URI` — your MongoDB connection string
   - `MONGO_DB` — the database name

3. From the project folder, start the API:

   ```bash
   uvicorn app.main:app --port 8000
   ```

4. Open `http://localhost:8000/docs` to try the endpoints. The health endpoint is available at `http://localhost:8000/health`.

The assessment environment can load its own test data, so database credentials and local data files are not included in this submission. Keep your `.env` file out of GitHub.
