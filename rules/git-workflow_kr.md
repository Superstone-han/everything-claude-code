# Git 워크플로우

## 커밋 메시지 형식

```
<type>: <description>

<optional body>
```

타입: feat, fix, refactor, docs, test, chore, perf, ci

참고: ~/.claude/settings.json을 통해 전역으로 attribution이 비활성화되었습니다.

## Pull Request 워크플로우

PR을 생성할 때:
1. 전체 커밋 기록 분석 (최신 커밋만 아님)
2. `git diff [base-branch]...HEAD`를 사용하여 모든 변경사항 보기
3. 포괄적인 PR 요약 작성
4. TODO가 포함된 테스트 계획 포함
5. 새 브랜치인 경우 `-u` 플래그로 푸시

## 기능 구현 워크플로우

1. **먼저 계획**
   - **planner** agent 사용하여 구현 계획 생성
   - 종속성 및 위험 식별
   - 단계로 분해

2. **TDD 접근**
   - **tdd-guide** agent 사용
   - 테스트 먼저 작성 (RED)
   - 통과할 구현 (GREEN)
   - 리팩토링 (IMPROVE)
   - 80% 이상 커버리지 확인

3. **코드 검토**
   - 코드 작성 직후 **code-reviewer** agent 사용
   - 중요 및 높은 문제 처리
   - 가능한 중간 문제 수정

4. **커밋 및 푸시**
   - 상세한 커밋 메시지
   - 관례적 커밋 형식 준수
