# 보안 지침

## 필수 보안 점검

모든 커밋 전에:
- [ ] 하드코딩된 비밀키 없음 (API 키, 비밀번호, 토큰)
- [ ] 모든 사용자 입력 검증
- [ ] SQL 인젝션 방지 (매개변수화된 쿼리)
- [ ] XSS 방지 (HTML 정화)
- [ ] CSRF 보호 활성화
- [ ] 인증/인가 확인
- [ ] 모든 엔드포인트에 속도 제한
- [ ] 오류 메시지에 민감한 데이터 노출 금지

## 비밀키 관리

```typescript
// 절대: 하드코딩된 비밀키
const apiKey = "sk-proj-xxxxx"

// 항상: 환경 변수
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## 보안 대응 프로토콜

보안 문제를 발견하면:
1. 즉시 중지
2. **security-reviewer** 에이전트 사용
3. CRITICAL 문제를 먼저 수정
4. 노출된 비밀키 교체
5. 유사한 문제를 코드베이스 전체에서 검토
