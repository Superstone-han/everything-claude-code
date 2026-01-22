---
name: security-review
description: 인증 추가, 사용자 입력 처리, 비밀키 작업, API 엔드포인트 생성 또는 결제/민감 기능 구현 시 이 skill을 사용하세요. 포괄적인 보안 체크리스트 및 패턴을 제공합니다.
---

# 보안 검토 Skill

이 skill은 모든 코드가 보안 모범 사례를 따르고 잠재적 취약점을 식별하도록 합니다.

## 활성화 시점

- 인증 또는 인가 구현
- 사용자 입력 또는 파일 업로드 처리
- 새 API 엔드포인트 생성
- 비밀키 또는 자격증명 작업
- 결제 기능 구현
- 민감한 데이터 저장 또는 전송
- 서드파티 API 통합

## 보안 체크리스트

### 1. 비밀키 관리

#### ❌ 절대 이렇게 하지 마세요
```typescript
const apiKey = "sk-proj-xxxxx"  // 하드코딩된 비밀키
```

#### ✅ 항상 이렇게 하세요
```typescript
const apiKey = process.env.OPENAI_API_KEY
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 검증 단계
- [ ] 하드코딩된 API 키, 토큰, 비밀번호 없음
- [ ] 모든 비밀키가 환경 변수에 있음
- [ ] .gitignore에 .env.local 있음
- [ ] git 기록에 비밀키 없음
- [ ] 프로덕션 비밀키가 호스팅 플랫폼에 있음

### 2. 입력 유효성 검사

#### 항상 사용자 입력 유효성 검사
```typescript
import { z } from 'zod'

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

### 3. SQL 인젝션 방지

#### ❌ 절대 SQL을 연결하지 마세요
```typescript
// 위험함 - SQL 인젝션 취약점
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
```

#### ✅ 항상 매개변수화된 쿼리를 사용하세요
```typescript
// 안전함 - 매개변수화된 쿼리
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)
```

### 4. 인증 및 인가

#### JWT 토큰 처리
```typescript
// ❌ 잘못됨: localStorage (XSS에 취약)
localStorage.setItem('token', token)

// ✅ 올바름: httpOnly 쿠키
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict`)
```

#### 인가 확인
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 항상 먼저 인가를 확인
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  await db.users.delete({ where: { id: userId } })
}
```

### 5. XSS 방지

#### HTML 검증
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 항상 사용자 제공 HTML을 검증
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p']
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

### 6. CSRF 방지

```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }
}
```

### 7. 속도 제한

```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 창당 100개 요청
  message: 'Too many requests'
})

app.use('/api/', limiter)
```

### 8. 민감한 데이터 노출

#### 로깅
```typescript
// ❌ 잘못됨: 민감한 데이터 로깅
console.log('User login:', { email, password })

// ✅ 올바름: 로그 검증
console.log('User login:', { email, userId })
```

#### 오류 메시지
```typescript
// ❌ 잘못됨: 내부 세부정보 노출
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 올바름: 일반적인 오류 메시지
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

### 9. 블록체인 보안 (Solana)

#### 지갑 검증
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

### 10. 종속성 보안

```bash
# 취약점 확인
npm audit

# 자동으로 수정 가능한 문제 수정
npm audit fix

# 종속성 업데이트
npm update
```

## 보안 테스트

### 자동화된 보안 테스트
```typescript
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})
```

## 배포 전 보안 체크리스트

모든 프로덕션 배포 전:

- [ ] **비밀키**: 하드코딩 없음, 모두 환경 변수
- [ ] **입력 유효성 검사**: 모든 사용자 입력이 유효성 검사됨
- [ ] **SQL 인젝션**: 모든 쿼리가 매개변수화됨
- [ ] **XSS**: 사용자 콘텐츠가 검증됨
- [ ] **CSRF**: 보호 활성화
- [ ] **인증**: 적절한 토큰 처리
- [ ] **인가**: 역할 확인이 배치됨
- [ ] **속도 제한**: 모든 엔드포인트에 활성화
- [ ] **HTTPS**: 프로덕션에서 강제됨
- [ ] **보안 헤더**: CSP, X-Frame-Options 구성됨
- [ ] **오류 처리**: 오류에 민감한 데이터 없음
- [ ] **로깅**: 로그에 민감한 데이터 없음
- [ ] **종속성**: 최신 상태, 취약점 없음
- [ ] **행 수준 보안**: Supabase에서 활성화
- [ ] **CORS**: 적절히 구성됨
- [ ] **파일 업로드**: 유효성 검사됨 (크기, 타입)

## 리소스

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**기억하기**: 보안은 선택 사항이 아닙니다. 하나의 취약점으로 전체 플랫폼이 손상될 수 있습니다. 의심스러우면 신중하게 대처하세요.
