# 보안 가이드라인

## 필수 보안 체크

모든 커밋 전:
- [ ] 하드코딩된 비밀 없음 (API 키, 비밀번호, 토큰)
- [ ] 모든 사용자 입력 검증됨
- [ ] SQL 인젝션 방지 (파라미터화된 쿼리)
- [ ] XSS 방지 (살균된 HTML)
- [ ] CSRF 보호 활성화
- [ ] 인증/권한 확인됨
- [ ] 모든 엔드포인트에 속도 제한
- [ ] 오류 메시지가 민감한 데이터 노출 안함

## 비밀 관리

```typescript
// 절대 안됨: 하드코딩된 비밀
const apiKey = "sk-proj-xxxxx"

// 항상: 환경 변수
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 설정되지 않음')
}
```

## 보안 대응 프로토콜

보안 이슈 발견 시:
1. 즉시 중지
2. **security-reviewer** 에이전트 사용
3. 계속하기 전에 치명적 이슈 수정
4. 노출된 비밀 순환
5. 유사한 이슈를 위해 전체 코드베이스 검토
