---
layout: home
---

도메인 논리 : 애플리케이션이 수행해야 하는 도메인과 관련된 작업. 입력과 저장된 데이터를 바탕으로 하는 계산, 프레젠테이션에서 받은 데이터의 유효성 검사, 프레젠테이션에서 받은 명령을 기준으로 작업 대상이 될 데이터 원본 논리를 결정하는 등의 작업

Ex) Controller , Service 에서 수행하는 비즈니스 로직 및 Validation 체크 등은 도메인 논리에 해당됨.

도메인 논리 주요 패턴 : 트랜잭션 스크립트, 도메인 모델, 테이블 모듈

트랜잭션 스크립트 : 프레젠테션에서 입력 받은 데이터에 대해 유효성 검사와 계산을 통해 입력 처리 후 데이터를 저장하고 다른 시스템에서 작업을 호출하는 프로시저
(Request 별로 Servlet 안에 모든 비즈니스 로직을 정의하는 것?? Service 의 메소드를 호출하는 형태가 아닌 Servlet 안에 모든 로직 정의)

장점
1. 간단한 절차적 모델
2. 로우 데이터 게이트웨이나 테이블 데이터 게이트웨이를 적용해 데이터 원본 계층과 함계 사용하기에 적합함
3. 트랜잭션의 경계를 설정하기가 쉽다 (?)

로우 데이터 게이트웨이(Row Data Gateway)는 데이터베이스 테이블의 한 레코드를 정확하게 묘사하는 객체의 역할을 한다. 오직 관련된 하나의 행에 대해서만 작업을 수행한다.(https://javacan.tistory.com/entry/98)

테이블 데이터 게이트웨이(Table Data Gateway)는 데이터베이스 테이블에 대한 게이트웨이 역할을 하는 객체로서, 하나의 테이블 데이터 게이트웨이 객체를 통해서 테이블의 모든 행을 처리하는 패턴(https://javacan.tistory.com/entry/97)

단점
1. 여러 트랜잭션이 비슷한 작업을 수행해야 하므로 코드가 많이 중복됨에 따라 애플리케이션이 명확한 구조가 없어짐.

도메인 모델 : 동작과 데이터를 모두 포함하는 도메인의 객체 모델(객체지향)

상품 주문 시 트랜잭션 스크립트 동작 예시
1. 상품 코드로 상품 정보 존재 여부 Select
2. 고객 정보 확인 Select
3. 주문 Table에 데이터 insert

상품 주문 시 도메인 모델 예시
1. 상품 객체의 상품 존재여부 확인 메서드 호출
2. 고객 객체의 고객 정보 확인 메서드 호출
3. 주문 객체의 주문 정보 입력 메서드 호출

차이점 : 도메인 모델은 각 객체에 맡는 동작을 위임하여 동작에 대한 결과를 받는 방식으로 구현되나 트랜잭션 스크립트는 하나의 요청을 절차적으로 해결하는 방식으로 구현됨.

테이블 모듈 : 데이터베이스 테이블이나 뷰의 모든 행에 대한 비즈니스 논리를 처리하는 단일 인스턴스 (Spring 의 DAO 를 통하여 비즈니스 논리를 처리하는 것이 테이블 모듈의 예시인것 같음)

도메인 논리 예시 코드

회원가입  
Member member = new Member();  
member.setId(id);  
member.setName(name);  
member.regist();  

회원탈퇴  
Member member = someFindMemberMethod(memberId);  
member.secede();  

테이블모듈 예시코드

회원가입  
RecordSet newData = new RecordSet();  
newData.set("id", id);  
newData.set("name", name);  
MemberTableModule tableModule = MemberTableModule.getInstance();  
tableModule.regist(newData);  

회원탈퇴  
MemberTableModule tableModule = MemberTableModule.getInstance();  
tableModule.secede(someId);  

차이점 : 도메인 논리는 객체를 통하여 회원가입, 회원탈퇴에 대한 기능이 구현되나 테이블모듈은 싱글턴 패턴으로 구성된 MemberTableModule 을 통하여 비즈니스논리를 처리함.
(https://javacan.tistory.com/tag/%ED%85%8C%EC%9D%B4%EB%B8%94%20%EB%AA%A8%EB%93%88)

서비스계층 : 프레젠테이션 계층과 애플리케이션의 API 역할을 하는 계층
(Spring의 dispatcherservlet 이 하는 역할과 비슷해보임)  
서비스 계층은 명확한 API를 제공하여 ㅡ랜잭션 제어, 보안과 같은 기능을 넣기도 좋은 위치.  

service / controller 차이는 무엇인가?  
1. Controller : 클라이언트의 요청에 해당하는 비즈니스 로직을 호출해주는 역할
2. Service : 비즈니스 로직을 수행하는 역할  
차이를 두어 구분하는 이유는 Service에 비즈니스 로직이 존재하므로 재사용성이 좋아짐. (Controller에 비즈로직을 구현한 경우 중복되는 로직에대한 처리가 용이하지못함.)

서비스만 있으면 controller가 필요 없나?  
사용자 요청을 받아 처리할수 있는 Controller의 역할이 필요해보임.

만일 api 를 노출한다면 어디다가 노출하는가?  
Controller 에 API를 정의하고 비즈니스로직은 Service에 정의해야함.

mvc 모델에서의 컨트롤러와는 무슨 차이가 있단 말인가?  
MVC 모델의 Controller가 구현된 것이 Spring의 Controller 라고 생각함.  
Model : Service 와 같이 비즈니스로직을 처리함.
View : 사용자가 보는 화면상의 출력되는 내용(HTML,javascript,Css 등)  
Controller : 사용자의 요청을 받아 View에 반영하여 사용자에게 알려줌

비즈니스 로직은 어디에 들어있으면 좋을까?  
Controller, Service 역할에 맡게 Service에 들어있는 것이 바람직하나, 프로젝트 특성에 따라 트랜잭션스크립트처럼 절차적으로 구현되어도 무방하다고 생각함.
