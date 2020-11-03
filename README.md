# cna-note

0. DDD 의 적용
각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity로 선언
![image](https://user-images.githubusercontent.com/69283682/97958414-79567d80-1df0-11eb-8740-037789d8b552.png)

Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용
![image](https://user-images.githubusercontent.com/69283682/97958566-cc303500-1df0-11eb-8f79-303af9e2ddec.png)


1. 동기식 호출
order > payment 동기식 호출 구현
주문을 받은 직후(@PostPersist) 결제를 요청하도록 처리
![image](https://user-images.githubusercontent.com/69283682/97558818-fadc9300-1a1f-11eb-8585-2bb964977187.png)

동기식 설정에 따라 payment 서비스 중단시 500 에러 발생
![image](https://user-images.githubusercontent.com/69283682/97559049-48f19680-1a20-11eb-8ce6-b4e2d6fe91c2.png)

payment 서비스 정상 작동시 정상 결과 출력
![image](https://user-images.githubusercontent.com/69283682/97559248-87875100-1a20-11eb-9e92-f7bd19be6969.png)

2. 비동기 호출
order > delivery비동기식 구현
결제가 이루어진 후에 delivery 시스템으로 이를 알려주는 행위는 비 동기식으로 처리하여 시스템이 주문 처리를 위하여 결제 주문이 블로킹 되지 않도록 처리한다

Payment 서비스에서는 결제 이력에 기록을 남긴 후 곧바로 결제승인이 되었다는 에빈트를 카프카 송출
![image](https://user-images.githubusercontent.com/69283682/97959189-24b40200-1df2-11eb-8fb4-57b6fdb9ac86.png)

Delivery  서비스에서는 결제 승인 이벤트에 대해서 이를 수신하여 자신의 정책을 처리하도록 PolicyHandler 구현
![image](https://user-images.githubusercontent.com/69283682/97560098-9c181900-1a21-11eb-8eca-1d71886c5f13.png)

delivery 서비스 down

구매주문 정상 실행
![image](https://user-images.githubusercontent.com/69283682/97561062-ecdc4180-1a22-11eb-809e-3300e63c0ef2.png)

서비스 다운되어 있어서 delivery 서비스 조회 시 에러 발생
![image](https://user-images.githubusercontent.com/69283682/97561168-11381e00-1a23-11eb-956e-5962ed45d9ac.png)

delivery 서비스 시작

delivery 서비스 조회 시 정상 확인
![image](https://user-images.githubusercontent.com/69283682/97561273-40e72600-1a23-11eb-8770-1e6e58cdc94a.png)

3. CQRS
dashboard 통해 복수개의 서비스 현황을 조회

dashboard에서 paid, delivery 정보 view 구현
![image](https://user-images.githubusercontent.com/69283682/97561906-31b4a800-1a24-11eb-9bc3-178666142348.png)

dashboard 조회 결과
![image](https://user-images.githubusercontent.com/69283682/97562008-57da4800-1a24-11eb-91d7-ab0a8a80abd0.png)

4. Gateway

8088로 dash 조회
![image](https://user-images.githubusercontent.com/69283682/97562218-9e2fa700-1a24-11eb-8306-36f435499814.png)

8088로 payment 조회
![image](https://user-images.githubusercontent.com/69283682/97562314-c4554700-1a24-11eb-82d7-675d2da0bda7.png)

5.0 동기식 호출 / 서킷 브레이킹 / 장애격리
서킷 브레이킹 프레임워크의 선택: Spring FeignClient + Hystrix 옵션을 사용하여 구현

주문(order)-->결제(payment) 시의 연결을 RESTful Request/Response 로 연동하여 구현이 되어있고, 결제 요청이 과도할 경우 CB 를 통하여 장애격리

Hystrix 를 설정: 요청처리 쓰레드에서 처리시간이 680 밀리가 넘어서기 시작하여 어느정도 유지되면 CB 회로가 닫히도록 (요청을 빠르게 실패처리, 차단) 설정
![image](https://user-images.githubusercontent.com/69283682/97959652-2cc07180-1df3-11eb-9793-8151cf1a1f22.png)

피호출 서비스(결제:payment) 의 임의 부하 처리 - 400 밀리에서 증감 300 밀리 정도 왔다갔다 하게
![image](https://user-images.githubusercontent.com/69283682/97959785-6f824980-1df3-11eb-96f5-ae05a898294d.png)

5. CI
각 서비스를 Pipeline 등록
![image](https://user-images.githubusercontent.com/69283682/97562511-14340e00-1a25-11eb-9f8a-f053d8b5020d.png)

6. CD
각 서비스 CD (Release) 등록
![image](https://user-images.githubusercontent.com/69283682/97562673-5c533080-1a25-11eb-89d8-3ffede6c309c.png)

kubectl 설정
![image](https://user-images.githubusercontent.com/69283682/97562752-7db41c80-1a25-11eb-849b-f154bb53043e.png)

7. configMap
컨피그맵은 키-값 쌍으로 기밀이 아닌 데이터를 저장하는 데 사용하는 API 오브젝트이다.파드는 볼륨에서 환경 변수, 커맨드-라인 인수 또는 구성 파일로 컨피그맵을 사용할 수 있다.

컨피그맵을 사용하면 컨테이너 이미지에서 환경별 구성을 분리하여 애플리케이션을 쉽게 이식할 수 있다.

사용 동기 : 애플리케이션 코드와 별도로 구성 데이터를 설정하려면 컨피그맵을 사용하자.

예를들어, 자신의 컴퓨터(개발용)와 클라우드(실제 트래픽 처리)에서 실행할 수 있는 애플리케이션을 개발한다고 가정해보자. DATABASE_HOST라는 환경 변수를 찾기 위해 코드를 작성한다. 로컬에서는 해당 변수를 localhost로 설정한다. 클라우드에서는 데이터베이스 컴포넌트를 클러스터에 노출하는 쿠버네티스 서비스를 참조하도록 설정한다.

이를 통해 클라우드에서 실행 중인 컨테이너 이미지를 가져와 필요한 경우 정확히 동일한 코드를 로컬에서 디버깅할 수 있다.

link: https://kubernetes.io/ko/docs/concepts/configuration/configmap/

Feign Client : Netflix에서 개발된 Http client binder

참조 link: https://woowabros.github.io/experience/2019/05/29/feign.html

configmap 설치 (yaml)
![image](https://user-images.githubusercontent.com/69283682/97563002-d1266a80-1a25-11eb-8117-80c9c1390211.png)

- application.yml
![image](https://user-images.githubusercontent.com/69283682/97563435-6295dc80-1a26-11eb-836f-8cf5a2e86b67.png)

- deployment.yml
![image](https://user-images.githubusercontent.com/69283682/97563694-c15b5600-1a26-11eb-9810-539fe289d2fa.png)

- PaymentService.java
![image](https://user-images.githubusercontent.com/69283682/97563880-11d2b380-1a27-11eb-823b-5d689377c321.png)

8. HPA (Horizontal Pod Autoscaler)
siege 로그인
kubectl exec -it siege --container siege -- /bin/bash

부하발생
siege -c1 -t10S -r5 -v --content-type "application/json" 'http://gateway:8080/orders POST {"branchId":"1","sauceId":"1", "qty":10, "price":10000}'
![image](https://user-images.githubusercontent.com/69283682/97793969-c42c9580-1c36-11eb-9767-c7f04eabdd6e.png)

AutoScale
kubectl autoscale deploy payment --min=1 --max=10 --cpu-percent=15

![image](https://user-images.githubusercontent.com/69283682/97790615-50c25e00-1c0d-11eb-85fe-9ffb20600c71.png)

- 체크 (변화 없네??)
![image](https://user-images.githubusercontent.com/69283682/97790782-e4485e80-1c0e-11eb-8608-8fe2f883558f.png)

부하발생
![image](https://user-images.githubusercontent.com/69283682/97790782-e4485e80-1c0e-11eb-8608-8fe2f883558f.png)

로그 시작
![image](https://user-images.githubusercontent.com/69283682/97844200-f40a9480-1d2d-11eb-84a9-da09333bf375.png)

로그 후반
![image](https://user-images.githubusercontent.com/69283682/97844119-ce7d8b00-1d2d-11eb-9cff-02566cab726b.png)

부하발생 시 체크
![image](https://user-images.githubusercontent.com/69283682/97843781-3b445580-1d2d-11eb-93bc-c0e1ad1e9af0.png)

![image](https://user-images.githubusercontent.com/69283682/97845029-5dd76e00-1d2f-11eb-8a3d-3f2abd77eb3d.png)

9. liveness
delivery > deployment.yml 원본
![image](https://user-images.githubusercontent.com/69283682/97844680-cbcf6580-1d2e-11eb-8743-3290db4f4bce.png)

liveness 모니터링
![image](https://user-images.githubusercontent.com/69283682/97845655-2e753100-1d30-11eb-8634-3b56cda3eb5e.png)ㅣ

10. Polyglot
polyglot persitency
- order 등 h2 사용
![image](https://user-images.githubusercontent.com/69283682/97567886-311f1000-1a2a-11eb-9878-3b854f9b09e0.png)

- delivery는 hsql
![image](https://user-images.githubusercontent.com/69283682/97567350-0e8cf700-1a2a-11eb-92d7-7f6ee3110255.png)

11. 무정지
hpa 제거
kubectl delete hpa payment

hpa 제거 확인
kubectl get hpa
![image](https://user-images.githubusercontent.com/69283682/97957608-eb2dc780-1dee-11eb-9174-748bae1479a6.png)

부하발생
siege -c1 -t120S -r5 -v --content-type "application/json" 'http://gateway:8080/orders POST {"brancdId":"1","sauceId":"1","qty":10,"price":10000}'
![image](https://user-images.githubusercontent.com/69283682/97956128-3940cc00-1deb-11eb-935f-cb5ffaae8e0b.png)

readiness 설정
![image](https://user-images.githubusercontent.com/69283682/97956520-3e524b00-1dec-11eb-8df4-455752de77b6.png)
