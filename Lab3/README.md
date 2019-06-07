# Lab 3: Amazon Glue를 사용한 Serverless ETL 및 Data Discovery

* [IAM 역할 생성하기](#IAM-역할-생성하기)
* [Amazon S3 bucket 생성하기](#Amazon-S3-bucket-생성하기)
* [데이터 발견하기](#데이터-발견하기)
* [쿼리를 최적화 하고 Parquet로 변환하기](#쿼리를-최적화-하고-Parquet로-변환하기)
* [Amazon Athena를 사용하여 분할 된 데이터 쿼리](#Amazon-Athena를-사용하여-분할-된-데이터-쿼리)
* [이 실습에서 만든 Glue 데이터베이스, 크롤러 및 ETL Jobs 삭제](#이-실습에서-만든-Glue-데이터베이스,-크롤러-및-ETL-Jobs-삭제)
* [요약](#요약)

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


## Amazon Athena를 사용하여 분할 된 데이터 쿼리

AWS Glue가 지원되는 지역에서 Athena는 AWS Glue Data Catalog를 중앙 위치로 사용하여 AWS 계정 전체에 테이블 메타 데이터를 저장하고 검색합니다. Athena 실행 엔진은 어디서 데이터를 읽을지, 어떻게 읽을지, 데이터를 처리하는데 필요한 다른 정보를 알려주는 테이블 메타데이터를 필요로 합니다.  AWS Glue Data Catalog는 Athena뿐만 아니라 Amazon S3, Amazon RDS, Amazon Redshift, Amazon Redshift Spectrum, Amazon EMR 및 Apache Hive metastore와 호환되는 모든 애플리케이션과 통합되는 다양한 데이터 소스와 데이터 형식에 걸쳐 통합된 메타데이터 저장소를 제공합니다.

1. [AWS Management console for Amazon Athena](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2) 를 여십시오. 

   > **미국 서부(오레곤)** 지역인지 확인하십시오. 

2. 이전 섹션에서 작성된 **nycitytaxianalysis-reinv17** 데이터베이스를 보십시오.

3.  데이터베이스의 드롭다운 바로 아래 **Create Table** 을 클릭하고 **Automatically (AWS Glue Crawler)** 를 클릭하십시오.

4. 이제 크롤러를 설치하기 위해 AWS Glue 콘솔로 이동됩니다. 크롤러는 데이터 저장소에 연결되고 테이블의 메타데이터를 작성하기 위해 해당 구조를 자동으로 결정합니다. **Continue** 를 클릭하십시오.

5. 크롤러의 이름을  **nycitytaxianalysis-crawlerparquet-reinv17** 으로 입력하고 **Next** 를 클릭하십시오.

6. 데이터 저장소를 **S3** 으로 선택하십시오.

7. Crawl data in 에서 **Specified path in my account** 를 선택하십시오.

8. 경로 설정을 위해 폴더 아이콘을 클릭하고 parquet 데이터를 포함하고 있는 이전에 작성된 **target** 폴더를 선택하고 다음을 클릭하십시오.

![glue18](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_18.PNG)

9. 다른 데이터 저장소 추가에서 **No** 를 선택하고 **Next** 를 클릭하십시오.  

10. IAM 규칙을 선택하려면 Choose an existing IAM role를 선택하고 드롭다운에서 이전 섹션에서 만든 규칙을 선택하고 **Next**를 클릭하십시오.

11. Create a schedule for this crawler에서 pick frequency as **Run on demand** and click on **Next**.빈도를 **Run on demand** 으로 선택하고 **Next** 를 클릭하십시오.

12. Configure the crawler's output에서 **Add Database** 를 클릭하고 데이터베이스의 이름을 **nycitytaxianalysis-reinv17-parquet** 으로 입력하고 **create** 를 클릭하십시오. Prefix added to tables에서 접두사에 **parq_** 를 입력하고 **Next** 를 클릭하십시오. 

13. Review the Crawler Info and click **Finish**. Click on **Run it Now?**. 크롤러의 정보를 확인하고 **Finish** 를 클릭하십시오. **Run it Now?** 를 클릭하십시오.

14. 왼쪽에 있는 **Table**을 클릭하고 데이터베이스 nycitytaxianalysis-reinv17-parquet에 대해 parq_target 테이블을 보십시오.

15. [AWS Management console for Amazon Athena](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2) 를 여십시오. 

    > **미국 서부(오레곤)** 지역인지 확인하십시오.  

16. 데이터베이스에서 방금 생성 된 데이터베이스 **nycitytaxianalysis-reinv17-parquet** 을 볼 수 있습니다. 이 데이터베이스를 선택하면 테이블 **parq_target** 아래에 표시됩니다.

17. 오른쪽의 쿼리 편집기에서 다음을 입력하고

    ```
    select count(*) from parq_target;
    ```

    그리고 실행시간과 데이터 스캔 값을 기록하십시오.

    ![glue19](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_comp_scanresult.PNG)

     우리가 보는것은 Amazon Athena가 **parquet 데이터를 쿼리하고 스캔 할 수 있는 ** 실행 시간과 데이터 스캔 값 입니다.

18. 데이터베이스에서  이전 섹션에서 작성된 데이터베이스 **nycitytaxianalysis-reinv17** 를 봐야 합니다. 이 데이터베이스를 선택하고 테이블 아래에 있는  **reinv17_yellow** 를 확인하십시오.

19. 오른쪽의 쿼리 편집기에서 다음을 입력하고

    ```
    select count(*) from reinv17_yellow;
    ```

    그리고 실행시간과 데이터 스캔 값을 기록하십시오.

    ![glue20](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab3/glue_uncomp_scanresult.PNG)

20. 우리가 보는 것은 Amazon Athena가 이전 섹션에서 압축되지 않은 데이터를 쿼리하고 스캔하기위해 실행 시간과 데이터 스캔 값 입니다.


> Note: Athena는 쿼리 당 스캔된 데이터 양으로 요금을 청구합니다. 데이터를 분할하거나 데이터를 압축하거나 Apache Parquet과 같은 컬럼 형식으로 변환하면 비용을 절약하고 성능을 향상시킬 수 있습니다.

## 이 실습에서 만든 Glue 데이터베이스, 크롤러 및 ETL Jobs 삭제

이제 Amazon Glue와 Amazon Athena를 사용하여 데이터 세트를 성공적으로 검색하고 분석했으므로, 이 실습에서 생성된 리소스를 삭제해야 합니다.

1. Open the [AWS Management console for Amazon Glue](https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2#) 를 여십시오. 오레곤 지역인지 확인하십시오.
2. 왼쪽의 Data Catalog 열 아래에 **Databases** 를 클릭하십시오.
3. 이번 실습에서 생성된 데이터베이스를 선택하십시오. **Action** 을 클릭하고 **Delete Database** 를 선택하십시오. 그리고 **Delete** 를 클릭하십시오. 그러면 이 데이터베이스 아래의 테이블도 삭제됩니다.
4. 왼쪽의 Data Catalog 열 아래에 **Crawlers** 를 클릭하십시오.
5. 이번 실습에서 생성된 크롤러를 선택하십시오. **Action** 을 클릭하고 **Delete Crawler** 를 선택하십시오. 그리고 **Delete** 를 클릭하십시오.
6. 왼쪽의 ETL 열 아래에 **Jobs** 를 클릭하십시오.
7. 이번 실습에서 생성된 jobs를 선택하십시오. Click on **Action** and select **Delete**. **Action** 을 클릭하고 **Delete** 를 선택하십시오. 그리고 **Delete** 를 클릭하십시오.
8. [AWS Management console for Amazon S3](https://s3.console.aws.amazon.com/s3/home) 를 여십시오.
9. 이번 실습에서 생성된 S3 버킷를 클릭하십시오. You need to click on its corresponding **Bucket icon** to select the bucket instead of opening the bucket. 버킷을 열지 않고 선택하려면 **버킷 아이콘** 을 클릭해야합니다. S3 버킷을 삭제하려면 상단의 **버킷 삭제** 버튼을 클릭하십시오. 팝업 창에서 이번 실습에 생성된 버킷 이름을 입력하고 **확인**을 클릭하십시오.

## 요약

이번실습에서 단일서버를 시작하고 설정하지 않고 데이터 검색에서 표준데이터 세트 분석으로 이동하였습니다. 먼저 알지 못했던 데이터 세트를 크롤링하여 크롤러가 구조, 열 및 레코드 수를 알려 줬습니다.

그리고 데이터세트가 다른 포맷이지만 같은 것임을 보았습니다. 그런 다음  Athena 및 QuickSight에서 쉽게 쿼리 할 수있는 표준 (정규화 된) 양식으로 변환했으며, 이 포스트에는 포함되지 않은 다양한도구도 많이 있었습니다.

---
## License

This library is licensed under the Apache 2.0 License. 







































