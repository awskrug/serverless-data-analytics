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

## AMAZON QUICKSIGHT

### What is Amazon QuickSight?

Amazon QuickSight는 시각화를 쉽게 구축하고, 임시 분석을 수행하며, 데이터에서 비지니스 통찰력을 빠르게 얻을 수 있는 클라우드 기반의 비지니스 분석 서비스이다. 클라우드 기반 서비스를 사용하여 데이터에 쉽게 연결하고 고급 분석을 수행하며 모든 브라우저 또는 모바일 장치에서 액세스 할수 있는 놀라운 시각화 및 풍부한 대시 보드를 만들 수 있다. 

### How is Amazon QuickSight different from traditional Business Intelligence (BI) solutions?

기존 BI 솔루션은 보고서를 생성하기 전에 데이터 엔지니어 팀이 복잡한 데이터 모델을 구축하는 데 수 개월을 투자해야 하는 경우가 많다. 일반적으로 대화형 임시 데이터 탐색 및 시각화가 부족하여 사용자를 통조림 보고서와 미리 선택된 쿼리로 제한한다. 기존 BI 솔루션은 또한 복잡하고 비용이 많이 드는 하드웨어와 소프트웨어에 대한 상당한 초기 투자가 필요하며,  데이터베이스 크기가 증가함에 따라 빠른 쿼리 성능을 유지하기 위해 훨씬 더 많은 인프라에 투자해야 한다. 이러한 비용과 복잡성으로 인해 기업은 조직 전반에 걸쳐 분석 솔루션을 구현하기가 어렵워진다. Amazon QuickSight는 AWS Cloud의 규모와 유연성을 비즈니스 분석에 도입하여 이러한 문제를 해결하도록 설계되었다. 기존의 BI 또는 데이터 검색 솔루션과 달리 Amazon QuickSight를 시작하는 것은 간단하고 빠르다. 로그인하면 Amazon QuickSight는 Amazon Redshift, Amazon RDS, Amazon Athena, Amazon Simple Storage Service(Amazon S3)와 같은 AWS 서비스에서 데이터 소스를 원활하게 검색한다. Amazon QuickSight에서 발견한 모든 데이터 소스에 연결하여 몇 분 만에 이 데이터로부터 통찰력을 얻을 수 있다. Amazon QuickSight를 선택하면 기본 소스의 데이터가 변경될 때 SPICE의 데이터를 최신 상태로 유지할 수 있다. SPICE는 풍부한 데이터 검색 및 비즈니스 분석 기능을 지원하여 고객이 인프라 프로비저닝 또는 관리에 대한 걱정없이 데이터에서 가치있는 통찰력을 얻을 수 있도록 지원한다. 기업은 Amazon QuickSight 사용자 한 명당 낮은 월 사용료를 지불하여 장기 사용권 비용을 없앤다.  Amazon QucikSight를 통해 기업은 막대한 비용을 들이지 않고도 모든 직원에게 풍부한 비지니스 분석기능을 제공할 수 있다.

### Which data sources does Amazon QuickSight support?

Amazon RDS, Amazon Aurora, Amazon Redshift, Amazon Athena 및 Amazon S3를 비롯한 AWS 데이터 소스에 연결할 수 있다. xcel 스프레드 시트 또는 플랫 파일 (CSV, TSV, CLF 및 ELF)을 업로드하고, SQL Server, MySQL 및 PostgreSQL과 같은 온 프레미스 데이터베이스에 연결하고 Salesforce와 같은 SaaS 애플리케이션에서 데이터를 가져올 수도 있다.

