# 4장. 잡과 스텝 이해하기
> 잡이란? 처음부터 끝까지 독립적으로 실행할 수 있는 고유하며 순서가 지정된 여러 스텝의 목록
* 유일하다: 스프링 배치의 잡은 코어 스프링 프레임워크를 사용한 빈 구성 방식과 동일하게 자바나 XML을 사용하며, 구성한 내용을 재사용할 수 있다. 동일한 구성으로 필요한 만큼 잡을 실행할 수 있다. 잡을 여러 번 실행하려고 동일한 잡을 여러번 정의할 필요가 없다
* 순서를 가진 여러 스텝의 목록이다: 잡에서 스텝의 순서는 중요하다. 모든 스텝을 논리적인 순서대로 실행할 수 있도록 잡을 구성한다
* 처음부터 끝까지 실행 가능하다: "배치 처리는 어떠한 완료 상태에 도달할 때까지 추가적인 상호작용 없이 실행하는 처리이다" -> 잡은 외부 의존성 없이 실행할 수 있는 일련의 스텝이다. 예를 들어 특정 디렉터리 내에 처리할 파일이 수신되기를 세 번째 스텝이 기다리도록 잡을 구성하지 않는다. 대신 파일이 도착했을 때 잡을 시작한다.
* 독집적이다: 각 배치 잡은 외부 의존성의 영향을 받지 않고 실행할 수 있어야 한다. 그렇다고 잡이 의존성을 가질 수 없다는 것을 의미하지는 않는다. 잡의 실행은 스케쥴러와 같은 것이 책임진다. 대신 잡은 자신이 처리하기로 정의된 모든 요소를 제어할 수 있다.

## 잡의 생명주기
### JobRunner
잡의 실행은 잡 러너에서 시작된다. 잡 러너는 잡 이름과 여러 파라미터를 받아들여 잡을 실행시키는 역할을 한다
* CommandLineJobRunner: 스크립트를 이용하거나 명령행에서 직접 잡을 싱핼할 때 사용한다
* JobRegistryBackgroundJobRunner: 스프링을 부트스트랩해서 기동한 자바 프로세스 내에서 쿼츠나 JMX 후크와 같은 잡 스케줄러를 사용해 잡을 실행한다면, 스프링이 부트스르탭될 때 실행가능한 잡을 가지고 있는 JobRegistry를 생성한다. JobRegistryBackgroundJobRunner는 JobRegistry를 생성하는데 사용한다.
    * `Deprecated: since 5.0 with no replacement. Scheduled for removal in 5.2`
* JobLauncherCommandLineRunner: 위 두개의 Runner는 스프링 배치(springframework.batch.core.launch.support 에 포함)에서 제공하는 러너이다. 이와 별개로 스프링 부트는 JobLauncherCommandLineRunner를 사용해 잡을 시작하는 또 다른 방법을 제공한다.
    * `Deprecated: since 2.3.0 for removal in 2.6.0 in favor of JobLauncherApplicationRunner`

> 사용자가 스프링 배치를 실행할 때 잡 러너를 사용하긴 하지만, 잡 러너는 프레임워크가 제공하는 표준 모듈이 아니다. 또, 각 시나리오마다 서로 다른 구현체가 필요하기때문에 프레임워크가 JobRunner라는 인터페이스를 별도로 제공하지 않는다. 
실제로 프레임워크를 실행할 때 실제 진입점은 잡 러너가 아닌 JobLauncher 인터페이스의 구현체다.

### JobInstance 
`Entity`: BATCH_JOB_INSTANCE

잡의 논리적 실행을 나타내며 잡 이름과 잡에 전달되 실행 시에 사용되는 식별 파라미터로 식별된다
> JobInstance 는 한 번 성공적으로 완료되면 다시 실행시킬 수 없다. JobInstance는 잡 이름과 전달된 식별 파라미터로 식별되므로, 동일한 식별 파라미터를 사용하는 잡은 한 번만 실행할 수 있다

