# Main OR Important Lead Interview Quetions and Answers

#### 1. what is rebalancing in kafka how heart beat works in consumer ?
#### 2. In springboot application is having application.properties and application.yml file which one will be consider ? and why ? and how to know which one os taken first ?
#### 3. spring resilience4J vs circuit breaker in spring boot
#### 4. how to create immutable class
#### 5. purpose of functional interfaces in java and default methods in interfaces
#### 6. What is the difference JPA vs Hibernates ?
#### 7. Can you please give proper examples of Hibernate associations/mappings
#### 8. Explain in detail Twelve-Factor Methodology in a Spring Boot Microservice
#### 9. In Spring Application how shed lock works for cron jobs  can you explain in detail
#### 10. How @Transactional works in Spring Data Explain in detail
#### 11. Microservice A->B->C->D  how to handle tracing for these servces
#### 12 what is Spring react how it is working and purpose of Spring react and real time use cases
#### 13 Explain Micro service architecture and Micro service design patterns
#### 14 What is idempotency and the idempotency key are critical in any high-throughput financial/payment system
#### 15 How can you ensure cron or schedule jobs are running in on instance same cron job would be triggered on every instance unless you explicitly coordinate
#### 16 Generally fintech companies how they implemented SSO (Single Sign-On (SSO))  
#### 17 how ditributed tracing happens
#### 18 JPMC Exp Question
#### 19 Java 17+ Features: Records, sealed classes, switch expressions, var, streams vs collectors, optional, pattern matching (AWS Mini Project)
#### 20 SpringBoot+Kafka+Mongo DB: how to consume 3M records and save into mongo DB Then how to produce to 3M records to Kafka from DB Give most optimized solution with proper example with all required configurations

#### 21 In batch we have shod lock to manage one job will not trigger two time if the application deployed in mupltiple instances in same way how multiple consumers are taking / consuming if application is deployed multiple instances

#### 22 what are the partitions in oracle

#### 23 what is the difference between veiws and partions

#### 24 Q. Explain difference between CompletableFuture and Future. When would you use each?

#### 25 What is the difference between Future Object and runnable interface ? and what is the runable vs callable interfaces ?

#### 26 Some of the Lead interview questions

#### 27 How to maintain the pagination for React JS with Springboot for large data sets


#### Java8 Employee Salary questions

* https://chatgpt.com/c/68e7ba9f-91e8-8321-9b5c-14fc81c20f8d







#### 1) what is rebalancing in kafka how heart beat works in consumer ?

<img width="805" height="280" alt="image" src="https://github.com/user-attachments/assets/47feacc3-01c8-4ca4-9ab7-21fda74a4a06" />

<img width="661" height="256" alt="image" src="https://github.com/user-attachments/assets/853e9f8f-b4aa-489f-9157-e613f042847c" />

<img width="817" height="531" alt="image" src="https://github.com/user-attachments/assets/50d91bfc-412c-4475-b61c-87ac6bff4e1c" />

<img width="773" height="328" alt="image" src="https://github.com/user-attachments/assets/dd6d93b6-729c-43c5-b2a5-36e1f1309e68" />

<img width="810" height="266" alt="image" src="https://github.com/user-attachments/assets/e35f97f3-9060-4e06-aa8a-72732f4abf9c" />

<img width="811" height="576" alt="image" src="https://github.com/user-attachments/assets/daea4340-b644-4213-a982-ff4423b3060b" />

<img width="812" height="494" alt="image" src="https://github.com/user-attachments/assets/47350257-176f-48ec-96fc-215550a1f4cf" />

<img width="838" height="341" alt="image" src="https://github.com/user-attachments/assets/40c05bdb-c86a-4bb2-a363-1d3af9ddf982" />

<img width="821" height="485" alt="image" src="https://github.com/user-attachments/assets/b9d5557f-3ee9-4d32-91c2-71593b475245" />

```java
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;

import java.time.Duration;
import java.util.*;

public class RebalanceAwareConsumer {

    private static final String TOPIC = "my-topic";
    private static final String GROUP_ID = "my-group";
    private static final String BOOTSTRAP_SERVERS = "localhost:9092";

    private static final Map<TopicPartition, OffsetAndMetadata> currentOffsets = new HashMap<>();

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");

        // Heartbeat-related configs (optional tuning)
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "10000");
        props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, "3000");
        props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, "300000");

        // Disable auto-commit, we'll commit manually
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");

        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {

            consumer.subscribe(Collections.singletonList(TOPIC), new ConsumerRebalanceListener() {

                @Override
                public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                    // Called BEFORE a rebalance starts (we're about to lose these partitions)
                    System.out.println("Partitions revoked: " + partitions);
                    consumer.commitSync(currentOffsets); // commit current offsets before losing them
                }

                @Override
                public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                    // Called AFTER a rebalance completes
                    System.out.println("Partitions assigned: " + partitions);
                }
            });

            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));

                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Consumed: key=%s, value=%s, partition=%d, offset=%d%n",
                            record.key(), record.value(), record.partition(), record.offset());

                    // Track offsets for manual commit
                    TopicPartition partition = new TopicPartition(record.topic(), record.partition());
                    OffsetAndMetadata offset = new OffsetAndMetadata(record.offset() + 1);
                    currentOffsets.put(partition, offset);
                }

                // Commit after processing each batch
                if (!currentOffsets.isEmpty()) {
                    consumer.commitAsync(currentOffsets, (offsets, exception) -> {
                        if (exception != null) {
                            System.err.println("Commit failed: " + exception.getMessage());
                        }
                    });
                }
            }
        }
    }
}

```

