# Lab 1: Serverless Analysis of data in Amazon S3 using Amazon Athena

* [Amazon Athena 데이터 베이스 및 테이블 생성](#creating-amazon-athena-database-and-table)
    * [Athena 데이터 베이스 생성하기](#데이터-베이스-생성하기)
    * [Athena 테이블 생성](#테이블-생성하기)  
* [Querying data from Amazon S3 using Amazon Athena](#querying-data-from-amazon-s3-using-amazon-athena)
* [Querying partitioned data using Amazon Athena](#querying-partitioned-data-using-amazon-athena)
    * [Create Athena Table with Partitions](#create-a-table-with-partitions)
    * [Adding partition metadata to Amazon Athena](#adding-partition-metadata-to-amazon-athena)
    * [Querying partitioned data set](#querying-partitioned-data-set)
* [Creating Views with Amazon Athena](#creating-views-with-amazon-athena)
* [CTAS Query with Amazon Athena](#ctas-query-with-amazon-athena)
    * [Create an Amazon S3 Bucket](#create-an-amazon-s3-bucket)
    * [Repartitioning the dataset using CTAS Query](#repartitioning-the-dataset-using-ctas-query)
    * [Repartitioning and Bucketing the dataset using CTAS Query](#repartitioning-and-bucketing-the-dataset-using-ctas-query)
      
## Architectural Diagram
![architecture-overview-lab1.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/Screen+Shot+2017-11-17+at+1.11.18+AM.png)

## Amazon Athena 데이터베이스 및 테이블 생성

Amazon Athena는 Apache Hive를 사용하여 테이블을 정의하고 데이터베이스를 생성합니다. 데이터 베이스는 테이블의 논리적 그룹입니다. Athena에서 데이터베이스와 테이블을 만들 때, 당신은 단순히 Amazon S3에 있는 테이블 데이터의 스키마와 위치를 기술하고 있습니다. Hive의 경우, 기존의 관계형 데이터베이스 시스템과 달리 데이터베이스와 테이블은 스키마 정의와 함께 데이터를 저장하지 않습니다.  이 데이터는 테이블을 query 할 때만 Amazon S3에서 읽히게 됩니다. Hive 사용의 또 다른 이점은 Hive에서 발견된 [metastore](https://wikidocs.net/23282)가 Spark, Hadoop, Presto와 같은 다른 많은 빅데이터 어플리케이션에서 사용될 수 있다는 것입니다. Athena 카탈로그를 사용하면, Hadoop 클러스터 또는 RDS 인스턴스를 [provisioning]([https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EB%B9%84%EC%A0%80%EB%8B%9D](https://ko.wikipedia.org/wiki/프로비저닝)) 할 필요없이 클라우드에 Hive와 호환되는 metastore를 구축할 수 있습니다. 데이터베이스 및 테이블 생성에 대한 지침은 [Apache Hive documentation](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL) 을 참조하십시오.  다음 단계에서 특별히 Amazon Athena를 위한 지침을 제공합니다. 

### 데이터 베이스 생성하기

1.  [AWS Management Console for Athena](https://console.aws.amazon.com/athena/home) 를 여십시오.
2.  AWS Management Console for Athena를 처음 방문하는 경우 시작하기 페이지가 표시됩니다. **Get Start** 를 선택하여 **Query Editor** 를 여십시오. 이번이 처음이 아니라면 Athena **Query Editor** 가 열립니다.
3. AWS 지역이름을 기록하십시오. 예를들면 이 lab의 경우 **US West (Oregon)** 지역을 선택해야 합니다.
4. Athena **Query Editor** 에서 예제 query가 있는 query 창을 볼 수 있습니다. 이제 query창에서 query 입력을 시작할 수 있습니다.
5. *mydatabase* 라는 데이터베이스를 생성하려면 다음 문장을 복사하고 **Run Query** 를 선택하십시오: 

````sql
    CREATE DATABASE mydatabase
````

6.	**Catalog** 대시보드의 데이터베이스 목록에 *mydatabase*가 표시되는지 확인하십시오. 

![athenacatalog.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacatalog.png)

### 테이블 생성하기
이제 데이터베이스를 가지고 있으므로 뉴욕 택시 샘플 데이터를 기반으로 한 테이블을 만들 준비가 되었습니다. 데이터에 매핑되는 열을 정의하고, 데이터의 구분 방법을 지정하고, 파일에 대한 Amazon S3의 위치를 제공하십시오.

>**Note:** 
>테이블을 작성할 때 다음 사항을 고려하십시오 :
>
>-	Amazon S3  위치에서 데이터로 작업하려면 적절한 사용권한이 있어야 합니다. 자세한 내용은 [Setting User and Amazon S3 Bucket Permissions](http://docs.aws.amazon.com/athena/latest/ug/access.html) 을 참조하십시오. 
>-	데이터가 Amazon S3에서 암호화되지 않는 한 Athena를 실행하는 기본영역과 다른영역에 데이터가 있을수 있습니다. Amazon S3의 표준 지역간 데이터 전송 속도는 표준 Athena 요금과 함께 적용됩니다.
>-	데이터가 Amazon S3에서 암호화된 경우, 동일한 영역에 있어야 하며, 테이블을 만드는 사용자나 주체가 데이터를 복호화 할 수 있는 적절한 권한을 가지고 있어야 합니다. 자세한 내용은 [Configuring Encryption Options](http://docs.aws.amazon.com/athena/latest/ug/encryption.html) 를 참조하십시오.
>-	Athena는 LOCATION 조항에 의해 지정된 버킷 내에서 서로 다른 스토리지 클래스를 지원하지 않으며, GLACIER 스토리지 클래스를 지원하지 않으며, Requester Pays 버킷을 지원하지 않는다. 자세한 내용은 Amazon Simple Storage Service Developer Guide에서 [Storage Classes](http://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html), [Changing the Storage Class of an Object in Amazon S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/ChgStoClsOfObj.html) 와 [Requester Pays Buckets](http://docs.aws.amazon.com/AmazonS3/latest/dev/RequesterPaysBuckets.html) 를 참조하십시오.

1. 현재 지역이 **US West (Oregon)** 지역인지 확인하십시오.
2. 데이터베이스 목록에서 **mydatabase**가 선택되었는지 확인한 다음 **New Query**를 선택하십시오.
3. query 창에서 다음 문장을 복사하여 TaxiDataYellow 테이블을 생성한 다음 **Run Query**를 선택하십시오 :

````sql
    CREATE EXTERNAL TABLE IF NOT EXISTS TaxiDataYellow (
      VendorID STRING,
      tpep_pickup_datetime TIMESTAMP,
      tpep_dropoff_datetime TIMESTAMP,
      passenger_count INT,
      trip_distance DOUBLE,
      pickup_longitude DOUBLE,
      pickup_latitude DOUBLE,
      RatecodeID INT,
      store_and_fwd_flag STRING,
      dropoff_longitude DOUBLE,
      dropoff_latitude DOUBLE,
      payment_type INT,
      fare_amount DOUBLE,
      extra DOUBLE,
      mta_tax DOUBLE,
      tip_amount DOUBLE,
      tolls_amount DOUBLE,
      improvement_surcharge DOUBLE,
      total_amount DOUBLE
    )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    STORED AS TEXTFILE
    LOCATION 's3://us-west-2.serverless-analytics/NYC-Pub/yellow/'
````

>**Note:** 
>-	If you use CREATE TABLE without the EXTERNAL keyword, you will get an error as only tables with the EXTERNAL keyword can be created in Amazon Athena. We recommend that you always use the EXTERNAL keyword. When you drop a table, only the table metadata is removed and the data remains in Amazon S3.
>-	You can also query data in regions other than the region where you are running Amazon Athena. Standard inter-region data transfer rates for Amazon S3 apply in addition to standard Amazon Athena charges. 
>-	Ensure the table you just created appears on the Catalog dashboard for the selected database.

![athenatablecreatequery-yellowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenatablecreatequery-yellowtaxi.png)

## Querying data from Amazon S3 using Amazon Athena

Now that you have created the table, you can run queries on the data set and see the results in AWS Management Console for Amazon Athena.

1. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query**.

````sql
    SELECT * FROM TaxiDataYellow limit 10
````

Results for the above query look like the following:
![athenaselectquery-yellowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenaselectquery-yellowtaxi.png)

2.	Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to get the total number of taxi rides for yellow cabs. 

````sql
    SELECT COUNT(1) as TotalCount FROM TaxiDataYellow
````
Results for the above query look like the following:
![athenacountquery-yelllowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacountquery-yelllowtaxi.png)

>**Note:** 
The current data format is CSV and this query is scanning **~207GB** of data and takes **~20.06** seconds to execute the query.

3. Make a note of query execution time for later comparison while querying the data set in Apache Parquet format. 

4. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to query for the number of rides per vendor, along with the average fair amount for yellow taxi rides

````sql
    SELECT 
    CASE vendorid 
         WHEN '1' THEN 'Creative Mobile Technologies'
         WHEN '2' THEN 'VeriFone Inc'
         ELSE vendorid END AS Vendor,
    COUNT(1) as RideCount, 
    avg(total_amount) as AverageAmount
    FROM TaxiDataYellow
    WHERE total_amount > 0
    GROUP BY (1)
````
Results for the above query look like the following:
![athenacasequery-yelllowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacasequery-yelllowtaxi.png)

## Querying partitioned data using Amazon Athena

By partitioning your data, you can restrict the amount of data scanned by each query, thus improving performance and reducing cost. Athena leverages Hive for [partitioning](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterPartition) data. You can partition your data by any key. A common practice is to partition the data based on time, often leading to a multi-level partitioning scheme. For example, a customer who has data coming in every hour might decide to partition by year, month, date, and hour. Another customer, who has data coming from many different sources but loaded one time per day, may partition by a data source identifier and date.

### Create a Table with Partitions

1. Ensure that current AWS region is **US West (Oregon)** region

2. Ensure **mydatabase** is selected from the DATABASE list and then choose **New Query**.

3. In the query pane, copy the following statement to create a the NYTaxiRides table, and then choose **Run Query**:

````sql
  CREATE EXTERNAL TABLE NYTaxiRides (
    vendorid STRING,
    pickup_datetime TIMESTAMP,
    dropoff_datetime TIMESTAMP,
    ratecode INT,
    passenger_count INT,
    trip_distance DOUBLE,
    fare_amount DOUBLE,
    total_amount DOUBLE,
    payment_type INT
    )
  PARTITIONED BY (YEAR INT, MONTH INT, TYPE string)
  STORED AS PARQUET
  LOCATION 's3://us-west-2.serverless-analytics/canonical/NY-Pub'
````

4.Ensure the table you just created appears on the Catalog dashboard for the selected database.

![athenatablecreatequery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenatablecreatequery-nytaxi.png)

>**Note:**
>	Running the following sample query on the NYTaxiRides table you just created will not return any result as no metadata about the partition is added to the Amazon Athena table catalog.  
>```sql 
>   SELECT * FROM NYTaxiRides limit 10
>```

### Adding partition metadata to Amazon Athena

Now that you have created the table you need to add the partition metadata to the Amazon Athena Catalog.

1. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to add partition metadata.

```sql
    MSCK REPAIR TABLE NYTaxiRides
```
The returned result will contain information for the partitions that are added to NYTaxiRides for each taxi type (yellow, green, fhv) for every month for the year from 2009 to 2016

>**Note:**
> The MSCK REPAIR TABLE automatically adds partition data based on the New York taxi ride data to in the Amazon S3 bucket is because the data is already converted to Apache Parquet format partitioned by year, month and type, where type is the taxi type (yellow, green or fhv). If the data layout does not confirm with the requirements of MSCK REPAIR TABLE the alternate approach is to add each partition manually using ALTER TABLE ADD PARTITION. You can also automate adding partitions by using the JDBC driver.

### Querying partitioned data set

Now that you have added the partition metadata to the Athena data catalog you can now run your query.

1. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to get the total number of taxi rides

```sql
    SELECT count(1) as TotalCount from NYTaxiRides
```
Results for the above query look like the following:

![athenacountquery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacountquery-nytaxi.png)

>**Note:**
> This query executes much faster because the data set is partitioned and it in optimal format - Apache Parquet (an open source columnar). Following is a comparison of the execution time and amount of data scanned between the data formats:
>
>>**CSV Format:**
>>```sql
>>  SELECT count(*) as count FROM TaxiDataYellow 
>>```
>>Run time: **~20.06 seconds**, Data scanned: **~207.54GB**, Count: **1,310,911,060**
>>```sql
>>SELECT * FROM TaxiDataYellow limit 1000
>>```
>>Run time: **~3.13 seconds**, Data scanned: **~328.82MB**
>
>>**Parquet Format:**
>>```sql
>>SELECT count(*) as count FROM NYTaxiRides
>>```
>>Run time: **~5.76 seconds**, Data scanned: **0KB**, Count: **2,870,781,820**
>>```sql
>>SELECT * FROM NYTaxiRides limit 1000
>>```
>>Run time: **~1.13 seconds**, Data scanned: **5.2MB**


2. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to get the total number of taxi rides by year

```sql
    SELECT YEAR, count(1) as TotalCount from NYTaxiRides GROUP BY YEAR
```

Results for the above query look like the following:
![athenagroupbyyearquery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenagroupbyyearquery-nytaxi.png)

3. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to get the top 12 months by total number of rides across all the years

```sql
    SELECT YEAR, MONTH, COUNT(1) as TotalCount 
    FROM NYTaxiRides 
    GROUP BY (1), (2) 
    ORDER BY (3) DESC LIMIT 12
```
Results for the above query look like the following:

![athenacountbyyearquery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacountbyyearquery-nytaxi.png)

4. Choose **New Query**, copy the following statement into the query pane, and then choose **Run Query** to get the monthly ride counts per taxi time for the year 2016.

```sql
    SELECT MONTH, TYPE, COUNT(1) as TotalCount 
    FROM NYTaxiRides 
    WHERE YEAR = 2016 
    GROUP BY (1), (2)
    ORDER BY (1), (2)
```
Results for the above query look like the following:

![athenagroupbymonthtypequery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenagroupbymonthtypequery-nytaxi.png)

>**Note:**
Now the execution time is ~ 3 second, as the amount of data scanned by the query is restricted thus improving performance. This is because the data set is partitioned and it in optimal format – Apache Parquet, an open source columnar format.

5. Choose **New Query**, copy the following statement anywhere into the query pane, and then choose **Run Query**.

```sql
    SELECT MONTH,
      TYPE,
      avg(trip_distance) as  avgDistance,
      avg(total_amount/trip_distance) as avgCostPerMile,
      avg(total_amount) as avgCost, 
      approx_percentile(total_amount, .99) percentile99
    FROM NYTaxiRides
    WHERE YEAR = 2016 AND (TYPE = 'yellow' OR TYPE = 'green') AND trip_distance > 0 AND total_amount > 0
    GROUP BY MONTH, TYPE
    ORDER BY MONTH
```
Results for the above query look like the following:

![athenapercentilequery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenapercentilequery-nytaxi.png)


## Creating Views with Amazon Athena

A view in Amazon Athena is a logical, not a physical table. The query that defines a view runs each time the view is referenced in a query. You can create a view from a SELECT query and then reference this view in future queries. For more information, see [CREATE VIEW](https://docs.aws.amazon.com/athena/latest/ug/create-view.html).

1. Ensure that current AWS region is **US West (Oregon)** region

2. Ensure **mydatabase** is selected from the DATABASE list.

3. Choose **New Query**, copy the following statement anywhere into the query pane, and then choose **Run Query**.

```sql
CREATE VIEW nytaxiridesmonthly AS
SELECT 
    year,
    month,
    vendorid,
    avg(total_amount) as avg_Amt,
    sum (total_amount) as sum_Amt
FROM nytaxirides
where total_amount > 0
group by vendorid, year, month
```

You will see a new view called **nytaxiridesmonthly** created under **Views** under **Database** section in the left.

4. Choose **New Query**, copy the following statement anywhere into the query pane, and then choose **Run Query**.

```sql
SELECT * FROM nytaxiridesmonthly WHERE vendorid = '1'
```

Some of the view specific commands to try out are [SHOW COLUMNS](https://docs.aws.amazon.com/athena/latest/ug/show-columns.html), [SHOW CREATE VIEW](https://docs.aws.amazon.com/athena/latest/ug/show-create-view.html), [DESCRIBE VIEW](https://docs.aws.amazon.com/athena/latest/ug/describe-view.html), and [DROP VIEW](https://docs.aws.amazon.com/athena/latest/ug/drop-view.html).

## CTAS Query with Amazon Athena

A CREATE TABLE AS SELECT (CTAS) query creates a new table in Athena from the results of a SELECT statement from another query. Athena stores data files created by the CTAS statement in a specified location in Amazon S3. For syntax, see [CREATE TABLE AS](https://docs.aws.amazon.com/athena/latest/ug/create-table-as.html).

Use CTAS queries to:

Create tables from query results in one step, without repeatedly querying raw data sets. This makes it easier to work with raw data sets.
Transform query results into other storage formats, such as Parquet and ORC. This improves query performance and reduces query costs in Athena. For information, see [Columnar Storage Formats](https://docs.aws.amazon.com/athena/latest/ug/columnar-storage.html).
Create copies of existing tables that contain only the data you need.

### Create an Amazon S3 Bucket

1. Open the [AWS Management console for Amazon S3](https://s3.console.aws.amazon.com/s3/home?region=us-west-2)
2. On the S3 Dashboard, Click on **Create Bucket**. 

![createbucket.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucket.png)

3. In the **Create Bucket** pop-up page, input a unique **Bucket name**. So it’s advised to choose a large bucket name, with many random characters and numbers (no spaces). 

    1. Select the region as **Oregon**. 
    2. Click **Next** to navigate to next tab. 
    3. In the **Set properties** tab, leave all options as default. 
    4. In the **Set permissions** tag, leave all options as default.
    5. In the **Review** tab, click on **Create Bucket**

![createbucketpopup.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucketpopup.png)

### Repartitioning the dataset using CTAS Query 

1. Ensure that current AWS region is **US West (Oregon)** region

2. Ensure **mydatabase** is selected from the DATABASE list.

3. Choose **New Query**, copy the following statement anywhere into the query pane, and then choose **Run Query**.

```sql
CREATE TABLE ctas_nytaxride_partitioned 
WITH (
     format = 'PARQUET', 
     external_location = 's3://<name-of-the-bucket-your-created>/ctas_nytaxride_partitioned/', 
     partitioned_by = ARRAY['month','type','vendorid']
     ) 
AS select 
    ratecode, passenger_count, trip_distance, fare_amount, total_amount, month, type, vendorid
FROM nytaxirides where year = 2016 and (vendorid = '1' or vendorid = '2')
```

Go the Amazon S3 bucket specified as the external location and inspect the format and key structure in which the new objects are written in.

>**Note:**
> Please delete the Amazon S3 location specified as the external location before retrying the query. Donot delete the Amazon S3 bucket.

### Repartitioning and Bucketing the dataset using CTAS Query 

4. Choose **New Query**, copy the following statement anywhere into the query pane, and then choose **Run Query**.

```sql
CREATE TABLE ctas_nytaxride_bucketed_partitioned 
WITH (
     format = 'PARQUET', 
     external_location = 's3://<name-of-the-bucket-your-created>/ctas_nytaxride_bucketed/', 
     partitioned_by = ARRAY['month', 'type'],
     bucketed_by = ARRAY['vendorid'],
     bucket_count = 3) 
AS select 
    ratecode, passenger_count, trip_distance, fare_amount, total_amount,vendorid, month, type
FROM nytaxirides where year = 2016 
```

>**Note:**
> This query will take approximately 6 minutes.

Go the Amazon S3 bucket specified as the external location and inspect the format and key structure in which the new objects are written in.

>**Note:**
> Please delete the Amazon S3 location specified as the external location before retrying the query. Donot delete the Amazon S3 bucket.

Please refer to [Partitioning Vs. Bucketing](https://docs.aws.amazon.com/athena/latest/ug/bucketing-vs-partitioning.html) for more details.

---

## License

This library is licensed under the Apache 2.0 License. 
