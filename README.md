# Spotify Podcast Pipeline

This project is an Apache Airflow DAG that automatically fetches podcast episode metadata from the Marketplace podcast RSS feed, stores episode info in a SQLite database, and downloads the audio files locally.

## Features

- Creates a SQLite table to store podcast episodes
- Fetches episode metadata daily from the Marketplace podcast RSS feed
- Loads new episodes into the SQLite database
- Downloads episode audio files to a local `episodes/` folder
- Built using Airflow's modern `@dag` and `@task` decorators for simplicity and clarity

## Setup & Requirements

- Python 3.11+
- Apache Airflow with SQLite provider
- `requests` and `xmltodict` Python libraries
- SQLite database (configured via Airflow connection `podcasts`)

Install required packages:

```bash
pip install apache-airflow apache-airflow-providers-sqlite requests xmltodict pendulum

## How It Works

### Database Setup  
The DAG creates an `episodes` table in the SQLite database (if it doesn't already exist).

### Get Episodes  
Downloads the podcast RSS feed and parses episode metadata.

### Load Episodes  
Compares fetched episodes against existing records and inserts new episodes into the database.

### Download Episodes  
Downloads audio files for new episodes to the `episodes/` directory if they don’t already exist.

## Usage

- Configure your Airflow `podcasts` connection to point to your local SQLite database.
- Place the DAG script in your Airflow DAGs folder.
- Start the Airflow scheduler and webserver.
- The DAG `podcast_summary2` will run daily by default and process new episodes.

## Notes

- The DAG uses the `@daily` schedule and does **not** catch up past runs.
- Ensure the `episodes/` directory exists in your project root to store audio files.
  - If the `episodes/` folder is not already in your project, update the `download_episodes` task as follows to create it automatically:

```python
def download_episodes(episodes):
    for episode in episodes:
        filename = f"{episode['link'].split('/')[-1]}.mp3"
        audio_path = os.path.join("episodes", filename)
        if not os.path.exists("episodes"):
            os.makedirs("episodes")
        if not os.path.exists(audio_path):
            print(f"Downloading {filename}")
            audio = requests.get(episode["enclosure"]["@url"])
            with open(audio_path, "wb+") as f:
                f.write(audio.content)

- If the second task get_episodes() hangs or has network issues, add this import and environment variable at the top of your DAG file to disable proxy:

```python
import os
os.environ['NO_PROXY'] = '*'

- Using airflow standalone is not an industry production practice. It is intended for local development and testing only. For this project, it is sufficient, but in production environments, a proper Airflow deployment setup should be used.

