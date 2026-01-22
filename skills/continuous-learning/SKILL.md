---
name: continuous-learning
description: Claude Code 세션에서 재사용 가능한 패턴을 자동으로 추출하고 향후 사용을 위해 학습된 스킬로 저장합니다.
---

# 지속적 학습 스킬

각 세션 종료 시 Claude Code 세션을 자동으로 평가하여 학습된 스킬로 저장할 수 있는 재사용 가능한 패턴을 추출합니다.

## 작동 방식

이 스킬은 각 세션 종료 시 **Stop 훅**으로 실행됩니다:

1. **세션 평가**: 세션에 충분한 메시지가 있는지 확인 (기본값: 10+)
2. **패턴 감지**: 세션에서 추출 가능한 패턴 식별
3. **스킬 추출**: 유용한 패턴을 `~/.claude/skills/learned/`에 저장

## 설정

`config.json`을 편집하여 커스터마이즈:

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## 패턴 유형

| 패턴 | 설명 |
|------|------|
| `error_resolution` | 특정 오류 해결 방법 |
| `user_corrections` | 사용자 수정에서의 패턴 |
| `workarounds` | 프레임워크/라이브러리 특이점에 대한 해결책 |
| `debugging_techniques` | 효과적인 디버깅 접근법 |
| `project_specific` | 프로젝트별 규칙 |

## 훅 설정

`~/.claude/settings.json`에 추가:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## 왜 Stop 훅인가?

- **경량**: 세션 종료 시 한 번만 실행
- **비차단**: 모든 메시지에 지연 추가 안함
- **완전한 컨텍스트**: 전체 세션 트랜스크립트에 접근 가능

## 관련

- [Longform 가이드](https://x.com/affaanmustafa/status/2014040193557471352) - 지속적 학습 섹션
- `/learn` 명령어 - 세션 중간 수동 패턴 추출
