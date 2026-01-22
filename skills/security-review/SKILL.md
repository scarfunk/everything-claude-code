---
name: security-review
description: 인증 추가, 사용자 입력 처리, 비밀 작업, API 엔드포인트 생성, 결제/민감한 기능 구현 시 이 스킬을 사용하세요. 포괄적인 보안 체크리스트와 패턴을 제공합니다.
---

# 보안 리뷰 스킬

이 스킬은 모든 코드가 보안 모범 사례를 따르고 잠재적 취약점을 식별하도록 보장합니다.

## 활성화 시점

- 인증 또는 권한 구현 시
- 사용자 입력 또는 파일 업로드 처리 시
- 새 API 엔드포인트 생성 시
- 비밀 또는 자격 증명 작업 시
- 결제 기능 구현 시
- 민감한 데이터 저장 또는 전송 시
- 타사 API 통합 시

## 보안 체크리스트

### 1. 비밀 관리

#### ❌ 절대 하지 말 것
```typescript
const apiKey = "sk-proj-xxxxx"  // 하드코딩된 비밀
const dbPassword = "password123" // 소스 코드에
```

#### ✅ 항상 할 것
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 비밀이 존재하는지 확인
if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 설정되지 않음')
}
```

#### 확인 단계
- [ ] 하드코딩된 API 키, 토큰, 비밀번호 없음
- [ ] 모든 비밀이 환경 변수에 있음
- [ ] `.env.local`이 .gitignore에 있음
- [ ] git 히스토리에 비밀 없음
- [ ] 프로덕션 비밀이 호스팅 플랫폼에 있음 (Vercel, Railway)

### 2. 입력 검증

#### 항상 사용자 입력 검증
```typescript
import { z } from 'zod'

// 검증 스키마 정의
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 처리 전 검증
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### 파일 업로드 검증
```typescript
function validateFileUpload(file: File) {
  // 크기 체크 (최대 5MB)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('파일이 너무 큼 (최대 5MB)')
  }

  // 타입 체크
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('유효하지 않은 파일 타입')
  }

  // 확장자 체크
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('유효하지 않은 파일 확장자')
  }

  return true
}
```

### 3. SQL 인젝션 방지

#### ❌ 절대 SQL을 연결하지 마세요
```typescript
// 위험 - SQL 인젝션 취약점
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 항상 파라미터화된 쿼리 사용
```typescript
// 안전 - 파라미터화된 쿼리
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 또는 raw SQL로
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

### 4. 인증 & 권한

#### JWT 토큰 처리
```typescript
// ❌ 잘못된: localStorage (XSS에 취약)
localStorage.setItem('token', token)

// ✅ 올바른: httpOnly 쿠키
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 권한 체크
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 항상 먼저 권한 확인
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: '권한 없음' },
      { status: 403 }
    )
  }

  // 삭제 진행
  await db.users.delete({ where: { id: userId } })
}
```

#### 행 수준 보안 (Supabase)
```sql
-- 모든 테이블에 RLS 활성화
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 사용자는 자신의 데이터만 볼 수 있음
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- 사용자는 자신의 데이터만 업데이트할 수 있음
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

### 5. XSS 방지

#### HTML 살균
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 항상 사용자 제공 HTML 살균
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

### 6. 속도 제한

#### API 속도 제한
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 윈도우당 100개 요청
  message: '요청이 너무 많음'
})

// 라우트에 적용
app.use('/api/', limiter)
```

### 7. 민감한 데이터 노출

#### 로깅
```typescript
// ❌ 잘못된: 민감한 데이터 로깅
console.log('사용자 로그인:', { email, password })
console.log('결제:', { cardNumber, cvv })

// ✅ 올바른: 민감한 데이터 수정
console.log('사용자 로그인:', { email, userId })
console.log('결제:', { last4: card.last4, userId })
```

#### 오류 메시지
```typescript
// ❌ 잘못된: 내부 세부사항 노출
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 올바른: 일반적인 오류 메시지
catch (error) {
  console.error('내부 오류:', error)
  return NextResponse.json(
    { error: '오류가 발생했습니다. 다시 시도해주세요.' },
    { status: 500 }
  )
}
```

## 배포 전 보안 체크리스트

모든 프로덕션 배포 전:

- [ ] **비밀**: 하드코딩된 비밀 없음, 모두 환경 변수에
- [ ] **입력 검증**: 모든 사용자 입력 검증됨
- [ ] **SQL 인젝션**: 모든 쿼리 파라미터화됨
- [ ] **XSS**: 사용자 콘텐츠 살균됨
- [ ] **CSRF**: 보호 활성화됨
- [ ] **인증**: 적절한 토큰 처리
- [ ] **권한**: 역할 체크 구현됨
- [ ] **속도 제한**: 모든 엔드포인트에 활성화됨
- [ ] **HTTPS**: 프로덕션에서 강제됨
- [ ] **보안 헤더**: CSP, X-Frame-Options 설정됨
- [ ] **오류 처리**: 오류에 민감한 데이터 없음
- [ ] **로깅**: 민감한 데이터 로깅 안됨
- [ ] **의존성**: 최신, 취약점 없음
- [ ] **행 수준 보안**: Supabase에서 활성화됨
- [ ] **CORS**: 적절히 설정됨
- [ ] **파일 업로드**: 검증됨 (크기, 타입)

---

**기억하세요**: 보안은 선택사항이 아닙니다. 하나의 취약점이 전체 플랫폼을 손상시킬 수 있습니다. 확신이 없으면 조심하는 쪽으로 하세요.
