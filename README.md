# budget-tracker
This is a Google Sheets project that is useful to easily track and monitor your budget in real time, utilizing Google Sheets, Google Apps Script (JavaScript) for automation, and Google Visualization API Query Language (SQL) for the pivot tables.

## Links for the file
Blank Template: (Make a Copy to create your own file)

https://docs.google.com/spreadsheets/d/1DG5FwlgYq-KhKUDiiY1bjjneiYZiAY2Zk5Sd00zzuFU/edit?usp=sharing

File with Sample Data:

https://docs.google.com/spreadsheets/d/19lMLMcpTbJ-I65Bf0T1wnrcShYoQ96qTU78W_RAJqb0/edit?usp=sharing

## Description of Tabs
### Record Transaction

This is the tab where you can record all your transactions daily, including your transfer of money from a wallet or account to the other wallet or account. Using Google Apps Script, transactions can be automatically loaded into the dataset in "All Transactions" tab.

This includes two frames: "My Transaction" and "Transfer Money"

"My Transaction"
The frame where you can record your daily transaction. This includes the following data where you can input:
+ Date - the current date
+ Wallet - A dropdown on what wallet or account to be used.
+ Type - A dropdown to choose if that transaction is Income or Expenses.
+ Category - A dropdown on what category to choose.
+ Description - Write a short description about your transaction.
+ Amount - Type the amount of money to add (income) or lessen (expenses).
+ Add Transaction button - Tick the checkbox to record the transaction into the dataset.

"Transfer Money"
The frame where you can record your money transfer from different wallet or account. This includes the following data where you can input:
+ Date - the current date
+ From - The wallet or account where you will get the money. (From what wallet you will get the money)
+ To - The wallet or account where you will transfer the money. (Into what wallet you will transfer the money)
+ Amount - Type the amount of money to transfer.
+ Transfer button - Tick the checkbox to record the transfer into the dataset.

### All Transactions

This is the tab where you can see the dataset of all your recorded transactions.

### Dashboard

This is the tab where you can see the dashboard of your budget.

You can filter out the data using the three dropdowns to filter by:
+ Year
+ Month
+ Wallet

The following scorecards are shown in this dashboard:
+ Total Income
+ Total Expenses
+ Net Income
+ Remaining

The following charts are also shown in this dashboard:
+ Income - Shows the pie chart of all your income per category. This shows the Top 5 categories with highest income, and the Other Income for other categories not in Top 5.
+ Expenses - Shows the pie chart of all your expenses per category. This shows the Top 5 categories with highest expenses, and the Other Expenses for other categories not in Top 5.
+ Monthly Income and Expenses - Shows the bar chart of the income and expenses per month.
+ Daily Budget Tracking - Shows the line chart of your budget in a daily basis to see if your budget is increasing or decreasing.

### Breakdown

This is the tab that shows the breakdown or pivot tables from the dataset to filter specific data. I also used SQL or the Google Visualization API Query Language to make these pivot tables. This include the following tables:

+ Daily Budget Breakdown - This shows the income, expenses, savings, and remaining money (running total) for each day. This data is presented in Daily Budget Tracking line chart in "Dashboard" tab.

Query:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT A, SUM(E), SUM(F), SUM(G), SUM(E)-SUM(F)+SUM(G)
WHERE A IS NOT NULL AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY A
LABEL SUM(E) '', SUM(F) '', SUM(G) '', SUM(E)-SUM(F)+SUM(G) ''
"),"(No data)"),"(No data)")

Formula for "Remaining Column":
> =ARRAYFORMULA(IF(A10:A<>"",sumif(row(A10:A),"<="&row(A10:A),E10:E),))

+ Monthly Budget Breakdown - This shows the income and expenses for each month. This data is presented in Monthly Income and Expenses bar chart in "Dashboard" tab.

Query:
> =IFERROR(IFNA(QUERY('All Transactions'!$A$8:$M,"
SELECT H, I, J, SUM(E), SUM(F)
WHERE A IS NOT NULL AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY H, I, J
ORDER BY H, I
LABEL H '', I '', J '', SUM(E) '', SUM(F) ''
"),"(No data)"),"(No data)")

+ Total Income - This shows the income for each category, sorted in decreasing order. The first five rows shows the Top 5 categories with highest income, and then the last row for Other Income for other categories not in Top 5. The breakdown for the other categories are shown in the table under.

Query for Top 5 Income:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT C, SUM(E)
WHERE (C IS NOT NULL AND C <> 'Others') AND A IS NOT NULL AND E <> 0 AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY C
ORDER BY SUM(E) DESC
LIMIT 5
LABEL C '', SUM(E) ''
"),""),"(No data)")

Query for "Others" Category:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT C, SUM(E)
WHERE C = 'Others' AND A IS NOT NULL AND E <> 0 AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY C
ORDER BY SUM(E) DESC
LABEL C '', SUM(E) ''
"),""),"")

Query for Income below the Top 5 excluding "Others" category:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT C, SUM(E)
WHERE (C IS NOT NULL AND C <> 'Others') AND A IS NOT NULL AND E <> 0 AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY C
ORDER BY SUM(E) DESC
OFFSET 5
LABEL C '', SUM(E) ''
"),""),"")

+ Total Expenses - This shows the expenses for each category, sorted in decreasing order. The first five rows shows the Top 5 categories with highest expenses, and then the last row for Other Expenses for other categories not in Top 5. The breakdown for the other categories are shown in the table under.

Query for Top 5 Expenses:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT C, SUM(F)
WHERE (C IS NOT NULL AND C <> 'Others') AND A IS NOT NULL AND F <> 0 AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY C
ORDER BY SUM(F) DESC
LIMIT 5
LABEL C '', SUM(F) ''
"),""),"(No data)")

Query for "Others" Category:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT C, SUM(F)
WHERE C = 'Others' AND A IS NOT NULL AND F <> 0 AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY C
ORDER BY SUM(F) DESC
LABEL C '', SUM(F) ''
"),""),"")

Query for Income below the Top 5 excluding "Others" category:
> =IFERROR(IFNA(QUERY('All Transactions'!A8:M,"
SELECT C, SUM(F)
WHERE (C IS NOT NULL AND C <> 'Others') AND A IS NOT NULL AND F <> 0 AND (K = TRUE AND L = TRUE) AND (M = TRUE OR B IS NULL)
GROUP BY C
ORDER BY SUM(F) DESC
OFFSET 5
LABEL C '', SUM(F) ''
"),""),"")

+ Periods - This shows the years and months included in the whole dataset. If the year is not filtered, the months will not be shown.

### Wallets

This is the tab that shows all your wallets or accounts, including their starting salary, which is the money that you have in that wallet as you put this in your database.

In this tab, you can add, edit, and delete wallet or account with their starting salary.

To edit wallet, you can do the following:
1. Rename your wallet in this tab.
2. Filter the data in "All Transactions" tab in Wallet column by the wallet you renamed.
3. Highlight the Wallet column.
4. Rename the wallet in that column.

### Categories

This is the tab that shows all categories for Income and all categories for Expenses.

In this tab, you can add, edit, and delete your categories for Income and Expenses.

To edit category, you can do the following:
1. Rename your category in this tab.
2. Filter the data in "All Transactions" tab in Category column by the category you renamed, and the non-zero values of Income in Income column or Expenses in Expenses column.
3. Highlight the Category column.
4. Rename the category in that column.
