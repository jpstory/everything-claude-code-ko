# 테스트 요구사항

## 최소 테스트 커버리지: 80%

테스트 유형 (모두 필수):
1. **Unit 테스트** - 개별 함수, 유틸리티, 컴포넌트
2. **Integration 테스트** - API endpoint, database 연산
3. **E2E 테스트** - 핵심 사용자 flow (Playwright)

## 테스트 주도 개발

필수 workflow:
1. 먼저 테스트 작성 (RED)
2. 테스트 실행 - 실패해야 함
3. 최소한의 구현 작성 (GREEN)
4. 테스트 실행 - 통과해야 함
5. 리팩토링 (IMPROVE)
6. 커버리지 확인 (80%+)

## 테스트 실패 문제 해결

1. **tdd-guide** agent 사용
2. 테스트 격리 확인
3. mock이 올바른지 검증
4. 구현 수정, 테스트 수정 아님 (테스트가 잘못된 경우 제외)

## Agent 지원

- **tdd-guide** - 새 기능에 적극적으로 사용, 테스트 먼저 작성 강제
- **e2e-runner** - Playwright E2E 테스트 전문가
