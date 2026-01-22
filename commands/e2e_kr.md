---
description: Playwright로 엔드투엔드 테스트를 생성하고 실행합니다. 테스트 여정을 만들고, 테스트를 실행하며, 스크린샷/비디오/트레이스를 캡처하고 아티팩트를 업로드합니다.
---

# E2E 명령어

이 명령어는 **e2e-runner** 에이전트를 호출하여 Playwright로 엔드투엔드 테스트를 생성, 유지, 실행합니다.

## 이 명령어가 하는 일

1. **테스트 여정 생성** - 사용자 흐름을 위한 Playwright 테스트 생성
2. **E2E 테스트 실행** - 여러 브라우저에서 테스트 실행
3. **아티팩트 캡처** - 실패 시 스크린샷, 비디오, 트레이스
4. **결과 업로드** - HTML 보고서와 JUnit XML
5. **불안정한 테스트 식별** - 불안정한 테스트 격리

## 사용 시점

다음 상황에 `/e2e`를 사용하세요:
- 중요한 사용자 여정 테스트 (로그인, 거래, 결제)
- 멀티스텝 플로우가 E2E로 정상 동작하는지 확인
- UI 상호작용과 네비게이션 테스트
- 프론트엔드/백엔드 통합 검증
- 프로덕션 배포 준비

## 작동 방식

e2e-runner 에이전트는 다음을 수행합니다:

1. **사용자 플로우 분석** 후 테스트 시나리오 식별
2. **Playwright 테스트 생성** - Page Object Model 패턴 사용
3. **테스트 실행** - 여러 브라우저에서 실행 (Chrome, Firefox, Safari)
4. **실패 캡처** - 스크린샷, 비디오, 트레이스
5. **보고서 생성** - 결과와 아티팩트 포함
6. **불안정한 테스트 식별** 및 수정 권장

## 사용 예시

```
User: /e2e Test the market search and view flow

Agent (e2e-runner):
# E2E Test Generation: Market Search and View Flow

## Test Scenario Identified

**User Journey:** Market Search → View Results → Click Market → View Details

**Test Coverage:**
1. Navigate to markets page
2. Perform semantic search
3. Verify search results
4. Click on first result
5. Verify market details page loads
6. Verify chart renders

## Generated Test Code

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('Market Search and View Flow', () => {
  test('user can search markets and view details', async ({ page }) => {
    // 1. Navigate to markets page
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Verify page loaded
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. Perform semantic search
    await marketsPage.searchMarkets('election')

    // Wait for API response
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. Verify search results
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // Take screenshot of search results
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. Click on first result
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. Verify market details page loads
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. Verify chart renders
    await expect(detailsPage.priceChart).toBeVisible()

    // Verify market name matches
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // Take screenshot of market details
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('search with no results shows empty state', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Search for non-existent market
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // Verify empty state
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('can clear search and see all markets again', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Initial market count
    const initialCount = await marketsPage.marketCards.count()

    // Perform search
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // Verify filtered results
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // Clear search
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // Verify all markets shown again
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

  ✓  [chromium] › search-and-view.spec.ts:5:3 › user can search markets and view details (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › search with no results shows empty state (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › can clear search and see all markets again (2.9s)

  3 passed (9.1s)

Artifacts generated:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

✅ CI/CD 통합을 위한 E2E 테스트 스위트 준비 완료!
```

## 테스트 보고서

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E Test Results                          ║
╠══════════════════════════════════════════════════════════════╣
║ Status:     ✅ ALL TESTS PASSED                              ║
║ Total:      3 tests                                          ║
║ Passed:     3 (100%)                                         ║
║ Failed:     0                                                ║
║ Flaky:      0                                                ║
║ Duration:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

Artifacts:
📸 Screenshots: 2 files
📹 Videos: 0 files (only on failure)
🔍 Traces: 0 files (only on failure)
📊 HTML Report: playwright-report/index.html

View report: npx playwright show-report
```

✅ E2E test suite ready for CI/CD integration!
```

## 테스트 아티팩트

테스트가 실행되면 다음 아티팩트가 캡처됩니다:

**모든 테스트에서:**
- 타임라인과 결과가 포함된 HTML 보고서
- CI 통합을 위한 JUnit XML

