# Test Coverage

테스트 커버리지 분석 및 누락된 테스트 생성:

1. 커버리지로 테스트 실행: npm test --coverage 또는 pnpm test --coverage

2. 커버리지 리포트 분석 (coverage/coverage-summary.json)

3. 80% 커버리지 기준 미달 파일 식별

4. 커버리지 부족 파일 각각에 대해:
   - 테스트되지 않은 코드 경로 분석
   - 함수용 unit 테스트 생성
   - API용 integration 테스트 생성
   - 핵심 flow용 E2E 테스트 생성

5. 새 테스트 통과 확인

6. 전후 커버리지 메트릭 표시

7. 프로젝트가 80%+ 전체 커버리지에 도달하도록 보장

집중 영역:
- Happy path 시나리오
- 에러 처리
- Edge case (null, undefined, empty)
- 경계 조건
