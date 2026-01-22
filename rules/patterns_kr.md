# 일반적인 패턴

## API 응답 형식

```typescript
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

## 사용자 정의 Hooks 패턴

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

## 저장소 패턴

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## 스켈레톤 프로젝트

새 기능을 구현할 때:
1. 입증된 스켈레톤 프로젝트 검색
2. 옵션을 평가하기 위해 병렬 agent 사용:
   - 보안 평가
   - 확장성 분석
   - 관련성 점수
   - 구현 계획
3. 기초로 최적 일치를 복제
4. 검증된 구조 내에서 반복
