---
title: "💰 How I manage my finances with Python's pandas"
excerpt: "This is how I track my personal finances using Python's pandas library and Excel. My script cleans and categorizes bank statement csvs to create a neat
neat overview over average expenses per category. <br/><img src='https://github.com/m-guseva/personal/assets/63409978/4bb115f5-0e3e-43df-967d-2bfae21df559'>"
collection: portfolio
---




I needed a way to track my income and expenses because I was not happy with my bank's (practically non-existent) out-of-the-box tools. Since it's super easy to download the csv files with all my expenses from the online banking website, I 
decided to apply some Python pandas magic to create an overview over my expenses (see example overview below). I'm using this script since about 2 years and it works like a charm 💪🏻. 


<img src="https://github.com/m-guseva/personal/assets/63409978/9cfb6290-dde0-4803-9efa-1582d5099c05" alt="Overview" height="300">


*This is an example expense overview sheet (not my own)*



At the end of each month, I download the month's csv file from each of my banks (see image below), run the python code and end up with a neat excel file where the first sheet shows me the average expenses per category for each month as well as a neat dataframe for each month in the other sheets. This helps me compare each month's expenses and identify any spending issues, such as overspending in certain categories or unexpected cost increases like rent.

<img src="https://github.com/m-guseva/personal/assets/63409978/f326d8db-7681-4668-84f4-123191ad0663" alt="Example csv file" height="300">

*Example csv file (Btw. I generated this table was generate with ChatGPT, very handy!)*


# The script

The following is a step-by-step description for my specific my specific use case (some changes were made for privacy reasons 😎). I use two different banks so it needs a bit of combining. The script is made up of separate functions that do one job and are then called with `main()`.


First, `main()` calls `importData()` which handles the import of csv files from both of my banks and stores it in two dataframes. Then, `cleanData()` cleans the dataframes by removing unnecessary columns, standardizing column names, accounting for decimal separation conventions and before concatenating them into a single dataframe `expenses`. 

```python
import os
import pandas as pd


def main():
    #Import data files:
    print("Input current month (e.g. August -> 08)")
    current_month = input()
    fileB1, fileB2, worksheetName, month = importData(current_month,"2023")

    #Clean data:
    expenses = cleanData(fileB1, fileB2)
   
    #Parse categories
    expenses = categoryParse(expenses)
    expenses = manualAssignment(expenses)
   
    #Create sums of categories
    summedExpenses = createSummedExpenses(expenses, month)
    
    #Write data to excel sheet:
    append_rawData_to_excelSheet(rootFolder + '/budget.xlsx', expenses, worksheetName)
    writeSumExp_to_ExcelOverview(summedExpenses, month)
    
    return expenses, summedExpenses
```
## Import data
To import the csv files, the user is prompted to type the relevant month in the terminal. The corresponding csv sheets are then selected and imported as pandas dataframes. Since I'm using two banks, I need to import two separate csv files `fileB1` and `fileB2`.

```python
def importData(month, year):
    worksheetName = f"{year}_{month}"
    os.chdir(rootFolder + "/MonthlyTransactions" +"/"+year+"_"+ month)
    fileB1 = pd.read_csv(year+"_"+month+"_ING.csv",delimiter=";", encoding = "ISO-8859-1")
    fileB2 = pd.read_csv(year+"_"+month+"_SP.csv",delimiter=";", encoding = "ISO-8859-1")
    return fileB1, fileB2, worksheetName, month
```

## Clean data
This function removes unnecessary columns in both dataframes. Since both banks have slightly different column names denoting the same thing, I equalize them. Next, both dataframes are concatenated to the dataframe `expenses` and sorted by day. The "Betrag" column causes some issues because a) it's a string and b) German decimal separator system is different, so I use a little helper function make_float() to deal with that.


>❕ Note: Since I live in Germany the csv files' column names are in  German. For the purpose of this example I decided to keep them, so here is a translation:
>- Buchung/Buchungstag: booking/booking day
>- Buchungstext: booking Text
>- Auftraggeber/Empfänger: client/recipient
>- Beguenstigter/Zahlungspflichtiger: beneficiary/payee
>- Verwendungszweck: reference
>- Betrag: amount

```python
def cleanData(fileB1, fileB2):
    # Keep only relevant columns:
    fileB1 = fileB1[['Buchung', 'Auftraggeber/Empfänger', 'Buchungstext', 'Verwendungszweck', 'Betrag']]
    fileB2 = fileB2[['Buchungstag', 'Buchungstext','Verwendungszweck', 'Beguenstigter/Zahlungspflichtiger', 'Betrag']]

    #Equalize slightly different column names between dataframes that mean the same:
    fileB1.columns = ["Buchungstag","Auftraggeber", "Buchungstext", "Verwendungszweck", "Betrag" ]

    #switch column positions
    fileB1 = fileB1[["Buchungstag", "Betrag","Auftraggeber","Buchungstext", "Verwendungszweck"]]

    #change names at fileB2:
    fileB2.columns = ["Buchungstag", "Buchungstext", "Verwendungszweck","Auftraggeber", "Betrag" ]
    fileB2 = fileB2[["Buchungstag", "Betrag","Auftraggeber","Buchungstext", "Verwendungszweck"]]
    
    #concatenate both dataframes
    expenses = pd.concat([fileB1,fileB2])
    expenses = expenses.sort_values("Buchungstag")
    expenses=expenses.reset_index(drop=True)
    
    #Convert decimal separator in amount ("Betrag") column and change to float:
    expenses["Betrag"] = list(map(make_float, expenses["Betrag"]))
    expenses=expenses.reset_index(drop=True)
    return expenses
```

