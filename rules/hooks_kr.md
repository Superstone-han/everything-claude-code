# Hooks 시스템

## Hook 유형

- **PreToolUse**: 도구 실행 전 (검증, 매개변수 수정)
- **PostToolUse**: 도구 실행 후 (자동 포맷, 체크)
- **Stop**: 세션 종료 시 (최종 검증)

## 현재 Hooks (~/.claude/settings.json)

### PreToolUse
- **tmux reminder**: 긴 실행 명령에 tmux 제안 (npm, pnpm, yarn, cargo 등)
- **git push review**: push 전에 Zed로 리뷰 열기
- **doc blocker**: 불필요한 .md/.txt 파일 생성 차단

### PostToolUse
- **PR creation**: PR URL과 GitHub Actions 상태 로깅
- **Prettier**: JS/TS 파일 편집 후 자동 포맷
- **TypeScript check**: .ts/.tsx 파일 편집 후 tsc 실행
- **console.log warning**: 편집된 파일의 console.log 경고

### Stop
- **console.log audit**: 세션 종료 전 변경된 모든 파일에서 console.log 확인

## 자동 승인 권한

주의해서 사용:
- 신뢰할 수 있고 명확한 계획에 대해 활성화
- 탐색 작업에는 비활성화
- dangerously-skip-permissions 플래그는 절대 사용하지 않기
- 대신 `~/.claude.json`의 `allowedTools`를 구성

## TodoWrite 모범 사례

TodoWrite 도구를 사용하여:
- 다단계 작업 진행 상황 추적
- 지침 이해 확인
- 실시간 조정 가능
- 세부 구현 단계 표시

Todo 목록이 드러내는 것:
- 순서가 잘못된 단계
- 누락된 항목
- 불필요한 추가 항목
- 잘못된 세분성
- 잘못 해석된 요구사항
