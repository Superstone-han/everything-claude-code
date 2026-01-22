---
name: backend-patterns
description: Node.js, Express, Next.js API 경로를 위한 백엔드 아키텍처 패턴, API 설계, 데이터베이스 최적화 및 서버 측 모범 사례.
---

# 백엔드 개발 패턴

확장 가능한 서버 측 애플리케이션을 위한 백엔드 아키텍처 패턴 및 모범 사례입니다.

## API 설계 패턴

### RESTful API 구조

```typescript
// ✅ 리소스 기반 URL
GET    /api/markets                 # 리소스 목록
GET    /api/markets/:id             # 단일 리소스 가져오기
POST   /api/markets                 # 리소스 생성
PUT    /api/markets/:id             # 리소스 교체
PATCH  /api/markets/:id             # 리소스 업데이트
DELETE /api/markets/:id             # 리소스 삭제

// ✅ 필터링, 정렬, 페이지네이션용 쿼리 매개변수
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### 저장소 패턴

데이터 액세스 로직을 추상화하세요.

### 서비스 계층 패턴

데이터 액세스에서 비즈니스 로직을 분리하세요.

### 미들웨어 패턴

요청/응답 처리 파이프라인을 구현하세요.

## 데이터베이스 패턴

### 쿼리 최적화

```typescript
// ✅ 좋음: 필요한 열만 선택
const { data } = await supabase
  .from('markets')
  .select('id, name, status, volume')
  .eq('status', 'active')
  .order('volume', { ascending: false })
  .limit(10)

// ❌ 나쁨: 모두 선택
const { data } = await supabase
  .from('markets')
  .select('*')
```

### N+1 쿼리 방지

```typescript
// ❌ 나쁨: N+1 쿼리 문제
const markets = await getMarkets()
for (const market of markets) {
  market.creator = await getUser(market.creator_id)  // N개 쿼리
}

// ✅ 좋음: 일괄 가져오기
const markets = await getMarkets()
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds)  // 1개 쿼리
```

### 트랜잭션 패턴

데이터베이스 트랜잭션을 사용하여 원자성을 보장하세요.

## 캐싱 전략

### Redis 캐싱 계층

캐싱을 통해 성능을 향상시키세요.

### Cache-Aside 패턴

캐시 미스 시 데이터베이스에서 가져오세요.

## 오류 처리 패턴

### 중앙 집중식 오류 처리기

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message)
  }
}

export function errorHandler(error: unknown) {
  if (error instanceof ApiError) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: error.statusCode })
  }
  // ...
}
```

### 지수 백오프로 재시도

일시적인 오류에 대한 재시도 논리를 구현하세요.

## 인증 및 인가

### JWT 토큰 검증

```typescript
export function verifyToken(token: string): JWTPayload {
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload
    return payload
  } catch (error) {
    throw new ApiError(401, 'Invalid token')
  }
}
```

### 역할 기반 액세스 제어

```typescript
export function hasPermission(user: User, permission: Permission): boolean {
  return rolePermissions[user.role].includes(permission)
}
```

## 속도 제한

### 간단한 인메모리 속도 제한기

API 엔드포인트를 보호하세요.

## 백그라운드 작업 및 큐

### 간단한 큐 패턴

비동기 작업을 큐에 넣으세요.

## 로깅 및 모니터링

### 구조화된 로깅

```typescript
class Logger {
  log(level: 'info' | 'warn' | 'error', message: string, context?: LogContext) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...context
    }
    console.log(JSON.stringify(entry))
  }
}
```

**기억하기**: 백엔드 패턴은 확장 가능하고 유지 관리 가능한 서버 측 애플리케이션을 가능하게 합니다. 복잡도 수준에 맞는 패턴을 선택하세요.
