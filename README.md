
# Table of contents

- [황명현 - 영화예매](#---)
  - [조직](#조직)
  - [서비스 시나리오](#서비스-시나리오)
  - [분석/설계](#분석설계)
    - [AS-IS 조직 (Horizontally-Aligned)]
    - [TO-BE 조직 (Vertically-Aligned)]
    - [Event Storming 결과]
    - [2차 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증]
    - [헥사고날 아키텍처 다이어그램 도출]
  - [구현:](#구현-)
    - [DDD 의 적용](#ddd-의-적용)
    - [Gateway 적용](#Gateway-적용)
    - [비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트](#비동기식-호출-과-Eventual-Consistency)
  - [운영](#운영)
    - [CI/CD 설정](#cicd설정)
    - [오토스케일 아웃](#오토스케일-아웃)
    - [무정지 재배포](#무정지-재배포)

# 조직
- 고객 : 정확한 정보를 입력, 오류를 최소화 한다(Core)
- 예약관리 : 고객에게 정확한 정보를 제공하고 예약 오류를 최소화 한다(Supporting)
- 고객관리 : 예약 현황에 대해 예매 고객에게 정확한 알람을 제공한다(외주 Supporting)

# 서비스 시나리오

- [기능적 요구사항]
1. 시스템은 현재 상영중인 영화 정보를 보여준다.
1. 고객은 보고 싶은 영화를 선택한다.
1. 시스템은 선택한 영화의 상영가능 영화관과 날짜, 시간를 보여준다.
1. 고객은 시간을 선택 후 예약현황을 보고 좌석을 선택한다.
1. 고객은 결제를 요청한다.
1. 시스템은 결제 기능을 제공한다.
1. 고객은 MyPage에서 예약된 정보를 조회할 수 있다.
1. 예약한 정보는 고객은 취소할 수 있다.
1. 예약 정보가 바뀔때 마다 시스템은 카톡으로 알림을 보낸다.

- [비기능적 요구사항]
1. 트랜잭션
    1. 결재 완료되어야 영화 관람이 가능하다. Sync 호출 
1. 장애격리
    1. 결재 중 시스템 오류시 결제 보류를 유도한다. Circuit breaker
1. 성능
    1. 예약상태 변경시마다 카톡등으로 알림을 줄 수 있어야 한다.  Event driven
    1. 고객이 최종 예약 상태를 확인할수 있어야 한다.  CQRS





# 분석/설계


## AS-IS 조직 (Horizontally-Aligned)
  ![image](https://user-images.githubusercontent.com/487999/79684144-2a893200-826a-11ea-9a01-79927d3a0107.png)

## TO-BE 조직 (Vertically-Aligned)
  ![image](https://user-images.githubusercontent.com/63028492/92347406-f88a4680-f10a-11ea-9711-d746f993854e.png)


## Event Storming 결과

### 이벤트 도출
![image](https://user-images.githubusercontent.com/63028492/92348656-ad723280-f10e-11ea-9e1c-c7319e55c7c3.png)


### 부적격 이벤트 탈락
![image](https://user-images.githubusercontent.com/63028492/92348662-b19e5000-f10e-11ea-99b4-e6a5da6df1e6.png)

### 어그리게잇으로 묶기
![image](https://user-images.githubusercontent.com/63028492/92348665-b4994080-f10e-11ea-8a49-5b9d9af6d213.png)

### 바운디드 컨텍스트로 묶기
![image](https://user-images.githubusercontent.com/63028492/92348757-fb873600-f10e-11ea-93c3-93b4131f178a.png)

### 완성된 모형
![image](https://user-images.githubusercontent.com/63028492/92347965-83b80c00-f10c-11ea-9be9-35ece3645d00.png)

### 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증

기능적 요구사항(검증)
![image](https://user-images.githubusercontent.com/63028492/92349384-df849400-f110-11ea-8d18-0dafafbb736c.png)

1. 시스템은 현재 상영중인 영화 정보를 보여준다.
1. 고객은 보고 싶은 영화를 선택한다.
1. 시스템은 선택한 영화의 상영가능 영화관과 날짜, 시간를 보여준다.
1. 고객은 시간을 선택 후 예약현황을 보고 좌석을 선택한다.
1. 고객은 결제를 요청한다.
1. 시스템은 결제 기능을 제공한다.
1. 고객은 MyPage에서 예약된 정보를 조회할 수 있다.
1. 예약한 정보는 고객은 취소할 수 있다.
1. 예약 정보가 바뀔때 마다 시스템은 카톡으로 알림을 보낸다.










 비기능적 요구사항(검증)
 ![image](https://user-images.githubusercontent.com/63028492/92349484-38542c80-f111-11ea-86d9-705d3982073f.png)

1. 트랜잭션
    1. 결재 완료되어야 영화 관람이 가능하다. Sync 호출 
1. 장애격리
    1. 결재 중 시스템 오류시 결제 보류를 유도한다. Circuit breaker
1. 성능
    1. 예약상태 변경시마다 카톡등으로 알림을 줄 수 있어야 한다.  Event driven
    1. 고객이 최종 예약 상태를 확인할수 있어야 한다.  CQRS











## 헥사고날 아키텍처 다이어그램 도출
    
![image](https://user-images.githubusercontent.com/63028492/92348104-fb863680-f10c-11ea-9391-be90872b9837.png)




# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍쳐에 따라 스프링부트로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다(각자의 포트 넘버는 8081~ 808n이다)

```
cd payment
mvn spring-boot:run 

cd reservation
mvn spring-boot:run 

cd gateway
mvn spring-boot:run
  
```

## DDD 의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하여 DDD 구현하고자 했다: (예시는 Payment 마이크로 서비스). 

```
package movie;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;

@Entity
@Table(name="PayProcess_table")
public class PayProcess extends AbstractEvent{

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String cardid;
    private String customerNm;
    private Integer qty;
    private String reservationiId;

    @PostPersist
    public void onPostPersist(){
        Payed payed = new Payed();
        BeanUtils.copyProperties(this, payed);
        payed.publish();
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public String getCardid() {
        return cardid;
    }

    public void setCardid(String cardid) {
        this.cardid = cardid;
    }
    public String getCustomerNm() {
        return customerNm;
    }

    public void setCustomerNm(String customerNm) {
        this.customerNm = customerNm;
    }
    public Integer getQty() {
        return qty;
    }

    public void setQty(Integer qty) {
        this.qty = qty;
    }
    public String getReservationiId() {
        return reservationiId;
    }

    public void setReservationiId(String reservationiId) {
        this.reservationiId = reservationiId;
    }
}



```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package movie;

import org.springframework.data.repository.PagingAndSortingRepository;

public interface PayProcessRepository extends PagingAndSortingRepository<PayProcess, Long>{
}
```

## Gateway 적용

Clous 환경에서는Clous 환경에서는 //서비스명:8080 에서 Gateway API가 작동해야함 application.yml 파일에 profile별 gateway 설정히였다

```
spring:
  profiles: docker
  cloud:
    gateway:
      routes:
        - id: reservation
          uri: http://reservation:8080
          predicates:
            - Path=/reservationProcesses/**,/movieLists/**
        - id: caCenter
          uri: http://caCenter:8080
          predicates:
            - Path= 
        - id: payment
          uri: http://payment:8080
          predicates:
            - Path=/payProcesses/** 

```

## 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트


결제가 이루어진 후에 고객에게 알림을 제공한다. 결제 승인이 되었다는 도메인 이벤트를 카프카로 송출한다 (Publish)
 
```
@PostPersist
public void onPostPersist(){
    MovieReserved movieReserved = new MovieReserved();
    BeanUtils.copyProperties(this, movieReserved);
    movieReserved.publish();
}


@StreamListener(KafkaProcessor.INPUT)
public void wheneverMovieReserved_Katalk(@Payload MovieReserved movieReserved){

    if(movieReserved.isMe()){
        System.out.println("##### listener Katalk : " + movieReserved.toJson());
    }
}

```



# 운영

## CI/CD 설정

각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 Azure(Dev)를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 azure-pipelines.yml 에 포함되었다.


### 오토스케일 아웃

- Customer/Delivery 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다.  
  
    Customer : CPU 사용량이 50프로를 넘어서면 replica를 10개까지 늘려준다(최소 1)  
    Delivery : CPU 사용량이 50프로를 넘어서면 replica를 5개까지 늘려준다(최소 1)
```
kubectl autoscale deploy customer --min=1 --max=10 --cpu-percent=50
kubectl autoscale deploy delivery --min=1 --max=5 --cpu-percent=50
```
- 부하 테스트를 위해 cpu-percent를 임시 조정
```
kubectl autoscale deploy customer --min=1 --max=10 --cpu-percent=10
```

- siege 를 통해 부하 Test
```
siege -c100 -t120S -r10 -v --content-type "application/json" 'http://customer:8080/orders POST {"product": "iceAmericano", "qty":1, "price":1000}'
```
- siege 를 통해 test를 진행하였으나 충분한 부하가 걸리지 않아 hpa는 확인하지 못하였다.
```
NAME                            READY   STATUS    RESTARTS   AGE
pod/customer-6865d894bc-jdkdk   1/1     Running   0          30s
pod/delivery-68b66976d6-5cxz9   1/1     Running   0          162m
pod/gateway-7784474fc-cnttl     1/1     Running   0          104m
pod/httpie                      1/1     Running   2          23h
```
```
Transactions:                  12858 hits
Availability:                 100.00 %
Elapsed time:                 119.46 secs
Data transferred:               3.42 MB
Response time:                  0.43 secs
Transaction rate:             107.63 trans/sec
Throughput:                     0.03 MB/sec
Concurrency:                   46.27
Successful transactions:       12858
Failed transactions:               0
Longest transaction:            2.44
Shortest transaction:           0.00
```



## 무정지 재배포

* readiness 설정
```
(azure-pipelines.yaml)

readinessProbe:
httpGet:
    path: /actuator/health
    port: 8080
initialDelaySeconds: 10
timeoutSeconds: 2
periodSeconds: 5
failureThreshold: 10
```

- seige 로 배포작업 직전에 워크로드를 모니터링 함.
```
siege -c100 -t120S -r10 --content-type "application/json" 'http://customer:8080/orders POST {"product": "iceAmericano", "qty":1, "price":1000}'


```

- Pipeline을 통한 새버전으로의 배포 시작 - UI
![image](https://user-images.githubusercontent.com/28293389/81762438-94ae9300-9507-11ea-8e94-8a761529ecac.png)

```
NAME                            READY   STATUS        RESTARTS   AGE
pod/customer-5d479469b9-5wlxp   0/1     Terminating   0          55m
pod/customer-85f968c94c-b254s   1/1     Running       0          87s
pod/customer-85f968c94c-k5pq4   1/1     Running       0          27s
pod/customer-85f968c94c-pgwd7   1/1     Running       0          55s
pod/delivery-78f4bb9f9b-7h42s   1/1     Running       0          54m
pod/gateway-675c9b584d-tr7cv    1/1     Running       0          17h
pod/httpie                      1/1     Running       1          18h
```


- seige 의 화면으로 넘어가서 Availability 확인
```
defaulting to time-based testing: 120 seconds
** SIEGE 3.0.8
** Preparing 100 concurrent users for battle.
The server is now under siege...
Lifting the server siege...      done.

Transactions:                  12976 hits
Availability:                 100.00 %
Elapsed time:                 119.67 secs
Data transferred:               3.45 MB
Response time:                  0.42 secs
Transaction rate:             108.43 trans/sec
Throughput:                     0.03 MB/sec
Concurrency:                   45.44
Successful transactions:       12976
Failed transactions:               0
Longest transaction:            2.29
Shortest transaction:           0.00
```

배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.


## 구현  

기존의 마이크로 서비스에 수정을 발생시키지 않도록 Inbund 요청을 REST 가 아닌 Event 를 Subscribe 하는 방식으로 구현. 기존 마이크로 서비스에 대하여 아키텍처나 기존 마이크로 서비스들의 데이터베이스 구조와 관계없이 추가됨. 

## 운영과 Retirement

Request/Response 방식으로 구현하지 않았기 때문에 서비스가 더이상 불필요해져도 Deployment 에서 제거되면 기존 마이크로 서비스에 어떤 영향도 주지 않음.

* [비교] 결제 (pay) 외부 서비스의 경우 API 변화나 Retire 시에 주문 (customer) 마이크로 서비스의 변경을 초래함:


## 시연용 프로세스

- 고객이 온라인으로 주문 내역을 만든다.
```
http POST http://customer:8080/orders product="IceAmericano" qty=1 price=2000
```
- 고객이 주문 내역으로 결제한다. (외부 API를 통한 동기식 진행)
```
http http://customer:8080/orderStatuses
```
- 주문 내역이 결제 되면 해당 내역을 매장에서 접수하거나 거절한다.
```
http http://delivery:8080/deliveries
```
- 매장에서 접수하면 커피를 제작하고 고객이 주문을 취소할 수 없다.
```
http PUT http://delivery:8080/deliveries/1 product="IceAmericano" qty=1 status="receive"
```
- 매장에서 거절하면 주문 내역이 취소되고 결제가 환불 된다.
```
http PUT http://delivery:8080/deliveries/1 product="IceAmericano" qty=1 status="reject"
```
- 고객이 주문 내역을 취소를 요청할 수 있다.
```
http PUT http://customer:8080/orders/1 product="IceAmericano" qty=1 status="order_cancel_request"
```

- 매장에서 주문 내역 취소 요청을 받으면 취소가 가능하다면 취소한다.
```
http http://delivery:8080/deliveries
```
- 주문 내역이 취소되면 결제가 환불 된다.
- 매장에서 커피가 제작 되면 고객이 주문 건을 픽업한다.
- 주문 진행 상태가 바뀔 때 마다 SMS로 알림을 보낸다.

## 참고 명령어
- httpie pod 접속
```
kubectl exec -it httpie bin/bash
```
- Order Status 확인
```
http http://customer:8080/orderStatuses
```
