---
title: "💰 How I manage my finances with Python's pandas"
excerpt: "This is how I track my personal finances using Python's pandas library and Excel. My script cleans and categorizes bank statement csvs to create a neat
neat overview over average expenses per category. "
collection: portfolio
---

I needed a way to track my income and expenses because I was not happy with my banks' (practically non-existent) out-of-the-box tools. Since it's super easy to download the CSV files with all my expenses from the online banking website, I 
decided to apply some Python pandas magic to create an overview over my expenses. I'm using this script since about 2 years and it works like a charm 💪🏻. 

At the end of each month, I download the month's csv file from each of my banks, run the python code and end up with a neat excel file where the first sheet shows me the average expenses per category for each month as well as a neat dataframe for each month in the other sheets. This helps me compare each month's expenses and identify any spending issues, such as overspending in certain categories or unexpected cost increases like rent.



## What the script does step-by-step:

The following is a step-by-step description for my specific my specific use case (some changes were made for privacy reasons 😎). I use two different banks so it needs a bit of combining. The script is made up of separate functions that do one job and then called with `main()`.

### Main
This is the main() section of the script.
The script calls `importData()` to import the csv files from both of my banks and stores it in two dataframes. Then, `cleanData()` cleans the dataframes by removing unnecessary columns, standardizing column names, accounting for decimal conventions and finally concatenates them to a big dataframe `finanzen`. 

```
def main():
    #Import data files:
    print("Input current month (e.g. August -> 08)")
    current_month = input()
    fileING, fileSP, worksheetName, month = importData(current_month,"2023") month and year as strings

    #Clean data:
    finanzen = cleanData(fileING, fileSP)
   
    #Parse categories
    finanzen = categoryParse(finanzen)
    finanzen = manualAssignment(finanzen)
   
    #Create sums of categories
    ausgaben = createAusgaben(finanzen, month)
    
    #Write data to excel sheet:
    append_rawData_to_excelSheet('/Users/majaguseva/Desktop/Budget/budget.xlsx', finanzen, worksheetName)
    writeAusgaben_to_ExcelOverview(ausgaben, month)
    
    return finanzen, ausgaben
```
### Import data
The first part is to import the csv files. The terminal prompts the user to type the relevant month and selects the corresponding csv sheet(s). This is the importData function:
```
def importData(month, year):
    worksheetName = f"{year}_{month}"
    os.chdir(transactionFilesFolder+"/"+year+"_"+ month)
    fileING = pd.read_csv(year+"_"+month+"_ING.csv",delimiter=";", encoding = "ISO-8859-1")
    fileSP = pd.read_csv(year+"_"+month+"_SP.csv",delimiter=";", encoding = "ISO-8859-1")
    return fileING, fileSP, worksheetName, month
```
Since I'm using two banks, I have two separate csv files which are imported using Panda's read_csv function.

### Clean data
This function first removes unnecessary columns in both dataframes. Since both banks have slightly different column names denoting the same thing, I equalize them. Next, both dataframes are concatenated to the dataframe "finanzen" and sorted by day. The "Betrag" column causes some issues because a) it's a string and b) German decimal system is different, so I use a little helper function make_float() to deal with that.
```
def cleanData(fileING, fileSP):
    # Keep only relevant columns:
    fileING = fileING[['Buchung', 'Auftraggeber/Empfänger', 'Buchungstext', 'Verwendungszweck', 'Betrag']]
    fileSP = fileSP[['Buchungstag', 'Buchungstext','Verwendungszweck', 'Beguenstigter/Zahlungspflichtiger', 'Betrag']]

    #Equalize slightly different column names between dataframes that mean the same:
    fileING.columns = ["Buchungstag","Auftraggeber", "Buchungstext", "Verwendungszweck", "Betrag" ]

    #switch column positions
    fileING = fileING[["Buchungstag", "Betrag","Auftraggeber","Buchungstext", "Verwendungszweck"]]

    #change names at fileSP:
    fileSP.columns = ["Buchungstag", "Buchungstext", "Verwendungszweck","Auftraggeber", "Betrag" ]
    fileSP = fileSP[["Buchungstag", "Betrag","Auftraggeber","Buchungstext", "Verwendungszweck"]]
    
    #concatenate both dataframes
    finanzen = pd.concat([fileING,fileSP])
    finanzen = finanzen.sort_values("Buchungstag")
    finanzen=finanzen.reset_index(drop=True)
    
    #change Betrag string to float:
    finanzen["Betrag"] = list(map(make_float, finanzen["Betrag"]))
    finanzen=finanzen.reset_index(drop=True)
    return finanzen
```
```
def make_float(num):
    if len(num) >= 8:
        num = num.replace('.','')
        num = num.replace(' ','').replace(',','.').replace("−", "-")
    else:
        num = num.replace(' ','').replace(',','.').replace("−", "-")
    return float(num)
    ```
