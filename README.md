# Everything Claude Code

**Anthropic 해커톤 수상자가 제공하는 완전한 Claude Code 설정 컬렉션.**

실제 제품을 구축하며 10개월 이상의 집중적인 일일 사용을 통해 발전시킨 프로덕션 준비 에이전트, 스킬, 훅, 명령어, 규칙, MCP 설정.

---

## 가이드

이 저장소는 원시 코드만 포함합니다. 가이드가 모든 것을 설명합니다.

### 여기서 시작: 간략 가이드

<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />

**[Everything Claude Code 간략 가이드](https://x.com/affaanmustafa/status/2012378465664745795)**

기초 - 각 설정 유형이 무엇을 하는지, 설정 구조 방법, 컨텍스트 윈도우 관리, 이 설정들의 철학. **이것을 먼저 읽으세요.**

---

### 다음: 상세 가이드

<img width="609" height="428" alt="image" src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" />

**[Everything Claude Code 상세 가이드](https://x.com/affaanmustafa/status/2014040193557471352)**

고급 기술 - 토큰 최적화, 세션 간 메모리 지속성, 검증 루프 & eval, 병렬화 전략, 서브에이전트 오케스트레이션, 지속적 학습. 이 가이드의 모든 내용에 이 저장소에 작동하는 코드가 있습니다.

| 주제 | 배울 내용 |
|------|----------|
| 토큰 최적화 | 모델 선택, 시스템 프롬프트 슬리밍, 백그라운드 프로세스 |
| 메모리 지속성 | 세션 간 컨텍스트를 자동으로 저장/로드하는 훅 |
| 지속적 학습 | 세션에서 패턴을 재사용 가능한 스킬로 자동 추출 |
| 검증 루프 | 체크포인트 vs 지속적 eval, 채점자 유형, pass@k 메트릭 |
| 병렬화 | Git 워크트리, 캐스케이드 방식, 인스턴스 확장 시점 |
| 서브에이전트 오케스트레이션 | 컨텍스트 문제, 반복적 검색 패턴 |


---

## 내용물

이 저장소는 **Claude Code 플러그인**입니다 - 직접 설치하거나 구성 요소를 수동으로 복사하세요.

```
everything-claude-code/
|-- .claude-plugin/   # 플러그인 및 마켓플레이스 매니페스트
|   |-- plugin.json         # 플러그인 메타데이터 및 컴포넌트 경로
|   |-- marketplace.json    # /plugin marketplace add를 위한 마켓플레이스 카탈로그
|
|-- agents/           # 위임을 위한 전문 서브에이전트
|   |-- planner.md           # 기능 구현 계획
|   |-- architect.md         # 시스템 설계 결정
|   |-- tdd-guide.md         # 테스트 주도 개발
|   |-- code-reviewer.md     # 품질 및 보안 리뷰
|   |-- security-reviewer.md # 취약점 분석
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 테스트
|   |-- refactor-cleaner.md  # 데드 코드 정리
|   |-- doc-updater.md       # 문서 동기화
|
|-- skills/           # 워크플로우 정의 및 도메인 지식
|   |-- coding-standards/           # 언어 모범 사례
|   |-- backend-patterns/           # API, 데이터베이스, 캐싱 패턴
|   |-- frontend-patterns/          # React, Next.js 패턴
|   |-- continuous-learning/        # 세션에서 패턴 자동 추출 (상세 가이드)
|   |-- strategic-compact/          # 수동 압축 제안 (상세 가이드)
|   |-- tdd-workflow/               # TDD 방법론
|   |-- security-review/            # 보안 체크리스트
|   |-- eval-harness/               # 검증 루프 평가 (상세 가이드)
|   |-- verification-loop/          # 지속적 검증 (상세 가이드)
|
|-- commands/         # 빠른 실행을 위한 슬래시 명령어
|   |-- tdd.md              # /tdd - 테스트 주도 개발
|   |-- plan.md             # /plan - 구현 계획
|   |-- e2e.md              # /e2e - E2E 테스트 생성
|   |-- code-review.md      # /code-review - 품질 리뷰
|   |-- build-fix.md        # /build-fix - 빌드 오류 수정
|   |-- refactor-clean.md   # /refactor-clean - 데드 코드 제거
|   |-- learn.md            # /learn - 세션 중 패턴 추출 (상세 가이드)
|   |-- checkpoint.md       # /checkpoint - 검증 상태 저장 (상세 가이드)
|   |-- verify.md           # /verify - 검증 루프 실행 (상세 가이드)
|
|-- rules/            # 항상 따라야 할 가이드라인 (~/.claude/rules/에 복사)
|   |-- security.md         # 필수 보안 체크
|   |-- coding-style.md     # 불변성, 파일 구성
|   |-- testing.md          # TDD, 80% 커버리지 요구사항
|   |-- git-workflow.md     # 커밋 형식, PR 프로세스
|   |-- agents.md           # 서브에이전트에 위임 시점
|   |-- performance.md      # 모델 선택, 컨텍스트 관리
|
|-- hooks/            # 트리거 기반 자동화
|   |-- hooks.json                # 모든 훅 설정 (PreToolUse, PostToolUse, Stop 등)
|   |-- memory-persistence/       # 세션 라이프사이클 훅 (상세 가이드)
|   |-- strategic-compact/        # 압축 제안 (상세 가이드)
|
|-- contexts/         # 동적 시스템 프롬프트 주입 컨텍스트 (상세 가이드)
|   |-- dev.md              # 개발 모드 컨텍스트
|   |-- review.md           # 코드 리뷰 모드 컨텍스트
|   |-- research.md         # 연구/탐색 모드 컨텍스트
|
|-- examples/         # 예시 설정 및 세션
|   |-- CLAUDE.md           # 프로젝트 레벨 설정 예시
|   |-- user-CLAUDE.md      # 사용자 레벨 설정 예시
|
|-- mcp-configs/      # MCP 서버 설정
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, Railway 등
|
|-- marketplace.json  # 셀프 호스팅 마켓플레이스 설정 (/plugin marketplace add용)
```

---

## 설치

### 옵션 1: 플러그인으로 설치 (권장)

이 저장소를 사용하는 가장 쉬운 방법 - Claude Code 플러그인으로 설치:

```bash
# 이 저장소를 마켓플레이스로 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

또는 `~/.claude/settings.json`에 직접 추가:

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

이렇게 하면 모든 명령어, 에이전트, 스킬, 훅에 즉시 접근할 수 있습니다.

---

### 옵션 2: 수동 설치

설치 항목을 수동으로 제어하려면:

```bash
# 저장소 클론
git clone https://github.com/affaan-m/everything-claude-code.git

# 에이전트를 Claude 설정에 복사
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 규칙 복사
cp everything-claude-code/rules/*.md ~/.claude/rules/

# 명령어 복사
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 스킬 복사
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

#### settings.json에 훅 추가

`hooks/hooks.json`의 훅을 `~/.claude/settings.json`에 복사하세요.

#### MCP 설정

`mcp-configs/mcp-servers.json`에서 원하는 MCP 서버를 `~/.claude.json`에 복사하세요.

**중요:** `YOUR_*_HERE` 플레이스홀더를 실제 API 키로 교체하세요.

---

### 가이드 읽기

진지하게, 가이드를 읽으세요. 이 설정들은 컨텍스트와 함께 10배 더 이해됩니다.

1. **[간략 가이드](https://x.com/affaanmustafa/status/2012378465664745795)** - 설정 및 기초
2. **[상세 가이드](https://x.com/affaanmustafa/status/2014040193557471352)** - 고급 기술 (토큰 최적화, 메모리 지속성, eval, 병렬화)

---

## 핵심 개념

### 에이전트

서브에이전트는 제한된 범위로 위임된 작업을 처리합니다. 예시:

```markdown
---
name: code-reviewer
description: 코드를 품질, 보안, 유지보수성을 위해 리뷰합니다
tools: Read, Grep, Glob, Bash
model: opus
---

당신은 시니어 코드 리뷰어입니다...
```

### 스킬

스킬은 명령어나 에이전트에 의해 호출되는 워크플로우 정의입니다:

```markdown
# TDD 워크플로우

1. 먼저 인터페이스 정의
2. 실패하는 테스트 작성 (RED)
3. 최소 코드 구현 (GREEN)
4. 리팩토링 (IMPROVE)
5. 80%+ 커버리지 확인
```

### 훅

훅은 도구 이벤트에서 발생합니다. 예시 - console.log 경고:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] console.log 제거' >&2"
  }]
}
```

### 규칙

규칙은 항상 따라야 할 가이드라인입니다. 모듈화하세요:

```
~/.claude/rules/
  security.md      # 하드코딩된 비밀 없음
  coding-style.md  # 불변성, 파일 제한
  testing.md       # TDD, 커버리지 요구사항
```

---

## 기여

**기여를 환영하고 권장합니다.**

이 저장소는 커뮤니티 리소스입니다. 다음이 있다면:
- 유용한 에이전트나 스킬
- 영리한 훅
- 더 나은 MCP 설정
- 개선된 규칙

기여해 주세요! 가이드라인은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.

### 기여 아이디어

- 언어별 스킬 (Python, Go, Rust 패턴)
- 프레임워크별 설정 (Django, Rails, Laravel)
- DevOps 에이전트 (Kubernetes, Terraform, AWS)
- 테스트 전략 (다양한 프레임워크)
- 도메인별 지식 (ML, 데이터 엔지니어링, 모바일)

---

## 배경

저는 실험적 롤아웃 때부터 Claude Code를 사용해 왔습니다. 2025년 9월 Anthropic x Forum Ventures 해커톤에서 [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 구축하여 수상했습니다 - 전적으로 Claude Code를 사용하여.

이 설정들은 여러 프로덕션 애플리케이션에서 실전 테스트되었습니다.

---

## 중요 참고사항

### 컨텍스트 윈도우 관리

**중요:** 모든 MCP를 한 번에 활성화하지 마세요. 너무 많은 도구가 활성화되면 200k 컨텍스트 윈도우가 70k로 줄어들 수 있습니다.

경험 법칙:
- 20-30개 MCP 설정
- 프로젝트당 10개 미만 활성화
- 80개 미만 도구 활성화

사용하지 않는 것을 비활성화하려면 프로젝트 설정에서 `disabledMcpServers`를 사용하세요.

### 커스터마이즈

이 설정은 제 워크플로우에 맞습니다. 당신은:
1. 공감하는 것부터 시작
2. 스택에 맞게 수정
3. 사용하지 않는 것 제거
4. 자신의 패턴 추가

---

## 링크

- **간략 가이드 (여기서 시작):** [Everything Claude Code 간략 가이드](https://x.com/affaanmustafa/status/2012378465664745795)
- **상세 가이드 (고급):** [Everything Claude Code 상세 가이드](https://x.com/affaanmustafa/status/2014040193557471352)
- **팔로우:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## 라이선스

MIT - 자유롭게 사용하고, 필요에 따라 수정하고, 가능하면 기여하세요.

---

**도움이 된다면 이 저장소에 별을 주세요. 두 가이드를 읽으세요. 멋진 것을 만드세요.**
