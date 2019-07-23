# Sorting

Sorting is straightforward. Use proc sort with data= to set the dataset that needs sorting. Add by with the variables that you want to sort on. The default is ascending; use the descending keyword for sorting high-to-low.

## Nodupkey

Proc sort has an attractive feature where you can specify unique records for specific variables.

For example, if you want to have only 1 record for each firm-year, you can add 'nodupkey' with 'by gvkey fyear'. You can have an initial sort where the records are ordered such that the 'best' record comes first. For example, if Audit Analytics has some multiple entries for some firm-years, and you want to keep the audit firm with the highest fee, first sort by firm-year and descending audit fee, and then the nodupkey as follows:

```SAS
/* Sort by firm, year and audit fee (highest first) */
proc sort data=dataIn; by gvkey fyear descending auditfee; run;

/* Keep each first firm-year (keeps highest audit fee) */
proc sort data=dataIn nodupkey dupout=dataDropped; by gkvey fyear;run;
```

# Nodup

nodup or (noduprec) is similar to nodupkey. The difference is that completely duplicate observations are dropped (i.e., all variables are identical, not just the same key).

# dupout

Including `dupout=dataOut` will create a dataset with the observations that are deleted when including the nodup or nodupkey keyword. For sample use see example for nodupkey above.