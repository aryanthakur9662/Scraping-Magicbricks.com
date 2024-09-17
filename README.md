# Extract real estate related data from Magicbricks.com and load it in postgres database:
### Install the Required Libraries:
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

