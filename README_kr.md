# Everything Claude Code

**Anthropic 해커톤 우승자가 만든 Claude Code 설정의 완전한 컬렉션입니다.**

실제 제품을 만들며 10개월 이상 매일 집중적으로 사용하면서 발전시킨, 프로덕션 준비가 된 에이전트, 스킬, 훅, 명령어, 규칙, MCP 설정 모음입니다.

---

## 가이드

이 저장소는 원본 코드만 포함합니다. 모든 설명은 가이드에 있습니다.

### 여기부터: 축약 가이드

<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />

**[Claude Code 모든 것의 축약 가이드](https://x.com/affaanmustafa/status/2012378465664745795)**

기초 내용 - 각 설정 유형이 무엇을 하는지, 설정을 어떻게 구성하는지, 컨텍스트 윈도우 관리, 그리고 이 설정들의 철학을 설명합니다. **먼저 이 글을 읽어주세요.**

---

### 다음: 장문 가이드

<img width="609" height="428" alt="image" src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" />

**[Claude Code 모든 것의 장문 가이드](https://x.com/affaanmustafa/status/2014040193557471352)**

고급 기법 - 토큰 최적화, 세션 간 메모리 유지, 검증 루프와 평가, 병렬화 전략, 서브에이전트 오케스트레이션, 지속 학습. 이 가이드의 모든 내용은 이 저장소에 실제 코드로 포함되어 있습니다.

| 주제 | 배울 내용 |
|-------|-------------------|
| 토큰 최적화 | 모델 선택, 시스템 프롬프트 축소, 백그라운드 프로세스 |
| 메모리 유지 | 세션 간 컨텍스트를 자동으로 저장/로드하는 훅 |
| 지속 학습 | 세션에서 패턴을 자동 추출해 재사용 가능한 스킬로 만들기 |
| 검증 루프 | 체크포인트 vs 연속 평가, 그레이더 유형, pass@k 지표 |
| 병렬화 | Git 워크트리, 캐스케이드 방식, 인스턴스 확장 시점 |
| 서브에이전트 오케스트레이션 | 컨텍스트 문제, 반복적 검색 패턴 |

---

## 구성

```
everything-claude-code/
|-- agents/           # 위임용 전문 서브에이전트
|   |-- planner.md           # 기능 구현 계획
|   |-- architect.md         # 시스템 설계 결정
|   |-- tdd-guide.md         # 테스트 주도 개발
|   |-- code-reviewer.md     # 품질 및 보안 리뷰
|   |-- security-reviewer.md # 취약점 분석
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 테스트
|   |-- refactor-cleaner.md  # 데드 코드 정리
|   |-- doc-updater.md       # 문서 동기화
|
|-- skills/           # 워크플로우 정의 및 도메인 지식
|   |-- coding-standards.md         # 언어 모범 사례
|   |-- backend-patterns.md         # API, 데이터베이스, 캐싱 패턴
|   |-- frontend-patterns.md        # React, Next.js 패턴
|   |-- continuous-learning/        # 세션에서 패턴 자동 추출 (장문 가이드)
|   |-- strategic-compact/          # 수동 컴팩션 제안 (장문 가이드)
|   |-- tdd-workflow/               # TDD 방법론
|   |-- security-review/            # 보안 체크리스트
|
|-- commands/         # 빠른 실행용 슬래시 명령어
|   |-- tdd.md              # /tdd - 테스트 주도 개발
|   |-- plan.md             # /plan - 구현 계획
|   |-- e2e.md              # /e2e - E2E 테스트 생성
|   |-- code-review.md      # /code-review - 품질 리뷰
|   |-- build-fix.md        # /build-fix - 빌드 오류 수정
|   |-- refactor-clean.md   # /refactor-clean - 데드 코드 제거
|   |-- learn.md            # /learn - 세션 중 패턴 추출 (장문 가이드)
|
|-- rules/            # 항상 따라야 할 지침
|   |-- security.md         # 필수 보안 체크
|   |-- coding-style.md     # 불변성, 파일 조직
|   |-- testing.md          # TDD, 80% 커버리지 요구사항
|   |-- git-workflow.md     # 커밋 형식, PR 프로세스
|   |-- agents.md           # 서브에이전트 위임 시기
|   |-- performance.md      # 모델 선택, 컨텍스트 관리
|
|-- hooks/            # 트리거 기반 자동화
|   |-- hooks.json                # 모든 훅 설정 (PreToolUse, PostToolUse, Stop 등)
|   |-- memory-persistence/       # 세션 라이프사이클 훅 (장문 가이드)
|   |   |-- pre-compact.sh        # 컴팩션 전 상태 저장
|   |   |-- session-start.sh      # 이전 컨텍스트 로드
|   |   |-- session-end.sh        # 종료 시 학습 내용 저장
|   |-- strategic-compact/        # 컴팩션 제안 (장문 가이드)
|
|-- contexts/         # 동적 시스템 프롬프트 주입 컨텍스트 (장문 가이드)
|   |-- dev.md              # 개발 모드 컨텍스트
|   |-- review.md           # 코드 리뷰 모드 컨텍스트
|   |-- research.md         # 리서치/탐색 모드 컨텍스트
|
|-- examples/         # 설정 예시
|   |-- CLAUDE.md           # 예시 프로젝트 수준 설정
|   |-- user-CLAUDE.md      # 예시 사용자 수준 설정
|   |-- sessions/           # 예시 세션 로그 파일 (장문 가이드)
|
|-- mcp-configs/      # MCP 서버 설정
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, Railway 등
|
|-- plugins/          # 플러그인 생태계 문서
    |-- README.md           # 플러그인, 마켓플레이스, 스킬 가이드
```

---

## 빠른 시작

### 1. 필요한 것을 복사

```bash
# 저장소 클론
git clone https://github.com/affaan-m/everything-claude-code.git

# Claude 설정에 에이전트 복사
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 규칙 복사
cp everything-claude-code/rules/*.md ~/.claude/rules/

# 명령어 복사
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 스킬 복사
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. settings.json에 훅 추가

`hooks/hooks.json`의 훅을 `~/.claude/settings.json`에 복사하세요.

### 3. MCP 설정

`mcp-configs/mcp-servers.json`에서 원하는 MCP 서버를 `~/.claude.json`에 복사하세요.

**중요:** `YOUR_*_HERE` 플레이스홀더를 실제 API 키로 교체하세요.

### 4. 가이드 읽기

진심으로 가이드를 읽어주세요. 이 설정들은 컨텍스트가 있으면 10배 더 잘 이해됩니다.

1. **[축약 가이드](https://x.com/affaanmustafa/status/2012378465664745795)** - 설정과 기초
2. **[장문 가이드](https://x.com/affaanmustafa/status/2014040193557471352)** - 고급 기법 (토큰 최적화, 메모리 유지, 평가, 병렬화)

---

## 핵심 개념

### 에이전트

서브에이전트는 제한된 범위의 위임 작업을 처리합니다. 예시:

```markdown
---
name: code-reviewer
description: 코드의 품질, 보안, 유지보수성을 리뷰
tools: Read, Grep, Glob, Bash
model: opus
---

당신은 시니어 코드 리뷰어입니다...
```

### 스킬

스킬은 명령어나 에이전트가 호출하는 워크플로우 정의입니다:

```markdown
# TDD 워크플로우

1. 먼저 인터페이스 정의
2. 실패하는 테스트 작성 (RED)
3. 최소한의 코드 구현 (GREEN)
4. 리팩토링 (IMPROVE)
5. 80%+ 커버리지 확인
```

### 훅

훅은 툴 이벤트에서 실행됩니다. 예시 - console.log 경고:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \\\"\\.(ts|tsx|js|jsx)$\\\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### 규칙

규칙은 항상 따라야 할 지침입니다. 모듈식으로 유지하세요:

```
~/.claude/rules/
  security.md      # 하드코딩된 비밀번호 금지
  coding-style.md  # 불변성, 파일 제한
  testing.md       # TDD, 커버리지 요구사항
```

---

## 기여

**기여를 환영하며 장려합니다.**

이 저장소는 커뮤니티 리소스입니다. 다음이 있다면 기여해주세요:
- 유용한 에이전트나 스킬
- 영리한 훅
- 더 나은 MCP 설정
- 개선된 규칙

[CONTRIBUTING.md](CONTRIBUTING.md)에서 가이드라인을 확인하세요.

### 기여 아이디어

- 언어별 스킬 (Python, Go, Rust 패턴)
- 프레임워크별 설정 (Django, Rails, Laravel)
- DevOps 에이전트 (Kubernetes, Terraform, AWS)
- 테스트 전략 (다양한 프레임워크)
- 도메인별 지식 (ML, 데이터 엔지니어링, 모바일)

---

## 배경

저는 실험적 롤아웃부터 Claude Code를 사용해왔습니다. 2025년 9월 [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 구축하며 Anthropic x Forum Ventures 해커톤에서 우승했습니다 - 전적으로 Claude Code를 사용해서요.

이 설정들은 여러 프로덕션 애플리케이션에서 검증되었습니다.

---

## 중요 참고사항

### 컨텍스트 윈도우 관리

**중요:** 모든 MCP를 한 번에 활성화하지 마세요. 너무 많은 툴을 활성화하면 200k 컨텍스트 윈도우가 70k로 축소될 수 있습니다.

규칙:
- 20-30개의 MCP를 설정
- 프로젝트당 10개 미만 활성화
- 80개 미만의 툴 활성

프로젝트 설정에서 `disabledMcpServers`를 사용하여 사용하지 않는 MCP를 비활성화하세요.

### 사용자 정의

이 설정은 제 워크플로우에 맞춰져 있습니다. 다음을 수행하세요:
1. 마음에 드는 것으로 시작
2. 사용하는 스택에 맞게 수정
3. 사용하지 않는 것 제거
4. 자신만의 패턴 추가

---

## 링크

- **축약 가이드 (시작):** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **장문 가이드 (고급):** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **팔로우:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## 라이선스

MIT - 자유롭게 사용하고 필요에 맞게 수정하세요. 가능하면 기여해 주세요.

---

**이 저장소가 도움이 된다면 Star를 눌러주세요. 두 가이드를 읽고 멋진 것을 만드세요.**
