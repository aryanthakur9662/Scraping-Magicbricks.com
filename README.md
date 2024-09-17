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
##### The base URL template is designed to paginate through the pages of property listings on MagicBricks. The ***{page_number}*** placeholder will be replaced with the actual page number during scraping.
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
##### Ensure that the ***magicbricks_listings*** table exists in the database before inserting any data. If the table does not exist, create with attributes given below.
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

### The SQl insertion query for dynamic data insertion:
##### This loop goes through the pages ***as specified by num_pages***. For each page, it Formats the ***base_url*** with the correct ***page_number*** and sends an HTTP GET request using the session to fetch the page.
```python
insert_query = """
    INSERT INTO magicbricks_listings (Price, Landmarks, Min_Price, Max_Price, Carpet_Area, Price_Sqft, City, Unit, Bedroom, Title, Tenants_Preference, Project_Name)
    VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
"""

for page in range(1, num_pages + 1):
    url = base_url.format(page_number=page)
    
    response = session.get(url)
```

### Process the API response:
##### Once a successful response ***(status code 200)*** is received, Convert the response to JSON and check if the key ***resultList*** is present. Iterate through each property and extract attributes and execute the SQL INSERT query for each property, by adding its data to the PostgreSQL table.
```python
    if response.status_code == 200:
        data = response.json()  
        
        if 'resultList' in data:
            for property in data['resultList']:
                record = (
                    property.get("price"),
                    "; ".join(property.get("landmarkDetails", [])),  
                    property.get("minPrice"),
                    property.get("maxPrice"),
                    property.get("caSqFt"),
                    property.get("sqFtPrice"),
                    property.get("ctName"),
                    property.get("coverAreaUnitD"),
                    property.get("bedroomD"),
                    property.get("auto_desc"),
                    property.get("tenantsPreference"),
                    property.get("prjname"),
                )
                
                cur.execute(insert_query, record)
```

### Handle the errors and page control:
##### Print an error and breaks the loop, stopping further page requests. This helps avoid redundant requests when there’s a problem.
```python
 else:
            print(f"No results found on page {page}")
            break  
    else:
        print(f"Failed to retrieve data on page {page}, Status code: {response.status_code}")
        break  
```
### Random delay in seconds:
##### Introduce a random delay ***(between 2 to 5 seconds)*** after each page request to avoid overwhelming the server and being blocked for scraping too fast.
```python
 sleep_time = randint(2, 5)  
 print(f"Sleeping for {sleep_time} seconds...")
 time.sleep(sleep_time)
```
### Close the Database Connction
##### After all data is scraped and inserted into the database, Ensure that changes are saved ***(commit())*** and then closes the database connection and cursor properly.
```python
conn.commit()

cur.close()
conn.close()

print("Data has been written to the PostgreSQL database.")
```
