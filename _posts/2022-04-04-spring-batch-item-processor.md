---
layout: post
title: "Spring Batch ItemProcessor"
tags: [SpringBatch]
comments: true
date: 2022-04-04
---

# Item Processor

## ItemProcessor 소개

**ItemProcessor 사용시 장점**

- ItemReader 에서 읽어들인 데이터에 대해 유효성 검증 가능
- ItemProcessorAdapter 를 사용한 기존 서비스 재사용 가능
- ScriptItemProcessor을 통해 스크립트 실행 가능. 스크립트 입력으로 아이템을 재공하고, 스크립트의 출력을 반환 값으로 가져올 수 있음
- ItemProcessor 체인을 통해, 하나의 프로세서에 모든 로직을 담지 않고, 순서대로 실행될 ItemProcessor 목록을 만들 수 있음

**동작방식**

- ItemReader 에서 읽어들인 아이템 리스트를 하나씩 처리 후 ItemWriter로 전달
- ItemProcessor는 입력 타입과 반환 타입이 같을 필요 없음. 단, ItemProcessor의 반환 타입은 ItemWriter의 입력타입과 반드시 일치해야 함
- ItemProcessor에서 null 반환시, 해당 아이템의 이후 모든 처리가 중지됨. 
즉, 해당 아이템에 대해 ItemProcessor에 남아있는 로직도 수행안하고 ItemWriter로도 전달하지 않음. 
그리고 다음 아이템의 처리가 이루어짐.

## 스프링 배치의 ItemProcessor 사용하기

### ValidatingItemProcessor

- 스프링 배치는 ItemReader에서 읽어들인 데이터에 대해 유효성 검증을 할 수 있는 ItemProcessor를 제공
- 기본적으로 유효성 검증에 실패하면 ValidationException이 발생해 잡이 실패로 끝남
- setFilter(true) 메소드로 ValidationException을 던지지 않고 필터링만 되도록 설정가능(default : false)

1. **BeanValidatingItemProcessor 와 Annotation을 사용한 방법**

```kotlin
data class Customer(
    @field: NotNull(message = "First name is required")
    @field: Pattern(regexp = "[a-zA-Z]+", message = "First name must be alphabetical")
    val firstName: String,

    @field: @Size(min = 1, max = 1)
    @field: @Pattern(regexp = "[a-zA-Z]", message = "Middle initial must be alphabetical")
    val middleInitial: String
    
		//...    
) 
```

- Customer 객체에 애너테이션 `@NotNull`, `@Size`, `@Pattern` 등과 같은 유효성 검증 규칙 애너테이션 적용 (`org.springframework.boot:spring-boot-starter-validation` dependency 필요)
- 애너테이션 기능을 동작시키기 위해 `BeanValidatingItemProcessor` 를 제공해야 함
- `BeanValidatingItemProcessor` 는 `ValidatingItemProcessor` 클래스를 상속한 ItemProcessor.
- `ValidatingItemProcessor` 는 `org.springframework.batch.item.validator.Validator` 인터페이스의 구현체를 통해 제공됨

![No image](/assets/posts/20220404/Untitled.png)
    

- 배치에서 사용하는 Validator 인터페이스는 Core Spring Framework 에서 제공하는 `org.springframework.validation.Validator` 인터페이스와 다름

```kotlin
// org.springframework.batch.item.validator.Validator
package org.springframework.batch.item.validator;
public interface Validator<T> {
    void validate(T var1) throws ValidationException;
}

// org.springframework.validation.Validator
package org.springframework.validation;
public interface Validator {
    boolean supports(Class<?> var1);

    void validate(Object var1, Errors var2);
}
```

배치 잡 적용

```kotlin
@Configuration
@EnableBatchProcessing
class ValidationJob(
        private val jobBuilderFactory: JobBuilderFactory,
        private val stepBuilderFactory: StepBuilderFactory
) {

    @Bean
    @StepScope
    fun customerItemReader(@Value("#{jobParameter['customerFile']}") inputFile: Resource?) : FlatFileItemReader<Customer> {
        return FlatFileItemReaderBuilder<Customer>()
                .name("customerItemReader")
                .delimited()
                .names(*arrayOf("firstName", "middleInitial", "lastName", "address", "city", "state", "zip"))
                .targetType(Customer::class.java)
                .resource(inputFile!!)
                .build()
    }

    @Bean
    fun itemWriter(): ItemWriter<Customer> {
        return ItemWriter { items: List<Customer> -> items.forEach(System.out::println) }
    }

    @Bean
    fun customerValidatingItemProcessor() : BeanValidatingItemProcessor<Customer> {
        return BeanValidatingItemProcessor()
    }

    @Bean
    fun copyFileStep(): Step {
        return stepBuilderFactory["copyFileStep"]
                .chunk<Customer, Customer>(5)
                .reader(customerItemReader(null))
                .processor(customerValidatingItemProcessor())
                .writer(itemWriter())
                .build()
    }

    @Bean
    fun job(): Job {
        return jobBuilderFactory["job"]
                .start(copyFileStep())
                .build()
    }
}
```

