# 코딩 스타일

## 불변성 (중요)

항상 새 객체를 생성하고, 절대 변형하지 않기:

```javascript
// 잘못됨: 변형
function updateUser(user, name) {
  user.name = name  // 변형!
  return user
}

// 올바름: 불변성
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## 파일 구성

많은 작은 파일 > 적은 큰 파일:
- 높은 응집도, 낮은 결합도
- 일반적으로 200-400줄, 최대 800줄
- 큰 컴포넌트에서 유틸리티 추출
- 타입별이 아닌 기능/도메인별 구성

## 에러 처리

항상 포괄적으로 에러 처리:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('상세한 사용자 친화적 메시지')
}
```

## 입력 유효성 검사

항상 사용자 입력 유효성 검사:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## 코드 품질 체크리스트

작업 완료 표시 전:
- [ ] 코드가 읽기 쉽고 이름이 잘 지어졌는지
- [ ] 함수가 작은지 (<50줄)
- [ ] 파일이 집중되어 있는지 (<800줄)
- [ ] 깊은 중첩이 없는지 (>4단계)
- [ ] 적절한 에러 처리
- [ ] console.log 문 없음
- [ ] 하드코딩된 값 없음
- [ ] 변형 없음 (불변 패턴 사용)
