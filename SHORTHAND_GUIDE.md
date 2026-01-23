10개월간의 일일 사용 후 완성된 설정: skill, hook, subagent, MCP, plugin, 그리고 실제로 작동하는 것들.

2월 실험 단계부터 열렬한 Claude Code 사용자였으며, Anthropic x Forum Ventures 해커톤에서 완전히 Claude Code만 사용하여 우승했습니다.

NYC에서 열린 해커톤에서 우승 - 훌륭한 이벤트 주최에 감사드립니다 (그리고 15k Anthropic Credit에도)

PMFProbe를 만들어 창업자들이 0에서 1로 가는 것을 돕고, MVP 이전 단계에서 아이디어를 검증합니다. 더 많은 것이 곧 나올 예정입니다.

## 

Skill과 Command

Skill은 rule처럼 작동하지만, 특정 범위와 workflow에 제한됩니다. 특정 workflow를 실행할 때 프롬프트의 축약형입니다.

Opus 4.5와 긴 코딩 세션 후, dead code와 느슨한 .md 파일을 정리하고 싶다면?

/refactor-clean을 실행하세요. 테스트가 필요하면? /tdd, /e2e, /test-coverage. Skill과 command는 단일 프롬프트에서 함께 연결할 수 있습니다.

[

![이미지](https://pbs.twimg.com/media/G-0-_fZagAA9Kqk?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012333868985253888)

command 연결하기

checkpoint에서 codemap을 업데이트하는 skill을 만들 수 있습니다 - Claude가 탐색에 context를 소모하지 않고 codebase를 빠르게 탐색하는 방법입니다.

~/.claude/skills/codemap-updater.md

Command는 slash command로 실행되는 skill입니다. 겹치지만 저장 위치가 다릅니다:

-   Skill: ~/.claude/skills - 더 넓은 workflow 정의
-   Command: ~/.claude/commands - 빠른 실행 프롬프트
    

```perl
# Skill 구조 예시
~/.claude/skills/
  pmx-guidelines.md      # 프로젝트별 패턴
  coding-standards.md    # 언어 best practice
  tdd-workflow/          # README.md가 있는 다중 파일 skill
  security-review/       # 체크리스트 기반 skill
```

## 

Hook

Hook은 특정 이벤트에서 발동되는 trigger 기반 자동화입니다. Skill과 달리, tool 호출과 lifecycle 이벤트에 제한됩니다.

1.  PreToolUse \- tool 실행 전 (유효성 검사, 알림)
2.  PostToolUse - tool 완료 후 (포맷팅, 피드백 loop)
3.  UserPromptSubmit - 메시지 전송 시
4.  Stop - Claude 응답 완료 시
5.  PreCompact - context compaction 전
6.  Notification - 권한 요청
    

예시: 장시간 실행 명령어 전 tmux 알림

```swift
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] 세션 영속성을 위해 tmux를 고려하세요' >&2; fi"
        }
      ]
    }
  ]
}
```

[

![이미지](https://pbs.twimg.com/media/G-1Gwvab0AM7Xr9?format=png&name=small)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012342411679485955)

PostToolUse hook 실행 시 Claude Code에서 받는 피드백 예시

Pro tip: JSON을 수동으로 작성하는 대신 \`hookify\` plugin을 사용하여 대화형으로 hook을 만들 수 있습니다. /hookify를 실행하고 원하는 것을 설명하세요.

## 

Subagent

Subagent는 orchestrator(메인 Claude)가 제한된 범위로 작업을 위임할 수 있는 프로세스입니다. background 또는 foreground에서 실행할 수 있어, 메인 agent의 context를 확보합니다.

Subagent는 skill과 잘 작동합니다 - skill의 하위 집합을 실행할 수 있는 subagent는 작업을 위임받아 해당 skill을 자율적으로 사용할 수 있습니다. 특정 tool 권한으로 sandbox 처리할 수도 있습니다.

```bash
# Subagent 구조 예시
~/.claude/agents/
  planner.md           # 기능 구현 계획
  architect.md         # 시스템 설계 결정
  tdd-guide.md         # 테스트 주도 개발
  code-reviewer.md     # 품질/보안 리뷰
  security-reviewer.md # 취약점 분석
  build-error-resolver.md
  e2e-runner.md
  refactor-cleaner.md
```

적절한 범위 지정을 위해 subagent별로 허용된 tool, MCP, 권한을 설정하세요.

## 

Rule과 Memory

\`.rules\` 폴더는 Claude가 항상 따라야 하는 best practice가 담긴 \`.md\` 파일을 보관합니다. 두 가지 접근법:

1.  단일 \- 모든 것을 하나의 파일에 (사용자 또는 프로젝트 레벨)
2.  Rules 폴더 - 관심사별로 그룹화된 모듈식 \`.md\` 파일
    

```perl
~/.claude/rules/
  security.md      # 하드코딩된 secret 금지, 입력 유효성 검사
  coding-style.md  # 불변성, 파일 구성
  testing.md       # TDD workflow, 80% 커버리지
  git-workflow.md  # commit 형식, PR 프로세스
  agents.md        # subagent 위임 시점
  performance.md   # model 선택, context 관리
```

-   codebase에 이모지 없음
-   frontend에서 보라색 색조 자제
-   배포 전 항상 코드 테스트
-   메가 파일보다 모듈식 코드 우선
-   console.log 절대 커밋 금지
    

## 

MCP (Model Context Protocol)

MCP는 Claude를 외부 서비스에 직접 연결합니다. API의 대체가 아닙니다 - 정보 탐색에 더 많은 유연성을 허용하는 프롬프트 기반 wrapper입니다.

예시: Supabase MCP는 Claude가 특정 데이터를 가져오고, 복사-붙여넣기 없이 직접 upstream에서 SQL을 실행할 수 있게 합니다. database, 배포 플랫폼 등도 마찬가지입니다.

[

![이미지](https://pbs.twimg.com/media/G-1KHqfawAA-PPK?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012346104030085120)

public 스키마 내 테이블을 나열하는 supabase mcp 예시

Chrome in Claude: Claude가 브라우저를 자율적으로 제어할 수 있게 해주는 내장 plugin MCP입니다 - 클릭하며 작동 방식을 확인합니다.

중요: Context Window 관리

MCP 선택에 신중하세요. 모든 MCP를 사용자 설정에 두지만 사용하지 않는 것은 비활성화합니다. /plugins로 이동하여 스크롤하거나 /mcp를 실행하세요.

너무 많은 tool이 활성화되면 compaction 전 200k context window가 70k만 될 수 있습니다. 성능이 크게 저하됩니다.

[

![이미지](https://pbs.twimg.com/media/G-1K2ZJawAAQnV3?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012346906828259328)

/plugins를 사용하여 현재 설치된 MCP와 상태를 확인하기 위해 MCP로 이동

경험 법칙: 설정에 20-30개 MCP를 두고, 10개 미만 활성화 / 80개 미만 tool 활성 유지.

## 

Plugin

Plugin은 번거로운 수동 설정 대신 쉬운 설치를 위해 tool을 패키징합니다. Plugin은 skill + MCP 조합이거나, hook/tool이 함께 번들된 것일 수 있습니다.

```sql
# marketplace 추가
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# Claude 열고, /plugins 실행, 새 marketplace 찾기, 거기서 설치
```

[

![이미지](https://pbs.twimg.com/media/G-1Loo1bYAAI_tz?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012347770032840704)

새로 설치된 Mixedbread-Grep marketplace 표시

LSP Plugin: 에디터 밖에서 Claude Code를 자주 실행한다면 특히 유용합니다. Language Server Protocol은 IDE를 열지 않고도 Claude에게 실시간 타입 검사, go-to-definition, 지능적인 완성을 제공합니다.

```graphql
# 활성화된 plugin 예시
typescript-lsp@claude-plugins-official  # TypeScript intelligence
pyright-lsp@claude-plugins-official     # Python 타입 검사
hookify@claude-plugins-official         # 대화형으로 hook 생성
mgrep@Mixedbread-Grep                   # ripgrep보다 나은 검색
```

MCP와 같은 경고 - context window를 주시하세요.

## 

팁과 트릭

-   Ctrl+U - 전체 줄 삭제 (백스페이스 연타보다 빠름)
-   ! - 빠른 bash 명령어 접두사
-   @ - 파일 검색
-   / - slash command 시작
-   Shift+Enter - 다중 줄 입력
-   Tab - thinking 표시 토글
-   Esc Esc - Claude 중단 / 코드 복원
    

/fork - 대기 중인 메시지를 스팸하는 대신 겹치지 않는 작업을 병렬로 수행하기 위해 대화 분기

Git Worktree - 충돌 없이 겹치는 병렬 Claude를 위해. 각 worktree는 독립적인 checkout

```graphql
git worktree add ../feature-branch feature-branch
# 이제 각 worktree에서 별도의 Claude 인스턴스 실행
```

장시간 실행 명령어를 위한 tmux: Claude가 실행하는 log/bash 프로세스를 스트리밍하고 관찰.

![](https://pbs.twimg.com/amplify_video_thumb/2012355175609188352/img/W8EylFWmB9IKfdTV.jpg)

claude code가 frontend와 backend 서버를 스핀업하고 tmux를 사용하여 세션에 연결하여 log 모니터링

```php
tmux new -s dev
# Claude가 여기서 명령어 실행, detach하고 다시 attach 가능
tmux attach -t dev
```

mgrep > grep: \`mgrep\`은 ripgrep/grep에서 크게 개선되었습니다. plugin marketplace를 통해 설치한 후 /mgrep skill을 사용하세요. 로컬 검색과 웹 검색 모두 작동합니다.

```bash
mgrep "function handleSubmit"  # 로컬 검색
mgrep --web "Next.js 15 app router changes"  # 웹 검색
```

-   /rewind - 이전 상태로 돌아가기
-   /statusline - branch, context %, todo로 커스터마이즈
-   /checkpoints - 파일 레벨 undo 포인트
-   /compact \- 수동으로 context compaction 트리거
    

GitHub Actions로 PR에 code review 설정. 설정하면 Claude가 자동으로 PR을 리뷰할 수 있습니다.

[

![이미지](https://pbs.twimg.com/media/G-1U7nSbAAAK7hf?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012357991639744512)

버그 수정 PR을 승인하는 claude

위험한 작업에는 sandbox 모드 사용 - Claude가 실제 시스템에 영향을 주지 않고 제한된 환경에서 실행합니다. (--dangerously-skip-permissions를 사용하면 반대로 claude가 자유롭게 돌아다니게 할 수 있습니다, 주의하지 않으면 파괴적일 수 있습니다.)

## 

에디터에 대해

에디터가 필수는 아니지만 Claude Code workflow에 긍정적 또는 부정적 영향을 줄 수 있습니다. Claude Code는 모든 터미널에서 작동하지만, 유능한 에디터와 함께 사용하면 실시간 파일 추적, 빠른 탐색, 통합 명령어 실행이 가능합니다.

저는 가볍고 빠르며 고도로 커스터마이즈 가능한 Rust 기반 에디터를 사용합니다.

Zed가 Claude Code와 잘 작동하는 이유:

-   Agent Panel 통합 - Zed의 Claude 통합으로 Claude가 편집하는 파일 변경을 실시간 추적. 에디터를 떠나지 않고 Claude가 참조하는 파일 사이를 점프
-   성능 - Rust로 작성되어 즉시 열리고 큰 codebase도 지연 없이 처리
-   CMD+Shift+R Command Palette - 모든 커스텀 slash command, 디버거, tool에 빠르게 접근 가능한 검색 UI. 터미널로 전환하지 않고 빠른 명령어만 실행하고 싶을 때도
-   최소 리소스 사용 - 무거운 작업 중 Claude와 시스템 리소스 경쟁하지 않음
-   Vim Mode - 원한다면 전체 vim 키바인딩
    

[

![이미지](https://pbs.twimg.com/media/G-1Cy8gbAAA2fE-?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012338051507486720)

CMD+Shift+R을 사용한 커스텀 command 드롭다운이 있는 Zed Editor.

오른쪽 하단의 과녁 표시가 Following mode.

1.  화면 분할 - 한쪽에 Claude Code가 있는 터미널, 다른 쪽에 에디터
2.  Ctrl + G \- Zed에서 Claude가 현재 작업 중인 파일을 빠르게 열기
3.  Auto-save - Claude의 파일 읽기가 항상 최신이 되도록 자동 저장 활성화
4.  Git 통합 - 커밋 전 Claude의 변경 사항을 리뷰하기 위해 에디터의 git 기능 사용
5.  File watcher - 대부분의 에디터가 변경된 파일을 자동 리로드, 활성화되어 있는지 확인
    

이것도 실행 가능한 선택이며 Claude Code와 잘 작동합니다. \\ide를 사용하여 에디터와 자동 동기화하는 터미널 형식으로 사용하여 LSP 기능을 활성화하거나 (plugin과 다소 중복), 더 통합되고 일치하는 UI를 가진 extension을 선택할 수 있습니다.

[

![이미지](https://pbs.twimg.com/media/G-1b3F_aMAApve3?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012365610563547136)

문서에서 직접

## 

내 설정

설치됨: (보통 한 번에 4-5개만 활성화)

```graphql
ralph-wiggum@claude-code-plugins       # Loop 자동화
frontend-design@claude-code-plugins    # UI/UX 패턴
commit-commands@claude-code-plugins    # Git workflow
security-guidance@claude-code-plugins  # 보안 검사
pr-review-toolkit@claude-code-plugins  # PR 자동화
typescript-lsp@claude-plugins-official # TS intelligence
hookify@claude-plugins-official        # Hook 생성
code-simplifier@claude-plugins-official
feature-dev@claude-code-plugins
explanatory-output-style@claude-code-plugins
code-review@claude-code-plugins
context7@claude-plugins-official       # 라이브 문서
pyright-lsp@claude-plugins-official    # Python 타입
mgrep@Mixedbread-Grep                  # 더 나은 검색
```

```perl
{
  "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
  "firecrawl": { "command": "npx", "args": ["-y", "firecrawl-mcp"] },
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"]
  },
  "memory": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-memory"] },
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  },
  "vercel": { "type": "http", "url": "https://mcp.vercel.com" },
  "railway": { "command": "npx", "args": ["-y", "@railway/mcp-server"] },
  "cloudflare-docs": { "type": "http", "url": "https://docs.mcp.cloudflare.com/mcp" },
  "cloudflare-workers-bindings": {
    "type": "http",
    "url": "https://bindings.mcp.cloudflare.com/mcp"
  },
  "cloudflare-workers-builds": { "type": "http", "url": "https://builds.mcp.cloudflare.com/mcp" },
  "cloudflare-observability": {
    "type": "http",
    "url": "https://observability.mcp.cloudflare.com/mcp"
  },
  "clickhouse": { "type": "http", "url": "https://mcp.clickhouse.cloud/mcp" },
  "AbletonMCP": { "command": "uvx", "args": ["ableton-mcp"] },
  "magic": { "command": "npx", "args": ["-y", "@magicuidesign/mcp@latest"] }
}
```

프로젝트별 비활성화 (context window 관리):

```makefile
# ~/.claude.json의 projects.[path].disabledMcpServers 아래
disabledMcpServers: [
  "playwright",
  "cloudflare-workers-builds",
  "cloudflare-workers-bindings",
  "cloudflare-observability",
  "cloudflare-docs",
  "clickhouse",
  "AbletonMCP",
  "context7",
  "magic"
]
```

이것이 핵심입니다 - 14개 MCP가 설정되어 있지만 프로젝트당 ~5-6개만 활성화. Context window를 건강하게 유지합니다.

```json
{
  "PreToolUse": [
    // 장시간 실행 명령어에 tmux 알림
    { "matcher": "npm|pnpm|yarn|cargo|pytest", "hooks": ["tmux 알림"] },
    // 불필요한 .md 파일 생성 차단
    { "matcher": "Write && .md file", "hooks": ["README/CLAUDE가 아니면 차단"] },
    // git push 전 리뷰
    { "matcher": "git push", "hooks": ["리뷰를 위해 에디터 열기"] }
  ],
  "PostToolUse": [
    // Prettier로 JS/TS 자동 포맷
    { "matcher": "Edit && .ts/.tsx/.js/.jsx", "hooks": ["prettier --write"] },
    // 편집 후 TypeScript 검사
    { "matcher": "Edit && .ts/.tsx", "hooks": ["tsc --noEmit"] },
    // console.log 경고
    { "matcher": "Edit", "hooks": ["grep console.log 경고"] }
  ],
  "Stop": [
    // 세션 종료 전 console.log 감사
    { "matcher": "*", "hooks": ["수정된 파일에서 console.log 검사"] }
  ]
}
```

사용자, 디렉토리, dirty indicator가 있는 git branch, 남은 context %, model, 시간, todo 개수 표시:

[

![이미지](https://pbs.twimg.com/media/G-1iYlHaEAAbS0C?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2012378465664745795/media/2012372782924042240)

Mac 루트 디렉토리의 statusline 예시

```bash
~/.claude/rules/
  security.md      # 필수 보안 검사
  coding-style.md  # 불변성, 파일 크기 제한
  testing.md       # TDD, 80% 커버리지
  git-workflow.md  # Conventional commit
  agents.md        # Subagent 위임 규칙
  patterns.md      # API response 형식
  performance.md   # Model 선택 (Haiku vs Sonnet vs Opus)
  hooks.md         # Hook 문서
```

```bash
~/.claude/agents/
  planner.md           # 기능 분해
  architect.md         # 시스템 설계
  tdd-guide.md         # 테스트 먼저 작성
  code-reviewer.md     # 품질 리뷰
  security-reviewer.md # 취약점 스캔
  build-error-resolver.md
  e2e-runner.md        # Playwright 테스트
  refactor-cleaner.md  # Dead code 제거
  doc-updater.md       # 문서 동기화 유지
```

## 

핵심 요점

1.  과도하게 복잡하게 만들지 마세요 - 설정을 아키텍처가 아닌 fine-tuning처럼 다루세요
2.  Context window는 소중합니다 - 사용하지 않는 MCP와 plugin 비활성화
3.  병렬 실행 - 대화 분기, git worktree 사용
4.  반복적인 것을 자동화 - 포맷팅, lint, 알림을 위한 hook
5.  Subagent 범위 지정 - 제한된 tool = 집중된 실행
    

## 

참고자료

참고: 이것은 세부 사항의 일부입니다. 관심이 있다면 구체적인 내용에 대해 더 많은 게시물을 만들 수 있습니다.
