# 프로젝트 지침 Skill (예시)

프로젝트별 skill의 예시입니다. 프로젝트용 템플릿으로 사용하세요.

실제 프로덕션 애플리케이션 기반: [Zenith](https://zenith.chat) - AI 기반 고객 발견 플랫폼.

---

## 사용 시점

이 skill은 해당 skill이 설계된 특정 프로젝트에서 작업할 때 참조하세요. 프로젝트 skill은 다음을 포함합니다:
- 아키텍처 개요
- 파일 구조
- 코드 패턴
- 테스트 요구사항
- 배포 워크플로우

---

## 아키텍처 개요

**기술 스택:**
- **프론트엔드**: Next.js 15 (App Router), TypeScript, React
- **백엔드**: FastAPI (Python), Pydantic 모델
- **데이터베이스**: Supabase (PostgreSQL)
- **AI**: Claude API with tool calling 및 구조화된 출력
- **배포**: Google Cloud Run
- **테스트**: Playwright (E2E), pytest (백엔드), React Testing Library

---

## 파일 구조

```
project/
├── frontend/
│   └── src/
│       ├── app/              # Next.js app router 페이지
│       ├── components/       # React 구성요소
│       ├── hooks/            # 사용자 정의 React hooks
│       └── lib/              # 유틸리티
├── backend/
│   ├── routers/              # FastAPI 경로 핸들러
│   ├── models.py             # Pydantic 모델
│   └── tests/                # pytest 테스트
└── deploy/                   # 배포 구성
```

---

## 코드 패턴

### API 응답 형식 (FastAPI)

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, data: T) -> "ApiResponse[T]":
        return cls(success=True, data=data)

    @classmethod
    def fail(cls, error: str) -> "ApiResponse[T]":
        return cls(success=False, error=error)
```

### Claude AI 통합 (구조화된 출력)

```python
from anthropic import Anthropic
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    summary: str
    key_points: list[str]
    confidence: float

async def analyze_with_claude(content: str) -> AnalysisResult:
    client = Anthropic()

    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "input_schema": AnalysisResult.model_json_schema()
        }],
        tool_choice={"type": "tool", "name": "provide_analysis"}
    )

    return AnalysisResult(**tool_use.input)
```

---

## 테스트 요구사항

### 백엔드 (pytest)

```bash
# 모든 테스트 실행
poetry run pytest tests/

# 커버리지로 실행
poetry run pytest tests/ --cov=. --cov-report=html
```

### 프론트엔드 (React Testing Library)

```bash
# 테스트 실행
npm run test

# 커버리지로 실행
npm run test -- --coverage
```

---

## 배포 워크플로우

### 배포 전 체크리스트

- [ ] 모든 테스트가 로컬에서 통과
- [ ] `npm run build` 성공
- [ ] `poetry run pytest` 통과
- [ ] 하드코딩된 비밀키 없음
- [ ] 환경 변수가 문서화됨

---

## 중요 규칙

1. **이모지 금지** - 코드, 주석, 문서에
2. **불변성** - 객체나 배열을 절대 변경하지 마세요
3. **TDD** - 구현 전 테스트 작성
4. **80% 커버리지** 최소
5. **다수의 소형 파일** - 일반적 200-400줄, 최대 800
6. **console.log 금지** - 프로덕션 코드에서
7. **적절한 오류 처리** - try/catch로
8. **입력 유효성 검사** - Pydantic/Zod로

---

## 관련 Skills

- `coding-standards.md` - 일반적인 코딩 모범 사례
- `backend-patterns.md` - API 및 데이터베이스 패턴
- `frontend-patterns.md` - React 및 Next.js 패턴
- `tdd-workflow/` - 테스트 주도 개발 방법론
