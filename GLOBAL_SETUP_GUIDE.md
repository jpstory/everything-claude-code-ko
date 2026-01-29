# Claude Code Global 설정 가이드

이 문서는 `everything-claude-code-ko` 프로젝트의 모든 설정을 Claude Code에서 **전역(Global)**으로 사용하기 위한 상세 가이드입니다.

## 목차

- [개요](#개요)
- [디렉토리 구조](#디렉토리-구조)
- [1. Agents 설정](#1-agents-설정)
- [2. Skills 설정](#2-skills-설정)
- [3. Commands 설정](#3-commands-설정)
- [4. Hooks 설정](#4-hooks-설정)
- [5. MCP Servers 설정](#5-mcp-servers-설정)
- [6. Rules 설정](#6-rules-설정)
- [7. Contexts 설정](#7-contexts-설정)
- [8. 사용자 CLAUDE.md 설정](#8-사용자-claudemd-설정)
- [전체 설치 스크립트](#전체-설치-스크립트)
- [설치 후 추가 작업](#설치-후-추가-작업)
- [요약](#요약)

---

## 개요

Claude Code는 `~/.claude/` 디렉토리에 저장된 설정을 **모든 프로젝트에서 전역으로** 사용할 수 있습니다. 이 가이드는 프로젝트에 포함된 모든 설정 요소를 전역으로 설치하는 방법을 설명합니다.

### 전역 설정의 장점

- 모든 프로젝트에서 동일한 agents, skills, commands 사용 가능
- 일관된 코딩 스타일과 규칙 적용
- 반복적인 설정 작업 제거
- 개인화된 워크플로우 구축

---

## 디렉토리 구조

설치 후 `~/.claude/` 디렉토리 구조:

```
~/.claude/
├── CLAUDE.md              # 사용자 레벨 설정
├── settings.json          # Hooks 및 기타 설정
├── agents/                # Agent 정의 파일들
│   ├── planner.md
│   ├── architect.md
│   ├── tdd-guide.md
│   ├── code-reviewer.md
│   ├── security-reviewer.md
│   ├── build-error-resolver.md
│   ├── e2e-runner.md
│   ├── refactor-cleaner.md
│   └── doc-updater.md
├── skills/                # Skill 정의 파일들
│   ├── backend-patterns.md
│   ├── frontend-patterns.md
│   ├── coding-standards.md
│   ├── clickhouse-io.md
│   ├── project-guidelines-example.md
│   ├── tdd-workflow/
│   ├── security-review/
│   ├── continuous-learning/
│   └── strategic-compact/
├── commands/              # Slash command 파일들
│   ├── plan.md
│   ├── tdd.md
│   ├── code-review.md
│   └── ...
├── rules/                 # Rule 정의 파일들
│   ├── security.md
│   ├── coding-style.md
│   └── ...
├── contexts/              # Context 파일들
│   ├── dev.md
│   ├── review.md
│   └── research.md
└── hooks/                 # Hook 스크립트들
    ├── memory-persistence/
    └── strategic-compact/

~/.claude.json             # MCP 서버 설정
```

---

## 1. Agents 설정

### 대상 파일 (9개)

| Agent | 파일 | 용도 | 사용 시점 |
|-------|------|------|----------|
| **planner** | `planner.md` | 기능 구현 계획 수립 | 새 기능, 복잡한 리팩토링 |
| **architect** | `architect.md` | 시스템 설계 및 아키텍처 | 아키텍처 결정 |
| **tdd-guide** | `tdd-guide.md` | 테스트 주도 개발 | 새 기능, 버그 수정 |
| **code-reviewer** | `code-reviewer.md` | 코드 품질 및 보안 리뷰 | 코드 작성 직후 |
| **security-reviewer** | `security-reviewer.md` | 보안 취약점 분석 | commit 전 |
| **build-error-resolver** | `build-error-resolver.md` | 빌드/TypeScript 오류 해결 | 빌드 실패 시 |
| **e2e-runner** | `e2e-runner.md` | Playwright E2E 테스트 | E2E 테스트 실행 |
| **refactor-cleaner** | `refactor-cleaner.md` | Dead code 정리 | 코드 유지보수 |
| **doc-updater** | `doc-updater.md` | 문서 동기화 | 문서 업데이트 |

### 설치 명령어

```bash
mkdir -p ~/.claude/agents
cp agents/*.md ~/.claude/agents/
```

### Agent 특징

- 각 agent는 특정 도구 세트 사용 (Read, Grep, Glob, Write, Edit, Bash)
- Claude Opus 모델 사용으로 고품질 결과
- Proactive 활성화 지원으로 자동 위임 가능

---

## 2. Skills 설정

### 대상 파일 (5개 파일 + 4개 디렉토리)

Skills는 **참조형**과 **실행형** 두 가지 유형으로 구분됩니다. 디렉토리 구조 여부는 내용의 "수준" 차이가 아니라, **스크립트나 설정 파일 포함 여부**에 따른 것입니다.

#### 참조형 Skills (패턴, 표준, 가이드라인)

마크다운 문서로 제공되며, Claude가 참조하는 도메인 지식입니다.

| Skill | 파일 | 내용 |
|-------|------|------|
| 백엔드 패턴 | `backend-patterns.md` | API 설계, Repository, Service Layer, Middleware |
| 프론트엔드 패턴 | `frontend-patterns.md` | Component Composition, Custom Hook, Context |
| 코딩 표준 | `coding-standards.md` | 언어별 best practice |
| ClickHouse IO | `clickhouse-io.md` | 데이터 분석 쿼리 |
| 프로젝트 가이드라인 | `project-guidelines-example.md` | 프로젝트 특정 규칙 템플릿 |
| TDD 워크플로우 | `tdd-workflow/SKILL.md` | 테스트 주도 개발 상세 가이드 |
| 보안 리뷰 | `security-review/SKILL.md` | OWASP Top 10 체크리스트 |

#### 실행형 Skills (스크립트 + 설정 포함)

마크다운 외에 스크립트나 설정 파일을 포함하여 자동화 기능을 제공합니다.

| Skill | 디렉토리 | 포함 파일 | 용도 |
|-------|----------|-----------|------|
| 지속적 학습 | `continuous-learning/` | `config.json`, `evaluate-session.sh` | 세션 종료 시 패턴 자동 추출 |
| 전략적 컴팩션 | `strategic-compact/` | `suggest-compact.sh` | 논리적 간격에서 컴팩션 제안 |

### 설치 명령어

```bash
mkdir -p ~/.claude/skills

# 기본 Skills
cp skills/*.md ~/.claude/skills/

# 고급 Skills (디렉토리)
cp -r skills/tdd-workflow ~/.claude/skills/
cp -r skills/security-review ~/.claude/skills/
cp -r skills/continuous-learning ~/.claude/skills/
cp -r skills/strategic-compact ~/.claude/skills/
```

---

## 3. Commands 설정

### 대상 파일 (10개)

| 명령어 | 파일 | 기능 |
|--------|------|------|
| `/plan` | `plan.md` | 구현 계획 생성 (planner agent 활용) |
| `/tdd` | `tdd.md` | TDD 워크플로우 실행 |
| `/code-review` | `code-review.md` | 코드 품질 리뷰 |
| `/build-fix` | `build-fix.md` | 빌드 오류 수정 |
| `/e2e` | `e2e.md` | E2E 테스트 생성/실행 |
| `/refactor-clean` | `refactor-clean.md` | Dead code 정리 |
| `/learn` | `learn.md` | 세션 중 패턴 추출 |
| `/test-coverage` | `test-coverage.md` | 테스트 커버리지 리포트 |
| `/update-docs` | `update-docs.md` | 문서 자동 업데이트 |
| `/update-codemaps` | `update-codemaps.md` | 코드맵 업데이트 |

### 설치 명령어

```bash
mkdir -p ~/.claude/commands
cp commands/*.md ~/.claude/commands/
```

### 사용 방법

Claude Code 세션에서 슬래시 명령어로 실행:

```
/plan 사용자 인증 기능 구현
/tdd UserService 클래스
/code-review src/auth/
```

---

## 4. Hooks 설정

### 개요

Hooks는 Claude Code의 특정 이벤트에 반응하여 자동으로 실행되는 스크립트입니다.

### Hook 종류

#### PreToolUse (도구 실행 전)

| Hook | 기능 |
|------|------|
| tmux-notify | 장시간 실행 명령어 알림 |
| git-push-review | 변경 사항 확인 후 push |
| doc-blocker | .md/.txt 파일 생성 제한 |
| strategic-compact | Edit/Write 시 컴팩션 제안 |

#### PostToolUse (도구 실행 후)

| Hook | 기능 |
|------|------|
| pr-url-logger | PR 생성 후 URL 로깅 및 GitHub Actions 상태 확인 |
| prettier-format | JS/TS 파일 편집 후 Prettier 자동 포맷 |
| tsc-check | TypeScript 파일 편집 후 tsc 실행 |
| console-log-warn | console.log 사용 경고 |

#### Stop (세션 종료 시)

| Hook | 기능 |
|------|------|
| console-log-audit | 모든 수정된 파일에서 console.log 최종 감사 |
| session-save | 세션 상태 저장 |

### 설치 명령어

```bash
# Hook 스크립트 복사
mkdir -p ~/.claude/hooks/memory-persistence
mkdir -p ~/.claude/hooks/strategic-compact

cp hooks/memory-persistence/*.sh ~/.claude/hooks/memory-persistence/
cp hooks/strategic-compact/*.sh ~/.claude/hooks/strategic-compact/

# 실행 권한 부여
chmod +x ~/.claude/hooks/memory-persistence/*.sh
chmod +x ~/.claude/hooks/strategic-compact/*.sh
```

### settings.json 설정

`hooks/hooks.json` 내용을 `~/.claude/settings.json`에 병합해야 합니다:

```bash
# hooks.json 확인
cat hooks/hooks.json

# ~/.claude/settings.json에 hooks 섹션 추가 (수동 병합 필요)
```

**주의:** 기존 `settings.json`이 있다면 `hooks` 섹션만 추가하세요.

---

## 5. MCP Servers 설정

### 대상 파일

`mcp-configs/mcp-servers.json`

### 포함된 MCP 서버 (12개)

| 서버 | 용도 |
|------|------|
| **github** | GitHub PR, issue, repo 작업 |
| **firecrawl** | 웹 스크래핑 및 크롤링 |
| **supabase** | 데이터베이스 작업 |
| **memory** | 세션 간 메모리 |
| **sequential-thinking** | 연쇄적 사고 |
| **vercel** | 배포 및 프로젝트 관리 |
| **railway** | 배포 |
| **cloudflare** | 문서, 워커, 바인딩, 옵저버빌리티 |
| **clickhouse** | 분석 쿼리 |
| **context7** | 실시간 문서 조회 |
| **magic** | Magic UI 컴포넌트 |
| **filesystem** | 파일 시스템 작업 |

### 설치 방법

`~/.claude.json` 파일에 `mcpServers` 섹션을 추가합니다:

```bash
# mcp-servers.json 내용 확인
cat mcp-configs/mcp-servers.json

# ~/.claude.json에 병합 (수동 작업 필요)
```

### 환경 변수 설정

각 MCP 서버에 필요한 API 키를 환경 변수로 설정해야 합니다:

```bash
# ~/.zshrc 또는 ~/.bashrc에 추가
export GITHUB_TOKEN="your_github_token"
export FIRECRAWL_API_KEY="your_firecrawl_key"
export SUPABASE_ACCESS_TOKEN="your_supabase_token"
# ... 기타 필요한 API 키들
```

---

## 6. Rules 설정

### 대상 파일 (8개)

| Rule | 파일 | 내용 |
|------|------|------|
| 보안 | `security.md` | 필수 보안 검사, OWASP Top 10 |
| 코딩 스타일 | `coding-style.md` | 불변성, 파일 구성, 에러 처리 |
| 테스팅 | `testing.md` | TDD, 80% 커버리지 요구 |
| Git 워크플로우 | `git-workflow.md` | Conventional commits, PR 프로세스 |
| Agents | `agents.md` | subagent 위임 시점 및 방법 |
| 패턴 | `patterns.md` | API 응답, Repository 패턴 |
| 성능 | `performance.md` | 모델 선택, context 관리 |
| Hooks | `hooks.md` | Hook 시스템 사용법 |

### 설치 명령어

```bash
mkdir -p ~/.claude/rules
cp rules/*.md ~/.claude/rules/
```

### CLAUDE.md에서 참조

`~/.claude/CLAUDE.md`에서 rules를 import하여 사용:

```markdown
# 전역 Rules

@import rules/security.md
@import rules/coding-style.md
@import rules/testing.md
@import rules/git-workflow.md
@import rules/agents.md
@import rules/patterns.md
@import rules/performance.md
```

---

## 7. Contexts 설정

### 대상 파일 (3개)

| Context | 파일 | 용도 |
|---------|------|------|
| 개발 모드 | `dev.md` | 활성 개발, 구현 중심, 코드 작성 먼저 |
| 리뷰 모드 | `review.md` | 품질 검증, 보안 검사, best practice |
| 연구 모드 | `research.md` | 행동 전 탐색, 문제 분석, 옵션 평가 |

### 설치 명령어

```bash
mkdir -p ~/.claude/contexts
cp contexts/*.md ~/.claude/contexts/
```

### Shell Alias 설정

`~/.zshrc` 또는 `~/.bashrc`에 추가:

```bash
# Claude Code Context Aliases
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'
alias claude-research='claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'
```

### 사용 방법

```bash
# 개발 모드로 Claude Code 시작
claude-dev

# 리뷰 모드로 Claude Code 시작
claude-review

# 연구 모드로 Claude Code 시작
claude-research
```

---

## 8. 사용자 CLAUDE.md 설정

### 참조 파일

`examples/user-CLAUDE.md` - 사용자 레벨 설정 템플릿

### 설치 명령어

```bash
# 템플릿 복사
cp examples/user-CLAUDE.md ~/.claude/CLAUDE.md

# 개인화 편집
code ~/.claude/CLAUDE.md  # 또는 원하는 에디터 사용
```

### 주요 설정 항목

- 개인 코딩 선호도
- 전역 규칙
- 모듈화된 규칙 링크 (@import)
- 자주 사용하는 패턴

---

## 전체 설치 스크립트

아래 스크립트를 실행하면 모든 설정을 한 번에 설치할 수 있습니다:

```bash
#!/bin/bash
# install-global-settings.sh
# Claude Code Global 설정 설치 스크립트

set -e

# 프로젝트 경로 (필요시 수정)
REPO_PATH="$(cd "$(dirname "$0")" && pwd)"
CLAUDE_HOME="$HOME/.claude"

echo "=== Claude Code Global 설정 설치 시작 ==="
echo "소스: $REPO_PATH"
echo "대상: $CLAUDE_HOME"
echo ""

# 디렉토리 생성
echo "[1/8] 디렉토리 생성..."
mkdir -p "$CLAUDE_HOME"/{agents,skills,commands,rules,contexts,hooks/memory-persistence,hooks/strategic-compact}

# Agents
echo "[2/8] Agents 설치..."
cp "$REPO_PATH"/agents/*.md "$CLAUDE_HOME/agents/"
echo "  - $(ls "$REPO_PATH"/agents/*.md | wc -l | tr -d ' ')개 agent 설치됨"

# Skills
echo "[3/8] Skills 설치..."
cp "$REPO_PATH"/skills/*.md "$CLAUDE_HOME/skills/" 2>/dev/null || true
cp -r "$REPO_PATH"/skills/tdd-workflow "$CLAUDE_HOME/skills/" 2>/dev/null || true
cp -r "$REPO_PATH"/skills/security-review "$CLAUDE_HOME/skills/" 2>/dev/null || true
cp -r "$REPO_PATH"/skills/continuous-learning "$CLAUDE_HOME/skills/" 2>/dev/null || true
cp -r "$REPO_PATH"/skills/strategic-compact "$CLAUDE_HOME/skills/" 2>/dev/null || true
echo "  - Skills 설치 완료"

# Commands
echo "[4/8] Commands 설치..."
cp "$REPO_PATH"/commands/*.md "$CLAUDE_HOME/commands/"
echo "  - $(ls "$REPO_PATH"/commands/*.md | wc -l | tr -d ' ')개 command 설치됨"

# Rules
echo "[5/8] Rules 설치..."
cp "$REPO_PATH"/rules/*.md "$CLAUDE_HOME/rules/"
echo "  - $(ls "$REPO_PATH"/rules/*.md | wc -l | tr -d ' ')개 rule 설치됨"

# Contexts
echo "[6/8] Contexts 설치..."
cp "$REPO_PATH"/contexts/*.md "$CLAUDE_HOME/contexts/"
echo "  - $(ls "$REPO_PATH"/contexts/*.md | wc -l | tr -d ' ')개 context 설치됨"

# Hooks
echo "[7/8] Hook 스크립트 설치..."
cp "$REPO_PATH"/hooks/memory-persistence/*.sh "$CLAUDE_HOME/hooks/memory-persistence/" 2>/dev/null || true
cp "$REPO_PATH"/hooks/strategic-compact/*.sh "$CLAUDE_HOME/hooks/strategic-compact/" 2>/dev/null || true
find "$CLAUDE_HOME/hooks" -name "*.sh" -exec chmod +x {} \;
echo "  - Hook 스크립트 설치 완료"

# User CLAUDE.md
echo "[8/8] 사용자 CLAUDE.md 템플릿 설치..."
if [ ! -f "$CLAUDE_HOME/CLAUDE.md" ]; then
    cp "$REPO_PATH"/examples/user-CLAUDE.md "$CLAUDE_HOME/CLAUDE.md"
    echo "  - CLAUDE.md 템플릿 설치됨"
else
    echo "  - CLAUDE.md가 이미 존재함 (건너뜀)"
fi

echo ""
echo "=== 설치 완료 ==="
echo ""
echo "추가 작업이 필요합니다:"
echo ""
echo "1. Hooks 설정 병합:"
echo "   cat $REPO_PATH/hooks/hooks.json"
echo "   # 위 내용을 ~/.claude/settings.json에 병합"
echo ""
echo "2. MCP 서버 설정 병합:"
echo "   cat $REPO_PATH/mcp-configs/mcp-servers.json"
echo "   # 위 내용을 ~/.claude.json에 병합"
echo ""
echo "3. Shell alias 설정 (~/.zshrc 또는 ~/.bashrc에 추가):"
echo '   alias claude-dev='\''claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'\'''
echo '   alias claude-review='\''claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'\'''
echo '   alias claude-research='\''claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'\'''
echo ""
echo "4. 사용자 CLAUDE.md 개인화:"
echo "   code ~/.claude/CLAUDE.md"
echo ""
echo "5. 환경 변수 설정 (MCP 서버용 API 키):"
echo "   export GITHUB_TOKEN=\"your_token\""
echo "   # 기타 필요한 API 키들..."
```

### 스크립트 실행 방법

```bash
# 스크립트에 실행 권한 부여
chmod +x install-global-settings.sh

# 실행
./install-global-settings.sh
```

---

## 설치 후 추가 작업

### 1. Hooks 설정 병합

```bash
# hooks.json 내용 확인
cat hooks/hooks.json

# ~/.claude/settings.json 편집
code ~/.claude/settings.json
```

`settings.json` 예시:
```json
{
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Stop": [...]
  }
}
```

### 2. MCP 서버 설정 병합

```bash
# mcp-servers.json 내용 확인
cat mcp-configs/mcp-servers.json

# ~/.claude.json 편집
code ~/.claude.json
```

### 3. 환경 변수 설정

`~/.zshrc` 또는 `~/.bashrc`에 추가:

```bash
# Claude Code MCP 서버 API 키
export GITHUB_TOKEN="your_github_token"
export FIRECRAWL_API_KEY="your_firecrawl_key"
export SUPABASE_ACCESS_TOKEN="your_supabase_token"
export VERCEL_TOKEN="your_vercel_token"
export RAILWAY_TOKEN="your_railway_token"
export CLOUDFLARE_API_TOKEN="your_cloudflare_token"
export CLICKHOUSE_HOST="your_clickhouse_host"
export CLICKHOUSE_PASSWORD="your_clickhouse_password"
```

### 4. Shell 재시작

```bash
source ~/.zshrc  # 또는 source ~/.bashrc
```

---

## 요약

| 카테고리 | 개수 | 대상 경로 | 설치 방법 |
|----------|------|-----------|-----------|
| Agents | 9 | `~/.claude/agents/` | 파일 복사 |
| Skills | 5 파일 + 4 디렉토리 | `~/.claude/skills/` | 파일/디렉토리 복사 |
| Commands | 10 | `~/.claude/commands/` | 파일 복사 |
| Rules | 8 | `~/.claude/rules/` | 파일 복사 |
| Contexts | 3 | `~/.claude/contexts/` | 파일 복사 + alias |
| Hooks | 1 JSON + 5 스크립트 | `~/.claude/settings.json` + `~/.claude/hooks/` | 수동 병합 + 파일 복사 |
| MCP Servers | 12 | `~/.claude.json` | 수동 병합 |
| User CLAUDE.md | 1 | `~/.claude/CLAUDE.md` | 템플릿 복사 후 개인화 |

---

## 문제 해결

### Hooks가 작동하지 않는 경우

1. 스크립트 실행 권한 확인:
   ```bash
   ls -la ~/.claude/hooks/**/*.sh
   ```

2. settings.json 문법 확인:
   ```bash
   cat ~/.claude/settings.json | jq .
   ```

### MCP 서버 연결 실패

1. API 키 환경 변수 확인:
   ```bash
   echo $GITHUB_TOKEN
   ```

2. claude.json 문법 확인:
   ```bash
   cat ~/.claude.json | jq .
   ```

### Commands가 인식되지 않는 경우

1. 파일 위치 확인:
   ```bash
   ls ~/.claude/commands/
   ```

2. Claude Code 재시작

---

## 참고 자료

- [SHORTHAND_GUIDE.md](./SHORTHAND_GUIDE.md) - 간편 가이드
- [LONGFORM_GUIDE.md](./LONGFORM_GUIDE.md) - 심화 가이드
- [examples/CLAUDE.md](./examples/CLAUDE.md) - 프로젝트 레벨 설정 예제
- [examples/user-CLAUDE.md](./examples/user-CLAUDE.md) - 사용자 레벨 설정 예제
