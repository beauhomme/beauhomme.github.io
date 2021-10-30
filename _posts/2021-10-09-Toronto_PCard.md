---
layout: post
title: "Purchase Card Data Analysis"
subtitle: "City of Toronto's monthly reports of its employee's expense card transactions"
date: 2021-10-15 10:45:13 -0400
background: '/img/toronto_city.jpg'
---

## TASK 1 – Obtain and Clean Data Clone the Repository

Run the steps below to begin

  1. Download or clone the repository from [https://github.com/beauhomme/toronto_pcard.git](https://github.com/beauhomme/toronto_pcard.git).
  2. Install the project dependencies according to the `requirements.txt file`.
  3. Execute the `getfiles.py` to fetch/download the datafiles to the infiles directory path. This is the location we the application expects to load files from. Files are expected in .xlsx formats. A scheduled job will run daily and would process any files in the directory.
  4. Log on to your demo environment, then execute the database scripts in the directory, to prepare the database and warehouse for the rest of the project.
  5. When steps 1, 2 and 3 are completed. You can open the mergecleanfile.py. Update/Populate your DB credentials and connection properties (MSSQL Database was used for this project).


## TASK 2 - Create a Data Model

The data files downloaded are in an unnormalized state. To normalize the data, I applied the 1st to 3rd data normalization rules, which involves removing repeating groups, creating tables for each sets of related data, creating primary keys, ensuring no partial dependencies on the keys and also no transitive dependencies on non-prime attributes. I was able to create the below data model, mapping entities, their attributes and relationships. 

*Reasons behind the choices -*

To satisfy **First normal form**, each column of a table must have a single value. Columns which contain sets of values or nested records are not allowed – The dataset is already in this form.

The Transaction table has one candidate key `Batch-ID` (which is therefore the primary key). To conform to **Second Normal Form** and remove duplicities, every non candidate-key attribute must depend on the whole candidate key, not just part of it. – All the columns on the dataset depend on the candidate key Batch ID that uniquely identifies the transaction.

To satisfy the **Third Normal Form**, there shall be no transitive dependences on non-prime attributes, in this case the table or data violates this form {Merchant Type, Merchant Description} depend on the Merchant Name, {GL Description} on GL Account, {Cost Center Description} on Cost Center No. We take care of this by placing the sets in their own respective tables and have a foreign key relationship on them.

Hence our Data Model ERD looks like this
![Pcard ERD ](/img/pcard_erd.png)

## Task 3 - Data Pipeline and Warehouse

Using a data ETL/pipeline and Orchestrator tool – Airflow and SSIS, I defined two (2) tasks with different intervals (unfortunately I could not meet the deadline to complete the design”

1. `fetchClean` – will execute different functions to download the files from the online repository as in the demo case or otherwise, extract and move the files to the /infiles directory path, cleanse, save the resulting datafiles to the output directory - /outfiles. It will also create, update and load accordingly the pcard Database Tables in line with the model above. Schedule Interval is daily at a set time.
2. `loadDW` – will extract and load the data from the various databases into a raw-data environment, stage and transform the data into the dimension and facts table of the pcard Data Warehouse. Schedule Interval is weekly at a set time.

To preserve history, we will implement a slowly changing dimension (SCD – Type 2) across the dimensions. Using the DimDivision (Division Dimension) table on the warehouse. The primary key division_id which is also the surrogate key is an identity property which will generate a new value for every row. Let’s imagine we’d like to keep history on the purchases authorized by the division supervisor (represented by the `division_sup_id`). Any time a new supervisor is assigned to the division, we don’t update the row, but insert a new record. Using the start_date and end_date fields we would track when a record was valid in time. A new surrogate key (`division_id`) is generated, but the business key – `division_name` remains the same. when a fact table is loaded, a lookup will be done on the `DimDivision` table, depending on the flag column (IsCurrent) the current attributes of the row are returned. Hence, we are able to track data lineage and history. For this data warehouse, the schema we have implemented is a STAR – Schema to keep the structure simple and flexible.

Therefore our Data warehouse Model ERD looks like this
![Pcard DW ERD ](/img/pcardDW_erd.png)


## Task 4 – Visualization

The visualization tells a visual story across five (5) reports. Each of the reports makes it easy to analyze, slice, drill down and up across multiple indices.

The visualization can be found [here](https://github.com/beauhomme/toronto_pcard/blob/main/City%20of%20Toronto.pdf)

A quick breakdown of the visualizations:
*	Page1 - a Summary report or a quick overview of the numbers in the dataset, 
*	Page2 – is about expenses across different categories – divisions, GLs, Cost Centers, Merchants etc.
*	Page3 - that displays the weekly purchases/expenses by divisions, one can also filter by merchants, by months and other date dimensions
*	Page4 – shows spending across different merchants by year. We can also drill down by divisions, and time
*	Page5 – shows a view over the period of the dataset. 


