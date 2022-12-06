# Purchasing Power Parity: The "Starbucks Index"
![sbucks](sbucks.png "sbucks")

## Background
Purchasing Power Parity is essentially a concept that describes how goods and services should cost the same amount in different countries once exhange rates have been applied. In reality, no two items will be priced exactly the same, due to several factors, but using this concept, we can determine how pricey an item is in different countries compared to one another. 
## Starbucks
Starbucks coffee is a luxury item that remains accessible without being an essential item. Anywhere you go in the world, you can expect to order a standard Starbucks coffee and recieve the same product; even if the other menu items may rotate across the board. Essentially, Starbucks is a brand that exports its base products around the globe. If you decided to travel to another country today, you could order the same comfort drink there as you could back home. 

## So, How do the Prices Compare?
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

In the hopes of not overwhelming the audience, I hoped to present the following two sources together. I web-scraped which currency codes were accociated with each currency in the first code cell.
I also collected information that I believed may partially explain why the prices were different across the globe in the second code cell.
The second code contained a lot of irrelevant information, so I dropped quite a few columns of information. The remaining columns were "Country" "Pay Peroid"(hourly) "Time"(2021) "Unit Code"(USD) and "Value"the actual amount

**Clearly, this data needs to be cleaned and combined in order for us to gleam any relevant information from it.**

**So far, almost every graph I selected had at least one column in common:** **_Country_**
There was one exception, as mentioned above, and it will be addressed later

```
def UPCOUNTRY(column):
  newvariable = column.upper()
  return newvariable

def Apply_to_all(table,value):
  uppercase = table.apply(UPCOUNTRY, value)
  table = table.append_column("COUNTRY", uppercase).drop(value)
  return table

costofacoffee = Apply_to_all(costofacoffee,'Country')
tax_rates = Apply_to_all(tax_rates, "country")
Min_Wage = Apply_to_all(Min_Wage, "Country")
codes = Apply_to_all(codes, "Country")
```

Here, I applied a function to each table with a column "Country" to ensure that the values in the column were all formatted in the same way using uppercase letters. This ensured that all values representing Canada were spelled "CANADA" and so on for each country in the tables.

**Then, I paused to do some additional data cleaning**

```
strarray= Min_Wage.column("Value")
for value in strarray:
  value = float(value)
floatarray=strarray

Min_Wage = Min_Wage.append_column("Float_Values", floatarray)
```
```
convertedCAD = Min_Wage.apply(lambda x: x * 1.34, "Float_Values")
Min_Wage = Min_Wage.append_column('CAD_MinWage/hour', convertedCAD)
Min_Wage = Min_Wage.drop("Value", "Float_Values", "Unit Code")
```

The values previously returned by the Min_Wage table I created before were formatted as strings. In order to eventually do calculations using this information, I needed to conver the values into floats. 
Following that, I had to convert the information from USD to CAD so that it would be the same currency as the cost of coffee which had been calculated earlier. 
To do so, I used a lambda calculation to manually convert the values using the exchange rate between USD and CAD: 1.34.
After doing these conversions, I was able to drop the extra columns I created or no longer had use of. 

### joining the tables

```
almostfull = costofacoffee.join('COUNTRY', tax_rates, 'COUNTRY')
almostfull = almostfull.join('COUNTRY', Min_Wage, 'COUNTRY')
almostfull = almostfull.join("COUNTRY", codes, 'COUNTRY')
```

One at a time, I joined each table to eachother using the values in the "Country" columns I had worked on before. Some graphs had different information; however, if the data was not present in each data source it was not stored in the new 'almostfull' table. Personally, I chose to lose this data because I only needed 10 countries (including Canada) in my final data source. 
After this step, there were 21 countries remaining in the data.

** Now, there was still one table of information I had collected which was not included in my final table**

```
full= almostfull.join("Code", conversion_codes, "Currency\xa0Code")
full= full.drop("incomeTax","corpTax","Currency","Number","Currency Description","Pay period", "Time")
```
If you recall, one table did not have a value for "Country" but did have currency codes. This graph was essential to the problem as it contained the conversion rates of each currency to CAD.
I was able to use the Currency Codes to join this graph to the others thanks to the 'codes' table which had been joined earlier. 
Surprisingly, quite a bit of data was lost here as well.
In the end, there was a total of **18 countries** in the data.

