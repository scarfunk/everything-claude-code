# Orchestrate 명령어

복잡한 작업을 위한 순차적 에이전트 워크플로우.

## 사용법

`/orchestrate [워크플로우-유형] [작업-설명]`

## 워크플로우 유형

### feature
전체 기능 구현 워크플로우:
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
버그 조사 및 수정 워크플로우:
```
explorer -> tdd-guide -> code-reviewer
```

### refactor
안전한 리팩토링 워크플로우:
```
architect -> code-reviewer -> tdd-guide
```

### security
보안 중심 리뷰:
```
security-reviewer -> code-reviewer -> architect
```

## 실행 패턴

워크플로우의 각 에이전트에 대해:

1. 이전 에이전트의 컨텍스트로 **에이전트 호출**
2. 구조화된 핸드오프 문서로 **출력 수집**
3. 체인의 **다음 에이전트로 전달**
4. **결과 집계**하여 최종 보고서 작성

## 핸드오프 문서 형식

에이전트 간에 핸드오프 문서 생성:

```markdown
## 핸드오프: [이전-에이전트] -> [다음-에이전트]

### 컨텍스트
[수행한 작업 요약]

### 발견사항
[주요 발견 또는 결정]

### 수정된 파일
[수정된 파일 목록]

### 미해결 질문
[다음 에이전트를 위한 미해결 항목]

### 권장사항
[제안된 다음 단계]
```

## 예시: Feature 워크플로우

```
/orchestrate feature "사용자 인증 추가"
```

실행:

1. **Planner 에이전트**
   - 요구사항 분석
   - 구현 계획 생성
   - 의존성 식별
   - 출력: `핸드오프: planner -> tdd-guide`

2. **TDD Guide 에이전트**
   - planner 핸드오프 읽기
   - 테스트 먼저 작성
   - 테스트 통과하도록 구현
   - 출력: `핸드오프: tdd-guide -> code-reviewer`

3. **Code Reviewer 에이전트**
   - 구현 리뷰
   - 이슈 체크
   - 개선 제안
   - 출력: `핸드오프: code-reviewer -> security-reviewer`

4. **Security Reviewer 에이전트**
   - 보안 감사
   - 취약점 체크
   - 최종 승인
   - 출력: 최종 보고서

## 최종 보고서 형식

```
오케스트레이션 보고서
====================
워크플로우: feature
작업: 사용자 인증 추가
에이전트: planner -> tdd-guide -> code-reviewer -> security-reviewer

요약
-------
[한 단락 요약]

에이전트 출력
-------------
Planner: [요약]
TDD Guide: [요약]
Code Reviewer: [요약]
Security Reviewer: [요약]

변경된 파일
-------------
[모든 수정된 파일 목록]

테스트 결과
------------
[테스트 통과/실패 요약]

보안 상태
---------------
[보안 발견사항]

권장사항
--------------
[출시 / 추가 작업 필요 / 차단됨]
```

## 병렬 실행

독립적인 체크의 경우 에이전트를 병렬로 실행:

```markdown
### 병렬 단계
동시 실행:
- code-reviewer (품질)
- security-reviewer (보안)
- architect (설계)

### 결과 병합
출력을 단일 보고서로 결합
```

## 인수

$ARGUMENTS:
- `feature <설명>` - 전체 기능 워크플로우
- `bugfix <설명>` - 버그 수정 워크플로우
- `refactor <설명>` - 리팩토링 워크플로우
- `security <설명>` - 보안 리뷰 워크플로우
- `custom <에이전트들> <설명>` - 커스텀 에이전트 시퀀스

## 커스텀 워크플로우 예시

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "캐싱 레이어 재설계"
```

## 팁

1. 복잡한 기능은 **planner로 시작**
2. 머지 전에는 **항상 code-reviewer 포함**
3. 인증/결제/PII에는 **security-reviewer 사용**
4. **핸드오프를 간결하게** 유지 - 다음 에이전트가 필요한 것에 집중
5. 필요시 에이전트 간 **검증 실행**
