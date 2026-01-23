# 보안 가이드라인

## 필수 보안 검사

모든 commit 전:
- [ ] 하드코딩된 secret 없음 (API key, password, token)
- [ ] 모든 사용자 입력 유효성 검사됨
- [ ] SQL injection 방지 (parameterized query)
- [ ] XSS 방지 (HTML sanitize)
- [ ] CSRF 보호 활성화
- [ ] 인증/인가 검증됨
- [ ] 모든 endpoint에 rate limiting
- [ ] 에러 메시지가 민감한 데이터 노출하지 않음

## Secret 관리

```typescript
// 절대 안됨: 하드코딩된 secret
const apiKey = "sk-proj-xxxxx"

// 항상: 환경 변수
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 설정되지 않았습니다')
}
```

## 보안 대응 프로토콜

보안 이슈 발견 시:
1. 즉시 중단
2. **security-reviewer** agent 사용
3. CRITICAL 이슈 해결 전 계속하지 않기
4. 노출된 secret 교체
5. 유사한 이슈에 대해 전체 codebase 리뷰
