> 💡 이 리포지토리는 [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)의 한국어 번역본입니다.

# Everything Claude Code

**Anthropic 해커톤 수상자의 Claude Code 설정 완전 모음집.**

10개월 이상 실제 프로덕션 제품을 구축하며 매일 집중적으로 사용해온 실전 검증된 agent, skill, hook, command, rule, MCP 설정들입니다.

---

## 가이드

이 저장소는 코드만 포함되어 있습니다. 가이드에서 모든 것을 설명합니다.

### 시작점: 요약 가이드

<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />

**[The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)**

기초 내용 - 각 설정 유형의 역할, 설정 구조화 방법, context window 관리, 그리고 이 설정들의 철학을 다룹니다. **이것을 먼저 읽으세요.**

---

### 그 다음: 상세 가이드

<img width="609" height="428" alt="image" src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" />

**[The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)**

고급 기법 - token 최적화, 세션 간 memory 영속화, 검증 loop와 eval, 병렬화 전략, subagent 오케스트레이션, 지속적 학습을 다룹니다. 이 가이드의 모든 내용은 이 저장소에 작동하는 코드가 있습니다.

| 주제 | 배울 내용 |
|------|----------|
| Token 최적화 | model 선택, system prompt 슬림화, background process |
| Memory 영속화 | 세션 간 context를 자동으로 저장/로드하는 hook |
| 지속적 학습 | 세션에서 패턴을 자동 추출하여 재사용 가능한 skill로 변환 |
| 검증 Loop | checkpoint vs 연속 eval, grader 유형, pass@k 메트릭 |
| 병렬화 | Git worktree, cascade 방법, instance 확장 시점 |
| Subagent 오케스트레이션 | context 문제, iterative retrieval 패턴 |


---

## 구성 내용

```
everything-claude-code/
|-- agents/           # 위임을 위한 특화된 subagent
|   |-- planner.md           # 기능 구현 계획
|   |-- architect.md         # 시스템 설계 결정
|   |-- tdd-guide.md         # 테스트 주도 개발
|   |-- code-reviewer.md     # 품질 및 보안 리뷰
|   |-- security-reviewer.md # 취약점 분석
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 테스트
|   |-- refactor-cleaner.md  # dead code 정리
|   |-- doc-updater.md       # 문서 동기화
|
|-- skills/           # workflow 정의 및 도메인 지식
|   |-- coding-standards.md         # 언어별 best practice
|   |-- backend-patterns.md         # API, database, caching 패턴
|   |-- frontend-patterns.md        # React, Next.js 패턴
|   |-- continuous-learning/        # 세션에서 패턴 자동 추출 (상세 가이드)
|   |-- strategic-compact/          # 수동 compaction 제안 (상세 가이드)
|   |-- tdd-workflow/               # TDD 방법론
|   |-- security-review/            # 보안 체크리스트
|
|-- commands/         # 빠른 실행을 위한 slash command
|   |-- tdd.md              # /tdd - 테스트 주도 개발
|   |-- plan.md             # /plan - 구현 계획
|   |-- e2e.md              # /e2e - E2E 테스트 생성
|   |-- code-review.md      # /code-review - 품질 리뷰
|   |-- build-fix.md        # /build-fix - 빌드 오류 수정
|   |-- refactor-clean.md   # /refactor-clean - dead code 제거
|   |-- learn.md            # /learn - 세션 중 패턴 추출 (상세 가이드)
|
|-- rules/            # 항상 따라야 할 가이드라인
|   |-- security.md         # 필수 보안 검사
|   |-- coding-style.md     # 불변성, 파일 구성
|   |-- testing.md          # TDD, 80% 커버리지 요구사항
|   |-- git-workflow.md     # commit 형식, PR 프로세스
|   |-- agents.md           # subagent 위임 시점
|   |-- performance.md      # model 선택, context 관리
|
|-- hooks/            # trigger 기반 자동화
|   |-- hooks.json                # 모든 hook 설정 (PreToolUse, PostToolUse, Stop 등)
|   |-- memory-persistence/       # 세션 lifecycle hook (상세 가이드)
|   |   |-- pre-compact.sh        # compaction 전 상태 저장
|   |   |-- session-start.sh      # 이전 context 로드
|   |   |-- session-end.sh        # 종료 시 학습 내용 영속화
|   |-- strategic-compact/        # compaction 제안 (상세 가이드)
|
|-- contexts/         # 동적 system prompt 주입 context (상세 가이드)
|   |-- dev.md              # 개발 모드 context
|   |-- review.md           # 코드 리뷰 모드 context
|   |-- research.md         # 연구/탐색 모드 context
|
|-- examples/         # 예제 설정 및 세션
|   |-- CLAUDE.md           # 프로젝트 레벨 설정 예제
|   |-- user-CLAUDE.md      # 사용자 레벨 설정 예제
|   |-- sessions/           # 세션 로그 파일 예제 (상세 가이드)
|
|-- mcp-configs/      # MCP server 설정
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, Railway 등
|
|-- plugins/          # plugin 생태계 문서
    |-- README.md           # plugin, marketplace, skill 가이드
```

