# Lab 1: Serverless Analysis of data in Amazon S3 using Amazon Athena

* [Amazon Athena 데이터 베이스 및 테이블 생성](#creating-amazon-athena-database-and-table)
    * [Athena 데이터 베이스 생성하기](#데이터-베이스-생성하기)
    * [Athena 테이블 생성](#테이블-생성하기)  
* [Athena를 사용해서 S3에서 데이터 Querying하기](#Athena를-사용해서-S3에서-데이터-Querying하기)
* [Amazon Athena를 사용하여 분한될 데이터 쿼리하기](#Amazon-Athena를-사용하여-분한될-데이터-쿼리하기)
    * [파티션으로 테이블 만들기](#파티션으로-테이블-만들기)
    * [Amazon athena에 파티션 메타 데이터 추가하기](#Amazon-athena에-파티션-메타-데이터-추가하기)
    * [분할 된 데이터 집합 쿼리하기](#분할-된-데이터-집합-쿼리하기)
* [Amazon  Athena로 View 생성하기](#Amazon--Athena로-View-생성하기)
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
>-	Amazon Athena에서 EXTERNAL 키워드가 있는 테이블만 만들수 있기 때문에 EXTERNAL 키워드 없이 CREATE TABLE을 사용하면 오류가 발생합니다. 항상 EXTERNAL 키워드를 사용하는 것이 좋습니다. 테이블을 drop 하면 테이블 메타데이터만 제거되고 데이터는 Amazon S3에 남아있습니다.
>-	또한 Amazon Athena를 실행하는 지역 이외의 지역에서 데이터를 query할 수 도 있습니다. Amazon S3 표준 지역간 데이터 전송 속도는 표준 Athena 요금과 함께 적용됩니다.
>-	선택한 데이터베이스에 대한 카탈로그 대시보드에 방금 생성한 테이블이 나타나는지 확인하십시오.

![athenatablecreatequery-yellowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenatablecreatequery-yellowtaxi.png)

## Athena를 사용해서 S3에서 데이터 Querying하기

이제 생성된 테이블이 있습니다. 데이터 세트에서 query를 실행할 수 있고 그 결과를 AWS Management Console for Athena 에서 볼 수 있습니다.

1. **New Query** 를 선택하고 다음 문장을 query 창에 복사한 다음 **Run Query** 를 선택하십시오.

````sql
    SELECT * FROM TaxiDataYellow limit 10
````

위의 query에 대한 결과는 다음과 같습니다 :
![athenaselectquery-yellowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenaselectquery-yellowtaxi.png)

2.	**New Query** 를 선택하고 다음 문장을 query 창에 복사한 다음 **Run Query**를 선택하여 노란 택시를 위한 total  number of taxi  rides를 가져오십시오.

````sql
    SELECT COUNT(1) as TotalCount FROM TaxiDataYellow
````
위의 query에 대한 결과는 다음과 같습니다 :
![athenacountquery-yelllowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacountquery-yelllowtaxi.png)

>**Note:** 
현재 데이터 형식은 CSV 이며 이 query는 **~207GB**의 데이터를 스캔하고 있으며 query를 실행하는데 최대  **20.06**초가 소요됩니다.

3. Apache Paraquet 형식으로 데이터 세트를 querying 하는 동안 나중 비교를 위해 query 실행 시간을 기록하십시오.
4. **New Query**를 선택하고 다음 문장을 query 창에 복사한 다음 **Run Query**를 선택하여 노란 택시 승차권의 평균 요금과 함께 회사 당  승차 수를 query 하십시오.

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
위의 query에 대한 결과는 다음과 같습니다 :
![athenacasequery-yelllowtaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacasequery-yelllowtaxi.png)

## Amazon Athena를 사용하여 분한될 데이터 쿼리하기

데이터를 분할시킴으로써 각 쿼리가 스캔하는 데이터의 양을 제한할 수 있게 되고, 그것으로 인해 성능을 향상 시키고 비용을 절감 할 수 있습니다. Athena는 Hive를 활용하여 데이터를 [파티셔닝](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterPartition) 합니다. 임의의 키로 데이터를 분할 할 수 있습니다. 일반적으로 시간을 기준으로 데이터를 분할하여 여러 수준(multi-level)의 파티셔닝 스키마를 생성하는 경우가 많습니다. 예를 들면, 매시간마다 데이터가 들어오는 고객은 년, 월, 일 및 시간별로 파티션을 결정할 수 있습니다. 여러 다른 소스에서 데이터를 전송하지만 하루에 한번만 로드되는 다른 고객의 경우에어는 데이터 소스 식별자 및 날짜별로 분할 할 수 있습니다.

### 파티션으로 테이블 만들기

1. 현재 AWS지역이  **US West (Oregon)**  지역 인지 확인하십시오.
2. 데이터베이스 목록에서 **mydatabase**가 선택 되었는지 확인후 **New Query**를 선택하십시오.
3. 쿼리 창에서 다음 문장들을 복사하여 NYTaxiRides 테이블을 만든 다음 **Run Query**를 선택합니다.

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

4. 방금 생성 한 테이블이 선택된 데이터베이스의 카탈로그 대시 보드에 표시되는지 확인합니다.

![athenatablecreatequery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenatablecreatequery-nytaxi.png)

>**Note:** 방금 만든 NYTaxiRides 테이블에서 다음 샘플 쿼리를 실행해도 파티션에 대한 메타 데이터가 Amazon Athena 테이블 카탈로그에 추가되지 않으므로 결과를 반환하지 않습니다.
>
>```sql 
>SELECT * FROM NYTaxiRides limit 10
>```

### Amazon athena에 파티션 메타 데이터 추가하기

이제 파티션 메타 데이터를 Amazon Athena Catalog에 추가해야하는 테이블을 만들었습니다.

1. **New Query** 를 선택하고 다음 문장을 쿼리 창에 복사 한 다음 **Run Query**를 선택 하여 파티션 메타 데이터를 추가합니다.

```sql
    MSCK REPAIR TABLE NYTaxiRides
```
반환 된 결과에는 2009년 부터 2016년 까지 매달 택시 type (노란색, 녹색, fhv) 별로 NYTaxiRides에 추가 된 파티션에 대한 정보가 포함됩니다.

> **Note:** MSCK REPAIR TABLE은 뉴욕 택시 승차 데이터를 기반으로 한 데이터를 Amazon S3 버킷에 자동으로 추가하는 것은 데이터가 이미 년도, 월 및 type별로 분할된 Apache Parquet 형식으로 변환되었기 때문이며, 여기서 type은 택시의 type(노란색, 녹색, fhv) 입니다.  만일 데이터 레이아웃이 MSCK REPAIR TABLE의 요구 사항으로 확인되지 않다면 대체 방법은 ALTER TABLE ADD PARTITION 을 사용하여 각 파티션을 수동으로 추가하는 것입니다. JDBC 드라이버를 사용하여 파티션 추가를 자동화 할 수도 있습니다.

### 분할 된 데이터 집합 쿼리하기

Athena 데이터 카탈로그에 파티션 메타 데이터를 추가 했으므로 이제 쿼리를 실행할 수 있습니다.

1. **New Query** 를 선택 하고 다음 문장을 쿼리 창에 복사한 다음 **Run Query** 를 선택하여 총 택시 수를 가져옵니다.

```sql
    SELECT count(1) as TotalCount from NYTaxiRides
```
위 쿼리의 결과는 다음과 같습니다.

![athenacountquery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacountquery-nytaxi.png)

> **Note:** 이 쿼리는 데이터 세트가 분할되어 있으며 Apache Parquet (오픈 소스 컬럼) 인 최적의 형식으로 훨씬 빠르게 실행됩니다. 다음은 실행 시간과 형식간에 스캔 된 데이터의 양을 비교 한 것입니다.
>
> > **CSV 형식:**
> >
> > ```sql
> > SELECT count(*) as count FROM TaxiDataYellow 
> > ```
> > 실행 시간: **~20.06 초**, 데이터 스캔: **~207,54GB**, 카운트: **1,310,911,060**
> > ````sql
> > SELECT * FROM TaxiDataYellow limit 1000
> > ````
> > 실행 시간: **~3.13 초**, 데이터 스캔: **~328.82MB**
> 
> > **Parquet 형식:**
> > ````sql
> > SELECT count(*) as count FROM NYTaxiRides
> > ````
> > 실행 시간: **~5.76 초**, 데이터 스캔: **0KB**, 카운트: **2,870,781,820**
> > ````sql
> > SELECT * FROM NYTaxiRides limit 1000
> > ````
> > 실행 시간: **~1.13 초**, 데이터 스캔: **~5.2MB**

2. **New Query**를 선택하고 다음 문장을 쿼리 창에 복사한 다음 **Run Query**를 선택하여 총 택시 수를 얻습니다.

```sql
    SELECT YEAR, count(1) as TotalCount from NYTaxiRides GROUP BY YEAR
```
위 쿼리의 결과는 다음과 같습니다.
![athenagroupbyyearquery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenagroupbyyearquery-nytaxi.png)

3. **New Query**를 선택하고 다음 문장을 쿼리 창에 복사한 다음 **Run Query**를 선택 하면 모든 년도를 통틀어 승차수가 가장 높은 12달의 결과를 얻을 수 있습니다.

```sql
    SELECT YEAR, MONTH, COUNT(1) as TotalCount 
    FROM NYTaxiRides 
    GROUP BY (1), (2) 
    ORDER BY (3) DESC LIMIT 12
```
위 쿼리의 결과는 다음과 같습니다.
![athenacountbyyearquery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacountbyyearquery-nytaxi.png)

4. **New Query**를 선택하고 다음 문장을 쿼리 창에 복사한 다음 **Run Query**를 선택하여 2016년 택시 당 월간 승차수를 계산 합니다.

```sql
    SELECT MONTH, TYPE, COUNT(1) as TotalCount 
    FROM NYTaxiRides 
    WHERE YEAR = 2016 
    GROUP BY (1), (2)
    ORDER BY (1), (2)
```
위 쿼리의 결과는 다음과 같습니다.
![athenagroupbymonthtypequery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenagroupbymonthtypequery-nytaxi.png)

>**Note:** 이제 쿼리에 의해 검색되는 데이터의 양이 제한됨으로써 성능이 향상되기 때문에 실행시간은 ~3초 입니다. 이는 데이터 세트가 최적의 형식으로 분할되어 있기 때문입니다. Apache Parquet은 오픈 소스 컬럼 형식입니다.

5. **New Query**를 선택하고 다음 문장을 쿼리 창에 복사한 다음 **Run Query**를 선택합니다.

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
위 쿼리의 결과는 다음과 같습니다.
![athenapercentilequery-nytaxi.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenapercentilequery-nytaxi.png)


## Amazon  Athena로 View 생성하기

Amazon Athena의 view는 물리적 테이블이 아닌 논리적인 테이블 입니다.  view를 정의하는 query는 view가 참조될 때 마다 실행됩니다. SELCET query 에서 view를 생성한 다음 이후 query에서 이 view를 참조하십시오. 자세한 내용은 [CREATE VIEW](https://docs.aws.amazon.com/athena/latest/ug/create-view.html)를 참조하십시오.

1. 현재 AWS 지역이 **US West (Oregon)** 지역인지 확인하십시오.
2. 데이터베이스 목록에서 **mydatabase**가 선택되었는지 확인하십시오.
3. **New Query**를 선택하고 다음 문장을 query 창에 복사한 다음 **Run Query**를 선택하십시오.

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

왼쪽은 **Database** 섹션에 있는 **Views**에서 **nytaxiridesmonthly** 라는 view가 생성된 것을 볼 수 있습니다.

4. **New Query**를 선택하고 다음 문장을 query 창에 복사한 다음 **Run Query**를 선택하십시오.

```sql
SELECT * FROM nytaxiridesmonthly WHERE vendorid = '1'
```

시도할 view 명령어로는  [SHOW COLUMNS](https://docs.aws.amazon.com/athena/latest/ug/show-columns.html), [SHOW CREATE VIEW](https://docs.aws.amazon.com/athena/latest/ug/show-create-view.html), [DESCRIBE VIEW](https://docs.aws.amazon.com/athena/latest/ug/describe-view.html), 그리고 [DROP VIEW](https://docs.aws.amazon.com/athena/latest/ug/drop-view.html) 가 있습니다.

## Amazon Athena의 CTAS 쿼리

A CREATE TABLE AS SELECT(CTAS) 쿼리는 다른 쿼리의 SELECT 문의 결과를 Athena에 새로운 테이블로 만들 수 있습니다. Altena는 CTAS문으로 생성된 데이터 파일들을 Amazon S3의 특정 위치에 저장합니다. 문법을 보려면 [CREATE TABLE AS](https://docs.aws.amazon.com/athena/latest/ug/create-table-as.html) 를 참조하십시오.

CTAS 쿼리의 기능:

raw data set에 반복적인 쿼리 없이 한 번에 쿼리 결과를 사용하여 테이블을 생성합니다. 이렇게 하면 raw data set으로 작업하기가 더 쉬워집니다.
쿼리 결과를 Parquet 및 ORC 같은 다른 저장 형식으로 변환합니다. 이는 쿼리의 성능을 높혀주고 Athena에서 쿼리의 비용을 줄여줍니다. 자세한 정보는 [Columnar Storage Formats](https://docs.aws.amazon.com/athena/latest/ug/columnar-storage.html).를 참조하십시오.
필요한 데이터만 포함하는 기존 테이블의 복사본을 생성합니다.

### Amazon S3 Bucket 생성

1. [AWS Management console for Amazon S3](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) 열기
2. S3 대쉬보드에 있는 **Create bucket**을 클릭

![createbucket.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucket.png)



3. **버킷만들기** 팝업 페이지에서 고유한 **버킷 이름**을 입력합니다. 고유한 버킷 이름을 사용해야 하기 때문에 비컷의 이름으로 많은 랜덤한 문자와 숫자를 공백없이 사용할것을 추천드립니다.
   1. 리전을 미국 서부(오레곤)으로 선택
   2. 다음 탭으로 이동하려면 **Next** 클릭
   3. **옵션 구성** 탭에서 모든 옵션을 기본값으로 설정
   4. **권한 설정** 탭에서 모든 옵션을 기본값으로 설정
   5. **검토** 탭에서 **버킷 만들기** 클릭

![createbucketpopup.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucketpopup.png)

### Repartitioning the dataset using CTAS Query 

### CTAS 쿼리를 사용하여 데이터셋 Repatitioning

1. 현재 AWS 리전이 **미국 서부(오레곤)** 리전인지 확인합니다.
2. 데이터베이스 리스트에서 **mydatabase**가 선택되어있는지 확인합니다.
3. **New Query**를 선택하고 쿼리 창 어디에나 다음 명령문을 붙여넣은 다음에 **Run Query**를 선택합니다. 

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

외부 저장소로 지정된 Amazon S3 버킷으로 이동하여 새 객체가 기록된 형식 및 키 구조를 검사합니다.

> **Note:**
> 쿼리를 다시 시도하기 전에 외부 저장소로 지정된 Amazon S3 저장소를 삭제해야합니다. Amazon S3 버킷을 삭제하면 안됩니다.

### Repartitioning and Bucketing the dataset using CTAS Query 

### CTAS 쿼리를 이용하여 데이터셋 Repatitioning 및 Bucketing

4. **New Query**를 선택하고 쿼리 창 어디에나 다음 명령문을 붙여넣은 다음에 **Run Query**를 선택합니다. 

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

> **Note:**
> 이 쿼리는 약 6분이 소요됩니다.

외부 저장소로 지정된 Amazon S3 버킷으로 이동하여 새 객체가 기록된 형식 및 키 구조를 검사합니다.

> **Note:**
> 쿼리를 다시 시도하기 전에 외부 저장소로 지정된 Amazon S3 저장소를 삭제해야합니다. Amazon S3 버킷을 삭제하면 안됩니다.

자세한 정보는 [Partitioning Vs. Bucketing](https://docs.aws.amazon.com/athena/latest/ug/bucketing-vs-partitioning.html) 을 참조하십시오.

------

## License

This library is licensed under the Apache 2.0 License. 