
> 목차
> ---
> 1. [분석 요건에 대한 정의](scenario_1.md#분석-요건에-대한-정의)
>    1. 실제 공유오피스 분석 화면
> 1. [데이터 준비](scenario_1.md#데이터-준비)
>    1. Dataset 1: 좌석 예약 데이터
>    1. Dataset 2: 좌석 메타 데이터
> 1. [데이터 처리 (Batch Pipeline)](scenario_1.md#데이터-처리-batch-pipeline)
>     1. [데이터파일 2개 조인하여 ETL작업 수행하기](scenario_1.md#데이터파일-2개-조인하여-etl작업-수행하기)
>         1. [워크플로우 생성 메뉴로 이동](scenario_1.md#생성-메뉴로-이동)
>         1. [데이터 로딩](scenario_1.md#데이터-로딩)
>         1. [필요한 컬럼 선택](scenario_1.md#필요한-컬럼-선택)
>         1. [데이터 정제](scenario_1.md#데이터-정제)
>         1. [통계](scenario_1.md#통계)
>         1. [join key 생성](scenario_1.md#join-key-생성)
>         1. [2개파일 조인](scenario_1.md#2개파일-조인)
>         1. [새로운 column 생성](scenario_1.md#새로운-column-생성)
>         1. [불필요한 column 삭제](scenario_1.md#불필요한-column-삭제)
>         1. [null 값 제거](scenario_1.md#null-값-제거)
>         1. [컬럼명 변경](scenario_1.md#컬럼명-변경)
>         1. [ICOS에 저장](scenario_1.md#ICOS에-저장)
>     1. [워크플로우 저장 및 실행](scenario_1.md#워크플로우-저장-및-실행)
>     1. [워크플로우 상태 확인](scenario_1.md#워크플로우-상태-확인)
>     1. [실행결과 확인](scenario_1.md#실행결과-확인)
> 1. [데이터 시각화 (Data Insight)](scenario_1.md#데이터-시각화-data-insight)
>     1. [데이터 셋 준비](scenario_1.md#데이터-셋-준비)
>         1. [데이터 셋 추가](scenario_1.md#데이터-셋-추가)
>     1. [차트 작성](scenario_1.md#차트-작성)
>         1. [데이터 셋 선택](scenario_1.md#데이터-셋-선택)
>         1. [필터 설정](scenario_1.md#필터-설정)
>         1. [스택 차트 작성](scenario_1.md#스택-차트-작성)
>         1. [파이 차트 작성](scenario_1.md#파이-차트-작성)
>         1. [대시보드 생성](scenario_1.md#대시보드-생성)
>     1. [DASHBOARD](scenario_1.md#DASHBOARD)

<br/><br/>

# 분석 요건에 대한 정의

> 약 25개의 SK 관계사들의 건물별 좌석 이용 현황을 분석한다.

<br/>

## 실제 공유오피스 분석 화면

- 회사별 일평균 좌석이용률


![ex_screenshot](./img/scenario1.png)

<br/><br/>

## 데이터 준비
> ICOS에 업로드된 데이터를 사용합니다.  
> 스토리지 : IBMOSC1146611-6  
> 버킷 : handson-bucket  

<br/>

(1) Dataset 1 : 좌석 예약 데이터  

| column명 | column 설명 | Value Example |
|---|:---:|---|
| `bldg` | 건물명 | SKC 본사, 서린 빌딩, ... |
| `desk_type` | 좌석 유형 | 일반좌석, 스탠딩좌석, ... |
| `desk_id` | 각 좌석의 ID | 3a27b3bf-d8de-11e8-975a-0a9726611f46 |
| `dt` | 예약 날짜 | 20190803 |
| `com` | 회사명 | SK C&C, SK이노베이션, … |
| `age` | 나이대 | 20대,30대,... |

<br/>

(2) Dataset 2 : 좌석 메타 데이터  

| column명 | column 설명 | Value Example |
|---|:---:|---|
| `bldg` | 건물명 | SKC 본사, 서린 빌딩, ... |
| `desk_type` | 좌석 유형 | 일반좌석, 스탠딩좌석, ... |
| `occupied` | 고정좌석 여부 | N, Y |
| `desk_id` | 각 좌석의 ID | 3a27b3bf-d8de-11e8-975a-0a9726611f46 |
| `dt` | 예약 날짜 | 20190803 |

<br/>

## 데이터 처리 (Batch Pipeline)

<br/>

### 데이터파일 2개 조인하여 ETL작업 수행하기

<br/>

 > **ETL flow 구성내용** : `ICOS에서 파일 불러오기` → `사용할 column만 선택` → `데이터 정제` → `통계` → `join key 생성` → `2개 파일 join` → `새로운 column 생성` → `불필요한 column 삭제` → `null 값 제거`→ `column명 rename` → `ICOS에 저장`

<br/>

#### 생성 메뉴로 이동

![ex_screenshot](./img/BP_start.png)

<br/>

#### 데이터 로딩

![ex_screenshot](./img/DataLoading_1027.png)

<br/>

`데이터 불러오기` 패널에서 `ICOS 불러오기` 노드를 캔버스 영역에 drag & drop하여 node 생성  
우측 property 패널  
- **file** : 불러오려는 ICOS 파일의 경로 지정
    - 좌석 예약 데이터의 경로 : IBMOSC1146611-6 스토리지에서 handson-bucket/DI/Dataset/desk_resv
    - 좌석 메타 데이터의 경로 : IBMOSC1146611-6 스토리지에서 handson-bucket/DI/Dataset/desk_info
- **option** : 데이터에 header가 없는 경우 false로 변경
- **schema** : `자동 파싱 열기` 클릭

<br/>

    파싱을 처음 수행하는 경우, [세션 생성] 팝업이 뜸 ex) 지금 [1st2063] 클러스터에 세션을 생성하시겠습니까?
    [예] 클릭
    [취소] 클릭해서 자동 파싱 창을 나갔다가 다시 [자동 파싱 열기] 클릭

<br/>

schema - `자동 파싱 열기` - sample data 탭에서 데이터 샘플을 확인<br/>
schema - `자동 파싱 열기` - column info 탭에서 컬럼명 변경<br/>

<br/>

    column info 탭에서 컬럼 인식이 안된 경우, 전체파싱 [+] 클릭

> 좌석 예약 데이터 : bldg, desk_type, desk_id, dt, com, age

`확인` 클릭 

     Note : 우측 property 패널 - schema 에서 직접 컬럼명 변경 가능

<br/>

동일한 방법으로 좌석 메타 데이터도 불러오기 & 컬럼명 변경  
> 좌석 메타 데이터 : bldg, desk_type, occupied, desk_id, dt  

<br/>

#### 필요한 컬럼 선택  

![ex_screenshot](./img/s1_select.png)

<br/>

`데이터 처리(기본)` 패널에서 `select` 노드 drag & drop하여 생성  
`ICOS 불러오기` 노드에서 `select` 노드로 연결

우측 property 패널  
- **col** checkbox에서 각각 아래 4가지 column 체크
    - 좌석 예약 데이터 : bldg, desk_id, dt, com
    - 좌석 메타 데이터 : bldg, occupied, desk_id, dt

<br/>

#### 데이터 정제

![ex_screenshot](./img/s1_distinct_filter.png)

<br/>

`데이터 처리(기본)` 패널에서 `distinct` 노드 drag & drop하여 좌석 예약 데이터에서 중복 예약 제거<br/>
좌석 예약 데이터의 `select` 노드에서 `distinct` 노드로 연결<br/><br/>
`데이터 처리(기본)` 패널에서 `filter` 노드 drag & drop하여 좌석 메타 데이터에서 고정 좌석 제외<br/>
좌석 메타 데이터의 `select` 노드에서 `filter` 노드로 연결<br/>
우측 property 패널  
- **col** : occupied
- **filterOption** : =:equal
- **filterValue** : 'N'  

<br/>

#### 통계

![ex_screenshot](./img/s1_agg.png)

`데이터 처리(기본)` 패널에서 `agg` 노드 drag & drop하여 좌석 예약 데이터에서 건물별 회사별 예약좌석수 계산  
`distinct` 노드에서 `agg` 노드로 연결<br/>
우측 property 패널  
- **aggcol** : bldg, com, dt (aggcol 우측 [+] 로 컬럼 추가)
- **target** : func count, col desk_id  

<br/>

`데이터 처리(기본)` 패널에서 `agg` 노드 drag & drop하여 좌석 메타 데이터에서 건물별 전체좌석수 계산  
`filter` 노드에서 `agg` 노드로 연결<br/>
우측 property 패널  
- **aggcol** : bldg, dt
- **target** : func count, col desk_id  

<br/>

#### join key 생성

![ex_screenshot](./img/s1_aql.png)


`데이터 처리(고급)` 패널에서 `SQL` 노드 drag & drop하여 좌석 예약 데이터에서 join key 생성  
좌석 예약 데이터의 `agg` 노드에서 `SQL` 노드로 연결<br/>
우측 property 패널  
- **query**  

```SQL
select bldg,dt,com,countdesk_id as resv, concat(bldg,'_',dt) as key
from default;
``` 

- **overwriteSchema** 체크 (SQL 결과로 변경된 데이터 적용)  

<br/>

`데이터 처리(고급)` 패널에서 `SQL` 노드 drag & drop하여 좌석 메타 데이터에서 join key 생성  
좌석  데이터의 `agg` 노드에서 `SQL` 노드로 연결<br/>
우측 property 패널  
- **query**  

```SQL
select countdesk_id as total, concat(bldg,'_',dt) as key
from default;
```

- **overwriteSchema** 체크 (SQL 결과로 변경된 데이터 적용)  

<br/>

#### 2개파일 조인

![ex_screenshot](./img/s1_join.png)

좌측의 `데이터 처리(기본)` 패널에서 `dataJoin` 노드 우측 캔버스에 drag & drop하여 생성  
`SQL` 노드 두개를 `dataJoin` 노드로 연결<br/>
우측 property패널  
- **col1** : 좌석 예약 데이터의 key 선택  
- **col2** : 좌석 메타 데이터의 key 선택  
- **how** : left_outer 선택 (col1,col2 순서가 바뀐 경우 right_outer 선택)  

<br/>

#### 새로운 column 생성

![ex_screenshot](./img/s1_withColumn.png)

`데이터 처리(기본)` 패널에서 `withColumn` 노드 drag & drop하여 좌석이용률 계산  
`dataJoin` 노드에서 `withColumn` 노드로 연결<br/>
우측 property 패널  
- **selectType** : column
- **col1** : resv_dt_~
- **operator** : /
- **col2** : total_dt_~
- **newColumn** : ratio  

<br/>

#### 불필요한 column 삭제

![ex_screenshot](./img/s1_drop.png)

`데이터 처리(기본)` 패널에서 `drop` 노드 drag & drop하여 불필요한 데이터 삭제  
`withColumn` 노드에서 `drop` 노드로 연결<br/>
우측 property 패널  
- **col** checkbox에서 아래 4가지 column 체크
    - 좌석 예약 데이터의 resv_df_~
    - 좌석 예약 데이터의 key_df_~
    - 좌석 메타 데이터의 total_df_~
    - 좌석 메타 데이터의 key_df_~  

<br/>

#### null 값 제거

![ex_screenshot](./img/s1_filter.png)

`데이터 처리(기본)` 패널에서 `filter` 노드 drag & drop하여 회사명이 '\N'이면 삭제 
`drop` 노드에서 `filter` 노드로 연결<br/>
우측 property 패널  
- **col** : com_df_~
- **filterOption** : !=:not equal
- **filterValue** : '\N'  

<br/>

#### 컬럼명 변경

![ex_screenshot](./img/s1_withColumnRenamed.png)

`데이터 처리(기본)` 패널에서 `withColumnRenamed` 노드 drag&drop하여 생성  
`filter` 노드에서 `withColumnRenamed` 노드로 연결<br/>
우측 property 패널  
- `전체 컬럼 불러오기` 아이콘을 통해 자동 파싱  

<br/>

#### ICOS에 저장

![ex_screenshot](./img/s1_export.png)

좌측 `데이터 내보내기` 패널에서 `ICOS 내보내기` 노드 drag & drop 하여 생성  
`withColumnRenamed` 노드에서 `ICOS 내보내기` 노드로 연결  

<br/>

우측 property 패널  
- **path**의 browse 아이콘을 클릭하여 열리는 팝업에서 저장할 디렉토리 위치 선택 후 확인 클릭 (ex. ~/DI/results)  
- **folder**에 생성할 디렉토리명 입력 (ex. edu01_scenario1)  

<br/>

### 워크플로우 저장 및 실행

캔버스 상단에 워크플로우 이름 수정 (ex. 회사별 일평균 좌석이용률)

<br/>

캔버스에서 **ETL** 클릭 후 **appName** 입력 (ex. scenario1)

![ex_screenshot](./img/s1_final.png)

<br/>

    Note : source 우측 [Source Viewer] 아이콘을 클릭하면 변환된 pyspark 코드 확인 가능

<br/>

`저장` 클릭<br/>
`실행` 클릭  

<br/>

### 워크플로우 상태 확인

![ex_screenshot](./img/s1_stateCheck.png)

`워크플로우 관리` 화면으로 이동하여 상태 확인<br/>
이미 성공으로 종료되었거나 오류가 발생하여 종료된 경우 INACTIVE 상태<br/>
상세정보 컬럼의 `인스턴스 목록` 아이콘 클릭하면 이전 실행 이력 관리 가능<br/>

<br/>

### 실행결과 확인

![ex_screenshot](./img/s1_resultCheck.png)

`브라우저` 탭 > `ICOS` 메뉴로 이동<br/>
지정했던 ICOS directory 위치로 이동하여 내보내기로 생성한 디렉토리 하단의 part-00000-~파일 클릭<br/>
우측 상단의 `다운로드` 버튼 클릭하여 파일내용 확인<br/>

<br/><br/>

## 데이터 시각화 (Data Insight) 

<br/>

### 데이터 셋 준비

<br/>

#### 데이터 셋 추가
`데이터 셋 관리` 페이지에서 `데이터 셋 추가`를 클릭하면 추가 페이지로 이동합니다.<br/>

<br/>

![ex_screenshot](./img/addDataset.png)

<br/>

DataInsight 분석 및 시각화를 위해 MariaDB, ICOS, HIVE 등 다양한 종류의 데이터 셋을 추가할 수 있습니다.<br/>
지원하는 Data Source 타입<br/>
① 로컬 파일 : csv 타입 등 로컬에 있는 파일을 업로드<br/>
② MariaDB : MariaDB 테이블 연동하여 데이터 업로드<br/>
③ AWS RDS: AWS RDS 테이블 연동하여 데이터 업로드<br/>
④ HIVE : 하둡 클러스터의 데이터를 hive를 통하여 업로드<br/>
⑤ ICOS : IBM 오브젝트 스토리지의 파일을 업로드<br/>
⑥ MySQL : MySQL 테이블을 연동하여 데이터 업로드<br/>

<br/>

> 위에서 Batch Pipeline을 통해 실행한 결과 파일을 사용합니다.<br/>
> 스토리지 : IBMOSC1146611-6<br/>
> 버킷 : handson-bucket<br/>
> 위에서 실행한 결과 파일 경로 ex) DI/results/edu01_scenario1/part_00000-~.csv

`데이터 셋 추가` 클릭 후 `ICOS` 선택<br/>
**1단계_ICOS** 에서 스토리지 선택<br/>
**2단계_Bucket** 버킷 선택<br/>
`스키마 편집` 클릭<br/>
**그룹** 선택<br/>
`확인` 클릭

<br/>

![ex_screenshot](./img/s1_schema_1027.png)

<br/>

데이터 셋 이름 우측 `수정` 아이콘 클릭하여 그룹 설정 및 데이터 셋 이름 변경<br/>
- part-00000-~ -> 회사별 일평균 좌석이용률<br/>


각 컬럼의 좌측 `필드명 변경` 아이콘을 클릭하여 컬럼명 변경<br/>
- **bldg** -> 건물명<br/>
- **dt** -> 날짜<br/>
- **com** -> 회사명<br/>
- **ratio** -> 좌석이용률<br/>

**좌석이용률**의 데이터 타입을 Float으로 변경<br/>

데이터 셋 이름 우측 `저장` 아이콘 클릭<br/>
_새로운 분석 작업 페이지로 이동 하겠습니까?_ 라는 팝업이 뜨면 `이동`하거나 `아니요`하고 다음 단계<br/>

* `아니요`를 클릭한 경우, [필터 설정](scenario_1.md#필터-설정) 으로 이동하게 됨

<br/>

### 차트 작성

<br/>

#### 데이터 셋 선택
`분석 작업 관리` 페이지에서 `분석 작업 생성`을 클릭하면 추가 페이지로 이동합니다.<br/>
차트 작성할 데이터 셋 `적용` 아이콘 클릭<br/>

![ex_screenshot](./img/s1_selectDataset.png)

<br/>

`분석 작업 생성` 팝업이 뜨면 **그룹** 지정 및 **분석 작업 제목** 입력<br/>
`저장` 클릭<br/>

<br/>

![ex_screenshot](./img/makeChart.png)

<br/>

#### 필터 설정

좌측 `필터` 목록에서 `(+)` 버튼으로 필터 추가<br/>
필터의 **이름** 입력<br/>

- 이름 없음 -> 날짜


해당 필터의 `수정` 아이콘 클릭<br/>
필터 이름 오른쪽에 위치한 `[+]` 버튼으로 필터 조건 입력<br/>


- 첫번째 콤보박스에서 **날짜** 선택
- 두번째 콤보박스에서 **=** 선택
- 세번째 박스에 **20190820** 입력
<br/>

    Note : 필터 왼쪽 체크박스를 통해 각 필터의 적용/미적용에 따른 차트 변화를 실시간으로 확인 가능

<br/>

필터 이름 왼쪽에 위치한 `<` 버튼 클릭 <br/>

<br/>

#### 스택 차트 작성

![ex_screenshot](./img/s1_makeChart_1027.png)

`차트 생성` 버튼 클릭<br/><br/>

좌측 `그래프 선택` 목록에서 `컬럼 스택 차트` 선택<br/>
`필드값` 입력<br/>

- **Axis** 콤보박스에서 **회사명** 선택<br/>
- **Value** 콤보박스에서 **좌석이용률** 선택<br/>
- **Flag** 콤보박스에서 **SUM** 선택
<br/>

    Note : Flag를 선택하지 않으면, Value값이 Integer나 Float면 SUM, String인 경우 COUNT하도록 설정되어 있다

<br/>

![ex_screenshot](./img/s1_makeChart1_1027.png)
  
`저장` 클릭<br/>

<br/>

#### 파이 차트 작성

`차트 생성` 버튼 클릭<br/>

좌측 `그래프 선택` 목록에서 `파이 차트` 선택<br/>
`필드값` 입력<br/>

- **Grouping** 콤보박스에서 **회사명** 선택<br/>
- **Value** 콤보박스에서 **좌석이용률** 선택
<br/>

![ex_screenshot](./img/s1_makeChart2_1027.png)

`저장` 클릭<br/>

<br/>

#### 대시보드 생성

`대시보드 생성` 버튼 클릭<br/>

`대시보드 생성` 팝업이 뜨면 **대시보드 제목** 입력 후 `저장` 버튼 클릭<br/>


<br/>

### DASHBOARD

대시보드는 작업했던 분석 결과들을 보관하고 확인할 수 있는 공간입니다.<br/>
분석 작업 만들기에서 대시보드 생성 버튼을 클릭하여 작업 결과물들을 대시보드에 보관할 수 있습니다<br/>

<br/>

위에서 생성한 대시보드 클릭

![ex_screenshot](./img/dashboard.png)

<br/>

`대시보드 이미지로 내보내기` 버튼 클릭<br/>

![ex_screenshot](./img/dashboard_img_1027.png)

<br/>

다운받은 이미지 확인<br/>