<img width="805" height="649" alt="image" src="https://github.com/user-attachments/assets/00ab5ce2-cbbc-4d71-928c-bd33d2313952" />

<img width="795" height="422" alt="image" src="https://github.com/user-attachments/assets/62b04c1c-f071-40eb-9e59-e77d30dd45b9" />

<img width="845" height="355" alt="image" src="https://github.com/user-attachments/assets/cbcc1069-eb74-47cb-b54d-4724ac572e4c" />

<img width="831" height="413" alt="image" src="https://github.com/user-attachments/assets/a5b7e5ce-9f11-4d9d-a5f2-2f0cfa393cdd" />

<img width="808" height="552" alt="image" src="https://github.com/user-attachments/assets/2a2a012b-1867-4f00-8439-727ad0051228" />

<img width="842" height="553" alt="image" src="https://github.com/user-attachments/assets/b685d6e7-7012-4b57-b9ce-48e9e0e7ce16" />

<img width="815" height="635" alt="image" src="https://github.com/user-attachments/assets/fd34d75e-15b4-40fc-9967-cb974f70523a" />

#### 2) In springboot application is having application.properties and application.yml file which one will be consider ? and why ?

<img width="797" height="526" alt="image" src="https://github.com/user-attachments/assets/633106d4-4b50-4213-9b16-356d3d5b6e44" />

<img width="806" height="781" alt="image" src="https://github.com/user-attachments/assets/febd93a0-ac4d-4410-a071-5baeed057800" />

<img width="815" height="717" alt="image" src="https://github.com/user-attachments/assets/57be930e-ad02-471a-93f7-7466df7d73b6" />

<img width="799" height="566" alt="image" src="https://github.com/user-attachments/assets/c262cbf9-4966-428a-a081-657723ddf335" />

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.env.*;
import org.springframework.boot.CommandLineRunner;
import org.springframework.beans.factory.annotation.Autowired;

@SpringBootApplication
public class DemoApplication implements CommandLineRunner {

    @Autowired
    private ConfigurableEnvironment environment;

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    public void run(String... args) {
        printPropertySource("server.port");
        printPropertySource("custom.name");
    }

    private void printPropertySource(String key) {
        for (PropertySource<?> propertySource : environment.getPropertySources()) {
            if (propertySource.containsProperty(key)) {
                Object value = propertySource.getProperty(key);
                System.out.printf("🔍 Key: %-20s | Value: %-15s | Source: %s%n",
                        key, value, propertySource.getName());
                return;
            }
        }
        System.out.println("❌ Property not found: " + key);
    }
}

```
<img width="814" height="388" alt="image" src="https://github.com/user-attachments/assets/2a7fc961-412e-415a-bd5e-cd942051d4fa" />

<img width="802" height="462" alt="image" src="https://github.com/user-attachments/assets/6ec63e0e-f1f5-46f6-9584-485ca6e87316" />

#### 3) spring resilience vs circuit breaker in spring boot

 <img width="828" height="605" alt="image" src="https://github.com/user-attachments/assets/1fdd0695-072e-4772-8c64-1e683b01ccc9" />

 <img width="823" height="501" alt="image" src="https://github.com/user-attachments/assets/fcf0c964-6b33-4b99-9661-0f9b990fad6b" />

 <img width="838" height="260" alt="image" src="https://github.com/user-attachments/assets/5e55d158-3f3a-430e-9f2a-15c2a17d6c2b" />

 <img width="819" height="389" alt="image" src="https://github.com/user-attachments/assets/12cb6817-7c2e-412a-9035-01a470b6601f" />

 <img width="802" height="538" alt="image" src="https://github.com/user-attachments/assets/28c2fcde-53d9-4adc-bcf7-630b9a7c4dc7" />

 <img width="810" height="774" alt="image" src="https://github.com/user-attachments/assets/d6fada86-eb0f-4e75-9692-91a6a5ce0eac" />

 #### From G-Gemini Answer

 <img width="770" height="669" alt="image" src="https://github.com/user-attachments/assets/54b18c6f-4a41-41f4-beec-cdf6bc6f9539" />

 <img width="793" height="334" alt="image" src="https://github.com/user-attachments/assets/c794b2b6-af79-4568-acaa-b6edc87d6c6c" />

<img width="734" height="552" alt="image" src="https://github.com/user-attachments/assets/bb76a698-85e4-4541-97ef-391c78db991e" />

<img width="766" height="163" alt="image" src="https://github.com/user-attachments/assets/ba2eb6d6-4bea-46f8-9b16-c9826bcf2c86" />

<img width="741" height="616" alt="image" src="https://github.com/user-attachments/assets/6d5b723e-6918-4da1-bd41-ce9475b282da" />

<img width="732" height="53" alt="image" src="https://github.com/user-attachments/assets/8675f699-ddd5-4743-a185-af10fb62fdcc" />

```java
resilience4j.circuitbreaker:
  instances:
    serviceA:
      registerHealthIndicator: true
      slidingWindowSize: 10 # Number of calls to consider for failure rate
      minimumNumberOfCalls: 5 # Minimum calls before failure rate calculation starts
      failureRateThreshold: 50 # Percentage of failures to open the circuit
      waitDurationInOpenState: 5s # Time to wait in open state before half-open
      permittedNumberOfCallsInHalfOpenState: 3 # Number of calls allowed in half-open state
