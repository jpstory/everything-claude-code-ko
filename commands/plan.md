---
description: Restate requirements, assess risks, and create step-by-step implementation plan. WAIT for user CONFIRM before touching any code.
---

# Plan Command

이 command는 **planner** agent를 호출하여 코드 작성 전에 종합적인 구현 계획을 생성합니다.

## 이 Command의 기능

1. **요구사항 재진술** - 무엇을 만들어야 하는지 명확화
2. **위험 식별** - 잠재적 이슈 및 차단 요소 도출
3. **단계별 계획 생성** - 구현을 단계로 분해
4. **확인 대기** - 진행 전 반드시 사용자 승인 필요

## 사용 시점

다음 경우 `/plan` 사용:
- 새 기능 시작
- 중요한 아키텍처 변경
- 복잡한 리팩토링 작업
- 여러 파일/컴포넌트에 영향을 주는 경우
- 요구사항이 불명확하거나 모호한 경우

## 작동 방식

planner agent가 수행하는 작업:

1. **요청 분석** 및 요구사항을 명확한 용어로 재진술
2. 구체적이고 실행 가능한 단계로 **단계별 분해**
3. 컴포넌트 간 **의존성 식별**
4. **위험 평가** 및 잠재적 차단 요소
5. **복잡도 추정** (High/Medium/Low)
6. **계획 제시** 및 명시적 확인 대기

## 사용 예시

```
User: /plan 마켓이 해결될 때 실시간 알림을 추가해야 해

Agent (planner):
# 구현 계획: 실시간 마켓 해결 알림

## 요구사항 재진술
- 사용자가 관심 있는 마켓이 해결될 때 알림 전송
- 여러 알림 채널 지원 (in-app, email, webhook)
- 알림이 안정적으로 전달되도록 보장
- 마켓 결과 및 사용자 포지션 결과 포함

## 구현 단계

### Phase 1: Database Schema
- id, user_id, market_id, type, status, created_at 컬럼이 있는 notifications 테이블 추가
- 채널 선호도를 위한 user_notification_preferences 테이블 추가
- 성능을 위해 user_id 및 market_id에 index 생성

### Phase 2: Notification Service
- lib/notifications.ts에 notification service 생성
- BullMQ/Redis를 사용한 notification queue 구현
- 실패한 전달에 대한 retry 로직 추가
- notification template 생성

### Phase 3: Integration Point
- 마켓 해결 로직에 hook (status가 "resolved"로 변경될 때)
- 마켓에 포지션이 있는 모든 사용자 조회
- 각 사용자에 대해 notification enqueue

### Phase 4: Frontend Component
- 헤더에 NotificationBell 컴포넌트 생성
- NotificationList modal 추가
- Supabase subscription을 통한 실시간 업데이트 구현
- notification 설정 페이지 추가

## 의존성
- Redis (queue용)
- Email service (SendGrid/Resend)
- Supabase 실시간 subscription

## 위험
- HIGH: Email 전달성 (SPF/DKIM 필요)
- MEDIUM: 마켓당 1000+ 사용자 시 성능
- MEDIUM: 마켓이 자주 해결될 경우 알림 스팸
- LOW: 실시간 subscription 오버헤드

## 예상 복잡도: MEDIUM
- Backend: 4-6시간
- Frontend: 3-4시간
- Testing: 2-3시간
- Total: 9-13시간

**확인 대기 중**: 이 계획으로 진행할까요? (yes/no/modify)
```

## 중요 참고사항

**중요**: planner agent는 "yes" 또는 "proceed" 또는 유사한 긍정 응답으로 계획을 명시적으로 확인할 때까지 코드를 작성하지 **않습니다**.

변경을 원하면 다음과 같이 응답:
- "modify: [변경 사항]"
- "different approach: [대안]"
- "skip phase 2 and do phase 3 first"

## 다른 Command와의 통합

계획 후:
- `/tdd`로 테스트 주도 개발 구현
- `/build-and-fix`로 빌드 오류 발생 시 해결
- `/code-review`로 완료된 구현 리뷰

## 관련 Agent

이 command는 다음 위치의 `planner` agent를 호출합니다:
`~/.claude/agents/planner.md`
