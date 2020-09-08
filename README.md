
# Table of contents

- [황명현 - 영화예매](#---)
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
    - [실습 테스트](#실습 테스트)
  - [운영](#운영)
    - [CI/CD 설정](#cicd설정)
    - [오토스케일 아웃](#오토스케일-아웃)
    - [무정지 재배포](#무정지-재배포)


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

    - 도메인 서열 분리 
        - Core Domain:  예약 - 없어서는 안될 핵심 서비스이며, 연견 Up-time SLA 수준을 99.999% 목표, 배포주기는 app 의 경우 1주일 1회 미만
        - Supporting Domain:  알림 - Core Domain을 위한 서비스
        - General Domain:   결제 : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 



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


## 실습 테스트

```
고객이 영화를 예약한다(Publish)
```

![image](https://user-images.githubusercontent.com/63028492/92446551-f1d5ff00-f1f0-11ea-8e54-2495775d28d9.png)

```
예약된 정보를 결제한다
```

![image](https://user-images.githubusercontent.com/63028492/92446581-fd292a80-f1f0-11ea-9047-1953cab68225.png)
```
```



# 운영

## CI/CD 설정

각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 Azure(Dev)를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 azure-pipelines.yml 에 포함되었다.
  ![image](https://user-images.githubusercontent.com/63028492/92424007-0cdb4b80-f1be-11ea-86e4-bd66766fa773.png)

### 오토스케일 아웃

- Customer/Delivery 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다.  
  
    payment : CPU 사용량이 50프로를 넘어서면 replica를 10개까지 늘려준다(최소 1)  
    reservation : CPU 사용량이 50프로를 넘어서면 replica를 5개까지 늘려준다(최소 1)
```
kubectl autoscale deploy payment  --min=1 --max=10 --cpu-percent=15
kubectl autoscale deploy reservation --min=1 --max=5 --cpu-percent=50
```
- 부하 테스트를 위해 cpu-percent를 임시 조정
```
kubectl autoscale deploy payment --min=1 --max=10 --cpu-percent=10
```
- siege 를 통해 test를 진행하였으나 충분한 부하가 걸리지 않아 hpa는 확인하지 못하였다.
```
NAME                            READY   STATUS    RESTARTS   AGE
cacenter-57d97fd8bf-nqdtj       1/1     Running   0          81m
gateway-5d96d567db-78m68        1/1     Running   0          99m
httpie                          1/1     Running   0          97m
my-nginx-5b8c577c4b-s8cbp       1/1     Running   0          99m
payment-7649bf8875-fgmqg        1/1     Running   23         99m
payment-7649bf8875-pl9xq        1/1     Running   23         99m
payment-7649bf8875-vmplc        1/1     Running   23         99m
reservation-74d75f75d5-wq6vf    1/1     Running   0          82m
```
```

## 무정지 재배포

* readiness 설정
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

```

