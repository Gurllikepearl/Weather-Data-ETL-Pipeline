# Weather-Data-ETL-Pipeline
This project demonstrates a basic ETL (Extract, Transform, Load) pipeline built using Python and executed entirely from the Command Line (CMD). The pipeline extracts real-time weather data from a public REST API, transforms the data into a structured format, and loads it into a SQLite database for storage and analysis.
scripts/
 extract.py/
 import requests
import pandas as pd
from datetime import datetime

def extract_weather():
    url = (
        "https://api.open-meteo.com/v1/forecast"
        "?latitude=12.97&longitude=77.59"
        "&current_weather=true"
    )

    response = requests.get(url).json()
    print("API RESPONSE:", response)

    if "current_weather" not in response:
        raise Exception("API ERROR:", response)

    data = {
        "city": "BANGALORE",
        "temperature": response["current_weather"]["temperature"],
        "windspeed": response["current_weather"]["windspeed"],
        "timestamp": datetime.now()
    }

    return pd.DataFrame([data])
Transform.py/
def transform_data(df):
    df["city"] = df["city"].str.upper()
    df["timestamp"] = df["timestamp"].astype(str)  
    return df
Load.py/
import sqlite3

def load_data(df):
    conn = sqlite3.connect("../data/weather.db")
    cursor = conn.cursor()

    cursor.execute("""
    CREATE TABLE IF NOT EXISTS weather_data (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        city TEXT,
        temperature REAL,
        windspeed REAL,
        timestamp TEXT
    )
    """)

    for _, row in df.iterrows():
        cursor.execute("""
        INSERT INTO weather_data (city, temperature, windspeed, timestamp)
        VALUES (?, ?, ?, ?)
        """, tuple(row))

    conn.commit()
    conn.close()
Run_etl.py/
from extract import extract_weather
from transform import transform_data
from load import load_data

df = extract_weather()
df = transform_data(df)
load_data(df)

print("ETL pipeline executed successfully")

Command to Run/
python run_etl.py

Lastly fr verification of loaded data:
import sqlite3
conn = sqlite3.connect("../data/weather.db")
cursor = conn.cursor()
cursor.execute("SELECT * FROM weather_data")
print(cursor.fetchall())
conn.close()
#CMD only

🔄 ETL Workflow
1️⃣ Extract

Fetches real-time weather data from a public REST API

Parses JSON response into structured format

2️⃣ Transform

Cleans and standardizes data

Converts timestamps for database compatibility

3️⃣ Load

Creates database table if not exists

Loads transformed data into SQLite database

⚙️ Tech Stack

Programming Language: Python

Libraries: Pandas, Requests, SQLite3

Database: SQLite

Execution: Command Line (CMD)

API: Open-Meteo Public Weather API

