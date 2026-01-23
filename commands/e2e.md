---
description: Generate and run end-to-end tests with Playwright. Creates test journeys, runs tests, captures screenshots/videos/traces, and uploads artifacts.
---

# E2E Command

이 command는 **e2e-runner** agent를 호출하여 Playwright를 사용한 end-to-end 테스트를 생성, 유지, 실행합니다.

## 이 Command의 기능

1. **테스트 Journey 생성** - 사용자 flow에 대한 Playwright 테스트 생성
2. **E2E 테스트 실행** - 브라우저 간 테스트 실행
3. **아티팩트 캡처** - 실패 시 스크린샷, 비디오, trace
4. **결과 업로드** - HTML 리포트 및 JUnit XML
5. **Flaky 테스트 식별** - 불안정한 테스트 격리

## 사용 시점

다음 경우 `/e2e` 사용:
- 핵심 사용자 journey 테스트 (로그인, 거래, 결제)
- 다단계 flow가 end-to-end로 작동하는지 확인
- UI 상호작용 및 네비게이션 테스트
- 프론트엔드와 백엔드 간 통합 검증
- 프로덕션 배포 준비

## 작동 방식

e2e-runner agent가 수행하는 작업:

1. **사용자 flow 분석** 및 테스트 시나리오 식별
2. Page Object Model 패턴을 사용한 **Playwright 테스트 생성**
3. 여러 브라우저에서 **테스트 실행** (Chrome, Firefox, Safari)
4. 스크린샷, 비디오, trace로 **실패 캡처**
5. 결과 및 아티팩트가 포함된 **리포트 생성**
6. **Flaky 테스트 식별** 및 수정 권장

## 사용 예시

