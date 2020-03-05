# Script for Kafka Introduction
Slides: https://docs.google.com/presentation/d/17yAIQ61tyHV5oEF7tCss-YRKuYmanr70nKtxRzoudFM/edit?usp=sharing 


# Step-by-step:
1. Dependencies

        implementation 'org.springframework.kafka:spring-kafka'
        testImplementation 'org.springframework.kafka:spring-kafka-test'
2. Create test class, something like `KafkaNotificationProducerTest` for testing Producer side
or `KafkaNotificationConsumerTest` for testing consumer side, with `@SpringBootTest` annotation
3. Create empty `@Test`
3. Add `@EmbeddedKafka` to class level annotation to enable embedded in-memory Kafka
4. Create `@Configuration` for consumers and producers
    ```
    @Configuration
    public class TestKafkaConfiguration {
    
        @Autowired
        private EmbeddedKafkaBroker embeddedKafka;
    
        @Bean
        public ConsumerFactory<String, String> createConsumerFactory() {
            Map<String, Object> props = new HashMap<>();
            props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, embeddedKafka.getBrokersAsString());
            props.put(ConsumerConfig.GROUP_ID_CONFIG, "group1");
            props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
            props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
            props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
            props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
            return new DefaultKafkaConsumerFactory<>(props);
        }
    
        @Bean
        public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
            ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
            factory.setConsumerFactory(createConsumerFactory());
            return factory;
        }
    
    
        @Bean
        public ProducerFactory<String, String> producerFactory() {
            Map<String, Object> props = new HashMap<>();
            props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, embeddedKafka.getBrokersAsString());
            props.put(ProducerConfig.RETRIES_CONFIG, 0);
            props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
            props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
            return new DefaultKafkaProducerFactory<>(props);
        }
    
        @Bean
        public KafkaTemplate<String, String> kafkaTemplate(ProducerFactory<String, String> producerFactory) {
            return new KafkaTemplate<>(producerFactory);
        }
    
        @Bean
        public Consumer<String, String> consumer(ConsumerFactory<String, String> createConsumerFactory) {
            return createConsumerFactory.createConsumer();
        }
    }
    ```
6. `@Autowired` class that you want to test, but there's no class? Yes. First, you autowire it, then you create it.
7. `@Autowired` producer or consumer, depends on what you want to test
8. TDD.
9. How to test producer:
    ```
    consumer.subscribe(List.of("our_topic_with_real_name"));
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(2));
    assertFalse(records.isEmpty());

    ConsumerRecord<String, String> lastRecord = null;
    for (ConsumerRecord<String, String> record : records) {
        lastRecord = record;
    }
    assertEquals("payload", lastRecord.value());
   ```
10. Implement sendEvent method in producer class. `KafkaTemplate` are user to send events, and `Consumer` used to consumers events.
11. To configure topic on Producer side use this:
    ```
    @Bean
    public NewTopic topic() {
        return TopicBuilder.name(OUR_TOPIC_WITH_COOL_UNDERSTANDABLE_NAME)
                           .partitions(1)
                           .replicas(1)
                           .compact()
                           .build();
    }
    ```
12. ???
13. Profit! You can test whatever you want!
14. Config:
    ```
      kafka:
        bootstrap-servers: localhost:9092
        consumer:
          bootstrap-servers: localhost:9092
          group-id: appointment
          auto-offset-reset: earliest
          key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
          value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        producer:
          bootstrap-servers: localhost:9092
          key-serializer: org.apache.kafka.common.serialization.StringSerializer
          value-serializer: org.apache.kafka.common.serialization.StringSerializer
    ```
