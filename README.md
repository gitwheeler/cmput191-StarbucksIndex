# Purchasing Power Parity: The "Starbucks Index"
![sbucks](sbucks.png "sbucks")

## Background
Purchasing Power Parity is essentially a concept that describes how goods and services should cost the same amount in different countries once exhange rates have been applied. In reality, no two items will be priced exactly the same, due to several factors, but using this concept, we can determine how pricey an item is in different countries compared to one another. 

## Starbucks
> Starbucks coffee is a luxury item
> Anywhere you go in the world, you can expect to order a standard Starbucks coffee and recieve the same product
> You can find starbucks in several locations

## So, How do the Prices Compare?

``` 
r = requests.get('https://www.finder.com/ca/starbucks-index') 
page = BeautifulSoup(r.text, 'html.parser') 
starbucks = page.find_all('table')[0] 

import pandas as pd
def scrape_table(table):
    df = pd.read_html(str(table))
    df = pd.DataFrame(df[0])
    return Table.from_df(df)
    
costofacoffee = scrape_table(starbucks)
costofacoffee.drop("City")
costofacoffee= costofacoffee.relabel("Ranking","Ranking $$coffee")

tax_rates = Table.read_table("/content/drive/MyDrive/Colab Notebooks/csvData.csv")
tax_rates.drop("incomeTax","corpTax")

r2 = requests.get('https://www.tpsgc-pwgsc.gc.ca/cgi-bin/recgen/er.pl?Language=E')
page2 = BeautifulSoup(r2.text, 'html.parser')
CADconversion= page2.find_all('table')[0]
conversion_codes = scrape_table(CADconversion)

r3 = requests.get('https://www.iban.com/currency-codes')
page3 = BeautifulSoup(r3.text, 'html.parser')
CODESANDCOUNTRIES = page3.find_all('table')[0]
codes = scrape_table(CODESANDCOUNTRIES)

Min_Wage = Table.read_table('/content/drive/MyDrive/Colab Notebooks/hourlywage.csv').where("TIME",are.equal_to(2021)).where("Pay period", are.equal_to("Hourly")).where("SERIES", are.equal_to ("PPP"))
Min_Wage= Min_Wage.drop("COUNTRY","SERIES","Series","PERIOD","TIME","Unit","PowerCode Code","PowerCode","Reference Period Code", "Reference Period", "Flag Codes", "Flags")
```
In these files, I webscraped or used CSV files to return information regarding: The cost in CAD of stabucks coffee globally, Conversion rates, Sales Tax rates, currency codes, and, for my explanatory factor, the minimum wage in several countries.
All of these files, except for one, had the column **"Country"**

**Clearly, this data needs to be cleaned and combined in order for us to gleam any relevant information from it.**

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

Each **row** in the **column** "country" was converted into **uppercase** as a **new array**. This array then replaced the previous column for country so all the graphs would have similar data

ex. Canada, canada, CANADA == CANADA

```
strarray= Min_Wage.column("Value")
for value in strarray:
  value = float(value)
floatarray=strarray

Min_Wage = Min_Wage.append_column("Float_Values", floatarray)
```
Min_Wage had to be converted from **str** to **float** values.

All of our data has been CAD so far, but these values were USD...
```
convertedCAD = Min_Wage.apply(lambda x: x * 1.34, "Float_Values")
Min_Wage = Min_Wage.append_column('CAD_MinWage/hour', convertedCAD)
Min_Wage = Min_Wage.drop("Value", "Float_Values", "Unit Code")
```

Using **lambda** and a  **set conversion rate** I changed each minimum wage rate from **USD to CAD**.

### joining the tables

```
almostfull = costofacoffee.join('COUNTRY', tax_rates, 'COUNTRY')
almostfull = almostfull.join('COUNTRY', Min_Wage, 'COUNTRY')
almostfull = almostfull.join("COUNTRY", codes, 'COUNTRY')
```

**COUNTRY** == the same for each table

Some tables had different information. **I chose to lose data** that was not accounted for in all tables due to the fact the assignment did not require 76 countries, but only 10.

