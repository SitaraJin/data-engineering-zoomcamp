## Module 2 Homework

ATTENTION: At the end of the submission form, you will be required to include a link to your GitHub repository or other public code-hosting site. This repository should contain your code for solving the homework. If your solution includes code that is not in file format, please include these directly in the README file of your repository.

> In case you don't get one option exactly, select the closest one 

For the homework, we'll be working with the _green_ taxi dataset located here:

`https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/green/download`

To get a `wget`-able link, use this prefix (note that the link itself gives 404):

`https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/`

### Assignment

So far in the course, we processed data for the year 2019 and 2020. Your task is to extend the existing flows to include data for the year 2021.

As a hint, Kestra makes that process really easy:

1. You can leverage the backfill functionality in the [scheduled flow](../../../02-workflow-orchestration/flows/07_gcp_taxi_scheduled.yaml) to backfill the data for the year 2021. Just make sure to select the time period for which data exists i.e. from `2021-01-01` to `2021-07-31`. Also, make sure to do the same for both `yellow` and `green` taxi data (select the right service in the `taxi` input).
2. Alternatively, run the flow manually for each of the seven months of 2021 for both `yellow` and `green` taxi data. Challenge for you: find out how to loop over the combination of Year-Month and `taxi`-type using `ForEach` task which triggers the flow for each combination using a `Subflow` task.

### Quiz Questions

Complete the Quiz shown below. It’s a set of 6 multiple-choice questions to test your understanding of workflow orchestration, Kestra and ETL pipelines for data lakes and warehouses.

1) Within the execution for `Yellow` Taxi data for the year `2020` and month `12`: what is the uncompressed file size (i.e. the output file `yellow_tripdata_2020-12.csv` of the `extract` task)? 
- 128.3 MB

- 134.5 MB

- 364.7 MB

- 692.6 MB

> Answer: 128.3MB.
> 
> Excute the flow code 02_postgres_taxi_extract.yaml, select the Yellow, the year 2020 and month 12, and then check the outputs of the flow.

2) What is the rendered value of the variable `file` when the inputs `taxi` is set to `green`, `year` is set to `2020`, and `month` is set to `04` during execution?  
- `{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv` 
- `green_tripdata_2020-04.csv`
- `green_tripdata_04_2020.csv`
- `green_tripdata_2020.csv`

> Answer: green_tripdata_2020-04.csv
> 
> Excute the flow code 02_postgres_taxi_extract.yaml, select the green, the year 2020 and month 04, and then check the outputs of the flow.

3) How many rows are there for the `Yellow` Taxi data for all CSV files in the year 2020? 
- 13,537.299
- 24,648,499
- 18,324,219
- 29,430,127

> Answer: 24,648,499
> 
> Create the flow by flow code 02_postgres_taxi_scheduled.yaml, and then set the Triggers to backfill the data of the yellow taxi and the year 2020. Query the postgres database.

4) How many rows are there for the `Green` Taxi data for all CSV files in the year 2020? 
- 5,327,301
- 936,199
- 1,734,051
- 1,342,034

> Answer: 1,734,051
> 
> Create the flow by flow code 02_postgres_taxi_scheduled.yaml, and then set the Triggers to backfill the data of the green taxi and the year 2020. Query the postgres database.

5) How many rows are there for the `Yellow` Taxi data for the March 2021 CSV file? 
- 1,428,092
- 706,911
- 1,925,152
- 2,561,031

> Answer: 1,925,152

6) How would you configure the timezone to New York in a Schedule trigger?
- Add a `timezone` property set to `EST` in the `Schedule` trigger configuration  
- Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration
- Add a `timezone` property set to `UTC-5` in the `Schedule` trigger configuration
- Add a `location` property set to `New_York` in the `Schedule` trigger configuration 

>  Answer: Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration

## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2025/homework/hw2
* Check the link above to see the due date

## Solution

Will be added after the due date
