# Data step

## Overview

The 'data step' is one of the main procedures in SAS. Typically, in the data step the input dataset is transformed into the output dataset. The transformation can be adding newly created variables, an aggregation (sum, min, max, etc), filtering of records, apply conditional code ('if-then-else'), loops, etcetera. The data step can also be used to create/import data from disk. 

### Examples

Example usage of the data step where data is input with 'datalines':

## Example using 'datalines'

```SAS
data ratingdata;
  input @01 id        1.
        @03 startyr   4.
        @08 endyr     4.
	@13 rating    $1.
	;
datalines;
1 2002 2004 A
1 2005 2007 A
1 2007 2009 B
1 2009 2010 A
2 2002 2004 A
3 2001 2003 B
3 2003 2004 B
3 2005 2007 B
run;
```

'datalines' is helpful to create small datasets/examples. The input portion (lines 2-4) describe the variables. It provides the starting position (@01 means first position), the variable name, and the so-called format ('1.' means a number of length 1, '4.' means a number of length 4, '$1.' means a string of length 1). So '@13 rating $1.' means that the letter at the 13th position will be the variable 'rating'.

## Using 'set'

When an existing dataset is used as an input to create a new dataset, the 'set' statement is used to specify that dataset.

Example:

```SAS
data rating2;
set ratingdata; /* input dataset */
/* filter - eq is short for equal */
if rating eq 'A';
/* another filter - ne is short for not equal */
if startyr ne 2001;
/* new variable */
length = endyr - startyr;
run;
```

In the above example, the data is filtered (only records where rating is 'A' are included, and where startyr does not equal 2001), and a new variable 'length' is added.

## Conditional code

The above example uses the 'if' statement to filter the records. It can also be used to conditionally create new variable:

```SAS
data rating3;
set rating2;
if startyr < 2005 then do;
	period = 1;
	/* other statements to execute if startyr is less than 2005 can go here */
end;
else do;
	period = 0;
	/* other statements can go here */
end;
run;
```

In case there is just one assignment in the if/else, it can be shortened to:

```SAS
data rating3;
set rating2;
if startyr < 2005 then period = 1; else period = 0;	
run;
```

In this particular setting (to create an indicator variable), there is a shorthand. The code within parenthesis (`startyr < 2005`) is evaluated. If true, it evaluates to 1, if false, to 0:

```SAS
data rating3;
set rating2;
period = (startyr < 2005); 
run;
```

### do-while, do-until

There are also the do while and do until statements, which use an expression as opposed to a known number of repeats. See [do while](http://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a000201927.htm) and [do until](http://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a000201924.htm) in the SAS documentation.

## Missing values

A missing value is '.' (a period) for numbers, and "" (empty string) for text. Internally, missing numbers are represented by a large negative number. So, `period = (startyr < 2005); ` will give a value of 1 for period when startyr is missing. 

To test for missing values, use the 'missing' function:

```SAS
data rating4;
set rating2;
/* lets set startyr to missing for odd start years */
if mod(startyr,2) eq 0 then startyr = .;
/* create dummy for missing values */
startMiss = missing(startyr);
/* let's see if it indeed is a problem */
period = (startyr < 2005); 
run;
```

Another helpful function is cmiss, which returns the number of missing variables. For example `nrMiss = cmiss (of startyr endyr);`, which will set nrMiss to however variables are missing. 

## Loops

To repeat/loop over values, use the 'do' loop:

```SAS
data randomSales;
do id = 1 to 100;
   do year = 1 to 10;
	   x = 2010 + year ;
	   u = rand("Uniform"); /* random number uniform distribution between 0 and 1 */
	   sales =  100 * u ;  /* random number between 0 and 100 */
	   output; /* force new record */
   end; /* end of year loop */	  
end; /* end of id loop */
run;
```

In this example, two loops are nested to create a dataset of 100 'random' firms with 10 years of data. j will loop from 10 to 20 ten times. The 'output' keyword forces a new record 100 x 10 times. Without this keyword, only one record would be created (where id is 100, year is 20).


## 'drop', 'keep'

The 'drop' and 'keep' keywords specifiy which variables to drop or keep. See for an example below (wide to long transformation).


## 'wide' to 'long' transformation

Consider the following dataset:

```SAS
data somedata;
  input @01 id        1.
        @03 year2010  4.
		@08 year2011  4.
        @13 year2012  4.
		@18 year2013  4.
	;
datalines;
1 1870 1242 2022 1325
2 9822 3186 1505 8212
3      1221 4321 9120
4 4701 2323 3784
run;
```

The above data is in 'wide' format (that is, a record has data for multiple years for each firm). To conver this to 'long' format (each record has data for one firm-year):

```SAS
data long (keep = id value);
set somedata;
value = year2010; output;
value = year2011; output;
value = year2012; output;
value = year2013; output;
run;
```
The 'output' triggers the output of a new record. Each time, a year201x variable is set into 'value'. The relevant columns for the output dataset are 'id' and 'value'.

## 'retain', 'by', 'first.' and 'last.'

Many of the procedures in SAS allow some code to be applied by groups. To indicate that something needs to be done for each group, the 'by' statement is included to indicate that is the case. This requires the dataset to be sorted first.

Using the dataset with randomly generated sales (see above, loops):

```SAS
/* sort by year */
proc sort data=randomSales; by year;run;

data sumSale;
set randomSales;
/* by indicates to partition on year */
by year;
/* retain remembers variables across records */
retain sum;
/* initialization is needed first.year is true if the first record for a year is processed*/
if first.year then sum = 0;
/* update sum -- this is remembered through retain */
sum = sum + sales;
/* last.year is true for the last record in a year */
if last.year then output;
run;
```

The above code sums sales by year, so the 100 firms x 10 years (1,000 records) will be aggregated to 10 records (one for each year).

'retain' is used to remember things across records. In this case, a new variable sum is created, and sales in each record is added to sum. It is important that whatever is retained is not already on the dataset. For example, if a variable sum already existed, the retained sum would be overwritten by the sum on the dataset. 

In the above example, the sum of sales could also be determined in other ways (proc means, proc sql), but the 'retain' mechanism is very flexible and can be used to solve issues that are hard to solve otherwise.

