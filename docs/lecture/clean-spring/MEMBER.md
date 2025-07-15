도메인 모델 정의 따라 쳐보기

도메인 모델은
속성 - 식별자
행위
규칙
을 가지고 있어야 한다.

회원

속성
- email - 식별자
- nickname
- password
- status

행위
- 회원 생성 : email, nickname, password, status
- 가입을 완료 시킨다

규칙
- 회원 생성후 상태는 대기 상태
- 일정 조건을 만족하면 완료가 된다

회원 상태 - enum(MemberStatus)
- pending, active, deactivated


테스트에서는 var 사용

Objects.requiredNonNull()

테스트를 만드는데 시간이 너무 오래 걸린다? - 빠르게 테스트를 만들고 테스트를 빠르게 실행되게 해라
- 라이브 템플릿 사용

package-info.java로 @NonNull을 쓰면 기본적으로 해당 패키지는 nonNull이 된다.

Assert 쓰기

PasswordEncoder class 생성