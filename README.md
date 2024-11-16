# API-Crypto-Project-using-Python
*This project using Python to call API and get data automately in order to tracking crypto price trending. Data was collectd  from CoinMarketCap.com website every miniute to create price trending and local data file csv*
<img width="432" alt="image" src="https://github.com/user-attachments/assets/788d95cb-e352-4ca0-8578-33038f37b468">

# **Definition of API**
An API (Application Programming Interface) is a set of rules and protocols that allow different software applications to communicate with each other. APIs define the methods and data formats that applications can use to request and exchange information, enabling seamless integration and functionality sharing.

In this project, we will use the BeautifulSoup package to interact with the API of the MarketCoin website to achieve the project's objectives

# **Python package using in project**
- BeautifuSoup
- OS/ time
- Pandas
- Matplot/Seaborn

## **Set up url API and get data from website**
```
from requests import Request, Session
from requests.exceptions import ConnectionError, Timeout, TooManyRedirects
import json

url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest' 
#Original Sandbox Environment: 'https://sandbox-api.coinmarketcap.com/v1/cryptocurrency/listings/latest'
parameters = {
  'start':'1',
  'limit':'10',
  'convert':'USD'
}
headers = {
  'Accepts': 'application/json',
  'X-CMC_PRO_API_KEY': 'bc558454-9599-491d-972d-92a12d355ec9',
}

session = Session()
session.headers.update(headers)

try:
  response = session.get(url, params=parameters)
  data = json.loads(response.text)
  print(data)
except (ConnectionError, Timeout, TooManyRedirects) as e:
  print(e)
```
<img width="562" alt="image" src="https://github.com/user-attachments/assets/750aa1b8-400f-4286-be22-4e8b3fc7ce3d">

## **Define function and create file csv auto get data**
```
def api_runner():
    global df
    url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest' 
    parameters = {
    'start':'1',
    'limit':'10',
    'convert':'USD'
    }
    headers = {
    'Accepts': 'application/json',
    'X-CMC_PRO_API_KEY': 'bc558454-9599-491d-972d-92a12d355ec9',
    }

    session = Session()
    session.headers.update(headers)

    try:
        response = session.get(url, params=parameters)
        data = json.loads(response.text)
    except (ConnectionError, Timeout, TooManyRedirects) as e:
        print(e)

    df2 = pd.json_normalize(data['data'])
    df2['timestamp'] = pd.to_datetime('now')
    df=df.append(df2)

    if not os.path.isfile(r'C:\Users\Vinh\OneDrive\Tài liệu\Data python\API Scraping Project\API.csv'):
        df.to_csv(r'C:\Users\Vinh\OneDrive\Tài liệu\Data python\API Scraping Project\API.csv',header='column_name')
    else:
        df.to_csv(r'C:\Users\Vinh\OneDrive\Tài liệu\Data python\API Scraping Project\API.csv',mode ='a', header=False)
```
<img width="529" alt="image" src="https://github.com/user-attachments/assets/670f6cfe-ee28-40c4-9713-4b59f9bbd9cc">


## **Visualization**

### **Trending percentage_change of top 10 coin**

```
df2 = pd.json_normalize(data['data'])
df2['timestamp'] = pd.to_datetime('now')
df=df.append(df2)
df3 = df.groupby('name', sort=False)[['quote.USD.percent_change_1h','quote.USD.percent_change_24h','quote.USD.percent_change_7d','quote.USD.percent_change_30d','quote.USD.percent_change_60d','quote.USD.percent_change_90d']].mean()
df4 = df3.stack()
df4
df5 = df4.to_frame(name='values')
df5
index = pd.Index(range(60))
df6 = df5.set_index(index)
df6 = df5.reset_index()
df6
df7 = df6.rename(columns={'level_1': 'percent_change'})
df7['percent_change'] = df7['percent_change'].replace(['quote.USD.percent_change_1h','quote.USD.percent_change_24h','quote.USD.percent_change_7d','quote.USD.percent_change_30d','quote.USD.percent_change_60d','quote.USD.percent_change_90d'],['1h','24h','7d','30d','60d','90d'])
df7
import seaborn as sns
import matplotlib.pyplot as plt
sns.catplot(x='percent_change', y='values', hue='name', data=df7, kind='point')
```
<img width="382" alt="image" src="https://github.com/user-attachments/assets/b1616fb0-fd93-4c38-ae02-9ce89f56392e">

### **Trending Bitcoin Price by Timestamp**

```
df10 = df[['name','quote.USD.price','timestamp']]
df10 = df10.query("name == 'Bitcoin'")
df10['quote.USD.price'] = df10['quote.USD.price'].apply(lambda x: f"{x/1000:.2f}K")
df10['timestamp'] = pd.to_datetime(df10['timestamp']).dt.strftime('%Y-%m-%d %H:%M:%S')
df10
sns.set_theme(style="darkgrid")

plt.figure(figsize=(10, 6))
sns.lineplot(x='timestamp', y='quote.USD.price', data=df10)
plt.xticks(rotation=90) 
plt.title("Bitcoin Price by Timestamp", fontsize=14)
plt.xlabel("Timestamp", fontsize=12)
plt.ylabel("Price (USD)", fontsize=12)
plt.tight_layout()
plt.show()
```
<img width="608" alt="image" src="https://github.com/user-attachments/assets/41dc2bf2-5e10-48d9-9493-0ce0b6761dbb">
