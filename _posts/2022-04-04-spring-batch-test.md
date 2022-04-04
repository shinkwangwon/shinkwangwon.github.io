---
layout: post
title: "Spring Batch 테스트"
tags: [SpringBatch]
comments: true
date: 2022-04-04
---


# 배치 처리 테스트하기

## JUnit

- 자바 클래스의 단위 테스트를 할 수 있는 프레임워크
- JUnit 을 사용하는 클래스는 Argument를 갖는 생성자가 있으면 안된다
- 각 테스트 메서드는 public이고 void이며 아규먼트를 갖지 않아야 한다
- Assert 클래스 : 테스트 유효성 검증
- @Test 어노테이션을 통해 테스트 메서드임을 나타낸다
- @BeforEach : 각 테스트 메서드 실행 전에 특정 메서드 실행 가능
- @AfterEach : 각 테스트 메서드 실행 이후에 특정 메서드 실행 가능
- @BeforeAll : 테스트 클래스 내의 어떠한 테스트 메서드도 실행되기 전에 단 한번만 실행해야할때 사용
- @Disabled : 테스트에서 실행하지 않고 무시 (이전에는 @Ignore)
- @RunWith : JUnit이 제공하는 클래스가 아닌 다른 클래스로 지정하고 싶을때 사용
    - JUnit5 부터는 @ExtendWith 를 사용
    - SpringBoot 에서는 @SpringBootTest 어노테이션이 @ExtendWith 를 갖고 있음
    
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @BootstrapWith(SpringBootTestContextBootstrapper.class)
    @ExtendWith(SpringExtension.class)
    public @interface SpringBootTest {
      ...
    }
    ```
    

## Mock 객체

- 대부분의 배치는 어플리케이션 서버, 데이터베이스 등과 같은 외부시스템에 의존한다
- 이런 의존성까지 모두 테스트하는건 단위테스트에 어긋난다
- 이런 경우 mock 객체를 사용해 테스트 환경에서 필요한 의존성을 대체하고 외부 의존성의 영향없이 비즈니스 로직을 테스트 한다
- 스텁(Stub)은 목 객체가 아니다. 스텁은 테스트에서 사용되는 하드코딩된 구현체이다

### Mockito

Mock객체를 활용할 수 있는 라이브러리 

```kotlin
@Component
class CustomerItemValidator(
        val jdbcTemplate: NamedParameterJdbcTemplate
) : Validator<CustomerUpdate> {

    companion object {
        const val FIND_CUSTOMER = "SELECT COUNT(*) FROM CUSTOMER WHERE customer_id = :id"
    }

    override fun validate(customer: CustomerUpdate) {
        val parameterMap = Collections.singletonMap("id", customer.customerId)

        val count = jdbcTemplate.queryForObject(FIND_CUSTOMER, parameterMap, Long::class.java)

        if (count == 0L) {
            throw ValidationException("Customer id ${customer.customerId} was not able to be found")
        }
    }
}
```

- 단순히 사용자가 존재하는지 유효성 검증하는 Validator 생성

```kotlin
class CustomerItemValidatorTest {
    @Mock
    lateinit var template: NamedParameterJdbcTemplate

    lateinit var  validator: CustomerItemValidator

    @BeforeEach
    fun setUp() {
        MockitoAnnotations.initMocks(this)
        this.validator = CustomerItemValidator(template)
    }

