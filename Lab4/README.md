# Lab 4: Analysis of data in Amazon S3 using Amazon Redshift Spectrum

* [Deploying Amazon Redshift Cluster](#deploying-amazon-redshift-cluster)
* [AWS Glue Crawlers 실행하기 - CSV 및 Parquet 크롤러](#AWS-Glue-Crawlers-실행하기---CSV-및-Parquet-크롤러)
* [AWS Glue 데이터 카탈로그 데이터베이스를 구성하는 Redshift Spectrum Scehma 및 참조 외부 테이블 생성하기](#AWS-Glue-데이터-카탈로그-데이터베이스를-구성하는-Redshift-Spectrum-Scehma-및-참조-외부-테이블-생성하기)
* [Querying data from Amazon S3 using Amazon Redshift Spectrum](#querying-data-from-amazon-s3-using-amazon-redshift-spectrum)
* [Querying partitioned data using Amazon Redshift Spectrum](#querying-partitioned-data-using-amazon-redshift-spectrum)


## Architectural Diagram
![architecture-overview-lab4.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-17+at+1.11.45+AM.png)

## Deploying Amazon Redshift Cluster 

In this section you will use the CloudFromation template to create Amazon RedShift cluster resources. The template will also install [pgweb](https://github.com/sosedoff/pgweb), SQL Client for PostgreSQL, in an  Amazon EC2 instance to connect and run your queries on the launched Amazon Redshift cluster. Alternatively, you can connect to the Amazon Redshift cluster using standard SQL Clients such as SQL Workbench/J. For more information refer http://docs.aws.amazon.com/redshift/latest/mgmt/connecting-using-workbench.html.

1. Login in to your AWS console and open the [Amazon CloudFormation Dashboard](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2]) 
2. Make a note of the AWS region name, for example, for this lab you will need to choose the **US West (Oregon)** region.
3. Click **Create Stack**
4. Select **Specify an Amazon S3 template URL**
5. Copy paste the following S3 template URL in the text box
```
https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/us-west-2.serverless-data-analytics/labcontent/redshiftspectrumglue-lab4.template
```
6. Click **Next**

>**Note:** 
>Click on the link [redshiftspectrumglue-lab4.template](../Lab4/redshiftspectrumglue-lab4.template) to view the Amazon CloudFormation template file


![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.38.08+PM.png)

8. Type a name *(e.g. RedshiftSpectrumLab)* for the **Stack Name**

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.38.39+PM.png)

9. Enter the following **Parameters** for **Redshift Cluster Configuration**
   
    1. Choose *multi-node* for **ClusterType**
    2. Type *2* for the **NumberOfNodes**
    3. For **NodeType** select *dc1.xlarge*

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.38.57+PM.png)

10.  Enter the following **Parameters** for **Redshift Database Configuration**.
    i. Type a name (e.g. dbadmin) for **MasterUserName**.
    ii. Type a password for **MasterUserPassword**.
    iii. Type the a name (e.g. taxidb) for **DatabaseName**.
    iv. Type the IP address of your local machine for **ClientIP**.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.39.23+PM.png)

11. Enter the following **Parameters** for **Glue Crawler Configuration**
    1. Type the name(e.g. taxi-spectrum-db) for **GlueCatalogDBName**.    
    2. Type the name(e.g. csvCrawler) for **CSVCrawler**.
    3. Type the name(e.g. parquetCrawler) for **ParquetCrawler**.
    
12. Click **Next**

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.40.04+PM.png)

13. [Optional] In the **Tags** sub-sections in **Options** type a **Key** name *(e.g. Name)* and **Value** for key.
14. Click **Next**

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.40.31+PM.png)

15. Check **I acknowledge that AWS CloudFormation might create IAM resources.**
16. Click **Create**

> **Note:** This is may take approximately 15 minutes 

17. Ensure that status of the Amazon CloudFromation stack that you just create is **CREATE_COMPLETE**
18. Select your Amazon CloudFormation stack *(RedshiftSpectrumLab)*
19. Click on the **Outputs** tab
20. Review the list of **Key** and thier **Value** which will look like the following. 

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.30.42+PM.png)

## AWS Glue Crawlers 실행하기 - CSV 및 Parquet 크롤러
1. [AWS Management Console for Glue](https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2#)를 여십시오.
2. 탐색 창에서 **Crawlers**를 클릭하여 AWS Glues Crawlers 페이지로 이동하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/Screen+Shot+2017-11-17+at+3.02.35+AM.png)

3. CSV 용 AWS Glue Crawler를 선택하십시오 (예 : csvCrawler).
4. **Run crawler**를 클릭하십시오.
5. CSV 용 AWS Glue Crawler를 선택하십시오 (예 : csvCrawler).
6. **Run crawler**를 클릭하십시오.