```python
def make_float(num):
    if len(num) >= 8:
        num = num.replace('.','')
        num = num.replace(' ','').replace(',','.').replace("−", "-")
    else:
        num = num.replace(' ','').replace(',','.').replace("−", "-")
    return float(num)
    ```
```

## Expense categorization by rule
The next part turned out trickier than I thought: **How should I assign categories to the entries?** 🤔

I found recurring patterns in the "Auftraggeber" column, like the word "geba" that is in most supermarket related expenses and decided to use those as rules as they are unlikely to change. So I created a new variable `category` and checked elementwise whether the "Auftraggeber" column in my `expenses` dataframe contained the keyword. If yes, then it was saved in the category column. In the end the categorization column was added to the `expenses` dataframe.



```python
def categoryParse(expenses):
    #Initialize empty catAssign array where we will insert categories
    catAssign =pd.DataFrame([0] * len(expenses))
    #set category names
    cats = ["geba", "Rossmann", "Vodafone"]

    #assign category to catAssign for each row:
    for cat in cats:
        catInstance = expenses[expenses.Auftraggeber.str.contains(cat,case = False)==True]
        idx =list(catInstance.index)
        catAssign.iloc[idx] = cat

    #add the category assignment array to expenses dataframe
    expenses = pd.concat([catAssign,expenses], axis=1)
    expenses = expenses.rename(columns={0: "category"})

    return expenses
```


## Manual categorization of remaining entries in terminal 
Some cases couldn't be easily categorized based on rules alone, like one-time purchases in foreign shops. This is what the `manualAssignment()` function is for: It outputs those remaining expenses to the terminal window and goes through each entry prompoting the user to type the relevant category. The string is then assigned to the category column in the `expenses` dataframe. In this way we end up with a complete categorization of each item.

```python
def manualAssignment(expenses):
    idx  = expenses[(expenses.loc[:,"category"] == 0)].index
    undefinedCat = expenses[(expenses.loc[:,"category"] == 0)]
    for position in range(len(undefinedCat)):
        itemText = undefinedCat.iloc[position].loc["Verwendungszweck"]
        print(undefinedCat)
        x=input("\n \n What category is: \n "+ str(itemText)+"?\n Possible inputs: 'gastro', 'misc', 'supermarkt'")
        expenses.at[idx[position],'category'] = x
    
    return expenses
```

I'm also considering using machine learning to automate the categorization of these tricky expenses in the future. However, since there aren't many cases needing manual categorization, the current solution works well enough.


## Create overview for each expense category
I created another dataframe "summedExpenses" where using the category column, I create top-level categories and sum up the expenses. For example, I tend to end up with different variations for my supermarket expenses "geba" and "Edeka", so I combine them all into one category "supermarket". 

```python
def createSummedExpenses(expenses, month): 
    summedExpenses = {
    "monat": month,
    "supermarkt":expenses[expenses['category'].isin(['geba', 'edeka', supermarkt'])]['Betrag'].sum(),
    "misc":expenses[expenses['category'].isin(['misc','0'])]['Betrag'].sum()
    }
    summedExpenses = pd.DataFrame.from_dict([summedExpenses])
    return summedExpenses
```

## Save overview to excel sheet
I keep an excel sheet where I track all this expense information. The first sheet contains the sums for all expense categories for each month and the remaining sheets store the raw `expenses` dataframes for every month so that I can access and reference them easily. I had to be careful to append the row of sums to the correct row in the Excel sheet, so the `writeSumExp_to_ExcelOverview()` function does exactly that.

```python
def append_rawData_to_excelSheet(fpath, df, sheet_name):
    with pd.ExcelWriter(fpath, mode="a") as f:
        df.to_excel(f, sheet_name=sheet_name,index=False)

def writeSumExp_to_ExcelOverview(summedExpenses, month):
    fn = rootFolder + '/budget.xlsx'
    writer = pd.ExcelWriter(fn, mode="a", if_sheet_exists='overlay')
    summedExpenses.to_excel(writer,sheet_name='Tabelle1', startrow=int(month),header=None, index=False)
    writer.save()
    return
```

## Run main
Of course in the end I set the path to my rootFolder structure and run `main()` 

```python
rootFolder = "/PATH/PersonalFinance"

if __name__ == "__main__":
    expenses, summedExpenses=main()
 
```