    @Test
    fun testValidCustomer() {
        // given
        val customer = CustomerUpdate(customerId = 5L)

        // when
        val parameterMap = ArgumentCaptor.forClass(Map::class.java)
        `when`(this.template.queryForObject(eq(CustomerItemValidator.FIND_CUSTOMER), parameterMap.capture() as Map<String, Long>, eq(Long::class.java)))
                .thenReturn(2L)

        this.validator.validate(customer)

        // then
        Assertions.assertEquals(5L, parameterMap.value["id"])
    }
}
```

- NamedParameterJdbcTemplate 을 목객체로 생성하여 실제로 DB조회는 발생하지 않도록 함
- given : 유효성 검증에 사용할 데이터를 미리 만들어 둠
- when : 유효성 검증에서 실제 DB 조회가 발생하지 않도록, 목객체를 통해 어떤 입력에 따라 어떤 결과를 나타나게 할지 기술함
- then : 로직이 정상적으로 수행했는지 확인하는 부분이지만, 여기서는 단순히 유효성 검증로직이 수행될때 파라미터가 정상적으로 전달됐는지 확인하는 것으로 한다
- **참고** : when().thenReturn() 같은 메소드는 `org.mockito.Mockito` 패키지에 있고, given().willReturn() 같은 메소드는 `org.mockito.BDDMockito` 패키지에 있다. BDDMockito클래스는 Mockito 클래스를 상속받은 클래스로 좀 더 BDD(Behavior-Driven-Development)에 초점을 맞춘 클래스이다

## 스프링 클래스를 사용해 통합 테스트하기

- 통합테스트는 서로 다른 여러 컴포넌트 간의 상호작용이 정상적으로 수행되는지 테스트하는 것
- 서비스로직에서 데이터베이스와 연결이 잘되며 정상적으로 DB를 조회하여 Entity 에 매핑이 되는지 등등

**테스트 환경 구성하기**

- 테스트 실행시 특정 데이터베이스에 영향받지 않으려면 인메모리 HSQLDB 인스턴스를 생성하도록 데이터베이스 테스트 구성을 해야 함
- `testImplementation("org.hsqldb:hsqldb")`

**스프링 배치 테스트하기**

- 스프링 배치는 스텝 내에서 TestExecutionListener를 사용해 JobExecution 이나 StepExecution 을 에뮬레이트하는 방법을 제공함으로써 컨텍스트에 값을 주입할 수 있게 해준다
- TestExecutionListener 는 테스트 메서드 실행 전후에 수행돼야 하는 일을 정의하는 스프링 API이다
- @BeforeEach, @AfterEach 대신, TestExecutionListener를 사용하면 테스트 케이스의 모든 메서드에 원하는 동작을 더욱 재사용 가능한 방식으로 삽입할 수 있다
- StepScopeTestExecutionListener, JobScopeTestExecutionListener 가 있는데 주로 StepScopeTestExecutionListener 을 사용한다
- StepScopeTestExecutionListener는 테스트 케이스에서 팩토리 메서드를 사용해 StepExecution을 가져오고, 반환된 컨텍스트를 현재 테스트 메서드의 컨텍스트로 사용한다
- StepScopeTestExecutionListener는 또한 각 테스트 메서드가 실행되는 동안 StepContext를 제공한다

**테스트 파일 설정**

예시에서는 ItemReader를 테스트하는 클래스를 작성한다

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@TestExecutionListeners(
		listeners = {StepScopeTestExecutionListener.class, JobScopeTestExecutionListener.class},
		mergeMode = TestExecutionListeners.MergeMode.MERGE_WITH_DEFAULTS
)
public @interface SpringBatchTest { }
```

- @SpringBatchTest 는 아래의 것들을 자동으로 Bean으로 등록해준다
    - 잡이나 스텝을 실행하는 JobLauncherTestUtils 인스턴스
    - JobRepository에서 JobExecutions를 생성하는데 사용하는 JobRepositoryTestUteils
    - 스텝스코프와 잡스코프 빈을 테스트할 수 있는 StepScopeTestExecutionListener, JobScopeTestExecutionListener
    - StepScopeTestExecutionListener만 사용할 것이라면, @SpringBatchTest 대신 `@TestExecutionListeners(listeners = [StepScopeTestExecutionListener::class])` 처럼 사용해도 된다

```kotlin
@SpringBootTest
@SpringBatchTest
@JdbcTest
class FlatFileItemReaderTests {

    fun getStepExecution(): StepExecution {
        val jobParameters = JobParametersBuilder()
                .addString("customerUpdateFile", "input/customerUpdateFile.csv")
                .toJobParameters()

        return MetaDataInstanceFactory.createStepExecution(jobParameters)
    }
}
```

- getStepExecution 메소드는 테스트 케이스 수행시 새로운 StepExecution을 가져오기 위해 각 테스트 메서드 전에 호출됨
- 아무런 어노테이션이 붙어 있지 않고, 직접호출하는 곳은 없지만, StepScopeTestExecutionListener가 사용하는 팩토리 메서드로 사용되고, 이것이 없다면 스프링 배치는 기본적으로 제공되는 StepExecution을 사용한다
- JobParameter 과 같은 것을 테스트하기 위해선 getStepExecution() 메서드를 선언해주어야 한다
- MetaDataInstanceFactory 는 StepExecution과 JobExecution 인스턴스를 생성하는 유틸리티 클래스
- @JdbcTest 는 스프링 부트에서 인메모리 데이터베이스를 생성함

