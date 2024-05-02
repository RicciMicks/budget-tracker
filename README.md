# budget-tracker
This is a Google Sheets project that is useful to easily track and monitor your budget in real time.

# Links for the file
Blank Template: (Make a Copy then create your own file)

https://docs.google.com/spreadsheets/d/1DG5FwlgYq-KhKUDiiY1bjjneiYZiAY2Zk5Sd00zzuFU/edit?usp=sharing

File with Sample Data:

https://docs.google.com/spreadsheets/d/19lMLMcpTbJ-I65Bf0T1wnrcShYoQ96qTU78W_RAJqb0/edit?usp=sharing

# Description of Tabs and Tutorial
### Record Transaction

This is the tab where you can record all your transactions daily, including your transfer of money from a wallet or account to the other wallet or account. Using Google Apps Script, transactions can be automatically loaded into the dataset in "All Transactions" tab. I put the code in the "Apps Script Code" section. 

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
+ Income - Shows the pie chart of all your income, sorted by the amount in decreasing order.
+ Expenses - Shows the pie chart of all your expenses, sorted by the amount in decreasing order.
+ Monthly Income and Expenses - Shows the bar chart of the income and expenses per month.
+ Daily Budget Tracking - Shows your budget in a daily basis to see if your budget is increasing or decreasing.

### Breakdown

This is the tab that shows the breakdown or pivot tables from the dataset to filter specific data. I also used SQL or the Google Visualization API Query Language to make these pivot tables. This include the following tables:


### Wallets


### Categories


