# Using WRDS' website 

When you are working with the main databases (Compustat, CRSP, IBES, etc) the WRDS website has many useful resources:
- List of databases that you have access to
- Description of the data (tables with variables, manuals)
- Web form to collect data (fill out the form and download the data)  
- Sample programs


## Finding data

To find data: Click 'support' (top bar) and the 'Dataset List'. Navigate through vendors, tables. 

Helpful pages (require login):
- [List of databases/manuals](https://wrds-www.wharton.upenn.edu/pages/support/manuals-and-overviews/)
- [Search form to find variables](https://wrds-www.wharton.upenn.edu/search/?queryTerms=&activeTab=navVariablesSearchTab)
- [Find individual companies](https://wrds-www.wharton.upenn.edu/attribute-search/)
- [List of subscriptions](https://wrds-www.wharton.upenn.edu/pages/get-data/)



### Examples:

[Variables in Compustat Fundamental Annual](https://wrds-www.wharton.upenn.edu/pages/get-data/compustat-capital-iq-standard-poors/compustat/north-america-daily/fundamentals-annual/) -- then click on 'Variable Descriptions' (second option on form)


### Using the web form to download data

To retrieve data from WRDS you can use the web forms. Click 'Get Data' (top bar) and select the vendor. Note that it helps to first go through the manuals/variable lists to see what data is available. 

In this course we will programmatically retrieve the data with SAS code, which is more efficient. SAS also allows us to filter the dataset and join it with other datasets, and download the resulting dataset (likely to be much smaller in size compared with the full datasets). 