After this step, there were 21 countries remaining in the data.

# What about the other table?

```
full= almostfull.join("Code", conversion_codes, "Currency\xa0Code")
full= full.drop("incomeTax","corpTax","Currency","Number","Currency Description","Pay period", "Time")
```

If you recall, one table did not have a value for "Country" but **did** have _currency codes_. This graph was essential to the problem as it contained the **conversion rates** of each currency to CAD.

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

# $4.25 vs 4.25
**str vs float**


>**Cost** and **Rate**

```
clean = full.apply(scrubadub,"Cost (CAD$)")
cleaner = full.apply(adub,"Rate")

full = full.append_column("Cost of Coffee",clean).drop("Cost (CAD$)")
full = full.append_column("Rate",cleaner)
```

# My data is fairly clean, but since my source was CAD, I don't have every value required in the the assignment.
# So, let's add it in!

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

**How much does coffee cost in the _original_ currency with taxes?**

```
StarbucksIndex = full
StarbucksIndex = StarbucksIndex.sort("Cost of Coffee(CAD)")
```

With 18 rows, I could easily **visually confirm** that my data had no more flaws.
ex. nan, str, gaps in the data

The final column values were: 
**"Code"	"COUNTRY"	"MinWage/hour(CAD)" 	"Cost of Coffee(CAD)"	"Cost in og units w tax"**

Ex.
**"CAD"	"CANADA"	"14.1504" 	"4.15"	"4.36"**


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

Here, I chose to use the **index**.  

```
StarbucksIndex.barh("COUNTRY", "Difference")
```

![difference cost of coffee](https://user-images.githubusercontent.com/113123766/205983370-18b0853b-0ef5-40d6-8636-e1166a140495.png)

This graph is on a scale of **-2 to 3,** so the price of coffee fluxuates by about $5 depending on where you go.

```
StarbucksIndex.scatter("Difference","MinWage/hour(CAD)")
StarbucksIndex.scatter("Difference","MinWage/hour(CAD)", fit_line = True)
```


# How does Minimum Wage Influence the Cost of a Coffee Globally?

![WageGraph_noline](https://user-images.githubusercontent.com/113123766/205983383-342dcd40-918c-47e1-bace-04a64c4cbf7d.png)

This graph does **not** appear to have a _strong_ correlation; however, if we add a line of best fit:

![WageGraph_withline](https://user-images.githubusercontent.com/113123766/205983385-904417da-3f42-408e-bce5-e13b04bb5196.png)

You can see that there **_is_ a linear association**.
As the price of coffee increases, so does the amount of money people make.

# Why would Minimum Wage Effect The Price of Coffee?

If you recall, I mentioned that Starbucks would be considered a **_luxury_** to most people. 
Since it is not a need, people will likely not purchase a coffee unless they can afford it. As such, people who make less money would pay less for a cup of coffee, and people who make more money would be more willing to spend more. 
There are several other factors at play as well, factors which may have a stronger correlation with the data, however, to a certain extent this relationship did prove to explain the results. 

# Conclusion

## In the end,

Canada has fairly cheap starbucks coffee in comparison to the other countries, and the minimum wage a country creates does have a relation to the cost of a coffee. 

Mexico had the cheapest minimum wage, Luxembourg had the greatest minimum wage.

**Colombia had the cheapest coffee, Luxembourg had the most expensive coffee.**

Despite using the same currency, several EUR countries had different prices for their Starbucks coffee. IN CAD units, these prices ranged from $3.63-$6.82. 


### Difficulties

I lost quite a bit of data when I was joining tables. If I had needed to include every country from the original source I would likely have needed to check every webscraped source to ensure they had the same len, or a greater len than the original data source.
Moreover, I lost the USD conversion, which was easy to find online and use; however, this was a conversion rate which would have been useful to maintain.

Minimum wage does not appear to be the best predictor for the cost of coffee; however, there are several confounding factors that all interact to influence this, and minimum wage is certainly part of the equation. Tourism, GDP, demand, import fees, duties and tariffs, and several other factors could influence how expensive a product is in each country. 