**실패 시에만:**
- 실패 상태의 스크린샷
- 테스트 비디오 녹화
- 디버깅용 트레이스 파일 (단계별 재생)
- 네트워크 로그
- 콘솔 로그

## 아티팩트 보기

```bash
# 브라우저에서 HTML 보고서 보기
npx playwright show-report

# 특정 트레이스 파일 보기
npx playwright show-trace artifacts/trace-abc123.zip

# 스크린샷은 artifacts/ 디렉터리에 저장됨
open artifacts/search-results.png
```

## 불안정한 테스트 감지

테스트가 간헐적으로 실패한다면:

```
⚠️  FLAKY TEST DETECTED: tests/e2e/markets/trade.spec.ts

Test passed 7/10 runs (70% pass rate)

Common failure:
"Timeout waiting for element '[data-testid="confirm-btn"]'"

Recommended fixes:
1. Add explicit wait: await page.waitForSelector('[data-testid="confirm-btn"]')
2. Increase timeout: { timeout: 10000 }
3. Check for race conditions in component
4. Verify element is not hidden by animation

Quarantine recommendation: Mark as test.fixme() until fixed
```

## 브라우저 구성

테스트는 기본적으로 여러 브라우저에서 실행됩니다:
- ✅ Chromium (Desktop Chrome)
- ✅ Firefox (Desktop)
- ✅ WebKit (Desktop Safari)
- ✅ Mobile Chrome (optional)

`playwright.config.ts`에서 브라우저를 조정하세요.

## CI/CD 통합

CI 파이프라인에 다음을 추가하세요:

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

## PMX 전용 핵심 흐름

PMX에서는 다음 E2E 테스트를 우선하세요:

**🔴 CRITICAL (항상 통과해야 함):**
1. User can connect wallet
2. User can browse markets
3. User can search markets (semantic search)
4. User can view market details
5. User can place trade (with test funds)
6. Market resolves correctly
7. User can withdraw funds

**🟡 IMPORTANT:**
1. Market creation flow
2. User profile updates
3. Real-time price updates
4. Chart rendering
5. Filter and sort markets
6. Mobile responsive layout

## 모범 사례

**DO:**
- ✅ 유지보수를 위해 Page Object Model 사용
- ✅ 선택자에 data-testid 속성 사용
- ✅ 임의 타임아웃 대신 API 응답 대기
- ✅ 핵심 사용자 여정을 E2E로 테스트
- ✅ main 병합 전 테스트 실행
- ✅ 테스트 실패 시 아티팩트 검토

**DON'T:**
- ❌ 취약한 선택자 사용 (CSS 클래스는 변경 가능)
- ❌ 구현 세부 사항 테스트
- ❌ 프로덕션 대상으로 테스트 실행
- ❌ 불안정한 테스트 무시
- ❌ 실패 시 아티팩트 검토 생략
- ❌ 모든 엣지 케이스를 E2E로 테스트 (단위 테스트 사용)

## 중요 참고사항

**PMX에 중요:**
- 실제 돈이 오가는 E2E 테스트는 반드시 테스트넷/스테이징에서만 실행
- 프로덕션에서는 거래 테스트를 절대 실행하지 않음
- 금융 테스트에는 `test.skip(process.env.NODE_ENV === 'production')` 설정
- 테스트 지갑에는 소액 테스트 자금만 사용

## 다른 명령어와의 연계

- `/plan`으로 테스트할 핵심 여정 식별
- `/tdd`로 단위 테스트 작성 (더 빠르고 더 세밀함)
- `/e2e`로 통합/사용자 여정 테스트
- `/code-review`로 테스트 품질 검증

## 관련 에이전트

이 명령어는 다음 위치의 `e2e-runner` 에이전트를 호출합니다:
`~/.claude/agents/e2e-runner.md`

## 빠른 명령어

```bash
# 모든 E2E 테스트 실행
npx playwright test

# 특정 테스트 파일 실행
npx playwright test tests/e2e/markets/search.spec.ts

# 헤디드 모드로 실행 (브라우저 표시)
npx playwright test --headed

# 테스트 디버그
npx playwright test --debug

# 테스트 코드 생성
npx playwright codegen http://localhost:3000

# 보고서 보기
npx playwright show-report
```