```
<img width="745" height="142" alt="image" src="https://github.com/user-attachments/assets/1f3480b4-0df8-4290-9462-d91e1d0524e8" />

#### OR

<img width="851" height="449" alt="image" src="https://github.com/user-attachments/assets/57bd63bb-deaa-4d2f-8ab2-9454cd7fb8e1" />

<img width="859" height="641" alt="image" src="https://github.com/user-attachments/assets/ee80af3f-5e0e-43e8-ae69-d3f3e05ba59c" />

```yml
resilience4j:
  circuitbreaker:
    instances:
      myServiceCB:
        registerHealthIndicator: true  # Enables health indicator for actuator endpoint
        slidingWindowType: COUNT_BASED  # Use COUNT_BASED (or TIME_BASED) sliding window strategy
        slidingWindowSize: 5  # Number of calls to evaluate for failure rate
        failureRateThreshold: 50  # Circuit breaker opens if failure rate exceeds this threshold (in %)
        waitDurationInOpenState: 10s # Time circuit breaker stays OPEN before switching to HALF_OPEN
        permittedNumberOfCallsInHalfOpenState: 3 # Number of calls allowed in HALF_OPEN state before determining outcome
        minimumNumberOfCalls: 5  # Minimum number of calls required to calculate failure rate
        automaticTransitionFromOpenToHalfOpenEnabled: true  # Automatically transition from OPEN to HALF_OPEN after wait duration

  retry:
    instances:
      myServiceRetry:
        maxAttempts: 3 # Max retry attempts including the initial call
        waitDuration: 2s # Wait duration between retry attempts
        retryExceptions:   # Only retry these exception types
          - java.io.IOException
        ignoreExceptions:   # Never retry for these exception types
          - java.lang.IllegalArgumentException

  ratelimiter:
    instances:
      myServiceRateLimiter:  
        limitForPeriod: 5    # Max number of calls allowed per refresh period
        limitRefreshPeriod: 1s
        timeoutDuration: 500ms

  bulkhead:
    instances:
      myServiceBulkhead:
        maxConcurrentCalls: 5
        maxWaitDuration: 500ms  # Max time a thread waits for permission if rate limit is exceeded

  thread-pool-bulkhead:
    instances:
      myServiceThreadPool:
        coreThreadPoolSize: 5   # Core thread pool size (kept alive always)
        maxThreadPoolSize: # Maximum thread pool size (burst capacity)
        queueCapacity: 20  # Queue size to hold tasks before rejection

  timelimiter:
    instances:
      myServiceTimeLimiter:
        timeoutDuration: 2s    # Max duration allowed for a call before timing out
        cancelRunningFuture: true # Cancel the running future task on timeout

```

<img width="850" height="586" alt="image" src="https://github.com/user-attachments/assets/1e66acb9-7af1-47c3-ab2a-94e42c2b6538" />

<img width="906" height="415" alt="image" src="https://github.com/user-attachments/assets/9f25c5e3-6fe9-4ae7-bec6-5bff8b723803" />

<img width="819" height="389" alt="image" src="https://github.com/user-attachments/assets/47a36867-fefa-4eb7-8529-2f85f82247d0" />


#### 4) how to create immutable class

<img width="795" height="569" alt="image" src="https://github.com/user-attachments/assets/06ab7d51-b1aa-4ce7-8347-5a07834f5a13" />

```java
public final class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Only getters, no setters
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}

```
<img width="761" height="64" alt="image" src="https://github.com/user-attachments/assets/8fb7ac62-46b3-42f9-924f-d9b98b309a28" />

```java
import java.util.Date;

public final class Employee {
    private final String name;
    private final Date joiningDate; // Mutable

    public Employee(String name, Date joiningDate) {
        this.name = name;
        // Defensive copy
        this.joiningDate = new Date(joiningDate.getTime());
    }

    public String getName() {
        return name;
    }

    public Date getJoiningDate() {
        // Return defensive copy
        return new Date(joiningDate.getTime());
    }
}

