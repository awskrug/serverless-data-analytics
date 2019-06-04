# Lab 2:  Amazon QuickSight를 이용한 시각화

* [Create an Amazon S3 bucket](#create-an-amazon-s3-bucket)
* [Creating Amazon Athena Database and Table](#creating-amazon-athena-database-and-table)
    * [Create Athena Database](#create-database)
    * [Create Athena Table](#create-a-table)
* [Signing up for Amazon Quicksight Standard Edition](#signing-up-for-amazon-quicksight-standard-edition)
* [Configuring Amazon QuickSight to use Amazon Athena as data source](#configuring-amazon-quicksight-to-use-amazon-athena-as-data-source)
* [Amazon QuickSight를 사용하여 데이터 시각화](#amazon-quicksight를-사용하여-데이터-시각화)
  
    * [연도 기반 필터를 추가하여 2016년 데이터 셋을 시각화](#연도-기반-필터를-추가하여-2016년-데이터-셋을-시각화)
    * [1월을 위한 월 기준 필터 추가](#1월을-위한-월-기준-필터-추가)
    * [2016년 1월 한 달 동안의 시간 데이터를 시각화](#2016년-1월-한-달-동안의-시간-데이터를-시각화)
    * [모든 택시 타입(yellow, green, fhv)에 대한 2016년 1월 한 달동안의 데이터를 시각화](#모든-택시-타입(yellow,-green,-fhv)에-대한-2016년-1월-한-달동안의-데이터를-시각화)
    
      

## Architectural Diagram
![architecture-overview-lab2.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/architecture-overview-lab2.png)

## Amazon S3 bucket 생성하기

> Note: AWS 계정에 S3 버킷이 이미 있는 경우 이 섹션을 생략할 수 있습니다.

1. [AWS Management console for Amazon S3](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) 을 여십시오.
2. S3 대시보드에서 **Create Bucket** 을 클릭하십시오.

![createbucket.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucket.png)

3. **Create Bucket** 팝업 페이지에서 고유한 **Bucket name** 을 입력하십시오. 고유한 이름을 위해 많은 임의의 문자와 숫자(공백 없는)가 있는 큰 버킷 이름을 선택하는 것이 좋습니다.
1. 지역을 **Oregon** 으로 선택하십시오. 
    2. 다음 탭으로 이동하려면 **Next** 을 클릭하십시오.
    3. **Set properties** 탭에서 모든 옵션을 기본옵션으로 유지하십시오.
    4. **Set permissions** 태그에서 모든 옵션을 기본값으로 유지하십시오.
    5. **Review** 탭에서 **Create Bucket** 를 클릭하십시오.

![createbucketpopup.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucketpopup.png)

## Amazon Athena Database and Table 생성하기

> Note: [Lab 1: Serverless Analysis of data in Amazon S3 using Amazon Athena](../Lab1) 을 완료한 경우 이 섹션을 생략하고 다음 섹션 [Signing up for Amazon Quicksight Standard Edition](#signing-up-for-amazon-quicksight-standard-edition) 으로 넘어가십시오.

Amazon Athena는 Apache Hive를 사용하여 테이블을 정의하고 데이터베이스를 생성합니다. 데이터베이스는 테이블의 논리적 그룹입니다. Amazon S3에 있는 테이블 데이터의 스키마와 위치를 기술하는 것으로 Athena에서 데이터베이스와 테이블을 작성할 수 있습니다. Hive의 경우, 기존의 관계형 데이터베이스 시스템과 달리 데이터베이스와 테이블은 데이터를 스키마 정의와 함께 저장하지 않습니다. 이 데이터는 테이블을 조회할 때만 Amazon S3에서 읽힙니다. Hive 사용의 또 다른 이점은 Hive에서 발견된 metastore가 Spark, Hadoop, Presto와 같은 많은 다른 빅데이터 어플리케이션에서 사용될 수 있다는 것입니다. Athena 카탈로그를 사용하면 Hadoop 클러스터 또는 RDS 인스턴스를 프로비저닝 할 필요 없이 클라우드에 Hive와 호환되는 메타스토어를 구축할 수 있습니다. 데이터베이스 및 테이블 작성에 대한 가이드는 [Apache Hive documentation](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL) 를 참조하십시오. 다음 단계에서는 Amazon Athena를 위한 가이드를 제공합니다.

![createbucket.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucket.png)

3. **Create Bucket**  팝업 페이지에서 고유한 **Bucket name** 을 입력하십시오. 고유한 이름을 위해 많은 임으의 문자와 숫자(공백 없는)가 있는 긴 버킷 이름을 선택하는 것이 좋습니다.
i. **Oregon** 지역을 선택하십시오. 
ii. 다음 탭으로 이동하려면 **Next**  을 클릭하십시오.
iii. **Set properties** 탭에서 모든 옵션을 기본값으로 유지하십시오.
iv. **Set permissions** 태그에서 모든 옵션을 기본값으로 유지하십시오.
v. **Review** 탭에서 **Create Bucket** 를 클릭하십시오.

![createbucketpopup.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucketpopup.png)

### 데이터 베이스 생성하기

1. [AWS Management Console for Athena](https://console.aws.amazon.com/athena/home) 를 여십시오.
2. AWS Management Console for Athena를 처음 방문하는 경우 시작하기 페이지가 표시됩니다. **Get Start** 를 선택하여 **Query Editor** 를 여십시오. 이번이 처음이 아니라면 Athena **Query Editor** 가 열립니다.
3. AWS 지역이름을 기록하십시오. 예를들면 이 lab의 경우 **US West (Oregon)** 지역을 선택해야 합니다.
4. Athena **Query Editor** 에서 예제 query가 있는 query 창을 볼 수 있습니다. 이제 query창에서 query 입력을 시작할 수 있습니다.
5. *mydatabase* 라는 데이터베이스를 생성하려면 다음 문장을 복사하고 **Run Query** 를 선택하십시오: 

```sql
    CREATE DATABASE mydatabase
```

6. **Catalog** 대시보드의 데이터베이스 목록에 *mydatabase*가 표시되는지 확인하십시오. 

![athenacatalog.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/athenacatalog.png)

### Create a Table

1. 현재 지역이 **US West (Oregon)** 지역인지 확인하십시오.
2. 데이터베이스 목록에서 **mydatabase**가 선택되었는지 확인한 다음 **New Query**를 선택하십시오.
3. query 창에서 다음 문장을 복사하여 TaxiDataYellow 테이블을 생성한 다음 **Run Query**를 선택하십시오 :

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

4.방금 생성한 테이블이 선택한 데이터베이스에 대한 카탈로그 대시보드에 나타나는지 확인하십시오.

테이블이 만들어졌으니 이제 파티션 메타데이터를 Amazon Athena 카탈로그에 추가해야 합니다. 

1. **New Query** 를 선택하고 다음 문장을 쿼리 창에 복사한 다음 **Run Query** 를 선택하여 파티션 메타데이터를 추가하십시오.

```sql
    MSCK REPAIR TABLE NYTaxiRides
```
반환된 결과는 2009년부터 2016년까지 매월 각 택시 유형(노란색, 녹색, FHV)에 대해 NYTaxiRides에 추가된 파티션에 대한 정보를 포함하고 있습니다. 

## Signing up for Amazon Quicksight Standard Edition

1. Open the [AWS ManagementConsole for QuickSight](https://us-east-1.quicksight.aws.amazon.com/sn/start).

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage1.PNG)

2. If this is the first time you are accessing QuickSight, you will see a sign-uplanding page for QuickSight. 
3. Click on **Sign up for QuickSight**.

> **Note:** Chrome browser might timeout at this step. If that's the case, try this step in Firefox/Microsoft Edge/Safari.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage2.PNG)

4. On the next page, for the subscription type select the **"Standard Edition"** and click **Continue**. 

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage3.PNG)

5. On the next page,

   i. Enter a unique **QuickSight account name.**

   ii. Enter avalid email for **Notification email address**.

   iii. Just for this step, leave the **QuickSight capacity region **as **N.Virginia**. 

   iv. Ensure that **Enable autodiscovery of your data and users in your Amazon Redshift, Amazon RDS and AWS IAM Services** and **Amazon Athena** boxes are checked. 

   v. **Click Finish**. 

   vi. You will be presented with a with message **Congratulations**! **You are signed up for Amazon QuickSight! **on successful sign up. Click on **Go to Amazon QuickSight**. 

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage4.PNG)

6. On the Amazon QuickSight dashboard, navigate to User Settings page on the Top-Right section and click **Manage QuickSight**.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage5.PNG)

7. In this section, click on **Account Settings**.
8. Under Account Settings, in **Account Permissions** Click **Edit AWS Permissions**.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage6.PNG)

9. Check the box for **Amazon S3** and you will see a pop-up to select Amazon S3 buckets.
10. Ensure **Select All **is checked.
11. Click on **Select buckets**.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage7.PNG)

12. Check the box for **Amazon S3 Storage Analytics**[Optional].
13. Click **Apply**.

## Configuring Amazon QuickSight to use Amazon Athena as data source

> For this lab, you will need to choose the **US West (Oregon)** region. 

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage8.PNG)

1. Click on the region icon on the top-right corner of the page, and select **US West (Oregon)**. 

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage9.PNG)

2. Click on **Manage data** on the top-right corner of the webpage to review existing data sets.
3. Click on **New data set** on the top-left corner of the webpage and review the options. 

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage10.PNG)

4. Select **Athena** as a Data source.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage11.PNG)

5. Enter the **Data source** **name** (e.g. *AthenaDataSource*).
6. Click **Create data source**.
7. Select the **mydatabase** database.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage12.PNG)

8. Choose the **nytaxirides** table.
9. Choose **Edit/Preview** data.

> This is a crucial step. Please ensure you choose **Edit/Preview** data.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage13.PNG)

10. Under **Fields** on the left column, choose **New field**

    i. Select the **extract** operation from Function list.

    ii. Select **pickup_datetime** from the **Field list**.

    iii. For **Calculated field name**, type **hourofday**.

    iv. Type ‘HH’ so the Formula is **extract('HH',{pickup_datetime})**

    v. Choose **Create** to add a field which is calculated from an existing field. In this case, the **hourofday** field is calculated from the **pickup_datetime filed** based on the specified formula.



11. Choose **Save and Visualize** on top of the page.

## Amazon QuickSight를 사용하여 데이터 시각화

이제 데이터 소스를 구성하고 시간을 나타내는 새 파일을 만들었으므로 이 섹션에서는 연도별로 데이터를 필터링하여 **pickup_datetime** 필드를 기반으로 2016년 1월 전체 달 동안 택시 데이터를 시각화 합니다.

### 연도 기반 필터를 추가하여 2016년 데이터 셋을 시각화


![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage14.PNG)

1. 현재 AWS 리전이 **미국 서부(오레곤)** 인지 확인하십시오.

2. **필드 목록**에서 **year** 필드를 선택하여 연도별 요금 분포를 표시하십시오.

3. **year**를 쉼표없이 재 포맷 하려면,

   i. **year** 필드에 대한 드롭다운 화살표를 선택하십시오

   ii. 드롭다운 메뉴에서  **Format 1,234.5678 **를 선택하십시오.

   iii. **1235** 를 선택하십시오.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage15.PNG)

4. **year** 필드에 필터를 추가하려면, 

   i. **Fields list** 에서 **year** 필드에 대한 드롭다운 메뉴를 선택하십시오.

   ii. 드롭다운메뉴에서 **Add filter to the field**를 선택하십시오.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage16.PNG)

