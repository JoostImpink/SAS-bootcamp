# Importing/exporting data

## Import from Excel

Excel files can be imported using 'proc import'. 

Example:

```SAS
proc import 
  OUT       = work.myData DATAFILE= "C:\temp\myData.xlsx" 
  DBMS      = xlsx REPLACE;
  SHEET     = "worksheet1"; 
  GETNAMES  = YES;
run;
```

Note: the value for the 'sheet' keyword needs to exactly match the worksheet name. (Run proc import for each worksheet.)

The benefit of importing an excel sheet (as opposed to a text file, or comma separated values file) is that columns types are preserved well (for example, a column with dates in Excel will also be a column of dates in the SAS dataset).

## Import from text file (.txt or .csv)

Importing a text file requires defining the variables (setting names, their type -- number vs text, and length).

Example:

```SAS
filename MYFILE "C:\temp\myData.csv";

data myData;
infile MYFILE dsd delimiter=","  firstobs=2 LRECL=32767 missover;
input id var1 var2 var3;
run;
```

For tab delimited files, use `delimiter='09'x`. Variables that are text, add a $ after the variable name; e.g. input id var1 $ var2 var3 if var1 is a string. For long strings, add the LENGHT statement after data (but before infile) with the variable length (example: `length var1 $20;`).


## Export 

### Export to csv

'Proc export' is used to export to text files. For example:

```SAS
proc export data =work.dataset  outfile='C:\temp\dataOut.csv' dbms=csv replace;run;
```


### Export to Stata

SAS will write in Stata format when the filename ends with '.dta' (and no dbms set), example:

```SAS
proc export data =work.dataset  outfile='C:\temp\dataOut.dta' replace;run;
```

By the way, to import a Stata dataset into SAS: `proc import out= fromStata datafile = "c:\stata_dataset.dta"; run;`