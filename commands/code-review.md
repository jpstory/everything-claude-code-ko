# Code Review

커밋되지 않은 변경 사항에 대한 종합 보안 및 품질 리뷰:

1. 변경된 파일 가져오기: git diff --name-only HEAD

2. 각 변경된 파일에서 다음 검사:

**보안 이슈 (CRITICAL):**
- 하드코딩된 credential, API key, token
- SQL injection 취약점
- XSS 취약점
- 누락된 입력 유효성 검사
- 안전하지 않은 의존성
- path traversal 위험

**코드 품질 (HIGH):**
- 50줄 초과 함수
- 800줄 초과 파일
- 4단계 초과 중첩 깊이
- 누락된 에러 처리
- console.log 문
- TODO/FIXME 주석
- public API에 누락된 JSDoc

**Best Practice (MEDIUM):**
- 변형 패턴 (대신 불변 사용)
- 코드/주석에 이모지 사용
- 새 코드에 누락된 테스트
- 접근성 이슈 (a11y)

3. 다음을 포함한 리포트 생성:
   - 심각도: CRITICAL, HIGH, MEDIUM, LOW
   - 파일 위치 및 줄 번호
   - 이슈 설명
   - 제안된 수정

4. CRITICAL 또는 HIGH 이슈가 발견되면 commit 차단

보안 취약점이 있는 코드는 절대 승인하지 마세요!