```

### Expense categorization by rule
The next part turned out more difficult than I thought: **How can I automatize the assignment of categories to the entries?** 🤔

I found recurring patterns in the expense information column, like the word "geba" in supermarket expenses and decided hard-code them as they are unlikely to change. So I created a new variable `category` and checked elementwise whether the "Auftraggeber" column in the `finanzen` dataframe contained the keyword. If yes, then it was saved in the category column. In the end the categorization column was added to the `finanzen` dataframe.

ToDo: Describe the special caes with "Buchungstext"

```
def categoryParse(finanzen):
    #Initialize empty catAssign array where we will insert categories
    catAssign =pd.DataFrame([0] * len(finanzen))
    #set category names
    cats = ["geba", "Rossmann", "Vodafone"]

    #assign category to catAssign for each row:
    #for case where I take from "Auftraggeber" column:
    for cat in cats:
        catInstance = finanzen[finanzen.Auftraggeber.str.contains(cat,case = False)==True]
        idx =list(catInstance.index)
        catAssign.iloc[idx] = cat

    #for case where I take from "Buchungstext" column:
    catInstance = finanzen[finanzen.Buchungstext.str.contains("bargeldauszahlung",case = False)==True]
    idx=list(catInstance.index)
    catAssign.iloc[idx] = "bargeld"

    #add the category assignment array to finanzen dataframe
    finanzen = pd.concat([catAssign,finanzen], axis=1)
    finanzen = finanzen.rename(columns={0: "category"})

    return finanzen
```


### Categorization of the rest in terminal 
Some cases couldn't be categorized based on rules alone, like one-time purchases in foreign shops.

This is what the `manualAssignment()` function is for: It outputs those remaining expenses to the terminal window and goes through each entry prompoting the user to type the relevant category. The string is then assigned to the category column in the `finanzen` dataframe. In this way we end up with a complete categorization of each item.

```
def manualAssignment(finanzen):
    idx  = finanzen[(finanzen.loc[:,"category"] == 0)].index
    undefinedCat = finanzen[(finanzen.loc[:,"category"] == 0)]
    for position in range(len(undefinedCat)):
        itemText = undefinedCat.iloc[position].loc["Verwendungszweck"]
        print(undefinedCat)
        x=input("\n \n What category is: \n "+ str(itemText)+"?\n Possible inputs: 'gastro', 'misc', 'supermarkt'")
        finanzen.at[idx[position],'category'] = x
    
    return finanzen
```

I'm also considering using machine learning to automate the categorization of these tricky expenses in the future. However, since there aren't many cases needing manual categorization, the current solution works well.


### Create overview for each expense category
I created another dataframe "ausgaben" (=Expenses) where I use the category column, create top-level categories and sum up the expenses. For example, I tend to end up with different variations for my supermarket expenses "geba" and "Edeka", so I combine them all into one category "supermarket". 

```
def createAusgaben(finanzen, month):
    #Preparatory stuff to create ausgaben dictionary:

    ausgaben = {
    "monat": month,
    "supermarkt":finanzen[finanzen['category'].isin(['geba', 'edeka', supermarkt'])]['Betrag'].sum(),
    "misc":finanzen[finanzen['category'].isin(['misc','0'])]['Betrag'].sum()
    }
    ausgaben = pd.DataFrame.from_dict([ausgaben])
    return ausgaben
```

### Save overview to excel sheet

I have an excel sheet where I save this information. The first sheet contains the sums for all expense categories for each month and the remaining sheets are just the raw `finanzen` dataframe for easy access and reference. I had to be careful to append the row of sums to the correct month,so the `writeAusgaben_to_ExcelOverview()` function does exactly that.

```
def append_rawData_to_excelSheet(fpath, df, sheet_name):
    with pd.ExcelWriter(fpath, mode="a") as f:
        df.to_excel(f, sheet_name=sheet_name,index=False)

def writeAusgaben_to_ExcelOverview(ausgaben, month):
    fn = '/Users/majaguseva/Desktop/Budget/budget.xlsx'
    writer = pd.ExcelWriter(fn, mode="a", if_sheet_exists='overlay')
    ausgaben.to_excel(writer,sheet_name='Tabelle1', startrow=int(month),header=None, index=False)
    writer.save()
    return
```

### Run main
Of course in the end I specify my folder structure and run `main()` 

```
rootFolder = ".../PersonalFinance"
transactionFilesFolder = rootFolder + "/MonthlyTransactions" 

if __name__ == "__main__":
    finanzen, ausgaben=main()
 
```