자세한 내용은 [Amazon Athena FAQ](https://aws.amazon.com/athena/faqs/) 를 참조하십시오.

## Amazon Redshift Spectrum

### What is Amazon Redshift Spectrum?

Amazon Redshift Spectrum은 Amazon S3의 비 압축된 엑사바이트의 데이터에 대한 쿼리를 로딩이나 ETL 없이 실행할 수 있는 Amazon RedShift의 기능이다.쿼리를 실행하면 Amazon RedShift SQL 끝점으로 이동하여 쿼리 계획을 생성하고 최적화 한다. Amazon Redshift는 로컬 데이터 및 Amazon S3에 있는 데이터를 결정하고, 읽어야하는 Amazon S3 데이터의 양을 최소화하기 위해 Redshift Spectrum 작업자에게 공유 리소스 풀에서 Amazon S3의 데이터를 읽고 처리하도록 요청한다.

Redshift Spectrum은 필요할 경우 수천 개의 인스턴스로 확장되므로 데이터 크기에 관계없이 쿼리가 빠르게 실행된다. 그리고 현재 Amazon Redshift 쿼리에 대해 사용하는 것과 동일한 SQL을 Amazon S3 데이터에 사용하고 동일한 BI 도구를 사용하여 동일한 Amazon Redshift Endpoint에 연결할 수 있다. Redshift Spectrum을 사용하면 스토리지와 컴퓨팅을 분리하여 각각 독립적으로 확장할 수 있다. Amazon S3 데이터 lake를 쿼리하는 데 필요한만큼 많은 Amazon Redshift 클러스터를 설정하여 높은 가용성과 무한한 동시성을 제공 할 수 있다. Redshift Spectrum은 데이터를 원하는 위치에 원하는 형식으로 저장할수 있게 해주며 필요할 때 언제든지 처리 할수 있도록 해준다.

### Can Redshift Spectrum replace Amazon EMR?

아니다. Redshift Spectrum은 Amazon Redshift 및 S3의 데이터에 대한 쿼리를 실행하는 데 적합하지만 실제로 Amazon EMR과 같은 처리 프레임 워크에서 일반적으로 요청하는 사용 사례 유형에는 적합하지 않다. Amazon EMR은 단순히 SQL 쿼리를 실행하는 것을 훨씬 능가한다. Amazon EMR은 사용자 정의가 가능한 클러스터에서 Spark, Hadoop 및 Presto와 같이 널리 사용되는 대형 데이터 처리 프레임 워크의 최신 버전을 사용하여 대용량 데이터 세트를 처리하고 분석 할 수있는 관리 서비스이다. Amazon EMR을 사용하면 기계 학습, 그래프 분석, 데이터 변환, 스트리밍 데이터 및 코드 작성이 가능한 거의 모든 애플리케이션과 같은 다양한 스케일 아웃 데이터 처리 작업을 실행할 수 있다. 또한 Redshift Spectrum을 EMR과 함께 사용할 수도 있다. Amazon Redshift Spectrum은 Amazon EMR과 같은 방식으로 테이블 정의를 저장한다. 따라서 이미 EMR을 사용하여 대규모 데이터 저장소를 처리하고 있다면 Redshift Spectrum을 사용하여 Amazon EMR 작업을 방해하지 않고도 데이터를 동시에 쿼리할 수 있다.

쿼리서비스, 데이터 웨어하우스, 복잡한 데이터 처리 프레임워크는 모두 각 자의 역할이 있으며, 서로 다른 용도로 사용된다. 단지 작업에 있어서 적합한 도구를 선택하기만 하면 된다. 

### When should I use Amazon Athena vs. Redshift Spectrum?

Amazon Athena는 모든 직원에게 Amazon S3의 데이터에 대한 임의 (ad-hoc) 쿼리를 실행할 수있는 가장 간단한 방법이다. Athena는 서버가 없으므로 설정 또는 관리 할 인프라가 없으므로 데이터를 즉시 분석 할 수 있습니다.

자주 접근하는 데이터를 일관성 있고 구조화 된 형식으로 저장해야하는 경우 Amazon Redshift와 같은 데이터웨어 하우스를 사용해야한다. 이를 통해 구조화되고 자주 접근되는 데이터를 Amazon Redshift에 저장하고 Redshift Spectrum을 사용하여 Amazon Redshift 쿼리를 Amazon S3 데이터 lake의 전체 데이터 범위로 확장 할 수 있다. 이렇게하면 원하는 형식으로 원하는 위치에 자유롭게 데이터를 저장할 수 있으며 필요할 때 처리 할 수 있다.

### Can I use Redshift Spectrum to query data that I process using Amazon EMR?

네, Redshift Spectrum은 Amazon EMR이 데이터 및 테이블 정의를 찾는 데 사용하는 것과 동일한 Apache Hive Metastore를 지원할 수 있다. Amazon EMR을 사용하고 있고 이미 Hive Metastore를 사용하고 있다면 Amazon Redshift 클러스터를 구성하여 사용하면된다. 그런 다음 Amazon EMR 작업과 함께 즉시 해당 데이터 쿼리를 시작할 수 있다. 

자세한 내용은 [Amazon Athena FAQ](https://aws.amazon.com/athena/faqs/) 를 참조하십시오.
