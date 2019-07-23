# proc SQL

SAS support SQL, which is a language to query databases.



## General

A proc sql statement typically has the following sections:

- 'select': what to select into the output dataset (includes calculations),
- 'from': the input dataset (or datasets),
- 'where': filter on the input dataset (also used to specify what to join on for multiple datasets), 
- 'having': filter on the output dataset (for example newly calculated variables), and
- 'group by': grouping

## Overview

For an overview of proc SQL, please read [this pdf](pdf/intro_sql.pdf) first.  

## Example select and filter

```SAS
proc sql;
  create table work.a_funda as 
  select gvkey, fyear, datadate, sale, at
  from comp.funda
  where indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C';
quit;
```

The SQL specifies which table to create (work.a_funda), and which variables need to be included (select gvkey, fyear, datadate, sale, at), and what the source dataset is (from comp.funda). Finally, a filter is applied (where indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C'). 

## Group by

The group by is used to aggregate. Example:

```SAS
proc sql;
  create table work.dsOut as select fyear, complex_d, count(*) as numobs
  from work.dsIn group by fyear, complex_d;
quit;
```

The above query counts the number of records for each fyear-complex_d combination. Other functions you can use are min, max, median, mean, etc.


## Example inner join

```SAS
/* match with Company */
proc sql;
  create table work.b_comp as select a.*, b.fic, b.sic, b.ipodate
  from work.a_funda a, comp.company b
  where a.gvkey = b.gvkey;
quit;
```

The SQL has two datasets as inputs. The 'a' and 'b' after the dataset names allow to refer to the datasets by a and b, respectively. The 'a.*' means all columns from a (work.a_funda). The where specifies how the two datasets need to be linked, `a.gvkey = b.gvkey` means that we want to match on the firm identifier. Without this where clause, all potential matches are made (so if a is 10,000 records and b is 1,000 records, then that would result in 10,000 x 1,000 = 10,000,000 records).

This type of join is an inner join. If there is some gvkey on a, but not on b (or the other way around), it will not be on the output dataset. In other words, records need to be on both the left and right side to be included in the output.



## Left join

A left join includes records from the left side that have no matches with the right side.

Example: 

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

> Note that with a left join, the 'where' becomes an 'on'

## Self join

A self join is a match of a table on itself, meaning that the left and right side of the join is the same table. This is helpful for creating leading and lagging variables.

The first example computes sales growth, the second example selects the last 5 years of return on assets.

```SAS
proc sql;
  create table work.dsOut as
  /* select a.* means select all variables in a
     select a.sale / b.sale - 1 means divide sales of year a by sales of year b minus 1 
     as sales_growth renames this to sales growth
     b.fyear is included for debugging (to make sure prev_fyear is indeed previous year)
     */
  select a.*, a.sale / b.sale - 1 as sales_growth, b.fyear as prev_fyear
  from
    work.dsIn a, work.dsIn b
  where
    a.gvkey = b.gvkey
    /* require that b is one year before 1 (i.e. lagged) */
    and a.fyear -1 = b.fyear; 
quit;
```

```SAS
proc sql;
  create table work.dsOut as
  /* roa is computed on the fly (ni: net income, at: total assets)*/
  select a.*, b.ni / b.at as roa
  from
   work.dsIn a, work.dsIn b
  where
    a.gvkey = b.gvkey
    /* this year and 4 years before */
    and a.fyear -4<= b.fyear <= a.fyear -0; 
quit;
```

## Subqueries

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