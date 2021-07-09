# myCafe
*****

## 서비스 시나리오

#### 기능적 요구사항
1. 고객이 음료를 주문한다
2. 고객이 결제한다
3. 결제가 되면 주문 내역이 직원에게 전달된다
4. 직원은 주문내역을 확인하여 음료를 제조한다
5. 고객이 주문을 취소할 수 있다
6. 주문이 취소되면 음료를 취소한다.
7. 음료가 취소되면 결제를 취소한다.
8. 고객이 주문상태를 조회한다.
9. 주문상태가 바뀔 때 마다 알림을 보낸다.
*****

#### 비기능적 요구사항    
1. 트랜잭션 
  - 결제가 되지 않은 주문건은 아예 거래가 성립되지 않아야 한다. Sync 호출
  - 주문이 취소되어도 바리스타가 접수하여 음료제조를 시작한 주문인 경우 주문 취소는 원복되어야 한다. Saga(보상 트랜잭션)

2. 장애격리 
  - 음료제조 기능이 수행되지 않더라도 주문은 받을 수 있어야 한다. Async (event-driven), Eventual Consistency
  - 결제시스템이 과중되면 결제를 받지 않고 결제를 잠시 후에 하도록 유도한다. (Circuit breaker, fallback)

3. 성능
  - 고객이 자주 주문상태를 확인할 수 있어야 한다. (CQRS)
  - 주문상태 변경 시 고객에게 알림을 줄 수 있어야 한다. (Event Driven)
  
*****

#### 체크포인트
- 분석설계 (40)
  - 이벤트스토밍: 
    - 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
    - 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
    - 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
    - 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?    

  - 서브 도메인, 바운디드 컨텍스트 분리
    - 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
      - 적어도 3개 이상 서비스 분리
    - 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
    - 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
  - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처 
    - 업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
    - Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
    - 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
    - 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
    - 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?

  - 헥사고날 아키텍처
    - 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?
    