> Note : 두크롤러가 모두 CSV 및 Parquet 형식의 데이터를 구문 부속하는 데 약 5분이 걸릴 수 있습니다.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+11.08.23+PM.png)

7. 두 크롤러의 **Status**가 *Ready*가 될 때 까지 기다리십시오.

이제 크롤러를 실행하여 새테이블 *taxi* 와 *ny_pub* 가 생성되지 않도록 하십시오.

8. AWS Glue Data Catalog 에 있는 데이터베이스 몰고에서 탐색 창에 있는 **Databases**를 클릭하십시오.
9. **taxi-spectrum-db**를 클릭하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+11.09.32+PM.png)

10. **Tables in taxi-spectrum-db**를 클릭하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+11.09.50+PM.png)

11. **taxi**를 클릭하여 테이블 정의 및 스키마 컴토를 하십시오.
12. 뒤로 이동하여 **ny_hub**을 클릭하고 테이블 정의 및 스키마 검토를 하십시오.

>**Note :**  방금 살펴본 CSV 문서를 읽을 새 테이블이나 정의를 만들 필요가 없습니다. AWS Glue 크롤러를 사용하면 미리 스키마를 추론하고 taxi 및 ny_pub이라는 테이블을 생성합니다.

13. **View partitions**를 클릭하여 파티션 메타 데이터를 검토 하십시오.

>**Note :** Glue Crawlers의 가장 큰 장점은 S3 객체 prefix를 기반 파티션을 이해하고 크롤링의 일부로 파티션이 있는 테이블을 자동으로 생성한다는 것 입니다.


## AWS Glue 데이터 카탈로그 데이터베이스를 구성하는 Redshift Spectrum Scehma 및 참조 외부 테이블 생성하기

1. [Amazon CloudFormation Dashboard](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2]) 를 여십시오.
2. 이 AWS 지역을 **US West (Oregin)** 지역으로 선택하십시오.
3. Amazon CloudFormation 스택 선택을 하십시오.(RedshiftSpectrumLab)
4. **Outputs** 탭을 클릭하십시오.
5. **pgWeb**의 URL을 읽으십시오.
6. pgWeb 콘솔에서 **SQL Query** 탭이 선택 되었는지 확인 하십시오.
7. 다음 문장을 복사하여 Redshift Spectrum에서 데이터 베이스 *(예. taxispectrum)*를 만드십시오.

```sql
  create external schema taxispectrum from data catalog
  database 'taxi-spectrum-db' 
  iam_role '<specify the redshift IAM Role arn from the CloudFormation outputs section>'
```
8. 문장에 있는 *<specify the redshift IAM Role arn from the CloudFormation output section'>*를 앞선 실험에서 만들었던 Amazon CloudFormation 스택*(RedshiftSpectrumLab)*에 있는  **Outputs** 탭의 **redshiftIAMRole**의 값으로 바꾸십시오.

> **Note:** IAM 역할은 작은 따움표로 묶여야 합니다.

