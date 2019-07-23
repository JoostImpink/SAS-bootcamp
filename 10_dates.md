# Working with dates

In SAS, a date is stored internally as a number, but can be displayed (formatted) in a recognizable way.

```SAS
data myDate;
d = '11Nov2018'd;
run;

proc print; run;
```

The above code will print the internal value of 21499 that represents November 11, 2018. For displaying/printing purposes, we can apply a date format.

```SAS
data myDate;
d = '11Nov2018'd;
format d date9.;
run;

proc print; run;
```

In general, formats end with a period ('.'), the date9. format will print it as '11NOV2018'. For an overview of the various formats, [see](http://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a001263753.htm).


## Filtering

Example to filter a date variable:

set a_funda;
if '01jan2000'd <= datadate <= '31dec2000'd;
run;
```SAS
rsubmit;

data myDate (keep = gvkey fyear datadate sale at ceq);
set comp.funda;
/* only records where datadate is in this date range */
if '10Dec2015'd <=datadate <= '20Dec015'd;
/* standard filter (otherwise duplicate records) */
if indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C';
run;

/* download */
proc download data=myDate out=myDate;run;

endrsubmit;
```

The above code will retrieve all records in Compustat Annual where the datadate is between 10 and 20 December 2015.


## INTNX


The 'INTNX' function can be used to add/subtract days/weeks/months/years from date variables (and also go to the beginning or ending of a period).

The following example gives the beginning of the year, relative to datadate, by taking datadate (which is typically the last day of the month), and going to the beginning of the month 11 months prior. (So a datadate of December 31, will give January 1).

```SAS
data myDate (keep = gvkey fyear datadate sale at ceq boy);
set comp.funda;
/* only records from 2015 */
if  fyear eq 2015; 
/* standard filter (otherwise duplicate records) */
if indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C';
/* this will set boy to 12 months before datadate */
boy = intnx('month', datadate,-11, 'b');
format boy date9.;
run;
```



## Convert string to date

To convert a string to a date, you can use the input function.   `d = input( cdate , ANYDTDTE11.);`, or `d = input( cdate , ANYDTDTE10.);`.

```SAS

data myDate;
s = '27mar2018'; /* string, notice no 'd */
run;

data myDate2;
set myDate;
d = input( s , ANYDTDTE11.);
run;

proc print;run;
```

The above code turns the string into a date, and prints 21270, which is the internal representation of March 27, 2018.

## INTCK

The 'INTCK' function computes the difference between two dates (in terms of days, weeks, months, years). For example `diff = intck("month", boy, datadate)` will give the difference in months between boy and datadate (11 if boy is Jan 1 and datadate is Dec 31 of the same year).

See the [documentation](http://support.sas.com/documentation/cdl/en/lefunctionsref/63354/HTML/default/viewer.htm#p1md4mx2crzfaqn14va8kt7qvfhr.htm)