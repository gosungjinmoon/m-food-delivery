# Lv3 통합실습 배달의 민족 마이크로서비스 분석/설계, 구현, 배포/운영

## 서비스 시나리오

### 기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다.
2. 고객이 선택한 메뉴에 대해 결제한다.
3. 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다
4. 상점주는 주문을 수락하거나 거절할 수 있다
5. 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다
6. 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다
7. 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다
8. 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다.
9. 고객이 주문상태를 중간중간 조회한다
10. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다


### 비기능적 요구사항
1. 장애격리
 - 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다 Async (event-driven), Eventual Consistency
 - 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시후에 하도록 유도한다 Circuit breaker, fallback
2. 성능
 - 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다 CQRS
 - 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다 Event driven


## 완성된 1차 모형
![image](https://user-images.githubusercontent.com/61446346/206143689-14f04447-700b-4ac0-822f-ca2c3ef64b0c.png)

### 시나리오 적용
![image](https://user-images.githubusercontent.com/61446346/206149719-cb7a2d68-5f6d-478e-995e-717b95c769c9.png)

1. 고객이 메뉴를 선택하여 주문한다. (ok)
2. 고객이 선택한 메뉴에 대해 결제한다.(ok)
6. 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다(ok)
9. 고객이 주문상태를 중간중간 조회한다(ok)

![image](https://user-images.githubusercontent.com/61446346/206151197-7b8c9b4e-0b3f-4f5b-b4e2-f07e6066ab06.png)
```
3. 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다 (ok)
4. 상점주는 주문을 수락하거나 거절할 수 있다 (ok)
5. 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다 (ok)
7. 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다 (ok)
```

![image](https://user-images.githubusercontent.com/61446346/206153621-7bedc111-ec8c-4209-80d1-177e95c6e122.png)
```
8. 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다.
10. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다
```

## L3 평가 체크포인트
## ■  Microservice Implementation
### 1. Saga (Pub / Sub)

  #### 구현 : Pub / Sub 구현 코드
  - ![image](https://user-images.githubusercontent.com/2777247/219011956-605a74c3-5923-4cb7-957e-9b208696b7dd.png)
  
  #### 실행
  - Order커맨드 실행
  - ![image](https://user-images.githubusercontent.com/61446346/205813102-0f9f9a12-7f77-495c-a055-af23e0da81e8.png)
  - Store에서 오더정보 확인
  - ![image](https://user-images.githubusercontent.com/61446346/205835532-0b51f886-871e-4151-afcc-720e605cf599.png)


  #### kafka 확인
  - ![image](https://user-images.githubusercontent.com/61446346/205813194-69878c3d-f958-4399-ae41-6200670551c3.png)


### 2. CQRS

  - 구현 : 오더주문시 orderView 정보를 생성한고, 각 단계전진시 orderStatus상태를 현행화 관리한다.
  - ![image](https://user-images.githubusercontent.com/61446346/205814117-7aa5d785-2d93-4d1a-90bd-8eb501648efe.png)
  -  CQRS 구현된 코드
  -  ![image](https://user-images.githubusercontent.com/2777247/219012947-a64463d1-daeb-43a2-9fe2-07f8412122be.png)
  

### 3. Compensation / Correlation

  #### 구현 : 오더주문커맨드 실행시 오더정보 kafka에 적재, 오더캔슬커맨드 실행시 오더정보를 삭제한다.
  - 오더주문 
  - ![image](https://user-images.githubusercontent.com/61446346/205846913-33451e38-0195-4ba1-a04e-4199dd09fd93.png)

  - 오더취소
  - ![image](https://user-images.githubusercontent.com/61446346/205847021-47ff372f-da6d-409c-9ec3-d19fe2859461.png)

  #### 실행 : 
  - 오더주문
  - ![image](https://user-images.githubusercontent.com/61446346/205847148-833f5e98-7df1-4639-b246-6c8c3b2dd445.png)

  #### kafka 확인 : 
  - 오더주문
  - ![image](https://user-images.githubusercontent.com/61446346/205847939-52f2d793-e957-4a17-883f-b6c008893146.png)

  - 오더취소
  - ![image](https://user-images.githubusercontent.com/61446346/205848028-f28981bb-3cec-48a1-babb-c0a153ea8248.png)

  - 오더정보 확인 : orderId = 5
  - ![image](https://user-images.githubusercontent.com/61446346/205848158-51867ac1-c3d2-4edf-8b55-ca709a3faa43.png)

## ■  Microservice Orchestration

### 4. Deploy to EKS Cluster

  #### 서비스 화인
  - ![image](https://user-images.githubusercontent.com/55925545/219265239-ecde6a67-7016-416b-94e6-e8c54d35797f.png)

### 5.Gateway & Service Router 설치
 #### 서비스 목록 확인
  - ![image](https://user-images.githubusercontent.com/55925545/219265350-af105614-10ab-4e7e-9b3c-21825d1f14d5.png)
 
### 6.Autoscale (HPA)
 #### autoscale 부하 발생 전 확인 
  - ![image](https://user-images.githubusercontent.com/55925545/219265798-06327d5d-5b4e-42fc-8774-13af1b19bb01.png)
 #### autoscale 부하 발생 후 확인 
  - ![image](https://user-images.githubusercontent.com/55925545/219265910-53338e9d-5754-47a7-acdb-b198ddfddaf6.png)
 #### Pod 증가 확인
  - ![image](https://user-images.githubusercontent.com/55925545/219265991-5aa1a9e5-167e-49ec-94ec-21777f9cd6f6.png)