```kotlin
@SpringBootTest
@SpringBatchTest
@JdbcTest
class FlatFileItemReaderTests {
		@Autowired
		lateinit var customerUpdateItemReader: CustomerUpdateItemReader<CustomerUpdate>

    fun getStepExecution(): StepExecution {
        val jobParameters = JobParametersBuilder()
                .addString("customerUpdateFile", "input/customerUpdateFile.csv")
                .toJobParameters()

        return MetaDataInstanceFactory.createStepExecution(jobParameters)
    }
		
		@Test
		fun testTypeConversion() {
		    customerUpdateItemReader.open(ExecutionContext())
		    Assert.assertTrue(customerUpdateItemReader.read() is CustomerAddressUpdate)
		    Assert.assertTrue(customerUpdateItemReader.read() is CustomerContactUpdate)
		    Assert.assertTrue(customerUpdateItemReader.read() is CustomerNameUpdate)
		}
}

```

- 테스트 클래스에 주입된 ItemReader의 open() 메서드를 호출한 다음에, read() 메서드를 호출하면 된다

**스텝 테스트하기**

JobLauncherTestUtils.launchStep() 메소드를 이용하여 특정 스텝만 테스트한다.

```kotlin
@Configuration
@EnableAutoConfiguration
@EnableBatchProcessing
@ComponentScan(basePackages = ["com.naverfin.paylater.batch"])
class JobTestUtils {
		@Autowired
    lateinit var applicationContext: ApplicationContext

    @Autowired
    lateinit var jobRepository: JobRepository

    @Autowired
    lateinit var jobLauncher: JobLauncher

    fun getJob(jobName: String): JobLauncherTestUtils {
        val job: Job = applicationContext.getBean(jobName, Job::class.java)
        val jobLauncherTestUtils = JobLauncherTestUtils()
        jobLauncherTestUtils.jobLauncher = jobLauncher
        jobLauncherTestUtils.jobRepository = jobRepository
        jobLauncherTestUtils.job = job

        return jobLauncherTestUtils
    }

    fun launchStep(jobName: String, stepName: String, jobParameters: JobParameters): JobExecution {
        return getJob(jobName).launchStep(stepName, jobParameters)
    }

		fun launchJob(jobName: String, jobParameters: JobParameters): JobExecution {
        return getJob(jobName).launchJob(jobParameters)
    }
}

@SpringBootTest
class NormalRedemptionJobConfigTest {
    @Autowired
    lateinit var jobTestUtils: JobTestUtils

    @Test
    fun `스텝 테스트`() {
        // ..

        val jobParam: JobParameters = JobParametersBuilder()
                .addString("fireTime", System.currentTimeMillis().toString())
                .addString("time", "20210615060000")
                .toJobParameters()

				val jobExecution: JobExecution = jobTestUtils.launchStep(
                jobName = NormalRedemptionJobConfig.JOB_NAME, 
                stepName = "normalRedemptionStep", 
                jobParameters = jobParam)
				Assertions.assertEquals(BatchStatus.COMPLETED, jobExecution.status)

				val stepExecution = jobExecution.stepExecutions.iterator().next()
				Assertions.assertEquals(BatchStatus.COMPLETED, stepExecution.status)
				Assertions.assertEquals(2, stepExecution.readCount)
				Assertions.assertEquals(2, stepExecution.writeCount)

        userList.forEach {
            val redemption = redemptionRepository.findTopByUserAndMonthAndTypeOrderByCreateDttmDesc(it, nowMonth, RedemptionType.NORMAL)
            Assertions.assertNotNull(redemption)
            Assertions.assertEquals(RedemptionStatus.WAITING, redemption!!.redemptionStatus)
        }

				// .. 
    }
```

**잡 테스트하기**

JobLauncherTestUtils.launchJob() 메서드를 이용해 전체 Job을 테스트한다

```kotlin
@Test
fun `잡 테스트`() {
    
    val jobParam: JobParameters = JobParametersBuilder()
            .addString("fireTime", System.currentTimeMillis().toString())
            .addString("time", "20210615060000")
            .toJobParameters()

    val jobExecution: JobExecution = jobTestUtils.launchJob(
            jobName = NormalRedemptionJobConfig.JOB_NAME,
            jobParameters = jobParam)

    Assertions.assertEquals(BatchStatus.COMPLETED, jobExecution.status)

    val stepIterator = jobExecution.stepExecutions.iterator()
		while(stepIterator.hasNext()) {
        val stepExecution = stepIterator.next()
        Assertions.assertEquals(BatchStatus.COMPLETED, stepExecution.status)
        println("stepExecution:  $stepExecution")
    }
		
		// .. 
}
```