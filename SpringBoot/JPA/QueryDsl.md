# QueryDsl 사용 이유
- Query는 휴먼 에러 잡지 못한다. Ex) 컬럼 오타
- 가독성 문제, 쿼리가 길어질 경우 +로 연결 해야 한다.
- if, choose 와 같은 옵션을 서비스에서 처리해야함 (정적 쿼리 혹은 자바 코드를 통한 정적 처리) + 마이배티스 참고)


## 설정
1. gradle
   - 다음과 같이 설정할 경우 build -> generated -> querydsl 경로에 QEntity 들이 자동으로 생성된다.
   ```groovy 
   implementation group: 'com.querydsl', name: 'querydsl-jpa', version: '5.1.0'


   def querydslDir = layout.buildDirectory.dir("generated/querydsl").get().asFile

   sourceSets {
   main.java.srcDir querydslDir
   }

   configurations {
   querydsl.extendsFrom compileClasspath
   }

   tasks.withType(JavaCompile).configureEach {
   options.generatedSourceOutputDirectory = querydslDir
   }

   clean {
   delete file(querydslDir)
   }
   ```
   
2. QueryRepository + QueryRepositoryImpl 생성 후 impl, JPAQueryFactory 주입, Entity 생성
   ```java
    private final JPAQueryFactory queryFactory;

    private static final QWatchList watchList = QWatchList.watchList;
    private static final QMatch match = QMatch.match;
    private static final QTeam homeTeam = new QTeam("homeTeam");
    private static final QTeam awayTeam = new QTeam("awayTeam");
    private static final QStadium stadium = QStadium.stadium;

    ```
   
3. 로직에 따라 BooleanBuilder 및 fetchJoin 메소드 생성 그 외 로직 QueryFactory 메소드를 통해 변수 생성 후 리턴