- 구현 (35)
  - [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?
    - Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
    - [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
    - 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
  - Request-Response 방식의 서비스 중심 아키텍처 구현
    - 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
    - 서킷브레이커를 통하여  장애를 격리시킬 수 있는가?
  - 이벤트 드리븐 아키텍처의 구현
    - 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
    - Correlation-key:  각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
    - Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
    - Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
    - CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?

  - 폴리글랏 플로그래밍
    - 각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
    - 각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?
  - API 게이트웨이
    - API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
    - 게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
- 운영(25)
  - SLA 준수
    - 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
    - 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
    - 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
    - 모니터링, 앨럿팅: 
  - 무정지 운영 CI/CD (10)
    - Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명 
    - Contract Test :  자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?
*****


## 분석설계
*****

### Event Storming 결과
MSAEz 로 모델링한 이벤트스토밍 결과: http://www.msaez.io/#/storming/lQMZGUshp9WP1apR2496YcS40332/mine/83bc10e4ac905edc72761db670852eae
#### 이벤트 도출
![event01](https://user-images.githubusercontent.com/26791027/125020025-086c1180-e0b3-11eb-86bc-669cbb5cf8be.PNG)
### 부적격 이벤트 탈락
![event02](https://user-images.githubusercontent.com/26791027/125020119-39e4dd00-e0b3-11eb-9735-e254d6b76ef3.PNG)

```
- 과정중 도출된 잘못된 도메인 이벤트들을 걸러내는 작업을 수행함
    - 주문시>메뉴카테고리선택됨, 주문시>메뉴검색됨 :  UI 의 이벤트이지, 업무적인 의미의 이벤트가 아니라서 제외
```
#### 액터, 커맨드 부착하여 읽기 좋게
![2](https://user-images.githubusercontent.com/26791027/125021973-a6ada680-e0b6-11eb-886f-4420ffa7b397.PNG)
#### 어그리게잇으로 묶기
![3](https://user-images.githubusercontent.com/26791027/125021996-b4632c00-e0b6-11eb-986a-69f66da9f879.PNG)
#### 바운디드 컨텍스트로 묶기
![4](https://user-images.githubusercontent.com/26791027/125022263-3bb09f80-e0b7-11eb-81e7-3183af0ddfa4.PNG)
#### 폴리시 부착 (괄호는 수행주체, 폴리시 부착을 둘째단계에서 해놔도 상관 없음. 전체 연계가 초기에 드러남)
![5](https://user-images.githubusercontent.com/26791027/125022417-9cd87300-e0b7-11eb-8048-6fe7f9120abf.PNG)

#### 기능적/비기능적 요구사항을 커버하는지 검증
![11](https://user-images.githubusercontent.com/26791027/125022941-bd54fd00-e0b8-11eb-9d59-f293ef3dd272.png)

```
- 고객이 주문한다 (ok)
- 고객이 결제한다 (ok)
- 주문이 되면 직원에게 전달된다 (ok)
- 상점주인이 확인하여 접수 후 제조한다 (ok)
```
![222](https://user-images.githubusercontent.com/26791027/125023076-0f961e00-e0b9-11eb-9977-7d4f847fc45d.png)

```
- 고객이 주문을 취소할 수 있다 (ok)
- 주문이 취소되면 결제가 취소된다 (ok)
- 고객이 주문상태를 중간중간 조회한다 (ok) 
- 주문상태가 바뀔 때 마다 알림을 보낸다 (ok)
```

### Event Storming 모델 image
![Event Storming](https://user-images.githubusercontent.com/26791027/125021587-f0e25800-e0b5-11eb-88f4-6368abd69fb0.PNG)


### 헥사고날 아키텍처 다이어그램 도출

![헥사고날](https://user-images.githubusercontent.com/26791027/125020546-0191ce80-e0b4-11eb-9930-77fcdb9f1b24.PNG)

## 구현
*****

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현하였다. 
구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd order
mvn spring-boot:run

cd payment
mvn spring-boot:run 

cd drink
mvn spring-boot:run  

cd customercneter
mvn spring-boot:run
```
## DDD의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하였다 (예시는 Order  마이크로 서비스)
```
package cafeteria;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.PostPersist;
import javax.persistence.PostUpdate;
import javax.persistence.Table;

import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;

import cafeteria.external.Payment;
import cafeteria.external.PaymentService;

@Entity
@Table(name="ORDER_MANAGEMENT")
public class Order {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String phoneNumber;
    private String productName;
    private Integer qty;
    private Integer amt;
    private String status = "Ordered";

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    
    public String getPhoneNumber() {
    	return phoneNumber;
    }
    public void setPhoneNumber(String phoneNumber) {
    	this.phoneNumber = phoneNumber;
    }
    
    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        this.productName = productName;
    }
    public Integer getQty() {
        return qty;
    }

    public void setQty(Integer qty) {
        this.qty = qty;
    }
    public Integer getAmt() {
        return amt;
    }

    public void setAmt(Integer amt) {
        this.amt = amt;
    }
    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 별도 처리 없이 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package cafeteria;

import org.springframework.data.repository.PagingAndSortingRepository;

public interface OrderRepository extends PagingAndSortingRepository<Order, Long>{
}

```
- 적용 후 REST API 의 테스트

order 서비스의 주문처리

![001](https://user-images.githubusercontent.com/26791027/124974095-5ce99f80-e067-11eb-8bec-2e87920e9053.PNG)

payment 조회

![002](https://user-images.githubusercontent.com/26791027/124974346-a76b1c00-e067-11eb-9a64-d27a28c4b68c.PNG)
- drink 서비스의 처리
![003](https://user-images.githubusercontent.com/26791027/124974755-23656400-e068-11eb-91f5-e6c5baa5d1be.PNG)

customercenter 서비스 확인



## API Gateway
- API Gateway를 통하여 동일 진입점으로 진입하여 각 마이크로 서비스를 접근할 수 있다. 
```
# application.yml

spring:
  profiles: docker
  cloud:
    gateway:
      routes:
        - id: order
          uri: http://order:8080
          predicates:
            - Path=/orders/** 
        - id: payment
          uri: http://payment:8080
          predicates:
            - Path=/payments/** 
        - id: drink
          uri: http://drink:8080
          predicates:
            - Path=/drinks/**,/orderinfos/**
        - id: customercenter
          uri: http://customercenter:8080
          predicates:
            - Path= /mypages/**

# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: gateway
  labels:
    app: gateway
spec:
  type: LoadBalancer
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    app: gateway
    
$ kubectl get svc
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP                                                                  PORT(S)          AGE
customercenter   ClusterIP      10.100.52.95     <none>                                                                       8080/TCP         9h
drink            ClusterIP      10.100.136.6     <none>                                                                       8080/TCP         9h
gateway          LoadBalancer   10.100.164.152   a6826d83b5c8e4f5dad7129c7cdf0ded-93964597.ap-southeast-2.elb.amazonaws.com   8080:30109/TCP   9h
order            ClusterIP      10.100.197.15    <none>                                                                       8080/TCP         9h
payment          ClusterIP      10.100.242.153   <none>         

```
![001](https://user-images.githubusercontent.com/26791027/124976058-bc48af00-e069-11eb-9de9-16ec5a4451e8.PNG)



## 폴리글랏 퍼시스턴스
*****

## 폴리글랏 프로그래밍
*****

## 동기식 호출 과 Fallback 처리
*****
분석단계에서의 조건 중 하나로 order->payment 간의 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리하기로 하였다. 
호출 프로토콜은 이미 앞서 Rest Repository 에 의해 노출되어있는 REST 서비스를 FeignClient 를 이용하여 호출하도록 한다.

- 결제서비스를 호출하기 위하여 Stub과 (FeignClient) 를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현
```
package cafeteria.external;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient(name="payment", url="${feign.client.payment.url}")
public interface PaymentService {

    @RequestMapping(method= RequestMethod.POST, path="/payments")
    public void pay(@RequestBody Payment payment);

}
```
- 음료 주문 직후(@PostPersist) 결제를 요청하도록 처리
```
    @PostPersist
    public void onPostPersist(){

        Payment payment = new Payment();
        payment.setOrderId(this.id);
        payment.setPhoneNumber(this.phoneNumber);
        payment.setAmt(this.amt);
        
        OrderApplication.applicationContext.getBean(PaymentService.class).pay(payment);
    }
```
- 동기식 호출에서는 호출 시간에 따른 타임 커플링이 발생하며, 결제 시스템이 장애가 나면 주문도 못받는다는 것을 확인

```
# 결제 (payment) 서비스를 잠시 내려놓음
$ kubectl delete deploy payment
deployment.apps "payment" deleted

#주문처리

root@siege-5b99b44c9c-8qtpd:/# http http://order:8080/orders phoneNumber="01012345679" productName="coffee" qty=3 amt=5000
HTTP/1.1 500 
Connection: close
Content-Type: application/json;charset=UTF-8
Date: Sat, 20 Feb 2021 14:39:23 GMT
Transfer-Encoding: chunked
{
    "error": "Internal Server Error",
    "message": "Could not commit JPA transaction; nested exception is javax.persistence.RollbackException: Error while committing the transaction",
    "path": "/orders",
    "status": 500,
    "timestamp": "2021-02-20T14:39:23.185+0000"
}

#결제서비스 재기동
$ kubectl apply -f deployment.yml
deployment.apps/payment created

#주문처리

root@siege-5b99b44c9c-8qtpd:/# http http://order:8080/orders phoneNumber="01012345679" productName="coffee" qty=3 amt=5000
HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Sat, 20 Feb 2021 14:51:42 GMT
Location: http://order:8080/orders/6
Transfer-Encoding: chunked

{
    "_links": {
        "order": {
            "href": "http://order:8080/orders/6"
        },
        "self": {
            "href": "http://order:8080/orders/6"
        }
    },
    "amt": 5000,
    "createTime": "2021-02-20T14:51:40.580+0000",
    "phoneNumber": "01012345679",
    "productName": "coffee",
    "qty": 3,
    "status": "Ordered"
}

```
- 또한 과도한 요청시에 서비스 장애가 도미노 처럼 벌어질 수 있다. 

## 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트
*****
결제가 이루어진 후에 상점시스템으로 이를 알려주는 행위는 동기식이 아니라 비 동기식으로 처리하여 상점 시스템의 처리를 위하여 결제주문이 블로킹 되지 않아도록 처리한다.
- 결제이력에 기록을 남긴 후에 곧바로 결제승인이 되었다는 도메인 이벤트를 카프카로 송출한다 (Publish)
```
package cafeteria;

@Entity
@Table(name="Payment")
public class Payment {

 :
    @PostPersist
    public void onPostPersist(){
        PaymentApproved paymentApproved = new PaymentApproved();
        BeanUtils.copyProperties(this, paymentApproved);
        paymentApproved.publishAfterCommit();

    }
}
```
- 음료 서비스에서는 Ordered 이벤트에 대해서 이를 수신하여 자신의 정책을 처리하도록 PolicyHandler 를 구현
```
@StreamListener(KafkaProcessor.INPUT)
    public void wheneverOrdered_(@Payload Ordered ordered){

        if(ordered.isMe()){
            log.info("##### listener  : " + ordered.toJson());
            
            List<Drink> drinks = drinkRepository.findByOrderId(ordered.getId());
            for(Drink drink : drinks) {
           	drink.setPhoneNumber(ordered.getPhoneNumber());
            	drink.setProductName(ordered.getProductName());
               	drink.setQty(ordered.getQty());
               	drinkRepository.save(drink);
            }
        }
    }

```
- 음료 시스템은 주문/결제와 완전히 분리되어있으며, 이벤트 수신에 따라 처리되기 때문에, 음료시스템이 유지보수로 인해 잠시 내려간 상태라도 주문을 받는데 문제 없음
```
# 음료 서비스 (drink) 를 잠시 내려놓음
$ kubectl delete deploy drink
deployment.apps "drink" deleted

#주문처리
root@siege-5b99b44c9c-8qtpd:/# http http://order:8080/orders phoneNumber="01012345679" productName="coffee" qty=3 amt=5000
HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Sat, 20 Feb 2021 14:53:25 GMT
Location: http://order:8080/orders/7
Transfer-Encoding: chunked

{
    "_links": {
        "order": {
            "href": "http://order:8080/orders/7"
        },
        "self": {
            "href": "http://order:8080/orders/7"
        }
    },
    "amt": 5000,
    "createTime": "2021-02-20T14:53:25.115+0000",
    "phoneNumber": "01012345679",
    "productName": "coffee",
    "qty": 3,
    "status": "Ordered"
}
#음료 서비스 기동
kubectl apply -f deployment.yml
deployment.apps/drink created

#음료등록 확인

root@siege-5b99b44c9c-8qtpd:/# http http://drink:8080/drinks/search/findByOrderId?orderId=7
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Sat, 20 Feb 2021 14:54:14 GMT
Transfer-Encoding: chunked

{
    "_embedded": {
        "drinks": [
            {
                "_links": {
                    "drink": {
                        "href": "http://drink:8080/drinks/4"
                    },
                    "self": {
                        "href": "http://drink:8080/drinks/4"
                    }
                },
                "createTime": "2021-02-20T14:53:25.194+0000",
                "orderId": 7,
                "phoneNumber": "01012345679",
                "productName": "coffee",
                "qty": 3,
                "status": "PaymentApproved"
            }
        ]
    },
    "_links": {
        "self": {
            "href": "http://drink:8080/drinks/search/findByOrderId?orderId=7"
        }
    }
}

```

## Saga Pattern / 보상 트랜잭션
*****
직원이 음료를 접수하기 전에만 주문 취소가 가능하다. 만약 음료가 접수된 후에 취소할 경우에 보상 트랜잭션을 통해 취소를 원복한다.
주문 취소는 Saga Pattern으로 만들어져 있어, 직원이 음료를 이미 접수했을 경우에 취소 실패를 Event로 Publish하고 Order 서비스에서 취소 실패 Event를 Subscribe하여 주문 취소를 원복한다.
```
# 주문
root@siege-5b99b44c9c-8qtpd:/# http http://order:8080/orders/5
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Sat, 20 Feb 2021 08:58:19 GMT
Transfer-Encoding: chunked
{
    "_links": {
        "order": {
            "href": "http://order:8080/orders/5"
        },
        "self": {
            "href": "http://order:8080/orders/5"
        }
    },
    "amt": 100,
    "createTime": "2021-02-20T08:51:17.441+0000",
    "phoneNumber": "01033132570",
    "productName": "coffee",
    "qty": 2,
    "status": "Ordered"
}

# 결제 상태 확인 
root@siege-5b99b44c9c-8qtpd:/# http http://payment:8080/payments/search/findByOrderId?orderId=5
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Sat, 20 Feb 2021 08:58:54 GMT
Transfer-Encoding: chunked

{
    "_embedded": {
        "payments": [
            {
                "_links": {
                    "payment": {
                        "href": "http://payment:8080/payments/5"
                    },
                    "self": {
                        "href": "http://payment:8080/payments/5"
                    }
                },
                "amt": 100,
                "createTime": "2021-02-20T08:51:17.452+0000",
                "orderId": 5,
                "phoneNumber": "01033132570",
                "status": "PaymentApproved"
            }
        ]
    },
    "_links": {
        "self": {
            "href": "http://payment:8080/payments/search/findByOrderId?orderId=5"
        }
    }
}

# 음료 상태 확인
root@siege-5b99b44c9c-8qtpd:/# http http://drink:8080/drinks/search/findByOrderId?orderId=5                              
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Sat, 20 Feb 2021 08:52:14 GMT
Transfer-Encoding: chunked

{
    "_embedded": {
        "drinks": [
            {
                "_links": {
                    "drink": {
                        "href": "http://drink:8080/drinks/5"
                    },
                    "self": {
                        "href": "http://drink:8080/drinks/5"
                    }
                },
                "createTime": "2021-02-20T08:51:17.515+0000",
                "orderId": 5,
                "phoneNumber": "01033132570",
                "productName": "coffee",
                "qty": 2,
                "status": "PaymentApproved"
            }
        ]
    },
    "_links": {
        "self": {
            "href": "http://drink:8080/drinks/search/findByOrderId?orderId=5"
        }
    }
}

# 음료 접수
root@siege-5b99b44c9c-8qtpd:/# http patch http://drink:8080/drinks/5 status="Receipted"
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Sat, 20 Feb 2021 08:53:29 GMT
Transfer-Encoding: chunked
{
    "_links": {
        "drink": {
            "href": "http://drink:8080/drinks/5"
        },
        "self": {
            "href": "http://drink:8080/drinks/5"
        }
    },
    "createTime": "2021-02-20T08:51:17.515+0000",
    "orderId": 5,
    "phoneNumber": "01033132570",
    "productName": "coffee",
    "qty": 2,
    "status": "Receipted"
}

# 주문 취소
root@siege-5b99b44c9c-8qtpd:/# http patch http://order:8080/orders/5 status="OrderCanceled"
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Sat, 20 Feb 2021 08:54:29 GMT
Transfer-Encoding: chunked
{
    "_links": {
        "order": {
            "href": "http://order:8080/orders/5"
        },
        "self": {
            "href": "http://order:8080/orders/5"
        }
    },
    "amt": 100,
    "createTime": "2021-02-20T08:51:17.441+0000",
    "phoneNumber": "01033132570",
    "productName": "coffee",
    "qty": 2,
    "status": "OrderCanceled"
}

# 주문 조회
root@siege-5b99b44c9c-8qtpd:/# http http://order:8080/orders/5
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Sat, 20 Feb 2021 09:07:49 GMT
Transfer-Encoding: chunked

{
    "_links": {
        "order": {
            "href": "http://order:8080/orders/5"
        },
        "self": {
            "href": "http://order:8080/orders/5"
        }
    },
    "amt": 100,
    "createTime": "2021-02-20T09:07:24.114+0000",
    "phoneNumber": "01033132570",
    "productName": "coffee",
    "qty": 2,
    "status": "Ordered"
}

```


## CQRS / Meterialized View
*****
CustomerCenter의 Mypage를 구현하여 Order 서비스, Payment 서비스, Drink 서비스의 데이터를 Composite서비스나 DB Join없이 조회할 수 있다.
```
root@siege-5b99b44c9c-8qtpd:/# http http://customercenter:8080/mypages/search/findByPhoneNumber?phoneNumber="01012345679"
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Sat, 20 Feb 2021 14:57:45 GMT
Transfer-Encoding: chunked

[
    {
        "amt": 5000,
        "id": 4544,
        "orderId": 4,
        "phoneNumber": "01012345679",
        "productName": "coffee",
        "qty": 3,
        "status": "Made"
    },
    {
        "amt": 5000,
        "id": 4545,
        "orderId": 5,
        "phoneNumber": "01012345679",
        "productName": "coffee",
        "qty": 3,
        "status": "Ordered"
    },
    {
        "amt": 5000,
        "id": 4546,
        "orderId": 6,
        "phoneNumber": "01012345679",
        "productName": "coffee",
        "qty": 3,
        "status": "Receipted"
    },
    {
        "amt": 5000,
        "id": 4547,
        "orderId": 7,
        "phoneNumber": "01012345679",
        "productName": "coffee",
        "qty": 3,
        "status": "Ordered"
    }
]
```

## 운영
*****
### Liveness / Readiness 설정
*****
Pod 생성 시 준비되지 않은 상태에서 요청을 받아 오류가 발생하지 않도록 Readiness Probe와 Liveness Probe를 설정했다.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
  labels:
    app: order
spec:
  :
        readinessProbe:
          httpGet:
            path: '/actuator/health'
            port: 8080
          initialDelaySeconds: 10 
          timeoutSeconds: 2 
          periodSeconds: 5 
          failureThreshold: 10
        livenessProbe:
          httpGet:
            path: '/actuator/health'
            port: 8080
          initialDelaySeconds: 120
          timeoutSeconds: 2
          periodSeconds: 5
          failureThreshold: 5

```

### Self Healing
*****
livenessProbe를 설정하여 문제가 있을 경우 스스로 재기동 되도록 한다.


### CI/CD 설정
*****

```
SA 생성
```
![sa생성](https://user-images.githubusercontent.com/26791027/124943205-45022380-e047-11eb-8dc7-e910dbcf6db7.PNG)

```
Role 생성
```
![role생성](https://user-images.githubusercontent.com/26791027/124943528-88f52880-e047-11eb-9e83-cf16ac636abe.PNG)

```
만들어진 eks-admin SA 의 토큰 가져오기
kubectl -n kube-system describe secret eks-admin
```
![token](https://user-images.githubusercontent.com/26791027/124944092-03be4380-e048-11eb-96c3-07ab7618bed9.PNG)

```
KUBE_URL 확인
```
![kube_url](https://user-images.githubusercontent.com/26791027/124947164-91029780-e04a-11eb-88d8-8edbffc45f85.PNG)

```
codeBuild
```
![build](https://user-images.githubusercontent.com/26791027/124979414-fcaa2c00-e06d-11eb-820a-044482789a47.PNG)


### 동기식 호출 / 서킷 브레이킹 / 장애격리
*****

서킷 브레이킹 프레임워크의 선택: Spring FeignClient + Hystrix 옵션을 사용하여 구현함
시나리오는 order -> payment 호출 시의 연결을 RESTful Request/Response 로 연동하여 구현이 되어있고, 결제 요청이 과도할 경우 CB 를 통하여 장애격리.

- Hystrix 를 설정: 요청처리 쓰레드에서 처리시간이 610 밀리가 넘어서기 시작하여 어느정도 유지되면 CB 회로가 닫히도록 (요청을 빠르게 실패처리, 차단) 설정
```
# application.yml

hystrix:
  command:
    # 전역설정
    default:
      execution.isolation.thread.timeoutInMilliseconds: 610
```
- 피호출 서비스(payment) 의 임의 부하 처리 - 400 밀리에서 증감 220 밀리 정도 왔다갔다 하게
```
@PrePersist
    public void onPrePersist(){ 
        
        try {
            Thread.currentThread().sleep((long) (400 + Math.random() * 220));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```
- 부하테스터 siege 툴을 통한 서킷 브레이커 동작 확인:
- 동시사용자 100명
- 60초 동안 실시
```
root@siege-5b99b44c9c-ldf2l:/# siege -v -c100 -t60s --content-type "application/json" 'http://order:8080/orders POST {"phoneNumber":"01087654321", "productName":"coffee", "qty":2, "amt":1000}'
** SIEGE 4.0.4
** Preparing 100 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.57 secs:     317 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.57 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.63 secs:     317 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.64 secs:     317 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.63 secs:     317 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.69 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.73 secs:     317 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.74 secs:     317 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.75 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.79 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.22 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.11 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.18 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.21 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.20 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.24 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.26 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.28 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.37 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.37 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.40 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.65 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.71 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.76 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.80 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.78 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.88 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     1.89 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     1.89 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.00 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     2.01 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.12 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.16 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.21 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.31 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.33 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     2.44 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.48 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     2.48 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.51 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.52 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.57 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     2.67 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     2.66 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.80 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     2.83 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.84 secs:     319 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     2.89 secs:     319 bytes ==> POST http://order:8080/orders
...
Lifting the server siege...siege aborted due to excessive socket failure; you
can change the failure threshold in $HOME/.siegerc

Transactions:		         701 hits
Availability:		       69.58 %
Elapsed time:		       59.21 secs
Data transferred:	        0.47 MB
Response time:		        8.18 secs
Transaction rate:	       11.84 trans/sec
Throughput:		        0.01 MB/sec
Concurrency:		       96.90
Successful transactions:         701
Failed transactions:	        1070
Longest transaction:	        9.81
Shortest transaction:	        0.05
``` 
운영시스템은 siege 툴로 동작 확인 운영시스템이 비정상종료 없이 적절히 열리고 닫힘으로써 자원 보존 확인.
하지만, 69% 가 성공하였고, 31%가 실패하여 고객 사용성에 있어 좋지 않음 -> 동적 Scale out (replica의 자동적 추가,HPA) 을 통하여 시스템을 확장 해주는 후속처리가 필요.
#### 오토스케일 아웃
사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다.
결제서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 15프로를 넘어서면 replica 를 10개까지 늘려준다:
```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
customercenter-7f57cf5f9f-csp2b   1/1     Running   1          20h
drink-7cb565cb4-d2vwb             1/1     Running   0          37m
gateway-5dd866cbb6-czww9          1/1     Running   0          3d1h
order-595c9b45b9-xppbf            1/1     Running   0          36m
payment-698bfbdf7f-vp5ft          1/1     Running   0          2m32s
siege-5b99b44c9c-8qtpd            1/1     Running   0          3d1h


$ kubectl autoscale deploy payment --min=1 --max=10 --cpu-percent=15
horizontalpodautoscaler.autoscaling/payment autoscaled

$ kubectl get hpa
NAME      REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
payment   Deployment/payment   2%/15%    1         10        1          2m35s

# CB 에서 했던 방식대로 워크로드를 1분 동안 걸어준다.

root@siege-5b99b44c9c-ldf2l:/# siege -v -c100 -t60s --content-type "application/json" 'http://order:8080/orders POST {"phoneNumber":"01087654321", "productName":"coffee", "qty":2, "amt":1000}'
** SIEGE 4.0.4
** Preparing 100 concurrent users for battle.
The server is now under siege...

$ kubectl get pods
NAME                              READY     STATUS    RESTARTS   AGE
customercenter-59f4d6d897-lnpsh   1/1       Running   0          97m
drink-64bc64d49c-sdwlb            1/1       Running   0          112m
gateway-6dcdf4cb9-pghzz           1/1       Running   0          74m
order-7ff9b5458-4wn28             1/1       Running   2          21m
payment-6f75856f77-b6ctw          1/1       Running   0          118s
payment-6f75856f77-f2l5m          1/1       Running   0          102s
payment-6f75856f77-gl24n          1/1       Running   0          41m
payment-6f75856f77-htkn5          1/1       Running   0          118s
payment-6f75856f77-rplpb          1/1       Running   0          118s
siege-5b99b44c9c-ldf2l            1/1       Running   0          96m
```
- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다:
```
kubectl get deploy payment -w

```
- 어느정도 시간이 흐른 후 (약 30초) 스케일 아웃이 벌어지는 것을 확인할 수 있다:
```
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
payment   1         1         1         1         1h
payment   4         1         1         1         1h
payment   4         1         1         1         1h
payment   4         1         1         1         1h
payment   4         4         4         1         1h
payment   5         4         4         1         1h
payment   5         4         4         1         1h
payment   5         4         4         1         1h
payment   5         5         5         1         1h
```
- siege 의 로그를 보아도 전체적인 성공률이 높아진 것을 확인 할 수 있다.
```
Transactions:                   909 hits
Availability:                  99.98 %
Elapsed time:                 143.38 secs
Data transferred:               3.06 MB
Response time:                  3.88 secs
Transaction rate:              63.40 trans/sec
Throughput:                     0.02 MB/sec
Concurrency:                  245.75
Successful transactions:        9090
Failed transactions:               2
Longest transaction:           34.12
Shortest transaction:           0.01
```

### 무정지 재배포
*****
먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함.
seige 로 배포작업 직전에 워크로드를 모니터링 함.
```
# siege -v -c100 -t60s --content-type "application/json" 'http://order:8080/orders POST {"phoneNumber":"01087654321", "productName":"coffee", "qty":2, "amt":1000}'
** SIEGE 4.0.4
** Preparing 100 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.20 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.34 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.39 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.38 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.40 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.40 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.40 secs:     321 bytes ==> POST http://order:8080/orders
HTTP/1.1 201     0.41 secs:     321 bytes ==> POST http://order:8080/orders
:

```
- 새버전으로의 배포 시작
seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인
```
root@siege-5b99b44c9c-ldf2l:/# siege -v -c100 -t60s --content-type "application/json" 'http://order:8080/orders POST {"phoneNumber":"01087654321", "productName":"coffee", "qty":2, "amt":1000}'
** SIEGE 4.0.4
** Preparing 100 concurrent users for battle.
The server is now under siege...
Lifting the server siege...
Transactions:		        4300 hits
Availability:		       99.79 %
Elapsed time:		       59.08 secs
Data transferred:	        1.33 MB
Response time:		        1.05 secs
Transaction rate:	       72.78 trans/sec
Throughput:		        0.02 MB/sec
Concurrency:		       76.67
Successful transactions:        4300
Failed transactions:	           9
Longest transaction:	        4.07
Shortest transaction:	        0.03
```

- 배포기간중 Availability 가 평소 100%에서 70% 대로 떨어지는 것을 확인. 원인은 쿠버네티스가 성급하게 새로 올려진 서비스를 READY 상태로 인식하여 서비스 유입을 진행한 것이기 때문. 이를 막기위해 Readiness Probe 를 설정
```
  readinessProbe:
      httpGet:
        path: '/actuator/health'
        port: 8080
      initialDelaySeconds: 10
      timeoutSeconds: 2
      periodSeconds: 5
      failureThreshold: 10
```
```
kubectl apply -f kubernetes/deployment.yaml

```
- 동일한 시나리오로 재배포 한 후 Availability 확인:
```
root@siege-5b99b44c9c-ldf2l:/# siege -v -c100 -t60s --content-type "application/json" 'http://order:8080/orders POST {"phoneNumber":"01087654321", "productName":"coffee", "qty":2, "amt":1000}'
** SIEGE 4.0.4
** Preparing 100 concurrent users for battle.
The server is now under siege...
Lifting the server siege...
Transactions:		        5261 hits
Availability:		      100.00 %
Elapsed time:		       59.28 secs
Data transferred:	        1.62 MB
Response time:		        1.09 secs
Transaction rate:	       88.75 trans/sec
Throughput:		        0.03 MB/sec
Concurrency:		       97.08
Successful transactions:        5261
Failed transactions:	           0
Longest transaction:	        7.52
Shortest transaction:	        0.01
```
- 배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.


### Persistence Volum Claim
*****

### ConfigMap / Secret
*****
