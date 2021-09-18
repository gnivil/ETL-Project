# ETL Project - Rich Kirschenheiter, Stefany Lima, Christian A. Reyes


# Project Proposal

We will be comparing hospital data to income data. We will be getting data from Kaggle, links below:
https://www.kaggle.com/carlosaguayo/usa-hospitals
https://www.kaggle.com/goldenoakresearch/us-household-income-stats-geo-locations?select=kaggle_income.csv

The data will be saved in .csv files and extracted from there. 
Significant transformations will be done to get the tables we want. 
We plan to aggregate Zip codes and create or tables based on that.
Data will be stored on SQL through postgres.

# Project Report

* Extract
Project data is sourced from Kaggle.com. We worked with data from .csv files for USA Hospitals and US Household Income Statistics. The files were saved locally and the data was extracted into our Jupyter Notebook. 

* Transform
We performed several transformations on both datasets in order to compare them based on the information we found the most relevant. 

The data for both datasets is aggregated by Zip Code and both tables are compared based on this index. 

For the USA Hospitals dataset, we dropped the following columns; INSERT COLUMNS DROPPED. We filtered on INSERT ORIGINAL COLUMNS NAMED. These columns were renamed to the following: Zip, Us_State, City, Pop_100k, Beds, Name, Owner, Car. The dataset was cleaned by removing duplicates.

For the US Household Income Statistics dataset, we dropped the following columns; INSERT COLUMNS DROPPED. We filtered on INSERT ORIGINAL COLUMNS NAMED. These columns were renamed to the following: INSERT NEW COLUMN NAMES. The dataset was cleaned by removing duplicates.

* Load
PostgreSQL was selected as our final production database because it offers totally integrated data storage and access, allowing us to perform complex queries while offering speed and security.

The data was loaded into a database named ‘income_vs_healthcare’. The final tables storing the data are named ‘healthcare’ and ‘income.’