```

<img width="827" height="334" alt="image" src="https://github.com/user-attachments/assets/69cbb6a1-a7e7-42d3-8fc4-c811198499db" />

<img width="852" height="604" alt="image" src="https://github.com/user-attachments/assets/f76b4cb1-c1ec-438b-b624-d717d253648d" />

<img width="847" height="460" alt="image" src="https://github.com/user-attachments/assets/2f1057e3-d85a-4ee3-b628-ea85e5121f19" />

<img width="785" height="767" alt="image" src="https://github.com/user-attachments/assets/1a1ace7e-97ef-4aeb-92d6-e06925cc865a" />

#### 5) purpose of functional interfaces in java

<img width="829" height="495" alt="image" src="https://github.com/user-attachments/assets/2550597b-1433-4412-a7ef-6f9a9bf5deb1" />

<img width="838" height="398" alt="image" src="https://github.com/user-attachments/assets/1e1ca77a-afaf-4487-9636-8852a4c46508" />

<img width="809" height="708" alt="image" src="https://github.com/user-attachments/assets/86ad5eff-f056-4b7d-8125-109416ed8f1a" />

<img width="814" height="430" alt="image" src="https://github.com/user-attachments/assets/c98d1c12-e480-4799-b830-844ce9a8b133" />

<img width="830" height="298" alt="image" src="https://github.com/user-attachments/assets/da70c7e0-18c0-45ea-811d-45d43e8b93fa" />

<img width="869" height="556" alt="image" src="https://github.com/user-attachments/assets/3c06a011-d806-4fca-a26c-b8cba1278321" />

<img width="796" height="709" alt="image" src="https://github.com/user-attachments/assets/890b7125-081d-43f5-b456-c01fcfebd6c2" />

<img width="822" height="643" alt="image" src="https://github.com/user-attachments/assets/d0f349fe-612e-40ed-8228-9b35600621c4" />

<img width="824" height="757" alt="image" src="https://github.com/user-attachments/assets/9ce8f54b-8ba8-4ada-8554-7cce13e67809" />

#### 6. What is the difference JPA vs Hibernates ?

<img width="798" height="622" alt="image" src="https://github.com/user-attachments/assets/d5261bc3-8f0a-4505-b283-d25717c4e12c" />

<img width="849" height="528" alt="image" src="https://github.com/user-attachments/assets/d9260ced-e45c-4f44-8c49-3ab0cdf3bc57" />

<img width="819" height="388" alt="image" src="https://github.com/user-attachments/assets/9907fd02-468f-402a-95a3-f0d6e93f0851" />

<img width="835" height="695" alt="image" src="https://github.com/user-attachments/assets/92c54d2a-3b44-40dc-9236-cdad931501b8" />

<img width="883" height="595" alt="image" src="https://github.com/user-attachments/assets/fb36cc2f-1852-4bc1-bf48-4bd6fff4d39a" />

<img width="868" height="655" alt="image" src="https://github.com/user-attachments/assets/bdd3f089-d57d-41b3-8948-b6a3b8311ef3" />

<img width="830" height="383" alt="image" src="https://github.com/user-attachments/assets/6b475b14-17b3-47b7-9cc2-a979da10afa8" />

<img width="804" height="423" alt="image" src="https://github.com/user-attachments/assets/d47a253a-c081-4ee4-9670-5743ba0ed79f" />

<img width="802" height="484" alt="image" src="https://github.com/user-attachments/assets/45f65d26-14d9-4163-b5fc-d97067702010" />

<img width="802" height="346" alt="image" src="https://github.com/user-attachments/assets/5e6a66ee-2562-4c40-8911-c4ca36631521" />

<img width="845" height="746" alt="image" src="https://github.com/user-attachments/assets/083bd77a-0c24-42ab-9ab3-1c850d4dbc6e" />

<img width="807" height="661" alt="image" src="https://github.com/user-attachments/assets/e7a8f09b-a3ba-4be2-97cd-afb7e9d223dd" />

<img width="813" height="580" alt="image" src="https://github.com/user-attachments/assets/d6e81dc1-17e9-4961-860e-d2ccf70c9baf" />

<img width="839" height="509" alt="image" src="https://github.com/user-attachments/assets/8533fafd-e150-4227-8a98-3a3c0e430222" />

#### Code Example

```java
// IMPORTANT: Use jakarta.persistence instead of javax.persistence for Spring Boot 3+
import jakarta.persistence.CascadeType;
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.JoinTable;
import jakarta.persistence.ManyToMany;
import jakarta.persistence.ManyToOne;
import jakarta.persistence.OneToMany;
import jakarta.persistence.OneToOne;
import jakarta.persistence.Table;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

/**
 * ========================================
 * 1. ONE-TO-ONE MAPPING
 * ========================================
 *
 * Example: A User has one and only one UserProfile.
 * The User entity is the "owning" side, as it contains the foreign key.
 * The UserProfile entity is the "inverse" side.
 */

// User.java - The Owning Side
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username")
    private String username;

    /**
     * @OneToOne: Defines a one-to-one relationship.
     * cascade = CascadeType.ALL: If a User is saved or deleted, the associated
     * UserProfile is also saved or deleted. This simplifies persistence.
     * @JoinColumn(name = "user_profile_id"): Specifies the foreign key column
     * in the `users` table that links to the `user_profiles` table's primary key.
     */
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "user_profile_id", referencedColumnName = "id")
    private UserProfile userProfile;

    // Getters and Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public UserProfile getUserProfile() { return userProfile; }
    public void setUserProfile(UserProfile userProfile) { this.userProfile = userProfile; }
}