# Apps Script Code
```
//Call Spreadsheet
var ss = SpreadsheetApp.getActiveSpreadsheet();
var form = ss.getSheetByName("Record Transaction");
var datasheet = ss.getSheetByName("All Transactions");
var wallet_dropdown = ss.getSheetByName("Wallets");
var cat_dropdown = ss.getSheetByName("Categories");

//My Transaction
var date = form.getRange("date");
var wallet = form.getRange("wallet_acc");
var type = form.getRange("type");
var cat = form.getRange("category");
var desc = form.getRange("description");
var val = form.getRange("value");
var remaining_money = form.getRange("remaining_money");
var button = form.getRange("button_addtrans");

//Transfer Money
var transDate = form.getRange("transDate");
var transFrom = form.getRange("transFrom");
var transTo = form.getRange("transTo");
var transVal = form.getRange("transValue");
var transBtn = form.getRange("button_transfer");

function recordValue() {

  //Validate Transaction

  date.setBorder(false, false, false, false, null, null);
  type.setBorder(false, false, false, false, null, null);
  wallet.setBorder(false, false, false, false, null, null);
  cat.setBorder(false, false, false, false, null, null);
  val.setBorder(false, false, false, false, null, null);

  var isvalid = true;
  form.getRangeList(["valid_date","valid_wallet","valid_type","valid_cat","valid_value"]).setValue("");

  if(date.isBlank()==true){
    form.getRange("valid_date").setValue("Please enter the date.");
    date.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(wallet.isBlank()==true){
    form.getRange("valid_wallet").setValue("Please enter which wallet.");
    wallet.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(type.isBlank()==true){
    form.getRange("valid_type").setValue("Please enter the type.");
    type.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(cat.isBlank()==true){
    form.getRange("valid_cat").setValue("Please enter which category.");
    cat.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(val.isBlank()==true){
    form.getRange("valid_value").setValue("Please enter the amount.");
    val.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(remaining_money.getValue()<val.getValue() && type.getValue()=="Expenses"){
    form.getRange("valid_value").setValue("You lack money for that amount.");
    val.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  //Record Transaction

  if(isvalid==true){

    prevTrans();

    var addRow = datasheet.getLastRow()+1; 
    
    datasheet.getRange(addRow, 1).setValue(date.getValue()); //Date
    datasheet.getRange(addRow, 2).setValue(wallet.getValue()); //Wallet
    datasheet.getRange(addRow, 3).setValue(cat.getValue()); //Category
    datasheet.getRange(addRow, 4).setValue(desc.getValue()); //Description
    
    if(type.getValue()=="Income"){
      datasheet.getRange(addRow, 5).setValue(val.getValue()); //Income
      datasheet.getRange(addRow, 6).setValue(0);
      datasheet.getRange(addRow, 7).setValue(0);
    }
    else if(type.getValue()=="Expenses"){
      datasheet.getRange(addRow, 6).setValue(val.getValue()); //Expenses
      datasheet.getRange(addRow, 5).setValue(0);
      datasheet.getRange(addRow, 7).setValue(0);
    }

    var cola = datasheet.getRange(addRow, 1).getA1Notation(); 
    var colb = datasheet.getRange(addRow, 2).getA1Notation();

    datasheet.getRange(addRow, 8).setFormula("=YEAR("+cola+")")
    datasheet.getRange(addRow, 9).setFormula("=MONTH("+cola+")")
    datasheet.getRange(addRow, 10).setFormula("=TEXT("+cola+",\"mmm\")&\" '\"&RIGHT(YEAR("+cola+"),2)")

    var colh = datasheet.getRange(addRow, 8).getA1Notation();
    var colj = datasheet.getRange(addRow, 10).getA1Notation(); 

    datasheet.getRange(addRow, 11).setFormula("=IF(Year=\"\",TRUE,IF(Year="+colh+",TRUE,\"\"))")
    datasheet.getRange(addRow, 12).setFormula("=IF(Month=\"\",TRUE,IF(Month="+colj+",TRUE,\"\"))")
    datasheet.getRange(addRow, 13).setFormula("=IF(Wallet=\"\",TRUE,IF(Wallet="+colb+",TRUE,\"\"))")

    for(b=1;b<=datasheet.getMaxColumns();b++){
      datasheet.getRange(addRow, b).setBorder(true,true,true,true,null,null,"#D3D3D3",SpreadsheetApp.BorderStyle.SOLID);
    }

    date.setFormula("=today()");
    type.setValue("");
    cat.setValue("");
    desc.setValue("");
    val.setValue("");

  }

}

function recordTransfer(){

  //Validate

  transDate.setBorder(false, false, false, false, null, null);
  transFrom.setBorder(false, false, false, false, null, null);
  transTo.setBorder(false, false, false, false, null, null);
  transVal.setBorder(false, false, false, false, null, null);

  var isvalid = true;
  form.getRangeList(["valid_transdate","valid_transfrom","valid_transto","valid_transval"]).setValue("");

  if(transDate.isBlank()==true){
    form.getRange("valid_transdate").setValue("Please enter the date.");
    transDate.activate();
    transDate.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(transFrom.isBlank()==true){
    form.getRange("valid_transfrom").setValue("Please enter which wallet.");
    transFrom.activate();
    transFrom.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(transTo.isBlank()==true){
    form.getRange("valid_transto").setValue("Please enter which wallet.");
    transTo.activate();
    transTo.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  if(transVal.isBlank()==true){
    form.getRange("valid_transval").setValue("Please enter the amount.");
    transVal.activate();
    transVal.setBorder(true, true, true, true, null, null, '#ff0000', SpreadsheetApp.BorderStyle.SOLID);
    isvalid *= false;
  }

  //Record

  if(isvalid==true){

    prevTrans();

    var addRowFrom = datasheet.getLastRow()+1;
  
    datasheet.getRange(addRowFrom, 1).setValue(transDate.getValue()); //Date
    datasheet.getRange(addRowFrom, 2).setValue(transFrom.getValue()); //Wallet
    datasheet.getRange(addRowFrom, 3).setValue("Transfer"); //Category
    datasheet.getRange(addRowFrom, 4).setValue("Transfer to "+transTo.getValue()); //Description
    datasheet.getRange(addRowFrom, 7).setValue(transVal.getValue()*-1); //Value
    datasheet.getRange(addRowFrom, 5).setValue(0);
    datasheet.getRange(addRowFrom, 6).setValue(0);

    var fcola = datasheet.getRange(addRowFrom, 1).getA1Notation(); 
    var fcolb = datasheet.getRange(addRowFrom, 2).getA1Notation();

    datasheet.getRange(addRowFrom, 8).setFormula("=YEAR("+fcola+")")
    datasheet.getRange(addRowFrom, 9).setFormula("=MONTH("+fcola+")")
    datasheet.getRange(addRowFrom, 10).setFormula("=TEXT("+fcola+",\"mmm\")&\" '\"&RIGHT(YEAR("+fcola+"),2)")

    var fcolh = datasheet.getRange(addRowFrom, 8).getA1Notation();
    var fcolj = datasheet.getRange(addRowFrom, 10).getA1Notation(); 

    datasheet.getRange(addRowFrom, 11).setFormula("=IF(Year=\"\",TRUE,IF(Year="+fcolh+",TRUE,\"\"))")
    datasheet.getRange(addRowFrom, 12).setFormula("=IF(Month=\"\",TRUE,IF(Month="+fcolj+",TRUE,\"\"))")
    datasheet.getRange(addRowFrom, 13).setFormula("=IF(Wallet=\"\",TRUE,IF(Wallet="+fcolb+",TRUE,\"\"))")

    for(fb=1;fb<=datasheet.getMaxColumns();fb++){
      datasheet.getRange(addRowFrom, fb).setBorder(true,true,true,true,null,null,"#D3D3D3",SpreadsheetApp.BorderStyle.SOLID);
    }

    var addRowTo = datasheet.getLastRow()+1;

    datasheet.getRange(addRowTo, 1).setValue(transDate.getValue()); //Date
    datasheet.getRange(addRowTo, 2).setValue(transTo.getValue()); //Wallet
    datasheet.getRange(addRowTo, 3).setValue("Transfer"); //Category
    datasheet.getRange(addRowTo, 4).setValue("Transfer from "+transFrom.getValue()); //Description
    datasheet.getRange(addRowTo, 7).setValue(transVal.getValue()); //Value
    datasheet.getRange(addRowTo, 5).setValue(0);
    datasheet.getRange(addRowTo, 6).setValue(0);

    var tcola = datasheet.getRange(addRowTo, 1).getA1Notation(); 
    var tcolb = datasheet.getRange(addRowTo, 2).getA1Notation();

    datasheet.getRange(addRowTo, 8).setFormula("=YEAR("+tcola+")")
    datasheet.getRange(addRowTo, 9).setFormula("=MONTH("+tcola+")")
    datasheet.getRange(addRowTo, 10).setFormula("=TEXT("+tcola+",\"mmm\")&\" '\"&RIGHT(YEAR("+tcola+"),2)")

    var tcolh = datasheet.getRange(addRowTo, 8).getA1Notation();
    var tcolj = datasheet.getRange(addRowTo, 10).getA1Notation(); 

    datasheet.getRange(addRowTo, 11).setFormula("=IF(Year=\"\",TRUE,IF(Year="+tcolh+",TRUE,\"\"))")
    datasheet.getRange(addRowTo, 12).setFormula("=IF(Month=\"\",TRUE,IF(Month="+tcolj+",TRUE,\"\"))")
    datasheet.getRange(addRowTo, 13).setFormula("=IF(Wallet=\"\",TRUE,IF(Wallet="+tcolb+",TRUE,\"\"))")

    for(tb=1;tb<=datasheet.getMaxColumns();tb++){
      datasheet.getRange(addRowTo, tb).setBorder(true,true,true,true,null,null,"#D3D3D3",SpreadsheetApp.BorderStyle.SOLID);
    }

    transDate.setFormula("=today()");
    //transFrom.setValue("");
    //transTo.setValue("");
    transVal.setValue("");
    
  }

}

function prevTrans(){

  var date_diff = form.getRange("date_diff").getValue();
  var addRow = datasheet.getLastRow()+1;
  var cola = datasheet.getRange(addRow, 1).getA1Notation();
  var colh = datasheet.getRange(addRow, 8).getA1Notation();
  var colj = datasheet.getRange(addRow, 10).getA1Notation(); 

  if(date_diff!=0){      
    for(i=date_diff-1;i>0;i--){
      datasheet.getRange(addRow, 1).setFormula("=today()-"+i);

      datasheet.getRange(addRow, 5).setFormula(0);
      datasheet.getRange(addRow, 6).setFormula(0);
      datasheet.getRange(addRow, 7).setFormula(0);  

      datasheet.getRange(addRow, 8).setFormula("=YEAR("+cola+")")
      datasheet.getRange(addRow, 9).setFormula("=MONTH("+cola+")")
      datasheet.getRange(addRow, 10).setFormula("=TEXT("+cola+",\"mmm\")&\" '\"&RIGHT(YEAR("+cola+"),2)")

      datasheet.getRange(addRow, 11).setFormula("=IF(Year=\"\",TRUE,IF(Year="+colh+",TRUE,\"\"))")
      datasheet.getRange(addRow, 12).setFormula("=IF(Month=\"\",TRUE,IF(Month="+colj+",TRUE,\"\"))")

      for(b=1;b<=datasheet.getMaxColumns();b++){
        datasheet.getRange(addRow, b).setBorder(true,true,true,true,null,null,"#D3D3D3",SpreadsheetApp.BorderStyle.SOLID);
      }
      
    }
  }
}

function onEdit(e){

  let activeCell = ss.getActiveCell();
  let reference = activeCell.getA1Notation();
  let sheetName = activeCell.getSheet().getName();
  let activeValue = activeCell.getValue();

  if(reference==button.getA1Notation() && sheetName=="Record Transaction" && activeValue==true){ 
    recordValue();
    activeCell.setValue("false");
  }

  if(reference==transBtn.getA1Notation() && sheetName=="Record Transaction" && activeValue==true){ 
    recordTransfer();
    activeCell.setValue("false");
  }

}
```
