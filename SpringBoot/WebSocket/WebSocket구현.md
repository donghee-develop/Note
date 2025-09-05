# 스프링 부트를 통해 웹 소켓 구현하기

1. JPA 통해 유저 테이블을 만든 후 두 개의 유저를 생성하여 브라우저에서 채팅 테스트를 하고자 한다.
    - 의존성 : WebSocket, Mybatis, JPA, MySql + 스프링 웹 

2. 파일 추가
   - WebSocketConfig, 컨트롤러, 서비스, 레파지터리

   ```java
   // Config
   @Configuration
   @EnableWebSocketMessageBroker // STOMP를 사용하기 위해 선언하는 어노테이션
   public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
   
      @Override
      public void configureMessageBroker(MessageBrokerRegistry config) {
         // 메시지 브로커가 /topic으로 시작하는 주소를 구독하는 클라이언트들에게 메시지를 전달하도록 설정
         config.enableSimpleBroker("/topic");
         // 클라이언트가 서버로 메시지를 보낼 때 /app으로 시작하는 주소로 보내도록 설정
         config.setApplicationDestinationPrefixes("/app");
      }
   
      @Override
      public void registerStompEndpoints(StompEndpointRegistry registry) {
         // 클라이언트들이 웹소켓에 접속하기 위한 엔드포인트를 /ws로 설정
         // SockJS는 웹소켓을 지원하지 않는 브라우저에서도 유사한 경험을 제공
         registry.addEndpoint("/ws").withSockJS();
      }
   }
   ```
   
   ```java
   // Controller
   /*
           로직은 다음과 같다. 
           이름, 채팅 입력 -> 조회 시 이름이 없을 경우 유저 추가 그리고 채팅 저장
           서비스 생략
    */
   @Controller
   @RequiredArgsConstructor
   public class WebSocketController {
   
      private final ChatMessageRepository chatMessageRepository;
      private final UserRepository userRepository;
      /**
       * 사용자 입장 및 메시지 전송 처리
       * /app/chat.sendMessage
       */
      @MessageMapping("/chat.sendMessage")
      @SendTo("/topic/public")
      public ChatMessage sendMessage(@Payload ChatMessage chatMessage) {
         String senderName = chatMessage.getSender();
         userRepository.findByName(senderName)
                 .orElseGet(() -> {
                    User newUser = new User();
                    newUser.setName(senderName);
                    return userRepository.save(newUser);
                 });
   
         chatMessageRepository.save(chatMessage);
         return chatMessage;
      }
   }
   ```
   
   ```java
   // Entity
   @Entity
   @Table(name = "tb_chat_message")
   @Getter
   public class ChatMessage {
   
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      private String sender;
      private String content;
   }
   
   @Entity
   @Table(name = "tb_users")
   @Getter
   @AllArgsConstructor
   @NoArgsConstructor
   @Setter
   public class User {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
   
      private String name;
   
   }
   
   // Repository
   public interface UserRepository extends JpaRepository<User,Long> {
      Optional<User> findByName(String username);
   }
   
   public interface ChatMessageRepository extends JpaRepository<ChatMessage, Long> {
   }
   
   ```
   
   ```html
   <!--GPT 사용해서 간단하게 구현-->
   <!-- 동작원리
   1. stompClient.subscribe('/topic/public', onMessageReceived); : 주소 구독 통신 시도
   2. submit 클릭 시 sendMessage 호출 후 stompClient.send() 실행
   3. 컨트롤러 도달 후 서버 로직 실행
   -->
   <!DOCTYPE html>
   <html lang="ko">
   <head>
       <meta charset="UTF-8"> <title>간단한 웹소켓 채팅</title>
       <style>
           body { font-family: sans-serif; }
           #chat-container { width: 500px; margin: auto; }
           #messages { border: 1px solid #ccc; padding: 10px; height: 300px; overflow-y: scroll; margin-bottom: 10px; }
           #message-form { display: flex; }
           #message-input { flex-grow: 1; padding: 5px; }
           #name-input { margin-bottom: 10px; padding: 5px; width: 150px; }
           button { padding: 5px 10px; }
       </style>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/sockjs-client/1.5.1/sockjs.min.js"></script>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/stomp.js/2.3.3/stomp.min.js"></script>
   </head>
   <body>
   
   <div id="chat-container">
       <h2>스프링부트 웹소켓 채팅</h2>
       <div>
           <input type="text" id="name-input" placeholder="이름을 입력하세요">
       </div>
       <div id="messages"></div>
       <form id="message-form" name="message-form">
           <input type="text" id="message-input" placeholder="메시지 입력..." autocomplete="off">
           <button type="submit">전송</button>
       </form>
   </div>
   
   <script>
       const messageForm = document.querySelector('#message-form');
       const messageInput = document.querySelector('#message-input');
       const nameInput = document.querySelector('#name-input');
       const messages = document.querySelector('#messages');
   
       let stompClient = null;
   
       function connect() {
           // 1. SockJS를 사용해 웹소켓 클라이언트 생성. '/ws'는 WebSocketConfig에서 설정한 엔드포인트
           const socket = new SockJS('/ws');
           // 2. STOMP 프로토콜 위에서 동작할 클라이언트 생성
           stompClient = Stomp.over(socket);
           // 3. STOMP 클라이언트 연결
           stompClient.connect({}, onConnected, onError);
       }
   
       function onConnected() {
           console.log('서버에 연결되었습니다!');
           // 4. '/topic/public' 주소를 구독. 이 주소로 오는 메시지를 onMessageReceived 함수로 처리
           stompClient.subscribe('/topic/public', onMessageReceived);
       }
   
       function onError(error) {
           console.error('연결에 실패했습니다.', error);
       }
   
       function sendMessage(event) {
           event.preventDefault(); // 폼 기본 전송 방지
           const messageContent = messageInput.value.trim();
           const senderName = nameInput.value.trim();
   
           if (messageContent && stompClient) {
               const chatMessage = {
                   sender: senderName || '익명',
                   content: messageInput.value
               };
               // 5. 서버의 /app/chat.sendMessage로 메시지 전송
               stompClient.send("/app/chat.sendMessage", {}, JSON.stringify(chatMessage));
               messageInput.value = ''; // 입력창 비우기
           }
       }
   
       function onMessageReceived(payload) {
           const message = JSON.parse(payload.body); // 수신된 메시지(JSON) 파싱
   
           const messageElement = document.createElement('div');
           messageElement.innerHTML = `<strong>${message.sender}:</strong> ${message.content}`;
           messages.appendChild(messageElement);
           messages.scrollTop = messages.scrollHeight; // 스크롤을 맨 아래로
       }
   
       // 스크립트 시작 시 바로 연결
       connect();
       messageForm.addEventListener('submit', sendMessage, true);
   </script>
   
   </body>
   </html>
   ```
    
3. 위는 기초적인 틀만 구현했다. 확장을 한다면 어떻게 할 수 있을까?
   - /RoomId 값을 통해 방 구현할 수 있음
   - 개별 큐로 개인 메시지를 보낼 수 있음
   - 편의성 : 메시지 이력 불러오기, 입력 중 표시
   - 인증/인가 추가
   - 스케일 아웃 환경 고려, Redis 혹은 RabitMQ 사용 가능
   - 굳이 RDB에 채팅 내역을 저장할 필요가 없기 때문에 몽고 DB도 사용 가능