# 쿼리 어노테이션 사용법
1. 사용 이유 : JPQL 이나 순수 SQL 쿼리를 작성하게 해준다. 메소드 이름 기반 쿼리 생성 한계로 인해 사용한다.
2. 동적 쿼리에는 :를 붙이고, nativeQuery = true 옵션을 통해 순수 SQL 쿼리를 작성할 수 있다.

예제)
```java
    @Query("SELECT p FROM Post p WHERE p.board.id = :boardId AND p.title LIKE %:keyword%") // 클래스명
    List<Post> findByBoardIdAndTitleContaining(@Param("boardId") Long boardId, @Param("keyword") String keyword);

    @Query(value = "SELECT * FROM posts ORDER BY RAND() LIMIT 1", nativeQuery = true) // native table 명
    Optional<Post> findRandomPost();

    @Modifying // 대량 데이터 DML
    @Transactional // 변경이므로 Transactional
    @Query("UPDATE Post p SET p.viewCount = p.viewCount + 1 WHERE p.id = :postId")
    int incrementViewCount(@Param("postId") Long postId);
```