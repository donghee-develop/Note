# 린트 적용하기
1. build.gradle 에서 spotless 플러그인 작성

    ``` groovy
    plugins {  
        id 'java'  
        id 'org.springframework.boot' version '3.4.7'  
        id 'io.spring.dependency-management' version '1.1.7'  
        id 'com.diffplug.spotless'  
    }
    ```

2. gradle.properties 파일 생성 후 환경 변수 등록
    ``` groovy
    spotlessVersion=6.23.3
    ```
   
3. lint.gradle에 다음과 같이 작성한다. 다음과 같은 규칙을 정해서 코드 포매팅
    ``` groovy
    allprojects {
    
        spotless {  
            java {  
                googleJavaFormat().aosp()  
                // 아래 순서로 import문 정렬  
                importOrder('java', 'javax', 'jakarta', 'org', 'lombok', 'com.sparta')  
                // 사용하지 않는 import 제거  
                removeUnusedImports()  
                // 각 라인 끝에 있는 공백을 제거  ****
                trimTrailingWhitespace()  
                // 파일 끝에 새로운 라인 추가  
                endWithNewline()  
            }  
        }  
        tasks.named('compileJava') {  
            dependsOn 'spotlessApply'  
        }  
    }
    ```
   
4.  tasks.named 설정으로 인해 프로젝트 실행, 컴파일 시 린트가 실행된다.
    협업 시 CI/CD에 린트가 적용되어 재 검사할 경우 문제가 생길 수 있으니 매 번 커밋 전 실행 