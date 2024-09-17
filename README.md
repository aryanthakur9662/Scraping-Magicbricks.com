# Extract real estate related data from Magicbricks.com and load it in a postgres database:
### Import the Required Libraries:
```python
import time
from time import sleep
from random import randint
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
from requests import Session
import psycopg2
```

### Create a Session:
##### This function creates an HTTP session with custom headers to mimic a regular browser, avoiding blocks from the server when scraping. It sets a User-Agent header to pretend the requests come from a real browser, avoiding detection as a bot.
```python
def create_incognito_session():
    session = Session()
    session.headers.update({
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36'
    })
    return session
```

### Define the Base URL:
##### The base URL template is designed to paginate through the pages of property listings on MagicBricks. The {page_number} placeholder will be replaced with the actual page number during scraping.
```python
base_url = "https://www.magicbricks.com/mbsrp/propertySearch.html?editSearch=Y&category=R&propertyType=10001,10017,10002,10003,10021,10022,10020&bedrooms=11700,11701,11702,11703&city=2481&page={page_number}&groupstart=90&offset=0&maxOffset=100&sortBy=premiumRecent&pType=10001,10017,10002,10003,10021,10022,10020&isNRI=N&showPrimePropsinFixedSlotsSEO=N&multiLang=en"
```

### Connect to the Postgres database:
##### Establish a connection to a local PostgreSQL database using your credentials. This will be used to store the scraped data.
```python
conn = psycopg2.connect(
    host="localhost",
    port="XXXX",
    dbname="XXXX",
    user="XXXX",
    password="XXXX"
)
```
### Create a table in Postgres <sub><sup>(If not done before)</sup></sub> :
##### Ensures that the magicbricks_listings table exists in the database before inserting any data. If the table does not exist, it is created with attributes given below.
```python
cur.execute("""
    CREATE TABLE IF NOT EXISTS magicbricks_listings (
        id SERIAL PRIMARY KEY,
        Price VARCHAR(255),
        Landmarks TEXT,
        Min_Price VARCHAR(255),
        Max_Price VARCHAR(255),
        Carpet_Area VARCHAR(255),
        Price_Sqft VARCHAR(255),
        City VARCHAR(255),
        Unit VARCHAR(50),
        Bedroom VARCHAR(50),
        Title TEXT,
        Tenants_Preference TEXT,
        Project_Name VARCHAR(255)
    );
""")
```
