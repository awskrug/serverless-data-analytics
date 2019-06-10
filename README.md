> 한국어 번역에는 안시원(ssibongee@yonsei.ac.kr), 장용주(yongju1264@gmail.com), 권민(min94913@gmail.com)님이 수고해주셨습니다.

# Building an End-to-End Serverless Data Analytics Solution on AWS

## Overview

이 실습에서는 [Amazon Athena](https://aws.amazon.com/ko/athena/)를 이용하여 Amazon S3에서 직접 데이터를 분석하고 [Amazon QuickSight](https://quicksight.aws/)에서 데이터를시각화하는 serverless architecture를 구축하려고 합니다.  사용할 데이터 세트는 공개 데이터 세트로 2009년부터 2016년까지 뉴욕시의 노랜색과 녹색 택시에서 완료된 모든 주행기록과 2015년에서 2016년까지 상용 차량(FHV)의 모든 주행기록을 포함하는 데이터 세트입니다.  기록에는 픽업 및 드롭오프 날짜 / 시간, 픽업 및 드롭오프 장소, 주행거리, 항목별 요금, 요금 유형, 지불 유형,  운전자가 보고한 승객 수를 캡처하는 필드를 포함합니다. 데이터 세트는 사전에 분할되어 있으며 CSV에서 Apache Parquet로 변환되어 있습니다.  실습의 첫번째 파트에서 당신은 Amazon Athena를 사용하여 Query와 같은 SQL을 구축하여 Amazon S3에서 직접적으로 형성되는 데이터 포멧을 Query화 하고 Query 성능을 비교하게 될 것입니다. 두번째 파트에서는 Amazon Athena table을 사용하여 Amazon QuickSight의 데이터 소스로 Query를 생성하면 Amazon S3의 데이터 세트에서 시각화 및 의미있는 통찰력을 얻을 수 있습니다. Query의 성능을 최적화 하기위해 AWS Glue를 사용하는 Serverless ETL을 통합하기 위한 선택적 실습이 포함되어 있습니다. 또한  동일한 설계를 재적용하고 Redshift Spectrum을 사용하여 Amazon Redshift 데이터 웨어하우스에서 Amazon S3의 동일한 데이터 세트를 직접 query할 수 있는 take-home lab 액세스 권한을 제공합니다.

![architecture-overview.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/architectureoveriew.PNG)

------

## AWS Console

### Verifying your region in the AWS Management Console

Amazon EC2를 사용하면 인스턴스를 여러 위치에 배치할 수 있습니다. Amazon EC2 위치는 하나 이상의 이용가능한 영역이 포함된 지역으로 구성됩니다. 지역은 분산되어 있으며 분리된 지리적 지역(US, EU, etc.)에 위치합니다. 이용가능한 영역은 지역 내에서 별개의 위치입니다. 이들은 다른 이용가능한 영역의 장애로부터 격리되고 동일한 지역의 다른 이용가능한 영역에 대한 저렴하고 지연이 짧은 네트워크 연결을 제공하도록 설계되었습니다.



개별 지역에서 인스턴트를 시작하므로서 특정 고객에게 더 가까이 다가가거나 법적 또는 기타 요구 사항을 충족하도록 어플리케이션을 설계할 수 있습니다. 분리된 이용가능한 영역에서 인스턴트를 시작하므로서 지역화된 지역 장애로부터 어플리케이션을 보호할 수 있습니다.

### Verify your Region

AWS 지역 이름은 항상 AWS Management Console의 오른쪽 상단 모서리의 탐색 바에 나열됩니다.

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

Amazon Athena는 표준 SQL을 사용하여 Amazon S3의 데이터를 쉽게 분석할 수 있는 대화형 query 서비스입니다. Athena는 서버가 없기 때문에 설정하거나 관리할 수 있는 인프라가 필요 없으며, 즉시 데이터 분석을 시작할 수 있습니다. Athena에 데이터를 로드 할 필요가 없으며 S3에 저장된 데이터로 직접 작동합니다. 시작하려며 Athena Management Console에 로그인하고 스키마를 정의한 다음 querying을 시작하십시오. Amazon Athena는 완전한 표준 SQL 지원 기능을 갖춘 Presto를 사용하며 CSV, JSON, ORC, Apache Parquet, Avro 등 다양한 표준 데이터 형식과 연동합니다. Amazon Athena는 빠르고 임시적인 query에 이상적이며 Amazon QuickSight와 통합하여 쉽게 시각화 하고 large joins, window functions, 배열 등 복잡한 분석도 처리할 수 있습니다.

### What can I do with Amazon Athena?

Amazon Athena는 Amazon S3에 저장된 데이터를 분석하는 것을 돕습니다. 데이터를 Athena에 집계하거나 로드할 필요없이 ANSI SQL을 사용하여 임시 query를 실행할 수 있습니다. Amazon Athena는 비정형, 반정형, 정형화된 데이터 세트를 처리할 수 있습니다. 예를 들어, CSV, JSON, Avro 또는 Apache Parquet 및 Apache ORC와 같은 columnar 데이터 형식이 있습니다. Amazon Athena는 Amazon QuickSight와 통합하여 쉽게 시각화 할 수 있습니다. 또한 Amazon Athena를 사용하여 [JDBC 드라이버](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html) 를 통해 연결된 비즈니스 인텔리전스 툴 또는 SQL 클라이언트로 보고서를 생성하거나 데이터를 탐색할 수 있습니다.

### How do you access Amazon Athena?

Amazon Athena는 AWS management console과 JDBC 드라이버를 통해 접속할 수 있습니다. [JDBC 드라이버](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html) 를 사용하여 프로그램적으로 query를 실행하거나 테이블 또는 파티션을 추가할 수 있습니다.

### What is the underlying technology behind Amazon Athena?

Amazon Athena는 완전한 표준 SQL 지원 기능을 갖춘 Presto를 사용하며, CSV, JSON, ORC, Avro, Parquet 등 다양한 표준 데이터 형식으로 작동합니다. Athena는 large joins, window functions, 배열 등을 포함한 복잡한 분석을 처리할 수 있습니다. Amazon Athena는  Amazon S3를 기본 데이터 저장소로 사용하기 때문에, 여러 시설과 각 시설에 있는 여러 기기에서 중복적으로 데이터를 저장하기 때문에 가용성과 내구성이 높습니다.

### How do I create tables and schemas for my data on Amazon S3?

Amazon Athena는 테이블을 정의하기 위해 Apache Hive DDL을 사용합니다. Athena console, JDBC 드라이버를 사용하거나 Athena create table wizard를 사용하여 DDL문을 실행할 수 있습니다. Amazon Athena에서 새로운 테이블 스키마를 만들 때 스키마는 데이터 카탈로그에 저장되어 query를 실행할 때 사용되지만, S3에서 데이터를 수정하지는 않습니다. Athena는 query를 싱행할 때 스키마를 데이터에 투영할 수 있도록 schema-on-read라고 알려진 접근법을 사용합니다. 따라서 데이터 로드 또는 변환이 필요하지 않습니다. [테이블 만들기](http://docs.aws.amazon.com/athena/latest/ug/creating-tables.html) 에 대해 자세히 알아보십시오.

### What data formats does Amazon Athena support?

Amazon Athena는 CSV, TSV, JSON 또는 텍스트파일과 같은 광범위한 데이터 형식을 지원하며 Apache ORC 및 Apache Parquet와 같은 오픈소스 columnar 형식도 지원합니다. Athena는 또한 압축된 데이터를 Snappy, Zlib, LZO, GZIP 형식으로 지원합니다. 압축, 파티셔닝 및 columnar 형식을 사용하면 성능을 개선하고 비용을 절감할 수 있습니다.



자세한 내용은 [Amazon Athena FAQ](https://aws.amazon.com/athena/faqs/) 를 참조하십시오.

## AMAZON QUICKSIGHT

### What is Amazon QuickSight?

Amazon QuickSight는 시각화를 쉽게 구축하고, 임시 분석을 수행하며, 데이터에서 비지니스 통찰력을 빠르게 얻을 수 있는 클라우드 기반의 비지니스 분석 서비스 입니다. 클라우드 기반 서비스를 사용하여 데이터에 쉽게 연결하고 고급 분석을 수행하며 모든 브라우저 또는 모바일 장치에서 액세스 할수 있는 놀라운 시각화 및 풍부한 대시 보드를 만들 수 있습니다.

### How is Amazon QuickSight different from traditional Business Intelligence (BI) solutions?

기존 BI 솔루션은 보고서를 생성하기 전에 데이터 엔지니어 팀이 복잡한 데이터 모델을 구축하는 데 수 개월을 투자해야 하는 경우가 많습니다. 일반적으로 대화형 임시 데이터 탐색 및 시각화가 부족하여 사용자를 통조림 보고서와 미리 선택된 쿼리로 제한합니다. 기존 BI 솔루션은 또한 복잡하고 비용이 많이 드는 하드웨어와 소프트웨어에 대한 상당한 초기 투자가 필요하며,  데이터베이스 크기가 증가함에 따라 빠른 쿼리 성능을 유지하기 위해 훨씬 더 많은 인프라에 투자해야 합니다. 이러한 비용과 복잡성으로 인해 기업은 조직 전반에 걸쳐 분석 솔루션을 구현하기가 어렵워집니다. Amazon QuickSight는 AWS Cloud의 규모와 유연성을 비즈니스 분석에 도입하여 이러한 문제를 해결하도록 설계되었습니다. 기존의 BI 또는 데이터 검색 솔루션과 달리 Amazon QuickSight를 시작하는 것은 간단하고 빠릅니다. 로그인하면 Amazon QuickSight는 Amazon Redshift, Amazon RDS, Amazon Athena, Amazon Simple Storage Service(Amazon S3)와 같은 AWS 서비스에서 데이터 소스를 원활하게 검색합니다. Amazon QuickSight에서 발견한 모든 데이터 소스에 연결하여 몇 분 만에 이 데이터로부터 통찰력을 얻을 수 있습니다. Amazon QuickSight를 선택하면 기본 소스의 데이터가 변경될 때 SPICE의 데이터를 최신 상태로 유지할 수 있습니다. SPICE는 풍부한 데이터 검색 및 비즈니스 분석 기능을 지원하여 고객이 인프라 프로비저닝 또는 관리에 대한 걱정없이 데이터에서 가치있는 통찰력을 얻을 수 있도록 지원합니다. 기업은 Amazon QuickSight 사용자 한 명당 낮은 월 사용료를 지불하여 장기 사용권 비용을 없앱니다.  Amazon QucikSight를 통해 기업은 막대한 비용을 들이지 않고도 모든 직원에게 풍부한 비지니스 분석기능을 제공할 수 있습니다.

### Which data sources does Amazon QuickSight support?

Amazon RDS, Amazon Aurora, Amazon Redshift, Amazon Athena 및 Amazon S3를 비롯한 AWS 데이터 소스에 연결할 수 있습니다. xcel 스프레드 시트 또는 플랫 파일 (CSV, TSV, CLF 및 ELF)을 업로드하고, SQL Server, MySQL 및 PostgreSQL과 같은 온 프레미스 데이터베이스에 연결하고 Salesforce와 같은 SaaS 애플리케이션에서 데이터를 가져올 수도 있습니다.

자세한 내용은 [Amazon Athena FAQ](https://aws.amazon.com/athena/faqs/) 를 참조하십시오.

## Amazon Redshift Spectrum

### What is Amazon Redshift Spectrum?

Amazon Redshift Spectrum은 Amazon S3의 비 압축된 엑사바이트의 데이터에 대한 쿼리를 로딩이나 ETL 없이 실행할 수 있는 Amazon RedShift의 기능입니다.쿼리를 실행하면 Amazon RedShift SQL 끝점으로 이동하여 쿼리 계획을 생성하고 최적화 합니다. Amazon Redshift는 로컬 데이터 및 Amazon S3에 있는 데이터를 결정하고, 읽어야하는 Amazon S3 데이터의 양을 최소화하기 위해 Redshift Spectrum 작업자에게 공유 리소스 풀에서 Amazon S3의 데이터를 읽고 처리하도록 요청합니다.

Redshift Spectrum은 필요할 경우 수천 개의 인스턴스로 확장되므로 데이터 크기에 관계없이 쿼리가 빠르게 실행되비다. 그리고 현재 Amazon Redshift 쿼리에 대해 사용하는 것과 동일한 SQL을 Amazon S3 데이터에 사용하고 동일한 BI 도구를 사용하여 동일한 Amazon Redshift Endpoint에 연결할 수 있습니다. Redshift Spectrum을 사용하면 스토리지와 컴퓨팅을 분리하여 각각 독립적으로 확장할 수 있습니다. Amazon S3 데이터 lake를 쿼리하는 데 필요한만큼 많은 Amazon Redshift 클러스터를 설정하여 높은 가용성과 무한한 동시성을 제공 할 수 있습니다. Redshift Spectrum은 데이터를 원하는 위치에 원하는 형식으로 저장할수 있게 해주며 필요할 때 언제든지 처리 할수 있도록 해줍니다.

### Can Redshift Spectrum replace Amazon EMR?

아닙니다. Redshift Spectrum은 Amazon Redshift 및 S3의 데이터에 대한 쿼리를 실행하는 데 적합하지만 실제로 Amazon EMR과 같은 처리 프레임 워크에서 일반적으로 요청하는 사용 사례 유형에는 적합하지 않습니다. Amazon EMR은 단순히 SQL 쿼리를 실행하는 것을 훨씬 능가합니다. Amazon EMR은 사용자 정의가 가능한 클러스터에서 Spark, Hadoop 및 Presto와 같이 널리 사용되는 대형 데이터 처리 프레임 워크의 최신 버전을 사용하여 대용량 데이터 세트를 처리하고 분석 할 수있는 관리 서비스입니다. Amazon EMR을 사용하면 기계 학습, 그래프 분석, 데이터 변환, 스트리밍 데이터 및 코드 작성이 가능한 거의 모든 애플리케이션과 같은 다양한 스케일 아웃 데이터 처리 작업을 실행할 수 있습니다. 또한 Redshift Spectrum을 EMR과 함께 사용할 수도 있습니다. Amazon Redshift Spectrum은 Amazon EMR과 같은 방식으로 테이블 정의를 저장합니다. 따라서 이미 EMR을 사용하여 대규모 데이터 저장소를 처리하고 있다면 Redshift Spectrum을 사용하여 Amazon EMR 작업을 방해하지 않고도 데이터를 동시에 쿼리할 수 있습니다.

쿼리서비스, 데이터 웨어하우스, 복잡한 데이터 처리 프레임워크는 모두 각 자의 역할이 있으며, 서로 다른 용도로 사용됩니다. 단지 작업에 있어서 적합한 도구를 선택하기만 하면 됩니다. 

### When should I use Amazon Athena vs. Redshift Spectrum?

Amazon Athena는 모든 직원에게 Amazon S3의 데이터에 대한 임의 (ad-hoc) 쿼리를 실행할 수있는 가장 간단한 방법입니다. Athena는 서버가 없으므로 설정 또는 관리 할 인프라가 없으므로 데이터를 즉시 분석 할 수 있습니다.

자주 접근하는 데이터를 일관성 있고 구조화 된 형식으로 저장해야하는 경우 Amazon Redshift와 같은 데이터웨어 하우스를 사용해야합니다. 이를 통해 구조화되고 자주 접근되는 데이터를 Amazon Redshift에 저장하고 Redshift Spectrum을 사용하여 Amazon Redshift 쿼리를 Amazon S3 데이터 lake의 전체 데이터 범위로 확장 할 수 있습니다. 이렇게하면 원하는 형식으로 원하는 위치에 자유롭게 데이터를 저장할 수 있으며 필요할 때 처리 할 수 있습니다.

### Can I use Redshift Spectrum to query data that I process using Amazon EMR?

네, Redshift Spectrum은 Amazon EMR이 데이터 및 테이블 정의를 찾는 데 사용하는 것과 동일한 Apache Hive Metastore를 지원할 수 있습니다. Amazon EMR을 사용하고 있고 이미 Hive Metastore를 사용하고 있다면 Amazon Redshift 클러스터를 구성하여 사용하면 됩니다. 그런 다음 Amazon EMR 작업과 함께 즉시 해당 데이터 쿼리를 시작할 수 있습니다. 

자세한 내용은 [Amazon Athena FAQ](https://aws.amazon.com/athena/faqs/) 를 참조하십시오.

## AWS Glue

### What is AWS Glue?

AWS Glue는 분석을위한 데이터 전처리 시간을 자동화하는 ETL (Extract, Transform and Load) 서비스입니다. AWS Glue는 Glue Data Catalog를 통해 데이터를 자동으로 찾고 프로파일링하며 소스 데이터를 대상 스키마로 변환하는 ETL 코드를 추천 및 생성하고 완전히 관리되고 scale-out Apache Spark 환경에서 ETL job을 실행하여 데이터를 대상에 로드합니다. 또한 복잡한 데이터 흐름을 설정, 조정 및 모니터링 할 수 있습니다.

### What are the main components of AWS Glue?

AWS Glue는 중앙 메타데이터 저장소인 Data Catalog, 파이썬 코드를 자동으로 생성할 수 있는 ETL 엔진, 의존성 해결, 작업 모니터링 및 재시도를 처리하는 유연한 스케줄러로 구성됩니다. 이와 함께 데이터 검색, 분류, 정리, enriching, 이동 같은 어려운 업무들을 자동화하여 데이터 분석에 더 많은 시간을 할애 할 수 있습니다.

### When should I use AWS Glue?

분석을 위해 AWS Glue를 사용하여 데이터의 속성을 찾고 변환하고 전처리를 할 수 있습니다. Glue는 Amazon S3의 data lake, Amazon Redshift의 data warehouse, AWS에서 실행되는 다양한 데이터베이스에서 structured 데이터와 semi-structured 데이터를 자동으로 찾을 수 있습니다. Glue Data Catalog를 통해 ETL, Amazon Athena, Amazon EMR 및 Amazon Redshift Spectrum 같은 서비스를 사용하여 쿼리 및 보고 데이터의 unified view를 제공합니다. Glue는 손에 익은 툴을 사용하여 추가로 customize 할 수 있는 ETL job을 위한 Python 코드를 자동으로 생성합니다.(Glue automatically generates Python code for your ETL jobs that you can further customize using tools you are already familiar with.) AWS Glue는 서버가 없으므로 구성 및 관리할 컴퓨팅 리소스가 없습니다.

### What data sources does AWS Glue support?

AWS Glue는 기본적으로 Amazon Aurora, MySQL용 Amazon RDS, Oracle용 Amazon RDS, PostgreSQL용 Amazon RDS, SQL Server용 Amazon RDS, Amazon Redshift, Amazon S3 뿐만 아니라 Amazon EC2에서 실행되고 있는 Virtual Private Cloud (Amazon VPC)에 있는 MySQL, Oracle, Microsoft SQL Server, PostgreSQL 데이터베이스도 데이터 저장을 지원합니다. AWS Glue Data Catalog에 저장된 메타데이터는 Amazon Athena, Amazon EMR, Amazon Redshift Spectrum에서 쉽게 액세스 할 수 있습니다. AWS Glue에서 기본적으로 지원하지 않는 data source에 액세스하기 위해 Glue ETL jobs에 custom PySpark 코드를 작성하거나 custom 라이브러리를 가져올 수 있습니다.

### What is the AWS Glue Data Catalog?

AWS Glue Data Catalog는 모든 데이터 assets에 대한 구조 및 운영 메타데이터를 저장하는 중앙 저장소입니다. 데이터 셋이 주어진다면 데이터 셋의 테이블 정의, 물리적 위치, 비즈니스 관련 속성 뿐만아니라 시간에 따라 데이터가 변하는지 추적할 수 있습니다.

AWS Glue Data Catalog는 Apache Hive Metastore와 호환되며 Amazon EMR에서 실행되는 빅데이터 어플리케이션용 Apache Hive Metastore를 대채할 수 있습니다. AWS Glue Data Catalog를 Apache Hive Metastore로 사용하도록 EMR 클러스터를 설정하는 방법에 대한 자세한 내용을 보려면 여기를 클릭하십시오.

AWS Glue Data Catalog는 Amazon Athena, Amazon EMR 및 Amazon Redshift Spectrum과의 즉각적인 통합을 제공합니다. Glue Data Catalog에 테이블 정의를 추가하면 ETL에서 사용할 수 있으며 Amazon Athena, Amazon EMR 및 Amazon Redshift Spectrum에서 쉽게 쿼리 할 수 ​​있으므로 이러한 서비스간에 common view를 가질 수 있습니다.

### How can I customize the ETL code generated by AWS Glue?

AWS Glue’s ETL 스크립트 추천 시스템은 PySpark 코드를 생성합니다. Glue의 custom ETL 라이브러리를 활용하여 data source에 대한 액세스를 단순화하고 작업 실행을 관리합니다. 라이브러리에 대한 자세한 정보는 documentation에서 찾을 수 있습니다. AWS Glue의 custom library 를 사용하거나 AWS Glue 콘솔 스크립트 편집기를 통한 인라인 편집, 자동 생성 코드 다운로드, IDE에서 편집을 통해 임의의 Spark 코드를 Python(PySpark 코드)으로 작성하여 ETL 코드를 작성할 수 있습니다. 또한 Github 리포지토리에서 호스트되는 많은 샘플 중 하나에서 시작하여 해당 코드를 customize할 수 있습니다.

### When should I use AWS Glue vs. AWS Data Pipeline?

AWS Glue는 서버 없이 실행되는 Apache Spark 환경에서의 ETL 서비스를 제공합니다. 이는 ETL job에 집중하고 기본 컴퓨팅 리소스를 구성하고 관리하는것에 대해 걱정하지 않아도 됩니다. AWS Glue는 데이터 우선 접근 방식을 취하며 데이터 속성 및 데이터 조작에 중점을 두어 비즈니스 통찰력을 이끌어 낼 수있는 형태로 데이터를 변환 할 수 있도록 합니다.(AWS Glue takes a data first approach and allows you to focus on the data properties and data manipulation to transform the data to a form where you can derive business insights.) Amazon Athena 및 Amazon Redshift Spectrum을 통해 쿼리하는 것뿐만 아니라 ETL에 메타 데이터를 사용할 수있게 해주는 통합 데이터 카탈로그를 제공합니다.

AWS Data Pipeline은 데이터를 처리하는 코드 자체뿐만 아니라 실행 환경, 코드를 실행하는 컴퓨팅 리소스에 대한 액세스 및 제어 측면에서 유연성을 제공하는 관리형 오케스트레이션 서비스를 제공합니다. AWS Data Pipeline은 Amazon EC2 인스턴스 또는 Amazon EMR 클러스터에 직접 액세스 할 수 있도록 계정에 컴퓨팅 리소스를 실행합니다.

또한 AWS Glue ETL jobs는 PySpark 기반입니다. 만약 Apache Spark 이외의 엔진을 사용해야하는 경우 또는 Hive, Pig 등과 같은 다양한 엔진에서 실행되는 이기종 작업 집합을 실행하려는 경우 AWS Data Pipeline을 선택하는 것이 좋습니다.

자세한 정보는 [AWS Glue FAQ](https://aws.amazon.com/glue/faqs/) 를 참조하십시오.

## **ADDITIONAL RESOURCES**

### Amazon Athena:

- <https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/>
- <http://docs.aws.amazon.com/athena/latest/ug/convert-to-columnar.html>

### Redshift Spectrum
- https://aws.amazon.com/blogs/big-data/10-best-practices-for-amazon-redshift-spectrum/

### Serverless Analysis Architecture Blogs:
- <https://aws.amazon.com/blogs/big-data/derive-insights-from-iot-in-minutes-using-aws-iot-amazon-kinesis-firehose-amazon-athena-and-amazon-quicksight/>
- <https://aws.amazon.com/blogs/big-data/build-a-serverless-architecture-to-analyze-amazon-cloudfront-access-logs-using-aws-lambda-amazon-athena-and-amazon-kinesis-analytics/>

---
## License

This library is licensed under the Apache 2.0 License.
