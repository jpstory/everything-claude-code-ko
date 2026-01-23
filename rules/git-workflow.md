# Git Workflow

## Commit 메시지 형식

```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

참고: Attribution은 ~/.claude/settings.json에서 전역적으로 비활성화됨.

## Pull Request Workflow

PR 생성 시:
1. 전체 commit 히스토리 분석 (최신 commit만이 아님)
2. `git diff [base-branch]...HEAD`로 모든 변경 사항 확인
3. 종합적인 PR 요약 작성
4. TODO가 포함된 테스트 계획 포함
5. 새 branch인 경우 `-u` flag로 push

## 기능 구현 Workflow

1. **먼저 계획**
   - **planner** agent로 구현 계획 생성
   - 의존성과 위험 식별
   - 단계별로 분해

2. **TDD 접근**
   - **tdd-guide** agent 사용
   - 먼저 테스트 작성 (RED)
   - 테스트를 통과하도록 구현 (GREEN)
   - 리팩토링 (IMPROVE)
   - 80%+ 커버리지 확인

3. **코드 리뷰**
   - 코드 작성 직후 **code-reviewer** agent 사용
   - CRITICAL 및 HIGH 이슈 해결
   - 가능하면 MEDIUM 이슈 수정

4. **Commit & Push**
   - 상세한 commit 메시지
   - conventional commit 형식 따르기