// UserProfile.java - The Inverse Side
@Entity
@Table(name = "user_profiles")
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "bio")
    private String bio;

    /**
     * @OneToOne(mappedBy = "userProfile"): Declares that this is the inverse
     * side of the relationship. The `mappedBy` attribute points to the
     * field name in the owning entity (the `User` class) that manages the relationship.
     * This means the foreign key is in the `users` table, not here.
     */
    @OneToOne(mappedBy = "userProfile")
    private User user;

    // Getters and Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getBio() { return bio; }
    public void setBio(String bio) { this.bio = bio; }
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
}

/**
 * ========================================
 * 2. ONE-TO-MANY / MANY-TO-ONE MAPPING
 * ========================================
 *
 * Example: A Department has many Employees, but an Employee belongs to only one Department.
 * The Employee entity is the "owning" side of the relationship.
 */

// Department.java - The One-to-Many Side (Inverse)
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name")
    private String name;

    /**
     * @OneToMany(mappedBy = "department"): This is the inverse side. It points to
     * the `department` field in the `Employee` entity which owns the relationship.
     * The `cascade` and `orphanRemoval` attributes ensure that when a Department
     * is deleted, all its associated Employees are also deleted.
     */
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees;

    // Getters and Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public List<Employee> getEmployees() { return employees; }
    public void setEmployees(List<Employee> employees) { this.employees = employees; }
}

// Employee.java - The Many-to-One Side (Owning)
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    /**
     * @ManyToOne: Defines a many-to-one relationship.
     * @JoinColumn(name = "department_id"): Specifies the foreign key column in the
     * `employees` table that links to the `departments` table.
     */
    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    // Getters and Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public Department getDepartment() { return department; }
    public void setDepartment(Department department) { this.department = department; }
}

/**
 * ========================================
 * 3. MANY-TO-MANY MAPPING
 * ========================================
 *
 * Example: A Student can enroll in many Courses, and a Course can have many Students.
 * The `Student` entity is the "owning" side of the relationship.
 */

// Student.java - The Owning Side
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name")
    private String name;

    /**
     * @ManyToMany: Defines a many-to-many relationship.
     * @JoinTable: Specifies the intermediate "join" table that will be created
     * to manage the relationship.
     * name = "student_course": The name of the join table.
     * joinColumns = @JoinColumn(name = "student_id"): The column in the join table
     * that references the `Student` entity's primary key.
     * inverseJoinColumns = @JoinColumn(name = "course_id"): The column in the
     * join table that references the `Course` entity's primary key.
     */
    @ManyToMany(cascade = { CascadeType.PERSIST, CascadeType.MERGE })
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    // Getters and Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public Set<Course> getCourses() { return courses; }
    public void setCourses(Set<Course> courses) { this.courses = courses; }

    public void addCourse(Course course) {
        this.courses.add(course);
        course.getStudents().add(this);
    }

    public void removeCourse(Course course) {
        this.courses.remove(course);
        course.getStudents().remove(this);
    }
}