5. 2016년에만 데이터를 필터링하려면,

   i.  **필터 편집** 메뉴에서 필터 이름 옆에 있는 **#**을 클릭하여 방금 만든 새로운 필터를 선택하십시오.
  
   ii. 필터 이름 아래의 두 드롭타운에 대하여 **필터 리스트**를 선택하십시오.
  
   iii. **Select All**를 체크해제하십시오.
  
   iv. **2016** 만 선택하십시오.
  
   v. **Apply**를 클릭하십시오.
  
   vi. **Close**를 클릭하십시오.

### 1월을 위한 월 기준 필터 추가

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage17.PNG)

1. 현재 AWS 리전이 **미국 서부(오레곤)** 인지 확인하십시오.

2. 왼쪽에 있는 네비게이션 메뉴에서 **Visualize**를 선택하십시오.

3. **Fields list**에서 **year**필드를 클릭하여 **year**를 선택해제하십시오.

4. **Fields list**의 **month** 필드를 클릭하여 **month**를 선택하십시오.

5. 1월에 대한 데이터 셋을 필터링하려면,

   i. **Fields list** 에서 **month** 필드에 대한 드롭다운 메뉴를 선택하십시오.

   ii. 드롭다운메뉴에서 **Add filter to the field**를 선택하십시오.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage18.PNG)

