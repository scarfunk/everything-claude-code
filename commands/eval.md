# Eval 명령어

평가 기반 개발 워크플로우를 관리합니다.

## 사용법

`/eval [define|check|report|list] [기능-이름]`

## Eval 정의

`/eval define 기능-이름`

새 eval 정의 생성:

1. 템플릿으로 `.claude/evals/기능-이름.md` 생성:

```markdown
## EVAL: 기능-이름
생성일: $(date)

### 기능 Eval
- [ ] [기능 1 설명]
- [ ] [기능 2 설명]

### 회귀 Eval
- [ ] [기존 동작 1이 여전히 작동]
- [ ] [기존 동작 2가 여전히 작동]

### 성공 기준
- 기능 eval에 대해 pass@3 > 90%
- 회귀 eval에 대해 pass^3 = 100%
```

2. 사용자에게 구체적 기준 작성 요청

## Eval 체크

`/eval check 기능-이름`

기능에 대한 eval 실행:

1. `.claude/evals/기능-이름.md`에서 eval 정의 읽기
2. 각 기능 eval에 대해:
   - 기준 확인 시도
   - 통과/실패 기록
   - 시도를 `.claude/evals/기능-이름.log`에 기록
3. 각 회귀 eval에 대해:
   - 관련 테스트 실행
   - 기준선과 비교
   - 통과/실패 기록
4. 현재 상태 보고:

```
EVAL 체크: 기능-이름
========================
기능: X/Y 통과
회귀: X/Y 통과
상태: 진행 중 / 준비됨
```

## Eval 보고서

`/eval report 기능-이름`

포괄적인 eval 보고서 생성:

```
EVAL 보고서: 기능-이름
=========================
생성일: $(date)

기능 EVAL
----------------
[eval-1]: 통과 (pass@1)
[eval-2]: 통과 (pass@2) - 재시도 필요
[eval-3]: 실패 - 노트 참조

회귀 EVAL
----------------
[test-1]: 통과
[test-2]: 통과
[test-3]: 통과

메트릭
-------
기능 pass@1: 67%
기능 pass@3: 100%
회귀 pass^3: 100%

노트
-----
[모든 이슈, 엣지 케이스, 또는 관찰 사항]

권장사항
--------------
[출시 / 추가 작업 필요 / 차단됨]
```

## Eval 목록

`/eval list`

모든 eval 정의 표시:

```
EVAL 정의
================
feature-auth      [3/5 통과] 진행 중
feature-search    [5/5 통과] 준비됨
feature-export    [0/4 통과] 시작 안됨
```

## 인수

$ARGUMENTS:
- `define <이름>` - 새 eval 정의 생성
- `check <이름>` - eval 실행 및 체크
- `report <이름>` - 전체 보고서 생성
- `list` - 모든 eval 표시
- `clean` - 오래된 eval 로그 제거 (최근 10회 실행 유지)
