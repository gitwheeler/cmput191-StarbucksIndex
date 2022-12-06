# Purchasing Power Parity: The "Starbucks Index"
![sbucks](sbucks.png "sbucks")
# Background
Purchasing Power Parity is essentially a concept that describes how goods and services should cost the same amount in different countries once exhange rates have been applied. In reality, no two items will be priced exactly the same, due to several factors, but using this concept, we can determine how pricey an item is in different countries compared to one another. 
# Starbucks
Starbucks is a luxury item that remains accessible without being an essential item. Anywhere you go in the world, you can expect to order a standard Starbucks coffee and recieve the same product; even if the other menu items may rotate across the board. Essentially, Starbucks is a brand that exports its base products around the globe. If you decided to travel to another country today, you could order the same comfort drink there as you could back home. 

# So, How do the prices compare?
Let's explore how I compared these prices:
``` 
r = requests.get('https://www.finder.com/ca/starbucks-index') 
page = BeautifulSoup(r.text, 'html.parser') 
starbucks = page.find_all('table')[0] 
```
Here, I webscraped an online index of global starbucks prices in CAD. This source used data from 76 countries to compare the price Starbucks coffee globally. This data was the first 'table' on the website, so I was able to locate it using an index of 0. 

```
import pandas as pd
def scrape_table(table):
    df = pd.read_html(str(table))
    df = pd.DataFrame(df[0])
    return Table.from_df(df)
    
costofacoffee = scrape_table(starbucks)
costofacoffee.drop("City")
costofacoffee= costofacoffee.relabel("Ranking","Ranking $$coffee")
```

Using the data from the previous webpage, I returned a table in which I removed irrelevant information. "Ranking $$coffee" described which country had the most expensive coffees using integer values. Country" returned where the product was found, and "Cost (CAD$)" returned a string of  a dollar sign and the value of the coffee.

