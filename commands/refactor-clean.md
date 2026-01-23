# Refactor Clean

테스트 검증으로 dead code를 안전하게 식별하고 제거:

1. Dead code 분석 tool 실행:
   - knip: 사용되지 않는 export 및 파일 찾기
   - depcheck: 사용되지 않는 의존성 찾기
   - ts-prune: 사용되지 않는 TypeScript export 찾기

2. .reports/dead-code-analysis.md에 종합 리포트 생성

3. 심각도별 발견 사항 분류:
   - SAFE: 테스트 파일, 사용되지 않는 유틸리티
   - CAUTION: API route, 컴포넌트
   - DANGER: 설정 파일, main entry point

4. 안전한 삭제만 제안

5. 각 삭제 전:
   - 전체 테스트 suite 실행
   - 테스트 통과 확인
   - 변경 적용
   - 테스트 재실행
   - 테스트 실패 시 롤백

6. 정리된 항목 요약 표시

테스트를 먼저 실행하지 않고는 절대 코드를 삭제하지 마세요!