9. **Run Query**를 클릭하십시오.
> **Note:**  Amazon Redshift, AWS Glue, Amazon Athena 또는 Apache Hive 메타 스토어에서 외부 테이블을 만들 수 있습니다. 자세한 내용은 AWS Glue Develpoer Guide의 [Getting Started Using AWS Glue](http://docs.aws.amazon.com/glue/latest/dg/getting-started.html) , Amazon Athena User Guide의 [Getting Started](http://docs.aws.amazon.com/athena/latest/ug/getting-started.html) 또는 Amazon Athena User Guide의 [Apache Hive](http://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hive.html)를 참조하십시오. 외부 테이블이 AWS Glue, Athena 또는 Hive 메타 스토어 로 정의 되어 있다면 먼저 외부 데이터베이스를 참조하는 외부 스키마를 만듭니다. 그런 다음 Amazon Redshift에서 테이블을 생성할 필요 없이 테이블 이름 앞에 스키마 이름을 붙임으로써 SELECT 문에서 외부 테이블을 참조 할 수 있습니더. 자세한 내용은 [Creating External Schemas for Amazon Redshift Spectrum](http://docs.aws.amazon.com/redshift/latest/dg/c-spectrum-external-schemas.html.)를 참조하십시오.

## Querying data from Amazon S3 using Amazon Redshift Spectrum

Now that you have created the schema, you can run queries on the data set and see the results in PGWeb Console.

1. Copy the following statement into the query pane, and then choose **Run Query**.

```sql
    SELECT * FROM taxispectrum.taxi limit 10
```

Results for the above query look like the following:

![Screen Shot 2017-11-14 at 9.16.45 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+9.16.45+PM.png)

2.	Copy the following statement into the query pane, and then choose **Run Query** to get the total number of taxi rides for yellow cabs. 

```sql
    SELECT COUNT(1) as TotalCount FROM taxispectrum.taxi
```
Results for the above query look like the following:

![Screen Shot 2017-11-14 at 9.25.23 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+9.25.23+PM.png)

3. Copy the following statement into the query pane, and then choose **Run Query** to query for the number of rides per vendor, along with the average fair amount for yellow taxi rides

```sql
    SELECT 
    CASE vendorid 
         WHEN '1' THEN 'Creative Mobile Technologies'
         WHEN '2' THEN 'VeriFone Inc'
         ELSE CAST(vendorid as VARCHAR) END AS Vendor,
    COUNT(1) as RideCount, 
    avg(total_amount) as AverageAmount
    FROM taxispectrum.taxi
    WHERE total_amount > 0
    GROUP BY (1)
```

Results for the above query look like the following:

![Screen Shot 2017-11-14 at 9.46.55 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+9.46.55+PM.png)

## Querying partitioned data using Amazon Redshift Spectrum

By partitioning your data, you can restrict the amount of data scanned by each query, thus improving performance and reducing cost. Amazon Redshift Spectrum leverages Hive for [partitioning](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterPartition) data. You can partition your data by any key. A common practice is to partition the data based on time, often leading to a multi-level partitioning scheme. For example, a customer who has data coming in every hour might decide to partition by year, month, date, and hour. Another customer, who has data coming from many different sources but loaded one time per day, may partition by a data source identifier and date.


Now that you have added the partition metadata to the Athena data catalog you can now run your query.

1. Copy the following statement into the query pane, and then choose **Run Query** to get the total number of taxi rides

```sql
    SELECT count(1) as TotalCount from taxispectrum.ny_pub
```
Results for the above query look like the following:

![Screen Shot 2017-11-14 at 10.08.50 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.08.50+PM.png)

>**Note:**
> This query executes much faster because the data set is partitioned and it in optimal format - Apache Parquet (an open source columnar).

2. Copy the following statement into the query pane, and then choose **Run Query** to get the total number of taxi rides by year

```sql
    SELECT YEAR, count(1) as TotalCount from taxispectrum.ny_pub GROUP BY YEAR
```
Results for the above query look like the following:
![Screen Shot 2017-11-14 at 10.11.47 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.11.47+PM.png)

3. Copy the following statement into the query pane, and then choose **Run Query** to get the top 12 months by total number of rides across all the years

```sql
    SELECT YEAR, MONTH, COUNT(1) as TotalCount 
    FROM taxispectrum.ny_pub
    GROUP BY (1), (2) 
    ORDER BY (3) DESC LIMIT 12
```
Results for the above query look like the following:
![Screen Shot 2017-11-14 at 10.13.54 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.13.54+PM.png)

4. Copy the following statement into the query pane, and then choose **Run Query** to get the monthly ride counts per taxi time for the year 2016.

```sql
    SELECT MONTH, TYPE, COUNT(1) as TotalCount 
    FROM taxispectrum.ny_pub
    WHERE YEAR = 2016 
    GROUP BY (1), (2)
    ORDER BY (1), (2)
```
Results for the above query look like the following:
![Screen Shot 2017-11-14 at 10.18.08 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.18.08+PM.png)

5. Copy the following statement anywhere into the query pane, and then choose **Run Query**.

```sql
    SELECT MONTH, TYPE,
      avg(trip_distance) avgDistance,
      avg(total_amount/trip_distance) avgCostPerMile,
      avg(total_amount) avgCost,
      percentile_cont(0.99)
      within group (order by total_amount)
    FROM taxispectrum.ny_pub
    WHERE YEAR = 2016 AND (TYPE = 'yellow' OR TYPE = 'green')
    AND trip_distance > 0 AND total_amount > 0
    GROUP BY MONTH, TYPE
    ORDER BY MONTH
```

Results for the above query look like the following:

![Screen Shot 2017-11-14 at 10.23.51 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.23.51+PM.png)

## Deleting the Amazon CloudFormation Stack

Now that you have successfully queried the dataset using Amazon Redshift Spectrum, you need to tear down the stack that you deployed using the Amazon CloudFormation template.

1. Open the [Amazon CloudFormation Dashboard](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2) 
2. Enable the check box next to the name of the stack *(e.g. RedshiftSpectrumLab)* that you deployed at the beginingo fo the Lab. 
3. Click on **Actions** drop down button.
4. Select **Delete Stack**'
5. Click **Yes, Delete** on the *Delete Stack* pop dialog
6. Ensure that Amazon CloudFromation stack name *(e.g. RedshiftSpectrumLab)* is no longer showing in the list of stacks.

---
## License

This library is licensed under the Apache 2.0 License. 