1. **Validator 인터페이스를 직접 구현한 방법**

```kotlin
class UniqueLastNameValidator: ItemStreamSupport(), Validator<Customer> {

    private var lastNames = hashSetOf<String>()

    override fun validate(value: Customer) {
        if (lastNames.contains(value.lastName)) {
            throw ValidationException("Duplicate last name was found: " + value.lastName)
        }
        this.lastNames.add(value.lastName)
    }

    override fun update(executionContext: ExecutionContext) {
        executionContext.put(getExecutionContextKey("lastNames"), this.lastNames)
    }

    override fun open(executionContext: ExecutionContext) {
        var lastName = getExecutionContextKey("lastNames")
        if (executionContext.containsKey(lastName)) {
            lastNames = executionContext[lastName] as HashSet<String>
        }
    }
}
```

- lastName 이 유일한지에 대한 유효성 검증을 가진 커스텀 Validator 클래스 생성
- Validator 인터페이스의 validate() 메소드 구현 (유효성 검증을 위한 작업)
- ItemStreamSupport 의 open(), update() 메소드 구현 (재시작시 상태확인을 위한 작업)

UniqueLastNameValidator를 사용하는 것으로 Step 및 ItemProcessor 코드 변경

```kotlin
@Configuration
@EnableBatchProcessing
class ValidationJob(
        private val jobBuilderFactory: JobBuilderFactory,
        private val stepBuilderFactory: StepBuilderFactory
) {
		// ...

		// 커스텀 validator 빈 등록
		@Bean
		fun validator(): UniqueLastNameValidator {
		    val uniqueLastNameValidator = UniqueLastNameValidator()
		    uniqueLastNameValidator.setName("validator")
		    return uniqueLastNameValidator
		}
		
		// 커스텀 validator를 사용하는 ItemProcessor 빈 등록
		@Bean
		fun customerValidatingItemProcessor() : ValidatingItemProcessor<Customer> {
		    return ValidatingItemProcessor<Customer>(validator())
		}
		
		@Bean
		fun copyFileStep(): Step {
		    return stepBuilderFactory["copyFileStep"]
		            .chunk<Customer, Customer>(5)
		            .reader(customerItemReader(null))
		            .processor(customerValidatingItemProcessor())  // ItemProcessor 등록
		            .writer(itemWriter())
		            .stream(validator())  // 재시작시 open(),update()를 위한 ItemStream 등록
		            .build()
		}

		// ...
}

```

## ItemProcessorAdapter

- ItemProcessorAdapter를 사용하면 기존 서비스 메소드 사용 가능
- 이번 예제에서는 고객 이름을 대문자로 변경하는 메소드가 이미 있을때, ItemProcessor에서 호출하는 예제를 살펴본다

Service 클래스 정의

```kotlin
@Service
class UpperCaseNameService {

    fun upperCase(customer: Customer): Customer {
        val newCustomer = Customer(customer)

        newCustomer.firstName = newCustomer.firstName.toUpperCase()
        newCustomer.middleInitial = newCustomer.middleInitial.toUpperCase()
        newCustomer.lastName = newCustomer.lastName.toUpperCase()

        return newCustomer
    }
}
```

Service를 사용하는 ItemProcessor 정의 및 사용

```kotlin
@Bean
fun itemProcessor(service: UpperCaseNameService?) : ItemProcessorAdapter<Customer, Customer> {
    val adapter = ItemProcessorAdapter<Customer, Customer>()
    adapter.setTargetObject(service!!)
    adapter.setTargetMethod("upperCase")
    return adapter
}

@Bean
fun copyFileStep(): Step {
    return stepBuilderFactory["copyFileStep"]
            .chunk<Customer, Customer>(5)
            .reader(customerItemReader(null))
            .processor(itemProcessor(null))
            .writer(itemWriter())
            .build()
}
```

## ScriptItemProcessor