### JobExecution
`Entity': BATCH_JOB_EXECUTION, BATCH_JOB_EXECUTION_CONTEXT

잡의 실행과 잡의 실행 시도는 다른 개념이다. JobExecution은 잡 실행의 실제 시도를 의미한다.
잡이 처음부터 끝까지 단번에 실행 완료됐다면 해당 JobInstance와 JobExecution은 단 하나씩만 존재한다. 첫 번째 잡 실행 후 오류 상태로 종료됐다면, 해당 JobInstance를 실행하려고 시도할 때마다 새로운 JobExecution이 생성된다

## 잡 파라미터
### JobParameters
잡이 전달받는 모든 파라미터의 컨테이너 역할을 한다
JobParameters 는 java.util.Map<String, JbobParameter> 객체의 래퍼에 불과하다
> (!) 스프링 배치의 JobParameters는 스프링 부트의 명령행 기능을 사용해 프로퍼티를 구성하는 것과 다르다. 따라서 -- 사용해 잡 파라미터를 전달하면 안된다. 또한 스프링 배치의 JobParameters는 시스템 프로퍼티와도 다르므로 명령행에서 -D 아큐먼트를 사용해 배치 애플리케이션에 전달해서도 안된다

#### 잡 파라미터 접근하기
* ChunkContext: 
    * ChunkContext 인스턴스는 실행 시점의 잡 상태를 제공한다. 또한 태스클릿 내에서는 처리 중인 청크와 관련된 정보도 갖고 있다. 해당 청크 정보는 스텝 및 잡과 관련된 정보도 갖고 있다
* 늦은 바인딩: 스텝이나 잡을 제외한 프레임워크 내 특정 부분에 파라미터를 전달하는 쉬원방법은 스프링 구성을 사용해 주입하는 것이다
#### 잡 파라미터 유효성 검증하기
org.springframework.batch.core.JobParametersValidator 인터페이스를 구현하고 해당 구현체를 잡 내에 구셩하면 된다.  

스프링 배치는 DefaultJobParametersValidator를 기본적으로 제공한다. 이 유효성 검증기는 requiredKeys, optionalKeys 두가지 선택적인 의존성이 있다. 둘 다 문자열 배열로써 파라미터 이름 목록이 담긴다.  

JobBuilder 의 메서드는 하나의 JobParameterValidator 인스턴스만 지정하게 되어 있다. -> 스프링 배치는 이런 사례에 사용할 수 있는 CompositeJobParametersValidator 를 제공한다.  

#### 잡 파라미터 증가시키기
기본 제공: org.springframework.batch.core.launch.support.RunIdIncrementer
```java
private static final String RUN_ID_KEY = "run.id";
private String key = "run.id";
public JobParameters getNext(@Nullable JobParameters parameters) {
        JobParameters params = parameters == null ? new JobParameters() : parameters;
        JobParameter<?> runIdParameter = (JobParameter)params.getParameters().get(this.key);
        long id = 1L;
        if (runIdParameter != null) {
            try {
                id = Long.parseLong(runIdParameter.getValue().toString()) + 1L;
            } catch (NumberFormatException var7) {
                throw new IllegalArgumentException("Invalid value for parameter " + this.key, var7);
            }
        }

        return (new JobParametersBuilder(params)).addLong(this.key, id).toJobParameters();
    }
