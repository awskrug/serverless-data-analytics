# Lab 3: Amazon Glue를 사용한 Serverless ETL 및 Data Discovery

* [IAM 역할 생성하기](#IAM-역할-생성하기)
* [Amazon S3 bucket 생성하기](#Amazon-S3-bucket-생성하기)
* [데이터 발견하기](#데이터-발견하기)
* [쿼리를 최적화 하고 Parquet로 변환하기](#쿼리를-최적화-하고-Parquet로-변환하기)
* [Query the Partitioned Data using Amazon Athena](#query-the-partitioned-data-using-amazon-athena)
* [Deleting the Glue database, crawlers and ETL Jobs created for this Lab](#deleting-the-glue-database-crawlers-and-etl-jobs-created-for-this-lab)
* [Summary](#summary)

## Architectural Diagram
![architecture-overview-lab3.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/Screen+Shot+2017-11-17+at+1.11.32+AM.png)

## IAM 역할 생성하기

Amazon S3 소스, 타겟, 임시 디렉토리, 스크립트 **AWSGlueServiceRole**  및 작업에서 사용하는 모든 라이브러리에 대한 권한이 있는 IAM 역할을 생성합니다.  [여기](https://console.aws.amazon.com/iam/home?region=us-west-2#/roles) 를 클릭하여 새 역할을 만들 수 있습니다. 역할을 생성하는 추가 문서를 보려면 [이곳](docs.aws.amazon.com/cli/latest/reference/iam/create-role.html)을 클릭하십시오.

1. IAM 페이지에서 **Create Role** 을 누르십시오.
2. 서비스를 **Glue** 로 선택하고 **Next: Permissions** 을 클릭하십시오.
3. 사용 권한 연결 정책에서 S3에 대한 정책을 검색하고 **AmazonS3FullAccess** 에 대한 확인란을 선택하십시오.

> 정책을 클릭하지 말고 해당 확인란을 선택하십시오.

4. 같은 페이지에서 Glue에 대한 정책을 검색하고 **AWSGlueServiceRole** 및 **AWSGlueConsoleFullAccess** 확인란을 선택하십시오.

> 정책을 클릭하지 말고 해당 확인란을 선택하십시오.

5. **Next: Review** 을 클릭하십시오.
6. 역할 이름을 다음과 같이 입력하십시오. 

```
nycitytaxianalysis-reinv
```

​	Finish 를 클릭하십시오.

## Amazon S3 bucket 생성하기

1. [AWS Management console for Amazon S3](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) 을 여십시오.
2. S3 대시보드에서 **Create Bucket** 을 클릭하십시오. 

![createbucket.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucket.png)

1. **Create Bucket** 팝업 페이지에서 고유한 **Bucket name**을 입력하십시오. 많은 임의의 문자와 숫자(공백 없는)가 있는 긴 버킷 이름을 선택하는 것이 좋습니다. 

   ```
   aws-glue-scripts-<YOURAWSACCOUNTID>-us-west-2
   ```

   i.**Oregon** 지역을 선택하십시오. 
ii. **Next** 를 눌러 다음 탭으로 이동하십시오. 
   iii. **Set properties** 탭에서 모든 옵션을 기본값으로 유지하십시오. 
   iv. **Set permissions** 태그에서 모든 옵션을 기본값으로 유지하십시오.
   v.  **Review** 탭에서 **Create Bucket** 을 클릭하십시오.

![createbucketpopup.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab1/createbucketpopup.png)

2. 이제 새로 생성된 버킷에서 위의 단계와 동일한 방법을 사용하여 두 개의 하위 버킷 **tmp** 와 **target** 을 생성하십시오. 이 버킷들은 Lab3의 나중에 사용할 것 입니다.

## 데이터 발견하기

이번 워크샵에서는 뉴욕시 택시 기록 데이터세트 1개월에 초점을 맞춥니다. 하지만 8년간의 전체 데이터 세트에 대해서도 이 작업을 쉽게 수행 할 수 있습니다. 알 수 없는 데이터 세트를 크롤링하게 되면 데이터 유형이 택시 유형에 따라 다른 형식임을 알게 됩니다. 그 다음 표준 형식으로 데이터를 변환하고 분석을 시작하고 일련의 시각화를 합니다. 모든 것은 단일 서버의 실행 없이 이루어 집니다.

> 이 실험에서는 **US West (Oregin)** 지역을 선택해야 합니다.

1. [AWS Management console for Amazon Glue](https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2#) 을 여십시오.

2. 2016년 1월의 모든 택시 승차수를 분석하기위해 S3에서의 데이터 세트로 시작합니다. 먼저 AWS Glue 내에서 이 워크샵을 위한 데이터 베이스를 만듭니다. 데이터베이스는 논리적인 테이블로 구성되는 연관 테이블 정의 세트 입니다. Athena에서는 입력하 내용에 관계없이 데이터베이스 이름이 모두 소문자입니다.

   i. 왼쪽의 Data catalog 열 에서  **Databases** 를 클릭하십시오.
   
   ![glue1](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_1.PNG)
   
   ii. **Add Database** 버튼을 클릭하십시오.
   
   iii. 데이터베이스 이름을 **nycitytaxianalysis-reinv17** 로 입력하십시오. 설명 등을 건너 뛰고 **Create**를 클릭하십시오.
   
3. 왼쪽에 있는 Data catalog 열 에서 **Crawlers** 를 클릭하십시오.

   ![glue2](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_2.PNG)
   
   i. **Add Crawler** 버튼을 클릭하십시오.
   
   ii. 크롤러에 대한 정보 추가에서 크롤러 이름에 **nycitytaxianalysis-crawler-reinv17** 를 입력 합니다. 설명 등을 건너 뛰고 **Next** 를 클릭하십시오.
   
   iii. Data Store에서 S3를 선택하십시오. 그리고 **Crawl Data in Specified path** 을 위한 버튼이 선택되어 있는지 확인하십시오.
   
   iv. Include path에 다음 S3 경로를 입력하고 **Next**를 클릭하십시오.

   ```
   s3://serverless-analytics/glue-blog
   ```
   
   v. 다른 데이터 저장소 추가에서 **No**를 선택하고 **Next**를 클릭하십시오.
   
   vi. IAM의 경우, **Create an IAM role**을 선택하고 다음과 같은 이름을 입력하고 **Next**를 클릭하십시오.
   
   ```
   nycitytaxianalysis-reinv17-crawler
   ```
   
   vii. 이 크롤러에 대한 일정을 만들기 위해 **Run on Demand**를 선택하고 **Next**를 클릭하십시오.
   
   viii. 크롤러의 output database와 prefix 설정하기

   ​	a. **Database**로는 앞서 만든 데이터베이스 **nycitytaxianalysis-reinv17** 를 선택하기
   
   ​	b. **Prefix added to tables (optional)** 에는  **reinv17_** 를 입력하고 **Next** 클릭하기

   ​	c. 설정을 검토하고 **Finish**클릭, 그 후 다음 페이지에서 맨 위에 있는 녹생상자에서 **Run it now**를 클릭하기

   ![glue14](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_14.PNG)
   ​	d. 크롤러가 실행이 되고 세개의 테이블을 찾았음을 나타내 줌
   
4. 왼쪽 열의 Data catalog에서 **Tables**를 클릭하십시오.

5. **Tables**를 아래를 보면 nycitytaxianalysis-reinv17 데이터베이스에서 생성 된 세 개의 새 테이블을 볼 수 있습니다.

    ![glue4](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_4.PNG)
    
6. 크롤러는 내장된 분류자(classifier)를 사용하고 테이블을 CSV로 식별하고 열/데이터 유형을 유추하며 각 표에 대한 속성 모음을 수집했습니다. 각 각의 테이블 정의를 보면 데이터 세트의 행의 수와 열이 테이블간 일치하지 않음을 알 수 있습니다. 예를 들면 reinv17_yellow 테이블을 클릭하면 2017년 1월의 870만 개의 노란색 데이터 세트와 S3에서의 위치 그리고 발견된 다양한 열들을 볼 수 있습니다.

   ![glue5](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_5.PNG)


## 쿼리를 최적화 하고 Parquet로 변환하기

이 데이터를 쿼리 최적화 형식으로 변환하기 위해 ETL 작업을 만듭니다. 데이터를 열 형식으로 변환하고 저장 형식을 Parquet으로 변경하고 소유하고 있는 버켓에 데이터를 작성합니다.

1. [AWS Management console for Amazon Glue](https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2#)를 엽니다.

2. 왼쪽 열의 ETL 아래에 있는 **Jobs**를 클릭 한 다음 **Add Job**버튼을 클릭하십시오.

3. Job properties에서 이름을 **nycitytaxianalysis-reinv17-yellow**로 입력하십시오. 왜냐하면 우리는 이번 워크샵에서 노란 데이터 세트 만을 사용할 것이기 때문입니다.

   i. IAM이라면 이번 lab의 시작부분에 작성한 IAM role을 선택하십시오.
   
   x. 이 작업에선 **A proposed script generated by AWS Glue**를 선택하십시오.
   
   xi. 스크립트 파일이름으로 **nycitytaxianalysis-reinv17-yellow**를 입력하십시오.
   
   > 이 워크샵에서는 노란색 데이터 세트에 대해서만 작업하고 있습니다. 녹색 및 FHV 데이터 세트를 변환하려면 이 단계를 자유롭게 실행하십시오.
   
   xii. 스크립트가 저장된 S3경로의 경우 폴더 아이콘을 클릭하고 워크샵 시작 부분에 생성된 S3 버킷을 선택하십시오. **폴더 아이콘을 통해 새로 생성된 S3 버킷을 선택하십시오.**
   
   xiii. 임시 디렉토리의 경우 워크샵 시작부부엔 생성된 tmp 폴더를 선택하십시오. **폴더 아이콘을 통해 S3 버킷을 선택** 하고 **Next**를 클릭하십시오.
   
   > 임시 버킷이 이미 S3 버킷에서 생성되었거나 사용 가능한지 확인하십시오.
   
   ![glue15](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_15.PNG)
   
   xiv. Advanced properties(고급 속성)을 클릭하고 Job bookmark를 **Enable**로 선택하십시오.
   
   xv. 다음은 완료된 작업 속성 창의 스크린 샷입니다.
   
   ![glue16](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_16.PNG)
   
4. **Next**를 클릭하십시오.

5. 데이터 원본 선택에서 데이터 원본으로 **reinv17_yellow** 테이블을 선택하고 **Next**를 클릭합니다.

   > 이 워크샵에서는 노란색 데이터 세트에 대해서만 작업하고 있습니다. 녹색 및 FHV 데이터 세트를 변환하려면 이 단계를 자유롭게 실행하십시오.
   
6. 데이터 대상 선택에서 **Create tables in your data target**을 선택하십시오.

   i. Data sotre의 경우 **Amazon S3**를 선택하십시오.
   
   ii. 형식의 경우 **Parquet**을 선택하십시오.
   
   iii. 타겟 경로에 대해서는 **폴더 아이콘을 클릭** 후 이전에 작성한 타겟 폴더를 선택하십시오. **S3 버킷/폴더 는 변형된 Parquet 데이터를 포함합니다.**
   

![glue17](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_17.PNG)

7. 원본 열에서 타켓 열로 매핑하는 페이지,

   i. 타겟에서 열 이름 **tpep_pickup_datetime**를 **pickup_date**로 변경하십시오. 해당 **data type** 필드 문자열을 클릭하고 열 타입을 **TIMESTAMP**로 변경하고 **Update**를 클릭하십시오.
   
   ii. 타겟에서 열 이름 **tpep_dropoff_datetime** 을 **dropoff_date**로 변경하십시오. 해당 **data type** 필드 문자열을 클릭하고, 열 타입을 **TIMESTAMP**로 변경하고 **Update**를 클릭하십시오.
   
   iii. **Next**를 선택하고 정보를 확인한 다음 **Finsh**를 클릭하십시오.

![glue9](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_9.PNG)

8. 자동 생성 스크립트 페이지에서 **Save** 및 **Run Job**을 클릭 하십시오.

![glue10](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_10.PNG)

9. 매개 변수 팝업에서 Job Bookmarks에 대해 **Enable**을 확인하고 **Run Job**을 클릭하십시오.

10. 이 작업은 대략 30분 정도 실행됩니다.

![glue11](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_11.PNG)

11. 같은 페이지의 하단에서 로그를 볼 수 있습니더.

![glue12](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_12.PNG)

12. 위에 지정된 대상 폴더 (S3 Bucket)에서 (step 6 iii)는 이제 변환 된 Parquet 데이터를 가지게 됩니다.


## Query the Partitioned Data using Amazon Athena

In regions where AWS Glue is supported, Athena uses the AWS Glue Data Catalog as a central location to store and retrieve table metadata throughout an AWS account. The Athena execution engine requires table metadata that instructs it where to read data, how to read it, and other information necessary to process the data. The AWS Glue Data Catalog provides a unified metadata repository across a variety of data sources and data formats, integrating not only with Athena, but with Amazon S3, Amazon RDS, Amazon Redshift, Amazon Redshift Spectrum, Amazon EMR, and any application compatible with the Apache Hive metastore.

1. Open the [AWS Management console for Amazon Athena](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2). 

   > Ensure you are in the **US West (Oregon)** region. 

2. Under Database, you should see the database **nycitytaxianalysis-reinv17** which was created during the previous section. 

3. Click on **Create Table** right below the drop-down for Database and click on **Automatically (AWS Glue Crawler)**.

4. You will now be re-directed to the AWS Glue console to set up a crawler. The crawler connects to your data store and automatically determines its structure to create the metadata for your table. Click on **Continue**.

5. Enter Crawler name as **nycitytaxianalysis-crawlerparquet-reinv17** and Click **Next**.

6. Select Data store as **S3**.

7. Choose Crawl data in **Specified path in my account**.

8. For Include path, click on the folder Icon and choose the **target** folder previously made which contains the parquet data and click on **Next**.

![glue18](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_18.PNG)

9. In Add another data store, choose **No** and click on **Next**.

10. For Choose an IAM role, select Choose an existing IAM role, and in the drop-down pick the role made in the previous section and click on **Next**.

11. In Create a schedule for this crawler, pick frequency as **Run on demand** and click on **Next**.

12. For Configure the crawler's output, Click **Add Database** and enter **nycitytaxianalysis-reinv17-parquet** as the database name and click **create**. For Prefix added to tables, you can enter a prefix **parq_** and click **Next**.

13. Review the Crawler Info and click **Finish**. Click on **Run it Now?**. 

14. Click on **Tables** on the left, and for database nycitytaxianalysis-reinv17-parquet you should see the table parq_target. Click on the table name and you will see the MetaData for this converted table. 

15. Open the [AWS Management console for Amazon Athena](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2). 

    > Ensure you are in the **US West (Oregon)** region. 

16. Under Database, you should see the database **nycitytaxianalysis-reinv17-parquet** which was just created. Select this database and you should see under Tables **parq_target**.

17. In the query editor on the right, type

    ```
    select count(*) from parq_target;
    ```

    and take note the Run Time and Data scanned numbers here. 

    ![glue19](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_comp_scanresult.PNG)

    What we see is the Run time and Data scanned numbers for Amazon Athena to **query and scan the parquet data**.

18. Under Database, you should see the earlier made database **nycitytaxianalysis-reinv17** which was created in a previous section. Select this database and you should see under Tables **reinv17_yellow**. 

19. In the query editor on the right, type

    ```
    select count(*) from reinv17_yellow;
    ```

    and take note the Run Time and Data scanned numbers here. 

    ![glue20](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_uncomp_scanresult.PNG)

20. What we see is the Run time and Data scanned numbers for Amazon Athena to query and scan the uncompressed data from the previous section.


> Note: Athena charges you by the amount of data scanned per query. You can save on costs and get better performance if you partition the data, compress data, or convert it to columnar formats such as Apache Parquet.

## Deleting the Glue database, crawlers and ETL Jobs created for this Lab

Now that you have successfully discovered and analyzed the dataset using Amazon Glue and Amazon Athena, you need to delete the resources created as part of this lab. 

1. Open the [AWS Management console for Amazon Glue](https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2#). Ensure you are in the Oregon region (as part of this lab).
2. Click on **Databases** under Data Catalog column on the left. 
3. Check the box for the Database that were created as part of this lab. Click on **Action** and select **Delete Database**. And click on **Delete**. This will also delete the tables under this database. 
4. Click on **Crawlers** under Data Catalog column on the left. 
5. Check the box for the crawler that were created as part of this lab. Click on **Action** and select **Delete Crawler**. And click on **Delete**. 
6. Click on **Jobs** under ETL column on the left. 
7. Check the box for the jobs that were created as part of this lab. Click on **Action** and select **Delete**. And click on **Delete**. 
8. Open the [AWS Management console for Amazon S3](https://s3.console.aws.amazon.com/s3/home).
9. Click on the S3 bucket that was created as part of this lab. You need to click on its corresponding **Bucket icon** to select the bucket instead of opening the bucket. Click on **Delete bucket** button on the top, to delete the S3 bucket. In the pop-up window, Type the name of the bucket (that was created as part of this lab), and click **Confirm**. 

## Summary

In the lab, you went from data discovery to analyzing a canonical dataset, without starting and setting up a single server. You started by crawling a dataset you didn’t know anything about and the crawler told you the structure, columns, and counts of records.

From there, you saw the datasets were in different formats, but represented the same thing: NY City Taxi rides. You then converted them into a canonical (or normalized) form that is easily queried through Athena and possible in QuickSight, in addition to a wide number of different tools not covered in this post.

---
## License

This library is licensed under the Apache 2.0 License. 







