# Now that All of the Information is together, we need to Clean it

```
def scrubadub(clean):
  clean = clean[1:]
  clean = float(clean)
  return clean
def adub(clean):
  clean = float(clean)
  return clean
```

Scrubadub removed the first index from a string. This was intended to remove any values such as a dollar sign from currency values. Following this removal, the previous string is changed to a float value so the calculations can be made using the data.
Adub is very similar; however, it is for columns that did not contain currency indicators such as a dollar sign. This will return numerical values as floats.

```
clean = full.apply(scrubadub,"Cost (CAD$)")
cleaner = full.apply(adub,"Rate")

full = full.append_column("Cost of Coffee",clean).drop("Cost (CAD$)")
full = full.append_column("Rate",cleaner)
```
As such, for the colums in full which needed to be cleaned, I ran the array values of the columns through the functions and appended them back into the table.

```
def conversion(value, rate, tax):
  value = value / rate
  tax = tax / 100
  tax = tax * value
  value = value + tax
  value = round(value,2)
  return value
```
```
Conversion = full.apply(conversion,"Cost of Coffee","Rate","salesTax")
full = full.append_column("Cost in og units w tax", Conversion)
full = full.drop("salesTax","Rate","Ranking $$coffee")
full = full.relabel("Cost of Coffee", "Cost of Coffee(CAD)")
full = full.relabel("CAD_MinWage/hour", "MinWage/hour(CAD)")
```
In order to meet some of the requirements of the assignment, I had to 'backtrack' a bit. 

Here, I am taking the cost of a coffee in CAD, converting it into it's original units, and then adding the price of sales tax to the value (rounded to 2 decimal places). Once I had the new information I once again dropped any irrelevant data that was no longer needed to make any conversions. 

```
StarbucksIndex = full
StarbucksIndex = StarbucksIndex.sort("Cost of Coffee(CAD)")
```

Finally, I was satisfied with my data, especially since my data source was small enough that I could visually confirm that were were no odd values remaining in my data. (ex. no nan values, no strings, and no gaps in the data).
As such, I made sure my table was renamed something that was relevant to the purpose of the project, and I organised it from least to most expensive coffee prices (CAD).

The final column values were: 
"Code"	"COUNTRY"	"MinWage/hour(CAD)" 	"Cost of Coffee(CAD)"	"Cost in og units w tax"

Ex.
"CAD"	"CANADA"	"14.1504" 	"4.15"	"4.36"


# The data is Clean, Now What?
## Presenting the Graphs

The first graph required should display the difference of prices in other countries compared to Canada.
As such, I first needed to calculate the difference

```
def difference(country):
  canada = StarbucksIndex.column("Cost of Coffee(CAD)")[7]
  country = country - canada
  return country

subtraction = StarbucksIndex.apply(difference,"Cost of Coffee(CAD)")

StarbucksIndex = StarbucksIndex.append_column("Difference", subtraction)
```

Here, I chose to use the index of Canada, as the data had already been sorted. However, this code would need to be changed to accomodate the data if the table were presented in any other order. 
For our current purposes, I was able to simply use the index of the graph to select for the value of canadian money. 
Then, using apply, I was able to calculate the difference for each country as it compared to the price of Canadian Coffee. 
Once again, I appended the new array I created to the original table. 

```
StarbucksIndex.barh("COUNTRY", "Difference")
```

![difference cost of coffee](https://user-images.githubusercontent.com/113123766/205983370-18b0853b-0ef5-40d6-8636-e1166a140495.png)

This graph is on a scale of -2 to 3, so the price of coffee fluxuates by about $5 depending on where you go. You can find the cheapest cup of coffee in columbia for $2.69, and you can get the most expensive cup in Luxembourg for $6.82.

```
StarbucksIndex.scatter("Difference","MinWage/hour(CAD)")
StarbucksIndex.scatter("Difference","MinWage/hour(CAD)", fit_line = True)
```


# How does Minimum Wage effect 

![WageGraph_noline](https://user-images.githubusercontent.com/113123766/205983383-342dcd40-918c-47e1-bace-04a64c4cbf7d.png)



![WageGraph_withline](https://user-images.githubusercontent.com/113123766/205983385-904417da-3f42-408e-bce5-e13b04bb5196.png)

theme: minimal
