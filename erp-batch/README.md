# 🔴 ERP-Batch 개선 진행

기존 코드가 유지보수를 진행하기에 너무 복잡하여(ErpSchedulerJob.java 경우 4000줄 이상) 역할과 책임을 분리하기 위한 의도로 진행

기존 erp-batch 프로젝트의 com.kakaovx.erp.batch.v2 내부에 코드 작성

## 🟠 역할 분리

### 🟢 batch(내부 로직)

batch에는 내부의 이벤트처리, 요금처리, 하드코딩을 위한 로직으로 역할을 부여했고 책임을 지도록 하였습니다.

### 🟢 erp(외부 호출)

외부 api 호출을 위한 factory

erp 데이터를 우리쪽 데이터로 가공하기 위한 adapter

를 정의하여 책임을 분리하였습니다.

## 🟠 Batch

### 🟢 ErpSchedulerService

erp를 호출하여 BookTime 정보에 대해 가져오는 Service로써 erp factory를 찾고 호출, timeout, 호출 횟수 체크등의 책임을 갖고 있습니다.

getErpBookTimeListByMultiThread 내부에 쓰레드의 pool size를 2로 고정하고 있는데 이는 테스트를 위한 사이즈로 이후 연동 골프장의 사용량에 따라 늘려서 적용해주시면 됩니다.

### 🟢 TimeInfoService

GolfInfoUse, BookTime, Purchase, BookName 등 조회한 타임들에 대해 내부적으로 처리하기 위한 데이터를 가져오기 위한 책임을 갖고 있습니다.

### 🟢 BookTimeService

api를 통해 조회된 타임을 저장하고 중복체크 등의 조회된 타임을 처리하는 책임을 갖고 있습니다.

### 🟢 HardCodingService

골프장별 당일예약, 카트비, 매입가, 내장인원 등 하드코딩을 위한 책임을 갖고 있습니다.

내부적으로 골프장별 하드코딩 전략을 HardCodingPicker를 통해 찾고 하드코딩이 있다면 하드코딩을 통해 로직이 진행되도록 처리하였습니다.

이 부분을 통해 기존에 골프장의 하드코딩을 적용시 전체 시스템에 영향을 주던 부분에서 해당 골프장만 영향이 있도록 분리할 수 있었습니다.

## 🟠 Erp

### 🟢 구성

erp내부에 erp코드로 패키지를 생성 후 내부에 common을 만들어줍니다.

common은 해당 erp 업체에 공통 처리 소스를 작성하며 내부적으로는 adapter, dto, factory, vo를 정의해주셔야 합니다.

erp 내부에 global 패키지 내부에는 adapter, factory, strategy, vo 등에 추상 class들과 interface를 정의해두었고 그외 필요한 class들을 정의해두었습니다.

### 🟢 Common

erp 업체에 공통 처리 소스를 작성하는 패키지로 기본값이 됩니다. 만약 이후 골프장 연동을 진행한다면 factory는 필수적으로 해당 common의 factory를 extend하여 사용하여야 합니다.

만약 내부적으로 수정이 필요한 경우 extend 후 override하여 사용하시면 됩니다.

그외 adapter, dto, vo 등도 필요에 따라 구현 후 factory 내부적으로 수정하여서 사용할 수 있습니다.

이 구조는 이후에 더 유연하게 설계할 수 있는 방향으로 고민해서 처리하는게 좋을거 같습니다!

### 🟢 adapter

erp 업체에서 전달받은 데이터를 우리쪽 데이터로 가공하기 위한 요소로 BookTimeAdapter를 상속하여 구현합니다.

내부적으로는 ErpConfig와 해당 erp 업체의 response 값인 vo를 필드로 두어 데이터 가공시에 사용할 수 있습니다.

### 🟢 dto

erp 업체에 요청시 필요한 body 값을 정의합니다.

get 방식에 파라미터라면 모델링 해두고 내부적으로 파라미터를 반환하도록 처리하셔도 되고 factory에서 내부적으로 사용하셔도 될것 같습니다.

### 🟢 vo.response

erp 업체에서 반환하는 내용을 모델링하여 사용하시면 됩니다.

## 🟠 연동 방법

### 🟢 common

연동할 erp 업체의 코드로 패키지를 생성 후 common에 필요한 dto, vo, adapter, factory를 정의합니다.

### 🟢 골프장 연동

골프장 seq 번호로 g{golfInfoSeq} 형태로 패키지를 생성 후 내부적으로 ErpFactory{golfInfoSeq} 형태로 factory를 정의하고 @Component로 bean으로 등록합니다.

### 🟢 하드코딩

하드코딩이 필요시 gi4.g29 를 예시로써 ErpHardCodingStrategy를 상속하여 구현하고 @Component로 bean으로 등록합니다.

# 🔴 데드락 개선

```
1. DB팀으로부터 데드락 자주 발생되는 쿼리를 로그와 함께 전달 받음.
2. 쿼리를 통해 실행되는 로직을 파악
3. 트랜잭션의 범위가 너무 크고 트랜잭션 내부에 많은 insert가 일어남
4. 그로인해 데드락이 발생할 확률이 큼
5. 총 3개의 부분으로 나누어서 적용함 (History, BookTime, BooktimeDisplay)
6. 오히려 데드락이 더 많이 발생함.
7. DB팀과 커뮤니케이션을 통해 그 이유에 대해 분석함
8. 3개로 나눈 쿼리가 insert into (select * from BookTime) 과 같은 형태임
9. insert가 되는 동안 select한 테이블은 write lock이 걸림 이로 인해 데드락이 발생확률이 높아지고 이전에는 큰 한덩이에서 작은 3덩이로 나뉘어 오히려 데드락이 더 많이 발생함
10. 해소를 위해 select를 외부에서 빼서 가져오고 insert는 가져온 select 데이터를 bulk 형식으로 넣는 방식으로 수정함.
```

---
기존 하루에 데드락 약 30건 발생
현재는 데드락 약 1~2건 발생 - 데드락 발생률 90프로를 해결.


# 🔴 isTest

## 🟠 test 구장 흐름

### 🟢 BatchScheduler

BatchScheduler.java 내에 scheduelerJob 메서드에서 jobStrategy.execute()를 통해 실행합니다.

### 🟢 JobStrategy

테스트인지 아닌지 구분하여 진행할 수 있도록 내부적으로 test구장과 일반 구장을 나누어 처리하고 있습니다.

일반 구장의 경우 isTest값이 false로 실행되며 테스트 구장은 isTest값을 true로 실행하도록 나누었습니다.

일반 구장과 테스트 구장을 나누는 기준은 ErpConfig 테이블의 batchVersion 컬럼의 값이 1이면 일반 구장이며 0이라면 테스트 구장입니다.

### 🟢 ErpSchedulerJob

실제 job 내에서 isTest값을 통해 분기처리하여 기존 로직과 테스트 로직을 구분하여 처리할 수 있습니다.