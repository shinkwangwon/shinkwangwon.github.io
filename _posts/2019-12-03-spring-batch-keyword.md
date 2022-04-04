---
layout: post
title: "Spring Batch 용어 정리"
tags: [SpringBatch]
comments: true
date: 2019-12-03
---

## Job
- Job은 배치 처리 과정을 하나의 단위로 만들어 표현한 객체이다. (배치 처리에 있어 항상 최상단 계층에 있다)
- 하나의 Job에는 여러 Step이 존재한다. 즉 Job객체는 여러 Step인스턴스를 포함하는 컨테이너이다.
- JobBuilderFactory로 JobBuilder을 생성하는데 JobBuilder는 Job을 직접적으로 생성하는 것이 아니라 별도의 구체적 빌더를 생성한 후 변환하기 때문에 Job생성 방법이 다를 수 있다는 것을 유연하게 처리할 수 있다.

## JobInstance
- JobInstance는 배치 처리에서 Job이 실행될 때 하나의 Job실행 단위이다.
- JobInstance는 Job이 실행될 때마다 생성되는데, 만약 Job이 실패하면 JobInstance는 없어지지 않고 다음 수행주기 때 동일한 없어지지 않은 JobInstance가 다시 실행된다. 즉, Job실행이 실패하면 JobInstance가 끝난것으로 간주하지 않기 때문에 없어지지 않고 남아 있는다.
- 하루에 한번 수행되는 배치가 있을 때, 어제 Job이 실패했다면 오늘 수행되는 배치는 어제 생성된 JobInstance를 가지고 다시 수행한다.


## JobExecution
- JobExecution은 JobInstance에 대한 한 번의 실행을 나타내는 객체이다. (JobInstance가 한 번 실행될 때마다 JobExecution는 생성된다)
- 하루에 한번 수행되는 배치가 있을 때, 어제 Job이 실패했다면 오늘 수행되는 배치는 어제 생성된 JobInstance를 가지고 다시 수행한다. 동일한 JobInstance가 2번 수행됐지만 JobExecution은 한번의 실행에 대해 하나씩 생성되기 때문에 2개가 생성되었다. 즉 하나의 JobInstance에 어제 실패한 JobExecution과 오늘 성공한 JobExecution 두개를 가지게 된다.
- JobExecution은 JobInstance, 배치 실행 상태, 시작시간, 끝난시간, 실패했을경우의 메세지 정보 등을 가지고 있다.

## JobParameter
- JobParameter는 Job이 실행될 때 필요한 파라미터들을 Map타입으로 지정하는 객체이다.
- JobParameter는 JobInstance를 구분하는 기준이 되기도 한다. (JobParameter와 JobInstance는 1:1 관계)

## StepExecution
- Step의 실행정보를 담는 객체이다.

## JobRepository
- JobRepository 컴포넌트는 다양한 배치 수행과 관련된 수치 데이터(시작 및 종료시간, 상태, 읽기/쓰기 횟수등)뿐만 아니라 잡의 상태를 유지 관리함
- 스텝 내의 청크 처리가 완료될 때마다 JobRepository 내에 있는 JobExecution 또는 StepExecution을 현재 상태로 갱신함 (청크 단위마다 갱신). 현재까지의 커밋 수, 시작 및 종료시간, 기타 정보등이 JobRepository에 저장됨

## JobLauncher
- 잡을 실행하는 역할을 담당. Job.execute 메서드를 호출하는 역할 이외에도, 잡의 재실행 가능여부 검증, 잡의 실행방법, 파라미터 유효성 검증 등의 처리를 수행

## ItemReader, ItemProcessor, ItemWriter
- 하나의 Step은 (ItemReader, ItemProcessor, ItemWriter) 한 쌍을 가질 수 있다.
- Step이 위의 3개를 반드시 가져야할 필요는 없다. 로직이 너무 복잡하여 분리할 필요가 있을 경우에 사용하면 된다.

## Tasklet
- Step에 대한 기능을 구현한 구현체 

## Chunk 
- reader -> processor -> writer 의 과정을 수행하기 위한 단위로써 commit-interval에 지정한 숫자만큼 writer에 쌓이면 실제 DB로 커밋된다.
- 기본적으로 reader -> processor은 데이터를 한건씩 읽어와서 처리한 후 writer에 넘기는데 성능상 문제가 있으므로 writer는 처리된 결과물을 모아뒀다가 한번에 commit하기 위해 사용한다. (reader는 PagingItemReader를 이용해 한번에 읽어올 데이터 개수를 변경할 수 있다.)
- 트랜잭션의 단위로 쓰이기도 하며 commit-interval=10 으로 설정하고 200개의 데이터를 처리한다고 했을 때, 중간에 오류가 발생하면 처리된 것들은 commit되고 현재 처리중인 10개만 롤백된다.

## Partitioner
- Step을 병렬로 처리할 때 사용 
- Master -> Slave 구조로 하나의 Master에서 여러개의 Slave를 구동시켜 병렬로 처리하도록 함
- partitioner에 지정한 작업을 먼저 수행한 후에 step으로 진입한다.
- grid-size 에 지정한 개수만큼 Slave가 생성되서 병렬처리한다. 만약 gird-size=10 이고 처리할 데이터가 200개이면 각 slave는 20개씩 나눠서 처리한다.

## decision
- 어떠한 작업의 결과에 따라 다음 Step을 진행할지 패스할지 아예 종료시킬지를 컨트롤 할 수 있다.

## Step
- Step은 실질적인 배치 처리를 정의하고 제어하는데 필요한 모든 정보가 있는 도메인 객체이다. 
- 모든 Job에는 1개 이상의 Step이 있어야 한다. Step은 Job을 처리하는 실질적인 단위로 쓰인다.
- 스텝은 독립적인 단위로 실행됨. 하나의 Job이 여러 스텝을 실행할 때도 각 스텝은 완전히 독립적인 단위로 실행됨 
- 스텝도 하나의 Bean 으로 등록되기 때문에 다른 Job에서 재사용이 가능함
- 스텝의 청크단위도 독립적인 트랜잭션 단위로 실행됨 
- 단순한 작업을 하는 태스크릿 기반 스텝인 Tasklet 으로 스텝을 구현할 수도 있고, 청크 기반 스텝인 ItemReader, ItemProcessor, ItemWriter로 구성할 수 도 있음  
- Tasklet은 스텝이 중지될때 까지 execute 메서드를 반복해서 호출하는데, execute 메서드가 호출될때마다 독립적인 트랜잭션이 생성됨

## @EnableBatchProcessing
- 이 애너테이션은 배치 인프라스트럭처를 부트스트랩하는데 사용
- 배치 인프라스트럭처를 위한 대부분의 스프링 빈 정의를 제공 하므로 다음과 같은 컴포넌트를 직접 포함 시킬 필요없음
  - JobRepository : 실행 중인 잡의 상태를 기록하는데 사용됨
  - JobLauncher : 잡을 구동하는데 사용됨
  - JobExplorer : JobRepository를 사용해 읽기 전용 작업을 수행하는데 사용됨
  - JobRegistry : 특정한 런처 구현체를 사용할 때 잡을 찾는 용도로 사용됨
  - PlatformTransactionManager : 잡 진행 과정에서 트랜잭션을 다루는데 사용됨
  - JobBuilderFactory : 잡을 생성하는 빌더
  - StepBuilderFactory : 스텝을 생성하는 빌더


#### 참고
- <https://cheese10yun.github.io/spring-batch-basic/>