# Update Codemaps

codebase 구조 분석 및 아키텍처 문서 업데이트:

1. 모든 소스 파일에서 import, export, dependency 스캔
2. 다음 형식으로 token 효율적인 codemap 생성:
   - codemaps/architecture.md - 전체 아키텍처
   - codemaps/backend.md - 백엔드 구조
   - codemaps/frontend.md - 프론트엔드 구조
   - codemaps/data.md - 데이터 모델 및 스키마

3. 이전 버전과의 diff 백분율 계산
4. 변경이 30% 초과면 업데이트 전 사용자 승인 요청
5. 각 codemap에 최신성 타임스탬프 추가
6. .reports/codemap-diff.txt에 리포트 저장

분석에 TypeScript/Node.js 사용. 구현 세부사항이 아닌 상위 레벨 구조에 집중.
