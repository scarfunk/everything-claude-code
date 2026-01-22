# Eval 하네스 스킬

Claude Code 세션을 위한 공식 평가 프레임워크로, eval 주도 개발 (EDD) 원칙을 구현합니다.

## 철학

Eval 주도 개발은 eval을 "AI 개발의 유닛 테스트"로 취급합니다:
- 구현 전에 예상 동작 정의
- 개발 중 지속적으로 eval 실행
- 각 변경마다 회귀 추적
- 신뢰성 측정을 위해 pass@k 메트릭 사용

## Eval 유형

### 기능 Eval
Claude가 이전에 할 수 없던 것을 할 수 있는지 테스트:
```markdown
[기능 EVAL: feature-name]
작업: Claude가 달성해야 할 것에 대한 설명
성공 기준:
  - [ ] 기준 1
  - [ ] 기준 2
  - [ ] 기준 3
예상 출력: 예상 결과 설명
```

### 회귀 Eval
변경이 기존 기능을 깨뜨리지 않는지 확인:
```markdown
[회귀 EVAL: feature-name]
기준선: SHA 또는 체크포인트 이름
테스트:
  - existing-test-1: 통과/실패
  - existing-test-2: 통과/실패
  - existing-test-3: 통과/실패
결과: X/Y 통과 (이전 Y/Y)
```

## 채점자 유형

### 1. 코드 기반 채점자
코드를 사용한 결정론적 체크:
```bash
# 파일이 예상 패턴을 포함하는지 체크
grep -q "export function handleAuth" src/auth.ts && echo "통과" || echo "실패"

# 테스트가 통과하는지 체크
npm test -- --testPathPattern="auth" && echo "통과" || echo "실패"

# 빌드가 성공하는지 체크
npm run build && echo "통과" || echo "실패"
```

### 2. 모델 기반 채점자
개방형 출력 평가에 Claude 사용:
```markdown
[모델 채점자 프롬프트]
다음 코드 변경을 평가하세요:
1. 명시된 문제를 해결하는가?
2. 잘 구조화되어 있는가?
3. 엣지 케이스가 처리되는가?
4. 오류 처리가 적절한가?

점수: 1-5 (1=나쁨, 5=우수)
이유: [설명]
```

### 3. 인간 채점자
수동 리뷰용으로 플래그:
```markdown
[인간 리뷰 필요]
변경: 변경된 내용 설명
이유: 인간 리뷰가 필요한 이유
위험 수준: 낮음/중간/높음
```

## 메트릭

### pass@k
"k번 시도 중 최소 한 번 성공"
- pass@1: 첫 시도 성공률
- pass@3: 3번 시도 내 성공
- 일반적인 목표: pass@3 > 90%

### pass^k
"k번 시도 모두 성공"
- 더 높은 신뢰성 기준
- pass^3: 3번 연속 성공
- 중요 경로에 사용

## Eval 워크플로우

### 1. 정의 (코딩 전)
```markdown
## EVAL 정의: feature-xyz

### 기능 Eval
1. 새 사용자 계정 생성 가능
2. 이메일 형식 검증 가능
3. 비밀번호를 안전하게 해시 가능

### 회귀 Eval
1. 기존 로그인 여전히 작동
2. 세션 관리 변경 없음
3. 로그아웃 흐름 유지됨

### 성공 메트릭
- 기능 eval에 pass@3 > 90%
- 회귀 eval에 pass^3 = 100%
```

### 2. 구현
정의된 eval을 통과하기 위한 코드 작성.

### 3. 평가
```bash
# 기능 eval 실행
[각 기능 eval 실행, 통과/실패 기록]

# 회귀 eval 실행
npm test -- --testPathPattern="existing"

# 보고서 생성
```

### 4. 보고
```markdown
EVAL 보고서: feature-xyz
========================

기능 Eval:
  create-user:     통과 (pass@1)
  validate-email:  통과 (pass@2)
  hash-password:   통과 (pass@1)
  전체:            3/3 통과

회귀 Eval:
  login-flow:      통과
  session-mgmt:    통과
  logout-flow:     통과
  전체:            3/3 통과

메트릭:
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

상태: 리뷰 준비됨
```

## 통합 패턴

### 구현 전
```
/eval define feature-name
```
`.claude/evals/feature-name.md`에 eval 정의 파일 생성

### 구현 중
```
/eval check feature-name
```
현재 eval 실행하고 상태 보고

### 구현 후
```
/eval report feature-name
```
전체 eval 보고서 생성

## Eval 저장

프로젝트에 eval 저장:
```
.claude/
  evals/
    feature-xyz.md      # Eval 정의
    feature-xyz.log     # Eval 실행 히스토리
    baseline.json       # 회귀 기준선
```

## 모범 사례

1. **코딩 전 eval 정의** - 성공 기준에 대한 명확한 사고 강제
2. **자주 eval 실행** - 조기에 회귀 포착
3. **시간에 따른 pass@k 추적** - 신뢰성 트렌드 모니터링
4. **가능하면 코드 채점자 사용** - 결정론적 > 확률적
5. **보안은 인간 리뷰** - 보안 체크 완전 자동화하지 않음
6. **eval을 빠르게 유지** - 느린 eval은 실행되지 않음
7. **코드와 함께 eval 버전 관리** - Eval은 일급 아티팩트