- 자바스크립트를 사용해 ItemProcessor를 정의할 수 있음

upperCase.js

- ItemProcessor의 입력을 받아들이고 ItemProcessor의 출력객체를 반환하는 스크립트 정의
- ScriptItemProcessor의 입력(Customer)이 변수아이템에 바인딩됨

```jsx
item.setFirstName(item.getFirstName().toUpperCase());
item.setMiddleInitial(item.getMiddleInitial().toUpperCase());
item.setLastName(item.getLastName().toUpperCase());
item;
```

ScriptItemProcessor 정의 

```kotlin
// script 잡파라미터에 upperCase.js 전달
@Bean
@StepScope
fun scriptItemProcessor(@Value("#{jobParameters['script']}") script: Resource?) : ScriptItemProcessor<Customer, Customer> {
    val itemProcessor = ScriptItemProcessor<Customer, Customer>()
    itemProcessor.setScript(script!!)
    return itemProcessor
}

@Bean
fun scriptCopyFileStep(): Step {
    return stepBuilderFactory["copyFileStep"]
            .chunk<Customer, Customer>(5)
            .reader(scriptCustomerItemReader(null))
            .processor(scriptItemProcessor(null))
            .writer(scriptItemWriter())
            .build()
}
```

## CompositeItemProcessor

- 스프링 배치에서는 스텝내에서 ItemProcessor를 체인처럼 연결할 수 있다
- 즉, CompositeItemProcessor를 사용하여 복잡한 비즈니스 로직을 여러 프로세서의 단계로 쪼갤 수 있다
- CompositeItemProcessor를 통해 여러 ItemProcessor를 연결하면 각 프로세서는 순서대로 진행되고, 어떤 프로세서든지 중간에 null 을 반환하는 순간 해당 아이템은 더이상 처리되지 않는다

예시로는 앞서 살펴본 ItemProcessor 들을 한번에 적용하는 예제를 수행해본다

1. 유효성 검증 수행하는 ValidatingItemProcessor
2. 기존 서비스인 UpperCaseNameService를 사용해 이름을 대문자로 변경하는 ItemProcessorAdapter
3. 자바스크립트를 사용해 address, city, state 필드의 값을 소문자로 변경하는 ScriptItemProcessor

```kotlin
@Configuration
@EnableBatchProcessing
class CompositeItemProcessorJob(
        private val jobBuilderFactory: JobBuilderFactory,
        private val stepBuilderFactory: StepBuilderFactory
) {

    @Bean
    @StepScope
    fun compositeItemReader(@Value("#{jobParameters['customerFile']}") inputFile: String?) : FlatFileItemReader<Customer> {
        return FlatFileItemReaderBuilder<Customer>()
                .name("compositeItemReader")
                .delimited()
                .names(*arrayOf("firstName", "middleInitial", "lastName", "address", "city", "state", "zip"))
                .targetType(Customer::class.java)
                .resource(ClassPathResource(inputFile!!))
                .build()
    }

    @Bean
    fun compositeItemWriter(): ItemWriter<Customer> {
        return ItemWriter { items: List<Customer> -> items.forEach(System.out::println) }
    }

    @Bean
    fun uniqueValidator(): UniqueLastNameValidator {
        val uniqueLastNameValidator = UniqueLastNameValidator()
        uniqueLastNameValidator.setName("validator")
        return uniqueLastNameValidator
    }

    @Bean
    fun uniqueValidatingItemProcessor() : ValidatingItemProcessor<Customer> {
        val validatorItemProcessor = ValidatingItemProcessor<Customer>(uniqueValidator())
        validatorItemProcessor.setFilter(true)  // 예외를 발생시키지는 않고, 유효성검증에 실패한 아이템은 그냥 필터링 처리함
        return validatorItemProcessor
    }

    @Bean
    fun upperCaseItemProcessorAdapter(service: UpperCaseNameService?) : ItemProcessorAdapter<Customer, Customer> {
        val itemProcessorAdapter = ItemProcessorAdapter<Customer, Customer>()
        itemProcessorAdapter.setTargetObject(service!!)
        itemProcessorAdapter.setTargetMethod("upperCase")
        return itemProcessorAdapter
    }

    @Bean
    @StepScope
    fun lowerCaseScriptItemProcessor(@Value("#{jobParameters['script']}") script: String?) : ScriptItemProcessor<Customer, Customer> {
        val scriptItemProcessor = ScriptItemProcessor<Customer, Customer>()
        scriptItemProcessor.setScript(ClassPathResource(script!!))
        return scriptItemProcessor
    }

    @Bean
    fun compositeItemProcessor() : CompositeItemProcessor<Customer, Customer> {
        val compositeItemProcessor = CompositeItemProcessor<Customer, Customer>()

        compositeItemProcessor.setDelegates(listOf(
                uniqueValidatingItemProcessor(),     // 유효성 검증 후 필터링 processor
                upperCaseItemProcessorAdapter(null), // 이름 대문자로 변경 processor
                lowerCaseScriptItemProcessor(null)   // 주소 소문자로 변경 processor
        ))

        return compositeItemProcessor
    }

    @Bean
    fun compositeCopyFileStep(): Step {
        return stepBuilderFactory["compositeCopyFileStep"]
                .chunk<Customer, Customer>(5)
                .reader(compositeItemReader(null))
                .processor(compositeItemProcessor())
                .writer(compositeItemWriter())
                .build()
    }

    @Bean
    fun compositeJob(): Job {
        return jobBuilderFactory["compositeJob"]
                .start(compositeCopyFileStep())
                .build()
    }
}

// Test Case 
@Test
fun `CompositeItemProcessor 테스트`() {
    val jobParam: JobParameters = JobParametersBuilder()
            .addString("fireTime", SimpleDateFormat("yyyyMMddHHmmss").format(Date()))
            .addString("customerFile", "input/customer.csv")
            .addString("script", "lowerCase.js")
            .toJobParameters()

    jobTestUtils.launchJob("compositeJob", jobParam)
}
```

