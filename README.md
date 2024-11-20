# -Mutual-fund-data-scraping-
Mutual funds are investment vehicles that pool money from multiple investors to purchase a diversified portfolio of securities such as stocks, bonds, and other assets. NAV (Net asset value) is the market price of a fund that is dependent on the day to day stock market activity.


import requests
import pandas as pd
from sqlalchemy import create_engine
from datetime import datetime

CREATE DATABASE mutual_fund_db;

USE mutual_fund_db;

CREATE TABLE mutual_fund_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    scheme_code VARCHAR(20) NOT NULL,
    scheme_name VARCHAR(255) NOT NULL,
    nav FLOAT NOT NULL,
    date DATE NOT NULL,
    UNIQUE (scheme_code, date)
);

DB_HOST = 'localhost'
DB_USER = 'root'
DB_PASSWORD = 'password'
DB_NAME = 'mutual_fund_db'
TABLE_NAME = 'mutual_fund_data'

BASE_URL = "https://portal.amfiindia.com/DownloadNAVHistoryReport_Po.aspx"

def fetch_data(start_date, end_date):
    url = f"{BASE_URL}?frmdt={start_date}&todt={end_date}"
    try:
        response = requests.get(url)
        response.raise_for_status()
        # Assume response content is a CSV
        return pd.read_csv(response.content.decode('utf-8'))
    except requests.RequestException as e:
        print(f"Error fetching data: {e}")
        return None
def clean_data(df):
    if df is None or df.empty:
        return None
    df.fillna("NA", inplace=True)
    df.rename(columns={"Scheme Code": "scheme_code", "Scheme Name": "scheme_name", "Net Asset Value": "nav", "Date": "date"}, inplace=True)
    df['date'] = pd.to_datetime(df['date'], format='%d-%b-%Y').dt.date  # Parse date
    return df
def store_data(df):
    engine = create_engine(f"mysql+mysqlconnector://{DB_USER}:{DB_PASSWORD}@{DB_HOST}/{DB_NAME}")
    try:
        df.to_sql(TABLE_NAME, con=engine, if_exists='append', index=False, method='multi')
        print("Data stored successfully!")
    except Exception as e:
        print(f"Error storing data: {e}")
def main():
    today = datetime.now().strftime('%d-%b-%Y')
    print(f"Fetching data for {today}")
     raw_data = fetch_data(today, today)
    if raw_data is None:
        print("No data fetched.")
        return
cleaned_data = clean_data(raw_data)
    if cleaned_data is None:
        print("No valid data after cleaning.")
        return
    store_data(cleaned_data)
if __name__ == "__main__":
    main()
