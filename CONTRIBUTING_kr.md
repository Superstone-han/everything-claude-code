# Everything Claude Code 기여 가이드

기여해주셔서 감사합니다. 이 저장소는 Claude Code 사용자를 위한 커뮤니티 리소스입니다.

## 찾고 있는 것

### Agents

특정 작업을 잘 처리하는 새로운 에이전트:
- 언어별 리뷰어 (Python, Go, Rust)
- 프레임워크 전문가 (Django, Rails, Laravel, Spring)
- DevOps 전문가 (Kubernetes, Terraform, CI/CD)
- 도메인 전문가 (ML 파이프라인, 데이터 엔지니어링, 모바일)

### Skills

워크플로우 정의 및 도메인 지식:
- 언어 모범 사례
- 프레임워크 패턴
- 테스트 전략
- 아키텍처 가이드
- 도메인별 지식

### Commands

유용한 워크플로우를 호출하는 슬래시 명령어:
- 배포 명령어
- 테스트 명령어
- 문서화 명령어
- 코드 생성 명령어

### Hooks

유용한 자동화:
- 린팅/포맷팅 훅
- 보안 체크
- 검증 훅
- 알림 훅

### Rules

항상 따라야 할 지침:
- 보안 규칙
- 코드 스타일 규칙
- 테스트 요구사항
- 명명 규칙

### MCP Configurations

새롭거나 개선된 MCP 서버 설정:
- 데이터베이스 통합
- 클라우드 제공업체 MCP
- 모니터링 도구
- 통신 도구

---

## 기여 방법

### 1. 저장소 포크

```bash
git clone https://github.com/YOUR_USERNAME/everything-claude-code.git
cd everything-claude-code
```

### 2. 브랜치 생성

```bash
git checkout -b add-python-reviewer
```

### 3. 기여 내용 추가

파일을 적절한 디렉토리에 배치하세요:
- `agents/` - 새로운 에이전트
- `skills/` - 스킬 (단일 .md 또는 디렉토리 가능)
- `commands/` - 슬래시 명령어
- `rules/` - 규칙 파일
- `hooks/` - 훅 설정
- `mcp-configs/` - MCP 서버 설정

### 4. 형식 따르기

**Agents**는 frontmatter가 있어야 합니다:

```markdown
---
name: agent-name
description: 기능 설명
tools: Read, Grep, Glob, Bash
model: sonnet
---

여기에 지시사항...
```

**Skills**는 명확하고 실행 가능해야 합니다:

```markdown
# 스킬 이름

## 사용 시기

...

## 작동 방식

...

## 예시

...
```

**Commands**는 수행하는 작업을 설명해야 합니다:

```markdown
---
description: 명령어에 대한 간단한 설명
---

# 명령어 이름

상세 지시사항...
```

**Hooks**는 설명을 포함해야 합니다:

```json
{
  "matcher": "...",
  "hooks": [...],
  "description": "이 훅이 하는 일"
}
```

### 5. 기여 내용 테스트

제출 전에 설정이 Claude Code에서 작동하는지 확인하세요.

### 6. PR 제출

```bash
git add .
git commit -m "Add Python code reviewer agent"
git push origin add-python-reviewer
```

다음 내용과 함께 PR을 여세요:
- 추가한 내용
- 유용한 이유
- 테스트 방법

---

## 가이드라인

### 해야 할 것

- 설정을 집중되고 모듈식으로 유지
- 명확한 설명 포함
- 제출 전 테스트
- 기존 패턴 따르기
- 의존성 문서화

### 하지 말아야 할 것

- 민감한 데이터 포함 (API 키, 토큰, 경로)
- 지나치게 복잡하거나 틈새 설정 추가
- 테스트되지 않은 설정 제출
- 중복 기능 생성
- 대안 없이 특정 유료 서비스를 요구하는 설정 추가

---

## 파일 명명

- 소문자와 하이픈 사용: `python-reviewer.md`
- 설명적으로: `workflow.md` 대신 `tdd-workflow.md`
- 에이전트/스킬 이름을 파일명과 일치시키기

---

## 질문이 있으신가요?

이슈를 열거나 X에서 연락주세요: [@affaanmustafa](https://x.com/affaanmustafa)

---

기여해주셔서 감사합니다. 함께 훌륭한 리소스를 만들어요.
