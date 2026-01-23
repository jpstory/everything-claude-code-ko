# Everything Claude Code 기여 가이드

기여에 관심을 가져주셔서 감사합니다. 이 저장소는 Claude Code 사용자를 위한 커뮤니티 리소스입니다.

## 찾고 있는 것

### Agent

특정 작업을 잘 처리하는 새로운 agent:
- 언어별 리뷰어 (Python, Go, Rust)
- 프레임워크 전문가 (Django, Rails, Laravel, Spring)
- DevOps 전문가 (Kubernetes, Terraform, CI/CD)
- 도메인 전문가 (ML 파이프라인, 데이터 엔지니어링, 모바일)

### Skill

Workflow 정의 및 도메인 지식:
- 언어 best practice
- 프레임워크 패턴
- 테스트 전략
- 아키텍처 가이드
- 도메인별 지식

### Command

유용한 workflow를 호출하는 slash command:
- 배포 command
- 테스트 command
- 문서화 command
- 코드 생성 command

### Hook

유용한 자동화:
- lint/formatting hook
- 보안 검사
- 유효성 검증 hook
- 알림 hook

### Rule

항상 따라야 할 가이드라인:
- 보안 rule
- 코드 스타일 rule
- 테스트 요구사항
- 네이밍 규칙

### MCP 설정

새로운 또는 개선된 MCP server 설정:
- database 통합
- 클라우드 제공자 MCP
- 모니터링 tool
- 커뮤니케이션 tool

---

## 기여 방법

### 1. 저장소 fork

```bash
git clone https://github.com/YOUR_USERNAME/everything-claude-code.git
cd everything-claude-code
```

### 2. branch 생성

```bash
git checkout -b add-python-reviewer
```

### 3. 기여 내용 추가

적절한 디렉토리에 파일 배치:
- `agents/` - 새로운 agent
- `skills/` - skill (단일 .md 또는 디렉토리)
- `commands/` - slash command
- `rules/` - rule 파일
- `hooks/` - hook 설정
- `mcp-configs/` - MCP server 설정

### 4. 형식 따르기

**Agent**는 frontmatter가 있어야 합니다:

```markdown
---
name: agent-name
description: What it does
tools: Read, Grep, Glob, Bash
model: sonnet
---

여기에 지시사항...
```

**Skill**은 명확하고 실행 가능해야 합니다:

```markdown
# Skill 이름

## 사용 시점

...

## 작동 방식

...

## 예시

...
```

**Command**는 무엇을 하는지 설명해야 합니다:

```markdown
---
description: Brief description of command
---

# Command 이름

상세 지시사항...
```

**Hook**은 설명을 포함해야 합니다:

```json
{
  "matcher": "...",
  "hooks": [...],
  "description": "What this hook does"
}
```

### 5. 기여 내용 테스트

제출 전에 Claude Code에서 설정이 작동하는지 확인하세요.

### 6. PR 제출

```bash
git add .
git commit -m "Add Python code reviewer agent"
git push origin add-python-reviewer
```

그런 다음 다음 내용과 함께 PR을 엽니다:
- 무엇을 추가했는지
- 왜 유용한지
- 어떻게 테스트했는지

---

## 가이드라인

### 해야 할 것

- 설정을 집중적이고 모듈화하여 유지
- 명확한 설명 포함
- 제출 전 테스트
- 기존 패턴 따르기
- 모든 의존성 문서화

### 하지 말아야 할 것

- 민감한 데이터 포함 (API key, token, 경로)
- 과도하게 복잡하거나 틈새 설정 추가
- 테스트하지 않은 설정 제출
- 중복 기능 생성
- 대안 없이 특정 유료 서비스가 필요한 설정 추가

---

## 파일 네이밍

- 소문자와 하이픈 사용: `python-reviewer.md`
- 설명적으로 작성: `tdd-workflow.md` (not `workflow.md`)
- agent/skill 이름과 파일명 일치

---

## 질문?

이슈를 열거나 X에서 연락하세요: [@affaanmustafa](https://x.com/affaanmustafa)

---

기여해 주셔서 감사합니다. 함께 훌륭한 리소스를 만들어 봅시다.