6. 2016년 1월에 대한 데이터를 필터링 하고 싶으면,

   i.  **필터 편집** 메뉴에서 필터 이름 옆에 있는 **#**을 클릭하여 방금 만든 새로운 필터를 선택하십시오.

   ii. 필터 이름 아래의 두 드롭타운에 대하여 **필터 리스트**를 선택하십시오.

   iii. **Select All**를 체크해제하십시오.

   iv. **1** 만 선택하십시오.

   v. **Apply**를 클릭하십시오.

<<<<<<< Updated upstream
   i. Choose the new filter that you just created by clicking on **#** next to filter name **month** under the **Edit Filter** menu.

   ii. Select **Filter list** for the two dropdowns under the filter name.

   iii. Deselect **ALL**.

   iv. Select only **1**.

   v. Click **Apply**

   vi. Click **Close**.
=======
   vi. **Close**를 클릭하십시오.
>>>>>>> Stashed changes

###  2016년 1월 한 달 동안의 시간 데이터를 시각화

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage19.PNG)

1. 왼쪽에 있는 네비게이션 메뉴에서 **Visualize**를 선택하십시오.
2. **Fields list**에서 **month**필드를 클릭하여 **month**를 선택해제하십시오.
3. **Fields list**의 **hourofday** 필드를 클릭하여 **hourofday**를 선택하십시오.
4. **Visual types**에 있는 스크린샷에서 강조된 꺽은선 차트 아이콘을 선택하여 보기형식을 꺽은선 차트로 변경하시오.
5. x축의 슬라이더를 사용하여 **hourofday** 필드에 대한 전체 범위를 선택하시오.

