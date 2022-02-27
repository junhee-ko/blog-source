---
layout: post 
title: Spring Kafka Test
date: 2022-02-27
categories: Kafka
---

Spring Kafka 를 사용할 때, 외부 카프카 서버에 의존하지 않고 테스트하는 방법을 정리한다. 

## Setup

spring-kafka 를 의존성으로 추가하자. 그러면, spring-kafka-test 도 같이 추가된다.
spring-kafka-test 에는 테스트를 돕는 다양한 util 이 존재한다.

![](/image/spring-kafka-test-project-setup.png)

이제 간단한 Producer 를 추가하자. 
KafkaTemplate 를 이용해서, 특정 토픽에 메세지를 전송한다.

```kotlin
import org.slf4j.LoggerFactory
import org.springframework.kafka.core.KafkaTemplate
import org.springframework.stereotype.Component

@Component
class Producer(
    private val kafkaTemplate: KafkaTemplate<String, String>
) {

    private val logger = LoggerFactory.getLogger(this.javaClass.name)

    fun produce(topic: String, message: String) {
        logger.info("Produced, (message: $message), (topic: $topic)")

        kafkaTemplate.send(topic, message)
    }
}
```

Consumer 를 추가하자. "jko-topic" 을 listen 하는 consumer 이다.
CountDownLatch 는 테스트 할 때, producer 가 send 한 record 를 consumer thread 가 consume 을 완료할 때 까지 대기하기 위해 사용된다. 

```kotlin
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.slf4j.LoggerFactory
import org.springframework.kafka.annotation.KafkaListener
import org.springframework.stereotype.Component
import java.util.concurrent.CountDownLatch

@Component
class Consumer(
    private val countDownLatch: CountDownLatch = CountDownLatch(1)
) {

    private val logger = LoggerFactory.getLogger(this.javaClass.name)
    private var message: String? = null

    @KafkaListener(topics = ["jko-topic"])
    fun consume(consumerRecord: ConsumerRecord<String, String>) {
        logger.info("Consumed, (record: $consumerRecord)")

        message = consumerRecord.value()
        countDownLatch.countDown()
    }

    fun await() = countDownLatch.await()

    fun equalsConsumedMessageWith(message: String) = this.message == message
}
```

## Test

테스트를 작성할 때의 핵심은, 외부 카프카 서버에 의존하지 않고 테스트하는 것이다.
이를 위해, spring-kafka-test 에서 지원하는 @EmbeddedKafka 를 사용한다.
@EmbeddedKafka 는, Spring for Apache Kafka 기반 테스트를 실행하는 테스트 클래스에 지정할 수 있는 어노테이션이다.
테스트를 실행할 때, in-memory kafka instance 를 사용하게 된다.

테스트 코드르 작성해보자. @EmbeddedKafka 의,
- partitions 는, 토픽 당 파티션을 몇으로 할지에 대한 설정이다.
- brokerProperties 는, broker 를 실행하기 전에 broker config 에 추가해야 하는 key=value 형식의 설정이다.

```kotlin
import org.junit.jupiter.api.Assertions.assertTrue
import org.junit.jupiter.api.Test
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.kafka.test.context.EmbeddedKafka

@SpringBootTest
@EmbeddedKafka(partitions = 1, brokerProperties = ["listeners=PLAINTEXT://localhost:9092"])
internal class SpringKafkaTest(
    @Autowired private val consumer: Consumer,
    @Autowired private val producer: Producer
) {

    @Test
    internal fun `embedded kafka 를 사용할 때, producer 가 메세지를 보내면 consumer 는 메세지를 받는다`() {
        // given
        val topic = "jko-topic"
        val message = "Messi is the best"

        // when
        producer.produce(topic, message)

        // then
        consumer.await()
        assertTrue(consumer.equalsConsumedMessageWith(message))
    }
}
```

테스틀 실행했을 때는, 아래와 같은 설정으로 EmbeddedKafka 가 동작하게 된다.

![](/image/spring-kafka-test-embedded-server-start.png)
![](/image/spring-kafka-test-embedded-server-localhost.png)

그리고, 아래와 같이 테스트가 정상적으로 통과한다.

![](/image/spring-kafka-test-pass.png)

---
- https://docs.spring.io/spring-kafka/api/org/springframework/kafka/test/context/EmbeddedKafka.html
- https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html
- https://docs.spring.io/spring-kafka/reference/html/#embedded-kafka-junit5
- https://www.baeldung.com/spring-boot-kafka-testing