```
* JobParametersIncrementer 직접 구현하기
예시: 하루에 한번만 실행되는 잡에 사용한다면?ㄹㅍ
```java
public class DailyJobTimestamper implements JobParametersIncrementer {
    @Override
    public JobParameters getNext(JobParameters parameters) {
        return new JobParametersBuilder(parameters)
                .addDate("currentDate", new Date())
                .toJobParameters();
    }
}
```

## 잡 리스너 적용하기
### JobExectuionListener
#### 사용사례
* 알림: 스프링 클라우드 태스크는 잡의 시작이나 종료를 다른 시스템에 알리는 큐 메시지를 생성하는 JobExecutionListener를 제공한다
* 초기화: 잡 실행전에 준비해둬야 할 뭔가 있다면 beforeJob 메서드가 해당 로직을 실행하기에 좋은 곳이다
* 정리: 많은 잡이 실행 이후에 정리 작업을 수행한다(파일을 삭제하거나 보관하는 작업 등.) 이 정리 작업은 잡의 성공/실패에 영향을 미치지 않지만 실행돼야 한다. afterJob은 이러한 일을 처리하기에 완벽한 곳이다.

#### 잡 리스너를 작성하는 방법
1. JobExecutionListener 인터페이스를 구현하는 방법
  * beforeJob, afterJob 두 가지를 가지고 있다
  * afterJob 메서드는 잡의 완료 상태에 관계없이 호출된다
2. JobExecutionListener 인터페이스 구현없이 @BeforJob, @AfterJob 을 이용하여 구현하는 방법
  * BeforeJob, AfterJob 을 이용하여 작성하는 경우, listener 에 바로 적용하는 것이 아니라 JobListenerFactoryBean 을 이용해야 한다
```java
@Bean
public Job simpleJob(JobRepository jobRepository, Step simpleTasklet) {
    return new JobBuilder("simpleJob", jobRepository)
            .start(simpleTasklet)
            .listener(JobListenerFactoryBean.getListener(new JobLoggerListener()))
            .build();
}
```

## ExecutionContext
ExecutionContext는 간단한 키-값 쌍을 보관하는 도구에 불과하다. 그러나 ExecutionContext는 잡의 상태를 안전하게 보관하는 방법을 제공한다.








#### spring-batch 5.1.0 을 적용해보면서 찾은 다른점?
##### JobParameter 중에 식별대상에서 제외할 수 있다
> 접두사 "-" 를 사용하면 특정 잡 파라미터를 식별 대상에서 제외할 수 있다
spring-batch-core:5.1.0 에서는 "-" 를 붙여도 식별 대상에서 제외되지 않는다

아래와 같이 parameter 를 전달한다면 식별대상에서 제외할 수 있다
``` 
--spring.batch.job.name=simpleParameterJob name=taemmy,java.lang.String,false
```
DefaultJobParametersConverter.decode 함수에서 value, type, identifying 를 구분한다
* type: Class.forName 을 통해서 Type를 가져온다. 즉 package 를 포함한 Class 구분이어야 한다
* identifying: Boolean.parseBoolean 을 통해서 true/false 를 구분한다

##### (버전과 상관없지만?) Scope Bean의 이해(@JobScope, @StepScope)
* Job을 생성하는 Bean에는 @JobScope를 붙일 수 없다
    * 붙이면 `No context holder available for job scope` 에러가 발생한다
    * Scope Bean 은 해당 Scope 내에서의 LifeCycle을 가지는 Bean 을 의미하는데, 어떤 Job이 시작되어 그 안으로 들어가면 그때부터 JobScope가 되는 것이다
    * 일단 Job이 만들어진 이후에야 JobScope가 존재할 수 있기때문에 Job 객체를 만드는 것 자체는 JobScope가 될 수 없다
* 같은 이유로 Step을 생성하는 Bean에는 @StepScope를 붙일 수 없다. 하지만 @JobScope는 붙일 수 있다.
* Job이나 Step의 실행 Context 내에서만 생존해도 상관없는 Job과 Step의 부속품들(Listener, Tasklet 등)은 Scope 여도 상관없고, JobScope를 붙이면 Job별로 StepScope를 붙이면 Step 별로 새당 context 내에 개별 생성된다.
* JobParameter는 Job/StepScope 내에서만 불러올 수 있다

참고: [(Spring Batch) scope bean의 이해](https://umbum.dev/2003/)
