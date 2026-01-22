# Codemaps 업데이트

코드베이스 구조를 분석하고 아키텍처 문서를 업데이트하세요:

1. 모든 소스 파일을 스캔하여 import, export, 종속성 확인
2. 다음 형식으로 토큰 린 codemaps 생성:
   - codemaps/architecture.md - 전체 아키텍처
   - codemaps/backend.md - 백엔드 구조
   - codemaps/frontend.md - 프론트엔드 구조
   - codemaps/data.md - 데이터 모델 및 스키마

3. 이전 버전과의 diff 백분율 계산
4. 변경사항이 30% 초과인 경우 업데이트 전에 사용자 승인 요청
5. 각 codemap에 신선도 타임스탬프 추가
6. .reports/codemap-diff.txt에 보고서 저장

분석에는 TypeScript/Node.js를 사용하세요. 구현 세부사항이 아닌 상위 수준 구조에 중점을 두세요.
