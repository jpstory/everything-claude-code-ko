"Claude Code 완벽 가이드 - 간편 버전"에서는 기본 설정을 다뤘습니다: skills와 commands, hooks, subagents, MCPs, plugins, 그리고 효과적인 Claude Code 워크플로우의 근간을 이루는 설정 패턴들. 그것은 설정 가이드이자 기본 인프라입니다.

![아티클 커버 이미지](https://pbs.twimg.com/media/G-1nccYaoAATrW2?format=jpg&name=medium)

Claude Code 완벽 가이드 - 간편 버전

10개월간의 일일 사용 후 완성된 설정입니다: skills, hooks, subagents, MCPs, plugins, 그리고 실제로 효과가 있는 것들. 2월 실험적 출시 때부터 열렬한 Claude Code 사용자였으며...

이 심화 가이드는 생산적인 세션과 낭비적인 세션을 구분하는 기법들을 다룹니다. 간편 버전 가이드를 읽지 않았다면, 먼저 설정을 완료하세요. 이후 내용은 skills, agents, hooks, MCPs가 이미 구성되어 작동 중이라고 가정합니다.

여기서 다루는 주제들: token 경제학, 메모리 지속성, 검증 패턴, 병렬화 전략, 그리고 재사용 가능한 워크플로우 구축의 복합 효과. 이것들은 10개월 이상의 일일 사용을 통해 다듬은 패턴으로, 첫 시간 내에 context rot에 시달리는 것과 몇 시간 동안 생산적인 세션을 유지하는 것의 차이를 만듭니다.

간편 버전과 심화 버전 문서에서 다루는 모든 내용은 GitHub에서 확인할 수 있습니다:

## Context 및 메모리 관리

세션 간 메모리를 공유하려면, 진행 상황을 요약하고 확인한 다음 `.claude` 폴더의 `.tmp` 파일에 저장하고 세션이 끝날 때까지 계속 추가하는 skill이나 command가 가장 좋습니다. 다음 날 그것을 context로 사용하여 중단한 곳에서 이어갈 수 있으며, 각 세션마다 새 파일을 생성하여 오래된 context가 새 작업을 오염시키지 않도록 합니다. 결국 이런 세션 로그들이 큰 폴더를 이루게 됩니다 - 의미 있는 곳에 백업하거나 필요 없는 세션 대화는 정리하세요.

Claude가 현재 상태를 요약하는 파일을 생성합니다. 검토하고, 필요하면 수정을 요청한 다음, 새로 시작합니다. 새 대화에서는 파일 경로만 제공하면 됩니다. context 한도에 도달했고 복잡한 작업을 계속해야 할 때 특히 유용합니다. 이 파일에는 다음이 포함되어야 합니다 - 어떤 접근 방식이 효과가 있었는지(증거로 검증 가능하게), 시도했지만 효과가 없었던 접근 방식, 아직 시도하지 않은 접근 방식과 남은 작업.

[

![이미지](https://pbs.twimg.com/media/G_Jqmo5asAAc_w3?format=png&name=900x900)



](https://x.com/affaanmustafa/article/2014040193557471352/media/2013789195433848832)

세션 저장 예시 ->

전략적 Context 정리:

계획이 설정되고 context가 정리되면(현재 Claude Code의 plan mode 기본 옵션), 계획에 따라 작업할 수 있습니다. 실행과 더 이상 관련 없는 탐색 context가 많이 쌓였을 때 유용합니다. 전략적 compacting을 위해 auto compact를 비활성화하세요. 논리적인 간격으로 수동 compact하거나, 정의된 기준에 따라 제안하는 skill을 만드세요.

(빠른 참조용으로 포함)

```bash
#!/bin/bash
# 전략적 Compact 제안기
# PreToolUse에서 실행되어 논리적 간격으로 수동 compaction 제안
#
# auto-compact 대신 수동을 사용하는 이유:
# - Auto-compact는 임의의 시점에, 종종 작업 중간에 발생
# - 전략적 compacting은 논리적 단계를 통해 context를 보존
# - 탐색 후, 실행 전에 compact
# - 마일스톤 완료 후, 다음 시작 전에 compact

COUNTER_FILE="/tmp/claude-tool-count-$$"
THRESHOLD=${COMPACT_THRESHOLD:-50}

# 카운터 초기화 또는 증가
if [ -f "$COUNTER_FILE" ]; then
  count=$(cat "$COUNTER_FILE")
  count=$((count + 1))
  echo "$count" > "$COUNTER_FILE"
else
  echo "1" > "$COUNTER_FILE"
  count=1
fi

# 임계값 도구 호출 후 compact 제안
if [ "$count" -eq "$THRESHOLD" ]; then
  echo "[StrategicCompact] $THRESHOLD 도구 호출 도달 - 단계 전환 중이라면 /compact 고려" >&2
fi
```

Edit/Write 작업의 PreToolUse에 연결하세요 - compacting이 도움이 될 만큼 충분한 context가 쌓이면 알려줍니다.

고급: 동적 System Prompt 주입

제가 도입하여 시험 중인 패턴은: 모든 세션에 로드되는 (user scope)이나 `.claude/rules/` (project scope)에만 모든 것을 넣는 대신, CLI 플래그를 사용하여 동적으로 context를 주입하는 것입니다.

```bash
claude --system-prompt "$(cat memory.md)"
```

이렇게 하면 어떤 context가 언제 로드되는지 더 정밀하게 제어할 수 있습니다. 작업 중인 내용에 따라 세션별로 다른 context를 주입할 수 있습니다.

@ 파일 참조와 비교했을 때 이것이 중요한 이유:

`@파일.md`를 사용하거나 `.claude/rules/`에 넣으면, Claude가 대화 중에 Read 도구를 통해 읽습니다 - 도구 출력으로 들어옵니다. `--system-prompt`를 사용하면, 대화가 시작되기 전에 실제 system prompt에 내용이 주입됩니다.

차이점은 명령어 계층입니다. System prompt 내용은 사용자 메시지보다 높은 권한을 가지며, 사용자 메시지는 도구 결과보다 높은 권한을 가집니다. 대부분의 일상 작업에서는 이 차이가 미미합니다. 하지만 엄격한 행동 규칙, 프로젝트별 제약, 또는 Claude가 반드시 우선시해야 하는 context의 경우 - system prompt 주입이 적절한 가중치를 보장합니다.

이를 수행하는 유효한 방법은 `.claude/rules/`를 기본 프로젝트 규칙에 활용하고, 전환할 수 있는 시나리오별 context를 위한 CLI alias를 만드는 것입니다:

```bash
# 일일 개발
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'

# PR 리뷰 모드
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'

# 리서치/탐색 모드
alias claude-research='claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'
```

-   구현에 집중
-   행동 전 탐색에 집중
    

다시 말하지만, 대부분의 경우 `.claude/rules/context1.md` 사용과 system prompt에 직접 추가하는 것의 차이는 미미합니다. CLI 접근 방식이 더 빠르고(도구 호출 없음), 더 신뢰할 수 있으며(시스템 수준 권한), 약간 더 token 효율적입니다. 하지만 이것은 사소한 최적화이고 많은 사람들에게는 그만한 가치가 없는 오버헤드입니다.

고급: 메모리 지속성 Hooks

대부분의 사람들이 모르거나 알아도 잘 활용하지 않는 메모리에 도움이 되는 hooks가 있습니다:

```css
세션 1                                 세션 2
─────────                              ─────────

[시작]                                 [시작]
   │                                      │
   ▼                                      ▼
┌──────────────┐                    ┌──────────────┐
│ SessionStart │ ◄─── 읽기 ──────── │ SessionStart │◄── 이전 context
│    Hook      │     아직 없음      │    Hook      │    로드
└──────┬───────┘                    └──────┬───────┘
       │                                   │
       ▼                                   ▼
   [작업 중]                           [작업 중]
       │                               (정보 있음)
       ▼                                   │
┌──────────────┐                           ▼
│  PreCompact  │──► 요약 전에         [계속...]
│    Hook      │    상태 저장
└──────┬───────┘
       │
       ▼
   [Compacted]
       │
       ▼
┌──────────────┐
│  Stop Hook   │──► 저장 위치 ──────────►
│ (세션 종료)  │    ~/.claude/sessions/
└──────────────┘
```

-   PreCompact Hook: context compaction 전에 중요한 상태를 파일에 저장
-   SessionComplete Hook: 세션 종료 시 학습 내용을 파일에 저장
-   SessionStart Hook: 새 세션에서 이전 context를 자동으로 로드
    

(빠른 참조용으로 포함)

```css
{
  "hooks": {
    "PreCompact": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/memory-persistence/pre-compact.sh"
      }]
    }],
    "SessionStart": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/memory-persistence/session-start.sh"
      }]
    }],
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/memory-persistence/session-end.sh"
      }]
    }]
  }
}
```

-   : compaction 이벤트 기록, compaction 타임스탬프로 활성 세션 파일 업데이트
-   : 최근 세션 파일(최근 7일) 확인, 사용 가능한 context와 학습된 skills 알림
-   : 일일 세션 파일 생성/업데이트(템플릿 사용), 시작/종료 시간 추적
    

수동 개입 없이 세션 간 지속적인 메모리를 위해 이것들을 연결하세요. 이것은 Article 1의 hook 유형(PreToolUse, PostToolUse, Stop)을 기반으로 하지만 특히 세션 라이프사이클을 대상으로 합니다.

## 지속적 학습 / 메모리

codemap 업데이트 형태의 지속적 메모리 업데이트에 대해 이야기했지만, 이것은 실수로부터 배우는 것과 같은 다른 것들에도 적용됩니다. 프롬프트를 여러 번 반복해야 했고 Claude가 같은 문제에 부딪히거나 이전에 들었던 응답을 했다면 이것이 해당됩니다.

아마도 Claude의 방향을 "재조정"하고 보정하기 위해 두 번째 프롬프트를 보내야 했을 것입니다. 이것은 그런 모든 시나리오에 적용됩니다 - 그런 패턴들은 skills에 추가되어야 합니다.

이제 Claude에게 기억하라고 하거나 rules에 추가하라고 말함으로써 자동으로 이 작업을 수행할 수 있고, 또는 정확히 그 작업을 수행하는 skill을 만들 수 있습니다.

문제점: 낭비되는 tokens, 낭비되는 context, 낭비되는 시간, 이전 세션에서 이미 하지 말라고 했던 것을 하지 말라고 답답하게 Claude에게 소리칠 때 코르티솔이 치솟습니다.

해결책: Claude Code가 사소하지 않은 것을 발견하면 - 디버깅 기법, 우회 방법, 프로젝트별 패턴 - 그 지식을 새로운 skill로 저장합니다. 다음에 비슷한 문제가 발생하면 skill이 자동으로 로드됩니다.

왜 UserPromptSubmit 대신 Stop hook을 사용했나요? UserPromptSubmit은 보내는 모든 메시지에서 실행됩니다 - 오버헤드가 많고, 모든 프롬프트에 지연을 추가하며, 솔직히 이 목적에는 과도합니다. Stop은 세션 종료 시 한 번만 실행됩니다 - 가볍고, 세션 중 속도를 늦추지 않으며, 조각조각이 아닌 전체 세션을 평가합니다.

```perl
# skills 폴더에 클론
git clone https://github.com/affaan-m/everything-claude-code.git ~/.claude/skills/everything-claude-code

# 또는 continuous-learning skill만 가져오기
mkdir -p ~/.claude/skills/continuous-learning
curl -sL https://raw.githubusercontent.com/affaan-m/everything-claude-code/main/skills/continuous-learning/evaluate-session.sh > ~/.claude/skills/continuous-learning/evaluate-session.sh
chmod +x ~/.claude/skills/continuous-learning/evaluate-session.sh
```

```css
{
  "hooks": {
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
          }
        ]
      }
    ]
  }
}
```

이것은 Stop hook을 사용하여 모든 프롬프트에서 activator 스크립트를 실행하고, 추출할 가치가 있는 지식을 세션에서 평가합니다. skill은 semantic matching을 통해서도 활성화될 수 있지만, hook은 일관된 평가를 보장합니다.

Stop hook은 세션이 종료될 때 트리거됩니다 - 스크립트가 추출할 가치가 있는 패턴(오류 해결, 디버깅 기법, 우회 방법, 프로젝트별 패턴 등)을 세션에서 분석하고 `~/.claude/skills/learned/`에 재사용 가능한 skills로 저장합니다.

/learn을 사용한 수동 추출:

세션 종료까지 기다릴 필요가 없습니다. 저장소에는 사소하지 않은 것을 막 해결했을 때 세션 중간에 실행할 수 있는 `/learn` command도 포함되어 있습니다. 바로 패턴을 추출하도록 프롬프트하고, skill 파일 초안을 작성하고, 저장 전에 확인을 요청합니다. 참조하세요.

skill은 `.tmp` 파일의 세션 로그를 예상합니다. 패턴은: `~/.claude/sessions/YYYY-MM-DD-topic.tmp` - 세션당 하나의 파일로 현재 상태, 완료된 항목, 차단 요소, 주요 결정, 다음 세션을 위한 context를 포함합니다. 예시 세션 파일은 저장소에 있습니다.

기타 자기 개선 메모리 패턴:

한 가지 접근 방식은 세션 로그를 반영하여 사용자 선호도를 추출하는 것입니다 - 본질적으로 무엇이 효과가 있고 무엇이 효과가 없는지에 대한 "일기"를 작성합니다. 각 세션 후에 reflection agent가 무엇이 잘 되었는지, 무엇이 실패했는지, 어떤 수정을 했는지 추출합니다. 이러한 학습 내용은 후속 세션에서 로드되는 메모리 파일을 업데이트합니다.

또 다른 접근 방식은 패턴을 알아차리기를 기다리는 대신 시스템이 15분마다 사전에 개선 사항을 제안하는 것입니다. agent가 최근 상호작용을 검토하고, 메모리 업데이트를 제안하면, 승인하거나 거부합니다. 시간이 지나면서 승인 패턴에서 학습합니다.

## Token 최적화

가격에 민감한 소비자들이나 파워 유저로서 자주 한도 문제에 부딪히는 분들로부터 많은 질문을 받았습니다. token 최적화와 관련하여 할 수 있는 몇 가지 트릭이 있습니다.

주요 전략: Subagent 아키텍처

주로 사용하는 도구와 낭비를 줄이기 위해 작업에 충분한 가장 저렴한 모델에 위임하도록 설계된 subagent 아키텍처를 최적화하는 것입니다. 여기에는 몇 가지 옵션이 있습니다 - 시행착오를 통해 진행하면서 적응할 수 있습니다. 무엇이 무엇인지 배우면, Haiku에 위임할 것과 Sonnet에 위임할 것, Opus에 위임할 것을 구분할 수 있습니다.

벤치마킹 접근 방식 (더 복잡함):

조금 더 복잡한 또 다른 방법은 Claude가 명확하게 정의된 목표와 작업, 명확하게 정의된 계획이 있는 저장소로 벤치마크를 설정하도록 하는 것입니다. 각 git worktree에서 모든 subagents를 하나의 모델로 설정합니다. 작업이 완료되면 기록합니다 - 이상적으로는 계획과 작업에. 각 subagent를 최소한 한 번은 사용해야 합니다.

전체 패스를 완료하고 Claude 계획에서 작업이 체크되면, 멈추고 진행 상황을 감사합니다. diff를 비교하고, 모든 worktree에서 동일한 unit, integration, E2E 테스트를 생성하여 이 작업을 수행할 수 있습니다. 이것은 통과한 케이스 대 실패한 케이스를 기반으로 수치적 벤치마크를 제공합니다. 모든 것이 모두 통과하면, 더 많은 테스트 엣지 케이스를 추가하거나 테스트의 복잡성을 높여야 합니다. 이것이 정말 중요한지에 따라 가치가 있을 수도 있고 없을 수도 있습니다.

모델 선택 빠른 참조:

[

![이미지](https://pbs.twimg.com/media/G_KO-ICaoAAyNtt?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2014040193557471352/media/2013829181348683776)

다양한 일반 작업에 대한 subagents의 가상 설정과 선택 이유

코딩 작업의 90%는 Sonnet을 기본으로 사용하세요. 첫 번째 시도가 실패했거나, 작업이 5개 이상의 파일에 걸쳐 있거나, 아키텍처 결정이거나, 보안에 중요한 코드일 때 Opus로 업그레이드하세요. 작업이 반복적이거나, 지시가 매우 명확하거나, multi-agent 설정에서 "worker"로 사용할 때 Haiku로 다운그레이드하세요. 솔직히 Sonnet 4.5는 현재 100만 입력 token당 $3, 100만 출력 token당 $15로 이상한 위치에 있습니다. Opus 대비 비용 절감은 ~66.7%로, 절대적으로 말하면 좋은 절감이지만 상대적으로는 대부분의 사람들에게 거의 무의미합니다. Haiku와 Opus 조합이 가장 합리적입니다. Haiku vs Opus는 5배 비용 차이인 반면, Sonnet 대비는 1.67배 가격 차이에 불과합니다.

[

![이미지](https://pbs.twimg.com/media/G_KSUOmaoAE-DVF?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2014040193557471352/media/2013832859602296833)

agent 정의에서 model을 지정하세요:

```yaml
---
name: quick-search
description: 빠른 파일 검색
tools: Glob, Grep
model: haiku # 저렴하고 빠름
---
```

도구별 최적화:

Claude가 가장 자주 호출하는 도구에 대해 생각해보세요. 예를 들어, grep을 mgrep으로 교체하면 - 다양한 작업에서 Claude가 기본으로 사용하는 전통적인 grep이나 ripgrep에 비해 평균적으로 약 절반의 효과적인 token 감소를 가져옵니다.

[

![이미지](https://pbs.twimg.com/media/G_KQApzX0AA0o3u?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2014040193557471352/media/2013830324283756544)

해당되는 경우, Claude가 전체 출력을 처리하고 직접 실시간 스트리밍할 필요가 없다면 Claude 외부에서 백그라운드 프로세스를 실행하세요. 이것은 tmux로 쉽게 달성할 수 있습니다 (참조). 터미널 출력을 가져와서 요약하거나 필요한 부분만 복사하세요. 이것은 많은 입력 token을 절약합니다 - 비용의 대부분이 여기서 발생합니다 - Opus 4.5의 경우 100만 token당 $5이고 출력은 100만 token당 $25입니다.

모듈러 코드베이스의 이점:

재사용 가능한 유틸리티, 함수, hooks 등을 갖춘 더 모듈러한 코드베이스 - 메인 파일이 수천 줄이 아닌 수백 줄 - 는 token 최적화 비용과 첫 번째 시도에서 작업을 올바르게 수행하는 데 모두 도움이 되며, 이 둘은 상관관계가 있습니다. Claude에게 여러 번 프롬프트해야 한다면 tokens를 태우고 있는 것입니다, 특히 매우 긴 파일을 반복해서 읽을 때. 파일 읽기를 완료하기 위해 많은 도구 호출을 해야 하는 것을 알 수 있을 것입니다. 중간에, 파일이 매우 길어서 계속 읽을 것이라고 알려줍니다. 이 과정 어딘가에서 Claude가 일부 정보를 잃을 수 있습니다. 또한 멈추고 다시 읽는 것은 추가 tokens 비용이 듭니다. 이것은 더 모듈러한 코드베이스를 갖춤으로써 피할 수 있습니다. 아래 예시 ->

```perl
root/
├── docs/                   # 전역 문서
├── scripts/                # CI/CD 및 빌드 스크립트
├── src/
│   ├── apps/               # 진입점 (API, CLI, Workers)
│   │   ├── api-gateway/    # 요청을 모듈로 라우팅
│   │   └── cron-jobs/      
│   │
│   ├── modules/            # 시스템의 핵심
│   │   ├── ordering/       # 자체 포함된 "Ordering" 모듈
│   │   │   ├── api/        # 다른 모듈을 위한 공개 인터페이스
│   │   │   ├── domain/     # 비즈니스 로직 & 엔티티 (순수)
│   │   │   ├── infrastructure/ # DB, 외부 클라이언트, 저장소
│   │   │   ├── use-cases/  # 애플리케이션 로직 (오케스트레이션)
│   │   │   └── tests/      # 유닛 및 통합 테스트
│   │   │
│   │   ├── catalog/        # 자체 포함된 "Catalog" 모듈
│   │   │   ├── domain/
│   │   │   └── ...
│   │   │
│   │   └── identity/       # 자체 포함된 "Auth/User" 모듈
│   │       ├── domain/
│   │       └── ...
│   │
│   ├── shared/             # 모든 모듈이 사용하는 코드
│   │   ├── kernel/         # 기본 클래스 (Entity, ValueObject)
│   │   ├── events/         # 전역 Event Bus 정의
│   │   └── utils/          # 깊이 범용적인 헬퍼
│   │
│   └── main.ts             # 애플리케이션 부트스트랩
├── tests/                  # End-to-End (E2E) 전역 테스트
├── package.json
└── README.md
```

간결한 코드베이스 = 더 저렴한 Tokens:

당연한 것일 수 있지만, 코드베이스가 간결할수록 token 비용이 저렴해집니다. skills와 commands를 사용하여 리팩토링으로 코드베이스를 지속적으로 정리하여 죽은 코드를 식별하는 것이 중요합니다. 또한 특정 시점에, 저는 전체 코드베이스를 훑어보면서 눈에 띄거나 반복적으로 보이는 것들을 찾고, 수동으로 그 context를 조합한 다음, refactor skill과 dead code skill과 함께 Claude에 제공하는 것을 좋아합니다.

System Prompt 슬리밍 (고급):

정말로 비용을 의식하는 분들을 위해: Claude Code의 system prompt는 ~18k tokens (~200k context의 9%)를 차지합니다. 이것은 패치로 ~10k tokens로 줄일 수 있으며, ~7,300 tokens (정적 오버헤드의 41%)를 절약합니다. 이 방법을 원하면 YK의 글을 참조하세요, 개인적으로 저는 이렇게 하지 않습니다.

## 검증 루프와 Evals

평가와 harness 튜닝 - 프로젝트에 따라 어떤 형태의 관찰 가능성과 표준화를 사용하고 싶을 것입니다.

이를 수행하는 한 가지 방법은 skill이 트리거될 때마다 thinking stream과 출력을 추적하도록 tmux 프로세스를 연결하는 것입니다. 또 다른 방법은 Claude가 구체적으로 실행한 것과 정확한 변경 및 출력이 무엇이었는지 기록하는 PostToolUse hook을 갖는 것입니다.

skill 없이 같은 것을 요청하고 출력 차이를 확인하여 상대적 성능을 벤치마크하는 것과 비교하세요:

```css
[같은 작업]
                         │
            ┌────────────┴────────────┐
            ▼                         ▼
    ┌───────────────┐         ┌───────────────┐
    │  Worktree A   │         │  Worktree B   │
    │  skill 있음   │         │  skill 없음   │
    └───────┬───────┘         └───────┬───────┘
            │                         │
            ▼                         ▼
       [출력 A]                  [출력 B]
            │                         │
            └──────────┬──────────────┘
                       ▼
                  [git diff]
                       │
                       ▼
              ┌────────────────┐
              │ 로그 비교,     │
              │ token 사용량,  │
              │ 출력 품질      │
              └────────────────┘
```

대화를 포크하고, 그 중 하나에서 skill 없이 새 worktree를 시작하고, 끝에 diff를 확인하고, 무엇이 기록되었는지 봅니다. 이것은 지속적 학습 및 메모리 섹션과 연결됩니다.

여기서 더 고급 eval 및 루프 프로토콜이 들어옵니다. 분기는 checkpoint 기반 evals와 RL 작업 기반 지속적 evals 사이입니다.

```less
CHECKPOINT 기반                          지속적
─────────────────                        ──────────

  [작업 1]                                 [작업]
     │                                        │
     ▼                                        ▼
  ┌─────────┐                            ┌─────────┐
  │Checkpoint│◄── 기준                   │ 타이머/ │
  │   #1    │    검증                    │ 변경    │
  └────┬────┘                            └────┬────┘
       │ 통과?                                │
   ┌───┴───┐                                  ▼
   │       │                            ┌──────────┐
  예      아니오 ──► 수정 ──┐           │테스트 실행│
   │              │    │                │  + Lint  │
   ▼              └────┘                └────┬─────┘
  [작업 2]                                   │
     │                                  ┌────┴────┐
     ▼                                  │         │
  ┌─────────┐                          통과     실패
  │Checkpoint│                          │         │
  │   #2    │                           ▼         ▼
  └────┬────┘                        [계속]   [멈추고 수정]
       │                                          │
      ...                                    └────┘

적합: 명확한 마일스톤이 있는           적합: 긴 세션
선형 워크플로우                        탐색적 리팩토링
```

-   워크플로우에 명시적 checkpoint 설정
-   각 checkpoint에서 정의된 기준에 대해 검증
-   검증 실패 시 Claude가 진행 전에 수정해야 함
-   명확한 마일스톤이 있는 선형 워크플로우에 적합

-   N분마다 또는 주요 변경 후 실행
-   전체 테스트 스위트, 빌드 상태, lint
-   즉시 regression 보고
-   계속하기 전에 멈추고 수정
-   장시간 세션에 적합
    

결정 요인은 작업의 성격입니다. Checkpoint 기반은 명확한 단계가 있는 기능 구현에 적합합니다. 지속적은 명확한 마일스톤이 없는 탐색적 리팩토링이나 유지보수에 적합합니다.

약간의 개입으로 검증 접근 방식은 대부분의 기술 부채를 피하기에 충분하다고 말할 수 있습니다. Claude가 작업을 완료한 후 skills와 PostToolUse hooks를 실행하여 검증하도록 하는 것이 도움이 됩니다. 지속적인 codemap 업데이트도 변경 사항과 codemap이 시간이 지남에 따라 어떻게 진화하는지 기록을 유지하여 저장소 자체 외부의 진실의 원천으로 작용하기 때문에 도움이 됩니다. 엄격한 규칙으로 Claude는 모든 것을 어지럽히는 임의의 .md 파일 생성, 유사한 코드를 위한 중복 파일, 죽은 코드의 황무지를 남기는 것을 피합니다.

코드 기반 Graders: 문자열 일치, 이진 테스트, 정적 분석, 결과 검증. 빠르고, 저렴하고, 객관적이지만 유효한 변형에 취약합니다.

모델 기반 Graders: 루브릭 채점, 자연어 assertions, 쌍별 비교. 유연하고 뉘앙스를 처리하지만 비결정적이고 더 비쌉니다.

인간 Graders: SME 검토, 크라우드소싱 판단, 스팟 체크 샘플링. 골드 스탠다드 품질이지만 비싸고 느립니다.

```sql
pass@k: k번 시도 중 최소 하나가 성공
        ┌─────────────────────────────────────┐
        │  k=1: 70%  k=3: 91%  k=5: 97%      │
        │  k가 높을수록 = 성공 확률 높음       │
        └─────────────────────────────────────┘

pass^k: k번 시도 모두 성공해야 함
        ┌─────────────────────────────────────┐
        │  k=1: 70%  k=3: 34%  k=5: 17%      │
        │  k가 높을수록 = 더 어려움 (일관성)   │
        └─────────────────────────────────────┘
```

작동하기만 하면 되고 어떤 검증 피드백이든 충분할 때 pass@k를 사용하세요. 일관성이 필수적이고 거의 결정적인 출력 일관성(결과/품질/스타일 측면에서)이 필요할 때 pass^k를 사용하세요.

Eval 로드맵 구축 (같은 Anthropic 가이드에서):

1.  일찍 시작 - 실제 실패에서 20-50개의 간단한 작업
2.  사용자가 보고한 실패를 테스트 케이스로 변환
3.  모호하지 않은 작업 작성 - 두 전문가가 같은 결론에 도달해야 함
4.  균형 잡힌 문제 세트 구축 - 동작이 발생해야 할 때와 발생하지 않아야 할 때 테스트
5.  견고한 harness 구축 - 각 시도는 깨끗한 환경에서 시작
6.  agent가 만든 것을 채점, 경로가 아닌
7.  많은 시도의 transcript 읽기
8.  포화 모니터링 - 100% 통과율은 더 많은 테스트 추가 필요를 의미
    

## 병렬화

multi-Claude 터미널 설정에서 대화를 포크할 때, 포크와 원래 대화의 작업 범위가 잘 정의되어 있는지 확인하세요. 코드 변경과 관련하여 최소한의 겹침을 목표로 하세요. 간섭 가능성을 방지하기 위해 서로 직교하는 작업을 선택하세요.

개인적으로 저는 메인 채팅이 코드 변경 작업을 하고 제가 하는 포크는 코드베이스와 현재 상태에 대한 질문, 또는 문서 가져오기, 작업에 도움이 될 적용 가능한 오픈소스 저장소를 GitHub에서 검색, 또는 도움이 될 다른 일반적인 리서치와 같은 외부 서비스에 대한 리서치를 위한 것을 선호합니다.

임의의 터미널 수에 대해:

Boris

(Claude Code를 만든 전설)가 병렬화에 대한 팁을 가지고 있는데 동의하는 것도 있고 동의하지 않는 것도 있습니다. 그는 로컬에서 5개의 Claude 인스턴스와 upstream에서 5개를 실행하는 것 같은 것을 제안했습니다. 저는 이렇게 임의의 터미널 수를 설정하는 것을 권하지 않습니다. 터미널 추가와 인스턴스 추가는 진정한 필요와 목적에서 나와야 합니다. 스크립트로 그 작업을 처리할 수 있다면 스크립트를 사용하세요. 메인 채팅에 머물면서 Claude가 tmux에서 인스턴스를 시작하고 별도의 터미널에서 스트리밍하도록 할 수 있다면 그렇게 하세요.

1/ 저는 터미널에서 5개의 Claude를 병렬로 실행합니다. 탭에 1-5 번호를 매기고, Claude가 입력을 필요로 할 때 시스템 알림을 사용합니다 [code.claude.com/docs/en/termin](https://t.co/nmRJ5km3oZ)

[

![이미지](https://pbs.twimg.com/media/G9rtc4EasAELEzh?format=jpg&name=medium)



](https://x.com/bcherny/status/2007179833990885678/photo/1)

목표는 정말로: 최소한의 실행 가능한 병렬화로 얼마나 많은 것을 할 수 있는가여야 합니다.

대부분의 초보자에게는 단일 인스턴스를 실행하고 그 안에서 모든 것을 관리하는 요령을 터득할 때까지 병렬화를 피하는 것을 권합니다. 스스로를 제한하라는 것이 아닙니다 - 조심하라는 것입니다. 대부분의 경우, 저도 총 4개 정도의 터미널만 사용합니다. 보통 2개 또는 3개의 Claude 인스턴스만 열어도 대부분의 작업을 할 수 있다는 것을 알았습니다.

인스턴스를 확장하기 시작하고 서로 겹치는 코드에서 여러 Claude 인스턴스가 작업하는 경우, git worktrees를 사용하고 각각에 대해 매우 잘 정의된 계획을 갖는 것이 필수적입니다. 또한 세션을 재개할 때 어떤 git worktree가 무엇을 위한 것인지 혼란스럽거나 잃어버리지 않으려면(트리 이름 외에), `/rename <이름>`을 사용하여 모든 채팅에 이름을 지정하세요.

병렬 인스턴스를 위한 Git Worktrees:

```bash
# 병렬 작업을 위한 worktrees 생성
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-refactor refactor-branch

# 각 worktree는 자체 Claude 인스턴스를 가짐
cd ../project-feature-a && claude
```

-   인스턴스 간 git 충돌 없음
-   각각 깨끗한 작업 디렉토리 보유
-   출력 비교 용이
-   다른 접근 방식으로 같은 작업 벤치마크 가능
    

여러 Claude Code 인스턴스를 실행할 때, "cascade" 패턴으로 구성하세요:

-   오른쪽의 새 탭에서 새 작업 열기
-   왼쪽에서 오른쪽으로, 오래된 것에서 새것으로 스윕
-   일관된 방향 흐름 유지
-   필요에 따라 특정 작업 확인
-   한 번에 최대 3-4개 작업에 집중 - 그 이상은 생산성보다 정신적 오버헤드가 더 빨리 증가
    

## 기반 작업

새로 시작할 때 실제 기반이 매우 중요합니다. 당연해 보이겠지만 코드베이스의 복잡성과 크기가 증가하면 기술 부채도 증가합니다. 관리하는 것이 매우 중요하며 몇 가지 규칙을 따르면 어렵지 않습니다. 해당 프로젝트를 위해 Claude를 효과적으로 설정하는 것 외에도 (간편 버전 가이드 참조).

두 인스턴스 킥오프 패턴:

제 워크플로우 관리를 위해 (필수는 아니지만 도움이 됨), 저는 2개의 열린 Claude 인스턴스로 빈 저장소를 시작하는 것을 좋아합니다.

인스턴스 1: Scaffolding Agent

-   scaffold와 기반을 놓을 것
-   프로젝트 구조 생성
-   설정 구성 (rules, agents - 간편 버전 가이드의 모든 것)
-   컨벤션 수립
-   골격을 제자리에 배치
    

인스턴스 2: Deep Research Agent

-   모든 서비스, 웹 검색 등에 연결
-   상세한 PRD 생성
-   아키텍처 mermaid 다이어그램 생성
-   실제 문서의 실제 클립으로 참조 자료 컴파일
    

[

![이미지](https://pbs.twimg.com/media/G_KYgQYawAA9rXk?format=jpg&name=medium)



](https://x.com/affaanmustafa/article/2014040193557471352/media/2013839663308652544)

시작 설정: 왼쪽 터미널은 코딩용, 오른쪽 터미널은 질문용 - /rename과 /fork 사용.

시작하는 데 최소한으로 필요한 것이면 충분합니다 - 매번 Context7이나 스크랩할 링크를 제공하거나 Firecrawl MCP 사이트를 사용하는 것보다 빠릅니다. 이것들은 이미 무언가에 깊이 빠져 있고 Claude가 명확히 문법을 틀리거나 오래된 함수나 엔드포인트를 사용할 때 작동합니다.

가능하다면, 문서 페이지에 도달한 후 `/llms.txt`를 수행하여 많은 문서 참조에서 llms.txt를 찾을 수 있습니다. 다음은 예시입니다:

이것은 Claude에 직접 제공할 수 있는 깨끗한 LLM 최적화 버전의 문서를 제공합니다.

철학: 재사용 가능한 패턴 구축

제가 전적으로 지지하는 한 가지 통찰: "초기에 재사용 가능한 워크플로우/패턴을 구축하는 데 시간을 보냈습니다. 만들기는 지루하지만 모델과 agent harness가 개선됨에 따라 엄청난 복리 효과가 있었습니다."

-   Subagents (간편 버전 가이드)
-   Skills (간편 버전 가이드)
-   Commands (간편 버전 가이드)
-   Planning 패턴
-   MCP 도구 (간편 버전 가이드)
-   Context 엔지니어링 패턴
    

왜 복리 효과가 있는가: "가장 좋은 점은 이 모든 워크플로우가 Codex와 같은 다른 agents로 이전 가능하다는 것입니다." 한 번 만들면 모델 업그레이드에 걸쳐 작동합니다. 패턴에 대한 투자 > 특정 모델 트릭에 대한 투자.

## Agents 및 Sub-Agents를 위한 모범 사례

간편 버전 가이드에서 subagent 구조를 나열했습니다 - planner, architect, tdd-guide, code-reviewer 등. 이 부분에서는 오케스트레이션과 실행 레이어에 집중합니다.

Sub-Agent Context 문제:

Sub-agents는 모든 것을 덤프하는 대신 요약을 반환하여 context를 절약하기 위해 존재합니다. 하지만 orchestrator는 sub-agent가 가지고 있지 않은 semantic context를 가지고 있습니다. Sub-agent는 요청 뒤의 목적/이유가 아닌 문자 그대로의 쿼리만 알고 있습니다. 요약은 종종 핵심 세부 사항을 놓칩니다.

비유: "상사가 회의에 보내고 요약을 요청합니다. 돌아와서 개요를 제공합니다. 열에 아홉은 후속 질문이 있을 것입니다. 요약에는 그가 필요로 하는 모든 것이 포함되지 않을 것입니다. 왜냐하면 그가 가진 암묵적 context가 없기 때문입니다."

반복적 검색 패턴:

```scss
┌─────────────────┐
│  ORCHESTRATOR   │
│  (context 있음) │
└────────┬────────┘
         │ 쿼리 + 목표와 함께 디스패치
         ▼
┌─────────────────┐
│   SUB-AGENT     │
│ (context 없음)  │
└────────┬────────┘
         │ 요약 반환
         ▼
┌─────────────────┐      ┌─────────────┐
│    평가         │─아니오►│  후속 질문  │
│   충분한가?     │      │             │
└────────┬────────┘      └──────┬──────┘
         │ 예                   │
         ▼                      │ sub-agent가
    [수락]                 답변 가져옴
                               │
         ◄──────────────────────┘
              (최대 3 사이클)
```

이를 해결하려면 orchestrator가:

-   모든 sub-agent 반환을 평가
-   수락하기 전에 후속 질문
-   Sub-agent가 소스로 돌아가 답변을 가져와 반환
-   충분할 때까지 루프 (무한 루프 방지를 위해 최대 3 사이클)
    

쿼리만이 아닌 목표 context를 전달하세요. subagent를 디스패치할 때 특정 쿼리와 더 넓은 목표를 모두 포함하세요. 이것은 subagent가 요약에 무엇을 포함할지 우선순위를 정하는 데 도움이 됩니다.

패턴: 순차적 단계가 있는 Orchestrator

```diff
Phase 1: RESEARCH (Explore agent 사용)

- context 수집
- 패턴 식별
- 출력: research-summary.md

Phase 2: PLAN (planner agent 사용)

- research-summary.md 읽기
- 구현 계획 생성
- 출력: plan.md

Phase 3: IMPLEMENT (tdd-guide agent 사용)

- plan.md 읽기
- 먼저 테스트 작성
- 코드 구현
- 출력: 코드 변경

Phase 4: REVIEW (code-reviewer agent 사용)

- 모든 변경 검토
- 출력: review-comments.md

Phase 5: VERIFY (필요시 build-error-resolver 사용)

- 테스트 실행
- 이슈 수정
- 출력: 완료 또는 루프백
```

1.  각 agent는 하나의 명확한 입력을 받고 하나의 명확한 출력을 생성
2.  출력이 다음 단계의 입력이 됨
3.  단계를 건너뛰지 않음 - 각각 가치를 추가
4.  agents 사이에 `/clear`를 사용하여 context를 신선하게 유지
5.  중간 출력을 파일에 저장 (메모리만이 아닌)
    

Agent 추상화 등급 목록:

Tier 1: 직접적 버프 (사용하기 쉬움)

-   Subagents - context rot 방지와 ad-hoc 전문화를 위한 직접적 버프. multi-agent의 절반 정도 유용하지만 복잡성은 훨씬 적음
-   Metaprompting - "20분 작업을 프롬프트하는 데 3분을 씁니다." 직접적 버프 - 안정성을 개선하고 가정을 검증
-   처음에 사용자에게 더 많이 질문 - 일반적으로 버프, plan mode에서 질문에 답해야 하지만
    

Tier 2: 높은 스킬 플로어 (잘 사용하기 어려움)

-   장시간 실행 agents - 15분 작업 vs 1.5시간 vs 4시간 작업의 형태와 트레이드오프를 이해해야 함. 약간의 조정이 필요하고 분명히 매우 긴 시행착오
-   병렬 multi-agent - 매우 높은 변동성, 매우 복잡하거나 잘 분할된 작업에서만 유용. "2개 작업이 10분 걸리고 임의의 시간을 프롬프트에 쓰거나, 더 나쁘게는 변경을 병합하면 비생산적"
-   역할 기반 multi-agent - "모델이 너무 빨리 진화하여 arbitrage가 매우 높지 않으면 하드코딩된 휴리스틱이 맞지 않음." 테스트하기 어려움
-   Computer use agents - 매우 초기 패러다임, 조정이 필요. "1년 전에는 분명히 하도록 의도되지 않은 것을 모델이 하도록 만들고 있음"
    

핵심: Tier 1 패턴으로 시작하세요. 기본을 마스터하고 진정한 필요가 있을 때만 Tier 2로 졸업하세요.

## 팁과 트릭

일부 MCPs는 대체 가능하며 Context Window를 확보해줍니다

버전 관리(GitHub), 데이터베이스(Supabase), 배포(Vercel, Railway) 등과 같은 MCPs의 경우 - 이러한 플랫폼 대부분은 MCP가 본질적으로 래핑하는 강력한 CLI를 이미 가지고 있습니다. MCP는 좋은 래퍼이지만 비용이 따릅니다.

MCP를 실제로 사용하지 않고(그리고 그에 따른 감소된 context window 없이) CLI가 MCP처럼 기능하도록 하려면, 기능을 skills와 commands로 번들링하는 것을 고려하세요. MCP가 노출하는 도구 중 작업을 쉽게 만드는 것들을 분리하여 commands로 바꾸세요.

예시: GitHub MCP를 항상 로드하는 대신, 선호하는 옵션으로 `gh pr create`를 래핑하는 `/gh-pr` command를 만드세요. Supabase MCP가 context를 먹는 대신, Supabase CLI를 직접 사용하는 skills를 만드세요. 기능은 같고, 편의성도 비슷하지만, context window는 실제 작업을 위해 확보됩니다.

이것은 제가 받은 다른 질문들과 연결됩니다. 원래 아티클을 게시한 이후 며칠 동안, Boris와 Claude Code 팀은 메모리 관리와 최적화에서 많은 진전을 이루었습니다, 주로 MCPs의 lazy loading으로 처음부터 window를 먹지 않도록 했습니다. 이전에는 가능한 곳에서 MCPs를 skills로 변환하는 것을 추천했을 것입니다, MCP를 실행하는 기능을 두 가지 방법 중 하나로 오프로드하여: 그때 활성화하거나(세션을 떠나고 재개해야 하므로 덜 이상적) MCP에 대한 CLI 유사체를 사용하는 skills를 갖고(존재하는 경우) skill이 그 주위의 래퍼가 되도록 - 본질적으로 pseudo-MCP로 작동하게 합니다.

lazy loading으로 context window 문제는 대부분 해결되었습니다. 하지만 token 사용량과 비용은 같은 방식으로 해결되지 않았습니다. CLI + skills 접근 방식은 여전히 MCP 사용과 동등하거나 가까운 효과를 가질 수 있는 token 최적화 방법입니다. 또한 in-context 대신 CLI를 통해 MCP 작업을 실행할 수 있어 token 사용량을 크게 줄입니다, 특히 데이터베이스 쿼리나 배포와 같은 무거운 MCP 작업에 유용합니다.

## 비디오?

제안하신 대로 이것과 다른 질문들을 함께 다루는 비디오를 이 아티클과 함께 제작할 생각입니다.

두 아티클의 전술을 활용한 END-TO-END 프로젝트 다루기:

-   간편 버전 가이드의 설정으로 전체 프로젝트 설정
-   이 심화 가이드의 고급 기법 실제 적용
-   실시간 token 최적화
-   실제 검증 루프
-   세션 간 메모리 관리
-   두 인스턴스 킥오프 패턴
-   git worktrees를 사용한 병렬 워크플로우
-   실제 워크플로우의 스크린샷과 녹화
    

## 참고 자료

- [Anthropic: AI agents를 위한 evals 이해하기](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (Jan 2026)
- Anthropic: "Claude Code 모범 사례" (Apr 2025)
- Fireworks AI: "Claude Code를 사용한 Eval 주도 개발" (Aug 2025)
- [YK: 32가지 Claude Code 팁](https://agenticcoding.substack.com/p/32-claude-code-tips-from-basics-to) (Dec 2025)
- Addy Osmani: "2026년을 향한 나의 LLM 코딩 워크플로우"
- @PerceptualPeak: Sub-Agent Context 협상
- @menhguin: Agent 추상화 등급 목록
- @omarsar0: 복리 효과 철학
- [RLanceMartin: 세션 반영 패턴](https://rlancemartin.github.io/2025/12/01/claude_diary/)
- @alexhillman: 자기 개선 메모리 시스템