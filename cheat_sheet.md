# Quick review

## Remote submit

```SAS
%let wrds = wrds-cloud.wharton.upenn.edu 4016;options comamid = TCP remote=WRDS;
signon username=_prompt_;
```
### Local viewing of remote library

```SAS
Libname rwork slibref=work server=wrds;
```

### proc upload, download

Proc upload/download can also upload/download files (for example a macro).

```SAS
rsubmit;
proc upload data=work.myLocalDataset out=work.myRemoteDataset;run;
endrsubmit;
```

```SAS
rsubmit;
proc download data=comp.company out=work.company;run;
endrsubmit;
```

## Data step

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

Typically used to add a variable to an existing dataset, or to filter.

### Data step with retain

Retain can be used to remember something across the records

```SAS
data sumSale;
set randomSales;
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

Retain can be useful in combination with 'by'; repeat some logic for some industry, firm, year, etc.

## proc means

Basic descriptive statistics like mean, min, max, standard deviation, 25th percentile etc.

Useful in combination with 'by' (same statistics for each industry, firm, year, etc).


```SAS
proc means data=work.myData n mean median stddev;
  OUTPUT OUT=work.meansout n= mean= median= stddev= /autoname;
  var var1 var2 var3;
  by groupvar;
run;
```

## proc rank

Useful for grouping observations into terciles, quartiles, etc. Can be used with `by`.

```SAS
proc rank data = myComp out=myComp2 groups = 10;
var roa size ;    
ranks roa_d size_d ; 
run;
```

## proc standard

Useful for manipulating variables, like demeaning (giving 0 mean), or standardizing (0 mean, 1 standard deviation). Again also works with 'by'.

```SAS
proc standard data=dsIn mean=0 std=1 out=dsOut;
var at sale;
by fyear;
run;
```

## proc sort

```SAS
/* Sort by firm, year and audit fee (highest first) */
proc sort data=dataIn; by gvkey fyear descending auditfee; run;

/* Keep each first firm-year (keeps highest audit fee) */
proc sort data=dataIn nodupkey dupout=dataDropped; by gkvey fyear;run;
```

nodupkey is helpful in getting unique records

## proc import - Excel

Use `sheet` to specify a certain worksheet. There is also a `range` option. (For example `Range    = "sheet1$A11:3000";`)

```
proc import datafile = "E:\temp\dataset.xls" out = mydata dbms = xlsx REPLACE; sheet = "from_sas"; getnames = YES; run;
```

## proc export - Excel

Use `sheet` to specify a certain worksheet. Works with existing Excel files.

```
proc export data=myData outfile= "E:\temp\dataset.xls" dbms=xlsx replace; sheet="from_sas"; run;
```

## proc sql

- 'select': what to select into the output dataset (includes calculations),
- 'from': the input dataset (or datasets),
- 'where': filter on the input dataset (also used to specify what to join on for multiple datasets), 
- 'having': filter on the output dataset (for example newly calculated variables), and
- 'group by': grouping

### Subqueries

A 'from' does not have to be a dataset, it can be a query also.

Example:

```SAS
proc sql;	
	create table myData2 as
		/* which variables to select: fiscal year and compute market to book */
		select fyear, count(*) as numFirms, median(mtb) as median_mtb
		/* where to get it from: a subquery */
		from (
			select fyear, (csho * prcc_f / ceq) as mtb from comp.funda
			/* filter: get all firms in industry 7370 after 2000 */
			where SICH eq 7370 and fyear > 2000
			/* this is some boilerplate filtering (gets rid of doubles) */
			and indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C'
			)
		/* compute it for each year => GROUP BY */
		group by fyear;
quit;
```

### group by

The group by is used to aggregate. Example:

```SAS
proc sql;
  create table work.dsOut as select fyear, complex_d, count(*) as numobs
  from work.dsIn group by fyear, complex_d;
quit;
```

### inner join

Matching datasets

```SAS
/* match with Company */
proc sql;
  create table work.b_comp as select a.*, b.fic, b.sic, b.ipodate
  from work.a_funda a, comp.company b
  where a.gvkey = b.gvkey;
quit;
```


### left join

A left join includes records from the left side that have no matches with the right side.

Suppose we have a sample from Compustat and are interested in getting the acquisitions for these firms from SDC. If a firm has made no acquisitions, we still want to keep it in the output dataset.

```SAS
proc sql;
  create table work.dsOut as
  /* a.* means all records from work.dsIn
    b.effdate means select effdate from misc.sdc
  */
  select a.*, b.effdate 
  from work.dsIn a 
  /* perform a left join */
  left join misc.sdc b 
  /* require that gvkey is the same */
  on a.gvkey = b.gvkey 
  /* and that the effdate is in the 365 day period before datadate */
  and a.datadate - 365 <= b.effdate <= a.datadate;
quit;
```

### self join

A self join is a join where one dataset is used for both the left and right side. For example helpful to get lagged variables (example below).

```SAS
proc sql;
  create table work.dsOut as
  	select a.*, a.sale / b.sale - 1 as sales_growth, b.fyear as prev_fyear
  from
    work.dsIn a, work.dsIn b
  where
    a.gvkey = b.gvkey
    /* require that b is one year before 1 (i.e. lagged) */
    and a.fyear -1 = b.fyear; 
quit;
```

