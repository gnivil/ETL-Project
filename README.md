## ETL Project - Rich Kirschenheiter, Stefany Lima, Christian A. Reyes

# Project Proposal
We will be comparing hospital data to income data. We will be getting data from [Kaggle](kaggle.com) and analysing the following:

[USA Hospitals](https://www.kaggle.com/carlosaguayo/usa-hospitals)

[US Household Income Statistics](https://www.kaggle.com/goldenoakresearch/us-household-income-stats-geo-locations?select=kaggle_income.csv)

The data will be saved in .csv files and extracted from there. Significant transformations will be done to get the tables we want. 
We plan to aggregate Zip codes and create or tables based on that. Data will be stored on SQL through PostgreSQL.

-----

# Project Report

## Extract
Project data is sourced from Kaggle.com. We worked with data from .csv files for USA Hospitals and US Household Income Statistics. The files were saved locally and the data was extracted into our Jupyter Notebook using the Pandas read_csv() function. 

Hospitals.csv: This dataset is provided by the Homeland Infrastructure Foundation-Level Data (HIFLD) without a license and for Public Use. HIFLD Open GP - Public Health. Shared By: jrayer_geoplatform. Data Source: services1.arcgis.com 
This dataset was downloaded on March 23, 2019 at [this link](https://hifld-geoplatform.opendata.arcgis.com/datasets/a2817bf9632a43f5ad1c6b0c153b0fab_0).

income.csv: 2011-2015 ACS 5-Year Documentation was provided by the U.S. Census Reports. Retrieved August 2, 2017, from [this link](https://www2.census.gov/programs-surveys/acs/summary_file/2015/data/5_year_by_state/).
```python
#import dependacies
import pandas as pd
from sqlalchemy import create_engine

#reading csv file into dataframe for hospitals
csv_file = 'Resources/Hospitals.csv'
hospital_data_df = pd.read_csv(csv_file)
hospital_data_df.head()

#reading csv file into dataframe for income
csv_file = 'Resources/income.csv'
income_data_df = pd.read_csv(csv_file)
income_data_df.head()
```
 
## Transform
We performed several transformations on both datasets in order to compare them. After reading in the csv files, we used the pandas.DataFrame.duplicated() function to check for duplicates within the hospital and the income data sets. After making sure they all read False, we started transforming the data. We did this to ensure the largest dataset possible was checked for duplicates before breaking it down. 
```python
#check for duplicates in hospitals dataframe
mask = hospital_data_df.duplicated()
hospital_data_df[mask]

#check for duplicates in income dataframe
mask = income_data_df.duplicated()
income_data_df[mask]
```
[***USA Hospitals***](https://github.com/gnivil/ETL-Project/blob/main/Resources/Hospitals.csv)

This dataset originally had 34 columns, and our final table is reduced down to 10 columns. 
* 11 columns dropped [“X”, “Y”, “ADDRESS”, “ZIP4”, “COUNTY”, “COUNTYFIPS”, “COUNTRY”, “LATITUDE”, “LONGITUDE”, “STATE_ID”] contained overly-specific geographical data that is not needed since the table is joined by Zip code. 
* 8 columns dropped [“WEBSITE”, “ALT_NAME”, “TTL_STAFF”, “TRAUMA”, “HELIPAD”, “TELEPHONE”, “NAICS_CODE”, NAICS_DESC”] contained hospital information beyond the scope of our analysis. 
* 6 columns dropped [“OBJECTID”, “SOURCE”, “SOURCEDATE”, “VAL_METHOD”, “VAL_DATE”, “ST_FIPS”] listed data relevant to the data collection process that was not relevant to our analysis. 
```python
#create a new df for just the columns we want for the hospitals dataframe
hospital_cols = ["ID", "ZIP", "STATE", "CITY", "POPULATION", "BEDS", "NAME", "OWNER", "STATUS", "TYPE"]
hospital_transformed = hospital_data_df[hospital_cols].copy()
```
The remaining columns were renamed to the following; "ID":"Hospital_id", "ZIP":"Zip", "STATE":"Us_State", "CITY":"City", "POPULATION":"Pop_100k", "BEDS":"Beds", "NAME":"Hospital_Name", "OWNER":"Type_Owner", "STATUS":"Status", "TYPE":"Care". We assigned variable hospital_cols to the list of columns we kept in our table containing hospital data. These columns were then set in a new dataframe named hospital_transfomred.
* "Hospital_id": used as unique primary key
* "Zip":column to join on between the two tables
* "State": for analysis based on state
* "City": for analysis based on city
* "Pop_100k": for analysis based on population served by each healthcare center per 100k people
* "Beds": for analysis based on hospital capacity
* "Hospital_Name": for more meaningful output options
* "Type_Owner": for analysis between non-profit, profit and government owned hospitals
* "Status": to indicate if a hospital is open or closed
* "Care": for information on type of care provided
```python
#rename the column headers
hospital_transformed = hospital_transformed.rename(columns= {"ID":"Hospital_id", 
                                                             "ZIP":"Zip", 
                                                             "STATE":"Us_State", 
                                                             "CITY":"City", 
                                                             "POPULATION":"Pop_100k", 
                                                             "BEDS":"Beds", 
                                                             "NAME":"Hospital_Name", 
                                                             "OWNER":"Type_Owner", 
                                                             "STATUS":"Status", 
                                                             "TYPE":"Care"})
income_transformed = income_transformed.rename(columns= {"Zip_Code":"Zip",
                                                         "sum_w":"Households"})
```
[***US Household Income Statistics***](https://github.com/gnivil/ETL-Project/blob/main/Resources/income.csv)

This dataset originally had 19 columns and our final table is reduced down to 6 columns. 
* The following 13 columns - [“State_Code”, “State_Name”, “State_ab”, “County”, “City”, “Place”, “Type”, “Primary”, “Area_Code”, “ALand”, “AWater”, “Lat”, “Lon”] - were dropped since they contained geographical information redundant with the zip code. We set variable income_cols for these columns kept: ["id", "Zip_Code", "Mean", "Median", "Stdev", "sum_w"].
```python
#create a new df for just the columns we want for the hospitals dataframe
income_cols = ["id", "Zip_Code", "Mean", "Median", "Stdev", "sum_w"]
income_transformed = income_data_df[income_cols].copy()
```
Only two of these columns were then renamed: "Zip_Code" to "Zip", "sum_w" to "Households".  We set variable income_cols for these columns kept: ["id", "Zip", "Mean", "Median", "Stdev", "Households"].
* "id": used as unique primary key
* "Zip": column to join on between the two tables
* "Mean": income summary statistic
* "Median": income summary statistic
* "Stdev": income summary statistic
* "Households": number of households used to calculate summary statistics
```python
#rename the column headers
income_transformed = income_transformed.rename(columns= {"Zip_Code":"Zip",
                                                         "sum_w":"Households"})
```
Upon reviewing the data, we uncovered locations of Hospitals for 50 US states and US territories of Puerto Rico(PR), Guam(GU), American Samoa(AS), Northern Mariana Islands(MP), Palau(PW), and Virgin Islands(VI). However, the income.csv only contained the income data for the 50 States and PR. We kept this data for future analysis when this data becomes available. We also determined that we could leave off the country column as the territories were listed under state as well. 
 
The tables for both datasets are joined by Zip Code and both tables are compared based on this index. 

## Load
PostgreSQL was selected as our final production database because it offers totally integrated data storage and access, allowing us to perform complex queries while offering speed and security.
 
The cleaned data was loaded into the database named ‘Income_vs_healthcare’ using the pandas .to_sql function. The final tables storing the data are named “healthcare” and “income”. The PRIMARY KEY in the "healthcare" table is "Hospital_id" and "id" in the “income” table. We found these unique identifiers were best suited to be PRIMARY KEYS, as they were unique to each table.
~~~~sql
DROP TABLE healthcare;
DROP TABLE income;

CREATE TABLE healthcare (
	"Hospital_id" INT PRIMARY KEY,
	"Zip" INT,
	"Us_State" TEXT,
	"City" TEXT,
	"Pop_100k" INT,
	"Beds" INT,
	"Hospital_Name" TEXT,
	"Type_Owner" TEXT,
	"Status" TEXT,
	"Care" TEXT
);

CREATE TABLE income (
	"id" INT PRIMARY KEY,
	"Zip" INT ,
	"Mean" INT,
	"Median" INT,
	"Households" INT,
	"Stdev" INT
);
~~~~
We encountered errors loading the table columns onto the SQL table. We debugged by adding double quotation marks to the column names and ensured the column names did not have a corresponding sql function, such as STATE. We had to rename the columns several times in the Jupyter notebook before pgAdmin pulled them correctly.

-----