```
User: /e2e 마켓 검색 및 보기 flow 테스트

Agent (e2e-runner):
# E2E 테스트 생성: 마켓 검색 및 보기 Flow

## 식별된 테스트 시나리오

**사용자 Journey:** 마켓 검색 → 결과 보기 → 마켓 클릭 → 상세 정보 보기

**테스트 커버리지:**
1. 마켓 페이지로 이동
2. 시맨틱 검색 수행
3. 검색 결과 확인
4. 첫 번째 결과 클릭
5. 마켓 상세 페이지 로드 확인
6. 차트 렌더링 확인

## 생성된 테스트 코드

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('마켓 검색 및 보기 Flow', () => {
  test('사용자가 마켓을 검색하고 상세 정보를 볼 수 있다', async ({ page }) => {
    // 1. 마켓 페이지로 이동
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 페이지 로드 확인
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. 시맨틱 검색 수행
    await marketsPage.searchMarkets('election')

    // API 응답 대기
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 검색 결과 확인
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 검색 결과 스크린샷
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 첫 번째 결과 클릭
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. 마켓 상세 페이지 로드 확인
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. 차트 렌더링 확인
    await expect(detailsPage.priceChart).toBeVisible()

    // 마켓 이름 일치 확인
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // 마켓 상세 스크린샷
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('결과가 없는 검색은 빈 상태를 표시한다', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 존재하지 않는 마켓 검색
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 빈 상태 확인
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('검색을 지우면 모든 마켓을 다시 볼 수 있다', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 초기 마켓 수
    const initialCount = await marketsPage.marketCards.count()

    // 검색 수행
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // 필터링된 결과 확인
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // 검색 지우기
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // 모든 마켓이 다시 표시되는지 확인
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## 테스트 실행

```bash
# 생성된 테스트 실행
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › search-and-view.spec.ts:5:3 › 사용자가 마켓을 검색하고 상세 정보를 볼 수 있다 (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › 결과가 없는 검색은 빈 상태를 표시한다 (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › 검색을 지우면 모든 마켓을 다시 볼 수 있다 (2.9s)

  3 passed (9.1s)

생성된 아티팩트:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## 테스트 리포트

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E 테스트 결과                            ║
╠══════════════════════════════════════════════════════════════╣
║ 상태:      ✅ 모든 테스트 통과                                ║
║ 총:        3 테스트                                          ║
║ 통과:      3 (100%)                                          ║
║ 실패:      0                                                 ║
║ Flaky:     0                                                 ║
║ 소요시간:   9.1s                                              ║
╚══════════════════════════════════════════════════════════════╝

아티팩트:
📸 스크린샷: 2 파일
📹 비디오: 0 파일 (실패 시에만)
🔍 Trace: 0 파일 (실패 시에만)
📊 HTML 리포트: playwright-report/index.html

리포트 보기: npx playwright show-report
```

✅ E2E 테스트 suite가 CI/CD 통합 준비 완료!
```

## 테스트 아티팩트

테스트 실행 시 다음 아티팩트가 캡처됩니다:

**모든 테스트에서:**
- 타임라인 및 결과가 포함된 HTML 리포트
- CI 통합을 위한 JUnit XML

**실패 시에만:**
- 실패 상태의 스크린샷
- 테스트 비디오 녹화
- 디버깅을 위한 trace 파일 (단계별 재생)
- 네트워크 로그
- 콘솔 로그

## 아티팩트 보기

```bash
# 브라우저에서 HTML 리포트 보기
npx playwright show-report

# 특정 trace 파일 보기
npx playwright show-trace artifacts/trace-abc123.zip

# 스크린샷은 artifacts/ 디렉토리에 저장됨
open artifacts/search-results.png
```

## Flaky 테스트 감지

테스트가 간헐적으로 실패하는 경우:

```
⚠️  FLAKY 테스트 감지: tests/e2e/markets/trade.spec.ts

테스트 통과율 7/10 (70%)

일반적인 실패:
"'[data-testid="confirm-btn"]' 요소 대기 시간 초과"

권장 수정:
1. 명시적 대기 추가: await page.waitForSelector('[data-testid="confirm-btn"]')
2. 타임아웃 증가: { timeout: 10000 }
3. 컴포넌트의 race condition 확인
4. 애니메이션에 의해 요소가 숨겨지지 않는지 확인

격리 권장: 수정될 때까지 test.fixme()로 표시
```

## 브라우저 설정

기본적으로 여러 브라우저에서 테스트 실행:
- ✅ Chromium (Desktop Chrome)
- ✅ Firefox (Desktop)
- ✅ WebKit (Desktop Safari)
- ✅ Mobile Chrome (선택사항)

브라우저 조정은 `playwright.config.ts`에서 설정.

## CI/CD 통합

CI 파이프라인에 추가:

```yaml
# .github/workflows/e2e.yml
- name: Install Playwright
  run: npx playwright install --with-deps

- name: Run E2E tests
  run: npx playwright test

- name: Upload artifacts
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX 전용 핵심 Flow

PMX의 경우 다음 E2E 테스트 우선순위:

**🔴 CRITICAL (항상 통과해야 함):**
1. 사용자가 지갑을 연결할 수 있음
2. 사용자가 마켓을 탐색할 수 있음
3. 사용자가 마켓을 검색할 수 있음 (시맨틱 검색)
4. 사용자가 마켓 상세를 볼 수 있음
5. 사용자가 거래를 할 수 있음 (테스트 자금)
6. 마켓이 올바르게 해결됨
7. 사용자가 자금을 인출할 수 있음

**🟡 IMPORTANT:**
1. 마켓 생성 flow
2. 사용자 프로필 업데이트
3. 실시간 가격 업데이트
4. 차트 렌더링
5. 마켓 필터 및 정렬
6. 모바일 반응형 레이아웃

## Best Practice

**해야 할 것:**
- ✅ 유지보수성을 위해 Page Object Model 사용
- ✅ selector에 data-testid attribute 사용
- ✅ 임의의 타임아웃이 아닌 API 응답 대기
- ✅ 핵심 사용자 journey를 end-to-end로 테스트
- ✅ main 병합 전에 테스트 실행
- ✅ 테스트 실패 시 아티팩트 검토

**하지 말아야 할 것:**
- ❌ 취약한 selector 사용 (CSS 클래스는 변경될 수 있음)
- ❌ 구현 세부사항 테스트
- ❌ 프로덕션에서 테스트 실행
- ❌ flaky 테스트 무시
- ❌ 실패 시 아티팩트 검토 건너뛰기
- ❌ 모든 edge case를 E2E로 테스트 (unit 테스트 사용)

## 중요 참고사항

**PMX의 경우 중요:**
- 실제 돈이 관련된 E2E 테스트는 반드시 testnet/staging에서만 실행
- 프로덕션에서 거래 테스트 절대 금지
- 금융 테스트에 `test.skip(process.env.NODE_ENV === 'production')` 설정
- 소량의 테스트 자금만 있는 테스트 지갑 사용

## 다른 Command와의 통합

- `/plan`으로 테스트할 핵심 journey 식별
- `/tdd`로 unit 테스트 (더 빠르고 세분화된)
- `/e2e`로 통합 및 사용자 journey 테스트
- `/code-review`로 테스트 품질 확인

## 관련 Agent

이 command는 다음 위치의 `e2e-runner` agent를 호출합니다:
`~/.claude/agents/e2e-runner.md`

## 빠른 명령어

```bash
# 모든 E2E 테스트 실행
npx playwright test

# 특정 테스트 파일 실행
npx playwright test tests/e2e/markets/search.spec.ts

# headed 모드로 실행 (브라우저 표시)
npx playwright test --headed

# 테스트 디버그
npx playwright test --debug

# 테스트 코드 생성
npx playwright codegen http://localhost:3000

# 리포트 보기
npx playwright show-report
```
