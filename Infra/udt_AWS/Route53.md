# Route53
- 도메인 등록

## 사용법
1. 도메인 생성 (가비아 구매)
2. 호스트 생성 후 라우팅 대상 값을 가비아 네임 서버로 등록
3. 레코드 생성 (test.도메인) 후 nslookup을 통해 도메인이 정상 작동하는 지 확인
4. 로드 밸랜서 생성 (다중 인스턴스)
5. 레코드 생성 후 A 타입 alias 설정을 통해 엔드 포인트 application/loadbalancer 선택 후 레코드 생성하면 전체 도메인에 대해서 내가 만들어둔 인스턴스에 접근 가능하다.

## CNAME ALIAS
- CNAME : 호스트 이름을 다른 호스트 네임으로 매핑, test.domain.com, IP4 방식
- Alias : S3, ELB CloudFront 매핑, DNS

## HTTPS 추가
1. 로드 밸랜서 보안 그룹에서 HTTPS 추가 
2. 로드 밸랜서 리스너에서 HTTPS 추가
3. 타겟 그룹에 만들어뒀던 타겟 그룹 추가 (이 통신은 내부에서 보내기에 HTTP로 동작한다.)
4. ACM 요청, (도메인.com + *.도메인.com 추가 후 DNS 인증)
5. 레코드 생성 (로드 밸랜서 alias)