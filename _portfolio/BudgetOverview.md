---
title: "Personal Finance Tracking with Python (pandas library)"
excerpt: "Python script that helps managing personal finances using Python's pandas library and Excel. Creates a handy overview over average expenses per category. "
collection: portfolio
---

I needed a way to track my income and expenses because I was not happy with my banks' (practically non-existent) out-of-the-box tools. Since it's super easy to download the CSV files with all my expenses from the online banking website, I 
decided to apply some Python pandas magic to create an overview over my expenses. I'm using this script since about 2 years and it works like a charm 💪🏻. 

At the end of each month, I download the month's csv file from each of my banks, run the python code and end up with a neat excel file where the first sheet shows me the average expenses per category for each month as well as a neat dataframe for each month in the other sheets. This helps me compare each month's expenses and identify any spending issues, such as overspending in certain categories or unexpected cost increases like rent.

The code will be uploaded on my github repository soon!

## What the script does step-by-step:
The script imports the csv files from both of my banks and stores it in two dataframes. Then, it cleans the dataframes by removing unnecessary columns and standardizing column names. After that, it concatenates both dataframes into one, which contains all the information.

The next part turned out more difficult than I thought: How can I automatize the assignment of categories to the entries? 

I analyzed the expense information column and found patterns, like the word "geba" in supermarket expenses. I created rules for common/recurring expenses, but some cases couldn't be categorized automatically, like one-time purchases in foreign shops. For these, I displayed them in the terminal and manually assigned the categories through a user input prompt.

I'm also considering using machine learning to automate the categorization of these tricky expenses in the future. However, since there aren't many cases needing manual categorization, the current solution works well.

The resulting neat, organized dataframe is written to an Excel file with a separate sheet for each month. I also include an overview sheet that displays the average expenses for each category in each month. Running the script updates this overview sheet with new data.

