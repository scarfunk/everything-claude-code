# 체크포인트 명령어

워크플로우에서 체크포인트를 생성하거나 확인합니다.

## 사용법

`/checkpoint [create|verify|list] [이름]`

## 체크포인트 생성

체크포인트 생성 시:

1. `/verify quick` 실행하여 현재 상태가 깨끗한지 확인
2. 체크포인트 이름으로 git stash 또는 커밋 생성
3. 체크포인트를 `.claude/checkpoints.log`에 기록:

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. 체크포인트 생성됨 보고

## 체크포인트 확인

체크포인트 대비 확인 시:

1. 로그에서 체크포인트 읽기
2. 현재 상태와 체크포인트 비교:
   - 체크포인트 이후 추가된 파일
   - 체크포인트 이후 수정된 파일
   - 현재 vs 당시 테스트 통과율
   - 현재 vs 당시 커버리지

3. 보고:
```
체크포인트 비교: $NAME
============================
변경된 파일: X
테스트: +Y 통과 / -Z 실패
커버리지: +X% / -Y%
빌드: [통과/실패]
```

## 체크포인트 목록

다음과 함께 모든 체크포인트 표시:
- 이름
- 타임스탬프
- Git SHA
- 상태 (현재, 뒤처짐, 앞섬)

## 워크플로우

일반적인 체크포인트 흐름:

```
[시작] --> /checkpoint create "기능-시작"
   |
[구현] --> /checkpoint create "핵심-완료"
   |
[테스트] --> /checkpoint verify "핵심-완료"
   |
[리팩토링] --> /checkpoint create "리팩토링-완료"
   |
[PR] --> /checkpoint verify "기능-시작"
```

## 인수

$ARGUMENTS:
- `create <이름>` - 명명된 체크포인트 생성
- `verify <이름>` - 명명된 체크포인트 대비 확인
- `list` - 모든 체크포인트 표시
- `clear` - 오래된 체크포인트 제거 (최근 5개 유지)
