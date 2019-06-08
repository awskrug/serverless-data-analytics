# Lab 4: Analysis of data in Amazon S3 using Amazon Redshift Spectrum

* [Amazon Redshift 클러스터 배포하기](#Amazon-Redshift-클러스터-배포하기)
* [AWS Glue Crawlers 실행하기 - CSV 및 Parquet 크롤러](#AWS-Glue-Crawlers-실행하기---CSV-및-Parquet-크롤러)
* [AWS Glue 데이터 카탈로그 데이터베이스를 구성하는 Redshift Spectrum Scehma 및 참조 외부 테이블 생성하기](#AWS-Glue-데이터-카탈로그-데이터베이스를-구성하는-Redshift-Spectrum-Scehma-및-참조-외부-테이블-생성하기)
* [Amazon Redshift Spectrum를 사용하여 Amazon S3에서 데이터 쿼리](#Amazon-Redshift-Spectrum를-사용하여-Amazon-S3에서-데이터-쿼리)
* [Amazon Redshift Spectrum를 사용하여 분할된 데이터 쿼리](#Amazon-Redshift-Spectrum를-사용하여-분할된-데이터-쿼리)


## Architectural Diagram
![architecture-overview-lab4.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-17+at+1.11.45+AM.png)

## Amazon Redshift 클러스터 배포하기

이 섹션에서는 Amazon Redshift 클러스터 리소스를 생성하기 위해 CloudFromation 템플릿을 사용할 것입니다. 이 템플릿은 Amazon EC2 인스턴스에 PostgreSQL 용 [pgweb](https://github.com/sosedoff/pgweb) , SQL Client를 설치하여 실행된 Amazon Redshift 클러스터에 쿼리를 연결하고 실행합니다. 다른 방법으로는 SQL Workbench/J 와 같은 표준 SQL 클라이언트를 사용하여 Amazon Redshift 클러스터에 연결할 수 있습니다. 자세한 내용은  http://docs.aws.amazon.com/redshift/latest/mgmt/connecting-using-workbench.html 을 참조하십시오.

1. AWS 콘솔에 로그인하여  [Amazon CloudFormation Dashboard](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2]) 를 여십시오.
2. AWS 지역 이름을 기록해 두십시오. 예를 들어 이 실습을 위해서 **US West (Oregon)** 지역을 선택해야 합니다.
3. **Create Stack** 을 클릭하십시오.
4. **Specify an Amazon S3 template URL** 을 선택하십시오.
5. 다음 S3 템플릿 URL을 텍스트 상자에 붙여넣으십시오.
```
https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/us-west-2.serverless-data-analytics/labcontent/redshiftspectrumglue-lab4.template
```
6. **Next** 을 클릭하십시오.

>**Note:** 
>[redshiftspectrumglue-lab4.template](../Lab4/redshiftspectrumglue-lab4.template) 링크를 클릭하여 Amazon CloudFormation 템플릿 파일을 확인하십시오.


![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.38.08+PM.png)

8. **Stack Name** 에 대한 이름 *(예. RedshiftSpectrumLab)* 을 입력하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.38.39+PM.png)

9. **Redshift Cluster Configuration** 에 대해 다음 **parameters** 를 입력하십시오.
  1. **ClusterType** : *multi-node* 를 선택하십시오.
        2. **NumberOfNodes** : *2* 를 입력하십시오.
            3. **NodeType**  : *dc1.xlarge* 를 선택하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.38.57+PM.png)

10. **Redshift Database Configuration** 에 대해 다음 **parameters** 를 입력하십시오. 

    i. **MasterUserName** : 이름 (예. dbadmin) 을 입력하십시오.
    ii. **MasterUserPassword** : 비밀번호를 입력하십시오.
    iii. **DatabaseName** : 이름 (예. taxidb) 을 입력하십시오.
    iv. **ClientIP** : 로컬 시스템의 IP 주소를 입력하십시오.
    
    

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.39.23+PM.png)

11. **Glue Crawler Configuration** 에 대해 다음 **parameters** 를 입력하십시오.
    1. **GlueCatalogDBName** : 이름 (예. taxi-spectrum-db) 을 입력하십시오.    
    2. **CSVCrawler** : 이름 (예. csvCrawler) 을 입력하십시오.
    3. **ParquetCrawler** : 이름 (예. parquetCrawler) 을 입력하십시오.
    
12.  **Next** 를 클릭하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.40.04+PM.png)

13. [선택사항] **Options** 의 **Tags** 하위 섹션에 **Key** 이름 *(예. Name)* 과  **Value** 를 입력하십시오 .
14.  **Next** 를 클릭하십시오.

![IMAGE](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-16+at+7.40.31+PM.png)

15. **I acknowledge that AWS CloudFormation might create IAM resources** 를 확인하십시오.
16. **Create** 을 클릭하십시오.

> **Note:** 대략 15분 정도 소요됩니다.

17. 방금 생성한 Amazon CloudFromation 스택의 상태가 **CREATE_COMPLETE** 인지 확인하십시오.
18. Amazon CloudFormation 스택을 선택하십시오 *(RedshiftSpectrumLab)*
19. **Outputs** 탭을 클릭하십시오.
20. 다음과 같은 **Key** 및  **Value**  목록을 검토하십시오.

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

## Amazon Redshift Spectrum를 사용하여 Amazon S3에서 데이터 쿼리

이제 스키마를 만들었으므로 데이터 세트에 대해 쿼리를 실행하고 PGWeb Console에서 결과를 볼 수 있습니다.

1. 다음 문장을 쿼리창에 복사한 후 **Run Query** 를 선택하십시오.

```sql
    SELECT * FROM taxispectrum.taxi limit 10
```

위 쿼리의 결과는 다음과 같습니다:

![Screen Shot 2017-11-14 at 9.16.45 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+9.16.45+PM.png)

2.	다음 문장을 쿼리창에 복사한 후 **Run Query** 를 선택해 yellow cabs의 총 택시 탑승 횟수를 조회하십시오.

```sql
    SELECT COUNT(1) as TotalCount FROM taxispectrum.taxi
```
위 쿼리의 결과는 다음과 같습니다:

![Screen Shot 2017-11-14 at 9.25.23 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+9.25.23+PM.png)

3. 다음 문장을 쿼리창에 복사한 후 **Run Query** 를 선택해 노란 택시 탑승횟수에 대한 평균 fair amount와 함께 공급 업체 당 타는 횟수를 조회하십시오.

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

위 쿼리의 결과는 다음과 같습니다:

![Screen Shot 2017-11-14 at 9.46.55 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+9.46.55+PM.png)

## Amazon Redshift Spectrum를 사용하여 분할된 데이터 쿼리

데이터를 분할함으로써 각 쿼리에 의해 스캔되는 데이터의 양을 제한할 수 있으므로 성능이 향상되고 비용이 절감됩니다. Amazon Redshift Spectrum은 데이터를 [분할](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterPartition)하기 위해 Hive를 활용합니다. 임의의 키로 데이터를 분할할 수 있습니다.일반적으로 시간을 기준으로 데이터를 분할하여 여러 수준의 파티셔닝 스키마를 생성하는 경우가 많습니다. 예를 들어, 매시간 데이터가 들어오는 고객은 년, 월, 일 및 시간별로 파티션을 결정할 수 있습니다. 많은 다른 소스에서 데이터를 가져오지만 하루에 한 번 로드하는 또 다른 고객은 데이터 소스 식별자와 날짜로 분할할 수 있습니다.


Athena 데이터 카탈로그에 파티션 메타 데이터를 추가 했으므로 이제 쿼리를 실행할 수 있습니다.

1. 다음 문장을 쿼리창에 복사한 후 **Run Query** 를 선택해 총 택시 탑승 횟수를 조회하십시오.

```sql
    SELECT count(1) as TotalCount from taxispectrum.ny_pub
```
위 쿼리의 결과는 다음과 같습니다:

![Screen Shot 2017-11-14 at 10.08.50 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.08.50+PM.png)

>**Note:**
> 이 쿼리는 데이터 세트가 분할되고 최적의 형식인 Apache Parquet(열린 소스 열)으로 이루어지기 때문에 훨씬 더 빨리 실행됩니다.

2. 다음 문을 쿼리 창에 복사 한 다음 **Run Query**을 선택하여 연도 별 총 택시 탑승 횟수를 조회하십시오.

```sql
    SELECT YEAR, count(1) as TotalCount from taxispectrum.ny_pub GROUP BY YEAR
```
위 쿼리의 결과는 다음과 같습니다:
![Screen Shot 2017-11-14 at 10.11.47 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.11.47+PM.png)

3. 다음 문을 쿼리 창에 복사 한 다음 **Run Query**을 선택하여 모든 연도의 총 택시 탑승 횟수를 기준으로 상위 12달을 조회하십시오.

```sql
    SELECT YEAR, MONTH, COUNT(1) as TotalCount 
    FROM taxispectrum.ny_pub
    GROUP BY (1), (2) 
    ORDER BY (3) DESC LIMIT 12
```
위 쿼리의 결과는 다음과 같습니다:
![Screen Shot 2017-11-14 at 10.13.54 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.13.54+PM.png)

4. 다음 문을 쿼리 창에 복사 한 다음 **Run Query**을 선택하여 2016년의 월간 택시 탑승 횟수를 조회하십시오.

```sql
    SELECT MONTH, TYPE, COUNT(1) as TotalCount 
    FROM taxispectrum.ny_pub
    WHERE YEAR = 2016 
    GROUP BY (1), (2)
    ORDER BY (1), (2)
```
위 쿼리의 결과는 다음과 같습니다:
![Screen Shot 2017-11-14 at 10.18.08 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.18.08+PM.png)

5. 다음 문을 쿼리 창에 복사 한 다음 **Run Query**을 선택하십시오.

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

위 쿼리의 결과는 다음과 같습니다:

![Screen Shot 2017-11-14 at 10.23.51 PM.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/lab4/Screen+Shot+2017-11-14+at+10.23.51+PM.png)

## Amazon CloudFormation Stack 삭제

이제 Amazon Redshift Spectrum을 사용하여 데이터 세트를 성공적으로 쿼리했으므로 Amazon CloudFormation 템플릿을 사용하여 배포한 스택을 분리해야 합니다.

1. [Amazon CloudFormation Dashboard](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2) 를 여십시오.
2. 실습의 시작 부분에 배포한 스택이름 *(e.g. RedshiftSpectrumLab)*  옆의 체크박스를 선택하십시오.
3. 드롭다운 버튼의 **Actions** 을 클릭하십시오.
4. **Delete Stack** 를 선택하십시오.
5. Click **Yes, Delete** on the *Delete Stack* 팝업 창에서 **Yes, Delete** 를 클릭하십시오.
6. Amazon CloudFromation 스택 이름 *(e.g. RedshiftSpectrumLab)* 이 더 이상 스택 목록에 나타나지 않는지 확인하십시오.

---
## License

This library is licensed under the Apache 2.0 License. 
