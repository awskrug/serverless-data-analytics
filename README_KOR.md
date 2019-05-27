#Building an End-to-End Serverless Data Analytics Solution on AWS


##Overview
이 실습에서는 [Amazon Athena](https://aws.amazon.com/ko/athena/)를 이용하여 Amazon S3에서 직접 데이터를 분석하고 [Amazon QuickSight](https://quicksight.aws/)에서 데이터를시각화하는 serverless architecture를 구축하려고 한다.  사용할 데이터 세트는 공개 데이터 세트로 2009년부터 2016년까지 뉴욕시의 노랜색과 녹색 택시에서 완료된 모든 주행기록과 2015년에서 2016년까지 상용 차량(FHV)의 모든 주행기록을 포함하는 데이터 세트다.  기록에는 픽업 및 드롭오프 날짜 / 시간, 픽업 및 드롭오프 장소, 주행거리, 항목별 요금, 요금 유형, 지불 유형,  운전자가 보고한 승객 수를 캡처하는 필드를 포함한다. 데이터 세트는 사전에 분할되어 있으며 CSV에서 Apache Parquet로 변환되어 있다.  실습의 첫번째 파트에서 당신은 Amazon Athena를 사용하여 Query와 같은 SQL을 구축하여 Amazon S3에서 직접적으로 형성되는 데이터 포멧을 Query화 하고 Query 성능을 비교하게 될 것이다. 두번째 파트에서는 Amazon Athena table을 사용하여 Amazon QuickSight의 데이터 소스로 Query를 생성하면 Amazon S3의 데이터 세트에서 시각화 및 의미있는 통찰력을 얻을 수 있다. Query의 성능을 최적화 하기위해 AWS Glue를 사용하는 Serverless ETL을 통합하기 위한 선택적 실습이 포함되어 있다. 또한  





![architecture-overview.png](https://s3.amazonaws.com/us-east-1.data-analytics/labcontent/reinvent2017content-abd313/architectureoveriew.PNG)

------

## AWS Console

### Verifying your region in the AWS Management Console




