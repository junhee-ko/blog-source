---
layout: post
title:  "Spring Kafka Listener Container BATCH AckMode"
date:   2021-04-22
categories: Kafka
---

Spring Kafka Listener Container 의 AckMode 가 BATCH 일 때,
어떻게 records 를 consume 하고 commit 하는지 정리한다.

## Offset

commit 을 정확히 이해하기 위해, offset 의 개념부터 정리하자.
Kakfa 는 파티션의 각각의 레코드에 대해 numerical offset 을 가지고 있다.
offset 은 두 가지 역할을 한다. 

1. 파티션의 레코드에 대한 unique identifier
2. 파티션에 대한 consumer 의 위치

그래서, commit 된 position 은 마지막 offset 을 가리킨다.

## AckMode

Spring Kakfa 에는 ContainerProperties 로 AckMode 일곱 가지가 있다.
offset commit 을 어떻게 할지에 대한 설정이다.
이중 BATCH AckMode 는 어떻게 동작하는지 알아보자.

우선, BATCH AckMode 는 문서에 따르면 다음과 같이 정의되어 있다.

> Commit the offsets of all records returned by the previous poll after they all have been processed by the listener.

번역하면 다음과 같다.
"이전 poll 에 의해 반환된 모든 레코드가" 리스너에 의해 모두 처리가 끝난 뒤에, offset 을 커밋한다.

## Test Scenario

test 는 다음 순으로 진행하자.

1. Create topic
2. Consumer group 과 lag 을 확인하기 위해, Consumer app 을 실행한 뒤 down
2. Producer app 을 실행하여 1000 개의 record 를 produce
3. Consumer app 에 break point 를 걸고 실행하여, 언제 record 를 가져오고 commit 하는지 확인

## Topic Create

Kafka package 에 있는 실행파일로 토픽을 생성하자. 
(물론, Zookeeper 와 Broker 는 구동된 상태이어야한다. 이 과정은 생략한다.)

```shell
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 2 --topic topic-for-commit-test
```

다음과 같이 정상적으로 생성된 것이 확인된다.

![](/image/kafka-topic-create.png)

## Run Consumer

consumer config 설정은 다음과 같다.

```java
@Configuration
public class MyConsumerConfig {

  @Bean
  public ConcurrentKafkaListenerContainerFactory<String, String> concurrentKafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();

    factory.setConcurrency(1);
    factory.getContainerProperties().setAckMode(AckMode.BATCH);
    factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(consumerConfigs()));

    return factory;
  }

  private Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9091");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "group-for-commit-test");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
    props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 500000);

    return props;
  }
}
```

주의해서 볼 점은,

1. ContainerProperties 의 AckMode 를 BATCH 를 설정
2. Consumer config 로 SESSION_TIMEOUT_MS_CONFIG 를 500000 로 설정

2번을 다시 보자. 
SESSION_TIMEOUT_MS_CONFIG 의 default 값은 10000 이다. 이 의미는,

1. consumer 가 10 초동안 브로커에게 heartbeats 를 보내지 않으면
2. broker 는 이 consumer 를 failure 이라고 판단하고 rebalancing 을 한다는 것이다.

이 값을 default 값 10000 이 아니라, 500000 으로 설정한 이유는 다음과 같다.
consumr application 을 debug mode 로 실행하여 libe by line 으로 commit 을 어떻게 하는지 확인할 건데, 확인하는 시간 동안 heartbeats 를 보내지 않아 rebalancing 에 실패하게 되면 commit 에 실패하기 때문에 여유롭게 설정한 것이다.

이제, consumer app 을 실행하자.
"group-for-commit-test" consumer group 은 현재 lag 과 offset 이 0 인 것을 알 수 있다.  

![](/image/kafka-consumer-group-offset.png)

debug 모드로 다시 consumer 를 실행하기 위해, 현재 실행중인 consumer app 은 down 시킨다.

## Run Producer