## ClassifierCompositeItemProcessor

- ItemProcessor로 아이템이 전달될 때, 조건에 따라 서로 다른 ItemProcessor 수행을 원하는 경우.
- Classifier(분류기) 인터페이스의 classify 메소드를 구현하여, 입력 아이템을 받아들인 후, 해당 아이템을 처리할 적절한 ItemProcessor를 사용하도록 설정할 수 있다

예제에서는 우편번호가 홀수일때, 짝수일때 서로 다른 ItemProcessor를 사용하는 것을 살펴본다

홀수 또는 짝수일 경우에 서로 다른 ItemProcessor를 반환하는 Classifier 구현 

```kotlin
class ZipCodeClassifier(
        private val oddItemProcess: ItemProcessor<Customer, Customer>,
        private val evenItemProcess: ItemProcessor<Customer, Customer>
) : Classifier<Customer, ItemProcessor<*, out Customer>> {

    override fun classify(classifiable: Customer): ItemProcessor<Customer, Customer> {
        return if(classifiable.zip!!.toInt() % 2 == 0) {
            evenItemProcess
        } else {
            oddItemProcess
        }
    }
}
```

ClassifierCompositeItemProcessor 빈 등록 

```kotlin
// Classifier 인터페이스를 구현한 ZipCodeClassifier에 서로 다른 2개의 ItemProcessor 전달 
@Bean
fun classifier() : Classifier<in Customer, ItemProcessor<*, out Customer>> {
    return ZipCodeClassifier(upperCaseItemProcessorAdapter(null), lowerCaseScriptItemProcessor(null))
}

// ClassifierCompositeItemProcessor 를 반환하는 빈 등록 
@Bean
fun classifierItemProcessor(): ClassifierCompositeItemProcessor<Customer, Customer> {
    val itemProcessor = ClassifierCompositeItemProcessor<Customer, Customer>()
    itemProcessor.setClassifier(classifier())
    return itemProcessor
}
```

## ItemProcessor 직접 만들기

### 아이템 필터링 하기

- 짝수 우편번호는 필터링하고 홀수 우편번호만 남겨두는 ItemProcessor를 작성한다
- ItemProcessor는 null을 반환하기만 하면 해당아이템이 필터링 되고, 필터링 된 레코드 수를 JobRepository에 저장한다
- 커스텀으로 만드려면 ItemProcessor 인터페이스를 구현하는 클래스만 만들면 된다

```kotlin
class EvenFilteringItemProcessor : ItemProcessor<Customer, Customer> {
    override fun process(item: Customer): Customer? {
        return if(item.zip!!.toInt() % 2 == 0) null else item
    }
}
```

```kotlin
@Bean
fun evenFilterItemProcessor(): EvenFilteringItemProcessor 
    = EvenFilteringItemProcessor()
```

JobRepository 조회 

`SELECT * FROM BATCH_STEP_EXECUTION`;

![No image](/assets/posts/20220404/Untitled1.png)