---
name: coding-standards
description: TypeScript, JavaScript, React, Node.js 개발을 위한 범용 코딩 표준, 모범 사례, 패턴.
---

# 코딩 표준 & 모범 사례

모든 프로젝트에 적용 가능한 범용 코딩 표준.

## 코드 품질 원칙

### 1. 가독성 우선
- 코드는 작성보다 더 많이 읽힘
- 명확한 변수 및 함수 이름
- 주석보다 자체 문서화 코드 선호
- 일관된 포맷팅

### 2. KISS (Keep It Simple, Stupid)
- 작동하는 가장 단순한 솔루션
- 과도한 엔지니어링 피하기
- 조기 최적화 없음
- 이해하기 쉬움 > 영리한 코드

### 3. DRY (Don't Repeat Yourself)
- 공통 로직을 함수로 추출
- 재사용 가능한 컴포넌트 생성
- 모듈 간 유틸리티 공유
- 복사-붙여넣기 프로그래밍 피하기

### 4. YAGNI (You Aren't Gonna Need It)
- 필요하기 전에 기능 빌드 안함
- 투기적 일반화 피하기
- 필요할 때만 복잡성 추가
- 단순하게 시작, 필요 시 리팩토링

## TypeScript/JavaScript 표준

### 변수 명명

```typescript
// ✅ 좋음: 설명적인 이름
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 나쁨: 불명확한 이름
const q = 'election'
const flag = true
const x = 1000
```

### 함수 명명

```typescript
// ✅ 좋음: 동사-명사 패턴
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 나쁨: 불명확하거나 명사만
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 불변성 패턴 (중요)

```typescript
// ✅ 항상 스프레드 연산자 사용
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 절대 직접 변이 안함
user.name = 'New Name'  // 나쁨
items.push(newItem)     // 나쁨
```

### 오류 처리

```typescript
// ✅ 좋음: 포괄적인 오류 처리
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch 실패:', error)
    throw new Error('데이터 가져오기 실패')
  }
}
```

### Async/Await 모범 사례

```typescript
// ✅ 좋음: 가능하면 병렬 실행
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 나쁨: 불필요하게 순차적
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 타입 안전성

```typescript
// ✅ 좋음: 적절한 타입
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 구현
}

// ❌ 나쁨: 'any' 사용
function getMarket(id: any): Promise<any> {
  // 구현
}
```

## React 모범 사례

### 컴포넌트 구조

```typescript
// ✅ 좋음: 타입이 있는 함수형 컴포넌트
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ 나쁨: 타입 없음, 불명확한 구조
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 상태 관리

```typescript
// ✅ 좋음: 적절한 상태 업데이트
const [count, setCount] = useState(0)

// 이전 상태 기반 상태에 함수형 업데이트
setCount(prev => prev + 1)

// ❌ 나쁨: 직접 상태 참조
setCount(count + 1)  // 비동기 시나리오에서 stale 될 수 있음
```

## API 설계 표준

### REST API 컨벤션

```
GET    /api/markets              # 모든 마켓 목록
GET    /api/markets/:id          # 특정 마켓 가져오기
POST   /api/markets              # 새 마켓 생성
PUT    /api/markets/:id          # 마켓 업데이트 (전체)
PATCH  /api/markets/:id          # 마켓 업데이트 (부분)
DELETE /api/markets/:id          # 마켓 삭제

# 필터링을 위한 쿼리 파라미터
GET /api/markets?status=active&limit=10&offset=0
```

### 응답 형식

```typescript
// ✅ 좋음: 일관된 응답 구조
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 성공 응답
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// 오류 응답
return NextResponse.json({
  success: false,
  error: '유효하지 않은 요청'
}, { status: 400 })
```

## 주석 & 문서화

### 언제 주석을 달 것인가

```typescript
// ✅ 좋음: WHAT이 아닌 WHY 설명
// 장애 시 API 과부하 방지를 위해 지수 백오프 사용
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 대규모 배열 성능을 위해 의도적으로 변이 사용
items.push(newItem)

// ❌ 나쁨: 명백한 것 진술
// 카운터를 1 증가
count++

// 이름을 사용자 이름으로 설정
name = user.name
```

## 코드 스멜 감지

다음 안티 패턴 주의:

### 1. 긴 함수
```typescript
// ❌ 나쁨: 50줄 넘는 함수
function processMarketData() {
  // 100줄의 코드
}

// ✅ 좋음: 작은 함수로 분리
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 깊은 중첩
```typescript
// ❌ 나쁨: 5+ 레벨 중첩
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 무언가 함
        }
      }
    }
  }
}

// ✅ 좋음: 조기 반환
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 무언가 함
```

### 3. 매직 넘버
```typescript
// ❌ 나쁨: 설명 없는 숫자
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 좋음: 명명된 상수
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**기억하세요**: 코드 품질은 협상 불가. 명확하고 유지보수 가능한 코드가 빠른 개발과 자신감 있는 리팩토링을 가능하게 합니다.
