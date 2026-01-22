# 코딩 스타일

## 불변성 (중요)

항상 새 객체를 생성하고, 절대 변이하지 마세요:

```javascript
// 잘못된: 변이
function updateUser(user, name) {
  user.name = name  // 변이!
  return user
}

// 올바른: 불변성
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
- 200-400줄 일반적, 800 최대
- 큰 컴포넌트에서 유틸리티 추출
- 유형별이 아닌 기능/도메인별 구성

## 오류 처리

항상 오류를 포괄적으로 처리:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('작업 실패:', error)
  throw new Error('상세한 사용자 친화적 메시지')
}
```

## 입력 검증

항상 사용자 입력 검증:

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
- [ ] 코드가 읽기 쉽고 이름이 잘 지어짐
- [ ] 함수가 작음 (<50줄)
- [ ] 파일이 집중됨 (<800줄)
- [ ] 깊은 중첩 없음 (>4레벨)
- [ ] 적절한 오류 처리
- [ ] console.log 문 없음
- [ ] 하드코딩된 값 없음
- [ ] 변이 없음 (불변 패턴 사용)
