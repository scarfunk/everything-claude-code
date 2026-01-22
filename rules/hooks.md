# 훅 시스템

## 훅 유형

- **PreToolUse**: 도구 실행 전 (검증, 파라미터 수정)
- **PostToolUse**: 도구 실행 후 (자동 포맷, 체크)
- **Stop**: 세션 종료 시 (최종 검증)

## 현재 훅 (~/.claude/settings.json에 있음)

### PreToolUse
- **tmux 알림**: 장시간 실행 명령어에 tmux 제안 (npm, pnpm, yarn, cargo 등)
- **git push 리뷰**: push 전 Zed로 리뷰 열기
- **문서 차단**: 불필요한 .md/.txt 파일 생성 차단

### PostToolUse
- **PR 생성**: PR URL 및 GitHub Actions 상태 기록
- **Prettier**: 편집 후 JS/TS 파일 자동 포맷
- **TypeScript 체크**: .ts/.tsx 파일 편집 후 tsc 실행
- **console.log 경고**: 편집된 파일의 console.log 경고

### Stop
- **console.log 감사**: 세션 종료 전 모든 수정된 파일에서 console.log 체크

## 자동 수락 권한

주의하여 사용:
- 신뢰할 수 있고 잘 정의된 계획에 활성화
- 탐색적 작업에는 비활성화
- dangerously-skip-permissions 플래그 절대 사용 금지
- 대신 `~/.claude.json`에서 `allowedTools` 설정

## TodoWrite 모범 사례

TodoWrite 도구 사용 목적:
- 멀티스텝 작업의 진행 상황 추적
- 지침 이해 확인
- 실시간 조정 활성화
- 세부 구현 단계 표시

Todo 목록이 보여주는 것:
- 순서가 틀린 단계
- 누락된 항목
- 불필요한 추가 항목
- 잘못된 세분성
- 잘못 해석된 요구사항
