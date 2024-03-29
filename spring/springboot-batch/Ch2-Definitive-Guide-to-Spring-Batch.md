# [스프링배치완벽가이드] 2장. 스프링배치
> * 배치 아키텍처: 배치 처리 구성과 관련된 내용을 좀더 상세히 살펴보고, 책의 나머지 장에서 사용하는 용어를 정의한다
> * 프로젝트 초기 구성: 실제로 실습해보면서 배운다. 이 책에서는 스프링 배치 프레임웍크가 어떻게 동작하는지 예제를 통해 설명하고, 코드를 따라 작성해볼 수 있는 기회를 제공한다.
> * Hello, World!: 열억학 제 1법칙은 에너지 보존과 관련된 이야기이다. 제1운동버칙은 외력이 작용하는 한 정지 상태의 물체가 정지 상태를 유지한다는 것을 다룬다. 컴퓨터 과학의 첫 번째 법칙은 여러분이 배우는 새로운 기술이 무엇이든 해당 기술을 사용해 "Hollw, World!" 프로그램을 작성해야 한다는 것이다. 우리는 그 법칙을 준수할 것이다
> * 잡 실행하기: 당장 첫 번째 잡이 어떻게 실행되는지 명확하게 알기 어려우므로 잡 실행 방법과 기본 파라미터 전달 방법을 살펴본다


## 스프링의 잡 구조화 방법이 가진 장점
* 유연성(Flexibility): 복잡한 로직을 가진 복잡한 작업 플로우를 구성할 때 개발자가 직접 재사용이 가능한 형태로 구현하기는 어렵다. 반면 스프링 배치는 개발자가 재사용이 가능하게 구성할 수 있도록 여러 빌더 클래스를 제공한다
* 유지 보수성(Maintainability): 각 스텝의 코드는 이전 스텝이나 다음 스텝과 독립적이므로 다른 스텝에 거의 영향을 미치지 않으면서 쉽게 각 스텝의 단위 테스트, 디버그, 변경을 할 수 있다
* 확장성(Scalability): 잡 내에 존재하는 독립적인 스텝은 확장 가능한 다양한 방법을 제공한다. 
* 신뢰성(Reliability): 스프링 배치는 스텝의 여러 단계에 적용할 수 있는 강력한 오류 처리 방법을 제공하는데, 예외 발생 시 해당 아이템의 처리를 재시도하거나 건너뛰기하는 등의 동작을 수행할 수 있다

## 잡과 스텝의 처리 방식은 매우 유사하다
잡은 구성된 스텝 목록에 따라 각 스텝을 실행한다. 여러 아이템으로 이뤄진 청크의 처리가 스텝 내에서 완료될 때, 스프링 배치는 JobRepository 내에 있는 JobExecution 또는 StepExecution 을 현재 상태로 갱신한다. 스텝은 ItemReader가 읽은 아이템 목록을 따라간다. 스텝이 각 청크를 처리할 때마다, JobRepository 내 StepExecution의 스텝 상태가 업데이트된다. 현재까지의 커밋 수, 시작 및 종료 시간, 기타 다른 정보 등이 JobRepository에 저장된다. 잡 또는 스텝이 완료되면, JobRepository 내에 있는 JobExecution 또는 StepExecution이 최종 상태로 업데이트된다.


* JobRepository, JobLauncher

* JobInstance:  잡의 논리적인 실행
  * 잡의 이름, 잡의 논리적 실행을 위해 제공되는 고유한 식별 파라미터 모음
  * 하나의 잡을 다른 파라미터로 실행될 때마다 새로운 JobInstance가 생성된다
* JobExecution: 스프링 배치 잡의 실제 실행
  * 잡을 구동할 때마다 매번 새로운 JobExecution을 얻게 된다 
* StepExecution: 스텝의 실제 실행
  * StepInstance 는 존재하지 않는다. 일반적으로 JobExecution 은 여러 개의 StepExecution과 연관된다

> 잡을 처음 실행하면 새로운 JobInstance 및 JobExecution을 얻는다. 실행에 실패한 이후 다시 실행하면, 해당 실행은 어젼히 동일한 논리적 실행(파라미터가 동일함)이므로 새 JobInstance를 얻지 못한다. 그 대신 두 번째 실제 실행을 추적하기 위한 새로운 JobExecution을 얻을 것이다


## 실행하기
mysql 을 이용하여 스프링 배치를 사용하기 위해서는 meta data 를 위한 테이블이 필요하다.
> spring-batch-core 안에 schema-xxx.sql 파일이 있다

