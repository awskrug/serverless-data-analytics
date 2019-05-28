# Building an End-to-End Serverless Data Analytics Solution on AWS

## Overview

이 실습에서는 [Amazon Athena](https://aws.amazon.com/ko/athena/)를 이용하여 Amazon S3에서 직접 데이터를 분석하고 [Amazon QuickSight](https://quicksight.aws/)에서 데이터를시각화하는 serverless architecture를 구축하려고 한다.  사용할 데이터 세트는 공개 데이터 세트로 2009년부터 2016년까지 뉴욕시의 노랜색과 녹색 택시에서 완료된 모든 주행기록과 2015년에서 2016년까지 상용 차량(FHV)의 모든 주행기록을 포함하는 데이터 세트다.  기록에는 픽업 및 드롭오프 날짜 / 시간, 픽업 및 드롭오프 장소, 주행거리, 항목별 요금, 요금 유형, 지불 유형,  운전자가 보고한 승객 수를 캡처하는 필드를 포함한다. 데이터 세트는 사전에 분할되어 있으며 CSV에서 Apache Parquet로 변환되어 있다.  실습의 첫번째 파트에서 당신은 Amazon Athena를 사용하여 Query와 같은 SQL을 구축하여 Amazon S3에서 직접적으로 형성되는 데이터 포멧을 Query화 하고 Query 성능을 비교하게 될 것이다. 두번째 파트에서는 Amazon Athena table을 사용하여 Amazon QuickSight의 데이터 소스로 Query를 생성하면 Amazon S3의 데이터 세트에서 시각화 및 의미있는 통찰력을 얻을 수 있다. Query의 성능을 최적화 하기위해 AWS Glue를 사용하는 Serverless ETL을 통합하기 위한 선택적 실습이 포함되어 있다. 또한  동일한 설계를 재적용하고 Redshift Spectrum을 사용하여 Amazon Redshift 데이터 웨어하우스에서 Amazon S3의 동일한 데이터 세트를 직접 query할 수 있는 take-home lab 액세스 권한을 제공한다.

![architecture-overview.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/architectureoveriew.PNG)

------

## AWS Console

### Verifying your region in the AWS Management Console

Amazon EC2를 사용하면 인스턴스를 여러 위치에 배치할 수 있다. Amazon EC2 위치는 하나 이상의 이용가능한 영역이 포함된 지역으로 구성된다. 지역은 분산되어 있으며 분리된 지리적 지역(US, EU, etc.)에 위치한다. 이용가능한 영역은 지역 내에서 별개의 위치이다. 이들은 다른 이용가능한 영역의 장애로부터 격리되고 동일한 지역의 다른 이용가능한 영역에 대한 저렴하고 지연이 짧은 네트워크 연결을 제공하도록 설계된다.



개별 지역에서 인스턴트를 시작하므로서 특정 고객에게 더 가까이 다가가거나 법적 또는 기타 요구 사항을 충족하도록 어플리케이션을 설계할 수 있다. 분리된 이용가능한 영역에서 인스턴트를 시작하므로서 지역화된 지역 장애로부터 어플리케이션을 보호할 수 있다.

### Verify your Region

AWS 지역 이름은 항상 AWS Management Console의 오른쪽 상단 모서리의 탐색 바에 나열된다.

- AWS 지역 이름을 기록해 두십시오. 예를들어 이 실습의 경우 **US West (Oregon)** 지역을 선택하십시오.
- 아래 표를 사용하여 지역 코드를 확인하십시오. 이 실습에 대해 **us-west-2** 를 선택하십시오.

| Region Name                        | Region Code    |
| ---------------------------------- | -------------- |
| US East (Northern Virginia) Region | us-east-1      |
| US West (Oregon) Region            | us-west-2      |
| Asia Pacific (Tokyo) Region        | ap-northeast-1 |
| Asia Pacific (Seoul) Region        | ap-northeast-2 |
| Asia Pacific (Singapore) Region    | ap-southeast-1 |
| Asia Pacific (Sydney) Region       | ap-southeast-2 |
| EU (Ireland) Region                | eu-west-1      |
| EU (Frankfurt) Region              | eu-central-1   |

------

## Labs

### Pre-requisites

Amazon  계정이 없으면 [새로운 Amazon 계정을 생성하십시오.](https://aws.amazon.com/free/) 

| Lab   | Name                                                         |
| ----- | ------------------------------------------------------------ |
| Lab 1 | [Serverless Analysis of data in Amazon S3 using Amazon Athena](./Lab1) |
| Lab 2 | [Visualization using Amazon QuickSight](./Lab2)              |
| Lab 3 | [Serverless ETL and Data Discovery using Amazon Glue](./Lab3) |
| Lab 4 | [Analysis of data in Amazon S3 using Amazon Redshift Spectrum](./Lab4) |

## AMAZON ATHENA

### What is Amazon Athena?

Amazon Athena는 표준 SQL을 사용하여 Amazon S3의 데이터를 쉽게 분석할 수 있는 대화형 query 서비스다. Athena는 서버가 없기 때문에 설정하거나 관리할 수 있는 인프라가 필요 없으며, 즉시 데이터 분석을 시작할 수 있다. Athena에 데이터를 로드 할 필요가 없으며 S3에 저장된 데이터로 직접 작동한다. 시작하려며 Athena Management Console에 로그인하고 스키마를 정의한 다음 querying을 시작하십시오. Amazon Athena는 완전한 표준 SQL 지원 기능을 갖춘 Presto를 사용하며 CSV, JSON, ORC, Apache Parquet, Avro 등 다양한 표준 데이터 형식과 연동한다. Amazon Athena는 빠르고 임시적인 query에 이상적이며 Amazon QuickSight와 통합하여 쉽게 시각화 하고 large joins, window functions, 배열 등 복잡한 분석도 처리할 수 있다.

### What can I do with Amazon Athena?

Amazon Athena는 Amazon S3에 저장된 데이터를 분석하는 것을 돕는다. 데이터를 Athena에 집계하거나 로드할 필요없이 ANSI SQL을 사용하여 임시 query를 실행할 수 있다. Amazon Athena는 비정형, 반정형, 정형화된 데이터 세트를 처리할 수 있다. 예를 들어, CSV, JSON, Avro 또는 Apache Parquet 및 Apache ORC와 같은 columnar 데이터 형식이 있다. Amazon Athena는 Amazon QuickSight와 통합하여 쉽게 시각화 할 수 있다. 또한 Amazon Athena를 사용하여 [JDBC 드라이버](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html) 를 통해 연결된 비즈니스 인텔리전스 툴 또는 SQL 클라이언트로 보고서를 생성하거나 데이터를 탐색할 수 있다.

### How do you access Amazon Athena?

Amazon Athena는 AWS management console과 JDBC 드라이버를 통해 접속할 수 있다. [JDBC 드라이버](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html) 를 사용하여 프로그램적으로 query를 실행하거나 테이블 또는 파티션을 추가할 수 있다.

### What is the underlying technology behind Amazon Athena?

Amazon Athena는 완전한 표준 SQL 지원 기능을 갖춘 Presto를 사용하며, CSV, JSON, ORC, Avro, Parquet 등 다양한 표준 데이터 형식으로 작동한다. Athena는 large joins, window functions, 배열 등을 포함한 복잡한 분석을 처리할 수 있다. Amazon Athena는  Amazon S3를 기본 데이터 저장소로 사용하기 때문에, 여러 시설과 각 시설에 있는 여러 기기에서 중복적으로 데이터를 저장하기 때문에 가용성과 내구성이 높다.

### How do I create tables and schemas for my data on Amazon S3?

Amazon Athena는 테이블을 정의하기 위해 Apache Hive DDL을 사용한다. Athena console, JDBC 드라이버를 사용하거나 Athena create table wizard를 사용하여 DDL문을 실행할 수 있다. Amazon Athena에서 새로운 테이블 스키마를 만들 때 스키마는 데이터 카탈로그에 저장되어 query를 실행할 때 사용되지만, S3에서 데이터를 수정하지는 않는다. Athena는 query를 싱행할 때 스키마를 데이터에 투영할 수 있도록 schema-on-read라고 알려진 접근법을 사용한다. 따라서 데이터 로드 또는 변환이 필요하지 않는다. [테이블 만들기](http://docs.aws.amazon.com/athena/latest/ug/creating-tables.html) 에 대해 자세히 알아보십시오.

### What data formats does Amazon Athena support?

Amazon Athena는 CSV, TSV, JSON 또는 텍스트파일과 같은 광범위한 데이터 형식을 지원하며 Apache ORC 및 Apache Parquet와 같은 오픈소스 columnar 형식도 지원한다. Athena는 또한 압축된 데이터를 Snappy, Zlib, LZO, GZIP 형식으로 지원한다. 압축, 파티셔닝 및 columnar 형식을 사용하면 성능을 개선하고 비용을 절감할 수 있다.



자세한 내용은 [Amazon Athena FAQ](https://aws.amazon.com/athena/faqs/) 를 참조하십시오.

