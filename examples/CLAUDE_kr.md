# 예제 프로젝트 CLAUDE.md

프로젝트 루트에 배치하는 예제 프로젝트 수준 CLAUDE.md 파일입니다.

## 프로젝트 개요

[프로젝트에 대한 간단한 설명 - 기능, 기술 스택]

## 중요 규칙

### 1. 코드 구성

- 소수의 대형 파일보다 다수의 소형 파일
- 높은 응집력, 낮은 결합도
- 일반적 200-400줄, 최대 800줄
- 유형별이 아닌 기능/도메인별로 구성

### 2. 코드 스타일

- 코드, 주석, 문서에 이모지 금지
- 항상 불변성 - 객체나 배열을 절대 변경하지 마세요
- 프로덕션 코드에 console.log 금지
- try/catch로 적절한 오류 처리
- Zod 또는 유사한 것으로 입력 유효성 검사

### 3. 테스트

- TDD: 테스트 먼저 작성
- 80% 최소 커버리지
- 유틸리티용 단위 테스트
- API용 통합 테스트
- 중요 플로우용 E2E 테스트

### 4. 보안

- 하드코딩된 비밀키 금지
- 민감한 데이터는 환경 변수
- 모든 사용자 입력 유효성 검사
- 매개변수화된 쿼리만
- CSRF 보호 활성화

## 파일 구조

```
src/
|-- app/              # Next.js app router
|-- components/       # 재사용 가능한 UI 구성요소
|-- hooks/            # 사용자 정의 React hooks
|-- lib/              # 유틸리티 라이브러리
|-- types/            # TypeScript 정의
```

## 핵심 패턴

### API 응답 형식

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### 오류 처리

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('Operation failed:', error)
  return { success: false, error: 'User-friendly message' }
}
```

## 환경 변수

```bash
# 필수
DATABASE_URL=
API_KEY=

# 선택적
DEBUG=false
```

## 사용 가능한 명령어

- `/tdd` - 테스트 주도 개발 워크플로우
- `/plan` - 구현 계획 생성
- `/code-review` - 코드 품질 검토
- `/build-fix` - 빌드 오류 수정

## Git 워크플로우

- 관례적 커밋: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- main에 직접 커밋하지 마세요
- PR은 검토 필요
- 병합 전 모든 테스트 통과 필수
