---
description: 테스트 주도 개발 워크플로우를 강제합니다. 인터페이스를 스캐폴드하고, 테스트를 먼저 생성한 다음, 통과를 위한 최소 코드를 구현합니다. 80%+ 커버리지를 보장합니다.
---

# TDD 명령어

이 명령어는 **tdd-guide** 에이전트를 호출하여 테스트 주도 개발 방법론을 강제합니다.

## 이 명령어가 하는 일

1. **인터페이스 스캐폴드** - 먼저 타입/인터페이스 정의
2. **테스트 먼저 생성** - 실패하는 테스트 작성 (RED)
3. **최소 코드 구현** - 통과하기에 충분한 코드만 작성 (GREEN)
4. **리팩토링** - 테스트를 녹색으로 유지하며 코드 개선 (REFACTOR)
5. **커버리지 확인** - 80%+ 테스트 커버리지 보장

## 사용 시점

다음 경우에 `/tdd` 사용:
- 새 기능 구현
- 새 함수/컴포넌트 추가
- 버그 수정 (먼저 버그를 재현하는 테스트 작성)
- 기존 코드 리팩토링
- 중요 비즈니스 로직 빌드

## 작동 방식

tdd-guide 에이전트는 다음을 수행합니다:

1. 입력/출력용 **인터페이스 정의**
2. (코드가 아직 없어서) **실패할 테스트 작성**
3. **테스트 실행** 및 올바른 이유로 실패하는지 확인
4. 테스트 통과를 위한 **최소 구현 작성**
5. **테스트 실행** 및 통과 확인
6. 테스트를 녹색으로 유지하며 코드 **리팩토링**
7. **커버리지 체크** 및 80% 미만이면 테스트 추가

## TDD 사이클

```
RED → GREEN → REFACTOR → REPEAT

RED:      실패하는 테스트 작성
GREEN:    통과를 위한 최소 코드 작성
REFACTOR: 코드 개선, 테스트 통과 유지
REPEAT:   다음 기능/시나리오
```

## 사용 예시

```
사용자: /tdd 마켓 유동성 점수를 계산하는 함수 필요

에이전트 (tdd-guide):
# TDD 세션: 마켓 유동성 점수 계산기

## 1단계: 인터페이스 정의 (스캐폴드)

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: 구현
  throw new Error('구현되지 않음')
}
```

## 2단계: 실패하는 테스트 작성 (RED)

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('유동성 높은 마켓에 대해 높은 점수 반환', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('유동성 낮은 마켓에 대해 낮은 점수 반환', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1일 전
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('엣지 케이스 처리: 볼륨 0', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## 3단계: 테스트 실행 - 실패 확인

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ 유동성 높은 마켓에 대해 높은 점수 반환 (2 ms)
    Error: 구현되지 않음

1개 테스트 실패, 0개 통과
```

✅ 예상대로 테스트 실패. 구현 준비됨.

## 4단계: 최소 코드 구현 (GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // 볼륨 0 엣지 케이스 처리
  if (market.totalVolume === 0) {
    return 0
  }

  // 컴포넌트 점수 계산 (0-100 스케일)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // 최근 활동 보너스
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // 가중 평균
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // 0-100으로 클램프
}
```

## 5단계: 테스트 실행 - 통과 확인

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 유동성 높은 마켓에 대해 높은 점수 반환 (3 ms)
  ✓ 유동성 낮은 마켓에 대해 낮은 점수 반환 (2 ms)
  ✓ 엣지 케이스 처리: 볼륨 0 (1 ms)

3개 테스트 통과
```

✅ 모든 테스트 통과!

## 6단계: 리팩토링 (개선)

```typescript
// lib/liquidity.ts - 상수와 가독성 개선으로 리팩토링
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## 7단계: 테스트 여전히 통과 확인

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 유동성 높은 마켓에 대해 높은 점수 반환 (3 ms)
  ✓ 유동성 낮은 마켓에 대해 낮은 점수 반환 (2 ms)
  ✓ 엣지 케이스 처리: 볼륨 0 (1 ms)

3개 테스트 통과
```

✅ 리팩토링 완료, 테스트 여전히 통과!

## 8단계: 커버리지 체크

```bash
npm test -- --coverage lib/liquidity.test.ts

파일            | % Stmts | % Branch | % Funcs | % Lines
----------------|---------|----------|---------|--------
liquidity.ts    |   100   |   100    |   100   |   100

커버리지: 100% ✅ (목표: 80%)
```

✅ TDD 세션 완료!
```

## TDD 모범 사례

**권장:**
- ✅ 구현 전에 테스트를 먼저 작성
- ✅ 테스트 실행하고 구현 전 실패 확인
- ✅ 테스트 통과를 위한 최소 코드 작성
- ✅ 테스트가 녹색일 때만 리팩토링
- ✅ 엣지 케이스 및 오류 시나리오 추가
- ✅ 80%+ 커버리지 목표 (중요 코드는 100%)

**비권장:**
- ❌ 테스트 전에 구현 작성
- ❌ 변경 후 테스트 실행 건너뛰기
- ❌ 한 번에 너무 많은 코드 작성
- ❌ 실패하는 테스트 무시
- ❌ 구현 세부사항 테스트 (동작 테스트)
- ❌ 모든 것 모킹 (통합 테스트 선호)

## 포함할 테스트 유형

**유닛 테스트** (함수 수준):
- 정상 경로 시나리오
- 엣지 케이스 (빈 값, null, 최대값)
- 오류 조건
- 경계값

**통합 테스트** (컴포넌트 수준):
- API 엔드포인트
- 데이터베이스 작업
- 외부 서비스 호출
- 훅을 사용하는 React 컴포넌트

**E2E 테스트** (`/e2e` 명령어 사용):
- 중요 사용자 흐름
- 멀티스텝 프로세스
- 풀스택 통합

## 커버리지 요구사항

- **80% 최소** 모든 코드
- **100% 필수** 다음에 대해:
  - 금융 계산
  - 인증 로직
  - 보안에 중요한 코드
  - 핵심 비즈니스 로직

## 중요 참고사항

**필수**: 테스트는 구현 전에 작성되어야 합니다. TDD 사이클은:

1. **RED** - 실패하는 테스트 작성
2. **GREEN** - 통과하도록 구현
3. **REFACTOR** - 코드 개선

RED 단계를 절대 건너뛰지 마세요. 테스트 전에 코드를 작성하지 마세요.

## 다른 명령어와의 통합

- 무엇을 빌드할지 이해하려면 먼저 `/plan` 사용
- 테스트와 함께 구현하려면 `/tdd` 사용
- 빌드 오류 발생 시 `/build-and-fix` 사용
- 구현 리뷰에 `/code-review` 사용
- 커버리지 확인에 `/test-coverage` 사용

## 관련 에이전트

이 명령어는 다음 위치의 `tdd-guide` 에이전트를 호출합니다:
`~/.claude/agents/tdd-guide.md`

그리고 다음의 `tdd-workflow` 스킬을 참조할 수 있습니다:
`~/.claude/skills/tdd-workflow/`
