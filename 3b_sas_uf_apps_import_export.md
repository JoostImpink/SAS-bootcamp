# SAS UF Apps example - Import from Excel, export to Stata

Below video shows how to access your local harddisk on SAS (UF Apps), how to import an Excel sheet, and export it as a Stata dataset.

## Youtube video

Link to recording: [https://www.youtube.com/watch?v=RT9TCD5y4QI](https://www.youtube.com/watch?v=RT9TCD5y4QI)


## SAS Code 

```SAS
/*

How to organize your project (dissertation) files 
	- Folder structure

How to set up SAS on UF Apps to access these files
	- Importing data from Excel
	- Libnames
	- Export (to Stata)

UF Apps using 'Workspace' App from Citrix: www.citrix.com/downloads

*/

/* for SAS on UF Apps */
%let myPrFolder = \\Client\E$\example_project\;

/* Using my local install of SAS */
/* %let myPrFolder = E:\example_project; */

libname p "&myPrFolder.sasdata";

/* Import Excel sheet */
proc import 
  OUT       = p.myData DATAFILE= "&myPrFolder.data_collected\data_collected_may_27.xlsx" 
  DBMS      = xlsx REPLACE;
  SHEET     = "Sheet1"; 
  GETNAMES  = YES;
run;

data p.myData2;
set p.myData;
a = x + y + z;
run;

/* Stata format */
proc export data= p.myData2 outfile="&myPrFolder.\stata\export_from_sas\myData.dta" replace; run; 
```