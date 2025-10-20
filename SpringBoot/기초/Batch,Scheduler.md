# 스프링 배치
- 대용랴 데이터 처리 프레임워크, 데이터 처리 + 반복적인 작업 시 

## 기능
- 로깅, 추적
- 트랜잭션 관리
- 작업 처리 통계
- 작업 재시작
- 건너뛰기
- 리소스 관리

## 스케줄러와 관계
- 배치의 본질은 데이터 처리 로직이다. 이 로직을 스케줄러와 함께 사용하여 대량 데이터를 일괄 처리 가능하다.

24 오전 9시 30분

## 배치 Config
```java
    private final JobRepository jobRepository; // JPA X
    private final PlatformTransactionManager transactionManager; // chuck() 사용
    private final EntityManagerFactory entityManagerFactory; // ItemReader, Writer가 DB에 접근하기 위한 객체
    private final EmailService emailService;
```
1. Job, Step
    - Chunk : 여러 개의 아이템을 묶은 덩어리, Item -(read)> 입력기 -> chunk(items) -(items)> 출력기, 아이템들을 받아 청크 단위로 만든 후 청크 단위의 트랜잭션, 커밋, 롤백을 함
    ```java
    /* 
            Job : 내부 Step을 포함하여 스타트, JobBuilder
    */
    @Bean
    public Job sendEmailJob() {
        return new JobBuilder("sendEmailJob", jobRepository)
                .start(sendEmailStep())
                .build();
    }
    /*
            Step : RPW(Reader, Processor, Write) 관리, 작업 정의, StepBuilder
     */
    @Bean
    public Step sendEmailStep() {
        return new StepBuilder("sendEmailStep", jobRepository)
                .<User, User>chunk(CHUNK_SIZE, transactionManager)
                .reader(userItemReader())
                .processor(userItemProcessor())
                .writer(userItemWriter())
                .build();
    }
    ```

2. ItemReader : 데이터를 읽어온다.
    ```java
    @Bean
        public JpaPagingItemReader<User> userItemReader() {
            // 대용량 데이터를 OOM(Out of Memory) 없이 읽기 위해 Paging Reader 사용
            return new JpaPagingItemReaderBuilder<User>()
                    .name("userItemReader")
                    .entityManagerFactory(entityManagerFactory)
                    .pageSize(CHUNK_SIZE)
                    .queryString("SELECT u FROM User u ORDER BY u.id ASC")
                    .build();
        }
    ```

3. ItemProcessor : 읽어온 데이터를 가공하여 서비스 로직을 수행한다.
    ```java
    @Bean
        public ItemProcessor<User, User> userItemProcessor() {
            return user -> {
                String title = "메일 제목" + user.getName();
                String body = "안녕하세요 " + user.getName();
    
                boolean sendSuccess = emailService.sendEmail(user.getEmail(), title, body);
    
                if (sendSuccess) {
                    user.updateLastEmailSentAt();
                    return user;
                } else {
                    // 발송 실패 시
                    System.out.println("발송 실패");
                    return null;
                }
            };
        }
    ```

4. ItemWriter : DB에 변경 사항을 저장한다.
    ```java
    @Bean
        public JpaItemWriter<User> userItemWriter() {
            return new JpaItemWriterBuilder<User>()
                    .entityManagerFactory(entityManagerFactory)
                    .build();
        }
    ```
   
5. 스케줄러 생성 : JOB + 파라미터를 통해 식별을 하여 새로운 작업임을 알림
    ```java
    @Component
    @RequiredArgsConstructor
    public class EmailJobScheduler {
    
        private final JobLauncher jobLauncher; // 어떻게 시킬지
        private final Job sendEmailJob; // 어떤 작업
    
        @Scheduled(cron = "0 * * * * *")
        public void runSendEmailJob() {
            JobParameters jobParameters = new JobParametersBuilder()
                    .addString("runTime", LocalDateTime.now().toString())
                    .toJobParameters();
    
            jobLauncher.run(sendEmailJob, jobParameters);
    
        }
    }
    ```
6. 그 외 추가적으로 이메일, 배치 관련 yaml 설정 추가, @EnableScheduling
    ```yaml
        spring:
            mail:
              host: smtp.naver.com
              username: ${MAIL_USERNAME}
              password: ${MAIL_PASSWORD}
              port: 465
              properties:
                mail.smtp.auth: true
                mail.smtp.ssl.enable: true
                mail.smtp.ssl.trust: smtp.naver.com
    
              batch:
                jdbc:
                  # BATCH_JOB_INSTANCE 등 배치 메타데이터 테이블 자동 생성 설정
                  # 'always': 앱 시작 시 항상 테이블 생성 시도 (개발/테스트용)
                  # 'never': 자동 생성 안 함 (운영용 - 수동 스크립트 실행 권장)
                  # 'embedded': 내장 DB일 때만 생성 (기본값)
                  initialize-schema: always
                job:
                  enabled: false
    ```

7. 메일 관련 비동기 처리
- @EnableAsync 추가 및 서비스에 @Async 작성 (메일 IO 많이 먹으니 비동기 처리를 통해 백그라운드에서 실행되도록 한다.)