---

## 빠른 시작

### 1. 필요한 것 복사

```bash
# 저장소 clone
git clone https://github.com/affaan-m/everything-claude-code.git

# agent를 Claude 설정에 복사
cp everything-claude-code/agents/*.md ~/.claude/agents/

# rule 복사
cp everything-claude-code/rules/*.md ~/.claude/rules/

# command 복사
cp everything-claude-code/commands/*.md ~/.claude/commands/

# skill 복사
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. settings.json에 hook 추가

`hooks/hooks.json`의 hook을 `~/.claude/settings.json`에 복사합니다.

### 3. MCP 설정

`mcp-configs/mcp-servers.json`에서 원하는 MCP server를 `~/.claude.json`에 복사합니다.

**중요:** `YOUR_*_HERE` placeholder를 실제 API key로 교체하세요.

### 4. 가이드 읽기

진심으로 가이드를 읽으세요. 이 설정들은 context와 함께 10배 더 이해됩니다.

1. **[요약 가이드](https://x.com/affaanmustafa/status/2012378465664745795)** - 설정 및 기초
2. **[상세 가이드](https://x.com/affaanmustafa/status/2014040193557471352)** - 고급 기법 (token 최적화, memory 영속화, eval, 병렬화)

---

## 핵심 개념

### Agent

Subagent는 제한된 범위의 위임된 작업을 처리합니다. 예시:

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

당신은 시니어 코드 리뷰어입니다...
```

### Skill

Skill은 command나 agent가 호출하는 workflow 정의입니다:

```markdown
# TDD Workflow

1. interface 먼저 정의
2. 실패하는 테스트 작성 (RED)
3. 최소한의 코드 구현 (GREEN)
4. 리팩토링 (IMPROVE)
5. 80%+ 커버리지 확인
```

### Hook

Hook은 tool 이벤트에서 발동됩니다. 예시 - console.log 경고:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### Rule

Rule은 항상 따라야 할 가이드라인입니다. 모듈화하여 유지하세요:

```
~/.claude/rules/
  security.md      # 하드코딩된 secret 금지
  coding-style.md  # 불변성, 파일 제한
  testing.md       # TDD, 커버리지 요구사항
```

---

## 기여

**기여를 환영하고 권장합니다.**

이 저장소는 커뮤니티 리소스입니다. 다음을 가지고 있다면:
- 유용한 agent나 skill
- 영리한 hook
- 더 나은 MCP 설정
- 개선된 rule

기여해 주세요! 가이드라인은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.

### 기여 아이디어

- 언어별 skill (Python, Go, Rust 패턴)
- 프레임워크별 설정 (Django, Rails, Laravel)
- DevOps agent (Kubernetes, Terraform, AWS)
- 테스트 전략 (다양한 프레임워크)
- 도메인별 지식 (ML, 데이터 엔지니어링, 모바일)

---

## 배경

저는 실험 단계부터 Claude Code를 사용해왔습니다. 2025년 9월 Anthropic x Forum Ventures 해커톤에서 [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 구축하여 우승했습니다 - 전적으로 Claude Code를 사용해서요.

이 설정들은 여러 프로덕션 애플리케이션에서 실전 검증되었습니다.

---

## 중요 참고사항

### Context Window 관리

**중요:** 모든 MCP를 한꺼번에 활성화하지 마세요. 너무 많은 tool이 활성화되면 200k context window가 70k로 줄어들 수 있습니다.

경험 법칙:
- 20-30개 MCP 설정
- 프로젝트당 10개 미만 활성화
- 80개 미만 tool 활성화

사용하지 않는 것은 프로젝트 설정의 `disabledMcpServers`를 사용하여 비활성화하세요.

### 커스터마이징

이 설정들은 제 workflow에 맞습니다. 당신은:
1. 공감되는 것부터 시작하세요
2. 당신의 stack에 맞게 수정하세요
3. 사용하지 않는 것은 제거하세요
4. 당신만의 패턴을 추가하세요

---

## 링크

- **요약 가이드 (시작점):** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **상세 가이드 (고급):** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **팔로우:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## 라이선스

MIT - 자유롭게 사용하고, 필요에 맞게 수정하고, 가능하면 기여해 주세요.

---

**이 저장소가 도움이 되었다면 star를 눌러주세요. 두 가이드를 모두 읽으세요. 멋진 것을 만드세요.**
