# AI 적용하기

1. Dependency
    - OpenAi 의존성 추가

2. yaml
   ```yaml
     ai:
       openai:
         api-key: ${OPEN_API_KEY}
         chat:
           options:
             model: gpt-4o
             temperature: 0.7
             max-tokens: 1000
   ```
   
3. Config : chatClient 빈 등록 왜 자동으로 안되는 지 확인해봐야 함
   ```java
   @Configuration
   public class OpenAiConfig {
       @Bean
       public ChatClient chatClient(ChatClient.Builder builder) {
           return builder.build();
       }
   
   }
   ```
   
4. Service
   ```java
   public String getChatResponse(String prompt) {
           return chatClient.prompt()
                   .system("물은 대답에 진정성 있게 알려줘")
                   .user(prompt)
                   .call()
                   .content();
       }
   ```
5. 위 방식으로 단순 구현할 경우 answer 값을 받을 수 있지만 모델에 학습 시점이 현재가 아니기에 오늘의 날씨나 최신 정보같은 것을 알 수 없다. 그렇기에 Google Search API를 적용해서 최신 정보까지 받을 수 있도록 구현하고자 한다.

