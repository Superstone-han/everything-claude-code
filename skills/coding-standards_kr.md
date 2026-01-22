---
name: coding-standards
description: TypeScript, JavaScript, React 및 Node.js 개발을 위한 보편적인 코딩 표준, 모범 사례 및 패턴.
---

# 코딩 표준 및 모범 사례

모든 프로젝트에 적용 가능한 보편적인 코딩 표준입니다.

## 코드 품질 원칙

### 1. 가독성 우선
- 코드는 읽히는 것보다 더 자주 작성됩니다
- 명확한 변수 및 함수 이름
- 주석보다 자체 문서화된 코드 선호
- 일관된 포맷팅

### 2. KISS (Keep It Simple, Stupid)
- 작동하는 가장 단순한 솔루션
- 과도한 엔지니어링 피하기
- 조기 최적화 금지
- 이해하기 쉬운 것 > 영리한 코드

### 3. DRY (Don't Repeat Yourself)
- 공통 로직을 함수로 추출
- 재사용 가능한 구성요소 생성
- 모듈 간 유틸리티 공유
- 복사-붙여넣기 프로그래밍 금지

### 4. YAGNI (You Aren't Gonna Need It)
- 필요할 때까지 기능을 구축하지 마세요
- 추론적인 일반성 피하기
- 필요할 때만 복잡성 추가
- 간단하게 시작, 필요 시 리팩토링

## TypeScript/JavaScript 표준

### 변수 명명

```typescript
// ✅ 좋음: 설명적인 이름
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 나쁨: 불명확한 이름
const q = 'election'
const flag = true
const x = 1000
```

### 함수 명명

```typescript
// ✅ 좋음: 동사-명사 패턴
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 나쁨: 불명확하거나 명사만
async function market(id: string) { }
function similarity(a, b) { }
```

### 불변성 패턴 (중요)

```typescript
// ✅ 항상 스프레드 연산자 사용
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 절대 직접 변경
user.name = 'New Name'  // 나쁨
items.push(newItem)     // 나쁨
```

### 오류 처리

```typescript
// ✅ 좋음: 포괄적인 오류 처리
async function fetchData(url: string) {
  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }
    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}
```

### Async/Await 모범 사례

```typescript
// ✅ 좋음: 가능할 때 병렬 실행
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])
```

## React 모범 사례

### 구성요소 구조

```typescript
// ✅ 좋음: 타입이 있는 함수형 구성요소
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
}

export function Button({ children, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  )
}
```

### 사용자 정의 Hooks

```typescript
// ✅ 좋음: 재사용 가능한 사용자 정의 hook
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

### 상태 관리

```typescript
// ✅ 좋음: 적절한 상태 업데이트
const [count, setCount] = useState(0)

// 이전 상태를 기반으로 한 함수형 업데이트
setCount(prev => prev + 1)
```

## API 설계 표준

### REST API 규칙

```
GET    /api/markets              # 모든 마켓 목록
GET    /api/markets/:id          # 특정 마켓 가져오기
POST   /api/markets              # 새 마켓 생성
PUT    /api/markets/:id          # 마켓 업데이트 (전체)
PATCH  /api/markets/:id          # 마켓 업데이트 (부분)
DELETE /api/markets/:id          # 마켓 삭제
```

### 응답 형식

```typescript
// ✅ 좋음: 일관된 응답 구조
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

### 입력 유효성 검사

```typescript
import { z } from 'zod'

const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime()
})

export async function POST(request: Request) {
  const body = await request.json()
  const validated = CreateMarketSchema.parse(body)
  // 유효성 검사된 데이터로 진행
}
```

## 파일 구성

### 프로젝트 구조

```
src/
├── app/                    # Next.js App Router
├── components/             # React 구성요소
├── hooks/                  # 사용자 정의 React hooks
├── lib/                    # 유틸리티
├── types/                  # TypeScript 타입
└── styles/                 # 전역 스타일
```

## 성능 모범 사례

### 메모이제이션

```typescript
// ✅ 좋음: 비용이 많은 계산 메모이제이션
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])
```

### 지연 로딩

```typescript
// ✅ 좋음: 무거운 구성요소 지연 로딩
const HeavyChart = lazy(() => import('./HeavyChart'))
```

**기억하기**: 코드 품질은 타협할 수 없습니다. 명확하고 유지 관리 가능한 코드는 빠른 개발과 자신 있는 리팩토링을 가능하게 합니다.