// Course.java - The Inverse Side
@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "title")
    private String title;

    /**
     * @ManyToMany(mappedBy = "courses"): This is the inverse side, managed by the
     * `courses` field in the `Student` entity.
     */
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();

    // Getters and Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public Set<Student> getStudents() { return students; }
    public void setStudents(Set<Student> students) { this.students = students; }
}
```



#### 7. Can you please give proper examples of Hibernate associations/mappings 

<img width="820" height="331" alt="image" src="https://github.com/user-attachments/assets/3c697e15-7bea-4b3f-8898-c5088dd87b3e" />

<img width="811" height="549" alt="image" src="https://github.com/user-attachments/assets/d7b36630-fd30-4c42-87f5-b854bf041daa" />

<img width="800" height="663" alt="image" src="https://github.com/user-attachments/assets/337c2543-9138-4792-928f-b7054cb36fdf" />

<img width="812" height="196" alt="image" src="https://github.com/user-attachments/assets/fbec4550-7f4b-4df6-8d17-16b7f867eb75" />

<img width="818" height="718" alt="image" src="https://github.com/user-attachments/assets/30cc530f-5df9-4b80-80e1-9bd51703b13f" />

<img width="768" height="530" alt="image" src="https://github.com/user-attachments/assets/5b1baf9b-1413-4c73-91a1-4e5de677c818" />

<img width="818" height="582" alt="image" src="https://github.com/user-attachments/assets/c91f3ab6-e8a9-44aa-bfe4-d89c9066183b" />

<img width="804" height="406" alt="image" src="https://github.com/user-attachments/assets/34cb9a85-40c0-4a4e-81e5-9aa162b3c27c" />

#### 8. Explain in detail Twelve-Factor Methodology in a Spring Boot Microservice

<img width="844" height="444" alt="image" src="https://github.com/user-attachments/assets/5be1454a-6667-4c6a-b4f8-8c0150f43996" />

<img width="836" height="549" alt="image" src="https://github.com/user-attachments/assets/e4acce6c-ef0b-45c4-9db5-412d15b50f31" />

<img width="710" height="543" alt="image" src="https://github.com/user-attachments/assets/4f4809e0-0bc9-4749-8d61-8542c36c573b" />

<img width="830" height="385" alt="image" src="https://github.com/user-attachments/assets/6c76b6af-f792-4f2d-b73d-6a63a53d6752" />

<img width="826" height="464" alt="image" src="https://github.com/user-attachments/assets/76901d77-8307-440b-8647-05a2f2e969ac" />

<img width="824" height="390" alt="image" src="https://github.com/user-attachments/assets/98e21917-ab71-413e-a80c-ed6270d77840" />

<img width="883" height="581" alt="image" src="https://github.com/user-attachments/assets/bb59cd71-51f3-48ee-9d34-e91e2c37a586" />

<img width="926" height="719" alt="image" src="https://github.com/user-attachments/assets/25120ecc-18cb-46f7-87c7-45ec9e1d685d" />

<img width="803" height="349" alt="image" src="https://github.com/user-attachments/assets/9bbd0eb6-9fc1-472a-b7dd-f25a023e9ae7" />

<img width="844" height="540" alt="image" src="https://github.com/user-attachments/assets/e56cf0d3-1f8a-4576-9bef-5dc8fec7dc77" />

<img width="820" height="499" alt="image" src="https://github.com/user-attachments/assets/c3597c4d-7214-4c2b-a2f8-c5f83612b63a" />

<img width="816" height="639" alt="image" src="https://github.com/user-attachments/assets/7941bc2e-ec20-4020-99f1-73d76dac18f7" />

#### 9. In Spring Application how shed lock works for cron jobs  can you explain in detail

<img width="826" height="352" alt="image" src="https://github.com/user-attachments/assets/96c353cd-6249-4dbc-8eef-2d582011200e" />

<img width="802" height="566" alt="image" src="https://github.com/user-attachments/assets/bf5bbc88-f0b3-4256-8435-d17f4ed26375" />

<img width="789" height="457" alt="image" src="https://github.com/user-attachments/assets/c161600f-78d3-4dc3-9ec0-5f71277327e8" />

<img width="795" height="725" alt="image" src="https://github.com/user-attachments/assets/0345804a-7123-49ad-9ed4-7d573fd66664" />

<img width="806" height="602" alt="image" src="https://github.com/user-attachments/assets/97a2e205-878f-4a75-a0ca-ba1db4632e81" />

<img width="798" height="615" alt="image" src="https://github.com/user-attachments/assets/57e59fb3-c171-42cc-8385-bbc90d42f54d" />

#### 10. How @Transactional works in Spring Data Explain in detail

<img width="819" height="372" alt="image" src="https://github.com/user-attachments/assets/070f27fd-16b8-494d-8ea5-935dfd598850" />

<img width="796" height="668" alt="image" src="https://github.com/user-attachments/assets/a9db03cd-716f-4720-bee5-728e5fd9bd27" />

<img width="794" height="730" alt="image" src="https://github.com/user-attachments/assets/1ecf1a21-e2af-43e9-9e58-b4973ee9c5bc" />

<img width="801" height="775" alt="image" src="https://github.com/user-attachments/assets/5f7081ad-c377-4ef4-ab0a-ed561a1304e6" />

<img width="817" height="688" alt="image" src="https://github.com/user-attachments/assets/4efd8841-d0dc-454f-a0b9-2535a9198e46" />

<img width="811" height="467" alt="image" src="https://github.com/user-attachments/assets/1ad422ca-21b1-4b39-8194-0ce7aaeb1856" />

<img width="826" height="418" alt="image" src="https://github.com/user-attachments/assets/d92a6b22-bd7b-4695-a5c3-05a3dc64dbd3" />


#### 11. Microservice A->B->C->D  how to handle tracing for these servces

* Chat GPT -> https://chatgpt.com/c/68ab00ba-0e54-8332-bd77-e43e528e31bc
* Perplexity -> https://www.perplexity.ai/search/microservice-a-b-c-d-how-to-ha-5tKcUU1CSS.dZdiM.EQOdQ
* ef -> https://chatgpt.com/c/68e7b686-b550-8321-b502-c6c6be392564

#### 12 what is Spring react how it is working and purpose of Spring react and real time use cases
* chat GPT -> https://chatgpt.com/c/68ab0328-9080-8333-88a8-480b5ac16fdb
* Perplecity -> https://www.perplexity.ai/search/what-is-spring-reactive-how-it-QtS9SWaARm2vXBSCZBsdhA

#### 13 Explain Micro service architecture and Micro service design patterns
* CHat GPT -> https://chatgpt.com/c/68ab056e-f544-8323-97a0-86ccaa7a450c
* Perplexity -> https://www.perplexity.ai/search/explain-micro-service-architec-dIrOr0FHR8yiiPkM0fqlmg

#### 14 What is idempotency and the idempotency key are critical in any high-throughput financial/payment system

<img width="889" height="772" alt="image" src="https://github.com/user-attachments/assets/d7d5c4db-5226-431e-9c92-750578efa317" />
<img width="870" height="709" alt="image" src="https://github.com/user-attachments/assets/58de0147-004a-4c3e-8c8e-f455d02564d9" />
For More details -> https://chatgpt.com/c/68d6785f-a820-832d-972e-5d93dd7a7ac0

#### 15 How can you ensure cron or schedule jobs are running in on instance same cron job would be triggered on every instance unless you explicitly coordinate

<img width="845" height="702" alt="image" src="https://github.com/user-attachments/assets/abb93241-a9ed-4a50-9de7-61967a542a30" />
<img width="796" height="686" alt="image" src="https://github.com/user-attachments/assets/267c5161-e645-4f5e-8c26-b070d94ba4d4" />
<img width="821" height="766" alt="image" src="https://github.com/user-attachments/assets/e362c5d3-32f0-475f-a0b3-7af80975b60c" />
<img width="814" height="792" alt="image" src="https://github.com/user-attachments/assets/918c787a-824f-4f2e-b779-a2cb446de6f6" />
<img width="824" height="466" alt="image" src="https://github.com/user-attachments/assets/ecfe0789-6226-44b6-8555-723f3b73ff37" />

* Spring boot example on schodLock -> https://chatgpt.com/c/68d6ac66-7918-8323-861b-8e597db760ad

#### 16 Generally fintech companies how they implemented SSO (Single Sign-On (SSO))   
* ans from chatgpt https://chatgpt.com/c/68d6b0e4-7574-8322-97d6-bb5916b0d858

#### 17 how ditributed tracing happens
* https://chatgpt.com/c/68d6b218-0edc-8332-bce3-84eb9192dda3

#### 18 JPMC Exp Question

<img width="852" height="615" alt="image" src="https://github.com/user-attachments/assets/c70ccc98-a5cf-4127-9a1a-27c35e452688" />
<img width="835" height="422" alt="image" src="https://github.com/user-attachments/assets/5b4b83d9-4cf4-4394-83d6-ac306d8bfc27" />
<img width="808" height="663" alt="image" src="https://github.com/user-attachments/assets/f5786b43-7b9d-47e4-bf6d-047b3fb9e654" />
<img width="823" height="688" alt="image" src="https://github.com/user-attachments/assets/d3e3b52f-bbe6-4266-9be7-871f19b57cd4" />
<img width="806" height="223" alt="image" src="https://github.com/user-attachments/assets/99241a38-d13a-4f87-8cf8-00b6f094157e" />

* -> https://chatgpt.com/c/68da50f3-da68-8327-b4da-f87098f1b583

#### 19 Java 17+ Features: Records, sealed classes, switch expressions, var, streams vs collectors, optional, pattern matching

* Ans -> https://chatgpt.com/c/68e5b79d-10ec-8321-b8ca-0f735d5e34b6

#### 20 SpringBoot+Kafka+Mongo DB: how to consume 3M records and save into mongo DB Then how to produce to 3M records to Kafka from DB Give most optimized solution with proper example with all required configurations

* in Details -> https://chatgpt.com/c/68e781ad-7f04-8323-a383-1b59d78cceda

#### 21 what is consumer lag ?

* In Detail -> https://chatgpt.com/c/68e7858a-fb9c-8322-9053-b5b8307685a3

#### 21 In batch we have shod lock to manage one job will not trigger two time if the application deployed in mupltiple instances in same way how multiple consumers are taking / consuming if application is deployed multiple instances

<img width="898" height="432" alt="image" src="https://github.com/user-attachments/assets/f36d6c0d-3aa7-42a8-a1e9-34a5b2dcd1ca" />
<img width="905" height="736" alt="image" src="https://github.com/user-attachments/assets/31009a4f-436a-4679-a9a7-9827e4f2d053" />
<img width="894" height="623" alt="image" src="https://github.com/user-attachments/assets/b5e170d3-a954-436f-9f3c-f63a976e036f" />

* In Detail -> https://chatgpt.com/c/68e7b205-1ff0-8321-8c1f-6f6c73fe24f1

#### 22 what are the partitions in oracle

<img width="881" height="401" alt="image" src="https://github.com/user-attachments/assets/7c57ec56-823c-46e8-96bc-efdb05be9151" />

<img width="465" height="352" alt="image" src="https://github.com/user-attachments/assets/0e51afc0-1389-4d90-b045-04f9d738f259" />

<img width="1221" height="565" alt="image" src="https://github.com/user-attachments/assets/4e1b0594-0263-4d7f-a603-9336e082b440" />

<img width="1185" height="590" alt="image" src="https://github.com/user-attachments/assets/4684bec6-eabc-4626-ade5-68586b748afa" />

<img width="1060" height="324" alt="image" src="https://github.com/user-attachments/assets/57c1787a-1609-44aa-ae6a-a760ea8c3411" />


#### 23 what is the difference between veiws and partions

<img width="889" height="661" alt="image" src="https://github.com/user-attachments/assets/53963dfc-8970-4696-b2db-8beba865d61a" />
<img width="879" height="459" alt="image" src="https://github.com/user-attachments/assets/12a51170-8b01-40b9-950d-0df699733f58" />
<img width="893" height="565" alt="image" src="https://github.com/user-attachments/assets/d7289720-729d-48db-90b5-5dcd5c069472" />
<img width="881" height="240" alt="image" src="https://github.com/user-attachments/assets/f39b7c86-cdf6-4ae7-86ea-0d3292d429fc" />

<img width="883" height="639" alt="image" src="https://github.com/user-attachments/assets/93998846-6cd3-4370-ae4b-58e577de6f73" />
<img width="883" height="647" alt="image" src="https://github.com/user-attachments/assets/5b6a9a7a-245c-4970-a289-c63a2a5e2b94" />
<img width="882" height="435" alt="image" src="https://github.com/user-attachments/assets/fc1a3b96-ded3-4eba-b85e-3996d512c935" />

#### 24 Q. Explain difference between CompletableFuture and Future. When would you use each?

<img width="738" height="717" alt="image" src="https://github.com/user-attachments/assets/236827e7-ec9b-4040-9bbd-41cdf6561ca8" />
<img width="752" height="614" alt="image" src="https://github.com/user-attachments/assets/e76750e5-c7bc-4b9a-b400-54d707402743" />
<img width="683" height="477" alt="image" src="https://github.com/user-attachments/assets/c0fd2b86-b384-45e4-8948-e874a0130ac4" />
<img width="752" height="422" alt="image" src="https://github.com/user-attachments/assets/dbe9cf10-0777-4457-9ac5-077e75461be5" />

#### 25 What is the difference between Future Object and runnable interface ? and what is the runable vs callable interfaces ?

<img width="740" height="393" alt="image" src="https://github.com/user-attachments/assets/69471bb5-7d71-4c73-8984-85b972797044" />
<img width="735" height="474" alt="image" src="https://github.com/user-attachments/assets/8f914dbf-065e-4156-8176-a3e90fc216c6" />
<img width="731" height="567" alt="image" src="https://github.com/user-attachments/assets/abfee9c1-3cd4-481b-b3bf-7f00908d9e5b" />
<img width="746" height="490" alt="image" src="https://github.com/user-attachments/assets/cb502d9e-4922-4c8c-a3fc-ce37121c769c" />
<img width="764" height="683" alt="image" src="https://github.com/user-attachments/assets/55f7cb91-2423-484c-a69a-591d35f1aded" />
<img width="687" height="780" alt="image" src="https://github.com/user-attachments/assets/9d4ca97c-fe00-454e-af7c-b8a7301220cf" />


#### 26 the Lead interview questions

<img width="872" height="391" alt="image" src="https://github.com/user-attachments/assets/5ae23595-a6f6-4c82-898a-d4bbfed8bd73" />
<img width="855" height="472" alt="image" src="https://github.com/user-attachments/assets/822b4687-622a-47a6-911c-0977845812d5" />
<img width="900" height="338" alt="image" src="https://github.com/user-attachments/assets/e2487c1a-bf88-4547-8eb9-b800afe2bbd7" />
<img width="880" height="419" alt="image" src="https://github.com/user-attachments/assets/52ff2ceb-eead-4a7b-8447-9d98ce5ef504" />
<img width="836" height="347" alt="image" src="https://github.com/user-attachments/assets/00c6bd58-9fcb-4f76-9435-188c8b45ff2b" />
<img width="881" height="518" alt="image" src="https://github.com/user-attachments/assets/7ac1535b-e2bd-4a13-897a-0e7d18eb2c6b" />
<img width="878" height="699" alt="image" src="https://github.com/user-attachments/assets/31514fb7-2e63-46de-976c-0049d40d1cdd" />
<img width="869" height="391" alt="image" src="https://github.com/user-attachments/assets/72df14c0-945d-4bc7-a539-b84f50eb74e8" />

#### 27 How to maintain the pagination for React JS with Springboot for large data sets

<img width="879" height="515" alt="image" src="https://github.com/user-attachments/assets/2104992d-f72f-4df8-82b2-6611edebf93c" />
<img width="874" height="510" alt="image" src="https://github.com/user-attachments/assets/ac6487b0-e804-4189-a301-2d1bb52b50ae" />
<img width="865" height="614" alt="image" src="https://github.com/user-attachments/assets/ea5b5f4c-225e-4887-ae18-4f7caefd5e7e" />
<img width="859" height="691" alt="image" src="https://github.com/user-attachments/assets/9db17d81-86ad-4876-acb4-73e40eef8be2" />
<img width="878" height="659" alt="image" src="https://github.com/user-attachments/assets/df5d7848-e86e-4938-81e1-2e62775a6aa6" />
<img width="874" height="602" alt="image" src="https://github.com/user-attachments/assets/1961c010-6287-4b21-80c7-256d77258a3a" />
<img width="884" height="707" alt="image" src="https://github.com/user-attachments/assets/5027b229-2431-4c40-866a-f866c2fce19e" />
<img width="889" height="494" alt="image" src="https://github.com/user-attachments/assets/d3f5dde7-a143-43e3-81e4-30223578deaa" />
<img width="859" height="419" alt="image" src="https://github.com/user-attachments/assets/5b246004-1d21-4ae6-8b36-45026a8034a2" />

  #### Would you like me to show cursor-based pagination (keyset) example in both Spring Boot + React (for ultra-large datasets like 10M records)?
    -> https://chatgpt.com/c/690d9321-2b7c-8324-8212-8e58457459cc
