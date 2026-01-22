---
name: tdd-workflow
description: 새로운 기능을 작성하거나, 버그를 수정하거나, 코드를 리팩토링할 때 이 skill을 사용하세요. 80% 이상의 커버리지를 보장하는 테스트 주도 개발을 강제합니다.
---

# 테스트 주도 개발 워크플로우

이 skill은 모든 코드 개발이 포괄적인 테스트 커버리지로 TDD 원칙을 따르도록 합니다.

## 활성화 시점

- 새로운 기능 또는 기능성 작성
- 버그 또는 이슈 수정
- 기존 코드 리팩토링
- API 엔드포인트 추가
- 새 구성요소 생성

## 핵심 원칙

### 1. 테스트 우선 코드
항상 테스트를 먼저 작성한 다음 테스트를 통과할 코드를 구현하세요.

### 2. 커버리지 요구사항
- 최소 80% 커버리지 (단위 + 통합 + E2E)
- 모든 엣지 케이스 커버
- 오류 시나리오 테스트
- 경계 조건 검증

### 3. 테스트 유형

#### 단위 테스트
- 개별 함수 및 유틸리티
- 구성요소 로직
- 순수 함수
- 헬퍼 및 유틸리티

#### 통합 테스트
- API 엔드포인트
- 데이터베이스 작업
- 서비스 상호작용
- 외부 API 호출

#### E2E 테스트 (Playwright)
- 중요한 사용자 플로우
- 완전한 워크플로우
- 브라우저 자동화
- UI 상호작용

## TDD 워크플로우 단계

### 단계 1: 사용자 여정 작성
```
[역할]로서, [작업]을 원합니다. 그래서 [혜택]

예시:
사용자로서, 정확한 키워드 없이도
관련 마켓을 찾을 수 있기를 원합니다.
```

### 단계 2: 테스트 케이스 생성
각 사용자 여정에 대해 포괄적인 테스트 케이스 작성:

```typescript
describe('Semantic Search', () => {
  it('쿼리에 대한 관련 마켓을 반환합니다', async () => {
    // 테스트 구현
  })

  it('빈 쿼리를 우아하게 처리합니다', async () => {
    // 엣지 케이스 테스트
  })

  it('Redis를 사용할 수 없을 때 하위 문자열 검색으로 대체합니다', async () => {
    // 대체 동작 테스트
  })
})
```

### 단계 3: 테스트 실행 (실패해야 함)
```bash
npm test
# 아직 구현하지 않았으므로 테스트가 실패해야 함
```

### 단계 4: 코드 구현
테스트를 통과하는 최소 코드 작성:

```typescript
// 테스트에 의해 안내되는 구현
export async function searchMarkets(query: string) {
  // 구현
}
```

### 단계 5: 테스트 재실행
```bash
npm test
# 이제 테스트가 통과해야 함
```

### 단계 6: 리팩토링
테스트를 녹색으로 유지하면서 코드 품질 개선:
- 중복 제거
- 이름 개선
- 성능 최적화
- 가독성 향상

### 단계 7: 커버리지 확인
```bash
npm run test:coverage
# 80% 이상 커버리지 달성 확인
```

## 테스트 패턴

### 단위 테스트 패턴 (Jest/Vitest)
```typescript
describe('Button Component', () => {
  it('올바른 텍스트로 렌더링합니다', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('클릭 시 onClick를 호출합니다', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click</Button>)
    fireEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### API 통합 테스트 패턴
```typescript
describe('GET /api/markets', () => {
  it('마켓을 성공적으로 반환합니다', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
  })

  it('데이터베이스 오류를 우아하게 처리합니다', async () => {
    // 데이터베이스 실패 모의
    const request = new NextRequest('http://localhost/api/markets')
    // 오류 처리 테스트
  })
})
```

### E2E 테스트 패턴 (Playwright)
```typescript
test('사용자가 마켓을 검색하고 필터링할 수 있습니다', async ({ page }) => {
  await page.goto('/')
  await page.fill('input[placeholder="Search markets"]', 'election')
  await page.waitForTimeout(600)

  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })
})
```

## 외부 서비스 모의

### Supabase 모의
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: mockMarkets,
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis 모의
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ]))
}))
```

## 테스트 파일 구성

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   └── Button.test.tsx          # 단위 테스트
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 통합 테스트
└── e2e/
    ├── markets.spec.ts               # E2E 테스트
    └── trading.spec.ts
```

## 테스트 커버리지 검증

### 커버리지 보고서 실행
```bash
npm run test:coverage
```

### 커버리지 임계값
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## 일반적인 테스트 실수 피하기

### ❌ 잘못됨: 구현 세부사항 테스트
```typescript
// 내부 상태를 테스트하지 마세요
expect(component.state.count).toBe(5)
```

### ✅ 올바름: 사용자에게 보이는 동작 테스트
```typescript
// 사용자가 보는 것을 테스트하세요
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### ❌ 잘못됨: 취약한 선택자
```typescript
// 쉽게 깨짐
await page.click('.css-class-xyz')
```

### ✅ 올바름: 시맨틱 선택자
```typescript
// 변경에 탄력적
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

## 지속적 테스트

### 개발 중 감시 모드
```bash
npm test -- --watch
```

### 사전 커밋 훅
```bash
# 모든 커밋 전 실행
npm test && npm run lint
```

### CI/CD 통합
```yaml
- name: Run Tests
  run: npm test -- --coverage
```

## 모범 사례

1. **테스트 먼저 작성** - 항상 TDD
2. **테스트당 하나의 어설션** - 단일 동작에 중점
3. **설명적인 테스트 이름** - 테스트 내용 설명
4. **배열-동작-어설션** - 명확한 테스트 구조
5. **외부 종속성 모의** - 단위 테스트 격리
6. **엣지 케이스 테스트** - Null, undefined, 비어있음, 큰 값
7. **오류 경로 테스트** - 정상 경로만 아님
8. **테스트를 빠르게 유지** - 단위 테스트 < 50ms
9. **테스트 후 정리** - 부작용 없음
10. **커버리지 보고서 검토** - 격차 식별

## 성공 메트릭

- 80% 이상 코드 커버리지 달성
- 모든 테스트 통과 (녹색)
- 건너뛰거나 비활성화된 테스트 없음
- 빠른 테스트 실행 (단위 테스트 < 30초)
- 중요 사용자 플로우를 위한 E2E 테스트
- 테스트가 프로덕션 버그를 포착

---

**기억하기**: 테스트는 선택 사항이 아닙니다. 테스트는 자신 있는 리팩토링, 빠른 개발, 프로덕션 신뢰성을 가능하게 하는 안전망입니다.
