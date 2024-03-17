# 우아콘2022 - 회원시스템 이벤트 기반 아키텍처 구축하기

- link: https://www.youtube.com/watch?v=b65zIH7sDug

- 1개의 microservice에서 어떻게 이벤트 기반 아키텍처를 다루고 있는지 설명
- 이벤트 기반 아키텍처
  - MSA 구조에서 많이 사용
  - application 사이의 결합을 느슨하게 유지하기 위한 구조

## event 도입

- `message queue를 사용 == event driven architecture`가 아니다
- `consumer의 action`이 아니라 `publisher에서 발생한 일`을 이벤트로 전송한다.
- ex)

  - 로직: member 계정 초기화시, 가족 계정 탈퇴
  - 기존

    ```java
    // member code
    public void initCertificationOwn(memberNumber memberNumber) {
        member.initCertificationOwn(memberNumber);
        family.leave(memberNumber);
    }
    ```

  - MSA로 분리 -> http 호출로 변경

    - http client, 비동기 호출로 변경해도 강결합은 유지됨

    ```java
    // member code
    public void initCertificationOwn(memberNumber memberNumber) {
        member.initCertificationOwn(memberNumber);
        familyClient.leave(memberNumber);
    }
    ```

    ```java
    // family member code
    public void leave(memberNumber memberNumber) {
        family.leave(memberNumber);
    }
    ```

  - message queue 도입 -> 가족 계정 해제 이벤트 발행

    - 물리적 의존 관계는 제거되었지만, 논리적 의존 관계가 유지됨
    - 가족 계정이 해지되는 것을 회원 코드에서 기대함

    ```java
    public void initCertificationOwn(memberNumber memberNumber) {
        member.initCertificationOwn(memberNumber);
        eventPublisher.familyLeave(memberNumber);
    }
    ```

  - message queue 도입 -> 계정 초기화 이벤트 발행

    - 논리적 의존 관계가 제거됨.

    ```java
    public void initCertificationOwn(memberNumber memberNumber) {
        member.initCertificationOwn(memberNumber);
        eventPublisher.initCertificationOwn(memberNumber);
    }
    ```

## event 종류

- application event

  - 단일 application 내에서 사용 가능한 event
  - 주 관심사인 도메인 로직과 나머지 로직(비관심사)의 결합을 느슨하게 유지할 수 있다
  - ex) spring event
  - 배민에서는 도메인 로직과, sns server로 message 발행을 분리했다.

- 내부 event

  - application event 외의 모든 비관심사를 처리
  - 도메인 행위가 수행되어야할 때 함께 처리되어야하는 부가 로직들 처리
    - ex) 디바이스가 로그인되면, \
       로그인된 디바이스 기록, 동일 계정의 타디바이스 로그아웃, ....
  - 배민에서는 sns server가 받아서 부가로직별 sqs로 이벤트를 발행

- application event vs. 내부 event

  - application event: 주요 행위와 트랜잭션, 성능 공유
  - 내부 event: 주요 행위와 트랜잭션, 성능 분리
  - 주요 행위와 강한 정합성이 필요하다면 -> application event. else -> 내부 event

- 외부 event

  - MSA를 위한 외부 이벤트
  - 배민에서는 내부 event를 처리한 event worker가 발행
  - 이벤트 일반화
    - 언제(시간), 누가(식별자), 무엇을 하여(행위), 어떤 변화가(속성) 만 담음
    - 특정 consumer를 위한 payload를 넣지 않음
    - zero payload
      - payload를 사용하지 않음
      - 이벤트 사이의 순서 보장 문제를 해결하기 위해 많이 사용되지만, \
        payload에 대한 의존을 제거하는 효과도 있음

- 내부 event vs. 외부 event
  - 내부에는 열려있고, 외부에는 닫혀있다
    - 변경이 자유롭다 / 자유롭지 않다.
  - 내부 event는 동일 system 내에서 처리되므로, \
     payload 등을 변경할 때 어떤 영향이 있는지 파악이 가능하다 \
     -> 변경이 자유롭다
  - 외부 event는 구독자에 미치는 영향을 알기 어렵다 \
    유관 부서와 협의하여 변경하는 것은 비용이 많이 든다.

## event 저장소

- 내부 이벤트 발행 구간은 http client를 이용해서 언제든지 발행에 실패할 수 있었다.
- 이벤트 발행을 transaction으로 묶으면, message 시스템 장애 -> 시스템 전체 장애로 이어진다.
- 실패 처리를 하지 않으면, 데이터 불일치가 발생한다.
- outbox pattern을 이용해서 이벤트를 저장
  - transaction을 `이벤트 발행`이 아니라 `이벤트 저장소에 저장`과 묶음
- event 저장소는 transaction으로 묶어야 하는데,
  - 다중 데이터베이스의 분산 transaction 구현은 쉽지 않음
  - 유실을 감안하고 다중 DB를 사용하거나 vs. 같은 저장소를 공유 -> 같은 저장소를 선택함
- batch application을 만들어서, 발행 실패한 event를 재발행

## 부가 효과

- 기존에는 로그를 다양한 형태로 DB에 저장했다.
- event 저장소에 어떤 일이 벌어지고 있는지 기록되고 있어서 \
   여러 테이블로 나누어서 저장하던 로그를 하나로 통합할 수 있었다.
