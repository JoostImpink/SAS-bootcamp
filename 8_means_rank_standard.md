# Proc means, proc rank, proc standard


## Proc means

Proc means is used to compute statistics. 

Common supported statistics:

- Keyword	Description
- N Counting
- SUM Sum
- MIN	Minimum
- MEAN	Mean
- MEDIAN	Median
- MAX	Maximum
- STD	Standard deviation
- QRANGE	Interquartile range
- VAR	Variance
- SKEW	Skew

In addition, there are several percentiles: P1, P5, P10, P25, P50 (=Median), P75, P90, P95 and P99.

Example:

```SAS
/* summary statistics for full sample */
proc means data=work.myData n mean  median stddev;
  OUTPUT OUT=work.meansout n= mean= median= stddev= /autoname;
  var var1 var2 var3;
run;
```

The above example computes the count, mean, median and standard deviation for var1, var2, and var3, and will print these, as well as generate a dataset (`output out=`).

### Suppressing output to screen


By default, proc means will print the statistics to the screen. To supress this, add the 'noprint' keyword (`proc means data=work.myData noprint`). 

### Autoname

The autoname keyword will automatically create variable names for the different statistics, by concatenating the variable names with the statistic (in the example, var1_N, var1_Mean, var1_Median, var1_Stddev, similarly for var2). The output dataset will also hold a frequency count (_FREQ_). The difference between _N and _FREQ_ is that missings are included in _FREQ_, but not counted towards _N_.


### By

Proc means supports 'by', which means that statistics can be computed by group (for example, by firm, industry, year, etc). Example:

```SAS
/* as above, but repeated by a group (requires sorting on that variable) */
proc means data=work.myData n mean median stddev;
  OUTPUT OUT=work.meansout n= mean= median= stddev= /autoname;
  var var1 var2 var3;
  by groupvar;
run;
```

When 'by' is used, the output dataset will have one row for each group. A column is added that holds the variable used to group by (in the example 'groupvar').


## Proc rank

Proc rank is used to create ranked variables, and variables to create low/high, terciles, quantiles, etc.

Example: 

```SAS
proc rank data = myComp out=myComp2 groups = 10;
var roa size ; 		
ranks roa_d size_d ; 
run;
```

'var' holds the list of variables to rank, and 'ranks' holds the variable names to give the ranked variables.

The above example will create two decile rank variables (roa_d and size_d), with values 0-9. If roa and size have missing values, the ranked variable will also have a missing value.

Proc rank supports the 'by' keyword, for example to rank by year, or industry. 


## Proc standard

Proc standard is used to demean (subtract the mean) or standardize (mean 0, standard deviation 1).

Example:

```SAS
proc standard data=dsIn mean=0 std=1 out=dsOut;
var at sale;
by fyear;
run;
```

The above example standardizes the data by year (so for each year the variables at and sale will have an average of 0 and standard deviation of 1).