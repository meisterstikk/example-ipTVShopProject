﻿# ipTVShopProject (인터넷 설치 가입신청 서비스)

4조 인터넷&인터넷TV 가입신청 서비스 CNA개발 실습을 위한 프로젝트 입니다.

# Table of contents

- [ipTVShopProject (인터넷 설치 가입신청 서비스)](#---)
  - [서비스 시나리오](#서비스-시나리오)
  - [체크포인트](#체크포인트)
  - [분석/설계](#분석설계)
  - [구현:](#구현-)
    - [DDD 의 적용](#ddd-의-적용)
    - [폴리글랏 퍼시스턴스](#폴리글랏-퍼시스턴스)
    - [폴리글랏 프로그래밍](#폴리글랏-프로그래밍)
    - [동기식 호출 과 Fallback 처리](#동기식-호출-과-Fallback-처리)
    - [비동기식 호출 과 Eventual Consistency](#비동기식-호출-과-Eventual-Consistency)
  - [운영](#운영)
    - [CI/CD 설정](#cicd설정)
    - [동기식 호출 / 서킷 브레이킹 / 장애격리](#동기식-호출-서킷-브레이킹-장애격리)
    - [오토스케일 아웃](#오토스케일-아웃)
    - [무정지 재배포](#무정지-재배포)
  - [신규 개발 조직의 추가](#신규-개발-조직의-추가)

# 서비스 시나리오

고객이 인터넷 가입신청을 하여 설치기사가 설치를 완료하거나, 가입 취소를 하였을 때 처리할 수 있도록 한다.

기능적 요구사항
1. 고객이 인터넷 가입신청을 한다.
1. 가입신청에 대한 접수가 되면, 고객서비스 담당자가 가입요청 지역 설치 기사를 배정한다.
1. 기사배정이 완료되면 해당지역 설치기사에게 설치 요청이 된다
1. 설치기사는 설치요청을 접수한다
1. 설치기사는 설치를 완료 후 설치 완료 처리를 한다.
1. 설치가 완료되면 인터넷가입신청이 완료 처리를 된다.
1. 고객이 가입 신청을 취소할 수 있다.
1. 가입신청이 취소되면 설치상태에 따라 가입 취소 또는 가입 취소 불가능 처리를 한다.
1. 고객서비스 담당자는 설치진행상태를 수시로 확인할 수 있다.

비기능적 요구사항
1. 트랜잭션
    1. 가입취소 신청은 설치상태를 확인 후 처리한다.(미설치상태일 때만 취소처리됨)
1. 장애격리
    1. 인터넷 가입신청과 취소는 고객서비스 담당자의 접수, 설치 처리와 관계없이 항상 처리 가능하다.

1. 성능
    1. 고객서비스 담당자는 설치 진행상태를 수시로 확인하여 모니터링 한다.(CQRS)



# 체크포인트

- 분석 설계


  - 이벤트스토밍: 
    -    

  - 서브 도메인, 바운디드 컨텍스트 분리
    - 
  - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처 
    - 

  - 헥사고날 아키텍처
    - 
    
- 구현
  - [DDD] 
    - 
    - [헥사고날 아키텍처] 
- 운영
  - SLA 준수
    - 
  - 무정지 운영 CI/CD (10)
    - 


# 분석/설계


## AS-IS 조직 (Horizontally-Aligned)
  ![image](https://user-images.githubusercontent.com/56263370/87296744-32433480-c542-11ea-9683-6b792f12cf55.png)  

## TO-BE 조직 (Vertically-Aligned)
  ![image](https://user-images.githubusercontent.com/56263370/87296805-4d15a900-c542-11ea-8fc2-15640ee62906.png)


## Event Storming 결과
* MSAEz 로 모델링한 이벤트스토밍 결과:  
  - http://msaez.io/#/storming/tumGnckjgrc4UVXq2EBT4EFYhnT2/mine/c03f2bb6625a2ed5bef6fcf78dde4b26/-MC01LpwJ3zz9a4MgvCj

### 이벤트 도출
 ![image](https://user-images.githubusercontent.com/56263370/87294520-de831c00-c53e-11ea-9011-f8ef1bf80398.png)


### 부적격 이벤트 탈락
  ![image](https://user-images.githubusercontent.com/56263370/87294672-08d4d980-c53f-11ea-9ee5-b572d8e5a6de.png)

    - 과정중 도출된 잘못된 도메인 이벤트들을 걸러내는 작업을 수행함
        - 중복/불필요 이벤트 제거

### 폴리시 부착
  ![image](https://user-images.githubusercontent.com/56263370/87294750-2609a800-c53f-11ea-841e-b373a52a2200.png)


### 액터, 커맨드 부착하여 읽기 좋게
![image](https://user-images.githubusercontent.com/56263370/87294847-49ccee00-c53f-11ea-8b39-0f9af2ed9a8f.png)


### 어그리게잇으로 묶기
![image](https://user-images.githubusercontent.com/56263370/87294942-6f59f780-c53f-11ea-8c69-af3f105bfdc1.png)

    - app의 

### 바운디드 컨텍스트로 묶기
![image](https://user-images.githubusercontent.com/56263370/87295108-acbe8500-c53f-11ea-897d-553412df4123.png)


    - 도메인 서열 분리 
        


### 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)
![image](https://user-images.githubusercontent.com/56263370/87295156-bf38be80-c53f-11ea-85c5-378d2b12d81d.png)


### 완성된 1차 모형
![image](https://user-images.githubusercontent.com/56263370/87294427-c0b5b700-c53e-11ea-932c-546d7244fbef.png)


    - View Model 추가

### 1차 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증
#### 시나리오 Coverage Check (1)
![image](https://user-images.githubusercontent.com/56263370/87295479-366e5280-c540-11ea-8c27-ec483a384b69.png)

#### 시나리오 Coverage Check (2)
![image](https://user-images.githubusercontent.com/56263370/87295543-5867d500-c540-11ea-9660-87e1c97591a2.png)

#### 비기능 요구사항 coverage
![image](https://user-images.githubusercontent.com/56263370/87295723-95cc6280-c540-11ea-8389-34c538f57e28.png)




### 모델 수정





## 헥사고날 아키텍처 다이어그램 도출
    ![image](https://user-images.githubusercontent.com/56263370/87295821-bdbbc600-c540-11ea-955f-e4835fe69de4.png)


# 구현:



## DDD 의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하였다: (예시는 pay 마이크로 서비스). 이때 가능한 현업에서 사용하는 언어 (유비쿼터스 랭귀지)를 그대로 사용하려고 노력했다. 하지만, 일부 구현에 있어서 영문이 아닌 경우는 실행이 불가능한 경우가 있기 때문에 계속 사용할 방법은 아닌것 같다. (Maven pom.xml, Kafka의 topic id, FeignClient 의 서비스 id 등은 한글로 식별자를 사용하는 경우 오류가 발생하는 것을 확인하였다)




## 폴리글랏 퍼시스턴스




## 폴리글랏 프로그래밍




## 동기식 호출 과 Fallback 처리




## 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트



# 운영

## CI/CD 설정


각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 GCP를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 cloudbuild.yml 에 포함되었다.


## 동기식 호출 / 서킷 브레이킹 / 장애격리

* 서킷 브레이킹 프레임워크의 선택: Spring FeignClient + Hystrix 옵션을 사용하여 구현함


```
# application.yml

hystrix:
  command:
    # 전역설정
    default:
      execution.isolation.thread.timeoutInMilliseconds: 610

```

- 피호출 서비스(결제:pay) 의 임의 부하 처리 - 400 밀리에서 증감 220 밀리 정도 왔다갔다 하게


* 부하테스터 siege 툴을 통한 서킷 브레이커 동작 확인:
- 동시사용자 100명
- 60초 동안 실시

- 운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌. 하지만, 63.55% 가 성공하였고, 46%가 실패했다는 것은 고객 사용성에 있어 좋지 않기 때문에 Retry 설정과 동적 Scale out (replica의 자동적 추가,HPA) 을 통하여 시스템을 확장 해주는 후속처리가 필요.

- Retry 의 설정 (istio)
- Availability 가 높아진 것을 확인 (siege)

### 오토스케일 아웃



## 무정지 재배포

* 먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함

- seige 로 배포작업 직전에 워크로드를 모니터링 함.
- 새버전으로의 배포 시작
```
kubectl set image ...
```

- seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인



# 신규 개발 조직의 추가

  



## 이벤트 스토밍 


## 헥사고날 아키텍처 변화 

## 구현  



## 운영과 Retirement


