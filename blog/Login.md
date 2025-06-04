# 🔴 분산 시스템에서 로그인 처리

## 🟠 토큰 발급

### 🟢 처리 방법

1. 타사 로그인 페이지를 통해 로그인 후 redirect url을 함께 전달하여 code값을 전달 받는다.
2. 전달 받은 code를 통해 token 값을 전달받을 수 있고 access token과 refresh token을 함께 전달해준다.
3. access token으로 해당 업체 user 정보를 조회하는 api 호출
4. user 정보에서 email or 연락처 정보로 내부 회원 조회
5. 회원 정보의 user_id 값을 access token으로 생성
6. refresh token을 key 값으로 redis에 access token을 저장
7. refresh token으로 토큰 재발급 시 access token을 parse 하여 user_id 값으로 재조회 후 재발급

## 🟠 중앙 관리 서버

### 🟢 처리 방법

1. 타사 로그인 페이지를 통해 로그인 후 redirect url을 함께 전달하여 code값을 전달 받는다.
2. 전달 받은 code를 통해 token 값을 전달받을 수 있고 access token과 refresh token을 함께 전달해준다.
3. access token으로 해당 업체 user 정보를 조회하는 api 호출
4. user 정보에서 email or 연락처 정보로 내부 회원 조회
5. 회원 정보의 user_id 값을 access token으로 생성
6. refresh token을 key 값으로 session에 access token을 저장
7. https의 환경에서 cors로 cookie 값이 전달되도록 설정되었을 때 response에 cookie로 http only로 내려줌
7. refresh token으로 토큰 재발급 시 access token을 parse 하여 user_id 값으로 재조회 후 재발급

### ✅ 두개의 차이

중앙 관리 서버에서 session을 통해 저장하는 방식도 있지만 redis를 통해 토큰을 관리하는 것이 가장 일반적인 방법으로 보인다. 그 이유는 session 처리 방법은 분산 처리 환경에서 session을 공유할 수 없기 때문이다.

또한 중앙 관리 서버로 사용하는 방법에서 http only 방식을 사용한다면 front에서 해당 cookie에 접근이 불가하여 보안적으로 유리하다.