###  모든 택시 타입(yellow, green, fhv)에 대한 2016년 1월 한 달동안의 데이터를 시각화

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage20.PNG)

1. 페이지의 오른쪽 상단 모서리에 있는 사용자 이름 아래의 더블 드롭다운 화살표를 클릭하여 **Field wells** 아래에 **X축**, **값** 및 **색상**을 표시하십시오.
2. **Fields list**에서 **hourofday**필드를 클릭하여 **hourofday**를 선택해제하십시오..
3. **Fields list**에서 **pickup_datetime ** 필드를 클릭하여 x축으로 **pickup_datetime**를 선택하시오 .
4. **Fields list**에서 **type** 필드를 클릭하여 색상으로 **type**를 선택하시오.

5. x축의 필드 **pickup_datetime**를 선택하여 하위 메뉴를 표시하시오.
6. 일별로 집계하기 위해 **Aggregate:Day**를 선택하시오.

![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage21.PNG)
8. x축의 슬라이더를 사용하여 **pickup_datetime** 필드에 대해 2016년 1월 전체를 선택하시오.


> Note: 위 그래프에서 흥미로운 이상치는 2016년 1월 23일에 모든 유형의 택시 수의 감소를 볼 수 있다는 것입니다. 그 날짜를 구글로 검색해 보면, NBC 뉴욕에서 이 날짜에 대한 날씨 정보를 얻을 수 있습니다.
> ![image](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab2/qsimage22.PNG)

*Amazon QuickSight를 사용하여 시각화를 구축하고, 애드혹 분석을 시행하며, 통찰력을 빠르게 생성함으로써 시계열 데이터의 패턴을 볼 수 있습니다*

---
## License

This library is licensed under the Apache 2.0 License. 











































