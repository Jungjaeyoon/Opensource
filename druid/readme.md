# Druid 실전 가이드

## 1. 설치

### 1.1. Prerequisites
- #### Java 8 or higher
  ##### 자바 미설치시 참고

  ```
  > sudo add-apt-repository ppa:webupd8team/java
  > sudo apt-get update
  > sudo apt-get install oracle-java8-installer
  > sudo apt-get install oracle-java8-set-default
  ```
  ​
  ##### 설치 확인

  ```
  > java -version
  java version "1.8.0_171"
  > javac -version
  javac 1.8.0_171
  ```

- #### Unix-like OS
 ##### 설치 가능 OS
 ##### - Linux
 ##### - OS X
 ##### - windows는 지원하지 않음
 ##### - 본 과제 사용 시스템
 ```
 Distributor ID: Ubuntu
 Description:    Ubuntu 16.04.4 LTS
 Release:        16.04
 Codename:       xenial
 ```


- #### 8G of RAM
##### 64GB ram 사용

- #### 2 vCPUs
##### i7-4930K
- #### Dependency
##### Zookeeper

## 1.2 Install Druid

#### Download Druid
```
> curl -O http://static.druid.io/artifacts/releases/druid-0.12.1-bin.tar.gz
> tar -xzf druid-0.12.1-bin.tar.gz
> cd druid-0.12.1
```

#### Download zookeeper
```
> curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz -o zookeeper-3.4.10.tar.gz
> tar -xzf zookeeper-3.4.10.tar.gz
> cd zookeeper-3.4.10
> cp conf/zoo_sample.cfg conf/zoo.cfg
> ./bin/zkServer.sh start
```
#### Start Druid service
##### In druid derectory(~/druid-0.12.1)
```
> bin/init
```

![alt text](https://github.com/Jungjaeyoon/Opensource/blob/master/druid.PNG "druid")


##### 이후 드루이드 폴더에 log 폴더 등 몇몇 폴더 생성 확인
#### 동일 위치(/druid-0.12.1)에서 다음 명령어 실행
#### 각 명령어는 서로 다른 터미널에서 실행(프론트 실행)
```
> java `cat conf-quickstart/druid/historical/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/historical:lib/*" io.druid.cli.Main server historical
> java `cat conf-quickstart/druid/broker/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/broker:lib/*" io.druid.cli.Main server broker
> java `cat conf-quickstart/druid/coordinator/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/coordinator:lib/*" io.druid.cli.Main server coordinator
> java `cat conf-quickstart/druid/overlord/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/overlord:lib/*" io.druid.cli.Main server overlord
> java `cat conf-quickstart/druid/middleManager/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/middleManager:lib/*" io.druid.cli.Main server middleManager
```

#### 모든 서비스 실행 확인한다면 druid에 batch data 불러올 준비 끝~

## 1.3 Test
```
> curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/wikiticker-index.json localhost:8090/druid/indexer/v1/task

# if success
{"task":"index_hadoop_wikipedia_2013-10-09T21:30:32.802Z"}
```
### http://localhost:8090/console.html 에서 확인가능


## 1.4 Install Tranquility
```
> curl -O http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz
> tar -xzf tranquility-distribution-0.8.0.tgz
> cd tranquility-distribution-0.8.0
```
### 실시간 데이터 스트리밍을 위한 서비스
```
> bin/tranquility server -configFile <path_to_druid_distro>/conf-quickstart/tranquility/server.json
```

## 2. 사용

### 1.2 에서 실행한 Druid + 1.4의 Tranquility 를 실행
### 쿼리를 json을 통해 작성
---> 이 때 static(local file), streaming(using tranquility), hdfs 등 
### POST를 통해 데이터 입력
### 모니터링(default address)
#### -> localhost:8081
### 콘솔
#### -> localhost:8090/colsole.html

image 
![alt text](https://github.com/Jungjaeyoon/Opensource/blob/master/druid/controlpanel.png  "druid")

### 실제 수집을 위한 JSON 쿼리
```
{
   "dataSources" : [
      {
         "spec" : {
            "dataSchema" : {
               "dataSource" : "bittrex3",
               "metricsSpec" : [
                  {
                     "type" : "count",
                     "name" : "count"
                  },
                  
               ],
               "granularitySpec" : {
                  "segmentGranularity" : "hour",
                  "queryGranularity" : "none",
                  "type" : "uniform"
               },
               "parser" : {
                  "type" : "string",
                  "parseSpec" : {
                     "format" : "json",
                     "timestampSpec" : {
                        "column" : "TimeStamp",
                        "format" : "auto"
                     },
                     "dimensionsSpec" : {
                        "dimensions" : [
                           "MarketName"
                        ]
                     }
                  }
               }
            },
            "tuningConfig" : {
               "type" : "realtime",
               "windowPeriod" : "PT1M",
               "intermediatePersistPeriod" : "PT1 M",
               "maxRowsInMemory" : 75000
            }
         },
         "properties" : {
            "task.partitions" : "1",
            "task.replicants" : "1"
         }
      }
   ],
   "properties" : {
      "zookeeper.connect" : "localhost",
     "druid.discovery.curator.path" : "/druid/discovery",
    "druid.selectors.indexing.serviceName" : "druid/overlord", 
      "http.port" : "8200",
      "http.threads" : "40"
   }
}
```

## 보다 편한 사용을 위한 Pydruid
#### https://github.com/druid-io/pydruid

### Top N 쿼리 예시
```
top = query.topn(
    datasource='twitterstream',
    granularity='all',
    intervals='2014-03-03/p1d',  # utc time of 2014 oscars
    aggregations={'count': doublesum('count')},
    dimension='user_mention_name',
    filter=(Dimension('user_lang') == 'en') & (Dimension('first_hashtag') == 'oscars') &
           (Dimension('user_time_zone') == 'Pacific Time (US & Canada)') &
           ~(Dimension('user_mention_name') == 'No Mention'),
    metric='count',
    threshold=10
)

df = query.export_pandas()
print df

   count                 timestamp user_mention_name
0   1303  2014-03-03T00:00:00.000Z      TheEllenShow
1     44  2014-03-03T00:00:00.000Z        TheAcademy
2     21  2014-03-03T00:00:00.000Z               MTV
3     21  2014-03-03T00:00:00.000Z         peoplemag
4     17  2014-03-03T00:00:00.000Z               THR
5     16  2014-03-03T00:00:00.000Z      ItsQueenElsa
6     16  2014-03-03T00:00:00.000Z           eonline
7     15  2014-03-03T00:00:00.000Z       PerezHilton
8     14  2014-03-03T00:00:00.000Z     realjohngreen
9     12  2014-03-03T00:00:00.000Z       KevinSpacey
```
