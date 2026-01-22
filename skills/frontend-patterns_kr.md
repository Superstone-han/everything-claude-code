---
name: frontend-patterns
description: React, Next.js, 상태 관리, 성능 최적화 및 UI 모범 사례를 위한 프론트엔드 개발 패턴.
---

# 프론트엔드 개발 패턴

React, Next.js 및 고성능 사용자 인터페이스를 위한 현대적 프론트엔드 패턴입니다.

## 구성요소 패턴

### 상속보다 구성

```typescript
// ✅ 좋음: 구성요소 구성
export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}

// 사용
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
</Card>
```

### 복합 구성요소

Context를 사용하여 관련 상태를 공유하세요.

### Render Props 패턴

```typescript
// 자식으로 함수 전달
<DataLoader url="/api/markets">
  {(markets, loading, error) => {
    if (loading) return <Spinner />
    return <MarketList markets={markets!} />
  }}
</DataLoader>
```

## 사용자 정의 Hooks 패턴

### 상태 관리 Hook

```typescript
export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)
  const toggle = useCallback(() => setValue(v => !v), [])
  return [value, toggle]
}
```

### 비동기 데이터 가져오기 Hook

```typescript
export function useQuery<T>(
  key: string,
  fetcher: () => Promise<T>
) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [loading, setLoading] = useState(false)

  const refetch = useCallback(async () => {
    setLoading(true)
    try {
      const result = await fetcher()
      setData(result)
    } catch (err) {
      setError(err as Error)
    } finally {
      setLoading(false)
    }
  }, [fetcher])

  useEffect(() => {
    refetch()
  }, [key, refetch])

  return { data, error, loading, refetch }
}
```

### 디바운스 Hook

```typescript
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

## 상태 관리 패턴

### Context + Reducer 패턴

```typescript
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_MARKETS':
      return { ...state, markets: action.payload }
    case 'SELECT_MARKET':
      return { ...state, selectedMarket: action.payload }
    default:
      return state
  }
}

export function MarketProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, {
    markets: [],
    selectedMarket: null,
    loading: false
  })

  return (
    <MarketContext.Provider value={{ state, dispatch }}>
      {children}
    </MarketContext.Provider>
  )
}
```

## 성능 최적화

### 메모이제이션

```typescript
// ✅ useMemo는 비용이 많은 계산용
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ useCallback은 자식에게 전달되는 함수용
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 코드 분할 및 지연 로딩

```typescript
// ✅ 무거운 구성요소 지연 로딩
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart data={data} />
    </Suspense>
  )
}
```

### 긴 목록의 가상화

```typescript
import { useVirtualizer } from '@tanstack/react-virtual'

export function VirtualMarketList({ markets }: { markets: Market[] }) {
  const parentRef = useRef<HTMLDivElement>(null)

  const virtualizer = useVirtualizer({
    count: markets.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100,
    overscan: 5
  })

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px` }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              transform: `translateY(${virtualRow.start}px)`
            }}
          >
            <MarketCard market={markets[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  )
}
```

## 양식 처리 패턴

### 유효성 검사가 포함된 제어 양식

```typescript
export function CreateMarketForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    description: '',
    endDate: ''
  })

  const [errors, setErrors] = useState<FormErrors>({})

  const validate = (): boolean => {
    const newErrors: FormErrors = {}
    if (!formData.name.trim()) {
      newErrors.name = 'Name is required'
    }
    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    if (!validate()) return
    await createMarket(formData)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
      />
      {errors.name && <span className="error">{errors.name}</span>}
    </form>
  )
}
```

## 오류 경계 패턴

```typescript
export class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error: Error | null }
> {
  state = { hasError: false, error: null }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error boundary caught:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
        </div>
      )
    }
    return this.props.children
  }
}
```

## 애니메이션 패턴

### Framer Motion 애니메이션

```typescript
import { motion, AnimatePresence } from 'framer-motion'

export function AnimatedMarketList({ markets }: { markets: Market[] }) {
  return (
    <AnimatePresence>
      {markets.map(market => (
        <motion.div
          key={market.id}
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -20 }}
        >
          <MarketCard market={market} />
        </motion.div>
      ))}
    </AnimatePresence>
  )
}
```

## 접근성 패턴

### 키보드 내비게이션

```typescript
export function Dropdown({ options, onSelect }: DropdownProps) {
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        setActiveIndex(i => Math.min(i + 1, options.length - 1))
        break
      case 'Enter':
        onSelect(options[activeIndex])
        break
    }
  }

  return (
    <div
      role="combobox"
      aria-expanded={isOpen}
      onKeyDown={handleKeyDown}
    >
      {/* 구현 */}
    </div>
  )
}
```

**기억하기**: 현대적 프론트엔드 패턴은 유지 관리 가능하고 고성능 사용자 인터페이스를 가능하게 합니다. 프로젝트 복잡도에 맞는 패턴을 선택하세요.