producer 는 1번~1000번의 number 를 각각 가지는 records 1000 개를 produce 한다.

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class MyProducer {

  private final KafkaTemplate<String, String> template;
  private final ObjectMapper objectMapper;

  public void produce() throws JsonProcessingException {
    log.info("produce starts");

    for (int i = 1; i <= 1000; i++) {
      String message = objectMapper.writeValueAsString(
          MyMessage.builder()
              .number(i)
              .build()
      );

      template
          .send("topic-for-commit-test", message)
          .addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onSuccess(SendResult<String, String> result) {
              log.info("Produce Success, Result : {}", result);
            }

            @Override
            public void onFailure(Throwable ex) {
              log.error("Produce Fail, Ex : {}", ex.getMessage());
            }
          });
    }
  }
}
```

"group-for-commit-test" consumer group 은 현재 lag 1000, offset 이 1000 인 것이 확인된다.

![](/image/kafka-consumer-group-offset-after-produce.png)

## Run Consumer

해당 지점에 break point 를 걸고, debug mode 로 app 을 다시 실행시키자.

![](/image/kakfa-consume-test-01.png)

여기서, KafkaMessageListenerContainer 는 ConcurrentMessageListenerContainer 가 가지고 있는 container 이다.
예를 들어, ConcurrentKafkaListenerContainerFactory 의 concurrency 를 3으로 설정하면 3 개의 KafkaMessageListenerContainer 를 독립적인 thread 로 실행된다.

![](/image/kafka-message-listene-container.png)

계속 진행 상황을 파악하자.
poll 을 통해서 500 개의 record 를 가져온것이 확인된다.
그런데 왜 500 개씩 가져올까 ? consumer 의 max.poll.records 의 값이 default 500 이기 때문이다.

![](/image/kakfa-consume-test-02.png)

500 개의 record 각각을 handler (@KafkaListner method) 에게 전달하여 비즈니스 로직을 처리한다.

![](/image/kakfa-consume-test-03.png)

![](/image/kakfa-consume-test-04.png)

commit 의 결과로 1000 이였던 lag 이 500 으로 줄어들었다.

![](/image/kakfa-consume-test-05.png)

이제 다시 poll 을 통해서 500 개의 record 를 가져온다.

![](/image/kakfa-consume-test-06.png)

마찬가지로, 500 개의 record 를 각각 handler 에게 전달하여 비즈니스 로직을 처리한다.

![](/image/kakfa-consume-test-07.png)

다음 poll 을 호출하기 전에 commit 을 한다.

![](/image/kakfa-consume-test-08.png)

commit 의 결과로 500 이였던 lag 이 0 으로 줄어들었다.

![](/image/kakfa-consume-test-09.png)

## 주의

max.poll.records 를 default 500 그대로 사용한다고 하자.
그러면, consumer thread 하나는, polling 할 때마다 500 건을 가져올 것이다.
가져온 500 건에 대해서 한 레코드 씩, @KafkaListener 가 붙은 메서드에 전달된다.
그런데, 한 레코드를 처리하는 비즈니스 로직이 1초가 걸린다고 하자.
그러면, 500 건 모두 처리하는데는 얼마나 걸릴까 ? 500 초이다. (1초 * 500건)

이제 500 건은 모두 처리했으니,

1. commit 을 하고
2. poll 을 통해 500 건을 가져와야한다.

그런데, commit 을 할 때 다음과 같은 Exception 이 발생한다. 

> Commit cannot be completed since the group has already rebalanced and assigned the partitions to another member. This means that the time between subsequent calls to poll() was longer than the configured max.poll.interval.ms, which typically implies that the poll loop is spending too much time message processing. You can address this either by increasing max.poll.interval.ms or by reducing the maximum size of batches returned in poll() with max.poll.records.

요약하면 다음과 같다.
poll 을 통해 가져간 records 들을 모두 처리하고 commit 할 때까지의 총 소요 시간이 max.poll.interval.ms를 초과하면 commit 에 실패한다는 것이다.

max.poll.interval.ms 를 설정하지 않으면, 다음과 같이 default 로 300000 ms (300초) 이다.

![](/image/kafka-consumer-max-poll-interval.png)

500 건을 처리하는데 500 초가 걸렸으니, max.poll.interval.ms 인 300 초를 초과했다.
그래서 commit 시점에 실패하게 된다.

따라서, 

1. max.poll.interval.ms 를 비즈니스 로직을 모두 처리하는 데 걸리는 시간 이상으로 조정하거나
2. max.poll.records 를 max.poll.interval.ms 내에 처리할 수 있을 정도로 적게 설정해야한다.

---
https://kafka.apache.org/23/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html
https://docs.spring.io/spring-kafka/api/org/springframework/kafka/listener/ContainerProperties.AckMode.html