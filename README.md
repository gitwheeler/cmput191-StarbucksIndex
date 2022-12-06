# Purchasing Power Parity: The "Starbucks Index"
![sbucks](sbucks.png "sbucks")

## Background
Purchasing Power Parity is essentially a concept that describes how goods and services should cost the same amount in different countries once exhange rates have been applied. In reality, no two items will be priced exactly the same, due to several factors, but using this concept, we can determine how pricey an item is in different countries compared to one another. 
## Starbucks
Starbucks is a luxury item that remains accessible without being an essential item. Anywhere you go in the world, you can expect to order a standard Starbucks coffee and recieve the same product; even if the other menu items may rotate across the board. Essentially, Starbucks is a brand that exports its base products around the globe. If you decided to travel to another country today, you could order the same comfort drink there as you could back home. 

## So, How do the prices compare?
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

```
tax_rates = Table.read_table("/content/drive/MyDrive/Colab Notebooks/csvData.csv")
tax_rates.drop("incomeTax","corpTax")
```
Here, I downloaded a CSV file from a webpage. After dropping the irrelevant files I was left with "country" and "salesTax" which was the percentage of sales tax applied to products globally. The original starbucks index I scrapped had accounted for this; however, I wanted to ensure I had the information for later.

```
r2 = requests.get('https://www.tpsgc-pwgsc.gc.ca/cgi-bin/recgen/er.pl?Language=E')
page2 = BeautifulSoup(r2.text, 'html.parser')
CADconversion= page2.find_all('table')[0]
conversion_codes = scrape_table(CADconversion)
```
Once again, I scrapped data from the internet. This source comes from the Government of Canada, and describes current exchange rates with other currencies. 
I was able to index by 0 to collect the table as this was the only table on this page of the site. 
This information is highly reliable due to coming from a reputable source; however, there was no direct information for which countries used which codes.

```
r3 = requests.get('https://www.iban.com/currency-codes')
page3 = BeautifulSoup(r3.text, 'html.parser')
CODESANDCOUNTRIES = page3.find_all('table')[0]
codes = scrape_table(CODESANDCOUNTRIES)
```
```
Min_Wage = Table.read_table('/content/drive/MyDrive/Colab Notebooks/hourlywage.csv').where("TIME",are.equal_to(2021)).where("Pay period", are.equal_to("Hourly")).where("SERIES", are.equal_to ("PPP"))
Min_Wage= Min_Wage.drop("COUNTRY","SERIES","Series","PERIOD","TIME","Unit","PowerCode Code","PowerCode","Reference Period Code", "Reference Period", "Flag Codes", "Flags")
```

In the hopes of not overwhelming the audience, I hoped to present the following two sources together. I web-scraped which currency codes were accociated with each currency in the first code cell and I also collected information that I believed may partially explain why the prices were different across the globe in the second code cell.
The second code contained a lot of irrelevant information, so I dropped quite a few columns of information. The remaining columns were "Country" "Pay Peroid"(hourly) "Time"(2021) "Unit Code"(USD) and "Value"the actual amount

**Clearly, this data needs to be cleaned and combined in order for us to gleam any relevant information from it.**
